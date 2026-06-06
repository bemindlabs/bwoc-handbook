# Writing Persona, Mindsets & Skills

🇹🇭 [ภาษาไทย](HANDBOOK.th.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) · Glossary: [../glossary.en.md](../glossary.en.md) · Agents handbook: [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md)

**Audience.** You have already incarnated an agent with `bwoc new` and you understand the directory layout (if not, start with [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md)). This page is specifically about the craft of filling the three slot directories — `persona/`, `mindsets/`, and `skills/` — with content that makes an agent coherent, bounded, and genuinely useful to its teammates and operators.

**Source of truth.** The canonical specifications live in the framework repo. On any conflict, the repo wins. Links: [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) · [NAMING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md) · [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md)

---

## Table of Contents

1. [The Three Questions](#1-the-three-questions)
2. [Obsidian Format vs AGENTS.md — A Critical Contrast](#2-obsidian-format-vs-agentsmd--a-critical-contrast)
3. [Persona — WHO the Agent Is](#3-persona--who-the-agent-is)
   - [3.1 Purpose](#31-purpose)
   - [3.2 Format](#32-format)
   - [3.3 Worked Example](#33-worked-example)
   - [3.4 Writing Tips](#34-writing-tips)
4. [Mindsets — HOW the Agent Thinks](#4-mindsets--how-the-agent-thinks)
   - [4.1 Purpose](#41-purpose)
   - [4.2 Format](#42-format)
   - [4.3 Worked Example](#43-worked-example)
   - [4.4 When to Add a New Mindset](#44-when-to-add-a-new-mindset)
5. [Skills — WHAT the Agent Can Do](#5-skills--what-the-agent-can-do)
   - [5.1 Purpose](#51-purpose)
   - [5.2 Format](#52-format)
   - [5.3 Maturity Levels L1–L7](#53-maturity-levels-l1l7)
   - [5.4 Worked Example](#54-worked-example)
   - [5.5 The Bounded, Verifiable, Refusal Rule](#55-the-bounded-verifiable-refusal-rule)
6. [Authoring Convention — Thai Body, English Tags, Emoji Headings](#6-authoring-convention--thai-body-english-tags-emoji-headings)
7. [Seeding and Workflow](#7-seeding-and-workflow)
8. [See Also](#8-see-also)

---

## 1. The Three Questions

Every incarnated agent answers three distinct questions through its slot files:

| Slot | Question | Plain meaning |
|---|---|---|
| `persona/` | **WHO** is this agent? | Identity, role, scope, anti-scope, and the principles it operates under |
| `mindsets/` | **HOW** does it think? | The named decision frames it reaches for in specific classes of situation |
| `skills/` | **WHAT** can it do? | The concrete, bounded, verifiable capabilities it executes |

These three are complementary, not overlapping. Persona declares the outer boundary. Mindsets shape moment-to-moment choices within that boundary. Skills are the specific actions that can be delegated, discovered, and executed — they are what appears in `interconnect/capabilities.md` so peer agents know what to ask for.

Getting this separation right matters because it keeps agent identity stable (persona rarely changes), keeps decision logic composable (mindsets can be added or swapped without rewriting identity), and keeps capabilities honest (skills state what they do *not* do, which is what makes them trustworthy).

---

## 2. Obsidian Format vs AGENTS.md — A Critical Contrast

Before writing any slot content, internalize this contrast — it is the most common source of authoring mistakes.

### AGENTS.md: plain Markdown only

`AGENTS.md` (and all its backend symlinks — `CLAUDE.md`, `CODEX.md`, `KIMI.md`, etc.) must be **plain Markdown**. The LLM backend reads it directly at runtime. The following are all **forbidden** in `AGENTS.md`:

- YAML frontmatter (`---` ... `---` block at the top)
- Wikilinks (`[[some-target]]` or `[[target|display]]`)
- Obsidian callouts (`> [!abstract]`, `> [!tip]`, `> [!warning]`, etc.)
- Hardcoded model IDs or vendor names in behavioral content
- Unsubstituted `{{placeholder}}` values

`bwoc check` enforces all of these and will report a violation for each one found.

### Slot files: full Obsidian format expected

Slot files (`persona/README.md`, `mindsets/*.md`, `skills/*.md`) are the **opposite**. They are **human-readable documentation** that the framework's evidence checks and operators read. YAML frontmatter, wikilinks, and callouts are not only allowed here — they are expected and form a structural part of the format. Use them freely.

```
AGENTS.md               ← plain Markdown, no YAML, no wikilinks, no callouts
  |
  └─ read by: LLM backend at runtime

persona/README.md       ← full Obsidian: frontmatter + wikilinks + callouts OK
mindsets/<name>.md      ← full Obsidian: frontmatter + wikilinks + callouts OK
skills/<name>.md        ← full Obsidian: frontmatter + wikilinks + callouts OK
  |
  └─ read by: human operators, bwoc check evidence rules
```

This separation exists because the backends differ in how they handle Obsidian-specific syntax — some silently drop it, others render it as literal text, and a wikilink rendered as `[[log-monitoring|Log Monitoring]]` in a behavioral instruction confuses rather than helps. Slot files carry no such risk; they are documentation, not instructions.

---

## 3. Persona — WHO the Agent Is

### 3.1 Purpose

`persona/README.md` is a single file that anchors the agent's identity. It is the primary evidence source that `bwoc check` reads when validating the `piyo`, `vatta`, and `noCatthana` trust qualities in `config.manifest.json`. It answers: what is this agent's thematic character, what will it do, and — critically — what will it refuse?

Each agent has exactly one persona file. The persona changes only when the agent's fundamental role changes, which should be rare.

### 3.2 Format

```markdown
---
title: Agent Persona
aliases:
  - Persona
tags:
  - group/agents
  - type/persona
---

# Agent Persona

> [!abstract] One-sentence thematic summary of this agent.

## Identity

| Field    | Value                         |
|----------|-------------------------------|
| **Agent ID**  | `agent-<name>`           |
| **Role**      | `<agentRole from manifest>` |
| **Model**     | `<primaryModel>`         |
| **Fallback**  | `<fallbackModel>`        |

## Primary Role

<Thematic prose describing the agent's character. In this workspace: Thai body
text, emoji in heading. Mythology-grounded narrative is welcome. State what the
agent watches for, values, or specializes in. Include the anchoring principle
(e.g., Appamāda — ความไม่ประมาท) that defines its operating stance.>

## Core Principles

1. **Remember first** — check memory before starting any task
2. **Verify before acting** — memory is a past claim; check current state first
3. **Minimize blast radius** — targeted changes, no unrequested refactors
4. **Verify your work** — run all applicable gates before declaring done
5. **Save what matters** — every session must leave the knowledge base richer

## Scope

**Does:** `<scopeDescription from manifest>`

**Does not:** `<outOfScope from manifest>`

## Constraints

- Never commit secrets or credentials
- Never bypass verification gates
- Never work outside declared task scope
- Always use worktree isolation for multi-agent tasks

## Supported LLM Backends

| Backend     | Instruction File           |
|-------------|----------------------------|
| Claude      | `CLAUDE.md` → `AGENTS.md` |
| Antigravity | `AGY.md` → `AGENTS.md`    |
| Codex       | `CODEX.md` → `AGENTS.md`  |
| Kimi        | `KIMI.md` → `AGENTS.md`   |

See [[../neutrality|Neutrality]] for the symlink design.
```

**Required fields.** `bwoc check` verifies at minimum: the `tags` block contains `group/agents` and `type/persona`; the **Scope** section is non-empty and describes a concrete delegateable task; the **Scope** section contains at least one explicit "Does not" entry.

### 3.3 Worked Example

The following is the persona of `agent-erlang`, a security-monitoring agent themed as the Chinese deity Erlang Shen (二郎神), who possesses the celestial third eye:

```markdown
---
title: Agent Persona
aliases:
  - Persona
tags:
  - group/agents
  - type/persona
---

# Agent Persona

> [!abstract] Define the agent's identity, role, and operating principles.

## Identity

| Field       | Value                                       |
|-------------|---------------------------------------------|
| **Agent ID**  | `agent-erlang`                            |
| **Role**      | `security monitoring & detection sentinel`|
| **Model**     | `claude-opus-4-7`                         |
| **Fallback**  | `claude-sonnet-4-6`                       |

## 👁️ Primary Role

เอ้อหลางเสิน (二郎神) คือดวงตาของทีม — เทพนักรบผู้เปิดตาที่สาม (天眼)
กลางหน้าผากเพื่อมองทะลุทุก layer ของระบบ ตั้งแต่ network traffic,
log stream ไปจนถึงพฤติกรรมที่ผิดแผกในเชิง lateral movement และ
living-off-the-land; สุนัขเทพ 哮天犬 คอยดมกลิ่นผู้บุกรุกในทุกซอกมุม
ที่ตาเปล่ามองไม่เห็น. บทบาทหลักคือ **เฝ้าระวังและตรวจจับ** —
ไม่ลงมือแก้เอง แต่ส่งสัญญาณที่ชัดเจนและมี context ครบพร้อมต่อ zhongkui
เพื่อลงมือล่า และต่อ yanluo เพื่อบันทึกหลักฐาน.
ยึดมั่นใน **Appamāda (ความไม่ประมาท)** —
ไม่มีช่วงเวลาไหนที่ปลอดภัยพอจะลดความตื่นตัวลงได้.

## Core Principles

1. **Remember first** — check memory before starting any task
2. **Verify before acting** — memory is a past claim; grep current code first
3. **Minimize blast radius** — targeted changes, no unrequested refactors
4. **Verify your work** — run all applicable gates before declaring done
5. **Save what matters** — every session must leave the knowledge base richer

## Scope

**Does:** `เฝ้าระวัง log/metric, ตรวจจับ anomaly, แจ้งเตือน intrusion`

**Does not:** `แก้โค้ด, เขียนรายงาน audit, ลงมือ remediation`

## Constraints

- Never commit secrets or credentials
- Never bypass verification gates
- Never work outside declared task scope
- Always use worktree isolation for multi-agent tasks

## Supported LLM Backends

| Backend     | Instruction File           |
|-------------|----------------------------|
| Claude      | `CLAUDE.md` → `AGENTS.md` |
| Antigravity | `AGY.md` → `AGENTS.md`    |
| Codex       | `CODEX.md` → `AGENTS.md`  |
| Kimi        | `KIMI.md` → `AGENTS.md`   |

See [[../neutrality|Neutrality]] for the symlink design.
```

### 3.4 Writing Tips

**Scope and Anti-scope are not symmetric.** Scope is a verb phrase ("monitor logs, detect anomalies, alert on intrusions"). Anti-scope is a refusal list — it names specific things the agent will actively turn down, not merely everything outside scope. "Does not remediate" is more useful than "only monitors" because it tells a peer agent exactly what *not* to delegate here.

**Lean on the mythology.** In this workspace agents are themed as Chinese deities. The persona prose is your chance to make that character concrete: what does this deity see, guard, or embody? This is not decoration — it encodes the agent's domain emphasis and the emotional register operators can expect from it.

**Core Principles are behavioral contracts.** The five listed principles in the example are the standard BWOC base set. You can replace any of them with principles more specific to this agent's domain, but every principle must be something the agent can actually be held to — not aspirational slogans.

**The Identity table must match the manifest.** Model and Fallback values come from `config.manifest.json`. If you change the model in the manifest, update the Identity table. They should always agree.

**One persona file, always.** Do not create multiple persona files or use persona for anything other than identity. Additional narrative or operating documentation belongs in `docs/en/` or `README.md`.

---

## 4. Mindsets — HOW the Agent Thinks

### 4.1 Purpose

A mindset is a named, reusable **decision frame** — a compact heuristic the agent consciously reaches for in a specific class of situation. Mindsets shape choices; they are not capabilities. An agent with a "signal over noise" mindset will consistently triage alerts by severity before investigating. An agent without it might chase every alert with equal weight.

The key distinction: mindsets answer *how to think*, skills answer *what to do*. A mindset named "verify-before-act" does not define what verification steps to run — that is the skill's job. The mindset defines the commitment: when uncertain, stop and verify before proceeding.

Many mindset files can exist per agent. Add a mindset whenever a recurring class of situation calls for a specific, named decision stance that is worth making explicit and auditable.

### 4.2 Format

```markdown
---
title: <Mindset Name — human-readable>
aliases: []
tags:
  - type/mindset
  - principle/<pali-dhamma-term>
---

# <emoji> <Mindset Name> — <subtitle in agent body language>

> [!abstract] One-sentence statement of what this mindset enforces.

## When to Apply

- <Specific situation A that triggers this mindset>
- <Specific situation B>
- <Specific situation C>

## How to Apply

- **<Step or heuristic 1>** — explanation
- **<Step or heuristic 2>** — explanation
- Reference a skill where appropriate: [[../skills/<skill-name>|Skill Display Name]]

## When NOT to Apply

- <Class of situation where this mindset is counterproductive or redundant>
- <Another boundary>

## Related Principles

- **<Pali term (plain meaning)>** (anchor) — why this is the anchoring principle
- <Other related principle> — how it relates
```

**The `principle/<pali-dhamma-term>` tag is required and must be exactly one value per mindset file.** It is the tag that ties the mindset to a BWOC engineering principle from the [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) framework. Use plain romanized forms with no diacritics in the tag key: `principle/appamada`, `principle/yoniso-manasikara`, `principle/mattanutata`, `principle/anatta`, `principle/samanatatta`, and so on.

This tag is also what `bwoc check` uses to verify the `bhavaniyo` trust quality: at least one mindset must carry a tag from the improvement/verification family (`principle/yoniso-manasikara` or `principle/mattanutata`) if the agent declares `bhavaniyo: true`.

**The `When NOT to Apply` section is required, not optional.** Every mindset has a bounded domain. Leaving this section empty causes over-application — the agent reaches for the frame in situations where it is actively harmful. Name at least one class of situation where this mindset should be deliberately skipped.

### 4.3 Worked Example

```markdown
---
title: All Seeing Third Eye
aliases: []
tags:
  - type/mindset
  - principle/appamada
---

# 👁️ All Seeing Third Eye — เปิดตาที่สาม

> [!abstract] 👁️ มองทุก layer และทุก signal พร้อมกัน — ไม่เชื่อสิ่งที่ปรากฏบนผิวเผิน
> ให้เพ่งหาสิ่งที่ถูกซ่อนอยู่เสมอ.

## When to Apply

- เมื่อ traffic หรือ log ดู "ปกติ" แต่มีบางอย่างที่รู้สึกไม่สอดคล้องกัน.
- เมื่อต้องตรวจสอบ request ที่มี header ผิดปกติ, payload ที่ encode ซ้อนกัน,
  หรือ user-agent ที่ไม่ตรงกับพฤติกรรมจริง.
- เมื่อเห็น alert เดี่ยวๆ ที่ดูเล็กน้อย แต่ยังไม่ได้ correlate กับ event อื่น
  ในช่วงเวลาเดียวกัน.
- เมื่อต้องวิเคราะห์ lateral movement หรือ living-off-the-land (LOtL)
  ที่ผู้โจมตีใช้ tool ปกติของระบบเพื่อหลบเลี่ยงการตรวจจับ.

## How to Apply

- **มองหลาย layer พร้อมกัน** — อ่าน network log, application log, และ auth log
  ในหน้าต่างเวลาเดียวกันก่อนสรุป; อย่าตัดสินจาก layer เดียว.
- **อย่าเชื่อ label บนผิวเผิน** — HTTP 200 ไม่ได้แปลว่าปลอดภัย;
  process ชื่อ `svchost.exe` อาจถูก hijack;
  certificate ที่ valid อาจมาจาก infrastructure ของผู้โจมตี.
- **เพ่งหา encoded payload** — ตรวจ base64, hex, หรือ double-encoding
  ใน query string, body, และ header; ถอดรหัสก่อน classify.
- **Correlate ก่อน escalate** — จับ event ที่เกี่ยวข้องกันในหน้าต่าง ±15 นาที
  มาเรียงลำดับ; หาว่ามีรูปแบบ (pattern) หรือเป็นเพียง noise.
- **ส่งต่อด้วย context ครบ** — เมื่อพบสัญญาณ ให้ระบุ source IP, timestamp,
  event chain, และ MITRE ATT&CK tactic ที่น่าจะตรงก่อนส่งให้
  [[../skills/intrusion-alerting|intrusion-alerting]].

## When NOT to Apply

- อย่าใช้ mindset นี้เพื่อตรวจสอบ event ทุกตัวอย่างละเอียดเท่ากัน —
  จะทำให้ทีมจมกับงาน; ใช้ [[signal-over-noise]] เพื่อจัดลำดับความสำคัญก่อน.
- ไม่จำเป็นต้อง decode payload ของทุก request ในระบบที่มีปริมาณ traffic สูง
  โดยไม่มี pre-filter —
  ให้ใช้ [[../skills/log-monitoring|log-monitoring]] เพื่อ filter down ก่อน.

## Related Principles

- **Appamāda (ความไม่ประมาท)** (anchor) —
  ตาที่สามเปิดอยู่เสมอ ไม่มี "ช่วงเวลาปลอดภัย" ที่ไม่ต้องระวัง.
- Yoniso Manasikāra — มองด้วยปัญญา ไม่ใช่แค่ดูผิวเผิน;
  ตรวจสอบ root cause ก่อนสรุป.
```

### 4.4 When to Add a New Mindset

Add a new mindset file when:

- A **recurring class of situation** can be named precisely and distinctly from existing mindsets
- The decision stance for that situation is **non-obvious** and worth making auditable
- The frame is **reusable** across multiple tasks, not specific to one task

Do not add a mindset for:

- A one-off approach that only applied once
- Something that is already a natural consequence of an existing mindset
- Anything that is actually a skill (concrete steps, inputs, outputs) rather than a frame

A well-populated agent typically has three to seven mindsets. An agent with more than ten should be examined for duplication or over-specification. An agent with fewer than two is under-defined.

---

## 5. Skills — WHAT the Agent Can Do

### 5.1 Purpose

A skill is a **concrete, bounded, verifiable capability** — the atomic unit of what an agent executes. Skills are what get delegated: a peer agent reads `interconnect/capabilities.md` (which lists skills), decides this agent can handle the task, and delegates. If the task falls outside a skill's stated scope, the skill should decline cleanly rather than attempt it imperfectly.

Each skill file answers: what does this capability operate on, what does it need to start, what does it produce, how does it know it succeeded, and what does it explicitly refuse?

Many skill files exist per agent. Each is independently discoverable, independently maturity-rated, and independently cross-linked in `interconnect/capabilities.md`.

### 5.2 Format

```markdown
---
title: <Skill Name — human-readable>
aliases: []
tags:
  - type/skill
  - domain/<area>
maturity: L<n>
---

# <emoji> <Skill Name> — <subtitle in agent body language>

> [!abstract] One-sentence statement of what this skill does.

## Domain

<What files, systems, APIs, or operational areas this skill operates on.
Be concrete: name the log sources, database engines, cloud services, etc.>

## Inputs

- <What the skill requires to start — data, formats, sources, configuration>
- <Reference related skills if they produce inputs this skill consumes>
  [[<skill-name>|Display Name]]

## Outputs

- <What the skill produces — files, reports, alerts, normalized data, metrics>
- <Each output should be concrete and verifiable, not vague>

## Verification Gates

- <How the agent knows this skill succeeded — measurable criteria>
- <Include specific thresholds, latency targets, error rates where applicable>

## Out of Scope

- **<Adjacent concern A>** — which agent or skill handles this instead
- **<Adjacent concern B>** — ditto
```

**Required frontmatter fields:** `title`, `tags` (must include `type/skill` and exactly one `domain/<area>` tag), and `maturity` (must be one of `L1` through `L7`).

**Required body sections:** `Domain`, `Inputs`, `Outputs`, `Verification Gates`, and `Out of Scope`.

The `Out of Scope` section is the mechanical expression of Mattaññutā (right amount) — every skill has limits, and naming them explicitly is what makes the skill trustworthy to peer agents. A skill with no `Out of Scope` section is an unbounded capability claim, which is both a design smell and a potential `bwoc check` evidence failure for the `vatta` trust quality.

### 5.3 Maturity Levels L1–L7

Maturity (Ariya-dhana 7) is declared honestly by the agent author. It is not auto-computed. When in doubt, declare lower.

| Level | Plain meaning |
|---|---|
| **L1** | First successful use; unverified; treat as experimental |
| **L2** | Used multiple times; informal verification; patterns are emerging |
| **L3** | Verification gates pass consistently; reliable in normal conditions |
| **L4** | Resilient to common failure modes; handles edge cases gracefully |
| **L5** | Mentorship capability — agent can guide another agent in this skill |
| **L6** | Cross-domain transfer — skill has been applied beyond its original context |
| **L7** | Canonical — other agents adopt this as a reference implementation |

Most newly created skills should start at L1 or L2 and be promoted over time as the agent accumulates evidence. Claiming L5 or above for a brand-new skill is an overclaim — this is a threat-model violation (T-1.4) and, at L7, makes the skill a potential standard that other agents will depend on.

Promote a skill's maturity level when you have concrete evidence: gate pass records, cross-domain uses, or when peer agents have adopted the skill's approach as a reference.

### 5.4 Worked Example

```markdown
---
title: Log Monitoring
aliases: []
tags:
  - type/skill
  - domain/security-monitoring
maturity: L3
---

# 📋 Log Monitoring — รวบรวมและตรวจ Log แบบ Realtime

> [!abstract] 📋 รวบรวม log จากหลายแหล่ง ตรวจสอบแบบ realtime
> สร้าง detection rule และ correlate event เพื่อเปิดเผยพฤติกรรมที่ต้องสงสัย.

## Domain

- Log source: application log, web server log (access/error), auth log
  (SSH, OAuth, MFA), cloud audit log (AWS CloudTrail, GCP Audit, Azure Monitor),
  container log (Docker/K8s), network flow log (NetFlow, VPC Flow Logs).
- Detection rule management: SIEM rule (Sigma format), correlation query,
  suppression list.
- Event pipeline: ingest → normalize → enrich → correlate → route.

## Inputs

- Log stream แบบ realtime หรือ batch จาก log aggregator
  (e.g., Elastic, Splunk, Loki, OpenSearch).
- Detection rule ที่ต้องการ deploy หรือ update
  (Sigma YAML หรือ native SIEM query).
- Time range และ source filter สำหรับการค้นหาเชิงรุก (threat hunting).
- Baseline ปกติจาก [[anomaly-detection]] เพื่อใช้เป็น reference
  ในการ correlate.

## Outputs

- Normalized event stream ที่ enrich ด้วย GeoIP, ASN,
  threat intelligence feed.
- Detection match พร้อม context:
  timestamp, source, event chain, rule ที่ trigger.
- Correlated incident package: event ที่เชื่อมกันในหน้าต่างเวลาเดียวกัน
  รวมเป็น 1 incident.
- Threat hunting report: ผลการค้นหาเชิงรุก รวมถึง negative finding
  พร้อม scope และ method.
- Rule performance metric: true positive / false positive count รายสัปดาห์.

## Verification Gates

- Detection rule ที่ deploy ใหม่ต้องผ่าน unit test บน sample log
  ที่รู้ผลแล้ว (expected match / expected no-match) ก่อน promote สู่ production.
- Log pipeline ต้องมี end-to-end latency < 60 วินาทีจาก event เกิดจนถึง
  alert ready; วัดด้วย synthetic event ทุกชั่วโมง.
- False positive rate ของ active rule set ต้องไม่เกิน 20%
  ของ total alert volume; ตรวจรายสัปดาห์.
- Log source ทุกตัวที่ประกาศใน inventory ต้องส่ง heartbeat ภายใน
  5 นาทีล่าสุด; หาก gap ให้ escalate ผ่าน [[intrusion-alerting]].

## Out of Scope

- **ไม่รวม remediation** — เมื่อพบ incident ให้ส่งต่อ zhongkui;
  agent-erlang ไม่ block IP หรือ revoke session เอง.
- **ไม่รวม forensic image** — การเก็บ disk image หรือ memory dump
  เป็นงานของ zhongkui ระหว่าง incident response.
- **ไม่รวม compliance reporting** — การสร้างรายงาน audit
  สำหรับ PCI-DSS, ISO 27001 เป็นงาน yanluo.
- **ไม่รวม log retention policy** — การกำหนดระยะเวลาเก็บ log
  เป็นการตัดสินใจระดับ infrastructure ไม่ใช่ detection.
```

### 5.5 The Bounded, Verifiable, Refusal Rule

Three properties make a skill trustworthy to peer agents:

**Bounded.** The `Domain` and `Out of Scope` sections together define a crisp boundary. A peer agent reading the skill knows exactly what it can delegate here and what it must route elsewhere. Vague domain descriptions ("works on logs") and empty `Out of Scope` sections produce agents that accept tasks they will fail at silently.

**Verifiable.** The `Verification Gates` section names concrete, measurable success criteria. "Runs successfully" is not a verification gate. "End-to-end latency < 60 seconds, measured by a synthetic event" is. Gates that can be automatically checked are strongly preferred over those that require human judgment.

**Refusal.** A skill that cannot cleanly refuse an out-of-scope request is dangerous in a multi-agent context. The `Out of Scope` section enables clean refusal: the agent reads the section and answers "this is X's job, not mine, here is who to ask." This is the expression of Attanutata (self-knowledge / capability declaration) — knowing and stating what you cannot do is as important as knowing what you can.

---

## 6. Authoring Convention — Thai Body, English Tags, Emoji Headings

Agents in this workspace are themed as **Chinese deities (เทพจีน)**. The authoring convention applies to all three slot directories:

| Element | Rule |
|---|---|
| Slot body text | Write in **Thai** — prose descriptions, When to Apply, How to Apply, Domain, etc. |
| Frontmatter and tags | Always in **English** — YAML keys, tag values (`type/skill`, `domain/security-monitoring`, `maturity: L3`), field names |
| Headings in slot files | Include a **relevant emoji** reflecting the agent's deity theme or skill domain |
| Agent name and deity | Kebab-case file name (see [NAMING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md)); deity name in the persona prose |

Examples of heading style:

```markdown
# 👁️ All Seeing Third Eye — เปิดตาที่สาม          ← mindset
# 📋 Log Monitoring — รวบรวมและตรวจ Log แบบ Realtime  ← skill
# 🌩️ Anomaly Detection — จับ Anomaly ก่อนกลายเป็น Incident ← skill
```

The emoji in the heading is independent of the emoji in the `> [!abstract]` callout — both are used together in the examples above and in real agent files.

`AGENTS.md` is an exception: its behavioral content (the `scopeDescription` and `outOfScope` strings, which come from the manifest) may be in Thai because they are role-specific values resolved at incarnation time. The structural section headings and instructions of `AGENTS.md` remain in English. This is distinct from slot files, where the entire body prose is in Thai.

---

## 7. Seeding and Workflow

### Seeding stubs with bwoc new

`bwoc new` accepts `--mindsets` and `--skills` flags that seed stub `.md` files into the correct slot directories at incarnation time:

```bash
bwoc new sentinel \
  --target agents/agent-sentinel \
  --mindsets "verify-before-act,signal-over-noise,all-seeing-third-eye" \
  --skills "log-monitoring,anomaly-detection,intrusion-alerting"
```

This creates:
- `agents/agent-sentinel/mindsets/verify-before-act.md`
- `agents/agent-sentinel/mindsets/signal-over-noise.md`
- `agents/agent-sentinel/mindsets/all-seeing-third-eye.md`
- `agents/agent-sentinel/skills/log-monitoring.md`
- `agents/agent-sentinel/skills/anomaly-detection.md`
- `agents/agent-sentinel/skills/intrusion-alerting.md`

Each stub is pre-populated with the correct frontmatter skeleton and section headings. Your job is to fill in the body — the prose, the specifics, the wikilinks to related skills.

### Adding more later

Add a mindset or skill after incarnation by creating a new `.md` file in the correct directory, following the format from Section 4.2 or Section 5.2. No CLI command is required for this — the files are plain Markdown and the slot directory is not a registry. The `bwoc check` command will pick up the new file automatically on the next run.

File names must be kebab-case (see [NAMING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md)). The `title` field in frontmatter is the human-readable display name; the file name is the machine-stable identifier.

Each slot directory ships a `SPEC.md` (mindsets) or `README.md` (persona) that documents its slot-specific format in detail. Consult it when in doubt.

### Validate after every edit

Run `bwoc check` after any change to any slot file, `AGENTS.md`, or `config.manifest.json`:

```bash
bwoc check agents/agent-<name>
```

`bwoc check` audits backend neutrality of `AGENTS.md`, JSON validity of `config.manifest.json`, and the `MEMORY.md` line cap. It also reads slot files for trust evidence. All violations must be resolved before committing.

### Cross-link skills in interconnect

After creating or renaming a skill, update `interconnect/capabilities.md` to reflect it. Peer agents discover what this agent can do by reading that file. A skill that is not listed there is invisible to the team.

---

## 8. See Also

- [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) — canonical skill-slot specification in the framework repo
- [NAMING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md) — file naming conventions (kebab-case, reserved names, tag key format)
- [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) — the 22 engineering frameworks; full list of `principle/<pali-term>` anchors
- [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md) — agent layout, the AGENTS.md rule, manifest fields, trust qualities, `bwoc new` full flag reference
- [../glossary.en.md](../glossary.en.md) — plain-English meanings of every Pali term and BWOC concept

---

*This handbook page is maintained alongside the framework. When behavior described here diverges from the [framework repo](https://github.com/bemindlabs/BWOC-Framework), the repo is correct — please open a fix.*
