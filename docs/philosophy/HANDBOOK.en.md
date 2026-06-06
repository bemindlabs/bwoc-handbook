# Why BWOC — Philosophy, Design, and Reasoning

> Language toggle: **English (canonical)** | [ภาษาไทย](HANDBOOK.th.md)
>
> This chapter is for a thoughtful engineer or architect deciding whether BWOC is the right foundation for building AI coding agents. It explains what BWOC is, why it is designed the way it is, and where the Buddhist-derived vocabulary comes from — and why it earns its place.

---

## A note on Pali terms

BWOC uses Pali-language labels — terms from early Buddhist canonical texts — as **section names and vocabulary shortcuts for engineering concepts**. They are thinking aids, not religious instruction. Every Pali term in BWOC has a plain-language engineering meaning that is stated first; the Pali label is parenthetical. No religious literacy is required to use, read, or contribute to BWOC. A reader who has never encountered Buddhism can read this chapter and ship code.

When you see a term like "Yoniso Manasikāra" in a doc or commit message, read it as shorthand for "verify against current state before acting on remembered claims." The Pali is just the label; the engineering constraint is what matters in code and reviews.

For every term in one table, see the [BWOC Glossary](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md). The conceptual core — the full 22-framework mapping — lives in [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md).

---

## The problem: building many agents gets messy fast

The first agent you build is tractable. You give it a name, set a system prompt, maybe wire up a tool or two. The second agent works the same way. By the fifth, you notice the cracks:

- You have three agents that accumulate memory in incompatible formats, so reviewing any one of them requires remembering how *that particular one* stores its state.
- One agent is tightly coupled to Claude's response style; if you swap the backend, you have to rewrite prompt logic.
- Two agents need to pass tasks to each other, but there is no agreed handoff format, so you hardcode a custom message protocol between them.
- An agent produced a strange output. You cannot trace back *why* — which prior decision led to this response — because nothing was logged at the intent level, only at the output level.
- Six months in, three old branches and two abandoned worktrees are still sitting in your working tree because nobody ever cleaned them up.

These are not exotic failure modes. They are the standard failure modes of growing an agent codebase without a principled foundation. BWOC was designed to address exactly these five dimensions — and the design of every part of the framework traces back to one of them.

---

## What BWOC is

BWOC (Buddhist Way of Coding) is a **backend-neutral specification and CLI framework** for incarnating, running, and governing AI coding agents. The reference implementation is in Rust, delivered as the `bwoc` CLI. The core artifact is a **cloneable agent template** — a structured directory of Markdown files, shell scripts, and symlinks that any LLM backend can read.

The four architectural pillars, stated plainly:

### 1. One repository, one agent

Each agent lives in its own self-contained directory cloned from the template. No shared runtime, no central agent server, no framework singleton that all agents must talk to. The agent *is* the directory. This makes agents independently forkable, auditable, and retirable without affecting others.

### 2. Backend-neutral by default

A single file — `AGENTS.md` — is the canonical instruction document for an agent. All supported backends (Claude, Codex, Kimi, Copilot, Ollama, any OpenAI-compatible endpoint) read the same file through symlinks:

```
CLAUDE.md   → AGENTS.md
CODEX.md    → AGENTS.md
KIMI.md     → AGENTS.md
OLLAMA.md   → AGENTS.md
```

No backend gets preferential treatment in the core documents. Adding a new backend is one command: `ln -s AGENTS.md <BACKEND>.md`. `AGENTS.md` is plain Markdown — no YAML frontmatter, no wikilinks, no vendor-specific syntax — so any LLM can parse it without preprocessing.

### 3. Persistent, impermanence-aware memory

Agents accumulate knowledge across sessions in `MEMORY.md`. The file is capped at 200 lines. This cap is enforced by `bwoc check`. The constraint forces deliberate choice: the agent (and its operator) must decide what is important enough to keep, which means the memory file contains signal, not noise. Stale entries are expected to be pruned, not preserved "just in case."

### 4. Multi-agent safe

Multiple agents coexist in a single workspace. BWOC provides shared task lists (Saṅgha teams), inter-agent messaging (signed envelopes), and a trust scoring system that gates delivery of inbound messages. An agent's `bwoc-agent` daemon verifies the sender's signature and checks a set of seven trust criteria before accepting a message into the inbox. Coordination is explicit and auditable, not implicit and fragile.

