# Agent Author & Operator Handbook

> Thai version: [HANDBOOK.th.md](HANDBOOK.th.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) · Glossary: [../glossary.en.md](../glossary.en.md) · End-user handbook: [../end-user/HANDBOOK.en.md](../end-user/HANDBOOK.en.md)

**Audience.** You are incarnating, configuring, tuning, or operating BWOC agents. You own the agent directory and are accountable for what that agent does. This handbook covers everything from the initial `bwoc new` invocation through daily operations, trust declarations, team membership, and eventual retirement.

**Source of truth.** The canonical specification lives in the framework repo. On any conflict, the repo wins and this handbook has a bug. Links: [agent-template AGENTS.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md) · [INCARNATION.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/INCARNATION.en.md) · [SIGNING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) · [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) · [FLEET-GOVERNANCE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md) · [NAMING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md) · [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md)

---

## Table of Contents

1. [What an Agent Is](#1-what-an-agent-is)
2. [Annotated Directory Layout](#2-annotated-directory-layout)
3. [The AGENTS.md Rule — Deep](#3-the-agentsmd-rule--deep)
4. [Slot Files — Obsidian Format](#4-slot-files--obsidian-format)
   - [4.1 persona/](#41-persona)
   - [4.2 mindsets/](#42-mindsets)
   - [4.3 skills/](#43-skills)
   - [4.4 interconnect/](#44-interconnect)
   - [4.5 memories/](#45-memories)
5. [The Manifest — config.manifest.json](#5-the-manifest--configmanifestjson)
6. [Incarnating with bwoc new](#6-incarnating-with-bwoc-new)
   - [6.1 Full Flag Reference](#61-full-flag-reference)
   - [6.2 TTY Prompting](#62-tty-prompting)
   - [6.3 Worked Example](#63-worked-example)
   - [6.4 Post-incarnation Checklist](#64-post-incarnation-checklist)
7. [bwoc check — Every Failure Mode](#7-bwoc-check--every-failure-mode)
8. [Trust and Signing](#8-trust-and-signing)
   - [8.1 The Seven Qualities (Kalyanamitta-7)](#81-the-seven-qualities-kalyanamitta-7)
   - [8.2 Evidence Rules](#82-evidence-rules)
   - [8.3 Keygen and Signing](#83-keygen-and-signing)
9. [Lifecycle — stop / start / retire](#9-lifecycle--stop--start--retire)
10. [Operating a Running Agent](#10-operating-a-running-agent)
11. [Teams (Sangha)](#11-teams-sangha)
12. [This Workspace's Authoring Conventions](#12-this-workspaces-authoring-conventions)
13. [Common Authoring Mistakes and Fixes](#13-common-authoring-mistakes-and-fixes)

---

## 1. What an Agent Is

In BWOC, **one folder = one agent**. Every agent is a self-contained directory cloned from the agent template. It holds the agent's identity, memory, decision-making principles, skills, and inter-agent coordination configuration — all as plain files. No database, no centralized state.

The framework is **backend-neutral**: the same agent directory runs on Claude Code, Codex, Kimi, Antigravity, or any OpenAI-compatible self-hosted endpoint via `bwoc-harness`. No vendor lock-in is baked into the agent itself; vendor selection is a runtime concern resolved in `config.manifest.json` and the workspace registry.

The agent's lifecycle maps to three phases the framework calls *uppada · thiti · vaya* — birth, live/work, and retirement. You don't need to use the Pali names; just think of them as: create, operate, retire. Every BWOC object follows this arc.

Agents are managed exclusively through the `bwoc` CLI. The registry lives at `.bwoc/agents.toml` in the workspace root. **Never hand-edit the registry or rename agent directories manually.** Use the CLI for all structural changes so the registry stays coherent.

---

## 2. Annotated Directory Layout

```
agents/agent-<name>/
│
├── AGENTS.md                   # Single source of truth for ALL LLM backends.
│                               # Plain Markdown only — no YAML frontmatter,
│                               # no wikilinks, no callouts. Every backend
│                               # reads this exact file.
│
├── CLAUDE.md -> AGENTS.md      # Symlink. Editing this edits AGENTS.md.
├── AGY.md    -> AGENTS.md      # Symlink for Antigravity backend.
├── CODEX.md  -> AGENTS.md      # Symlink for Codex/OpenAI backend.
├── KIMI.md   -> AGENTS.md      # Symlink for Kimi/Moonshot backend.
├── OLLAMA.md -> AGENTS.md      # Symlink for self-hosted Ollama.
├── OPENAI.md -> AGENTS.md      # Symlink for OpenAI-compatible endpoint.
│                               # Add more: ln -s AGENTS.md <BACKEND>.md
│
├── config.manifest.json        # Machine-readable identity + capabilities.
│                               # Must be valid JSON. bwoc check validates it.
│
├── MEMORY.md                   # Memory index. Hard cap: 200 lines.
│                               # One bullet per memory file in memories/.
│                               # bwoc check enforces the line cap.
│
├── README.md                   # Human-facing description of this agent.
│                               # Obsidian format (frontmatter + wikilinks OK).
│
├── conventions.md              # Naming, placeholder, and type-definition rules.
├── neutrality.md               # Why backend-neutrality matters; design notes.
├── task-log.jsonl              # Append-only JSONL task record.
│
├── persona/                    # SLOT: WHO the agent is.
│   └── README.md               # Scope, anti-scope, thematic identity.
│                               # Obsidian format. Thai body + EN frontmatter
│                               # in this workspace.
│
├── mindsets/                   # SLOT: HOW the agent thinks.
│   ├── SPEC.md                 # Slot spec (template ships this; keep as ref).
│   └── <name>.md               # One mindset per file. Obsidian format.
│                               # tags: type/mindset · principle/<pali-term>
│
├── skills/                     # SLOT: WHAT the agent can do.
│   ├── SPEC.md                 # Slot spec.
│   └── <name>.md               # One skill per file. Obsidian format.
│                               # tags: type/skill · domain/<area>
│                               # frontmatter field: maturity: L1..L7
│
├── interconnect/               # SLOT: inter-agent coordination config.
│   ├── capabilities.md         # Machine-readable skill registry for peers.
│   ├── messaging.md            # Envelope schema + Saraniya-dhamma 6 norms.
│   ├── trust.md                # Kalyanamitta-7 trust profile spec.
│   ├── routing.md              # Route declarations for cross-workspace peers.
│   └── sangha.md               # Team membership and task-flow conventions.
│
├── memories/                   # SLOT: WHAT the agent remembers.
│   ├── README.md               # Memory system spec.
│   └── <slug>.md               # One memory per file. YAML frontmatter required.
│
├── docs/                       # Extended documentation for this agent.
│   ├── en/                     # English docs (*.en.md).
│   └── th/                     # Thai parity docs (*.th.md).
│
├── scripts/                    # Agent-specific helper scripts.
│   ├── incarnate.sh            # Template-level clone helper (kept for reference).
│   └── check-agent-neutrality.sh   # Local neutrality check (pre-bwoc check).
│
├── .bwoc/                      # Runtime state (gitignored by bwoc init).
│   ├── agent.key               # Ed25519 private key (0600). Written by bwoc trust --keygen.
│   ├── inbox.jsonl             # Inbound message queue (JSONL).
│   └── agent.log               # Daemon stderr.
│
└── .claude/                    # Claude Code project config for this agent.
    ├── settings.json           # Hooks config (inbox-auto-reply Stop hook).
    └── hooks/
        └── inbox-auto-reply.sh # Auto-reply hook for agent-to-agent messaging.
```

Key principle: **everything under `.bwoc/` is runtime state** and is gitignored. Everything else is versioned configuration. The agent's identity, behavior, and memory live in plain Markdown — readable, diffable, and auditable without any tooling.

---

## 3. The AGENTS.md Rule — Deep

`AGENTS.md` is the single source of truth for every LLM backend that can run this agent. It is the file the backend actually reads when the agent starts. All backend entry-point files (`CLAUDE.md`, `AGY.md`, `CODEX.md`, `KIMI.md`, `OLLAMA.md`, `OPENAI.md`) are **symlinks** that resolve to `AGENTS.md`. Editing any of them edits `AGENTS.md`.

### What AGENTS.md must be

- **Plain Markdown only.** No YAML frontmatter block at the top. No `---` fence opening the file.
- **No wikilinks.** Wikilinks (`[[...]]`) are an Obsidian-only feature. Backend LLMs will render them as literal bracketed text or fail to parse cross-references. Use plain prose or plain Markdown links.
- **No Obsidian callouts.** The `> [!warning]`, `> [!note]`, `> [!abstract]` syntax is Obsidian-specific. Some backends silently ignore the `[!type]` prefix; others render it verbatim. Plain blockquotes or bold text achieve the same emphasis portably.
- **No hardcoded model IDs or vendor names in behavioral content.** Do not write "use claude-opus-4 for this step" or "Antigravity handles X differently." Backend-varying values belong in `config.manifest.json` and are referenced via `{{camelCase}}` placeholders.
- **Placeholders for all configurable values.** Use `{{agentId}}`, `{{agentRole}}`, `{{primaryModel}}`, `{{fallbackModel}}`, `{{scopeDescription}}`, `{{primaryCapability}}`, `{{lintCmd}}`, `{{formatCmd}}`, `{{testCmd}}`, `{{buildCmd}}`, `{{memoryPath}}`, `{{worktreeBase}}`, `{{deepMemoryCmd}}`. The `bwoc new` command substitutes these when copying the template; remaining unsubstituted `{{...}}` patterns are a `bwoc check` violation.

### What AGENTS.md contains

The canonical template ([spec](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md)) structures `AGENTS.md` into numbered sections:

| Section | Content |
|---|---|
| 0 — Backend Registration | Table of backend symlinks; instructions for adding a new backend |
| 1 — Identity (Samma-ditthi) | Who the agent is: agentId, role, primary capability, scope, anti-scope, thinking basis (the five most-applied principles), capability declaration protocol |
| 2 — Task Planning (Samma-sankappa) | Four Noble Truths task cycle; task-log.jsonl record format; scope discipline |
| 3 — Communication (Samma-vaca) | Inter-agent message protocol; error message structure; user-facing tone |
| 4 — Worktree Discipline (Samma-kammanta) | Isolation rules; branch naming; commit discipline; cleanup after merge |
| 5 — Trust & Neutrality (Samma-ajiva) | Backend-neutrality statement; verification script; threat model |
| 6 — Verification Gates (Samma-vayama) | The four gate commands (lint, format, test, build) and when each applies |
| 7 — Memory System (Samma-sati) | Tier 1 file-based memory lifecycle; memory file format; what to save vs discard; 200-line cap; Tier 2 deep backend |
| 8 — Focus & Stability (Samma-samadhi) | Session lifecycle; configuration reference; concurrent task cap; project switching; module boundaries; commit style |
| 9 — Baseline Security (Sila 5) | Five non-negotiable security precepts |
| 10 — Observability (Satipatthana 4) | Four foundations to monitor during a session |
| 11 — Self-Improvement (Panna 3) | Post-task reflection cycle |
| Appendices | Placeholder reference; quick-start checklist; document map |

The section names in parentheses are Pali terms from the Noble Eightfold Path — they are engineering thinking aids, not religious instruction. Each maps to a real concern: right-view means having accurate context; right-action means git hygiene; right-effort means running your verification gates. See the [glossary](../glossary.en.md) for every term's plain-English meaning.

### Symlink rule

When you receive an agent from someone else, verify the symlinks:

```bash
ls -la agents/agent-<name>/CLAUDE.md   # should show -> AGENTS.md
ls -la agents/agent-<name>/AGY.md      # same
```

If any backend file is a **regular file with its own content**, it has diverged from `AGENTS.md`. That is a neutrality violation. Delete it and re-create the symlink:

```bash
rm agents/agent-<name>/CLAUDE.md
ln -s AGENTS.md agents/agent-<name>/CLAUDE.md
```

To add a backend that the template didn't ship:

```bash
cd agents/agent-<name>
ln -s AGENTS.md OLLAMA.md
```

No code change, no manifest update needed — `bwoc spawn --backend ollama` will find `OLLAMA.md` via the symlink.

---

## 4. Slot Files — Obsidian Format

Slot files are the **opposite** of `AGENTS.md`: they use full Obsidian Markdown — YAML frontmatter, wikilinks, and callouts are not only allowed but expected. They are human-readable documentation living alongside the agent's instructions. They are **not** read directly by the LLM backend at runtime; they inform `bwoc check` evidence rules and provide context for operators.

### 4.1 persona/

`persona/README.md` defines who the agent is: its thematic identity, scope (what it does), and anti-scope (what it explicitly refuses). This file is the primary evidence source for several trust qualities (`piyo`, `vatta`, `noCatthana` — see Section 8).

Minimum required sections: **Scope** (non-empty, describes a concrete delegateable task) and **Anti-scope** (at least one explicit "will refuse" entry).

In this workspace, persona files are written in Thai body text with English frontmatter and English tags — see Section 12.

### 4.2 mindsets/

Each mindset is one `.md` file under `mindsets/`. A mindset is a named, reusable decision frame the agent applies in a specific class of situation. It answers: when do I use this? how do I apply it? when do I NOT apply it?

Required frontmatter:

```yaml
---
title: <Mindset Name>
aliases: []
tags:
  - type/mindset
  - principle/<pali-dhamma-term>
---
```

The `principle/<pali-dhamma-term>` tag is the anchor. Each mindset names exactly one Pali principle as its anchor. Pali terms used in tags are the plain romanized forms (no diacritics in tag keys): `appamada`, `yoniso-manasikara`, `anatta`, `samanatatta`, `mattanutata`, `sila`, and so on. See [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) for the full list of 22 frameworks.

Required sections in the body:

| Section | Purpose |
|---|---|
| `## When to Apply` | Specific situations that trigger this mindset |
| `## How to Apply` | Concrete steps or heuristics |
| `## When NOT to Apply` | Boundaries — over-applying any mindset is an anti-pattern |
| `## Related Principles` | Cross-links to other mindsets or skills |

One mindset per file. Do not bundle multiple decision frames into one file.

The `bhavaniyo` trust quality (see Section 8) requires at least one mindset whose name or content references improvement / verification / right-amount. Mindsets with tags like `principle/yoniso-manasikara` or `principle/mattanutata` satisfy this.

### 4.3 skills/

Each skill is one `.md` file under `skills/`. A skill is a concrete, bounded capability: "log monitoring", "anomaly detection", "schema migration". Skills are what the agent *executes*; mindsets are how it *thinks*.

Required frontmatter:

```yaml
---
title: <Skill Name>
aliases: []
tags:
  - type/skill
  - domain/<area>
maturity: L3
---
```

The `domain/<area>` tag categorizes the skill domain (examples: `domain/security-monitoring`, `domain/database`, `domain/infrastructure`, `domain/testing`). The `maturity` field is required and must be one of the seven levels below.

#### Maturity Levels (Ariya-dhana 7)

These seven levels measure capability maturity — declared honestly by the agent author, not auto-computed:

| Level | Plain meaning |
|---|---|
| L1 | First successful use; unverified; treat as experimental |
| L2 | Used multiple times; informal verification; patterns are emerging |
| L3 | Verification gates pass consistently; reliable in normal conditions |
| L4 | Resilient to common failure modes; handles edge cases gracefully |
| L5 | Mentorship capability — agent can guide another agent in this skill |
| L6 | Cross-domain transfer — skill has been applied beyond its original context |
| L7 | Canonical — other agents adopt this as a reference implementation |

Claiming a higher level than reality is an overclaim and a threat-model violation (T-1.4 in [THREAT-MODEL.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/THREAT-MODEL.en.md)). When in doubt, declare lower.

Required body sections:

| Section | Content |
|---|---|
| `## Domain` | What files, systems, or areas this skill operates on |
| `## Inputs` | What the skill needs to start |
| `## Outputs` | What the skill produces |
| `## Verification Gates` | How the agent knows it succeeded |
| `## Out of Scope` | What this skill does NOT do; which adjacent skill handles the difference |

The `garu` trust quality requires at least one skill or mindset stub to exist. The `Out of Scope` section is required — it is the mechanical expression of Mattanutata (right amount).

Cross-link every skill from `interconnect/capabilities.md` so peer agents can discover it for delegation.

### 4.4 interconnect/

`interconnect/` holds the inter-agent coordination configuration for this agent:

- `capabilities.md` — machine-readable skill registry. Peer agents read this to know what this agent can accept for delegation.
- `messaging.md` — envelope schema and Saraniya-dhamma 6 (the six conditions of conciliation) rendered as engineering norms for inter-agent speech.
- `trust.md` — Kalyanamitta-7 trust profile specification (this agent's declared qualities and required qualities from peers).
- `routing.md` — cross-workspace peer route declarations.
- `sangha.md` — team membership and task-flow norms.

These files use Obsidian format (frontmatter + wikilinks). They are documentation for human operators and for the framework's evidence checks — not direct LLM instructions.

### 4.5 memories/

`memories/` holds Tier 1 file-based memories — one `.md` per memory, each with YAML frontmatter:

```yaml
---
name: <descriptive-slug>
description: <one-line relevance hook>
type: user | feedback | project | reference
created: 2026-05-22
updated: 2026-05-22
---
```

Body format: content, then `**Why:**` (motivation), then `**How to apply:**` (when/where).

`MEMORY.md` in the agent root is the index — one bullet per memory file, hard-capped at 200 lines. `bwoc check` enforces the cap. When the index hits 200 lines, prune stale entries (anicca — impermanence; let go of what is no longer useful).

What to save: non-obvious decisions and their reasons; validated approaches the user confirmed; corrections; external resource locations.

What NOT to save: code patterns derivable from reading the code; git history; anything already in `AGENTS.md` or `conventions.md`; ephemeral session state.

---

## 5. The Manifest — config.manifest.json

`config.manifest.json` is the machine-readable identity card for the agent. It must be **valid JSON** — `bwoc check` parses it and fails hard on JSON syntax errors. It lives in the agent root.

### Full Field Table

| Field | Type | Required | Description | Example |
|---|---|---|---|---|
| `name` | string | yes | Agent name, kebab-case, no `agent-` prefix | `"erlang"` |
| `agentId` | string | yes | Full agent identifier; always `agent-<name>` | `"agent-erlang"` |
| `version` | string | yes | Manifest schema version | `"2.0"` |
| `agentRole` | string | yes | One-line role description; fills `{{agentRole}}` in AGENTS.md | `"security monitoring & detection sentinel"` |
| `primaryModel` | string | yes | Primary LLM model identifier, or `"auto"` to let the harness pick from `autoModels` at runtime | `"claude-opus-4-7"` |
| `fallbackModel` | string | no | Fallback model if primary is unavailable | `"claude-sonnet-4-6"` |
| `autoModels` | string[] | no | Candidate model pool in preference order; used when `primaryModel` is `"auto"`; for OpenAI-compatible endpoints put frontier models first | `["gpt-5.5", "gpt-4o"]` |
| `reasoningEffort` | string | no | Backend effort control hint; backend-neutral key, backend-specific value space (Claude: `low\|medium\|high\|xhigh\|max`; OpenAI-compatible: `low\|medium\|high`); backends without effort control ignore it | `"high"` |
| `baseUrl` | string | conditional | Required for `ollama` and `openai-compatible` backends; the harness passes this as `--endpoint`; `bwoc spawn` returns a clear error if absent for those backends | `"http://192.168.1.113:11434/v1"` |
| `memoryPath` | string | yes | File-based Tier 1 memory directory, relative to agent root | `"memories/"` |
| `sessionsPath` | string | no | Session data directory for Tier 2 memory mining | `"~/.claude/projects/"` |
| `deepMemoryCmd` | string | no | Tier 2 deep memory CLI command; invoked with `wake-up`, `search`, `mine` subcommands | `"mempalace"` |
| `lintCmd` | string | yes | Lint / static analysis command; the Samvara verification gate | `"cargo clippy -- -D warnings"` |
| `formatCmd` | string | yes | Format check command; the Pahana verification gate | `"cargo fmt --check"` |
| `testCmd` | string | yes | Test command; the Bhavana verification gate | `"cargo test"` |
| `buildCmd` | string | yes | Build command; the Anurakkhana verification gate | `"cargo build --release"` |
| `worktreeBase` | string | no | Base directory for git worktrees; default `/tmp` | `"/tmp"` |
| `scopeDescription` | string | yes (fills placeholder) | One-line "this agent does X"; fills `{{scopeDescription}}` | `"เฝ้าระวัง log/metric, ตรวจจับ anomaly, แจ้งเตือน intrusion"` |
| `outOfScope` | string | yes (fills placeholder) | One-line "this agent does NOT do Y"; fills anti-scope in AGENTS.md | `"แก้โค้ด, เขียนรายงาน audit, ลงมือ remediation"` |
| `primaryCapability` | string | no | Longer capability description; fills `{{primaryCapability}}`; defaults to `agentRole` value when absent | `"ตาที่สามมองทะลุทุก traffic"` |
| `trust` | object | no | Kalyanamitta-7 trust block; see below | see below |
| `trust.schemaVersion` | integer | yes (if trust present) | Schema version for forward-compat; currently `1` | `1` |
| `trust.declared` | object | yes (if trust present) | The seven boolean qualities this agent claims to satisfy; absent key treated as `false` | see Section 8 |
| `trust.declared.piyo` | boolean | no | Likeable / pleasant to delegate to | `false` |
| `trust.declared.garu` | boolean | no | Respected / has demonstrated competency | `false` |
| `trust.declared.bhavaniyo` | boolean | no | Admirable / helps others improve | `false` |
| `trust.declared.vatta` | boolean | no | Speaks beneficial truth; honest about limits | `false` |
| `trust.declared.vacanakkhamo` | boolean | no | Patient listener; accepts feedback | `false` |
| `trust.declared.gambhira` | boolean | no | Explains depth; anchored to canonical philosophy | `false` |
| `trust.declared.noCatthana` | boolean | no | Does not lead astray; explicit refusal entries in anti-scope | `false` |
| `trust.requiredTrust` | string[] | no | Qualities required from peers who want to send messages to this agent; template scaffold defaults to `["vatta", "noCatthana"]` | `["vatta", "noCatthana"]` |
| `trust.mode` | string | no | Refusal mode: `"off"` \| `"warn"` \| `"refuse"`; absent field uses v1 compat rule: empty requiredTrust → off, non-empty → refuse; `"warn"` is strictly opt-in | `"warn"` |
| `trust.signingPublicKey` | string | no | Ed25519 public key (hex, 64 chars); written by `bwoc trust --keygen`; used for envelope verification | `"12e4bac908f6..."` |

> The template's `config.manifest.json` ships with `"agentId": "agent-{{name}}"` and a `requiredConfig` schema block that documents all fields. In incarnated agents, `bwoc new` substitutes the `{{name}}` placeholder and removes the `requiredConfig` schema block — it is template metadata, not runtime config.

---

## 6. Incarnating with bwoc new

`bwoc new` performs "uppada" (birth) — it copies the agent template to a new directory, substitutes placeholders, registers the agent in `.bwoc/agents.toml`, and creates backend symlinks.

**Never hand-copy the template directory.** The CLI handles registry registration, placeholder substitution, symlink creation, and stub generation atomically. Hand-copying produces an unregistered agent that `bwoc check --all` won't find.

### 6.1 Full Flag Reference

```
bwoc new <NAME> [OPTIONS]
```

| Flag | Type | Required | TTY-prompted? | Description |
|---|---|---|---|---|
| `<NAME>` | positional | yes | no | Agent name in kebab-case (e.g. `database-schema`). The directory will be `agent-<NAME>/`. |
| `--target <TARGET>` | path | no | no | Destination directory. Default: `../agent-<name>/` relative to the template. In a standard workspace, always pass `--target agents/agent-<name>`. |
| `--template <TEMPLATE>` | path | no | no | Path to the template directory. Default: auto-detect `modules/agent-template/` from cwd ancestors. The auto-detect walks up from cwd until it finds a directory containing `modules/agent-template/`. |
| `--backend <BACKEND>` | enum | no | no | Primary backend for the workspace registry. Default: `claude`. Values: `claude` \| `antigravity` \| `codex` \| `kimi` \| `ollama` \| `openai-compatible`. Affects which CLI `bwoc spawn` invokes, not the agent's content. |
| `--role <ROLE>` | string | no | **yes** (if missing on TTY) | One-line role description. Written to `agentRole` in the manifest. Fills `{{agentRole}}` in AGENTS.md. |
| `--primary-model <PRIMARY_MODEL>` | string | no | **yes** (if missing on TTY) | Primary LLM model identifier. Written to `primaryModel` in the manifest. |
| `--fallback-model <FALLBACK_MODEL>` | string | no | no | Fallback LLM model identifier. Truly optional. |
| `--memory-path <MEMORY_PATH>` | path | no | no | File-based memory directory. Default: `memories/`. |
| `--sessions-path <SESSIONS_PATH>` | path | no | no | Session data directory for Tier 2 mining. Truly optional. |
| `--deep-memory-cmd <DEEP_MEMORY_CMD>` | string | no | no | Tier 2 memory CLI command. Truly optional. |
| `--lint-cmd <LINT_CMD>` | string | no | **yes** (if missing on TTY) | Lint command for the Samvara verification gate. |
| `--format-cmd <FORMAT_CMD>` | string | no | **yes** (if missing on TTY) | Format command for the Pahana verification gate. |
| `--test-cmd <TEST_CMD>` | string | no | **yes** (if missing on TTY) | Test command for the Bhavana verification gate. |
| `--build-cmd <BUILD_CMD>` | string | no | **yes** (if missing on TTY) | Build command for the Anurakkhana gate. |
| `--worktree-base <WORKTREE_BASE>` | path | no | no | Base directory for git worktrees. Default: `/tmp`. Truly optional. |
| `--scope <SCOPE>` | string | no | no | One-line "this agent does X". Fills `{{scopeDescription}}` and `scopeDescription` in manifest. |
| `--out-of-scope <OUT_OF_SCOPE>` | string | no | no | One-line "this agent does NOT do Y". Fills anti-scope in AGENTS.md and `outOfScope` in manifest. |
| `--primary-capability <PRIMARY_CAPABILITY>` | string | no | no | Longer capability description. Fills `{{primaryCapability}}`. Defaults to the `--role` value when omitted. |
| `--mindsets <MINDSETS>` | string | no | no | Comma-separated kebab-case mindset names to seed (e.g. `verify-before-act,right-amount`). Creates one stub `.md` per name under `mindsets/`. |
| `--skills <SKILLS>` | string | no | no | Comma-separated kebab-case skill names to seed. Creates one stub `.md` per name under `skills/`. |
| `--json` | flag | no | no | Emit JSON `{ agent_id, target, registered_in, symlinks, mindset_stubs, skill_stubs, persona_filled }` instead of human-readable report. Useful for scripted multi-agent setup. |
| `--lang <LANG>` | string | no | no | CLI output language. Precedence: `--lang` > `BWOC_LANG` env > `$LANG` > `en` fallback. |
| `-h, --help` | flag | no | no | Print help. |

### 6.2 TTY Prompting

When running on an interactive TTY and certain flags are omitted, `bwoc new` prompts for them:

- `--role` — prompted with "Agent role (one line):"
- `--primary-model` — prompted with "Primary model:"
- `--lint-cmd`, `--format-cmd`, `--test-cmd`, `--build-cmd` — prompted individually

In a non-TTY context (CI, scripts, piped input), **all four gate commands and `--role` and `--primary-model` must be provided explicitly**, or the command will fail with a non-TTY error. Use `--json` in scripted contexts.

### 6.3 Worked Example

Creating a security-monitoring agent in this workspace:

```bash
# From the workspace root. Template auto-detected from cwd.
bwoc new sentinel \
  --target agents/agent-sentinel \
  --backend claude \
  --role "security monitoring & anomaly detection" \
  --primary-model "claude-opus-4-7" \
  --fallback-model "claude-sonnet-4-6" \
  --scope "monitor logs and metrics, detect anomalies, alert on intrusions" \
  --out-of-scope "code remediation, audit reporting, incident response" \
  --primary-capability "sees anomalies before anyone else and routes alerts to the team" \
  --mindsets "verify-before-act,signal-over-noise,all-seeing-third-eye" \
  --skills "log-monitoring,anomaly-detection,intrusion-alerting" \
  --lint-cmd "echo 'gate: static analysis'" \
  --format-cmd "echo 'gate: format check'" \
  --test-cmd "echo 'gate: detection rules test'" \
  --build-cmd "echo 'gate: ruleset build'"
```

After this completes:

1. `agents/agent-sentinel/` exists with the full template layout
2. Backend symlinks are created: `CLAUDE.md AGY.md CODEX.md KIMI.md → AGENTS.md`
3. Three mindset stubs and three skill stubs are created
4. `config.manifest.json` has all placeholders substituted
5. The agent is registered in `.bwoc/agents.toml`

Then run the audit:

```bash
bwoc check agents/agent-sentinel
```

If it passes, the agent is ready. If it fails, see Section 7.

### 6.4 Post-incarnation Checklist

```
[ ] bwoc check agents/agent-<name>  passes with zero violations
[ ] Fill persona/README.md — Scope and Anti-scope sections are non-empty
[ ] Flesh out each mindset stub:
      - frontmatter: tags include principle/<pali-term>
      - body: When to Apply / How to Apply / When NOT to Apply / Related Principles
[ ] Flesh out each skill stub:
      - frontmatter: tags include domain/<area>, maturity: L<n>
      - body: Domain / Inputs / Outputs / Verification Gates / Out of Scope
[ ] Cross-link each skill in interconnect/capabilities.md
[ ] Add first entry to task-log.jsonl
[ ] Generate signing keypair: bwoc trust --keygen agents/agent-<name>
[ ] Verify trust profile: bwoc trust agents/agent-<name>
[ ] In this workspace: write slot bodies in Thai, keep frontmatter in English, include emoji in headings
[ ] bwoc check again — must still pass
```

---

## 7. bwoc check — Every Failure Mode

`bwoc check` is a **read-only audit**. It does not modify any files. Run it after every edit to any agent file.

```bash
bwoc check agents/agent-<name>          # audit one agent
bwoc check --all                        # audit every registered agent in workspace
bwoc check agents/agent-<name> --json   # machine-readable output
bwoc check --all --workspace /path      # fleet audit at a specific workspace root
```

`bwoc check` audits three things:

1. **Backend neutrality** of `AGENTS.md`
2. **JSON validity** of `config.manifest.json`
3. **Line count** of `MEMORY.md` (must be ≤ 200)

### Failure Modes

| Failure | Cause | Fix |
|---|---|---|
| `AGENTS.md has YAML frontmatter` | The file starts with `---` ... `---` | Remove the frontmatter block entirely from AGENTS.md |
| `AGENTS.md contains wikilinks` | Found `[[...]]` pattern | Replace all `[[Target\|Display]]` with plain text or plain Markdown links `[Display](path)` |
| `AGENTS.md contains Obsidian callouts` | Found `> [!type]` pattern | Replace with plain blockquote `>` or bold text |
| `AGENTS.md contains unsubstituted placeholder` | Found `{{something}}` that bwoc new should have replaced | Edit AGENTS.md: substitute the placeholder with the real value, or re-run bwoc new with the missing flag |
| `AGENTS.md references hardcoded model ID` | A line contains a vendor-specific model name (e.g., `claude-opus-4`) in behavioral content | Replace with `{{primaryModel}}` placeholder; actual value lives in config.manifest.json |
| `AGENTS.md references hardcoded vendor name in behavioral content` | "Claude will..." / "Antigravity supports..." outside the Backend Registration section | Rewrite to backend-neutral phrasing |
| `Backend file is not a symlink` | `CLAUDE.md` (or AGY.md etc.) is a regular file with its own content | `rm CLAUDE.md && ln -s AGENTS.md CLAUDE.md` |
| `Backend symlink does not point to AGENTS.md` | Symlink exists but resolves to a different target | `rm AGY.md && ln -s AGENTS.md AGY.md` |
| `config.manifest.json is not valid JSON` | Syntax error: trailing comma, missing quote, unescaped character, comment | Open in any JSON validator; fix the syntax error |
| `config.manifest.json missing required field` | A required field from the schema (name, agentId, agentRole, primaryModel, lintCmd, formatCmd, testCmd, buildCmd, memoryPath) is absent | Add the missing field |
| `config.manifest.json has unsubstituted placeholder` | Field value still contains `{{...}}` | Replace with the real value |
| `MEMORY.md exceeds 200 lines` | The memory index has grown beyond the cap | Prune: remove bullets for memories that are stale, redundant, or derivable from reading the code. Delete the corresponding files from memories/ |
| `trust.declared.piyo is true but evidence missing` | `piyo` declared true but `persona/README.md` is empty or Scope section is missing | Fill `persona/README.md` Scope section with a concrete delegateable task description |
| `trust.declared.garu is true but evidence missing` | `garu` declared true but neither `skills/` nor `mindsets/` contains any file beyond the SPEC stub | Create at least one real skill or mindset file |
| `trust.declared.bhavaniyo is true but evidence missing` | `bhavaniyo` declared true but no mindset references improvement/verification/right-amount | Add a mindset with tag `principle/yoniso-manasikara` or `principle/mattanutata` |
| `trust.declared.vatta is true but evidence missing` | `vatta` declared true but `outOfScope` in manifest is empty | Fill `outOfScope` field and the Anti-scope section of `persona/README.md` |
| `trust.declared.vacanakkhamo is true but evidence missing` | `vacanakkhamo` declared true but `.bwoc/inbox.jsonl` is absent or empty and `interconnect/feedback.md` does not document feedback handling | Exercise the inbox at least once (send a test message: `bwoc send <agent> "test"`) or create `interconnect/feedback.md` documenting the feedback protocol |
| `trust.declared.gambhira is true but evidence missing` | `gambhira` declared true but no doc file under the agent root is both ≥ 50 lines AND contains a `[[PHILOSOPHY.en.md]]` or `[[PHILOSOPHY.th.md]]` wikilink | Create a doc file that meets both criteria (length + wikilink back to the philosophy graph) |
| `trust.declared.noCatthana is true but evidence missing` | `noCatthana` declared true but `persona/README.md` Anti-scope section is absent or has no explicit "will refuse" entry | Add at least one explicit refusal to the Anti-scope section |

### Reading the JSON output

```bash
bwoc check agents/agent-<name> --json
```

Output shape:

```json
{
  "agent": "agent-<name>",
  "path": "agents/agent-<name>",
  "pass": false,
  "violations": [
    {
      "rule": "neutrality.no_wikilinks",
      "severity": "error",
      "detail": "AGENTS.md line 47: wikilink [[trust.md|Trust Model]] found",
      "fix": "Replace with plain Markdown link or plain text"
    }
  ],
  "warnings": []
}
```

Exit codes: `0` = pass, `1` = violations found, `2` = invocation error (bad path, unreadable file).

---

## 8. Trust and Signing

BWOC's inter-agent trust model is based on **Kalyanamitta-7** ("seven qualities of a good friend") from the Mitta Sutta (AN 7.36 of the Anguttara Nikaya — [SuttaCentral AN 7.36](https://suttacentral.net/an7.36)). Each agent declares seven booleans describing its own trustworthiness. Peer agents read those booleans and can refuse messages from senders who lack required qualities.

See the full spec: [SIGNING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) and `interconnect/trust.md` in the agent template.

### 8.1 The Seven Qualities (Kalyanamitta-7)

| Pali term | Literal meaning | In BWOC | Manifest key |
|---|---|---|---|
| Piyo | Likeable / endearing | Pleasant to delegate to; scope is clear and concrete | `piyo` |
| Garu | Respected / weighty | Has demonstrated competency; at least one skill or mindset | `garu` |
| Bhavaniyo | Admirable / cultivating | Helps others improve; has an improvement-oriented mindset | `bhavaniyo` |
| Vatta | One who speaks | Speaks beneficial truth; honest about what it does NOT do | `vatta` |
| Vacanakkhamo | Patient listener | Can take feedback; inbox flow exercised | `vacanakkhamo` |
| Gambhira | Speaker of profound things | Explains depth; doc anchored to canonical philosophy | `gambhira` |
| No catthana | Does not urge to unworthy ground | Does not lead astray; explicit refusal entries in anti-scope | `noCatthana` |

Manifest template scaffold defaults `requiredTrust` to `["vatta", "noCatthana"]` — "speaks beneficial truth" and "does not lead astray." These are the minimum civic floor that every agent should reasonably demand from peers.

The `trust.mode` field controls what happens when a peer is missing a required quality:

| Mode | Effective when | Daemon behavior |
|---|---|---|
| `off` | `mode: "off"` explicitly, or mode absent and `requiredTrust` empty | Envelope passes; no log entry |
| `warn` | `mode: "warn"` explicitly (opt-in only) | Envelope passes AND daemon emits `trust_warn` log line; use `bwoc log -f` to observe |
| `refuse` | `mode: "refuse"` explicitly, or mode absent and `requiredTrust` non-empty (v1 default) | Envelope marked refused in `inbox.refusals.jsonl`; never deleted |

User-originated messages (`from: "user"`) always pass regardless of mode. Trust gates govern agent-to-agent messaging only.

### 8.2 Evidence Rules

Declaring a quality `true` without the corresponding evidence is a `bwoc check` violation. The framework does not auto-measure trustworthiness — it catches obvious structural lies:

| Quality | Required evidence |
|---|---|
| `piyo` | `persona/README.md` Scope section is non-empty and describes a concrete delegateable task |
| `garu` | At least one real skill or mindset file exists under `skills/` or `mindsets/` (not just the SPEC stub) |
| `bhavaniyo` | A mindset file exists whose name or body content references improvement, verification, or right-amount (e.g., tag `principle/yoniso-manasikara` or `principle/mattanutata`) |
| `vatta` | `outOfScope` in manifest is non-empty; `persona/README.md` Anti-scope section is non-empty |
| `vacanakkhamo` | `.bwoc/inbox.jsonl` exists and is non-empty, OR `interconnect/feedback.md` documents feedback handling |
| `gambhira` | At least one doc file under the agent root is ≥ 50 lines AND contains `[[PHILOSOPHY.en.md]]` or `[[PHILOSOPHY.th.md]]` wikilink |
| `noCatthana` | `persona/README.md` Anti-scope section exists AND includes at least one explicit "will refuse" entry |

Declare `false` for any quality whose evidence you cannot honestly provide. `false` is always valid — no evidence check needed.

### 8.3 Keygen and Signing

Ed25519 signing keypairs are used for envelope signing (Phase 3 / HV2-4). The private key never leaves the agent's `.bwoc/` directory.

```bash
# Generate a keypair for one agent
bwoc trust --keygen agents/agent-<name>

# Generate for every registered agent (backfill)
bwoc trust --keygen --all

# Rotate the keypair (overwrites existing key -- irreversible for old signed envelopes)
bwoc trust --keygen agents/agent-<name> --force

# Read the trust profile (human table)
bwoc trust agents/agent-<name>

# Read the trust profile (JSON)
bwoc trust agents/agent-<name> --json
```

`bwoc trust --keygen` writes:

- **Private key** → `agents/agent-<name>/.bwoc/agent.key` with permissions `0600`. This file is gitignored. Do not commit it, copy it, or share it.
- **Public key** → `config.manifest.json` field `trust.signingPublicKey` (64-char hex string).

The `--force` flag overwrites an existing keypair. This rotates the agent's cryptographic identity. Any signed envelopes from before the rotation cannot be verified against the new key. Use with deliberate intent.

`bwoc trust --keygen --all` accepts `--force` to rotate all agents.

All `bwoc trust` flags:

| Flag | Description |
|---|---|
| `[AGENT]` | Agent name ("agent-foo" or "foo"). Optional only with `--keygen --all`. |
| `--keygen` | Generate an ed25519 keypair instead of reading the profile |
| `--all` | With `--keygen`: generate for every registered agent |
| `--force` | With `--keygen`: overwrite existing key |
| `--json` | Emit JSON instead of human-readable table |
| `--workspace <WORKSPACE>` | Workspace root override |
| `--lang <LANG>` | CLI output language |

---

## 9. Lifecycle — stop / start / retire

### bwoc stop

Pauses an agent: sets its registry status to `"stopped"` without removing files. The daemon (if running) receives a STOP signal.

```bash
bwoc stop sentinel                     # interactive confirmation
bwoc stop sentinel --yes               # skip confirmation (required for CI)
bwoc stop --all --yes                  # stop every non-stopped agent
bwoc stop sentinel --yes --json        # JSON output (requires --yes)
```

| Flag | Description |
|---|---|
| `<NAME>` | Agent name ("agent-foo" or "foo"). Mutually exclusive with `--all`. |
| `--all` | Stop every non-stopped agent in the workspace |
| `--yes` | Skip interactive confirmation; required for non-TTY |
| `--json` | Emit JSON `{ workspace, agent, daemon_outcome, registry_updated }`; requires `--yes`; single-agent only |
| `--workspace <WORKSPACE>` | Workspace root override |
| `--lang <LANG>` | CLI output language |

A stopped agent retains all its files, memory, and task log. It can be restarted at any time.

### bwoc start

Reactivates a stopped agent: sets status to `"active"` and optionally spawns the daemon.

```bash
bwoc start sentinel                    # interactive confirmation
bwoc start sentinel --yes              # skip confirmation
bwoc start sentinel --yes --no-daemon  # flip status only, do not spawn daemon
bwoc start --all --yes                 # start every stopped agent
```

| Flag | Description |
|---|---|
| `<NAME>` | Agent name. Mutually exclusive with `--all`. |
| `--all` | Start every stopped agent in the workspace |
| `--yes` | Skip confirmation; required for non-TTY |
| `--no-daemon` | Only flip registry status; do not spawn `bwoc-agent --serve` |
| `--json` | Emit JSON `{ workspace, agent, daemon_spawned, daemon_pid, ... }`; requires `--yes`; single-agent only |
| `--workspace <WORKSPACE>` | Workspace root override |
| `--lang <LANG>` | CLI output language |

### bwoc retire

Retirement is **vaya** (the third phase — cessation). It removes the agent from the workspace registry. By default it also removes the agent directory from disk.

```bash
bwoc retire sentinel                           # interactive confirmation; removes registry entry + dir
bwoc retire sentinel --yes                     # skip confirmation
bwoc retire sentinel --keep-files --yes        # remove registry entry only; keep directory
bwoc retire sentinel --keep-memory --yes       # keep memories/ only; remove rest of dir
bwoc retire sentinel --yes --json              # JSON output (requires --yes)
```

| Flag | Description |
|---|---|
| `<NAME>` | Agent name. Matches by id ("agent-foo") or bare name ("foo"). |
| `--yes` | Skip interactive confirmation; required for non-TTY and for `--json` |
| `--keep-files` | Remove registry entry only; preserve the full agent directory on disk. Mutually exclusive with `--keep-memory`. |
| `--keep-memory` | Preserve `memories/` only; remove the rest of the agent directory. Useful when retiring an agent but keeping its accumulated knowledge. Mutually exclusive with `--keep-files`. |
| `--json` | Emit JSON `{ workspace, agent, path, mode, registry_updated }`; requires `--yes` |
| `--workspace <WORKSPACE>` | Workspace root override |
| `--lang <LANG>` | CLI output language |

Before retiring an agent that is a member of a team, edit `.bwoc/teams/<team>.toml` to remove it from the member list. Retiring an agent does not automatically update team membership files.

---

## 10. Operating a Running Agent

### bwoc status

Read-only health and identity snapshot.

```bash
bwoc status sentinel                  # full detail block for one agent
bwoc status                           # summary table for all agents
bwoc status --all                     # full detail for every agent
bwoc status sentinel --banner         # replay startup liveness banner (no daemon required)
bwoc status sentinel --json           # JSON output
```

### bwoc spawn / chat / run

```bash
# Exec the backend CLI in the agent's directory (uppada → thiti transition)
bwoc spawn --path agents/agent-sentinel
bwoc spawn --path agents/agent-sentinel --backend ollama

# Chat with a named agent (resolves from workspace registry)
bwoc chat sentinel
bwoc chat sentinel --tmux              # open in a new tmux window
bwoc chat sentinel --ghostty           # open in a Ghostty terminal (macOS)
bwoc chat sentinel --tui               # full-screen ratatui chat (harness backends only)

# Run a single task non-interactively
bwoc run sentinel --task "check inbox and summarize pending alerts"
bwoc run sentinel --task "..." --json --timeout 120
```

### bwoc supervise

Runs a daemon supervisor that restarts the agent on crash, and exits cleanly when the agent is stopped.

```bash
bwoc supervise sentinel
bwoc supervise sentinel --max-restarts-per-min 5
bwoc supervise sentinel --json         # emit one JSON event per action
```

The `--max-restarts-per-min` default is 10. Beyond that threshold the supervisor gives up rather than burning CPU in a crash loop.

### bwoc send / inbox

Inter-agent and user-to-agent messaging:

```bash
# Send a message (from user, default)
bwoc send sentinel "please check for anomalies in the last 15 minutes"

# Send from another agent
bwoc send sentinel "anomaly detected in auth log" --from agent-luban

# Reply to a prior message
bwoc send sentinel "confirmed, escalating" --reply-to msg-20260606T120000Z-a3f2b

# Skip tmux wakeup (CI / daemon contexts)
bwoc send sentinel "task done" --from agent-luban --no-wakeup

# Read inbox
bwoc inbox sentinel
bwoc inbox sentinel --json --limit 10
bwoc inbox sentinel --watch             # tail mode
bwoc inbox sentinel --clear --yes       # acknowledge all messages
bwoc inbox --all                        # all agents' inboxes
```

### bwoc log / ping

```bash
bwoc log sentinel                       # last 50 lines of daemon log
bwoc log sentinel -f                    # follow (stream)
bwoc log sentinel -n 100 --clear        # last 100 lines, then truncate

bwoc ping sentinel                      # PING → PONG over Unix socket
bwoc ping --all                         # ping every agent
```

---

## 11. Teams (Sangha)

A **Sangha** (the Pali word for a community of practitioners, here adapted as "a team of agents") is a named subset of agents sharing a task list. Teams enable multi-agent coordination: shared tasks, plan-approval workflows, and cross-agent accountability.

### Creating and Managing Teams

```bash
# Create a team with initial members
bwoc team create monitoring --members sentinel,luban,yanluo

# List teams with member + task counts
bwoc team list

# Retire a team (removes membership file + task list)
bwoc team retire monitoring
```

**To add or remove members after creation:** edit `.bwoc/teams/<team>.toml` directly. There is no `bwoc team member-add` subcommand. The TOML file is the membership record; the CLI provides no wrapper for editing it.

```toml
# .bwoc/teams/monitoring.toml
[team]
id = "monitoring"
members = ["agent-sentinel", "agent-luban", "agent-yanluo"]
```

### Shared Task Workflow

```bash
# Add a task to a team's list
bwoc task add monitoring "investigate auth log spike between 02:00 and 03:00 UTC"

# Add with dependencies (task is blocked until deps complete)
bwoc task add monitoring "generate incident report" --deps TASK-001,TASK-002

# List tasks with state + claimant
bwoc task list monitoring

# Claim a pending task as an agent
bwoc task claim monitoring TASK-001 --as sentinel

# Submit a plan for review (Pavarana — the Pali term for "invitation to be corrected")
bwoc task plan monitoring TASK-001 "Plan: 1) fetch auth logs 2) correlate IPs 3) score anomaly"

# Team lead approves the plan — claimant may now complete
bwoc task approve monitoring TASK-001

# Team lead rejects — claimant must revise and resubmit
bwoc task reject monitoring TASK-001 "need to include VPN log correlation"

# Complete an in-progress task
bwoc task complete monitoring TASK-001 --as sentinel
```

Task states: `pending` → `in_progress` (after claim) → `plan_submitted` (after plan) → `approved` (after approve) → `completed`. Rejected tasks return to `in_progress` for revision.

---

## 12. This Workspace's Authoring Conventions

Agents in this workspace are **themed as Chinese deities (เทพจีน)**. This section records the conventions that apply to all agents incarnated here.

### Language

- **Slot bodies** (`persona/README.md`, `mindsets/*.md`, `skills/*.md`, `interconnect/*.md`, `memories/*.md`): write in **Thai**. The content — descriptions, instructions, "When to Apply" sections — is in Thai.
- **Frontmatter and tags**: always in **English**. YAML keys, tag values, field names, maturity levels, domain labels.
- **`AGENTS.md`**: backend-neutral; its behavioral content uses Thai for scope/role descriptions (as shown in `agent-erlang`), which is acceptable because those are role-specific strings resolved from the manifest. The structural sections of `AGENTS.md` remain in English.
- **Headings in slot files**: include a relevant **emoji** that reflects the agent's deity theme or skill domain.

### Themes

Current theme: Chinese deities. Agent names can reference the deity (e.g., `erlang` for Erlang Shen, the third-eye deity of surveillance) or a function (e.g., `sentinel`). The persona file gives the agent its mythological context.

### Example slot heading (mindsets)

```markdown
# 👁️ All Seeing Third Eye — เปิดตาที่สาม
```

### Validation

After every edit to any slot file or `AGENTS.md`:

```bash
bwoc check agents/agent-<name>
```

All passes must be clean before committing.

---

## 13. Common Authoring Mistakes and Fixes

### Mistake: YAML frontmatter in AGENTS.md

**Symptom:** `bwoc check` reports `AGENTS.md has YAML frontmatter`.

**Cause:** The file was created from a documentation template or edited with Obsidian, which added a frontmatter block.

**Fix:** Remove the `---` ... `---` block at the top of `AGENTS.md`. The file must start with a plain Markdown heading or paragraph.

---

### Mistake: Wikilinks in AGENTS.md

**Symptom:** `bwoc check` reports `AGENTS.md contains wikilinks`.

**Cause:** Copy-paste from a slot file or conventions doc that uses `[[...]]` syntax.

**Fix:** Replace every `[[Target|Display]]` with either plain text or a standard Markdown link. If the wikilink is a cross-reference to another agent file, the cross-reference does not belong in `AGENTS.md` — put it in a slot doc or `interconnect/` file instead.

---

### Mistake: Hardcoded model IDs in AGENTS.md

**Symptom:** `bwoc check` reports neutrality violation for hardcoded model name.

**Cause:** Wrote a model name like `claude-sonnet-4-6` directly in `AGENTS.md` outside the Backend Registration section.

**Fix:** Replace with `{{primaryModel}}` or `{{fallbackModel}}`. The actual model ID belongs in `config.manifest.json`. Note: model IDs in the Backend Registration section (Section 0) as a documentation table of "what backends exist" are acceptable; model IDs in behavioral content (e.g., "run this task on {{primaryModel}}") must use placeholders.

---

### Mistake: Backend file is a regular file, not a symlink

**Symptom:** `bwoc check` reports `Backend file is not a symlink` for `CLAUDE.md` or another backend file.

**Cause:** Someone edited `CLAUDE.md` directly or created it with an editor rather than `ln -s`.

**Fix:**
```bash
cd agents/agent-<name>
rm CLAUDE.md
ln -s AGENTS.md CLAUDE.md
```

---

### Mistake: config.manifest.json has a trailing comma

**Symptom:** `bwoc check` reports `config.manifest.json is not valid JSON`.

**Cause:** JSON does not allow trailing commas after the last item in an object or array.

**Fix:** Remove the trailing comma. Use a JSON validator or `python3 -m json.tool config.manifest.json` to find the line.

---

### Mistake: MEMORY.md exceeds 200 lines

**Symptom:** `bwoc check` reports `MEMORY.md exceeds 200 lines`.

**Cause:** Too many memory entries have accumulated without pruning.

**Fix:** Review `memories/` for stale, redundant, or derivable entries. Delete their files and remove their bullets from `MEMORY.md`. This is the practice of anicca (impermanence) — let go of what is no longer useful.

---

### Mistake: Trust quality declared true without evidence

**Symptom:** `bwoc check` reports a trust evidence violation (e.g., `trust.declared.gambhira is true but evidence missing`).

**Cause:** The manifest declares the quality but the structural evidence (see Section 8.2) is absent.

**Fix:** Either provide the evidence (create the required file, fill the required section) or set the quality to `false` in `config.manifest.json`. Do not claim qualities you cannot support.

---

### Mistake: Mindset file missing the "When NOT to Apply" section

**Symptom:** The mindset is technically valid (passes `bwoc check`) but causes over-application at runtime.

**Cause:** Every mindset has a bounded domain. Without an explicit "When NOT to Apply" section, the agent may apply it indiscriminately.

**Fix:** Add the section. Name at least one class of situation where this mindset is counterproductive. This is the mechanical expression of mattanutata (right amount) — every frame has limits.

---

### Mistake: Hand-editing `.bwoc/agents.toml`

**Symptom:** Registry becomes inconsistent; `bwoc list` shows phantom entries or misses live agents; `bwoc check --all` produces confusing errors.

**Cause:** The registry was edited directly instead of through CLI commands.

**Fix:** Use `bwoc retire --keep-files` to cleanly remove the bad entry and re-register via `bwoc new` with `--target` pointing to the existing directory. If the directory is already fully configured, `bwoc new` with `--target` pointing to an existing agent directory will re-register it without overwriting files (verify this behavior against the current CLI version).

---

### Mistake: Running bwoc new without --target in a multi-agent workspace

**Symptom:** The new agent directory appears at the wrong path (relative to the template, not the workspace root).

**Cause:** `--target` defaults to `../agent-<name>/` relative to the template directory, which may place the agent inside `projects/bwoc-framework/modules/` rather than `agents/`.

**Fix:** Always pass `--target agents/agent-<name>` when running from the workspace root. The template auto-detect is reliable; only `--target` needs to be explicit.

---

*This handbook is maintained alongside the framework. When behavior described here diverges from the [framework repo](https://github.com/bemindlabs/BWOC-Framework), the repo is correct — please open a fix.*
