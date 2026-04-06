# OpenClaw Plugin System 深度导读 / OpenClaw Plugin System Deep Dive

这篇文档专门深挖 OpenClaw 的插件系统，覆盖三个核心区域：

- `src/plugins/`
- `src/plugin-sdk/`
- `extensions/`

如果说 Gateway 是控制面中枢，Agents 是执行面引擎，那么 Plugin System 就是 OpenClaw 的 **可扩展能力平面**。

This document focuses on the OpenClaw plugin system across `src/plugins/`, `src/plugin-sdk/`, and `extensions/`. If Gateway is the control-plane center and Agents are the execution engine, the plugin system is the **capability expansion plane**.

---

## 一、先给结论 / Executive Takeaway

OpenClaw 的插件系统不是“扫目录然后动态 import 一堆模块”这么简单，它有一条很明确的宿主策略：

1. **先发现插件** / discover candidate plugins
2. **先读 manifest，再做校验** / read manifests before executing runtime code
3. **再决定是否启用和如何装载** / decide enablement and loading
4. **最后注册到统一 registry** / register capabilities into a unified registry

这意味着 OpenClaw 的 plugin system 本质上是：

**manifest-first + registry-driven + SDK-bounded**

That means the plugin system is fundamentally **manifest-first, registry-driven, and SDK-bounded**.

---

## 二、推荐阅读顺序 / Recommended Reading Order

### 第一轮：先理解元数据模型 / Pass 1: metadata model first

1. `src/plugins/manifest.ts`
2. `src/plugins/types.ts`
3. `src/plugins/config-state.ts`

### 第二轮：再理解发现与清单注册 / Pass 2: discovery and manifest registration

4. `src/plugins/discovery.ts`
5. `src/plugins/manifest-registry.ts`
6. `src/plugins/schema-validator.ts`
7. `src/plugins/bundled-plugin-scan.ts`

### 第三轮：看装载与运行时 / Pass 3: loading and runtime

8. `src/plugins/loader.ts`
9. `src/plugins/runtime.ts`
10. `src/plugins/runtime-state.ts`
11. `src/plugins/registry.ts`

### 第四轮：看扩展 API 边界 / Pass 4: extension API boundary

12. `src/plugins/api-builder.ts`
13. `src/plugin-sdk/index.ts`
14. `src/plugin-sdk/plugin-entry.ts`
15. `src/plugin-sdk/provider-entry.ts`
16. `src/plugin-sdk/channel-entry-contract.ts`

### 第五轮：看 bundled ecosystem / Pass 5: bundled ecosystem

17. 任选一个 `extensions/*/openclaw.plugin.json`
18. 再读一个对应的 `extensions/*/index.ts`

---

## 三、关键文件逐个解释 / Key Files Explained One by One

## 1. `src/plugins/manifest.ts`

### 中文
这是插件清单模型的核心文件。它定义了 plugin manifest 的结构、字段、contracts、configSchema 等规则。只要想理解“OpenClaw 认为什么才算一个插件”，就必须先读它。

### English
This is the core manifest model file. It defines the structure of plugin manifests, including contracts and config schema rules. If you want to understand what OpenClaw considers a valid plugin, start here.

---

## 2. `src/plugins/types.ts`

### 中文
这是插件系统最底层的类型总表，包含 `OpenClawPluginApi`、`ProviderPlugin`、`ChannelPlugin`、hook/tool/provider 等类型。它是宿主能力面和插件能力面的“类型总说明书”。

### English
This is the master type definition file for the plugin system, containing `OpenClawPluginApi`, `ProviderPlugin`, `ChannelPlugin`, and many related capability types. It is the type-level handbook of the host/plugin boundary.

---

## 3. `src/plugins/config-state.ts`

### 中文
这个文件决定插件的启用状态。也就是说，插件“被发现”不代表插件“会运行”，它还要经过 allow/deny、slot、memory 独占位等配置层判断。

### English
This file decides activation state. Being discovered does not mean a plugin will run; it still must pass allow/deny rules, slot selection, memory exclusivity, and other configuration policies.

---

## 4. `src/plugins/discovery.ts`

### 中文
这是插件发现引擎。它负责扫描 workspace、bundled、global 等来源，找出 candidate plugins。这个文件回答的是：“有哪些插件可能存在？”

### English
This is the plugin discovery engine. It scans workspace, bundled, and global sources to find candidate plugins. It answers: “which plugins may exist?”

---

## 5. `src/plugins/manifest-registry.ts`

### 中文
发现只是第一步，`manifest-registry.ts` 才开始真正把插件“变成宿主可理解的对象”。它负责读取 manifest、验证、归一化、去重，并生成 manifest registry 记录。

### English
Discovery is only the first step. `manifest-registry.ts` is where candidate plugins become host-understandable objects through manifest reading, validation, normalization, de-duplication, and registry record creation.

---

## 6. `src/plugins/schema-validator.ts`

### 中文
这个文件负责配置值的 schema 校验。它的重要性在于：OpenClaw 要尽量在“执行插件代码之前”就尽可能多地理解插件和插件配置。

