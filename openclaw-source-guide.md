# OpenClaw 源码导读版 / OpenClaw Source Guide

这份文档是 **源码导读版**，目标不是把所有文件都列出来，而是回答三个更有用的问题：

1. **先看哪里** / Where to start
2. **每个关键文件是干什么的** / What each key file does
3. **它依赖谁、又被谁依赖** / What depends on what

如果你已经看过总览调研，这份文档就是下一步。

If you have already read the high-level survey, this guide is the practical next step.

---

## 一、推荐阅读顺序 / Recommended Reading Order

## 第一轮：先建立全局骨架 / Pass 1: build the global skeleton

1. `package.json`  
   看发布面、workspace、`plugin-sdk` 导出面。  
   Read the published surface, workspace assumptions, and `plugin-sdk` exports.

2. `pnpm-workspace.yaml`  
   确认 monorepo 的真正边界：根包、`ui`、`packages/*`、`extensions/*`。  
   Confirms the true monorepo boundaries.

3. `tsdown.config.ts`  
   看构建图如何把 core entrypoints、plugin SDK、bundled plugins、hooks 一起打包。  
   Shows how core entrypoints, SDK subpaths, bundled plugins, and hooks are built together.

4. `docs/concepts/architecture.md`  
   理解 Gateway 是整个系统的控制面。  
   Establishes Gateway as the system control plane.

## 第二轮：看运行时主链路 / Pass 2: follow the runtime spine

5. `src/entry.ts`  
6. `src/index.ts`  
7. `src/gateway/boot.ts`  
8. `src/gateway/server-http.ts`  
9. `src/gateway/protocol/schema.ts`  
10. `src/sessions/` + `src/routing/`  
11. `src/agents/`  
12. `src/auto-reply/`

## 第三轮：看扩展边界 / Pass 3: understand the extension seam

13. `src/plugins/registry.ts`  
14. `src/plugins/loader.ts`  
15. `docs/plugins/architecture.md`  
16. `src/plugin-sdk/index.ts`  
17. 任选一个 channel plugin + 一个 provider plugin 深读  
    Read one representative channel plugin and one representative provider plugin.

## 第四轮：看产品表面 / Pass 4: read product surfaces

18. `ui/`
19. `apps/shared/OpenClawKit/Package.swift`
20. `apps/ios/`, `apps/android/`, `apps/macos/`
21. `Swabble/`

---

## 二、根目录关键文件 / Key Root Files

| 文件 / File | 作用（中文） | Role (English) |
| --- | --- | --- |
| `package.json` | 根包定义，决定 CLI 入口、导出面、发布内容，尤其是 `./plugin-sdk/*` 子路径。 | Defines the root package, CLI entry, published exports, and especially the `./plugin-sdk/*` surface. |
| `pnpm-workspace.yaml` | 决定 monorepo 的真正模块边界。 | Defines the actual workspace/module boundaries. |
| `openclaw.mjs` | CLI 可执行壳，负责把 `openclaw` 命令导入到真正的运行时入口。 | Thin executable wrapper that hands off to the real runtime entry. |
| `tsdown.config.ts` | 很关键的构建配置；它说明 OpenClaw 不是只编译 `src/`，而是同时考虑 SDK、bundled plugins、hooks。 | Key build graph config showing that core, SDK, hooks, and bundled plugins are packaged together. |
| `AGENTS.md` | 仓库级约束与边界说明。 | Repository-level architectural and workflow guidance. |
| `docs.acp.md` | 理解 ACP / control-plane 相关概念的辅助文档。 | Helpful ACP/control-plane context. |

### 根目录依赖关系 / Root Dependency View

```text
package.json
  -> 决定 published exports / published surface
pnpm-workspace.yaml
  -> 决定模块边界 / module boundaries
tsdown.config.ts
  -> 决定这些模块如何一起构建 / how those modules build together
openclaw.mjs
  -> 进入 CLI/runtime 启动链 / enters CLI/runtime boot chain
```

---

## 三、`src/` 总入口 / `src/` Core Entry Layer

