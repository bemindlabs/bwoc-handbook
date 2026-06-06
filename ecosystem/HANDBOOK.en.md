🇹🇭 [ภาษาไทย](HANDBOOK.th.md)

# BWOC Ecosystem — Projects Overview

The BWOC family is a core framework plus a set of companion applications and device targets, all orbiting the `bwoc` CLI as their single source of truth. The framework owns identity, lifecycle, memory, and inter-agent protocol; every other project consumes those facts through `bwoc <cmd> --json` or the `bwoc_core::chat_proto` event stream — no project reaches into workspace files directly. This page maps every member of the family: what it is, what it is built with, how it connects to the core, and where to find it.

---

## Overview

| Project | Role | Stack | Link |
|---|---|---|---|
| **BWOC-Framework** | Core framework + `bwoc` CLI | Rust (9 crates), macOS / Linux / Windows | [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) |
| **bwoc-handbook** | Bilingual, role-indexed documentation | Markdown (EN + TH) | this documentation set |
| **bwoc-chat** | Native desktop chat for agents | Rust + egui | [bemindlabs/bwoc-chat](https://github.com/bemindlabs/bwoc-chat) |
| **bwoc-devices-app** (BWOC Monitor) | Read-only macOS fleet dashboard | Rust + Tauri 2 + tokio + static HTML/JS | internal / not yet public |
| **bwoc-llm-pm** (LLM Provider Monitor) | macOS menu-bar: LLM provider auth + quota | Swift, SwiftUI, SwiftPM, macOS 13+ | [bemindlabs/LLMProviderMonitor](https://github.com/bemindlabs/LLMProviderMonitor) |
| **bwoc-mcc** | macOS menu-bar fleet control center | Swift 5.9, SwiftUI, macOS 13+ | [bemindlabs/bwoc-mcc](https://github.com/bemindlabs/bwoc-mcc) |
| **bwoc-devices** | Device monorepo — Rust core + multi-board firmware | Rust (4 crates), Arduino/PlatformIO C++ | [bemindlabs/bwoc-devices](https://github.com/bemindlabs/bwoc-devices) |

---

## Core

### BWOC-Framework

**What it is.** The foundation everything else depends on. BWOC-Framework is a backend-neutral specification and native Rust implementation for incarnating, running, and orchestrating AI coding agents. It ships as a Rust workspace of nine crates — `bwoc-cli`, `bwoc-harness`, `bwoc-core`, `bwoc-agent`, `bwoc-mqtt`, `bwoc-deep-memory`, and others — plus the `bwoc` CLI binary and the `modules/agent-template` cloneable scaffold.

**Stack.** Rust 1.85+ (2024 edition), multi-target (macOS / Linux / Windows), MIT. Current version: v2.24.0.

**What it provides to the ecosystem.** Every other project in the family is a consumer, not a peer:

- `bwoc list --json`, `bwoc sessions --json`, `bwoc fleet health` — the JSON APIs that desktop apps and device firmware poll.
- `bwoc-harness --chat` — the subprocess that `bwoc-chat` spawns; it owns model calls, tools, guardrails, and permissions.
- `bwoc_core::chat_proto` — the typed event stream (`token`, `tool_call`, `tool_result`, `permission_request`, `turn_end`) that `bwoc-chat` renders.
- Agent registry, manifest schema, team protocol, inbox, and scrum state that `bwoc-mcc` reads.

**Link.** [github.com/bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)

---

### bwoc-handbook

**What it is.** The documentation set you are reading. A bilingual (EN/TH), role-indexed handbook that orients users, developers, agent authors, AI search agents, and crawlers. It summarizes and cross-references the framework without duplicating the source of truth — on any conflict the framework repo wins.

**Stack.** Plain Markdown, `*.en.md` / `*.th.md` parity pairs per page.

**How it connects.** The handbook references the framework repo for canonical facts and links to the public GitHub for all code. It does not embed workspace-local paths.

**Link.** this documentation set — see [`../README.md`](../README.md) for the full index.

---

## Desktop and Menu-Bar Apps

### bwoc-chat

**What it is.** A native desktop chat window for BWOC agents. One agent or a whole team in one window — streaming replies, a live tool-activity pane per agent, and inline Allow / Deny permission prompts when a tool is in `ask` mode.

**Stack.** Rust (2024 edition), [egui](https://github.com/emilk/egui) / eframe — pure-Rust immediate-mode GUI, no webview. Depends on `bwoc-core` (by path) for the chat protocol and agent resolution; never depends on `bwoc-cli` or `bwoc-harness` at build time. Supports `ollama` and `openai-compatible` harness backends.

**How it connects to the framework.** `bwoc-chat` spawns `bwoc-harness --chat` as a child process for each named agent. It reads `bwoc_core::chat_proto` events from the child's stdout (tokens, tool calls, permission requests) and writes `ChatInput` lines to its stdin. The harness owns the session; the window only renders and routes. Agent names are resolved from the workspace registry — the same `.bwoc/agents.toml` the CLI reads.

**Team mode.** Pass several agent names on the command line; each becomes its own harness subprocess, all multiplexed into one shared transcript. Messages broadcast to every agent; `@name` addresses one.

**Why a separate repo.** Keeping the heavy GUI dependency tree (winit, glow) out of the lean framework workspace. `bwoc-chat` pulls in only `bwoc-core`, never `bwoc-cli` or `bwoc-harness`.

**Status.** Active. Vendor-CLI backends (claude / codex / kimi / agy) are not rendered here — use `bwoc spawn` for those.

**Link.** [github.com/bemindlabs/bwoc-chat](https://github.com/bemindlabs/bwoc-chat)

---

### bwoc-devices-app (BWOC Monitor)

**What it is.** A read-only macOS desktop dashboard for a BWOC workspace. It shells out to the `bwoc` CLI on a configurable timer and renders three live panels: **Fleet Health** (the seven governance signals), **Agents** (registry + status), and **Sessions** (running daemons / tmux panes).

**Stack.** Rust + Tauri 2 + tokio + static HTML/JS. No npm, no Node, no Vite. The Rust side exposes Tauri IPC commands (`list_agents`, `list_sessions`, `fleet_health`, `fleet_snapshot`) that the webview polls. The `fleet_snapshot` command composes all three into the canonical BWOC fleet JSON payload — the same shape that device firmware ingests over WebSocket.

**How it connects to the framework.** The CLI is the single source of truth; the app never reads workspace files directly. It spawns `bwoc --workspace <path> <cmd> --json` and parses the output. `fleet health` output (no `--json` flag in v1) is parsed from its block structure by the Rust side.

**Status.** Internal / not yet public. No public URL.

---

### bwoc-llm-pm (LLM Provider Monitor)

**What it is.** A lightweight macOS menu-bar app that shows, at a glance, the auth status and token / credit usage of the LLM provider CLIs installed on the machine, with quick login / switch actions for each.

**Supported providers.** Antigravity, Claude, Codex, Kimi, Ollama.

**Stack.** Swift, SwiftUI, SwiftPM, macOS 13+. Three targets: `LLMProviderMonitorCore` (pure logic, unit-tested), `LLMProviderMonitor` (SwiftUI menu-bar app), `CoreChecks` (plain-executable test runner — no XCTest needed). Activation policy `.accessory` — no Dock icon.

**How it connects to the framework.** It probes provider CLIs independently of BWOC; it does not call `bwoc` directly. Its relationship to the BWOC ecosystem is complementary: `bwoc-mcc` handles fleet operations (agents, sessions, inboxes) while `bwoc-llm-pm` handles provider credentials and quota. The two are designed to live side-by-side in the menu bar.

**Credit tracking.** Codex: reads `~/.codex/sessions/` rollout files. Claude: reads `~/.claude/usage-cache.json` written by a `statusLine` hook during active Claude Code sessions. Ollama: surfaces the currently-loaded model from `ollama ps`; no quota. Antigravity: quota not headlessly accessible, shown as em dash.

**Status.** Available. Not yet packaged as a signed `.app` bundle — runs as a bare SwiftPM executable for local use.

**Link.** [github.com/bemindlabs/LLMProviderMonitor](https://github.com/bemindlabs/LLMProviderMonitor)

---

### bwoc-mcc

**What it is.** A SwiftUI macOS menu-bar control center for the BWOC agent fleet. It surfaces live status — agents, sessions, inboxes — and provides quick actions to spawn, chat, stop, and supervise agents without leaving the menu bar.

**Stack.** Swift 5.9, SwiftUI `MenuBarExtra`, SwiftPM, macOS 13+ (Ventura+). Three targets: `BwocMccCore` (library), `BwocMcc` (SwiftUI app), `CoreChecks` (test runner). No Electron, no background daemons. Tests run via `swift run CoreChecks`, not XCTest.

**How it connects to the framework.** All CLI shell-outs go through `bwoc <cmd> --json` — the app is insulated from BWOC's Rust internals and depends only on the stable CLI surface. It reads the same `bwoc list --json` and `bwoc sessions --json` that `bwoc-devices-app` reads; polls every five seconds; refresh button forces a poll. Chat actions open a `bwoc-chat` egui window for `ollama` / `openai-compatible` agents and fall back to `bwoc chat` in Terminal for vendor-CLI agents.

**Scope.** Fleet operations only — agents, sessions, inboxes, and (planned) scrum state. LLM provider auth and quota are out of scope; that is `bwoc-llm-pm`'s job.

**Status.** Alpha. Scaffold builds, launches, and renders the live fleet. Quick actions, sessions view, inbox preview, and scrum integration are in progress under `BWOC-EPIC-5`.

**Link.** [github.com/bemindlabs/bwoc-mcc](https://github.com/bemindlabs/bwoc-mcc)

---

## Devices and Firmware

### bwoc-devices

**What it is.** A standalone monorepo that puts the BWOC fleet dashboard onto physical hardware. One portable Rust core abstracts hardware behind HAL traits; multiple board firmware targets implement display and transport. A device joins the fleet and renders the live dashboard in real time.

**Crates.**

| Crate | Role |
|---|---|
| `bwoc-device-proto` | Host-to-device JSON wire contract — source of truth for all targets |
| `bwoc-device-hal` | HAL traits: Net, Store, Input, Display — no `std` required |
| `bwoc-device-core` | Portable runtime: source arbitration (`SourceArbiter`), fleet state, render |
| `bwoc-device-linux` | Linux / Raspberry Pi HAL impl + `bwoc-device` binary; feature-gated transports |

**Board targets.**

| Board | Stack | Status |
|---|---|---|
| WT32-SC01 Plus (ESP32-S3, 3.5" ST7796) | Arduino / PlatformIO C++ | Production (absorbed from bwoc-penlee-sc01-plus) |
| Raspberry Pi 5 / Linux | Rust core, `--features pi` (MQTT + WS + fbdev + evdev) | Active; deploy via `deploy/rpi5/` |
| ESP32-S3 ball (1.28" round LCD, XiaoZhi voice) | ESP-IDF C++, LovyanGFX | Scaffolded, not yet built |

**Stack.** Rust workspace (`no_std + alloc` for `proto` / `hal` / `core`; `std` for `bwoc-device-linux`), Arduino / PlatformIO C++. Transports: MQTT (`rumqttc`), WebSocket (`tungstenite`), stdin (USB), framebuffer, evdev touch. Feature flags: `mqtt`, `ws`, `sim`, `fbdev`, `evdev`, `full`, `pi`.

**Protocol boundary.** The ESP32 C++ firmware does not link the Rust crates. It implements rendering natively (LovyanGFX) and conforms to `bwoc-device-proto` only through the generated `firmware/<board>/gen/bwoc_proto.h` header, emitted by the `gen-cpp-proto` tool crate. The protocol is the shared contract; the HAL code is not.

**How it connects to the framework.** The host pushes fleet snapshots — the same canonical JSON produced by `bwoc-devices-app`'s `fleet_snapshot` command — over MQTT or WebSocket to the device. The device runtime `bwoc-device-core` applies a `SourceArbiter` (USB takes priority over WS, WS over MQTT) and re-renders the dashboard each tick. The device does not call the `bwoc` CLI directly; it receives the already-composed payload from a pusher.

**Status.** Active. Raspberry Pi and ESP32-SC01-Plus targets operational.

**Link.** [github.com/bemindlabs/bwoc-devices](https://github.com/bemindlabs/bwoc-devices)

---

### bwoc-penlee-sc01-plus (historical)

**What it is.** The earlier, single-device project that originated the SC01 Plus fleet display concept: ESP32-S3 firmware (BLE for config, WebSocket for data) plus a Tauri host app that pushed the BWOC fleet JSON to the device. The firmware and the protocol concept were generalized and absorbed into **bwoc-devices** (`firmware/esp32-sc01plus/`). The original directory remains as a regression reference while the migrated firmware stabilizes. No separate public URL — it lives inside the bwoc-devices history.

---

## How They Fit Together

The `bwoc` CLI and the `bwoc-device-proto` wire schema are the shared contract that makes these projects composable without tight coupling.

```
bwoc CLI  ──────────────────────────────────────────────────────────────┐
  │  bwoc list --json            consumed by: bwoc-mcc, bwoc-devices-app│
  │  bwoc sessions --json        consumed by: bwoc-mcc, bwoc-devices-app│
  │  bwoc fleet health           consumed by: bwoc-mcc, bwoc-devices-app│
  │                                                                      │
  │  bwoc-harness --chat  ──────► bwoc-chat (egui renderer)             │
  │                          ▲                                           │
  │                          └── launched by bwoc-mcc (quick action)    │
  │                                                                      │
fleet snapshot (JSON)  ──────► bwoc-devices-app ──► (push) bwoc-devices │
                                                                         │
bwoc-llm-pm  ── independent of bwoc CLI; complements bwoc-mcc ──────────┘
```

Every desktop or menu-bar app calls `bwoc` by name and parses `--json` output. No app reads `.bwoc/` files directly. No app depends on `bwoc-cli` or `bwoc-harness` at build time (except `bwoc-chat`, which depends only on `bwoc-core` for the protocol types). Device firmware consumes only the fleet JSON payload, not the CLI. This means the CLI's `--json` surface and `bwoc-device-proto` are the stability boundary for the whole family: change them carefully, version them explicitly.

---

*Back to the handbook index: [`../README.md`](../README.md) — Glossary: [`../glossary.en.md`](../glossary.en.md)*
