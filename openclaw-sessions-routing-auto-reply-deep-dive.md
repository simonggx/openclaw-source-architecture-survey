# OpenClaw Sessions / Routing / Auto-Reply 深度导读 / OpenClaw Sessions, Routing, and Auto-Reply Deep Dive

这篇文档把三个目录放在一起讲：

- `src/sessions/`
- `src/routing/`
- `src/auto-reply/`

这样做是因为它们共同组成了 OpenClaw 最关键的一条中间链路：

**消息进入系统 → 绑定到哪个 agent/session → 决定是否自动回复 → 生成 reply 并写回 transcript**

This document treats `src/sessions/`, `src/routing/`, and `src/auto-reply/` together because they form one of OpenClaw’s most important middle-layer chains: **message enters the system → binds to an agent/session → decides whether to auto-reply → generates a reply and writes transcript updates**.

---

## 一、先给结论 / Executive Takeaway

这三块分别负责：

- **Sessions**：会话身份、session id、transcript 生命周期
- **Routing**：消息应该落到哪个 agent / account / channel / peer / thread
- **Auto-Reply**：什么时候触发自动回复、回复如何分发、回复上下文如何成形

Together they answer three questions:

- **Who owns this message?**
- **Which session should this affect?**
- **Should the system reply, and how?**

---

## 二、推荐阅读顺序 / Recommended Reading Order

### 第一轮：先抓 session key 和 route / Pass 1: session key and route first

1. `src/routing/session-key.ts`
2. `src/sessions/session-key-utils.ts`
3. `src/routing/account-id.ts`
4. `src/routing/bindings.ts`
5. `src/routing/resolve-route.ts`

### 第二轮：看 session identity 与 lifecycle / Pass 2: session identity and lifecycle

6. `src/sessions/session-id.ts`
7. `src/sessions/session-id-resolution.ts`
8. `src/sessions/session-lifecycle-events.ts`
9. `src/sessions/transcript-events.ts`
10. `src/sessions/send-policy.ts`

### 第三轮：看 auto-reply 主链 / Pass 3: auto-reply core flow

11. `src/auto-reply/dispatch.ts`
12. `src/auto-reply/reply/inbound-context.ts`
13. `src/auto-reply/reply.ts`
14. `src/auto-reply/fallback-state.ts`
15. `src/auto-reply/command-detection.ts`
16. `src/auto-reply/heartbeat.ts`

---

## 三、关键文件逐个解释 / Key Files Explained One by One

## 1. `src/routing/session-key.ts`

### 中文
这个文件定义了 session key 的构造逻辑。OpenClaw 不是随便拼一个字符串来标识会话，而是用一套带 agent/channel/account/peer 语义的 key 体系。理解这文件，是理解整个中间层的第一步。

### English
This file defines how session keys are constructed. OpenClaw does not identify sessions with arbitrary strings; it uses a structured key system that carries agent/channel/account/peer semantics. Understanding this file is the first step to understanding the entire middle layer.

---

## 2. `src/sessions/session-key-utils.ts`

### 中文
如果 `routing/session-key.ts` 偏构造，那么这个文件偏解析。它把 session key 解析回 agent/session/thread/subagent 等语义，是后续 lifecycle 和 transcript 系统理解 session 的基础。

### English
If `routing/session-key.ts` focuses on construction, this file focuses on parsing. It resolves a session key back into agent/session/thread/subagent semantics, making it foundational for lifecycle and transcript systems.

---

## 3. `src/routing/account-id.ts`

### 中文
这个文件负责 account id 的规范化，看似简单，其实很重要。因为 route resolution 往往会围绕 account/channel/peer 三元组展开，如果 account id 不稳定，整个路由层都会变脆。

### English
This file normalizes account ids. It looks simple, but it is important because route resolution often revolves around the account/channel/peer triple. Unstable account ids would make the whole routing layer fragile.

---

## 4. `src/routing/bindings.ts`

### 中文
这个文件管理 binding 关系，也就是“哪些 account/channel/peer 应该绑定到哪个 agent”。它是 route resolution 的数据来源之一。

### English
This file manages binding relationships, meaning which account/channel/peer combinations should map to which agent. It is one of the main data sources for route resolution.

---

## 5. `src/routing/resolve-route.ts`

### 中文
这是 routing 的主文件，回答“这条 inbound 最终归谁”。它会考虑 channel、account、peer、guild、team、role 等信息，并输出 agentId、sessionKey、mainSessionKey、matchedBy 等结果。

### English
This is the main routing file. It answers who ultimately owns an inbound event. It considers channel, account, peer, guild, team, role, and more, then outputs agentId, sessionKey, mainSessionKey, and match strategy.

---

## 6. `src/sessions/session-id.ts`

