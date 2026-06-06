# Quickstart — สร้าง Agent ตัวแรกใน ~10 นาที

> English version: [HANDBOOK.en.md](HANDBOOK.en.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) · Glossary: [../glossary.th.md](../glossary.th.md)

บทนี้เป็น walkthrough แบบ copy-paste ครบวงจร จบแล้วคุณจะได้: ติดตั้ง `bwoc`, สร้าง workspace, incarnate agent, คุยกับมัน, ส่งงาน, ตั้งทีม, แล้ว retire อย่างสะอาด แต่ละขั้นใช้เวลาประมาณหนึ่งนาที รวมไม่เกินสิบ

เมื่อต้องการรายละเอียดของหัวข้อใด กล่อง "ขั้นต่อไป" ท้ายบทจะพาไปยังบทที่ถูกต้อง

---

## วงชีวิตที่กำลังจะเดิน

ทุก object ใน BWOC วิ่งสามเฟส framework เรียกว่า *uppāda · ṭhiti · vaya* หรือพูดง่าย ๆ: **เกิด → ทำงาน → ปลดระวาง** บทนี้จะพาเดินครบทั้งสามเฟสในคราวเดียว

---

## ขั้นที่ 0 — สิ่งที่ต้องมีก่อน

ต้องใช้ **Rust 1.85 ขึ้นไป** ตรวจด้วย:

```bash
rustc --version
```

