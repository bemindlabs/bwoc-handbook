# Backends Handbook — Driving Agents Through Any LLM CLI

🇹🇭 [ภาษาไทย](HANDBOOK.th.md)

For people who have **already initialised a workspace, incarnated agents, and prepared their related projects**, and now want to **use and configure those agents by prompting through whichever LLM CLI they have**. If you have not done those earlier steps yet, start with [`../end-user/HANDBOOK.en.md`](../end-user/HANDBOOK.en.md) (workspace and agent lifecycle) and [`../agents/HANDBOOK.en.md`](../agents/HANDBOOK.en.md) (incarnation and authoring).

Term lookup: [`../glossary.en.md`](../glossary.en.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)

---

## Table of Contents

1. [Prerequisites — Where This Page Starts](#1-prerequisites--where-this-page-starts)
2. [One Agent, Many Backends](#2-one-agent-many-backends)
3. [Backend Reference Table](#3-backend-reference-table)
4. [Launch It](#4-launch-it)
   - [4.1 bwoc spawn — raw backend exec](#41-bwoc-spawn--raw-backend-exec)
   - [4.2 bwoc chat — interactive session](#42-bwoc-chat--interactive-session)
   - [4.3 bwoc run — headless one-shot task](#43-bwoc-run--headless-one-shot-task)
   - [4.4 Backend override with --backend](#44-backend-override-with---backend)
5. [Configure by Prompt](#5-configure-by-prompt)
6. [Make Config Stick — the Manifest](#6-make-config-stick--the-manifest)
7. [Add a New Backend](#7-add-a-new-backend)
8. [Troubleshooting](#8-troubleshooting)
9. [See Also](#9-see-also)

---

## 1. Prerequisites — Where This Page Starts

This page assumes all three of these are done:

1. **`bwoc init` completed.** A workspace directory exists with `.bwoc/` inside it.
2. **Agents incarnated.** At least one agent was created via `bwoc new`, passes `bwoc check`, and is listed in `bwoc list`.
3. **Related projects prepared.** Any project directories the agent will work on are in place and the agent's verification gates (`lintCmd`, `testCmd`, etc.) in `config.manifest.json` point to valid commands.

If any of those are not done, complete them first. This page only covers what happens _after_ setup — how you actually drive agents through prompts.

---

## 2. One Agent, Many Backends

Every BWOC agent has exactly one instruction file that every LLM backend reads: `AGENTS.md`. It is plain Markdown with `{{camelCase}}` placeholders for anything backend-specific. No vendor name, no model ID, no Obsidian syntax appears in it.

The per-backend entry-point files (`CLAUDE.md`, `AGY.md`, etc.) are **symlinks** that all resolve to `AGENTS.md`. Editing any of them edits `AGENTS.md`. When a backend CLI starts inside the agent directory it reads its own named file — and gets the same universal instruction set.

The full set of symlinks the template creates:

```
agents/agent-<name>/
├── AGENTS.md          ← the single source of truth
├── CLAUDE.md    → AGENTS.md   (Claude Code backend)
├── AGY.md       → AGENTS.md   (Antigravity backend)
├── CODEX.md     → AGENTS.md   (Codex backend)
├── KIMI.md      → AGENTS.md   (Kimi / Moonshot backend)
├── COPILOT.md   → AGENTS.md   (GitHub Copilot backend)
├── OLLAMA.md    → AGENTS.md   (self-hosted Ollama via bwoc-harness)
└── OPENAI.md    → AGENTS.md   (OpenAI-compatible endpoint via bwoc-harness)
```

Verify that these are symlinks and not regular files:

```bash
ls -la agents/agent-<name>/CLAUDE.md
# should show: CLAUDE.md -> AGENTS.md
```

If any backend file is a regular file with its own content, it has diverged and is a neutrality violation. Fix it:

```bash
cd agents/agent-<name>
rm CLAUDE.md
ln -s AGENTS.md CLAUDE.md
```

The implication of this design: you do not need to maintain separate instruction files per backend. One edit to `AGENTS.md` (or any of its symlinks) propagates everywhere. The only per-backend configuration lives in `config.manifest.json` — model identifiers, endpoint URLs, and verification gate commands.

---

## 3. Backend Reference Table

| Backend | `--backend` value | CLI binary on PATH | How it runs | Notes |
|---|---|---|---|---|
| Claude Code | `claude` | `claude` | Vendor CLI reads `CLAUDE.md` → `AGENTS.md` directly | Default backend when none is specified |
| Antigravity | `antigravity` | `agy` | Vendor CLI reads `AGY.md` → `AGENTS.md` directly | |
| Codex | `codex` | `codex` | Vendor CLI reads `CODEX.md` → `AGENTS.md` directly | |
| Kimi (Moonshot) | `kimi` | `kimi` | Vendor CLI reads `KIMI.md` → `AGENTS.md` directly | |
| GitHub Copilot | `copilot` | `copilot` | Vendor CLI reads `COPILOT.md` → `AGENTS.md`; runs in programmatic mode (`copilot -p … --no-ask-user`) | Added in a recent version of `bwoc`; see caveat in [Section 4.4](#44-backend-override-with---backend) |
| Ollama (self-hosted) | `ollama` | none needed | `bwoc-harness` bridges the agent to Ollama's OpenAI-compatible API | Default endpoint `http://localhost:11434/v1`; override with `baseUrl` in `config.manifest.json` |
| OpenAI-compatible | `openai-compatible` | none needed | `bwoc-harness` bridges the agent to any OpenAI-compatible HTTP endpoint | **`baseUrl` in `config.manifest.json` is required**; `bwoc spawn` returns a clear error if absent |

**Vendor CLIs** (the top five rows) must be installed and on your `PATH` before you can use them. BWOC does not bundle them. If `claude` is not on PATH, `bwoc spawn --backend claude` will fail immediately with a binary-not-found error.

**Harness backends** (`ollama`, `openai-compatible`) do not require a vendor CLI. `bwoc-harness` is bundled with BWOC and handles the HTTP communication with the endpoint. The `--tui` flag on `bwoc chat` is only available for harness backends.

---

## 4. Launch It

There are three commands for driving an agent. Choose based on what you want:

| Goal | Command |
|---|---|
| Open an interactive session with the configured backend CLI | `bwoc chat <agent>` |
| Exec the backend CLI directly, optionally passing extra flags | `bwoc spawn --path <agent-dir>` |
| Run a single task non-interactively (scripts, CI) | `bwoc run <agent> --task "..."` |

### 4.1 `bwoc spawn` — raw backend exec

`bwoc spawn` is the lowest-level command. It execs the agent's configured backend CLI inside the agent directory. Once launched, the backend CLI has full control.

```
bwoc spawn [OPTIONS] [-- <EXTRA>...]
```

| Flag | What it does | Default |
|---|---|---|
| `--path <PATH>` | Path to the agent directory | current directory |
| `--backend <BACKEND>` | Backend to invoke; see table in Section 3 | value from `config.manifest.json` |
| `--lang <LANG>` | Language for CLI output | system default |
| `[-- <EXTRA>...]` | Extra arguments passed verbatim to the backend CLI | none |

Examples:

```bash
# Spawn from inside the agent dir using its configured backend
cd agents/agent-sage && bwoc spawn

# Spawn from anywhere, naming the agent dir
bwoc spawn --path agents/agent-sage

# Spawn with a specific backend
bwoc spawn --path agents/agent-sage --backend claude

# Pass extra flags through to the backend CLI
bwoc spawn --path agents/agent-sage -- --verbose
```

### 4.2 `bwoc chat` — interactive session

`bwoc chat` wraps `bwoc spawn` with convenience options for tmux, Ghostty, and the full-screen TUI.

```
bwoc chat [OPTIONS] <NAME>
```

| Flag | What it does | Default |
|---|---|---|
| `<NAME>` (required) | Agent name (`agent-sage` or bare `sage`) | — |
| `--tmux` | Open in a tmux window (or start a `bwoc-<id>` session if not already in tmux) | off |
| `--ghostty` | Open in a new Ghostty terminal window (macOS) | off |
| `--tui` | Full-screen ratatui chat client; harness backends (`ollama`, `openai-compatible`) only | off |
| `--workspace <PATH>` | Workspace root override | auto |
| `--lang <LANG>` | Language for CLI output | system default |

Examples:

```bash
# Start a chat session with the configured backend
bwoc chat sage

# Keep the session in a tmux window so you can detach and reattach
bwoc chat sage --tmux

# Open in a dedicated Ghostty window (macOS)
bwoc chat sage --ghostty

# Full-screen TUI (ollama or openai-compatible only)
bwoc chat sage --tui

# Use the full agent ID
bwoc chat agent-zhongkui
```

### 4.3 `bwoc run` — headless one-shot task

`bwoc run` delivers a task to an agent non-interactively. The backend process runs, completes the task, and exits. Use this in scripts and CI pipelines.

```
bwoc run [OPTIONS] --task <TASK> <AGENT>
```

| Flag | What it does | Default |
|---|---|---|
| `<AGENT>` (required) | Agent name | — |
| `--task <TASK>` (required) | Task prompt to deliver; must be a flag, not a positional argument | — |
| `--timeout <SECONDS>` | Kill the process and report failure after this many seconds | none |
| `--json` | Emit `{ agent, backend, task, exit_code, duration_ms, output }` to stdout | off |
| `--workspace <PATH>` | Workspace root override | auto |
| `--lang <LANG>` | Language for CLI output | system default |

Examples:

```bash
# Ask an agent to do one thing and exit
bwoc run sage --task "summarise the last hour of error logs"

# With a timeout (seconds)
bwoc run sage --task "check for SQL injection patterns" --timeout 120

# Capture structured output for scripting
bwoc run sage --task "generate release notes" --json
bwoc run sage --task "run lint" --json | jq '.exit_code'
```

### 4.4 Backend override with `--backend`

All three launch commands accept `--backend` to override the backend recorded in `config.manifest.json`. This lets you test the same agent against a different model or endpoint without changing any files.

```bash
# Same agent, different backend
bwoc spawn --path agents/agent-sage --backend ollama
bwoc spawn --path agents/agent-sage --backend codex
bwoc spawn --path agents/agent-sage --backend openai-compatible
```

**Important caveat — check the live list before scripting:** The set of accepted `--backend` values grows as new backends are added to the framework. The Copilot backend, for example, was added in a recent version and older binaries may not list it yet. Always run the following to see which values your installed binary accepts:

```bash
bwoc spawn --help
```

The `--backend` flag's accepted values are listed in that output. If a backend you expect is absent, update your `bwoc` binary.

---

## 5. Configure by Prompt

Once inside any backend CLI session, you drive the agent with natural language. The agent auto-loads its full identity from `AGENTS.md` + the slot directories (`persona/`, `mindsets/`, `skills/`, `interconnect/`, `memories/`) as soon as the session starts. You do not need to re-introduce the agent to itself.

What the agent loads automatically at session start:

- **Identity and scope** — from `AGENTS.md` sections 1 (Identity) and 2 (Task Planning)
- **Mindsets** — from `mindsets/*.md` (decision frames for how it thinks)
- **Skills** — from `skills/*.md` (bounded capabilities it can execute)
- **Memory index** — from `MEMORY.md` and `memories/` (what it has learned across sessions)
- **Verification gates** — `lintCmd`, `formatCmd`, `testCmd`, `buildCmd` from `config.manifest.json` (run these to know the work is done)
- **Scope and anti-scope** — what this agent does and explicitly refuses

You can ask the agent about any of this before giving it work:

```
"Summarise your scope and out-of-scope boundaries."
"List the skills you have declared and their maturity levels."
"What mindset do you apply when you are unsure whether a change is safe?"
```

You can also direct the agent to configure itself through natural language. Because the agent is operating inside its own directory and has file-write access, it can update its own configuration files when you ask it to. Examples:

```
"Set my test gate to `cargo test --workspace` and update the manifest."

"Add a new skill called web-research with maturity L1 and domain/research tag."

"Update the outOfScope field in config.manifest.json to clarify that you do not modify database schemas directly."

"My primary model has changed to gemma3:27b — update the manifest and confirm the change."

"Add a mindset called minimal-footprint with the principle/mattanutata anchor."

"Review AGENTS.md section 6 and confirm the four verification gate commands match the manifest."
```

After any agent-driven edit to `AGENTS.md` or `config.manifest.json`, run `bwoc check` from your terminal (outside the session) to confirm the agent did not introduce a neutrality violation:

```bash
bwoc check agents/agent-<name>
```

---

## 6. Make Config Stick — the Manifest

Changes made during a session are only persistent if they land in a file. The fields that control agent behaviour across sessions all live in `config.manifest.json`. The agent can edit this file during a session, or you can edit it directly.

The fields most relevant to backend configuration:

| Field | What it controls | Example |
|---|---|---|
| `primaryModel` | The primary LLM model identifier for this agent | `"claude-opus-4-7"`, `"gemma3:27b"`, `"gpt-4o"` |
| `fallbackModel` | Fallback model if primary is unavailable | `"claude-sonnet-4-6"` |
| `baseUrl` | Endpoint URL for `ollama` or `openai-compatible` backends | `"http://localhost:11434/v1"`, `"http://192.168.1.113:11434/v1"` |
| `lintCmd` | Lint / static analysis command (the Samvara verification gate) | `"cargo clippy -- -D warnings"` |
| `formatCmd` | Format check command (the Pahana gate) | `"cargo fmt --check"` |
| `testCmd` | Test command (the Bhavana gate) | `"cargo test --workspace"` |
| `buildCmd` | Build command (the Anurakkhana gate) | `"cargo build --release"` |
| `scopeDescription` | One-line "this agent does X"; fills `{{scopeDescription}}` in `AGENTS.md` | `"review pull requests for security issues"` |
| `outOfScope` | One-line "this agent does NOT do Y"; fills anti-scope in `AGENTS.md` | `"write new features, modify production config"` |

A minimal example of the backend-relevant section of `config.manifest.json`:

```json
{
  "name": "sage",
  "agentId": "agent-sage",
  "version": "2.0",
  "agentRole": "security review assistant",
  "primaryModel": "claude-opus-4-7",
  "fallbackModel": "claude-sonnet-4-6",
  "baseUrl": "http://localhost:11434/v1",
  "lintCmd": "cargo clippy -- -D warnings",
  "formatCmd": "cargo fmt --check",
  "testCmd": "cargo test",
  "buildCmd": "cargo build --release",
  "scopeDescription": "review code for security issues",
  "outOfScope": "write new features, modify infrastructure",
  "memoryPath": "memories/"
}
```

**`baseUrl` notes:**
- For `ollama`: optional. If absent, `bwoc-harness` defaults to `http://localhost:11434/v1`. Set it when Ollama is on a different host.
- For `openai-compatible`: required. `bwoc spawn` returns a clear error if absent.

After editing `config.manifest.json` (whether by hand or through a prompt), always validate:

```bash
bwoc check agents/agent-<name>
```

`bwoc check` validates:
1. `AGENTS.md` backend neutrality (no frontmatter, no wikilinks, no hardcoded model IDs or vendor names)
2. `config.manifest.json` is valid JSON with all required fields present
3. `MEMORY.md` is at or below 200 lines

Exit code `0` means the agent is ready. Non-zero means there are violations; read the output and fix them before running the agent again.

---

## 7. Add a New Backend

If a backend CLI exists that BWOC does not ship a symlink for by default — or if you want to explicitly add `COPILOT.md` to an older agent — one command is all it takes:

```bash
cd agents/agent-<name>
ln -s AGENTS.md COPILOT.md
```

No code change. No manifest update. `bwoc spawn --backend copilot` will now find `COPILOT.md` through the symlink and exec the `copilot` binary inside the agent directory.

The same pattern works for any backend whose CLI reads a named Markdown file from the current directory:

```bash
cd agents/agent-<name>
ln -s AGENTS.md <BACKEND>.md
```

After adding the symlink, run `bwoc check` to confirm the new file resolves to `AGENTS.md` and is not a regular file:

```bash
bwoc check agents/agent-<name>
```

To spawn with the new backend:

```bash
bwoc spawn --path agents/agent-<name> --backend copilot
# or, if the --backend value is not yet in the CLI list, exec the binary directly:
cd agents/agent-<name> && copilot -p "..." --no-ask-user
```

---

## 8. Troubleshooting

| Symptom | What to try |
|---|---|
| `bwoc spawn` fails with "binary not found" | The vendor CLI (`claude`, `agy`, `codex`, `kimi`, `copilot`) is not on your `PATH`. Install it first; BWOC does not bundle vendor CLIs. |
| `bwoc spawn` fails with "`baseUrl` is required" | The agent uses the `openai-compatible` backend and `baseUrl` is missing from `config.manifest.json`. Add it. |
| `bwoc spawn --backend <value>` reports "unrecognized backend" | Your `bwoc` binary predates the backend you are naming. Run `bwoc spawn --help` to see the accepted values, then update `bwoc` if needed. |
| Backend CLI starts but agent seems confused about its identity | The backend file (`CLAUDE.md`, etc.) may have diverged from `AGENTS.md`. Run `ls -la agents/agent-<name>/CLAUDE.md` — it must show `-> AGENTS.md`. If it is a regular file, delete it and re-create the symlink. |
| `bwoc check` fails after a prompt-driven edit | The agent wrote something to `AGENTS.md` that violates neutrality (YAML frontmatter, wikilinks, hardcoded model ID). Read the `bwoc check` output, locate the line, fix it. |
| Ollama backend times out or refuses connections | Confirm Ollama is running and reachable at the URL in `baseUrl` (default `http://localhost:11434/v1`). Test with `curl http://localhost:11434/v1/models`. |
| OpenAI-compatible backend returns auth errors | Check that the endpoint at `baseUrl` is running and that any required API key is set in the environment variable the harness expects. |
| `bwoc chat --tui` prints "backend does not support TUI" | The TUI is only available for `ollama` and `openai-compatible` backends. For vendor CLIs, omit `--tui` and let the vendor CLI render its own interface. |
| Agent ignores a manifest change | Make sure you saved `config.manifest.json` and the JSON is valid (`bwoc check` will catch syntax errors). Model IDs in the manifest take effect at the next session start — the running session still uses whatever model was loaded when it launched. |

---

## 9. See Also

| Topic | Link |
|---|---|
| End-user handbook — workspace init, agent lifecycle, teams | [`../end-user/HANDBOOK.en.md`](../end-user/HANDBOOK.en.md) |
| Agent author handbook — incarnation, AGENTS.md rule, manifest fields, slots | [`../agents/HANDBOOK.en.md`](../agents/HANDBOOK.en.md) |
| Glossary — all specialized and Pali terms | [`../glossary.en.md`](../glossary.en.md) |
| Framework repo root | [github.com/bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) |
| Incarnation spec — deep `bwoc new` and template reference | [INCARNATION.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/INCARNATION.en.md) |
| Harness spec — `bwoc-harness` internals, Ollama and OpenAI-compatible protocol | [HARNESS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/HARNESS.en.md) |
| Agent template AGENTS.md spec | [modules/agent-template/AGENTS.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md) |
| VERSION.md (current CalVer) | [VERSION.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) |

---

*On any conflict between this handbook and the [framework repo](https://github.com/bemindlabs/BWOC-Framework), the repo is correct — this handbook has a bug.*
