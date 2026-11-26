# AGENTS.md

给将要搭工程并实现本项目的 coding agent 的说明。先读完再动手。产品需求、架构与阶段范围分别见 [docs/requirements.md](docs/requirements.md)、[docs/architecture.md](docs/architecture.md)、[docs/phase-1.md](docs/phase-1.md)；已决定与未决定项见 [docs/decisions.md](docs/decisions.md)。

当前状态：文档已就绪，实现尚未开始。本目录里没有 Android / Gradle 工程，也没有实现代码。不要假设任何类、模块或仓库已经存在。

## 从哪里开始

1. 读本文件和上述 `docs/`，不要重新打开已决定的架构方向（除非遇到硬阻塞，并在文档中写明原因）。
2. 搭工程前，按 [docs/phase-1.md](docs/phase-1.md) 中的版本表，对照官方文档与 Maven **重新核对**版本号。表中版本的上次核对日期是 **2026-11-20**；官方文档示例当时可能落后于 Maven 稳定版，以再核对结果为准，并记下核对日期。
3. 先搭两个 Gradle 模块（`:container` 库、`:app` 宿主演示）和 `sample-packages/`（演示 zip，不是 Android 模块）。许可证用仓库根目录的 MIT `LICENSE`。拟定 `applicationId` 见 [docs/decisions.md](docs/decisions.md)，所有者尚未明确确认，采用时在相关文档里保持「拟定」标记，或先向所有者确认。
4. 按第一阶段顺序实现：**先做 URL 入口，用来验证最近任务独立卡片**（覆盖在线 PWA 与普通非 PWA 网站）；**再在同一容器上接包入口**。不要颠倒，也不要并行铺开第一阶段范围外的能力。

