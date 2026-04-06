# OpenClaw Config / Secrets / Security / Process / Cron / Logging 深度导读 / OpenClaw Runtime Substrate Deep Dive

这篇文档专门讲 OpenClaw 的运行时底座，覆盖：

- `src/config/`
- `src/secrets/`
- `src/security/`
- `src/process/`
- `src/cron/`
- `src/logging/`

如果说 Gateway、Agents、Plugins 是显性系统骨架，那么这部分就是 OpenClaw 的 **运行时地基**。

This document focuses on OpenClaw’s runtime substrate across config, secrets, security, process, cron, and logging. If Gateway, Agents, and Plugins are the visible system skeleton, this layer is the **runtime foundation**.

---

## 一、先给结论 / Executive Takeaway

这套底座的任务不是直接做业务，而是保证系统能：

- 正确读取配置
- 安全解析 secrets
- 做运行时安全审计
- 管理进程与 PTY
- 调度定时工作
- 输出结构化日志

These subsystems do not directly implement end-user features. They ensure the system can load config correctly, resolve secrets safely, audit runtime security, manage processes and PTYs, schedule work, and emit structured logs.

---

## 二、推荐阅读顺序 / Recommended Reading Order

### 第一轮：先看配置和快照 / Pass 1: config and snapshots

1. `src/config/config.ts`
2. `src/config/io.ts`
3. `src/config/runtime-snapshot.ts`
4. `src/config/schema-base.ts`
5. `src/config/env-substitution.ts`

### 第二轮：看 secrets 与安全 / Pass 2: secrets and security

6. `src/secrets/runtime.ts`
7. `src/secrets/resolve.ts`
8. `src/security/audit.ts`
9. `src/security/audit-fs.ts`
10. `src/security/audit-tool-policy.ts`

### 第三轮：看进程与调度 / Pass 3: process and scheduling

11. `src/process/exec.ts`
12. `src/process/supervisor/supervisor.ts`
13. `src/cron/service.ts`
14. `src/cron/isolated-agent/run.ts`

### 第四轮：看日志 / Pass 4: logging

15. `src/logging/config.ts`
16. `src/logging/logger.ts`
17. `src/logging/subsystem.ts`
18. `src/logging/redact.ts`

---

## 三、关键文件逐个解释 / Key Files Explained One by One

## 1. `src/config/config.ts`

### 中文
这是 config 层的总出口。它本身不堆逻辑，而是把 `io.ts`、`mutate.ts`、`validation.ts` 等对外重新导出，形成统一入口。读它的意义在于确认 config 子系统的外部 API 面。

### English
This is the top-level export surface of the config layer. It does not contain heavy logic itself; instead, it re-exports `io.ts`, `mutate.ts`, `validation.ts`, and related modules as the unified config API.

---

## 2. `src/config/io.ts`

### 中文
这是 config 真正的核心文件之一。配置读取、缓存、runtime snapshot、source snapshot、文件写入等重逻辑都在这里。想理解 OpenClaw 如何把配置从磁盘带进运行时，必须读它。

### English
This is one of the real core config files. Config loading, caching, runtime snapshots, source snapshots, and file writes all happen here. If you want to understand how OpenClaw brings config from disk into runtime, read this file.

---

## 3. `src/config/runtime-snapshot.ts`

### 中文
这个文件说明配置并不是每次都临时现读，而是会被投影成运行时快照。它是“配置层”和“运行时层”之间的重要桥梁。

### English
This file shows that config is not always read directly in ad hoc fashion; it is projected into runtime snapshots. It is an important bridge between the config layer and the runtime layer.

---

## 4. `src/config/schema-base.ts`

### 中文
这是 schema 基础定义的重要文件。它决定 OpenClaw 配置对象的基础结构、schema 输出和 hints，是“配置长什么样”的核心来源。

### English
This is a foundational schema-definition file. It shapes the base config structure, schema output, and hints, making it central to what OpenClaw configuration looks like.

---

## 5. `src/config/env-substitution.ts`

### 中文
这个文件处理 `${VAR}` 类环境变量替换。它说明配置系统不是纯静态 JSON，而是允许一定程度的环境驱动动态化。

### English
This file handles `${VAR}`-style environment substitution. It shows that the config system is not purely static JSON, but supports a degree of environment-driven dynamism.

---

## 6. `src/secrets/runtime.ts`

### 中文
这是 secrets runtime 的主入口。它把 secret resolution 的结果组织成可激活的运行时快照，是 secrets 子系统的关键控制点。

### English
This is the main entrypoint for the secrets runtime. It organizes secret-resolution results into activatable runtime snapshots, making it a key control point of the secrets subsystem.

---

## 7. `src/secrets/resolve.ts`

### 中文
这个文件是真正的 secret resolution 引擎，负责处理 env/file/exec 等来源。它回答“一个 secret ref 最终怎样变成真实值”。