`src/` 是 OpenClaw 的核心运行时；如果 `extensions/` 是能力生态，那么 `src/` 就是宿主内核。

### 关键文件 / Key Files

| 文件 / File | 作用（中文） | Role (English) |
| --- | --- | --- |
| `src/entry.ts` | 主启动入口之一，负责把 CLI/runtime 的初始化接起来。 | Primary startup path for CLI/runtime initialization. |
| `src/index.ts` | 根模块导出与部分 legacy handoff。 | Root module export surface and legacy entry handoff. |
| `src/runtime.ts` | 抽象 stdout、stderr、logging、exit 等运行环境能力。 | Shared runtime abstraction for logging/output/exit behavior. |
| `src/global-state.ts` | 进程级别的小型共享状态，例如 verbose/yes 等标志。 | Small process-wide flags and shared state. |
| `src/extensionAPI.ts` | extension-facing runtime bridge。 | Bridge between core runtime and extension-facing API. |

### 这一层该怎么读 / How to read this layer

这一层不要停留太久。它的主要任务是把你引导到真正的核心：

- Gateway
- Agent Runtime
- Plugin Host

Do not overstay here. This layer mainly points you toward the real core: Gateway, Agent Runtime, and Plugin Host.

---

## 四、`src/gateway/` — 控制面核心 / Control Plane Center

如果只挑一个目录先精读，优先 `src/gateway/`。

If you only choose one directory to read deeply first, choose `src/gateway/`.

### 关键文件逐个解释 / File-by-File Guide

| 文件 / File | 中文解释 | English explanation |
| --- | --- | --- |
| `src/gateway/boot.ts` | Gateway 启动期逻辑，处理 boot session、BOOT.md 检查、启动前后的会话映射恢复。它说明 Gateway 不只是 server，还会参与启动期控制流程。 | Startup-time Gateway logic for boot sessions, BOOT.md handling, and session mapping restore. Shows Gateway is more than a plain server. |
| `src/gateway/server-http.ts` | HTTP/HTTPS 入口总装配文件，串起 OpenAI-compatible HTTP、OpenResponses、Control UI、hooks、plugin routes、canvas、session history 等。它基本是 Gateway 的外部门面。 | Central HTTP/HTTPS assembly file that wires OpenAI-compatible APIs, OpenResponses, Control UI, hooks, plugin routes, canvas, and session history. |
| `src/gateway/control-ui.ts` | 浏览器 Control UI 的静态资源、根页面和配套状态装配。 | Serves and configures the browser Control UI. |
| `src/gateway/protocol/schema.ts` | WebSocket / API 协议的强类型合同；这是客户端、节点、控制面之间的公共边界。 | Typed contract for the WebSocket/API protocol boundary. |
| `src/gateway/server-methods.ts` | Gateway 方法注册与调度的核心入口之一。 | One of the central method registration/dispatch files. |
| `src/gateway/server-chat.ts` | Gateway 侧聊天入口，连接会话、事件、聊天流。 | Gateway-side chat entry that links sessions and chat flow. |
| `src/gateway/server-plugin-bootstrap.ts` | Gateway 启动时如何引入和接通 plugins。 | Shows how Gateway bootstraps and exposes plugins. |
| `src/gateway/node-registry.ts` | 节点/设备注册表，连接移动端和远程节点能力。 | Registry for mobile/remote node capabilities. |
| `src/gateway/channel-health-monitor.ts` | 渠道健康状态监控。 | Tracks channel health and liveness. |
| `src/gateway/auth.ts` | Gateway 认证主逻辑。 | Primary Gateway auth logic. |

### 依赖链 / Dependency Chains

#### 链路 A：HTTP 入口链 / HTTP ingress chain

```text
server-http.ts
  -> auth.ts / http-utils.ts
  -> control-ui.ts / openai-http.ts / openresponses-http.ts
  -> server-methods.ts / plugin routes / session endpoints
```

#### 链路 B：启动链 / boot chain

```text
entry.ts
  -> gateway/boot.ts
  -> server-startup.ts / server-http.ts / plugin bootstrap
```

#### 链路 C：协议链 / protocol chain