### English
This file validates configuration values against schemas. It matters because OpenClaw tries to understand as much as possible about a plugin and its config before executing plugin code.

---

## 7. `src/plugins/bundled-plugin-scan.ts`

### 中文
这个文件主要面向 bundled ecosystem，负责扫描 `extensions/` 中的插件入口。它把内置生态接到插件主机的发现流程里。

### English
This file focuses on the bundled ecosystem and scans plugin entries in `extensions/`. It connects the built-in extension ecosystem into the host discovery flow.

---

## 8. `src/plugins/loader.ts`

### 中文
这是插件系统最重要的实现文件之一。它把 discovery、manifest、config、runtime module load、registry registration 串成完整的装载链。理解插件系统，必须理解 `loader.ts`。

### English
This is one of the most important implementation files in the plugin system. It connects discovery, manifests, config, runtime module loading, and registry registration into the full loading chain. Understanding the plugin system requires understanding `loader.ts`.

---

## 9. `src/plugins/runtime.ts`

### 中文
这个文件负责插件运行时状态的关键入口之一，尤其是当前 active registry 之类的全局运行态。它回答的是：“插件系统现在对宿主来说处于什么状态？”

### English
This file is one of the key entrypoints for plugin runtime state, especially global active registry state. It answers: “what is the current runtime state of the plugin system from the host’s perspective?”

---

## 10. `src/plugins/runtime-state.ts`

### 中文
和 `runtime.ts` 一起看，可以帮助理解插件系统在运行态如何记住 registry、channel pin、memory state 等动态信息。

### English
Read together with `runtime.ts`, this file helps explain how the runtime remembers dynamic plugin-system state such as registries, channel pinning, and memory state.

---

## 11. `src/plugins/registry.ts`

### 中文
这是插件注册表的核心。插件最终并不是“加载完就结束”，而是要把 tools、providers、channels、hooks、CLI、HTTP routes、services 等注册到统一 registry。这个文件就是那张总账。

### English
This is the core plugin registry. Plugins do not merely “load and stop”; they must register tools, providers, channels, hooks, CLI entries, HTTP routes, and services into a unified registry. This file is that ledger.

---

## 12. `src/plugins/hooks.ts`

### 中文
插件除了注册静态能力，还能注册生命周期钩子。`hooks.ts` 定义了 hook 的结构与运行方式，是插件“介入宿主生命周期”的关键点。

### English
Plugins can do more than register static capabilities; they can also register lifecycle hooks. `hooks.ts` defines hook structure and execution, making it a key point where plugins can influence host lifecycle behavior.

---

## 13. `src/plugins/api-builder.ts`

### 中文
这个文件非常关键，因为它告诉你宿主究竟给插件暴露了哪些 API。插件作者不是直接碰宿主内部，而是通过这里构造出的 `OpenClawPluginApi` 去注册能力。

### English
This file is crucial because it tells you exactly which API surface the host exposes to plugins. Plugin authors do not touch host internals directly; they use the `OpenClawPluginApi` built here.

---

## 14. `src/plugin-sdk/index.ts`

### 中文
这是 `plugin-sdk` 的根入口，而且刻意保持很薄。它的设计思想很明确：根入口只做公共聚合，具体能力应该落在更窄的子路径上。

### English
This is the root entrypoint of the `plugin-sdk`, and it is intentionally thin. The design is explicit: the root entry aggregates shared public exports, while concrete capabilities live in narrower subpaths.

---

## 15. `src/plugin-sdk/plugin-entry.ts`

### 中文
这个文件定义了“普通插件入口”的契约，是插件作者和宿主之间最直接的协议之一。

### English
This file defines the contract for a general plugin entry, making it one of the most direct agreements between plugin author and host.

---

## 16. `src/plugin-sdk/provider-entry.ts`

### 中文
这个文件定义 provider plugin 的入口契约。只要一个扩展要声明自己是 provider，它通常就得通过这条边界进入系统。

### English
This file defines the entry contract for provider plugins. If an extension wants to behave as a provider, it usually enters through this seam.

---

## 17. `src/plugin-sdk/channel-entry-contract.ts`

### 中文
这个文件定义 channel plugin 的入口契约。OpenClaw 把渠道扩展视为一等能力，所以这里非常重要。

### English
This file defines the entry contract for channel plugins. Channel extensions are first-class capabilities in OpenClaw, so this boundary is very important.

---

## 18. `extensions/*/openclaw.plugin.json`

### 中文
真正理解插件系统，不能只读宿主，还要看 bundled extensions 的 manifest。它们是最真实的“宿主边界如何被实际使用”的样本。

### English
To truly understand the plugin system, you cannot read only the host. You must also read bundled extension manifests, because they are the most concrete examples of how the host boundary is actually used.

---

## 四、按子系统看 / Plugin System Subareas

## 1. 发现系统 / Discovery System

- `discovery.ts`
- `bundled-plugin-scan.ts`
- `bundled-dir.ts`

回答的问题：**有哪些插件候选存在？**  
Question answered: **which candidate plugins exist?**

