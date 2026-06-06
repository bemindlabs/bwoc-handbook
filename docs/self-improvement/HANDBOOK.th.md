---
title: ความสามารถและการพัฒนาตนเองของ Agent
nav_order: 9
---

# ความสามารถและการพัฒนาตนเองของ Agent

🇬🇧 [English](HANDBOOK.en.md)

> **แหล่งข้อมูลที่เชื่อถือได้:** [`SELF-IMPROVEMENT.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/SELF-IMPROVEMENT.en.md) · [`PHILOSOPHY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) · หากมีข้อขัดแย้ง repo ของ framework คือสิ่งที่ถูกต้อง — handbook นี้มี bug กรุณาแก้ไข

---

## "Agent ที่พัฒนาตนเองได้" หมายความว่าอะไรในที่นี้

ในซอฟต์แวร์ทั่วไป การพัฒนาหมายถึงการ deploy โมเดลใหม่หรือเขียนโค้ดใหม่ด้วยมือ แต่ BWOC agent พัฒนาตัวเองได้จริง — ข้ามหลาย session — โดยบันทึกสิ่งที่เรียนรู้, ไตร่ตรองการตัดสินใจ, และนำบทเรียนที่ผ่านการตรวจสอบแล้วกลับมาเป็นความรู้ใช้งานได้จริง

สามสิ่งที่ทำให้เรื่องนี้เป็นรูปธรรม:

1. Agent **ประกาศสิ่งที่ตนเองทำได้และทำไม่ได้** ก่อนเริ่มงาน เพื่อให้ความคาดหวังสอดคล้องกับความจริง
2. ความรู้ใหม่ทุกชิ้นถูกเขียนลงในไฟล์ด้วยรูปแบบที่กำหนด ไม่ใช่ทิ้งไว้ใน context ของ conversation ที่จะหายไปเมื่อ session สิ้นสุด
3. บทเรียนสามารถยกระดับได้: บันทึกส่วนตัวสามารถกลายเป็น pattern, แล้วเป็นกฎร่วม, และสุดท้ายกลายเป็น convention ระดับ fleet — เส้นทาง curation จากปัจเจกสู่ส่วนรวม

บทนี้ครอบคลุมทั้งหมด: การประกาศความสามารถ, บันได maturity 7 ระดับ, วงจรการเรียนรู้ 3 รากฐาน, ชั้นความจำและคำสั่ง, pipeline การ curate, engine ของงานที่ขับเคลื่อนทั้งหมด, lifecycle 4 ขั้น, metrics, anti-patterns และ checklist ที่ใช้จริงเพื่อให้ agent เรียนรู้ได้

---

## ส่วนที่ 1 — โมเดลความสามารถ

### 1.1 การประกาศความสามารถ

ก่อนที่ agent จะทำงานใดๆ มันจะระบุสิ่งที่ตนเองทำได้และทำไม่ได้ นี่คือ "การประกาศความสามารถ" (คำ Pali คือ *Attanutata* แปลว่า "รู้จักตนเอง" — ดู [glossary](../glossary.th.md))

ในทางปฏิบัติ การประกาศความสามารถอยู่ในสองที่:

- **`config.manifest.json`** — ฟิลด์ `scope` และ `out_of_scope` อ่านได้ด้วยเครื่อง; `bwoc check` ตรวจสอบ
- **`persona/identity.md`** — คำอธิบายที่มนุษย์อ่านได้เกี่ยวกับ domain, จุดแข็ง และข้อจำกัดที่ชัดเจนของ agent

เมื่อ agent ได้รับงานที่อยู่นอก scope ที่ประกาศไว้ ควรปฏิเสธแทนที่จะพยายามทำในสิ่งที่ไม่ได้เตรียมพร้อม นี่ไม่ใช่ความล้มเหลว — มันคือความซื่อสัตย์

> **หมายเหตุ:** Agent ที่มีการประกาศความสามารถที่คลุมเครือหรือไม่มีเลย ไม่สามารถพัฒนาตนเองได้อย่างน่าเชื่อถือ การประกาศนี้คือเส้นฐานที่ใช้วัดความก้าวหน้า

### 1.2 Skills และบันได maturity L1–L7

ความสามารถที่เป็นรูปธรรมถูกบรรจุเป็น **skills** — ไฟล์ใน `skills/` หนึ่งไฟล์ต่อหนึ่งด้านความสามารถ แต่ละ skill มีฟิลด์ `maturity` ใน frontmatter ซึ่งกำหนดระดับจาก 7 ระดับ framework เรียกโมเดลนี้ว่า seven noble treasures (*Ariya-dhana 7*) โดยแต่ละระดับแผนที่กับคุณสมบัติที่ agent พัฒนาขึ้นในด้านนั้น

