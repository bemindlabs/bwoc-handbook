---
title: Council — การตัดสินใจของ fleet อย่างมีโครงสร้าง
nav_order: 15
---

# Council — การตัดสินใจของ fleet อย่างมีโครงสร้าง

> ฉบับภาษาอังกฤษ: [HANDBOOK.en.md](HANDBOOK.en.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) · อภิธานศัพท์: [../glossary.th.md](../glossary.th.md) · Fleet Operations: [../fleet-ops/HANDBOOK.th.md](../fleet-ops/HANDBOOK.th.md) · Agents: [../agents/HANDBOOK.th.md](../agents/HANDBOOK.th.md)

**แหล่งความจริง.** สเปคฉบับ canonical อยู่ใน framework repo เมื่อขัดแย้งกัน repo ถูกเสมอและ handbook นี้มี bug เอกสารที่เกี่ยวข้อง: [`PLUGINS.en.md` §Council Decision Schema](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md#council-decision-schema) · [BWOC-56 council design note](https://github.com/bemindlabs/BWOC-Framework/blob/main/notes/2026-05-28_council-plugin-architecture.md)

---

## สารบัญ

1. [council คืออะไร](#1-council-คืออะไร)
2. [โปรโตคอล — propose → discuss → vote → resolve](#2-โปรโตคอล--propose--discuss--vote--resolve)
3. [แบบจำลองการลงคะแนน](#3-แบบจำลองการลงคะแนน)
4. [รัน council ด้วย `bwoc council`](#4-รัน-council-ด้วย-bwoc-council)
5. [บันทึก Council Decision](#5-บันทึก-council-decision)
6. [Binding กับ advisory](#6-binding-กับ-advisory)
7. [ดูเพิ่มเติม](#7-ดูเพิ่มเติม)

---

## 1. council คืออะไร

**council** คือโปรโตคอลแบบมีโครงสร้างของ BWOC ที่ให้ agent หลายตัว *ตัดสินใจร่วมกัน* และเก็บบันทึกที่ถาวรและตรวจสอบได้ว่าตัดสินใจอย่างไร มันเป็น plugin kind ประเภท **coordination** ตัวแรกของ framework: ที่ซึ่ง plugin `workflow`/`jira` เอื้อม *ออกไป* ยังระบบภายนอก และ `audit`/`okr` รายงาน *เหนือ* workspace, council ทำงาน **ในหมู่ agent ของ fleet เอง**

กฎที่นิยามมัน: **council บันทึกการตัดสินใจ ไม่ได้ดำเนินการตามนั้น** การ resolve council สร้าง decision entry — ผลลัพธ์ ร่องรอยการอภิปราย และความเห็นแย้งใดๆ ถ้าผลลัพธ์เป็น *binding* council จะปล่อย `bwoc task` ให้ใครสักคนไปทำ มันไม่เคยแก้ workspace ด้วยตัวเอง สิ่งนี้แยกการพิจารณาออกจากการลงมือทำอย่างชัดเจน

> [!note]
> council เหมาะเมื่อทางเลือก **เป็นที่ถกเถียง มีผลตามมาสูง และคู่ควรกับเหตุผลที่เขียนไว้** — การรับ architecture, การ retire agent, การยอมรับความเสี่ยง สำหรับ agent ตัวเดียวที่ทำงานของตัวเอง ไม่ต้องใช้ council; สำหรับการ route incident ให้ใช้ task list ของทีม (ดู [Fleet Operations](../fleet-ops/HANDBOOK.th.md))

---

## 2. โปรโตคอล — propose → discuss → vote → resolve

ทุก council เดินผ่าน state machine ที่ชัดเจน:

```
proposed → discussing → voting → resolved
                                  ↘ abandoned   (ถ้าไม่ถึง quorum)
```

- **propose** — agent ใดก็ได้เปิด decision ด้วยคำถามและ **ตัวเลือก ≥2** โดยระบุ `bwoc team` ที่เข้าร่วม
- **discuss** — ผู้เข้าร่วมผลัดกันพูดข้าม **round** หนึ่งหรือหลายรอบ แต่ละ turn เป็นข้อความ `bwoc send` จริง; บันทึก council เพียง *อ้างอิง* envelope (inbox คือ transport บันทึกคือ ledger)
- **vote** — ผู้เข้าร่วมแต่ละคนลงคะแนนให้ตัวเลือกหนึ่ง หรืองดออกเสียง คะแนนเป็นแบบ **append-only** — การลงใหม่ต่อท้าย ไม่เขียนทับ เพราะร่องรอยคือหัวใจ
- **resolve** — เมื่อถึง **quorum** และแบบจำลองการลงคะแนนเป็นจริง ผลลัพธ์ถูกบันทึกและ decision ปิด ถ้าไม่เคยถึง quorum decision จะถูก **abandoned** (ไม่ใช่ล้มเหลว — เพียงไม่เคยตัดสิน)

ผู้เข้าร่วมถูกดึงจากทีมที่อ้างถึงอย่างเคร่งครัด; คะแนนจากนอกทีมถูกปฏิเสธ

---

## 3. แบบจำลองการลงคะแนน

council ประกาศว่า *อย่างไร* คะแนนจึงกลายเป็นผลลัพธ์ มีสี่แบบจำลอง:

| แบบจำลอง | กฎผลลัพธ์ |
|---|---|
| `simple-majority` | ตัวเลือกที่ได้คะแนนมากที่สุดชนะ |
| `consensus` | ตัวเลือกชนะก็ต่อเมื่อ **ไม่มีผู้เข้าร่วมแย้ง** เท่านั้น |
| `weighted` | คะแนนมีน้ำหนักต่อผู้เข้าร่วม (เช่น ตามอาวุโสหรือความเชี่ยวชาญ) |
| `sangha` | แบบจำลอง `council-sangha-7` จาก **อปริหานิยธรรม 7** (เงื่อนไขความไม่เสื่อมเจ็ดข้อ): การตัดสินใจเอนเอียงไปทางการมีส่วนร่วมเต็มที่ ประชุมโดยพร้อมเพรียง และไม่ล้มล้างสิ่งที่ตกลงไว้แล้ว |

> [!tip]
> ใช้ `consensus` สำหรับการเปลี่ยนแปลงที่ย้อนยาก (ผู้เข้าร่วมที่ต้องอยู่กับผลที่แย่สามารถยับยั้งได้) และ `simple-majority` สำหรับทางเลือกประจำที่ quorum ชัดเจนก็เพียงพอ

---

## 4. รัน council ด้วย `bwoc council`

โปรโตคอล map ตรงไปยัง CLI การรันทั่วไป:

```bash
# 1. เปิด decision ในทีมที่มีอยู่
bwoc council propose security \
  --question "รับ Phase 5 sandbox contract ตามที่เป็นไหม?" \
  --option adopt --option revise

# 2. ผู้เข้าร่วมอภิปรายข้าม round (แต่ละ turn เป็น envelope ของ bwoc send)
bwoc council discuss D1 --from agent-yudi --message "ขอบเขตที่ tool-effect สมเหตุผล; ให้สัตยาบัน"

# 3. ผู้เข้าร่วมแต่ละคนลงคะแนน
bwoc council vote D1 --from agent-luban --option adopt
bwoc council vote D1 --from agent-nezha --option revise

# 4. resolve เมื่อถึง quorum + แบบจำลองการลงคะแนนเป็นจริง
bwoc council resolve D1
```

ผู้เข้าร่วมมาจาก `bwoc team` ที่ระบุตอน propose ดังนั้น council สร้างบนพื้นฐานทีม Saṅgha เดียวกับ task list ที่ใช้ร่วมกัน turn การอภิปรายวิ่งบน messaging แบบ envelope ที่ลงนามตามปกติ (`bwoc send` / `bwoc inbox`) ดังนั้นทุก turn ผ่าน trust gate และตรวจสอบได้เหมือนข้อความระหว่าง agent อื่นๆ

---

## 5. บันทึก Council Decision

การ resolve council เก็บ **decision entry** — รูปทรงที่ normative และ `bwoc check` ตรวจสอบ ฟิลด์ที่สำคัญต่อ operator:

| ฟิลด์ | ความหมาย |
|---|---|
| `decision_id` | คีย์ถาวรของ decision — identifier ที่ทนทานเพียงตัวเดียว |
| `status` | `proposed` / `discussing` / `voting` / `resolved` / `abandoned` |
| `participants` | agent id ตรึงตอน propose (การเปลี่ยนคือ decision *ใหม่*) |
| `options` | ตัวเลือก (≥2) ตรึงตอน propose เช่นกัน |
| `rounds` | turn การอภิปรายเรียงลำดับ แต่ละอันอ้าง envelope ของ `bwoc send` แบบ append-only |
| `votes` | `{participant, option, abstain}` หนึ่งต่อผู้ลงคะแนน แบบ append-only |
| `outcome` | ตัวเลือกที่ตัดสิน — มีเฉพาะเมื่อ `status = resolved` |
| `dissent` | จุดยืนเสียงข้างน้อยพร้อมเหตุผล — **เก็บไว้ ไม่ทิ้ง** |
| `evidence_links` | referent ที่หนุนการตัดสิน ใช้ Evidence kinds ของ audit ซ้ำ |

สองคุณสมบัติทำให้บันทึกน่าเชื่อถือ: เป็น **append-only** (round และ votes สะสม ไม่มีอะไรถูกเขียนใหม่) และ **dissent เป็นฟิลด์ชั้นหนึ่ง** — การบันทึก *ว่าทำไมเสียงข้างน้อยไม่เห็นด้วย* เป็นจุดประสงค์ที่ชัดเจนของ council ไม่ใช่ของแถม

```json
{
  "decision_id": "D1",
  "status": "resolved",
  "participants": ["agent-yudi", "agent-luban", "agent-nezha"],
  "options": ["adopt", "revise"],
  "votes": [{ "participant": "agent-yudi", "option": "adopt", "abstain": false }],
  "outcome": "adopt",
  "dissent": [{ "participant": "agent-nezha", "option": "revise", "rationale": "ต้องการ seccomp ใน v1" }],
  "opened_at": "2026-06-07T12:00:00Z",
  "closed_at": "2026-06-07T12:30:00Z"
}
```

---

## 6. Binding กับ advisory

council ที่ resolve แล้วเป็น **advisory** (บันทึกการตัดสินไว้เป็นร่องรอยและแจ้งมนุษย์/agent) หรือ **binding** (fleet ตกลงจะลงมือ) ผลลัพธ์ binding **ไม่** เปลี่ยนอะไรด้วยตัวมันเอง — มัน **ปล่อย `bwoc task`** ลง task list ของทีมที่เกี่ยวข้อง ดังนั้นงานไหลผ่าน task lifecycle ปกติ (claim → plan → approve → complete) council ตัดสิน; task list ดำเนินการ

> [!warning]
> อย่าต่อ council ให้แก้ state โดยตรงตอน resolve เด็ดขาด การแยก — การตัดสินอยู่ในบันทึก council การลงมืออยู่ใน task — คือสิ่งที่ทำให้ audit trail ซื่อสัตย์: คุณตอบได้เสมอว่า "ใครตัดสินสิ่งนี้ บนหลักฐานอะไร" แยกจาก "ใครเป็นคนทำ"

---

## 7. ดูเพิ่มเติม

**source framework (GitHub สาธารณะ):**

- [PLUGINS.en.md §Council Decision Schema](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md#council-decision-schema) — บันทึก decision ที่ normative
- [BWOC-56 council design note](https://github.com/bemindlabs/BWOC-Framework/blob/main/notes/2026-05-28_council-plugin-architecture.md) — รายละเอียดโปรโตคอล แบบจำลองการลงคะแนน quorum + tie-break และ `council-sangha-7`

**บทอื่นใน handbook:**

- [../fleet-ops/HANDBOOK.th.md](../fleet-ops/HANDBOOK.th.md) — ทีม task list ที่ใช้ร่วมกัน และสุขภาพ fleet
- [../agents/HANDBOOK.th.md](../agents/HANDBOOK.th.md) — agent การประกาศ trust messaging ที่ลงนาม
- [../plugin-craft/HANDBOOK.th.md](../plugin-craft/HANDBOOK.th.md) — plugin kind อย่าง `council` สร้างอย่างไร
- [../glossary.th.md](../glossary.th.md) — ทุกคำในตารางเดียว

---

*handbook นี้ดูแลควบคู่กับ framework เมื่อพฤติกรรมที่อธิบายที่นี่ต่างจาก [framework repo](https://github.com/bemindlabs/BWOC-Framework) repo ถูกต้อง — โปรดเปิด fix*
