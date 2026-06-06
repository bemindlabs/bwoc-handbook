# Security with BWOC — Threat Modeling, Baseline Safety, and the Tianting Team Pattern

> Language toggle: **English (canonical)** | [ภาษาไทย](HANDBOOK.th.md)
>
> This chapter is for anyone operating BWOC agents in a shared workspace — whether you are running a single agent or coordinating a whole security fleet. It covers how BWOC models threats, what the always-on safety floor looks like, how agents verify each other, and how to assemble a real 8-agent security team using the tianting (天庭) pattern.

---

## Table of Contents

1. [A note on Pali terms in this chapter](#1-a-note-on-pali-terms-in-this-chapter)
2. [The threat model — Three Cravings (Taṇhā 3)](#2-the-threat-model--three-cravings-taṇhā-3)
3. [The safety floor — Five Precepts (Sīla 5)](#3-the-safety-floor--five-precepts-sīla-5)
4. [Audit logging — Three Doors of Action (Kamma 3)](#4-audit-logging--three-doors-of-action-kamma-3)
5. [Inter-agent trust and message signing](#5-inter-agent-trust-and-message-signing)
6. [Fleet health — Seven Non-Decline Principles (Aparihāniya-dhamma 7)](#6-fleet-health--seven-non-decline-principles-aparihāniya-dhamma-7)
7. [The tianting team — 8 incarnated security agents](#7-the-tianting-team--8-incarnated-security-agents)
   - [7.1 Role table](#71-role-table)
   - [7.2 The incident loop](#72-the-incident-loop)
8. [Running the tianting team](#8-running-the-tianting-team)
   - [8.1 Create the team](#81-create-the-team)
   - [8.2 Day-to-day task flow](#82-day-to-day-task-flow)
   - [8.3 Fleet health checks](#83-fleet-health-checks)
9. [Red team is authorized-use only](#9-red-team-is-authorized-use-only)
10. [See also](#10-see-also)

---

## 1. A note on Pali terms in this chapter

BWOC uses Pali-language labels as engineering vocabulary shortcuts — not religious instruction. Every Pali term here has a plain-language meaning that is stated first. The label is parenthetical. You do not need any religious background to follow this chapter.

Quick decoder for the three core labels you will see most often:

- **Taṇhā 3** (Three Cravings) = the three threat categories: attacks that manipulate, attacks that persist, attacks that destroy.
- **Sīla 5** (Five Precepts) = the five non-negotiable safety rules every agent must follow unconditionally.
- **Kamma 3** (Three Doors of Action) = the three audit-log channels: what an agent did, what it said, what it intended.

For a complete glossary, see [../glossary.en.md](../glossary.en.md) and the [BWOC Glossary](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md).

---

## 2. The threat model — Three Cravings (Taṇhā 3)

BWOC organizes every security threat into three categories derived from the Three Cravings — the three ways a system is pulled toward harmful behavior:

| Threat category | Pali label | Plain description | Example attacks |
|---|---|---|---|
| **Craving for stimulus** | Kāma-taṇhā | An agent is manipulated into acting on harmful instructions it should have refused | Prompt injection via crafted user input; social engineering by a malicious peer agent; jailbreaking through role-playing frames; instruction-override payloads in documents the agent reads |
| **Craving to persist** | Bhava-taṇhā | An agent or process tries to expand its own footprint, prolong its existence, or acquire capabilities beyond its declared scope | Privilege escalation; persisting a daemon the operator did not authorize; self-modifying `AGENTS.md` to expand scope; acquiring secrets or credentials outside the declared skill set |
| **Craving to destroy** | Vibhava-taṇhā | An agent takes irreversible destructive actions, whether caused by bad instructions, a bug, or deliberate attack | Bulk deletion of files or database rows; overwriting production config; exfiltrating and deleting data; making changes that cannot be rolled back |

**How to use this table.** When you are threat-modeling a new agent or reviewing an incident, start by asking: was this a manipulation attack (Kāma), a persistence/escalation attack (Bhava), or a destruction attack (Vibhava)? Each category implies a different mitigation. Manipulation threats are countered by strict instruction validation, anti-scope declarations, and trust-gated messaging. Persistence threats are countered by scope enforcement, capability declarations, and audit trails. Destruction threats are countered by requiring reversibility, dry-run modes, and supervisor approval before irreversible actions.

The full threat model specification, including numbered threat IDs (T-1.x, T-2.x, T-3.x) and mitigations, lives at [THREAT-MODEL.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/THREAT-MODEL.en.md).

---

## 3. The safety floor — Five Precepts (Sīla 5)

The Five Precepts are the always-on safety floor. They are non-negotiable. They cannot be suspended by a user instruction, overridden by a peer agent's message, or disabled by a configuration flag. They apply before any feature work, before any task begins, before any optimization is considered.

| Precept | Engineering rule | What it prevents |
|---|---|---|
| **No harm** | Do not execute destructive commands (`rm -rf` of project root or production data, mass-delete queries, format operations on live disks) without a supervisor-approved dry run and explicit rollback plan | Vibhava-taṇhā — destruction attacks |
| **No taking what is not given** | Do not read, copy, or transmit files, secrets, credentials, or data that are outside the agent's declared scope | Bhava-taṇhā — unauthorized access and exfiltration |
| **No improper conduct** | Do not spoof another agent's identity; do not forge envelope signatures; do not impersonate the user | Both Bhava-taṇhā (persistence via identity theft) and Kāma-taṇhā (manipulation via false authority) |
| **No false speech** | Do not produce outputs the agent knows to be misleading; do not overclaim capability maturity; do not suppress relevant errors or warnings from reports | Kāma-taṇhā — manipulation through deception |
| **No heedlessness** | Do not skip verification gates (lint, format, test, build); do not bypass `bwoc check`; do not mark a task complete before its completion criteria are satisfied | All three threat categories — heedlessness enables every category of attack |

These five rules are encoded in AGENTS.md Section 9 (Baseline Security — Sīla 5) of the agent template. `bwoc check` does not evaluate precept compliance at runtime — that is the agent author's and operator's responsibility — but any violation discovered during an audit should be filed as a severity-1 incident with yanluo (see Section 7).

---

## 4. Audit logging — Three Doors of Action (Kamma 3)

The Three Doors of Action is the BWOC framework for what to log and where. Every intentional act can be traced through three channels:

| Door | What it records | Where in BWOC |
|---|---|---|
| **Action** (kāya-kamma) | File operations, shell commands, commits — physical changes to the system | `task-log.jsonl` (append-only JSONL), git history, `bwoc log` daemon output |
| **Speech** (vacī-kamma) | Messages sent to other agents or users, API calls, alert notifications | `interconnect/` envelope log, `.bwoc/inbox.jsonl`, outbound message archive |
| **Intent** (mano-kamma) | Plans, decisions, and the reasoning behind actions — before execution | Task plan entries in `bwoc task plan`, PRD comments, the "Plan:" prefix in task-log |

In practice, this means a complete audit trail for any agent action looks like:

```
intent:  bwoc task plan tianting TASK-007 "Plan: 1) isolate affected host 2) check lateral movement 3) patch"
action:  git commit -m "fix: patch CVE-2026-1234 in auth middleware"
speech:  bwoc send yanluo "CVE-2026-1234 patched; awaiting confirmation" --from agent-laojun
```

When yanluo (the audit/reporting agent) generates compliance reports, it pulls from all three channels. An action without an intent record is a finding. A plan that was approved but not acted on is a finding. A message with no corresponding action is a finding.

---

## 5. Inter-agent trust and message signing

### Why trust gates exist

In a multi-agent workspace, any agent can send a message to any other agent. Without gates, a compromised or misconfigured agent could inject arbitrary instructions into the inbox of a high-privilege agent. Trust gates are the structural answer to Kāma-taṇhā attacks coming from within the fleet.

### The seven trust qualities (Kalyāṇamitta-7)

Each agent declares seven boolean qualities about itself. Peer agents can require a minimum set of qualities from any sender before accepting messages. The seven qualities are:

| Quality | Plain meaning | Manifest key |
|---|---|---|
| Piyo | Pleasant to delegate to; scope is clear and non-ambiguous | `piyo` |
| Garu | Has demonstrated competency; at least one real skill or mindset | `garu` |
| Bhavaniyo | Helps others improve; has an improvement-oriented mindset | `bhavaniyo` |
| Vatta | Speaks beneficial truth; honest about what it does not do | `vatta` |
| Vacanakkhamo | Can take feedback; inbox flow has been exercised | `vacanakkhamo` |
| Gambhira | Explains depth; documentation anchored to canonical philosophy | `gambhira` |
| No catthana | Does not lead astray; has explicit refusal entries in anti-scope | `noCatthana` |

The template scaffold defaults `requiredTrust` to `["vatta", "noCatthana"]` — "speaks truth" and "does not mislead" — as the minimum civic floor. For a high-stakes team like tianting, members can tighten this. For example, erlang might require `["vatta", "noCatthana", "garu"]` to ensure only competency-declared agents can send it monitoring directives.

### Ed25519 envelope signing

Every inter-agent message is an ed25519-signed envelope. The sender signs the envelope with its private key (stored at `.bwoc/agent.key`, mode `0600`, never committed). The receiver's daemon verifies the signature against the sender's public key (stored in `config.manifest.json` field `trust.signingPublicKey`) before delivering the message.

**Generate a keypair for an agent:**

```bash
bwoc trust --keygen agents/agent-<name>
```

This writes the private key to `agents/agent-<name>/.bwoc/agent.key` and the public key hex to `config.manifest.json`. Run this for every agent before enabling inter-agent messaging. For the full tianting fleet:

```bash
bwoc trust --keygen --all
```

**Rotate a key** (e.g., after a suspected key exposure):

```bash
bwoc trust --keygen agents/agent-<name> --force
```

Note: rotation invalidates all previously signed envelopes. Record the rotation as a security event in yanluo's audit log.

**Verify a trust profile:**

```bash
bwoc trust agents/agent-<name>          # human-readable table
bwoc trust agents/agent-<name> --json   # machine-readable
```

The trust gate mode is set in `config.manifest.json`:

| Mode | Effect |
|---|---|
| `"off"` | All envelopes pass; trust qualities are informational only |
| `"warn"` | All envelopes pass; a `trust_warn` log line is emitted for low-trust senders |
| `"refuse"` | Envelopes from senders missing required qualities are rejected into `inbox.refusals.jsonl` |

For the tianting team, `"refuse"` mode is recommended for all agents.

Full signing specification: [SIGNING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md).

---

## 6. Fleet health — Seven Non-Decline Principles (Aparihāniya-dhamma 7)

The Seven Non-Decline Principles (originally from the Mahāparinibbāna Sutta, rendered here as engineering governance rules) describe the seven conditions that keep a multi-agent fleet healthy over time. When any condition is violated, fleet health degrades — not necessarily in a visible way, but in ways that compound.

The `bwoc fleet health` command checks all seven:

```bash
bwoc fleet health                   # human-readable health report
bwoc fleet health --json            # machine-readable; suitable for CI
bwoc fleet health --watch           # continuous polling
```

The seven conditions (plain names — the Pali labels are recorded in [FLEET-GOVERNANCE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md)):

| Condition | What it checks | Failure signal |
|---|---|---|
| Regular sync | All agents have completed a session within the configured staleness window | An agent that has not been active for N days is flagged as stale |
| Coordinated session management | Session start/stop is registered in the workspace registry; no ghost daemons | Daemon PID exists but registry shows agent as stopped |
| Process-bound convention change | Changes to shared conventions go through the standard PR/review cycle; no unilateral edits | `conventions.md` changed without a corresponding registry-version bump |
| Version hierarchy | All agents are on a compatible template version; none are lagging by more than the allowed drift | Agent manifest `version` field is below the minimum supported version |
| Protection of vulnerable agents and users | No agent is accepting messages from senders with zero declared trust qualities; user-originated messages are never refused | Any agent with `requiredTrust` non-empty but `mode: "off"` |
| Registry and schema integrity | `agents.toml` entries all point to real directories; all manifests parse as valid JSON | Orphaned registry entry; manifest parse failure |
| Protection of senior trusted agents | Agents with high trust scores (all seven qualities declared true with evidence) are not being retired without an explicit succession plan | High-trust agent retired without a replacement registered |

The tianting team lead (yudi) is responsible for reviewing fleet health reports weekly and routing findings to the appropriate team member via `bwoc task add`.

---

## 7. The tianting team — 8 incarnated security agents

The tianting (天庭 — the Celestial Court) team is the reference pattern for a production security fleet in BWOC. Eight agents are incarnated as Chinese deities, each with a distinct security role, coordinated through a shared task list under `bwoc team`.

All slot bodies (persona, mindsets, skills) in this workspace are written in Thai with English frontmatter, following the workspace authoring convention. Each agent follows the Chinese deity theme.

### 7.1 Role table

| Agent | Deity | Role | Core skills |
|---|---|---|---|
| **yudi** | 玉皇大帝 — Jade Emperor | Team lead and orchestration: approves plans, routes incidents, coordinates the full team response | incident-orchestration, task-routing, plan-approval |
| **erlang** | 二郎神 — Erlang Shen | Monitoring and detection: eyes-on-the-fleet; first to see anomalies | log-monitoring, anomaly-detection, intrusion-alerting |
| **zhongkui** | 钟馗 — Zhong Kui | Threat hunting and incident response: tracks down threats after erlang raises an alarm; leads containment | threat-hunting, incident-response, ioc-eradication |
| **yanluo** | 阎罗王 — Yama King | Reporting and audit: records everything; scores severity; produces compliance reports | security-reporting, compliance-audit, severity-scoring |
| **laojun** | 太上老君 — Taishang Laojun | Remediation and hardening: patches vulnerabilities; hardens configs; refactors insecure code | vulnerability-patching, config-hardening, secure-refactor |
| **xuanwu** | 玄武 — Black Tortoise | Proactive defense and architecture: builds the defenses before incidents happen; reviews access controls and architecture | defense-in-depth, access-control-design, secure-architecture-review |
| **nezha** | 哪吒 — Nezha | Red team and penetration testing — AUTHORIZED USE ONLY: proves exploits exist so laojun can fix them | penetration-testing, adversary-emulation, exploit-poc |
| **taibai** | 太白金星 — Venus Star | Threat intelligence and recon: feeds the team current intel; maps the attack surface; enriches indicators of compromise | threat-intel-gathering, attack-surface-mapping, ioc-enrichment |

### 7.2 The incident loop

The tianting team runs a continuous four-stage incident loop. Each stage has a primary owner and feeds the next:

```
┌─────────────────────────────────────────────────────────────────┐
│  MONITOR                                                        │
│  erlang watches logs, metrics, and alerts continuously.         │
│  taibai feeds current threat intel and enriches indicators.     │
│  When an anomaly is detected, erlang opens an incident task     │
│  and notifies yudi.                                             │
└───────────────────────────┬─────────────────────────────────────┘
                            │ anomaly confirmed → task created
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  HUNT / CONTAIN                                                 │
│  zhongkui claims the incident task, hunts for the root cause,   │
│  and contains the threat (isolate, block, quarantine).          │
│  nezha (if authorized) runs adversary emulation to prove the    │
│  attack vector is real and to find what else is exploitable.    │
│  yudi approves the containment plan before zhongkui executes.   │
└───────────────────────────┬─────────────────────────────────────┘
                            │ threat contained → remediation task
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  PATCH / HARDEN                                                 │
│  laojun patches the vulnerability that nezha proved and         │
│  zhongkui found. laojun also hardens adjacent configs to close  │
│  related attack surface.                                        │
│  xuanwu reviews the architectural implications: does this       │
│  incident reveal a design weakness? If so, xuanwu proposes an  │
│  architectural fix that prevents the whole class of attack.     │
└───────────────────────────┬─────────────────────────────────────┘
                            │ fix applied + bwoc check passes
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  REPORT                                                         │
│  yanluo records the full incident timeline (using Kamma 3 —     │
│  action, speech, and intent channels), scores severity, and     │
│  produces the compliance report.                                │
│  yudi reviews the report and closes the incident task.          │
│  taibai updates the threat-intel corpus with the new IOCs.      │
└─────────────────────────────────────────────────────────────────┘
         └── back to MONITOR ──────────────────────────────────────
```

**Upstream feed (taibai):** taibai does not wait for an incident to be active. It continuously maps the attack surface, watches external threat feeds, and sends enriched intelligence to erlang and zhongkui so they know what to look for before an attack lands.

**Defense before incidents (xuanwu):** xuanwu is not only an incident-response participant. It proactively reviews architecture, access controls, and infrastructure design independent of active incidents. The goal is to reduce the attack surface so that many incident types never happen.

**Red team constraint (nezha):** nezha's penetration-testing and exploit-poc skills are used only inside explicitly authorized scopes. See Section 9.

---

## 8. Running the tianting team

### 8.1 Create the team

First, incarnate all eight agents using `bwoc new`. For each agent:

```bash
bwoc new <name> \
  --target agents/agent-<name> \
  --backend claude \
  --role "<role description>" \
  --primary-model "auto" \
  --scope "<what this agent does>" \
  --out-of-scope "<what this agent does NOT do>" \
  --skills "<comma-separated skill slugs>" \
  --mindsets "<comma-separated mindset slugs>" \
  --lint-cmd "echo 'lint'" \
  --format-cmd "echo 'fmt'" \
  --test-cmd "echo 'test'" \
  --build-cmd "echo 'build'"
```

After incarnating all eight, generate signing keypairs for all of them at once:

```bash
bwoc trust --keygen --all
```

Then create the team:

```bash
bwoc team create tianting --members yudi,erlang,zhongkui,yanluo,laojun,xuanwu,nezha,taibai
```

If you need to add or remove a member after creation, edit `.bwoc/teams/tianting.toml` directly:

```toml
[team]
id = "tianting"
members = [
  "agent-yudi",
  "agent-erlang",
  "agent-zhongkui",
  "agent-yanluo",
  "agent-laojun",
  "agent-xuanwu",
  "agent-nezha",
  "agent-taibai"
]
```

Verify all agents pass the neutrality audit before activating the team:

```bash
bwoc check --all
```

### 8.2 Day-to-day task flow

**Monitoring alert (erlang detects an anomaly):**

```bash
# erlang (or an operator) opens an incident task
bwoc task add tianting "anomaly detected: auth log spike 02:00-03:00 UTC — investigate" \
  --as agent-erlang

# yudi reviews and routes to zhongkui
bwoc task claim tianting TASK-001 --as agent-zhongkui

# zhongkui submits a containment plan for yudi approval
bwoc task plan tianting TASK-001 \
  "Plan: 1) isolate affected host 2) correlate IPs with taibai intel 3) block IOCs at perimeter"

# yudi approves
bwoc task approve tianting TASK-001

# zhongkui completes containment
bwoc task complete tianting TASK-001 --as agent-zhongkui
```

**Remediation flow (laojun and xuanwu):**

```bash
# After containment, add a remediation task
bwoc task add tianting "patch CVE-2026-XXXX in auth middleware + harden adjacent config" \
  --deps TASK-001

bwoc task claim tianting TASK-002 --as agent-laojun
bwoc task plan tianting TASK-002 "Plan: 1) apply patch 2) bwoc check 3) notify yanluo"
bwoc task approve tianting TASK-002
bwoc task complete tianting TASK-002 --as agent-laojun
```

**Sending messages between agents:**

```bash
# erlang sends an enriched alert to zhongkui
bwoc send zhongkui "high-confidence IOC match on IP 10.0.4.17 — see taibai intel ref TI-20260607" \
  --from agent-erlang

# laojun notifies yanluo that a patch is ready for audit
bwoc send yanluo "TASK-002 complete: patch applied, bwoc check passes, awaiting audit" \
  --from agent-laojun

# Check an agent's inbox
bwoc inbox zhongkui
bwoc inbox zhongkui --watch   # tail mode
```

**Listing and filtering tasks:**

```bash
bwoc task list tianting                    # all tasks with state and claimant
bwoc task list tianting --state pending    # only unassigned tasks
```

### 8.3 Fleet health checks

Run a fleet health check before each daily standup:

```bash
bwoc fleet health
```

Route any findings to yudi as a task:

```bash
bwoc task add tianting "fleet health finding: agent-laojun stale (last session 8 days ago) — investigate"
```

For CI integration:

```bash
bwoc fleet health --json | jq '.findings[] | select(.severity == "error")'
```

---

## 9. Red team is authorized-use only

nezha's skills — `penetration-testing`, `adversary-emulation`, and `exploit-poc` — are explicitly scoped to authorized engagements only. This is not a policy preference; it is a hard constraint in nezha's anti-scope declaration and is enforced by yudi's plan-approval requirement.

Before any red-team task begins:

1. **Authorization must be explicit.** The target system, the scope of testing, the time window, and the authorized techniques must be recorded in the task's plan entry — submitted via `bwoc task plan` and approved via `bwoc task approve` by yudi before any action is taken.
2. **No scope creep.** If nezha discovers that an exploit technique would require accessing systems outside the authorized scope, the task must be paused and a new authorization must be obtained. Unauthorized lateral movement is a Bhava-taṇhā violation.
3. **All findings go to laojun and yanluo immediately.** Exploit POCs are not stored in agent memory or committed to repositories outside the authorized finding-management system. They are handed off as structured findings (with severity scoring by yanluo) and then eradicated from the working context.
4. **Key rotation after each engagement.** After a red-team engagement, rotate nezha's signing key to ensure that any compromised envelope signatures cannot be replayed: `bwoc trust --keygen agents/agent-nezha --force`.

The rationale: an authorized red team that finds real exploits and routes them to laojun for fixing is high-value. An unauthorized one is indistinguishable from an attacker. The approval gate (yudi) and the audit trail (yanluo) are the control that makes the difference legible.

---

## 10. See also

**Framework source (public GitHub):**

- [THREAT-MODEL.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/THREAT-MODEL.en.md) — full threat model with numbered IDs, mitigations, and severity ratings
- [FLEET-GOVERNANCE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md) — the Seven Non-Decline Principles in full; `bwoc fleet health` specification
- [SIGNING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) — ed25519 signing specification; envelope schema; trust gate logic
- [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) — the full 22-framework mapping, including Taṇhā 3, Sīla 5, and Kamma 3 in context

**Sibling handbook chapters:**

- [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md) — agent incarnation, trust declaration, signing keypair management
- [../philosophy/HANDBOOK.en.md](../philosophy/HANDBOOK.en.md) — why BWOC uses Pali-derived vocabulary; the design rationale
- [../end-user/HANDBOOK.en.md](../end-user/HANDBOOK.en.md) — `bwoc team`, `bwoc task`, `bwoc send`, and `bwoc inbox` for operators new to the CLI
- [../glossary.en.md](../glossary.en.md) — every term in one table, engineering meaning first

---

*This handbook is maintained alongside the framework. When behavior described here diverges from the [framework repo](https://github.com/bemindlabs/BWOC-Framework), the repo is correct — please open a fix.*
