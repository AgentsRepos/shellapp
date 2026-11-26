# 决定记录

区分已经拍板、实现时直接执行的事项，以及仍开放、需要标记或再确认的事项。需求见 [requirements.md](requirements.md)，实现约定见 [architecture.md](architecture.md)，阶段范围见 [phase-1.md](phase-1.md)。

## 已决定

| 项 | 决定 |
| --- | --- |
| 组织 | [AgentsRepos](https://github.com/AgentsRepos) |
| 仓库名 | `shellapp`（一个词，所有者选定：壳 + 母应用）。不要改名，也不要为了与同组织 [mingle-cli](https://github.com/AgentsRepos/mingle-cli) 的 kebab-case 习惯而对齐成 `shell-app`。 |
| 许可证 | MIT。版权归属 AgentsRepos and contributors，年份 2026。见仓库根目录 [LICENSE](../LICENSE)。 |
| 平台 | 仅安卓。未经所有者明确要求，不扩展到 iOS 或其他平台。 |
| 产品定位 | 小程序专用壳 / 母 App（宿主）。文中优先用「小程序」；英文 “mini-program” 只作惯用对译。不做成微信 / 支付宝兼容，不在仓库名里把小程序译成 miniprogram。 |
| 第一阶段能力 | 必须同时具备：最近任务独立卡片、Web 渲染、包入口与 URL 入口共用同一容器。URL 须支持在线 PWA 与普通非 PWA 网站；打开 URL 不以 PWA 为前提。 |
| 实现策略 | 从零实现，不 fork 现有宿主；不依赖 FinClip / UniMP / hapjs；不复制 GPL 源码（包括 Klein-ops/MiniAppContainer）。 |
| 任务模型 | 单一 `MiniProgramActivity`，`launchMode=standard`（不是 `singleTask`），`documentLaunchMode=intoExisting`，启动带 `FLAG_ACTIVITY_NEW_DOCUMENT`，默认不加 `FLAG_ACTIVITY_MULTIPLE_TASK`，用 `setTaskDescription`（API 33+ 用 `TaskDescription.Builder`），宿主为普通非 document 任务。不用预注册 N 个 Activity / 进程 / `taskAffinity` 的槽位模型。桌面图标暂不在范围。 |
| 渲染 | 系统 `android.webkit.WebView` + `WebRenderer` 接口；本地包用 `WebViewAssetLoader` 虚拟 https origin（不用 `file://`）；第一阶段单进程、不内嵌 Chromium / Gecko、不用 Hilt / Koin；容器 UI 为 `ComponentActivity` + 系统 WebView（不用 Compose 包裹的 WebView）；宿主 UI 可用 Compose。 |
| 包形态 | 自有最小 zip + 与 Web App Manifest 对齐的元数据（id、name、icons、start_url/入口、version，以及 display、theme_color）。不是 wxapkg、不是 rpk、不把已停止的 W3C MiniApp Packaging 目录布局当作要求。包为不可变版本化快照；包模式不依赖 SW 离线或 SW 自更新。第一阶段不以 `.wbn` / `.swbn` / TWA 为容器。 |
| 模块 | `:container`（库）、`:app`（宿主演示）、`sample-packages/`（演示 zip，非 Android 模块）。 |
| 实现顺序 | 先 URL 入口验证最近任务卡片（PWA 与非 PWA），再在同一容器接包入口。 |
| 技术栈家族 | Kotlin、AGP、Gradle Kotlin DSL + Version Catalog、JDK 17、Jetpack Compose + Material 3、Coroutines、kotlinx.serialization、OkHttp、androidx.webkit、AndroidX Activity、Robolectric + JUnit + AndroidX Test。具体版本见 [phase-1.md](phase-1.md)，须在搭工程前再核对。 |

## 拟定（按此起步，但须保持标记，直到所有者确认或修改）

| 项 | 拟定值 | 说明 |
| --- | --- | --- |
| `applicationId` | `io.github.agentsrepos.shellapp` | 在没有自有域名的前提下提出，所有者**尚未明确确认**。README 与实现中均须标为拟定，或先向所有者确认后再写入正式构建配置并去掉「拟定」字样。 |
| Intent data URI | 包：`mp://app/{id}`；URL：`mp://url/{encoded}` | 为实现 document 复用所提出的方案。任务模型本身已决定，URI 字符串可以在实现前或实现中调整，但须同步测试与文档。 |
| `maxRecents` | 例如 20 | 需要有上限；20 是规划中的示例起点，不是不可改的产品承诺。 |

## 仍开放 / 明确延后

- 版权与仓库元信息以外的发布细节（GitHub 仓库是否已创建、谁来建仓推送）由所有者或后续任务处理；本文档包本身不创建远端仓库。
- Cookie / 存储按小程序隔离；`WebView.setDataDirectorySuffix` 无法在单进程内按实例隔离。
- 包的哈希与签名。
- 多实例（`FLAG_ACTIVITY_MULTIPLE_TASK`）——仅当所有者日后明确要求时再做。
- JS API / JS Bridge、账号体系、桌面图标、插件市场。
- `androidx.webkit` isolated-sites（1.19 在上次核对时仍为 alpha）是否进入后续版本。
- PortableWeb（`.pweb`）等草案是否需要兼容——第一阶段不要求，只关注。
- OEM 最近任务若按包名合并，第一阶段先验证并记录；是否做机型规避或产品层说明，待有真机结果后再决定，现在不要发明规避方案。

## 刻意不决定成「小程序标准」的事项

完整 PWA（HTTPS 源 + 在线服务器 + Service Worker）、TWA、Web Bundle、Isolated Web Apps、W3C MiniApps（已于 2026-08 停止）均不作为本项目的小程序标准。原因见 [architecture.md](architecture.md)。URL 入口会利用 Manifest（若有）或标题 / favicon，这不等于把 PWA 安装模型当成宿主协议。
