# FAQ & การแก้ปัญหา

🇬🇧 [English](HANDBOOK.en.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) · Glossary: [../glossary.th.md](../glossary.th.md)

หน้านี้เป็น **companion ฉบับ handbook** ของ FAQ ต้นฉบับ FAQ ที่ canonical — ดูแลพร้อมกับ source code ของ framework — อยู่ที่:

> [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md) (EN) · [FAQ.th.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/th/FAQ.th.md) (TH)

**ถ้ามีความขัดแย้งระหว่างหน้านี้กับ canonical FAQ ให้ยึด canonical FAQ เสมอ — แปลว่าหน้านี้มี bug** companion นี้รวบรวมคำถามที่พบบ่อยในรูปแบบที่อ่านง่าย เพิ่มตาราง troubleshooting และ cross-link ไปยัง handbook ที่เกี่ยวข้อง

---

## สารบัญ

1. [ทั่วไป](#1-ทั่วไป)
2. [Setup และ Workspace](#2-setup-และ-workspace)
3. [Agents และ Backends](#3-agents-และ-backends)
4. [Memory และ Self-Improvement](#4-memory-และ-self-improvement)
5. [ตาราง Troubleshooting](#5-ตาราง-troubleshooting)
6. [ดูเพิ่มเติม](#6-ดูเพิ่มเติม)

---

## 1. ทั่วไป

### ต้องรู้ภาษาบาลีหรือพุทธศาสนาไหมถึงจะใช้ BWOC ได้?

ไม่ต้อง คำบาลีที่ปรากฏใน BWOC — *uppada*, *vaya*, *sila*, *sangha* ฯลฯ — เป็น **label ทางวิศวกรรม** ไม่ใช่คำสอนทางศาสนา แต่ละคำถูกเลือกเพราะ map ตรงกับ concern ซอฟต์แวร์จริงๆ: phase ของ lifecycle, การ prune memory, security precepts, การ coordinate ทีม คุณสามารถเพิกเฉยคำบาลีทั้งหมดได้เลยและคิดเป็น plain English ("birth / live / retire", "prune memory", "อย่า escalate privileges") คำเหล่านี้มีอธิบายใน glossary เมื่อคุณอยากรู้ แต่ไม่ใช่ required reading

ดู [../glossary.th.md](../glossary.th.md) สำหรับความหมายภาษาไทยของทุกคำ และ [README](../README.md) สำหรับ overview ห้าประโยค

### BWOC ผูกติดกับ AI vendor รายใดรายหนึ่งไหม?

ไม่ Backend-neutrality เป็น design constraint หลัก agent directory เดียวกันรันบน Claude, Codex, Kimi, Antigravity, Copilot, Ollama หรือ OpenAI-compatible endpoint ใดก็ได้ ชื่อ vendor, model ID, หรือ syntax เฉพาะ vendor ไม่มีสิทธิ์อยู่ใน `AGENTS.md` และ CLI บังคับใช้ผ่าน `bwoc check`

### เมื่อ handbook กับ framework repo ขัดแย้งกัน ให้ยึดอะไร?

ยึด framework repo เสมอ handbook นี้ — และ companion page ทุกหน้า — เป็นแค่ส่วนที่ orient และ summarize spec ที่แท้จริงอยู่ใน framework source: `crates/`, `docs/en/`, `docs/th/`, และ `modules/agent-template/AGENTS.md` ถ้าเจอ discrepancy แปลว่า handbook มี bug กรุณาช่วย fix ด้วย

### ต้องใช้ Rust version ไหนในการ build BWOC?

Rust 1.85 ขึ้นไป รัน `rustup update stable` ถ้า toolchain เก่าเกินไป ดู walkthrough การ build แบบครบถ้วนที่ [../developer/HANDBOOK.en.md](../developer/HANDBOOK.en.md)

### "CalVer" ใน BWOC หมายความว่าอย่างไร?

BWOC ใช้ versioning สองระบบควบคู่กัน:

- **Cargo SemVer** — field `version` ใน `Cargo.toml` ติดตาม development crate version ตาม conventional SemVer (`MAJOR.MINOR.PATCH`)
- **CalVer release tags** — tag แบบ calendar-based (เช่น `v2.24.0` ที่ `2.24` encode year-week หรือ release cycle) นี่คือสิ่งที่ end-user และ operator ติดตาม

`Software-Version` และ `Document-Version` ปัจจุบันอยู่ใน [VERSION.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) ใน framework repo

---

## 2. Setup และ Workspace

### หลัง build แล้ว command `bwoc` ไม่พบ ต้องเช็คอะไร?

`cargo install` วางไบนารีไว้ที่ `~/.cargo/bin/` ตรวจสอบว่า directory นั้นอยู่ใน `$PATH`:

```bash
echo $PATH | tr ':' '\n' | grep cargo
```

ถ้าไม่มี เพิ่ม `export PATH="$HOME/.cargo/bin:$PATH"` ใน shell profile แล้ว reload

### ทำไม BWOC ไม่เจอ workspace ของฉัน?

BWOC resolve workspace root ด้วย priority order นี้ หยุดที่ match แรก:

1. flag `--workspace <path>` บน command line
2. environment variable `BWOC_WORKSPACE`
3. Ancestor walk — ค้นหา parent directory จาก current working directory จนเจอ directory ที่มี `.bwoc/`
4. Current working directory เป็น last resort

ถ้าอยู่ใน directory tree ของ workspace BWOC มักจะเจอเองอัตโนมัติ ถ้าไม่เจอ รัน `bwoc doctor` — มันระบุ resolution failure และรายงานสิ่งที่เจอและตำแหน่งที่ค้นหา ส่ง `--auto` เพื่อแก้ปัญหาที่ safe โดยอัตโนมัติ:

```bash
bwoc doctor
bwoc doctor --auto
```

ระบุ workspace ตรงๆ สำหรับ command เดียว:

```bash
bwoc list --workspace /path/to/my-workspace
```

### มี workspace หลายอันได้ไหม?

ได้ workspace แต่ละอันเป็น directory tree อิสระ ระบุด้วย `.bwoc/` marker agent ใน workspace A มองไม่เห็น agent ใน workspace B ผ่าน `bwoc list` — เป็น registry แยกกัน ใช้ `bwoc peer` เพื่อ setup cross-workspace view เมื่อต้องการ coordinate ข้าม workspace

### `bwoc doctor --auto` แก้อะไรได้บ้าง?

ปัญหาที่ safe และเป็น mechanical: directory ที่จำเป็นแต่หายไป, symlink เสีย, และปัญหาโครงสร้างที่มี repair ที่ถูกต้องชัดเจน ไม่เขียน config ใหม่ ไม่ลบข้อมูล และไม่ตัดสินใจ behavioral ใดๆ แทนคุณ รันโดยไม่มี `--auto` ก่อนเพื่อ review สิ่งที่จะทำ

---

## 3. Agents และ Backends

### สร้าง agent ใหม่อย่างไร?

ใช้ `bwoc new` ห้าม hand-copy template directory หรือ hand-edit `.bwoc/agents.toml` CLI จัดการ registry registration, placeholder substitution, และการสร้าง symlink แบบ atomic:

```bash
bwoc new sage --role "research assistant" --target agents/agent-sage
```

หลังสร้างแล้ว รัน `bwoc check agents/agent-sage` เพื่อยืนยันว่า agent ผ่าน backend-neutrality audit ดู flag reference ครบถ้วนที่ [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md) และ [../end-user/HANDBOOK.en.md](../end-user/HANDBOOK.en.md)

### ลบ agent อย่างไร?

ใช้ `bwoc retire` มันลบ agent ออกจาก registry และโดย default ลบ directory ด้วย:

```bash
bwoc retire sage --yes
```

ใช้ `--keep-files` เพื่อลบเฉพาะ registry entry แต่เก็บไฟล์ไว้ ใช้ `--keep-memory` เพื่อเก็บเฉพาะ `memories/` directory ไว้ ห้ามลบ agent directory ด้วยมือ — registry จะมี phantom entry ที่ทำให้ command ถัดไปสับสน

### Agent ใหม่ไม่ปรากฏใน `bwoc list` เป็นเพราะอะไร?

เช็คสามอย่างตามลำดับ:

1. **Workspace เดียวกัน** `bwoc list` แสดงเฉพาะ agent ที่ registered ใน workspace ที่ resolve ได้ ยืนยันว่าอยู่ใน directory tree ของ workspace ที่ถูกต้อง หรือส่ง `--workspace` ตรงๆ
2. **`--target` ถูกต้องหรือเปล่า** ถ้ารัน `bwoc new` โดยไม่มี `--target` agent directory ถูกสร้างใกล้กับ template location — ไม่ใช่ workspace root ส่ง `--target agents/agent-<name>` ตรงๆ เสมอ (แก้ไขใน framework PR #200 / v2.00+; binary เก่ายังต้องใช้ `--target`)
3. **รัน `bwoc doctor`** มันแสดง phantom registry entry และ orphan directory ซึ่งมักเผยให้เห็นสิ่งที่ผิดพลาด

### ควรใช้ backend ไหน?

ขึ้นอยู่กับว่ามี CLI ไหนติดตั้งไว้ และใช้ self-hosted model ไหม:

| Backend | วิธีรัน | เมื่อไหรที่ใช้ |
|---|---|---|
| `claude` | exec binary `claude` ตรงๆ | มี Claude Code CLI และใช้บริการของ Anthropic |
| `agy` (antigravity) | exec binary `antigravity` | มี Antigravity CLI |
| `codex` | exec binary `codex` | มี Codex CLI |
| `kimi` | exec binary `kimi` | มี Kimi / Moonshot CLI |
| `copilot` | exec binary `copilot` | มี GitHub Copilot CLI |
| `ollama` | `bwoc-harness` → Ollama | Self-hosted Ollama; ตั้ง `baseUrl` ใน `config.manifest.json` ถ้าไม่ใช้ค่า default `http://localhost:11434/v1` |
| `openai-compatible` | `bwoc-harness` → endpoint ใดก็ได้ | OpenAI-compatible API ใดก็ตาม; **ต้องมี `baseUrl`** ใน `config.manifest.json` |

สำหรับ list backend ที่รองรับปัจจุบันและที่เพิ่มใหม่:

```bash
bwoc spawn --help
```

เพื่อเปลี่ยน backend ขณะ spawn:

```bash
bwoc spawn --path agents/agent-sage --backend ollama
```

ดูรายละเอียด configuration แต่ละ backend ที่ [../backends/HANDBOOK.en.md](../backends/HANDBOOK.en.md)

### Backend `openai-compatible` fail ทันที มีอะไรขาดหายไป?

Backend `openai-compatible` ต้องการ field `baseUrl` ใน `config.manifest.json` ของ agent ถ้าไม่มี `bwoc spawn` จะ return error ชัดเจน ไม่ fail แบบเงียบๆ เพิ่ม field:

```json
{
  "baseUrl": "https://your-endpoint.example.com/v1"
}
```

สำหรับ Ollama ที่ address ไม่ใช่ default ตั้ง `baseUrl` แบบเดียวกัน สำหรับ Ollama local endpoint default (`http://localhost:11434/v1`) `baseUrl` เป็น optional

### Agent รู้สึกว่าผูกติดกับ vendor เดียว — `bwoc check` บอกว่าไม่ neutral ต้องหาอะไร?

`bwoc check` audit `AGENTS.md` ใน 4 category ของ backend-specific binding:

- **YAML frontmatter** — block `---` ... `---` ที่ต้นไฟล์
- **Wikilinks** — syntax `[[...]]`
- **Obsidian callouts** — syntax `> [!type]`
- **Model ID หรือชื่อ vendor ที่ hardcode** ใน behavioral content — เช่น `claude-opus-4`, `Antigravity`, `kimi-k2`

รัน audit และอ่าน output:

```bash
bwoc check agents/agent-sage
bwoc check agents/agent-sage --json   # สำหรับ scripting
```

แทนทุก violation ด้วย backend-neutral equivalents: plain Markdown link แทน wikilink, placeholder `{{primaryModel}}` แทน model ID ที่ hardcode, plain blockquote แทน Obsidian callout ดู failure mode ทุกรูปแบบของ `bwoc check` และวิธีแก้ที่ [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md)

### Agent daemon crash หรือไม่ตอบสนอง ต้องทำอะไร?

เริ่มจาก log:

```bash
bwoc log sage -f
```

นี่ stream stderr ของ daemon ส่วนใหญ่ crash ให้ error message ชัดเจนที่นี่

จากนั้น probe liveness:

```bash
bwoc ping sage
bwoc ping --all
```

ถ้า daemon ดับ ใช้ supervisor เพื่อเปิด automatic crash-restart:

```bash
bwoc supervise sage
```

Supervisor คอย watch daemon process และ restart เมื่อ crash มัน exit cleanly เมื่อคุณรัน `bwoc stop sage`

---

## 4. Memory และ Self-Improvement

### Memory limit คืออะไร และทำไมถึงมี?

`MEMORY.md` ของแต่ละ agent ถูก hard-cap ที่ **200 บรรทัด** `bwoc check` บังคับใช้และ fail ถ้าไฟล์เกิน limit นี้

limit นี้ตั้งใจ การสะสม memory ไม่มีขอบเขตทำให้คุณภาพ context ของ agent ลดลง — entry ที่ stale, redundant, หรือ low-signal ไปเบียดสิ่งที่สำคัญจริงๆ ออก cap 200 บรรทัดบังคับให้ curate: เก็บสิ่งที่ non-obvious และยังเกี่ยวข้อง; ทิ้งสิ่งที่อ่าน code ได้ หรือจับ git history ไว้แล้ว หรือไม่ relevant อีกต่อไป term ของ framework สำหรับ pruning discipline นี้คือ *anicca* (ความไม่เที่ยง) — เป็นเครื่องเตือนใจว่าการยึดถือทุกอย่างไว้เป็น failure mode ไม่ใช่คุณธรรม

### อะไรควรอยู่ใน `MEMORY.md` อะไรควรทิ้ง?

**เก็บ:**
- การตัดสินใจที่ non-obvious และเหตุผลเบื้องหลัง
- แนวทางที่ user ยืนยันหรือแก้ไขอย่างชัดเจน
- ตำแหน่งของ external resource (endpoints, repositories, ชื่อ tool) ที่ไม่ obvious จาก code
- Pattern ที่ค้นพบผ่านการลองผิดลองถูกที่ยังไม่ได้ document ที่ไหน

**ทิ้ง:**
- Code pattern ที่ derive ได้จากการอ่าน code
- Git history (นั่นคือหน้าที่ของ version control)
- Content ที่อยู่ใน `AGENTS.md`, `conventions.md`, หรือ slot doc อยู่แล้ว
- Ephemeral session state ที่ไม่ relevant อีกต่อไป

### Self-improvement ทำงานอย่างไร?

Agent สะสมความรู้ผ่าน loop แบบ study / reflect / practice ที่อธิบายละเอียดใน [../self-improvement/HANDBOOK.en.md](../self-improvement/HANDBOOK.en.md) ท้าย session `bwoc memory mine` persist session learning เข้า Tier 2 deep memory store (เมื่อ configure ไว้) ต้น session ใหม่ `bwoc memory wake-up` preload past context ที่เกี่ยวข้อง

Skill maturity level (`L1` ถึง `L7`) ใน slot file track declared capability — agent author ตั้งเองอย่างซื่อสัตย์ ไม่ใช่ framework คำนวณให้ การ overclaim level ที่สูงกว่าหลักฐานรองรับเป็น threat-model violation ดู maturity scale ครบถ้วนที่ [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md)

---

## 5. ตาราง Troubleshooting

| อาการ | สาเหตุที่เป็นไปได้ | วิธีแก้ |
|---|---|---|
| `bwoc: command not found` | `~/.cargo/bin` ไม่อยู่ใน `$PATH` | เพิ่ม `export PATH="$HOME/.cargo/bin:$PATH"` ใน shell profile; รัน `cargo install --path crates/bwoc-cli` อีกครั้ง |
| `error: workspace not found` หรือคล้ายกัน | ไม่ได้อยู่ใน workspace tree; `.bwoc/` หายหรืออยู่ผิดที่ | รัน `bwoc doctor --auto`; ตั้ง `BWOC_WORKSPACE` หรือส่ง `--workspace` ตรงๆ |
| Agent หายจาก `bwoc list` | Wrong workspace; ไม่ได้ส่ง `--target` ตอน `bwoc new`; registry entry หาย | รัน `bwoc doctor`; ส่ง `--workspace`; สร้าง agent ใหม่ด้วย `--target agents/agent-<name>` |
| `bwoc new` สร้าง agent ที่ directory ผิด | ไม่ได้ตั้ง `--target`; auto-detect resolve relative ต่อ template ไม่ใช่ workspace root | ส่ง `--target agents/agent-<name>` เสมอจาก workspace root |
| `bwoc check` fail: neutrality violation | `AGENTS.md` มี YAML frontmatter, wikilink, Obsidian callout, หรือ model ID / vendor name ที่ hardcode | รัน `bwoc check --json` เพื่อเห็น line ที่ผิด; แก้แต่ละ violation class; รันซ้ำจนผ่าน |
| `bwoc spawn` fail: `baseUrl is required` | Backend ของ agent เป็น `openai-compatible` แต่ `baseUrl` ไม่มีใน `config.manifest.json` | เพิ่ม `"baseUrl": "https://..."` ใน `config.manifest.json` |
| `bwoc check` fail: `MEMORY.md exceeds 200 lines` | Memory index ขยายเกิน cap โดยไม่ prune | Prune bullet ที่ stale ออกจาก `MEMORY.md`; ลบไฟล์ที่ correspond ออกจาก `memories/` |
| Agent daemon crash / ไม่ตอบสนอง | Backend process error; upstream CLI crash | `bwoc log <agent> -f` เพื่ออ่าน error; `bwoc supervise <agent>` สำหรับ auto-restart |
| `bwoc ping` ไม่ได้รับ response | Daemon ไม่ได้รัน | เช็ค `bwoc sessions`, `bwoc status <agent>`; เริ่ม daemon ด้วย `bwoc-agent --serve` หรือผ่าน `bwoc supervise` |
| Build fail: "error[E...]: requires Rust 1.85" | Toolchain เก่าเกินไป | `rustup update stable` |
| หน้า handbook บอกอย่างหนึ่ง GitHub docs บอกอีกอย่าง | Handbook มี bug | Framework repo คือ source of truth; ยึด [canonical FAQ](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md) และพิจารณา fix ด้วย |

---

## 6. ดูเพิ่มเติม

| แหล่งข้อมูล | ลิงก์ |
|---|---|
| Canonical FAQ (source of truth, EN) | [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md) |
| Canonical FAQ (Thai) | [github.com/bemindlabs/BWOC-Framework/blob/main/docs/th/FAQ.th.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/th/FAQ.th.md) |
| คู่มือ End-user (install, workspace, lifecycle) | [../end-user/HANDBOOK.en.md](../end-user/HANDBOOK.en.md) |
| คู่มือ Agent author และ operator | [../agents/HANDBOOK.en.md](../agents/HANDBOOK.en.md) |
| คู่มือ Backends | [../backends/HANDBOOK.en.md](../backends/HANDBOOK.en.md) |
| คู่มือ Self-improvement | [../self-improvement/HANDBOOK.en.md](../self-improvement/HANDBOOK.en.md) |
| Glossary (คำบาลีและคำเฉพาะทั้งหมด) | [../glossary.th.md](../glossary.th.md) |
| VERSION.md (CalVer + SemVer ปัจจุบัน) | [github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) |
| Framework repo root | [github.com/bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) |

---

*handbook นี้ดูแลควบคู่กับ framework เมื่อ behavior ที่อธิบายไว้ขัดแย้งกับ [framework repo](https://github.com/bemindlabs/BWOC-Framework) แปลว่า repo ถูกต้อง — กรุณา open fix ด้วย*