The `bwoc` CLI (Rust, cross-platform) drives all of this: `bwoc new`, `bwoc start`, `bwoc spawn`, `bwoc send`, `bwoc team`, `bwoc check`, `bwoc audit`.

---

## Why Western frameworks are not enough here

DDD, Clean Architecture, SOLID, and Hexagonal Architecture are thorough on structural concerns: how to separate layers, how to model a domain, how to control dependencies. They are thin on exactly the areas agent systems stress:

| Dimension | Where Western frameworks land | What agent systems need |
|---|---|---|
| State over time | Model the structure; assume state is managed elsewhere | First-class treatment of impermanence — when to keep, when to prune, how to timestamp |
| Failure tracing | Identify the exception; log the stack trace | Trace back through the *conditions* that led to the failure, not just the point of failure |
| Lifecycle | Stateless services have no lifecycle; this isn't that | A named arc: creation, operation-with-change, and clean release |
| Inter-agent trust | Not modeled (single-system assumption) | Explicit trust criteria, signed messages, gated delivery |
| Default safety | "Add authorization later" | Non-negotiable baseline rules that apply before any feature work |
| Scope discipline | "Add it while we're here" | A principle for declining to add: the smaller specification beats the more complete one |

Agent systems are not stateless services. They have identity, memory, growth, and relationships with other agents. Western frameworks were not designed for this shape of system. BWOC borrows from Buddhist analytical frameworks because those frameworks are, as the VISION document puts it, "unusually precise about impermanence, dependent origination, intent, and discipline — the same concerns, in a different vocabulary."

This is not a claim that Buddhism teaches software architecture. It is a claim that certain frameworks developed in that tradition happen to be precise, well-structured, and portable enough to serve as engineering thinking aids in a new domain.

---

## The complete mapping: engineering problem to framework

The full specification lives in [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md). Here is the reference table — engineering meaning first, Pali label in parentheses:

