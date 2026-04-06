# OpenClaw Source Guide

This guide is a more detailed, directory-oriented walkthrough of the OpenClaw source tree. It is meant for readers who already have the high-level architecture summary and now want to understand where the most important implementation lives.

It is not a complete file index. Instead, it focuses on the directories and files that most strongly explain how OpenClaw works.

## Reading order

If you are new to the codebase, this order gives the fastest path to understanding:

1. root entrypoints and workspace files
2. `src/gateway/`
3. `src/agents/`
4. `src/sessions/`, `src/routing/`, and `src/auto-reply/`
5. `src/plugins/` and `src/plugin-sdk/`
6. `extensions/`
7. `ui/`
8. `apps/` and `Swabble/`
9. `src/config/`, `src/security/`, `src/process/`, `src/logging/`

## 1. Root-level files

These are the best root files to read before descending into the tree.

| File | Why it matters |
| --- | --- |
| `package.json` | Defines the monorepo root package, exported plugin SDK subpaths, scripts, and the published surface of OpenClaw. |
| `pnpm-workspace.yaml` | Shows the real workspace boundaries: root package, `ui`, `packages/*`, and `extensions/*`. |
| `openclaw.mjs` | CLI launch shim and top-level executable handoff. |
| `tsdown.config.ts` | Explains how core entrypoints, plugin SDK subpaths, hooks, and bundled plugin entrypoints are built together. |
| `AGENTS.md` | High-level repository conventions and architecture guidance. |
| `docs.acp.md` | Useful context for ACP-related architecture and control-plane semantics. |

## 2. `src/` — core runtime

`src/` is the center of the product. It contains the runtime, gateway, agent engine, plugin host, config/security substrate, and many shared facilities.

### Key entry files in `src/`

| File | Role |
| --- | --- |
| `src/entry.ts` | Main startup path for the CLI/runtime process. |
| `src/index.ts` | Library/root module entrypoint and legacy CLI handoff. |
| `src/runtime.ts` | Shared runtime abstraction for stdout/logging/error/exit semantics. |
| `src/global-state.ts` | Small but important process-wide flags and shared runtime state. |
| `src/extensionAPI.ts` | Bridge for extension-facing runtime integration. |

## 3. `src/gateway/` — control plane center

If you only read one subsystem first, read the Gateway.

This directory is the control plane: the place where clients, nodes, automations, channels, and UI surfaces converge.

### Most important files

| File | Role |
| --- | --- |
| `src/gateway/boot.ts` | Bootstraps the Gateway runtime and wires major services together. |
| `src/gateway/server-http.ts` | HTTP server implementation for APIs, static assets, probes, and Gateway-adjacent routes. |
| `src/gateway/control-ui.ts` | Serves and configures the browser Control UI. |
| `src/gateway/protocol/schema.ts` | Typed wire contract for the WebSocket/API boundary. |
| `src/gateway/channel-health-monitor.ts` | Tracks health and liveness of channel integrations. |
| `src/gateway/auth*` files | Auth modes, shared-secret handling, trusted ingress, and client validation rules. |

### Why this directory matters

The Gateway owns:

- WebSocket sessions for operators and nodes
- HTTP and API serving
- control-plane auth and pairing
- session ingress
- event broadcasting
- Control UI hosting
- protocol validation

It is the closest thing OpenClaw has to a system kernel.

## 4. `src/agents/` — execution engine

This directory is the runtime that actually performs agent work after the Gateway accepts it.

### Most important files and sub-areas

| File / area | Role |
| --- | --- |
| `src/agents/openclaw-tools.ts` | Assembles tool exposure for the runtime. |
| `src/agents/tool-catalog.ts` | Central catalog for available tools and related metadata. |
| `src/agents/model-selection.ts` | Resolves which provider/model the agent should use. |
| `src/agents/auth-profiles.ts` | Handles provider auth profiles and selection logic. |
| `src/agents/context/` and compaction-related files | Manage prompt/context assembly and long-session reduction. |
| `src/agents/bash*`, process/tool host files | Handle executable tools, shells, and system-facing actions. |
| `src/agents/mcp*` and transport-related files | Support MCP/ACP or related external tool/runtime integration. |

### What to look for here

This is where OpenClaw stops being a routing system and becomes an agent runtime:

- prompt/context construction
- tool planning and invocation
- model resolution
- provider/auth selection
- execution approvals
- result streaming and finalization

## 5. `src/sessions/`, `src/routing/`, and `src/auto-reply/`

These directories are easier to understand together than separately.

### `src/sessions/`

This area owns session identity, transcript lifecycle, and continuity.

