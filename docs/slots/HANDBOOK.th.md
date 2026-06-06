# การเขียน Persona, Mindsets และ Skills

🇬🇧 [English](HANDBOOK.en.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) · อภิธานศัพท์: [../glossary.en.md](../glossary.en.md) · คู่มือ Agents: [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md)

**ผู้อ่านที่ตั้งใจไว้.** คุณได้สร้าง agent ด้วย `bwoc new` แล้ว และเข้าใจโครงสร้างไดเรกทอรีแล้ว (ถ้ายังไม่ได้อ่าน ให้เริ่มที่ [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md) ก่อน). หน้านี้เจาะลึกเฉพาะการ **เขียนเนื้อหา** ใน slot สามตัว ได้แก่ `persona/`, `mindsets/`, และ `skills/` เพื่อให้ agent มีตัวตน มีกรอบคิด และมีความสามารถที่ชัดเจน ตรวจสอบได้ และเชื่อถือได้โดยทีม

**แหล่งที่มาของความจริง.** spec ฉบับสมบูรณ์อยู่ใน framework repo หาก handbook นี้ขัดกับ repo ให้ถือว่า repo ถูกต้อง: [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) · [NAMING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md) · [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md)

---

## สารบัญ

1. [สามคำถามหลัก](#1-สามคำถามหลัก)
2. [Obsidian Format กับ AGENTS.md — ความแตกต่างที่สำคัญ](#2-obsidian-format-กับ-agentsmd--ความแตกต่างที่สำคัญ)
3. [Persona — WHO คือตัวตนของ Agent](#3-persona--who-คือตัวตนของ-agent)
   - [3.1 จุดประสงค์](#31-จุดประสงค์)
   - [3.2 รูปแบบไฟล์](#32-รูปแบบไฟล์)
   - [3.3 ตัวอย่างจริง](#33-ตัวอย่างจริง)
   - [3.4 เคล็ดลับการเขียน](#34-เคล็ดลับการเขียน)
4. [Mindsets — HOW กรอบคิดของ Agent](#4-mindsets--how-กรอบคิดของ-agent)
   - [4.1 จุดประสงค์](#41-จุดประสงค์)
   - [4.2 รูปแบบไฟล์](#42-รูปแบบไฟล์)
   - [4.3 ตัวอย่างจริง](#43-ตัวอย่างจริง)
   - [4.4 เมื่อไหร่ควรเพิ่ม Mindset ใหม่](#44-เมื่อไหร่ควรเพิ่ม-mindset-ใหม่)
5. [Skills — WHAT สิ่งที่ Agent ทำได้](#5-skills--what-สิ่งที่-agent-ทำได้)
   - [5.1 จุดประสงค์](#51-จุดประสงค์)
   - [5.2 รูปแบบไฟล์](#52-รูปแบบไฟล์)
   - [5.3 ระดับวุฒิภาวะ L1–L7](#53-ระดับวุฒิภาวะ-l1l7)
   - [5.4 ตัวอย่างจริง](#54-ตัวอย่างจริง)
   - [5.5 หลักการ Bounded / Verifiable / Refusal](#55-หลักการ-bounded--verifiable--refusal)
6. [ข้อตกลงการเขียน — ภาษาไทยใน Body, ภาษาอังกฤษใน Tags, Emoji ใน Heading](#6-ข้อตกลงการเขียน--ภาษาไทยใน-body-ภาษาอังกฤษใน-tags-emoji-ใน-heading)
7. [การสร้าง Stubs และขั้นตอนการทำงาน](#7-การสร้าง-stubs-และขั้นตอนการทำงาน)
8. [ดูเพิ่มเติม](#8-ดูเพิ่มเติม)

---

## 1. สามคำถามหลัก

agent ที่ถูก incarnate แล้วทุกตัวตอบสามคำถามที่แตกต่างกันผ่านไฟล์ใน slot:

| Slot | คำถาม | ความหมาย |
|---|---|---|
| `persona/` | **WHO** — นี่คือใคร? | ตัวตน บทบาท ขอบเขต สิ่งที่ปฏิเสธ และหลักการที่ยึดถือ |
| `mindsets/` | **HOW** — คิดอย่างไร? | กรอบคิดที่มีชื่อ ซึ่ง agent หยิบมาใช้ในสถานการณ์เฉพาะ |
| `skills/` | **WHAT** — ทำอะไรได้? | ความสามารถที่เป็นรูปธรรม มีขอบเขต และตรวจสอบได้ |

ทั้งสามส่วนนี้ **เสริมกัน ไม่ซ้อนทับกัน** — persona ประกาศขอบเขตภายนอก, mindsets กำหนดทางเลือกในชั่วขณะ, skills คือสิ่งที่ delegate ได้จริงและปรากฏใน `interconnect/capabilities.md` ให้ agent คู่ทีมค้นพบ

การแยกสามส่วนนี้ให้ถูกต้องทำให้ตัวตนของ agent มั่นคง (persona เปลี่ยนน้อย), กรอบคิดประกอบได้ (mindsets เพิ่มหรือเปลี่ยนโดยไม่ต้องเขียน identity ใหม่), และความสามารถน่าเชื่อถือ (skills ระบุว่า "ไม่ทำอะไร" ซึ่งทำให้มันน่าไว้วางใจ)

---

## 2. Obsidian Format กับ AGENTS.md — ความแตกต่างที่สำคัญ

ก่อนเขียนเนื้อหา slot ใดๆ ให้เข้าใจความแตกต่างนี้ให้ชัด — เป็นสาเหตุที่พบบ่อยที่สุดของข้อผิดพลาดในการ author

### AGENTS.md: plain Markdown เท่านั้น

`AGENTS.md` (และ symlink ทุกตัว — `CLAUDE.md`, `CODEX.md`, `KIMI.md` ฯลฯ) ต้องเป็น **plain Markdown** เท่านั้น เพราะ LLM backend อ่านไฟล์นี้โดยตรงตอน runtime สิ่งต่อไปนี้ **ห้ามใช้** ใน `AGENTS.md`:

- YAML frontmatter (บล็อก `---` ... `---` ที่ต้นไฟล์)
- Wikilinks (`[[some-target]]` หรือ `[[target|display]]`)
- Obsidian callouts (`> [!abstract]`, `> [!tip]`, `> [!warning]` ฯลฯ)
- model ID หรือชื่อ vendor ที่ hardcode ใน behavioral content
- `{{placeholder}}` ที่ยังไม่ได้ substitute

`bwoc check` ตรวจสอบทั้งหมดนี้และรายงาน violation ทุกรายการที่พบ

### ไฟล์ใน slot: ใช้ Obsidian format เต็มรูปแบบ

ไฟล์ใน slot (`persona/README.md`, `mindsets/*.md`, `skills/*.md`) คือ **ตรงกันข้าม** — เป็น documentation ที่มนุษย์และ evidence check ของ framework อ่าน YAML frontmatter, wikilinks, และ callouts ไม่ใช่แค่อนุญาต — แต่เป็นสิ่งที่คาดหวังและเป็นส่วนหนึ่งของโครงสร้าง

```
AGENTS.md               ← plain Markdown, ห้ามมี YAML / wikilinks / callouts
  |
  └─ อ่านโดย: LLM backend ตอน runtime

persona/README.md       ← Obsidian เต็มรูปแบบ: frontmatter + wikilinks + callouts OK
mindsets/<name>.md      ← Obsidian เต็มรูปแบบ: frontmatter + wikilinks + callouts OK
skills/<name>.md        ← Obsidian เต็มรูปแบบ: frontmatter + wikilinks + callouts OK
  |
  └─ อ่านโดย: ผู้ดูแล, bwoc check evidence rules
```

เหตุผลของการแยกนี้: backend แต่ละตัวจัดการ Obsidian syntax ต่างกัน — บางตัว drop มันทิ้งแบบเงียบๆ บางตัว render เป็น literal text และ wikilink ที่ render เป็น `[[log-monitoring|Log Monitoring]]` ใน behavioral instruction ทำให้งงแทนที่จะช่วย ไฟล์ใน slot ไม่มีความเสี่ยงนั้น เพราะมันคือ documentation ไม่ใช่ instruction

---

## 3. Persona — WHO คือตัวตนของ Agent

### 3.1 จุดประสงค์

`persona/README.md` เป็นไฟล์เดียวที่ยึดตัวตนของ agent ไว้ — เป็นแหล่ง evidence หลักที่ `bwoc check` อ่านเมื่อตรวจสอบ trust qualities `piyo`, `vatta`, และ `noCatthana` ใน `config.manifest.json` มันตอบว่า: agent นี้มีบุคลิกแบบไหน ทำอะไร และ — สำคัญที่สุด — ปฏิเสธอะไร?

แต่ละ agent มีไฟล์ persona เพียงไฟล์เดียว persona เปลี่ยนก็ต่อเมื่อบทบาทหลักของ agent เปลี่ยน ซึ่งไม่ควรเกิดบ่อย

### 3.2 รูปแบบไฟล์

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

> [!abstract] สรุปเป็นประโยคเดียวว่า agent นี้คืออะไร

## Identity

| Field       | Value                           |
|-------------|---------------------------------|
| **Agent ID**  | `agent-<name>`                |
| **Role**      | `<agentRole จาก manifest>`    |
| **Model**     | `<primaryModel>`              |
| **Fallback**  | `<fallbackModel>`             |

## Primary Role

<เนื้อหาเชิงตัวละครที่อธิบายบุคลิกของ agent ใน workspace นี้: เขียนเป็นภาษาไทย
ใส่ emoji ใน heading, ฝัง mythology ของเทพจีนที่เกี่ยวข้อง
ระบุ anchoring principle เช่น Appamāda — ความไม่ประมาท>

## Core Principles

1. **Remember first** — ตรวจสอบ memory ก่อนเริ่มงานทุกครั้ง
2. **Verify before acting** — memory คือสิ่งที่เคยเกิด; ตรวจสอบ state ปัจจุบันก่อนลงมือ
3. **Minimize blast radius** — แก้ไขเฉพาะที่จำเป็น ไม่ refactor โดยไม่ได้ขอ
4. **Verify your work** — ผ่าน gate ทุกตัวที่เกี่ยวข้องก่อนประกาศว่าเสร็จ
5. **Save what matters** — ทุก session ต้องทิ้งความรู้ไว้ในระบบมากขึ้นกว่าตอนเริ่ม

## Scope

**Does:** `<scopeDescription จาก manifest>`

**Does not:** `<outOfScope จาก manifest>`

## Constraints

- ห้าม commit secrets หรือ credentials
- ห้าม bypass verification gates
- ห้ามทำงานนอก scope ที่ประกาศไว้
- ใช้ worktree isolation เสมอสำหรับ multi-agent tasks

## Supported LLM Backends

| Backend     | Instruction File           |
|-------------|----------------------------|
| Claude      | `CLAUDE.md` → `AGENTS.md` |
| Antigravity | `AGY.md` → `AGENTS.md`    |
| Codex       | `CODEX.md` → `AGENTS.md`  |
| Kimi        | `KIMI.md` → `AGENTS.md`   |

ดู [[../neutrality|Neutrality]] สำหรับการออกแบบ symlink
```

**field ที่ต้องมี.** `bwoc check` ตรวจสอบอย่างน้อย: `tags` มี `group/agents` และ `type/persona`; ส่วน **Scope** ไม่ว่างและระบุงานที่ delegate ได้จริง; ส่วน **Scope** มีรายการ "Does not" อย่างน้อยหนึ่งรายการ

### 3.3 ตัวอย่างจริง

ตัวอย่างต่อไปนี้คือ persona ของ `agent-erlang` — agent เฝ้าระวังความปลอดภัยที่ theme ตามเทพจีนเอ้อหลางเสิน (二郎神) ผู้มีดวงตาที่สามบนหน้าผาก:

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

| Field       | Value                                        |
|-------------|----------------------------------------------|
| **Agent ID**  | `agent-erlang`                             |
| **Role**      | `security monitoring & detection sentinel` |
| **Model**     | `claude-opus-4-7`                          |
| **Fallback**  | `claude-sonnet-4-6`                        |

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

### 3.4 เคล็ดลับการเขียน

**Scope กับ Anti-scope ไม่ใช่สิ่งเดียวกัน.** Scope คือ verb phrase ("เฝ้าระวัง log, ตรวจจับ anomaly, แจ้งเตือน intrusion"). Anti-scope คือรายการสิ่งที่ปฏิเสธ — ระบุสิ่งที่ agent จะ "ไม่รับ" อย่างชัดเจน ไม่ใช่แค่ทุกอย่างที่อยู่นอก scope การบอกว่า "ไม่ทำ remediation" มีประโยชน์กว่า "แค่ monitor" เพราะบอก agent คู่ทีมได้ว่าอย่า delegate อะไรมาที่นี่

**ใช้ mythology ให้เต็มที่.** ใน workspace นี้ agent theme เป็นเทพจีน เนื้อหา persona prose เป็นโอกาสทำให้บุคลิกนั้นชัดเจน: เทพองค์นี้มองเห็นอะไร ปกป้องอะไร หรือเป็นสัญลักษณ์ของอะไร? ไม่ใช่แค่การตกแต่ง — มันเข้ารหัส domain emphasis และ emotional register ที่ผู้ดูแลคาดหวัง

**Core Principles คือสัญญาเชิงพฤติกรรม.** หลักการห้าข้อในตัวอย่างเป็นชุด base set มาตรฐานของ BWOC คุณสามารถแทนที่ด้วยหลักการที่เฉพาะเจาะจงกับ domain ของ agent ได้ แต่ทุกหลักการต้องเป็นสิ่งที่ agent ทำได้จริง ไม่ใช่ slogan

**ตาราง Identity ต้องตรงกับ manifest เสมอ.** ค่า Model และ Fallback มาจาก `config.manifest.json` หากเปลี่ยน model ใน manifest ให้อัปเดตตารางนี้ด้วย

**มีได้แค่ไฟล์ persona เดียวเสมอ.** ไม่ต้องสร้างหลายไฟล์หรือใช้ persona เพื่ออะไรนอกจากตัวตน documentation เพิ่มเติมไปอยู่ใน `docs/en/` หรือ `README.md`

---

## 4. Mindsets — HOW กรอบคิดของ Agent

### 4.1 จุดประสงค์

mindset คือ **กรอบตัดสินใจที่มีชื่อและนำมาใช้ซ้ำได้** — heuristic ขนาดกะทัดรัดที่ agent หยิบมาอย่างมีสติเมื่อเจอสถานการณ์เฉพาะประเภทหนึ่ง mindsets กำหนดทางเลือก — ไม่ใช่ความสามารถ agent ที่มี mindset "signal over noise" จะจัดลำดับความสำคัญ alert ตาม severity ก่อนตรวจสอบทุกครั้ง agent ที่ไม่มีอาจไล่ตาม alert ทุกตัวด้วยน้ำหนักเท่ากัน

ความแตกต่างหลัก: mindsets ตอบ *คิดอย่างไร*, skills ตอบ *ทำอะไร* mindset ชื่อ "verify-before-act" ไม่ได้กำหนดขั้นตอนการตรวจสอบ — นั่นเป็นหน้าที่ของ skill มัน กำหนดความมุ่งมั่น: เมื่อไม่แน่ใจ หยุดและตรวจสอบก่อนดำเนินการ

แต่ละ agent มีได้หลายไฟล์ mindset เพิ่ม mindset ใหม่เมื่อสถานการณ์ที่เกิดซ้ำเรียกร้องให้มีท่าทีการตัดสินใจที่เฉพาะเจาะจงและคุ้มค่าที่จะทำให้ชัดเจนและตรวจสอบได้

### 4.2 รูปแบบไฟล์

```markdown
---
title: <Mindset Name — ชื่อที่อ่านออกได้>
aliases: []
tags:
  - type/mindset
  - principle/<pali-dhamma-term>
---

# <emoji> <Mindset Name> — <คำอธิบายภาษาไทย>

> [!abstract] ประโยคเดียวที่บอกว่า mindset นี้บังคับอะไร

## When to Apply

- <สถานการณ์ A ที่ trigger mindset นี้>
- <สถานการณ์ B>
- <สถานการณ์ C>

## How to Apply

- **<ขั้นตอนหรือ heuristic 1>** — อธิบาย
- **<ขั้นตอนหรือ heuristic 2>** — อธิบาย
- อ้างอิง skill ที่เกี่ยวข้องด้วย wikilink: [[../skills/<skill-name>|Display Name]]

## When NOT to Apply

- <ประเภทของสถานการณ์ที่ mindset นี้ให้โทษหรือซ้ำซ้อน>
- <ขอบเขตอีกอย่าง>

## Related Principles

- **<Pali term (ความหมายธรรมดา)>** (anchor) — ทำไมนี่คือ anchoring principle
- <หลักการที่เกี่ยวข้องอื่น> — เกี่ยวข้องอย่างไร
```

**tag `principle/<pali-dhamma-term>` เป็น required และต้องมีค่าเดียวต่อไฟล์ mindset** — มันผูก mindset เข้ากับ engineering principle จาก [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) ใช้ romanized form ที่ไม่มี diacritics ใน tag key: `principle/appamada`, `principle/yoniso-manasikara`, `principle/mattanutata`, `principle/anatta`, `principle/samanatatta` เป็นต้น

tag นี้ยังเป็นสิ่งที่ `bwoc check` ใช้ตรวจสอบ trust quality `bhavaniyo`: mindset อย่างน้อยหนึ่งอันต้องมี tag จากกลุ่ม improvement/verification (`principle/yoniso-manasikara` หรือ `principle/mattanutata`) หาก agent ประกาศ `bhavaniyo: true`

**ส่วน `When NOT to Apply` เป็น required ไม่ใช่ optional** — ทุก mindset มีขอบเขต การเว้นส่วนนี้ทำให้ agent ใช้ frame นี้มากเกินไปในสถานการณ์ที่เป็นอันตราย ระบุอย่างน้อยหนึ่งประเภทสถานการณ์ที่ควร skip mindset นี้โดยตั้งใจ

### 4.3 ตัวอย่างจริง

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
- เมื่อต้องวิเคราะห์ lateral movement หรือ living-off-the-land (LOtL).

## How to Apply

- **มองหลาย layer พร้อมกัน** — อ่าน network log, application log,
  และ auth log ในหน้าต่างเวลาเดียวกันก่อนสรุป; อย่าตัดสินจาก layer เดียว.
- **อย่าเชื่อ label บนผิวเผิน** — HTTP 200 ไม่ได้แปลว่าปลอดภัย;
  process ชื่อ `svchost.exe` อาจถูก hijack.
- **เพ่งหา encoded payload** — ตรวจ base64, hex, หรือ double-encoding
  ใน query string, body, และ header; ถอดรหัสก่อน classify.
- **Correlate ก่อน escalate** — จับ event ที่เกี่ยวข้องกันในหน้าต่าง ±15 นาที
  มาเรียงลำดับ; หาว่ามีรูปแบบ (pattern) หรือเป็นเพียง noise.
- **ส่งต่อด้วย context ครบ** — เมื่อพบสัญญาณ ให้ระบุ source IP, timestamp,
  event chain, และ MITRE ATT&CK tactic ก่อนส่งให้
  [[../skills/intrusion-alerting|intrusion-alerting]].

## When NOT to Apply

- อย่าใช้ mindset นี้กับทุก event อย่างละเอียดเท่ากัน — จะทำให้ทีมจมกับงาน;
  ใช้ [[signal-over-noise]] เพื่อจัดลำดับความสำคัญก่อน.
- ไม่จำเป็นต้อง decode payload ของทุก request ในระบบ traffic สูง
  โดยไม่มี pre-filter;
  ให้ใช้ [[../skills/log-monitoring|log-monitoring]] เพื่อ filter down ก่อน.

## Related Principles

- **Appamāda (ความไม่ประมาท)** (anchor) —
  ตาที่สามเปิดอยู่เสมอ ไม่มี "ช่วงเวลาปลอดภัย" ที่ไม่ต้องระวัง.
- Yoniso Manasikāra — มองด้วยปัญญา ไม่ใช่แค่ดูผิวเผิน;
  ตรวจสอบ root cause ก่อนสรุป.
```

### 4.4 เมื่อไหร่ควรเพิ่ม Mindset ใหม่

เพิ่มไฟล์ mindset ใหม่เมื่อ:

- **สถานการณ์ที่เกิดซ้ำ** สามารถตั้งชื่อได้อย่างชัดเจนและแตกต่างจาก mindsets ที่มีอยู่
- ท่าทีการตัดสินใจสำหรับสถานการณ์นั้น **ไม่ใช่เรื่องธรรมดา** และคุ้มค่าที่จะทำให้ตรวจสอบได้
- frame นั้น **นำมาใช้ซ้ำได้** ข้ามหลาย task ไม่ใช่เฉพาะ task เดียว

ไม่ต้องเพิ่ม mindset สำหรับ:

- แนวทางที่ใช้ครั้งเดียว
- สิ่งที่เป็นผลธรรมชาติของ mindset ที่มีอยู่แล้ว
- อะไรก็ตามที่จริงๆ แล้วคือ skill (มีขั้นตอน, inputs, outputs) ไม่ใช่ frame

agent ที่ดีมักมี mindset สามถึงเจ็ดตัว agent ที่มีมากกว่าสิบตัวควรตรวจสอบว่ามี duplication หรือ over-specification ไหม agent ที่มีน้อยกว่าสองตัวยังกำหนดไม่ชัดพอ

---

## 5. Skills — WHAT สิ่งที่ Agent ทำได้

### 5.1 จุดประสงค์

skill คือ **ความสามารถที่เป็นรูปธรรม มีขอบเขต และตรวจสอบได้** — หน่วยอะตอมของสิ่งที่ agent ลงมือทำ Skills คือสิ่งที่ถูก delegate: agent คู่ทีมอ่าน `interconnect/capabilities.md` (ซึ่งแสดงรายการ skills), ตัดสินใจว่า agent นี้รับงานนั้นได้ แล้ว delegate ถ้างานอยู่นอก scope ของ skill นั้น skill ควรปฏิเสธอย่างชัดเจนแทนที่จะพยายามทำแล้วล้มเหลวแบบเงียบๆ

ไฟล์ skill แต่ละไฟล์ตอบ: capability นี้ทำงานกับอะไร ต้องการอะไรเพื่อเริ่ม สร้างอะไร รู้ว่าสำเร็จได้อย่างไร และปฏิเสธอะไรอย่างชัดเจน?

agent หนึ่งตัวมีได้หลายไฟล์ skill แต่ละ skill ค้นพบได้อิสระ วัด maturity ได้อิสระ และ cross-link ใน `interconnect/capabilities.md` ได้อิสระ

### 5.2 รูปแบบไฟล์

```markdown
---
title: <Skill Name — ชื่อที่อ่านออกได้>
aliases: []
tags:
  - type/skill
  - domain/<area>
maturity: L<n>
---

# <emoji> <Skill Name> — <คำอธิบายภาษาไทย>

> [!abstract] ประโยคเดียวที่บอกว่า skill นี้ทำอะไร

## Domain

<ไฟล์, ระบบ, API, หรือพื้นที่ปฏิบัติการที่ skill นี้ทำงานด้วย
ระบุให้ชัดเจน: ชื่อ log source, database engine, cloud service ฯลฯ>

## Inputs

- <สิ่งที่ skill ต้องการเพื่อเริ่ม — ข้อมูล, format, source, configuration>
- <อ้างอิง skill ที่เกี่ยวข้องถ้ามันสร้าง input ที่ skill นี้ใช้>
  [[<skill-name>|Display Name]]

## Outputs

- <สิ่งที่ skill สร้าง — ไฟล์, รายงาน, alert, normalized data, metrics>
- <แต่ละ output ควรเป็นรูปธรรมและตรวจสอบได้ ไม่คลุมเครือ>

## Verification Gates

- <วิธีที่ agent รู้ว่า skill นี้สำเร็จ — เกณฑ์ที่วัดได้>
- <รวม threshold เฉพาะเจาะจง, latency target, error rate ถ้าเกี่ยวข้อง>

## Out of Scope

- **<ความรับผิดชอบข้างเคียง A>** — agent หรือ skill ใดจัดการแทน
- **<ความรับผิดชอบข้างเคียง B>** — ditto
```

**frontmatter ที่ต้องมี:** `title`, `tags` (ต้องมี `type/skill` และ tag `domain/<area>` หนึ่งอัน), และ `maturity` (ต้องเป็นหนึ่งใน `L1` ถึง `L7`)

**ส่วนของ body ที่ต้องมี:** `Domain`, `Inputs`, `Outputs`, `Verification Gates`, และ `Out of Scope`

ส่วน `Out of Scope` คือการแสดงออกเชิงกลไกของ Mattaññutā (ความพอดี) — ทุก skill มีขอบเขต และการระบุมันอย่างชัดเจนทำให้ skill น่าเชื่อถือสำหรับ agent คู่ทีม skill ที่ไม่มีส่วน `Out of Scope` คือการอ้างความสามารถแบบไม่มีขอบเขต ซึ่งเป็นทั้ง design smell และ `bwoc check` evidence failure สำหรับ trust quality `vatta`

### 5.3 ระดับวุฒิภาวะ L1–L7

Maturity (Ariya-dhana 7) ประกาศอย่างซื่อสัตย์โดยผู้เขียน agent ไม่ได้คำนวณอัตโนมัติ เมื่อไม่แน่ใจให้ประกาศระดับที่ต่ำกว่า

| ระดับ | ความหมายธรรมดา |
|---|---|
| **L1** | ใช้สำเร็จครั้งแรก; ยังไม่ verified; ถือเป็น experimental |
| **L2** | ใช้หลายครั้งแล้ว; verification ไม่เป็นทางการ; pattern เริ่มชัดขึ้น |
| **L3** | verification gate ผ่านสม่ำเสมอ; เชื่อถือได้ในสภาพปกติ |
| **L4** | รับมือกับ failure mode ทั่วไปได้; จัดการ edge case ได้ดี |
| **L5** | ความสามารถด้าน mentorship — agent นำทาง agent อื่นในทักษะนี้ได้ |
| **L6** | Cross-domain transfer — ทักษะถูกนำไปใช้นอก context เดิม |
| **L7** | Canonical — agent อื่นใช้นี้เป็น reference implementation |

skill ที่สร้างใหม่ส่วนใหญ่ควรเริ่มที่ L1 หรือ L2 และเลื่อนระดับเมื่อ agent สะสม evidence ได้ การอ้าง L5 ขึ้นไปสำหรับ skill ใหม่เอี่ยมเป็น overclaim — ซึ่งเป็น threat-model violation (T-1.4) และที่ L7 ทำให้ skill กลายเป็น standard ที่ agent อื่นพึ่งพา

เลื่อนระดับ maturity เมื่อมี evidence จริง: บันทึก gate pass, การใช้งาน cross-domain, หรือเมื่อ agent คู่ทีมรับ approach ของ skill นี้เป็น reference

### 5.4 ตัวอย่างจริง

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

- Log source: application log, web server log (access/error),
  auth log (SSH, OAuth, MFA), cloud audit log (AWS CloudTrail, GCP Audit,
  Azure Monitor), container log (Docker/K8s),
  network flow log (NetFlow, VPC Flow Logs).
- Detection rule management: SIEM rule (Sigma format),
  correlation query, suppression list.
- Event pipeline: ingest → normalize → enrich → correlate → route.

## Inputs

- Log stream แบบ realtime หรือ batch จาก log aggregator
  (e.g., Elastic, Splunk, Loki, OpenSearch).
- Detection rule ที่ต้องการ deploy หรือ update
  (Sigma YAML หรือ native SIEM query).
- Time range และ source filter สำหรับการค้นหาเชิงรุก (threat hunting).
- Baseline ปกติจาก [[anomaly-detection]]
  เพื่อใช้เป็น reference ในการ correlate.

## Outputs

- Normalized event stream ที่ enrich ด้วย GeoIP, ASN,
  threat intelligence feed.
- Detection match พร้อม context:
  timestamp, source, event chain, rule ที่ trigger.
- Correlated incident package:
  event ที่เชื่อมกันในหน้าต่างเวลาเดียวกัน รวมเป็น 1 incident.
- Threat hunting report: ผลการค้นหาเชิงรุก รวมถึง negative finding
  พร้อม scope และ method.
- Rule performance metric:
  true positive / false positive count รายสัปดาห์.

## Verification Gates

- Detection rule ที่ deploy ใหม่ต้องผ่าน unit test บน sample log
  ที่รู้ผลแล้ว (expected match / expected no-match)
  ก่อน promote สู่ production.
- Log pipeline ต้องมี end-to-end latency < 60 วินาที
  จาก event เกิดจนถึง alert ready;
  วัดด้วย synthetic event ทุกชั่วโมง.
- False positive rate ของ active rule set
  ต้องไม่เกิน 20% ของ total alert volume; ตรวจรายสัปดาห์.
- Log source ทุกตัวที่ประกาศใน inventory ต้องส่ง heartbeat
  ภายใน 5 นาทีล่าสุด;
  หาก gap ให้ escalate ผ่าน [[intrusion-alerting]].

## Out of Scope

- **ไม่รวม remediation** — เมื่อพบ incident ให้ส่งต่อ zhongkui;
  agent-erlang ไม่ block IP หรือ revoke session เอง.
- **ไม่รวม forensic image** — การเก็บ disk image หรือ memory dump
  เป็นงานของ zhongkui ระหว่าง incident response.
- **ไม่รวม compliance reporting** — การสร้างรายงาน audit
  สำหรับ PCI-DSS, ISO 27001 เป็นงาน yanluo.
- **ไม่รวม log retention policy** —
  การกำหนดระยะเวลาเก็บ log เป็นการตัดสินใจระดับ infrastructure
  ไม่ใช่ detection.
```

### 5.5 หลักการ Bounded / Verifiable / Refusal

สามคุณสมบัตินี้ทำให้ skill น่าเชื่อถือสำหรับ agent คู่ทีม:

**Bounded.** ส่วน `Domain` และ `Out of Scope` ร่วมกันกำหนดขอบเขตที่ชัดเจน agent คู่ทีมที่อ่าน skill รู้ว่า delegate อะไรได้ที่นี่ และต้อง route อะไรไปที่อื่น คำอธิบาย domain ที่คลุมเครือ ("ทำงานกับ logs") และ `Out of Scope` ว่าง สร้าง agent ที่รับงานที่จะล้มเหลวแบบเงียบๆ

**Verifiable.** ส่วน `Verification Gates` ระบุเกณฑ์ความสำเร็จที่วัดได้จริง "รันสำเร็จ" ไม่ใช่ verification gate "End-to-end latency < 60 วินาที วัดด้วย synthetic event" คือ gate ที่ดี gates ที่ตรวจสอบอัตโนมัติได้ดีกว่า gates ที่ต้องใช้การตัดสินใจของมนุษย์

**Refusal.** skill ที่ปฏิเสธงานนอก scope ไม่ได้อย่างชัดเจนเป็นอันตรายใน multi-agent context ส่วน `Out of Scope` ทำให้ปฏิเสธได้อย่างสะอาด: agent อ่านส่วนนั้นและตอบว่า "นี่เป็นงานของ X ไม่ใช่ฉัน นี่คือคนที่ต้องถาม" นี่คือการแสดงออกของ Attanutata (การรู้จักตนเอง / การประกาศความสามารถ) — การรู้และบอกสิ่งที่ทำไม่ได้ สำคัญเท่ากับการรู้สิ่งที่ทำได้

---

## 6. ข้อตกลงการเขียน — ภาษาไทยใน Body, ภาษาอังกฤษใน Tags, Emoji ใน Heading

agent ใน workspace นี้ theme เป็น **เทพจีน (Chinese deities)** ข้อตกลงนี้ใช้กับ slot directory ทั้งสาม:

| ส่วนประกอบ | กฎ |
|---|---|
| เนื้อหา body ของ slot | เขียนเป็น **ภาษาไทย** — ประโยคอธิบาย, When to Apply, How to Apply, Domain ฯลฯ |
| Frontmatter และ tags | **ภาษาอังกฤษ** เสมอ — YAML keys, tag values (`type/skill`, `domain/security-monitoring`, `maturity: L3`), field names |
| Headings ในไฟล์ slot | ใส่ **emoji ที่เกี่ยวข้อง** ที่สะท้อน deity theme หรือ skill domain ของ agent |
| ชื่อ agent และ deity | ชื่อไฟล์ kebab-case (ดู [NAMING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md)); ชื่อ deity อยู่ใน persona prose |

ตัวอย่างรูปแบบ heading:

```markdown
# 👁️ All Seeing Third Eye — เปิดตาที่สาม             ← mindset
# 📋 Log Monitoring — รวบรวมและตรวจ Log แบบ Realtime  ← skill
# 🌩️ Anomaly Detection — จับ Anomaly ก่อนกลายเป็น Incident ← skill
```

emoji ใน heading เป็นอิสระจาก emoji ใน callout `> [!abstract]` — ทั้งคู่ใช้ร่วมกันในตัวอย่างด้านบนและในไฟล์ agent จริง

`AGENTS.md` เป็นข้อยกเว้น: เนื้อหา behavioral (สตริง `scopeDescription` และ `outOfScope` ที่มาจาก manifest) อาจเป็นภาษาไทย เพราะเป็นค่า role-specific ที่ resolve ตอน incarnation ส่วน heading และ instruction เชิงโครงสร้างของ `AGENTS.md` ยังคงเป็นภาษาอังกฤษ นี่แตกต่างจากไฟล์ slot ที่ body prose ทั้งหมดเป็นภาษาไทย

---

## 7. การสร้าง Stubs และขั้นตอนการทำงาน

### สร้าง stubs ด้วย bwoc new

`bwoc new` รับ flag `--mindsets` และ `--skills` ที่สร้างไฟล์ stub `.md` ใน slot directory ที่ถูกต้องตอน incarnation:

```bash
bwoc new sentinel \
  --target agents/agent-sentinel \
  --mindsets "verify-before-act,signal-over-noise,all-seeing-third-eye" \
  --skills "log-monitoring,anomaly-detection,intrusion-alerting"
```

คำสั่งนี้สร้าง:
- `agents/agent-sentinel/mindsets/verify-before-act.md`
- `agents/agent-sentinel/mindsets/signal-over-noise.md`
- `agents/agent-sentinel/mindsets/all-seeing-third-eye.md`
- `agents/agent-sentinel/skills/log-monitoring.md`
- `agents/agent-sentinel/skills/anomaly-detection.md`
- `agents/agent-sentinel/skills/intrusion-alerting.md`

แต่ละ stub มี frontmatter skeleton และ section heading ที่ถูกต้องแล้ว งานของคุณคือเติมเนื้อหา — prose, รายละเอียด, wikilinks ไปยัง skill ที่เกี่ยวข้อง

### เพิ่มทีหลัง

เพิ่ม mindset หรือ skill หลัง incarnation โดยสร้างไฟล์ `.md` ใหม่ใน directory ที่ถูกต้อง ตามรูปแบบจากส่วน 4.2 หรือ 5.2 ไม่ต้องใช้คำสั่ง CLI — ไฟล์เป็น plain Markdown และ slot directory ไม่ใช่ registry `bwoc check` จะเจอไฟล์ใหม่โดยอัตโนมัติในการ run ครั้งถัดไป

ชื่อไฟล์ต้องเป็น kebab-case (ดู [NAMING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md)) field `title` ใน frontmatter คือชื่อแสดงผลสำหรับมนุษย์; ชื่อไฟล์คือ identifier ที่เสถียรสำหรับเครื่อง

แต่ละ slot directory มี `SPEC.md` (mindsets) หรือ `README.md` (persona) ที่ document รูปแบบของ slot นั้นโดยละเอียด ดูเมื่อไม่แน่ใจ

### ตรวจสอบหลังทุกการแก้ไข

รัน `bwoc check` หลังการแก้ไขไฟล์ slot, `AGENTS.md`, หรือ `config.manifest.json` ใดๆ:

```bash
bwoc check agents/agent-<name>
```

`bwoc check` ตรวจ backend neutrality ของ `AGENTS.md`, JSON validity ของ `config.manifest.json`, และ line cap ของ `MEMORY.md` นอกจากนี้ยังอ่านไฟล์ slot เพื่อ trust evidence ต้องแก้ทุก violation ก่อน commit

### Cross-link skills ใน interconnect

หลังสร้างหรือเปลี่ยนชื่อ skill ให้อัปเดต `interconnect/capabilities.md` ด้วย agent คู่ทีมค้นพบว่า agent นี้ทำอะไรได้โดยอ่านไฟล์นั้น skill ที่ไม่ได้ list ไว้ที่นั่นจะมองไม่เห็นจากทีม

---

## 8. ดูเพิ่มเติม

- [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) — skill-slot specification ฉบับ canonical ใน framework repo
- [NAMING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md) — ข้อตกลงการตั้งชื่อไฟล์ (kebab-case, ชื่อสงวน, รูปแบบ tag key)
- [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) — 22 engineering frameworks; รายการ `principle/<pali-term>` anchors ทั้งหมด
- [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md) — agent layout, กฎ AGENTS.md, manifest fields, trust qualities, bwoc new full flag reference
- [../glossary.en.md](../glossary.en.md) — ความหมายภาษาธรรมดาของทุก Pali term และ concept ของ BWOC

---

*handbook หน้านี้ดูแลควบคู่กับ framework หากพฤติกรรมที่อธิบายที่นี่แตกต่างจาก [framework repo](https://github.com/bemindlabs/BWOC-Framework) ให้ถือว่า repo ถูกต้อง — กรุณาเปิด fix*
