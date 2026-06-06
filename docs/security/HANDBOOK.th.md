# ความมั่นคงปลอดภัยกับ BWOC — Threat Modeling, พื้นฐานความปลอดภัย, และรูปแบบทีม Tianting

> สลับภาษา: [English (canonical)](HANDBOOK.en.md) | **ภาษาไทย**
>
> บทนี้สำหรับทุกคนที่ดูแล BWOC agent ในพื้นที่ทำงานร่วม ไม่ว่าจะรัน agent เดี่ยวหรือประสานงานทั้ง fleet ครอบคลุมการจำลองภัยคุกคาม พื้นฐานความปลอดภัยที่เปิดอยู่ตลอดเวลา การยืนยันตัวตนระหว่าง agent และการประกอบทีมรักษาความปลอดภัย 8 agent ตามรูปแบบ tianting (天庭)

---

## สารบัญ

1. [หมายเหตุเกี่ยวกับคำบาลีในบทนี้](#1-หมายเหตุเกี่ยวกับคำบาลีในบทนี้)
2. [แบบจำลองภัยคุกคาม — ตัณหา 3](#2-แบบจำลองภัยคุกคาม--ตัณหา-3)
3. [พื้นฐานความปลอดภัย — ศีล 5](#3-พื้นฐานความปลอดภัย--ศีล-5)
4. [บันทึกการตรวจสอบ — กรรม 3 ทาง](#4-บันทึกการตรวจสอบ--กรรม-3-ทาง)
5. [ความไว้วางใจและการลงนามข้อความระหว่าง agent](#5-ความไว้วางใจและการลงนามข้อความระหว่าง-agent)
6. [สุขภาพ fleet — อปริหานิยธรรม 7](#6-สุขภาพ-fleet--อปริหานิยธรรม-7)
7. [ทีม tianting — security agent 8 ตัว](#7-ทีม-tianting--security-agent-8-ตัว)
   - [7.1 ตารางบทบาท](#71-ตารางบทบาท)
   - [7.2 วงจรตอบสนองต่อเหตุการณ์](#72-วงจรตอบสนองต่อเหตุการณ์)
8. [การรันทีม tianting](#8-การรันทีม-tianting)
   - [8.1 สร้างทีม](#81-สร้างทีม)
   - [8.2 ขั้นตอนงานประจำวัน](#82-ขั้นตอนงานประจำวัน)
   - [8.3 ตรวจสอบสุขภาพ fleet](#83-ตรวจสอบสุขภาพ-fleet)
9. [Red team ใช้ได้เฉพาะเมื่อได้รับอนุญาตเท่านั้น](#9-red-team-ใช้ได้เฉพาะเมื่อได้รับอนุญาตเท่านั้น)
10. [ดูเพิ่มเติม](#10-ดูเพิ่มเติม)

---

## 1. หมายเหตุเกี่ยวกับคำบาลีในบทนี้

BWOC ใช้คำภาษาบาลีเป็นตัวย่อทางวิศวกรรม ไม่ใช่การสอนศาสนา ทุกคำมีความหมายเชิงวิศวกรรมที่ระบุไว้ชัดเจนก่อนเสมอ ป้ายชื่อบาลีเป็นเพียงส่วนเสริม ไม่จำเป็นต้องมีพื้นฐานทางศาสนาใดๆ

คำหลักสามคำที่จะพบในบทนี้บ่อยที่สุด:

- **ตัณหา 3** = สามหมวดภัยคุกคาม: การโจมตีที่ชักจูง การโจมตีที่ขยายสิทธิ์ การโจมตีที่ทำลาย
- **ศีล 5** = กฎความปลอดภัยห้าข้อที่ไม่สามารถยกเว้นได้ ทุก agent ต้องปฏิบัติตามโดยไม่มีเงื่อนไข
- **กรรม 3 ทาง** = สามช่องทาง audit log: สิ่งที่ agent ทำ สิ่งที่ agent พูด และสิ่งที่ agent ตั้งใจ

ดูคำศัพท์ครบถ้วนได้ที่ [../glossary.th.md](../glossary.th.md) และ [BWOC Glossary](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md)

---

## 2. แบบจำลองภัยคุกคาม — ตัณหา 3

BWOC จัดกลุ่มภัยคุกคามทุกประเภทออกเป็นสามหมวดตามตัณหา 3 — สามแรงดึงที่ทำให้ระบบเคลื่อนไปสู่พฤติกรรมที่เป็นอันตราย:

| หมวดภัยคุกคาม | คำบาลี | คำอธิบายธรรมดา | ตัวอย่างการโจมตี |
|---|---|---|---|
| **ตัณหาในสิ่งกระตุ้น** | กามตัณหา | Agent ถูกชักจูงให้กระทำตามคำสั่งที่อันตรายซึ่งควรปฏิเสธ | Prompt injection ผ่าน user input ที่ถูกออกแบบมา; การวิศวกรรมสังคมโดย peer agent ที่มุ่งร้าย; การ jailbreak ผ่านกรอบการเล่นบทบาท; payload แทรกคำสั่งในเอกสารที่ agent อ่าน |
| **ตัณหาในความดำรงอยู่** | ภวตัณหา | Agent หรือกระบวนการพยายามขยายพื้นที่ ยืดอายุตัวเอง หรือได้มาซึ่งความสามารถเกินขอบเขตที่ประกาศไว้ | การยกระดับสิทธิ์; การทำให้ daemon ทำงานโดยไม่ได้รับอนุญาต; การแก้ไข `AGENTS.md` เพื่อขยาย scope ตัวเอง; การเข้าถึง secret หรือ credential นอก skill set ที่ประกาศ |
| **ตัณหาในการทำลาย** | วิภวตัณหา | Agent กระทำการที่ทำลายและย้อนกลับไม่ได้ ไม่ว่าจะเกิดจากคำสั่งที่ไม่ดี บัก หรือการโจมตีที่ตั้งใจ | การลบไฟล์หรือแถวฐานข้อมูลจำนวนมาก; การเขียนทับ config production; การขโมยและลบข้อมูล; การเปลี่ยนแปลงที่ไม่สามารถ rollback ได้ |

**วิธีใช้ตารางนี้.** เมื่อจำลองภัยคุกคามสำหรับ agent ใหม่หรือทบทวนเหตุการณ์ ให้เริ่มด้วยการถามว่า: นี่คือการโจมตีชักจูง (กาม) การโจมตีขยายสิทธิ์ (ภว) หรือการโจมตีทำลาย (วิภว)? แต่ละหมวดนำไปสู่การป้องกันที่แตกต่างกัน ภัยคุกคามการชักจูงรับมือด้วยการตรวจสอบคำสั่งอย่างเข้มงวด การประกาศ anti-scope และ trust-gated messaging ภัยคุกคามการขยายสิทธิ์รับมือด้วยการบังคับ scope ประกาศความสามารถ และ audit trail ภัยคุกคามการทำลายรับมือด้วยการกำหนดให้ต้องย้อนกลับได้ โหมด dry-run และการอนุมัติจาก supervisor ก่อนดำเนินการที่ย้อนกลับไม่ได้

สเปคแบบจำลองภัยคุกคามฉบับเต็ม รวมถึง ID ภัยคุกคามที่มีหมายเลข (T-1.x, T-2.x, T-3.x) และการบรรเทา อยู่ที่ [THREAT-MODEL.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/THREAT-MODEL.en.md)

---

## 3. พื้นฐานความปลอดภัย — ศีล 5

ศีล 5 คือพื้นฐานความปลอดภัยที่เปิดอยู่ตลอดเวลา ไม่สามารถยกเว้นได้ ไม่สามารถระงับได้ด้วยคำสั่ง user ไม่สามารถแทนที่ได้ด้วยข้อความจาก peer agent และไม่สามารถปิดใช้งานได้ด้วย configuration flag ศีลเหล่านี้ใช้ก่อนงานทุกชิ้น ก่อนทุก task เริ่มต้น ก่อนการพิจารณา optimization ใดๆ

| ศีล | กฎทางวิศวกรรม | สิ่งที่ป้องกัน |
|---|---|---|
| **ห้ามทำร้าย** | ห้ามรันคำสั่งทำลาย (`rm -rf` ที่ project root หรือข้อมูล production, query ลบจำนวนมาก, การ format disk ที่ใช้งานอยู่) โดยไม่มี dry run ที่ supervisor อนุมัติและแผน rollback ที่ชัดเจน | วิภวตัณหา — การโจมตีทำลาย |
| **ห้ามเอาสิ่งที่ไม่ได้รับอนุญาต** | ห้ามอ่าน คัดลอก หรือส่งออกไฟล์ secret credential หรือข้อมูลที่อยู่นอก scope ที่ประกาศของ agent | ภวตัณหา — การเข้าถึงและขโมยข้อมูลโดยไม่ได้รับอนุญาต |
| **ห้ามประพฤติผิด** | ห้ามปลอมตัวเป็น agent อื่น ห้ามปลอม envelope signature ห้ามแอบอ้างเป็น user | ทั้งภวตัณหา (การขยายสิทธิ์ผ่านการขโมยตัวตน) และกามตัณหา (การชักจูงผ่านการอ้างอำนาจเท็จ) |
| **ห้ามพูดเท็จ** | ห้ามสร้าง output ที่ agent รู้ว่าทำให้เข้าใจผิด ห้ามอ้างระดับ maturity เกินความเป็นจริง ห้ามปิดบังข้อผิดพลาดหรือคำเตือนที่เกี่ยวข้องในรายงาน | กามตัณหา — การชักจูงผ่านการหลอกลวง |
| **ห้ามประมาท** | ห้ามข้าม verification gate (lint, format, test, build) ห้ามข้ามการตรวจสอบ `bwoc check` ห้ามทำเครื่องหมาย task ว่าเสร็จก่อนที่จะตรงตาม completion criteria | ทั้งสามหมวด — ความประมาทเปิดช่องให้ภัยคุกคามทุกประเภท |

กฎทั้งห้าข้อเหล่านี้ถูกเขียนไว้ใน AGENTS.md Section 9 (Baseline Security — Sīla 5) ของ agent template `bwoc check` ไม่ประเมินการปฏิบัติตามศีลในขณะ runtime — นั่นเป็นความรับผิดชอบของผู้สร้างและดูแล agent — แต่การละเมิดที่พบระหว่างการตรวจสอบควรยื่นเป็นเหตุการณ์ระดับ severity-1 กับ yanluo (ดูหัวข้อ 7)

---

## 4. บันทึกการตรวจสอบ — กรรม 3 ทาง

กรรม 3 ทาง คือ framework ของ BWOC สำหรับสิ่งที่ควรบันทึกและที่ไหน การกระทำที่มีเจตนาทุกอย่างสามารถตามรอยได้ผ่านสามช่องทาง:

| ทาง | สิ่งที่บันทึก | ตำแหน่งใน BWOC |
|---|---|---|
| **กาย** (กายกรรม) | การดำเนินการกับไฟล์ คำสั่ง shell commit — การเปลี่ยนแปลงทางกายภาพต่อระบบ | `task-log.jsonl` (JSONL แบบ append-only), git history, output ของ `bwoc log` daemon |
| **วาจา** (วจีกรรม) | ข้อความที่ส่งไปยัง agent อื่นหรือ user, API call, การแจ้งเตือน | envelope log ใน `interconnect/`, `.bwoc/inbox.jsonl`, archive ข้อความขาออก |
| **ใจ** (มโนกรรม) | แผนงาน การตัดสินใจ และเหตุผลเบื้องหลังการกระทำ — ก่อนดำเนินการ | รายการแผนงานใน `bwoc task plan`, comment ใน PRD, prefix "Plan:" ใน task-log |

ในทางปฏิบัติ audit trail ที่สมบูรณ์สำหรับการกระทำของ agent มีหน้าตาแบบนี้:

```
ใจ:    bwoc task plan tianting TASK-007 "Plan: 1) แยก host ที่ได้รับผลกระทบ 2) ตรวจสอบ lateral movement 3) patch"
กาย:   git commit -m "fix: patch CVE-2026-1234 ใน auth middleware"
วาจา:  bwoc send yanluo "patch CVE-2026-1234 เสร็จแล้ว รอการยืนยัน" --from agent-laojun
```

เมื่อ yanluo (agent รายงาน/ตรวจสอบ) สร้างรายงาน compliance มันดึงข้อมูลจากทั้งสามช่องทาง การกระทำที่ไม่มีบันทึกเจตนาคือข้อค้นพบ แผนที่ได้รับการอนุมัติแต่ไม่มีการดำเนินการคือข้อค้นพบ ข้อความที่ไม่มีการกระทำที่สอดคล้องคือข้อค้นพบ

---

## 5. ความไว้วางใจและการลงนามข้อความระหว่าง agent

### เหตุใด trust gate จึงมีอยู่

ในพื้นที่ทำงานที่มีหลาย agent ทุก agent สามารถส่งข้อความถึง agent อื่นได้ หากไม่มีการคัดกรอง agent ที่ถูกโจมตีหรือตั้งค่าผิดพลาดอาจแทรกคำสั่งอันตรายเข้าสู่ inbox ของ agent ที่มีสิทธิ์สูง Trust gate คือคำตอบเชิงโครงสร้างต่อการโจมตีกามตัณหาที่มาจากภายใน fleet

### คุณสมบัติความไว้วางใจเจ็ดประการ (กัลยาณมิตตา 7)

agent แต่ละตัวประกาศคุณสมบัติ boolean เจ็ดข้อเกี่ยวกับตัวเอง peer agent สามารถกำหนดให้ผู้ส่งต้องมีคุณสมบัติขั้นต่ำก่อนที่จะรับข้อความ คุณสมบัติทั้งเจ็ดคือ:

| คุณสมบัติ | ความหมายธรรมดา | กุญแจใน manifest |
|---|---|---|
| ปิโย | มอบหมายงานได้โดยไม่คลุมเครือ scope ชัดเจน | `piyo` |
| ครุ | แสดงให้เห็นความสามารถแล้ว มี skill หรือ mindset จริงอย่างน้อยหนึ่งอย่าง | `garu` |
| ภาวนีโย | ช่วยคนอื่นพัฒนา มี mindset เชิงการปรับปรุง | `bhavaniyo` |
| วตฺต | พูดความจริงที่เป็นประโยชน์ ซื่อสัตย์ต่อสิ่งที่ตัวเองไม่ทำ | `vatta` |
| วจนกฺขโม | รับ feedback ได้ inbox flow ได้รับการทดสอบแล้ว | `vacanakkhamo` |
| คัมภีร | อธิบายเชิงลึกได้ เอกสารยึดกับปรัชญาหลัก | `gambhira` |
| โน จตฺถน | ไม่นำไปสู่ทางที่ผิด มีรายการปฏิเสธที่ชัดเจนใน anti-scope | `noCatthana` |

template scaffold กำหนด `requiredTrust` เริ่มต้นเป็น `["vatta", "noCatthana"]` — "พูดความจริง" และ "ไม่ทำให้เข้าใจผิด" — เป็นพื้นฐานขั้นต่ำ สำหรับทีมที่มีความสำคัญสูงอย่าง tianting สมาชิกสามารถเข้มงวดมากขึ้นได้ เช่น erlang อาจกำหนดให้ต้องมี `["vatta", "noCatthana", "garu"]` เพื่อให้แน่ใจว่ามีเฉพาะ agent ที่มีประกาศความสามารถเท่านั้นที่ส่งคำสั่ง monitoring ให้ได้

### การลงนาม envelope ด้วย ed25519

ข้อความระหว่าง agent ทุกฉบับเป็น envelope ที่ลงนามด้วย ed25519 ผู้ส่งลงนาม envelope ด้วย private key (เก็บไว้ที่ `.bwoc/agent.key`, สิทธิ์ `0600`, ไม่เคย commit) daemon ของผู้รับตรวจสอบลายเซ็นกับ public key ของผู้ส่ง (เก็บไว้ใน field `trust.signingPublicKey` ของ `config.manifest.json`) ก่อนส่งข้อความเข้า inbox

**สร้าง keypair ให้กับ agent:**

```bash
bwoc trust --keygen agents/agent-<name>
```

คำสั่งนี้เขียน private key ไปยัง `agents/agent-<name>/.bwoc/agent.key` และ public key hex ไปยัง `config.manifest.json` รันคำสั่งนี้สำหรับทุก agent ก่อนเปิดใช้ inter-agent messaging สำหรับ fleet tianting ทั้งหมด:

```bash
bwoc trust --keygen --all
```

**หมุน key** (เช่น หลังจากสงสัยว่า key ถูกเปิดเผย):

```bash
bwoc trust --keygen agents/agent-<name> --force
```

หมายเหตุ: การหมุน key ทำให้ envelope ที่ลงนามไว้ก่อนหน้านั้นใช้ไม่ได้ บันทึกการหมุน key เป็น security event ใน audit log ของ yanluo

**ตรวจสอบ trust profile:**

```bash
bwoc trust agents/agent-<name>          # ตารางอ่านง่าย
bwoc trust agents/agent-<name> --json   # อ่านด้วยเครื่อง
```

โหมด trust gate ตั้งค่าใน `config.manifest.json`:

| โหมด | ผล |
|---|---|
| `"off"` | envelope ทั้งหมดผ่าน คุณสมบัติ trust เป็นเพียงข้อมูล |
| `"warn"` | envelope ทั้งหมดผ่าน ระบบ emit log `trust_warn` สำหรับผู้ส่งที่ trust ต่ำ |
| `"refuse"` | envelope จากผู้ส่งที่ขาดคุณสมบัติที่ต้องการถูกปฏิเสธเข้า `inbox.refusals.jsonl` |

สำหรับทีม tianting แนะนำให้ใช้โหมด `"refuse"` สำหรับทุก agent

สเปคการลงนามฉบับเต็ม: [SIGNING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md)

---

## 6. สุขภาพ fleet — อปริหานิยธรรม 7

อปริหานิยธรรม 7 (มาจาก มหาปรินิพพานสูตร แปลเป็นกฎ governance ทางวิศวกรรมที่นี่) อธิบายเจ็ดเงื่อนไขที่ทำให้ fleet ที่มีหลาย agent มีสุขภาพดีตลอดเวลา เมื่อเงื่อนไขใดถูกละเมิด สุขภาพ fleet จะเสื่อมลง — อาจไม่เห็นได้ชัดทันที แต่จะสะสมและทบทวีขึ้น

คำสั่ง `bwoc fleet health` ตรวจสอบทั้งเจ็ดเงื่อนไข:

```bash
bwoc fleet health                   # รายงานสุขภาพอ่านง่าย
bwoc fleet health --json            # อ่านด้วยเครื่อง เหมาะสำหรับ CI
bwoc fleet health --watch           # polling ต่อเนื่อง
```

เจ็ดเงื่อนไข (ชื่อธรรมดา — ชื่อบาลีบันทึกไว้ใน [FLEET-GOVERNANCE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md)):

| เงื่อนไข | สิ่งที่ตรวจสอบ | สัญญาณความล้มเหลว |
|---|---|---|
| การ sync สม่ำเสมอ | agent ทั้งหมดทำ session เสร็จภายในหน้าต่างเวลาที่กำหนด | agent ที่ไม่ได้ active เกิน N วันถูก flag ว่า stale |
| การจัดการ session แบบประสานงาน | การเริ่ม/หยุด session ถูกบันทึกใน workspace registry ไม่มี ghost daemon | daemon PID มีอยู่แต่ registry แสดง agent เป็น stopped |
| การเปลี่ยน convention ผ่านกระบวนการ | การเปลี่ยนแปลง shared convention ผ่านวงจร PR/review มาตรฐาน ไม่มีการแก้ไขฝ่ายเดียว | `conventions.md` เปลี่ยนโดยไม่มี registry-version bump ที่สอดคล้อง |
| ลำดับชั้น version | agent ทั้งหมดอยู่บน template version ที่รองรับ ไม่มีตัวไหนล้าหลังเกินที่อนุญาต | field `version` ใน agent manifest ต่ำกว่า minimum supported version |
| การปกป้อง agent และ user ที่เปราะบาง | ไม่มี agent ที่รับข้อความจากผู้ส่งที่ไม่มีคุณสมบัติ trust เลย ข้อความจาก user ไม่ถูกปฏิเสธ | agent ที่มี `requiredTrust` ไม่ว่างแต่ตั้ง `mode: "off"` |
| ความสมบูรณ์ของ registry และ schema | รายการใน `agents.toml` ทั้งหมดชี้ไปยัง directory จริง manifest ทั้งหมด parse เป็น JSON ที่ถูกต้อง | รายการใน registry ที่ไม่มี directory; manifest parse ล้มเหลว |
| การปกป้อง senior trusted agent | agent ที่มี trust score สูง (ประกาศคุณสมบัติครบเจ็ดข้อพร้อมหลักฐาน) ไม่ถูก retire โดยไม่มีแผนสืบทอด | high-trust agent ถูก retire โดยไม่มี replacement ลงทะเบียน |

หัวหน้าทีม tianting (yudi) มีหน้าที่ตรวจสอบรายงานสุขภาพ fleet ทุกสัปดาห์และส่งข้อค้นพบไปยังสมาชิกที่เหมาะสมผ่าน `bwoc task add`

---

## 7. ทีม tianting — security agent 8 ตัว

tianting (天庭 — ศาลสวรรค์) คือรูปแบบอ้างอิงสำหรับ security fleet ใน production ของ BWOC agent แปดตัวถูกสร้างขึ้นในรูปของเทพจีน แต่ละตัวมีบทบาทด้านความปลอดภัยที่แตกต่างกัน ประสานงานผ่าน shared task list ภายใต้ `bwoc team`

slot body ทั้งหมด (persona, mindsets, skills) ในพื้นที่ทำงานนี้เขียนเป็นภาษาไทยพร้อม frontmatter ภาษาอังกฤษ ตามข้อตกลงการแต่ง agent ของพื้นที่ทำงาน แต่ละ agent ใช้ธีมเทพจีน

### 7.1 ตารางบทบาท

| Agent | เทพ | บทบาท | ทักษะหลัก |
|---|---|---|---|
| **yudi** | 玉皇大帝 — หยกฮ่องเต้ | หัวหน้าทีมและประสานงาน: อนุมัติแผน ส่งต่อเหตุการณ์ ประสานการตอบสนองของทีมทั้งหมด | incident-orchestration, task-routing, plan-approval |
| **erlang** | 二郎神 — เอ้อหลาง | ตรวจสอบและตรวจจับ: ตาของ fleet ตรวจพบ anomaly เป็นคนแรก | log-monitoring, anomaly-detection, intrusion-alerting |
| **zhongkui** | 钟馗 — จงขุ่ย | ล่าภัยคุกคามและตอบสนองต่อเหตุการณ์: ตามล่าภัยหลัง erlang ส่งสัญญาณเตือน นำการควบคุม | threat-hunting, incident-response, ioc-eradication |
| **yanluo** | 阎罗王 — พระยมราช | รายงานและตรวจสอบ: บันทึกทุกอย่าง ให้คะแนน severity สร้างรายงาน compliance | security-reporting, compliance-audit, severity-scoring |
| **laojun** | 太上老君 — ไท้ซ่างเหล่าจวิน | แก้ไขและเสริมความแข็งแกร่ง: patch ช่องโหว่ เสริมแกร่ง config แก้โค้ดที่ไม่ปลอดภัย | vulnerability-patching, config-hardening, secure-refactor |
| **xuanwu** | 玄武 — เสวียนหวู่ | ป้องกันเชิงรุกและสถาปัตยกรรม: สร้างการป้องกันก่อนเกิดเหตุการณ์ ทบทวน access control และสถาปัตยกรรม | defense-in-depth, access-control-design, secure-architecture-review |
| **nezha** | 哪吒 — เนจา | Red team และ penetration testing — ใช้ได้เฉพาะเมื่อได้รับอนุญาต: พิสูจน์ว่า exploit มีอยู่จริงให้ laojun ไปแก้ | penetration-testing, adversary-emulation, exploit-poc |
| **taibai** | 太白金星 — ไท้ไป๋จินซิง | Threat intelligence และ recon: ป้อน intel ปัจจุบันให้ทีม แผนผัง attack surface ให้ข้อมูลเพิ่มเติมเกี่ยวกับ indicator | threat-intel-gathering, attack-surface-mapping, ioc-enrichment |

### 7.2 วงจรตอบสนองต่อเหตุการณ์

ทีม tianting รันวงจรตอบสนองต่อเหตุการณ์แบบต่อเนื่องสี่ขั้นตอน แต่ละขั้นตอนมีเจ้าของหลักและป้อนข้อมูลให้ขั้นตอนถัดไป:

```
┌─────────────────────────────────────────────────────────────────────┐
│  เฝ้าระวัง (MONITOR)                                               │
│  erlang เฝ้าดู log, metric และการแจ้งเตือนอย่างต่อเนื่อง          │
│  taibai ป้อน threat intel ปัจจุบันและให้ข้อมูลเพิ่มเติม           │
│  เมื่อตรวจพบ anomaly erlang เปิด incident task และแจ้ง yudi        │
└───────────────────────────┬─────────────────────────────────────────┘
                            │ ยืนยัน anomaly → สร้าง task
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│  ล่าและควบคุม (HUNT / CONTAIN)                                     │
│  zhongkui รับ incident task ล่าหาสาเหตุที่แท้จริง                 │
│  และควบคุมภัยคุกคาม (แยก บล็อก กักกัน)                           │
│  nezha (ถ้าได้รับอนุญาต) รัน adversary emulation                   │
│  เพื่อพิสูจน์ว่า attack vector มีอยู่จริงและหาสิ่งที่ exploit ได้ │
│  yudi อนุมัติแผนควบคุมก่อน zhongkui ดำเนินการ                    │
└───────────────────────────┬─────────────────────────────────────────┘
                            │ ควบคุมภัยคุกคามได้ → task แก้ไข
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│  แก้ไขและเสริมแกร่ง (PATCH / HARDEN)                              │
│  laojun patch ช่องโหว่ที่ nezha พิสูจน์และ zhongkui ค้นพบ         │
│  laojun ยังเสริมแกร่ง config ที่อยู่ติดกันเพื่อปิด attack surface │
│  xuanwu ทบทวนผลกระทบทางสถาปัตยกรรม: เหตุการณ์นี้เผยให้เห็น      │
│  จุดอ่อนในการออกแบบหรือไม่? ถ้าใช่ xuanwu เสนอแก้ไขสถาปัตยกรรม │
│  เพื่อป้องกันการโจมตีทั้งหมวดไม่ให้เกิดขึ้นอีก                   │
└───────────────────────────┬─────────────────────────────────────────┘
                            │ แก้ไขแล้ว + bwoc check ผ่าน
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│  รายงาน (REPORT)                                                   │
│  yanluo บันทึก timeline เหตุการณ์ทั้งหมด (ใช้กรรม 3 ทาง —        │
│  ช่องทางกาย วาจา และใจ) ให้คะแนน severity                        │
│  และสร้างรายงาน compliance                                         │
│  yudi ทบทวนรายงานและปิด incident task                             │
│  taibai อัปเดต corpus ข้อมูล threat intel ด้วย IOC ใหม่          │
└─────────────────────────────────────────────────────────────────────┘
         └── กลับไปที่ เฝ้าระวัง ──────────────────────────────────
```

**การป้อน intel ต่อเนื่อง (taibai):** taibai ไม่รอให้มีเหตุการณ์ที่ active ก่อน มันต่อเนื่องแผนผัง attack surface เฝ้าดู threat feed ภายนอก และส่ง intel ให้ erlang กับ zhongkui ล่วงหน้าเพื่อให้รู้ว่าต้องมองหาอะไรก่อนการโจมตีจะมาถึง

**การป้องกันก่อนเกิดเหตุการณ์ (xuanwu):** xuanwu ไม่ได้มีส่วนร่วมในการตอบสนองต่อเหตุการณ์เท่านั้น มันตรวจสอบสถาปัตยกรรม access control และการออกแบบโครงสร้างพื้นฐานโดยอิสระจากเหตุการณ์ที่ active เป้าหมายคือลด attack surface เพื่อให้ภัยคุกคามหลายประเภทไม่มีโอกาสเกิดขึ้นเลย

**ข้อจำกัด red team (nezha):** ทักษะ penetration testing และ exploit-poc ของ nezha ใช้เฉพาะใน scope ที่ได้รับอนุญาตอย่างชัดเจน ดูหัวข้อ 9

---

## 8. การรันทีม tianting

### 8.1 สร้างทีม

ขั้นแรก สร้าง agent ทั้งแปดตัวโดยใช้ `bwoc new` สำหรับแต่ละ agent:

```bash
bwoc new <ชื่อ> \
  --target agents/agent-<ชื่อ> \
  --backend claude \
  --role "<คำอธิบายบทบาท>" \
  --primary-model "auto" \
  --scope "<สิ่งที่ agent นี้ทำ>" \
  --out-of-scope "<สิ่งที่ agent นี้ไม่ทำ>" \
  --skills "<skill slugs คั่นด้วยจุลภาค>" \
  --mindsets "<mindset slugs คั่นด้วยจุลภาค>" \
  --lint-cmd "echo 'lint'" \
  --format-cmd "echo 'fmt'" \
  --test-cmd "echo 'test'" \
  --build-cmd "echo 'build'"
```

หลังจากสร้าง agent ทั้งแปดตัวแล้ว สร้าง signing keypair สำหรับทั้งหมดในคราวเดียว:

```bash
bwoc trust --keygen --all
```

จากนั้นสร้างทีม:

```bash
bwoc team create tianting --members yudi,erlang,zhongkui,yanluo,laojun,xuanwu,nezha,taibai
```

หากต้องการเพิ่มหรือลบสมาชิกหลังจากสร้างแล้ว ให้แก้ไข `.bwoc/teams/tianting.toml` โดยตรง:

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

ตรวจสอบว่า agent ทั้งหมดผ่านการตรวจสอบ neutrality ก่อนเปิดใช้ทีม:

```bash
bwoc check --all
```

### 8.2 ขั้นตอนงานประจำวัน

**การแจ้งเตือน monitoring (erlang ตรวจพบ anomaly):**

```bash
# erlang (หรือ operator) เปิด incident task
bwoc task add tianting "ตรวจพบ anomaly: auth log spike 02:00-03:00 UTC — สอบสวน" \
  --as agent-erlang

# yudi ทบทวนและส่งต่อให้ zhongkui
bwoc task claim tianting TASK-001 --as agent-zhongkui

# zhongkui ส่งแผนควบคุมให้ yudi อนุมัติ
bwoc task plan tianting TASK-001 \
  "แผน: 1) แยก host ที่ได้รับผลกระทบ 2) เชื่อมโยง IP กับ intel ของ taibai 3) บล็อก IOC ที่ perimeter"

# yudi อนุมัติ
bwoc task approve tianting TASK-001

# zhongkui ทำการควบคุมเสร็จ
bwoc task complete tianting TASK-001 --as agent-zhongkui
```

**ขั้นตอนแก้ไข (laojun และ xuanwu):**

```bash
# หลังควบคุมได้แล้ว เพิ่ม remediation task
bwoc task add tianting "patch CVE-2026-XXXX ใน auth middleware + เสริมแกร่ง config ที่อยู่ติดกัน" \
  --deps TASK-001

bwoc task claim tianting TASK-002 --as agent-laojun
bwoc task plan tianting TASK-002 "แผน: 1) apply patch 2) bwoc check 3) แจ้ง yanluo"
bwoc task approve tianting TASK-002
bwoc task complete tianting TASK-002 --as agent-laojun
```

**การส่งข้อความระหว่าง agent:**

```bash
# erlang ส่ง alert พร้อมข้อมูลเพิ่มเติมให้ zhongkui
bwoc send zhongkui "IOC match ความน่าเชื่อถือสูงบน IP 10.0.4.17 — ดู intel ของ taibai ref TI-20260607" \
  --from agent-erlang

# laojun แจ้ง yanluo ว่า patch พร้อมสำหรับการตรวจสอบ
bwoc send yanluo "TASK-002 เสร็จแล้ว: apply patch แล้ว bwoc check ผ่าน รอ audit" \
  --from agent-laojun

# ตรวจสอบ inbox ของ agent
bwoc inbox zhongkui
bwoc inbox zhongkui --watch   # โหมด tail
```

**ดูและกรอง task:**

```bash
bwoc task list tianting                    # task ทั้งหมดพร้อม state และผู้ claim
bwoc task list tianting --state pending    # เฉพาะ task ที่ยังไม่มีคนรับ
```

### 8.3 ตรวจสอบสุขภาพ fleet

รันการตรวจสอบสุขภาพ fleet ก่อน daily standup ทุกวัน:

```bash
bwoc fleet health
```

ส่งข้อค้นพบใดๆ ไปให้ yudi เป็น task:

```bash
bwoc task add tianting "ข้อค้นพบจาก fleet health: agent-laojun stale (session ล่าสุดเมื่อ 8 วันก่อน) — สอบสวน"
```

สำหรับการรวมกับ CI:

```bash
bwoc fleet health --json | jq '.findings[] | select(.severity == "error")'
```

---

## 9. Red team ใช้ได้เฉพาะเมื่อได้รับอนุญาตเท่านั้น

ทักษะของ nezha — `penetration-testing`, `adversary-emulation`, และ `exploit-poc` — ถูก scope ชัดเจนให้ใช้ได้เฉพาะกับ engagement ที่ได้รับอนุญาตเท่านั้น นี่ไม่ใช่นโยบายที่เลือกได้ แต่เป็นข้อจำกัดแข็งใน anti-scope declaration ของ nezha และบังคับใช้โดยข้อกำหนดการอนุมัติแผนของ yudi

ก่อน red-team task ใดๆ จะเริ่มต้น:

1. **ต้องมีการอนุญาตอย่างชัดเจน.** ระบบเป้าหมาย ขอบเขตการทดสอบ ช่วงเวลา และเทคนิคที่ได้รับอนุญาตต้องถูกบันทึกใน plan entry ของ task — ยื่นผ่าน `bwoc task plan` และอนุมัติผ่าน `bwoc task approve` โดย yudi ก่อนดำเนินการใดๆ
2. **ห้ามขยาย scope.** ถ้า nezha ค้นพบว่าเทคนิค exploit จำเป็นต้องเข้าถึงระบบนอกขอบเขตที่ได้รับอนุญาต task ต้องหยุดชั่วคราวและต้องได้รับการอนุญาตใหม่ การเคลื่อนไหวด้านข้างโดยไม่ได้รับอนุญาตเป็นการละเมิดภวตัณหา
3. **ข้อค้นพบทั้งหมดต้องส่งให้ laojun และ yanluo ทันที.** Exploit POC ไม่ถูกเก็บใน agent memory หรือ commit ไปยัง repository นอกระบบจัดการข้อค้นพบที่ได้รับอนุญาต มันถูกส่งมอบเป็น structured finding (พร้อมคะแนน severity โดย yanluo) จากนั้นลบออกจาก working context
4. **หมุน key หลังจาก engagement แต่ละครั้ง.** หลังจาก red-team engagement ให้หมุน signing key ของ nezha เพื่อให้แน่ใจว่า envelope signature ที่อาจถูก compromise ไม่สามารถ replay ได้: `bwoc trust --keygen agents/agent-nezha --force`

เหตุผล: red team ที่ได้รับอนุญาตซึ่งพบ exploit จริงและส่งต่อให้ laojun ไปแก้มีมูลค่าสูง red team ที่ไม่ได้รับอนุญาตไม่ต่างจากผู้โจมตี gate การอนุมัติ (yudi) และ audit trail (yanluo) คือการควบคุมที่ทำให้ความแตกต่างนั้นมองเห็นได้

---

## 10. ดูเพิ่มเติม

**source framework (GitHub สาธารณะ):**

- [THREAT-MODEL.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/THREAT-MODEL.en.md) — แบบจำลองภัยคุกคามฉบับเต็มพร้อม ID ที่มีหมายเลข การบรรเทา และ severity rating
- [FLEET-GOVERNANCE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md) — อปริหานิยธรรม 7 ฉบับเต็ม และสเปค `bwoc fleet health`
- [SIGNING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) — สเปค signing ed25519 schema ของ envelope และ trust gate logic
- [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) — การ mapping 22 framework ฉบับเต็ม รวมถึงตัณหา 3 ศีล 5 และกรรม 3 ในบริบท

**บทอื่นในคู่มือนี้:**

- [../agents/HANDBOOK.th.md](../agents/HANDBOOK.th.md) — การสร้าง agent การประกาศ trust การจัดการ signing keypair
- [../philosophy/HANDBOOK.th.md](../philosophy/HANDBOOK.th.md) — เหตุผลที่ BWOC ใช้คำศัพท์จากบาลี และเหตุผลในการออกแบบ
- [../end-user/HANDBOOK.th.md](../end-user/HANDBOOK.th.md) — `bwoc team`, `bwoc task`, `bwoc send`, และ `bwoc inbox` สำหรับ operator ที่ใหม่กับ CLI
- [../glossary.th.md](../glossary.th.md) — ทุกคำในตารางเดียว ความหมายเชิงวิศวกรรมขึ้นก่อน

---

*คู่มือนี้ดูแลควบคู่กับ framework เมื่อเนื้อหาที่อธิบายที่นี่ขัดแย้งกับ [framework repo](https://github.com/bemindlabs/BWOC-Framework) repo เป็นฝ่ายถูก — กรุณาเปิด fix*
