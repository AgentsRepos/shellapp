# 需求

本文只定义要做成什么、什么算成功。任务模型、渲染与包格式的实现约定见 [architecture.md](architecture.md)；第一阶段范围、技术栈与验收步骤见 [phase-1.md](phase-1.md)；已决定与未决定项见 [decisions.md](decisions.md)。

**现状：** 规划已足以开始实现；本文对应的代码尚未编写。

## 目标

做一款安卓上的小程序 **专用壳 / 母 App（宿主）**：在宿主里打开多个小程序，每个小程序在系统最近任务中以独立应用的形态出现（自己的名称和图标），内容用 Web 页面呈现。入口可以是包，也可以是 URL，两者必须走同一套容器、同一套最近任务呈现。

这里的「小程序」指中文语境下的托管应用形态（宿主内运行、可以装包或用 URL 打开、在系统最近任务里像独立应用），不是「一个小的程序」，也不是微信 / 支付宝小程序的兼容实现。对外英文可用 “mini-program” 作为惯用对译，并保留上述含义说明。

仅安卓。除非所有者明确要求，不扩展到 iOS 或其他平台。

## 必备能力（第一阶段须全部具备）

1. **最近任务独立卡片**  
   每个已打开的小程序必须在系统最近任务 / 概览中显示为独立任务卡片，使用自己的名称和图标，不得并入宿主的单一任务。须正确使用 Android 的 document / task 模型（具体 API 与约束见 [architecture.md](architecture.md)）。这是第一阶段的明确目标，不是后续增强。  
   本能力只改变最近任务卡片，**不包括**在桌面创建图标。

2. **Web 页面渲染**  
   小程序层必须能渲染 Web 页面。

3. **包入口与 URL 入口共用同一容器**  
   - 包：本地已有，或下载后再打开。  
   - URL：必须同时支持 **在线 PWA** 和 **普通非 PWA 网站**。PWA 不是打开 URL 的前提。  
   两种入口使用同一套容器和同一套最近任务呈现，体感应接近独立应用。

## 入口模式

| 入口 | 第一阶段要求 |
| --- | --- |
| 小程序包（本地或下载） | 打开后进入与 URL 入口相同的容器；用包内与 Web App Manifest 对齐的元数据提供最近任务名称与图标（字段与包形态见 [architecture.md](architecture.md)）。 |
| 在线 PWA（URL） | 读取 Web App Manifest，用于最近任务名称与图标。 |
| 普通非 PWA 网站（URL） | 使用页面标题与 favicon；缺失时回退到 URL 本身和默认图标。不得因站点没有 Manifest 或 Service Worker 而拒绝打开。 |

在线站点上的 Service Worker 是可选增强，不是打开的前提。包模式与「完整 PWA（源站 + Service Worker 缓存）」不是同一件事，不要把后者当成小程序标准，说明见 [architecture.md](architecture.md)。

## 非目标（第一阶段不做）

- JS API / JS Bridge、账号体系
- 桌面图标
- Cookie / 存储按小程序隔离（第一阶段单进程；`WebView.setDataDirectorySuffix` 的限制见 [architecture.md](architecture.md)）
- iOS 或其他平台
- 插件市场、内嵌 Chromium / Gecko 内核
- 微信 / 支付宝兼容，或复制其 API 表面
- 以 `.wbn` / `.swbn` 或 TWA 作为容器
- 包的哈希校验与签名（列为后续增强）
- 引入 Hilt / Koin
- 默认启用 `FLAG_ACTIVITY_MULTIPLE_TASK` 的多实例（除非日后明确要求）
- Fork 或依赖 FinClip、UniMP、hapjs 等现有宿主；不要把 GPL 项目的源码带进本仓库

## 成功标准

第一阶段在实现与文档对齐的前提下，至少满足：

1. 从宿主分别打开两个不同的 URL 小程序（建议各选一个在线 PWA 和一个普通非 PWA 网站），最近任务中出现两张独立卡片，名称与图标分别对应该条目，且不与宿主自身的任务卡片合并。同一条目再次打开时，进入已有任务，而不是默认再堆一张新卡片。
2. 用包入口打开一个符合本项目最小包格式的演示包，进入与 URL 入口相同的容器，最近任务同样以该包的名称与图标显示为独立卡片。
3. 页面由系统 WebView 渲染；本地包不以 `file://` 作为对外 origin，而走虚拟 https origin。
4. 有覆盖 document task / Intent 行为的自动化测试（见 [../AGENTS.md](../AGENTS.md)）。OEM 真机上的最近任务表现列为必测风险项，通过与否需在真机上记录，不能仅靠模拟器或 Compose 预览宣布完成。

拟定的 `applicationId` 与 Intent URI 方案见 [decisions.md](decisions.md)，其中 `applicationId` 尚未经所有者明确确认。
