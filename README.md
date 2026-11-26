# shellapp

A dedicated Android **shell / host app** (壳 / 母 App, 宿主) for **小程序**.

**Status:** documentation is ready; implementation has not started. This tree currently contains only planning documents so other agents can scaffold and implement from them.

- **License:** [MIT](LICENSE)
- **Organization:** [AgentsRepos](https://github.com/AgentsRepos)
- **Proposed `applicationId`:** `io.github.agentsrepos.shellapp` *(proposed; the owner has not explicitly confirmed it)*

Intended GitHub location: [https://github.com/AgentsRepos/shellapp](https://github.com/AgentsRepos/shellapp). The remote repository may not exist yet; these files are the docs drop to commit later. Do not treat this tree as an already-published or already-implemented project.

## What “小程序” means here

In this project, **小程序** is the Chinese hosted-app category: it runs inside a host, can be opened from a package or a URL, and appears in the system Recents / overview like an independent app (its own name and icon). The English phrase “mini-program” is only a conventional gloss. It does **not** mean “a small program,” and this project is **not** a WeChat or Alipay compatibility layer.

The repository name `shellapp` is one word (shell + host / mother app). It was chosen by the owner and must not be renamed. The name itself is generic; the 小程序-host purpose is documented here and in `docs/`.

## Phase 1 capabilities

Phase 1 includes all three of the following:

1. **Independent Recents cards** — each opened 小程序 is its own task card, not merged into the host app’s single task. Use Android’s document / task model correctly.
2. **Web rendering** — the 小程序 layer renders Web pages.
3. **Two entry modes, one container** — a 小程序 package (local or downloaded) or a Web URL. Both use the same container and the same Recents presentation, and should feel like independent apps. URL entry must support both live PWAs and ordinary non-PWA websites. A PWA is not a prerequisite to open a URL.

Android only. Do not expand to iOS or other platforms unless the owner explicitly asks later.

## Documents

| Document | Contents |
| --- | --- |
| [AGENTS.md](AGENTS.md) | Instructions for implementing agents |
| [docs/requirements.md](docs/requirements.md) | Goals, must-haves, entry modes, non-goals, success criteria |
| [docs/architecture.md](docs/architecture.md) | Task model, rendering, package format, URL entry, modules |
| [docs/phase-1.md](docs/phase-1.md) | Phase 1 scope, stack and versions, implementation order, acceptance, risks |
| [docs/decisions.md](docs/decisions.md) | Decided items vs still-open items |

Planned toolchain versions were last checked on 2026-11-20 against official docs and Maven. Recheck before scaffolding. See [docs/phase-1.md](docs/phase-1.md).

---

# shellapp

面向 **小程序** 的安卓专用 **壳 / 母 App（宿主）**。

**现状：** 文档已就绪，实现尚未开始。当前目录只有规划文档，供其他 agent 据此搭工程并实现。

- **许可证：** [MIT](LICENSE)
- **组织：** [AgentsRepos](https://github.com/AgentsRepos)
- **拟定 `applicationId`：** `io.github.agentsrepos.shellapp`（*拟定，所有者尚未明确确认*）

预定的 GitHub 位置：[https://github.com/AgentsRepos/shellapp](https://github.com/AgentsRepos/shellapp)。远端仓库可能尚不存在；这些文件是稍后提交用的文档包。不要把本目录当成已经发布或已经实现的项目。

## 这里的「小程序」指什么

本项目中的 **小程序** 指中文语境下的托管应用形态：在宿主内运行，可以用包或 URL 打开，并在系统最近任务里像独立应用一样出现（自己的名称和图标）。英文 “mini-program” 只是惯用对译，**不是**「一个小的程序」，本项目也 **不是** 微信 / 支付宝兼容层。

仓库名 `shellapp` 是一个词（壳 + 母应用），由所有者选定，不得改名。这个名字本身比较泛，小程序专用壳的定位由本文和 `docs/` 说明。

## 第一阶段能力

第一阶段须同时具备以下三项：

1. **最近任务独立卡片** — 每个打开的小程序是独立任务卡片，不并入宿主的单一任务。须正确使用 Android 的 document / task 模型。
2. **Web 渲染** — 小程序层渲染网页。
3. **两种入口、同一容器** — 小程序包（本地或下载）或 Web URL。两者使用同一套容器和同一套最近任务呈现，体感应接近独立应用。URL 入口必须同时支持在线 PWA 和普通非 PWA 网站。打开 URL 不以 PWA 为前提。

仅安卓。除非所有者日后明确要求，不要扩展到 iOS 或其他平台。

## 文档

| 文档 | 内容 |
| --- | --- |
| [AGENTS.md](AGENTS.md) | 给实现用 agent 的说明 |
| [docs/requirements.md](docs/requirements.md) | 目标、必备能力、入口、非目标、成功标准 |
| [docs/architecture.md](docs/architecture.md) | 任务模型、渲染、包格式、URL 入口、模块划分 |
| [docs/phase-1.md](docs/phase-1.md) | 第一阶段范围、技术栈与版本、实现顺序、验收、风险 |
| [docs/decisions.md](docs/decisions.md) | 已决定项与仍开放项 |

计划中的工具链版本上次核对日期为 2026-11-20（对照官方文档与 Maven）。搭工程前须再核对。见 [docs/phase-1.md](docs/phase-1.md)。
