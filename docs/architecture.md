# 架构

本文给出已约定的实现方向。需求与非目标见 [requirements.md](requirements.md)；第一阶段顺序、版本与风险见 [phase-1.md](phase-1.md)；决定状态见 [decisions.md](decisions.md)。

除非遇到硬阻塞，不要重新打开这些方向。当前没有实现代码，下文中的类型名与模块名是规划约定，不是已经存在的工程。

## 模块划分

两个 Gradle 模块，外加演示包目录：

| 路径 | 角色 |
| --- | --- |
| `:container` | 库：启动、会话、`ui/MiniProgramActivity`、渲染、包、任务（Recents / document task） |
| `:app` | 宿主演示应用 |
| `sample-packages/` | 演示用 zip，**不是** Android 模块 |

`:app` 依赖 `:container`，用演示入口打开 URL 或包。业务逻辑放在 `:container`，便于日后被其他宿主引用；第一阶段不必先做成对外发布的 SDK。

## 最近任务 / document task

目标：每个打开的小程序一张独立卡片（自己的名称和图标）；宿主仍是普通的非 document 任务。只影响最近任务，不涉及桌面图标。

已约定的用法：

- 只有一个 `MiniProgramActivity`，`launchMode` 为 **`standard`**，**不要**用 `singleTask`。
- `documentLaunchMode` 为 **`intoExisting`**。
- 启动时带 `FLAG_ACTIVITY_NEW_DOCUMENT`。默认 **不要**加 `FLAG_ACTIVITY_MULTIPLE_TASK`：加上之后每次打开都会新建卡片。仅当所有者日后明确要求多实例时再考虑。
- 用唯一的 Intent data URI 区分文档。拟定方案（尚未与 `applicationId` 一并经所有者书面确认，但实现应按此起步，更改须同步文档）：
  - 包：`mp://app/{id}`
  - URL：`mp://url/{encoded}`
- 用 `Activity.setTaskDescription` 设置卡片名称与图标。API 33+ 使用 `TaskDescription.Builder`；先设占位名称/图标，图标异步就绪后再更新。
- `maxRecents` 取一个有上限的值，规划中的例子是 **20**；实现时可按此起步，改动须在文档中写明。
- 不要混用另一套历史模型：预先注册 N 个 Activity / 进程 / `taskAffinity`（微信 / hapjs / FinClip 一类，通常只有 3–5 个槽位）。

思路参考（可借鉴，不要整仓搬来）：

