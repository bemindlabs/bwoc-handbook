# Fleet Operations Handbook

🇹🇭 [ภาษาไทย](HANDBOOK.th.md)

For **operators and team leads** who run many agents together and need them to stay healthy, observable, and recoverable — not just one agent at a time.
If you are still learning basic agent usage, start at [`../end-user/HANDBOOK.en.md`](../end-user/HANDBOOK.en.md).
If you are tuning trust and signing between agents, see [`../security/HANDBOOK.en.md`](../security/HANDBOOK.en.md).
Term lookup: [`../glossary.en.md`](../glossary.en.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)

---

## Table of Contents

1. [The Ops Problem at Scale](#1-the-ops-problem-at-scale)
2. [Fleet Health — The Seven Signals](#2-fleet-health--the-seven-signals)
3. [Keeping Agents Alive — supervise](#3-keeping-agents-alive--supervise)
4. [Visibility — Sessions, Lists, Ping, Dashboard, Logs, Inboxes](#4-visibility--sessions-lists-ping-dashboard-logs-inboxes)
5. [Diagnose and Repair](#5-diagnose-and-repair)
6. [Bulk Operations](#6-bulk-operations)
7. [Daily Ops Checklist](#7-daily-ops-checklist)
8. [See Also](#8-see-also)

---

## 1. The Ops Problem at Scale

One agent is easy to manage. Ten agents sharing a workspace — each with its own daemon, inbox, memory, and task assignments — introduce a different class of problem:

- An agent crashes silently and no one notices for hours.
- Stale registry entries accumulate from experiments that were never cleaned up.
- A bug in one agent triggers a restart loop that consumes resources and blocks shared queues.
- You can't tell at a glance which agents are idle, which are busy, and which are dead.
- A scheduled bulk restart becomes risky because you have no single command to stop everything safely.

Fleet operations is the practice of keeping all of that under control. BWOC provides a set of commands specifically for this layer. None of them replace the individual-agent commands you already know; they compose with them.

---

## 2. Fleet Health — The Seven Signals

### What `bwoc fleet health` does

```bash
bwoc fleet health
```

This command checks seven health signals across every registered agent in the workspace and prints a report. It is **read-only** — it never changes anything.

The seven signals come from a Buddhist governance principle called *Aparihāniya-dhamma* (the seven conditions of non-decline). In BWOC the name is an engineering label, not a religious reference. You can read it as: *seven conditions that, if violated, cause a multi-agent system to decay*. When all seven pass, the fleet is considered healthy.

| # | Signal | What it checks |
|---|--------|----------------|
| 1 | **Assembly** | All registered agents have a readable, valid `AGENTS.md` |
| 2 | **Accord** | No conflicting task assignments across agents in the same team |
| 3 | **Non-addition** | No unregistered agent directories exist alongside registered ones (ghost dirs) |
| 4 | **Seniority** | Agents with declared trust levels have valid, unexpired signing evidence |
| 5 | **Non-coercion** | No agent inbox has been stalled (unread messages older than the configured threshold) |
| 6 | **Refuge** | Each active agent has a reachable supervise daemon or a recorded clean-exit |
| 7 | **Welfare** | Memory files (`MEMORY.md`) are within the 200-line limit across the fleet |

A passing run looks like:

```
Fleet health — 12 agents checked
  ✓ Assembly     12/12
  ✓ Accord       no conflicts
  ✓ Non-addition no ghost dirs
  ✓ Seniority    4 signed, 8 unsigned (trust level: none — ok)
  ✓ Non-coercion no stalled inboxes
  ✓ Refuge       10 supervised, 2 clean-exit
  ✓ Welfare      all MEMORY.md within 200 lines

All seven signals pass.
```

A failing run names the specific agents and signals that need attention:

```
Fleet health — 12 agents checked
  ✗ Refuge       agent-loki: no supervise daemon, last exit unrecorded
  ✗ Welfare      agent-atlas: MEMORY.md 247 lines (limit 200)

2 signals failed. Run `bwoc doctor --auto` to auto-fix safe issues.
```

Run `bwoc fleet health` at the start of each shift and after any bulk operation.

---

## 3. Keeping Agents Alive — supervise

### The problem supervise solves

An agent daemon can crash — bad network call, model timeout, unhandled panic. Without supervision, the crash is silent. `bwoc supervise` wraps an agent's daemon in a restart loop and provides a crash-loop backstop so a broken agent does not thrash indefinitely.

### Usage

```bash
bwoc supervise <agent-path>
bwoc supervise agents/agent-sage
bwoc supervise agents/agent-sage --max-restarts-per-min 5
bwoc supervise agents/agent-sage --json
```

| Flag | Default | What it does |
|------|---------|--------------|
| `--max-restarts-per-min` | `10` | If the agent crashes and restarts more than this many times in one minute, the supervisor stops trying and emits a `rate_limit_hit` event. This prevents a broken agent from burning resources. |
| `--json` | off | Emit one JSON event per action to stdout. Useful for piping into a log aggregator. |

### Event stream (`--json`)

Each event is one JSON object on a single line:

| `event` value | When it fires |
|---------------|---------------|
| `spawn` | Agent daemon started for the first time |
| `crash_respawn` | Daemon exited unexpectedly; supervisor is restarting it |
| `clean_exit` | Daemon exited with status 0 (normal stop); supervisor exits |
| `rate_limit_hit` | Restart rate exceeded `--max-restarts-per-min`; supervisor exits |
| `signal_stop` | Supervisor received SIGTERM/SIGINT; daemon stopped cleanly |

### Crash-loop backstop

The `--max-restarts-per-min` guard exists so a misconfigured or broken agent does not restart endlessly. When the rate limit fires, the supervisor exits and logs the event. You will see the failure in `bwoc fleet health` (signal 6: Refuge) and in `bwoc log <agent>`. Fix the underlying problem, then restart the supervisor manually.

### Stopping a supervised agent

Send SIGTERM to the supervisor process or use:

```bash
bwoc stop agents/agent-sage
```

The supervisor catches the signal, sends a clean shutdown to the daemon, waits for it to exit, emits a `signal_stop` event, and then exits itself.

---

## 4. Visibility — Sessions, Lists, Ping, Dashboard, Logs, Inboxes

### 4.1 `bwoc sessions` — what is actually running right now

```bash
bwoc sessions
bwoc sessions --idle-secs 120
bwoc sessions --json
```

`bwoc sessions` detects running sessions by scanning process markers and live Unix socket activity. It distinguishes **working** (recent socket traffic) from **idle** (no recent traffic). Use `--idle-secs` to adjust how many seconds of silence classify a session as idle (default varies by build; check `bwoc sessions --help`). `--json` outputs one object per session.

### 4.2 `bwoc list` — registry-level filtering

```bash
bwoc list                          # all registered agents
bwoc list --running                # only agents with a live daemon
bwoc list --inbox-pending          # only agents with unread inbox messages
bwoc list --status <value>         # filter by status field in manifest
bwoc list --backend <value>        # filter by backend (claude, codex, ollama, …)
bwoc list --count                  # print a number, not a table
bwoc list --names-only             # print agent names only — useful in scripts
```

These flags compose. For example, count running Claude agents:

```bash
bwoc list --running --backend claude --count
```

Or get names of agents with pending inbox items for use in a shell loop:

```bash
for agent in $(bwoc list --inbox-pending --names-only); do
  bwoc inbox "$agent"
done
```

### 4.3 `bwoc ping` — is it responding?

```bash
bwoc ping agents/agent-sage        # ping one agent
bwoc ping --all                    # ping every running agent
```

Sends a `PING` over the agent's Unix socket and expects a `PONG`. If the daemon is frozen (running in the process list but not responding), `ping` will time out, which `sessions` cannot detect. Use ping when you suspect a hung agent.

### 4.4 `bwoc dashboard` — interactive overview

```bash
bwoc dashboard
```

Opens a terminal UI (TUI) showing all registered agents, their statuses, recent log lines, and inbox counts in a single pane. Press `r` to force a manual refresh. Useful for watching a fleet during a long batch run or after a deployment. Exit with `q` or `Ctrl-C`.

### 4.5 `bwoc log` — tail an agent's log

```bash
bwoc log agents/agent-sage
bwoc log agents/agent-sage -f           # follow (like tail -f)
bwoc log agents/agent-sage -f -n 50     # follow, show last 50 lines first
```

Reads the agent's structured log. `-f` keeps the stream open. `-n N` sets the number of historical lines shown before following.

### 4.6 `bwoc inbox` — read and watch messages

```bash
bwoc inbox --all                        # show all unread inbox items across the fleet
bwoc inbox agents/agent-sage            # show that agent's inbox
bwoc inbox agents/agent-sage --watch    # stream new inbox messages as they arrive
```

Inboxes are how agents receive tasks and inter-agent messages. A stalled inbox (fleet health signal 5) means messages are sitting unread, which usually means the agent's daemon is not running or not processing.

---

## 5. Diagnose and Repair

### 5.1 `bwoc doctor` — environment and workspace diagnostics

```bash
bwoc doctor
bwoc doctor --auto
```

`bwoc doctor` without flags diagnoses the environment and workspace: checks that required binaries are available, verifies workspace structure, validates registry consistency, and reports any issues it finds. `--auto` applies fixes that are safe to apply without human judgment (for example, regenerating a missing socket directory or rewriting a malformed lock file). Issues that require a human decision are reported but not auto-fixed.

Run `bwoc doctor --auto` whenever `bwoc fleet health` reports failures you do not immediately understand. In many cases it resolves the problem in one command.

### 5.2 `bwoc workspace validate` — check registry integrity

```bash
bwoc workspace validate
```

Checks the workspace registry (`.bwoc/agents.toml`) against the filesystem. Reports:

- **Phantom entries**: agents in the registry whose directories no longer exist.
- **Orphan directories**: agent directories that exist on disk but are not in the registry.
- **Manifest errors**: `config.manifest.json` files that are missing or invalid JSON.

This command is read-only. It tells you what is wrong; `prune` cleans it up.

### 5.3 `bwoc workspace prune` — clean up the registry

```bash
bwoc workspace prune               # dry run — shows what would be removed
bwoc workspace prune --apply       # actually remove phantom entries and orphan dirs
```

Without `--apply`, `prune` is a dry run that prints what it would do. Always run the dry run first to confirm the list of removals is what you expect. With `--apply`, phantom registry entries are deleted and orphan directories are removed from disk. This is the safe way to clean up after failed experiments or interrupted `bwoc retire` runs.

### Repair decision tree

```
bwoc fleet health reports failures
│
├── Signal 6 (Refuge): agent has no daemon
│   └── bwoc supervise <agent>  or  bwoc doctor --auto
│
├── Signal 7 (Welfare): MEMORY.md over limit
│   └── Manually prune the agent's MEMORY.md to ≤ 200 lines
│
├── Signal 3 (Non-addition): ghost dirs
│   └── bwoc workspace validate → bwoc workspace prune --apply
│
├── Signal 1 (Assembly): invalid AGENTS.md
│   └── bwoc check <agent> for details; fix the file
│
└── Anything unclear
    └── bwoc doctor --auto
```

---

## 6. Bulk Operations

### Stop all agents

```bash
bwoc stop --all
bwoc stop --all --yes              # skip the confirmation prompt
```

Sends a clean shutdown signal to every running daemon. Without `--yes`, the CLI asks for confirmation before proceeding. The `--yes` flag is for scripts and CI where no interactive prompt is available.

### Start all agents

```bash
bwoc start --all
bwoc start --all --yes
bwoc start --all --no-daemon       # start inline (foreground), not as a background daemon
```

Starts a daemon for every registered agent. `--no-daemon` starts each agent in the foreground (blocking). This is rarely useful in fleet context but available for debugging.

### Typical rolling restart

```bash
bwoc stop --all --yes
# wait for processes to clear
bwoc doctor --auto
bwoc start --all --yes
bwoc fleet health
```

Run `bwoc fleet health` after every bulk start to confirm all seven signals pass before considering the restart complete.

---

## 7. Daily Ops Checklist

This checklist takes under two minutes for a fleet of up to 20 agents.

```markdown
Morning
- [ ] bwoc fleet health               — all seven signals pass?
- [ ] bwoc sessions                   — unexpected idle or missing sessions?
- [ ] bwoc inbox --all                — stalled messages?
- [ ] bwoc dashboard                  — visual scan for anomalies

After any deployment or bulk change
- [ ] bwoc doctor --auto              — fix safe issues
- [ ] bwoc workspace validate         — phantom/orphan entries?
- [ ] bwoc fleet health               — re-confirm all seven signals
- [ ] bwoc ping --all                 — all running agents responding?

Weekly
- [ ] Review MEMORY.md sizes          — bwoc list + manual check, prune if near 200 lines
- [ ] bwoc workspace prune (dry run)  — any stale entries to clean?
- [ ] Review supervise logs           — any agents hitting rate_limit_hit repeatedly?
```

---

## 8. See Also

**Framework documentation (public GitHub)**

- [FLEET-GOVERNANCE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md) — authoritative specification for fleet governance, Aparihāniya-dhamma signal definitions, and workspace-level trust rules
- [WORKSPACE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/WORKSPACE.en.md) — workspace structure, registry format, multi-workspace setups

**Sibling handbook chapters**

- [`../end-user/HANDBOOK.en.md`](../end-user/HANDBOOK.en.md) — individual agent usage, sessions, inbox, basic lifecycle
- [`../agents/HANDBOOK.en.md`](../agents/HANDBOOK.en.md) — incarnating and tuning agents; the AGENTS.md rule; `bwoc check`
- [`../security/HANDBOOK.en.md`](../security/HANDBOOK.en.md) — trust levels, signing, inter-agent trust declarations
- [`../glossary.en.md`](../glossary.en.md) — term definitions including Pali labels used as engineering terms

---

> **Source of truth.** This handbook summarizes and orients. On any conflict, the framework repo wins and this page has a bug — please fix it. Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework).