ถ้ายังไม่มีหรือเวอร์ชันเก่า ติดตั้งหรืออัปเดตผ่าน [rustup.rs](https://rustup.rs)

นั่นคือข้อกำหนดเดียว binary ของ `bwoc` พอใจตัวเอง ไม่ต้องการ runtime อื่นสำหรับ tutorial นี้

---

## ขั้นที่ 1 — ติดตั้ง CLI

Clone framework แล้วติดตั้ง binary:

```bash
git clone https://github.com/bemindlabs/BWOC-Framework
cd BWOC-Framework
cargo install --path crates/bwoc-cli
```

**สิ่งที่เกิดขึ้น.** Cargo คอมไพล์ CLI crate แล้ววาง binary `bwoc` ไว้ที่ `~/.cargo/bin/` ครั้งแรกใช้เวลาหนึ่งถึงสองนาที

ตรวจว่าทำงานได้:

```bash
bwoc --help
```

**ผลที่คาดหวัง.** help text แสดง subcommands ทั้งหมด ควรเห็น `new`, `check`, `chat`, `run`, `team`, `status` และอื่น ๆ

จากนั้น run doctor เพื่อตรวจสภาพแวดล้อมก่อนเดินหน้า:

```bash
bwoc doctor --auto
```

**ผลที่คาดหวัง.** แต่ละรายการพิมพ์ `ok` หรือซ่อมแซมตัวเอง ถ้ามีรายการที่ยังเป็น `warn` หรือ `error` แก้ให้เสร็จก่อนดำเนินต่อ

---

## ขั้นที่ 2 — สร้าง workspace

Workspace คือ directory ราก ที่เก็บ agent, config ทีม และ task list ร่วม

```bash
bwoc init ./my-workspace
cd my-workspace
```

**สิ่งที่เกิดขึ้น.** `bwoc init` สร้าง `.bwoc/` (registry + teams), folder `agents/` เริ่มต้น, และ workspace manifest ต่อจากนี้ทุก command รันจาก workspace root นี้

ยืนยันว่า register แล้ว:

```bash
bwoc workspace info
```

**ผลที่คาดหวัง.** สรุปย่อ: ชื่อ workspace, path, จำนวน agent ที่ลงทะเบียน (ศูนย์ตอนนี้)

ดูรายการ agent (ยังว่าง):

```bash
bwoc list
```

**ผลที่คาดหวัง.** "No agents registered" หรือตารางว่าง — ถูกต้อง ยังไม่ได้สร้าง

---

## ขั้นที่ 3 — สร้าง agent ตัวแรก

```bash
bwoc new sage --role "research assistant" --target agents/agent-sage
```

**สิ่งที่เกิดขึ้น.** `bwoc new` copy template ลง `agents/agent-sage/`, เขียน `config.manifest.json`, สร้าง slot directories (`persona/`, `mindsets/`, `skills/`, `interconnect/`, `memories/`), generate `AGENTS.md`, และเพิ่ม agent เข้า `.bwoc/agents.toml`

flags เสริมที่ใช้ได้ (ไม่จำเป็นสำหรับ tutorial นี้):

| Flag | หน้าที่ |
|---|---|
| `--backend ollama` | ตั้งค่า Ollama backend ล่วงหน้า |
| `--mindsets research,critical` | seed mindset file ที่ระบุ |
| `--skills writing,analysis` | seed skill file ที่ระบุ |

**ผลที่คาดหวัง.** เห็น creation summary และ path `agents/agent-sage/` ถูกสร้างพร้อม slot structure ครบ

---

## ขั้นที่ 4 — ตรวจสอบว่า agent ผ่าน validation

```bash
bwoc check agents/agent-sage
```

**สิ่งที่เกิดขึ้น.** auditor ตรวจว่า `AGENTS.md` ไม่มี syntax ที่ผูกกับ backend ใดเฉพาะ (ไม่มี YAML frontmatter, ไม่มี wikilinks, ไม่มีชื่อ model hardcode), ว่า `config.manifest.json` เป็น JSON ที่ valid, และว่า `MEMORY.md` ไม่เกิน 200 บรรทัด

**ผลที่คาดหวัง.** ทุก check พิมพ์ `ok` agent ที่ scaffold ใหม่ ๆ ผ่านเสมอ ถ้าไม่ผ่านให้ run `bwoc doctor --auto` แล้วลองใหม่

---

## ขั้นที่ 5 — คุยกับ agent

เปิด interactive chat session:

```bash
bwoc chat agent-sage
```

**สิ่งที่เกิดขึ้น.** harness โหลด manifest ของ agent, resolve backend (ใช้ default ของคุณถ้าไม่ระบุ `--backend`), แล้วเปิด prompt

ลองพิมพ์:

```
What can you help me with?
```

พิมพ์ `/exit` หรือกด `Ctrl-D` เพื่อออก

**ผลที่คาดหวัง.** agent ตอบโต้ตาม role ("research assistant") และ mindsets/skills ที่ประกาศใน slots

ถ้าต้องการเลือก backend เฉพาะสำหรับ session นี้โดยไม่แก้ manifest:

```bash
bwoc spawn agent-sage --backend claude
# หรือ
bwoc spawn agent-sage --backend ollama
```

---

## ขั้นที่ 6 — รัน task แบบ headless

สำหรับงาน automation ที่ไม่ต้องการ interactive session ใช้ `bwoc run`:

```bash
bwoc run agent-sage --task "Summarize what a research assistant does in three bullet points"
```

**สิ่งที่เกิดขึ้น.** harness ส่ง task prompt ไปยัง agent แล้ว stream ผลลัพธ์มาที่ stdout จากนั้นออก

flags ที่มีประโยชน์:

| Flag | หน้าที่ |
|---|---|
| `--json` | emit structured JSON output แทน plain text |
| `--timeout 60` | หยุดถ้า agent ไม่ตอบภายใน 60 วินาที |

**ผลที่คาดหวัง.** สาม bullet points พิมพ์ที่ terminal แล้ว process ออกอย่างสะอาด

---

## ขั้นที่ 7 — ตั้งทีม (ไม่บังคับ แต่แนะนำ)

ทีมช่วยให้ agent หลายตัวแชร์ task list และรับงานร่วมกัน แม้จะมีแค่ agent เดียวก็คุ้มลองเพื่อให้เห็น pattern

สร้างทีมและเพิ่ม `sage`:

```bash
bwoc team create research --members sage
```

เพิ่ม task เข้า task list ร่วมของทีม:

```bash
bwoc task add research "Draft a one-paragraph project brief"
```

ให้ `sage` รับงาน:

```bash
bwoc task claim research --as agent-sage
```

`sage` ทำงาน (รันหรือ chat ตามขั้นที่ 5–6) เมื่อเสร็จ mark complete:

```bash
bwoc task complete research --as agent-sage
```

**ผลที่คาดหวัง.** task เดินจาก `open → claimed → done` ใน task list ของทีม ดู end-user handbook สำหรับ command surface ครบของ task

---

## ขั้นที่ 8 — สังเกตสิ่งที่เกิดขึ้น

ขณะที่ agent ทำงาน มีเครื่องมือสามอย่าง:

```bash
# health status บรรทัดเดียว
bwoc status

# ติดตาม log แบบ live ของ agent ที่ระบุ (Ctrl-C เพื่อหยุด)
bwoc log agent-sage -f

# dashboard ครบ — agent, task, และ resource ทั้งหมด
bwoc dashboard
```

**ดูอะไร.** `bwoc status` แสดงว่าแต่ละ agent idle, running, หรือ stopped `bwoc log` stream structured events ขณะ agent ประมวลผลงาน `bwoc dashboard` ให้ภาพรวมที่มีประโยชน์เมื่อมี agent หลายตัวทำงานพร้อมกัน

---

## ขั้นที่ 9 — หยุดและปลดระวาง agent

หยุด process ของ agent โดยไม่ลบอะไร:

```bash
bwoc stop agent-sage
```

เมื่อพร้อมจะลบ agent ทั้งหมดแต่ต้องการเก็บ memory ไว้อ้างอิง:

```bash
bwoc retire agent-sage --keep-memory
```

**สิ่งที่เกิดขึ้น.** `bwoc retire` ลบ agent ออกจาก `.bwoc/agents.toml`, ทำความสะอาด process state, และ — เพราะใส่ `--keep-memory` — ทิ้ง slot `memories/` ไว้เพื่อไม่ให้ความรู้ที่สะสมหายไป ถ้าไม่ใส่ flag นั้น directory ทั้งหมดจะถูกลบ

**ผลที่คาดหวัง.** `bwoc list` ไม่แสดง `sage` อีกต่อไป folder `memories/` ยังอยู่ที่ `agents/agent-sage/memories/` ถ้าใช้ `--keep-memory`

นี่คือเฟส **ปลดระวาง** — เฟสที่สามและสุดท้ายของวงชีวิต เกิด → ทำงาน → ปลดระวาง

---

## ขั้นต่อไป

คุณเดินวงชีวิตครบแล้วหนึ่งรอบ ต่อไปนี้คือที่ที่ควรไปต่อ:

| หัวข้อ | อ่านต่อ |
|---|---|
| CLI surface ครบ — ทุก flag, ทุก command | [../end-user/HANDBOOK.th.md](../end-user/HANDBOOK.th.md) |
| Incarnation agent, slot files, กฎ AGENTS.md | [../agents/HANDBOOK.th.md](../agents/HANDBOOK.th.md) |
| เขียน persona, mindsets, และ skills | [../slots/HANDBOOK.th.md](../slots/HANDBOOK.th.md) |
| Backends — Claude, Codex, Kimi, Ollama และอื่น ๆ | [../backends/HANDBOOK.th.md](../backends/HANDBOOK.th.md) |
| ปรัชญาการออกแบบและ dhamma mapping | [../philosophy/HANDBOOK.th.md](../philosophy/HANDBOOK.th.md) |
| ค้นหาคำศัพท์ | [../glossary.th.md](../glossary.th.md) |
| Framework source — canonical spec | [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) |

docs ของ framework เองอยู่ที่ `docs/en/` ใน repo นั้น เช่น incarnation spec ที่ [docs/en/INCARNATION.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/INCARNATION.en.md) และ agent template spec ที่ [modules/agent-template/AGENTS.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md)
