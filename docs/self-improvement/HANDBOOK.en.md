---
title: Agent Capabilities & Self-Improvement
nav_order: 9
---

# Agent Capabilities & Self-Improvement

🇹🇭 [ภาษาไทย](HANDBOOK.th.md)

> **Source of truth:** [`SELF-IMPROVEMENT.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/SELF-IMPROVEMENT.en.md) · [`PHILOSOPHY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) · On any conflict, the framework repo wins and this handbook has a bug — please fix it.

---

## What "self-improving agent" means here

In most software, improvement means redeploying a new model or manually rewriting code. A BWOC agent improves *itself* — structurally, across sessions — by recording what it learned, reflecting on decisions, and feeding verified lessons back into its working knowledge.

Three things make this concrete:

1. The agent **declares what it can and cannot do** before it starts, so expectations are honest.
2. Every piece of new knowledge is written to a file in a specific format, not left in the conversation context that vanishes at session end.
3. Lessons can escalate: a personal note can become a pattern, then a shared rule, then a fleet-wide convention — a curation pipeline from individual to collective.

This chapter covers the full model: capability declaration, the seven-level skill maturity ladder, the three-root learning loop, memory tiers and commands, the curation pipeline, the work engine that sustains it, the four-stage lifecycle, metrics, anti-patterns, and the checklist you use to actually make an agent learn.

---

## Part 1 — The Capability Model

### 1.1 Capability declaration

Before an agent does any work, it states what it can and cannot do. This is called a capability declaration (the Pali label is *Attanutata*, meaning "knowing self" — see [glossary](../glossary.en.md)).

Mechanically, capability declaration lives in two places:

- **`config.manifest.json`** — the `scope` and `out_of_scope` fields. These are machine-readable; `bwoc check` validates them.
- **`persona/identity.md`** — the human-readable description of the agent's domain, strengths, and explicit limits.

An agent that receives a task outside its declared scope should refuse it rather than attempt something it is not equipped for. This is not failure — it is integrity.

> **Note:** An agent with a vague or absent capability declaration cannot self-improve reliably. The declaration is the baseline against which progress is measured.

### 1.2 Skills and the L1–L7 maturity ladder

Concrete capabilities are packaged as **skills** — files under `skills/`, one per area of competence. Each skill has a frontmatter `maturity` field set to one of seven levels. The framework calls this the seven noble treasures model (*Ariya-dhana 7*), where each level maps to a quality the agent has developed in that skill area.

| Level | Label | What it means in practice |
|---|---|---|
| **L1** | Beginner — *Saddhā* (trust) | Follows documented conventions by rote; relies on examples; cannot reason about edge cases |
| **L2** | Follower — *Sīla* (rule-following) | Applies rules consistently without being prompted; catches obvious violations |
| **L3** | Aware — *Hiri-Ottappa* (error awareness) | Notices when something is wrong before it breaks; can articulate why |
| **L4** | Knowledgeable — *Suta* (knowledge depth) | Understands the why behind rules; can consult and synthesize sources; explains to others |
| **L5** | Generous — *Cāga* (sharing) | Extracts patterns and contributes them back; begins mentoring; knowledge escapes the individual |
| **L6** | Judicious — *Paññā* (independent judgment) | Makes sound calls in novel situations; knows when rules do not cover the case |
| **L7** | Mastery | Designs new conventions; capable of retiring and being replaced cleanly; the skill outlives the agent |

**What can be verified at each level:**

- L1–L2: task completion on well-trodden paths.
- L3–L4: post-mortem quality — does the agent identify root causes, not just symptoms?
- L5: are patterns appearing in `skills/` or cross-agent knowledge bases?
- L6: are decisions well-reasoned in `memories/decision-*.md`, covering alternatives and revisit conditions?
- L7: is the agent mentoring others and leaving behind reusable conventions?

Skills are bounded and verifiable. A skill file that covers everything is worth nothing. Narrow scope, clear maturity, honest limits.

---

## Part 2 — The Three Ways an Agent Learns

The learning model is built on the Three Roots of Wisdom (*Paññā 3* — see [glossary](../glossary.en.md)). Wisdom has three independent sources; missing any one of them produces a shallow or wrong result.

```
Study ──────► Reflect ──────► Practice
  (suta)       (cintā)       (bhāvanā)
     ▲                            │
     └────── fed back in ─────────┘
              (after curation)
```

