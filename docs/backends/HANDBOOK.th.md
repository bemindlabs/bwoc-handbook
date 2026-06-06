# คู่มือ Backends — ขับเคลื่อน Agent ผ่าน LLM CLI ทุกตัว

🇬🇧 [English](HANDBOOK.en.md)

สำหรับคนที่ **init workspace, incarnate agent, และเตรียม project ที่เกี่ยวข้องเสร็จแล้ว** และต้องการ **ใช้งานและ configure agent โดยการพิมพ์ prompt ผ่าน LLM CLI ที่มีในมือ** ถ้ายังไม่ได้ทำขั้นตอนก่อนหน้า ให้อ่าน [`../end-user/HANDBOOK.th.md`](../end-user/HANDBOOK.th.md) (workspace และ lifecycle) และ [`../agents/HANDBOOK.th.md`](../agents/HANDBOOK.th.md) (การสร้างและ configure agent) ก่อน

ดูศัพท์เฉพาะ: [`../glossary.th.md`](../glossary.th.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)

---

## สารบัญ

1. [เงื่อนไขก่อนเริ่ม — จุดเริ่มต้นของหน้านี้](#1-เงื่อนไขก่อนเริ่ม--จุดเริ่มต้นของหน้านี้)
2. [Agent เดียว หลาย Backend](#2-agent-เดียว-หลาย-backend)
3. [ตารางอ้างอิง Backend](#3-ตารางอ้างอิง-backend)
4. [เปิดใช้งาน](#4-เปิดใช้งาน)
   - [4.1 bwoc spawn — exec backend ตรงๆ](#41-bwoc-spawn--exec-backend-ตรงๆ)
   - [4.2 bwoc chat — session แบบโต้ตอบ](#42-bwoc-chat--session-แบบโต้ตอบ)
   - [4.3 bwoc run — headless task เดียว](#43-bwoc-run--headless-task-เดียว)
   - [4.4 Override backend ด้วย --backend](#44-override-backend-ด้วย---backend)
5. [Configure ด้วย Prompt](#5-configure-ด้วย-prompt)
6. [ทำให้ Config อยู่ถาวร — Manifest](#6-ทำให้-config-อยู่ถาวร--manifest)
7. [เพิ่ม Backend ใหม่](#7-เพิ่ม-backend-ใหม่)
8. [แก้ปัญหา](#8-แก้ปัญหา)
9. [ดูเพิ่มเติม](#9-ดูเพิ่มเติม)

---

## 1. เงื่อนไขก่อนเริ่ม — จุดเริ่มต้นของหน้านี้

หน้านี้ถือว่าทำสิ่งต่อไปนี้ครบแล้วทั้งสาม:

1. **`bwoc init` เสร็จแล้ว** — มี workspace directory พร้อม `.bwoc/` อยู่ข้างใน
2. **Incarnate agent แล้ว** — สร้าง agent อย่างน้อยหนึ่งตัวด้วย `bwoc new`, ผ่าน `bwoc check`, และเห็นใน `bwoc list`
3. **เตรียม project ที่เกี่ยวข้องแล้ว** — directory ของ project ที่ agent จะทำงานด้วยอยู่พร้อมแล้ว และ verification gate (`lintCmd`, `testCmd` ฯลฯ) ใน `config.manifest.json` ชี้ไปยัง command ที่ใช้งานได้

ถ้าขั้นตอนใดยังไม่เสร็จ ให้ทำก่อน หน้านี้ครอบคลุมเฉพาะสิ่งที่เกิดขึ้น _หลัง_ การ setup เท่านั้น

---

## 2. Agent เดียว หลาย Backend

BWOC agent ทุกตัวมีไฟล์ instruction เพียงไฟล์เดียวที่ทุก LLM backend อ่าน: `AGENTS.md` เขียนเป็น plain Markdown พร้อม `{{camelCase}}` placeholders สำหรับค่าที่ขึ้นกับ backend ไม่มีชื่อ vendor, model ID, หรือ Obsidian syntax ปรากฏในนั้น

ไฟล์ entry-point ของแต่ละ backend (`CLAUDE.md`, `AGY.md` ฯลฯ) เป็น **symlink** ที่ชี้ไปยัง `AGENTS.md` ทั้งหมด การแก้ไขไฟล์ใดก็คือแก้ `AGENTS.md` เมื่อ backend CLI เริ่มทำงานภายใน agent directory มันอ่านไฟล์ชื่อของตัวเอง — แต่ได้รับ instruction ชุดเดียวกัน

Symlink ทั้งหมดที่ template สร้างให้:

```
agents/agent-<name>/
├── AGENTS.md          ← แหล่งความจริงเดียว
├── CLAUDE.md    → AGENTS.md   (Claude Code backend)
├── AGY.md       → AGENTS.md   (Antigravity backend)
├── CODEX.md     → AGENTS.md   (Codex backend)
├── KIMI.md      → AGENTS.md   (Kimi / Moonshot backend)
├── COPILOT.md   → AGENTS.md   (GitHub Copilot backend)
├── OLLAMA.md    → AGENTS.md   (self-hosted Ollama ผ่าน bwoc-harness)
└── OPENAI.md    → AGENTS.md   (OpenAI-compatible endpoint ผ่าน bwoc-harness)
```

ตรวจสอบว่าไฟล์เหล่านี้เป็น symlink ไม่ใช่ไฟล์ปกติ:

```bash
ls -la agents/agent-<name>/CLAUDE.md
# ต้องแสดง: CLAUDE.md -> AGENTS.md
```

ถ้าไฟล์ backend ใดเป็นไฟล์ปกติที่มีเนื้อหาเป็นของตัวเอง แสดงว่า diverge ออกมาแล้วและเป็น neutrality violation แก้โดย:

```bash
cd agents/agent-<name>
rm CLAUDE.md
ln -s AGENTS.md CLAUDE.md
```

ผลของ design นี้: ไม่ต้องดูแล instruction ไฟล์แยกต่างหากสำหรับแต่ละ backend การแก้ไข `AGENTS.md` (หรือ symlink ใดก็ตาม) กระจายไปทุกที่พร้อมกัน configuration เฉพาะ backend อยู่ใน `config.manifest.json` เท่านั้น ได้แก่ model identifier, endpoint URL, และ verification gate command

---

## 3. ตารางอ้างอิง Backend

| Backend | ค่า `--backend` | CLI binary บน PATH | วิธีทำงาน | หมายเหตุ |
|---|---|---|---|---|
| Claude Code | `claude` | `claude` | Vendor CLI อ่าน `CLAUDE.md` → `AGENTS.md` โดยตรง | Default เมื่อไม่ระบุ backend |
| Antigravity | `antigravity` | `agy` | Vendor CLI อ่าน `AGY.md` → `AGENTS.md` โดยตรง | |
| Codex | `codex` | `codex` | Vendor CLI อ่าน `CODEX.md` → `AGENTS.md` โดยตรง | |
| Kimi (Moonshot) | `kimi` | `kimi` | Vendor CLI อ่าน `KIMI.md` → `AGENTS.md` โดยตรง | |
| GitHub Copilot | `copilot` | `copilot` | Vendor CLI อ่าน `COPILOT.md` → `AGENTS.md`; รันในโหมด programmatic (`copilot -p … --no-ask-user`) | เพิ่มใน bwoc เวอร์ชันล่าสุด; ดูข้อควรระวังใน [หัวข้อ 4.4](#44-override-backend-ด้วย---backend) |
| Ollama (self-hosted) | `ollama` | ไม่จำเป็น | `bwoc-harness` เชื่อม agent กับ Ollama API แบบ OpenAI-compatible | Default endpoint `http://localhost:11434/v1`; override ด้วย `baseUrl` ใน `config.manifest.json` |
| OpenAI-compatible | `openai-compatible` | ไม่จำเป็น | `bwoc-harness` เชื่อม agent กับ HTTP endpoint แบบ OpenAI-compatible | **ต้องระบุ `baseUrl` ใน `config.manifest.json`**; `bwoc spawn` ส่ง error ชัดเจนถ้าไม่มี |

**Vendor CLI** (ห้าแถวบน) ต้องติดตั้งและอยู่ใน `PATH` ก่อนจึงจะใช้ได้ BWOC ไม่ได้รวม vendor CLI มาด้วย ถ้า `claude` ไม่อยู่ใน PATH คำสั่ง `bwoc spawn --backend claude` จะ fail ทันทีด้วย binary-not-found error

**Harness backend** (`ollama`, `openai-compatible`) ไม่ต้องใช้ vendor CLI `bwoc-harness` รวมอยู่กับ BWOC และจัดการการสื่อสาร HTTP กับ endpoint เอง flag `--tui` บน `bwoc chat` ใช้ได้เฉพาะ harness backend เท่านั้น

---

## 4. เปิดใช้งาน

มีสามคำสั่งสำหรับขับเคลื่อน agent เลือกตามเป้าหมาย:

| เป้าหมาย | คำสั่ง |
|---|---|
| เปิด session โต้ตอบกับ backend CLI ที่ configured ไว้ | `bwoc chat <agent>` |
| Exec backend CLI ตรงๆ พร้อมส่ง flag เพิ่มเติมได้ | `bwoc spawn --path <agent-dir>` |
| รัน task เดียวแบบไม่โต้ตอบ (script, CI) | `bwoc run <agent> --task "..."` |

### 4.1 `bwoc spawn` — exec backend ตรงๆ

`bwoc spawn` เป็นคำสั่ง low-level ที่สุด มัน exec backend CLI ของ agent ภายใน agent directory เมื่อ launch แล้ว backend CLI รับการควบคุมทั้งหมด

```
bwoc spawn [OPTIONS] [-- <EXTRA>...]
```

| Flag | ทำอะไร | Default |
|---|---|---|
| `--path <PATH>` | Path ไปยัง agent directory | current directory |
| `--backend <BACKEND>` | Backend ที่จะ invoke; ดูตารางใน หัวข้อ 3 | ค่าจาก `config.manifest.json` |
| `--lang <LANG>` | ภาษาสำหรับ output ของ CLI | system default |
| `[-- <EXTRA>...]` | Arguments เพิ่มเติมส่งตรงไปยัง backend CLI | none |

ตัวอย่าง:

```bash
# Spawn จากภายใน agent dir โดยใช้ backend ที่ configured ไว้
cd agents/agent-sage && bwoc spawn

# Spawn จากที่ไหนก็ได้ โดยระบุ agent dir
bwoc spawn --path agents/agent-sage

# Spawn ด้วย backend ที่เจาะจง
bwoc spawn --path agents/agent-sage --backend claude

# ส่ง flag เพิ่มเติมผ่านไปยัง backend CLI
bwoc spawn --path agents/agent-sage -- --verbose
```

### 4.2 `bwoc chat` — session แบบโต้ตอบ

`bwoc chat` ห่อหุ้ม `bwoc spawn` พร้อม option ที่สะดวกสำหรับ tmux, Ghostty, และ full-screen TUI

```
bwoc chat [OPTIONS] <NAME>
```

| Flag | ทำอะไร | Default |
|---|---|---|
| `<NAME>` (required) | ชื่อ agent (`agent-sage` หรือ `sage`) | — |
| `--tmux` | เปิดใน tmux window (หรือเริ่ม session `bwoc-<id>` ถ้ายังไม่ได้อยู่ใน tmux) | off |
| `--ghostty` | เปิดใน Ghostty terminal window ใหม่ (macOS) | off |
| `--tui` | Full-screen ratatui chat client; ใช้ได้เฉพาะ harness backend (`ollama`, `openai-compatible`) | off |
| `--workspace <PATH>` | Workspace root override | auto |
| `--lang <LANG>` | ภาษาสำหรับ output ของ CLI | system default |

ตัวอย่าง:

```bash
# เริ่ม chat session กับ backend ที่ configured ไว้
bwoc chat sage

# เก็บ session ไว้ใน tmux window ให้ detach/reattach ได้
bwoc chat sage --tmux

# เปิดใน Ghostty window แยกต่างหาก (macOS)
bwoc chat sage --ghostty

# Full-screen TUI (ollama หรือ openai-compatible เท่านั้น)
bwoc chat sage --tui

# ใช้ full agent ID
bwoc chat agent-zhongkui
```

### 4.3 `bwoc run` — headless task เดียว

`bwoc run` ส่ง task ไปยัง agent แบบไม่โต้ตอบ backend process รัน เสร็จ task แล้วออก ใช้ใน script และ CI pipeline

```
bwoc run [OPTIONS] --task <TASK> <AGENT>
```

| Flag | ทำอะไร | Default |
|---|---|---|
| `<AGENT>` (required) | ชื่อ agent | — |
| `--task <TASK>` (required) | Task prompt ที่จะส่ง; ต้องเป็น flag ไม่ใช่ positional argument | — |
| `--timeout <SECONDS>` | Kill process และรายงาน failure หลังจากกี่วินาที | none |
| `--json` | Emit `{ agent, backend, task, exit_code, duration_ms, output }` ไปยัง stdout | off |
| `--workspace <PATH>` | Workspace root override | auto |
| `--lang <LANG>` | ภาษาสำหรับ output ของ CLI | system default |

ตัวอย่าง:

```bash
# สั่งให้ agent ทำสิ่งหนึ่งแล้วออก
bwoc run sage --task "สรุป error log ชั่วโมงที่แล้ว"

# พร้อม timeout (วินาที)
bwoc run sage --task "ตรวจหา SQL injection" --timeout 120

# Capture structured output สำหรับ scripting
bwoc run sage --task "สร้าง release notes" --json
bwoc run sage --task "รัน lint" --json | jq '.exit_code'
```

### 4.4 Override backend ด้วย `--backend`

คำสั่ง launch ทั้งสามรับ `--backend` เพื่อ override backend ที่บันทึกไว้ใน `config.manifest.json` ช่วยให้ทดสอบ agent เดียวกับ model หรือ endpoint ต่างกันโดยไม่ต้องแก้ไฟล์ใด

```bash
# agent เดียวกัน backend ต่างกัน
bwoc spawn --path agents/agent-sage --backend ollama
bwoc spawn --path agents/agent-sage --backend codex
bwoc spawn --path agents/agent-sage --backend openai-compatible
```

**ข้อควรระวังสำคัญ — ตรวจสอบ list ล่าสุดก่อน scripting:** ค่าที่ `--backend` รับได้เพิ่มขึ้นเมื่อ framework เพิ่ม backend ใหม่ ตัวอย่างเช่น Copilot backend เพิ่มเข้ามาในเวอร์ชันล่าสุด และ binary เก่าอาจยังไม่มีให้ ให้รันคำสั่งต่อไปนี้เสมอเพื่อดูค่าที่ binary ที่ติดตั้งอยู่รับได้:

```bash
bwoc spawn --help
```

ค่าที่ `--backend` รับได้แสดงอยู่ใน output นั้น ถ้า backend ที่คาดหวังไม่ปรากฏ ให้ update `bwoc` binary

---

## 5. Configure ด้วย Prompt

เมื่ออยู่ใน backend CLI session แล้ว ให้ขับเคลื่อน agent ด้วยภาษาธรรมชาติ agent โหลด identity ทั้งหมดจาก `AGENTS.md` + slot directories (`persona/`, `mindsets/`, `skills/`, `interconnect/`, `memories/`) โดยอัตโนมัติทันทีที่ session เริ่ม ไม่ต้องแนะนำตัว agent ใหม่

สิ่งที่ agent โหลดอัตโนมัติเมื่อเริ่ม session:

- **Identity และ scope** — จาก `AGENTS.md` section 1 (Identity) และ 2 (Task Planning)
- **Mindsets** — จาก `mindsets/*.md` (กรอบความคิดสำหรับการตัดสินใจ)
- **Skills** — จาก `skills/*.md` (ความสามารถที่มีขอบเขตชัดเจน)
- **Memory index** — จาก `MEMORY.md` และ `memories/` (สิ่งที่เรียนรู้ข้ามหลาย session)
- **Verification gates** — `lintCmd`, `formatCmd`, `testCmd`, `buildCmd` จาก `config.manifest.json`
- **Scope และ anti-scope** — งานที่ agent ทำและสิ่งที่ปฏิเสธอย่างชัดเจน

ถาม agent เกี่ยวกับสิ่งเหล่านี้ก่อนมอบงานได้เสมอ:

```
"สรุป scope และ out-of-scope ของคุณให้ฟังหน่อย"
"ลิสต์ skill ที่ประกาศไว้พร้อม maturity level"
"คุณใช้ mindset ไหนเมื่อไม่แน่ใจว่าการเปลี่ยนแปลงนั้นปลอดภัย"
```

สั่งให้ agent configure ตัวเองผ่านภาษาธรรมชาติได้เช่นกัน เพราะ agent ทำงานอยู่ใน directory ของตัวเองและมีสิทธิ์เขียนไฟล์ มันอัปเดตไฟล์ configuration ของตัวเองได้เมื่อขอ ตัวอย่าง:

```
"ตั้ง test gate เป็น `cargo test --workspace` และอัปเดต manifest ด้วย"

"เพิ่ม skill ใหม่ชื่อ web-research ด้วย maturity L1 และ tag domain/research"

"อัปเดต field outOfScope ใน config.manifest.json ให้ชัดขึ้นว่าคุณไม่แก้ไข database schema โดยตรง"

"primary model เปลี่ยนเป็น gemma3:27b แล้ว อัปเดต manifest และยืนยันการเปลี่ยนแปลงด้วย"

"เพิ่ม mindset ชื่อ minimal-footprint พร้อม anchor principle/mattanutata"

"ทบทวน AGENTS.md section 6 และยืนยันว่า verification gate command ทั้งสี่ตรงกับ manifest"
```

หลังจาก agent แก้ไข `AGENTS.md` หรือ `config.manifest.json` ให้รัน `bwoc check` จาก terminal (นอก session) เพื่อยืนยันว่า agent ไม่ได้สร้าง neutrality violation:

```bash
bwoc check agents/agent-<name>
```

---

## 6. ทำให้ Config อยู่ถาวร — Manifest

การเปลี่ยนแปลงระหว่าง session จะ persist ได้ก็ต่อเมื่อบันทึกลงไฟล์ field ที่ควบคุมพฤติกรรม agent ข้ามหลาย session ล้วนอยู่ใน `config.manifest.json` agent แก้ไขไฟล์นี้ระหว่าง session ได้ หรือแก้ตรงๆ ก็ได้

Field ที่เกี่ยวกับ backend configuration มากที่สุด:

| Field | ควบคุมอะไร | ตัวอย่าง |
|---|---|---|
| `primaryModel` | Primary LLM model identifier ของ agent | `"claude-opus-4-7"`, `"gemma3:27b"`, `"gpt-4o"` |
| `fallbackModel` | Fallback model เมื่อ primary ไม่พร้อม | `"claude-sonnet-4-6"` |
| `baseUrl` | Endpoint URL สำหรับ backend `ollama` หรือ `openai-compatible` | `"http://localhost:11434/v1"`, `"http://192.168.1.113:11434/v1"` |
| `lintCmd` | คำสั่ง lint / static analysis (Samvara verification gate) | `"cargo clippy -- -D warnings"` |
| `formatCmd` | คำสั่งตรวจ format (Pahana gate) | `"cargo fmt --check"` |
| `testCmd` | คำสั่ง test (Bhavana gate) | `"cargo test --workspace"` |
| `buildCmd` | คำสั่ง build (Anurakkhana gate) | `"cargo build --release"` |
| `scopeDescription` | "agent นี้ทำ X" หนึ่งบรรทัด; เติมใน `{{scopeDescription}}` ของ `AGENTS.md` | `"review pull request ด้านความปลอดภัย"` |
| `outOfScope` | "agent นี้ไม่ทำ Y" หนึ่งบรรทัด; เติม anti-scope ใน `AGENTS.md` | `"เขียน feature ใหม่, แก้ production config"` |

ตัวอย่าง minimal ของส่วนที่เกี่ยวกับ backend ใน `config.manifest.json`:

```json
{
  "name": "sage",
  "agentId": "agent-sage",
  "version": "2.0",
  "agentRole": "security review assistant",
  "primaryModel": "claude-opus-4-7",
  "fallbackModel": "claude-sonnet-4-6",
  "baseUrl": "http://localhost:11434/v1",
  "lintCmd": "cargo clippy -- -D warnings",
  "formatCmd": "cargo fmt --check",
  "testCmd": "cargo test",
  "buildCmd": "cargo build --release",
  "scopeDescription": "review code ด้านความปลอดภัย",
  "outOfScope": "เขียน feature ใหม่, แก้ infrastructure",
  "memoryPath": "memories/"
}
```

**หมายเหตุ `baseUrl`:**
- สำหรับ `ollama`: optional ถ้าไม่ระบุ `bwoc-harness` ใช้ `http://localhost:11434/v1` เป็น default ตั้งค่าเมื่อ Ollama อยู่ host อื่น
- สำหรับ `openai-compatible`: required `bwoc spawn` ส่ง error ชัดเจนถ้าไม่มี

หลังแก้ `config.manifest.json` (ไม่ว่าจะแก้เองหรือผ่าน prompt) ให้ validate เสมอ:

```bash
bwoc check agents/agent-<name>
```

`bwoc check` ตรวจสอบสามอย่าง:
1. **Backend neutrality** ของ `AGENTS.md` (ไม่มี frontmatter, wikilinks, hardcoded model ID หรือ vendor name)
2. **JSON validity** ของ `config.manifest.json` พร้อม required field ครบ
3. **`MEMORY.md`** ไม่เกิน 200 บรรทัด

Exit code `0` = agent พร้อมใช้ Non-zero = มี violation; อ่าน output แก้ไขก่อนรัน agent อีกครั้ง

---

## 7. เพิ่ม Backend ใหม่

ถ้ามี backend CLI ที่ BWOC ยังไม่ได้ ship symlink มาให้ หรือต้องการเพิ่ม `COPILOT.md` ให้ agent เก่า ใช้คำสั่งเดียว:

```bash
cd agents/agent-<name>
ln -s AGENTS.md COPILOT.md
```

ไม่ต้องแก้โค้ด ไม่ต้องอัปเดต manifest `bwoc spawn --backend copilot` จะพบ `COPILOT.md` ผ่าน symlink และ exec binary `copilot` ภายใน agent directory

Pattern เดียวกันใช้ได้กับ backend ใดก็ตามที่ CLI ของมันอ่านไฟล์ Markdown ชื่อเฉพาะจาก current directory:

```bash
cd agents/agent-<name>
ln -s AGENTS.md <BACKEND>.md
```

หลังเพิ่ม symlink รัน `bwoc check` เพื่อยืนยันว่าไฟล์ใหม่ resolve ไปยัง `AGENTS.md` ไม่ใช่ไฟล์ปกติ:

```bash
bwoc check agents/agent-<name>
```

Spawn ด้วย backend ใหม่:

```bash
bwoc spawn --path agents/agent-<name> --backend copilot
# หรือถ้า --backend value ยังไม่อยู่ใน CLI list ให้ exec binary ตรงๆ:
cd agents/agent-<name> && copilot -p "..." --no-ask-user
```

---

## 8. แก้ปัญหา

| อาการ | วิธีแก้ |
|---|---|
| `bwoc spawn` fail ด้วย "binary not found" | Vendor CLI (`claude`, `agy`, `codex`, `kimi`, `copilot`) ไม่อยู่ใน `PATH` ติดตั้งก่อน BWOC ไม่ได้รวม vendor CLI มาด้วย |
| `bwoc spawn` fail ด้วย "`baseUrl` is required" | Agent ใช้ backend `openai-compatible` และไม่มี `baseUrl` ใน `config.manifest.json` เพิ่มเข้าไป |
| `bwoc spawn --backend <value>` แจ้ง "unrecognized backend" | `bwoc` binary เวอร์ชันเก่ากว่า backend ที่ระบุ รัน `bwoc spawn --help` เพื่อดูค่าที่รับได้ แล้ว update `bwoc` ถ้าจำเป็น |
| Backend CLI เริ่มได้แต่ agent ดูเหมือนสับสนกับ identity ตัวเอง | ไฟล์ backend (`CLAUDE.md` ฯลฯ) อาจ diverge จาก `AGENTS.md` รัน `ls -la agents/agent-<name>/CLAUDE.md` ต้องแสดง `-> AGENTS.md` ถ้าเป็นไฟล์ปกติ ให้ลบแล้วสร้าง symlink ใหม่ |
| `bwoc check` fail หลัง prompt-driven edit | Agent เขียนบางอย่างลง `AGENTS.md` ที่ละเมิด neutrality (YAML frontmatter, wikilinks, hardcoded model ID) อ่าน output ของ `bwoc check` หาบรรทัดนั้น แก้ไข |
| Ollama backend timeout หรือ connection refused | ยืนยันว่า Ollama รันอยู่และเข้าถึงได้ที่ URL ใน `baseUrl` (default `http://localhost:11434/v1`) ทดสอบด้วย `curl http://localhost:11434/v1/models` |
| OpenAI-compatible backend คืน auth error | ตรวจสอบว่า endpoint ที่ `baseUrl` รันอยู่และ API key ที่จำเป็นตั้งค่าใน environment variable ที่ harness คาดหวัง |
| `bwoc chat --tui` แสดง "backend does not support TUI" | TUI ใช้ได้เฉพาะ backend `ollama` และ `openai-compatible` สำหรับ vendor CLI ให้ตัด `--tui` ออกและปล่อยให้ vendor CLI render interface ของตัวเอง |
| Agent ไม่สนใจการเปลี่ยน manifest | ตรวจสอบว่าบันทึก `config.manifest.json` แล้วและ JSON valid (`bwoc check` จะจับ syntax error) Model ID ใน manifest มีผลในการเริ่ม session ครั้งต่อไป session ที่กำลังรันอยู่ยังใช้ model ที่โหลดตอนเริ่ม |

---

## 9. ดูเพิ่มเติม

| หัวข้อ | ลิงก์ |
|---|---|
| End-user handbook — workspace init, agent lifecycle, teams | [`../end-user/HANDBOOK.th.md`](../end-user/HANDBOOK.th.md) |
| Agent author handbook — incarnation, AGENTS.md rule, manifest fields, slots | [`../agents/HANDBOOK.th.md`](../agents/HANDBOOK.th.md) |
| Glossary — ศัพท์เฉพาะและคำ Pali ทั้งหมด | [`../glossary.th.md`](../glossary.th.md) |
| Framework repo root | [github.com/bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) |
| Incarnation spec — `bwoc new` และ template reference แบบละเอียด | [INCARNATION.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/INCARNATION.en.md) |
| Harness spec — `bwoc-harness` internals, Ollama และ OpenAI-compatible protocol | [HARNESS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/HARNESS.en.md) |
| Agent template AGENTS.md spec | [modules/agent-template/AGENTS.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md) |
| VERSION.md (CalVer ปัจจุบัน) | [VERSION.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) |

---

*เมื่อข้อมูลในคู่มือนี้ขัดแย้งกับ [framework repo](https://github.com/bemindlabs/BWOC-Framework) ให้ถือว่า repo ถูกต้อง — คู่มือนี้มี bug รบกวนแจ้งเพื่อแก้ไขด้วย*