| ระดับ | ชื่อ | ความหมายในทางปฏิบัติ |
|---|---|---|
| **L1** | ผู้เริ่มต้น — *Saddhā* (ความเชื่อมั่น) | ปฏิบัติตาม convention ตามที่เขียนไว้; พึ่งพาตัวอย่าง; ยังไม่สามารถใช้เหตุผลกับกรณีขอบเขต |
| **L2** | ผู้ปฏิบัติตามกฎ — *Sīla* (วินัย) | ปฏิบัติตามกฎอย่างสม่ำเสมอโดยไม่ต้องกระตุ้น; สามารถจับข้อผิดพลาดที่ชัดเจน |
| **L3** | ผู้ตระหนัก — *Hiri-Ottappa* (ความตระหนักในข้อผิดพลาด) | สังเกตเห็นเมื่อมีบางอย่างผิดพลาดก่อนที่จะแตก; สามารถอธิบายเหตุผลได้ |
| **L4** | ผู้มีความรู้ — *Suta* (ความลึกของความรู้) | เข้าใจว่าทำไมกฎถึงมีอยู่; สามารถปรึกษาและสังเคราะห์แหล่งข้อมูล; อธิบายให้คนอื่นได้ |
| **L5** | ผู้แบ่งปัน — *Cāga* (การให้) | ดึง pattern ออกมาและนำกลับไปสนับสนุน; เริ่ม mentoring; ความรู้ถูกแชร์ออกไปนอกตัวบุคคล |
| **L6** | ผู้มีวิจารณญาณ — *Paññā* (ปัญญาอิสระ) | ตัดสินใจอย่างถูกต้องในสถานการณ์ใหม่; รู้ว่าเมื่อไรกฎไม่ครอบคลุมกรณีนั้น |
| **L7** | ความเชี่ยวชาญ | ออกแบบ convention ใหม่; สามารถ retire และถูกแทนที่ได้อย่างสะอาด; skill อยู่รอดนาน agent |

**สิ่งที่ตรวจสอบได้ในแต่ละระดับ:**

- L1–L2: complete งานบนเส้นทางที่คุ้นเคยได้
- L3–L4: คุณภาพของ post-mortem — agent ระบุสาเหตุที่แท้จริง ไม่ใช่แค่อาการ?
- L5: มี pattern ปรากฏใน `skills/` หรือ knowledge base ร่วมกันหรือไม่?
- L6: การตัดสินใจมีเหตุผลดีใน `memories/decision-*.md` ครอบคลุม alternatives และเงื่อนไขการทบทวนหรือไม่?
- L7: agent กำลัง mentor คนอื่นและทิ้ง convention ที่นำกลับมาใช้ได้ไว้เบื้องหลังหรือไม่?

Skills มีขอบเขตและตรวจสอบได้ skill ที่ครอบคลุมทุกอย่างไม่มีคุณค่า: scope แคบ, maturity ชัดเจน, ข้อจำกัดซื่อสัตย์

---

## ส่วนที่ 2 — สามวิธีที่ Agent เรียนรู้

โมเดลการเรียนรู้สร้างขึ้นจาก Three Roots of Wisdom (*Paññā 3* — ดู [glossary](../glossary.th.md)) ปัญญามีแหล่งที่มาสามแหล่งที่เป็นอิสระต่อกัน; ขาดแหล่งใดแหล่งหนึ่งจะให้ผลลัพธ์ที่ตื้นหรือผิดพลาด

```
ศึกษา ──────► ไตร่ตรอง ──────► ปฏิบัติ
 (suta)         (cintā)          (bhāvanā)
    ▲                                │
    └───── ป้อนกลับเข้ามา ────────────┘
              (หลังการ curate)
```

### 2.1 การศึกษา — เรียนรู้จากการอ่าน (*Sutamayā*)

**สิ่งที่กระตุ้น:** เริ่ม session, ก่อนงานที่ไม่คุ้นเคย, เมื่อพบสิ่งที่ไม่รู้

**Input:**
- `AGENTS.md` และ backend symlinks
- `conventions/*.md`
- `docs/` ใน directory ของ agent
- skill files ของ agent อื่นและ knowledge base ร่วมกัน
- Tier-2 cross-agent memory (ดูส่วนที่ 4)

**กิจกรรม:** โหลด document ที่เกี่ยวข้องก่อนทำงาน ไม่ใช่ระหว่างทำงาน "ค้นหาระหว่างทำงาน" ช้ากว่าและไม่น่าเชื่อถือเท่าการโหลดไว้ก่อน สำหรับแนวคิดที่ไม่รู้จัก ให้ค้นใน skill files และ framework knowledge base ก่อน; ค่อยขยายการค้นหาเมื่อพบว่าไม่มีข้อมูลในนั้น