### 2.1 Study — learning from reading (*Sutamayā*)

**What triggers it:** session start, before an unfamiliar task, when an unknown comes up.

**Inputs:**
- `AGENTS.md` and backend symlinks
- `conventions/*.md`
- `docs/` in the agent directory
- Peer skill files and shared knowledge bases
- Tier-2 cross-agent memory (see Part 4)

**The activity:** load relevant documents before working, not during. "Looking it up mid-task" is slower and less reliable than pre-loading. For unknown concepts, search the skill files and framework knowledge base first; only escalate to live searching when those are silent.

**What it writes:** a `memories/reference-*.md` file.

```markdown
# memories/reference-postgres-naming.md
---
type: reference
source: conventions/database.md#naming
date: 2026-05-22
verifiedAgainst: schema.sql@abc123
---

PostgreSQL naming conventions in use here:
- Tables: snake_case plural
- Columns: snake_case
- Indexes: idx_<table>_<columns>
```

**Quality check for a reference file:**
- Is the source traceable (a link or file reference)?
- Does it have a verification date — was it checked against actual code?
- Is it selective? A full document dump is not a reference; it is noise.

### 2.2 Reflection — learning from reasoning (*Cintāmayā*)

**What triggers it:** a pattern repeating across tasks, before a significant architectural decision, after synthesizing several reference sources.

**The activity:** extract patterns, write decision rationales, connect sources that have not been connected before, and run mental simulations ("if I do X, what follows?"). This last step — tracing consequences before acting — is called wise attention (*Yoniso Manasikāra*, see [glossary](../glossary.en.md)). It is how an agent avoids applying a remembered fact to a situation that has changed.

**What it writes:** a `memories/decision-*.md` file.

```markdown
# memories/decision-2026-05-22-caching-strategy.md
---
type: decision
date: 2026-05-22
status: active
references:
  - reference-redis-cluster.md
  - feedback-PROJ-30-cache-thrashing.md
---

## Decision
Use Redis Sentinel instead of Cluster mode.

## Alternatives Considered
- Redis Cluster — complexity exceeds current scale
- Redis Sentinel — chosen
- Memcached — lacks persistence

## Rationale
Per reference-redis-cluster.md + feedback-PROJ-30:
Sentinel matches required scale and availability.

## Revisit If
- Scale exceeds 50 k req/s
- Multi-region requirement appears
```

**Quality check for a decision file:**
- Are alternatives listed, not just the winner?
- Is the rationale sourced (references present)?
- Are the revisit conditions explicit and testable?

### 2.3 Practice — learning from doing (*Bhāvanāmayā*)

**What triggers it:** task completion (especially with unexpected outcomes), any failure, a scheduled retrospective.

**The activity:** compare expected versus actual outcomes; trace the causal chain backward. The causal-chain trace comes from the framework's failure-analysis model (*Paṭiccasamuppāda*, see [glossary](../glossary.en.md)) — the visible problem is usually not the root; trace conditions backward until you find the assumption that was wrong.

**What it writes:** a `memories/feedback-*.md` file.

```markdown
# memories/feedback-PROJ-42-schema-migration.md
---
type: feedback
date: 2026-05-22
task: PROJ-42
outcome: success-with-issues
---

## Expected
Migration < 30 min, no downtime

## Actual
- 47 min (50% over estimate)
- Brief 2s lock on users table

## Why (causal trace)
- Didn't know real users-table size (count only, not indexes)
- Estimate was wrong → missed low-traffic window

## Lessons
- Add pre-migration size check to skill file
- Update reference-schema-migration.md

## Convention Impact
Yes → submit convention change proposal
```

**Quality check for a feedback file:**
- Are expected and actual both stated explicitly?
- Is the causal chain present (not just "what went wrong")?
- Are action items concrete — named files, specific changes?

---

## Part 3 — The Wisdom Loop

The three roots are not independent passes. They form a closed loop: study informs reflection, reflection generates hypotheses that practice tests, and practice results — after curation — become new study material for the next cycle.