Important files usually include session keys, transcript handling, and session metadata transforms.

### `src/routing/`

This area decides how inbound activity binds to:

- accounts
- agents
- channels
- threads
- session ids

It is the glue that makes a single Gateway safe for many senders and many channels.

### `src/auto-reply/`

This area turns inbound messages into runtime work. It decides when a message should trigger an agent reply and how that request is shaped.

### Why these directories matter together

Together they implement the path:

**incoming event → route resolution → session binding → agent invocation → outbound reply**

## 6. `src/plugins/` — plugin host internals

This directory owns plugin discovery, validation, loading, and registry construction.

### Most important files

| File | Role |
| --- | --- |
| `src/plugins/registry.ts` | Defines the real plugin capability surface and aggregates registered behavior. |
| `src/plugins/loader.ts` | Loads plugin runtime modules and applies manifest-first loading rules. |
| `src/plugins/runtime*.ts` | Holds runtime-level plugin integration state and helper behavior. |
| `src/plugins/services.ts` | Integrates long-running plugin services into the host runtime. |
| `src/plugins/install*` files | Manage plugin installation and local plugin lifecycle. |

### What to pay attention to

The key idea is that OpenClaw tries to validate and reason about plugins from manifests and metadata before executing plugin code. That is a major architectural choice.

## 7. `src/plugin-sdk/` — public extension boundary

This directory is the contract surface between core and plugins.

### What it contains

The SDK is broken into many narrow subpaths, including areas for:

- core plugin registration
- setup/runtime helpers
- channels
- providers
- config schema/runtime helpers
- approval and reply runtimes
- sandbox/runtime support
- routing/runtime bridges

### Why it matters

The repository explicitly treats this directory as the stable extension seam. If you are studying how OpenClaw wants extension authors to interact with the host, this is the directory to read.

## 8. `src/channels/` — internal channel core

This area is the internal side of channel support.

It is not the preferred import surface for extensions, but it is where core channel abstractions and some message/session logic live.

### What to focus on

- shared message semantics
- conversation and thread binding rules
- allowlist and identity handling
- core-side dispatch helpers

This directory becomes clearer after reading the plugin architecture docs, because many channel-specific execution details now live in bundled extension modules instead of core.

## 9. `src/hooks/` — lifecycle extensibility

Hooks are where behavior can be inserted around important runtime events.

### Important responsibilities

- before-agent and before-model hooks
- prompt mutation
- tool lifecycle hooks
- install/setup lifecycle hooks
- compatibility support for older hook-based integrations

This subsystem matters because it explains how OpenClaw evolved from pure hook-based extensibility toward explicit capability registration without fully dropping legacy patterns.

## 10. `src/config/` and `src/secrets/`

These directories define how OpenClaw is configured and how secrets enter the runtime.

### `src/config/` key files

| File | Role |
| --- | --- |
| `src/config/config.ts` | Main config export and access surface. |
| `src/config/io.ts` | Heavyweight config loading, caching, runtime snapshots, and file mutation plumbing. |
| `src/config/paths.ts` | Resolves state/config paths such as `~/.openclaw`. |
| `src/config/zod-schema.ts` | Full schema for validating the runtime config model. |
| `src/config/types.*` files | Typed config slices and schema support. |

### `src/secrets/`

This directory resolves secret values into runtime config using environment, file, and command-backed sources.

If you are tracing auth/provider behavior, these files are worth reading early.

## 11. `src/security/`, `src/process/`, `src/logging/`, `src/cron/`

These directories are the operations substrate.

### `src/security/`

Focus on:

- audit entrypoints
- dangerous tool/config detection
- filesystem checks
- channel security checks
- regex and execution safety helpers

### `src/process/`

Focus on:

- process supervisor
- child/PTY adapters
- cancellation and kill-tree logic
- command scheduling and execution helpers

### `src/logging/` and `src/logger.ts`

Focus on:

- subsystem loggers
- file transport and retention
- runtime logging settings

### `src/cron/`

This area handles scheduled/background job behavior and is part of the runtime orchestration substrate rather than the visible user-facing product surface.

## 12. `src/media/`, `src/media-understanding/`, `src/realtime-*`, `src/tts/`, `src/image-generation/`, `src/video-generation/`

These directories are not always where the final provider implementation lives, but they define the core-side contracts and runtime support for multimodal capabilities.

They matter because they show that multimodal support is a first-class platform concept rather than an ad hoc add-on.

## 13. `extensions/` — bundled plugin ecosystem

This directory is huge and should be read as a taxonomy, not as one unit.

