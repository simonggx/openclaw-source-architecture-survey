# OpenClaw Gateway 深度导读 / OpenClaw Gateway Deep Dive

这篇文档专门深挖 `src/gateway/`。如果说 OpenClaw 的整体架构里有一个最接近“系统背板”的地方，那就是 Gateway。

This document is a focused deep dive into `src/gateway/`. If one part of OpenClaw behaves most like the system backplane, it is the Gateway.

---

## 一、先给结论 / Executive Takeaway

Gateway 不是“一个 HTTP 服务目录”，而是 OpenClaw 的 **控制面中枢**：

- 客户端从这里接入
- 节点从这里接入
- 协议从这里定义
- 鉴权从这里执行
- Chat / Sessions / Plugins / Nodes / Control UI 都从这里汇合

Gateway is not just “the HTTP server folder.” It is the **control-plane hub** where clients, nodes, protocol contracts, auth, chat, plugins, and UI all converge.

---

## 二、推荐阅读顺序 / Recommended Reading Order

### 第一轮：先看启动骨架 / Pass 1: startup skeleton

1. `src/gateway/server.ts`  
   这是极薄的入口壳，用来指向真正的实现。  
   Thin entry shell pointing to the real implementation.

2. `src/gateway/server.impl.ts`  
   整个 Gateway 的主装配文件。  
   Main Gateway assembly file.

3. `src/gateway/server-startup.ts`  
   启动阶段拆分逻辑。  
   Startup-phase orchestration.

4. `src/gateway/boot.ts`  
   boot session、BOOT.md、启动期行为。  
   Boot session and BOOT.md handling.

### 第二轮：看入口流量如何被接住 / Pass 2: ingress handling

5. `src/gateway/server-http.ts`  
6. `src/gateway/auth.ts`  
7. `src/gateway/auth-rate-limit.ts`  
8. `src/gateway/http-utils.ts`  
9. `src/gateway/http-common.ts`

### 第三轮：看协议与方法面 / Pass 3: protocol and method surface

10. `src/gateway/protocol/index.ts`  
11. `src/gateway/protocol/schema.ts`  
12. `src/gateway/server-methods.ts`  
13. `src/gateway/server-methods/`  

### 第四轮：看重点业务簇 / Pass 4: subsystem clusters

14. `src/gateway/server-chat.ts`  
15. `src/gateway/server-plugin-bootstrap.ts`  
16. `src/gateway/node-registry.ts`  
17. `src/gateway/control-ui.ts`  
18. `src/gateway/channel-health-monitor.ts`

---

## 三、关键文件逐个解释 / Key Files Explained One by One

## 1. `src/gateway/server.ts`

### 中文
这是 Gateway 的表层入口。它本身通常很薄，真正的价值不在“写了什么业务逻辑”，而在“它确认了 Gateway 的主实现位置”。读它的意义，是让你快速进入 `server.impl.ts`。

### English
This is the surface entrypoint for the Gateway. It is usually thin, and its main value is not business logic but clarifying that the real implementation lives in `server.impl.ts`.

---

## 2. `src/gateway/server.impl.ts`

### 中文
这是 Gateway 深读时最重要的文件。它负责把各个子系统装起来：HTTP server、plugin bootstrap、channel manager、node registry、health monitor、runtime state 等。你可以把它理解成“Gateway 的主机箱内部布线图”。

### English
This is the most important file in the Gateway deep dive. It wires together the HTTP server, plugin bootstrap, channel manager, node registry, health monitor, and runtime state. Think of it as the internal wiring diagram of the Gateway.

---

## 3. `src/gateway/server-startup.ts`

### 中文
这个文件承接了 Gateway 初始化过程中的阶段性逻辑，帮助把 `server.impl.ts` 的大体量拆薄。看它可以知道启动顺序被怎样显式建模。

### English
This file carries staged startup logic and helps split the large initialization flow away from `server.impl.ts`. It shows how startup order is modeled explicitly.

---

## 4. `src/gateway/boot.ts`

### 中文
`boot.ts` 很有代表性，因为它说明 Gateway 不只是“监听端口”，还参与系统启动期控制逻辑。这里会处理 `BOOT.md`、boot session 映射、启动前后 session 恢复等问题，所以它是“系统开机流程”的一部分。

### English
`boot.ts` is revealing because it shows Gateway is not just a listener. It participates in system boot logic such as `BOOT.md`, boot-session mapping, and session restoration.

---

## 5. `src/gateway/server-http.ts`

### 中文
这是 Gateway 对外 HTTP/HTTPS 面的总装配文件。它把很多分散的能力挂到同一个控制面入口上：Control UI、OpenAI-compatible HTTP、OpenResponses、embeddings、hooks、plugin routes、session history、tool invoke 等。读这个文件时，不要只把它当成“路由表”，它其实是在定义 Gateway 的对外产品面。

### English
This is the main HTTP/HTTPS assembly file. It mounts many external surfaces onto a single control-plane ingress: Control UI, OpenAI-compatible HTTP, OpenResponses, embeddings, hooks, plugin routes, session history, and tool invocation. It is more than a route table; it defines the outward product surface of the Gateway.

