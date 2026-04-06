# OpenClaw 文档总索引与阅读路径 / OpenClaw Document Index and Reading Paths

这份文档是整个调研仓库的“总导航页”。它回答三个问题：

1. 现在已经写了哪些文档？
2. 每篇文档解决什么问题？
3. 如果读者目标不同，应该按什么顺序读？

This document is the master navigation page for the survey repository. It answers which documents exist, what each document is for, and how different readers should navigate them.

---

## 一、当前文档总览 / Current Document Set

截至目前，这个仓库已经包含以下文档：

The repository currently contains the following documents:

### A. 总览与导读 / Overview and Orientation

| 文档 | 作用 |
| --- | --- |
| `openclaw-source-architecture-survey.md` | 系统总览，帮助建立 OpenClaw 的整体认知 |
| `openclaw-source-guide.md` | 从目录和关键文件入手的源码导读 |
| `openclaw-layered-architecture.md` | 用四层图解释 OpenClaw 的整体分层 |

### B. 子系统深挖 / Subsystem Deep Dives

| 文档 | 作用 |
| --- | --- |
| `openclaw-gateway-deep-dive.md` | 深挖 Gateway 控制面 |
| `openclaw-agents-deep-dive.md` | 深挖 Agent Runtime 执行面 |
| `openclaw-plugin-system-deep-dive.md` | 深挖 Plugin System 与 SDK 边界 |
| `openclaw-sessions-routing-auto-reply-deep-dive.md` | 深挖中间编排层：sessions / routing / auto-reply |
| `openclaw-ui-platform-deep-dive.md` | 深挖 Web UI、原生客户端、OpenClawKit 与 Swabble |
| `openclaw-runtime-substrate-deep-dive.md` | 深挖配置、安全、进程、调度、日志等运行时底座 |

### C. 运行链与导航 / Flows and Navigation

| 文档 | 作用 |
| --- | --- |
| `openclaw-execution-flows.md` | 以时序和 handoff 解释关键执行链 |
| `openclaw-doc-index-and-reading-paths.md` | 当前这份总导航 |

---

## 二、按目的分组 / Grouped by Purpose

## 1. 想快速知道 OpenClaw 是什么 / Want a quick understanding of OpenClaw

读这三篇：

1. `openclaw-source-architecture-survey.md`
2. `openclaw-layered-architecture.md`
3. `openclaw-source-guide.md`

Read these three if you want the fast mental model.

## 2. 想理解系统内部怎么跑 / Want to understand how the system runs internally

读这五篇：

1. `openclaw-gateway-deep-dive.md`
2. `openclaw-sessions-routing-auto-reply-deep-dive.md`
3. `openclaw-agents-deep-dive.md`
4. `openclaw-runtime-substrate-deep-dive.md`
5. `openclaw-execution-flows.md`

Read these five if you want the runtime internals and end-to-end behavior.

## 3. 想理解扩展和生态 / Want to understand extensions and ecosystem

读这三篇：

1. `openclaw-plugin-system-deep-dive.md`
2. `openclaw-layered-architecture.md`
3. `openclaw-execution-flows.md`

Read these three if your goal is plugin/SDK and extension understanding.

## 4. 想理解用户表面和客户端 / Want to understand product surfaces and clients

读这两篇：

1. `openclaw-ui-platform-deep-dive.md`
2. `openclaw-layered-architecture.md`

Read these if your goal is UI, apps, OpenClawKit, and client-side architecture.

---

## 三、推荐阅读路径 / Suggested Reading Paths

## Persona 1：快速概览型 / Quick Overview Reader

适合：第一次接触 OpenClaw，希望 30 分钟内建立整体认知。  
Best for first-time readers who want a mental model in under 30 minutes.

推荐顺序：

1. `openclaw-source-architecture-survey.md`
2. `openclaw-layered-architecture.md`

## Persona 2：架构理解型 / Architecture Reader

适合：想从整体结构出发理解目录、模块、分层与边界。  
Best for readers who want to understand structure, modules, layering, and boundaries.

推荐顺序：

1. `openclaw-source-architecture-survey.md`
2. `openclaw-source-guide.md`
3. `openclaw-layered-architecture.md`
4. `openclaw-doc-index-and-reading-paths.md`

## Persona 3：运行时 internals 型 / Runtime Internals Reader

适合：想深入理解消息怎么进来、Agent 怎么跑、输出怎么回去。  
Best for readers who want to understand inbound handling, agent execution, and outbound delivery.