```text
protocol/schema.ts
  -> Gateway runtime validates requests
  -> clients and nodes consume the same contract
```

### 为什么先看它 / Why read it early

因为 Gateway 是 OpenClaw 的 **控制面总线**：

- 客户端从这里进入
- 节点从这里进入
- channel/plugin route 也从这里接入
- session / chat / auth / pairing 都经过这里

Gateway is the control-plane bus where clients, nodes, chat, auth, pairing, and plugin routes converge.

---

## 五、`src/agents/` — 执行面核心 / Execution Plane Core

这个目录决定“系统如何真正完成一次 agent run”。

This directory answers how the system actually performs an agent run.

### 关键文件逐个解释 / File-by-File Guide

| 文件 / File | 中文解释 | English explanation |
| --- | --- | --- |
| `src/agents/openclaw-tools.ts` | 组织 OpenClaw 运行时可见的 tools，是 runtime 能力暴露的核心入口之一。 | One of the main assembly points for runtime-visible tools. |
| `src/agents/tool-catalog.ts` | tool catalog，决定工具如何被描述、发现、暴露。 | Central tool catalog and exposure model. |
| `src/agents/model-selection.ts` | 决定 provider/model 选择，是“问哪个模型”的关键文件。 | Core model/provider selection logic. |
| `src/agents/auth-profiles.ts` | 处理 provider auth profiles、轮换、优先级等。 | Handles provider auth profiles, ordering, and rotation. |
| `src/agents/compaction.ts` | 长对话压缩逻辑，是上下文工程的重要文件。 | Context compaction for long conversations. |
| `src/agents/context.ts` | 上下文构建主入口之一。 | One of the main context-assembly entrypoints. |
| `src/agents/bash-tools.ts` | Bash 工具能力的顶层聚合入口。 | Top-level Bash tool aggregation. |
| `src/agents/bash-tools.exec-runtime.ts` | bash/exec 运行时执行细节。 | Runtime execution details for bash/exec tools. |
| `src/agents/mcp-transport.ts` | MCP transport 的关键桥接点。 | Main MCP transport bridge. |
| `src/agents/agent-command.ts` | agent 命令层与 runtime 的连接点。 | Connects agent commands to the runtime. |

### 依赖链 / Dependency Chains

#### 链路 A：一次 agent run / one agent run

```text
Gateway request
  -> sessions/routing
  -> agent-command.ts or equivalent runtime entry
  -> context.ts + compaction.ts
  -> model-selection.ts + auth-profiles.ts
  -> openclaw-tools.ts + tool-catalog.ts
  -> final streamed output
```

#### 链路 B：工具执行 / tool execution

```text
tool-catalog.ts
  -> openclaw-tools.ts
  -> bash-tools.ts / process hosts / plugin tools
```

### 阅读重点 / Reading Focus

这一层重点看四件事：

1. 上下文怎么组装 / how context is assembled
2. 模型怎么选 / how model/provider is selected
3. 工具怎么调 / how tools are invoked
4. 输出怎么流式返回 / how outputs are streamed back

---

## 六、`src/sessions/`、`src/routing/`、`src/auto-reply/`

这三块最好一起看，因为它们共同构成“消息 → 会话 → 运行”的中间层。

These three areas are best read together because they form the middle layer from inbound message to executable run.

### `src/sessions/`

- 管 session key / session id
- 管 transcript 生命周期
- 管会话连续性

Handles session keys, transcript lifecycle, and continuity.

### `src/routing/`

- 决定消息归属到哪个 account / agent / channel / thread / session

Resolves which account, agent, channel, thread, and session an event belongs to.

### `src/auto-reply/`

- 决定什么时候应该自动触发 agent reply
- 决定自动回复的入参如何成形

Decides when auto-reply should trigger and how the runtime input is shaped.

### 关键依赖链 / Key Dependency Chain

```text
inbound message
  -> routing
  -> session binding
  -> auto-reply decision
  -> agents runtime
  -> outbound dispatch
```

---

## 七、`src/plugins/` — 插件宿主内部 / Plugin Host Internals

这一层不是给插件作者看的“公共 API”，而是宿主如何发现、验证、装载、注册插件的内部机制。

