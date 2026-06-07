---
title: Plugin Authoring — Extending the Framework
nav_order: 6
---

# Plugin Authoring — Extending the Framework

> Thai version: [HANDBOOK.th.md](HANDBOOK.th.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) · Glossary: [../glossary.en.md](../glossary.en.md) · Developer: [../developer/HANDBOOK.en.md](../developer/HANDBOOK.en.md)

**Source of truth.** The canonical specification lives in the framework repo. On any conflict, the repo wins and this handbook has a bug. Relevant docs: [`PLUGINS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) (the full plugin spec) · [`SKILLS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md).

---

## Table of Contents

1. [Skill or plugin?](#1-skill-or-plugin)
2. [The plugin kinds](#2-the-plugin-kinds)
3. [Anatomy of a plugin](#3-anatomy-of-a-plugin)
4. [The lifecycle — `bwoc plugin`](#4-the-lifecycle--bwoc-plugin)
5. [Normative schemas — the contract](#5-normative-schemas--the-contract)
6. [Write verbs and the operator-confirm gate](#6-write-verbs-and-the-operator-confirm-gate)
7. [What a plugin is not](#7-what-a-plugin-is-not)
8. [See also](#8-see-also)

---

## 1. Skill or plugin?

Skills and plugins share a substrate — a TOML manifest, the backend-neutrality gate, per-workspace opt-in — and split on **who turns them on**.

| | Skill | Plugin |
|---|---|---|
| Audience | Agent author | Workspace operator |
| Opt-in via | `<agent>/config.manifest.json` | `workspace.toml [plugins]` |
| Invoked by | The agent during its own operation | The framework runtime |
| Scope | Per-agent | Per-workspace |
| Example | worktree discipline, bilingual-parity check | Tier 2 memory backend, an extra LLM backend |

The rule of thumb: **if an individual agent's logic calls it, it is a skill; if the workspace loads it once for everyone, it is a plugin.** This chapter is about plugins; for skills, see [`SKILLS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md).

---

## 2. The plugin kinds

Every plugin declares exactly one `kind`. The kind defines which lifecycle hooks the framework calls. A plugin sets `kind` once — **cross-kind plugins are not supported; split them.**

| Kind | What it extends |
|---|---|
| `memory-backend` | Tier 2 memory (semantic search / vector store) |
| `llm-backend` | Backends beyond the six declared ones |
| `workflow` | External integrations (issue trackers, CI, code review) |
| `audit` | Inspecting the workspace against a standard (ISO/IEC 29110, 9001, 27001…) |
| `jira` | Bidirectional sync with Jira Cloud — the first **write-capable** kind |
| `okr` | Tracking operator-authored Objectives + Key Results |
| `council` | A structured decision protocol among fleet agents (see [Council](../council/HANDBOOK.en.md)) |
| `figma` | Read-mostly Figma REST integration; design→dev bridge |
| `gws` | Read-mostly Google Workspace (Drive / Gmail / Calendar) |

> [!note]
> The deciding principle for a *new* kind: **own-kind when BWOC defines a normative schema over the integration; reuse `workflow` when it is a passthrough with no BWOC-owned shape.** `figma` and `jira` earn their own kinds because they carry normative mapping schemas; a thin shell-out to some CLI does not.

---

## 3. Anatomy of a plugin

A plugin is a directory with a **`manifest.toml`** at its root. The manifest declares identity, the `kind`, the verbs the plugin exposes, and a JSON-schema-lite description of the config keys the workspace may set:

```toml
name = "my-audit"
kind = "audit"
version = "0.1.0"

# Config the operator sets in workspace.toml [plugins.my-audit];
# validated against this block at load time.
[config]
storage_path = { type = "string",  required = false, default = "memories/tier2" }
max_results  = { type = "integer", required = true }
```

The operator then opts in from `workspace.toml`:

```toml
[plugins.my-audit]
max_results = 50
```

The framework validates the workspace's `[plugins.<name>]` table against the manifest's `[config]` block — a missing required key or a wrong type fails the load, not the run.

---

## 4. The lifecycle — `bwoc plugin`

Authoring and operating a plugin runs through one command family:

```bash
bwoc plugin init my-audit --kind audit   # scaffold a manifest + stub
bwoc plugin verify my-audit              # neutrality + manifest + schema checks
bwoc plugin install ./my-audit           # register into the workspace
bwoc plugin enable my-audit              # turn on for this workspace
bwoc plugin list                         # what's installed + enabled
bwoc plugin show my-audit                # manifest + config in effect
bwoc plugin disable my-audit             # turn off without removing
bwoc plugin remove my-audit              # unregister
```

> [!tip]
> Run `bwoc plugin verify` before every install. It runs the same backend-neutrality gate as `bwoc check` — a plugin that hardcodes a vendor name or model id fails here, so you catch it before it reaches a workspace.

---

## 5. Normative schemas — the contract

A plugin's value is its **output contract**. The reporting and coordination kinds each carry a normative schema the framework validates at every `invoke` boundary — an unknown enum value is a *plugin bug that fails the run*, not a finding the operator must triage. The shipped schemas:

- **Audit Findings** — `{criterion_id, severity, status, evidence, remedy}`; `severity` and `status` are closed enums; `evidence` must be reproducible (the **Musāvāda** guard — no claim without a referent).
- **Jira Issue Mapping**, **OKR Progress**, **Council Decision**, **Figma Asset Mapping**, **Workspace Resource** — each a normative shape for its kind.

A shared rule across all of them: **optional fields are omitted when absent, never serialized as `null`**, and identifiers are **stable across releases** — renaming a `criterion_id` or keying on a mutable field is a breaking change.

---

## 6. Write verbs and the operator-confirm gate

Most plugin verbs **read** and carry no gate — they are free. A **write verb** — one whose `invoke` produces a durable side-effect outside the plugin's own address space — carries a **normative operator-confirm gate**:

1. **The gate lives at the operator boundary** — the `bwoc <cli>` command, not the plugin. One confirmation per write; the plugin does not re-implement or bypass it.
2. **It shows the exact effect before acting** — target, current state, and the literal command that will run (with `--` so user values can't be parsed as flags).
3. **Default is No.** An interactive operator answers `y/N`; a headless agent must pass an explicit `--yes`, set **only** when the operator authorized that specific action.
4. **A refused write reports "no change" with the reason** (Dhammānupassanā) — never a silent write, never a bare failure.

> [!warning]
> Destructive, irreversible verbs (e.g. `delete`) are held to a higher bar than reversible writes like `start`/`stop`, and are introduced deliberately per-slice with their own stronger gate — not shipped just because they sit next to a reversible write.

---

## 7. What a plugin is not

- **Not a loophole for vendor-specific framework logic.** The six declared backends are first-class and live in spec. Vendor phrasing is still forbidden (**Samānattatā**).
- **Not a place for one-off scripts.** Those belong with the agent that uses them.
- **Not skills with extra steps.** If an agent calls it during its own operation, it is a skill.

---

## 8. See also

**Framework source (public GitHub):**

- [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) — the full plugin specification: every kind, schema, and the manifest format
- [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) — the skill layer, for agent-author extensions

**Sibling handbook chapters:**

- [../developer/HANDBOOK.en.md](../developer/HANDBOOK.en.md) — the crate map and how the runtime loads extensions
- [../council/HANDBOOK.en.md](../council/HANDBOOK.en.md) — the `council` kind in depth
- [../okr/HANDBOOK.en.md](../okr/HANDBOOK.en.md) and [../iso-audit/HANDBOOK.en.md](../iso-audit/HANDBOOK.en.md) — worked examples of `okr` and `audit` plugins
- [../glossary.en.md](../glossary.en.md) — every term in one table

---

*This handbook is maintained alongside the framework. When behavior described here diverges from the [framework repo](https://github.com/bemindlabs/BWOC-Framework), the repo is correct — please open a fix.*
