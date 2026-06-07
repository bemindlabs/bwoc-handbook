---
title: Council — Structured Fleet Decisions
nav_order: 15
---

# Council — Structured Fleet Decisions

> Thai version: [HANDBOOK.th.md](HANDBOOK.th.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) · Glossary: [../glossary.en.md](../glossary.en.md) · Fleet Operations: [../fleet-ops/HANDBOOK.en.md](../fleet-ops/HANDBOOK.en.md) · Agents: [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md)

**Source of truth.** The canonical specification lives in the framework repo. On any conflict, the repo wins and this handbook has a bug. Relevant docs: [`PLUGINS.en.md` §Council Decision Schema](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md#council-decision-schema) · the [BWOC-56 council design note](https://github.com/bemindlabs/BWOC-Framework/blob/main/notes/2026-05-28_council-plugin-architecture.md).

---

## Table of Contents

1. [What a council is](#1-what-a-council-is)
2. [The protocol — propose → discuss → vote → resolve](#2-the-protocol--propose--discuss--vote--resolve)
3. [Voting models](#3-voting-models)
4. [Running a council with `bwoc council`](#4-running-a-council-with-bwoc-council)
5. [The Council Decision record](#5-the-council-decision-record)
6. [Binding vs advisory](#6-binding-vs-advisory)
7. [See also](#7-see-also)

---

## 1. What a council is

A **council** is BWOC's structured protocol for letting several agents *decide something together* and keep a durable, auditable record of how they decided. It is the framework's first **coordination** plugin kind: where a `workflow`/`jira` plugin reaches *outward* to an external system and an `audit`/`okr` plugin reports *over* the workspace, a `council` acts **among the fleet's own agents**.

The defining rule: **a council records decisions, it does not execute them.** Resolving a council produces a decision entry — the outcome, the discussion trail, and any dissent. If the outcome is *binding*, the council emits a `bwoc task` for someone to carry out; it never mutates the workspace itself. This keeps deliberation and action cleanly separated.

> [!note]
> A council is the right tool when a choice is **contested, consequential, and worth a written rationale** — adopting an architecture, retiring an agent, accepting a risk. For a single agent doing its own work, no council is needed; for routing an incident, use the team task list (see [Fleet Operations](../fleet-ops/HANDBOOK.en.md)).

---

## 2. The protocol — propose → discuss → vote → resolve

Every council moves through an explicit state machine:

```
proposed → discussing → voting → resolved
                                  ↘ abandoned   (if quorum is never met)
```

- **propose** — any agent opens a decision with a question and **≥2 options**, naming the participating `bwoc team`.
- **discuss** — participants take turns across one or more **rounds**. Each turn is a real `bwoc send` message; the council record only *references* the envelope (the inbox is the transport, the record is the ledger).
- **vote** — each participant casts one vote for an option, or abstains. Votes are **append-only** — a re-cast appends a new vote rather than overwriting, because the trail is the point.
- **resolve** — once the **quorum** is met and the voting model is satisfied, the outcome is recorded and the decision closes. If quorum is never reached, the decision is **abandoned** (not failed — simply never decided).

Participants are drawn strictly from the referenced team; a vote from outside the team is rejected.

---

## 3. Voting models

A council declares *how* votes become an outcome. Four models ship:

| Model | Outcome rule |
|---|---|
| `simple-majority` | The option with the most votes wins. |
| `consensus` | An option wins only if **no participant dissents** against it. |
| `weighted` | Votes carry per-participant weights (e.g. by seniority or expertise). |
| `sangha` | The `council-sangha-7` model, drawn from **Aparihāniya-dhamma 7** (the seven non-decline conditions): decisions favour full participation, meeting in concord, and not overriding settled practice. |

> [!tip]
> Use `consensus` for changes that are hard to reverse (a participant who would have to live with a bad outcome can block it), and `simple-majority` for routine choices where a clear quorum is enough.

---

## 4. Running a council with `bwoc council`

The protocol maps directly onto the CLI. A typical run:

```bash
# 1. Open a decision among an existing team
bwoc council propose security \
  --question "Adopt the Phase 5 sandbox contract as-is?" \
  --option adopt --option revise

# 2. Participants discuss across rounds (each turn is a bwoc send envelope)
bwoc council discuss D1 --from agent-yudi --message "Boundary at tool-effect is sound; ratify."

# 3. Each participant votes
bwoc council vote D1 --from agent-luban --option adopt
bwoc council vote D1 --from agent-nezha --option revise

# 4. Resolve once quorum + the voting model are satisfied
bwoc council resolve D1
```

Participants come from the `bwoc team` named at propose time, so a council is built on the same Saṅgha-team substrate as the shared task list. Discussion turns ride the normal signed-envelope messaging (`bwoc send` / `bwoc inbox`), so every turn is trust-gated and auditable like any other inter-agent message.

---

## 5. The Council Decision record

Resolving a council persists a **decision entry** — a normative, `bwoc check`-validated shape. The fields that matter to an operator:

| Field | Meaning |
|---|---|
| `decision_id` | Stable key for the decision — the one durable identifier. |
| `status` | `proposed` / `discussing` / `voting` / `resolved` / `abandoned`. |
| `participants` | Agent ids, fixed at propose time (a change is a *new* decision). |
| `options` | The choices (≥2), also fixed at propose time. |
| `rounds` | Ordered discussion turns, each referencing a `bwoc send` envelope. Append-only. |
| `votes` | One `{participant, option, abstain}` per voter. Append-only. |
| `outcome` | The resolved option — present only once `status = resolved`. |
| `dissent` | Minority positions with rationale — **preserved, never discarded.** |
| `evidence_links` | Referents backing the decision; reuses the audit Evidence kinds. |

Two properties make the record trustworthy: it is **append-only** (rounds and votes accumulate; nothing is rewritten), and **dissent is a first-class field** — recording *why the minority disagreed* is an explicit purpose of the council, not an afterthought.

```json
{
  "decision_id": "D1",
  "status": "resolved",
  "participants": ["agent-yudi", "agent-luban", "agent-nezha"],
  "options": ["adopt", "revise"],
  "votes": [{ "participant": "agent-yudi", "option": "adopt", "abstain": false }],
  "outcome": "adopt",
  "dissent": [{ "participant": "agent-nezha", "option": "revise", "rationale": "wants seccomp in v1" }],
  "opened_at": "2026-06-07T12:00:00Z",
  "closed_at": "2026-06-07T12:30:00Z"
}
```

---

## 6. Binding vs advisory

A resolved council is either **advisory** (the decision is recorded for the trail and informs humans/agents) or **binding** (the fleet has agreed to act). A binding outcome does **not** change anything by itself — it **emits a `bwoc task`** onto the relevant team's list, so the work flows through the normal task lifecycle (claim → plan → approve → complete). The council decides; the task list executes.

> [!warning]
> Never wire a council to mutate state directly on resolve. The separation — decision in the council record, action in a task — is what keeps the audit trail honest: you can always answer "who decided this, and on what evidence?" separately from "who did it."

---

## 7. See also

**Framework source (public GitHub):**

- [PLUGINS.en.md §Council Decision Schema](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md#council-decision-schema) — the normative decision record
- [BWOC-56 council design note](https://github.com/bemindlabs/BWOC-Framework/blob/main/notes/2026-05-28_council-plugin-architecture.md) — protocol detail, voting models, quorum + tie-break, the `council-sangha-7` reference

**Sibling handbook chapters:**

- [../fleet-ops/HANDBOOK.en.md](../fleet-ops/HANDBOOK.en.md) — teams, the shared task list, and fleet health
- [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md) — agents, trust declaration, signed messaging
- [../plugin-craft/HANDBOOK.en.md](../plugin-craft/HANDBOOK.en.md) — how plugin kinds like `council` are built
- [../glossary.en.md](../glossary.en.md) — every term in one table

---

*This handbook is maintained alongside the framework. When behavior described here diverges from the [framework repo](https://github.com/bemindlabs/BWOC-Framework), the repo is correct — please open a fix.*
