# OpenClaw Agent Runtime 深度导读 / OpenClaw Agent Runtime Deep Dive

这篇文档专门深挖 `src/agents/`。如果 Gateway 是控制面中枢，那么 `src/agents/` 就是 OpenClaw 真正的执行引擎。

This document is a focused deep dive into `src/agents/`. If Gateway is the control-plane center, `src/agents/` is the real execution engine.

---

## 一、先给结论 / Executive Takeaway

Agent Runtime 的本质不是“调用一个模型”，而是：

- 解析命令与上下文
- 选择模型与认证配置
- 组装工具面
- 驱动工具执行
- 必要时生成压缩摘要
- 管理子代理与外部 transport
- 最终把结果流式返回

The agent runtime is not just “calling a model.” It parses commands and context, selects models and auth profiles, assembles tools, drives execution, compacts context when needed, manages subagents and transports, and streams results back.

---

## 二、推荐阅读顺序 / Recommended Reading Order

### 第一轮：先抓主入口 / Pass 1: main entry first

1. `src/agents/agent-command.ts`
2. `src/agents/agent-scope.ts`
3. `src/agents/command/attempt-execution.ts`
4. `src/agents/command/delivery.ts`

### 第二轮：看上下文与模型选择 / Pass 2: context and model selection

5. `src/agents/context.ts`
6. `src/agents/compaction.ts`
7. `src/agents/model-selection.ts`
8. `src/agents/auth-profiles.ts`

### 第三轮：看工具执行面 / Pass 3: tool execution

9. `src/agents/tool-catalog.ts`
10. `src/agents/bash-tools.ts`
11. `src/agents/bash-tools.exec.ts`
12. `src/agents/bash-process-registry.ts`

### 第四轮：看高级执行能力 / Pass 4: advanced runtime features

13. `src/agents/mcp-transport.ts`
14. `src/agents/mcp-transport-config.ts`
15. `src/agents/subagent-spawn.ts`
16. `src/agents/subagent-registry.ts`
17. `src/agents/skills.ts`
18. `src/agents/system-prompt.ts`

---

## 三、关键文件逐个解释 / Key Files Explained One by One

## 1. `src/agents/agent-command.ts`

### 中文
这是 Agent Runtime 的主入口文件之一。它承接用户输入或系统发起的 agent command，把 scope、model、tooling、execution、delivery 串起来。你可以把它看成“执行面调度器”。

### English
This is one of the main entrypoints for the Agent Runtime. It connects scope resolution, model selection, tooling, execution, and delivery. Think of it as the execution-plane dispatcher.

---

## 2. `src/agents/agent-scope.ts`

### 中文
这个文件负责把 agent 配置解析成运行时可用的 scope：工作区、skills、模型配置、会话相关信息等。它解释了“这次 run 处在什么上下文里”。

### English
This file resolves agent configuration into runtime scope: workspace, skills, model config, and session-related context. It explains the environment in which a run happens.

---

## 3. `src/agents/command/attempt-execution.ts`

### 中文
这是一次执行尝试的核心逻辑位置。很多 runtime 编排不会直接堆在 `agent-command.ts`，而是下沉到这里，使“执行尝试”成为显式阶段。

### English
This is a core location for a single execution attempt. Instead of dumping all runtime orchestration into `agent-command.ts`, the system makes the attempt phase explicit here.

---

## 4. `src/agents/command/delivery.ts`

### 中文
这个文件关注“结果如何被交付出去”。它非常重要，因为 Agent Runtime 不只负责计算，还要负责输出在系统里如何落地。

### English
This file focuses on how results are delivered. It matters because the Agent Runtime is not only about computation, but also about how results land in the wider system.

---

## 5. `src/agents/context.ts`

### 中文
`context.ts` 是理解 Agent Runtime 的关键文件之一。它管理上下文窗口、token 估算、模型上下文能力等，是“这次能带多少信息进模型”的核心位置。

### English
`context.ts` is one of the key files for understanding the runtime. It manages context windows, token estimation, and model context capabilities—the core of how much information can be carried into a turn.

---

## 6. `src/agents/compaction.ts`

### 中文
当对话变长、上下文开始逼近上限时，`compaction.ts` 决定如何压缩历史，保留关键标识符和足够上下文。它是长会话可持续运行的关键机制。

### English
When conversations grow long and context approaches limits, `compaction.ts` decides how to summarize history while preserving key identifiers and enough continuity. It is essential for sustainable long-running sessions.

---

## 7. `src/agents/model-selection.ts`

### 中文
这个文件回答“本次到底选哪个 provider/model”。它处理模型引用、provider 规范化、thinking 层级等，是模型路由的中枢。