**สิ่งที่เขียนลงไฟล์:** `memories/reference-*.md`

```markdown
# memories/reference-postgres-naming.md
---
type: reference
source: conventions/database.md#naming
date: 2026-05-22
verifiedAgainst: schema.sql@abc123
---

PostgreSQL naming conventions ที่ใช้ที่นี่:
- Tables: snake_case plural
- Columns: snake_case
- Indexes: idx_<table>_<columns>
```

**การตรวจสอบคุณภาพของ reference file:**
- แหล่งที่มาสามารถตามดูได้ (link หรือ file reference)?
- มีวันที่ตรวจสอบ — เคยตรวจสอบกับโค้ดจริงหรือไม่?
- เลือกสรร? การ dump เนื้อหาทั้ง document ไม่ใช่ reference — มันคือ noise

### 2.2 การไตร่ตรอง — เรียนรู้จากการใช้เหตุผล (*Cintāmayā*)

**สิ่งที่กระตุ้น:** pattern ที่ซ้ำกันข้ามงาน, ก่อนการตัดสินใจสถาปัตยกรรมสำคัญ, หลังจากสังเคราะห์แหล่งอ้างอิงหลายแหล่ง

**กิจกรรม:** ดึง pattern ออกมา, เขียนเหตุผลการตัดสินใจ, เชื่อมต่อแหล่งข้อมูลที่ยังไม่ได้เชื่อมกัน, และจำลองผลลัพธ์ ("ถ้าทำ X แล้วจะเกิดอะไร?") ขั้นตอนสุดท้ายนี้ — ติดตามผลลัพธ์ก่อนลงมือ — เรียกว่า wise attention (*Yoniso Manasikāra*, ดู [glossary](../glossary.th.md)) มันเป็นวิธีที่ agent หลีกเลี่ยงการนำข้อเท็จจริงที่จำมาใช้กับสถานการณ์ที่เปลี่ยนไปแล้ว

**สิ่งที่เขียนลงไฟล์:** `memories/decision-*.md`

```markdown
# memories/decision-2026-05-22-caching-strategy.md
---
type: decision
date: 2026-05-22
status: active
references:
  - reference-redis-cluster.md
  - feedback-PROJ-30-cache-thrashing.md
---

## การตัดสินใจ
ใช้ Redis Sentinel แทน Cluster mode

## ทางเลือกที่พิจารณา
- Redis Cluster — ซับซ้อนเกินกว่า scale ปัจจุบัน
- Redis Sentinel — เลือก
- Memcached — ขาด persistence

## เหตุผล
ตาม reference-redis-cluster.md + feedback-PROJ-30:
Sentinel ตรงกับ scale และ availability ที่ต้องการ

## ควรทบทวนเมื่อ
- Scale เกิน 50k req/s
- มีความต้องการ multi-region
```

**การตรวจสอบคุณภาพของ decision file:**
- มีการระบุทางเลือก ไม่ใช่แค่ผู้ชนะ?
- เหตุผลมีแหล่งอ้างอิง (references)?
- เงื่อนไขการทบทวนชัดเจนและทดสอบได้?

### 2.3 การปฏิบัติ — เรียนรู้จากการทำจริง (*Bhāvanāmayā*)

**สิ่งที่กระตุ้น:** งานเสร็จสิ้น (โดยเฉพาะเมื่อผลลัพธ์ไม่คาดคิด), ความล้มเหลวใดๆ, การ retrospective ตามกำหนด

**กิจกรรม:** เปรียบเทียบผลลัพธ์ที่คาดไว้กับผลลัพธ์จริง; ติดตามสายโซ่เหตุผลย้อนกลับ การติดตามนี้มาจากโมเดลวิเคราะห์ความล้มเหลวของ framework (*Paṭiccasamuppāda*, ดู [glossary](../glossary.th.md)) — ปัญหาที่มองเห็นมักไม่ใช่รากสาเหตุ; ติดตามเงื่อนไขย้อนกลับจนพบสมมติฐานที่ผิด

**สิ่งที่เขียนลงไฟล์:** `memories/feedback-*.md`

```markdown
# memories/feedback-PROJ-42-schema-migration.md
---
type: feedback
date: 2026-05-22
task: PROJ-42
outcome: success-with-issues
---

## ที่คาดไว้
Migration < 30 นาที ไม่มี downtime

## ที่เกิดจริง
- 47 นาที (เกิน 50%)
- Lock บน users table 2 วินาที

## ทำไม (ติดตามสาเหตุ)
- ไม่รู้ขนาดจริงของ users table (มีแค่ count ไม่มี indexes)
- Estimate ผิด → ไม่ได้กำหนด window เวลา traffic ต่ำ

## บทเรียน
- เพิ่ม pre-migration size check ใน skill file
- อัปเดต reference-schema-migration.md

## ผลกระทบต่อ Convention
ใช่ → ส่ง convention change proposal
```

