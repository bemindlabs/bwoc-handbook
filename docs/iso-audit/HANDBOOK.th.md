---
title: "การตรวจสอบมาตรฐาน ISO ด้วย BWOC"
version: "1.0.0"
audience: "Workspace operator ที่ดำเนินการตรวจสอบ ISO compliance"
language: th
counterpart: HANDBOOK.en.md
---

> English version: [HANDBOOK.en.md](HANDBOOK.en.md) — Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) — Plugin source: [modules/plugins](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins)

🇬🇧 [English](HANDBOOK.en.md)

# การตรวจสอบมาตรฐาน ISO ด้วย BWOC

BWOC มาพร้อมกับ plugin ประเภท `audit` จำนวนเจ็ดตัวที่ตรวจสอบ workspace เทียบกับมาตรฐาน ISO / IEC / IEEE ทำงานตามคำสั่ง `bwoc audit run` ไม่เขียนข้อมูลใดออกไปภายนอก และส่งออกรายงาน JSON แบบ structured ที่ framework ตรวจสอบความถูกต้องในทุกครั้งที่รัน บทนี้ครอบคลุมว่าแต่ละ plugin ตรวจสอบอะไร โมเดลหลักฐานที่ใช้ร่วมกัน วิธีตั้งค่า และวิธีรัน

---

## สารบัญ