### Best way to navigate it

Group extensions by responsibility:

#### Channel plugins

Examples:

- `telegram`
- `slack`
- `discord`
- `feishu`
- `signal`
- `matrix`
- `mattermost`
- `msteams`
- `line`
- `whatsapp`
- `zalo`

#### Provider plugins

Examples:

- `openai`
- `anthropic`
- `google`
- `deepseek`
- `openrouter`
- `mistral`
- `groq`
- `qwen`
- `minimax`
- `moonshot`

#### Multimodal and voice plugins

Examples:

- `browser`
- `speech-core`
- `media-understanding-core`
- `image-generation-core`
- `video-generation-core`
- `voice-call`
- `elevenlabs`
- `deepgram`

#### Search/tooling plugins

Examples:

- `exa`
- `firecrawl`
- `brave`
- `duckduckgo`
- `searxng`
- `tavily`

### Common files to read inside an extension

| File | Role |
| --- | --- |
| `index.ts` | Plugin entrypoint and registration path. |
| `api.ts` | Local API barrel or host-facing helper exports. |
| `runtime-api.ts` | Runtime-level extension integration surface when present. |
| `openclaw.plugin.json` | Manifest metadata used during discovery and validation. |

Read one or two representative plugins deeply rather than skimming fifty of them.

## 14. `packages/` — internal support packages

This directory is much smaller than `extensions/`, but still useful.

### Important packages

| Package | Role |
| --- | --- |
| `packages/plugin-package-contract` | Internal plugin packaging and contract support. |
| `packages/memory-host-sdk` | Shared memory host SDK. |
| `packages/clawdbot` | Compatibility shim package. |
| `packages/moltbot` | Compatibility shim package. |

This directory is best read after the plugin architecture is already clear.

## 15. `ui/` — browser Control UI

The `ui/` workspace is the main web client for the Gateway.

### What to focus on

| Area | Role |
| --- | --- |
| `ui/package.json` | Shows the frontend stack and test tooling. |
| `ui/src/` | Actual UI implementation. |
| gateway-related client files | WebSocket/API communication back to the Gateway. |
| settings/session/chat screens | Operational product surface for managing the running system. |

This workspace is best understood as a control-plane client, not a standalone app.

## 16. `apps/` — native clients

This directory contains the native/mobile surfaces.

### Subdirectories

| Path | Role |
| --- | --- |
| `apps/android/` | Android client/node. |
| `apps/ios/` | iOS client/node. |
| `apps/macos/` | macOS client and related desktop tooling. |
| `apps/shared/` | Shared Apple-side packages and protocol/UI code. |

### Most important shared file

| File | Role |
| --- | --- |
| `apps/shared/OpenClawKit/Package.swift` | Shows the shared Apple package split into `OpenClawProtocol`, `OpenClawKit`, and `OpenClawChatUI`. |

That file is a useful orientation point because it proves the native clients share protocol models and reusable client runtime pieces instead of duplicating them separately.

## 17. `Swabble/` — wake-word and local voice support

This Swift project appears to support local voice/wake-word behavior and related reusable Apple-side voice capabilities.

It is not the main runtime center, but it is important if you want the full picture of OpenClaw’s Apple-side voice stack.

## 18. `docs/` — public architecture context

When reading source, these docs are especially useful because they encode intended boundaries rather than just current implementation:

| File | Why it matters |
| --- | --- |
| `docs/concepts/architecture.md` | Explains the Gateway-centric control-plane model. |
| `docs/plugins/architecture.md` | Explains plugin discovery, loading, capability ownership, and the message tool boundary. |
| `docs/gateway/protocol.md` | Helps map protocol schema and WS behavior back to implementation. |
| `docs/platforms/*` | Adds context for native/macOS/iOS integration areas. |

## Suggested study pass

For a strong first pass through the repo, read in this order:

1. `package.json`, `pnpm-workspace.yaml`, `tsdown.config.ts`
2. `docs/concepts/architecture.md`
3. `src/gateway/boot.ts`, `src/gateway/server-http.ts`, `src/gateway/protocol/schema.ts`
4. `src/agents/` tool/model/runtime files
5. `src/sessions/`, `src/routing/`, `src/auto-reply/`
6. `src/plugins/registry.ts`, `src/plugins/loader.ts`
7. `docs/plugins/architecture.md`
8. `src/plugin-sdk/`
9. one representative channel extension and one representative provider extension
10. `ui/`, `apps/shared/OpenClawKit/Package.swift`, and platform app directories

That sequence preserves the real logic of the system: control plane first, execution plane second, extension seams third, clients last.