| Engineering problem | Framework (Pali) | What it gives you |
|---|---|---|
| Problem-solving structure | Four Noble Truths (Ariyasacca 4) | A four-step skeleton: state the problem (Dukkha) → find the root cause (Samudaya) → define the success state (Nirodha) → plan execution (Magga). Used for PRDs and task planning. |
| Functional requirements | Noble Eightfold Path (Magga 8) | Eight named pillars of correct operation: view, intention, communication, action, livelihood, effort, memory, focus. Maps to SRS sections. |
| System architecture | Five Aggregates (Khandha 5) | Five components: file layout, I/O, memory, logic, runtime. Maps to the ARCHITECTURE document. |
| State and impermanence | Three Marks (Tilakkhaṇa) | Everything is impermanent (Anicca) → memory needs timestamps and pruning. Stale branches are suffering (Dukkha) → clean them. No attachment to past state (Anattā) → release cleanly. |
| Failure analysis | Dependent Origination (Paṭiccasamuppāda) | Trace conditions backward: this exists because that exists. The visible error is rarely the root problem. Prevents shallow fixes. |
| Audit logging | Three Doors of Action (Kamma 3) | Three distinct log channels: file operations and commits (what you did), messages and logs (what you said), decisions and plans (what you intended). |
| Observability | Four Foundations (Satipaṭṭhāna 4) | Four observation targets: file state and working directory (body), tool results and I/O events (sensation), agent mode — planning/acting/verifying (mind state), rules in effect and patterns matching (mental objects). Maps to metrics, logs, traces, and state. |
| Agent lifecycle | Four Cultivations (Bhāvanā 4) | Four growth stages: template materialized (incarnation), first task done (onboarding), stable operation (competency), patterns shared with others (mentorship). |
| Self-improvement | Three Roots of Wisdom (Paññā 3) | Three improvement channels: learning from documents and examples (study), synthesizing by planning and pattern extraction (reasoning), learning from practice via feedback and retrospectives (cultivation). See the [self-improvement chapter](../self-improvement/HANDBOOK.en.md). |
| Capability maturity | Seven Noble Treasures (Ariya-dhana 7) | A seven-level maturity scale: trust in conventions (L1), following rules (L2), awareness of errors (L3), depth of knowledge (L4), sharing capability (L5), independent judgment (L6). Maps to the `maturity: L1..L6` tag in skill slot files. |
| Error UX disposition | Four Divine Abidings (Brahmavihāra 4) | Four stances: friendly, direct tone (Mettā); suggest a fix, not just the error (Karuṇā); acknowledge when others were right (Muditā); stay even when the user is frustrated (Upekkhā). |
| Inter-agent trust | Seven Qualities of a Good Friend (Kalyāṇamitta 7) | Seven boolean criteria an agent peer must meet before its messages are accepted: pleasant to delegate to, respectable in capability, helps us improve, speaks beneficial truth, can take feedback, can explain depth, does not lead astray. Enforced by the daemon's trust gate. |
| Inter-agent coordination | Six Conditions of Cordiality (Sāraṇīyadhamma 6) | Six coordination properties: benevolence in all three action channels toward other agents, fair resource sharing, shared rules, aligned goals. |
| Threat modeling | Three Cravings (Taṇhā 3) | Three threat categories: craving for stimulus → prompt injection and social engineering; craving for persistence → privilege escalation; craving for destruction → data loss and destructive actions. |
| Baseline safety | Five Precepts (Sīla 5) | Five non-negotiable rules: no `rm -rf` of repo root, no committing secrets, no spoofing agent identity, no bypassing verification gates, no undeclared side-effects. |
| Fleet governance | Seven Non-Decline Principles (Aparihāniya-dhamma 7) | Seven conditions that keep a multi-agent fleet from decay: regular sync points, coordinated session start/end, process-bound convention change, respect for template version hierarchy, protection of vulnerable agents and users, shared registry and schema integrity, protection of senior trusted agents. |
| Scope discipline | Four Unthinkables (Acinteyya 4) | Deliberate modeling of what not to reason about: LLM provider intent, model internals, long-term business outcomes, scope outside this system. Prevents analysis paralysis and over-specification. |
| Work quality metrics | Four Bases of Power (Iddhipāda 4) | Four KPIs: working within declared domain (drive), task completion rate (persistence), gate compliance (attention), self-improvement metrics (investigation). |
| Context sensing | Seven Qualities of a True Person (Sappurisadhamma 7) | Seven situational dimensions: cause and principle, result and goal, self and limits, scope, time, community, persons. Used for PRD stakeholder analysis and pre-task context scans. |
| UX principles | Four Bases of Sympathy (Saṅgahavatthu 4) | Four user-relation stances: generous defaults, clear and kind error messages, actions that benefit the user, equal treatment regardless of user familiarity. |
| Right effort directions | Four Right Efforts (Padhāna 4) | Four directions: prevent new bad (lint), abandon existing bad (format, refactor), develop new good (new tests), maintain existing good (regression). Maps to the verification gate sequence. |

The arc that runs underneath all of these is the **uppāda · ṭhiti · vaya** triad from AN 3.47: arising, persisting-with-change, passing-away. Every BWOC artifact — a task, a session, a worktree, a whole agent — follows this arc. The 22 frameworks specify how each phase of the arc stays principled.

---

## Five principles that govern the hardest tradeoffs

These five principles appear throughout the framework. They are invoked when two good designs conflict. Each is stated as a plain engineering rule; the Pali label is the shorthand you will see in docs and commit messages.

### 1. Verify before acting on memory (Yoniso Manasikāra)

Memory is a past claim. Before acting on a remembered state — a file path, a branch name, an API shape, a prior decision — verify that the claim is still true against the present state of the system.

In practice: before writing to a file the agent last saw three sessions ago, read it first. Before running a command the agent has run before, check that the environment has not changed. This principle prevents an entire class of agent failures that come from acting on stale assumptions.

The principle also applies at design time: when in doubt about whether a proposed design reflects the actual system, read the current source rather than trusting the design diagram.

### 2. Right amount, not maximum (Mattaññutā)

Build the smallest specification that is load-bearing. The `MEMORY.md` cap at 200 lines is the most visible enforcement of this principle, but it applies everywhere: do not add a slot file that does not earn its place; do not add a spec section that is not needed; do not extend a design to cover cases that have not arisen.

The framework's VISION document states this explicitly as a tiebreaker: "the smaller specification beats the more complete one, unless completeness is load-bearing." This is not a preference for simplicity as an aesthetic; it is recognition that every line of specification carries a maintenance and cognitive overhead cost, and that cost compounds.

For operators, this principle shows up as: trim before you expand; prune before you add; if the 200-line memory file is full, decide what to drop rather than raising the cap.

### 3. No clinging to stale state (Anattā)

