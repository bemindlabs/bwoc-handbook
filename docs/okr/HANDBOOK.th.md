# OKR กับ BWOC

🇬🇧 [English](HANDBOOK.en.md)

สำหรับ **ผู้ดูแล workspace และหัวหน้าทีม** ที่ต้องการติดตามเป้าหมายควบคู่ไปกับงานของ agent — ไม่ใช่ใน spreadsheet แยกต่างหาก แต่อยู่ใน workspace เดียวกับที่ framework จัดการอยู่แล้ว
หากยังไม่ได้ตั้งค่า workspace ให้เริ่มที่ [`../quickstart/HANDBOOK.th.md`](../quickstart/HANDBOOK.th.md)
หากต้องการทำความเข้าใจระบบ plugin ในภาพรวม ดูที่ [PLUGINS spec](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md)
ค้นหาคำศัพท์: [`../glossary.th.md`](../glossary.th.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)

---

## สารบัญ

1. [OKR คืออะไร และทำไม BWOC ถึงรองรับ](#1-okr-คืออะไร-และทำไม-bwoc-ถึงรองรับ)
2. [plugin okr มอบอะไรให้คุณบ้าง](#2-plugin-okr-มอบอะไรให้คุณบ้าง)
3. [ติดตั้งและเปิดใช้งาน](#3-ติดตั้งและเปิดใช้งาน)
4. [เขียน objectives และ key results](#4-เขียน-objectives-และ-key-results)
5. [สาม verb ที่ใช้งาน](#5-สาม-verb-ที่ใช้งาน)
6. [ตัวอย่างจริง](#6-ตัวอย่างจริง)
7. [OKR กับ fleet governance](#7-okr-กับ-fleet-governance)
8. [ดูเพิ่มเติม](#8-ดูเพิ่มเติม)

---

## 1. OKR คืออะไร และทำไม BWOC ถึงรองรับ

**OKR** (Objectives and Key Results) คือวิธีตั้งเป้าหมาย *Objective* คือประโยคเชิงคุณภาพที่บอกทิศทาง ("ส่ง OKR plugin kind") ส่วน *Key Result* คือผลลัพธ์ที่วัดได้ซึ่งบอกว่า objective บรรลุถึงไหนแล้ว ("reference plugin ส่งมอบได้และ verb ทั้งสามทำงานได้") คุณตั้ง target ติดตาม current value และประเมิน confidence

ในทีมส่วนใหญ่ OKR อยู่ใน tool แยกต่างหาก — spreadsheet หรือแอป project management — ห่างไกลจากโค้ดและ agent ที่กำลังทำงานอยู่จริง ช่องว่างนั้นมีต้นทุน: คุณไม่สามารถชี้ให้เป้าหมายอ้างอิงไฟล์ commit หรือ task ของ agent โดยตรงได้ และไม่สามารถรันสคริปต์ query ความคืบหน้าโดยไม่ออกจาก workspace

BWOC ปิดช่องว่างนั้นด้วย plugin kind `okr` objectives และ key results ของคุณอยู่ในไฟล์ TOML ธรรมดาสองไฟล์ภายใน workspace CLI `bwoc okr` ให้คุณอัปเดตความคืบหน้า ตรวจสถานะ และสร้าง progress report ทั้งหมดโดยไม่ต้องออกจาก terminal evidence ของแต่ละ key result คือ path ไฟล์ใน workspace output ของ command หรือ attestation — vocabulary เดียวกับที่ plugin `audit` ใช้ จึงมีวิธีเดียวในการพูดว่า "นี่คือหลักฐานของฉัน" ทั่วทั้ง framework

plugin `okr` เป็น **local-file-only** อ่านและเขียน TOML ใน workspace ของคุณ ไม่เชื่อมต่อระบบภายนอก ไม่เก็บ credential และไม่ส่ง network traffic ใดๆ

---

## 2. plugin okr มอบอะไรให้คุณบ้าง

reference plugin คือ `workspace-okrs` (`kind = "okr"` version ปัจจุบัน `0.1.0` ต้องการ framework `>=2.9.0`)

มีให้:

- **`objectives.toml`** — ที่คุณเขียน objectives (หนึ่ง `[[objective]]` block ต่อ objective)
- **`key_results.toml`** — ที่คุณเขียน key results (หนึ่ง `[[key_result]]` block ต่อ key result) แต่ละอันอ้างอิง objective แม่
- **สาม verb** ผ่าน `bwoc okr`: `track`, `check-progress`, และ `report`
- output JSON แบบ **OKR Progress Schema** ที่เป็น normative (รูปแบบเดียวกับที่ `bwoc check` ตรวจสอบ)
- **heuristic ของ check-progress** ที่รวมสถานะต่อ KR (on-track / at-risk / off-track) ไปเป็น summary ต่อ objective

สิ่งที่ plugin นี้จงใจ **ไม่** ทำ:

- ไม่มี network call ไม่มี external system of record
- ไม่มี confirmation gate สำหรับ `track` — เพราะการเขียนเดียวแตะแค่ไฟล์ TOML ของคุณเอง ซึ่ง `git diff` ดูได้ตลอดเวลา
- ไม่มีโมเดล "expected attainment ตามช่วงเวลาที่ผ่านไป" ใน v1 — status ใช้ threshold attainment แบบ flat ที่ 0.7 (เส้น "green" มาตรฐาน OKR)

---

## 3. ติดตั้งและเปิดใช้งาน

plugin นี้มาพร้อม framework ที่ `modules/plugins/okr/workspace-okrs/` หาก workspace ของคุณ init จาก framework อาจมีอยู่แล้ว ตรวจสอบด้วย:

```bash
bwoc plugin list
```

หากไม่ได้อยู่ในรายการ ติดตั้งด้วย:

```bash
bwoc plugin install ./modules/plugins/okr/workspace-okrs/
bwoc plugin enable workspace-okrs
```

หลังจาก enable แล้ว `workspace.toml` จะมี entry นี้:

```toml
[plugins.workspace-okrs]
enabled = true
```

นั่นคือ configuration ทั้งหมดที่จำเป็น plugin `workspace-okrs` ไม่มี config key เพิ่มเติมใน v1 — อ่านข้อมูลจากไฟล์ TOML ภายใน plugin directory

ตรวจสอบการติดตั้ง:

```bash
bwoc plugin show workspace-okrs
```

plugin ต้องการ `jq` บน `PATH` หาก `jq` ไม่มี plugin จะออกพร้อม error message ที่ชัดเจนบอกให้ติดตั้ง

---

## 4. เขียน objectives และ key results

plugin มาพร้อม seed data (objective ส่งมอบ EPIC-4 ที่ใช้ติดตาม development ของ plugin เอง) แทนที่ด้วยเป้าหมายของคุณ

### `objectives.toml`

หนึ่ง `[[objective]]` block ต่อ objective:

```toml
[[objective]]
objective_id = "O1"
title        = "ส่ง agent memory skill ขึ้น production"
owner        = "agent-jisoo"
period       = "2026-Q3"
parent       = ""
```

| Field | Required | Notes |
|---|---|---|
| `objective_id` | ใช่ | String id ที่เสถียร key results อ้างอิง field นี้ การเปลี่ยนชื่อทำให้ referential integrity เสียหาย — `bwoc check` จะจับได้ |
| `title` | ใช่ | ประโยคเดียวบอกทิศทางของ objective |
| `owner` | ใช่ | agent id หรือชื่อบุคคล |
| `period` | ใช่ | ช่วงเวลา เช่น `2026-Q3` |
| `parent` | ไม่บังคับ | `objective_id` ของ objective แม่สำหรับโครงสร้างซ้อนกัน ใช้ `""` สำหรับ top-level multi-level rollup เลื่อนไปทำในเวอร์ชันถัดไป |

### `key_results.toml`

หนึ่ง `[[key_result]]` block ต่อ key result:

```toml
[[key_result]]
key_result_id = "O1-KR1"
objective_id  = "O1"
description   = "Skill spec merged และ bwoc check ผ่านบน agent ทั้งหมด"
target        = 1
current       = 0
unit          = "count"
confidence    = "medium"
evidence      = { kind = "none", value = "" }
```

| Field | Required | Notes |
|---|---|---|
| `key_result_id` | ใช่ | String id ที่เสถียร ไม่ซ้ำกันภายใน plugin |
| `objective_id` | ใช่ | ต้องอ้างอิงถึง id ใน `objectives.toml` `bwoc check` ตรวจสอบจุดนี้ |
| `description` | ใช่ | สิ่งที่ key result วัด |
| `target` | ใช่ | ค่าเป้าหมาย (ตัวเลข) |
| `current` | ใช่ | ค่าที่ติดตามล่าสุด อัปเดตโดย `track` |
| `unit` | ใช่ | หนึ่งใน `count`, `percent`, `currency`, `ratio`, `boolean` |
| `confidence` | ใช่ | หนึ่งใน `high`, `medium`, `low` — การอ่านเชิงคุณภาพว่า trajectory ยังดีอยู่ไหม |
| `evidence` | ใช่ | `{ kind, value }` ใช้ evidence kinds มาตรฐาน: `file`, `content`, `command`, `attestation`, `sample`, หรือ `none` |
| `as_of` | ไม่บังคับ | วันที่ ISO-8601 ที่ track ค่า `current` ครั้งล่าสุด ละบรรทัดนี้เมื่อยังไม่เคย track `track` เขียนให้อัตโนมัติ |

**หมายเหตุ unit `boolean`:** ใช้ `0` สำหรับ false และ `1` สำหรับ true target เป็น `1` และ current เป็น `1` หมายถึงบรรลุแล้ว

---

## 5. สาม verb ที่ใช้งาน

verb ทั้งสามเข้าถึงได้ผ่าน `bwoc okr` และยังสามารถเรียกใช้ด้วยมือโดยตรงผ่าน entry script สำหรับ smoke test ได้

### `track` — บันทึกค่า current ใหม่

```bash
bwoc okr track --key-result O1-KR1 --current 1 --evidence "file:docs/en/MEMORY.en.md"
```

อัปเดต field `current` (และ `evidence` ถ้าระบุ) ของ key result ที่ต้องการใน `key_results.toml` จากนั้น stamp `as_of` ด้วยวันนี้ คืนค่า key result ที่อัปเดตแล้วเป็น progress entry เดียว

| Flag | Required | Notes |
|---|---|---|
| `--key-result <id>` | ใช่ | `key_result_id` ที่ต้องการอัปเดต |
| `--current <value>` | ใช่ | ค่าตัวเลขใหม่ สามารถเกิน `target` ได้ — over-attainment ถูกเก็บไว้ |
| `--evidence <kind:value>` | ไม่บังคับ | หลักฐานสำหรับ check-in นี้ รูปแบบ: `kind:value` เช่น `file:crates/bwoc-cli/src/memory.rs` หรือ `none:` |

`track` เป็น idempotent: การ track key result ด้วย `current` + `evidence` เดิมจะเขียน bytes เดิม (เฉพาะ `as_of` ที่เลื่อนเป็นวันนี้) ปลอดภัยที่จะ retry หลังจาก transient failure

`track` **ไม่มี confirmation gate** — การเขียนแตะแค่ไฟล์ TOML ของคุณเอง หาก track ค่าผิด `git diff` จะแสดงให้เห็น และ `git checkout` สามารถย้อนกลับได้

### `check-progress` — สถานะต่อ KR และต่อ objective

```bash
bwoc okr check-progress
```

Read-only คืน JSON object ที่มี:

- Attainment และสถานะต่อ KR (`on-track`, `at-risk`, `off-track`)
- Rollup ต่อ objective พร้อม counts และสถานะ worst-KR

Heuristic สถานะ:

```
attainment = current / target
  (boolean: current >= target → 1, else 0; target เป็น 0 → ถือว่าบรรลุ)

on-track  — attainment >= 0.7  หรือ  confidence == "high"
at-risk   — ถ้าไม่ใช่ข้างบน และ confidence == "medium"
off-track — ถ้าไม่ใช่ข้างบน (confidence == "low")
```

สถานะของ objective รวมมาจาก worst KR status ของ key results ทั้งหมดภายใต้ objective นั้น

### `report` — OKR Progress Schema JSON เต็มรูปแบบ

```bash
bwoc okr report
```

Read-only คืน JSON array ของ key results ทั้งหมดในรูปแบบ progress entry แต่ละอันเป็นไปตาม [OKR Progress Schema](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md#okr-progress-schema) นี่คือ canonical wire format ที่ `bwoc check` ตรวจสอบและ tooling downstream นำไปใช้

ตัวอย่าง output element:

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

key result ที่ยังไม่เคย track จะละ `as_of` ทั้งหมด (ไม่ใช่ `null`)

---

## 6. ตัวอย่างจริง

สมมุติว่าคุณกำลัง ship agent skill ใหม่และต้องการติดตามการส่งมอบเป็น OKR

**ขั้นตอนที่ 1 — เขียน objective**

```toml
# objectives.toml
[[objective]]
objective_id = "O1"
title        = "Ship memory-tier2 skill ไปยัง agent ทั้งหมด"
owner        = "agent-jisoo"
period       = "2026-Q3"
parent       = ""
```

**ขั้นตอนที่ 2 — เขียน key results สามรายการ**

```toml
# key_results.toml
[[key_result]]
key_result_id = "O1-KR1"
objective_id  = "O1"
description   = "Skill spec merged และ bwoc check ผ่าน"
target        = 1
current       = 0
unit          = "count"
confidence    = "medium"
evidence      = { kind = "none", value = "" }

[[key_result]]
key_result_id = "O1-KR2"
objective_id  = "O1"
description   = "agent ที่ลงทะเบียนทั้ง 6 ตัวเปิดใช้ skill แล้ว"
target        = 6
current       = 0
unit          = "count"
confidence    = "medium"
evidence      = { kind = "none", value = "" }

[[key_result]]
key_result_id = "O1-KR3"
objective_id  = "O1"
description   = "Memory retrieval latency ต่ำกว่า 200ms (sample 50 queries)"
target        = 1
current       = 0
unit          = "boolean"
confidence    = "low"
evidence      = { kind = "none", value = "" }
```

**ขั้นตอนที่ 3 — ตรวจสถานะเริ่มต้น**

```bash
bwoc okr check-progress
```

key result ทั้งสามแสดง `at-risk` หรือ `off-track` เพราะ `current` เป็น 0 objective rollup เป็น `off-track` นี่คือสิ่งที่คาดหวัง — เพิ่งเริ่มต้น

**ขั้นตอนที่ 4 — ทำงาน เมื่อ KR1 ส่งมอบได้แล้ว track มัน**

```bash
bwoc okr track \
  --key-result O1-KR1 \
  --current 1 \
  --evidence "file:modules/skills/memory-tier2/SPEC.md"
```

Output: progress entry ของ KR1 ที่อัปเดตแล้ว พร้อม `current: 1` และ `as_of: 2026-06-07`

**ขั้นตอนที่ 5 — เมื่ออัปเดต agent ทีละตัว track KR2 เป็นส่วนๆ**

```bash
bwoc okr track --key-result O1-KR2 --current 3
bwoc okr track --key-result O1-KR2 --current 6 --evidence "command:bwoc list --skill memory-tier2 --count"
```

**ขั้นตอนที่ 6 — เมื่อมีหลักฐาน latency แล้ว track KR3**

```bash
bwoc okr track \
  --key-result O1-KR3 \
  --current 1 \
  --evidence "sample:49 ใน 50 queries ต่ำกว่า 200ms"
```

**ขั้นตอนที่ 7 — สร้าง report สุดท้าย**

```bash
bwoc okr report
```

pipe ไปที่ `jq` เพื่อ filter:

```bash
bwoc okr report | jq '.[] | select(.confidence == "low")'
```

---

## 7. OKR กับ fleet governance

OKR ใน BWOC ไม่ใช่แค่เครื่องมือ reporting — มันเป็น governance layer สำหรับ fleet

**มอบหมาย objective ให้ agent** field `owner` ของ objective รับ agent id ได้ (เช่น `agent-jisoo`) นี่คือ naming convention ไม่ใช่การมอบหมายอัตโนมัติ: CLI `bwoc okr` ไม่ spawn agent task โดยอัตโนมัติ ความเชื่อมโยงเป็นสิ่งที่มนุษย์ตั้งใจ: คุณตั้ง objective ระบุ owner และมอบหมาย task แยกผ่าน `bwoc task add`

**Evidence ผูกเป้าหมายกับ artifact จริง** ทุก key result มี field `evidence` การใช้ `kind = "file"` พร้อม path แบบ workspace-relative หมายความว่าหลักฐานคือไฟล์จริงที่เปิดและตรวจสอบได้ — ไม่ใช่ข้อความอ้างสิทธิ์ลอยๆ นี่คือ guard เดียวกับที่ plugin `audit` ใช้: ไม่มีการอ้างสิทธิ์โดยไม่มีสิ่งอ้างอิง

**OKR และ audit results เสริมกัน** plugin `audit` ตรวจว่า workspace เป็นไปตามมาตรฐาน (ISO, license headers, doc parity) plugin `okr` ติดตามว่าเป้าหมายที่ operator ตั้งไว้กำลังบรรลุหรือไม่ การรันทั้งสองจะให้มุมมอง compliance และ goals ในเทอร์มินัลเดียวกัน

**เชื่อมกับทีม** เมื่อทีมเป็นเจ้าของ objective ใส่ชื่อทีมใน `owner` ใช้ `bwoc team list` เพื่อดูทีมที่มีอยู่ และ `bwoc task add <team> "..."` เพื่อสร้าง task ที่สอดคล้องกับ key results ข้อมูล OKR ไม่ sync อัตโนมัติไปยัง team task list — คุณเชื่อมโดย naming convention และการสร้าง task ด้วยมือ

**Version control** เพราะ `objectives.toml` และ `key_results.toml` เป็นไฟล์ TOML ธรรมดาใน workspace จึงอยู่ใน git ทุก update จาก `track` คือการเปลี่ยนแปลงที่ `git diff` ดูได้ `git log` บน `key_results.toml` คือ audit trail น้ำหนักเบาของทุก check-in

---

## 8. ดูเพิ่มเติม

**Framework documentation (public GitHub)**

- [PLUGINS.en.md — OKR Progress Schema](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md#okr-progress-schema) — schema normative สำหรับ output ของ `report`
- [PLUGINS.en.md — okr kind](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md#plugin-kinds) — นิยาม kind, lifecycle owner, เหตุผลของ design
- [workspace-okrs plugin source](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/okr) — `manifest.toml`, `SPEC.md`, `okr.sh`, ไฟล์ seed data

**บท handbook ที่เกี่ยวข้อง**

- [`../fleet-ops/HANDBOOK.th.md`](../fleet-ops/HANDBOOK.th.md) — fleet health, supervise, visibility; complement เชิงปฏิบัติการของ OKR governance
- [`../agents/HANDBOOK.th.md`](../agents/HANDBOOK.th.md) — การ incarnate agent; `bwoc check`; ความสัมพันธ์ระหว่าง plugin กับ agent
- [`../glossary.th.md`](../glossary.th.md) — นิยามคำศัพท์

---

> **แหล่งข้อมูลที่เชื่อถือได้ที่สุด** handbook นี้สรุปและนำทาง หากมีความขัดแย้ง framework repo ถูกเสมอ และหน้านี้มี bug — โปรดแก้ไข Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)