---

## 6. `src/gateway/auth.ts`

### 中文
`auth.ts` 是 Gateway 鉴权主逻辑。它决定 token/password/trusted-proxy/Tailscale 等模式如何被解析和执行。只要你想理解“谁可以接入 Gateway”，这个文件就必读。

### English
`auth.ts` is the primary auth decision engine for the Gateway. It resolves and enforces token/password/trusted-proxy/Tailscale modes. If you want to understand who may enter the Gateway, read this file.

---

## 7. `src/gateway/auth-rate-limit.ts`

### 中文
这个文件把 auth 失败尝试的限流逻辑单独拆出来，说明 Gateway 的控制面安全不是附带功能，而是内建约束。它通常跟 `auth.ts` 一起看。

### English
This file isolates rate limiting for failed auth attempts, showing that control-plane security is built into the Gateway rather than added later. Read it together with `auth.ts`.

---

## 8. `src/gateway/protocol/index.ts`

### 中文
这是 Gateway protocol 的聚合入口，负责 schema 导入、验证函数和错误码导出。对读协议的人来说，它是“目录索引”；对读实现的人来说，它是“验证边界”。

### English
This is the aggregation point for the Gateway protocol: schema imports, validation helpers, and error code exports. For protocol readers it is the index; for implementation readers it is the validation boundary.

---

## 9. `src/gateway/protocol/schema.ts`

### 中文
这是最关键的协议合同文件之一。WebSocket 和 API 面到底能说什么、怎么说、哪些字段是强约束，最终都要回到这里。

### English
This is one of the key protocol contract files. What the WebSocket/API surface may say, how it is structured, and which fields are strongly constrained all trace back here.

---

## 10. `src/gateway/server-methods.ts`

### 中文
这是 Gateway 方法注册和调度的重要汇合点。它把协议方法与具体 handler 串起来，是“控制面方法总线”的感觉。

### English
This is a major convergence point for Gateway method registration and dispatch. It links protocol methods to concrete handlers, acting like the control-plane method bus.

---

## 11. `src/gateway/server-chat.ts`

### 中文
`server-chat.ts` 代表 Gateway 里的聊天主链路：聊天请求如何进入、如何绑定 session、如何衔接 agent 事件与输出。它是 Gateway 与执行面的一个关键接缝。

### English
`server-chat.ts` represents the main chat path within the Gateway: how chat requests enter, bind to sessions, and connect to agent events and outputs. It is a key seam between control and execution.

---

## 12. `src/gateway/server-plugin-bootstrap.ts`

### 中文
这个文件说明 Gateway 启动时是如何把 plugins 接进系统的。它既关系到 runtime capability 的暴露，也关系到 routes、services、channels 等扩展能力如何在 Gateway 中可见。

### English
This file shows how plugins are brought into the system at Gateway startup. It affects runtime capability exposure as well as how routes, services, and channels become visible in the Gateway.

---

## 13. `src/gateway/node-registry.ts`

### 中文
这是节点/设备的注册中心。OpenClaw 不只是一个本地 CLI 或 Web UI，它还有 iOS/Android/macOS/headless nodes，所以 Gateway 必须维护节点连接、能力和待处理工作。

### English
This is the registry for devices/nodes. OpenClaw is not only a local CLI or web UI; it also has iOS/Android/macOS/headless nodes, so the Gateway must track node connections, capabilities, and pending work.

---

## 14. `src/gateway/control-ui.ts`

### 中文
这个文件说明 Gateway 如何把浏览器 Control UI 暴露出来。它把 UI 作为 Gateway 的一个产品表面，而不是单独的后端系统。

### English
This file shows how the browser Control UI is served by the Gateway, treating the UI as one Gateway product surface rather than an entirely separate backend.

---

## 15. `src/gateway/channel-health-monitor.ts`

### 中文
这个文件说明 Gateway 还承担“运行时巡检”角色，尤其是 channel 集成的健康状态检查、异常检测和自动恢复配合。

### English
This file shows that the Gateway also acts as a runtime watchdog, especially around channel health checks, stale-state detection, and recovery coordination.

---

## 四、按子系统看 / Gateway Subareas

## 1. 启动与总装配 / Startup and Assembly

- `server.ts`
- `server.impl.ts`
- `server-startup.ts`
- `boot.ts`

这一组文件回答的是：**Gateway 是怎么被装起来的？**  
This cluster answers: **how is the Gateway assembled and started?**

## 2. HTTP 与入口流量 / HTTP and Ingress

- `server-http.ts`
- `http-utils.ts`
- `http-common.ts`
- `net.ts`

这一组文件回答的是：**外界请求怎样进入 Gateway？**  
This cluster answers: **how does external traffic enter the Gateway?**

## 3. 鉴权与安全 / Auth and Security

- `auth.ts`
- `auth-rate-limit.ts`
- `connection-auth.ts`
- `origin-check.ts`
- `probe-auth.ts`