### 中文
session key 是结构化 identity，session id 更偏 runtime identity。这个文件定义 session id 的生成与基本处理，是会话身份的另一半。

### English
Session keys provide structured identity, while session ids are closer to runtime identity. This file defines session-id generation and basic handling, making it the other half of session identity.

---

## 7. `src/sessions/session-id-resolution.ts`

### 中文
当系统拿到 UUID、别名或模糊 session 输入时，这个文件负责把它折叠回真正的 session key。它解释了 OpenClaw 为什么既能保留内部 key 语义，又能给用户相对友好的会话引用方式。

### English
When the system receives a UUID, alias, or fuzzy session reference, this file resolves it back to the real session key. It explains how OpenClaw preserves structured internal identity while offering friendlier external references.

---

## 8. `src/sessions/session-lifecycle-events.ts`

### 中文
这个文件负责 session 生命周期事件的发布与订阅，例如创建、重置、标签变更等。它是“会话状态变化如何被广播”的关键点。

### English
This file publishes and subscribes to session lifecycle events such as creation, reset, and label changes. It is the key point where session-state changes are broadcast.

---

## 9. `src/sessions/transcript-events.ts`

### 中文
这个文件负责 transcript 更新事件，也就是消息写入、变更、补丁后如何通知外界。它是 session state 和 UI/update stream 之间的重要桥梁。

### English
This file handles transcript update events, meaning how message writes, changes, and patches notify the rest of the system. It is an important bridge between session state and UI/update streams.

---

## 10. `src/sessions/send-policy.ts`

### 中文
并不是所有 session 都以同样策略发送输出。这个文件处理 send policy，是“这条回复该不该发、怎么发”的规则层之一。

### English
Not every session sends output with the same policy. This file handles send policy and is one of the rule layers behind whether and how a reply should be sent.

---

## 11. `src/auto-reply/dispatch.ts`

### 中文
这是 inbound auto-reply 的总入口之一。它负责把外部消息上下文 finalize 后交给 reply pipeline，并保证 dispatcher 生命周期完整结束。

### English
This is one of the main entrypoints for inbound auto-reply. It finalizes inbound message context, hands it to the reply pipeline, and ensures dispatcher lifecycle completion.

---

## 12. `src/auto-reply/reply/inbound-context.ts`

### 中文
这个文件负责把 inbound message context 规范化。很多 auto-reply 问题最终不是“模型怎么答”，而是“输入上下文有没有被标准化”。

### English
This file normalizes inbound message context. Many auto-reply issues are not really about model behavior, but about whether the input context was standardized correctly.

---

## 13. `src/auto-reply/reply.ts`

### 中文
这是 auto-reply 的核心总装配文件之一。它会处理 directives、模型选择、session state 初始化、agent runner 调用、reply shaping 等关键步骤。

### English
This is one of the core assembly files of auto-reply. It handles directives, model selection, session-state initialization, agent-runner invocation, and reply shaping.

---

## 14. `src/auto-reply/fallback-state.ts`

### 中文
这个文件管理 fallback 状态。当模型、provider 或某些回复路径失败时，系统如何切换、怎样把 fallback 信息变成用户可见状态，这里非常关键。

### English
This file manages fallback state. When models, providers, or reply paths fail, this file determines how the system falls back and how that state becomes visible.

---

## 15. `src/auto-reply/command-detection.ts`

### 中文
并不是所有 inbound message 都是普通聊天消息，有些是命令、指令或特殊触发。这个文件负责命令检测，是 auto-reply 决策前的一道分流层。

### English
Not every inbound message is a plain chat message; some are commands, directives, or special triggers. This file handles command detection and acts as a pre-routing branch in auto-reply decisions.

---

## 16. `src/auto-reply/heartbeat.ts`

### 中文
这个文件说明 auto-reply 不只响应即时 inbound，还能响应 heartbeat 驱动的节拍型触发。这让 auto-reply 从“被动回复”扩展成“调度触发”的一部分。

### English
This file shows that auto-reply does not only respond to immediate inbound messages; it can also respond to heartbeat-driven triggers. That expands auto-reply from passive reply behavior into part of the scheduling fabric.

---

## 四、按子系统看 / Sessions-Routing-AutoReply Subareas

## 1. Session Identity / 会话身份层

- `routing/session-key.ts`
- `sessions/session-key-utils.ts`
- `sessions/session-id.ts`
- `sessions/session-id-resolution.ts`

回答的问题：**这个会话到底是谁？系统内部怎么标识它？**  
Question answered: **who is this session, and how does the system identify it internally?**

## 2. Route Resolution / 路由决策层

- `routing/account-id.ts`
- `routing/bindings.ts`
- `routing/resolve-route.ts`
- `routing/account-lookup.ts`