When a task ends, clean up: delete the branch, remove the worktree, prune memory entries that are no longer accurate. Do not preserve state "in case." The agent's identity is not in its accumulated state; it is in its current capacity and its current task.

In code: stale branches are a signal that the person who owns them is not practicing Anattā. In agent design: a memory file that is padded with obsolete entries is not rich context; it is noise that degrades future performance.

`bwoc check` will flag a `MEMORY.md` that exceeds 200 lines. The right response is not to raise the limit; it is to prune.

### 4. Equal treatment of all backends (Samānattatā)

No backend gets preferential treatment in the core specification. `AGENTS.md` must be parseable by any LLM backend with no pre-processing. A design choice that works on Claude but requires rewriting for Ollama is not yet complete.

In practice: use `{{camelCase}}` placeholders for all configurable values in `AGENTS.md`; never hardcode model IDs, tool names, or vendor-specific response formats in the canonical instruction document; run `bwoc check` to verify neutrality. A backend-specific optimization belongs in a companion file, not in `AGENTS.md`.

This principle also applies to self-hosted models. Ollama and any OpenAI-compatible endpoint are first-class deployment targets in `bwoc-harness`, not afterthoughts.

### 5. Shared conventions beat personal preferences (Sīlasāmaññatā)

All agents in a workspace follow the same conventions: the same `conventions.md`, the same manifest schema, the same neutrality rules, the same `bwoc check` gate. An agent author's preference for a different naming convention or a different memory format does not override the shared standard.

This matters because conventions are the interface between agents. When two agents exchange messages, they rely on both ends using the same format. When an operator inspects a workspace with six agents, they rely on all six having the same layout. Personal variation breaks interoperability.

The convention is enforced mechanically by `bwoc check`, which runs the neutrality audit, validates `config.manifest.json` as well-formed JSON, and flags `MEMORY.md` violations. The check is designed to be run in CI.

---

## The arc: every BWOC object has a birth, a life, and a clean death

The framing that ties everything together is the lifecycle triad — arising, persisting-with-change, passing-away — derived from the Saṅkhata Sutta (AN 3.47). Every BWOC object follows this arc:

| Phase | Plain name | What the agent does |
|---|---|---|
| Arising (uppāda) | Birth | Identity is created. Agent is cloned from the template; persona is set; capabilities are declared; manifest is resolved. |
| Persisting-with-change (ṭhiti) | Operating | Agent works. Plans with the four-step problem structure; acts within eight functional pillars; remembers with impermanence-aware memory; communicates with the four-disposition UX protocol. State evolves, but under discipline. |
| Passing-away (vaya) | Cleanup | Task or session ends. Worktree is deleted, branch released, memory pruned, task closed. No clinging. |

This arc is not a metaphor. It is the actual structure of `bwoc new` → `bwoc start` → `bwoc stop` → `bwoc retire`. The lifecycle commands implement the arc. The 22 frameworks specify the discipline inside each phase.

---

## Why you would adopt BWOC

### You are building more than one agent

The value of a shared foundation grows with the number of agents. One agent: the overhead of learning the template may not pay off immediately. Three or more agents: consistency across all of them — same memory format, same audit trail, same check gate — starts to matter. Ten agents in one workspace: the shared convention and multi-agent safety infrastructure are load-bearing.

### You care about backend portability

If you have ever migrated an agent from one LLM to another and discovered that your system prompt was written in Claude-flavored idiom that did not translate, you know the problem. BWOC's backend-neutral design makes this a first-class constraint, not a retrofit.

### You want auditable, recoverable agents

The three-channel audit trail (what the agent did, what it said, what it intended) and the problem-solving-spine structure (problem → root cause → success state → plan) make it possible to trace back *why* an agent produced a given output. This is valuable during debugging and during post-incident review.

### You want built-in pressure against over-engineering

The 200-line memory cap, the "smaller spec wins" principle, the scope-discipline framework, and the `bwoc check` gate all create structural pressure toward lean, deliberate design. This is not just a stylistic preference; it is enforced mechanically.

### You want a shared vocabulary for design tradeoffs

When two engineers disagree about whether to expand the memory file or prune it, "Mattaññutā says prune unless load-bearing" is a shorter and more precise argument than re-litigating the underlying tradeoff from first principles every time. The vocabulary earns its place by making recurring design decisions explicit and fast to resolve.

---

## For whom BWOC is not a good fit