这一组文件回答的是：**谁可以接入、怎么被信任、怎么被拒绝？**  
This cluster answers: **who may connect, how trust is established, and how access is denied?**

## 4. 协议与方法面 / Protocol and Method Surface

- `protocol/index.ts`
- `protocol/schema.ts`
- `server-methods.ts`
- `server-methods/`

这一组文件回答的是：**Gateway 能对外说什么、接受什么方法、如何校验？**  
This cluster answers: **what the Gateway exposes, which methods it accepts, and how those methods are validated.**

## 5. 聊天与会话接缝 / Chat and Session Seam

- `server-chat.ts`
- `server-session-key.ts`
- `server-broadcast.ts`

这一组文件回答的是：**聊天是怎么在 Gateway 里变成 session/event/output 的？**  
This cluster answers: **how chat becomes session/event/output inside the Gateway.**

## 6. 插件接入 / Plugin Bootstrap

- `server-plugin-bootstrap.ts`
- `server-plugins.ts`

这一组文件回答的是：**插件怎样被接到 Gateway 运行时。**  
This cluster answers: **how plugins are connected into the Gateway runtime.**

## 7. 节点与设备 / Nodes and Devices

- `node-registry.ts`
- `node-catalog.ts`
- `server-node-events.ts`
- `server-mobile-nodes.ts`

这一组文件回答的是：**移动节点/远程节点怎么成为 Gateway 的一部分。**  
This cluster answers: **how mobile and remote nodes become part of the Gateway system.**

---

## 五、最重要的调用链 / Most Important Dependency Chains

## 链路 1：Gateway 启动链 / Gateway startup chain

```text
server.ts
  -> server.impl.ts
  -> server-startup.ts
  -> server-http.ts
  -> server-plugin-bootstrap.ts
  -> channel-health-monitor.ts
```

### 中文解释
这是“Gateway 从空进程变成完整控制面”的主链。

### English explanation
This is the main path by which an empty process becomes a full control plane.

## 链路 2：HTTP 请求入口链 / HTTP request ingress chain

```text
server-http.ts
  -> auth.ts / http-utils.ts
  -> protocol validation and method routing
  -> server-methods.ts or specialized handlers
```

### 中文解释
这条链说明 `server-http.ts` 并不是一个普通 router，而是 Gateway 外部入口的总分发器。

### English explanation
This chain shows that `server-http.ts` is not a plain router but the primary external ingress dispatcher.

## 链路 3：插件接入链 / Plugin bootstrap chain

```text
server.impl.ts
  -> server-plugin-bootstrap.ts
  -> server-plugins.ts
  -> plugin host runtime
  -> plugin routes/services/channels become visible
```

### 中文解释
这条链解释了为什么 Gateway 和 plugin system 是紧密耦合但又边界清晰的：Gateway 接入，plugins 提供能力。

### English explanation
This explains why Gateway and the plugin system are tightly connected yet clearly bounded: Gateway admits capabilities; plugins provide them.

## 链路 4：节点接入链 / Node connection chain

```text
client/node connect
  -> auth.ts
  -> node-registry.ts
  -> node-catalog.ts / server-node-events.ts
  -> node capability becomes invokable
```

### 中文解释
这条链解释了 OpenClaw 为什么不只是本地聊天程序，而是一个可连接移动节点的控制系统。

### English explanation
This chain explains why OpenClaw is more than a local chat app: it is a control system that can connect mobile/remote nodes.

## 链路 5：聊天链 / Chat chain

```text
Gateway request
  -> server-chat.ts
  -> session resolution / broadcast
  -> agent runtime
  -> output returns through Gateway
```

### 中文解释
这条链是 Gateway 与 Agent Runtime 的关键接缝。

### English explanation
This chain is the main seam between Gateway and the Agent Runtime.

---

## 六、如果你时间有限，最少读哪些 / Minimal Must-Read Set

如果你只有 2 小时，建议只读这 8 个文件：

If you only have 2 hours, read these 8 files:

1. `server.impl.ts`
2. `server-http.ts`
3. `auth.ts`
4. `protocol/index.ts`
5. `protocol/schema.ts`
6. `server-chat.ts`
7. `server-plugin-bootstrap.ts`
8. `node-registry.ts`

这 8 个文件足够让你理解：

- Gateway 怎么启动
- Gateway 怎么接请求
- Gateway 怎么做鉴权
- Gateway 怎么定义协议
- Gateway 怎么衔接聊天、插件、节点

These eight files are enough to understand startup, ingress, auth, protocol, and the major Gateway seams for chat, plugins, and nodes.

---

## 七、总结 / Final Summary

`src/gateway/` 的本质不是“server implementation details”，而是 OpenClaw 的 **控制面操作系统**。

它负责：

- 对外暴露边界
- 对内组织能力
- 维持系统秩序
- 连接 clients、nodes、plugins、chat、sessions

The essence of `src/gateway/` is not “server implementation details” but a **control-plane operating layer** for OpenClaw.
