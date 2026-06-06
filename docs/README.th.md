# BWOC Handbook (ฉบับภาษาไทย)

จุดเริ่มต้นเดียวสำหรับทุกคนที่ **ใช้ สร้าง ดูแล หรือ index** **BWOC framework** — เฟรมเวิร์กสำหรับสร้างและสั่งงาน AI agent ที่ทำงานได้กับหลาย backend ผ่าน CLI ตัวเดียวคือ `bwoc`

> 🇬🇧 English (ฉบับหลัก/canonical): [`README.md`](README.md) · repo เฟรมเวิร์ก: [`bemindlabs/BWOC-Framework`](https://github.com/bemindlabs/BWOC-Framework) · v2.24.0 · เอกสารต้นฉบับ (EN/TH): [`docs/`](https://github.com/bemindlabs/BWOC-Framework/tree/main/docs)

คู่มือชุดนี้ **แยกตามบทบาทและมีสองภาษา** อังกฤษเป็นฉบับหลัก (canonical) ไทยเป็นฉบับคู่ขนาน (`*.en.md` ↔ `*.th.md`) เปิดเล่มของคุณ อ่านจบในเล่มเดียว แล้วค่อยตามลิงก์เข้าไปอ่านเอกสารเฟรมเวิร์กฉบับเต็มเมื่อต้องการ

---

## เปิดเล่มของคุณ

| คุณคือ… | เปิด | สรุปสั้น |
|---|---|---|
| 🧭 **ทำไมต้อง BWOC** — การออกแบบ เหตุผล & ความสัมพันธ์กับหลักธรรม | [TH](philosophy/HANDBOOK.th.md) · [`EN`](philosophy/HANDBOOK.en.md) | เสาหลักการออกแบบ, จุดที่ DDD/SOLID บาง, ตารางแมป 22 กรอบ, ทำไมควรใช้ |
| 🧑‍💻 **ผู้ใช้งาน** — รันเอเจนต์ คุย ตั้งทีม สั่งงาน (ไม่แก้โค้ด) | [TH](end-user/HANDBOOK.th.md) · [`EN`](end-user/HANDBOOK.en.md) | ติดตั้ง สร้าง workspace สั่งงานด้วย `bwoc` |
| 🛠️ **นักพัฒนา** — ต่อยอด/แก้ตัวเฟรมเวิร์ก (Rust, CLI, harness) | [TH](developer/HANDBOOK.th.md) · [`EN`](developer/HANDBOOK.en.md) | แผนผัง crate, build/test, hooks, เวอร์ชัน, PR gates |
| 🤖 **ผู้สร้าง/ดูแลเอเจนต์** | [TH](agents/HANDBOOK.th.md) · [`EN`](agents/HANDBOOK.en.md) | โครงไฟล์, กฎ AGENTS.md, slot, manifest, วงจรชีวิต, `bwoc check` |
| ✍️ **Persona · Mindsets · Skills** — เขียนตัวตน & ความสามารถของเอเจนต์ | [TH](slots/HANDBOOK.th.md) · [`EN`](slots/HANDBOOK.en.md) | slot WHO/HOW/WHAT; frontmatter, tag `principle/`/`domain/`, maturity L1–L7 |
| 🌱 **Self-improvement** — เอเจนต์เรียนรู้ ทบทวน และพัฒนาตัวเองยังไง | [TH](self-improvement/HANDBOOK.th.md) · [`EN`](self-improvement/HANDBOOK.en.md) | learning loop (ศึกษา/ไตร่ตรอง/ลงมือ), memory tiers, curation, maturity, เมตริก |
| 🔌 **Backends** — ขับ & ตั้งค่าผ่านแต่ละ CLI | [TH](backends/HANDBOOK.th.md) · [`EN`](backends/HANDBOOK.en.md) | Claude/Codex/AGY/Kimi/Copilot/Ollama; ตั้งค่าด้วย prompt หลัง `init` |
| 🔎 **AI search / ตัวค้นหา** | [TH](ai-search/HANDBOOK.th.md) · [`EN`](ai-search/HANDBOOK.en.md) + [`llms.txt`](llms.txt) | ข้อเท็จจริงตัวจริงอยู่ไหน อ้างอิงยังไง อย่ามั่ว |
| 🕷️ **Crawler / ตัว index** | [TH](crawler/HANDBOOK.th.md) · [`EN`](crawler/HANDBOOK.en.md) + [`robots.txt`](robots.txt) + [`sitemap.md`](sitemap.md) | นโยบายเก็บข้อมูล สัญญาณความสด index อะไรบ้าง |
| 🧩 **อยากเห็นภาพทั้งตระกูล** — ทุกโปรเจกต์ BWOC & `bwoc-*` | [TH](ecosystem/HANDBOOK.th.md) · [`EN`](ecosystem/HANDBOOK.en.md) | แต่ละโปรเจกต์คืออะไร ใช้ stack อะไร เชื่อมกับ core ยังไง |

บทเพิ่มเติม:

| หัวข้อ | เปิด | สรุปสั้น |
|---|---|---|
| 🚀 **Quickstart** — สร้าง agent แรกใน ~10 นาที | [TH](quickstart/HANDBOOK.th.md) · [`EN`](quickstart/HANDBOOK.en.md) | ครบจบ: install → init → new → run → team → retire |
| 🛡️ **Security & ทีม tianting** | [TH](security/HANDBOOK.th.md) · [`EN`](security/HANDBOOK.en.md) | threat model, Sīla 5, trust ลงนาม, ทีม security 8 องค์ |
| 🌐 **ข้าม workspace & โปรโตคอล** | [TH](cross-workspace/HANDBOOK.th.md) · [`EN`](cross-workspace/HANDBOOK.en.md) | `bwoc peer`, A2A, MQTT — เอเจนต์ข้ามเครื่อง |
| 🖥️ **โฮสต์เอง (Harness)** | [TH](harness/HANDBOOK.th.md) · [`EN`](harness/HANDBOOK.en.md) | รันบน Ollama / OpenAI-compatible ผ่าน `bwoc-harness` |
| 🛰️ **ปฏิบัติการระดับฝูง** | [TH](fleet-ops/HANDBOOK.th.md) · [`EN`](fleet-ops/HANDBOOK.en.md) | รันหลายเอเจนต์: fleet health, supervise, sessions, doctor |
| ❓ **FAQ & แก้ปัญหา** | [TH](faq/HANDBOOK.th.md) · [`EN`](faq/HANDBOOK.en.md) | คำถามบ่อย + ตาราง อาการ→สาเหตุ→วิธีแก้ |

เปิดหาความหมายศัพท์: [TH](glossary.th.md) · [`EN`](glossary.en.md)

---

## BWOC คืออะไร — สรุป 5 ข้อ

1. **หนึ่งโฟลเดอร์ = หนึ่งเอเจนต์** สร้างจากเทมเพลต ตัวตน/วิธีคิด/ทักษะ แยกเก็บเป็นโฟลเดอร์ย่อย (slot)
2. **ไม่ผูกกับ backend ใด** เอเจนต์ตัวเดียวรันได้ทั้ง Claude, Codex, Kimi หรือโมเดลที่โฮสต์เอง (Ollama / endpoint แบบ OpenAI-compatible) ผ่าน `bwoc-harness` — ไม่เอนเอียงเจ้าใด
3. **ออกแบบบนหลักคิดที่ชัดเจน** มีศัพท์เฉพาะบ้าง แต่ทุกคำแปลเป็นเรื่องวิศวกรรมจริง — **ไม่ต้องรู้ศัพท์ก็ใช้งานได้** อยากรู้ค่อยเปิด glossary
4. **หน่วยความจำถาวรที่รู้จักลืม** สะสมความรู้ข้ามรอบงาน และตัดของเก่าอย่างมีวินัย (`MEMORY.md` ไม่เกิน 200 บรรทัด)
5. **หลายเอเจนต์ทำงานร่วมกันได้ปลอดภัย** แชร์ task list มีทีม มีคะแนนความเชื่อใจ โดยไม่ชนกัน

---

## วงจรชีวิตที่ทุกอย่างใน BWOC เดินตาม

| ช่วง | เรียกง่าย ๆ | เกิดอะไร | คำสั่ง |
|---|---|---|---|
| 1 | **เกิด** | สร้างตัวตน อ่าน manifest ประกาศความสามารถ | `bwoc new`, `bwoc init` |
| 2 | **อยู่/ทำงาน** | ทำงานจริง สถานะ+หน่วยความจำค่อย ๆ เปลี่ยน | `bwoc spawn`, `chat`, `run`, `supervise` |
| 3 | **ปลด** | เก็บกวาด ปิด branch ตัดหน่วยความจำ ปิดงาน | `bwoc retire`, `stop` |

> เอกสารเฟรมเวิร์กเรียกสามช่วงนี้ด้วยศัพท์บาลี *uppāda · ṭhiti · vaya* — ไม่ต้องจำ ใช้ "เกิด / อยู่ / ปลด" พอ

---

## ข้อตกลงร่วม

- **คู่มือ vs ต้นฉบับ** คู่มือนี้ *ปูพื้นและสรุป* ตัวจริงคือ repo เฟรมเวิร์ก [`bemindlabs/BWOC-Framework`](https://github.com/bemindlabs/BWOC-Framework) (`crates/`, `docs/en`+`docs/th`, สเปก `modules/agent-template/AGENTS.md`) ขัดกันเมื่อไหร่ยึด repo แล้วช่วยแก้คู่มือ
- **สองภาษา อังกฤษหลัก** ทุกหน้าเป็น `*.en.md` (canonical) คู่กับ `*.th.md` แก้พร้อมกันในครั้งเดียว
- **ภาษาเรียบง่าย** ศัพท์เฉพาะอธิบายตอนใช้ครั้งแรก และลิงก์ไป glossary
- **คำสั่ง** เป็นของ CLI `bwoc` — ดูของจริงล่าสุด `bwoc help getting-started` หรือ `bwoc <คำสั่ง> --help`
- **เวอร์ชันขยับตลอด** เช็คเลขปัจจุบันที่ [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md)
- **ลิงก์ห้ามชี้เข้า private workspace** ทุกการอ้างอิงซอร์สเฟรมเวิร์กชี้ไป GitHub repo สาธารณะเท่านั้น ไม่ใช้ path ในเครื่อง เช่น `projects/bwoc-framwork`

---

## ทางลัด

```bash
# แค่อยากใช้งาน
bwoc init ./my-workspace && cd my-workspace && bwoc list

# อยากสร้างเอเจนต์ (template ตรวจจับเองจาก framework clone ใน ancestor ของ cwd)
bwoc new sage --role "research assistant" --target agents/agent-sage
bwoc check agents/agent-sage

# อยากพัฒนาตัวเฟรมเวิร์ก (จาก clone ของ github.com/bemindlabs/BWOC-Framework)
git clone https://github.com/bemindlabs/BWOC-Framework && cd BWOC-Framework
cargo build && cargo test

# เป็นเครื่องที่มา index เนื้อหานี้
cat llms.txt
```