### English
This file answers which provider/model is selected for a run. It handles model references, provider normalization, and thinking-level mapping.

---

## 8. `src/agents/auth-profiles.ts`

### 中文
模型能不能被真正调用，不只取决于选择了谁，还取决于认证配置。这个文件处理 auth profiles、顺序、轮换、冷却期等，是“可用模型路由”的一半。

### English
Selecting a model is only half the story; the runtime also needs valid credentials. This file handles auth profiles, ordering, rotation, and cooldown logic.

---

## 9. `src/agents/tool-catalog.ts`

### 中文
工具不是零散函数集合，而是被整理成 catalog 的系统能力。这个文件定义工具如何分组、如何描述、如何进入 profile。

### English
Tools are not a pile of loose functions; they are organized as a capability catalog. This file defines grouping, descriptions, and profile inclusion.

---

## 10. `src/agents/bash-tools.ts`

### 中文
这是 Bash 工具面的聚合入口。它把命令执行能力以 Agent 可消费的 tool 形式暴露出来。

### English
This is the top-level aggregation point for Bash tools, exposing command execution as runtime-consumable tools.

---

## 11. `src/agents/bash-tools.exec.ts`

### 中文
如果你想知道“命令到底怎么跑”，看这个文件。它涉及 exec runtime、超时、PTY、输出处理、审批与执行细节。

### English
If you want to know how commands actually run, read this file. It covers exec runtime, timeouts, PTY handling, output processing, approvals, and execution details.

---

## 12. `src/agents/bash-process-registry.ts`

### 中文
这个文件说明 Agent Runtime 不只是“发一个命令再等结果”，它还需要追踪后台/长生命周期进程。

### English
This file shows that the runtime is not just “fire a command and wait”; it must also track background and long-lived processes.

---

## 13. `src/agents/mcp-transport-config.ts`

### 中文
MCP transport 的配置解析入口，说明外部工具或系统是如何通过 stdio/http/sse 等方式接进 Runtime 的。

### English
Configuration entrypoint for MCP transport, showing how external systems connect via stdio/http/sse-style mechanisms.

---

## 14. `src/agents/mcp-transport.ts`

### 中文
这个文件真正把 transport 解析成运行时可用形态，是 Agent Runtime 与外部工具生态的重要桥。

### English
This file resolves transport config into runtime-usable transport objects, acting as an important bridge to external tooling ecosystems.

---

## 15. `src/agents/subagent-spawn.ts`

### 中文
OpenClaw 不只是单 agent loop，它还能 spawn subagents。这个文件定义了子代理如何被创建、带哪些能力、以什么模式运行。

### English
OpenClaw is not just a single-agent loop; it can spawn subagents. This file defines how subagents are created, what capabilities they carry, and how they run.

---

## 16. `src/agents/subagent-registry.ts`

### 中文
子代理不是“一次 spawn 就结束”的黑盒，还要被跟踪状态、管理生命周期、处理 announce 和结束原因。这个文件就是那个注册表。

### English
Subagents are not fire-and-forget black boxes. Their state, lifecycle, completion, and announce flow must be tracked here.

---

## 17. `src/agents/skills.ts`

### 中文
skills 是 runtime 能力组合的重要层。这个文件是 skill 子系统的入口，让运行时把技能作为结构化能力注入 prompt 和行为面。

### English
Skills are a major capability-composition layer for the runtime. This file acts as the skill subsystem entrypoint, injecting skills into prompts and runtime behavior.

---

## 18. `src/agents/system-prompt.ts`

### 中文
这是系统提示词的拼装中心。它回答“运行时最终把哪些规则、工具说明、能力描述交给模型”。

### English
This is the system-prompt composition center. It answers which rules, tool descriptions, and runtime capabilities are ultimately handed to the model.

---

## 四、按子系统看 / Agent Runtime Subareas

## 1. 命令桥接层 / Command Bridge

- `agent-command.ts`
- `command/attempt-execution.ts`
- `command/delivery.ts`
- `command/session.ts`

这一组文件回答：**一次 runtime 执行是怎样被发起并交付完成的。**  
This cluster answers: **how a runtime execution is initiated and delivered.**

## 2. 上下文层 / Context Layer

- `context.ts`
- `compaction.ts`
- `context-cache.ts`
- `context-window-guard.ts`

这一组文件回答：**上下文怎么被管理、什么时候压缩。**  
This cluster answers: **how context is managed and when compaction happens.**

## 3. 模型与认证层 / Model and Auth Layer

- `model-selection.ts`
- `models-config.*`
- `auth-profiles.ts`
- `auth-profiles/`

