# OpenClaw UI / Apps / OpenClawKit / Swabble 深度导读 / OpenClaw UI, Apps, OpenClawKit, and Swabble Deep Dive

这篇文档专门深挖 OpenClaw 的“产品表面”层，覆盖四块：

- `ui/`
- `apps/`
- `apps/shared/OpenClawKit`
- `Swabble/`

如果说 Gateway 是控制面中枢、Agents 是执行面引擎，那么这部分就是 OpenClaw 面向用户和设备的 **产品接触面**。

This document focuses on OpenClaw’s product surface across `ui/`, `apps/`, `apps/shared/OpenClawKit`, and `Swabble/`. If Gateway is the control-plane center and Agents are the execution engine, this layer is OpenClaw’s **human and device contact surface**.

---

## 一、先给结论 / Executive Takeaway

OpenClaw 的 UI / Apps 层不是一套孤立前端，而是一组共同围绕 Gateway 协议展开的客户端和节点表面：

- Web Control UI：浏览器控制台
- Android / iOS / macOS：设备与原生客户端
- OpenClawKit：Apple 侧共享协议与运行时能力
- Swabble：更偏本地语音唤醒/语音触发能力

These are not isolated frontends. They are a family of client and node surfaces built around the Gateway protocol.

---

## 二、推荐阅读顺序 / Recommended Reading Order

### 第一轮：先看 Web Control UI / Pass 1: web Control UI first

1. `ui/package.json`
2. `ui/src/ui/gateway.ts`
3. `ui/src/ui/app.ts`
4. `ui/src/ui/app-render.ts`
5. `ui/src/ui/navigation.ts`

### 第二轮：看 Apple 共享层 / Pass 2: shared Apple layer

6. `apps/shared/OpenClawKit/Package.swift`
7. `apps/shared/OpenClawKit/Sources/OpenClawKit/GatewayNodeSession.swift`
8. `apps/shared/OpenClawKit/Sources/OpenClawKit/DeviceCommands.swift`
9. `apps/shared/OpenClawKit/Sources/OpenClawKit/TalkCommands.swift`

### 第三轮：看平台端实现 / Pass 3: platform-specific clients

10. `apps/android/app/src/main/java/ai/openclaw/app/GatewaySession.kt`
11. `apps/ios/Sources/GatewayConnectionController.swift`
12. `apps/macos/Package.swift` 或 `apps/macos/Sources/`

### 第四轮：看语音唤醒补充层 / Pass 4: wake-word companion layer

13. `Swabble/Package.swift`
14. `Swabble/Sources/SwabbleKit/WakeWordGate.swift`
15. `Swabble/Sources/SwabbleCore/Speech/SpeechPipeline.swift`

---

## 三、关键文件逐个解释 / Key Files Explained One by One

## 1. `ui/package.json`

### 中文
这个文件定义了 Control UI 的技术栈：Vite + Lit + TypeScript + Vitest/Playwright。它先告诉你这不是 React 风格的 SPA，而是基于 Web Components 的控制台。

### English
This file defines the Control UI stack: Vite + Lit + TypeScript + Vitest/Playwright. It tells you early that this is not a React-style SPA, but a Web Components-based control console.

---

## 2. `ui/src/ui/gateway.ts`

### 中文
这是 Web Control UI 与后端 Gateway 通信的核心桥梁。它负责 WebSocket/协议层交互，是前端能不能“像一个控制面客户端那样工作”的关键文件。

### English
This is the core bridge between the web Control UI and the backend Gateway. It handles the WebSocket/protocol interaction and is central to making the frontend behave like a real control-plane client.

---

## 3. `ui/src/ui/app.ts`

### 中文
这是主应用组件，负责聚合大量全局状态：连接、聊天、节点、tabs、主题等。它是浏览器端的控制面入口壳。

### English
This is the main application component, aggregating global state such as connection, chat, nodes, tabs, and theme. It is the browser-side shell of the control plane.

---

## 4. `ui/src/ui/app-render.ts`

### 中文
这个文件是 UI 渲染逻辑的核心总装配。它体量大，说明 UI 不是几个独立页面，而是一个大型控制面视图系统。

### English
This file is the main rendering assembly for the UI. Its large size suggests the UI is not a handful of isolated pages but a substantial control-plane view system.

---

## 5. `ui/src/ui/navigation.ts`

### 中文
这个文件定义 UI 的导航结构。它能帮助你快速知道 Control UI 想把系统能力切成哪些操控面板。

### English
This file defines the UI navigation structure. It quickly tells you how the Control UI divides system capabilities into operator panels.

---

## 6. `apps/shared/OpenClawKit/Package.swift`

