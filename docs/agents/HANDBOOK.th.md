# คู่มือสร้างและดูแล Agent

> ฉบับภาษาอังกฤษ: [HANDBOOK.en.md](HANDBOOK.en.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) · อภิธาน: [../glossary.en.md](../glossary.en.md) · คู่มือ End-user: [../end-user/HANDBOOK.en.md](../end-user/HANDBOOK.en.md)

**ผู้อ่าน** คุณกำลัง incarnate, ตั้งค่า, ปรับแต่ง หรือดูแล BWOC agent คุณเป็นเจ้าของ directory ของ agent และรับผิดชอบต่อสิ่งที่ agent นั้นทำ คู่มือนี้ครอบคลุมทุกอย่างตั้งแต่การเรียก `bwoc new` ครั้งแรกไปจนถึงการดำเนินงานประจำวัน การประกาศความน่าเชื่อถือ การเข้าร่วมทีม และการ retire ในที่สุด

**แหล่งข้อมูลหลัก** สเปคที่เป็น canonical อยู่ใน framework repo เมื่อมีความขัดแย้ง repo ถือเป็นถูก — คู่มือนี้มีข้อผิดพลาด: [agent-template AGENTS.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md) · [INCARNATION.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/INCARNATION.en.md) · [SIGNING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) · [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) · [FLEET-GOVERNANCE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md) · [NAMING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md) · [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md)

---

## สารบัญ