这一组文件回答：**选谁来回答、拿什么凭证去调用。**  
This cluster answers: **which model responds and which credentials make it possible.**

## 4. 工具与执行层 / Tools and Execution Layer

- `tool-catalog.ts`
- `bash-tools.ts`
- `bash-tools.exec.ts`
- `bash-process-registry.ts`

这一组文件回答：**Agent Runtime 拥有什么能力、命令如何落地执行。**  
This cluster answers: **which capabilities the runtime has and how execution actually happens.**

## 5. 外部 transport 层 / External Transport Layer

- `mcp-transport.ts`
- `mcp-transport-config.ts`
- `mcp-stdio.ts`
- `mcp-http.ts`
- `mcp-sse.ts`

这一组文件回答：**Runtime 怎样接外部工具/服务。**  
This cluster answers: **how the runtime connects to external tools/services.**

## 6. 子代理与技能层 / Subagents and Skills

- `subagent-spawn.ts`
- `subagent-registry.ts`
- `skills.ts`
- `system-prompt.ts`

这一组文件回答：**Runtime 如何扩展成多代理、多技能的执行系统。**  
This cluster answers: **how the runtime grows into a multi-agent, multi-skill execution system.**

---

## 五、最重要的执行链 / Most Important Execution Chains

## 链路 1：一次 agent run / One agent run

```text
agent-command.ts
  -> agent-scope.ts
  -> command/attempt-execution.ts
  -> context.ts + compaction.ts
  -> model-selection.ts + auth-profiles.ts
  -> tool-catalog.ts + bash-tools.ts
  -> command/delivery.ts
```

### 中文解释
这是 Agent Runtime 的总主链，从入口到交付完整走一遍。

### English explanation
This is the main end-to-end runtime chain from entry to delivery.

## 链路 2：上下文压缩链 / Context compaction chain

```text
context.ts
  -> token estimation and context window checks
  -> compaction.ts
  -> summary generation
  -> compacted history re-enters execution
```

### 中文解释
这条链解释了长对话为什么还能持续跑下去。

### English explanation
This chain explains how long conversations remain sustainable.

## 链路 3：模型与认证链 / Model-and-auth chain

```text
model-selection.ts
  -> provider/model resolution
  -> auth-profiles.ts
  -> selected credential path
  -> actual provider invocation becomes possible
```

### 中文解释
模型选择和认证配置不是两件分离的小事，而是一次调用能否成立的完整链路。

### English explanation
Model selection and credential resolution are not separate details; together they decide whether a call can actually happen.

## 链路 4：工具执行链 / Tool execution chain

```text
tool-catalog.ts
  -> tool selection/exposure
  -> bash-tools.ts
  -> bash-tools.exec.ts
  -> bash-process-registry.ts
```

### 中文解释
这条链解释了工具不是“静态说明”，而是真正被运行时组织成可执行能力。

### English explanation
This chain shows that tools are not just static descriptions but executable runtime capabilities.

## 链路 5：子代理链 / Subagent chain

```text
subagent-spawn.ts
  -> subagent capability resolution
  -> subagent-registry.ts
  -> subagent runtime work
  -> announce/delivery back to parent flow
```

### 中文解释
这条链解释了 OpenClaw 为什么可以从单 agent loop 走向多 agent 协作。

### English explanation
This chain explains how OpenClaw grows from a single-agent loop into a multi-agent collaboration system.

---

## 六、如果你时间有限，最少读哪些 / Minimal Must-Read Set

如果你只有 2 小时，建议优先读这 10 个文件：

If you only have 2 hours, prioritize these 10 files:

1. `agent-command.ts`
2. `agent-scope.ts`
3. `command/attempt-execution.ts`
4. `context.ts`
5. `compaction.ts`
6. `model-selection.ts`
7. `auth-profiles.ts`
8. `tool-catalog.ts`
9. `bash-tools.exec.ts`
10. `subagent-spawn.ts`

这 10 个文件足够让你理解：

- runtime 从哪里进入
- 上下文怎么组装
- 模型怎么选
- 工具怎么执行
- 子代理怎么扩展执行面

These ten files are enough to understand entry, context assembly, model choice, tool execution, and how subagents extend the runtime.

---

## 七、总结 / Final Summary

`src/agents/` 的本质不是一个“LLM 调用目录”，而是 OpenClaw 的 **执行面操作系统**。

它负责把：

- 命令
- 上下文
- 模型
- 工具
- transport
- 子代理

组织成一次可持续、可扩展、可审计的 agent run。

The essence of `src/agents/` is not a simple “LLM calling directory,” but an **execution-plane operating layer** that organizes commands, context, models, tools, transports, and subagents into a sustainable, extensible, and auditable agent run.
