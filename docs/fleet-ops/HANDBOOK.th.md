# คู่มือการดำเนินงาน Fleet

🇬🇧 [English](HANDBOOK.en.md)

สำหรับ **ผู้ดำเนินงานและหัวหน้าทีม** ที่รัน agent หลายตัวพร้อมกันและต้องการให้ระบบทำงานต่อเนื่อง ตรวจสอบได้ และฟื้นคืนได้ — ไม่ใช่แค่ทีละตัว
หากยังเรียนรู้การใช้งาน agent พื้นฐานอยู่ เริ่มที่ [`../end-user/HANDBOOK.th.md`](../end-user/HANDBOOK.th.md)
หากต้องการปรับ trust และ signing ระหว่าง agent ดูที่ [`../security/HANDBOOK.th.md`](../security/HANDBOOK.th.md)
ค้นหาคำศัพท์: [`../glossary.th.md`](../glossary.th.md) · Repository: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)

---

## สารบัญ

1. [ปัญหาด้านการดำเนินงานเมื่อขยายขนาด](#1-ปัญหาดานการดำเนนงานเมอขยายขนาด)
2. [สุขภาพ Fleet — สัญญาณทั้งเจ็ด](#2-สขภาพ-fleet--สญญาณทงเจด)
3. [รักษา Agent ให้ทำงานต่อเนื่อง — supervise](#3-รกษา-agent-ใหทำงานตอเนอง--supervise)
4. [การมองเห็นระบบ — Sessions, Lists, Ping, Dashboard, Logs, Inboxes](#4-การมองเหนระบบ--sessions-lists-ping-dashboard-logs-inboxes)
5. [วินิจฉัยและซ่อมแซม](#5-วนจฉยและซอมแซม)
6. [การดำเนินการแบบกลุ่ม](#6-การดำเนนการแบบกลม)
7. [Checklist การดำเนินงานประจำวัน](#7-checklist-การดำเนนงานประจำวน)
8. [ดูเพิ่มเติม](#8-ดเพมเตม)

---

## 1. ปัญหาด้านการดำเนินงานเมื่อขยายขนาด

Agent ตัวเดียวจัดการได้ง่าย แต่เมื่อมี Agent สิบตัวใช้ workspace ร่วมกัน — แต่ละตัวมี daemon, inbox, memory และ task assignment เป็นของตัวเอง — ปัญหาจะเปลี่ยนรูปแบบ:

- Agent หยุดทำงานเงียบ ๆ โดยไม่มีใครสังเกตเห็นนานหลายชั่วโมง
- Registry entries เก่าสะสมมาจากการทดลองที่ไม่เคย cleanup
- Bug ใน agent ตัวหนึ่งทำให้เกิด restart loop ที่กินทรัพยากรและบล็อก queue ที่ใช้ร่วมกัน
- ไม่สามารถรู้ได้ด้วยตาว่า agent ไหนกำลัง idle ไหนกำลังทำงาน และไหนตายไปแล้ว
- การ restart กลุ่มตามกำหนดเวลามีความเสี่ยง เพราะไม่มีคำสั่งเดียวที่จะหยุดทุกอย่างได้อย่างปลอดภัย

Fleet operations คือแนวปฏิบัติในการควบคุมสิ่งเหล่านี้ทั้งหมด BWOC มีชุดคำสั่งสำหรับระดับนี้โดยเฉพาะ ไม่มีคำสั่งใดทดแทนคำสั่ง agent-level ที่รู้จักอยู่แล้ว แต่ทำงานร่วมกันได้

---

## 2. สุขภาพ Fleet — สัญญาณทั้งเจ็ด

### `bwoc fleet health` ทำอะไร

```bash
bwoc fleet health
```

คำสั่งนี้ตรวจสอบสัญญาณสุขภาพเจ็ดอย่างในทุก agent ที่ลงทะเบียนใน workspace และพิมพ์รายงาน คำสั่งนี้ **อ่านอย่างเดียว** — ไม่เปลี่ยนแปลงอะไรทั้งนั้น

สัญญาณทั้งเจ็ดมาจากหลักธรรม *อปริหานิยธรรม* (เจ็ดเงื่อนไขแห่งความไม่เสื่อม) ใน BWOC ชื่อนี้เป็น label ทางวิศวกรรม ไม่ใช่การอ้างอิงทางศาสนา อ่านได้ว่า: *เจ็ดเงื่อนไขที่ถ้าละเมิดแล้วจะทำให้ระบบ multi-agent เสื่อมลง* เมื่อทุกสัญญาณผ่าน fleet ถือว่ามีสุขภาพดี

| # | สัญญาณ | ตรวจสอบอะไร |
|---|--------|-------------|
| 1 | **การประชุม** (Assembly) | Agent ทุกตัวที่ลงทะเบียนมี `AGENTS.md` ที่อ่านได้และถูกต้อง |
| 2 | **ความตกลง** (Accord) | ไม่มี task assignments ที่ขัดแย้งกันใน agent ทีมเดียวกัน |
| 3 | **การไม่เพิ่ม** (Non-addition) | ไม่มีไดเรกทอรี agent ที่ยังไม่ลงทะเบียนอยู่ควบคู่กับตัวที่ลงทะเบียนแล้ว |
| 4 | **อาวุโส** (Seniority) | Agent ที่ประกาศ trust level มี signing evidence ที่ถูกต้องและไม่หมดอายุ |
| 5 | **การไม่บังคับ** (Non-coercion) | inbox ของ agent ไม่มีการหยุดนิ่ง (ข้อความที่ยังไม่ได้อ่านเกิน threshold ที่กำหนด) |
| 6 | **ที่พึ่ง** (Refuge) | Agent ที่ active แต่ละตัวมี supervise daemon ที่เข้าถึงได้ หรือมีบันทึก clean-exit |
| 7 | **สวัสดิภาพ** (Welfare) | ไฟล์ memory (`MEMORY.md`) อยู่ภายใน 200 บรรทัดทั่วทั้ง fleet |

ผลลัพธ์เมื่อผ่านทั้งหมด:

```
Fleet health — 12 agents checked
  ✓ Assembly     12/12
  ✓ Accord       no conflicts
  ✓ Non-addition no ghost dirs
  ✓ Seniority    4 signed, 8 unsigned (trust level: none — ok)
  ✓ Non-coercion no stalled inboxes
  ✓ Refuge       10 supervised, 2 clean-exit
  ✓ Welfare      all MEMORY.md within 200 lines

All seven signals pass.
```

ผลลัพธ์เมื่อล้มเหลว จะระบุ agent และสัญญาณที่ต้องแก้ไข:

```
Fleet health — 12 agents checked
  ✗ Refuge       agent-loki: no supervise daemon, last exit unrecorded
  ✗ Welfare      agent-atlas: MEMORY.md 247 lines (limit 200)

2 signals failed. Run `bwoc doctor --auto` to auto-fix safe issues.
```

รัน `bwoc fleet health` ตอนเริ่มต้นแต่ละ shift และหลังการดำเนินการกลุ่มทุกครั้ง

---

## 3. รักษา Agent ให้ทำงานต่อเนื่อง — supervise

### ปัญหาที่ supervise แก้ไข

daemon ของ agent อาจ crash ได้ — network call ผิดพลาด model timeout หรือ panic ที่ไม่ได้จัดการ หากไม่มีการดูแล การ crash จะเงียบ `bwoc supervise` ครอบ daemon ของ agent ด้วย restart loop และมี crash-loop backstop เพื่อป้องกัน agent ที่เสียหายจากการ thrash ไม่หยุด

### การใช้งาน

```bash
bwoc supervise <agent-path>
bwoc supervise agents/agent-sage
bwoc supervise agents/agent-sage --max-restarts-per-min 5
bwoc supervise agents/agent-sage --json
```

| Flag | ค่าเริ่มต้น | ทำอะไร |
|------|------------|--------|
| `--max-restarts-per-min` | `10` | ถ้า agent crash และ restart เกินจำนวนนี้ในหนึ่งนาที supervisor จะหยุดพยายามและส่ง event `rate_limit_hit` นี่คือการป้องกัน agent เสียจากการกิน resource |
| `--json` | ปิด | ส่ง JSON event หนึ่งรายการต่อ action ไปยัง stdout มีประโยชน์สำหรับ pipe เข้า log aggregator |

### Event stream (`--json`)

แต่ละ event เป็น JSON object หนึ่งบรรทัด:

| ค่า `event` | เมื่อเกิดขึ้น |
|-------------|--------------|
| `spawn` | daemon ของ Agent เริ่มต้นครั้งแรก |
| `crash_respawn` | daemon ออกโดยไม่คาดคิด; supervisor กำลัง restart |
| `clean_exit` | daemon ออกด้วย status 0 (หยุดปกติ); supervisor ออก |
| `rate_limit_hit` | อัตรา restart เกิน `--max-restarts-per-min`; supervisor ออก |
| `signal_stop` | supervisor ได้รับ SIGTERM/SIGINT; daemon หยุดอย่างสะอาด |

### Crash-loop backstop

guard `--max-restarts-per-min` มีไว้เพื่อป้องกัน agent ที่กำหนดค่าผิดหรือเสียหายไม่ให้ restart ไม่หยุด เมื่อ rate limit เกิดขึ้น supervisor จะออกและบันทึก event จะเห็นความล้มเหลวใน `bwoc fleet health` (สัญญาณ 6: Refuge) และใน `bwoc log <agent>` แก้ปัญหาพื้นฐานก่อน จากนั้น restart supervisor ด้วยตนเอง

### การหยุด agent ที่ถูก supervise

ส่ง SIGTERM ไปยัง supervisor process หรือใช้:

```bash
bwoc stop agents/agent-sage
```

supervisor รับ signal หยุด daemon อย่างสะอาด รอให้ออก ส่ง event `signal_stop` แล้วออกเอง

---

## 4. การมองเห็นระบบ — Sessions, Lists, Ping, Dashboard, Logs, Inboxes

### 4.1 `bwoc sessions` — อะไรกำลังรันจริง ๆ ตอนนี้

```bash
bwoc sessions
bwoc sessions --idle-secs 120
bwoc sessions --json
```

`bwoc sessions` ตรวจจับ session ที่กำลังรันโดยสแกน process markers และ Unix socket activity แยกความต่างระหว่าง **working** (มี traffic ล่าสุด) กับ **idle** (ไม่มี traffic) ใช้ `--idle-secs` เพื่อปรับจำนวนวินาทีที่ไม่มีกิจกรรมก่อนจะจัดว่า idle `--json` ส่งออก object หนึ่งรายการต่อ session

### 4.2 `bwoc list` — การกรองระดับ registry

```bash
bwoc list                          # agent ที่ลงทะเบียนทั้งหมด
bwoc list --running                # เฉพาะ agent ที่มี daemon ทำงานอยู่
bwoc list --inbox-pending          # เฉพาะ agent ที่มีข้อความ inbox ที่ยังไม่ได้อ่าน
bwoc list --status <value>         # กรองตาม status ใน manifest
bwoc list --backend <value>        # กรองตาม backend (claude, codex, ollama, …)
bwoc list --count                  # พิมพ์ตัวเลข ไม่ใช่ตาราง
bwoc list --names-only             # พิมพ์ชื่อ agent เท่านั้น — ใช้ใน script
```

flags เหล่านี้ใช้รวมกันได้ ตัวอย่าง นับ Claude agent ที่กำลังรัน:

```bash
bwoc list --running --backend claude --count
```

หรือรับชื่อ agent ที่มีข้อความ inbox ค้างอยู่เพื่อใช้ใน shell loop:

```bash
for agent in $(bwoc list --inbox-pending --names-only); do
  bwoc inbox "$agent"
done
```

### 4.3 `bwoc ping` — ตอบสนองอยู่ไหม

```bash
bwoc ping agents/agent-sage        # ping agent หนึ่งตัว
bwoc ping --all                    # ping ทุก agent ที่กำลังรัน
```

ส่ง `PING` ผ่าน Unix socket ของ agent และรอ `PONG` หาก daemon หยุดนิ่ง (อยู่ใน process list แต่ไม่ตอบสนอง) `ping` จะ timeout ซึ่ง `sessions` ตรวจไม่พบ ใช้ ping เมื่อสงสัยว่า agent ค้างอยู่

### 4.4 `bwoc dashboard` — ภาพรวม interactive

```bash
bwoc dashboard
```

เปิด terminal UI (TUI) แสดง agent ที่ลงทะเบียนทั้งหมด สถานะ log ล่าสุด และจำนวน inbox ในหน้าเดียว กด `r` เพื่อ refresh ด้วยตนเอง มีประโยชน์สำหรับดู fleet ระหว่าง batch run ยาวหรือหลัง deployment ออกด้วย `q` หรือ `Ctrl-C`

### 4.5 `bwoc log` — ดู log ของ agent

```bash
bwoc log agents/agent-sage
bwoc log agents/agent-sage -f           # follow (เหมือน tail -f)
bwoc log agents/agent-sage -f -n 50     # follow แสดง 50 บรรทัดสุดท้ายก่อน
```

อ่าน log ที่มีโครงสร้างของ agent `-f` เปิด stream ต่อเนื่อง `-n N` กำหนดจำนวนบรรทัดประวัติที่แสดงก่อน follow

### 4.6 `bwoc inbox` — อ่านและดูข้อความ

```bash
bwoc inbox --all                        # แสดงรายการ inbox ที่ยังไม่ได้อ่านทั่วทั้ง fleet
bwoc inbox agents/agent-sage            # แสดง inbox ของ agent นั้น
bwoc inbox agents/agent-sage --watch    # stream ข้อความ inbox ใหม่แบบ real-time
```

inbox คือวิธีที่ agent รับ task และข้อความระหว่าง agent inbox ที่หยุดนิ่ง (สัญญาณสุขภาพ fleet ข้อ 5) หมายความว่าข้อความค้างอยู่ ซึ่งมักหมายความว่า daemon ของ agent ไม่ได้รันหรือไม่ได้ประมวลผล

---

## 5. วินิจฉัยและซ่อมแซม

### 5.1 `bwoc doctor` — วินิจฉัย environment และ workspace

```bash
bwoc doctor
bwoc doctor --auto
```

`bwoc doctor` โดยไม่มี flag วินิจฉัย environment และ workspace: ตรวจว่า binary ที่จำเป็นมีอยู่ ยืนยันโครงสร้าง workspace ตรวจความสอดคล้องของ registry และรายงานปัญหาที่พบ `--auto` แก้ไขปัญหาที่ปลอดภัยโดยไม่ต้องการการตัดสินใจจากคน (เช่น สร้าง socket directory ที่หายไปใหม่ หรือเขียน lock file ที่ผิดรูปแบบใหม่) ปัญหาที่ต้องการการตัดสินใจของคนจะถูกรายงานแต่ไม่แก้โดยอัตโนมัติ

รัน `bwoc doctor --auto` เมื่อ `bwoc fleet health` รายงานความล้มเหลวที่ไม่เข้าใจในทันที ส่วนใหญ่แก้ปัญหาได้ในคำสั่งเดียว

### 5.2 `bwoc workspace validate` — ตรวจความสมบูรณ์ของ registry

```bash
bwoc workspace validate
```

ตรวจ workspace registry (`.bwoc/agents.toml`) เทียบกับ filesystem รายงาน:

- **Phantom entries**: agent ใน registry ที่ไดเรกทอรีไม่มีอยู่แล้ว
- **Orphan directories**: ไดเรกทอรี agent ที่มีอยู่บน disk แต่ไม่ได้อยู่ใน registry
- **Manifest errors**: ไฟล์ `config.manifest.json` ที่หายไปหรือมี JSON ไม่ถูกต้อง

คำสั่งนี้อ่านอย่างเดียว บอกว่ามีอะไรผิด `prune` จัดการ cleanup

### 5.3 `bwoc workspace prune` — ทำความสะอาด registry

```bash
bwoc workspace prune               # dry run — แสดงว่าจะลบอะไร
bwoc workspace prune --apply       # ลบ phantom entries และ orphan dirs จริง ๆ
```

หากไม่มี `--apply` `prune` เป็น dry run ที่พิมพ์สิ่งที่จะทำ รัน dry run ก่อนเสมอเพื่อยืนยันรายการที่จะลบ ด้วย `--apply` phantom registry entries จะถูกลบ และ orphan directories จะถูกลบออกจาก disk วิธีนี้ปลอดภัยสำหรับ cleanup หลังการทดลองที่ล้มเหลวหรือ `bwoc retire` ที่ถูกขัดจังหวะ

### แผนการซ่อมแซม

```
bwoc fleet health รายงานความล้มเหลว
│
├── สัญญาณ 6 (Refuge): agent ไม่มี daemon
│   └── bwoc supervise <agent>  หรือ  bwoc doctor --auto
│
├── สัญญาณ 7 (Welfare): MEMORY.md เกิน limit
│   └── Prune MEMORY.md ของ agent ด้วยตนเองให้ ≤ 200 บรรทัด
│
├── สัญญาณ 3 (Non-addition): ghost dirs
│   └── bwoc workspace validate → bwoc workspace prune --apply
│
├── สัญญาณ 1 (Assembly): AGENTS.md ไม่ถูกต้อง
│   └── bwoc check <agent> เพื่อดูรายละเอียด; แก้ไขไฟล์
│
└── อะไรก็ตามที่ไม่ชัดเจน
    └── bwoc doctor --auto
```

---

## 6. การดำเนินการแบบกลุ่ม

### หยุด agent ทั้งหมด

```bash
bwoc stop --all
bwoc stop --all --yes              # ข้าม confirmation prompt
```

ส่ง signal shutdown อย่างสะอาดไปยัง daemon ทุกตัวที่กำลังรัน หากไม่มี `--yes` CLI จะขอการยืนยันก่อนดำเนินการ `--yes` สำหรับ script และ CI ที่ไม่มี interactive prompt

### เริ่ม agent ทั้งหมด

```bash
bwoc start --all
bwoc start --all --yes
bwoc start --all --no-daemon       # เริ่มแบบ inline (foreground) ไม่ใช่ background daemon
```

เริ่ม daemon สำหรับ agent ที่ลงทะเบียนทุกตัว `--no-daemon` เริ่มแต่ละ agent ใน foreground (blocking) ไม่ค่อยมีประโยชน์ใน fleet context แต่มีไว้สำหรับ debug

### Rolling restart ทั่วไป

```bash
bwoc stop --all --yes
# รอให้ process ออก
bwoc doctor --auto
bwoc start --all --yes
bwoc fleet health
```

รัน `bwoc fleet health` หลัง bulk start ทุกครั้งเพื่อยืนยันว่าสัญญาณทั้งเจ็ดผ่านก่อนถือว่า restart เสร็จสมบูรณ์

---

## 7. Checklist การดำเนินงานประจำวัน

Checklist นี้ใช้เวลาไม่ถึงสองนาทีสำหรับ fleet ที่มี agent ไม่เกิน 20 ตัว

```markdown
เช้า
- [ ] bwoc fleet health               — สัญญาณทั้งเจ็ดผ่านทั้งหมด?
- [ ] bwoc sessions                   — มี session ที่ idle หรือหายไปโดยไม่คาดคิด?
- [ ] bwoc inbox --all                — มีข้อความค้าง?
- [ ] bwoc dashboard                  — สแกนภาพรวมหาความผิดปกติ

หลัง deployment หรือการเปลี่ยนแปลงกลุ่ม
- [ ] bwoc doctor --auto              — แก้ไขปัญหาที่ปลอดภัย
- [ ] bwoc workspace validate         — มี phantom/orphan entries?
- [ ] bwoc fleet health               — ยืนยันสัญญาณทั้งเจ็ดอีกครั้ง
- [ ] bwoc ping --all                 — agent ทุกตัวที่รันอยู่ตอบสนอง?

รายสัปดาห์
- [ ] ตรวจขนาด MEMORY.md             — bwoc list + ตรวจด้วยตนเอง prune ถ้าใกล้ 200 บรรทัด
- [ ] bwoc workspace prune (dry run)  — มี entries เก่าที่ต้องทำความสะอาด?
- [ ] ตรวจ supervise logs             — มี agent ที่เจอ rate_limit_hit ซ้ำ ๆ?
```

---

## 8. ดูเพิ่มเติม

**เอกสาร framework (GitHub สาธารณะ)**

- [FLEET-GOVERNANCE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md) — specification ที่เป็น source of truth สำหรับ fleet governance คำนิยามสัญญาณ Aparihāniya-dhamma และกฎ trust ระดับ workspace
- [WORKSPACE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/WORKSPACE.en.md) — โครงสร้าง workspace รูปแบบ registry และ multi-workspace setup

**บทในคู่มือที่เกี่ยวข้อง**

- [`../end-user/HANDBOOK.th.md`](../end-user/HANDBOOK.th.md) — การใช้ agent แต่ละตัว sessions inbox lifecycle พื้นฐาน
- [`../agents/HANDBOOK.th.md`](../agents/HANDBOOK.th.md) — การ incarnate และปรับ agent; กฎ AGENTS.md; `bwoc check`
- [`../security/HANDBOOK.th.md`](../security/HANDBOOK.th.md) — trust levels การ signing การประกาศ trust ระหว่าง agent
- [`../glossary.th.md`](../glossary.th.md) — คำนิยามรวมถึง label ภาษาบาลีที่ใช้เป็นคำทางวิศวกรรม

---

> **Source of truth.** คู่มือนี้สรุปและช่วยในการเข้าใจภาพรวม หากมีความขัดแย้งใด ๆ framework repo ชนะและหน้านี้มี bug — กรุณาแก้ไข Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)