远端目标是 [https://github.com/AgentsRepos/shellapp](https://github.com/AgentsRepos/shellapp)，仓库可能尚不存在。本说明不授权创建、推送或克隆 Git 仓库；是否建仓、以谁的身份推送，由所有者或后续任务单独指示。

## 已经决定、不要改方向

- 仓库名就是 `shellapp`（一个词，壳 + 母应用），不要改名，也不要改成 kebab-case。
- 许可证 MIT；组织 [AgentsRepos](https://github.com/AgentsRepos)；仅安卓。
- 第一阶段必须同时具备：最近任务独立卡片、Web 渲染、包入口与 URL 入口共用同一容器。
- 最近任务使用 **document / task 模型**，不要改用「预注册 N 个 Activity / 进程 / taskAffinity」（微信 / hapjs / FinClip 常见的 3–5 槽位模型）。具体清单见 [docs/architecture.md](docs/architecture.md)：单一 `MiniProgramActivity`、`launchMode=standard`（**不是** `singleTask`）、`documentLaunchMode=intoExisting`、默认只带 `FLAG_ACTIVITY_NEW_DOCUMENT`、默认 **不加** `FLAG_ACTIVITY_MULTIPLE_TASK`、用 `setTaskDescription` 设置名称和图标、宿主保持普通非 document 任务。
- 渲染：系统 `android.webkit.WebView`，放在 `WebRenderer` 接口后面；本地包用 `androidx.webkit` 的 `WebViewAssetLoader` 以虚拟 https origin 提供，**不要用 `file://`**。第一阶段单进程，不内嵌 Chromium / Gecko，不用 Hilt / Koin。容器 UI 用 `ComponentActivity` + 系统 WebView（不要用 Compose 包裹的 WebView）；宿主 UI 可以用 Jetpack Compose。
- 包格式：自有的最小 zip + 与 Web App Manifest 对齐的元数据。不要用微信 wxapkg、快应用 rpk，也不要把已停止的 W3C MiniApp Packaging 目录布局（`manifest.json`、`app.js`、`app.css`、`pages/`）当作要求。包是不可变的版本化快照；包模式下不要依赖 Service Worker 做离线，也不要让 SW 自行更新。第一阶段不要把 `.wbn` / `.swbn` / TWA 做成容器。
- 从零实现，不要 fork 现有项目；不要依赖 FinClip / UniMP / hapjs；不要抄微信 / 支付宝 API 表面。

## 不要在第一阶段扩展

以下均不在第一阶段范围，未获所有者明确要求不要做：

- JS API / JS Bridge、账号体系、桌面图标、cookie / 存储隔离
- iOS 或其他平台
- 插件市场、内嵌 Chromium / Gecko 内核
- 微信 / 支付宝兼容
- `.wbn` / `.swbn`、包哈希 / 签名
- Hilt / Koin
- 默认加上 `FLAG_ACTIVITY_MULTIPLE_TASK`（除非日后明确要求多实例；加上后每次打开都会新卡片）
- 把 PWA 设成打开 URL 的前提，或把「完整 PWA」当成小程序标准（原因见 [docs/architecture.md](docs/architecture.md)）

## 如何对待本仓库与第三方代码

- 文中的「小程序」用中文；英文只用 “mini-program” 作为惯用对译，并说明它是中文语境下的托管应用形态，不是「一个小的程序」。不要把本项目宣传成微信 / 支付宝兼容。
- 参考实现只可借鉴思路，**整段复制不行**：
  - 最近任务 / WebView：[shiaho777/web-to-app](https://github.com/shiaho777/web-to-app)（Unlicense）最接近，另见官方 Recents 文档与 [googlearchive/android-DocumentCentricApps](https://github.com/googlearchive/android-DocumentCentricApps)（已归档）。生产环境 `TaskDescription` 可参考 Firefox Installed Web Apps。
  - [Klein-ops/MiniAppContainer](https://github.com/Klein-ops/MiniAppContainer) 架构相近，但是 **GPL-3.0，禁止复制其代码**。本仓库是 MIT，不能引入 GPL 源码或由 GPL 源码改出来的实现。
  - Dimina、OpenMini 等可作为调研对照，不要当依赖，也不要搬它们的任务模型或微信对齐 API。细节见 [docs/architecture.md](docs/architecture.md)。
- 不要发明本文和 `docs/` 未写明的 API、版本号或「已实现」的事实。版本以再核对后的官方来源为准。

## 测试预期

- 搭工程后应有针对 Intent / document task 行为的单元测试（Robolectric + JUnit + AndroidX Test）：例如同一 `mp://…` URI 再次打开应进入已有任务（`intoExisting`），默认不加 `MULTIPLE_TASK` 时不应每次打开都新建卡片，宿主任务与小程序 document 任务分离。测试应覆盖拟定的 URI 形态 `mp://app/{id}` 与 `mp://url/{encoded}`。
- 最近任务在真机（尤其小米、华为、OPPO、vivo 等 OEM 桌面）上的呈现，是后续必做的验收，不是第一阶段搭工程当天就能用单测替代的。OEM 可能按包名合并任务并忽略 `TaskDescription`。公开线索：DCloud 论坛有 vivo / OriginOS 把宿主和小程序并成一个窗口的报告（https://ask.dcloud.net.cn/question/221170）。`setTaskDescription` 在部分环境会抛异常（web-to-app 相关讨论中有记录），实现时须防护，不要让异常打崩 Activity。
- Compose `@Preview` 或 [Kakao Prevue](https://github.com/kakao/prevue)（仅 macOS，偏 layoutlib）只适合迭代宿主 Compose 界面，**不能**用来验收最近任务、document task 或系统 WebView。这些需要真机（优先，尤其 OEM 最近任务）或模拟器。

## 文档与实现不同步时

实现过程中若发现硬阻塞、必须偏离已决定项，或核对出版本与 2026-11-20 记录不一致：先改 `docs/` 和本文件中的对应陈述，再改代码，并写明原因与来源。不要静默改架构。拟定项（如 `applicationId`、Intent data URI 方案）被所有者确认或修改后，同步更新 [docs/decisions.md](docs/decisions.md)。