### English
This file is the real secret-resolution engine, handling env/file/exec sources. It answers how a secret reference ultimately becomes a real value.

---

## 8. `src/security/audit.ts`

### 中文
这是安全审计主入口。它会检查文件权限、gateway 配置、exec 安全、browser 控制等，是 OpenClaw 自检能力的重要一环。

### English
This is the main security-audit entrypoint. It checks file permissions, gateway config, exec safety, browser controls, and more, making it an important self-inspection layer in OpenClaw.

---

## 9. `src/security/audit-fs.ts`

### 中文
这个文件聚焦文件系统权限与目录安全。很多平台系统最后的安全问题都落在权限边界上，所以它非常关键。

### English
This file focuses on filesystem permissions and directory safety. Many platform-level security issues eventually reduce to permission boundaries, so it is especially important.

---

## 10. `src/security/audit-tool-policy.ts`

### 中文
这个文件把安全审计与工具策略连接起来，说明 OpenClaw 把“工具可不可以这样跑”当成安全层的一部分，而不是只靠业务逻辑兜底。

### English
This file connects security auditing with tool policy, showing that OpenClaw treats “whether a tool may run this way” as part of the security layer rather than relying only on business logic.

---

## 11. `src/process/exec.ts`

### 中文
这是低层命令执行核心文件之一。它处理 spawn/execFile、Windows shim、安全 shell 策略、timeout、输出等，是执行子系统的底层入口。

### English
This is one of the core low-level command execution files. It handles spawn/execFile, Windows shims, safe shell policy, timeouts, and outputs, making it the low-level entrypoint of execution.

---

## 12. `src/process/supervisor/supervisor.ts`

### 中文
这个文件提供进程监督器 abstraction。它把 child/pty 两类运行模式统一到一个 supervisor 接口下，是高层执行系统能稳定依赖进程管理的关键。

### English
This file provides the process-supervisor abstraction. It unifies child and PTY execution modes behind one interface, which is key for stable higher-level process management.

---

## 13. `src/cron/service.ts`

### 中文
这是 Cron 服务入口。它说明 OpenClaw 不是只做即时响应，而是有自己的计划任务/调度系统。

### English
This is the Cron service entrypoint. It shows that OpenClaw is not only about immediate interaction, but also has its own scheduled job system.

---

## 14. `src/cron/isolated-agent/run.ts`

### 中文
这个文件很关键，因为它说明 scheduled work 最终如何变成一次独立的 agent turn。它是 cron 与 agent runtime 的接缝。

### English
This file is important because it shows how scheduled work ultimately becomes an isolated agent turn. It is the seam between cron and the agent runtime.

---

## 15. `src/logging/config.ts`

### 中文
这个文件负责 logging 配置读取。它看起来简单，但它决定了日志级别、文件路径、console 行为等基础控制。

### English
This file handles logging config reads. It looks small, but it determines log levels, file paths, and console behavior.

---

## 16. `src/logging/logger.ts`

### 中文
这是日志系统的核心实现文件。它基于 tslog 组装 OpenClaw logger，处理默认文件路径、滚动日志、环境变量 override、silent 测试模式等。

### English
This is the core logging implementation. It builds the OpenClaw logger on top of tslog, handling default log paths, rolling logs, env overrides, and silent test behavior.

---

## 17. `src/logging/subsystem.ts`

### 中文
这个文件提供 subsystem logger，让不同模块可以有自己的日志命名空间。它是 OpenClaw 日志可读性的重要来源。

### English
This file provides subsystem loggers so that different modules can have named logging namespaces. It is a major contributor to log readability across OpenClaw.

---

## 18. `src/logging/redact.ts`

### 中文
日志系统如果不做脱敏，很快就会变成泄密面。这个文件就是日志安全的一部分，负责 redaction。

### English
Without redaction, a logging system quickly becomes a leakage surface. This file is part of logging safety and handles redaction.

---

## 四、按子系统看 / Runtime Substrate Subareas

## 1. Config Loading / 配置加载层

- `config.ts`
- `io.ts`
- `runtime-snapshot.ts`
- `schema-base.ts`
- `env-substitution.ts`

回答的问题：**配置如何被读入、校验、快照化？**  
Question answered: **how is config loaded, validated, and projected into runtime snapshots?**

## 2. Secrets Runtime / Secret 解析层

- `secrets/runtime.ts`
- `secrets/resolve.ts`

回答的问题：**secret ref 如何从配置变成可用真实值？**  
Question answered: **how do secret refs become usable real values?**

## 3. Security Audit / 安全审计层

- `security/audit.ts`
- `security/audit-fs.ts`
- `security/audit-tool-policy.ts`

回答的问题：**系统如何在运行前后检查自己的安全姿态？**  
Question answered: **how does the system inspect its own security posture before and during runtime?**

