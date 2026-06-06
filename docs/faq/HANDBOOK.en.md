# FAQ & Troubleshooting

> Thai version: [HANDBOOK.th.md](HANDBOOK.th.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) · Glossary: [../glossary.en.md](../glossary.en.md)

This page is the **handbook-friendly companion** to the canonical FAQ. The canonical FAQ — maintained alongside the framework source — lives at:

> [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md)

**On any conflict between this page and the canonical FAQ, the canonical FAQ is correct — and this page has a bug.** This companion gathers the most common questions in a task-oriented format, adds a troubleshooting table, and cross-links to the relevant handbook sections. It is not a duplicate; it is a navigational layer.

---

## Table of Contents

1. [General](#1-general)
2. [Setup and Workspace](#2-setup-and-workspace)
3. [Agents and Backends](#3-agents-and-backends)
4. [Memory and Self-Improvement](#4-memory-and-self-improvement)
5. [Troubleshooting Table](#5-troubleshooting-table)
6. [See Also](#6-see-also)

---

## 1. General

### Do I need to know Pali or Buddhism to use BWOC?

No. The Pali terms that appear in BWOC — *uppada*, *vaya*, *sila*, *sangha*, and so on — are engineering labels, not religious instruction. Each one was chosen because it maps precisely to a real software concern: lifecycle phases, memory pruning, security precepts, team coordination. You can ignore every Pali term entirely and think in plain English ("birth / live / retire", "prune memory", "don't escalate privileges"). The terms appear in the glossary when you are curious; they are never required reading.

See [../glossary.en.md](../glossary.en.md) for a plain-English definition of every term, and the [README](../README.md) for a quick five-sentence overview.

### Is BWOC tied to a specific AI vendor?

No. Backend-neutrality is a core design constraint. The same agent directory runs on Claude, Codex, Kimi, Antigravity, Copilot, Ollama, or any OpenAI-compatible endpoint. No vendor name, model ID, or vendor-specific syntax belongs in `AGENTS.md`. The CLI enforces this with `bwoc check`.

### What is the source of truth when the handbook and the framework repo disagree?

The framework repo always wins. This handbook — and its companion pages — orient and summarize. The specification lives in the framework source: `crates/`, `docs/en/`, `docs/th/`, and `modules/agent-template/AGENTS.md`. When you find a discrepancy, the handbook has a bug; please fix it.

### What Rust version is required to build BWOC?

Rust 1.85 or later. Run `rustup update stable` if your toolchain is behind. See the developer handbook at [../developer/HANDBOOK.en.md](../developer/HANDBOOK.en.md) for the full build walkthrough.

### What does "CalVer" mean for BWOC versioning?

BWOC uses two parallel versioning namespaces:

- **Cargo SemVer** — the `version` field in `Cargo.toml`. This tracks the development crate version and follows conventional SemVer (`MAJOR.MINOR.PATCH`).
- **CalVer release tags** — calendar-based tags (e.g., `v2.24.0` where `2.24` encodes the year-week or release cycle). These are what end-users and operators track.

The current `Software-Version` and `Document-Version` are in [VERSION.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) in the framework repo.

---

## 2. Setup and Workspace

### My `bwoc` command is not found after building. What do I check?

`cargo install` places binaries in `~/.cargo/bin/`. Confirm that directory is on your `$PATH`:

```bash
echo $PATH | tr ':' '\n' | grep cargo
```

If it is missing, add `export PATH="$HOME/.cargo/bin:$PATH"` to your shell profile and reload.

### Why is BWOC not finding my workspace?

BWOC resolves the workspace root using this priority order, stopping at the first match:

1. `--workspace <path>` flag on the command line
2. `BWOC_WORKSPACE` environment variable
3. Ancestor walk — searches parent directories from the current working directory until it finds a directory containing `.bwoc/`
4. Current working directory as a last resort

If you are inside the workspace directory tree, BWOC usually finds it automatically. If it does not, run `bwoc doctor` — it identifies resolution failures and reports what it found and where. Pass `--auto` to fix safe issues automatically:

```bash
bwoc doctor
bwoc doctor --auto
```

You can also set the workspace explicitly for a single command:

```bash
bwoc list --workspace /path/to/my-workspace
```

### Can I have multiple workspaces?

Yes. Each workspace is an independent directory tree identified by its `.bwoc/` marker. Agents in workspace A cannot see agents in workspace B through `bwoc list` — they are separate registries. Use `bwoc peer` to set up cross-workspace views when you need coordination across workspaces.

### What does `bwoc doctor --auto` actually fix?

Safe, mechanical issues: missing required directories, broken symlinks, and similar structural problems that have an unambiguous correct repair. It does not rewrite configuration, delete data, or make behavioral choices on your behalf. Run it without `--auto` first to review what it would do.

---

## 3. Agents and Backends

### How do I create a new agent?

Use `bwoc new`. Never hand-copy the template directory or hand-edit `.bwoc/agents.toml`. The CLI handles registry registration, placeholder substitution, and symlink creation atomically:

```bash
bwoc new sage --role "research assistant" --target agents/agent-sage
```

After creation, run `bwoc check agents/agent-sage` to confirm the agent passes the backend-neutrality audit. Full flag reference: [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md) and [../end-user/HANDBOOK.en.md](../end-user/HANDBOOK.en.md).

### How do I remove an agent?

Use `bwoc retire`. This removes the agent from the registry and, by default, deletes its directory:

```bash
bwoc retire sage --yes
```

To remove the registry entry but keep the files on disk, use `--keep-files`. To keep the memory directory while removing everything else, use `--keep-memory`. Never delete agent directories by hand — the registry will contain a phantom entry that confuses subsequent commands.

### My new agent does not appear in `bwoc list`. Why?

Three things to check in order:

1. **Same workspace.** `bwoc list` only shows agents registered in the workspace it resolves to. Confirm you are inside the correct workspace directory tree, or pass `--workspace` explicitly.
2. **`--target` was correct.** If you ran `bwoc new` without `--target`, the agent directory was created relative to the template location — not the workspace root. Pass `--target agents/agent-<name>` explicitly. See the memory note in the developer handbook about this (fixed in framework PR #200 / v2.00+; old binaries still need `--target`).
3. **Run `bwoc doctor`.** It lists phantom registry entries and orphan directories, which often reveals what went wrong.

### Which backend should I use?

It depends on what CLI you have installed and whether you are using a self-hosted model:

| Backend | How it runs | When to use it |
|---|---|---|
| `claude` | Execs the `claude` binary directly | You have Claude Code CLI installed and are using Anthropic's service |
| `agy` (antigravity) | Execs the `antigravity` binary | You have the Antigravity CLI installed |
| `codex` | Execs the `codex` binary | You have the Codex CLI installed |
| `kimi` | Execs the `kimi` binary | You have the Kimi / Moonshot CLI installed |
| `copilot` | Execs the `copilot` binary | You have the GitHub Copilot CLI installed |
| `ollama` | `bwoc-harness` → Ollama | Self-hosted Ollama; set `baseUrl` in `config.manifest.json` if not using the default `http://localhost:11434/v1` |
| `openai-compatible` | `bwoc-harness` → any endpoint | Any OpenAI-compatible API; **`baseUrl` is required** in `config.manifest.json` |

For the live list of supported backends and any new additions, run:

```bash
bwoc spawn --help
```

To switch an existing agent to a different backend at spawn time:

```bash
bwoc spawn --path agents/agent-sage --backend ollama
```

See [../backends/HANDBOOK.en.md](../backends/HANDBOOK.en.md) for configuration details for each backend.

### The `openai-compatible` backend fails immediately. What is missing?

The `openai-compatible` backend requires a `baseUrl` field in the agent's `config.manifest.json`. Without it, `bwoc spawn` returns a clear error rather than silently failing. Add the field:

```json
{
  "baseUrl": "https://your-endpoint.example.com/v1"
}
```

For Ollama with a non-default address, set `baseUrl` in the same way. For the default local Ollama endpoint (`http://localhost:11434/v1`), `baseUrl` is optional.

### My agent feels locked to one vendor — `bwoc check` says it is not neutral. What do I look for?

`bwoc check` audits `AGENTS.md` for four categories of backend-specific binding:

- **YAML frontmatter** — `---` ... `---` block at the top of the file
- **Wikilinks** — `[[...]]` syntax
- **Obsidian callouts** — `> [!type]` syntax
- **Hardcoded model IDs or vendor names** in behavioral content — e.g., `claude-opus-4`, `Antigravity`, `kimi-k2`

Run the audit and read its output:

```bash
bwoc check agents/agent-sage
bwoc check agents/agent-sage --json   # machine-readable for scripting
```

Replace every violation with backend-neutral equivalents: plain Markdown links instead of wikilinks, `{{primaryModel}}` placeholder instead of a hardcoded model ID, plain blockquotes instead of Obsidian callouts. The canonical list of every `bwoc check` failure mode and its fix is in [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md).

### The agent daemon crashed or is not responding. What do I do?

Start with the log:

```bash
bwoc log sage -f
```

This streams the daemon's stderr. Most crashes produce a clear error message there.

Then probe liveness:

```bash
bwoc ping sage
bwoc ping --all
```

If the daemon is down, use the supervisor to enable automatic crash-restarts:

```bash
bwoc supervise sage
```

The supervisor watches the daemon process and restarts it on crash. It exits cleanly when you run `bwoc stop sage`.

---

## 4. Memory and Self-Improvement

### What is the memory limit and why does it exist?

Each agent's `MEMORY.md` is hard-capped at **200 lines**. `bwoc check` enforces this and fails if the file exceeds the limit.

The limit is deliberate. Unbounded memory accumulation degrades agent context quality — stale, redundant, or low-signal entries crowd out what actually matters. The 200-line cap forces curation: keep what is non-obvious and still relevant; drop what is derivable from code, captured in git history, or no longer applicable. The framework's term for this pruning discipline is *anicca* (impermanence) — useful as a reminder that holding on to everything is a failure mode, not a virtue.

### What belongs in `MEMORY.md` versus what should be dropped?

**Save:**
- Non-obvious decisions and the reasoning behind them
- Approaches the user explicitly validated or corrected
- External resource locations (endpoints, repositories, tool names) that are not obvious from the code
- Patterns discovered through trial and error that are not yet documented elsewhere

**Drop:**
- Code patterns that can be derived by reading the code
- Git history (that is what version control is for)
- Content already in `AGENTS.md`, `conventions.md`, or slot docs
- Ephemeral session state that is no longer relevant

### How does self-improvement work?

Agents accumulate knowledge through a study / reflect / practice loop described in detail in [../self-improvement/HANDBOOK.en.md](../self-improvement/HANDBOOK.en.md). At the end of a session, `bwoc memory mine` persists session learnings into the Tier 2 deep memory store (when configured). At the start of a new session, `bwoc memory wake-up` preloads relevant past context.

Skill maturity levels (`L1` through `L7`) in slot files track declared capability — the agent author sets these honestly, not the framework. Overclaiming a higher level than the evidence supports is a threat-model violation; see the full maturity scale in [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md).

---

## 5. Troubleshooting Table

| Symptom | Likely Cause | Fix |
|---|---|---|
| `bwoc: command not found` | `~/.cargo/bin` not on `$PATH` | Add `export PATH="$HOME/.cargo/bin:$PATH"` to your shell profile; run `cargo install --path crates/bwoc-cli` again |
| `error: workspace not found` or similar | Not inside a workspace tree; `.bwoc/` missing or in wrong directory | Run `bwoc doctor --auto`; set `BWOC_WORKSPACE` or pass `--workspace` explicitly |
| Agent missing from `bwoc list` | Wrong workspace; `--target` was omitted during `bwoc new`; registry entry absent | Run `bwoc doctor`; pass `--workspace`; re-create agent with `--target agents/agent-<name>` |
| `bwoc new` created agent in wrong directory | `--target` not set; auto-detect resolves relative to template, not workspace root | Always pass `--target agents/agent-<name>` from the workspace root |
| `bwoc check` fails: neutrality violation | `AGENTS.md` contains YAML frontmatter, wikilinks, Obsidian callouts, or hardcoded model IDs / vendor names | Run `bwoc check --json` to get the precise line; fix each violation class; re-run until clean |
| `bwoc spawn` fails: `baseUrl is required` | Agent backend is `openai-compatible` but `baseUrl` is absent in `config.manifest.json` | Add `"baseUrl": "https://..."` to `config.manifest.json` |
| `bwoc check` fails: `MEMORY.md exceeds 200 lines` | Memory index has grown beyond the cap without pruning | Prune stale bullets from `MEMORY.md`; delete the corresponding files from `memories/` |
| Agent daemon crashed / not responding | Backend process error; upstream CLI crash | `bwoc log <agent> -f` to read error; `bwoc supervise <agent>` for auto-restart |
| `bwoc ping` gets no response | Daemon is not running | Check `bwoc sessions`, `bwoc status <agent>`; start daemon with `bwoc-agent --serve` or via `bwoc supervise` |
| Build fails: "error[E...]: requires Rust 1.85" | Toolchain is outdated | `rustup update stable` |
| Handbook page says one thing, GitHub docs say another | Handbook has a bug | The framework repo is the source of truth; follow the [canonical FAQ](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md) and consider filing a fix |

---

## 6. See Also

| Resource | Link |
|---|---|
| Canonical FAQ (source of truth) | [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md) |
| Canonical FAQ — Thai | [github.com/bemindlabs/BWOC-Framework/blob/main/docs/th/FAQ.th.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/th/FAQ.th.md) |
| End-user handbook (install, workspace, lifecycle) | [../end-user/HANDBOOK.en.md](../end-user/HANDBOOK.en.md) |
| Agent author and operator handbook | [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md) |
| Backends handbook | [../backends/HANDBOOK.en.md](../backends/HANDBOOK.en.md) |
| Self-improvement handbook | [../self-improvement/HANDBOOK.en.md](../self-improvement/HANDBOOK.en.md) |
| Glossary (all Pali and specialized terms) | [../glossary.en.md](../glossary.en.md) |
| VERSION.md (current CalVer + SemVer) | [github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) |
| Framework repo root | [github.com/bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) |

---

*This handbook is maintained alongside the framework. When behavior described here diverges from the [framework repo](https://github.com/bemindlabs/BWOC-Framework), the repo is correct — please open a fix.*