```
┌─────────────────────────────┐
│  Study (Suta)               │
│  reference-*.md             │
└──────────────┬──────────────┘
               │ informs
               ▼
┌─────────────────────────────┐
│  Reflect (Cintā)            │
│  decision-*.md              │
└──────────────┬──────────────┘
               │ becomes hypothesis
               ▼
┌─────────────────────────────┐
│  Practice (Bhāvanā)         │
│  feedback-*.md              │
└──────────────┬──────────────┘
               │ feeds back (after curation)
               ▼
        Updated study material
```

One cycle does not produce mastery. Improvement is cumulative: each loop sharpens the reference base, produces better-reasoned decisions, and reduces the gap between expected and actual outcomes. An agent that runs all three consistently over many tasks is building genuine, verifiable competence — not accumulating token context.

---

## Part 4 — Memory Tiers and Commands

BWOC uses a two-tier memory model. Tier 1 is the lightweight, per-workspace fast store. Tier 2 is persistent, cross-session deep memory.

### 4.1 Tier 1 — `MEMORY.md`

A single file, capped at 200 lines, enforced by `bwoc check`. This constraint reflects the impermanence principle (*Anicca* — see [glossary](../glossary.en.md)): memory that is never pruned becomes stale and misleading. Lean memory is accurate memory.

Tier-1 content is indexed, searchable, and visible to any agent in the workspace.

```bash
bwoc memory put "key" "value"     # write a Tier-1 entry
bwoc memory list                  # list all Tier-1 entries
bwoc memory search "query"        # search Tier-1
bwoc memory rm "key"              # delete a Tier-1 entry
```

### 4.2 Tier 2 — Deep memory

Tier-2 memory persists across sessions and can be shared across agents. It is the destination for curated patterns and cross-agent knowledge.

```bash
bwoc memory wake-up       # session start — recall relevant prior context
bwoc memory mine          # session end — persist learnings from this session
bwoc memory t2-search "query"   # search past decisions and feedback across sessions
```

**Practical session discipline:**
1. Run `bwoc memory wake-up` at the start of every session.
2. Do the work.
3. Run `bwoc memory mine` at the end of every session.

Skipping `mine` means the session's learning is lost. Skipping `wake-up` means the agent starts blind.

### 4.3 The `memories/` directory

Individual memory files (`reference-*.md`, `decision-*.md`, `feedback-*.md`) live in the agent's `memories/` directory. These are the raw material that `bwoc memory mine` draws from when promoting knowledge to Tier 2.

---

## Part 5 — The Curation Pipeline

Not every note deserves to become shared knowledge. Curation is the process of deciding what escalates and what stays local.

```
L1 Personal  ──────►  L2 Pattern  ──────►  L3 Cross-agent  ──────►  L4 Convention
(Tier 1,              (mine to             (Tier-2 memory,           (fleet-wide rule,
 memories/)           decision-*.md)        skill files)              convention proposal)
```

### Level 1 — Personal (Tier 1)

The agent keeps the note under `memories/`. It verifies the note in the next session. Most observations stay here — they are too specific or too early to generalize.

### Level 2 — Pattern detected

After the same pattern appears three or more times, it earns extraction to a `decision-*.md` and begins appearing in `skills/capabilities.md`. The threshold of three is a discipline: it prevents premature generalization from one data point.

### Level 3 — Cross-agent (Tier 2)

When the pattern is useful to more than one agent, it is mined to Tier-2 memory and added to shared skill files. Other agents can now benefit from it without independently rediscovering it.

### Level 4 — Convention

