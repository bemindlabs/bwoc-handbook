---
title: การเขียน Plugin — การขยาย framework
nav_order: 6
---

# การเขียน Plugin — การขยาย framework

> ฉบับภาษาอังกฤษ: [HANDBOOK.en.md](HANDBOOK.en.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) · อภิธานศัพท์: [../glossary.th.md](../glossary.th.md) · Developer: [../developer/HANDBOOK.th.md](../developer/HANDBOOK.th.md)

**แหล่งความจริง.** สเปคฉบับ canonical อยู่ใน framework repo เมื่อขัดแย้งกัน repo ถูกเสมอและ handbook นี้มี bug เอกสารที่เกี่ยวข้อง: [`PLUGINS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) (สเปค plugin ฉบับเต็ม) · [`SKILLS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md)

---

## สารบัญ

1. [skill หรือ plugin?](#1-skill-หรือ-plugin)
2. [ชนิดของ plugin](#2-ชนิดของ-plugin)
3. [กายวิภาคของ plugin](#3-กายวิภาคของ-plugin)
4. [วงจรชีวิต — `bwoc plugin`](#4-วงจรชีวิต--bwoc-plugin)
5. [schema ที่ normative — สัญญา](#5-schema-ที่-normative--สัญญา)
6. [write verb และ operator-confirm gate](#6-write-verb-และ-operator-confirm-gate)
7. [plugin ไม่ใช่อะไร](#7-plugin-ไม่ใช่อะไร)
8. [ดูเพิ่มเติม](#8-ดูเพิ่มเติม)

---

## 1. skill หรือ plugin?

skill และ plugin ใช้พื้นฐานร่วมกัน — manifest TOML, gate ความเป็นกลางของ backend, การ opt-in ต่อ workspace — และแยกกันที่ **ใครเป็นคนเปิดใช้**

| | Skill | Plugin |
|---|---|---|
| กลุ่มเป้าหมาย | ผู้เขียน agent | operator ของ workspace |
| opt-in ผ่าน | `<agent>/config.manifest.json` | `workspace.toml [plugins]` |
| ผู้เรียกใช้ | agent ระหว่างทำงานของตัวเอง | runtime ของ framework |
| ขอบเขต | ต่อ agent | ต่อ workspace |
| ตัวอย่าง | วินัย worktree, การตรวจ bilingual parity | Tier 2 memory backend, LLM backend เพิ่ม |

หลักง่ายๆ: **ถ้า logic ของ agent ตัวหนึ่งเรียกใช้ มันคือ skill; ถ้า workspace โหลดครั้งเดียวให้ทุกคน มันคือ plugin** บทนี้ว่าด้วย plugin; สำหรับ skill ดู [`SKILLS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md)

---

## 2. ชนิดของ plugin

ทุก plugin ประกาศ `kind` หนึ่งเดียว kind นิยามว่า framework เรียก lifecycle hook ใด plugin ตั้ง `kind` ครั้งเดียว — **plugin ข้าม kind ไม่รองรับ; ให้แยกออก**

| Kind | ขยายอะไร |
|---|---|
| `memory-backend` | Tier 2 memory (semantic search / vector store) |
| `llm-backend` | backend นอกเหนือหกตัวที่ประกาศไว้ |
| `workflow` | integration ภายนอก (issue tracker, CI, code review) |
| `audit` | ตรวจ workspace กับมาตรฐาน (ISO/IEC 29110, 9001, 27001…) |
| `jira` | sync สองทางกับ Jira Cloud — kind **เขียนได้** ตัวแรก |
| `okr` | ติดตาม Objective + Key Result ที่ operator เขียน |
| `council` | โปรโตคอลการตัดสินใจแบบมีโครงสร้างในหมู่ agent (ดู [Council](../council/HANDBOOK.th.md)) |
| `figma` | integration Figma REST แบบอ่านเป็นหลัก; สะพาน design→dev |
| `gws` | Google Workspace แบบอ่านเป็นหลัก (Drive / Gmail / Calendar) |

> [!note]
> หลักตัดสินสำหรับ kind *ใหม่*: **เป็น kind ของตัวเองเมื่อ BWOC นิยาม schema ที่ normative เหนือ integration นั้น; ใช้ `workflow` ซ้ำเมื่อมันเป็น passthrough ที่ไม่มีรูปทรงที่ BWOC เป็นเจ้าของ** `figma` และ `jira` ได้ kind ของตัวเองเพราะถือ mapping schema ที่ normative; การ shell-out บางๆ ไปยัง CLI ไม่ได้

---

## 3. กายวิภาคของ plugin

plugin คือ directory ที่มี **`manifest.toml`** ที่ราก manifest ประกาศ identity, `kind`, verb ที่ plugin เปิดเผย, และคำอธิบายแบบ JSON-schema-lite ของคีย์ config ที่ workspace ตั้งได้:

```toml
name = "my-audit"
kind = "audit"
version = "0.1.0"

# config ที่ operator ตั้งใน workspace.toml [plugins.my-audit];
# validate กับ block นี้ตอนโหลด
[config]
storage_path = { type = "string",  required = false, default = "memories/tier2" }
max_results  = { type = "integer", required = true }
```

จากนั้น operator opt-in จาก `workspace.toml`:

```toml
[plugins.my-audit]
max_results = 50
```

framework validate ตาราง `[plugins.<name>]` ของ workspace กับ block `[config]` ของ manifest — คีย์ required ที่หายไปหรือ type ผิดทำให้โหลดล้มเหลว ไม่ใช่ตอนรัน

---

## 4. วงจรชีวิต — `bwoc plugin`

การเขียนและใช้งาน plugin วิ่งผ่าน command family เดียว:

```bash
bwoc plugin init my-audit --kind audit   # scaffold manifest + stub
bwoc plugin verify my-audit              # ตรวจ neutrality + manifest + schema
bwoc plugin install ./my-audit           # ลงทะเบียนเข้า workspace
bwoc plugin enable my-audit              # เปิดใช้สำหรับ workspace นี้
bwoc plugin list                         # อะไรติดตั้ง + เปิดใช้
bwoc plugin show my-audit                # manifest + config ที่มีผล
bwoc plugin disable my-audit             # ปิดโดยไม่ลบ
bwoc plugin remove my-audit              # ถอนทะเบียน
```

> [!tip]
> รัน `bwoc plugin verify` ก่อน install ทุกครั้ง มันรัน gate ความเป็นกลางของ backend ชุดเดียวกับ `bwoc check` — plugin ที่ฝังชื่อ vendor หรือ model id ตายตัวจะ fail ที่นี่ คุณจึงจับได้ก่อนถึง workspace

---

## 5. schema ที่ normative — สัญญา

คุณค่าของ plugin คือ **output contract** ของมัน kind ประเภทรายงานและ coordination แต่ละตัวถือ schema ที่ normative ซึ่ง framework validate ที่ทุกขอบเขต `invoke` — ค่า enum ที่ไม่รู้จักคือ *bug ของ plugin ที่ทำให้ run ล้มเหลว* ไม่ใช่ finding ที่ operator ต้อง triage schema ที่ ship แล้ว:

- **Audit Findings** — `{criterion_id, severity, status, evidence, remedy}`; `severity` และ `status` เป็น closed enum; `evidence` ต้องทำซ้ำได้ (guard **มุสาวาท** — ไม่มีการอ้างโดยไม่มี referent)
- **Jira Issue Mapping**, **OKR Progress**, **Council Decision**, **Figma Asset Mapping**, **Workspace Resource** — แต่ละอันเป็นรูปทรง normative ของ kind นั้น

กฎร่วมข้ามทั้งหมด: **ฟิลด์ optional ถูกละเมื่อไม่มี ไม่เคย serialize เป็น `null`** และ identifier **เสถียรข้าม release** — การเปลี่ยนชื่อ `criterion_id` หรือ key บนฟิลด์ที่เปลี่ยนได้คือ breaking change

---

## 6. write verb และ operator-confirm gate

verb ส่วนใหญ่ **อ่าน** และไม่มี gate — ฟรี **write verb** — verb ที่ `invoke` สร้างผลข้างเคียงถาวรนอก address space ของ plugin เอง — ถือ **operator-confirm gate ที่ normative**:

1. **gate อยู่ที่ขอบเขต operator** — command `bwoc <cli>` ไม่ใช่ plugin ยืนยันหนึ่งครั้งต่อหนึ่ง write; plugin ไม่ทำซ้ำหรือข้ามมัน
2. **มันแสดงผลที่แน่นอนก่อนลงมือ** — เป้าหมาย, state ปัจจุบัน, และคำสั่งจริงที่จะรัน (พร้อม `--` เพื่อให้ค่าของผู้ใช้ถูกตีความเป็น flag ไม่ได้)
3. **ค่าเริ่มต้นคือ No** operator แบบโต้ตอบตอบ `y/N`; agent แบบ headless ต้องส่ง `--yes` ชัดเจน ตั้ง **เฉพาะ** เมื่อ operator อนุญาตการกระทำนั้นโดยเจาะจง
4. **write ที่ถูกปฏิเสธรายงาน "ไม่มีการเปลี่ยนแปลง" พร้อมเหตุผล** (ธัมมานุปัสสนา) — ไม่เคยเขียนเงียบ ไม่เคย fail เปล่าๆ

> [!warning]
> verb ที่ทำลายและย้อนไม่ได้ (เช่น `delete`) ถูกถือมาตรฐานสูงกว่า write ที่ย้อนได้อย่าง `start`/`stop` และถูกเพิ่มอย่างตั้งใจต่อ slice พร้อม gate ที่แข็งแกร่งกว่าของตัวเอง — ไม่ ship เพียงเพราะมันอยู่ข้าง write ที่ย้อนได้

---

## 7. plugin ไม่ใช่อะไร

- **ไม่ใช่ช่องโหว่สำหรับ logic เฉพาะ vendor ใน framework** backend หกตัวที่ประกาศเป็นชั้นหนึ่งและอยู่ใน spec วลีเฉพาะ vendor ยังต้องห้าม (**สมานัตตตา**)
- **ไม่ใช่ที่สำหรับ script ครั้งเดียว** พวกนั้นอยู่กับ agent ที่ใช้มัน
- **ไม่ใช่ skill ที่มีขั้นตอนเพิ่ม** ถ้า agent เรียกใช้ระหว่างทำงานของตัวเอง มันคือ skill

---

## 8. ดูเพิ่มเติม

**source framework (GitHub สาธารณะ):**

- [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) — สเปค plugin ฉบับเต็ม: ทุก kind, schema, และรูปแบบ manifest
- [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) — ชั้น skill สำหรับส่วนขยายของผู้เขียน agent

**บทอื่นใน handbook:**

- [../developer/HANDBOOK.th.md](../developer/HANDBOOK.th.md) — แผนที่ crate และ runtime โหลดส่วนขยายอย่างไร
- [../council/HANDBOOK.th.md](../council/HANDBOOK.th.md) — kind `council` เชิงลึก
- [../okr/HANDBOOK.th.md](../okr/HANDBOOK.th.md) และ [../iso-audit/HANDBOOK.th.md](../iso-audit/HANDBOOK.th.md) — ตัวอย่างจริงของ plugin `okr` และ `audit`
- [../glossary.th.md](../glossary.th.md) — ทุกคำในตารางเดียว

---

*handbook นี้ดูแลควบคู่กับ framework เมื่อพฤติกรรมที่อธิบายที่นี่ต่างจาก [framework repo](https://github.com/bemindlabs/BWOC-Framework) repo ถูกต้อง — โปรดเปิด fix*