This is not the public extension API. It is the host-side machinery for discovering, validating, loading, and registering plugins.

### 关键文件逐个解释 / File-by-File Guide

| 文件 / File | 中文解释 | English explanation |
| --- | --- | --- |
| `src/plugins/registry.ts` | 插件注册中心；这里最能看出插件究竟可以注册哪些能力：tools、channels、providers、hooks、services、HTTP routes、gateway handlers 等。 | The registry that reveals the true capability surface plugins can register. |
| `src/plugins/loader.ts` | 插件装载主逻辑，处理 manifest-first、jiti 动态加载、setup/full runtime 分离、alias 解析等。 | Main plugin loader handling manifest-first logic, jiti loading, setup/full runtime splits, and alias resolution. |
| `src/plugins/discovery.ts` | 负责发现 candidate plugins。 | Discovers candidate plugins. |
| `src/plugins/manifest-registry.ts` | 管 manifest 元数据注册和读取。 | Registry for manifest metadata. |
| `src/plugins/runtime.ts` | 宿主侧运行时插件状态入口之一。 | One of the host runtime entrypoints for plugin state. |
| `src/plugins/services.ts` | 插件 services 如何接入宿主。 | Connects plugin services into the host. |
| `src/plugins/cli.ts` | 插件 CLI surface 如何懒加载/注册。 | Plugin CLI registration and lazy-loading path. |

### 依赖链 / Dependency Chains

#### 链路 A：发现到注册 / discovery to registration

```text
discovery.ts
  -> manifest-registry.ts
  -> loader.ts
  -> registry.ts
  -> runtime/services/commands/routes/tools become visible
```

#### 链路 B：Gateway 与 plugin route / Gateway and plugin routes

```text
plugins/registry.ts
  -> gateway handlers / http routes registrations
  -> gateway/server-http.ts consumes them
```

### 阅读重点 / Reading Focus

这里最重要的设计思想是：

**先从 manifest 和 metadata 推理插件，再决定是否执行插件代码。**

The key design idea is: **reason from manifests and metadata first, execute plugin code later.**

---

## 八、`src/plugin-sdk/` — 公共扩展边界 / Public Extension Boundary

这一层是给扩展作者看的正式 contract。

This layer is the formal contract for extension authors.

### 关键文件逐个解释 / File-by-File Guide

| 文件 / File | 中文解释 | English explanation |
| --- | --- | --- |
| `src/plugin-sdk/index.ts` | 根级 SDK surface。文件本身刻意保持很薄，只暴露公共类型和聚合导出。 | Root SDK surface. Intentionally thin; aggregates stable public exports. |
| `src/plugin-sdk/core.ts` | 公共核心能力入口之一。 | One of the central shared SDK core entrypoints. |
| `src/plugin-sdk/plugin-entry.ts` | 插件入口 contract 的关键位置。 | Key contract for plugin entry definition. |
| `src/plugin-sdk/provider-entry.ts` | provider plugin 的入口合同。 | Entry contract for provider plugins. |
| `src/plugin-sdk/channel-entry-contract.ts` | channel plugin 的入口合同。 | Entry contract for channel plugins. |
| `src/plugin-sdk/gateway-runtime.ts` | 供插件接入 Gateway runtime 的桥梁。 | Bridge for plugin access to Gateway runtime. |
| `src/plugin-sdk/config-runtime.ts` | 配置读取/写入相关 runtime seam。 | Config-related runtime seam. |
| `src/plugin-sdk/runtime-secret-resolution.ts` | secret resolution 的扩展边界。 | Secret-resolution boundary for plugins. |

### 依赖链 / Dependency Chains

```text
plugin author code
  -> plugin-sdk/*
  -> plugin host (src/plugins/*)
  -> runtime registry
```

### 你应该如何理解它 / How to interpret it

`src/plugins/` 是 **宿主内部实现**，`src/plugin-sdk/` 是 **宿主对外合同**。

`src/plugins/` is host internals. `src/plugin-sdk/` is the public host contract.

---

## 九、`src/config/`、`src/secrets/`、`src/security/`