When the pattern is a fleet-wide best practice, it enters the formal convention change process (described in [`FLEET-GOVERNANCE.en.md`](https://://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md)). At this level, the knowledge is no longer "this agent's learning" — it is part of the framework's rules.

> **Tip:** The curation pipeline is one-way under normal operation. Conventions do not get quietly demoted. If a convention turns out to be wrong, fix it through the same formal process.

---

## Part 6 — The Work Engine and Effort Discipline

Self-improvement does not happen automatically. It requires sustained effort — but effort without discipline leads to over-engineering, burnout, or thrashing. BWOC frames this through two complementary models.

### 6.1 The engine of work (*Iddhipāda 4*)

Four qualities that drive accomplishment:

| Quality | Plain meaning | In practice |
|---|---|---|
| **Drive** (*Chanda*) | Working in your declared domain | Stay within scope; don't drift into adjacent tasks |
| **Persistence** (*Viriya*) | Task completion rate | Finish what you start; don't abandon work mid-stream |
| **Focus** (*Citta*) | Compliance with process gates | Follow the verification sequence (lint → test → build), don't skip |
| **Investigation** (*Vīmaṃsā*) | Self-improvement metrics | Measure, review, adjust — not just do |

Investigation (*Vīmaṃsā*) is the self-improvement engine within the engine. It is what takes the raw outputs of the three learning roots and asks: "is it working? what is improving? what is stagnant?"

### 6.2 Effort discipline (*Padhāna 4*)

Four directions of right effort, used to decide where to spend improvement energy:

| Direction | Plain meaning | When to apply it |
|---|---|---|
| **Restrain** | Prevent new problems from entering | When a process gap is visible — close it before the next task |
| **Abandon** | Remove existing problems | When a stale memory, bad pattern, or wrong convention is found — prune it |
| **Develop** | Build something new and good | When a capability gap is identified — write the skill, create the reference |
| **Maintain** | Sustain what already works | When passing tests and good conventions exist — protect them from regression |

These four directions prevent two common failure modes: doing only "develop" (perpetual new features, stale foundation) and doing only "maintain" (no growth). Improvement requires all four, in balance.

---

## Part 7 — The Growth Lifecycle

An agent does not stay at the same maturity level. The four-stage lifecycle (*Bhāvanā 4* — see [glossary](../glossary.en.md)) tracks the arc from newly incarnated to fully retired.

| Stage | Plain name | Indicator | What the agent does |
|---|---|---|---|
| **Kāya-bhāvanā** | Growth | Template materialized, placeholders set | Learns conventions, completes first tasks, builds reference files |
| **Sīla-bhāvanā** | Maturity | Conventions internalized, low retry rate | Works stably; starts extracting patterns (L3–L4 skill range) |
| **Citta-bhāvanā** | Mentoring | Patterns shared; other agents benefit | Contributes to shared skills and conventions; assists newer agents |
| **Paññā-bhāvanā** | Release | Patterns stable and shared; agent is replaceable | Cleans up memory, writes final lessons, retires gracefully |

The lifecycle is not linear in real time — an agent at L4 in one skill may still be at L1 in a newly acquired skill. Maturity is per-skill, not per-agent overall. The agent-level lifecycle tracks the dominant pattern.

**Retirement is not failure.** An agent that reaches the release stage and retires cleanly — having transferred its knowledge to shared conventions and Tier-2 memory — has succeeded. The knowledge outlives the agent. This is the framework's expression of non-clinging (*Anattā* — see [glossary](../glossary.en.md)).

---

## Part 8 — Metrics: How to Tell an Agent Is Actually Improving

Improvement claimed without measurement is decoration. The framework defines per-root metrics and combines them under investigation (*Vīmaṃsā*).

### Study metrics

| Metric | What it measures | Good sign |
|---|---|---|
| Source diversity | Number of distinct sources cited per decision | Rising over time |
| Verification rate | Proportion of reference files checked against current code | >80% recently verified |
| Pre-load habit | Does the agent load docs before tasks, not during? | Consistent yes |

### Reflection metrics

| Metric | What it measures | Good sign |
|---|---|---|
| Alternatives per decision | Count of alternatives in `decision-*.md` files | At least 2–3 per decision |
| Cross-references | References to prior feedback and reference files in decisions | Present in every decision |
| Revisit accuracy | When a decision was revisited, was it revised correctly? | Revisions are well-calibrated |

### Practice metrics

| Metric | What it measures | Good sign |
|---|---|---|
| Post-mortem completion rate | Fraction of notable failures with a `feedback-*.md` | Near 100% |
| Action item closure | Fraction of feedback action items that were actually done | Rising |
| Pattern latency | How many occurrences before a pattern is named | Declining — faster detection |

### Combined: Investigation (*Vīmaṃsā*)

| Metric | What it measures |
|---|---|
| Improvement velocity | Time from feedback creation to action taken |
| Knowledge half-life | How long a reference file stays valid before needing re-verification |

A healthy agent shows: rising source diversity, consistent post-mortems, shrinking pattern latency, and a manageable knowledge half-life. An agent that scores well only on one root (say, lots of reference files but zero feedback files) is not improving — it is studying but not doing, or doing but not reflecting.

---

## Part 9 — Anti-Patterns and Pitfalls

| Anti-pattern | What is missing | What to do |
|---|---|---|
| Memorizing docs without testing | Practice (Bhāvanā) | Write a feedback file after the next task using those docs |
| Patching without analysis | Reflection (Cintā) | Before the next patch, write a one-sentence decision rationale |
| Reinventing patterns — solving a solved problem again | Study (Suta) | Run `bwoc memory t2-search` before starting; check if the problem is known |
| Endless reflection, no action | Practice (Bhāvanā) | Cap decision-writing to 30 minutes; execute and iterate |
| Cargo-culting from other agents — copy-paste without understanding | Reflection + Practice | Ask: "do I understand why this works? does it apply here?" |
| Hoarding stale memory | Impermanence discipline (Anicca) | Prune `MEMORY.md` when it hits the 200-line cap; remove entries not verified recently |
| Acting on remembered facts without checking current state | Verification before act (Yoniso Manasikāra) | Before acting on a reference file, check: is the source still current? |
| Learning without verifying | Same as above | Every reference file needs a `verifiedAgainst` field |

---

## Part 10 — Triggers and Checklist

Use this checklist to make an agent learn systematically. Each trigger maps to an action.

### Trigger: task failure or unexpected outcome

- [ ] Write `memories/feedback-TASKID-<short-name>.md`
- [ ] Fill in Expected, Actual, Why (causal trace), Lessons, Convention-Impact
- [ ] Check if any existing reference or decision file is now contradicted
- [ ] If the root cause was a missing reference, write `memories/reference-*.md`

### Trigger: same issue appearing a second time

- [ ] Search past memory: `bwoc memory t2-search "<topic>"`
- [ ] If a pattern exists and you missed it, fix the wake-up habit
- [ ] If no pattern exists, mark the second occurrence in the feedback file

### Trigger: same issue appearing a third time (or more)

- [ ] Extract to `memories/decision-*.md` with the pattern named explicitly
- [ ] Update `skills/capabilities.md` to reference the pattern
- [ ] Consider whether cross-agent promotion is warranted

### Trigger: promotion eligibility (L4 → L5 and above)

- [ ] Verify all three learning roots are present: reference files, decision files, feedback files
- [ ] Confirm at least one pattern has been shared (Tier 2 or skill file)
- [ ] Run `bwoc check` — `MEMORY.md` must be under 200 lines

### Trigger: convention update (another agent or the framework changes a convention)

- [ ] Re-read the affected `reference-*.md` files
- [ ] Update any `decision-*.md` files that cited the old convention
- [ ] Run `bwoc memory wake-up` in the next session to reload fresh context

### Trigger: session end

- [ ] Run `bwoc memory mine` — do not skip this
- [ ] Review `MEMORY.md` line count; prune if approaching 200

### Trigger: fleet sync or knowledge sharing

- [ ] Check if any personal pattern (Level 2) is ready for cross-agent promotion (Level 3)
- [ ] Review Tier-2 memory for new insights from other agents: `bwoc memory t2-search`

---

## Part 11 — Real-World Reference: Hermes-Agent

The Nous Research **hermes-agent** (Python) is a real self-improving agent that implements a learning loop, auto skill creation, and cross-session memory. It is available as a read-only study reference in this workspace. The design decisions in that system — particularly how it structures memory across sessions and triggers skill updates — are worth reading alongside this framework's approach.

BWOC's approach differs: it is backend-neutral (same agent on any LLM), uses structured file-based memory rather than a vector store as the primary surface, and enforces the curation pipeline via `bwoc check` constraints. The principles are compatible; the implementation surfaces are different.

---

## See Also

| Resource | What you get |
|---|---|
| [`SELF-IMPROVEMENT.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/SELF-IMPROVEMENT.en.md) | Authoritative learning-loop spec — memory schemas, curation pipeline, metrics |
| [`PHILOSOPHY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) | All 22 frameworks — Paññā 3, Iddhipāda 4, Bhāvanā 4, Ariya-dhana 7 in full |
| [`../slots/HANDBOOK.en.md`](../slots/HANDBOOK.en.md) | Writing persona, mindset, and skill files — including maturity frontmatter |
| [`../agents/HANDBOOK.en.md`](../agents/HANDBOOK.en.md) | Agent layout, `bwoc check`, manifest, the full agent arc |
| [`../glossary.en.md`](../glossary.en.md) | All Pali terms used above in one-line engineering definitions |