回答的问题：**这条消息应该交给哪个 agent / account / session？**  
Question answered: **which agent/account/session should own this message?**

## 3. Session Lifecycle / 生命周期层

- `session-lifecycle-events.ts`
- `transcript-events.ts`
- `send-policy.ts`

回答的问题：**会话变化怎样被记录、广播、约束发送？**  
Question answered: **how are session changes recorded, broadcast, and constrained by send policy?**

## 4. Auto-Reply Ingress / 自动回复入口层

- `auto-reply/dispatch.ts`
- `auto-reply/reply/inbound-context.ts`
- `auto-reply/command-detection.ts`

回答的问题：**一条 inbound message 怎样被标准化并进入 reply pipeline？**  
Question answered: **how is an inbound message normalized and admitted into the reply pipeline?**

## 5. Reply Runtime / 回复运行时层

- `auto-reply/reply.ts`
- `auto-reply/fallback-state.ts`
- `auto-reply/heartbeat.ts`

回答的问题：**系统如何做回复决策、fallback、heartbeat 驱动行为？**  
Question answered: **how does the system make reply decisions, handle fallback, and support heartbeat-driven behavior?**

---

## 五、最重要的依赖链 / Most Important Dependency Chains

## 链路 1：session key 构造与解析链 / Session key construction and parsing chain

```text
routing/session-key.ts
  -> sessions/session-key-utils.ts
  -> resolve-route.ts / session lifecycle consumers
```

### 中文解释
这条链说明 session key 不是一个普通 ID，而是一种结构化路由身份。

### English explanation
This chain shows that the session key is not a plain id but a structured routing identity.

## 链路 2：route resolution 链 / Route resolution chain

```text
account-id.ts
  -> bindings.ts
  -> resolve-route.ts
  -> resolved agentId/sessionKey/mainSessionKey
```

### 中文解释
这条链说明 route resolution 本质上是在 account/channel/peer 维度上做多层匹配。

### English explanation
This chain shows that route resolution is fundamentally a layered match across account/channel/peer dimensions.

## 链路 3：auto-reply 入口链 / Auto-reply ingress chain

```text
dispatch.ts
  -> reply/inbound-context.ts
  -> reply.ts
```

### 中文解释
这条链说明一条 inbound message 先被标准化，再进入真正的 reply runtime。

### English explanation
This chain shows that an inbound message is normalized before entering the real reply runtime.

## 链路 4：reply runtime 链 / Reply runtime chain

```text
reply.ts
  -> session init / route-aware context
  -> agent runner
  -> transcript events / send policy
```

### 中文解释
这条链说明 auto-reply 不是一个小工具函数，而是 Gateway 与 Agent Runtime 的关键中间层。

### English explanation
This chain shows that auto-reply is not a tiny helper function; it is a key middle layer between Gateway and the Agent Runtime.

## 链路 5：fallback 链 / Fallback chain

```text
reply.ts
  -> fallback-state.ts
  -> adjusted model/runtime behavior
  -> visible reply state
```

### 中文解释
这条链解释了为什么 auto-reply 能在模型异常或 provider 变化时继续工作。

### English explanation
This chain explains why auto-reply can continue working when models fail or providers shift.

---

## 六、如果你时间有限，最少读哪些 / Minimal Must-Read Set

如果你只有 2 小时，建议优先读这 8 个文件：

If you only have 2 hours, prioritize these 8 files:

1. `routing/session-key.ts`
2. `routing/resolve-route.ts`
3. `sessions/session-key-utils.ts`
4. `sessions/session-id-resolution.ts`
5. `sessions/transcript-events.ts`
6. `auto-reply/dispatch.ts`
7. `auto-reply/reply/inbound-context.ts`
8. `auto-reply/reply.ts`

这 8 个文件足够让你理解：

- session 怎么被标识
- route 怎么决定归属
- auto-reply 怎么从消息走向 reply
- transcript 怎么被更新

These eight files are enough to understand session identity, route ownership, how auto-reply turns a message into a reply, and how transcript updates propagate.

---

## 七、总结 / Final Summary

`src/sessions/`、`src/routing/`、`src/auto-reply/` 共同构成了 OpenClaw 的 **中间编排层**。

它们不直接做模型推理，也不负责控制面 admission，但它们决定：

- 一条消息属于谁
- 落在哪个会话
- 要不要回复
- 回复怎么走回系统状态

`src/sessions/`, `src/routing/`, and `src/auto-reply/` together form OpenClaw’s **middle orchestration layer**. They do not directly perform model inference and they are not the admission boundary, but they determine who owns a message, which session it lands in, whether the system should reply, and how the reply flows back into system state.
