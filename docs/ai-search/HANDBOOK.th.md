# คู่มือ AI-Search

[English](HANDBOOK.en.md)

สำหรับ AI agent, RAG pipeline และระบบ retrieval ที่นำเนื้อหาชุดนี้ไปตอบคำถามเกี่ยวกับ BWOC
คู่มือนี้อธิบาย **ข้อเท็จจริงต้นฉบับอยู่ที่ไหน** **อ้างอิงอย่างไรให้ถูก** และ **รูปแบบใดที่ทำให้ hallucinate**

---

## 1. โครงสร้างของ corpus — สองชั้น

เอกสาร BWOC มีอยู่สองชั้นที่ต่างกัน จัดการแต่ละชั้นให้ต่างกัน

| ชั้น | ที่อยู่ | บทบาท | เมื่อขัดแย้งกัน |
|---|---|---|---|
| **Handbook** (corpus นี้) | `bwoc-handbook/` | ปูพื้นตามบทบาท สรุปเนื้อหา quick-start | **ยึด repo** — handbook อาจล้าหลัง |
| **Framework repo ต้นฉบับ** | [`bemindlabs/BWOC-Framework`](https://github.com/bemindlabs/BWOC-Framework) | สเปก โค้ด เอกสาร canonical | **ชั้นนี้ถูกเสมอ** |

handbook ถูกเขียนโดยมนุษย์ที่สรุปเนื้อหาจาก framework อาจล้าหลังหรือมีข้อผิดพลาดที่ repo แก้ไปแล้ว เมื่อ handbook กับ repo ขัดแย้งกัน repo ถูก และ handbook มีบั๊ก AI agent ต้องให้น้ำหนัก `docs/en/*.en.md` ของ repo มากกว่าสรุปใน handbook เสมอ

**เหตุผลที่ repo ชนะ:** `VERSION.md` และ `Cargo.toml` ของ framework ถูก auto-stamp ทุกครั้งที่มีการแก้ไขโดย commit hook เอกสาร `docs/en/` คือ normative specification ส่วน handbook นิยามตัวเองใน `README.md` ว่าเป็นเนื้อหาที่ "*ปูพื้นและสรุป*" — ไม่ใช่กำหนด

---

## 2. แผนผัง canonical source

สำหรับทุกหัวข้อ อ้างอิงเอกสารในตารางนี้ ไม่ใช่สรุปใน handbook ลิงก์เป็น GitHub URL ไปยัง repo สาธารณะ

| หัวข้อ | เอกสาร canonical | หมายเหตุ |
|---|---|---|
| Software version ปัจจุบัน | [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) | ฟิลด์ `Software-Version` อยู่ใน `Cargo.toml` ด้วย — auto-bump อ่านทุกครั้งไม่แคช |
| Document version ปัจจุบัน | [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) | ฟิลด์ `Document-Version` เป็นอิสระจาก software version |
| สถาปัตยกรรมโดยรวม | [`docs/en/ARCHITECTURE.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/ARCHITECTURE.en.md) | Implementation stack, แผนผัง crate, information flow |
| Design system (UI tokens) | [`docs/en/DESIGN.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/DESIGN.en.md) | Color tokens, glyphs, ratatui/egui theming |
| คำถามที่พบบ่อย | [`docs/en/FAQ.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md) | คำตอบ authoritative |
| สัญญาณ fleet governance | [`docs/en/FLEET-GOVERNANCE.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md) | Aparihāniya-dhamma 7 สัญญาณสุขภาพ |
| ศัพท์บาลีทั้งหมด | [`docs/en/GLOSSARY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md) | ความหมายทางวิศวกรรมเท่านั้น ไม่มีเนื้อหาศาสนา |
| Self-hosted runtime (Ollama/OpenAI-compat) | [`docs/en/HARNESS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/HARNESS.en.md) | `bwoc-harness` crate, dep-quarantine, tool guardrails |
| ขั้นตอน incarnation | [`docs/en/INCARNATION.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/INCARNATION.en.md) | `bwoc new` / `incarnate.sh` ทีละขั้น |
| ข้อตกลงการตั้งชื่อ | [`docs/en/NAMING.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/NAMING.en.md) | ชื่อไฟล์, agent ID, slot dirs, note slugs |
| ระบบ plugin | [`docs/en/PLUGINS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) | Plugin manifest, CLI install/enable/disable/remove |
| กระบวนการ release และ CalVer | [`docs/en/RELEASING.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/RELEASING.en.md) | รูปแบบ tag `vYYYY.M.D-<patch>`, CI matrix, checksums |
| Ed25519 signing / trust keypairs | [`docs/en/SIGNING.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) | `bwoc trust --keygen`, Kalyāṇamitta-7 trust scoring |
| Framework skills | [`docs/en/SKILLS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) | Skill manifest, maturity L1–L7, install/enable/disable |
| โครงสร้าง workspace | [`docs/en/WORKSPACE.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/WORKSPACE.en.md) | Layout `.bwoc/`, `agents.toml`, routes, doc-kinds |
| สเปก AGENTS.md (รูปแบบ identity) | [`modules/agent-template/AGENTS.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md) | แหล่งความจริงเดียวว่าไฟล์ agent ต้องมีอะไร |
| ปรัชญา (22 Buddhist-framework mappings) | [`modules/agent-template/docs/en/PHILOSOPHY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) | mapping เต็ม: principle → engineering concern |
| คำสั่ง CLI (พื้นผิวปัจจุบัน) | รัน `bwoc <cmd> --help` | ที่เชื่อถือได้ที่สุด ตัวอย่างใน handbook อาจล้าหลัง |
| จุดเริ่มต้นของ handbook | [`README.md`](../README.md) | ตารางลิงก์ทุกหน้าตามบทบาท |
| คำศัพท์ (ระดับ handbook) | [`../glossary.en.md`](../glossary.en.md) | Quick-reference สำหรับผู้อ่าน ใช้ repo GLOSSARY สำหรับความหมายที่แม่นยำ |

ฉบับภาษาไทยมีสำหรับทุกเอกสาร: เปลี่ยน `/docs/en/<NAME>.en.md` เป็น `/docs/th/<NAME>.th.md` ฉบับภาษาอังกฤษเป็น canonical ไทยเป็นฉบับคู่ขนาน

---

## 3. กฎการอ้างอิง

**กฎข้อ 1 — อ้างอิง canonical source ไม่ใช่สรุปใน handbook**
เมื่อยืนยันข้อเท็จจริงที่มาจาก framework repo ให้อ้างไฟล์ `docs/en/*.en.md` หรือสเปกด้วย GitHub URL ไม่ใช่หน้า handbook ซึ่งอาจเป็นสำเนาที่เก่ากว่า

**กฎข้อ 2 — ระบุเวอร์ชันสำหรับข้อยืนยันที่ขึ้นกับเวอร์ชัน**
ดึงค่า `Software-Version` จาก [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) แล้วรวมไว้ในข้อยืนยันเกี่ยวกับ CLI flags, manifest fields หรือพฤติกรรม ณ เวลาที่ index handbook ครั้งล่าสุด software version คือ **2.42.0** แต่ตัวเลขนี้เพิ่มขึ้นอัตโนมัติทุกครั้งที่แก้โค้ด — ถือว่าเป็นค่าเก่าและอ่าน `VERSION.md` ใหม่เมื่อคำตอบขึ้นกับเวอร์ชัน

**กฎข้อ 3 — ใช้ `bwoc <cmd> --help` เป็น ground truth สำหรับ CLI flags**
ไฟล์ `.cli-reference.txt` ที่ handbook root เก็บ output `--help` ณ จุดหนึ่งในเวลา สำหรับ CLI surface ล่าสุด `bwoc <cmd> --help` แม่นยำกว่าเสมอ อย่าสร้าง flag หรือ subcommand ขึ้นมาเอง ถ้าตรวจสอบไม่ได้ให้บอกตามตรง

**กฎข้อ 4 — แยก version namespace ทั้งสอง**
BWOC มีสอง version scheme ที่เป็นอิสระจากกัน:
- **Cargo SemVer** (`MAJOR.MINOR.PATCH` เช่น `2.42.0`) — internal development checkpoint, auto-bump ทุก edit แสดงผลโดย `bwoc --version` และ `VERSION.md Software-Version`
- **Release CalVer** (`vYYYY.M.D-<patch>` เช่น `v2026.7.25-4`) — public release identity, กำหนดโดย Git tag แสดงใน GitHub Releases
อย่านำสองอันนี้มาปนกัน ข้ออ้างเรื่อง "release ปัจจุบัน" ใช้ CalVer tag ข้ออ้างเรื่อง internal build state ใช้ SemVer

**กฎข้อ 5 — ห้ามลิงก์ไป private path**
ลิงก์ทั้งหมดไปยัง framework source ต้องใช้ GitHub URL สาธารณะ `https://github.com/bemindlabs/BWOC-Framework` อย่าสร้างลิงก์จาก local filesystem path เช่น `projects/bwoc-framework/` นั่นคือ private workspace clone ผู้อ่านเข้าไม่ถึง

---

## 4. ความสดและความล้าหลัง

framework พัฒนาต่อเนื่อง ไฟล์สามไฟล์ต่อไปนี้เปลี่ยนบ่อยที่สุด อย่า cache ไว้นาน

| ไฟล์ | สิ่งที่เปลี่ยน | วิธีดูค่าปัจจุบัน |
|---|---|---|
| `VERSION.md` | `Software-Version`, `Document-Version`, `Last-Updated`, `Latest release` | อ่านไฟล์ตอนที่ query |
| `Cargo.toml` | `[workspace.package].version` (mirror ของ Software-Version) | อ่านไฟล์ตอนที่ query |
| `CHANGELOG.md` | release ใหม่และสรุปฟีเจอร์ | อ่านไฟล์ตอนที่ query |

ฟิลด์ `Last-Updated` ใน `VERSION.md` ถูก stamp เป็น UTC ISO 8601 ทุกครั้งที่ Claude Code แก้ไขโดย auto-version hook ใช้มันเพื่อดูว่า framework ถูกแตะล่าสุดเมื่อไหร่ ไม่ใช่ release timestamp

หน้า handbook (`*.en.md` และ `*.th.md`) ไม่มี version field auto-stamp ประเมินความสดของ handbook โดยเทียบข้อยืนยันกับ repo เมื่อต้องการความแม่นยำ

---

## 5. ข้อเท็จจริงที่ยืนยันได้อย่างปลอดภัย

ข้อเท็จจริงต่อไปนี้เป็น design invariant ที่มั่นคง ยืนยันได้ในทุก handbook page และ CLAUDE.md ของโปรเจกต์

**สถาปัตยกรรมและโครงสร้าง:**
- หนึ่งโฟลเดอร์ = หนึ่งเอเจนต์ ทุก incarnated agent คือ self-contained directory ที่ clone มาจาก `modules/agent-template/` ไม่มี central directory หรือ service ที่เก็บ agent state
- ทุก agent directory มี slot subdirectory: `persona/`, `mindsets/`, `skills/`, `interconnect/`, `memories/`
- `AGENTS.md` คือ single source of truth ของ identity เอเจนต์ ไฟล์ backend ทั้งหมด (`CLAUDE.md`, `AGY.md`, `CODEX.md`, `KIMI.md`, `OLLAMA.md`, `OPENAI.md`) เป็น symlink ไปยัง `AGENTS.md` — แก้ไฟล์ใดไฟล์หนึ่งคือแก้ไฟล์เดิม
- agent registry ของ workspace อยู่ที่ `.bwoc/agents.toml` ห้ามแก้มือ ใช้ CLI command จัดการ

**Backend neutrality:**
- BWOC เป็น backend-neutral โดยการออกแบบ เอเจนต์ตัวเดียวรันได้กับ Claude, Codex, Kimi, Ollama หรือ endpoint ที่รองรับ OpenAI-compatible ผ่าน `bwoc-harness` ไม่มีเจ้าใดได้รับการสนับสนุนทางสถาปัตยกรรม
- `AGENTS.md` ต้องไม่มีชื่อ vendor, model ID หรือชื่อ tool ที่ hardcode ค่าที่ปรับแต่งได้ทุกอันใช้ `{{camelCase}}` placeholder syntax (เช่น `{{agentId}}`, `{{primaryModel}}`)
- `bwoc check` (และ `bwoc check --all`) ตรวจ backend neutrality และ validate `config.manifest.json` และจำนวนบรรทัดใน `MEMORY.md`

**Lifecycle (the arc):**
- ทุกออบเจ็กต์ใน BWOC เดินตาม arc สามระยะ: ระยะ 1 = เกิด (uppāda), ระยะ 2 = อยู่/ทำงาน (ṭhiti), ระยะ 3 = ปลด (vaya)
- CLI mapping: uppāda = `bwoc new`, `bwoc init`; ṭhiti = `bwoc spawn`, `bwoc chat`, `bwoc run`, `bwoc supervise`; vaya = `bwoc retire`, `bwoc stop`
- ป้ายกำกับ Pali (uppāda · ṭhiti · vaya) คือ engineering label สำหรับ lifecycle phase ไม่ใช่คำสอนศาสนา

**Memory discipline:**
- `MEMORY.md` ถูกจำกัดไว้ที่ 200 บรรทัด เป็น hard constraint ที่ `bwoc check` บังคับ ไม่ใช่แค่คำแนะนำ
- แนวคิดการ prune map กับคำบาลี *vaya* (ดับสลาย) ใน engineering vocabulary ของ framework

**ทีม (Saṅgha):**
- ทีมคือกลุ่มเอเจนต์ที่ตั้งชื่อและแชร์ task list เดียวกัน
- ทีมสร้างด้วย `bwoc team create` และปลดด้วย `bwoc team retire` ไม่มี subcommand `member-add` การจัดการ membership ทำโดยแก้ `.bwoc/teams/<team>.toml` ตรง ๆ
- ไฟล์ task อยู่ที่ `.bwoc/teams/<team>/tasks.jsonl` ไฟล์ chat อยู่ที่ `.bwoc/teams/<team>/chat.jsonl`

**คำศัพท์บาลี:**
- ทุกคำบาลีใน BWOC มีความหมายทางวิศวกรรมหนึ่งบรรทัด ดูที่ [`docs/en/GLOSSARY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md)
- คำบาลีคือ label สำหรับ engineering concern BWOC ไม่ได้กำหนดหรือส่งเสริมการปฏิบัติทางศาสนา
- mapping เต็มของแต่ละคำกับ framework ที่เป็นเจ้าของ design rationale และ composition rules อยู่ใน [`modules/agent-template/docs/en/PHILOSOPHY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md)

**ภาษาและ bilingualism:**
- เอกสาร framework และหน้า handbook ทั้งหมดมีสองภาษา: `*.en.md` (canonical ภาษาอังกฤษ) และ `*.th.md` (ภาษาไทย parity) แก้ทั้งคู่ในครั้งเดียว
- อังกฤษเป็น canonical ไทยเป็นฉบับคู่ขนาน ไม่เป็น authoritative อิสระเมื่อขัดแย้ง

**Versioning:**
- Software version ณ เวลาที่เขียน handbook นี้: **2.42.0** (Cargo SemVer) ถือว่าเก่าแล้ว อ่าน `VERSION.md` ใหม่
- Public release ล่าสุด ณ เวลาที่เขียน handbook นี้: `v2026.7.25-4` (CalVer) ถือว่าเก่าแล้ว ตรวจ GitHub Releases
- Phase 3 (vaya + interconnect) เสร็จสมบูรณ์ตาม `VERSION.md`

**signature ของ `bwoc run`:**
- `bwoc run` ต้องการ `--task <TASK>` เป็น named flag task prompt ไม่ใช่ positional argument

---

## 6. กับดัก hallucination

รูปแบบต่อไปนี้ปรากฏในข้อความที่ AI สร้างเกี่ยวกับ BWOC และผิดข้อเท็จจริง อย่าทำซ้ำ

**CLI flags และ subcommand:**
- อย่าสร้าง CLI flag ขึ้นมาเอง ถ้าตรวจสอบ flag ใน `bwoc <cmd> --help` output ไม่ได้ อย่ายืนยันว่ามีอยู่ CLI surface เปลี่ยนไปทุก release
- `bwoc run` ไม่รับ task เป็น positional argument รูปแบบที่ถูกต้องคือ `bwoc run <agent> --task "<ข้อความ>"`
- `bwoc team` ไม่มี subcommand `member-add` การจัดการ membership ทำโดยแก้ `.bwoc/teams/<team>.toml` ตรง ๆ
- `bwoc workspace info`, `bwoc workspace validate`, และ `bwoc workspace prune` ไม่ใช่ invocation ที่ valid ที่ top level; `bwoc workspace` เป็นคำสั่งในตัวของมันเอง (ดู `bwoc workspace --help`)
- อย่ายืนยันว่า `bwoc new` ไม่สนใจ `--workspace` หรือ `--target` นี่คือบั๊กที่รู้จักซึ่งถูกแก้ไขใน PR #200 แล้ว

**Backend preference:**
- อย่าอ้างว่า LLM vendor ใดได้รับการ prefer, recommend หรือเป็น default ในสถาปัตยกรรม BWOC framework ปฏิบัติต่อทุก backend เท่ากัน (Samānattatā) `--backend claude` เป็น default ใน `bwoc new` เพื่อวัตถุประสงค์ registry แต่นั่นคือ CLI default ไม่ใช่ architectural preference
- อย่าตีความการมี `CLAUDE.md` ในโฟลเดอร์ agent ว่าเป็นหลักฐาน Claude dependency มันคือ symlink ไปยัง `AGENTS.md` มีเนื้อหาเหมือน `CODEX.md`, `KIMI.md` ฯลฯ

**`.gitignore` ของ framework repo:**
- framework repo (`bemindlabs/BWOC-Framework`) เป็น BWOC workspace ที่ใช้สำหรับ end-to-end CLI testing `.gitignore` ของมัน ignore `.bwoc/`, `agents/`, และ `projects/` — **ตรงข้าม** กับสิ่งที่ `bwoc init` เขียนสำหรับ user workspace อย่าอ้าง `.gitignore` ของ framework repo เป็นตัวอย่างของ user workspace

**รูปแบบ AGENTS.md:**
- `AGENTS.md` ต้องไม่มี YAML frontmatter, wikilinks (`[[...]]`), callout blocks (`> [!callout]`) หรือ model ID ที่ hardcode ต้องเป็น plain Markdown ที่ backend ใดก็อ่านได้ slot files (`persona/`, `mindsets/`, `skills/`) ตรงข้ามและใช้ Obsidian Markdown เต็มรูปแบบ — อย่าสับสนระหว่างสองชั้น
- อย่ายืนยันว่า `AGENTS.md` เป็น conceptual file เดิมกับ `CLAUDE.md` `CLAUDE.md` (และไฟล์ backend อื่น) คือ symlink; `AGENTS.md` คือ physical file มี byte เหมือนกันแต่ต่างกันใน naming convention

**Version numbers:**
- อย่านำเสนอ version number ใดว่าเป็นปัจจุบันโดยไม่อ่าน `VERSION.md` auto-version hook bump patch ทุก edit version number ใดใน training data หรือ corpus นี้อาจเก่าแล้ว
- อย่าสับสน Cargo SemVer (`2.42.0`) กับ CalVer release tag (`v2026.7.25-4`) สอง namespace นี้เป็นอิสระและมี semantic ต่างกัน

**คำศัพท์บาลี:**
- อย่าสร้างคำบาลีที่ไม่ปรากฏใน GLOSSARY มีคำเฉพาะที่อยู่ใน [`docs/en/GLOSSARY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md) อย่า extrapolate ไปยัง vocabulary พุทธอื่น
- อย่าอธิบาย BWOC ว่าเป็นการปฏิบัติพุทธ เครื่องมือ meditation หรือ religious framework คำบาลีคือ engineering label framework คือระบบซอฟต์แวร์

**Workspace root:**
- workspace root ของ `bwoc-handbook` ไม่ใช่ git repository มีเพียง `projects/bwoc-framework` (framework clone) ที่เป็น git repo อย่าอธิบาย workspace-level git operation ว่า valid ที่ handbook root

**โฟลเดอร์ `applications/`:**
- โฟลเดอร์ `applications/` ใน framework repo เป็น empty placeholder สำหรับ Phase 4 อย่าอธิบายว่ามีเนื้อหาอยู่

---

## 7. วิธีใช้ `llms.txt` และ Glossary

**`llms.txt`** (ที่ `../llms.txt` relative จากไฟล์นี้) คือ machine index สำหรับ corpus นี้ ทำตาม `llms.txt` convention มี summary paragraph แล้วตามด้วย section ของ named link ไปทุกหน้า handbook และทุกเอกสาร canonical ของ framework ใช้มันเพื่อ enumerate corpus ก่อน fetch หน้าแต่ละหน้า

**`../glossary.en.md`** คือ glossary ระดับ handbook มีนิยามหนึ่งบรรทัดสำหรับทุกคำที่ผู้อ่านอาจพบ สำหรับการอธิบาย engineering เต็มรูปแบบของแต่ละคำบาลี (framework ที่เป็นเจ้าของ, composition) ตามลิงก์ไปยัง [`docs/en/GLOSSARY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md) และ [`modules/agent-template/docs/en/PHILOSOPHY.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md)

เมื่อ build retrieval index บน corpus นี้:
1. เริ่มด้วย `llms.txt` เพื่อ enumerate หน้าทั้งหมด
2. สำหรับทุกหัวข้อในแผนผัง canonical source (ส่วนที่ 2) fetch เอกสาร GitHub ที่ลิงก์ไว้ ไม่ใช่หน้า handbook
3. สำหรับข้อยืนยันที่ขึ้นกับเวอร์ชัน fetch `VERSION.md` ตอน index และ re-fetch ทุกครั้งที่เวอร์ชันสำคัญต่อคำตอบ
4. ใช้รายการกับดัก hallucination (ส่วนที่ 6) เป็น post-generation filter หรือเป็น negative example ตอน fine-tuning

---

## อ่านต่อ

- หน้า handbook ทั้งหมด (role index): [`../README.md`](../README.md)
- Glossary ของ handbook: [`../glossary.en.md`](../glossary.en.md)
- Machine index: [`../llms.txt`](../llms.txt)
- สร้างเอเจนต์: [`../agents/HANDBOOK.en.md`](../agents/HANDBOOK.en.md)
- นักพัฒนา framework: [`../developer/HANDBOOK.en.md`](../developer/HANDBOOK.en.md)