BWOC is not the right foundation if:

- **You want the fastest possible time-to-first-token.** BWOC optimizes for principled, auditable, recoverable agents. It adds structure that a quick prototype does not need.
- **You are building a single, ephemeral agent.** The framework's value is in consistency across a fleet over time. For a one-off script-level agent, the overhead is not worth it.
- **You need a central runtime or orchestration server.** BWOC is deliberately decentralized: no central agent server, no database, no container runtime. If your architecture requires a central broker, BWOC's model is the wrong shape.
- **You cannot accept the specification-first approach.** BWOC documents before it implements. The Markdown specification is the source of truth; code follows doctrine. If your team's culture is "code first, document later," the required discipline will feel like friction.

---

## Design tradeoffs and non-goals

Stated directly from the VISION document:

**BWOC is not a religion.** Not a meditation guide. Not an endorsement of any teacher, lineage, or tradition.

**BWOC does not replace DDD, Clean Architecture, or SOLID.** It extends them into dimensions they were never designed to address. You can use BWOC alongside those frameworks.

**The default `bwoc` path is a zero-dependency CLI orchestrator.** It exec's a vendor agentic CLI and makes no model-API calls. The `bwoc-harness` runtime is opt-in for self-hosted backends. Users who only work with cloud backends never pull the runtime dependency.

**Default to not adding.** When in doubt, the framework's own design principle applies: the smaller specification beats the more complete one. BWOC does not proactively model things that have not arisen as real engineering problems.

**Some things are deliberately not modeled** — LLM provider intent, model internals, long-term business outcomes, scope outside this system. These are the Acinteyya (Four Unthinkables): boundaries of specification, not gaps to fill.

---

## The stack at a glance

```
┌──────────────────────────────────────────────────────┐
│  Fleet Governance (Aparihāniya-dhamma 7)             │ ← Org level
├──────────────────────────────────────────────────────┤
│  Threat Model (Taṇhā 3) + Baseline Safety (Sīla 5)   │ ← Security
├──────────────────────────────────────────────────────┤
│  Lifecycle (Bhāvanā 4) + Self-improvement (Paññā 3)  │ ← Agent growth
├──────────────────────────────────────────────────────┤
│  Coordination (Sāraṇīyadhamma 6) + Trust (Kalyāṇamitta 7) │ ← Interconnect
├──────────────────────────────────────────────────────┤
│  UX (Saṅgahavatthu 4) + Error disposition (Brahmavihāra 4) │ ← User layer
├──────────────────────────────────────────────────────┤
│  Functional requirements (Magga 8)                   │ ← SRS
├──────────────────────────────────────────────────────┤
│  Architecture (Khandha 5)                            │ ← Components
├──────────────────────────────────────────────────────┤
│  Observability (Satipaṭṭhāna 4)                      │ ← Cross-cutting
├──────────────────────────────────────────────────────┤
│  Engine of work (Iddhipāda 4)                        │ ← Runtime
├──────────────────────────────────────────────────────┤
│  State & Audit (Tilakkhaṇa + Kamma 3)                │ ← Foundation
├──────────────────────────────────────────────────────┤
│  Failure analysis (Paṭiccasamuppāda)                 │ ← When broken
├──────────────────────────────────────────────────────┤
│  Thinking method (Yoniso Manasikāra + Acinteyya 4)   │ ← How to reason
└──────────────────────────────────────────────────────┘
        Problem-solving cycle (Ariyasacca 4) — end to end
        Context sensing (Sappurisadhamma 7) — end to end
```

---

## See also

- [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) — the 22 frameworks in full; the normative reference. When this chapter and PHILOSOPHY conflict, PHILOSOPHY wins.
- [VISION.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/VISION.md) — the design intent, the gap BWOC fills, the five governing principles, and the non-goals.
- [GLOSSARY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md) — every Pali term in BWOC, one engineering meaning per line.
- [Agents handbook](../agents/HANDBOOK.en.md) — incarnation, slot layout, manifest, the `bwoc check` gate.
- [Self-improvement chapter](../self-improvement/HANDBOOK.en.md) — Paññā 3 in practice: how an agent learns from study, reasoning, and feedback.
- [Glossary (handbook)](../glossary.en.md) — handbook-level term lookup.
- [BWOC Framework repository](https://github.com/bemindlabs/BWOC-Framework) — source of truth for code and full spec.