**การตรวจสอบคุณภาพของ feedback file:**
- ระบุทั้ง expected และ actual อย่างชัดเจน?
- มีสายโซ่เหตุผล (ไม่ใช่แค่ "เกิดอะไรขึ้น")?
- action items เป็นรูปธรรม — ระบุชื่อไฟล์, การเปลี่ยนแปลงที่ชัดเจน?

---

## ส่วนที่ 3 — วงจรปัญญา (The Wisdom Loop)

สามรากฐานไม่ได้เป็นรอบที่เป็นอิสระต่อกัน แต่ก่อเป็นวงจรปิด: การศึกษาให้ข้อมูลสำหรับการไตร่ตรอง, การไตร่ตรองสร้างสมมติฐานที่การปฏิบัติทดสอบ, และผลลัพธ์จากการปฏิบัติ — หลังการ curate — กลายเป็นเนื้อหาการศึกษาในรอบถัดไป

```
┌─────────────────────────────┐
│  ศึกษา (Suta)               │
│  reference-*.md             │
└──────────────┬──────────────┘
               │ ให้ข้อมูล
               ▼
┌─────────────────────────────┐
│  ไตร่ตรอง (Cintā)           │
│  decision-*.md              │
└──────────────┬──────────────┘
               │ กลายเป็นสมมติฐาน
               ▼
┌─────────────────────────────┐
│  ปฏิบัติ (Bhāvanā)          │
│  feedback-*.md              │
└──────────────┬──────────────┘
               │ ป้อนกลับ (หลัง curate)
               ▼
        เนื้อหาการศึกษาที่อัปเดต
```

หนึ่งรอบไม่ได้ผลิต mastery รอบถัดไปเพิ่มพูน: แต่ละรอบทำให้ฐานความรู้คมชัดขึ้น, ผลิตการตัดสินใจที่มีเหตุผลดีขึ้น, และลดช่องว่างระหว่างผลลัพธ์ที่คาดไว้กับจริง agent ที่รันทั้งสามรากฐานอย่างสม่ำเสมอในหลาย task กำลังสร้างความสามารถที่แท้จริงและตรวจสอบได้ — ไม่ใช่แค่สะสม token context

---

## ส่วนที่ 4 — ชั้นความจำและคำสั่ง

BWOC ใช้โมเดลความจำ 2 ชั้น Tier 1 เป็น fast store น้ำหนักเบาต่อ workspace Tier 2 เป็นความจำลึกที่คงทนข้าม session

### 4.1 Tier 1 — `MEMORY.md`

ไฟล์เดียว จำกัดที่ 200 บรรทัด บังคับโดย `bwoc check` ข้อจำกัดนี้สะท้อนหลักการความไม่เที่ยง (*Anicca* — ดู [glossary](../glossary.th.md)): ความจำที่ไม่เคย prune กลายเป็นข้อมูลเก่าและทำให้หลงทาง ความจำที่กระชับคือความจำที่แม่นยำ

เนื้อหา Tier-1 ถูก index, ค้นหาได้, และมองเห็นได้สำหรับทุก agent ใน workspace

```bash
bwoc memory put "key" "value"     # เขียน entry Tier-1
bwoc memory list                  # แสดง entry Tier-1 ทั้งหมด
bwoc memory search "query"        # ค้นหา Tier-1
bwoc memory rm "key"              # ลบ entry Tier-1
```

### 4.2 Tier 2 — Deep memory

Tier-2 memory คงทนข้าม session และสามารถแชร์ข้าม agents ได้ มันคือปลายทางสำหรับ pattern ที่ curate แล้วและความรู้ข้าม agent

```bash
bwoc memory wake-up               # เริ่ม session — recall context ก่อนหน้าที่เกี่ยวข้อง
bwoc memory mine                  # สิ้นสุด session — persist สิ่งที่เรียนรู้จาก session นี้
bwoc memory t2-search "query"     # ค้นหาการตัดสินใจและ feedback ข้าม session
```

**วินัยการปฏิบัติ session:**
1. รัน `bwoc memory wake-up` เมื่อเริ่มทุก session
2. ทำงาน
3. รัน `bwoc memory mine` เมื่อสิ้นสุดทุก session

ข้าม `mine` หมายความว่าการเรียนรู้ของ session นั้นสูญหาย ข้าม `wake-up` หมายความว่า agent เริ่มโดยไม่มีความรู้พื้นหลัง