1. [Agent คืออะไร](#1-agent-คืออะไร)
2. [โครงสร้าง Directory พร้อมคำอธิบาย](#2-โครงสร้าง-directory-พร้อมคำอธิบาย)
3. [กฎ AGENTS.md — เจาะลึก](#3-กฎ-agentsmd--เจาะลึก)
4. [Slot Files — รูปแบบ Obsidian](#4-slot-files--รูปแบบ-obsidian)
   - [4.1 persona/](#41-persona)
   - [4.2 mindsets/](#42-mindsets)
   - [4.3 skills/](#43-skills)
   - [4.4 interconnect/](#44-interconnect)
   - [4.5 memories/](#45-memories)
5. [Manifest — config.manifest.json](#5-manifest--configmanifestjson)
6. [การ Incarnate ด้วย bwoc new](#6-การ-incarnate-ด้วย-bwoc-new)
   - [6.1 ตาราง Flag ทั้งหมด](#61-ตาราง-flag-ทั้งหมด)
   - [6.2 TTY Prompting](#62-tty-prompting)
   - [6.3 ตัวอย่างการใช้งาน](#63-ตัวอย่างการใช้งาน)
   - [6.4 Checklist หลัง Incarnate](#64-checklist-หลัง-incarnate)
7. [bwoc check — ทุก Failure Mode](#7-bwoc-check--ทุก-failure-mode)
8. [Trust และการ Sign](#8-trust-และการ-sign)
   - [8.1 เจ็ดคุณสมบัติ (Kalyanamitta-7)](#81-เจ็ดคุณสมบัติ-kalyanamitta-7)
   - [8.2 กฎ Evidence](#82-กฎ-evidence)
   - [8.3 Keygen และการ Sign](#83-keygen-และการ-sign)
9. [Lifecycle — stop / start / retire](#9-lifecycle--stop--start--retire)
10. [การ Operate Agent ที่กำลังทำงาน](#10-การ-operate-agent-ที่กำลังทำงาน)
11. [ทีม (Sangha)](#11-ทีม-sangha)
12. [ข้อกำหนดการเขียนของ Workspace นี้](#12-ข้อกำหนดการเขียนของ-workspace-นี้)
13. [ข้อผิดพลาดที่พบบ่อยและวิธีแก้ไข](#13-ข้อผิดพลาดที่พบบ่อยและวิธีแก้ไข)

---

## 1. Agent คืออะไร

ใน BWOC **หนึ่ง folder = หนึ่ง agent** ทุก agent คือ directory ที่ดูแลตนเองได้ cloned จาก agent template มันเก็บ identity, memory, หลักการตัดสินใจ, skills, และ configuration การประสานงานระหว่าง agent — ทั้งหมดเป็นไฟล์ธรรมดา ไม่มีฐานข้อมูล ไม่มี state กลาง

Framework นี้ **backend-neutral**: agent directory เดียวกันทำงานบน Claude Code, Codex, Kimi, Antigravity หรือ OpenAI-compatible endpoint ที่ host เองผ่าน `bwoc-harness` โดยไม่ต้อง lock-in vendor ใด ๆ ใน agent เอง — การเลือก vendor เป็นเรื่อง runtime ที่แก้ไขใน `config.manifest.json` และ workspace registry

Lifecycle ของ agent ผ่านสามระยะที่ framework เรียกว่า *อุปปาทะ · ฐิติ · วยะ* — เกิด, ทำงาน, และ retire ไม่จำเป็นต้องใช้คำบาลี คิดง่าย ๆ ว่า: สร้าง, ดำเนินการ, ปลดระวาง ทุก BWOC object เดินตาม arc นี้

Agent ถูกจัดการผ่าน `bwoc` CLI เท่านั้น Registry อยู่ที่ `.bwoc/agents.toml` ใน workspace root **อย่าแก้ไข registry ด้วยมือหรือเปลี่ยนชื่อ directory ของ agent เอง** ใช้ CLI สำหรับการเปลี่ยนแปลงโครงสร้างทั้งหมดเพื่อให้ registry สอดคล้องกัน

---

## 2. โครงสร้าง Directory พร้อมคำอธิบาย

```
agents/agent-<name>/
│
├── AGENTS.md                   # แหล่งข้อมูลเดียว (Single Source of Truth) สำหรับ
│                               # ทุก LLM backend. Plain Markdown เท่านั้น —
│                               # ห้าม YAML frontmatter, ห้าม wikilinks, ห้าม callouts
│                               # ทุก backend อ่านไฟล์นี้โดยตรง
│
├── CLAUDE.md -> AGENTS.md      # Symlink. การแก้ไขไฟล์นี้แก้ไข AGENTS.md
├── AGY.md    -> AGENTS.md      # Symlink สำหรับ Antigravity backend
├── CODEX.md  -> AGENTS.md      # Symlink สำหรับ Codex/OpenAI backend
├── KIMI.md   -> AGENTS.md      # Symlink สำหรับ Kimi/Moonshot backend
├── OLLAMA.md -> AGENTS.md      # Symlink สำหรับ self-hosted Ollama
├── OPENAI.md -> AGENTS.md      # Symlink สำหรับ OpenAI-compatible endpoint
│                               # เพิ่ม backend ใหม่: ln -s AGENTS.md <BACKEND>.md
│
├── config.manifest.json        # Identity + capabilities แบบ machine-readable
│                               # ต้องเป็น valid JSON. bwoc check validate มัน
│
├── MEMORY.md                   # Memory index. ขีดจำกัดที่บังคับ: 200 บรรทัด
│                               # หนึ่ง bullet ต่อ memory file ใน memories/
│                               # bwoc check บังคับ line cap
│
├── README.md                   # คำอธิบายสำหรับมนุษย์ของ agent นี้
│                               # รูปแบบ Obsidian (frontmatter + wikilinks ได้)
│
├── conventions.md              # กฎการตั้งชื่อ, placeholder, และ type definitions
├── neutrality.md               # เหตุผลที่ backend-neutrality สำคัญ; design notes
├── task-log.jsonl              # JSONL task record แบบ append-only
│
├── persona/                    # SLOT: agent เป็นใคร (WHO)
│   └── README.md               # Scope, anti-scope, thematic identity
│                               # รูปแบบ Obsidian. เนื้อหาเป็นภาษาไทย +
│                               # frontmatter ภาษาอังกฤษใน workspace นี้
│
├── mindsets/                   # SLOT: agent คิดอย่างไร (HOW)
│   ├── SPEC.md                 # Slot spec (template ส่งมาให้; เก็บเป็น reference)
│   └── <name>.md               # หนึ่ง mindset ต่อไฟล์ รูปแบบ Obsidian
│                               # tags: type/mindset · principle/<pali-term>
│
├── skills/                     # SLOT: agent ทำอะไรได้ (WHAT)
│   ├── SPEC.md                 # Slot spec
│   └── <name>.md               # หนึ่ง skill ต่อไฟล์ รูปแบบ Obsidian
│                               # tags: type/skill · domain/<area>
│                               # frontmatter field: maturity: L1..L7
│
├── interconnect/               # SLOT: configuration การประสานงานระหว่าง agent
│   ├── capabilities.md         # Machine-readable skill registry สำหรับ peer
│   ├── messaging.md            # Envelope schema + Saraniya-dhamma 6 norms
│   ├── trust.md                # Kalyanamitta-7 trust profile spec
│   ├── routing.md              # Route declarations สำหรับ cross-workspace peers
│   └── sangha.md               # ข้อกำหนดการเป็นสมาชิกทีมและ task flow
│
├── memories/                   # SLOT: สิ่งที่ agent จำได้ (WHAT IT KNOWS)
│   ├── README.md               # Memory system spec
│   └── <slug>.md               # หนึ่ง memory ต่อไฟล์ ต้องมี YAML frontmatter
│
├── docs/                       # เอกสารเพิ่มเติมสำหรับ agent นี้
│   ├── en/                     # เอกสารภาษาอังกฤษ (*.en.md)
│   └── th/                     # เอกสารภาษาไทย parity (*.th.md)
│
├── scripts/                    # Helper scripts เฉพาะ agent
│   ├── incarnate.sh            # Template-level clone helper (เก็บไว้เป็น reference)
│   └── check-agent-neutrality.sh  # Local neutrality check (ก่อน bwoc check)
│
├── .bwoc/                      # Runtime state (gitignored โดย bwoc init)
│   ├── agent.key               # Ed25519 private key (0600) เขียนโดย bwoc trust --keygen
│   ├── inbox.jsonl             # Inbound message queue (JSONL)
│   └── agent.log               # Daemon stderr
│
└── .claude/                    # Claude Code project config สำหรับ agent นี้
    ├── settings.json           # Hooks config (inbox-auto-reply Stop hook)
    └── hooks/
        └── inbox-auto-reply.sh # Auto-reply hook สำหรับ agent-to-agent messaging
```

หลักการสำคัญ: **ทุกอย่างใต้ `.bwoc/` คือ runtime state** และถูก gitignore ทุกอย่างอื่นคือ versioned configuration Identity, พฤติกรรม, และ memory ของ agent อยู่ใน plain Markdown — อ่านได้, diff ได้, และ audit ได้โดยไม่ต้องใช้ tooling พิเศษ

---

## 3. กฎ AGENTS.md — เจาะลึก

`AGENTS.md` คือแหล่งข้อมูลเดียวสำหรับทุก LLM backend ที่สามารถรัน agent นี้ มันคือไฟล์ที่ backend อ่านเมื่อ agent เริ่มต้น ไฟล์ entry point ของ backend ทั้งหมด (`CLAUDE.md`, `AGY.md`, `CODEX.md`, `KIMI.md`, `OLLAMA.md`, `OPENAI.md`) คือ **symlinks** ที่ resolve ไปยัง `AGENTS.md` การแก้ไขไฟล์ใดก็ตามในนั้นก็คือการแก้ไข `AGENTS.md`

### AGENTS.md ต้องเป็นอะไร

- **Plain Markdown เท่านั้น** ห้ามมี YAML frontmatter block ที่ด้านบน ห้ามขึ้นต้นด้วย `---`
- **ห้าม wikilinks** Wikilinks (`[[...]]`) เป็นฟีเจอร์เฉพาะ Obsidian Backend LLM บางตัวจะ render เป็น text ตัวอักษรหรือไม่สามารถ parse cross-references ได้ ใช้ plain prose หรือ plain Markdown links แทน
- **ห้าม Obsidian callouts** ไวยากรณ์ `> [!warning]`, `> [!note]`, `> [!abstract]` เป็น Obsidian-specific บาง backend ละเว้น prefix `[!type]` เงียบ ๆ บาง backend render ตัวอักษรตรง ๆ ใช้ blockquote ธรรมดาหรือ bold text แทน
- **ห้าม hardcode model IDs หรือ vendor names ในเนื้อหาพฤติกรรม** อย่าเขียน "ใช้ claude-opus-4 สำหรับขั้นตอนนี้" หรือ "Antigravity รองรับ X แตกต่างกัน" ค่าที่แตกต่างตาม backend ต้องอยู่ใน `config.manifest.json` และอ้างถึงผ่าน `{{camelCase}}` placeholders
- **ใช้ placeholders สำหรับค่าที่ปรับได้ทั้งหมด** ใช้ `{{agentId}}`, `{{agentRole}}`, `{{primaryModel}}`, `{{fallbackModel}}`, `{{scopeDescription}}`, `{{primaryCapability}}`, `{{lintCmd}}`, `{{formatCmd}}`, `{{testCmd}}`, `{{buildCmd}}`, `{{memoryPath}}`, `{{worktreeBase}}`, `{{deepMemoryCmd}}` คำสั่ง `bwoc new` จะแทนที่ placeholder เหล่านี้เมื่อ copy template รูปแบบ `{{...}}` ที่ยังไม่ถูกแทนที่คือการละเมิด `bwoc check`

### AGENTS.md มีเนื้อหาอะไร

[สเปค canonical](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md) จัด `AGENTS.md` เป็น sections ที่มีหมายเลข:

| Section | เนื้อหา |
|---|---|
| 0 — Backend Registration | ตาราง backend symlinks; คำแนะนำในการเพิ่ม backend ใหม่ |
| 1 — Identity (Samma-ditthi / สัมมาทิฏฐิ — ความเห็นชอบ) | agentId, role, primary capability, scope, anti-scope, ฐานการคิด, protocol การประกาศ capability |
| 2 — Task Planning (Samma-sankappa / สัมมาสังกัปปะ — ความดำริชอบ) | วงจร Four Noble Truths สำหรับงาน; format ของ task-log.jsonl; ความมีวินัยด้าน scope |
| 3 — Communication (Samma-vaca / สัมมาวาจา — การพูดชอบ) | Protocol ข้อความระหว่าง agent; โครงสร้างข้อความ error; โทนเสียงต่อผู้ใช้ |
| 4 — Worktree Discipline (Samma-kammanta / สัมมากัมมันตะ — การกระทำชอบ) | กฎ isolation; การตั้งชื่อ branch; วินัยการ commit; การ cleanup หลัง merge |
| 5 — Trust & Neutrality (Samma-ajiva / สัมมาอาชีวะ — การเลี้ยงชีพชอบ) | คำประกาศ backend-neutrality; script การตรวจสอบ; threat model |
| 6 — Verification Gates (Samma-vayama / สัมมาวายามะ — ความเพียรชอบ) | gate commands สี่ตัว (lint, format, test, build) และเวลาที่ใช้แต่ละตัว |
| 7 — Memory System (Samma-sati / สัมมาสติ — ความระลึกชอบ) | วงจร Tier 1 file-based memory; format ไฟล์ memory; สิ่งที่ควรบันทึก vs ทิ้ง; ขีดจำกัด 200 บรรทัด; Tier 2 deep backend |
| 8 — Focus & Stability (Samma-samadhi / สัมมาสมาธิ — สมาธิชอบ) | Session lifecycle; configuration reference; การจำกัดงานพร้อมกัน; การสลับโปรเจกต์; ขอบเขต module; สไตล์ commit |
| 9 — Baseline Security (Sila 5 / ศีล 5) | ห้ากฎความปลอดภัยที่ไม่มีทางต่อรอง |
| 10 — Observability (Satipatthana 4 / สติปัฏฐาน 4) | สี่ฐานที่ต้องติดตามตลอด session |
| 11 — Self-Improvement (Panna 3 / ปัญญา 3) | วงจรสะท้อนหลังงาน |
| Appendices | Placeholder reference; quick-start checklist; document map |

ชื่อ section ในวงเล็บคือคำบาลีจาก Noble Eightfold Path (อริยมรรคมีองค์แปด) — เป็นเครื่องมือการคิดเชิงวิศวกรรม ไม่ใช่คำสอนทางศาสนา แต่ละคำ map กับ concern จริง: สัมมาทิฏฐิหมายถึงการมี context ที่ถูกต้อง; สัมมากัมมันตะหมายถึง git hygiene; สัมมาวายามะหมายถึงการรัน verification gates ดูคำอธิบายภาษาไทยของทุก term ได้ที่ [glossary](../glossary.en.md)

### กฎ Symlink

เมื่อได้รับ agent จากคนอื่น ตรวจสอบ symlinks:

```bash
ls -la agents/agent-<name>/CLAUDE.md   # ควรแสดง -> AGENTS.md
ls -la agents/agent-<name>/AGY.md      # เหมือนกัน
```

ถ้า backend ไฟล์ใดเป็น **regular file ที่มีเนื้อหาของตัวเอง** มันได้แยกออกจาก `AGENTS.md` แล้ว นั่นคือการละเมิด neutrality ลบมันและสร้าง symlink ใหม่:

```bash
rm agents/agent-<name>/CLAUDE.md
ln -s AGENTS.md agents/agent-<name>/CLAUDE.md
```

ในการเพิ่ม backend ที่ template ไม่ได้ส่งมา:

```bash
cd agents/agent-<name>
ln -s AGENTS.md OLLAMA.md
```

ไม่ต้องเปลี่ยนโค้ด ไม่ต้อง update manifest — `bwoc spawn --backend ollama` จะหา `OLLAMA.md` ผ่าน symlink

---

## 4. Slot Files — รูปแบบ Obsidian

Slot files เป็น **ตรงกันข้าม** กับ `AGENTS.md`: ใช้ Obsidian Markdown เต็มรูปแบบ — YAML frontmatter, wikilinks, และ callouts ไม่เพียงแต่อนุญาต แต่ยังคาดหวัง พวกมันคือเอกสารที่มนุษย์อ่านได้ที่อยู่คู่กับ instructions ของ agent **ไม่ถูก** อ่านโดย LLM backend โดยตรงตอน runtime — พวกมันแจ้ง evidence rules ของ `bwoc check` และให้ context สำหรับ operator

### 4.1 persona/

`persona/README.md` กำหนดว่า agent นี้เป็นใคร: identity เชิงธีม, scope (สิ่งที่ทำ), และ anti-scope (สิ่งที่ปฏิเสธโดยชัดแจ้ง) ไฟล์นี้เป็นแหล่ง evidence หลักสำหรับคุณสมบัติ trust หลายตัว (`piyo`, `vatta`, `noCatthana` — ดู Section 8)

sections ที่ต้องมีอย่างน้อย: **Scope** (ไม่ว่าง, อธิบาย task ที่ delegate ได้จริง) และ **Anti-scope** (มีรายการ "จะปฏิเสธ" อย่างน้อยหนึ่งรายการโดยชัดแจ้ง)

ใน workspace นี้ ไฟล์ persona เขียนเนื้อหาเป็นภาษาไทยโดยมี frontmatter และ tags เป็นภาษาอังกฤษ — ดู Section 12

### 4.2 mindsets/

แต่ละ mindset คือไฟล์ `.md` หนึ่งไฟล์ใต้ `mindsets/` Mindset คือ decision frame ที่มีชื่อ, นำกลับมาใช้ได้, ที่ agent ใช้ในสถานการณ์ประเภทเฉพาะ มันตอบคำถาม: ฉันใช้สิ่งนี้เมื่อไร? ฉันใช้มันอย่างไร? ฉัน **ไม่** ใช้มันเมื่อไร?

Frontmatter ที่ต้องการ:

```yaml
---
title: <Mindset Name>
aliases: []
tags:
  - type/mindset
  - principle/<pali-dhamma-term>
---
```

Tag `principle/<pali-dhamma-term>` คือ anchor แต่ละ mindset ตั้งชื่อหลักบาลีหนึ่งตัวเป็น anchor คำบาลีที่ใช้ใน tags เป็นรูปแบบ romanized ธรรมดา (ไม่มี diacritics ใน tag keys): `appamada`, `yoniso-manasikara`, `anatta`, `samanatatta`, `mattanutata`, `sila` เป็นต้น ดู [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) สำหรับรายการ 22 frameworks ทั้งหมด

Sections ที่ต้องการในเนื้อหา:

| Section | จุดประสงค์ |
|---|---|
| `## When to Apply` | สถานการณ์เฉพาะที่ trigger mindset นี้ |
| `## How to Apply` | ขั้นตอนหรือ heuristics ที่เป็นรูปธรรม |
| `## When NOT to Apply` | ขอบเขต — การ over-apply mindset ใด ๆ ก็ตามกลายเป็น anti-pattern |
| `## Related Principles` | Cross-links ไปยัง mindsets หรือ skills อื่น ๆ |

หนึ่ง mindset ต่อไฟล์ อย่ารวม decision frames หลายอันไว้ในไฟล์เดียว

Trust quality `bhavaniyo` (ดู Section 8) ต้องการ mindset อย่างน้อยหนึ่งอันที่มีชื่อหรือเนื้อหาที่อ้างถึงการปรับปรุง/การตรวจสอบ/right-amount Mindsets ที่มี tags เช่น `principle/yoniso-manasikara` หรือ `principle/mattanutata` ตรงตามเกณฑ์นี้

### 4.3 skills/

แต่ละ skill คือไฟล์ `.md` หนึ่งไฟล์ใต้ `skills/` Skill คือ capability ที่เป็นรูปธรรมและมีขอบเขต: "log monitoring", "anomaly detection", "schema migration" Skills คือสิ่งที่ agent *execute*; mindsets คือวิธีที่มัน *คิด*

Frontmatter ที่ต้องการ:

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

Tag `domain/<area>` จัดหมวดหมู่ domain ของ skill (ตัวอย่าง: `domain/security-monitoring`, `domain/database`, `domain/infrastructure`, `domain/testing`) field `maturity` ต้องการและต้องเป็นหนึ่งในเจ็ดระดับด้านล่าง

#### ระดับ Maturity (Ariya-dhana 7 / อริยธนะ 7)

เจ็ดระดับนี้วัดความเจริญงอกงามของ capability — ประกาศอย่างซื่อสัตย์โดย agent author ไม่ใช่คำนวณอัตโนมัติ:

| ระดับ | ความหมายที่เข้าใจง่าย |
|---|---|
| L1 | ใช้สำเร็จครั้งแรก; ยังไม่ได้ verify; ถือว่าเป็น experimental |
| L2 | ใช้หลายครั้ง; การตรวจสอบไม่เป็นทางการ; patterns กำลังเกิดขึ้น |
| L3 | Verification gates ผ่านอย่างสม่ำเสมอ; เชื่อถือได้ในสภาวะปกติ |
| L4 | ทนทานต่อ failure modes ทั่วไป; จัดการ edge cases ได้ดี |
| L5 | Mentorship capability — agent สามารถแนะนำ agent อื่นใน skill นี้ได้ |
| L6 | Cross-domain transfer — skill ถูกนำไปใช้นอกบริบทดั้งเดิมแล้ว |
| L7 | Canonical — agent อื่นใช้อันนี้เป็น reference implementation |

การอ้างระดับสูงกว่าความเป็นจริงคือการ overclaim และเป็นการละเมิด threat model (T-1.4 ใน THREAT-MODEL) เมื่อไม่แน่ใจ ประกาศระดับต่ำกว่า

Sections เนื้อหาที่ต้องการ:

| Section | เนื้อหา |
|---|---|
| `## Domain` | ไฟล์, ระบบ, หรือ areas ที่ skill นี้ดำเนินการบน |
| `## Inputs` | สิ่งที่ skill ต้องการเพื่อเริ่มต้น |
| `## Outputs` | สิ่งที่ skill ผลิต |
| `## Verification Gates` | Agent รู้ว่าสำเร็จอย่างไร |
| `## Out of Scope` | สิ่งที่ skill นี้ **ไม่ทำ**; skill ใกล้เคียงใดที่จัดการส่วนต่างนั้น |

Trust quality `garu` ต้องการ skill หรือ mindset stub อย่างน้อยหนึ่งอัน Section `Out of Scope` เป็นสิ่งจำเป็น — มันคือการแสดงออกเชิงกลไกของ Mattanutata (right amount / ความพอดี)

Cross-link ทุก skill จาก `interconnect/capabilities.md` เพื่อให้ peer agents สามารถค้นพบมันสำหรับ delegation

### 4.4 interconnect/

`interconnect/` เก็บ configuration การประสานงานระหว่าง agent สำหรับ agent นี้:

- `capabilities.md` — skill registry แบบ machine-readable Peer agents อ่านอันนี้เพื่อรู้ว่า agent นี้รับอะไรได้เพื่อ delegation
- `messaging.md` — envelope schema และ Saraniya-dhamma 6 (หกเงื่อนไขแห่งความสมานฉันท์ / สาราณียธรรม 6) ที่แสดงเป็น engineering norms สำหรับการพูดระหว่าง agent
- `trust.md` — Kalyanamitta-7 trust profile specification (คุณสมบัติที่ประกาศของ agent นี้และคุณสมบัติที่ต้องการจาก peers)
- `routing.md` — Cross-workspace peer route declarations
- `sangha.md` — ข้อกำหนดการเป็นสมาชิกทีมและ task flow

ไฟล์เหล่านี้ใช้รูปแบบ Obsidian (frontmatter + wikilinks) พวกมันคือเอกสารสำหรับ human operators และ evidence checks ของ framework — ไม่ใช่ LLM instructions โดยตรง

### 4.5 memories/

`memories/` เก็บ Tier 1 file-based memories — หนึ่ง `.md` ต่อ memory แต่ละไฟล์มี YAML frontmatter:

```yaml
---
name: <descriptive-slug>
description: <one-line relevance hook>
type: user | feedback | project | reference
created: 2026-05-22
updated: 2026-05-22
---
```

Format เนื้อหา: content, แล้ว `**Why:**` (motivation), แล้ว `**How to apply:**` (เมื่อไร/ที่ไหน)

`MEMORY.md` ใน agent root คือ index — หนึ่ง bullet ต่อ memory file, ขีดจำกัดที่บังคับ 200 บรรทัด `bwoc check` บังคับ cap เมื่อ index ถึง 200 บรรทัด ให้ prune entries ที่ไม่ทันสมัย (อนิจจา / anicca — ความไม่เที่ยง; ปล่อยสิ่งที่ไม่มีประโยชน์อีกต่อไป)

สิ่งที่ควรบันทึก: การตัดสินใจที่ไม่ชัดเจนและเหตุผล; approaches ที่ validated โดยผู้ใช้; การแก้ไข; ตำแหน่ง external resources

สิ่งที่ไม่ควรบันทึก: code patterns ที่ derive ได้จากการอ่านโค้ด; git history; สิ่งที่อยู่ใน `AGENTS.md` หรือ `conventions.md` แล้ว; ephemeral session state

---

## 5. Manifest — config.manifest.json

`config.manifest.json` คือบัตรประจำตัว machine-readable ของ agent ต้องเป็น **valid JSON** — `bwoc check` parse มันและ fail hard กับ JSON syntax errors มันอยู่ใน agent root

### ตาราง Field ทั้งหมด

| Field | Type | Required | คำอธิบาย | ตัวอย่าง |
|---|---|---|---|---|
| `name` | string | ใช่ | ชื่อ agent, kebab-case, ไม่มี `agent-` prefix | `"erlang"` |
| `agentId` | string | ใช่ | Agent identifier เต็ม; ต้องเป็น `agent-<name>` เสมอ | `"agent-erlang"` |
| `version` | string | ใช่ | Manifest schema version | `"2.0"` |
| `agentRole` | string | ใช่ | คำอธิบาย role หนึ่งบรรทัด; เติม `{{agentRole}}` ใน AGENTS.md | `"security monitoring & detection sentinel"` |
| `primaryModel` | string | ใช่ | Primary LLM model identifier หรือ `"auto"` ให้ harness เลือกจาก `autoModels` ตอน runtime | `"claude-opus-4-7"` |
| `fallbackModel` | string | ไม่ | Fallback model ถ้า primary ไม่พร้อมใช้ | `"claude-sonnet-4-6"` |
| `autoModels` | string[] | ไม่ | Candidate model pool ตาม preference order; ใช้เมื่อ `primaryModel` เป็น `"auto"`; สำหรับ OpenAI-compatible endpoints วาง frontier models ไว้ก่อน | `["gpt-5.5", "gpt-4o"]` |
| `reasoningEffort` | string | ไม่ | Hint ควบคุม effort ของ backend; key เป็น neutral แต่ value space ขึ้นกับ backend (Claude: `low\|medium\|high\|xhigh\|max`; OpenAI-compatible: `low\|medium\|high`); backends ที่ไม่มี effort control ละเว้น | `"high"` |
| `baseUrl` | string | conditional | ต้องการสำหรับ backend `ollama` และ `openai-compatible`; harness ส่งค่านี้เป็น `--endpoint`; `bwoc spawn` return error ชัดเจนถ้าไม่มีสำหรับ backends เหล่านั้น | `"http://192.168.1.113:11434/v1"` |
| `memoryPath` | string | ใช่ | Tier 1 memory directory แบบ file-based, relative กับ agent root | `"memories/"` |
| `sessionsPath` | string | ไม่ | Session data directory สำหรับ Tier 2 memory mining | `"~/.claude/projects/"` |
| `deepMemoryCmd` | string | ไม่ | Tier 2 deep memory CLI command; invoke ด้วย subcommands `wake-up`, `search`, `mine` | `"mempalace"` |
| `lintCmd` | string | ใช่ | Lint / static analysis command; Samvara verification gate | `"cargo clippy -- -D warnings"` |
| `formatCmd` | string | ใช่ | Format check command; Pahana verification gate | `"cargo fmt --check"` |
| `testCmd` | string | ใช่ | Test command; Bhavana verification gate | `"cargo test"` |
| `buildCmd` | string | ใช่ | Build command; Anurakkhana verification gate | `"cargo build --release"` |
| `worktreeBase` | string | ไม่ | Base directory สำหรับ git worktrees; default `/tmp` | `"/tmp"` |
| `scopeDescription` | string | ใช่ (เติม placeholder) | หนึ่งบรรทัด "agent นี้ทำ X"; เติม `{{scopeDescription}}` | `"เฝ้าระวัง log/metric, ตรวจจับ anomaly, แจ้งเตือน intrusion"` |
| `outOfScope` | string | ใช่ (เติม placeholder) | หนึ่งบรรทัด "agent นี้ไม่ทำ Y"; เติม anti-scope ใน AGENTS.md | `"แก้โค้ด, เขียนรายงาน audit, ลงมือ remediation"` |
| `primaryCapability` | string | ไม่ | คำอธิบาย capability ที่ยาวกว่า; เติม `{{primaryCapability}}`; default เป็นค่า `agentRole` ถ้าไม่ระบุ | `"ตาที่สามมองทะลุทุก traffic"` |
| `trust` | object | ไม่ | Kalyanamitta-7 trust block; ดูด้านล่าง | ดู Section 8 |
| `trust.schemaVersion` | integer | ใช่ (ถ้า trust มี) | Schema version สำหรับ forward-compat; ปัจจุบัน `1` | `1` |
| `trust.declared` | object | ใช่ (ถ้า trust มี) | เจ็ด boolean qualities ที่ agent นี้อ้างว่าตรงตาม; key ที่ไม่มีถือว่าเป็น `false` | ดู Section 8 |
| `trust.declared.piyo` | boolean | ไม่ | Likeable / น่า delegate ไปให้ | `false` |
| `trust.declared.garu` | boolean | ไม่ | Respected / มี demonstrated competency | `false` |
| `trust.declared.bhavaniyo` | boolean | ไม่ | Admirable / ช่วยคนอื่นปรับปรุง | `false` |
| `trust.declared.vatta` | boolean | ไม่ | พูดความจริงที่มีประโยชน์; ซื่อสัตย์เรื่องข้อจำกัด | `false` |
| `trust.declared.vacanakkhamo` | boolean | ไม่ | Listener ที่อดทน; ยอมรับ feedback | `false` |
| `trust.declared.gambhira` | boolean | ไม่ | อธิบายความลึก; ยึดกับ canonical philosophy | `false` |
| `trust.declared.noCatthana` | boolean | ไม่ | ไม่นำทางไปสู่ทางที่ไม่สมควร; มีรายการ refusal โดยชัดแจ้งใน anti-scope | `false` |
| `trust.requiredTrust` | string[] | ไม่ | คุณสมบัติที่ต้องการจาก peers ที่ต้องการส่งข้อความถึง agent นี้; template scaffold default เป็น `["vatta", "noCatthana"]` | `["vatta", "noCatthana"]` |
| `trust.mode` | string | ไม่ | Refusal mode: `"off"` \| `"warn"` \| `"refuse"`; ไม่มี field ใช้กฎ v1 compat: requiredTrust ว่าง → off, ไม่ว่าง → refuse; `"warn"` เป็น opt-in อย่างเคร่งครัด | `"warn"` |
| `trust.signingPublicKey` | string | ไม่ | Ed25519 public key (hex, 64 chars); เขียนโดย `bwoc trust --keygen`; ใช้สำหรับ envelope verification | `"12e4bac908f6..."` |

---

## 6. การ Incarnate ด้วย bwoc new

`bwoc new` ทำ "อุปปาทะ" (การเกิด) — copy agent template ไปยัง directory ใหม่, แทนที่ placeholders, register agent ใน `.bwoc/agents.toml`, และสร้าง backend symlinks

**อย่า hand-copy template directory** CLI จัดการ registry registration, placeholder substitution, symlink creation, และ stub generation แบบ atomic การ hand-copy ผลิต agent ที่ไม่ถูก register ซึ่ง `bwoc check --all` จะไม่พบ

### 6.1 ตาราง Flag ทั้งหมด

```
bwoc new <NAME> [OPTIONS]
```

| Flag | Type | Required | TTY-prompted? | คำอธิบาย |
|---|---|---|---|---|
| `<NAME>` | positional | ใช่ | ไม่ | ชื่อ agent ใน kebab-case (เช่น `database-schema`) Directory จะเป็น `agent-<NAME>/` |
| `--target <TARGET>` | path | ไม่ | ไม่ | Destination directory Default: `../agent-<name>/` relative กับ template ใน workspace มาตรฐาน ต้องส่ง `--target agents/agent-<name>` เสมอ |
| `--template <TEMPLATE>` | path | ไม่ | ไม่ | Path ไปยัง template directory Default: auto-detect `modules/agent-template/` จาก cwd ancestors |
| `--backend <BACKEND>` | enum | ไม่ | ไม่ | Primary backend สำหรับ workspace registry Default: `claude` ค่า: `claude` \| `antigravity` \| `codex` \| `kimi` \| `ollama` \| `openai-compatible` |
| `--role <ROLE>` | string | ไม่ | **ใช่** (ถ้าไม่มีบน TTY) | คำอธิบาย role หนึ่งบรรทัด เขียนใน manifest เป็น `agentRole` |
| `--primary-model <PRIMARY_MODEL>` | string | ไม่ | **ใช่** (ถ้าไม่มีบน TTY) | Primary LLM model identifier เขียนใน manifest เป็น `primaryModel` |
| `--fallback-model <FALLBACK_MODEL>` | string | ไม่ | ไม่ | Fallback LLM model identifier Optional อย่างแท้จริง |
| `--memory-path <MEMORY_PATH>` | path | ไม่ | ไม่ | File-based memory directory Default: `memories/` |
| `--sessions-path <SESSIONS_PATH>` | path | ไม่ | ไม่ | Session data directory สำหรับ Tier 2 mining Optional อย่างแท้จริง |
| `--deep-memory-cmd <DEEP_MEMORY_CMD>` | string | ไม่ | ไม่ | Tier 2 memory CLI command Optional อย่างแท้จริง |
| `--lint-cmd <LINT_CMD>` | string | ไม่ | **ใช่** (ถ้าไม่มีบน TTY) | Lint command สำหรับ Samvara verification gate |
| `--format-cmd <FORMAT_CMD>` | string | ไม่ | **ใช่** (ถ้าไม่มีบน TTY) | Format command สำหรับ Pahana verification gate |
| `--test-cmd <TEST_CMD>` | string | ไม่ | **ใช่** (ถ้าไม่มีบน TTY) | Test command สำหรับ Bhavana verification gate |
| `--build-cmd <BUILD_CMD>` | string | ไม่ | **ใช่** (ถ้าไม่มีบน TTY) | Build command สำหรับ Anurakkhana gate |
| `--worktree-base <WORKTREE_BASE>` | path | ไม่ | ไม่ | Base directory สำหรับ git worktrees Default: `/tmp` Optional อย่างแท้จริง |
| `--scope <SCOPE>` | string | ไม่ | ไม่ | หนึ่งบรรทัด "agent นี้ทำ X" เติม `{{scopeDescription}}` และ `scopeDescription` ใน manifest |
| `--out-of-scope <OUT_OF_SCOPE>` | string | ไม่ | ไม่ | หนึ่งบรรทัด "agent นี้ไม่ทำ Y" เติม anti-scope ใน AGENTS.md และ `outOfScope` ใน manifest |
| `--primary-capability <PRIMARY_CAPABILITY>` | string | ไม่ | ไม่ | คำอธิบาย capability ที่ยาวกว่า เติม `{{primaryCapability}}` Default เป็นค่า `--role` ถ้าไม่ระบุ |
| `--mindsets <MINDSETS>` | string | ไม่ | ไม่ | ชื่อ mindset kebab-case ที่คั่นด้วย comma (เช่น `verify-before-act,right-amount`) สร้าง stub `.md` หนึ่งไฟล์ต่อชื่อใน `mindsets/` |
| `--skills <SKILLS>` | string | ไม่ | ไม่ | ชื่อ skill kebab-case ที่คั่นด้วย comma สร้าง stub `.md` หนึ่งไฟล์ต่อชื่อใน `skills/` |
| `--json` | flag | ไม่ | ไม่ | Emit JSON `{ agent_id, target, registered_in, symlinks, mindset_stubs, skill_stubs, persona_filled }` แทน human-readable report มีประโยชน์สำหรับการ setup หลาย agent แบบ script |
| `--lang <LANG>` | string | ไม่ | ไม่ | ภาษา CLI output Precedence: `--lang` > `BWOC_LANG` env > `$LANG` > `en` fallback |
| `-h, --help` | flag | ไม่ | ไม่ | Print help |

### 6.2 TTY Prompting

เมื่อรันบน interactive TTY และ flags บางตัวถูกละไว้ `bwoc new` จะ prompt:

- `--role` — prompt "Agent role (one line):"
- `--primary-model` — prompt "Primary model:"
- `--lint-cmd`, `--format-cmd`, `--test-cmd`, `--build-cmd` — prompt ทีละตัว

ใน non-TTY context (CI, scripts, piped input) **gate commands ทั้งสี่และ `--role` และ `--primary-model` ต้องระบุอย่างชัดเจน** หรือคำสั่งจะ fail พร้อม non-TTY error ใช้ `--json` ใน scripted contexts

### 6.3 ตัวอย่างการใช้งาน

การสร้าง security-monitoring agent ใน workspace นี้:

```bash
# จาก workspace root Template ถูก auto-detect จาก cwd
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

หลังจากนี้เสร็จสิ้น:

1. `agents/agent-sentinel/` มีอยู่พร้อม layout ของ template ทั้งหมด
2. Backend symlinks ถูกสร้าง: `CLAUDE.md AGY.md CODEX.md KIMI.md → AGENTS.md`
3. Mindset stubs สามอันและ skill stubs สามอันถูกสร้าง
4. `config.manifest.json` มีทุก placeholder ที่ถูกแทนที่แล้ว
5. Agent ถูก register ใน `.bwoc/agents.toml`

แล้วรัน audit:

```bash
bwoc check agents/agent-sentinel
```

ถ้าผ่าน agent พร้อมใช้งาน ถ้า fail ดู Section 7

### 6.4 Checklist หลัง Incarnate

```
[ ] bwoc check agents/agent-<name>  ผ่านโดยไม่มี violations
[ ] เติม persona/README.md — sections Scope และ Anti-scope ไม่ว่าง
[ ] เติมเนื้อหา mindset stub แต่ละอัน:
      - frontmatter: tags รวม principle/<pali-term>
      - เนื้อหา: When to Apply / How to Apply / When NOT to Apply / Related Principles
[ ] เติมเนื้อหา skill stub แต่ละอัน:
      - frontmatter: tags รวม domain/<area>, maturity: L<n>
      - เนื้อหา: Domain / Inputs / Outputs / Verification Gates / Out of Scope
[ ] Cross-link ทุก skill ใน interconnect/capabilities.md
[ ] เพิ่ม entry แรกใน task-log.jsonl
[ ] สร้าง signing keypair: bwoc trust --keygen agents/agent-<name>
[ ] Verify trust profile: bwoc trust agents/agent-<name>
[ ] ใน workspace นี้: เขียน slot bodies เป็นภาษาไทย, เก็บ frontmatter เป็นภาษาอังกฤษ, รวม emoji ใน headings
[ ] bwoc check อีกครั้ง — ต้องผ่านอยู่
```

---

## 7. bwoc check — ทุก Failure Mode

`bwoc check` คือ **audit แบบ read-only** ไม่ modify ไฟล์ใด รันหลังจากทุกการแก้ไขไฟล์ agent ใด ๆ

```bash
bwoc check agents/agent-<name>          # audit หนึ่ง agent
bwoc check --all                        # audit ทุก registered agent ใน workspace
bwoc check agents/agent-<name> --json   # output แบบ machine-readable
bwoc check --all --workspace /path      # fleet audit ที่ workspace root เฉพาะ
```

`bwoc check` audit สามสิ่ง:

1. **Backend neutrality** ของ `AGENTS.md`
2. **JSON validity** ของ `config.manifest.json`
3. **Line count** ของ `MEMORY.md` (ต้อง ≤ 200)

### Failure Modes

| Failure | สาเหตุ | วิธีแก้ไข |
|---|---|---|
| `AGENTS.md has YAML frontmatter` | ไฟล์ขึ้นต้นด้วย `---` ... `---` | ลบ frontmatter block ทั้งหมดออกจาก AGENTS.md |
| `AGENTS.md contains wikilinks` | พบรูปแบบ `[[...]]` | แทนที่ `[[Target\|Display]]` ทั้งหมดด้วย plain text หรือ plain Markdown links `[Display](path)` |
| `AGENTS.md contains Obsidian callouts` | พบรูปแบบ `> [!type]` | แทนที่ด้วย plain blockquote `>` หรือ bold text |
| `AGENTS.md contains unsubstituted placeholder` | พบ `{{something}}` ที่ bwoc new ควรจะแทนที่แล้ว | แก้ไข AGENTS.md: แทนที่ placeholder ด้วยค่าจริง หรือรัน bwoc new ใหม่พร้อม flag ที่ขาด |
| `AGENTS.md references hardcoded model ID` | บรรทัดมี vendor-specific model name ในเนื้อหาพฤติกรรม | แทนที่ด้วย `{{primaryModel}}` placeholder; ค่าจริงอยู่ใน config.manifest.json |
| `AGENTS.md references hardcoded vendor name` | "Claude will..." / "Antigravity supports..." นอก Backend Registration section | เขียนใหม่เป็น backend-neutral phrasing |
| `Backend file is not a symlink` | `CLAUDE.md` (หรือ AGY.md ฯลฯ) เป็น regular file ที่มีเนื้อหาของตัวเอง | `rm CLAUDE.md && ln -s AGENTS.md CLAUDE.md` |
| `Backend symlink does not point to AGENTS.md` | Symlink มีอยู่แต่ resolve ไปยัง target อื่น | `rm AGY.md && ln -s AGENTS.md AGY.md` |
| `config.manifest.json is not valid JSON` | Syntax error: trailing comma, missing quote, unescaped character, comment | เปิดใน JSON validator; แก้ไข syntax error |
| `config.manifest.json missing required field` | Required field (name, agentId, agentRole, primaryModel, lintCmd, formatCmd, testCmd, buildCmd, memoryPath) ไม่มี | เพิ่ม field ที่ขาด |
| `config.manifest.json has unsubstituted placeholder` | ค่า field ยังมี `{{...}}` | แทนที่ด้วยค่าจริง |
| `MEMORY.md exceeds 200 lines` | Memory index โตเกิน cap | Prune: ลบ bullets สำหรับ memories ที่เก่า, ซ้ำซ้อน, หรือ derive ได้จากการอ่านโค้ด ลบไฟล์ที่เกี่ยวข้องจาก memories/ |
| `trust.declared.piyo is true but evidence missing` | `piyo` ประกาศ true แต่ `persona/README.md` ว่างหรือไม่มี Scope section | เติม Scope section ของ `persona/README.md` ด้วยคำอธิบาย task ที่ delegate ได้จริง |
| `trust.declared.garu is true but evidence missing` | `garu` ประกาศ true แต่ไม่มีไฟล์จริงใน `skills/` หรือ `mindsets/` (นอกจาก SPEC stub) | สร้าง skill หรือ mindset file จริงอย่างน้อยหนึ่งไฟล์ |
| `trust.declared.bhavaniyo is true but evidence missing` | `bhavaniyo` ประกาศ true แต่ไม่มี mindset ที่อ้างถึงการปรับปรุง/การตรวจสอบ/right-amount | เพิ่ม mindset ที่มี tag `principle/yoniso-manasikara` หรือ `principle/mattanutata` |
| `trust.declared.vatta is true but evidence missing` | `vatta` ประกาศ true แต่ `outOfScope` ใน manifest ว่าง | เติม field `outOfScope` และ Anti-scope section ของ `persona/README.md` |
| `trust.declared.vacanakkhamo is true but evidence missing` | `vacanakkhamo` ประกาศ true แต่ `.bwoc/inbox.jsonl` ไม่มีหรือว่าง และ `interconnect/feedback.md` ไม่ได้ document feedback handling | ส่งข้อความทดสอบ: `bwoc send <agent> "test"` หรือสร้าง `interconnect/feedback.md` ที่ document feedback protocol |
| `trust.declared.gambhira is true but evidence missing` | `gambhira` ประกาศ true แต่ไม่มีไฟล์ doc ใต้ agent root ที่ทั้ง ≥ 50 บรรทัด AND มี wikilink `[[PHILOSOPHY.en.md]]` หรือ `[[PHILOSOPHY.th.md]]` | สร้างไฟล์ doc ที่ตรงตามทั้งสองเกณฑ์ (ความยาว + wikilink กลับไปยัง philosophy graph) |
| `trust.declared.noCatthana is true but evidence missing` | `noCatthana` ประกาศ true แต่ Anti-scope section ของ `persona/README.md` ไม่มีหรือไม่มีรายการ "will refuse" โดยชัดแจ้ง | เพิ่ม refusal ที่ชัดเจนอย่างน้อยหนึ่งรายการใน Anti-scope section |

### การอ่าน JSON output

```bash
bwoc check agents/agent-<name> --json
```

รูปร่าง output:

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

Exit codes: `0` = pass, `1` = violations found, `2` = invocation error (bad path, unreadable file)

---

## 8. Trust และการ Sign

Trust model ระหว่าง agent ของ BWOC อิงกับ **กัลยาณมิตร-7** (Kalyanamitta-7 — "เจ็ดคุณสมบัติของมิตรที่ดี") จาก มิตตสูตร (AN 7.36 ของ อังคุตตรนิกาย — [SuttaCentral AN 7.36](https://suttacentral.net/an7.36)) แต่ละ agent ประกาศ 7 booleans ที่อธิบายความน่าเชื่อถือของตัวเอง Peer agents อ่าน booleans เหล่านั้นและสามารถปฏิเสธข้อความจาก senders ที่ขาดคุณสมบัติที่ต้องการ

ดูสเปคเต็ม: [SIGNING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) และ `interconnect/trust.md` ใน agent template

### 8.1 เจ็ดคุณสมบัติ (Kalyanamitta-7)

| คำบาลี | ความหมายตามตัวอักษร | ใน BWOC | Manifest key |
|---|---|---|---|
| ปิโย (Piyo) | น่าชื่นชอบ / น่ารัก | น่า delegate ไปให้; scope ชัดเจนและเป็นรูปธรรม | `piyo` |
| คะรุ (Garu) | น่าเคารพ / มีน้ำหนัก | มี demonstrated competency; มี skill หรือ mindset อย่างน้อยหนึ่งอัน | `garu` |
| ภาวะนีโย (Bhavaniyo) | น่าชื่นชม / ส่งเสริม | ช่วยคนอื่นปรับปรุง; มี improvement-oriented mindset | `bhavaniyo` |
| วัตตา (Vatta) | ผู้พูด | พูดความจริงที่มีประโยชน์; ซื่อสัตย์เรื่องสิ่งที่ไม่ทำ | `vatta` |
| วะจะนักขะโม (Vacanakkhamo) | ผู้ฟังที่อดทน | รับ feedback ได้; inbox flow ถูก exercise แล้ว | `vacanakkhamo` |
| คัมภีระ (Gambhira) | ผู้พูดสิ่งลึกซึ้ง | อธิบายความลึก; doc ยึดกับ canonical philosophy | `gambhira` |
| โน จัตถาเน นิโยชะเย (No catthana) | ไม่แนะนำสู่ทางที่ไม่สมควร | ไม่นำทางไปผิด; มีรายการ refusal โดยชัดแจ้งใน anti-scope | `noCatthana` |

Template scaffold default `requiredTrust` เป็น `["vatta", "noCatthana"]` — "พูดความจริงที่มีประโยชน์" และ "ไม่นำทางไปผิด" เหล่านี้คือ civic floor ขั้นต่ำที่ทุก agent ควรเรียกร้องจาก peers อย่างสมเหตุสมผล

Field `trust.mode` ควบคุมสิ่งที่เกิดขึ้นเมื่อ peer ขาดคุณสมบัติที่ต้องการ:

| Mode | มีผลเมื่อ | พฤติกรรม Daemon |
|---|---|---|
| `off` | `mode: "off"` อย่างชัดแจ้ง หรือ mode ไม่มีและ `requiredTrust` ว่าง | Envelope ผ่าน; ไม่มี log entry |
| `warn` | `mode: "warn"` อย่างชัดแจ้ง (opt-in เท่านั้น) | Envelope ผ่าน AND daemon emit `trust_warn` log line; ใช้ `bwoc log -f` เพื่อ observe |
| `refuse` | `mode: "refuse"` อย่างชัดแจ้ง หรือ mode ไม่มีและ `requiredTrust` ไม่ว่าง (v1 default) | Envelope ถูก mark ว่า refused ใน `inbox.refusals.jsonl`; ไม่ถูกลบ |

ข้อความที่มาจากผู้ใช้ (`from: "user"`) ผ่านเสมอโดยไม่คำนึงถึง mode Trust gates ควบคุมการส่งข้อความ agent-to-agent เท่านั้น

### 8.2 กฎ Evidence

การประกาศคุณสมบัติ `true` โดยไม่มี evidence ที่สอดคล้องกันคือการละเมิด `bwoc check` Framework ไม่วัดความน่าเชื่อถือจริง — มันตรวจสอบการโกหกที่ชัดเจนในเชิงโครงสร้าง:

| คุณสมบัติ | Evidence ที่ต้องการ |
|---|---|
| `piyo` | Scope section ของ `persona/README.md` ไม่ว่างและอธิบาย task ที่ delegate ได้จริง |
| `garu` | มี skill หรือ mindset file จริงอย่างน้อยหนึ่งไฟล์ใต้ `skills/` หรือ `mindsets/` (ไม่ใช่แค่ SPEC stub) |
| `bhavaniyo` | มี mindset file ที่มีชื่อหรือเนื้อหาที่อ้างถึงการปรับปรุง, การตรวจสอบ, หรือ right-amount (เช่น tag `principle/yoniso-manasikara` หรือ `principle/mattanutata`) |
| `vatta` | `outOfScope` ใน manifest ไม่ว่าง; Anti-scope section ของ `persona/README.md` ไม่ว่าง |
| `vacanakkhamo` | `.bwoc/inbox.jsonl` มีอยู่และไม่ว่าง หรือ `interconnect/feedback.md` document feedback handling |
| `gambhira` | มี doc file อย่างน้อยหนึ่งไฟล์ใต้ agent root ที่ ≥ 50 บรรทัด AND มี wikilink `[[PHILOSOPHY.en.md]]` หรือ `[[PHILOSOPHY.th.md]]` |
| `noCatthana` | Anti-scope section ของ `persona/README.md` มีอยู่ AND มีรายการ "will refuse" อย่างชัดแจ้งอย่างน้อยหนึ่งรายการ |

ประกาศ `false` สำหรับคุณสมบัติใด ๆ ที่คุณไม่สามารถ support evidence ได้อย่างซื่อสัตย์ `false` ถูกต้องเสมอ — ไม่ต้องตรวจสอบ evidence

### 8.3 Keygen และการ Sign

Ed25519 signing keypairs ใช้สำหรับ envelope signing (Phase 3 / HV2-4) Private key ไม่ออกจาก directory `.bwoc/` ของ agent

```bash
# สร้าง keypair สำหรับ agent หนึ่งตัว
bwoc trust --keygen agents/agent-<name>

# สร้างสำหรับทุก registered agent (backfill)
bwoc trust --keygen --all

# หมุน keypair (เขียนทับ key ที่มีอยู่ -- ย้อนกลับไม่ได้สำหรับ signed envelopes เก่า)
bwoc trust --keygen agents/agent-<name> --force

# อ่าน trust profile (human table)
bwoc trust agents/agent-<name>

# อ่าน trust profile (JSON)
bwoc trust agents/agent-<name> --json
```

`bwoc trust --keygen` เขียน:

- **Private key** → `agents/agent-<name>/.bwoc/agent.key` ด้วย permissions `0600` ไฟล์นี้ถูก gitignore อย่าให้ commit, copy, หรือแชร์มัน
- **Public key** → field `trust.signingPublicKey` ใน `config.manifest.json` (hex string 64 chars)

Flag `--force` เขียนทับ keypair ที่มีอยู่ อันนี้หมุน cryptographic identity ของ agent Signed envelopes ก่อนการหมุนไม่สามารถ verify กับ key ใหม่ได้ ใช้ด้วยความตั้งใจ

`bwoc trust --keygen --all` รับ `--force` เพื่อหมุนทุก agents

ทุก flags ของ `bwoc trust`:

| Flag | คำอธิบาย |
|---|---|
| `[AGENT]` | ชื่อ agent ("agent-foo" หรือ "foo") Optional เฉพาะกับ `--keygen --all` |
| `--keygen` | สร้าง ed25519 keypair แทนการอ่าน profile |
| `--all` | กับ `--keygen`: สร้างสำหรับทุก registered agent |
| `--force` | กับ `--keygen`: เขียนทับ key ที่มีอยู่ |
| `--json` | Emit JSON แทน human-readable table |
| `--workspace <WORKSPACE>` | Workspace root override |
| `--lang <LANG>` | ภาษา CLI output |

---

## 9. Lifecycle — stop / start / retire

### bwoc stop

หยุด agent ชั่วคราว: ตั้งค่า registry status เป็น `"stopped"` โดยไม่ลบไฟล์ Daemon (ถ้ารันอยู่) จะได้รับ STOP signal

```bash
bwoc stop sentinel                     # interactive confirmation
bwoc stop sentinel --yes               # ข้าม confirmation (ต้องใช้สำหรับ CI)
bwoc stop --all --yes                  # หยุดทุก non-stopped agent
bwoc stop sentinel --yes --json        # JSON output (ต้องใช้ --yes)
```

| Flag | คำอธิบาย |
|---|---|
| `<NAME>` | ชื่อ agent ("agent-foo" หรือ "foo") Mutually exclusive กับ `--all` |
| `--all` | หยุดทุก non-stopped agent ใน workspace |
| `--yes` | ข้าม interactive confirmation; ต้องใช้สำหรับ non-TTY |
| `--json` | Emit JSON `{ workspace, agent, daemon_outcome, registry_updated }`; ต้องใช้ `--yes`; single-agent only |
| `--workspace <WORKSPACE>` | Workspace root override |
| `--lang <LANG>` | ภาษา CLI output |

Agent ที่ stopped เก็บรักษาทุกไฟล์, memory, และ task log ไว้ สามารถ restart ได้ตลอดเวลา

### bwoc start

เปิดใช้งาน stopped agent อีกครั้ง: ตั้งค่า status เป็น `"active"` และ optionally spawn daemon

```bash
bwoc start sentinel                    # interactive confirmation
bwoc start sentinel --yes              # ข้าม confirmation
bwoc start sentinel --yes --no-daemon  # เปลี่ยน status เท่านั้น ไม่ spawn daemon
bwoc start --all --yes                 # start ทุก stopped agent
```

| Flag | คำอธิบาย |
|---|---|
| `<NAME>` | ชื่อ agent Mutually exclusive กับ `--all` |
| `--all` | Start ทุก stopped agent ใน workspace |
| `--yes` | ข้าม confirmation; ต้องใช้สำหรับ non-TTY |
| `--no-daemon` | เปลี่ยน registry status เท่านั้น; ไม่ spawn `bwoc-agent --serve` |
| `--json` | Emit JSON `{ workspace, agent, daemon_spawned, daemon_pid, ... }`; ต้องใช้ `--yes`; single-agent only |
| `--workspace <WORKSPACE>` | Workspace root override |
| `--lang <LANG>` | ภาษา CLI output |

### bwoc retire

การ retire คือ **วยะ** (ระยะที่สาม — ความดับ) มันลบ agent ออกจาก workspace registry โดย default มันยังลบ agent directory บน disk ด้วย

```bash
bwoc retire sentinel                           # interactive confirmation; ลบ registry entry + dir
bwoc retire sentinel --yes                     # ข้าม confirmation
bwoc retire sentinel --keep-files --yes        # ลบ registry entry เท่านั้น; เก็บ directory
bwoc retire sentinel --keep-memory --yes       # เก็บ memories/ เท่านั้น; ลบส่วนที่เหลือของ dir
bwoc retire sentinel --yes --json              # JSON output (ต้องใช้ --yes)
```

| Flag | คำอธิบาย |
|---|---|
| `<NAME>` | ชื่อ agent Matches โดย id ("agent-foo") หรือชื่อเปลือย ("foo") |
| `--yes` | ข้าม interactive confirmation; ต้องใช้สำหรับ non-TTY และสำหรับ `--json` |
| `--keep-files` | ลบ registry entry เท่านั้น; เก็บ agent directory เต็มบน disk Mutually exclusive กับ `--keep-memory` |
| `--keep-memory` | เก็บ `memories/` เท่านั้น; ลบส่วนที่เหลือของ agent directory มีประโยชน์เมื่อ retire agent แต่เก็บความรู้ที่สะสม Mutually exclusive กับ `--keep-files` |
| `--json` | Emit JSON `{ workspace, agent, path, mode, registry_updated }`; ต้องใช้ `--yes` |
| `--workspace <WORKSPACE>` | Workspace root override |
| `--lang <LANG>` | ภาษา CLI output |

ก่อน retire agent ที่เป็นสมาชิกของทีม ให้แก้ไข `.bwoc/teams/<team>.toml` เพื่อลบมันออกจากรายการสมาชิก การ retire agent ไม่ update team membership files โดยอัตโนมัติ

---

## 10. การ Operate Agent ที่กำลังทำงาน

### bwoc status

Read-only health และ identity snapshot

```bash
bwoc status sentinel                  # full detail block สำหรับ agent หนึ่งตัว
bwoc status                           # summary table สำหรับทุก agents
bwoc status --all                     # full detail สำหรับทุก agent
bwoc status sentinel --banner         # replay startup liveness banner (ไม่ต้อง daemon)
bwoc status sentinel --json           # JSON output
```

### bwoc spawn / chat / run

```bash
# Exec backend CLI ใน agent directory (uppada → thiti transition)
bwoc spawn --path agents/agent-sentinel
bwoc spawn --path agents/agent-sentinel --backend ollama

# Chat กับ named agent (resolve จาก workspace registry)
bwoc chat sentinel
bwoc chat sentinel --tmux              # เปิดใน tmux window ใหม่
bwoc chat sentinel --ghostty           # เปิดใน Ghostty terminal (macOS)
bwoc chat sentinel --tui               # full-screen ratatui chat (harness backends เท่านั้น)

# รัน single task แบบ non-interactive
bwoc run sentinel --task "check inbox and summarize pending alerts"
bwoc run sentinel --task "..." --json --timeout 120
```

### bwoc supervise

รัน daemon supervisor ที่ restart agent เมื่อ crash และออกอย่าง clean เมื่อ agent ถูก stop

```bash
bwoc supervise sentinel
bwoc supervise sentinel --max-restarts-per-min 5
bwoc supervise sentinel --json         # emit JSON event หนึ่งต่อ action
```

Default ของ `--max-restarts-per-min` คือ 10 เกินกว่าเกณฑ์นั้น supervisor จะหยุดแทนที่จะเผา CPU ใน crash loop

### bwoc send / inbox

การส่งข้อความระหว่าง agent และ user-to-agent:

```bash
# ส่งข้อความ (จาก user, default)
bwoc send sentinel "please check for anomalies in the last 15 minutes"

# ส่งจาก agent อื่น
bwoc send sentinel "anomaly detected in auth log" --from agent-luban

# Reply ข้อความก่อนหน้า
bwoc send sentinel "confirmed, escalating" --reply-to msg-20260606T120000Z-a3f2b

# ข้าม tmux wakeup (CI / daemon contexts)
bwoc send sentinel "task done" --from agent-luban --no-wakeup

# อ่าน inbox
bwoc inbox sentinel
bwoc inbox sentinel --json --limit 10
bwoc inbox sentinel --watch             # tail mode
bwoc inbox sentinel --clear --yes       # acknowledge ทุกข้อความ
bwoc inbox --all                        # inboxes ของทุก agents
```

### bwoc log / ping

```bash
bwoc log sentinel                       # 50 บรรทัดล่าสุดของ daemon log
bwoc log sentinel -f                    # follow (stream)
bwoc log sentinel -n 100 --clear        # 100 บรรทัดล่าสุด แล้ว truncate

bwoc ping sentinel                      # PING → PONG ผ่าน Unix socket
bwoc ping --all                         # ping ทุก agent
```

---

## 11. ทีม (Sangha)

**สังฆะ** (Sangha — ในบริบทนี้คือชุมชนนักปฏิบัติ, ที่นี่ adapte เป็น "ทีมของ agents") คือกลุ่มย่อยที่มีชื่อของ agents ที่แชร์ task list ทีมช่วย multi-agent coordination: shared tasks, plan-approval workflows, และ cross-agent accountability

### การสร้างและจัดการทีม

```bash
# สร้างทีมพร้อมสมาชิกเริ่มต้น
bwoc team create monitoring --members sentinel,luban,yanluo

# List ทีมพร้อม member + task counts
bwoc team list

# Retire ทีม (ลบ membership file + task list)
bwoc team retire monitoring
```

**ในการเพิ่มหรือลบสมาชิกหลังสร้าง:** แก้ไข `.bwoc/teams/<team>.toml` โดยตรง ไม่มี subcommand `bwoc team member-add` TOML file คือ membership record; CLI ไม่มี wrapper สำหรับการแก้ไข

```toml
# .bwoc/teams/monitoring.toml
[team]
id = "monitoring"
members = ["agent-sentinel", "agent-luban", "agent-yanluo"]
```

### Shared Task Workflow

```bash
# เพิ่ม task ใน task list ของทีม
bwoc task add monitoring "investigate auth log spike between 02:00 and 03:00 UTC"

# เพิ่มพร้อม dependencies (task ถูก block จนกว่า deps จะเสร็จ)
bwoc task add monitoring "generate incident report" --deps TASK-001,TASK-002

# List tasks พร้อม state + claimant
bwoc task list monitoring

# Claim pending task ในฐานะ agent
bwoc task claim monitoring TASK-001 --as sentinel

# Submit plan เพื่อรีวิว (ปวารณา / Pavarana — การเชิญให้แก้ไข)
bwoc task plan monitoring TASK-001 "Plan: 1) fetch auth logs 2) correlate IPs 3) score anomaly"

# Team lead approve plan — claimant สามารถ complete ได้
bwoc task approve monitoring TASK-001

# Team lead reject — claimant ต้อง revise และ resubmit
bwoc task reject monitoring TASK-001 "need to include VPN log correlation"

# Complete in-progress task
bwoc task complete monitoring TASK-001 --as sentinel
```

Task states: `pending` → `in_progress` (หลัง claim) → `plan_submitted` (หลัง plan) → `approved` (หลัง approve) → `completed` Tasks ที่ถูก reject กลับไป `in_progress` เพื่อ revision

---

## 12. ข้อกำหนดการเขียนของ Workspace นี้

Agents ใน workspace นี้มี **ธีมเป็นเทพจีน (Chinese deities)** Section นี้บันทึกข้อกำหนดที่ใช้กับทุก agent ที่ incarnate ที่นี่

### ภาษา

- **Slot bodies** (`persona/README.md`, `mindsets/*.md`, `skills/*.md`, `interconnect/*.md`, `memories/*.md`): เขียนเป็น**ภาษาไทย** เนื้อหา — คำอธิบาย, คำแนะนำ, sections "When to Apply" — เป็นภาษาไทย
- **Frontmatter และ tags**: **ภาษาอังกฤษ** เสมอ YAML keys, tag values, field names, maturity levels, domain labels
- **`AGENTS.md`**: backend-neutral; เนื้อหาพฤติกรรมของมันใช้ภาษาไทยสำหรับ scope/role descriptions (ตามที่เห็นใน `agent-erlang`) ซึ่งยอมรับได้เพราะเหล่านั้นคือ role-specific strings ที่ resolve จาก manifest sections โครงสร้างของ `AGENTS.md` คงอยู่เป็นภาษาอังกฤษ
- **Headings ใน slot files**: รวม **emoji** ที่เกี่ยวข้องที่สะท้อน deity theme หรือ skill domain ของ agent

### ธีม

ธีมปัจจุบัน: เทพจีน ชื่อ agent สามารถอ้างอิงเทพ (เช่น `erlang` สำหรับ Erlang Shen เทพตาที่สามแห่งการเฝ้าระวัง) หรือ function (เช่น `sentinel`) ไฟล์ persona ให้บริบทเทพนิยายแก่ agent

### ตัวอย่าง slot heading (mindsets)

```markdown
# 👁️ All Seeing Third Eye — เปิดตาที่สาม
```

### การตรวจสอบ

หลังทุกการแก้ไข slot file หรือ `AGENTS.md`:

```bash
bwoc check agents/agent-<name>
```

ทุก pass ต้องสะอาดก่อน commit

---

## 13. ข้อผิดพลาดที่พบบ่อยและวิธีแก้ไข

### ข้อผิดพลาด: YAML frontmatter ใน AGENTS.md

**อาการ:** `bwoc check` รายงาน `AGENTS.md has YAML frontmatter`

**สาเหตุ:** ไฟล์ถูกสร้างจาก documentation template หรือแก้ไขด้วย Obsidian ซึ่งเพิ่ม frontmatter block

**วิธีแก้ไข:** ลบ block `---` ... `---` ที่ด้านบนของ `AGENTS.md` ทั้งหมด ไฟล์ต้องเริ่มด้วย plain Markdown heading หรือ paragraph

---

### ข้อผิดพลาด: Wikilinks ใน AGENTS.md

**อาการ:** `bwoc check` รายงาน `AGENTS.md contains wikilinks`

**สาเหตุ:** Copy-paste จาก slot file หรือ conventions doc ที่ใช้ไวยากรณ์ `[[...]]`

**วิธีแก้ไข:** แทนที่ `[[Target|Display]]` ทุกอันด้วย plain text หรือ standard Markdown link ถ้า wikilink เป็น cross-reference ไปยัง agent file อื่น cross-reference นั้นไม่ควรอยู่ใน `AGENTS.md` — วางไว้ใน slot doc หรือไฟล์ `interconnect/` แทน

---

### ข้อผิดพลาด: Hardcoded model IDs ใน AGENTS.md

**อาการ:** `bwoc check` รายงาน neutrality violation สำหรับชื่อ model ที่ hardcode

**สาเหตุ:** เขียนชื่อ model เช่น `claude-sonnet-4-6` โดยตรงใน `AGENTS.md` นอก Backend Registration section

**วิธีแก้ไข:** แทนที่ด้วย `{{primaryModel}}` หรือ `{{fallbackModel}}` Model ID จริงอยู่ใน `config.manifest.json` หมายเหตุ: model IDs ใน Backend Registration section (Section 0) ในฐานะ documentation table ของ "backends ที่มีอยู่" ยอมรับได้; model IDs ในเนื้อหาพฤติกรรมต้องใช้ placeholders

---

### ข้อผิดพลาด: Backend file เป็น regular file ไม่ใช่ symlink

**อาการ:** `bwoc check` รายงาน `Backend file is not a symlink` สำหรับ `CLAUDE.md` หรือ backend file อื่น

**สาเหตุ:** มีคนแก้ไข `CLAUDE.md` โดยตรงหรือสร้างมันด้วย editor แทนที่จะใช้ `ln -s`

**วิธีแก้ไข:**
```bash
cd agents/agent-<name>
rm CLAUDE.md
ln -s AGENTS.md CLAUDE.md
```

---

### ข้อผิดพลาด: config.manifest.json มี trailing comma

**อาการ:** `bwoc check` รายงาน `config.manifest.json is not valid JSON`

**สาเหตุ:** JSON ไม่อนุญาต trailing commas หลัง item สุดท้ายใน object หรือ array

**วิธีแก้ไข:** ลบ trailing comma ใช้ JSON validator หรือ `python3 -m json.tool config.manifest.json` เพื่อหาบรรทัด

---

### ข้อผิดพลาด: MEMORY.md เกิน 200 บรรทัด

**อาการ:** `bwoc check` รายงาน `MEMORY.md exceeds 200 lines`

**สาเหตุ:** Memory entries สะสมมากเกินไปโดยไม่ได้ prune

**วิธีแก้ไข:** ตรวจสอบ `memories/` สำหรับ entries ที่เก่า, ซ้ำซ้อน, หรือ derive ได้ ลบไฟล์และลบ bullets ของพวกมันออกจาก `MEMORY.md` นี่คือการปฏิบัติ anicca (อนิจจา — ความไม่เที่ยง) — ปล่อยสิ่งที่ไม่มีประโยชน์อีกต่อไป

---

### ข้อผิดพลาด: Trust quality ประกาศ true โดยไม่มี evidence

**อาการ:** `bwoc check` รายงาน trust evidence violation (เช่น `trust.declared.gambhira is true but evidence missing`)

**สาเหตุ:** Manifest ประกาศคุณสมบัติแต่ structural evidence (ดู Section 8.2) ไม่มี

**วิธีแก้ไข:** ให้ evidence (สร้างไฟล์ที่ต้องการ, เติม section ที่ต้องการ) หรือตั้งค่าคุณสมบัติเป็น `false` ใน `config.manifest.json` อย่าอ้างคุณสมบัติที่ไม่สามารถ support ได้

---

### ข้อผิดพลาด: Mindset file ขาด section "When NOT to Apply"

**อาการ:** Mindset technically valid (ผ่าน `bwoc check`) แต่ทำให้เกิดการ over-apply ตอน runtime

**สาเหตุ:** ทุก mindset มี bounded domain โดยไม่มี section "When NOT to Apply" อย่างชัดแจ้ง agent อาจใช้มันโดยไม่เลือก

**วิธีแก้ไข:** เพิ่ม section ตั้งชื่อสถานการณ์อย่างน้อยหนึ่งคลาสที่ mindset นี้เป็น counterproductive นี่คือการแสดงออกเชิงกลไกของ mattanutata (ความพอดี / right amount) — ทุก frame มีขีดจำกัด

---

### ข้อผิดพลาด: Hand-editing `.bwoc/agents.toml`

**อาการ:** Registry กลายเป็น inconsistent; `bwoc list` แสดง phantom entries หรือพลาด live agents; `bwoc check --all` ผลิต errors ที่สับสน

**สาเหตุ:** Registry ถูกแก้ไขโดยตรงแทนที่จะผ่าน CLI commands

**วิธีแก้ไข:** ใช้ `bwoc retire --keep-files` เพื่อลบ entry ที่ไม่ดีอย่าง clean แล้ว re-register ผ่าน `bwoc new` พร้อม `--target` ที่ชี้ไปยัง directory ที่มีอยู่

---

### ข้อผิดพลาด: รัน bwoc new โดยไม่มี --target ใน multi-agent workspace

**อาการ:** Agent directory ใหม่ปรากฏที่ path ผิด (relative กับ template ไม่ใช่ workspace root)

**สาเหตุ:** `--target` default เป็น `../agent-<name>/` relative กับ template directory ซึ่งอาจวาง agent ไว้ใน `projects/bwoc-framework/modules/` แทน `agents/`

**วิธีแก้ไข:** ส่ง `--target agents/agent-<name>` เสมอเมื่อรันจาก workspace root Template auto-detect เชื่อถือได้; มีเพียง `--target` เท่านั้นที่ต้องระบุอย่างชัดแจ้ง

---

*คู่มือนี้ดูแลรักษาควบคู่กับ framework เมื่อพฤติกรรมที่อธิบายที่นี่แตกต่างจาก [framework repo](https://github.com/bemindlabs/BWOC-Framework) repo ถือเป็นถูก — กรุณาเปิด fix*
