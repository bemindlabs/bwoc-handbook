# Self-Hosting with bwoc-harness

🇹🇭 [ภาษาไทย](HANDBOOK.th.md)

For people who want to run BWOC agents on **Ollama or any OpenAI-compatible model** without installing a vendor CLI. This chapter covers what `bwoc-harness` is, how to configure and launch it, and what capabilities it adds on top of raw model access.

Term lookup: [`../glossary.en.md`](../glossary.en.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)

---

## Table of Contents

1. [Why Self-Host?](#1-why-self-host)
2. [Vendor-CLI Backends vs Harness Backends](#2-vendor-cli-backends-vs-harness-backends)
3. [Set Up the Ollama Backend](#3-set-up-the-ollama-backend)
4. [Set Up an OpenAI-Compatible Endpoint](#4-set-up-an-openai-compatible-endpoint)
5. [The TUI Chat Client](#5-the-tui-chat-client)
6. [What the Harness Gives You](#6-what-the-harness-gives-you)
   - [6.1 Agentic loop and core tools](#61-agentic-loop-and-core-tools)
   - [6.2 Tool authorization](#62-tool-authorization)
   - [6.3 Task queue](#63-task-queue)
   - [6.4 Telemetry](#64-telemetry)
   - [6.5 Safety guardrails](#65-safety-guardrails)
   - [6.6 Tier-2 deep memory](#66-tier-2-deep-memory)
   - [6.7 The chat_proto event stream](#67-the-chat_proto-event-stream)
7. [Dependency Quarantine](#7-dependency-quarantine)
8. [See Also](#8-see-also)

---

## 1. Why Self-Host?

Vendor CLIs (Claude Code, Codex, AGY, Kimi, Copilot) are maintained by their respective providers. They handle the model API, the agentic loop, and the tool surface for you. That is convenient when you have a subscription, but it comes with constraints: you depend on external services, you cannot choose a model that the vendor does not offer, and you cannot run anything air-gapped.

Self-hosting with `bwoc-harness` removes those constraints:

- **Any model.** Run Ollama locally, point at a remote Ollama server, or connect to any endpoint that speaks the OpenAI Chat Completions API — open-weight models, private fine-tunes, cloud inference providers that are not native BWOC backends.
- **No vendor CLI on PATH.** `bwoc-harness` is bundled with BWOC. There is nothing extra to install.
- **Full agentic runtime.** The harness is not a thin proxy. It provides the same agentic loop, core tools, task queue, telemetry, safety guardrails, and memory integration that makes BWOC agents productive — just talking to your endpoint instead of a vendor.
- **Air-gap friendly.** An agent on a local Ollama instance with no outbound network access still gets the full BWOC runtime.

The trade-off: you are responsible for running the model endpoint and keeping it reachable. The harness will report a clear error if it cannot connect.

---

## 2. Vendor-CLI Backends vs Harness Backends

BWOC supports two categories of backend. Understanding the difference tells you which one to use and why the launch flags differ.

| | Vendor-CLI backends | Harness backends |
|---|---|---|
| **Examples** | `claude`, `antigravity`, `codex`, `kimi`, `copilot` | `ollama`, `openai-compatible` |
| **`--backend` values** | `claude` · `antigravity` · `codex` · `kimi` · `copilot` | `ollama` · `openai-compatible` |
| **What runs the agent** | The vendor's own CLI binary (`claude`, `agy`, `codex`, `kimi`, `copilot`) | `bwoc-harness` (bundled with BWOC) |
| **Binary on PATH required** | Yes — BWOC does not bundle vendor CLIs | No |
| **Agentic loop** | Vendor-defined | BWOC-defined (consistent across all harness backends) |
| **`bwoc chat --tui` support** | No — vendor CLI renders its own interface | Yes |
| **`baseUrl` in manifest** | Not used | Optional for `ollama`; required for `openai-compatible` |
| **Tool authorization** | Vendor-defined policy | BWOC safety guardrail layer |

The instruction file is the same for all backends. `OLLAMA.md` and `OPENAI.md` are symlinks to `AGENTS.md` exactly like `CLAUDE.md`. The only things that differ between a vendor session and a harness session are who runs the loop and where the model traffic goes.

---

## 3. Set Up the Ollama Backend

**Prerequisites:** Ollama is installed and running. You have at least one model pulled (`ollama pull gemma3:27b`, for example). The agent is incarnated and passes `bwoc check`.

### Step 1 — Set the model in the manifest

Open (or edit through a prompt) `config.manifest.json` inside the agent directory and set `primaryModel` to the Ollama model tag you want to use:

```json
{
  "primaryModel": "gemma3:27b",
  "fallbackModel": "gemma3:4b"
}
```

The `fallbackModel` field is optional but recommended. The harness switches to it automatically if the primary fails to respond.

### Step 2 — Set `baseUrl` (only when Ollama is not on localhost)

If Ollama is running on a remote host, add `baseUrl` to the manifest:

```json
{
  "primaryModel": "gemma3:27b",
  "baseUrl": "http://192.168.1.113:11434/v1"
}
```

If `baseUrl` is absent, `bwoc-harness` defaults to `http://localhost:11434/v1`. For a standard local Ollama install you can omit it entirely.

### Step 3 — Spawn the agent

```bash
bwoc spawn <agent> --backend ollama
# or from inside the agent directory
bwoc spawn --backend ollama
```

You can also set `"backend": "ollama"` in the manifest so that `bwoc spawn <agent>` uses Ollama without the `--backend` flag every time.

### Verify the endpoint is reachable

```bash
curl http://localhost:11434/v1/models
# should return a JSON list of pulled models
```

If that fails, `bwoc spawn --backend ollama` will also fail — fix Ollama first.

---

## 4. Set Up an OpenAI-Compatible Endpoint

Any HTTP server that implements the OpenAI Chat Completions API (`POST /v1/chat/completions`) works as a harness backend. This includes hosted inference services, local inference servers (vLLM, llama.cpp server, LM Studio, etc.), and cloud providers that are not native BWOC backends.

**`baseUrl` is required.** There is no default. If it is absent, `bwoc spawn --backend openai-compatible` exits immediately with a clear error.

### Step 1 — Set `baseUrl` and `primaryModel`

```json
{
  "primaryModel": "my-custom-model",
  "baseUrl": "http://localhost:8080/v1"
}
```

### Step 2 — Set any required API key

If the endpoint requires an API key, set it in the environment variable the harness reads. The exact variable name depends on your endpoint configuration. A common pattern:

```bash
export OPENAI_API_KEY="your-key-here"
bwoc spawn <agent> --backend openai-compatible
```

The harness forwards the key in the `Authorization: Bearer` header. If no key is needed, the env var can be empty or absent.

### Step 3 — Spawn

```bash
bwoc spawn <agent> --backend openai-compatible
```

### Endpoint compatibility checklist

- Implements `POST /v1/chat/completions` with `messages`, `model`, and `stream` fields
- Returns server-sent events when `stream: true`
- Returns `choices[].delta.content` in streaming chunks
- Accepts `tools` and `tool_calls` for function-calling (required for core tools to work)

If the endpoint does not support tool calls, core tools will not function. The agent will still run but will be limited to pure-text responses.

---

## 5. The TUI Chat Client

`bwoc chat <agent> --tui` opens a full-screen terminal chat interface built with ratatui. It is only available for harness backends (`ollama` and `openai-compatible`). Vendor CLI backends render their own interface; the `--tui` flag has no effect on them.

```bash
# Full-screen TUI with the agent's configured backend (must be a harness backend)
bwoc chat sage --tui

# Explicit backend
bwoc chat sage --backend ollama --tui
bwoc chat sage --backend openai-compatible --tui
```

What the TUI provides:

- **Streaming token display.** Model tokens appear as they arrive; no waiting for the full response.
- **Tool call visibility.** When the harness invokes a core tool, the TUI shows which tool was called and its result before the model continues.
- **Session history.** The conversation scrolls with keyboard navigation.
- **Status bar.** Current model, backend, and latency visible at all times.

Under the hood, `bwoc chat --tui` starts `bwoc-harness --chat` and renders the `chat_proto` event stream it emits. The same stream is what the `bwoc-chat` desktop app consumes — they share an identical wire format.

To exit the TUI: `Ctrl+C` or `q` from the input field.

---

## 6. What the Harness Gives You

`bwoc-harness` is more than an HTTP bridge. Every agent running on a harness backend gets the full BWOC agentic runtime described below.

### 6.1 Agentic loop and core tools

The harness implements the BWOC agentic loop: it sends the agent's context to the model, receives a response, detects tool calls, executes approved tools, appends results, and loops until the model produces a final response with no pending tool calls.

Core tools bundled with the harness:

| Tool | What it does |
|---|---|
| `read_file` | Read a file from the filesystem |
| `write_file` | Write or overwrite a file |
| `edit_file` | Apply a targeted diff to a file |
| `run_command` | Execute a shell command and capture output |
| `search` | Grep or glob across the workspace |
| `memory_get` / `memory_put` | Read and write Tier-1 memory entries |
| `http_fetch` | Fetch a URL (subject to safety guardrails) |

Tool availability is subject to the authorization layer described in Section 6.2.

### 6.2 Tool authorization

Before a tool call executes, the harness passes it through a tool-authorization check. You can configure which tools are enabled, disabled, or require confirmation per-agent in `config.manifest.json` or through a prompt during a session.

This is distinct from the safety guardrail layer (Section 6.5), which operates at a higher level and cannot be overridden by the agent. Tool authorization is an operator-level control for tuning what a specific agent is allowed to do.

### 6.3 Task queue

When running under `bwoc run` or in a supervised fleet, the harness connects to the workspace task queue. The agent can dequeue tasks, report progress, and mark completion. This is what enables multi-agent pipelines and asynchronous task dispatch — one agent posts a task, another picks it up.

### 6.4 Telemetry

The harness emits structured telemetry for each session: token counts (prompt and completion), tool call counts, latency per turn, and exit status. This data is available to `bwoc supervise` and any monitoring tooling you connect to the workspace.

Telemetry stays local. Nothing is sent to external services unless you configure an exporter.

### 6.5 Safety guardrails

The harness includes a safety guardrail layer that runs before tool execution. It is aligned to two framework principles:

**Sīla 5 (five precepts) — forbidden actions.** The harness refuses to execute tool calls that map to one of the five forbidden categories regardless of what the model requests. These are hard stops, not soft warnings. The agent cannot override them through a prompt.

**Taṇhā 3 (three threat categories) — threat detection.** The guardrail layer classifies each tool call against three categories of risky intent (destructive intent, privacy violation, deception). A classified call is blocked and logged, not silently skipped.

The guardrails are in addition to tool authorization, not a replacement. An action must pass both layers to execute.

### 6.6 Tier-2 deep memory

When `deepMemoryCmd` and `sessionsPath` are set in `config.manifest.json`, the harness activates Tier-2 deep memory. This is cross-session, cross-agent persistent memory that the agent can recall at startup and mine at session end.

Relevant manifest fields:

```json
{
  "deepMemoryCmd": "bwoc memory",
  "sessionsPath": "memories/sessions/"
}
```

With these set, the harness speaks the Tier-2 contract automatically:

- `wake-up` — called at session start to recall relevant prior context
- `search` — called when the agent needs to look up past decisions or feedback
- `mine` — called at session end to persist learnings from the session

Without these fields, Tier-2 is inactive and the agent operates on Tier-1 (`MEMORY.md`) only. See [`../self-improvement/HANDBOOK.en.md`](../self-improvement/HANDBOOK.en.md) for how the memory tiers fit into the learning loop.

### 6.7 The chat_proto event stream

All harness backends emit events on the `chat_proto` stream — a structured wire format that carries:

- `token` events as the model streams output
- `tool_call` / `tool_result` events for each tool invocation
- `turn_start` / `turn_end` events with latency metadata
- `error` events with structured error codes

This is the same stream that `bwoc chat --tui` renders and that the `bwoc-chat` desktop app consumes. If you are building tooling on top of BWOC (dashboards, logging pipelines, test harnesses), consume this stream directly.

---

## 7. Dependency Quarantine

`bwoc-harness` is compiled as a separate binary, not linked into `bwoc` core. This is a deliberate design choice called **dependency quarantine**: the network stack, HTTP client, streaming parser, and all runtime dependencies that touch external endpoints live in the harness binary, not in `bwoc` core.

The practical effect for operators:

- **`bwoc` core stays lean.** Building or auditing `bwoc` core does not pull in HTTP or async runtime dependencies.
- **Harness can be updated independently.** A new model API version or a security patch in the HTTP client ships in a harness update without touching the core CLI.
- **Clear security boundary.** Everything that talks to the network is isolated in one binary. Auditing the harness binary audits all network-facing code.

You do not need to manage the harness binary separately. `bwoc spawn` finds and execs it automatically. If the harness binary is missing or outdated, `bwoc` will report the version mismatch.

---

## 8. See Also

| Resource | What you get |
|---|---|
| [HARNESS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/HARNESS.en.md) | Authoritative harness spec — protocol details, guardrail rules, manifest schema |
| [bwoc-harness crate](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-harness) | Source code for the harness binary |
| [`../backends/HANDBOOK.en.md`](../backends/HANDBOOK.en.md) | All backends side by side — launch commands, manifest fields, troubleshooting |
| [`../self-improvement/HANDBOOK.en.md`](../self-improvement/HANDBOOK.en.md) | Memory tiers, the learning loop, and how Tier-2 deep memory works end to end |
| [`../glossary.en.md`](../glossary.en.md) | Sīla, Taṇhā, and all other specialized terms used above |
| [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) | Framework repo root |

---

*On any conflict between this handbook and the [framework repo](https://github.com/bemindlabs/BWOC-Framework), the repo is correct — this handbook has a bug.*
