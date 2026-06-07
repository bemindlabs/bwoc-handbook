# OKRs with BWOC

🇹🇭 [ภาษาไทย](HANDBOOK.th.md)

For **workspace operators and team leads** who want to track goals alongside agent work — not just in a separate spreadsheet, but inside the same workspace the framework already manages.
If you have not set up a workspace yet, start at [`../quickstart/HANDBOOK.en.md`](../quickstart/HANDBOOK.en.md).
If you want to understand the broader plugin system, see the [PLUGINS spec](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md).
Term lookup: [`../glossary.en.md`](../glossary.en.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)

---

## Table of Contents

1. [OKRs and why BWOC carries them](#1-okrs-and-why-bwoc-carries-them)
2. [What the okr plugin gives you](#2-what-the-okr-plugin-gives-you)
3. [Install and enable](#3-install-and-enable)
4. [Author your objectives and key results](#4-author-your-objectives-and-key-results)
5. [The three verbs](#5-the-three-verbs)
6. [Worked example](#6-worked-example)
7. [OKRs and fleet governance](#7-okrs-and-fleet-governance)
8. [See Also](#8-see-also)

---

## 1. OKRs and why BWOC carries them

**OKRs** (Objectives and Key Results) are a goal-setting method. An *objective* is a qualitative statement of direction ("ship the OKR plugin kind"). A *key result* is a measurable outcome that tells you whether the objective is being met ("reference plugin ships and all three verbs run"). You set targets, track current values, and check confidence.

In most teams OKRs live in a separate tool — a spreadsheet, a project management app — disconnected from the code and agents that are actually doing the work. That gap has a cost: you cannot easily point a goal at a specific file, commit, or agent task, and you cannot run a script that queries progress without leaving the workspace.

BWOC closes that gap with the `okr` plugin kind. Your objectives and key results live in two plain TOML files inside the workspace. The `bwoc okr` CLI lets you update progress, check status, and generate a full progress report — all without leaving the terminal. Evidence for each key result is a workspace-relative file path, a command output, or an attestation — the same evidence vocabulary the `audit` plugin uses, so there is one way to say "here is my proof" across the whole framework.

The `okr` plugin is **local-file-only**. It reads and writes TOML in your workspace; it reaches no external system, holds no credentials, and emits no network traffic.

---

## 2. What the okr plugin gives you

The reference plugin is `workspace-okrs` (`kind = "okr"`, current version `0.1.0`, requires framework `>=2.9.0`).

It provides:

- **`objectives.toml`** — where you write your objectives (one `[[objective]]` block per objective).
- **`key_results.toml`** — where you write your key results (one `[[key_result]]` block per key result), each referencing a parent objective.
- **Three verbs** via `bwoc okr`: `track`, `check-progress`, and `report`.
- A normative **OKR Progress Schema** JSON output (the same shape `bwoc check` validates against).
- A **check-progress heuristic** that rolls up per-KR status (on-track / at-risk / off-track) into a per-objective summary.

What it deliberately does **not** do:

- No network calls, no external system of record.
- No operator-confirmation gate on `track` — because the only write is to your own local TOML file, which you can `git diff` at any time.
- No time-phased "expected attainment by period elapsed" model in v1 — status uses a flat 0.7 attainment threshold (the canonical OKR green line).

---

## 3. Install and enable

The plugin ships alongside the framework under `modules/plugins/okr/workspace-okrs/`. If your workspace was initialized from the framework, it may already be present. Check with:

```bash
bwoc plugin list
```

If it is not listed, install it:

```bash
bwoc plugin install ./modules/plugins/okr/workspace-okrs/
bwoc plugin enable workspace-okrs
```

After enabling, your `workspace.toml` gains this entry:

```toml
[plugins.workspace-okrs]
enabled = true
```

That is the only required configuration. The `workspace-okrs` plugin takes no additional config keys in v1 — it reads its data from the TOML files inside the plugin directory.

Verify installation:

```bash
bwoc plugin show workspace-okrs
```

The plugin requires `jq` on `PATH`. If `jq` is missing, the plugin exits with a clear error message pointing you to install it.

---

## 4. Author your objectives and key results

The plugin ships with seed data (the EPIC-4 delivery objective used to track the plugin's own development). Replace it with your own goals.

### `objectives.toml`

One `[[objective]]` block per objective:

```toml
[[objective]]
objective_id = "O1"
title        = "Ship the agent memory skill to production"
owner        = "agent-jisoo"
period       = "2026-Q3"
parent       = ""
```

| Field | Required | Notes |
|---|---|---|
| `objective_id` | yes | Stable string id. Key results reference this. Renaming breaks referential integrity — `bwoc check` will catch dangling references. |
| `title` | yes | One-line statement of the objective. |
| `owner` | yes | An agent id or a person's name. |
| `period` | yes | The scoping window, e.g. `2026-Q3`. |
| `parent` | no | Parent `objective_id` for nested objectives. Use `""` for top-level. Multi-level rollup is deferred to a future version. |

### `key_results.toml`

One `[[key_result]]` block per key result:

```toml
[[key_result]]
key_result_id = "O1-KR1"
objective_id  = "O1"
description   = "Memory skill published and bwoc check passes on all agents"
target        = 1
current       = 0
unit          = "count"
confidence    = "medium"
evidence      = { kind = "none", value = "" }
```

| Field | Required | Notes |
|---|---|---|
| `key_result_id` | yes | Stable string id, unique within the plugin. |
| `objective_id` | yes | Must resolve to an `objectives.toml` id. `bwoc check` validates this. |
| `description` | yes | What the key result measures. |
| `target` | yes | The goal value (number). |
| `current` | yes | Latest tracked value. Updated by `track`. |
| `unit` | yes | One of `count`, `percent`, `currency`, `ratio`, `boolean`. |
| `confidence` | yes | One of `high`, `medium`, `low` — your qualitative read of whether the trajectory holds. |
| `evidence` | yes | `{ kind, value }` using the standard evidence kinds: `file`, `content`, `command`, `attestation`, `sample`, or `none`. |
| `as_of` | no | ISO-8601 date the `current` value was last tracked. Omit the line when never tracked. Written automatically by `track`. |

**`boolean` unit note:** use `0` for false and `1` for true. A target of `1` and a current of `1` means met.

---

## 5. The three verbs

All three verbs are reached through `bwoc okr`. They can also be invoked by hand directly against the entry script for smoke tests.

### `track` — record a new current value

```bash
bwoc okr track --key-result O1-KR1 --current 1 --evidence "file:docs/en/MEMORY.en.md"
```

Updates the `current` field (and optionally `evidence`) for a specific key result in `key_results.toml`, then stamps `as_of` with today's date. Returns the updated key result as a single progress entry.

| Flag | Required | Notes |
|---|---|---|
| `--key-result <id>` | yes | The `key_result_id` to update. |
| `--current <value>` | yes | New numeric value. Can exceed `target` — over-attainment is preserved. |
| `--evidence <kind:value>` | no | Evidence for this check-in. Format: `kind:value`, e.g. `file:crates/bwoc-cli/src/memory.rs` or `none:`. |

`track` is idempotent: tracking a key result to the same `current` + `evidence` rewrites the same bytes (only `as_of` advances to today). It is safe to retry after a transient failure.

`track` carries **no confirmation gate** — the write touches only your own local TOML file. If you track the wrong value, `git diff` will show it and `git checkout` reverts it.

### `check-progress` — per-KR and per-objective status

```bash
bwoc okr check-progress
```

Read-only. Returns a JSON object with:

- Per-KR attainment and status (`on-track`, `at-risk`, `off-track`).
- Per-objective rollup, with counts and worst-KR status.

Status heuristic:

```
attainment = current / target
  (for boolean: current >= target → 1, else 0; target 0 → met)

on-track  — attainment >= 0.7  OR  confidence == "high"
at-risk   — otherwise, when confidence == "medium"
off-track — otherwise (confidence == "low")
```

Objective status rolls up to the **worst** KR status across all key results for that objective.

### `report` — full OKR Progress Schema JSON

```bash
bwoc okr report
```

Read-only. Returns a JSON array of all key results as progress entries, each conforming to the [OKR Progress Schema](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md#okr-progress-schema). This is the canonical wire format consumed by `bwoc check` validation and any downstream tooling.

Example output element:

```json
{
  "objective_id":  "O1",
  "key_result_id": "O1-KR1",
  "target":        1,
  "current":       1,
  "unit":          "count",
  "confidence":    "high",
  "evidence":      { "kind": "file", "value": "docs/en/MEMORY.en.md" },
  "as_of":         "2026-06-07"
}
```

A key result that has never been tracked omits `as_of` entirely (not `null`).

---

## 6. Worked example

Suppose you are shipping a new agent skill and want to track delivery as OKRs.

**Step 1 — Author the objective.**

```toml
# objectives.toml
[[objective]]
objective_id = "O1"
title        = "Ship the memory-tier2 skill to all agents"
owner        = "agent-jisoo"
period       = "2026-Q3"
parent       = ""
```

**Step 2 — Author three key results.**

```toml
# key_results.toml
[[key_result]]
key_result_id = "O1-KR1"
objective_id  = "O1"
description   = "Skill spec merged and bwoc check passes"
target        = 1
current       = 0
unit          = "count"
confidence    = "medium"
evidence      = { kind = "none", value = "" }

[[key_result]]
key_result_id = "O1-KR2"
objective_id  = "O1"
description   = "All 6 registered agents have the skill enabled"
target        = 6
current       = 0
unit          = "count"
confidence    = "medium"
evidence      = { kind = "none", value = "" }

[[key_result]]
key_result_id = "O1-KR3"
objective_id  = "O1"
description   = "Memory retrieval latency under 200ms (sample over 50 queries)"
target        = 1
current       = 0
unit          = "boolean"
confidence    = "low"
evidence      = { kind = "none", value = "" }
```

**Step 3 — Check initial state.**

```bash
bwoc okr check-progress
```

All three key results show `at-risk` or `off-track` because `current` is 0. The objective rolls up to `off-track`. This is expected — you just started.

**Step 4 — Work. When KR1 ships, track it.**

```bash
bwoc okr track \
  --key-result O1-KR1 \
  --current 1 \
  --evidence "file:modules/skills/memory-tier2/SPEC.md"
```

Output: the updated KR1 progress entry with `current: 1`, `as_of: 2026-06-07`.

**Step 5 — As agents are updated, track KR2 incrementally.**

```bash
bwoc okr track --key-result O1-KR2 --current 3
bwoc okr track --key-result O1-KR2 --current 6 --evidence "command:bwoc list --skill memory-tier2 --count"
```

**Step 6 — When latency evidence is in, track KR3.**

```bash
bwoc okr track \
  --key-result O1-KR3 \
  --current 1 \
  --evidence "sample:49 of 50 queries under 200ms"
```

**Step 7 — Generate the final report.**

```bash
bwoc okr report
```

Pipe to `jq` for filtering:

```bash
bwoc okr report | jq '.[] | select(.confidence == "low")'
```

---

## 7. OKRs and fleet governance

OKRs in BWOC are not just a reporting tool — they are a governance layer for the fleet.

**Assigning objectives to agents.** The `owner` field of an objective accepts an agent id (e.g. `agent-jisoo`). This is a naming convention, not an automated assignment: the `bwoc okr` CLI does not spawn agent tasks automatically. The connection is intentional and human: you set the objective, name the owner, and separately assign tasks through `bwoc task add`.

**Evidence ties goals to artifacts.** Every key result carries an `evidence` field. Using `kind = "file"` with a workspace-relative path means the proof is a real file you can open and verify — not a free-text claim. This is the same **Musāvāda** guard the `audit` plugin uses: no claim without a referent.

**OKRs and audit results complement each other.** The `audit` plugin checks whether the workspace meets a standard (ISO, license headers, doc parity). The `okr` plugin tracks whether operator-set goals are being met. Running both gives you a compliance view and a goals view in the same terminal.

**Integrating with teams.** When a team owns an objective, list the team name in `owner`. Use `bwoc team list` to see available teams and `bwoc task add <team> "..."` to create tasks aligned with the key results. The OKR data does not auto-sync to team task lists — you connect them by naming conventions and manual task creation.

**Version control.** Because `objectives.toml` and `key_results.toml` are plain TOML files in the workspace, they live in git. Every `track` update is a git-diffable change. A `git log` on `key_results.toml` is a lightweight audit trail of every check-in.

---

## 8. See Also

**Framework documentation (public GitHub)**

- [PLUGINS.en.md — OKR Progress Schema](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md#okr-progress-schema) — normative schema for the `report` output
- [PLUGINS.en.md — okr kind](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md#plugin-kinds) — kind definition, lifecycle owner, design rationale
- [workspace-okrs plugin source](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/okr) — `manifest.toml`, `SPEC.md`, `okr.sh`, seed data files

**Sibling handbook chapters**

- [`../fleet-ops/HANDBOOK.en.md`](../fleet-ops/HANDBOOK.en.md) — fleet health, supervise, visibility; the operational complement to OKR governance
- [`../agents/HANDBOOK.en.md`](../agents/HANDBOOK.en.md) — incarnating agents; `bwoc check`; how plugins relate to agents
- [`../glossary.en.md`](../glossary.en.md) — term definitions

---

> **Source of truth.** This handbook summarizes and orients. On any conflict, the framework repo wins and this page has a bug — please fix it. Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework).