## 4. Process Substrate / 进程执行层

- `process/exec.ts`
- `process/supervisor/supervisor.ts`

回答的问题：**命令怎样以安全、可控、可终止的方式执行？**  
Question answered: **how are commands executed in a safe, controllable, and stoppable way?**

## 5. Scheduled Work / 调度层

- `cron/service.ts`
- `cron/isolated-agent/run.ts`

回答的问题：**定时任务怎样落成一次独立运行的 agent turn？**  
Question answered: **how does a scheduled job become an isolated agent turn?**

## 6. Logging / 日志层

- `logging/config.ts`
- `logging/logger.ts`
- `logging/subsystem.ts`
- `logging/redact.ts`

回答的问题：**系统怎样输出可观测、分层、可脱敏的日志？**  
Question answered: **how does the system emit observable, layered, redactable logs?**

---

## 五、最重要的依赖链 / Most Important Dependency Chains

## 链路 1：配置到运行时快照 / Config to runtime snapshot

```text
config.ts
  -> io.ts
  -> runtime-snapshot.ts
  -> runtime consumers
```

### 中文解释
这条链说明配置不是只在读取那一刻有意义，而是会被变成运行时快照供全局消费。

### English explanation
This chain shows that config matters not only at read time, but is projected into runtime snapshots consumed across the system.

## 链路 2：secret ref 解析链 / Secret resolution chain

```text
config values with secret refs
  -> secrets/runtime.ts
  -> secrets/resolve.ts
  -> resolved runtime values
```

### 中文解释
这条链解释了配置与 secret resolution 之间的关键桥梁。

### English explanation
This chain explains the critical bridge between config values and secret resolution.

## 链路 3：安全审计链 / Security audit chain

```text
security/audit.ts
  -> audit-fs.ts
  -> audit-tool-policy.ts
  -> security report / enforcement decisions
```

### 中文解释
这条链说明安全不是单一规则，而是一组 filesystem、tool policy、runtime posture 的组合检查。

### English explanation
This chain shows that security is not one rule but a combined inspection across filesystem, tool policy, and runtime posture.

## 链路 4：进程执行链 / Process execution chain

```text
exec.ts
  -> supervisor.ts
  -> child/pty execution
  -> managed run state
```

### 中文解释
这条链说明 OpenClaw 的命令执行不是直接裸跑，而是被 supervisor abstraction 包住。

### English explanation
This chain shows that command execution in OpenClaw is not raw process launching, but wrapped by a supervisor abstraction.

## 链路 5：cron 到 agent run 链 / Cron to agent run chain

```text
cron/service.ts
  -> cron/isolated-agent/run.ts
  -> isolated agent turn
  -> delivery / transcript / status effects
```

### 中文解释
这条链解释了 Scheduled Work 如何真正进入系统执行面。

### English explanation
This chain explains how scheduled work truly enters the execution plane of the system.

## 链路 6：日志链 / Logging chain

```text
logging/config.ts
  -> logging/logger.ts
  -> logging/subsystem.ts
  -> redact.ts
  -> file/console output
```

### 中文解释
这条链说明日志不是简单打印，而是配置驱动、分 subsystem、带脱敏的可观测系统。

### English explanation
This chain shows that logging is not mere printing, but a config-driven, subsystem-aware, redactable observability system.

---

## 六、如果你时间有限，最少读哪些 / Minimal Must-Read Set

如果你只有 2 小时，建议优先读这 8 个文件：

If you only have 2 hours, prioritize these 8 files:

1. `src/config/io.ts`
2. `src/config/runtime-snapshot.ts`
3. `src/secrets/runtime.ts`
4. `src/secrets/resolve.ts`
5. `src/security/audit.ts`
6. `src/process/exec.ts`
7. `src/cron/service.ts`
8. `src/logging/logger.ts`

这 8 个文件足够让你理解：

- 配置怎样变成运行时状态
- secrets 怎样被解析
- 系统怎样做自检
- 进程怎样安全运行
- 定时任务怎样驱动 agent turn
- 日志怎样落盘并保持可观测性

These eight files are enough to understand how config becomes runtime state, how secrets resolve, how the system audits itself, how processes run safely, how scheduled jobs trigger agent turns, and how logs are persisted and observed.

---

## 七、总结 / Final Summary

OpenClaw 的运行时底座并不显眼，但它决定了整个平台是不是“真平台”：

- config 决定系统可配置性
- secrets 决定安全接入外部能力
- security 决定系统是否有自检能力
- process 决定执行是否可控
- cron 决定系统能否做后台自动化
- logging 决定系统是否可观测

OpenClaw’s runtime substrate is not the most visible part of the codebase, but it is what makes the platform a real platform: config provides configurability, secrets provide secure external access, security provides self-audit, process management provides controlled execution, cron enables background automation, and logging provides observability.
