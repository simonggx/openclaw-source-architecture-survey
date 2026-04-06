# OpenClaw 源码架构调研 / OpenClaw Source Architecture Survey

这个仓库保存了对 **OpenClaw** 源码的系统化调研文档。文档以 **中文为主、英文为辅**，目标是把 OpenClaw 从“一个很大的代码仓库”变成“可以按主题阅读、按链路理解、按子系统深入”的知识库。

This repository contains a structured architecture survey of the **OpenClaw** source code, written **Chinese-first with English support**. The goal is to turn OpenClaw from “a large codebase” into a readable knowledge base organized by theme, execution flow, and subsystem.

## 文档索引 / Documents

### 1. 总览与导读 / Overview and Orientation

- `openclaw-source-architecture-survey.md`  
  总览版调研 / high-level architecture survey

- `openclaw-source-guide.md`  
  源码导读版：关键目录、关键文件逐个解释、推荐阅读顺序、依赖关系  
  Source guide: key directories, file-by-file explanations, reading order, dependency chains

- `openclaw-layered-architecture.md`  
  架构图版：总图 + Gateway 图 + Agent Runtime 图 + Plugin System 图  
  Diagram-oriented architecture: overall view + Gateway + Agent Runtime + Plugin System

### 2. 子系统深挖 / Subsystem Deep Dives

- `openclaw-gateway-deep-dive.md`  
  Gateway 深度导读 / Gateway deep dive

- `openclaw-agents-deep-dive.md`  
  Agent Runtime 深度导读 / Agent Runtime deep dive

- `openclaw-plugin-system-deep-dive.md`  
  Plugin System 深度导读 / Plugin System deep dive

- `openclaw-sessions-routing-auto-reply-deep-dive.md`  
  Sessions / Routing / Auto-Reply 深度导读  
  Sessions / Routing / Auto-Reply deep dive

- `openclaw-ui-platform-deep-dive.md`  
  UI / Apps / OpenClawKit / Swabble 深度导读  
  UI / Apps / OpenClawKit / Swabble deep dive

- `openclaw-runtime-substrate-deep-dive.md`  
  Config / Secrets / Security / Process / Cron / Logging 深度导读  
  Runtime substrate deep dive

### 3. 运行链路与阅读路径 / Flow and Navigation

- `openclaw-execution-flows.md`  
  执行链与时序文档：从 inbound message 到 reply、从 plugin discovery 到 registry activation  
  Execution-flow guide with sequence diagrams

- `openclaw-doc-index-and-reading-paths.md`  
  总索引与阅读路径指南 / master index and reading-path guide

## 上游仓库 / Upstream Repository

- https://github.com/openclaw/openclaw

## 阅读建议 / Suggested Reading Order

### 快速概览 / Quick Overview

1. `openclaw-source-architecture-survey.md`
2. `openclaw-layered-architecture.md`

### 架构理解 / Architecture Understanding

1. `openclaw-source-architecture-survey.md`
2. `openclaw-source-guide.md`
3. `openclaw-layered-architecture.md`
4. `openclaw-doc-index-and-reading-paths.md`

### 运行时 internals / Runtime Internals

1. `openclaw-gateway-deep-dive.md`
2. `openclaw-sessions-routing-auto-reply-deep-dive.md`
3. `openclaw-agents-deep-dive.md`
4. `openclaw-runtime-substrate-deep-dive.md`
5. `openclaw-execution-flows.md`

### 扩展与平台 / Extensions and Platform

1. `openclaw-plugin-system-deep-dive.md`
2. `openclaw-ui-platform-deep-dive.md`
3. `openclaw-execution-flows.md`

## 调研范围 / Scope

- Gateway 控制面 / Gateway control plane
- Agent Runtime 执行面 / agent execution plane
- Plugin SDK 与扩展生态 / plugin SDK and extension ecosystem
- 会话、路由、自动回复链路 / sessions, routing, and auto-reply pipeline
- Web UI、原生客户端、移动节点 / web UI, native clients, and mobile nodes
- Config / Secrets / Security / Process / Cron / Logging 运行时底座  
  Config / Secrets / Security / Process / Cron / Logging runtime substrate