### 中文
这个文件是 Apple 侧共享层最重要的入口。它直接声明了三个产品：`OpenClawProtocol`、`OpenClawKit`、`OpenClawChatUI`。只看它，你就知道 iOS/macOS 不是各写各的，而是走共享协议 + 共享运行时 + 共享 UI 的路线。

### English
This file is the key entrypoint for the Apple shared layer. It declares three products: `OpenClawProtocol`, `OpenClawKit`, and `OpenClawChatUI`. From this alone, you can see that iOS/macOS are not built independently but share protocol, runtime, and UI layers.

---

## 7. `apps/shared/OpenClawKit/Sources/OpenClawKit/GatewayNodeSession.swift`

### 中文
这是 Apple 侧连接 Gateway 的核心文件之一。它描述了设备/客户端如何作为 node 或会话终端与 Gateway 协议交互。

### English
This is one of the core Apple-side files for connecting to the Gateway. It describes how a device/client behaves as a node or session endpoint against the Gateway protocol.

---

## 8. `apps/shared/OpenClawKit/Sources/OpenClawKit/DeviceCommands.swift`

### 中文
这个文件很重要，因为它让你看到“设备能力”在客户端侧是怎样被建模的。OpenClaw 不是只有聊天，还把设备能力变成可调用命令。

### English
This file is important because it shows how device capabilities are modeled on the client side. OpenClaw is not only about chat; it turns device capabilities into callable commands.

---

## 9. `apps/shared/OpenClawKit/Sources/OpenClawKit/TalkCommands.swift`

### 中文
这是语音/Talk 侧的重要协议与命令定义文件。它说明 Apple 共享层不仅包含 Gateway session，还包含音频/语音交互语义。

### English
This is an important protocol and command-definition file for voice/Talk behavior. It shows that the Apple shared layer includes not only Gateway sessions but also speech/voice interaction semantics.

---

## 10. `apps/android/app/src/main/java/ai/openclaw/app/GatewaySession.kt`

### 中文
这是 Android 侧的 Gateway session 实现核心。它回答 Android 设备如何接入 Gateway、维护会话、参与节点能力执行。

### English
This is the core Android-side implementation of Gateway sessions. It explains how Android devices connect to the Gateway, maintain sessions, and participate in node capability execution.

---

## 11. `apps/ios/Sources/GatewayConnectionController.swift`

### 中文
这是 iOS 侧的连接控制核心之一。它说明 iPhone 端如何把连接、状态和系统表面（如 Share Extension / Widget / Watch）组织起来。

### English
This is one of the core connection-control files on iOS. It shows how the iPhone side organizes connection state and system surfaces such as share extensions, widgets, and watch integration.

---

## 12. `apps/macos/Package.swift`

### 中文
这是 macOS 侧 package 切分的入口。它有助于看清 macOS app、CLI、IPC、discovery 等部分是如何被拆开的。

### English
This is the entrypoint for macOS-side package decomposition. It helps clarify how the macOS app, CLI, IPC, and discovery components are separated.

---

## 13. `Swabble/Package.swift`

### 中文
这是 Swabble 项目的入口。它告诉你 Swabble 不是一个随手塞进仓库的小工具，而是一个独立的 Swift package / voice companion。

### English
This is the entrypoint of the Swabble project. It shows that Swabble is not a throwaway helper tucked into the repo, but an independent Swift package and voice companion.

---

## 14. `Swabble/Sources/SwabbleKit/WakeWordGate.swift`

### 中文
这是 Swabble 的唤醒词核心文件之一。它定义 wake-word 检测与触发逻辑，体现了 OpenClaw 对本地唤醒路径的支持。

### English
This is one of the core wake-word files in Swabble. It defines wake-word detection and trigger logic, showing OpenClaw’s support for local wake flows.

---

## 15. `Swabble/Sources/SwabbleCore/Speech/SpeechPipeline.swift`

### 中文
这个文件位于更底层的语音处理管线。它把音频采集、识别和唤醒处理串起来，是理解 Swabble 的关键文件之一。

### English
This file sits lower in the speech-processing stack. It connects audio capture, speech recognition, and wake handling, making it one of the key files for understanding Swabble.

---

## 四、按子系统看 / UI and Platform Subareas

## 1. Web Control UI

- `ui/package.json`
- `ui/src/ui/gateway.ts`
- `ui/src/ui/app.ts`
- `ui/src/ui/app-render.ts`
- `ui/src/ui/navigation.ts`

回答的问题：**浏览器控制台如何成为 Gateway 的一个控制面客户端？**  
Question answered: **how does the browser console become a control-plane client of the Gateway?**

## 2. Apple 共享层 / Shared Apple Layer