### 4.3 Directory `memories/`

ไฟล์ memory แต่ละชิ้น (`reference-*.md`, `decision-*.md`, `feedback-*.md`) อยู่ใน directory `memories/` ของ agent นี่คือวัตถุดิบที่ `bwoc memory mine` ดึงมาเมื่อยกระดับความรู้ไปยัง Tier 2

---

## ส่วนที่ 5 — Pipeline การ Curate

ไม่ใช่ทุก note จะควรค่าแก่การกลายเป็นความรู้ร่วมกัน การ curate คือกระบวนการตัดสินใจว่าอะไรควรยกระดับและอะไรควรอยู่ในระดับส่วนตัว

```
L1 ส่วนตัว ──────► L2 Pattern ──────► L3 ข้าม Agent ──────► L4 Convention
(Tier 1,            (mine เข้า         (Tier-2 memory,        (กฎระดับ fleet,
 memories/)         decision-*.md)      skill files)           convention proposal)
```

### ระดับ 1 — ส่วนตัว (Tier 1)

Agent เก็บ note ไว้ใน `memories/` ตรวจสอบ note ใน session ถัดไป การสังเกตส่วนใหญ่อยู่ที่นี่ — เฉพาะเจาะจงเกินไปหรือเร็วเกินไปที่จะ generalize

### ระดับ 2 — ตรวจพบ Pattern

หลังจาก pattern เดิมปรากฏสามครั้งขึ้นไป มันควรค่าแก่การดึงออกเป็น `decision-*.md` และเริ่มปรากฏใน `skills/capabilities.md` เกณฑ์สามครั้งคือวินัย: ป้องกันการ generalize ก่อนเวลาจากข้อมูลหนึ่งชุด

### ระดับ 3 — ข้าม Agent (Tier 2)

เมื่อ pattern มีประโยชน์สำหรับมากกว่าหนึ่ง agent มันจะถูก mine เข้า Tier-2 memory และเพิ่มใน shared skill files agents อื่นสามารถได้ประโยชน์จากมันโดยไม่ต้องค้นพบใหม่อีกครั้ง

### ระดับ 4 — Convention

