# Architect-Flash · 架构师模式（Pro+Flash）

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![DeepSeek Harness](https://img.shields.io/badge/DeepSeek_Harness-agent_preset-4B32C3)](https://github.com/deepseek-ai/dsh)

> **Cut your coding-token bill.** A DeepSeek Harness agent preset that plans and
> reviews on your main model, and hands every low-level code implementation to a
> cheap Flash sub-agent — so the expensive model never writes code.

---

## English

### Why Architect-Flash

- 💸 **Save tokens and cost** — code implementation is the token-heavy part of
  any coding task; here it runs on a cheap Flash sub-model, not your main model.
- 🧠 **Clean separation of concerns** — one *architect* plans, designs, and
  reviews; one *worker* implements. No blurred responsibilities.
- 🔒 **Isolated context** — the worker edits files in its own context, keeping
  the architect's context lean and cache-friendly.
- 🔌 **No hardcoded model** — the worker follows the sub-model you select in the
  sub-agent settings, so the preset is portable across deployments.
- 🧰 **Full native tooling** — the standard coding catalog, with a directly
  callable `code_flash` delegation tool.

### Architecture

```
┌────────────────────────────────────────────────────────────┐
│  Architect  (main model, e.g. deepseek-v4-pro)            │
│  plan · design · review · git · verify builds             │
└────────────────────────────┬───────────────────────────────┘
                             │  delegate to code_flash
                             ▼
┌────────────────────────────────────────────────────────────┐
│  code_flash worker  (selected sub-model, e.g. Flash)      │
│  read · write · edit · glob · grep · pwsh · todo_write    │
│  isolated context · maxDepth 1 (no recursion)             │
└────────────────────────────────────────────────────────────┘
```

### How it compares

| | `standard` | `ptc` (PTC mode) | **`architect-flash`** |
| --- | --- | --- | --- |
| Tool presentation | native | single `run_code` | native |
| Model tiering | single model | single model | **Pro (main) + Flash (worker)** |
| Who writes code | the model itself | the model itself (in `run_code`) | **the Flash worker** |
| Delegation tool | `subagent` | hidden behind `run_code` SDK | **`code_flash`, direct** |
| Token cost | full price on one model | full price on one model | **code runs on the cheaper model** |
| Cache/schema | medium | best (tiny schema) | medium (but cheaper model) |

PTC mode optimizes schema and round-trips; Architect-Flash optimizes *which
model does the heavy work*. If your code implementation dominates the token
spend, the Flash split usually saves more than the PTC schema savings.

### Quick start

```sh
# Linux / macOS
cp -r architect-flash "$HOME/.dsh/.agent-presets/architect-flash"

# Windows (PowerShell)
Copy-Item -Recurse architect-flash "$env:USERPROFILE\.dsh\.agent-presets\architect-flash"
```

Restart the host, start a new session, and select **Architect-Flash（架构师模式）**.

### Requirements

- DeepSeek Harness with the `dsh-agent-tool-presentation` architecture
  (`native` / `ptc` / `both`) and `@deepseek-ai/dsh-tool-subagent` with
  `modelSelectionSettings`.
- A Flash-capable LLM route registered under the session's
  `subagent-model-selection.allowedModels` policy. `code_flash` hardcodes no
  route — it follows the sub-agent's selected sub-model.

### Selecting the sub-model

The sub-model is a session-level setting, not a preset knob. Register the routes
the `code_flash` worker may use under `subagent-model-selection.allowedModels` in
`settings.yaml`:

```yaml
subagent-model-selection:
  enabled: true
  allowedModels:
    - provider: ark
      model: deepseek-v4-flash
```

`code_flash` uses `modelSelectionSettings: true` to follow this policy. The
architect discovers the sub-model with `list_subagent_models` and passes its
`provider`/`model`, so the sub-model is whatever the sub-agent settings select.

### Behavior contract

The persona encodes two rules:

1. *For code implementation — especially repetitive or low-level code —
   delegate to the `code_flash` worker on the selected sub-model instead of
   writing it inline.*
2. *While a delegated worker is running, do not poll with goal updates; stop
   and wait for its result to be injected.*

These are behavioral (the persona), not physical. To make "write code" a single
physical path, remove `write`/`edit` from the main agent's tools — but the
architect still needs them for documentation (requirements/bug tables), so the
code-vs-docs boundary is best kept as a workspace instruction.

### FAQ

**Why native instead of PTC?** PTC collapses all tools into one `run_code` tool
and hides the delegation tool behind its SDK — a distinct, directly callable
`code_flash` cannot exist there. Native keeps `code_flash` first-class.

**Which sub-model should I pick?** Any cheaper reasoning-capable model your
provider offers (for example `deepseek-v4-flash`). Register it under
`subagent-model-selection.allowedModels`.

**How do I force *all* code into the worker?** Remove `write`/`edit` from the
main agent's catalog. Then documentation edits also go through the worker, so
decide whether you want that.

**Does it work on Windows?** Yes. On Windows the shell tool is `pwsh`; on other
platforms it is `bash`. The `code_flash` tool filter names the right one.

### Files

| File | Purpose |
| --- | --- |
| `agent.cordis.yml` | The agent-plane composition (persona, tools, delegation). |
| `preset.yml` | Display metadata (`name`, `description`). |
| `NOTICE` | Attribution for the derived `standard` preset. |
| `LICENSE` | MIT. |

---

## 中文

### 为什么用 Architect-Flash

- 💸 **省 token、省成本** —— 代码实现是编码任务里 token 最重的部分，这里它跑在便宜的
  Flash 子模型上，而不是主模型上。
- 🧠 **职责清晰** —— 一个*架构师*负责规划/设计/审查，一个*工人*负责实现，互不混淆。
- 🔒 **上下文隔离** —— 工人在自己的上下文里改文件，架构师的上下文保持精简、利于缓存。
- 🔌 **不硬编码模型** —— 工人跟随你在 sub-agent 设置里选择的子模型，preset 跨部署可移植。
- 🧰 **完整原生工具** —— 标准编码目录 + 可直接调用的 `code_flash` 委派工具。

### 架构

```
┌────────────────────────────────────────────────────────────┐
│  架构师  （主模型，如 deepseek-v4-pro）                    │
│  规划 · 设计 · 审查 · git · 验证构建                       │
└────────────────────────────┬───────────────────────────────┘
                             │  委派给 code_flash
                             ▼
┌────────────────────────────────────────────────────────────┐
│  code_flash 工人  （所选子模型，如 Flash）                 │
│  read · write · edit · glob · grep · pwsh · todo_write     │
│  独立上下文 · maxDepth 1（禁止递归）                       │
└────────────────────────────────────────────────────────────┘
```

### 对比

| | `standard` | `ptc`（PTC 模式） | **`architect-flash`** |
| --- | --- | --- | --- |
| 工具呈现 | 原生 | 单一 `run_code` | 原生 |
| 模型分级 | 单模型 | 单模型 | **Pro（主）+ Flash（工人）** |
| 谁写代码 | 模型自己 | 模型自己（在 `run_code` 里） | **Flash 工人** |
| 委派工具 | `subagent` | 藏在 `run_code` SDK 后 | **`code_flash`，直接调用** |
| token 成本 | 单模型全价 | 单模型全价 | **代码跑在便宜模型上** |
| 缓存/schema | 中 | 最优（极小 schema） | 中（但模型更便宜） |

PTC 优化的是 schema 和往返；Architect-Flash 优化的是*哪个模型干重活*。当代码实现占大头
时，Flash 分级通常比 PTC 省得更多。

### 快速开始

```sh
# Linux / macOS
cp -r architect-flash "$HOME/.dsh/.agent-presets/architect-flash"

# Windows (PowerShell)
Copy-Item -Recurse architect-flash "$env:USERPROFILE\.dsh\.agent-presets\architect-flash"
```

重启宿主，新建会话，选择 **Architect-Flash（架构师模式）**。

### 依赖

- 具备 `dsh-agent-tool-presentation`（`native`/`ptc`/`both`）架构、以及支持
  `modelSelectionSettings` 的 `@deepseek-ai/dsh-tool-subagent` 的 DeepSeek Harness。
- 一条在会话 `subagent-model-selection.allowedModels` 策略下注册、能承载 Flash 的
  LLM 路由。`code_flash` 不硬编码任何路由 —— 它跟随 sub-agent 选择的子模型。

### 选择子模型

子模型是会话级设置，不是 preset 的开关。在 `settings.yaml` 的
`subagent-model-selection.allowedModels` 下注册 `code_flash` 可用的路由：

```yaml
subagent-model-selection:
  enabled: true
  allowedModels:
    - provider: ark
      model: deepseek-v4-flash
```

`code_flash` 用 `modelSelectionSettings: true` 遵循该策略。架构师通过
`list_subagent_models` 发现子模型并传入其 `provider`/`model`，因此子模型就是 sub-agent
设置里选中的那一个。

### 行为契约

persona 编码了两条规则：

1. *代码实现 —— 尤其是重复性或底层代码 —— 委派给所选子模型上的 `code_flash`，不要
   内联编写。*
2. *子代理运行时，不要用 goal 轮询；停下并等待其结果注入。*

这是行为约束（靠 persona），不是物理约束。要让"写代码"成为唯一物理路径，可把主 Agent
的 `write`/`edit` 摘掉 —— 但注意架构师维护文档仍需要它们，所以"代码 vs 文档"的边界
最好放在工作区指令里。

### FAQ

**为什么用原生而不是 PTC？** PTC 把所有工具折叠成一个 `run_code`，委派工具藏在其 SDK
后面 —— 那里无法存在可直接调用的 `code_flash`。原生让 `code_flash` 保持一等工具。

**该选哪个子模型？** 你的 provider 提供的任意更便宜的推理模型（例如
`deepseek-v4-flash`），把它注册进 `subagent-model-selection.allowedModels` 即可。

**怎么强制所有代码都进工人？** 把主 Agent 目录里的 `write`/`edit` 摘掉。那样文档编辑
也得走工人，想清楚是否需要。

**Windows 能用吗？** 能。Windows 上 shell 工具是 `pwsh`，其它平台是 `bash`；`code_flash`
的工具过滤会点名正确的那一个。

### 文件

| 文件 | 用途 |
| --- | --- |
| `agent.cordis.yml` | agent-plane 组合（persona、工具、委派）。 |
| `preset.yml` | 展示元数据（`name`、`description`）。 |
| `NOTICE` | 对派生来源 `standard` 预设的归属声明。 |
| `LICENSE` | MIT。 |