- `apps/shared/OpenClawKit/Package.swift`
- `GatewayNodeSession.swift`
- `DeviceCommands.swift`
- `TalkCommands.swift`

回答的问题：**iOS/macOS 共用的协议、命令和 UI 基础在哪里？**  
Question answered: **where do iOS/macOS share protocol, command, and UI foundations?**

## 3. Android / iOS / macOS 平台实现

- `apps/android/.../GatewaySession.kt`
- `apps/ios/Sources/GatewayConnectionController.swift`
- `apps/macos/Package.swift`

回答的问题：**各平台如何接入共享层，并落地为本地设备/客户端？**  
Question answered: **how do platform-specific clients build on the shared layer and become local devices/clients?**

## 4. Swabble 语音唤醒层

- `Swabble/Package.swift`
- `WakeWordGate.swift`
- `SpeechPipeline.swift`

回答的问题：**本地唤醒词和语音触发如何形成独立能力层？**  
Question answered: **how do local wake-word and speech triggers form an independent capability layer?**

---

## 五、最重要的依赖链 / Most Important Dependency Chains

## 链路 1：Web UI 到 Gateway / Web UI to Gateway

```text
ui app.ts / app-render.ts
  -> ui gateway.ts
  -> Gateway protocol
  -> backend Gateway
```

### 中文解释
这条链说明 Web UI 不是“静态页面”，而是协议驱动的控制面客户端。

### English explanation
This chain shows that the web UI is not a static page but a protocol-driven control-plane client.

## 链路 2：Apple 共享层链 / Shared Apple layer chain

```text
Package.swift
  -> OpenClawProtocol
  -> OpenClawKit
  -> OpenClawChatUI
  -> iOS/macOS consumers
```

### 中文解释
这条链解释了 Apple 平台代码为什么能维持一致性：协议、运行时和 UI 被显式拆层。

### English explanation
This chain explains how Apple-side code stays coherent: protocol, runtime, and UI are explicitly layered.

## 链路 3：Android / iOS 到 Gateway / Mobile clients to Gateway

```text
GatewaySession.kt / GatewayConnectionController.swift
  -> shared protocol/session logic
  -> Gateway control plane
```

### 中文解释
这条链说明移动端不是旁路系统，而是 Gateway 的正式客户端/节点。

### English explanation
This chain shows that mobile clients are not side systems but formal clients/nodes of the Gateway.

## 链路 4：设备能力链 / Device capability chain

```text
DeviceCommands.swift
  -> platform runtime
  -> device APIs
  -> result returns to Gateway session
```

### 中文解释
这条链说明 OpenClaw 把设备能力建模成可调用命令，而不是 UI 按钮背后的私有逻辑。

### English explanation
This chain shows that OpenClaw models device capabilities as callable commands rather than UI-only private logic.

## 链路 5：Swabble 唤醒词链 / Swabble wake-word chain

```text
WakeWordGate.swift
  -> SpeechPipeline.swift
  -> trigger/hook behavior
  -> OpenClaw-facing local voice flow
```

### 中文解释
这条链解释了 Swabble 如何成为 OpenClaw 的本地语音入口补充层。

### English explanation
This chain explains how Swabble becomes a local voice-entry companion for OpenClaw.

---

## 六、如果你时间有限，最少读哪些 / Minimal Must-Read Set

如果你只有 2 小时，建议优先读这 8 个文件：

If you only have 2 hours, prioritize these 8 files:

1. `ui/src/ui/gateway.ts`
2. `ui/src/ui/app.ts`
3. `ui/src/ui/app-render.ts`
4. `apps/shared/OpenClawKit/Package.swift`
5. `GatewayNodeSession.swift`
6. `DeviceCommands.swift`
7. `apps/android/.../GatewaySession.kt`
8. `Swabble/Sources/SwabbleKit/WakeWordGate.swift`

这些文件足够让你理解：

- Web UI 如何接 Gateway
- Apple 共享层如何构成平台骨架
- 移动端如何作为节点加入系统
- Swabble 如何补本地语音入口

These files are enough to understand how the web UI connects to Gateway, how the Apple shared layer forms the platform skeleton, how mobile clients join as nodes, and how Swabble extends local voice entry.

---

## 七、总结 / Final Summary

OpenClaw 的 UI / Apps / OpenClawKit / Swabble 共同形成了一个很有特点的产品面：

- Web 是控制台
- Mobile/Desktop 是客户端与节点
- OpenClawKit 是 Apple 共享底座
- Swabble 是本地语音入口补充

Together, OpenClaw’s UI, apps, OpenClawKit, and Swabble form a distinctive product surface: the web acts as the console, mobile/desktop act as clients and nodes, OpenClawKit is the Apple shared foundation, and Swabble extends local voice entry.
