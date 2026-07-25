---
title: "ISO Standards Auditing with BWOC"
version: "1.0.0"
audience: "Workspace operators running ISO compliance audits"
language: en
counterpart: HANDBOOK.th.md
---

> Thai version: [HANDBOOK.th.md](HANDBOOK.th.md) — Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) — Plugin source: [modules/plugins](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins)

🇹🇭 [ภาษาไทย](HANDBOOK.th.md)

# ISO Standards Auditing with BWOC

BWOC ships seven `audit`-kind plugins that check a workspace against ISO, ISO/IEC, ISO/IEC/IEEE, and IEEE standards. They run on demand via `bwoc audit run`, write no external state, and emit a structured JSON report the framework validates on every run. This chapter covers what each plugin audits, the evidence model they share, how to configure them, and how to run them.

---

## Contents

1. [Why machine-assisted ISO audit](#1-why-machine-assisted-iso-audit)
2. [The seven audit plugins](#2-the-seven-audit-plugins)
3. [The audit evidence model](#3-the-audit-evidence-model)
4. [Install and enable](#4-install-and-enable)
5. [Configure evidence in workspace.toml](#5-configure-evidence-in-workspacetoml)
6. [Running an audit](#6-running-an-audit)
7. [Governance and security integration](#7-governance-and-security-integration)
8. [See Also](#8-see-also)

---

## 1. Why machine-assisted ISO audit

ISO audits fail for predictable reasons: evidence is scattered across documents, status is unclear until the auditor asks, and gaps are discovered too late to fix. BWOC's `audit` kind addresses this without replacing the human auditor.

The plugins operate on a principle called Musāvāda — no claim without a referent. Every passing finding points at a concrete artifact: a file, a command exit code, an operator-signed attestation, or a sampled rate from a tool. A finding cannot pass because an agent guessed. Gaps surface as `fail` findings on every run, not as surprises the day before an external audit.

What machine assistance provides here is consistency and repeatability. The same `bwoc audit run` command produces the same findings on the same workspace. You can run it in CI, diff the output between sprints, and feed the JSON into dashboards or evidence packages — without manual checklists.

The framework does not impose thresholds or make certification decisions. That remains the auditor's role. The plugins surface reproducible, machine-verified evidence and honest gap reports; the human auditor interprets them.

---

## 2. The seven audit plugins

| Plugin | Standard | What it audits | Evidence kinds |
|---|---|---|---|
| `audit-iso-29110` | ISO/IEC TR 29110 (VSE software lifecycle, Basic profile) | Six Basic-profile work products — Project Plan, SRS, Software Design, Test Plan, Verification Results, Construction Records — by checking for the files in the workspace | `file` |
| `audit-iso-9001` | ISO 9001:2015 (Quality Management System) | Eight headline QMS clauses — organizational context, quality policy, risk actions, competence, documented information, internal audit, management review, corrective action | `attestation` |
| `audit-iso-20000-1` | ISO/IEC 20000-1:2018 (IT Service Management) | Eight ITSM criteria across scope, policy, service catalogue, SLA performance, change management, incident management, problem management, and continual improvement | `attestation` + `sample` |
| `audit-iso-27001` | ISO/IEC 27001:2022 (Information Security Management System) | Five main-body ISMS clauses plus three Annex A controls, with the Statement of Applicability driving Annex A scope | `attestation` + `sample` (SoA-gated) |
| `audit-iso-iec-ieee-29148` | ISO/IEC/IEEE 29148:2018 (Requirements Engineering) | Seven RE criteria — StRS, SyRS/SRS, individual + set requirement characteristics, verifiability, bidirectional traceability, requirements management | `attestation` |
| `audit-iso-iec-ieee-12207` | ISO/IEC/IEEE 12207:2017 (Software Life Cycle Processes) | Nine process criteria — agreement, project planning, assessment & control, configuration management, requirements, architecture/design, implementation/integration, V&V, maintenance | `attestation` |
| `audit-ieee-1012` | IEEE 1012:2016 (Verification & Validation) | Eight V&V criteria — integrity levels, V&V planning (SVVP), independence, and requirements/design/implementation/test V&V + anomaly reporting | `attestation` |

All seven plugins declare `kind = "audit"` in their manifests and emit findings conforming to the [framework's Audit Findings Schema](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md#audit-findings-schema). The `audit` kind's lifecycle owner is the `bwoc audit` CLI — plugins of this kind are never invoked automatically; they run only when the operator calls `bwoc audit run`.

**The kind is body-agnostic.** It began with pure-**ISO** (9001) and joint **ISO/IEC** (27001, 20000-1, 29110), then added joint **ISO/IEC/IEEE** (29148 requirements, 12207 life cycle) and standalone **IEEE** (1012 V&V) — a criterion id, standard designation, and clause reference are data the runtime reads, not a constraint on which body (or how many) publishes the standard. The three software-assurance lanes form a coherent trio: 29148 asks *are the requirements right?*, 12207 *is the life cycle governed?*, and 1012 *is it verified & validated?*

### Plugin summary

**audit-iso-29110** is the most straightforward of the seven. It runs file-existence checks against the Basic-profile work products defined by ISO/IEC TR 29110-5-1-2. The plugin reads a `candidates` list for each criterion — alternate file paths the workspace might use — and passes the first one that exists. No configuration beyond `enabled = true` is needed.

**audit-iso-9001** checks eight headline clauses of ISO 9001:2015. None of these reduce to file-existence — "has management review been held?" is not answerable by looking at a filename. The plugin reads operator-signed attestations from `workspace.toml` and emits an attestation finding for each covered criterion, or a `fail` pointing at the uncovered one.

**audit-iso-20000-1** covers both documented-artifact clauses (scope, policy, service catalogue) and operational-rate clauses (SLA, change, incident, problem, improvement). The documented-artifact criteria use attestations; the operational-rate criteria use samples — a rate the operator transcribes from their ITSM tool (`sampled_count` / `sampled_of` / optional `window`).

**audit-iso-27001** is the most sophisticated of the ISO-management-system four. Its eight criteria split across main-body attestation clauses and Annex A sample-driven controls. The Statement of Applicability (`[[plugins.audit-iso-27001.soa]]`) is machine-readable: it declares which Annex A controls are in scope (`applicable = true`) and provides justification for both inclusions and exclusions. The sampling population (`sampled_of`) for Annex A findings is derived from the SoA — the operator never types it manually.

---

## 3. The audit evidence model

All seven plugins share the [Audit Findings Schema](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md#audit-findings-schema). Every finding has a `criterion_id`, a `severity`, a `status`, and an `evidence` block. A passing finding omits `remedy`; any non-pass finding requires one.

Three evidence kinds are used by the ISO audit plugins:

**`file`** — the evidence is a workspace-relative file path. Used by `audit-iso-29110` (the file must exist) and as a fallback pointer when another evidence kind is missing (e.g., "add your attestation to `.bwoc/workspace.toml`").

**`attestation`** — the evidence is an operator-signed assertion. Used by all three management-system plugins (9001, 20000-1, 27001) for criteria that cannot be verified by machine inspection alone — quality policy ratification, risk assessment completion, ISMS scope definition. Required sub-fields are `signer` (free-text identity, e.g. `"CISO: Suchada N."`) and `signed_at` (ISO 8601 date). An optional `valid_through` date marks when the attestation expires; the framework records it but does not enforce expiry — that is downstream tooling's job.

**`sample`** — the evidence is a measured rate. Used by `audit-iso-20000-1` (all five operational criteria) and `audit-iso-27001` (the three Annex A controls). For `audit-iso-20000-1`, the operator supplies `sampled_count`, `sampled_of`, and an optional `window` directly. For `audit-iso-27001`, `sampled_count` and `sampled_of` are SoA-derived — the plugin computes them from the Statement of Applicability.

### Example: the 27001 evidence triad

The ISO/IEC 27001 plugin uses all three blocks together. Here is an abbreviated but complete picture:

```toml
[plugins.audit-iso-27001]
enabled = true

# Attestation: main-body management-system clauses
[[plugins.audit-iso-27001.attestations]]
criterion_id  = "27001-risk-assessment"
statement     = "Information security risk assessment performed 2026-03-10; methodology documented and results reproducible."
signer        = "CISO: Suchada N."
signed_at     = "2026-03-10"
valid_through = "2027-03-10"

# Statement of Applicability: machine-readable scope for Annex A
[[plugins.audit-iso-27001.soa]]
control       = "A.5.15"
applicable    = true
justification = "Access control is central to protecting source, credentials, and customer data."

[[plugins.audit-iso-27001.soa]]
control       = "A.5.29"
applicable    = false
justification = "No formal continuity programme; risk accepted by management for a solo workspace."

# Samples: Annex A controls in scope (sampled_count/of are SoA-derived — omit them here)
[[plugins.audit-iso-27001.samples]]
criterion_id = "27001-access-control"
summary      = "Access reviews completed across in-scope systems; 0 orphaned accounts found."
window       = "2026-Q1"
```

The runtime routes each criterion by its `expected_evidence_kind`:
- `27001-risk-assessment` is an `attestation` criterion — it checks for a matching `[[…attestations]]` entry.
- `27001-access-control` is a `sample` criterion — it looks up `A.5.15` in the SoA, confirms `applicable = true`, computes `sampled_of` from the count of in-scope controls, then checks for a `[[…samples]]` entry.
- If `A.5.29` is excluded (`applicable = false`) and has a justification, the finding is `status = "not_applicable"` — not a pass, not a fail.

A criterion absent from the workspace evidence emits `status = "fail"` with a `remedy` naming the exact block to add in `workspace.toml`.

---

## 4. Install and enable

The seven ISO/IEC/IEEE audit plugins are bundled with the framework under `modules/plugins/`. No separate install step is needed; they are present whenever the framework is available.

To enable one, add its block to `.bwoc/workspace.toml`:

```toml
[plugins.audit-iso-29110]
enabled = true

[plugins.audit-iso-9001]
enabled = true

[plugins.audit-iso-20000-1]
enabled = true

[plugins.audit-iso-27001]
enabled = true
```

`enabled = true` is the only manifest-declared key for all seven plugins. All evidence configuration is in per-plugin array-of-tables blocks (see the next section) — there is no `[config.schema]` in any of their manifests.

To disable a plugin without removing its evidence blocks:

```bash
bwoc plugin disable audit-iso-27001
```

This sets `enabled = false` in `workspace.toml` and keeps the `[[…attestations]]` / `[[…soa]]` / `[[…samples]]` blocks intact.

---

## 5. Configure evidence in workspace.toml

Each plugin reads its evidence from `.bwoc/workspace.toml`. The shapes differ by plugin.

### audit-iso-29110

No evidence configuration. The plugin reads `criteria.toml` from its own directory and probes the workspace for each criterion's `candidates` list. Set `enabled = true` and run.

### audit-iso-9001

Eight attestation criteria. Provide one `[[plugins.audit-iso-9001.attestations]]` block per criterion you can attest. Criteria without an attestation emit `fail`.

```toml
[[plugins.audit-iso-9001.attestations]]
criterion_id  = "9001-management-review"
statement     = "Management review held 2026-04-15 covering Q1 QMS performance, customer feedback, and improvement opportunities."
signer        = "Quality Manager: Tonkla K."
signed_at     = "2026-04-15"
valid_through = "2027-04-15"   # optional
```

The eight criterion IDs are: `9001-context-of-organization`, `9001-leadership-and-policy`, `9001-risks-and-opportunities`, `9001-competence-and-awareness`, `9001-documented-information`, `9001-internal-audit`, `9001-management-review`, `9001-corrective-action`.

### audit-iso-20000-1

Three attestation + five sample criteria. Attestation criteria use `[[plugins.audit-iso-20000-1.attestations]]`; sample criteria use `[[plugins.audit-iso-20000-1.samples]]`.

```toml
[[plugins.audit-iso-20000-1.attestations]]
criterion_id = "20000-1-service-policy-and-objectives"
statement    = "Service management policy v2.1 ratified 2026-01-15; objectives reviewed quarterly."
signer       = "Service Owner: Tonkla K."
signed_at    = "2026-01-15"

[[plugins.audit-iso-20000-1.samples]]
criterion_id  = "20000-1-incident-management"
summary       = "49 of 50 incidents resolved within SLA"
sampled_count = 49
sampled_of    = 50
window        = "2026-Q1"
```

For sample criteria, `sampled_count` and `sampled_of` are required integers; `window` is optional. The runtime does not enforce a pass threshold — it records the rate and surfaces it for the human auditor.

The attestation criterion IDs: `20000-1-service-management-system-scope`, `20000-1-service-policy-and-objectives`, `20000-1-service-catalogue`. The sample criterion IDs: `20000-1-service-level-management`, `20000-1-change-management`, `20000-1-incident-management`, `20000-1-problem-management`, `20000-1-continual-improvement`.

### audit-iso-27001

Five attestation + three SoA-gated sample criteria. Three blocks are used: `[[…attestations]]`, `[[…soa]]`, and `[[…samples]]`.

The `[[…soa]]` block is the Statement of Applicability. Every Annex A control the plugin checks (`A.5.15`, `A.5.24`, `A.5.29`) must appear here with `applicable` and `justification`. ISO/IEC 27001 clause 6.1.3 requires justification for both inclusions and exclusions.

For `[[…samples]]` blocks under 27001, do **not** supply `sampled_count` or `sampled_of` — the runtime derives them from the SoA. Supply only `criterion_id`, `summary`, and optionally `window`.

The attestation criterion IDs: `27001-isms-scope`, `27001-information-security-policy`, `27001-risk-assessment`, `27001-statement-of-applicability`, `27001-internal-audit`. The sample (Annex A) criterion IDs: `27001-access-control` (A.5.15), `27001-incident-management` (A.5.24), `27001-business-continuity` (A.5.29).

---

## 6. Running an audit

```bash
# Run one plugin
bwoc audit run --plugin audit-iso-29110

# Run all enabled audit plugins
bwoc audit run

# JSON output (machine-parseable)
bwoc audit run --plugin audit-iso-27001 --json
```

The dispatcher spawns each plugin's `audit.sh` with three environment variables: `BWOC_WORKSPACE` (absolute path to the workspace root), `BWOC_PLUGIN_DIR` (absolute path to the plugin directory), and `BWOC_AUDIT_OPERATION` (always `audit_run`). The plugin writes a JSON findings array to stdout. The framework wraps it into an envelope:

```json
{
  "workspace": "/path/to/workspace",
  "runs": [
    {
      "plugin":      "audit-iso-27001",
      "version":     "0.2.0",
      "started_at":  "2026-06-07T10:00:00Z",
      "finished_at": "2026-06-07T10:00:01Z",
      "findings":    [ ... ]
    }
  ],
  "summary": {
    "fail_count": 2,
    "framework_error": false
  }
}
```

### Exit codes

| Code | Meaning |
|---|---|
| `0` | No `fail` findings |
| `1–254` | Count of `fail` findings (clamped at 254; exact count in `summary.fail_count`) |
| `255` | Framework or plugin error — the report cannot be trusted |
| `2` | Operator/usage error — no workspace found, or named plugin not installed |

`not_applicable` and `not_implemented` findings do not count as failures. Use `$?` in CI to gate on audit results without parsing stdout.

### Reading the output

Each finding in `--json` output carries:

- `criterion_id` — the stable identifier (e.g. `27001-risk-assessment`). These are stable across plugin versions; renaming one is a major version bump.
- `severity` — `info`, `low`, `medium`, `high`, or `critical`. This is the criterion's intrinsic importance, not the outcome. A `critical` finding with `status = "pass"` means the most important check passed.
- `status` — `pass`, `fail`, `not_applicable`, or `not_implemented`.
- `evidence` — where the plugin looked. A `pass` finding always has a concrete referent here.
- `remedy` — present on any non-pass finding. Actionable: names exactly what to add to `workspace.toml`, or what file to create.

---

## 7. Governance and security integration

The seven plugins cover different governance layers and compose well with the rest of BWOC.

**ISO/IEC 29110** is the foundation layer for any BWOC workspace used for software development. The six work products it checks — Project Plan, SRS, Design, Test Plan, Verification Results, Construction Records — map directly onto the documentation that agents produce and reference. An `audit-iso-29110` run gives you a gap report on your project artifacts.

**ISO 9001** covers the management-system layer — whether the organization operating the workspace has documented its quality practices and holds leadership accountable. This complements the agent-level governance provided by BWOC's Aparihāniya-dhamma 7 fleet health signals (see the [glossary](../glossary.en.md)).

**ISO/IEC 20000-1** covers IT service delivery. For workspaces where agents are part of a service team (incident response, change management, SLA tracking), this plugin surfaces whether the service management practices are documented and measured.

**ISO/IEC 27001** is the information-security layer. Its machine-readable Statement of Applicability in `workspace.toml` is a live record of control scope decisions. Combined with the framework's agent-level threat model (see the [security chapter](../security/HANDBOOK.en.md) if present in your handbook build), it provides an integrated security posture view.

Because all seven plugins are read-only — they inspect the workspace and emit a report, writing nothing — they carry no operator-confirmation gate. You can run them as frequently as you like without side effects.

---

## 8. See Also

- [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) — the framework plugin spec; Audit Findings Schema, evidence kinds, exit codes.
- Plugin source — each plugin's `SPEC.md` has the full criteria table and configuration reference:
  - [audit-iso-29110](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/audit-iso-29110)
  - [audit-iso-9001](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/audit-iso-9001)
  - [audit-iso-20000-1](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/audit-iso-20000-1)
  - [audit-iso-27001](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/audit-iso-27001)
- [../glossary.en.md](../glossary.en.md) — Pali term lookup (Musāvāda, Aparihāniya-dhamma 7, Samānattatā).
- [../developer/HANDBOOK.en.md](../developer/HANDBOOK.en.md) — Plugin development; how to author a new `audit`-kind plugin.

---

*Back to handbook index: [`../README.md`](../README.md)*