## 2. Manifest-first 校验系统 / Manifest-First Validation

- `manifest.ts`
- `manifest-registry.ts`
- `schema-validator.ts`

回答的问题：**在执行代码前，宿主能先知道什么？**  
Question answered: **what can the host know before executing plugin code?**

## 3. 装载与运行时 / Loading and Runtime

- `loader.ts`
- `runtime.ts`
- `runtime-state.ts`
- `config-state.ts`

回答的问题：**插件何时启用、如何装载、运行时状态如何被记住？**  
Question answered: **when plugins are activated, how they load, and how runtime state is remembered.**

## 4. 注册与能力汇总 / Registration and Capability Aggregation

- `registry.ts`
- `hooks.ts`
- `api-builder.ts`

回答的问题：**插件的能力最终如何进入系统？**  
Question answered: **how plugin capabilities finally enter the system.**

## 5. SDK 边界 / SDK Boundary

- `plugin-sdk/index.ts`
- `plugin-sdk/plugin-entry.ts`
- `plugin-sdk/provider-entry.ts`
- `plugin-sdk/channel-entry-contract.ts`

回答的问题：**插件作者被允许以什么方式接触宿主？**  
Question answered: **how plugin authors are allowed to touch the host.**

## 6. Bundled 生态 / Bundled Extension Ecosystem

- `extensions/*/openclaw.plugin.json`
- `extensions/*/index.ts`

回答的问题：**官方/内置生态怎样真实使用这套插件系统？**  
Question answered: **how the built-in ecosystem actually uses this plugin system.**

---

## 五、最重要的依赖链 / Most Important Dependency Chains

## 链路 1：发现 → Manifest → 注册表 / Discovery to Manifest to Registry

```text
discovery.ts
  -> manifest.ts
  -> manifest-registry.ts
  -> registry.ts
```

### 中文解释
这条链解释了“候选目录”如何逐步变成“宿主可理解的插件记录”。

### English explanation
This chain explains how a candidate directory gradually becomes a host-understandable plugin record.

## 链路 2：装载链 / Loading chain

```text
loader.ts
  -> config-state.ts
  -> runtime.ts / runtime-state.ts
  -> registry.ts
```

### 中文解释
这条链解释了插件并不是“发现了就执行”，而是先经过 enablement 决策，再进入 runtime registry。

### English explanation
This chain explains that plugins are not simply “discovered and executed”; they first pass enablement decisions before entering the runtime registry.

## 链路 3：宿主 API 链 / Host API chain

```text
api-builder.ts
  -> OpenClawPluginApi
  -> plugin register(api)
  -> registry.ts / hooks.ts / command registrations
```

### 中文解释
这条链解释了插件作者和宿主之间的真正交互界面。

### English explanation
This chain explains the real interaction interface between plugin authors and the host.

## 链路 4：SDK 边界链 / SDK boundary chain

```text
plugin-sdk/index.ts
  -> provider-entry.ts / channel-entry-contract.ts / plugin-entry.ts
  -> extension implementation
  -> loader.ts / registry.ts
```

### 中文解释
这条链说明 `plugin-sdk` 不是装饰层，而是插件进入宿主的正式边界。

### English explanation
This chain shows that the `plugin-sdk` is not decorative; it is the formal boundary by which plugins enter the host.

## 链路 5：bundled ecosystem 链 / Bundled ecosystem chain

```text
extensions/*/openclaw.plugin.json
  -> bundled-plugin-scan.ts
  -> discovery.ts
  -> manifest-registry.ts
  -> loader.ts
```

### 中文解释
这条链解释了官方/内置扩展怎样沿着和第三方插件相同的系统边界进入运行时。

### English explanation
This chain explains how bundled extensions enter the runtime through the same system boundaries as third-party plugins.

---

## 六、如果你时间有限，最少读哪些 / Minimal Must-Read Set

如果你只有 2 小时，建议优先读这 8 个文件：

If you only have 2 hours, prioritize these 8 files:

1. `manifest.ts`
2. `discovery.ts`
3. `manifest-registry.ts`
4. `loader.ts`
5. `registry.ts`
6. `api-builder.ts`
7. `plugin-sdk/index.ts`
8. 一个 `extensions/*/openclaw.plugin.json`

这 8 个文件足够让你理解：

- 插件怎么被发现
- manifest 为什么这么重要
- 为什么 OpenClaw 强调 manifest-first
- 宿主怎么把插件能力统一装进 registry
- SDK 为什么是正式边界

These eight files are enough to understand discovery, the importance of manifests, why OpenClaw is manifest-first, how capabilities enter the registry, and why the SDK is the formal boundary.

---

## 七、总结 / Final Summary

OpenClaw Plugin System 的核心不是“动态加载扩展”，而是：

- 用 manifest 建立可解释性
- 用 loader 建立装载秩序
- 用 registry 建立能力总账
- 用 plugin-sdk 建立宿主边界

The essence of the OpenClaw plugin system is not “dynamic loading of extensions,” but building explainability through manifests, loading order through the loader, a capability ledger through the registry, and a formal host boundary through the SDK.