推荐顺序：

1. `openclaw-gateway-deep-dive.md`
2. `openclaw-sessions-routing-auto-reply-deep-dive.md`
3. `openclaw-agents-deep-dive.md`
4. `openclaw-runtime-substrate-deep-dive.md`
5. `openclaw-execution-flows.md`

## Persona 4：插件 / 扩展开发型 / Plugin and Extension Reader

适合：准备开发插件，或理解 plugin host / plugin-sdk / extensions。  
Best for readers preparing to build plugins or understand the plugin host and SDK boundary.

推荐顺序：

1. `openclaw-plugin-system-deep-dive.md`
2. `openclaw-source-guide.md`（extensions 部分）
3. `openclaw-layered-architecture.md`
4. `openclaw-execution-flows.md`

## Persona 5：平台 / 客户端型 / Platform and Client Reader

适合：准备看 Web UI、iOS、Android、macOS、OpenClawKit 或 Swabble。  
Best for readers focused on UI, iOS, Android, macOS, OpenClawKit, or Swabble.

推荐顺序：

1. `openclaw-ui-platform-deep-dive.md`
2. `openclaw-layered-architecture.md`
3. `openclaw-source-guide.md`（ui/apps 部分）

---

## 四、推荐交叉阅读 / Suggested Cross-Links Between Docs

有些文档最好对照看：

Some documents are strongest when cross-read:

| 从 / From | 到 / To | 为什么 / Why |
| --- | --- | --- |
| `openclaw-source-architecture-survey.md` | `openclaw-layered-architecture.md` | 一个给结构总览，一个给图形化分层 |
| `openclaw-source-guide.md` | 所有深挖文档 | 前者告诉你从哪读，后者告诉你读深了会看到什么 |
| `openclaw-gateway-deep-dive.md` | `openclaw-sessions-routing-auto-reply-deep-dive.md` | Gateway 和中间编排层是连续链路 |
| `openclaw-sessions-routing-auto-reply-deep-dive.md` | `openclaw-agents-deep-dive.md` | 中间层与执行面紧密衔接 |
| `openclaw-plugin-system-deep-dive.md` | `openclaw-execution-flows.md` | 一个讲边界，一个讲装载激活链 |
| `openclaw-ui-platform-deep-dive.md` | `openclaw-layered-architecture.md` | 一个讲产品表面，一个讲分层位置 |
| `openclaw-runtime-substrate-deep-dive.md` | `openclaw-agents-deep-dive.md` | 底层支撑执行面 |

---

## 五、如果你只想看最少集合 / Minimal Reading Sets

## 最少 2 篇：建立整体认知 / Minimal 2 docs

1. `openclaw-source-architecture-survey.md`
2. `openclaw-layered-architecture.md`

## 最少 4 篇：理解主运行链 / Minimal 4 docs for runtime understanding

1. `openclaw-gateway-deep-dive.md`
2. `openclaw-sessions-routing-auto-reply-deep-dive.md`
3. `openclaw-agents-deep-dive.md`
4. `openclaw-execution-flows.md`

## 最少 3 篇：理解扩展机制 / Minimal 3 docs for extension understanding

1. `openclaw-plugin-system-deep-dive.md`
2. `openclaw-layered-architecture.md`
3. `openclaw-execution-flows.md`

---

## 六、当前还可能补的方向 / Possible Future Additions

虽然文档已经很完整，但如果要继续扩成“源码知识库”，最自然的下一步是：

Although the document set is already strong, the most natural next additions would be:

1. **Channel Plugins 专题**  
   Telegram / Discord / Feishu / Slack 等渠道插件横向比较

2. **Provider Plugins 专题**  
   OpenAI / Anthropic / Google / OpenRouter 等模型接入对比

3. **Hooks 与 Lifecycle 专题**  
   单独讲 hooks 的触发点、作用域和副作用边界

4. **多模态能力专题**  
   Browser / Media Understanding / Speech / Image / Video 等能力如何被插件化接入

---

## 七、结论 / Final Summary

到目前为止，这套文档已经覆盖了 OpenClaw 的：

- 整体结构
- 分层架构
- 源码导读
- 核心子系统
- 产品表面
- 运行时底座
- 关键执行链

At this point, the document set covers overall architecture, layered structure, source guidance, core subsystems, product surfaces, runtime substrate, and key execution flows.

所以这份索引文档的定位就是：**让读者不再迷路。**

The purpose of this index is simple: **make sure readers no longer get lost.**