1. [เหตุใดจึงใช้ machine-assisted ISO audit](#1-เหตุใดจึงใช้-machine-assisted-iso-audit)
2. [Plugin ตรวจสอบ ISO ทั้งเจ็ดตัว](#2-plugin-ตรวจสอบ-iso-ทั้งเจ็ดตัว)
3. [โมเดลหลักฐานการตรวจสอบ](#3-โมเดลหลักฐานการตรวจสอบ)
4. [ติดตั้งและเปิดใช้งาน](#4-ติดตั้งและเปิดใช้งาน)
5. [ตั้งค่าหลักฐานใน workspace.toml](#5-ตั้งค่าหลักฐานใน-workspacetoml)
6. [การรันการตรวจสอบ](#6-การรันการตรวจสอบ)
7. [การเชื่อมโยงกับ governance และ security](#7-การเชื่อมโยงกับ-governance-และ-security)
8. [ดูเพิ่มเติม](#8-ดูเพิ่มเติม)

---

## 1. เหตุใดจึงใช้ machine-assisted ISO audit

การตรวจสอบ ISO มักล้มเหลวด้วยเหตุผลที่คาดเดาได้: หลักฐานกระจัดกระจายอยู่ตามเอกสารต่าง ๆ สถานะไม่ชัดเจนจนกว่า auditor จะถาม และช่องว่างถูกพบช้าเกินไปจนไม่สามารถแก้ไขได้ทัน `audit` kind ของ BWOC แก้ปัญหานี้โดยไม่แทนที่ human auditor

Plugin เหล่านี้ทำงานตามหลักการที่เรียกว่า Musāvāda — ไม่มีการอ้างสิทธิ์โดยปราศจากหลักอ้างอิง ทุก finding ที่ผ่านจะชี้ไปยัง artifact ที่เป็นรูปธรรม ได้แก่ ไฟล์ ผลลัพธ์คำสั่ง attestation ที่ operator ลงนาม หรืออัตราที่วัดได้จากเครื่องมือ finding ไม่สามารถผ่านได้เพราะ agent คาดเดา ช่องว่างจะปรากฏเป็น `fail` finding ทุกครั้งที่รัน ไม่ใช่ความประหลาดใจในวันก่อนการตรวจสอบจากภายนอก

สิ่งที่ machine assistance ให้คือความสม่ำเสมอและความสามารถในการทำซ้ำ คำสั่ง `bwoc audit run` เดิมให้ผลลัพธ์เดิมบน workspace เดิม คุณสามารถรันใน CI เปรียบเทียบผลลัพธ์ระหว่าง sprint และนำ JSON ไปใช้กับ dashboard หรือชุดหลักฐาน — โดยไม่ต้องใช้ checklist manual

Framework ไม่กำหนดเกณฑ์หรือตัดสินใจเรื่องการรับรอง นั่นยังคงเป็นบทบาทของ auditor Plugin นำเสนอหลักฐานที่ machine ยืนยันแล้วและรายงานช่องว่างที่ตรงไปตรงมา ส่วน human auditor เป็นผู้ตีความ

---

## 2. Plugin ตรวจสอบ ISO ทั้งเจ็ดตัว

| Plugin | มาตรฐาน | สิ่งที่ตรวจสอบ | ประเภทหลักฐาน |
|---|---|---|---|
| `audit-iso-29110` | ISO/IEC TR 29110 (VSE software lifecycle, Basic profile) | Work product หกรายการของ Basic profile — Project Plan, SRS, Software Design, Test Plan, Verification Results, Construction Records — โดยตรวจสอบจากการมีอยู่ของไฟล์ใน workspace | `file` |
| `audit-iso-9001` | ISO 9001:2015 (ระบบบริหารคุณภาพ) | เกณฑ์หลัก QMS แปดข้อ — บริบทองค์กร นโยบายคุณภาพ การจัดการความเสี่ยง ความสามารถ ข้อมูลที่บันทึก การตรวจสอบภายใน การทบทวนของฝ่ายบริหาร และการดำเนินการแก้ไข | `attestation` |
| `audit-iso-20000-1` | ISO/IEC 20000-1:2018 (การบริหารจัดการบริการ IT) | เกณฑ์ ITSM แปดข้อครอบคลุม scope นโยบาย service catalogue ประสิทธิภาพ SLA การบริหารการเปลี่ยนแปลง การบริหาร incident และ problem และการปรับปรุงอย่างต่อเนื่อง | `attestation` + `sample` |
| `audit-iso-27001` | ISO/IEC 27001:2022 (ระบบบริหารความมั่นคงปลอดภัยสารสนเทศ) | เกณฑ์ ISMS หลักห้าข้อจาก main body และ Annex A controls สามข้อ โดยใช้ Statement of Applicability กำหนด scope ของ Annex A | `attestation` + `sample` (SoA-gated) |
| `audit-iso-iec-ieee-29148` | ISO/IEC/IEEE 29148:2018 (Requirements Engineering) | เกณฑ์ RE เจ็ดข้อ — StRS, SyRS/SRS, คุณลักษณะ requirement รายข้อ + ทั้งชุด, verifiability, traceability สองทาง, requirements management | `attestation` |
| `audit-iso-iec-ieee-12207` | ISO/IEC/IEEE 12207:2017 (Software Life Cycle Processes) | เกณฑ์ process เก้าข้อ — agreement, project planning, assessment & control, configuration management, requirements, architecture/design, implementation/integration, V&V, maintenance | `attestation` |
| `audit-ieee-1012` | IEEE 1012:2016 (Verification & Validation) | เกณฑ์ V&V แปดข้อ — integrity levels, V&V planning (SVVP), independence, และ requirements/design/implementation/test V&V + anomaly reporting | `attestation` |

Plugin ทั้งเจ็ดประกาศ `kind = "audit"` ใน manifest และส่งออก finding ตาม [Audit Findings Schema ของ framework](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md#audit-findings-schema) เจ้าของ lifecycle ของ `audit` kind คือ CLI `bwoc audit` — plugin ประเภทนี้ไม่เคยถูกเรียกใช้โดยอัตโนมัติ รันได้เฉพาะเมื่อ operator สั่ง `bwoc audit run` เท่านั้น

**kind นี้เป็นกลางต่อองค์กรมาตรฐาน** เริ่มจาก **ISO** ล้วน (9001) และ **ISO/IEC** ร่วม (27001, 20000-1, 29110) แล้วเพิ่ม **ISO/IEC/IEEE** ร่วม (29148 requirements, 12207 life cycle) และ **IEEE** เดี่ยว (1012 V&V) — criterion id, ชื่อมาตรฐาน, clause เป็นข้อมูลที่ runtime อ่าน ไม่ใช่ข้อจำกัดว่าองค์กรไหน (หรือกี่องค์กร) ประกาศ สาม lane assurance รวมเป็นชุดที่สอดคล้อง: 29148 ถาม *requirements ถูกไหม?*, 12207 *วงจรชีวิตถูกกำกับไหม?*, 1012 *verify & validate แล้วหรือยัง?*

### สรุปแต่ละ plugin

**audit-iso-29110** เป็น plugin ที่ตรงไปตรงมาที่สุดในเจ็ดตัว ทำการตรวจสอบการมีอยู่ของไฟล์เทียบกับ work product ของ Basic profile ตาม ISO/IEC TR 29110-5-1-2 Plugin อ่าน `candidates` list สำหรับแต่ละเกณฑ์ — เส้นทางไฟล์ทางเลือกที่ workspace อาจใช้ — และผ่านเกณฑ์เมื่อพบไฟล์แรกที่มีอยู่ ไม่ต้องตั้งค่าอะไรนอกจาก `enabled = true`

**audit-iso-9001** ตรวจสอบแปดข้อหลักของ ISO 9001:2015 ไม่มีข้อใดที่สามารถตรวจสอบจากการมีอยู่ของไฟล์ได้ — "มีการทบทวนของฝ่ายบริหารหรือไม่?" ไม่สามารถตอบได้จากชื่อไฟล์ Plugin อ่าน attestation ที่ operator ลงนามจาก `workspace.toml` และส่งออก attestation finding สำหรับแต่ละเกณฑ์ที่ครอบคลุม หรือ `fail` สำหรับที่ยังขาดอยู่

**audit-iso-20000-1** ครอบคลุมทั้งเกณฑ์ที่เป็น documented-artifact (scope นโยบาย service catalogue) และเกณฑ์อัตราปฏิบัติการ (SLA การเปลี่ยนแปลง incident problem การปรับปรุง) เกณฑ์ documented-artifact ใช้ attestation เกณฑ์อัตราปฏิบัติการใช้ sample — อัตราที่ operator คัดลอกจากเครื่องมือ ITSM (`sampled_count` / `sampled_of` / `window` แบบ optional)

**audit-iso-27001** เป็น plugin ที่ซับซ้อนที่สุดในกลุ่ม ISO management-system สี่ตัว เกณฑ์แปดข้อแบ่งระหว่างข้อ attestation จาก main body clauses และ Annex A controls ที่ขับเคลื่อนด้วย sample Statement of Applicability (`[[plugins.audit-iso-27001.soa]]`) เป็น machine-readable: ประกาศว่า Annex A controls ใดอยู่ใน scope (`applicable = true`) และให้เหตุผลสำหรับทั้งที่รวมและที่ยกเว้น population ในการ sampling (`sampled_of`) สำหรับ Annex A finding ถูกคำนวณจาก SoA — operator ไม่ต้องพิมพ์ตัวเลขเอง

---

## 3. โมเดลหลักฐานการตรวจสอบ

Plugin ทั้งเจ็ดใช้ [Audit Findings Schema](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md#audit-findings-schema) ร่วมกัน ทุก finding มี `criterion_id`, `severity`, `status`, และ `evidence` block finding ที่ผ่านจะไม่มี `remedy` finding ที่ไม่ผ่านต้องมี `remedy` เสมอ

ประเภทหลักฐานสามชนิดที่ใช้โดย ISO audit plugin:

**`file`** — หลักฐานคือเส้นทางไฟล์ที่เกี่ยวกับ workspace ใช้โดย `audit-iso-29110` (ไฟล์ต้องมีอยู่) และเป็น pointer สำรองเมื่อประเภทหลักฐานอื่นหายไป (เช่น "เพิ่ม attestation ของคุณใน `.bwoc/workspace.toml`")

**`attestation`** — หลักฐานคือการยืนยันที่ operator ลงนาม ใช้โดย plugin ระบบบริหารจัดการทั้งสาม (9001, 20000-1, 27001) สำหรับเกณฑ์ที่ไม่สามารถตรวจสอบด้วย machine ได้เพียงอย่างเดียว เช่น การให้สัตยาบันนโยบายคุณภาพ การเสร็จสิ้นการประเมินความเสี่ยง การกำหนด scope ISMS fields ที่ต้องการคือ `signer` (ตัวตนแบบ free-text เช่น `"CISO: สุชาดา น."`) และ `signed_at` (วันที่ ISO 8601) `valid_through` แบบ optional ระบุว่า attestation หมดอายุเมื่อใด framework บันทึกแต่ไม่บังคับใช้การหมดอายุ — นั่นเป็นหน้าที่ของเครื่องมือ downstream

**`sample`** — หลักฐานคืออัตราที่วัดได้ ใช้โดย `audit-iso-20000-1` (เกณฑ์ปฏิบัติการทั้งห้า) และ `audit-iso-27001` (Annex A controls สามข้อ) สำหรับ `audit-iso-20000-1` operator ระบุ `sampled_count`, `sampled_of` และ `window` แบบ optional โดยตรง สำหรับ `audit-iso-27001` `sampled_count` และ `sampled_of` ถูกคำนวณจาก SoA

### ตัวอย่าง: triad หลักฐานของ 27001

ISO/IEC 27001 plugin ใช้ทั้งสาม block ร่วมกัน ต่อไปนี้คือภาพรวมย่อแต่ครบถ้วน:

```toml
[plugins.audit-iso-27001]
enabled = true

# Attestation: main-body management-system clauses
[[plugins.audit-iso-27001.attestations]]
criterion_id  = "27001-risk-assessment"
statement     = "ดำเนินการประเมินความเสี่ยงความมั่นคงปลอดภัยสารสนเทศ 2026-03-10 โดยมีระเบียบวิธีที่บันทึกไว้และผลลัพธ์สามารถทำซ้ำได้"
signer        = "CISO: สุชาดา น."
signed_at     = "2026-03-10"
valid_through = "2027-03-10"

# Statement of Applicability: scope ที่ machine อ่านได้สำหรับ Annex A
[[plugins.audit-iso-27001.soa]]
control       = "A.5.15"
applicable    = true
justification = "การควบคุมการเข้าถึงเป็นสิ่งสำคัญในการปกป้อง source code ข้อมูลรับรอง และข้อมูลลูกค้า"

[[plugins.audit-iso-27001.soa]]
control       = "A.5.29"
applicable    = false
justification = "ไม่มีโปรแกรม continuity อย่างเป็นทางการ ความเสี่ยงได้รับการยอมรับโดยฝ่ายบริหารสำหรับ workspace ที่ดูแลคนเดียว"

# Samples: Annex A controls ที่อยู่ใน scope (ไม่ต้องระบุ sampled_count/of — คำนวณจาก SoA)
[[plugins.audit-iso-27001.samples]]
criterion_id = "27001-access-control"
summary      = "ดำเนินการ access review ในทุกระบบที่อยู่ใน scope ไม่พบบัญชีที่ถูกทิ้งร้าง"
window       = "2026-Q1"
```

Runtime เลือก route ให้แต่ละเกณฑ์ตาม `expected_evidence_kind`:
- `27001-risk-assessment` เป็นเกณฑ์ `attestation` — ตรวจสอบ entry `[[…attestations]]` ที่ตรงกัน
- `27001-access-control` เป็นเกณฑ์ `sample` — ค้นหา `A.5.15` ใน SoA ยืนยัน `applicable = true` คำนวณ `sampled_of` จากจำนวน controls ที่อยู่ใน scope จากนั้นตรวจสอบ entry `[[…samples]]`
- หาก `A.5.29` ถูกยกเว้น (`applicable = false`) และมีเหตุผล finding จะเป็น `status = "not_applicable"` — ไม่ใช่ pass ไม่ใช่ fail

เกณฑ์ที่ไม่มีหลักฐานใน workspace จะส่งออก `status = "fail"` พร้อม `remedy` ที่ระบุ block ที่ต้องเพิ่มใน `workspace.toml` อย่างชัดเจน

---

## 4. ติดตั้งและเปิดใช้งาน

Plugin ตรวจสอบ ISO ทั้งเจ็ดมาพร้อมกับ framework ใน `modules/plugins/` ไม่จำเป็นต้องติดตั้งแยก มีอยู่พร้อมใช้ทุกครั้งที่ framework พร้อมใช้งาน

เพื่อเปิดใช้งาน plugin หนึ่ง ให้เพิ่ม block ลงใน `.bwoc/workspace.toml`:

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

`enabled = true` เป็น key เดียวที่ประกาศใน manifest สำหรับ plugin ทั้งเจ็ด การตั้งค่าหลักฐานทั้งหมดอยู่ใน block array-of-tables ต่อ plugin (ดูหัวข้อถัดไป) — ไม่มี `[config.schema]` ใน manifest ของ plugin เหล่านี้

เพื่อปิดการใช้งาน plugin โดยไม่ลบ block หลักฐาน:

```bash
bwoc plugin disable audit-iso-27001
```

คำสั่งนี้จะตั้ง `enabled = false` ใน `workspace.toml` และเก็บ block `[[…attestations]]` / `[[…soa]]` / `[[…samples]]` ไว้ครบถ้วน

---

## 5. ตั้งค่าหลักฐานใน workspace.toml

แต่ละ plugin อ่านหลักฐานจาก `.bwoc/workspace.toml` รูปแบบแตกต่างกันตาม plugin

### audit-iso-29110

ไม่ต้องตั้งค่าหลักฐาน Plugin อ่าน `criteria.toml` จาก directory ของตัวเองและตรวจสอบ workspace สำหรับ `candidates` list ของแต่ละเกณฑ์ ตั้ง `enabled = true` แล้วรันได้เลย

### audit-iso-9001

เกณฑ์ attestation แปดข้อ ระบุ block `[[plugins.audit-iso-9001.attestations]]` หนึ่งรายการต่อเกณฑ์ที่สามารถ attest ได้ เกณฑ์ที่ไม่มี attestation จะส่งออก `fail`

```toml
[[plugins.audit-iso-9001.attestations]]
criterion_id  = "9001-management-review"
statement     = "ดำเนินการทบทวนของฝ่ายบริหาร 2026-04-15 ครอบคลุมผลการดำเนินงาน QMS ไตรมาส 1 ความคิดเห็นลูกค้า และโอกาสในการปรับปรุง"
signer        = "Quality Manager: ต้นกล้า ก."
signed_at     = "2026-04-15"
valid_through = "2027-04-15"   # optional
```

criterion IDs ทั้งแปดได้แก่: `9001-context-of-organization`, `9001-leadership-and-policy`, `9001-risks-and-opportunities`, `9001-competence-and-awareness`, `9001-documented-information`, `9001-internal-audit`, `9001-management-review`, `9001-corrective-action`

### audit-iso-20000-1

เกณฑ์ attestation สาม + sample ห้าข้อ เกณฑ์ attestation ใช้ `[[plugins.audit-iso-20000-1.attestations]]` เกณฑ์ sample ใช้ `[[plugins.audit-iso-20000-1.samples]]`

```toml
[[plugins.audit-iso-20000-1.attestations]]
criterion_id = "20000-1-service-policy-and-objectives"
statement    = "นโยบายการบริหารจัดการบริการ v2.1 ได้รับการให้สัตยาบัน 2026-01-15 มีการทบทวน objective รายไตรมาส"
signer       = "Service Owner: ต้นกล้า ก."
signed_at    = "2026-01-15"

[[plugins.audit-iso-20000-1.samples]]
criterion_id  = "20000-1-incident-management"
summary       = "49 จาก 50 incident ได้รับการแก้ไขภายใน SLA"
sampled_count = 49
sampled_of    = 50
window        = "2026-Q1"
```

สำหรับเกณฑ์ sample `sampled_count` และ `sampled_of` เป็น integer ที่ต้องระบุ `window` เป็น optional runtime ไม่กำหนดเกณฑ์ผ่าน — บันทึกอัตราและนำเสนอต่อ human auditor

criterion IDs สำหรับ attestation: `20000-1-service-management-system-scope`, `20000-1-service-policy-and-objectives`, `20000-1-service-catalogue` สำหรับ sample: `20000-1-service-level-management`, `20000-1-change-management`, `20000-1-incident-management`, `20000-1-problem-management`, `20000-1-continual-improvement`

### audit-iso-27001

เกณฑ์ attestation ห้า + sample สาม ที่ SoA กำหนด scope ใช้สาม block ได้แก่ `[[…attestations]]`, `[[…soa]]`, และ `[[…samples]]`

Block `[[…soa]]` คือ Statement of Applicability Annex A control ทุกข้อที่ plugin ตรวจสอบ (`A.5.15`, `A.5.24`, `A.5.29`) ต้องระบุที่นี่พร้อม `applicable` และ `justification` ข้อกำหนด ISO/IEC 27001 ข้อ 6.1.3 กำหนดให้มีเหตุผลสำหรับทั้งที่รวมและที่ยกเว้น

สำหรับ block `[[…samples]]` ของ 27001 **ไม่ต้อง** ระบุ `sampled_count` หรือ `sampled_of` — runtime คำนวณจาก SoA ระบุเฉพาะ `criterion_id`, `summary` และ `window` แบบ optional

criterion IDs สำหรับ attestation: `27001-isms-scope`, `27001-information-security-policy`, `27001-risk-assessment`, `27001-statement-of-applicability`, `27001-internal-audit` สำหรับ sample (Annex A): `27001-access-control` (A.5.15), `27001-incident-management` (A.5.24), `27001-business-continuity` (A.5.29)

---

## 6. การรันการตรวจสอบ

```bash
# รัน plugin เดียว
bwoc audit run --plugin audit-iso-29110

# รัน audit plugin ทั้งหมดที่เปิดใช้งาน
bwoc audit run

# JSON output (อ่านได้ด้วย machine)
bwoc audit run --plugin audit-iso-27001 --json
```

Dispatcher จะ spawn `audit.sh` ของแต่ละ plugin โดยส่งตัวแปร environment สามตัว: `BWOC_WORKSPACE` (absolute path ไปยัง workspace root ที่กำลังตรวจสอบ), `BWOC_PLUGIN_DIR` (absolute path ไปยัง directory ของ plugin), และ `BWOC_AUDIT_OPERATION` (เสมอเป็น `audit_run`) Plugin เขียน JSON findings array ไปยัง stdout framework ห่อด้วย envelope:

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

### Exit code

| Code | ความหมาย |
|---|---|
| `0` | ไม่มี `fail` findings |
| `1–254` | จำนวน `fail` findings (จำกัดที่ 254 ตัวเลขแน่นอนอยู่ใน `summary.fail_count`) |
| `255` | Framework หรือ plugin error — รายงานไม่น่าเชื่อถือ |
| `2` | Operator/usage error — ไม่พบ workspace หรือ plugin ที่ระบุไม่ได้ติดตั้ง |

Finding `not_applicable` และ `not_implemented` ไม่นับเป็น failure ใช้ `$?` ใน CI เพื่อตัดสินจากผลการตรวจสอบโดยไม่ต้อง parse stdout

### การอ่านผลลัพธ์

แต่ละ finding ใน `--json` output มี:

- `criterion_id` — identifier ที่คงที่ (เช่น `27001-risk-assessment`) คงที่ข้ามเวอร์ชัน plugin การเปลี่ยนชื่อถือเป็น major version bump
- `severity` — `info`, `low`, `medium`, `high` หรือ `critical` นี่คือความสำคัญของเกณฑ์ ไม่ใช่ผลลัพธ์ finding `critical` ที่มี `status = "pass"` หมายความว่าสิ่งที่สำคัญที่สุดผ่านการตรวจสอบ
- `status` — `pass`, `fail`, `not_applicable` หรือ `not_implemented`
- `evidence` — สิ่งที่ plugin ตรวจสอบ finding ที่ `pass` มี referent ที่เป็นรูปธรรมเสมอ
- `remedy` — มีใน finding ที่ไม่ใช่ pass ทุกตัว ระบุสิ่งที่ต้องเพิ่มใน `workspace.toml` หรือไฟล์ที่ต้องสร้างอย่างชัดเจน

---

## 7. การเชื่อมโยงกับ governance และ security

Plugin ทั้งเจ็ดครอบคลุม governance layer ต่างกันและทำงานร่วมกันกับ BWOC ได้ดี

**ISO/IEC 29110** เป็น foundation layer สำหรับ workspace BWOC ที่ใช้ในการพัฒนาซอฟต์แวร์ work product หกรายการที่ตรวจสอบ — Project Plan, SRS, Design, Test Plan, Verification Results, Construction Records — map ตรงกับเอกสารที่ agent ผลิตและอ้างอิง การรัน `audit-iso-29110` ให้ gap report ของ project artifact

**ISO 9001** ครอบคลุม management-system layer — ว่าองค์กรที่ดำเนินการ workspace ได้บันทึก quality practice และทำให้ leadership รับผิดชอบหรือไม่ เสริมกับ agent-level governance ที่ BWOC ให้ผ่านสัญญาณสุขภาพ fleet Aparihāniya-dhamma 7 (ดู [glossary](../glossary.th.md))

**ISO/IEC 20000-1** ครอบคลุม IT service delivery สำหรับ workspace ที่ agent เป็นส่วนหนึ่งของทีมบริการ (การตอบสนอง incident การบริหารการเปลี่ยนแปลง การติดตาม SLA) plugin นี้แสดงว่า service management practice ได้รับการบันทึกและวัดผลหรือไม่

**ISO/IEC 27001** คือ information-security layer Statement of Applicability ที่ machine อ่านได้ใน `workspace.toml` คือบันทึก live ของการตัดสินใจ scope ของ control รวมกับ threat model ระดับ agent ของ framework ให้มุมมอง security posture แบบบูรณาการ

เนื่องจาก plugin ทั้งเจ็ดเป็น read-only — ตรวจสอบ workspace และส่งออกรายงาน ไม่เขียนอะไร — จึงไม่มี operator-confirmation gate คุณสามารถรันบ่อยเท่าที่ต้องการโดยไม่มีผลข้างเคียง

---

## 8. ดูเพิ่มเติม

- [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) — plugin spec ของ framework Audit Findings Schema ประเภทหลักฐาน exit code
- Plugin source — `SPEC.md` ของแต่ละ plugin มีตาราง criteria ครบถ้วนและ configuration reference:
  - [audit-iso-29110](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/audit-iso-29110)
  - [audit-iso-9001](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/audit-iso-9001)
  - [audit-iso-20000-1](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/audit-iso-20000-1)
  - [audit-iso-27001](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/audit-iso-27001)
- [../glossary.th.md](../glossary.th.md) — คำศัพท์ภาษาบาลี (Musāvāda, Aparihāniya-dhamma 7, Samānattatā)
- [../developer/HANDBOOK.th.md](../developer/HANDBOOK.th.md) — การพัฒนา plugin วิธี author `audit`-kind plugin ใหม่

---

*กลับไปยัง handbook index: [`../README.md`](../README.md)*