这三块共同构成 OpenClaw 的治理底座。

These three areas form the operational governance substrate.

### `src/config/` 关键文件 / Key Files

| 文件 / File | 中文解释 | English explanation |
| --- | --- | --- |
| `src/config/config.ts` | config 访问总入口。 | Main config access surface. |
| `src/config/io.ts` | config 读取、缓存、runtime snapshot、文件变更等重逻辑。 | Heavyweight config IO, caching, runtime snapshots, and mutation plumbing. |
| `src/config/paths.ts` | 配置和 state 目录路径解析。 | Resolves config/state directory paths. |
| `src/config/zod-schema.ts` | 配置 schema 核心。 | Main runtime config schema. |

### `src/secrets/`

- 把 env / file / exec 三类 secret source 注入 runtime config  
- Resolves env/file/exec secret sources into runtime config

### `src/security/`

- audit
- dangerous config/tool detection
- filesystem checks
- trust/auth helpers

---

## 十、`extensions/` — bundled plugin 生态 / Bundled Plugin Ecosystem

这个目录不建议平铺看。建议按“类别”看。

Do not read this directory flat. Read it by category.

### A. 渠道插件 / Channel Plugins

- `telegram`
- `slack`
- `discord`
- `feishu`
- `signal`
- `matrix`
- `whatsapp`

### B. Provider 插件 / Provider Plugins

- `openai`
- `anthropic`
- `google`
- `deepseek`
- `openrouter`
- `qwen`
- `minimax`

### C. 多模态插件 / Multimodal Plugins

- `browser`
- `speech-core`
- `media-understanding-core`
- `image-generation-core`
- `video-generation-core`
- `voice-call`

### 每个 extension 里最值得看的文件 / Common Important Files

| 文件 / File | 作用（中文） | Role (English) |
| --- | --- | --- |
| `index.ts` | 插件入口，定义 register(api) 逻辑。 | Plugin entrypoint and `register(api)` path. |
| `api.ts` | extension 本地 API barrel。 | Local API barrel for the extension. |
| `runtime-api.ts` | 运行时桥接面。 | Runtime bridge surface. |
| `openclaw.plugin.json` | manifest 元数据。 | Manifest metadata. |

---

## 十一、`ui/`、`apps/`、`Swabble/` — 产品表面 / Product Surfaces

### `ui/`

这是 Gateway 的浏览器客户端，不是独立后端。  
This is a browser client of the Gateway, not an independent backend.

### `apps/shared/OpenClawKit/Package.swift`

这个文件很重要，因为它直接说明 Apple 侧的共享拆分：

- `OpenClawProtocol`
- `OpenClawKit`
- `OpenClawChatUI`

This file is important because it shows the Apple-side split into shared protocol, runtime kit, and chat UI.

### `apps/ios/`, `apps/android/`, `apps/macos/`

这些目录是客户端/节点表面，主要通过 Gateway protocol 接系统。  
These directories are client/node surfaces that mostly connect through the Gateway protocol.

### `Swabble/`

更偏 Apple 侧语音唤醒/本地语音能力补充。  
More of an Apple-side wake-word/local voice companion project.

---

## 十二、总结：最重要的三条主链 / Final Three Core Chains

### 链路 1：控制面主链 / Control Plane Spine

```text
entry.ts
  -> gateway/boot.ts
  -> gateway/server-http.ts
  -> gateway/protocol/schema.ts
```

### 链路 2：执行面主链 / Execution Plane Spine

```text
inbound request
  -> sessions/routing
  -> agents/context/model-selection
  -> tools/process hosts/plugins
  -> streamed result
```

### 链路 3：扩展面主链 / Extension Plane Spine

```text
plugin manifest
  -> plugins/discovery.ts
  -> plugins/loader.ts
  -> plugins/registry.ts
  -> plugin-sdk/runtime exposure
  -> extensions become active capabilities
```

如果你按这三条链去读，OpenClaw 这套系统会很快从“目录很多”变成“脊柱清晰”。

If you read along these three chains, OpenClaw quickly stops feeling like “too many directories” and starts feeling like a clear platform spine.
