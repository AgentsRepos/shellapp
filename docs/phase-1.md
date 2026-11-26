# 第一阶段

范围对齐 [requirements.md](requirements.md)，实现方式对齐 [architecture.md](architecture.md)。给实现用 agent 的约束见 [../AGENTS.md](../AGENTS.md)。

**现状：** 文档已就绪，实现尚未开始。

## 范围

必须交付：

1. 最近任务独立卡片（document / task 模型，见架构文档）。
2. 系统 WebView 渲染 Web 页面（经 `WebRenderer`）。
3. URL 入口（在线 PWA 与普通非 PWA 网站）与包入口（本地或下载），同一容器、同一套最近任务呈现。

模块：`:container`、`:app`、以及 `sample-packages/` 演示包。

不做的项以 [requirements.md](requirements.md) 的「非目标」和 [decisions.md](decisions.md) 为准，包括但不限于：JS Bridge、账号、桌面图标、存储隔离、iOS、插件市场、内嵌浏览器内核、微信 / 支付宝兼容、`.wbn` / `.swbn`、包签名、Hilt / Koin、默认 `MULTIPLE_TASK`。

## 计划中的技术栈与版本

下列版本的上次核对日期是 **2026-11-20**，对照过官方文档与 Maven。**搭工程前必须再核对**，不要直接假定仍是当前稳定版，也不要自行发明更新的版本号。若再核对结果与下表不同，以官方稳定来源为准，并在本文记下新的核对日期与差异。

核对当时已知的文档滞后：

- Maven 上 AGP 稳定版为 **9.5.1**，当时官方 AGP 文档示例仍写 **9.5.0**。
- Compose BOM **2026.11.01** 于 **2026-11-19** 发布到 Maven；当时官方 Compose BOM 文档页的示例仍写 **2026.09.00**。`compileSdk 37.1` 是该 BOM 的要求，不要为了迁就旧文档示例而降 BOM 或改低 `compileSdk`，除非再核对表明要求已变。

| 组件 | 计划版本（2026-11-20 记录） | 备注 |
| --- | --- | --- |
| Kotlin | 2.4.21 | |
| Android Gradle Plugin | 9.5.1 | Maven 稳定版；官方示例当时仍为 9.5.0 |
| Gradle | 9.6.0 | |
| JDK | 17 | |
| 构建脚本 | Gradle Kotlin DSL + Version Catalog | |
| minSdk | 26 | |
| targetSdk | 37 | |
| compileSdk | 37.1 | Compose BOM 2026.11.01 所需 |
| 宿主 UI | Jetpack Compose + Material 3 | |
| Compose BOM | 2026.11.01 | 2026-11-19 发布于 Maven |
| Coroutines | 1.11.0 | |
| kotlinx.serialization | 1.13.0 | |
| OkHttp | 5.5.1 | |
| androidx.webkit | 1.18.0 | 不要在 v1 使用当时仍为 alpha 的 1.19 isolated-sites API |
| AndroidX Activity | 1.13.0 | |
| 测试 | Robolectric 4.18.1 + JUnit + AndroidX Test | 用于 Intent / document task 测试 |
| Zip / 图标 | 仅 JDK / Android API | 不另引打包库 |

第一阶段不用 Hilt / Koin，不内嵌 Chromium / Gecko。

## 实现顺序

1. **先做 URL 入口**，用来证明最近任务独立卡片成立（同时覆盖在线 PWA 与普通非 PWA 网站：Manifest 或标题 / favicon，以及回退）。
2. **再在同一容器上接包入口**（最小 zip + Manifest 对齐元数据，`WebViewAssetLoader` 虚拟 https origin，演示包放到 `sample-packages/`）。

不要先做包格式再回头补 Recents，也不要在卡片行为未用 URL 验证之前铺开 JS API 或其他非目标。

建议的内部顺序（仍服从上面两条）：先搭 Gradle 工程与空的 `MiniProgramActivity` / 任务启动路径，再接系统 WebView，再补 Recents 元数据（`setTaskDescription` 占位后异步更新图标），然后做 URL 元数据解析，最后做包加载。自动化测试随任务启动路径一起加，不要等全部功能完成再补。

## 验收

实现完成后，至少检查：

1. **URL / PWA：** 打开一个提供 Web App Manifest 的在线站点，最近任务卡片使用 Manifest 中的名称与图标（图标可异步更新）。
2. **URL / 非 PWA：** 打开一个没有 Manifest 的普通网站，卡片使用页面标题与 favicon，或回退到 URL / 默认图标；站点没有 Service Worker 也能打开。
3. **同一条目复用任务：** 再次打开同一 `mp://url/{encoded}`（或同一 `mp://app/{id}`），进入已有任务，而不是默认新建卡片。
4. **与宿主分离：** 宿主自己的任务仍是普通非 document 任务；小程序卡片不并入宿主卡片（在 AOSP / 模拟器上应如此。OEM 真机见下方风险）。
5. **包入口：** 打开 `sample-packages/` 中的演示包，进入同一 `MiniProgramActivity` 容器，以包元数据中的名称与图标显示独立卡片；资源经虚拟 https origin 加载，不使用 `file://`。
6. **自动化测试：** Robolectric / AndroidX Test 覆盖 Intent data URI、`NEW_DOCUMENT` 且默认无 `MULTIPLE_TASK`、`intoExisting` 复用行为。
7. **OEM 真机（必做，但可能滞后于代码合并）：** 在小米、华为、OPPO、vivo 等设备上目视最近任务。若被按包名合并，记录机型、系统版本与现象，不要在未验证的情况下宣称「所有设备均独立卡片」。

Compose 预览或 Prevue 不能替代上述与最近任务、document task 或系统 WebView 相关的检查。

## 风险

- **OEM 最近任务分组：** 可能忽略 `TaskDescription`、按包名合并窗口。公开线索：https://ask.dcloud.net.cn/question/221170（vivo / OriginOS）。真机验证不可省。
- **`setTaskDescription` 抛异常：** 部分环境会失败；必须捕获，保证 Activity 不崩溃，并尽量保留已设置的占位标题。
- **单进程存储：** `WebView.setDataDirectorySuffix` 不能按小程序隔离；第一阶段接受这一限制，不要在本阶段改成多进程或上 alpha 的 isolated-sites API 来「顺便解决」。
- **版本漂移：** 2026-11-20 的版本记录可能过时；官方文档示例当时已落后于 Maven。搭工程前再核对。
- **Intent URI 与 `applicationId` 仍为拟定：** 见 [decisions.md](decisions.md)。改 URI 方案会影响任务复用，须同步测试与文档。
