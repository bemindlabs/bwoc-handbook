# BWOC End-User Handbook

🇹🇭 [ภาษาไทย](HANDBOOK.th.md)

For people who **use** BWOC — run agents, chat, manage teams, assign tasks, read results — without editing framework source code.
If you want to create or tune agents yourself, see [`../agents/HANDBOOK.en.md`](../agents/HANDBOOK.en.md).
If you want to contribute to the framework, see [`../developer/HANDBOOK.en.md`](../developer/HANDBOOK.en.md).
Term lookup: [`../glossary.en.md`](../glossary.en.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)

---

## Table of Contents

1. [Install](#1-install)
2. [Environment and Resolution Rules](#2-environment-and-resolution-rules)
3. [Workspace Management](#3-workspace-management)
4. [Driving an Agent](#4-driving-an-agent)
5. [Mailbox and Observability](#5-mailbox-and-observability)
6. [Memory](#6-memory)
7. [Teams and Shared Tasks](#7-teams-and-shared-tasks)
8. [Agent Lifecycle](#8-agent-lifecycle)
9. [Fleet and Cross-Workspace](#9-fleet-and-cross-workspace)
10. [Workspace Documents](#10-workspace-documents)
11. [Troubleshooting](#11-troubleshooting)
12. [Go Deeper](#12-go-deeper)

---

## 1. Install

BWOC is distributed as a Rust binary. You build and install it from a clone of the framework repository.

### 1.1 Build from source

Requires **Rust 1.85+**. Runs on macOS, Linux, and Windows.

```bash
git clone https://github.com/bemindlabs/BWOC-Framework
cd BWOC-Framework
cargo install --path crates/bwoc-cli
```

Verify the install:

```bash
bwoc --help
bwoc help getting-started      # built-in getting-started guide
```

### 1.2 `bwoc completion` — generate shell completions

**Purpose.** Emit a shell completion script and install it so that Tab-completing `bwoc` commands works in your shell.

**Signature.**

```
bwoc completion <SHELL>
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<SHELL>` (required) | Target shell. One of: `bash`, `zsh`, `fish`, `powershell`, `elvish` | — |
| `--lang <LANG>` | Language for CLI output (`en` or `th`). See [Section 2](#2-environment-and-resolution-rules). | system default |

**Examples.**

```bash
# zsh — append to .zshrc
bwoc completion zsh > ~/.zsh/completions/_bwoc

# bash
bwoc completion bash >> ~/.bash_completion

# fish
bwoc completion fish > ~/.config/fish/completions/bwoc.fish
```

### 1.3 `bwoc update` — check for and apply upgrades

**Purpose.** Compare the running binary's embedded CalVer version against the latest GitHub release, and optionally delegate the upgrade to a package manager.

**Signature.**

```
bwoc update [OPTIONS]
```

| Flag | What it does | Default |
|---|---|---|
| `--check` | Read-only. Report version drift without making any changes. | off |
| `--run` | Execute the delegated upgrade command (e.g. `brew upgrade bwoc`) instead of only printing it. Raw binaries are never self-swapped. | off |
| `--lang <LANG>` | Language for CLI output. | system default |

**Notes.** Without `--check` or `--run`, the command prints the current and latest version plus the upgrade command. Add `--run` only when you want it to actually execute that command.

**Examples.**

```bash
bwoc update               # show current version, latest release, and upgrade command
bwoc update --check       # read-only: just report whether you are behind
bwoc update --run         # report AND execute the upgrade command
```

---

## 2. Environment and Resolution Rules

Every command that operates on a workspace or emits text output accepts two global options. Rather than repeating them for every command, they are documented once here.

### 2.1 Workspace resolution

When a command needs to find the workspace root it applies this priority order — stopping at the first match:

1. `--workspace <path>` flag on the command line
2. `BWOC_WORKSPACE` environment variable
3. Ancestor walk — searches parent directories from the current working directory until it finds a folder containing `.bwoc/`
4. Current working directory as a last resort

**Practical implication.** If you `cd` into any directory inside a workspace, BWOC finds the workspace automatically. You only need `--workspace` when you are operating on a workspace that is not an ancestor of your current directory (e.g. in scripts that manage multiple workspaces from a single shell).

### 2.2 Output language

When a command emits human-readable text, it applies this priority order for language selection:

1. `--lang <LANG>` flag on the command line
2. `BWOC_LANG` environment variable
3. The system `$LANG` variable
4. `en` fallback

Current supported values: `en`, `th`. Most commands also accept `--json` to bypass language selection entirely and emit structured JSON.

---

## 3. Workspace Management

A workspace is the root directory that holds all agents, teams, memory, task lists, and configuration for one project. Its canonical marker is the `.bwoc/` subdirectory.

### 3.1 `bwoc init` — create a workspace

**Purpose.** Initialize a new BWOC workspace at a path, writing `workspace.toml`, `.bwoc/`, and scaffolding directories.

**Signature.**

```
bwoc init [OPTIONS] [PATH]
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `[PATH]` | Directory to initialize. | current directory |
| `--force` | Overwrite an existing `workspace.toml`. Without this flag the command refuses to clobber. | off |
| `--no-runtime` | Scaffold without agent runtime/daemon provisioning. For CI, read-only, or inspection workspaces that never spawn agents. The workspace is still valid (`bwoc check` passes); daemon-ephemeral `.gitignore` patterns are omitted. | off |
| `--single-agent` | Initialize a single-agent workspace (one agent slot) instead of the multi-agent fleet default. Scaffolds single-agent-oriented guidance. | off (fleet default) |
| `--lang <LANG>` | Language for CLI output. | system default |
| `--json` | Emit `{ workspace, name, version, defaults, runtime, profile, files_created }` instead of the human-readable creation report. | off |

**Files created.** `workspace.toml`, `.bwoc/agents.toml`, `.bwoc/memory/`, `.bwoc/teams/`, and standard `.gitignore` entries (unless `--no-runtime`).

**Notes.** The agent registry at `.bwoc/agents.toml` is managed exclusively by CLI commands. Do not edit it by hand; direct edits can corrupt the registry.

**Examples.**

```bash
bwoc init ./my-project                         # create workspace in new directory
bwoc init .                                    # initialize current directory
bwoc init ./ci-workspace --no-runtime          # CI workspace, no daemon scaffolding
bwoc init ./single-ws --single-agent           # one-agent workspace
bwoc init ./existing-ws --force                # overwrite existing workspace.toml
bwoc init ./new-ws --json                      # parse the created-files report in a script
```

### 3.2 `bwoc workspace` — inspect a workspace

**Purpose.** Show workspace state, run validation rules, or find and fix inconsistencies. Has three subcommands.

**Signature.**

```
bwoc workspace [OPTIONS] <COMMAND>
```

| Option | What it does | Default |
|---|---|---|
| `--lang <LANG>` | Language for CLI output. | system default |

#### `bwoc workspace info`

Show the resolved workspace path, parsed config values, and agent count.

```bash
bwoc workspace info
```

#### `bwoc workspace validate`

Run all validation rules against the workspace. Exits `0` if fully valid; exits `2` if there are violations.

```bash
bwoc workspace validate
```

**Use in CI.** A non-zero exit from `validate` means the workspace has structural problems. Fix them before running agents.

#### `bwoc workspace prune`

Find inconsistencies: phantom registry entries (an entry in `agents.toml` whose directory no longer exists) and orphan directories (a directory that looks like an agent but is not registered). By default this is a dry run.

| Flag | What it does | Default |
|---|---|---|
| `--apply` | Actually fix the safe inconsistencies found (delete phantom entries, register orphans that look valid). Without `--apply` the output is informational only. | off (dry run) |

```bash
bwoc workspace prune             # list problems only
bwoc workspace prune --apply     # fix safe problems in place
```

### 3.3 `bwoc list` — list registered agents

**Purpose.** Show every agent registered in the workspace, with rich filtering and sorting options.

**Signature.**

```
bwoc list [OPTIONS]
```

| Flag | What it does | Default |
|---|---|---|
| `--workspace <PATH>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--status <STATUS>` | Filter by status (exact match). Common values: `active`, `stopped`, `retired`. | (all) |
| `--backend <BACKEND>` | Filter by backend. Values: `claude`, `antigravity`, `codex`, `kimi`, `ollama`, `openai-compatible`. | (all) |
| `--running` | Filter to agents whose daemon is actually running (PID file + signal-0 check). | off |
| `--inbox-pending` | Filter to agents with at least one pending message in their inbox. | off |
| `--sort <SORT>` | Sort key. One of: `id`, `inbox`, `incarnated`, `backend`. | registry insertion order |
| `--count` | Print just the count of matching agents as one integer. With `--json`, emits `{"count": N}`. | off |
| `--names-only` | Print bare agent IDs, one per line. Good for shell loops: `for name in $(bwoc list --names-only); do ...`. With `--json`, emits `{"names": [...]}`. `--count` wins if both are set. | off |
| `--json` | Emit JSON instead of the human-readable table. | off |
| `--lang <LANG>` | Language for CLI output. | system default |

**Examples.**

```bash
bwoc list                                        # all agents, human-readable table
bwoc list --status active                        # only active agents
bwoc list --backend ollama                       # only agents using Ollama
bwoc list --running                              # only agents with live daemons
bwoc list --inbox-pending --sort inbox           # agents with pending messages, sorted by inbox count
bwoc list --count                                # just the number of agents
bwoc list --names-only --status active           # bare IDs for shell scripting
for name in $(bwoc list --names-only --running); do bwoc ping "$name"; done
bwoc list --json | jq '.[].id'                   # extract IDs with jq
```

### 3.4 `bwoc doctor` — diagnose and repair

**Purpose.** Inspect the environment and workspace for common problems (missing binaries, broken symlinks, invalid manifests). Optionally fix safe issues automatically.

**Signature.**

```
bwoc doctor [OPTIONS]
```

| Flag | What it does | Default |
|---|---|---|
| `--workspace <PATH>` | Workspace root to diagnose. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--auto` | Attempt to fix safe issues automatically (missing directories, broken symlinks). | off |
| `--json` | Emit structured JSON instead of the human-readable list. | off |
| `--lang <LANG>` | Language for CLI output. | system default |

**Notes.** `bwoc doctor` is a good first step any time something behaves unexpectedly. Run it without `--auto` first to see what it finds, then re-run with `--auto` to apply safe fixes.

**Examples.**

```bash
bwoc doctor                    # inspect and report, no changes
bwoc doctor --auto             # inspect and fix safe issues
bwoc doctor --json             # machine-readable output for monitoring
bwoc doctor --auto --json      # fix then emit result as JSON
```

---

## 4. Driving an Agent

There are four distinct ways to interact with an agent, depending on whether you want a live conversation, a one-shot headless task, direct backend access, or asynchronous queued messaging.

### Backend overview

Every agent declares a `backend` in its manifest. BWOC supports these values:

| Backend | What it runs | Notes |
|---|---|---|
| `claude` | Claude CLI | Default. Execs the `claude` binary. |
| `antigravity` | Antigravity CLI | Execs the `antigravity` binary. |
| `codex` | Codex CLI | Execs the `codex` binary. |
| `kimi` | Kimi CLI | Execs the `kimi` binary. |
| `ollama` | `bwoc-harness` → Ollama | Execs `bwoc-harness` with endpoint `http://localhost:11434/v1`, or with `baseUrl` from the agent's `config.manifest.json` when present. |
| `openai-compatible` | `bwoc-harness` → any endpoint | Execs `bwoc-harness` and passes `baseUrl` from the agent's `config.manifest.json` as `--endpoint`. **`baseUrl` is required** for this backend; spawn returns a clear error when it is absent. |

`baseUrl` is a field in the agent's `config.manifest.json`. Set it to override the default endpoint for `ollama` or to specify a required endpoint for `openai-compatible`.

### 4.1 `bwoc chat` — interactive conversation

**Purpose.** Start a live, interactive chat session with an agent by exec'ing the configured backend CLI inside the agent's directory. The model and endpoint are resolved from the agent's manifest.

**Signature.**

```
bwoc chat [OPTIONS] <NAME>
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<NAME>` (required) | Agent name. Accepts the full ID (`agent-foo`) or bare name (`foo`). | — |
| `--tmux` | Run `bwoc spawn` under tmux. Adds a window when already inside a tmux session; otherwise auto-starts a `bwoc-<id>` named session. | off |
| `--ghostty` | Open a new Ghostty terminal window instead of exec'ing in this shell. macOS-only. | off |
| `--tui` | Full-screen ratatui chat client. Spawns `bwoc-harness --chat` and renders the `chat_proto` event stream. Works with `ollama` and `openai-compatible` backends only; other backends print a hint and fall back to exec. | off |
| `--workspace <WORKSPACE>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--lang <LANG>` | Language for CLI output. | system default |

**Notes.** `bwoc chat` is an exec-based command: once the backend CLI starts, BWOC hands control to it. Use `--tmux` to keep the session in a tmux window so you can detach and reattach. Use `--ghostty` on macOS for a dedicated terminal window. Use `--tui` for a self-contained full-screen experience with `ollama` or `openai-compatible` agents.

**Examples.**

```bash
bwoc chat sage                          # exec backend directly in this shell
bwoc chat sage --tmux                   # open in tmux window
bwoc chat sage --ghostty                # open in new Ghostty window (macOS)
bwoc chat sage --tui                    # full-screen TUI (harness backends only)
bwoc chat agent-zhongkui                # using full agent ID
```

### 4.2 `bwoc run` — headless one-shot task

**Purpose.** Deliver a task prompt to an agent non-interactively and capture the result. Designed for scripts, CI pipelines, and automation where you do not want an interactive session.

**Signature.**

```
bwoc run [OPTIONS] --task <TASK> <AGENT>
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<AGENT>` (required) | Agent name (`agent-foo` or `foo`). | — |
| `--task <TASK>` (required) | The task prompt to deliver to the agent. Must be provided as a flag — it is not a positional argument. | — |
| `--timeout <TIMEOUT>` | Kill the agent process and report a timeout failure if it runs longer than this many seconds. | none (no timeout) |
| `--json` | Emit `{ agent, backend, task, exit_code, duration_ms, output }` to stdout. | off |
| `--workspace <WORKSPACE>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--lang <LANG>` | Language for CLI output. | system default |

**Notes.** The `--task` flag is required. Do not pass the task as a positional argument — it will not be recognized. The `--json` output includes `exit_code` and `duration_ms`, which are useful for CI pass/fail gates.

**Examples.**

```bash
bwoc run sage --task "summarise the last hour of error logs"
bwoc run sage --task "check this file for SQL injection" --timeout 120
bwoc run sage --task "generate release notes" --json
bwoc run sage --task "run lint" --json | jq '.exit_code'
```

### 4.3 `bwoc spawn` — raw backend exec

**Purpose.** Exec the agent's configured backend CLI directly inside its directory, with optional extra arguments. Lower-level than `bwoc chat` — no manifest-driven model resolution; the backend CLI takes over completely.

**Signature.**

```
bwoc spawn [OPTIONS] [-- <EXTRA>...]
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `[-- <EXTRA>...]` | Extra arguments passed verbatim to the backend CLI after the `--` separator. | none |
| `--path <PATH>` | Path to the agent directory. | current directory |
| `--backend <BACKEND>` | LLM backend CLI to invoke. Values: `claude`, `antigravity`, `codex`, `kimi`, `ollama`, `openai-compatible`. | `claude` |
| `--lang <LANG>` | Language for CLI output. | system default |

**Notes.** `bwoc spawn` is the primitive that `bwoc chat` and `bwoc run` are built on. Use it when you need to pass flags directly to the backend CLI. `--path` defaults to the current directory, so you can `cd agents/agent-sage && bwoc spawn` or use `--path` from anywhere.

**Examples.**

```bash
cd agents/agent-sage && bwoc spawn                        # default backend from manifest
bwoc spawn --path agents/agent-sage --backend ollama      # override backend
bwoc spawn --path agents/agent-sage -- --verbose          # pass extra flags to the backend CLI
bwoc spawn --path agents/agent-sage --backend openai-compatible   # requires baseUrl in manifest
```

### 4.4 `bwoc send` — queue a message to an agent's inbox

**Purpose.** Append a message envelope to an agent's `.bwoc/inbox.jsonl`. The agent reads it at its next session start (or on a push notification if the daemon supports it). This is the asynchronous, non-interactive way to give an agent work.

**Signature.**

```
bwoc send [OPTIONS] <TO> <MESSAGE|--file <FILE>>
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<TO>` (required) | Recipient agent. Accepts `agent-foo` or `foo`. | — |
| `<MESSAGE>` | Message text. Quote multi-word strings. Mutually exclusive with `--file`. | — |
| `--file <FILE>` | Read the message body from a file. The file's full contents (minus trailing newlines) become the message body. Mutually exclusive with `<MESSAGE>`. | — |
| `--from <FROM>` | Sender identity. Default is `"user"` (human operator). Pass an agent name (`foo` or `agent-foo`) for agent-to-agent messaging. The named sender must exist in the workspace registry. The recipient's trust gate (when active) evaluates against this sender's manifest. | `"user"` |
| `--reply-to <REPLY_TO>` | Thread this message as a reply to a prior envelope. Value is the prior envelope's `messageId` (format: `msg-<slug>-<hex>`). Stamped as `replyTo` in the new envelope. The auto-reply Stop hook uses this to close a request/response loop. | none |
| `--no-wakeup` | Skip the best-effort tmux `send-keys` wakeup ping. Use in CI, daemons, and the auto-reply hook to avoid side-effecting into an interactive TUI session. | off |
| `--workspace <WORKSPACE>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--lang <LANG>` | Language for CLI output. | system default |

**Files touched.** Appends one JSON envelope line to `<agent-dir>/.bwoc/inbox.jsonl`.

**Notes.** Use `--from` with an agent name to simulate agent-to-agent messages or to trigger a recipient's trust policy. The `--reply-to` flag links messages into threads — useful when building automated request/response workflows between agents.

**Examples.**

```bash
bwoc send sage "Review the database schema for normalization issues"
bwoc send sage --file /tmp/schema.sql                          # send file contents
bwoc send sage "Summarize this" --from agent-orchestrator      # agent-to-agent
bwoc send sage "Follow up" --reply-to msg-review-abc123        # threaded reply
bwoc send sage "Run checks" --no-wakeup                        # CI-safe, no tmux ping
```

---

## 5. Mailbox and Observability

### 5.1 `bwoc inbox` — read an agent's inbox

**Purpose.** Read the message envelopes waiting in an agent's `.bwoc/inbox.jsonl`. Supports tail mode, clearing, and counting.

**Signature.**

```
bwoc inbox [OPTIONS] <AGENT|--all>
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<AGENT>` | Agent name (`agent-foo` or `foo`). Mutually exclusive with `--all`. | — |
| `--all` | Print every agent's inbox concatenated (each with a header). `--clear` and `--watch` are refused when `--all` is set. | off |
| `--limit <LIMIT>` | Show only the last N messages. | (all) |
| `--watch` | Tail mode — block and print new envelopes as they arrive. Press Ctrl-C to stop. Mutually exclusive with `--all`. | off |
| `--clear` | Truncate the inbox after printing (acknowledge and delete all messages). Prompts on TTY. | off |
| `--yes` | Skip the interactive confirmation for `--clear`. Required for non-TTY use. | off |
| `--count` | Print just the envelope count as one integer. With `--json`, emits `{"count": N}`. | off |
| `--json` | Emit JSON instead of the human-readable layout. | off |
| `--workspace <WORKSPACE>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--lang <LANG>` | Language for CLI output. | system default |

**Files touched.** Reads `<agent-dir>/.bwoc/inbox.jsonl`. With `--clear`, truncates that file.

**Examples.**

```bash
bwoc inbox sage                               # read all messages
bwoc inbox sage --limit 5                     # last 5 messages only
bwoc inbox sage --watch                       # live tail, Ctrl-C to stop
bwoc inbox sage --count                       # how many messages are waiting
bwoc inbox sage --clear --yes                 # read and delete all (scripted)
bwoc inbox --all                              # every agent's inbox at once
bwoc inbox sage --json | jq '.[].body'        # extract message bodies
```

### 5.2 `bwoc log` — tail the daemon log

**Purpose.** Read or stream the agent daemon's log file (`.bwoc/agent.log`), which captures stderr from the backend process.

**Signature.**

```
bwoc log [OPTIONS] <AGENT>
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<AGENT>` (required) | Agent name (`agent-foo` or `foo`). | — |
| `-f` / `--follow` | Block and stream new lines as they arrive. Ctrl-C to stop. | off |
| `-n` / `--lines <LINES>` | Number of trailing lines to print before `--follow` blocks (or as the full output when not following). | `50` |
| `--clear` | Truncate the log file before printing. Useful for starting a fresh observation. | off |
| `--workspace <WORKSPACE>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--lang <LANG>` | Language for CLI output. | system default |

**Files touched.** Reads `<agent-dir>/.bwoc/agent.log`. With `--clear`, truncates that file.

**Examples.**

```bash
bwoc log sage                    # last 50 lines
bwoc log sage -n 100             # last 100 lines
bwoc log sage -f                 # stream new lines, Ctrl-C to stop
bwoc log sage --clear -f         # clear old log then stream fresh
```

### 5.3 `bwoc status` — health and identity snapshot

**Purpose.** Show a per-agent health summary and identity information from the manifest. Read-only.

**Signature.**

```
bwoc status [OPTIONS] [NAME]
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `[NAME]` | Agent name. If omitted, shows the summary table for all agents. Mutually exclusive with `--all`. | (summary table) |
| `--all` | Print the full detail block for every agent (equivalent to calling the single-agent view for each). Mutually exclusive with `[NAME]` and `--banner`. | off |
| `--banner` | Replay the agent's startup liveness banner from its manifest (read-only, no daemon required). Requires a named agent. Mutually exclusive with `--all`. | off |
| `--workspace <WORKSPACE>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--json` | Emit JSON instead of the human-readable layout. | off |
| `--lang <LANG>` | Language for CLI output. | system default |

**Examples.**

```bash
bwoc status                      # table summary for all agents
bwoc status sage                 # detail for one agent
bwoc status sage --banner        # replay the startup liveness banner
bwoc status --all                # full detail block for every agent
bwoc status --json               # machine-readable output
```

### 5.4 `bwoc sessions` — list running agent sessions

**Purpose.** List agent sessions that are currently running, detected via session markers and process scan.

**Signature.**

```
bwoc sessions [OPTIONS]
```

| Flag | What it does | Default |
|---|---|---|
| `--idle-secs <IDLE_SECS>` | Seconds of inactivity before a session transitions from `working` to `idle`. | `60` |
| `--workspace <WORKSPACE>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--json` | Emit `{ "sessions": [...] }` to stdout instead of the table. | off |
| `--lang <LANG>` | Language for CLI output. | system default |

**Examples.**

```bash
bwoc sessions                         # who is currently running
bwoc sessions --idle-secs 120         # mark idle after 2 minutes
bwoc sessions --json                  # machine-readable session list
```

### 5.5 `bwoc ping` — liveness probe

**Purpose.** Send a `PING` message to a `bwoc-agent --serve`'d agent over its Unix socket and verify it responds with `PONG`. Confirms the daemon is alive and accepting connections.

**Signature.**

```
bwoc ping [OPTIONS] <NAME|--all>
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<NAME>` | Agent name. Mutually exclusive with `--all`. | — |
| `--all` | Ping every agent in the workspace. Agents with no live socket (not running) are labeled but do not fail the run. | off |
| `--workspace <WORKSPACE>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--lang <LANG>` | Language for CLI output. | system default |

**Exit codes.** Exits non-zero if a named single agent does not respond. With `--all`, non-running agents are reported but do not cause a non-zero exit.

**Examples.**

```bash
bwoc ping sage               # probe one agent
bwoc ping --all              # probe every agent in the workspace
```

### 5.6 `bwoc supervise` — crash-restart supervisor

**Purpose.** Watch an agent's daemon process and restart it if it crashes. Exits cleanly when the agent's status is set to `stopped` (e.g. via `bwoc stop`). Use this instead of a separate process manager when you want BWOC-aware supervision.

**Signature.**

```
bwoc supervise [OPTIONS] <AGENT>
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<AGENT>` (required) | Agent name (`agent-foo` or `foo`). | — |
| `--max-restarts-per-min <N>` | Maximum restarts within a rolling 60-second window. Beyond this limit the supervisor gives up to avoid burning CPU in a crash loop. | `10` |
| `--json` | Emit one JSON event per action (`spawn`, `crash_respawn`, `clean_exit`, `rate_limit_hit`, `signal_stop`) to stdout. Suitable for monitoring pipelines. | off |
| `--workspace <WORKSPACE>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--lang <LANG>` | Language for CLI output. | system default |

**Notes.** Run `bwoc supervise` in a background process or a dedicated tmux window. When the agent's status transitions to `stopped` (either via `bwoc stop` or by the agent itself), the supervisor exits cleanly.

**Examples.**

```bash
bwoc supervise sage                            # supervise with default restart limit
bwoc supervise sage --max-restarts-per-min 3   # tighter limit for a flaky agent
bwoc supervise sage --json | tee supervisor.log  # log events to file
```

---

## 6. Memory

BWOC has a two-tier memory system. Tier 1 is the file-based memory in `.bwoc/memory/` — short YAML/markdown entries you author or that agents write. Tier 2 is an optional deep memory store backed by an external command (configured in the agent's manifest as `deepMemoryCmd`) for past decisions, session transcripts, and accumulated knowledge beyond the 200-line `MEMORY.md` cap.

Each agent's `MEMORY.md` is capped at 200 lines. This deliberate limit keeps agent context sharp and prevents stale knowledge from accumulating. The Pali term *vaya* (see [`../glossary.en.md`](../glossary.en.md)) describes this pruning discipline.

**Signature (top-level).**

```
bwoc memory [OPTIONS] <COMMAND>
```

| Flag | What it does | Default |
|---|---|---|
| `--lang <LANG>` | Language for CLI output. | system default |

### 6.1 `bwoc memory list` — list memory entries

**Purpose.** List all user-authored memory entries in `.bwoc/memory/`.

```bash
bwoc memory list
```

### 6.2 `bwoc memory show` — print a memory entry

**Purpose.** Print one memory entry's contents to stdout.

```bash
bwoc memory show <KEY>           # print one entry
bwoc memory show --all           # print every entry concatenated
```

### 6.3 `bwoc memory put` — write a memory entry

**Purpose.** Write or overwrite a memory entry. Source selection follows this precedence: inline `[content]` argument > `--file <path>` > stdin.

```bash
bwoc memory put <KEY> "inline content"
bwoc memory put <KEY> --file notes.md
echo "content" | bwoc memory put <KEY>
```

### 6.4 `bwoc memory search` — substring search

**Purpose.** Search across all memory entries for a substring (case-insensitive).

```bash
bwoc memory search "database schema"
bwoc memory search "auth token"
```

### 6.5 `bwoc memory rm` — delete a memory entry

**Purpose.** Delete a memory entry. Prompts for confirmation on TTY.

```bash
bwoc memory rm <KEY>             # prompts for confirmation
bwoc memory rm <KEY> --yes       # skip confirmation (scripted use)
```

### 6.6 Tier 2: `bwoc memory wake-up`

**Purpose.** Emit prior context at session start by calling `deepMemoryCmd wake-up`. Used by agents to preload relevant past decisions at the beginning of a new session.

```bash
bwoc memory wake-up
```

### 6.7 Tier 2: `bwoc memory t2-search`

**Purpose.** Search past decisions and notes in the Tier 2 store by calling `deepMemoryCmd search "<query>"`. Named `t2-search` to avoid collision with the Tier 1 `search` subcommand.

```bash
bwoc memory t2-search "why did we switch to async processing"
```

### 6.8 Tier 2: `bwoc memory mine`

**Purpose.** Persist session learnings at session end by calling `deepMemoryCmd mine <path> --mode <mode>`. The agent runs this at the end of each session to extract and store insights into the Tier 2 store.

```bash
bwoc memory mine
```

---

## 7. Teams and Shared Tasks

A team — called a *Saṅgha* in the framework vocabulary (plain meaning: a named group of agents sharing a task list and coordinating as a unit — see [`../glossary.en.md`](../glossary.en.md)) — groups agents around a single shared task list. Use teams when work must be picked up, handed off, and completed by different agents in a cycle.

Team files on disk:
- `.bwoc/teams/<team>.toml` — membership and metadata (edit directly to add/remove members)
- `.bwoc/teams/<team>/tasks.jsonl` — the shared task list

### 7.1 `bwoc team create` — create a team

**Purpose.** Create a new team with an initial member list and register it in the workspace.

**Signature.**

```
bwoc team create <ID> [OPTIONS]
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<ID>` (required) | Team identifier (kebab-case, e.g. `security-ops`). | — |
| `--members <a,b,c>` | Comma-separated list of agent names to add as initial members. Sets members at creation time. | (empty) |
| `--lang <LANG>` | Language for CLI output. | system default |

**Notes.** There is no `member-add` subcommand. To add or remove members after creation, edit `.bwoc/teams/<team>.toml` directly.

**Examples.**

```bash
bwoc team create security-ops --members sage,zhongkui,erlang
bwoc team create research-team --members sage
```

### 7.2 `bwoc team list` — list all teams

**Purpose.** List teams in the workspace with member counts and task counts.

```bash
bwoc team list
```

### 7.3 `bwoc team retire` — disband a team

**Purpose.** Remove a team's membership file and task list from the workspace.

```bash
bwoc team retire security-ops
```

### 7.4 `bwoc task` — manage shared task lists

**Purpose.** Add, list, claim, complete, plan, approve, and reject tasks on a team's shared task list. Has seven subcommands.

**Signature (top-level).**

```
bwoc task [OPTIONS] <COMMAND>
```

| Flag | What it does | Default |
|---|---|---|
| `--lang <LANG>` | Language for CLI output. | system default |

#### `bwoc task add` — add a task

Add a new task to a team's list.

```
bwoc task add <TEAM> "<DESCRIPTION>" [--requires-plan] [--deps <a,b>]
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<TEAM>` (required) | Team name. | — |
| `<DESCRIPTION>` (required) | Task description (quote multi-word). | — |
| `--requires-plan` | Require the claiming agent to submit a plan and have it approved before the task can be completed. See the plan-approval workflow below. | off |
| `--deps <a,b>` | Comma-separated list of task IDs that must be completed before this task can be claimed. | none |

```bash
bwoc task add security-ops "Audit authentication endpoints"
bwoc task add security-ops "Pen-test API gateway" --requires-plan
bwoc task add security-ops "Write final report" --deps task-001,task-002
```

#### `bwoc task list` — list tasks

List all tasks on a team's shared list with their state and current claimant.

```bash
bwoc task list security-ops
```

#### `bwoc task claim` — claim a task

Mark a pending, unblocked task as in-progress and assign it to an agent.

```
bwoc task claim <TEAM> <TASK_ID> --as <AGENT>
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<TEAM>` (required) | Team name. | — |
| `<TASK_ID>` (required) | Task ID from `bwoc task list`. | — |
| `--as <AGENT>` (required) | Agent claiming the task. | — |

```bash
bwoc task claim security-ops task-001 --as sage
```

A task with `--deps` that are not yet completed cannot be claimed — the command will report it as blocked.

#### `bwoc task plan` — submit or show a plan (Pavāraṇā)

Submit a plan for a claimed task, revise an existing plan, or show the current plan. The *Pavāraṇā* (see [`../glossary.en.md`](../glossary.en.md)) principle requires the claiming agent to invite review before acting on high-impact work.

```
bwoc task plan <TEAM> <TASK_ID> --as <AGENT> [--text "<PLAN>"]
bwoc task plan <TEAM> <TASK_ID>       # show current plan (no --as or --text)
```

```bash
bwoc task plan security-ops task-001 --as sage --text "1. Map endpoints. 2. Fuzz inputs. 3. Report."
bwoc task plan security-ops task-001        # show the current plan
```

#### `bwoc task approve` — lead approves a plan

A team lead (another agent or the operator) approves the submitted plan. Once approved, the claimant may complete the task.

```
bwoc task approve <TEAM> <TASK_ID> --as <LEAD_AGENT>
```

```bash
bwoc task approve security-ops task-001 --as zhongkui
```

#### `bwoc task reject` — lead rejects a plan

A team lead rejects the submitted plan. The claimant must revise and resubmit with `bwoc task plan`.

```
bwoc task reject <TEAM> <TASK_ID> --as <LEAD_AGENT> [--reason "<WHY>"]
```

```bash
bwoc task reject security-ops task-001 --as zhongkui --reason "Scope too narrow — include auth headers."
```

#### `bwoc task complete` — close a task

Mark an in-progress task as completed. The agent must be the current claimant.

```
bwoc task complete <TEAM> <TASK_ID> --as <AGENT>
```

```bash
bwoc task complete security-ops task-001 --as sage
```

If the task was created with `--requires-plan` and the plan has not been approved, `complete` is refused until approval is granted.

### 7.5 Plan-approval workflow — worked example

The plan-approval flow is designed for high-stakes tasks where the team lead must review the approach before execution proceeds.

```
operator adds task with --requires-plan
        |
        v
agent claims the task ("claim")
        |
        v
agent submits a plan ("plan --text ...")
        |
        v
  lead reviews
    /         \
approve       reject
    |             |
    v             v
agent can     agent revises plan
complete      ("plan --text ..." again)
        |
        v
    complete
```

Concrete shell sequence:

```bash
# 1. Operator adds a plan-gated task
bwoc task add audit-team "Audit production database access" --requires-plan

# 2. Agent claims it
bwoc task claim audit-team task-003 --as sage

# 3. Agent submits a plan
bwoc task plan audit-team task-003 --as sage \
  --text "Step 1: Export access logs. Step 2: Cross-reference against ACL. Step 3: Flag anomalies. Step 4: Report."

# 4. Lead reviews and approves
bwoc task approve audit-team task-003 --as zhongkui

# 5. Agent completes the task
bwoc task complete audit-team task-003 --as sage
```

---

## 8. Agent Lifecycle

### 8.1 `bwoc new` — create a new agent

**Purpose.** Incarnate (create) a new agent from the template. Copies the template, fills placeholders, registers the agent in `agents.toml`, and creates backend symlinks. The Pali term *uppāda* (birth/arising — see [`../glossary.en.md`](../glossary.en.md)) names this phase.

**Signature.**

```
bwoc new [OPTIONS] <NAME>
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<NAME>` (required) | Agent name in kebab-case (e.g. `database-schema`). | — |
| `--target <TARGET>` | Target directory for the new agent. | `../agent-<name>/` relative to template |
| `--template <TEMPLATE>` | Path to the template directory. | auto-detect `modules/agent-template/` from cwd ancestors |
| `--backend <BACKEND>` | Primary backend recorded in the workspace registry. Values: `claude`, `antigravity`, `codex`, `kimi`, `ollama`, `openai-compatible`. | `claude` |
| `--role <ROLE>` | One-line role description. Prompted interactively if missing on a TTY. | — |
| `--primary-model <MODEL>` | Primary LLM model identifier. Prompted if missing on a TTY. | — |
| `--fallback-model <MODEL>` | Fallback LLM model identifier. Truly optional. | none |
| `--memory-path <PATH>` | File-based memory directory. | `memories/` |
| `--sessions-path <PATH>` | Session data directory for Tier 2 mining. Truly optional. | none |
| `--deep-memory-cmd <CMD>` | Tier 2 memory CLI command. Truly optional. | none |
| `--lint-cmd <CMD>` | Lint command for the verification gate. Prompted if missing on a TTY. | — |
| `--format-cmd <CMD>` | Format command for the verification gate. Prompted if missing on a TTY. | — |
| `--test-cmd <CMD>` | Test command for the verification gate. Prompted if missing on a TTY. | — |
| `--build-cmd <CMD>` | Build command for the verification gate. Prompted if missing on a TTY. | — |
| `--worktree-base <PATH>` | Base directory for worktrees. Truly optional. | `/tmp` |
| `--scope <SCOPE>` | Persona scope: one-line "this agent does X". Fills `{{scopeDescription}}`. | — |
| `--out-of-scope <TEXT>` | Persona anti-scope: one-line "this agent does NOT do Y". | — |
| `--primary-capability <TEXT>` | Longer description of what this agent is skilled at. Fills `{{primaryCapability}}`. Defaults to the role value when not provided. | (role value) |
| `--mindsets <LIST>` | Initial mindsets to seed — comma-separated kebab-case names (e.g. `verify-before-act,right-amount`). One stub `.md` per name. | none |
| `--skills <LIST>` | Initial skills to seed — comma-separated kebab-case names. One stub `.md` per name. | none |
| `--json` | Emit `{ agent_id, target, registered_in, symlinks, mindset_stubs, skill_stubs, persona_filled }` instead of the human-readable report. Useful for scripted multi-agent setup. | off |
| `--lang <LANG>` | Language for CLI output. | system default |

**Notes.** The template is auto-detected from cwd ancestors — if you run `bwoc new` from anywhere inside a framework clone, it finds `modules/agent-template/` automatically. Specify `--template` explicitly only when working outside a clone. After `bwoc new`, run `bwoc check <agent>` to confirm the agent is backend-neutral.

Full agent-authoring guidance: [`../agents/HANDBOOK.en.md`](../agents/HANDBOOK.en.md) and the template spec at [github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md).

**Examples.**

```bash
# Minimal — interactive prompts fill in the rest
bwoc new sage --role "research assistant" --target agents/agent-sage

# Fully non-interactive (CI/scripted)
bwoc new sage \
  --role "research assistant" \
  --backend ollama \
  --primary-model llama3.2 \
  --lint-cmd "cargo clippy" \
  --format-cmd "cargo fmt" \
  --test-cmd "cargo test" \
  --build-cmd "cargo build" \
  --target agents/agent-sage

# Create with initial mindsets and skills
bwoc new sage --role "analyst" \
  --mindsets "verify-before-act,right-amount" \
  --skills "data-analysis,report-writing" \
  --target agents/agent-sage

# JSON output for scripted multi-agent setup
bwoc new sage --role "assistant" --json --target agents/agent-sage
```

### 8.2 `bwoc check` — backend-neutrality audit

**Purpose.** Verify that an agent's `AGENTS.md` and manifest contain no backend-specific bindings (no YAML frontmatter, no wikilinks, no hardcoded model IDs, no vendor names). Also validates `config.manifest.json` (valid JSON) and `MEMORY.md` (≤ 200 lines).

**Signature.**

```
bwoc check [OPTIONS] [PATH]
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `[PATH]` | Path to the agent or template to audit. Mutually exclusive with `--all`. | current directory |
| `--all` | Audit every incarnated agent in the workspace (fleet-wide audit). Mutually exclusive with `[PATH]`. | off |
| `--workspace <WORKSPACE>` | Workspace root (used with `--all`). Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--json` | Emit JSON instead of the human-readable report. | off |
| `--lang <LANG>` | Language for CLI output. | system default |

**Exit codes.** Exits `0` if the agent passes all checks. Exits non-zero if violations are found. Use in CI to gate agent promotion.

**Examples.**

```bash
bwoc check agents/agent-sage          # audit one agent
bwoc check --all                      # audit every agent in the workspace
bwoc check agents/agent-sage --json   # machine-readable report
```

### 8.3 `bwoc stop` — pause an agent

**Purpose.** Set the agent's registry status to `stopped` without deleting files. The daemon process is terminated if running. Use this to temporarily pause an agent.

**Signature.**

```
bwoc stop [OPTIONS] <NAME|--all>
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<NAME>` | Agent to stop. Accepts `agent-foo` or `foo`. Mutually exclusive with `--all`. | — |
| `--all` | Stop every non-stopped agent in the workspace. | off |
| `--yes` | Skip interactive confirmation. Required for non-TTY (scripted) use. | off |
| `--json` | Emit `{ workspace, agent, daemon_outcome, registry_updated }`. Requires `--yes`. Single-agent only. | off |
| `--workspace <WORKSPACE>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--lang <LANG>` | Language for CLI output. | system default |

**Examples.**

```bash
bwoc stop sage                        # stop one agent (prompts for confirmation)
bwoc stop sage --yes                  # stop without prompt
bwoc stop --all --yes                 # stop every running agent
bwoc stop sage --yes --json           # scripted stop with JSON output
```

### 8.4 `bwoc start` — reactivate a stopped agent

**Purpose.** Set a stopped agent's status back to `active`. Optionally spawns the daemon.

**Signature.**

```
bwoc start [OPTIONS] <NAME|--all>
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<NAME>` | Agent to start. Accepts `agent-foo` or `foo`. Mutually exclusive with `--all`. | — |
| `--all` | Start every stopped agent in the workspace. | off |
| `--no-daemon` | Only flip registry status; do not spawn `bwoc-agent --serve`. | off |
| `--yes` | Skip interactive confirmation. Required for non-TTY use. | off |
| `--json` | Emit `{ workspace, agent, daemon_spawned, daemon_pid, ... }`. Requires `--yes`. Single-agent only. | off |
| `--workspace <WORKSPACE>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--lang <LANG>` | Language for CLI output. | system default |

**Examples.**

```bash
bwoc start sage                        # reactivate one agent
bwoc start sage --no-daemon            # flip status only, no daemon spawn
bwoc start --all --yes                 # reactivate all stopped agents
bwoc start sage --yes --json           # scripted start with JSON output
```

### 8.5 `bwoc retire` — permanently retire an agent

**Purpose.** Remove an agent from the workspace registry (*vaya* — the final phase of the lifecycle, see [`../glossary.en.md`](../glossary.en.md)). By default also removes the agent directory from disk.

**Signature.**

```
bwoc retire [OPTIONS] <NAME>
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `<NAME>` (required) | Agent to retire. Accepts `agent-foo` or `foo`. | — |
| `--keep-files` | Keep the agent directory on disk; only remove the registry entry. Mutually exclusive with `--keep-memory`. | off |
| `--keep-memory` | Preserve `memories/` while removing the rest of the agent directory. Lets you retire an agent but keep its accumulated knowledge. Mutually exclusive with `--keep-files`. | off |
| `--yes` | Skip interactive confirmation. Required for non-TTY use. | off |
| `--json` | Emit `{ workspace, agent, path, mode, registry_updated }`. Requires `--yes` (scripted destructive ops must explicitly acknowledge). | off |
| `--workspace <WORKSPACE>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--lang <LANG>` | Language for CLI output. | system default |

**Notes.** Without `--keep-files` or `--keep-memory`, the entire agent directory is deleted. This is irreversible. Use `--keep-memory` when you want the knowledge to survive but the agent identity to be removed.

**Examples.**

```bash
bwoc retire sage                          # retire and delete all files (prompts)
bwoc retire sage --yes                    # no prompt
bwoc retire sage --keep-files             # only remove registry entry
bwoc retire sage --keep-memory --yes      # delete files but preserve memories/
bwoc retire sage --yes --json             # scripted, JSON output
```

### 8.6 `bwoc trust` — view or generate trust credentials

**Purpose.** Read an agent's Kalyāṇamitta-7 trust profile (declared trust claims and `requiredTrust` thresholds from the manifest), or generate an ed25519 signing keypair for the agent.

*Kalyāṇamitta* (see [`../glossary.en.md`](../glossary.en.md)) refers to the trust-scoring mechanism that governs which agents can communicate with which others.

**Signature.**

```
bwoc trust [OPTIONS] [AGENT]
```

| Argument / Flag | What it does | Default |
|---|---|---|
| `[AGENT]` | Agent name (`agent-foo` or `foo`). Optional only with `--keygen --all`. | — |
| `--keygen` | Generate an ed25519 signing keypair instead of reading the profile. Private key is written to `<agent>/.bwoc/agent.key` (permissions 0600). Public key is written to the manifest field `signingPublicKey`. | off |
| `--all` | With `--keygen`: generate keypairs for every registered agent (backfill). | off |
| `--force` | With `--keygen`: overwrite an existing key (rotates the agent's identity). | off |
| `--workspace <WORKSPACE>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--json` | Emit JSON instead of the human-readable table. | off |
| `--lang <LANG>` | Language for CLI output. | system default |

**Files touched.** With `--keygen`: writes `<agent>/.bwoc/agent.key` (0600) and updates `signingPublicKey` in `config.manifest.json`.

**Examples.**

```bash
bwoc trust sage                           # read trust profile
bwoc trust sage --keygen                  # generate ed25519 keypair
bwoc trust sage --keygen --force          # rotate existing key
bwoc trust --keygen --all                 # generate keys for all agents (backfill)
bwoc trust sage --json                    # machine-readable trust profile
```

---

## 9. Fleet and Cross-Workspace

### 9.1 `bwoc fleet health` — fleet governance health check

**Purpose.** Check all 7 Aparihāniya-dhamma fleet-governance signals across the workspace. Read-only — no changes are made. *Aparihāniya-dhamma* (non-decline principles — see [`../glossary.en.md`](../glossary.en.md)) are the seven signals BWOC uses to measure whether a fleet is operating healthily.

**Signature.**

```
bwoc fleet health [OPTIONS]
```

| Flag | What it does | Default |
|---|---|---|
| `--lang <LANG>` | Language for CLI output. | system default |

**Example.**

```bash
bwoc fleet health                  # check all 7 governance signals
```

### 9.2 `bwoc peer` — view peer workspaces

**Purpose.** View peer workspaces declared in this workspace's `routes.toml`. Provides a read-only cross-workspace view for monitoring and collaboration.

**Signature.**

```
bwoc peer [OPTIONS] <COMMAND>
```

| Flag | What it does | Default |
|---|---|---|
| `--lang <LANG>` | Language for CLI output. | system default |

#### `bwoc peer list`

List all peers declared in this workspace's `routes.toml`.

```bash
bwoc peer list
```

#### `bwoc peer status <KEY>`

View a peer workspace's agents and open team tasks.

```bash
bwoc peer status my-peer-workspace
```

#### `bwoc peer learn <KEY>`

List or view shared documents from a peer's allowlist (`<peer>/.bwoc/interconnect/shared.toml`).

```bash
bwoc peer learn my-peer-workspace
```

#### `bwoc peer feedback <KEY> <AGENT>`

Give a peer agent signed cross-workspace feedback. The review is delivered as a signed envelope (`kind: feedback`) into the peer agent's inbox. The recipient verifies the sender's signature before accepting.

```bash
bwoc peer feedback my-peer-workspace sage
```

### 9.3 `bwoc dashboard` — interactive TUI dashboard

**Purpose.** Launch an interactive full-screen TUI showing the agents list with live status. Press `r` to refresh.

**Signature.**

```
bwoc dashboard [OPTIONS]
```

| Flag | What it does | Default |
|---|---|---|
| `--workspace <WORKSPACE>` | Workspace root. Resolution: see [Section 2.1](#21-workspace-resolution). | auto |
| `--lang <LANG>` | Language for CLI output. | system default |

**Example.**

```bash
bwoc dashboard
```

---

## 10. Workspace Documents

BWOC includes a lightweight document management system for workspace notes, retrospectives, and research docs. Three named aliases (`notes`, `retro`, `research`) are thin wrappers over the general `doc` command. Custom document kinds can be declared in `.bwoc/doc-kinds.toml`.

All document commands share the same three subcommands: `new`, `list`, `view`.

### 10.1 `bwoc notes` — workspace notes

```
bwoc notes new "<TITLE>"          # create YYYY-MM-DD_<slug>.md in notes/
bwoc notes list                   # list notes, newest first
bwoc notes view <DATE_PREFIX>     # print a note matching a date prefix or exact filename
```

### 10.2 `bwoc retro` — retrospectives

```
bwoc retro new "<TITLE>"          # create YYYY-MM-DD_<slug>.md in retrospectives/
bwoc retro list
bwoc retro view <DATE_PREFIX>
```

### 10.3 `bwoc research` — research documents

```
bwoc research new "<TITLE>"       # create YYYY-MM-DD_<slug>.md in research/
bwoc research list
bwoc research view <DATE_PREFIX>
```

### 10.4 `bwoc doc` — custom document kinds

**Purpose.** Manage documents of any kind — built-in or workspace-declared custom kinds (via `.bwoc/doc-kinds.toml`). Use this for custom kinds; the named aliases above are thin wrappers over this command.

**Signature.**

```
bwoc doc [OPTIONS] <COMMAND>
```

| Flag | What it does | Default |
|---|---|---|
| `--lang <LANG>` | Language for CLI output. | system default |

```bash
bwoc doc new <KIND> "<TITLE>"         # create a document of the given kind
bwoc doc list <KIND>                  # list documents of the given kind, newest first
bwoc doc view <KIND> <DATE_PREFIX>    # print a document matching a date prefix or exact filename
```

**Examples.**

```bash
bwoc notes new "Sprint 12 planning"
bwoc notes list
bwoc notes view 2026-06

bwoc retro new "Q2 retrospective"
bwoc retro list

bwoc research new "OAuth2 alternatives"
bwoc research view 2026-05-15

bwoc doc new incident "Database failover 2026-06-05"
bwoc doc list incident
```

---

## 11. Troubleshooting

| Symptom | What to try |
|---|---|
| `bwoc` command not found | Confirm `~/.cargo/bin` is on your `$PATH`. Re-run `cargo install --path crates/bwoc-cli` from the framework clone. |
| `error: workspace not found` or similar | Run `bwoc doctor`. Check that `.bwoc/` exists in or above your current directory. Set `BWOC_WORKSPACE` or use `--workspace`. |
| Agent not visible in `bwoc list` | You must be inside the same workspace. Verify `.bwoc/agents.toml` contains the agent. If you used `bwoc new`, confirm `--target` was correct. |
| `bwoc new` creates agent in wrong place | Use `--target agents/agent-<name>` explicitly. The auto-detect resolves relative to the template, not your cwd. See memory note [bwoc-new-ignores-workspace-env](../glossary.en.md) — fixed in v2.00+ with `--target`. |
| `bwoc run` ignores my task text | The task must be passed as `--task "<text>"`, not as a positional argument. |
| Agent is locked to one vendor | Run `bwoc check <agent>`. The neutrality audit will pinpoint hardcoded model IDs, vendor names, wikilinks, or YAML frontmatter. |
| `bwoc spawn` fails with `baseUrl is required` | The agent uses the `openai-compatible` backend, which requires a `baseUrl` field in its `config.manifest.json`. Add it. |
| Daemon is hung or dead | Run `bwoc log <agent> -f` to read the error. Use `bwoc supervise <agent>` to enable automatic crash-restarts. |
| `bwoc ping` gets no response | Confirm the daemon is running (`bwoc sessions`, `bwoc status <agent>`). The daemon must be started with `bwoc-agent --serve`. |
| `bwoc workspace validate` exits 2 | There are structural violations. Read the output, fix the flagged issues, re-run. |
| `bwoc workspace prune` shows phantom entries | Run `bwoc workspace prune --apply` to remove phantom registry entries or restore orphan directories. |
| `bwoc task complete` is refused | If the task was created with `--requires-plan`, the plan must be approved first (`bwoc task approve`). |
| Trust gate rejects a `bwoc send --from <agent>` | The sender agent must exist in the workspace registry. Run `bwoc list` to confirm. |
| MEMORY.md is over 200 lines | This will fail `bwoc check`. Prune the file to ≤ 200 lines before running the fleet-wide audit. |
| `bwoc update --run` does nothing | It delegates to your package manager's upgrade command (e.g. `brew upgrade bwoc`). Make sure `bwoc` was installed via that manager. |

If this page does not answer your question, see the FAQ at [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md) (Thai: [FAQ.th.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/th/FAQ.th.md)).

---

## 12. Go Deeper

| Topic | Link |
|---|---|
| All specialized / Pali terms | [`../glossary.en.md`](../glossary.en.md) |
| Creating and tuning agents | [`../agents/HANDBOOK.en.md`](../agents/HANDBOOK.en.md) |
| Framework source and developer guide | [`../developer/HANDBOOK.en.md`](../developer/HANDBOOK.en.md) |
| Agent template spec (AGENTS.md) | [github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md) |
| Full workspace structure docs | [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/WORKSPACE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/WORKSPACE.en.md) |
| Fleet governance docs | [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md) |
| Skills system docs | [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) |
| Plugins system docs | [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) |
| FAQ | [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md) |
| VERSION.md (current CalVer) | [github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) |
| Framework repo root | [github.com/bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) |
