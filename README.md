# OpenClaw 源码架构调研 / OpenClaw Source Architecture Survey

这个仓库保存了对 **OpenClaw** 源码的系统化调研文档。文档以 **中文为主、英文为辅**，方便先快速理解整体，再深入到具体目录、关键文件与架构关系。

This repository contains a structured architecture survey of the **OpenClaw** source code, written **Chinese-first with English support**.

## 文档索引 / Documents

- `openclaw-source-architecture-survey.md`  
  总览版调研 / high-level architecture survey

- `openclaw-source-guide.md`  
  源码导读版：关键目录、关键文件逐个解释、推荐阅读顺序、依赖关系  
  Source guide: key directories, file-by-file explanations, reading order, dependency chains

- `openclaw-layered-architecture.md`  
  架构图版：总图 + Gateway 图 + Agent Runtime 图 + Plugin System 图  
  Diagram-oriented architecture: overall view + Gateway + Agent Runtime + Plugin System

## 上游仓库 / Upstream Repository

- https://github.com/openclaw/openclaw

## 阅读建议 / Suggested Reading Order

1. `openclaw-source-architecture-survey.md`
2. `openclaw-source-guide.md`
3. `openclaw-layered-architecture.md`

## 调研范围 / Scope

- Gateway 控制面 / Gateway control plane
- Agent Runtime 执行面 / agent execution plane
- Plugin SDK 与扩展生态 / plugin SDK and extension ecosystem
- 会话、路由、自动回复链路 / sessions, routing, and auto-reply pipeline
- Web UI、原生客户端、移动节点 / web UI, native clients, and mobile nodes