- [shiaho777/web-to-app](https://github.com/shiaho777/web-to-app)（Unlicense）：`intoExisting`、默认无 `MULTIPLE_TASK`、`maxRecents`、先标题后图标、有测试。这是最接近的 Recents / WebView 参考。
- 官方 Recents / document-centric 文档，以及已归档的 [googlearchive/android-DocumentCentricApps](https://github.com/googlearchive/android-DocumentCentricApps)。
- Firefox Installed Web Apps：生产环境中使用 `TaskDescription` 的参考。

### OEM 与运行时风险

这套 API 只保证改变最近任务卡片的**意图**；部分 OEM 最近任务（小米、华为、OPPO、vivo 等）可能按包名分组，并忽略 `TaskDescription`。公开线索：DCloud 论坛有报告称 vivo / OriginOS 会把宿主和小程序并成一个窗口（https://ask.dcloud.net.cn/question/221170）。必须在真机上验证，不能只靠模拟器或官方行为说明。

另：web-to-app 相关讨论表明，`setTaskDescription` 在部分环境会抛异常。调用须捕获，避免打崩 Activity；不要假设所有设备上都能成功改标题和图标。

## 渲染

- 系统 `android.webkit.WebView`，抽象在 `WebRenderer` 接口之后，便于日后替换实现，但第一阶段只有系统 WebView。
- 本地包用 `androidx.webkit.WebViewAssetLoader` 映射为**虚拟 https origin**，不要用 `file://`。
- `WebView.setDataDirectorySuffix` 是进程级、且只能设置一次，不能用来按小程序隔离。Cookie / 存储隔离是后续工作。
- 第一阶段单进程。
- 第一阶段不内嵌 Chromium 或 Gecko。
- 第一阶段不引入 Hilt / Koin。
- 容器 UI：`ComponentActivity` + 系统 WebView，**不要**用 Compose 包裹的 WebView。宿主 UI 可以使用 Jetpack Compose。

`androidx.webkit` 1.19 的 isolated-sites API 在上次核对此事时仍为 alpha，第一阶段不要用，可以继续关注。版本号的核对日期与约束见 [phase-1.md](phase-1.md)。

开发期可用 Compose `@Preview` 或 [Kakao Prevue](https://github.com/kakao/prevue)（macOS，偏 layoutlib）迭代宿主 Compose 界面。它们不能验收最近任务、document task 或系统 WebView；这些需要真机（优先）或模拟器。

## 包格式

自有的最小 **zip + 与 Web App Manifest 对齐的元数据**。这是本项目的「打包 PWA」配置（Manifest + 静态 zip），不是微信 / 支付宝包格式。

元数据至少包括：`id`、`name`、`icons`、`start_url` / 入口、`version`，以及 `display`、`theme_color`。字段语义向 [Web App Manifest](https://www.w3.org/TR/appmanifest/) 对齐，不要另造一套平行字段名，除非文档明确记录差异。

明确不是：

- 微信 `wxapkg`
- 快应用 `rpk`
- 已停止的 W3C MiniApp Packaging 布局。该草案要求 `manifest.json`、`app.js`、`app.css`、`pages/`——**不要把这套目录布局当作本项目的要求**。
- 第一阶段不要把 `.wbn` / `.swbn` 或 TWA 当作容器来实现。

包是**不可变的版本化快照**。包模式下不要依赖 Service Worker 做离线，也不要让 Service Worker 自行更新（与不可变快照冲突）。哈希与签名是后续增强，第一阶段不做。

zip 与图标处理只用 JDK / Android API，不额外引入打包库。

## URL 入口

与包入口使用**同一容器、同一套最近任务呈现**。

- 在线 PWA：读取 Web App Manifest，用于最近任务的名称与图标。
- 普通非 PWA 网站：用页面标题与 favicon；没有则回退到 URL 和默认图标。
- 在线站点的 Service Worker 是可选增强，不是打开的前提，也不是把站点当成小程序的前提。

## 为什么不把完整 PWA 当作小程序标准

记录在此，不要在实现里对抗或「纠正」这一产品选择。

PWA 的模型是「把一个网站安装成应用」，绑定 HTTPS 源、在线服务器和 Service Worker 缓存；它不是一套可版本化、可审核、由宿主分发、且不必拥有公开源的多应用包格式。TWA / Digital Asset Links 以及与 Chrome 共享的存储，适合第一方 PWA，不适合一个承载第三方小程序的宿主。

相关标准与实验状态（均为本项目调研结论，实现时不要假定它们已经可在系统 WebView 里当容器用）：

- Chrome 实验性的 Web Bundle 导航已于 **2023 年 2 月**移除；系统 WebView 不能原生打开 `.wbn` / `.swbn`。
- Isolated Web Apps 截至 **2026 年 9 月**仍主要在 ChromeOS 上，没有面向 Android / WebView 的编译形态可当作第一阶段依赖。
- W3C Widgets（zip 打包的 Web 应用）已于 **2018 年**废弃，方向转向 Manifest + Service Worker。
- 2026 年的 PortableWeb（`.pweb`，`application/vnd.portableweb+zip`）是 Community Group 的预发布草案，形态相近，可以关注，但第一阶段**不要**把兼容它写成要求。

## 标准现状（本项目 2026 年调研）

- W3C MiniApps 的 Packaging / Manifest / Lifecycle 已于 **2026-08-11** 变为 Discontinued Draft，引用时只作为已放弃的工作，不要当现行规范实现。
- MiniApps Working Group 已于 **2026-08-26** 关闭；后续讨论指向 WebView Community Group 以及仍在孵化的 “embedded web apps”。
- 业界没有仍在广泛实施的共享包格式；微信 / 支付宝 / 百度 / 字节的格式是专有且互不兼容的。
- 目前仍广泛实现、且本项目应对齐的部分是 [Web App Manifest](https://www.w3.org/TR/appmanifest/)。

## 现有项目：从零做，不 fork

调研结论：本项目从零实现，不把下列项目当作依赖或起点。

- 最接近：[Klein-ops/MiniAppContainer](https://github.com/Klein-ops/MiniAppContainer)，架构相似，但许可证是 **GPL-3.0，禁止复制其代码**。项目很新，使用 `file://`，没有一等的在线 URL 入口，也不是可嵌入 SDK。本仓库是 MIT，不能引入 GPL 源码。
- [Dimina](https://github.com/didi/dimina)（Apache-2.0，活跃）走微信对齐，使用 `NEW_TASK | MULTIPLE_TASK` 再自己做恢复，主源码中未见 `setTaskDescription`。过重，且任务模型不符合本项目约定。
- FinClip / UniMP / hapjs：槽位有限，核心往往闭源，模型不同。不要依赖。
- OpenMini（Apache-2.0，React Native，带哈希校验的 `.mpkg`、AssetLoader、CSP）有可参考的想法，但绑定 RN 且很新——只借鉴思路，不当依赖。
- 不要抄微信 / 支付宝的 API 表面。