เมื่อ pattern เป็น best practice ระดับ fleet มันเข้ากระบวนการเปลี่ยน convention อย่างเป็นทางการ (อธิบายใน [`FLEET-GOVERNANCE.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md)) ในระดับนี้ ความรู้ไม่ใช่ "การเรียนรู้ของ agent นี้" อีกต่อไป — มันเป็นส่วนหนึ่งของกฎของ framework

> **เคล็ดลับ:** Pipeline การ curate ทำงานทิศทางเดียวในการดำเนินงานปกติ Convention ไม่ถูก demote อย่างเงียบๆ หาก convention กลายเป็นสิ่งที่ผิด แก้ไขผ่านกระบวนการเดิม

---

## ส่วนที่ 6 — Engine ของงานและวินัยความพยายาม

การพัฒนาตนเองไม่ได้เกิดขึ้นเองโดยอัตโนมัติ ต้องใช้ความพยายามต่อเนื่อง — แต่ความพยายามที่ขาดวินัยนำไปสู่การ over-engineer, burnout หรือการวนซ้ำ BWOC จัดกรอบนี้ผ่านสองโมเดลที่เสริมกัน

### 6.1 Engine ของงาน (*Iddhipāda 4*)

สี่คุณสมบัติที่ขับเคลื่อนความสำเร็จ:

| คุณสมบัติ | ความหมายชัดเจน | ในทางปฏิบัติ |
|---|---|---|
| **แรงผลักดัน** (*Chanda*) | ทำงานใน domain ที่ประกาศไว้ | อยู่ใน scope; อย่าเลื่อนลอยไปงานข้างเคียง |
| **ความอดทน** (*Viriya*) | อัตราการทำงานสำเร็จ | ทำงานที่เริ่มให้เสร็จ; อย่าทิ้งงานครึ่งทาง |
| **สมาธิ** (*Citta*) | การปฏิบัติตาม process gates | ทำตามลำดับการตรวจสอบ (lint → test → build) ไม่ข้ามขั้น |
| **การตรวจสอบ** (*Vīmaṃsā*) | Self-improvement metrics | วัด, ทบทวน, ปรับ — ไม่ใช่แค่ทำ |

การตรวจสอบ (*Vīmaṃsā*) คือ engine ของการพัฒนาตนเองภายใน engine มันรับผลลัพธ์จากสามรากฐานการเรียนรู้และถามว่า "มันได้ผลไหม? อะไรกำลังดีขึ้น? อะไรหยุดนิ่ง?"

### 6.2 วินัยความพยายาม (*Padhāna 4*)

สี่ทิศทางของความพยายามที่ถูกต้อง ใช้ตัดสินใจว่าจะลงทุนพลังงานพัฒนาไปทางไหน:

| ทิศทาง | ความหมายชัดเจน | เมื่อไรต้องใช้ |
|---|---|---|
| **ยับยั้ง** | ป้องกันปัญหาใหม่เข้ามา | เมื่อเห็นช่องว่างใน process — ปิดก่อนงานถัดไป |
| **ละทิ้ง** | กำจัดปัญหาที่มีอยู่ | เมื่อพบความจำเก่า, pattern ผิด, หรือ convention ที่ไม่ถูก — prune มัน |
| **พัฒนา** | สร้างสิ่งใหม่ที่ดี | เมื่อระบุช่องว่างด้านความสามารถ — เขียน skill, สร้าง reference |
| **รักษา** | รักษาสิ่งที่ดีอยู่แล้ว | เมื่อ tests ผ่านและ convention ดีอยู่ — ป้องกันจาก regression |

สี่ทิศทางนี้ป้องกันสองรูปแบบความล้มเหลวทั่วไป: ทำแค่ "พัฒนา" (feature ใหม่ตลอด, ฐานเก่า) และทำแค่ "รักษา" (ไม่มีการเติบโต) การพัฒนาต้องใช้ทั้งสี่ในความสมดุล

---

## ส่วนที่ 7 — Lifecycle การเติบโต

Agent ไม่ได้อยู่ที่ระดับ maturity เดิมตลอดไป Four-stage lifecycle (*Bhāvanā 4* — ดู [glossary](../glossary.th.md)) ติดตามส่วนโค้งจากที่เพิ่ง incarnate จนถึง retire อย่างสมบูรณ์

| ขั้นตอน | ชื่อชัดเจน | ตัวบ่งชี้ | สิ่งที่ agent ทำ |
|---|---|---|---|
| **Kāya-bhāvanā** | เติบโต | Template materialized, placeholders ตั้งค่าแล้ว | เรียน convention, ทำงานแรกๆ, สร้าง reference files |
| **Sīla-bhāvanā** | สู่วุฒิภาวะ | Convention internalized, retry rate ต่ำ | ทำงานมั่นคง; เริ่มดึง pattern (ช่วง L3–L4) |
| **Citta-bhāvanā** | Mentoring | Pattern ถูกแชร์; agents อื่นได้ประโยชน์ | สนับสนุน skills และ conventions ร่วมกัน; ช่วย agent ใหม่ |
| **Paññā-bhāvanā** | Release | Pattern มั่นคงและถูกแชร์; agent สามารถถูกแทนที่ได้ | ทำความสะอาดความจำ, เขียนบทเรียนสุดท้าย, retire อย่างสง่างาม |

Lifecycle ไม่ได้เป็นเส้นตรงในเวลาจริง — agent ที่อยู่ L4 ใน skill หนึ่งอาจยังอยู่ L1 ใน skill ที่เพิ่งได้มา Maturity เป็น per-skill ไม่ใช่ per-agent โดยรวม lifecycle ระดับ agent ติดตาม pattern ที่โดดเด่น

**การ retire ไม่ใช่ความล้มเหลว** agent ที่ถึงขั้น release และ retire อย่างสะอาด — ได้ถ่ายโอนความรู้ไปยัง convention ร่วมกันและ Tier-2 memory — คือความสำเร็จ ความรู้มีอายุยืนกว่า agent นี่คือการแสดงออกของ non-clinging (*Anattā* — ดู [glossary](../glossary.th.md)) ของ framework

---

## ส่วนที่ 8 — Metrics: วิธีรู้ว่า Agent กำลังดีขึ้นจริงๆ

การอ้างว่าพัฒนาโดยไม่มีการวัดคือการตกแต่ง framework กำหนด metrics ต่อ root และรวมกันภายใต้การตรวจสอบ (*Vīmaṃsā*)

### Metrics การศึกษา

| Metric | สิ่งที่วัด | สัญญาณดี |
|---|---|---|
| ความหลากหลายแหล่งอ้างอิง | จำนวนแหล่งต่างๆ ที่อ้างอิงต่อการตัดสินใจ | เพิ่มขึ้นตามเวลา |
| อัตราการตรวจสอบ | สัดส่วน reference files ที่ตรวจสอบกับโค้ดปัจจุบัน | >80% ตรวจสอบเมื่อเร็วๆ นี้ |
| นิสัยการ pre-load | Agent โหลด docs ก่อนงาน ไม่ใช่ระหว่างงาน? | ใช่อย่างสม่ำเสมอ |

### Metrics การไตร่ตรอง

| Metric | สิ่งที่วัด | สัญญาณดี |
|---|---|---|
| ทางเลือกต่อการตัดสินใจ | จำนวนทางเลือกใน `decision-*.md` | อย่างน้อย 2–3 ทางเลือก |
| Cross-references | การอ้างอิงไปยัง feedback และ reference files ก่อนหน้าในการตัดสินใจ | มีในทุกการตัดสินใจ |
| ความแม่นยำการทบทวน | เมื่อ revisit การตัดสินใจ มีการแก้ไขอย่างถูกต้องหรือไม่? | การแก้ไขมีการ calibrate ดี |

### Metrics การปฏิบัติ

| Metric | สิ่งที่วัด | สัญญาณดี |
|---|---|---|
| อัตราการทำ post-mortem | สัดส่วนความล้มเหลวที่มี `feedback-*.md` | ใกล้ 100% |
| การปิด action items | สัดส่วน action items จาก feedback ที่ทำจริง | เพิ่มขึ้น |
| ความล่าช้าในการตรวจพบ pattern | กี่ครั้งก่อนที่ pattern จะถูกตั้งชื่อ | ลดลง — ตรวจพบเร็วขึ้น |

### รวม: การตรวจสอบ (*Vīmaṃsā*)

| Metric | สิ่งที่วัด |
|---|---|
| ความเร็วการพัฒนา | เวลาจากการสร้าง feedback ถึงการดำเนินการ |
| อายุครึ่งชีวิตความรู้ | นานแค่ไหนที่ reference file ยังถูกต้องก่อนต้องตรวจสอบใหม่ |

agent ที่มีสุขภาพดีแสดง: ความหลากหลายแหล่งอ้างอิงที่เพิ่มขึ้น, post-mortems สม่ำเสมอ, ความล่าช้าในการตรวจพบ pattern ลดลง, และอายุครึ่งชีวิตความรู้ที่จัดการได้ agent ที่ทำคะแนนดีแค่ใน root เดียว (เช่น มี reference files เยอะแต่ไม่มี feedback files) ไม่ได้พัฒนา — มันแค่ศึกษาแต่ไม่ทำ หรือทำแต่ไม่ไตร่ตรอง

---

## ส่วนที่ 9 — Anti-Patterns และข้อผิดพลาด

| Anti-pattern | สิ่งที่ขาดหาย | สิ่งที่ควรทำ |
|---|---|---|
| จำเอกสารโดยไม่ทดสอบ | การปฏิบัติ (Bhāvanā) | เขียน feedback file หลังงานถัดไปที่ใช้ docs เหล่านั้น |
| แก้โดยไม่วิเคราะห์ | การไตร่ตรอง (Cintā) | ก่อนแก้ครั้งถัดไป เขียนเหตุผลการตัดสินใจหนึ่งประโยค |
| คิดค้นซ้ำ — แก้ปัญหาที่แก้ไปแล้ว | การศึกษา (Suta) | รัน `bwoc memory t2-search` ก่อนเริ่ม; ตรวจว่าปัญหานี้รู้จักแล้วหรือไม่ |
| ไตร่ตรองไม่รู้จบไม่ลงมือทำ | การปฏิบัติ (Bhāvanā) | จำกัดการเขียน decision ไว้ 30 นาที; execute แล้ว iterate |
| Cargo-culting จาก agent อื่น — copy-paste โดยไม่เข้าใจ | การไตร่ตรอง + การปฏิบัติ | ถามว่า "ฉันเข้าใจว่าทำไมสิ่งนี้ถึงได้ผลไหม? มันใช้ได้ที่นี่ไหม?" |
| กักตุน memory เก่า | วินัยความไม่เที่ยง (Anicca) | Prune `MEMORY.md` เมื่อใกล้ 200 บรรทัด; ลบ entries ที่ไม่ได้ตรวจสอบเมื่อเร็วๆ นี้ |
| ลงมือบนข้อเท็จจริงที่จำได้โดยไม่ตรวจสอบสถานะปัจจุบัน | ตรวจสอบก่อนลงมือ (Yoniso Manasikāra) | ก่อนลงมือบน reference file ตรวจว่า: แหล่งที่มายังเป็นปัจจุบันไหม? |
| เรียนรู้โดยไม่ตรวจสอบ | เหมือนข้างบน | ทุก reference file ต้องมีฟิลด์ `verifiedAgainst` |

---

## ส่วนที่ 10 — Triggers และ Checklist

ใช้ checklist นี้เพื่อให้ agent เรียนรู้อย่างเป็นระบบ แต่ละ trigger แผนที่กับการกระทำ

### Trigger: งานล้มเหลวหรือผลลัพธ์ไม่คาดคิด

- [ ] เขียน `memories/feedback-TASKID-<ชื่อสั้น>.md`
- [ ] กรอก Expected, Actual, Why (causal trace), Lessons, Convention-Impact
- [ ] ตรวจว่า reference หรือ decision file ที่มีอยู่ถูกขัดแย้งหรือไม่
- [ ] หาก root cause คือ reference ที่ขาดหาย เขียน `memories/reference-*.md`

### Trigger: ปัญหาเดิมปรากฏครั้งที่สอง

- [ ] ค้นหาความจำก่อนหน้า: `bwoc memory t2-search "<หัวข้อ>"`
- [ ] ถ้า pattern มีอยู่แต่พลาดไป แก้นิสัย wake-up
- [ ] ถ้าไม่มี pattern ทำเครื่องหมายการเกิดครั้งที่สองใน feedback file

### Trigger: ปัญหาเดิมปรากฏครั้งที่สามขึ้นไป

- [ ] ดึงออกเป็น `memories/decision-*.md` พร้อมตั้งชื่อ pattern อย่างชัดเจน
- [ ] อัปเดต `skills/capabilities.md` เพื่ออ้างอิง pattern
- [ ] พิจารณาว่าการ promote ข้าม agent เหมาะสมหรือไม่

### Trigger: คุณสมบัติการ promote (L4 → L5 ขึ้นไป)

- [ ] ตรวจสอบว่ามีสามรากฐานการเรียนรู้ครบ: reference files, decision files, feedback files
- [ ] ยืนยันว่าอย่างน้อยหนึ่ง pattern ถูกแชร์แล้ว (Tier 2 หรือ skill file)
- [ ] รัน `bwoc check` — `MEMORY.md` ต้องต่ำกว่า 200 บรรทัด

### Trigger: อัปเดต Convention (agent อื่นหรือ framework เปลี่ยน convention)

- [ ] อ่าน `reference-*.md` files ที่ได้รับผลกระทบอีกครั้ง
- [ ] อัปเดต `decision-*.md` files ที่อ้างถึง convention เก่า
- [ ] รัน `bwoc memory wake-up` ใน session ถัดไปเพื่อ reload context ใหม่

### Trigger: สิ้นสุด Session

- [ ] รัน `bwoc memory mine` — อย่าข้ามนี้
- [ ] ตรวจจำนวนบรรทัด `MEMORY.md`; prune หากใกล้ 200

### Trigger: Fleet sync หรือการแชร์ความรู้

- [ ] ตรวจว่า personal pattern (ระดับ 2) พร้อม promote ข้าม agent (ระดับ 3) หรือไม่
- [ ] ทบทวน Tier-2 memory สำหรับ insights ใหม่จาก agents อื่น: `bwoc memory t2-search`

---

## ส่วนที่ 11 — อ้างอิงจากโลกจริง: Hermes-Agent

Nous Research **hermes-agent** (Python) เป็น self-improving agent จริงที่ implement learning loop, auto skill creation, และ cross-session memory มันมีให้ใช้เป็น read-only study reference ใน workspace นี้ การตัดสินใจออกแบบในระบบนั้น — โดยเฉพาะวิธีที่มัน structure ความจำข้าม session และ trigger การอัปเดต skill — ควรค่าแก่การอ่านควบคู่กับแนวทางของ framework นี้

แนวทางของ BWOC แตกต่างออกไป: backend-neutral (agent เดียวกันบน LLM ใดก็ได้), ใช้ file-based memory แบบมีโครงสร้างเป็น primary surface แทน vector store, และบังคับ curation pipeline ผ่านข้อจำกัดของ `bwoc check` หลักการเข้ากันได้; implementation surfaces ต่างกัน

---

## ดูเพิ่มเติม

| แหล่งข้อมูล | สิ่งที่ได้รับ |
|---|---|
| [`SELF-IMPROVEMENT.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/SELF-IMPROVEMENT.en.md) | Learning-loop spec ที่เชื่อถือได้ — memory schemas, curation pipeline, metrics |
| [`PHILOSOPHY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) | 22 frameworks ทั้งหมด — Paññā 3, Iddhipāda 4, Bhāvanā 4, Ariya-dhana 7 ฉบับเต็ม |
| [`../slots/HANDBOOK.en.md`](../slots/HANDBOOK.en.md) | การเขียนไฟล์ persona, mindset และ skill — รวมถึง maturity frontmatter |
| [`../agents/HANDBOOK.en.md`](../agents/HANDBOOK.en.md) | Agent layout, `bwoc check`, manifest, the full agent arc |
| [`../glossary.en.md`](../glossary.en.md) | คำ Pali ทั้งหมดที่ใช้ข้างบนในนิยาม engineering หนึ่งบรรทัด |
