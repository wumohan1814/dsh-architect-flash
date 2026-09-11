# Architect-Flash · 架构师模式（Pro+Flash）

> A [DeepSeek Harness](https://github.com/deepseek-ai/dsh) agent preset that splits coding work across two models: a main **architect** thread for planning, design, and review, and a **`code_flash`** worker for low-level code implementation on the sub-model selected in the sub-agent settings.

---

## English

### What it is

`architect-flash` is an agent preset for the DeepSeek Harness. It keeps the full
`standard` coding catalog in **native** tool presentation, then layers a two-tier
delegation split on top:

- **Main thread — the architect.** Runs on the session's main model
  (`deepseek-v4-pro` by default). It plans, designs, reviews, runs `git`, and
  verifies builds. It does **not** write product code itself.
- **`code_flash` worker.** A subagent that runs on the session's selected
  sub-model (the Flash model chosen in the sub-agent settings), restricted to
  the native implementation tools — `read`, `write`, `edit`, `glob`, `grep`,
  `pwsh` (or `bash` on non-Windows), `todo_write` — with `maxDepth: 1` so it
  cannot recurse.

The result: repetitive and low-level code implementation runs on the cheaper
sub-model with an isolated context, instead of consuming the expensive main
model.

### Why native instead of PTC

PTC mode collapses the whole tool catalog into a single `run_code` tool backed by
a generated SDK. That is excellent for cache locality, but it also buries the
delegation tool behind the `run_code` SDK — a distinct, directly callable
`code_flash` tool cannot exist under PTC. Native presentation exposes `code_flash`
as a first-class tool the architect can call directly, which is what the Pro/Flash
split requires.

### Requirements

- DeepSeek Harness with the `dsh-agent-tool-presentation` architecture
  (`native` / `ptc` / `both` modes) and `@deepseek-ai/dsh-tool-subagent` with
  `modelSelectionSettings`.
- A Flash-capable LLM route registered under the session's
  `subagent-model-selection.allowedModels` policy. `code_flash` hardcodes no
  route — it follows the sub-agent's selected sub-model.

### Install

Copy the whole directory into your user preset root, then restart the host:

```sh
# Linux / macOS
cp -r architect-flash "$HOME/.dsh/.agent-presets/architect-flash"

# Windows (PowerShell)
Copy-Item -Recurse architect-flash "$env:USERPROFILE\.dsh\.agent-presets\architect-flash"
```

Start a new session and select **架构师模式（Pro+Flash）**.

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

`code_flash` uses `modelSelectionSettings: true` to follow this policy. It
hardcodes no route; the architect discovers the sub-model with
`list_subagent_models` and passes its `provider`/`model`, so the sub-model is
whatever the sub-agent settings select.

### Files

| File | Purpose |
| --- | --- |
| `agent.cordis.yml` | The agent-plane composition (persona, tools, delegation). |
| `preset.yml` | Display metadata (`name`, `description`). |
| `NOTICE` | Attribution for the derived `standard` preset. |
| `LICENSE` | MIT. |

### Behavior contract

The persona encodes two rules:

1. *For code implementation — especially repetitive or low-level code —
   delegate to the `code_flash` worker on the selected sub-model instead of
   writing it inline.*
2. *While a delegated worker is running, do not poll with goal updates; stop
   and wait for its result to be injected.*

These are behavioral (the persona), not physical. To make "write code" a single
physical path, remove `write`/`edit` from the main agent's tools — but note the
architect still needs them for documentation maintenance (requirements/bug
tables), so the code-vs-docs boundary is best kept as a workspace instruction.

---

## 中文

### 它是什么

`architect-flash` 是 DeepSeek Harness 的一个 agent 预设。它在**原生**工具呈现下保留
完整 `standard` 编码目录，并叠加两层分级：

- **主线程 —— 架构师。** 运行在会话主模型上（默认 `deepseek-v4-pro`），负责规划、
  设计、审查、执行 `git`、验证构建，**不直接写产品代码**。
- **`code_flash` 代码工。** 一个运行在会话所选子模型上的子代理（即 sub-agent 设置里
  选择的 Flash 子代理），工具限制为原生实现集 —— `read`、`write`、`edit`、`glob`、
  `grep`、`pwsh`（非 Windows 为 `bash`）、`todo_write`，且 `maxDepth: 1` 禁止递归。

结果：重复性、底层代码实现落在更便宜的子模型上、拥有独立上下文，不再消耗昂贵的主模型。

### 为什么用原生而不是 PTC

PTC 模式把整个工具目录折叠成单一 `run_code` 工具（背后是生成的 SDK）。这对缓存局部性
极好，但也会把委派工具藏到 `run_code` SDK 后面 —— 在 PTC 下无法存在一个可直接调用的
`code_flash` 具名工具。原生呈现把 `code_flash` 暴露为一等工具，架构师可直接调用，这
正是 Pro/Flash 分级所需要的。

### 依赖

- 具备 `dsh-agent-tool-presentation`（`native`/`ptc`/`both`）架构、以及支持
  `modelSelectionSettings` 的 `@deepseek-ai/dsh-tool-subagent` 的 DeepSeek Harness。
- 一条在会话 `subagent-model-selection.allowedModels` 策略下注册、能承载 Flash 的
  LLM 路由。`code_flash` 不硬编码任何路由 —— 它跟随 sub-agent 选择的子模型。

### 安装

把整个目录复制到用户预设根目录，然后重启宿主：

```sh
# Linux / macOS
cp -r architect-flash "$HOME/.dsh/.agent-presets/architect-flash"

# Windows (PowerShell)
Copy-Item -Recurse architect-flash "$env:USERPROFILE\.dsh\.agent-presets\architect-flash"
```

新建会话，选择 **架构师模式（Pro+Flash）**。

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

`code_flash` 用 `modelSelectionSettings: true` 遵循该策略。它不硬编码路由；架构师通过
`list_subagent_models` 发现子模型并传入其 `provider`/`model`，因此子模型就是 sub-agent
设置里选中的那一个。

### 文件

| 文件 | 用途 |
| --- | --- |
| `agent.cordis.yml` | agent-plane 组合（persona、工具、委派）。 |
| `preset.yml` | 展示元数据（`name`、`description`）。 |
| `NOTICE` | 对派生来源 `standard` 预设的归属声明。 |
| `LICENSE` | MIT。 |

### 行为契约

persona 编码了两条规则：

1. *代码实现 —— 尤其是重复性或底层代码 —— 委派给所选子模型上的 `code_flash`，不要
   内联编写。*
2. *子代理运行时，不要用 goal 轮询；停下并等待其结果注入。*

这是行为约束（靠 persona），不是物理约束。要让"写代码"成为唯一物理路径，可把主 Agent
的 `write`/`edit` 摘掉 —— 但注意架构师维护文档（需求表 / Bug 表）仍需要它们，所以
"代码 vs 文档"的边界最好放在工作区指令里，而不是靠工具删除。
