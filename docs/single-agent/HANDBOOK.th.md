# Single-Agent Workspace (เอเจนต์เดียว)

🇬🇧 [English](HANDBOOK.en.md)

ไม่ใช่ทุกงานต้องมีทั้งฝูง BWOC มีโหมด **single-agent**: workspace ที่สร้างรอบเอเจนต์ **ตัวเดียว** แทนค่าเริ่มต้นแบบ multi-agent fleet บทนี้ว่าด้วยเมื่อไหร่ควรใช้ สร้างยังไง และสัมพันธ์กับ fleet อย่างไร

> ศัพท์: เอเจนต์ของ BWOC อยู่ใน **workspace** เสมอ ไม่มีคอนเซปต์ "global agent" ระดับเครื่อง — การใช้เอเจนต์ตัวเดียวคือ **single-agent workspace** ถ้าอยากให้เอเจนต์ตัวหนึ่งเข้าถึงได้จาก workspace/เครื่องอื่น นั่นคือเรื่อง [ข้าม workspace](../cross-workspace/HANDBOOK.th.md) (peer / A2A) ไม่ใช่บทนี้

## เมื่อไหร่ควรใช้ single-agent

| ใช้ single-agent เมื่อ… | ใช้ fleet (ค่าเริ่มต้น) เมื่อ… |
|---|---|
| ผู้ช่วยตัวเดียวโฟกัสงาน/repo เดียว | หลายเอเจนต์ บทบาทต่างกัน |
| ตั้งง่าย ไม่ต้องมีทีม/ประสานงาน | ต้องมีทีม, task list ร่วม, peer review |
| เรียนรู้ BWOC / ทดลองเร็ว ๆ | ทีม security, pipeline, แบ่งงานกันทำ |
| CI / ตรวจ workspace อย่างเดียว | ปฏิบัติการหลายเอเจนต์ต่อเนื่อง |

## สร้าง

```bash
bwoc init ./my-ws --single-agent     # workspace เอเจนต์เดียว (profile: single-agent)
cd my-ws
```

`--single-agent` ต่างจาก fleet ตรงไหน:

- บันทึก **profile** ของ workspace เป็น `single-agent` (ค่าเริ่มต้นคือ `fleet`)
- โฟลเดอร์ `agents/` ถูก scaffold ด้วย **คำแนะนำแบบเอเจนต์เดียว** แทน README แบบ fleet
- ที่เหลือเป็น workspace ปกติ: `.bwoc/workspace.toml`, ทะเบียน `agents.toml`, หน่วยความจำ ฯลฯ

> สำหรับ workspace แบบ CI / ตรวจอย่างเดียว ที่ไม่ spawn daemon เพิ่ม `--no-runtime` (ตัด `.gitignore` ของ daemon ออก; `bwoc check` ยังผ่าน) ใช้ร่วมกับ `--single-agent` ได้

## เพิ่มและสั่งงานเอเจนต์ตัวเดียว

```bash
bwoc new sage --role "project assistant" --target agents/agent-sage
bwoc check agents/agent-sage
```

แล้วใช้เครื่องมือครบสำหรับเอเจนต์เดียว — ไม่ต้องมีทีม/peer:

```bash
bwoc chat agent-sage                     # โต้ตอบ
bwoc run  agent-sage --task "สรุปงานวันนี้"   # headless
bwoc spawn agent-sage --backend ollama   # เลือก backend ใดก็ได้
bwoc memory put note "..."; bwoc memory search "..."   # หน่วยความจำ workspace
bwoc status ; bwoc log agent-sage -f      # ติดตาม
```

การเขียนตัวตน (persona / mindsets / skills) และหน่วยความจำ ทำเหมือนในบท [Agents](../agents/HANDBOOK.th.md) และ [Persona·Mindsets·Skills](../slots/HANDBOOK.th.md) ทุกประการ — single-agent workspace แค่มีเอเจนต์เดียวเท่านั้น

## สัมพันธ์กับ fleet

single-agent workspace มี **โครงสร้างบนดิสก์เหมือน fleet** ทุกอย่าง — แฟลกแค่ตั้ง profile กับคำแนะนำที่ scaffold ดังนั้น:

- **โตทีหลังได้:** เพิ่มเอเจนต์ด้วย `bwoc new` เมื่องานเกินหนึ่งเอเจนต์; ทีม (`bwoc team`) และ task ร่วม (`bwoc task`) พร้อมใช้ทันทีที่มีมากกว่าหนึ่งตัว
- **คงความลีน:** ถ้าเอเจนต์เดียวพอ ก็อยู่อย่างนั้น — *มัตตัญญุตา* (พอดี): อย่าเพิ่มฝูงที่ไม่ต้องการ
- **แชร์ข้ามเครื่อง:** ถ้าอยากเปิดเอเจนต์ตัวนี้ให้ workspace อื่นเข้าถึง ใช้ [`bwoc peer`](../cross-workspace/HANDBOOK.th.md) + โปรโตคอล A2A — นั่นคือ federation ไม่ใช่การติดตั้งแบบ "global"

## อ่านต่อ

- [Quickstart](../quickstart/HANDBOOK.th.md) · [End-user](../end-user/HANDBOOK.th.md) · [Agents](../agents/HANDBOOK.th.md) · [ข้าม workspace](../cross-workspace/HANDBOOK.th.md)
- ต้นฉบับ: [`WORKSPACE.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/WORKSPACE.en.md) · [`INCARNATION.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/INCARNATION.en.md)
