# Quickstart — Your First Agent in ~10 Minutes

> Thai version: [HANDBOOK.th.md](HANDBOOK.th.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) · Glossary: [../glossary.en.md](../glossary.en.md)

This is a single, copy-paste walkthrough. By the end you will have installed `bwoc`, created a workspace, incarnated an agent, talked to it, run a task, formed a one-agent team, and then retired the agent cleanly. Every step takes about a minute; the whole thing fits in ten.

When you want the full story for any topic, the "What next" box at the bottom links you to the right chapter.

---

## The lifecycle you are about to walk

Every BWOC object follows three phases. The framework calls them *uppāda · ṭhiti · vaya*; plain English: **birth → live → retire**. This tutorial threads all three end-to-end so you see the complete arc once before exploring the details.

---

## Step 0 — Prerequisites

You need **Rust 1.85 or later**. Check with:

```bash
rustc --version
```

If rustc is missing or older, install or update via [rustup.rs](https://rustup.rs).

That is the only hard requirement. The `bwoc` binary is self-contained; no other runtime is needed for the tutorial.

---

## Step 1 — Install the CLI

Clone the framework and install the `bwoc` binary:

```bash
git clone https://github.com/bemindlabs/BWOC-Framework
cd BWOC-Framework
cargo install --path crates/bwoc-cli
```

**What happens.** Cargo compiles the CLI crate and places the `bwoc` binary in `~/.cargo/bin/`. This takes a minute or two the first time.

Verify it works:

```bash
bwoc --help
```

**Expected result.** The help text lists all subcommands. You should see `new`, `check`, `chat`, `run`, `team`, `status`, and others.

Then run the doctor to catch any environment issues early:

```bash
bwoc doctor --auto
```

**Expected result.** Each check prints `ok` or self-repairs. Fix anything that stays `warn` or `error` before continuing.

---

## Step 2 — Create a workspace

A workspace is the root directory that holds your agents, team config, and shared task lists.

```bash
bwoc init ./my-workspace
cd my-workspace
```

**What happens.** `bwoc init` creates `.bwoc/` (registry + teams), a default `agents/` folder, and a workspace manifest. The working directory is now your workspace root — all subsequent commands run from here.

Confirm it is registered:

```bash
bwoc workspace info
```

**Expected result.** A short summary: workspace name, path, registered agent count (zero so far).

Check the (empty) agent list:

```bash
bwoc list
```

**Expected result.** "No agents registered" or an empty table — correct, you have not created one yet.

---

## Step 3 — Create your first agent

```bash
bwoc new sage --role "research assistant" --target agents/agent-sage
```

**What happens.** `bwoc new` copies the agent template into `agents/agent-sage/`, writes a `config.manifest.json`, creates the slot directories (`persona/`, `mindsets/`, `skills/`, `interconnect/`, `memories/`), generates `AGENTS.md`, and adds the agent to `.bwoc/agents.toml`.

Optional flags you can add (not needed for the tutorial):

| Flag | What it does |
|---|---|
| `--backend ollama` | Pre-configure a local Ollama backend |
| `--mindsets research,critical` | Seed specific mindset files |
| `--skills writing,analysis` | Seed specific skill files |

**Expected result.** You see a creation summary and the path `agents/agent-sage/` now exists with its full slot structure.

---

## Step 4 — Check the agent passes validation

```bash
bwoc check agents/agent-sage
```

**What happens.** The auditor verifies that `AGENTS.md` contains no backend-specific syntax (no YAML frontmatter, no wikilinks, no hardcoded model names), that `config.manifest.json` is valid JSON, and that `MEMORY.md` is within the 200-line limit.

**Expected result.** All checks print `ok`. If anything fails, the error message tells you exactly what to fix. A freshly scaffolded agent always passes — if it does not, run `bwoc doctor --auto` and retry.

---

## Step 5 — Talk to the agent

Start an interactive chat session:

```bash
bwoc chat agent-sage
```

**What happens.** The harness loads the agent's manifest, resolves the backend (uses your default unless you set `--backend`), and opens a prompt.

Type something like:

```
What can you help me with?
```

Then type `/exit` or press `Ctrl-D` to leave the session.

**Expected result.** The agent responds based on its role ("research assistant") and any mindsets/skills declared in its slots.

If you want to pick a specific backend for this session without changing the manifest:

```bash
bwoc spawn agent-sage --backend claude
# or
bwoc spawn agent-sage --backend ollama
```

---

## Step 6 — Run a headless task

For automation you do not need an interactive session. Use `bwoc run`:

```bash
bwoc run agent-sage --task "Summarize what a research assistant does in three bullet points"
```

**What happens.** The harness sends the task prompt to the agent and streams the result to stdout, then exits.

Useful flags:

| Flag | What it does |
|---|---|
| `--json` | Emit structured JSON output instead of plain text |
| `--timeout 60` | Abort if the agent has not responded within 60 seconds |

**Expected result.** Three bullet points printed to your terminal, then the process exits cleanly.

---

## Step 7 — Form a team (optional but recommended)

Teams let multiple agents share a task list and claim work cooperatively. Even with one agent it is worth trying so you see the pattern.

Create a team and add `sage` to it:

```bash
bwoc team create research --members sage
```

Add a task to the team's shared list:

```bash
bwoc task add research "Draft a one-paragraph project brief"
```

Have `sage` claim the task:

```bash
bwoc task claim research --as agent-sage
```

`sage` works the task (run or chat as in Steps 5–6). When done, mark it complete:

```bash
bwoc task complete research --as agent-sage
```

**Expected result.** The task moves through `open → claimed → done` in the team's task list. You can verify at any point by listing open tasks (see the end-user handbook for the full task command surface).

---

## Step 8 — Observe what is happening

While agents are running you have three tools:

```bash
# One-line health status
bwoc status

# Follow the live log for a specific agent (Ctrl-C to stop)
bwoc log agent-sage -f

# Full dashboard — all agents, tasks, and resource use
bwoc dashboard
```

**What to look for.** `bwoc status` shows whether each agent is idle, running, or stopped. `bwoc log` streams structured events as the agent processes tasks. `bwoc dashboard` gives a holistic view useful when you have multiple agents in flight.

---

## Step 9 — Stop and retire the agent

Stop the running agent process without removing anything:

```bash
bwoc stop agent-sage
```

When you are ready to remove the agent entirely but want to preserve its memory for future reference:

```bash
bwoc retire agent-sage --keep-memory
```

**What happens.** `bwoc retire` removes the agent from `.bwoc/agents.toml`, cleans up its process state, and — because you passed `--keep-memory` — leaves the `memories/` slot in place so nothing learned is lost. Without that flag the whole directory is removed.

**Expected result.** `bwoc list` no longer shows `sage`. The `memories/` folder remains at `agents/agent-sage/memories/` if you used `--keep-memory`.

This is the **retire** phase — the third and final stage of the birth → live → retire arc.

---

## What next

You have seen the full lifecycle once. Here is where to go deeper:

| Topic | Read |
|---|---|
| Full CLI surface — every flag, every command | [../end-user/HANDBOOK.en.md](../end-user/HANDBOOK.en.md) |
| Incarnating agents, slot files, `AGENTS.md` rules | [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md) |
| Writing persona, mindsets, and skills | [../slots/HANDBOOK.en.md](../slots/HANDBOOK.en.md) |
| Backends — Claude, Codex, Kimi, Ollama, and more | [../backends/HANDBOOK.en.md](../backends/HANDBOOK.en.md) |
| The design philosophy and dhamma mapping | [../philosophy/HANDBOOK.en.md](../philosophy/HANDBOOK.en.md) |
| Term lookup | [../glossary.en.md](../glossary.en.md) |
| Framework source — canonical spec | [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) |

The framework's own docs live at `docs/en/` in that repo — for example, the incarnation spec is at [docs/en/INCARNATION.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/INCARNATION.en.md) and the agent template spec at [modules/agent-template/AGENTS.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md).
