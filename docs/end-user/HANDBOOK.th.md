# คู่มือผู้ใช้งาน BWOC (End-User Handbook)

🇬🇧 [English](HANDBOOK.en.md)

สำหรับคนที่ **ใช้งาน** BWOC — รันเอเจนต์ คุยกับมัน จัดทีม สั่งงาน อ่านผล — โดยไม่ต้องแก้โค้ดเฟรมเวิร์ก
อยากสร้าง/ปรับแต่งเอเจนต์เอง → [`../agents/HANDBOOK.th.md`](../agents/HANDBOOK.th.md)
อยากแก้ตัวเฟรมเวิร์ก → [`../developer/HANDBOOK.th.md`](../developer/HANDBOOK.th.md)
ค้นหาคำศัพท์: [`../glossary.th.md`](../glossary.th.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)

---

## สารบัญ

1. [ติดตั้ง](#1-ติดตั้ง)
2. [Environment และกฎการ resolve](#2-environment-และกฎการ-resolve)
3. [จัดการ workspace](#3-จัดการ-workspace)
4. [สั่งงานเอเจนต์](#4-สั่งงานเอเจนต์)
5. [กล่องข้อความและการสังเกตระบบ](#5-กล่องข้อความและการสังเกตระบบ)
6. [หน่วยความจำ (Memory)](#6-หน่วยความจำ-memory)
7. [ทีมและ task list ร่วม](#7-ทีมและ-task-list-ร่วม)
8. [วงจรชีวิตเอเจนต์](#8-วงจรชีวิตเอเจนต์)
9. [Fleet และ cross-workspace](#9-fleet-และ-cross-workspace)
10. [เอกสารใน workspace](#10-เอกสารใน-workspace)
11. [แก้ปัญหาที่พบบ่อย](#11-แก้ปัญหาที่พบบ่อย)
12. [อยากรู้ลึกต่อ](#12-อยากรู้ลึกต่อ)

---

## 1. ติดตั้ง

BWOC แจกจ่ายเป็น Rust binary — สร้างและติดตั้งจาก clone ของ framework repo

### 1.1 สร้างจาก source

ต้องมี **Rust 1.85+** รองรับ macOS / Linux / Windows

```bash
git clone https://github.com/bemindlabs/BWOC-Framework
cd BWOC-Framework
cargo install --path crates/bwoc-cli
```

ตรวจว่าติดตั้งสำเร็จ:

```bash
bwoc --help
bwoc help getting-started      # คู่มือเริ่มต้นในตัว CLI
```

### 1.2 `bwoc completion` — สร้าง shell completion

**จุดประสงค์.** สร้างสคริปต์ shell completion และติดตั้ง เพื่อให้กด Tab แล้วเติมคำสั่ง `bwoc` ได้อัตโนมัติ

**รูปแบบ.**

```
bwoc completion <SHELL>
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<SHELL>` (จำเป็น) | shell เป้าหมาย ค่าที่รับได้: `bash`, `zsh`, `fish`, `powershell`, `elvish` | — |
| `--lang <LANG>` | ภาษาสำหรับ output (`en` หรือ `th`) ดู [หัวข้อ 2](#2-environment-และกฎการ-resolve) | ค่าจากระบบ |

**ตัวอย่าง.**

```bash
# zsh — เพิ่มใน .zshrc
bwoc completion zsh > ~/.zsh/completions/_bwoc

# bash
bwoc completion bash >> ~/.bash_completion

# fish
bwoc completion fish > ~/.config/fish/completions/bwoc.fish
```

### 1.3 `bwoc update` — เช็กและอัปเกรด

**จุดประสงค์.** เปรียบเทียบ CalVer ของ binary ปัจจุบันกับ release ล่าสุดบน GitHub และ delegate การอัปเกรดให้ package manager

**รูปแบบ.**

```
bwoc update [OPTIONS]
```

| Flag | ทำอะไร | Default |
|---|---|---|
| `--check` | อ่านอย่างเดียว — รายงานว่า version ล้าหลังไหม ไม่เปลี่ยนอะไร | off |
| `--run` | ดำเนิน upgrade command จริง (เช่น `brew upgrade bwoc`) แทนที่จะแค่พิมพ์ออกมา ไม่มีการ self-swap binary ดิบ | off |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**ตัวอย่าง.**

```bash
bwoc update               # แสดง version ปัจจุบัน, ล่าสุด, และคำสั่ง upgrade
bwoc update --check       # อ่านอย่างเดียว: รายงานว่าล้าหลังไหม
bwoc update --run         # รายงานแล้ว execute คำสั่ง upgrade ด้วย
```

---

## 2. Environment และกฎการ resolve

ทุกคำสั่งที่ทำงานกับ workspace หรือแสดงข้อความ รับ option สองตัวนี้ร่วมกัน แทนที่จะอธิบายซ้ำทุกคำสั่ง จึงรวมไว้ที่นี่ครั้งเดียว

### 2.1 การ resolve workspace

เมื่อคำสั่งต้องหา workspace root จะใช้ลำดับความสำคัญนี้ — หยุดที่อันแรกที่เจอ:

1. Flag `--workspace <path>` บน command line
2. Environment variable `BWOC_WORKSPACE`
3. Ancestor walk — ค้นหา parent directory จาก cwd ขึ้นไปจนเจอโฟลเดอร์ที่มี `.bwoc/`
4. Current working directory เป็น fallback สุดท้าย

**ผลในทางปฏิบัติ.** ถ้า `cd` เข้าไปในโฟลเดอร์ใดก็ได้ภายใน workspace, BWOC จะหาเจอ workspace อัตโนมัติ ต้องใช้ `--workspace` เฉพาะตอนจัดการ workspace ที่ไม่ใช่ ancestor ของ cwd ปัจจุบัน (เช่น script ที่จัดการหลาย workspace)

### 2.2 ภาษา output

เมื่อคำสั่งแสดงข้อความที่มนุษย์อ่าน จะใช้ลำดับนี้เพื่อเลือกภาษา:

1. Flag `--lang <LANG>` บน command line
2. Environment variable `BWOC_LANG`
3. System variable `$LANG`
4. `en` เป็น fallback

ค่าที่รองรับปัจจุบัน: `en`, `th` หลายคำสั่งรับ `--json` ด้วย ซึ่งข้ามการเลือกภาษาไปเลย และ emit JSON ตรง ๆ

---

## 3. จัดการ workspace

Workspace คือ root directory ที่เก็บ agent ทั้งหมด ทีม หน่วยความจำ task list และ config ไว้สำหรับโปรเจกต์หนึ่ง สัญลักษณ์บ่งชี้คือโฟลเดอร์ย่อย `.bwoc/`

### 3.1 `bwoc init` — สร้าง workspace

**จุดประสงค์.** Initialize BWOC workspace ที่ path ที่กำหนด เขียน `workspace.toml`, `.bwoc/`, และ directory scaffold

**รูปแบบ.**

```
bwoc init [OPTIONS] [PATH]
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `[PATH]` | Directory ที่จะ initialize | current directory |
| `--force` | Overwrite `workspace.toml` ที่มีอยู่แล้ว ถ้าไม่มี flag นี้คำสั่งจะปฏิเสธการ overwrite | off |
| `--no-runtime` | Scaffold โดยไม่มี runtime/daemon ของ agent เหมาะกับ CI, read-only, หรือ inspection workspace ที่ไม่ต้อง spawn agent workspace ยังคงใช้ได้ (`bwoc check` ผ่าน); แค่ตัด .gitignore pattern ของ daemon-ephemeral ออก | off (fleet default) |
| `--single-agent` | Initialize workspace แบบ single-agent (slot เดียว) แทน multi-agent fleet default scaffold guidance ออกมาเป็นแบบ single-agent | off (fleet default) |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |
| `--json` | Emit `{ workspace, name, version, defaults, runtime, profile, files_created }` แทนรายงานที่มนุษย์อ่าน | off |

**ไฟล์ที่สร้าง.** `workspace.toml`, `.bwoc/agents.toml`, `.bwoc/memory/`, `.bwoc/teams/`, และ `.gitignore` standard entries (ยกเว้นเมื่อใช้ `--no-runtime`)

**หมายเหตุ.** Registry ของ agent อยู่ที่ `.bwoc/agents.toml` — **อย่าแก้มือ** ให้ใช้คำสั่ง CLI จัดการเสมอ การแก้ตรง ๆ อาจทำ registry เสีย

**ตัวอย่าง.**

```bash
bwoc init ./my-project                         # สร้าง workspace ใน directory ใหม่
bwoc init .                                    # initialize directory ปัจจุบัน
bwoc init ./ci-workspace --no-runtime          # CI workspace ไม่มี daemon
bwoc init ./single-ws --single-agent           # workspace สำหรับเอเจนต์เดียว
bwoc init ./existing-ws --force                # overwrite workspace.toml ที่มีอยู่
bwoc init ./new-ws --json                      # parse รายงานไฟล์ที่สร้างใน script
```

### 3.2 `bwoc workspace` — ตรวจสอบ workspace

**จุดประสงค์.** แสดง state ของ workspace รัน validation rules หรือหาและแก้ไข inconsistency มีสาม subcommand

**รูปแบบ.**

```
bwoc workspace [OPTIONS] <COMMAND>
```

| Option | ทำอะไร | Default |
|---|---|---|
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

#### `bwoc workspace info`

แสดง resolved workspace path, ค่า config ที่ parse แล้ว, และจำนวน agent

```bash
bwoc workspace info
```

#### `bwoc workspace validate`

รัน validation rules ทั้งหมดกับ workspace Exit `0` ถ้าผ่านทุกอย่าง; exit `2` ถ้ามี violation

```bash
bwoc workspace validate
```

**ใน CI.** Exit ที่ไม่ใช่ 0 จาก `validate` หมายความว่า workspace มีปัญหาโครงสร้าง แก้ก่อนรัน agent

#### `bwoc workspace prune`

หา inconsistency: phantom registry entries (entry ใน `agents.toml` ที่ไม่มี directory) และ orphan directories (directory ที่ดูเหมือน agent แต่ไม่ได้ register) ค่าเริ่มต้นเป็น dry run

| Flag | ทำอะไร | Default |
|---|---|---|
| `--apply` | แก้ไข inconsistency ที่ปลอดภัยจริง ๆ (ลบ phantom entries, register orphan ที่ดูถูกต้อง) ถ้าไม่มี flag นี้ output จะแสดงข้อมูลอย่างเดียว | off (dry run) |

```bash
bwoc workspace prune             # แสดงปัญหาอย่างเดียว
bwoc workspace prune --apply     # แก้ไขปัญหาที่ปลอดภัย
```

### 3.3 `bwoc list` — แสดงรายชื่อ agent ที่ register ไว้

**จุดประสงค์.** แสดง agent ทุกตัวที่ register ใน workspace พร้อม filter และ sort ที่หลากหลาย

**รูปแบบ.**

```
bwoc list [OPTIONS]
```

| Flag | ทำอะไร | Default |
|---|---|---|
| `--workspace <PATH>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--status <STATUS>` | Filter ตาม status (exact match) ค่าทั่วไป: `active`, `stopped`, `retired` | (ทั้งหมด) |
| `--backend <BACKEND>` | Filter ตาม backend ค่าที่รับ: `claude`, `antigravity`, `codex`, `kimi`, `ollama`, `openai-compatible` | (ทั้งหมด) |
| `--running` | Filter เฉพาะ agent ที่ daemon กำลังรันอยู่จริง (PID file + signal-0 check) | off |
| `--inbox-pending` | Filter เฉพาะ agent ที่มีข้อความรออยู่ใน inbox อย่างน้อยหนึ่งข้อความ | off |
| `--sort <SORT>` | Sort key ค่าที่รับ: `id`, `inbox`, `incarnated`, `backend` | ลำดับการ insert ใน registry |
| `--count` | พิมพ์แค่จำนวน agent ที่ match เป็นตัวเลขเดียว กับ `--json` จะ emit `{"count": N}` | off |
| `--names-only` | พิมพ์ agent ID เปล่า ๆ บรรทัดละอัน เหมาะสำหรับ shell loop: `for name in $(bwoc list --names-only); do ...` กับ `--json` จะ emit `{"names": [...]}` `--count` ชนะถ้าตั้งทั้งคู่ | off |
| `--json` | Emit JSON แทน table ที่มนุษย์อ่าน | off |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**ตัวอย่าง.**

```bash
bwoc list                                        # agent ทั้งหมด ดู table
bwoc list --status active                        # เฉพาะ active
bwoc list --backend ollama                       # เฉพาะ agent ที่ใช้ Ollama
bwoc list --running                              # เฉพาะ agent ที่ daemon ทำงานอยู่
bwoc list --inbox-pending --sort inbox           # มีข้อความรอ เรียงตาม inbox
bwoc list --count                                # แค่นับ
bwoc list --names-only --status active           # ID เปล่า ๆ สำหรับ script
for name in $(bwoc list --names-only --running); do bwoc ping "$name"; done
bwoc list --json | jq '.[].id'
```

### 3.4 `bwoc doctor` — วินิจฉัยและซ่อม

**จุดประสงค์.** ตรวจสอบ environment และ workspace หาปัญหาทั่วไป (binary หาย symlink เสีย manifest ไม่ถูกต้อง) และซ่อมจุดที่ปลอดภัยโดยอัตโนมัติถ้าต้องการ

**รูปแบบ.**

```
bwoc doctor [OPTIONS]
```

| Flag | ทำอะไร | Default |
|---|---|---|
| `--workspace <PATH>` | Workspace ที่จะวินิจฉัย ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--auto` | พยายามซ่อมปัญหาที่ปลอดภัยอัตโนมัติ (directory หาย, symlink เสีย) | off |
| `--json` | Emit JSON แทนรายการที่มนุษย์อ่าน | off |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**หมายเหตุ.** `bwoc doctor` เป็นขั้นตอนแรกที่ดีที่สุดเมื่อมีอะไรผิดพลาด รันโดยไม่มี `--auto` ก่อนเพื่อดูว่าเจออะไร แล้ว re-run พร้อม `--auto` เพื่อแก้จุดที่ปลอดภัย

**ตัวอย่าง.**

```bash
bwoc doctor                    # ตรวจและรายงาน ไม่เปลี่ยนอะไร
bwoc doctor --auto             # ตรวจและซ่อมจุดที่ปลอดภัย
bwoc doctor --json             # output แบบ machine-readable
bwoc doctor --auto --json      # ซ่อมแล้ว emit JSON
```

---

## 4. สั่งงานเอเจนต์

มีสี่วิธีในการ interact กับเอเจนต์ ขึ้นอยู่กับว่าอยากคุยสด, รันงานเดี่ยวแบบ headless, เข้าถึง backend โดยตรง, หรือส่งข้อความแบบ async

### ภาพรวม backend

ทุก agent ประกาศ `backend` ใน manifest BWOC รองรับค่าเหล่านี้:

| Backend | รันอะไร | หมายเหตุ |
|---|---|---|
| `claude` | Claude CLI | Default exec binary `claude` |
| `antigravity` | Antigravity CLI | Exec binary `antigravity` |
| `codex` | Codex CLI | Exec binary `codex` |
| `kimi` | Kimi CLI | Exec binary `kimi` |
| `ollama` | `bwoc-harness` → Ollama | Exec `bwoc-harness` ด้วย endpoint `http://localhost:11434/v1` หรือ `baseUrl` จาก `config.manifest.json` ถ้ามี |
| `openai-compatible` | `bwoc-harness` → endpoint ใดก็ได้ | Exec `bwoc-harness` และส่ง `baseUrl` จาก `config.manifest.json` เป็น `--endpoint` **`baseUrl` บังคับ** สำหรับ backend นี้ — spawn จะ error ชัดเจนถ้าไม่มี |

`baseUrl` คือ field ใน `config.manifest.json` ของ agent ตั้งค่าเพื่อ override default endpoint สำหรับ `ollama` หรือระบุ endpoint ที่ต้องการสำหรับ `openai-compatible`

### 4.1 `bwoc chat` — คุยแบบ interactive

**จุดประสงค์.** เปิด live interactive chat session กับ agent โดย exec backend CLI ที่ config ไว้ในโฟลเดอร์ของ agent โมเดลและ endpoint resolve จาก manifest ของ agent

**รูปแบบ.**

```
bwoc chat [OPTIONS] <NAME>
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<NAME>` (จำเป็น) | ชื่อ agent รับทั้ง full ID (`agent-foo`) และชื่อเปล่า (`foo`) | — |
| `--tmux` | รัน `bwoc spawn` ใต้ tmux เพิ่ม window เมื่ออยู่ใน tmux session อยู่แล้ว หรือ auto-start session `bwoc-<id>` ถ้ายังไม่มี | off |
| `--ghostty` | เปิด Ghostty terminal window ใหม่แทนการ exec ใน shell ปัจจุบัน macOS เท่านั้น | off |
| `--tui` | Full-screen ratatui chat client spawn `bwoc-harness --chat` และ render `chat_proto` event stream รองรับเฉพาะ backend `ollama` และ `openai-compatible`; backend อื่นแสดง hint แล้ว fallback ไป exec | off |
| `--workspace <WORKSPACE>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**หมายเหตุ.** `bwoc chat` เป็นคำสั่ง exec — เมื่อ backend CLI เริ่มทำงาน BWOC ส่งการควบคุมให้มันแล้ว ใช้ `--tmux` เพื่อเก็บ session ไว้ใน tmux window เพื่อ detach และ reattach ได้ ใช้ `--ghostty` บน macOS สำหรับ terminal window แยก ใช้ `--tui` สำหรับประสบการณ์ full-screen กับ agent ที่ใช้ `ollama` หรือ `openai-compatible`

**ตัวอย่าง.**

```bash
bwoc chat sage                          # exec backend ตรงใน shell นี้
bwoc chat sage --tmux                   # เปิดใน tmux window
bwoc chat sage --ghostty                # เปิดใน Ghostty window ใหม่ (macOS)
bwoc chat sage --tui                    # full-screen TUI (harness backend เท่านั้น)
bwoc chat agent-zhongkui                # ใช้ full agent ID
```

### 4.2 `bwoc run` — รันงานเดียวแบบ headless

**จุดประสงค์.** ส่ง task prompt ให้ agent แบบ non-interactive และ capture ผลลัพธ์ ออกแบบมาสำหรับ script, CI pipeline, และ automation ที่ไม่ต้องการ session แบบ interactive

**รูปแบบ.**

```
bwoc run [OPTIONS] --task <TASK> <AGENT>
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<AGENT>` (จำเป็น) | ชื่อ agent (`agent-foo` หรือ `foo`) | — |
| `--task <TASK>` (จำเป็น) | Task prompt ที่จะส่งให้ agent **ต้องใช้เป็น flag** — ไม่ใช่ positional argument | — |
| `--timeout <TIMEOUT>` | Kill process ของ agent และรายงาน timeout ถ้ารันเกินจำนวนวินาทีที่กำหนด | ไม่มี (ไม่มี timeout) |
| `--json` | Emit `{ agent, backend, task, exit_code, duration_ms, output }` ไปที่ stdout | off |
| `--workspace <WORKSPACE>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**หมายเหตุ.** Flag `--task` บังคับ อย่าส่ง task เป็น positional argument — จะไม่ถูกรับรู้ Output `--json` มี `exit_code` และ `duration_ms` ที่มีประโยชน์สำหรับ CI pass/fail gate

**ตัวอย่าง.**

```bash
bwoc run sage --task "สรุป error log ชั่วโมงล่าสุด"
bwoc run sage --task "ตรวจไฟล์นี้หา SQL injection" --timeout 120
bwoc run sage --task "สร้าง release notes" --json
bwoc run sage --task "รัน lint" --json | jq '.exit_code'
```

### 4.3 `bwoc spawn` — exec backend โดยตรง

**จุดประสงค์.** Exec backend CLI ที่ config ไว้ของ agent ตรง ๆ ในโฟลเดอร์ของมัน พร้อม argument เพิ่มเติมถ้าต้องการ เป็น primitive ระดับต่ำกว่า `bwoc chat` — ไม่มีการ resolve โมเดลจาก manifest; backend CLI รับการควบคุมเลย

**รูปแบบ.**

```
bwoc spawn [OPTIONS] [-- <EXTRA>...]
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `[-- <EXTRA>...]` | Argument เพิ่มเติมที่ส่งตรง ๆ ไปยัง backend CLI หลัง separator `--` | ไม่มี |
| `--path <PATH>` | Path ไปยัง agent directory | current directory |
| `--backend <BACKEND>` | Backend CLI ที่จะ invoke ค่าที่รับ: `claude`, `antigravity`, `codex`, `kimi`, `ollama`, `openai-compatible` | `claude` |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**หมายเหตุ.** `bwoc spawn` คือ primitive ที่ `bwoc chat` และ `bwoc run` สร้างอยู่บนมัน ใช้เมื่อต้องการส่ง flag ตรง ๆ ไปยัง backend CLI `--path` default เป็น current directory ดังนั้น `cd agents/agent-sage && bwoc spawn` หรือใช้ `--path` จากที่ไหนก็ได้

**ตัวอย่าง.**

```bash
cd agents/agent-sage && bwoc spawn                        # backend default จาก manifest
bwoc spawn --path agents/agent-sage --backend ollama      # override backend
bwoc spawn --path agents/agent-sage -- --verbose          # ส่ง flag เพิ่มไปยัง backend CLI
bwoc spawn --path agents/agent-sage --backend openai-compatible   # ต้องมี baseUrl ใน manifest
```

### 4.4 `bwoc send` — ส่งข้อความเข้ากล่อง inbox

**จุดประสงค์.** Append message envelope ลงใน `.bwoc/inbox.jsonl` ของ agent agent จะอ่านมันเมื่อเริ่ม session ครั้งถัดไป (หรือรับ push notification ถ้า daemon รองรับ) นี่คือวิธีมอบงานให้ agent แบบ async ที่ไม่ interactive

**รูปแบบ.**

```
bwoc send [OPTIONS] <TO> <MESSAGE|--file <FILE>>
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<TO>` (จำเป็น) | ผู้รับ รับทั้ง `agent-foo` และ `foo` | — |
| `<MESSAGE>` | ข้อความ quote string ที่มีหลายคำ ใช้ร่วมกับ `--file` ไม่ได้ | — |
| `--file <FILE>` | อ่าน body ข้อความจากไฟล์ เนื้อหาทั้งหมด (ลบ trailing newlines) กลายเป็น message body ใช้ร่วมกับ `<MESSAGE>` ไม่ได้ | — |
| `--from <FROM>` | ตัวตนผู้ส่ง Default คือ `"user"` (มนุษย์ operator) ส่งชื่อ agent (`foo` หรือ `agent-foo`) สำหรับ agent-to-agent messaging ผู้ส่งที่ระบุต้องมีอยู่ใน workspace registry trust gate ของผู้รับ (ถ้าเปิดอยู่) จะ evaluate กับ manifest ของผู้ส่งนี้ | `"user"` |
| `--reply-to <REPLY_TO>` | Thread ข้อความนี้เป็น reply ต่อ envelope ก่อนหน้า ค่าคือ `messageId` ของ envelope นั้น (format: `msg-<slug>-<hex>`) stamp เป็น `replyTo` ใน envelope ใหม่ | ไม่มี |
| `--no-wakeup` | ข้าม tmux `send-keys` wakeup ping ที่พยายามทำ ใช้ใน CI, daemon, และ auto-reply hook เพื่อไม่กระทบ TUI session | off |
| `--workspace <WORKSPACE>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**ไฟล์ที่กระทบ.** Append JSON envelope หนึ่งบรรทัดไปยัง `<agent-dir>/.bwoc/inbox.jsonl`

**ตัวอย่าง.**

```bash
bwoc send sage "ตรวจ database schema ว่า normalize ดีไหม"
bwoc send sage --file /tmp/schema.sql                          # ส่งเนื้อหาไฟล์
bwoc send sage "สรุปสิ่งนี้" --from agent-orchestrator         # agent-to-agent
bwoc send sage "ติดตามผล" --reply-to msg-review-abc123         # threaded reply
bwoc send sage "รัน checks" --no-wakeup                        # CI-safe ไม่ ping tmux
```

---

## 5. กล่องข้อความและการสังเกตระบบ

### 5.1 `bwoc inbox` — อ่านกล่องข้อความของ agent

**จุดประสงค์.** อ่าน message envelope ที่รออยู่ใน `.bwoc/inbox.jsonl` ของ agent รองรับ tail mode, การล้าง, และการนับ

**รูปแบบ.**

```
bwoc inbox [OPTIONS] <AGENT|--all>
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<AGENT>` | ชื่อ agent (`agent-foo` หรือ `foo`) ใช้ร่วมกับ `--all` ไม่ได้ | — |
| `--all` | แสดง inbox ของทุก agent ต่อกัน (แต่ละอันมี header) `--clear` และ `--watch` ถูกปฏิเสธเมื่อใช้ `--all` | off |
| `--limit <LIMIT>` | แสดงเฉพาะ N ข้อความล่าสุด | (ทั้งหมด) |
| `--watch` | Tail mode — block และพิมพ์ envelope ใหม่ที่มาถึง Ctrl-C เพื่อหยุด ใช้ร่วมกับ `--all` ไม่ได้ | off |
| `--clear` | Truncate inbox หลัง print (acknowledge และลบข้อความทั้งหมด) จะ prompt บน TTY | off |
| `--yes` | ข้าม confirmation interactive สำหรับ `--clear` จำเป็นสำหรับ non-TTY | off |
| `--count` | พิมพ์แค่จำนวน envelope เป็นตัวเลขเดียว กับ `--json` จะ emit `{"count": N}` | off |
| `--json` | Emit JSON แทน layout ที่มนุษย์อ่าน | off |
| `--workspace <WORKSPACE>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**ไฟล์ที่กระทบ.** อ่าน `<agent-dir>/.bwoc/inbox.jsonl` กับ `--clear` จะ truncate ไฟล์นั้น

**ตัวอย่าง.**

```bash
bwoc inbox sage                               # อ่านข้อความทั้งหมด
bwoc inbox sage --limit 5                     # 5 ข้อความล่าสุด
bwoc inbox sage --watch                       # live tail Ctrl-C เพื่อหยุด
bwoc inbox sage --count                       # กี่ข้อความที่รืออยู่
bwoc inbox sage --clear --yes                 # อ่านและลบทั้งหมด (scripted)
bwoc inbox --all                              # inbox ทุก agent พร้อมกัน
bwoc inbox sage --json | jq '.[].body'
```

### 5.2 `bwoc log` — ดู daemon log

**จุดประสงค์.** อ่านหรือ stream ไฟล์ log ของ agent daemon (`.bwoc/agent.log`) ซึ่งเก็บ stderr จาก backend process

**รูปแบบ.**

```
bwoc log [OPTIONS] <AGENT>
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<AGENT>` (จำเป็น) | ชื่อ agent (`agent-foo` หรือ `foo`) | — |
| `-f` / `--follow` | Block และ stream บรรทัดใหม่ที่เกิดขึ้น Ctrl-C เพื่อหยุด | off |
| `-n` / `--lines <LINES>` | จำนวนบรรทัดท้ายที่จะ print ก่อน `--follow` block (หรือเป็น output ทั้งหมดเมื่อไม่ follow) | `50` |
| `--clear` | Truncate log file ก่อน print ใช้เมื่ออยากเริ่มสังเกตใหม่ | off |
| `--workspace <WORKSPACE>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**ไฟล์ที่กระทบ.** อ่าน `<agent-dir>/.bwoc/agent.log` กับ `--clear` จะ truncate ไฟล์นั้น

**ตัวอย่าง.**

```bash
bwoc log sage                    # 50 บรรทัดล่าสุด
bwoc log sage -n 100             # 100 บรรทัดล่าสุด
bwoc log sage -f                 # stream บรรทัดใหม่ Ctrl-C เพื่อหยุด
bwoc log sage --clear -f         # ล้าง log เก่าแล้ว stream ใหม่
```

### 5.3 `bwoc status` — สรุปสุขภาพและตัวตน

**จุดประสงค์.** แสดงสรุปสุขภาพต่อ agent และข้อมูล identity จาก manifest อ่านอย่างเดียว

**รูปแบบ.**

```
bwoc status [OPTIONS] [NAME]
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `[NAME]` | ชื่อ agent ถ้าไม่ระบุจะแสดง summary table สำหรับ agent ทั้งหมด ใช้ร่วมกับ `--all` ไม่ได้ | (summary table) |
| `--all` | แสดง detail block ทุก agent (เทียบเท่ากับเรียก single-agent view ทีละตัว) ใช้ร่วมกับ `[NAME]` และ `--banner` ไม่ได้ | off |
| `--banner` | Replay startup liveness banner ของ agent จาก manifest (อ่านอย่างเดียว ไม่ต้องมี daemon) ต้องระบุชื่อ agent ใช้ร่วมกับ `--all` ไม่ได้ | off |
| `--workspace <WORKSPACE>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--json` | Emit JSON แทน layout ที่มนุษย์อ่าน | off |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**ตัวอย่าง.**

```bash
bwoc status                      # summary table ทุก agent
bwoc status sage                 # detail ของ agent เดียว
bwoc status sage --banner        # replay startup liveness banner
bwoc status --all                # detail block ทุก agent
bwoc status --json               # machine-readable output
```

### 5.4 `bwoc sessions` — แสดง session ที่กำลังรัน

**จุดประสงค์.** แสดง agent session ที่กำลังรันอยู่ ตรวจสอบจาก session markers และ process scan

**รูปแบบ.**

```
bwoc sessions [OPTIONS]
```

| Flag | ทำอะไร | Default |
|---|---|---|
| `--idle-secs <IDLE_SECS>` | วินาทีที่ไม่มีกิจกรรมก่อน session จะเปลี่ยนจาก `working` ไปเป็น `idle` | `60` |
| `--workspace <WORKSPACE>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--json` | Emit `{ "sessions": [...] }` แทน table | off |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**ตัวอย่าง.**

```bash
bwoc sessions                         # ใครกำลังรันอยู่
bwoc sessions --idle-secs 120         # นับ idle หลัง 2 นาที
bwoc sessions --json                  # machine-readable session list
```

### 5.5 `bwoc ping` — liveness probe

**จุดประสงค์.** ส่ง `PING` ไปยัง agent ที่รันด้วย `bwoc-agent --serve` ผ่าน Unix socket และตรวจว่าตอบ `PONG` ยืนยันว่า daemon ยังอยู่และรับ connection ได้

**รูปแบบ.**

```
bwoc ping [OPTIONS] <NAME|--all>
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<NAME>` | ชื่อ agent ใช้ร่วมกับ `--all` ไม่ได้ | — |
| `--all` | Ping ทุก agent ใน workspace agent ที่ไม่มี live socket (ไม่ได้รัน) จะถูก label แต่ไม่ทำให้ run fail | off |
| `--workspace <WORKSPACE>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**Exit codes.** Exit non-zero ถ้า agent เดี่ยวที่ระบุไม่ตอบ กับ `--all` agent ที่ไม่ได้รันจะถูกรายงานแต่ไม่ทำให้ exit code เป็น non-zero

**ตัวอย่าง.**

```bash
bwoc ping sage               # probe agent เดียว
bwoc ping --all              # probe ทุก agent ใน workspace
```

### 5.6 `bwoc supervise` — supervisor restart อัตโนมัติ

**จุดประสงค์.** เฝ้าดู daemon process ของ agent และ restart เมื่อมัน crash Exit อย่าง clean เมื่อ status ของ agent ถูกตั้งเป็น `stopped` (ผ่าน `bwoc stop`) ใช้แทน process manager ภายนอกเมื่อต้องการ supervision ที่ BWOC-aware

**รูปแบบ.**

```
bwoc supervise [OPTIONS] <AGENT>
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<AGENT>` (จำเป็น) | ชื่อ agent (`agent-foo` หรือ `foo`) | — |
| `--max-restarts-per-min <N>` | จำนวน restart สูงสุดใน rolling window 60 วินาที เกินกว่านี้ supervisor จะยอมแพ้เพื่อไม่เปลือง CPU ใน crash loop | `10` |
| `--json` | Emit JSON event หนึ่งอันต่อ action (`spawn`, `crash_respawn`, `clean_exit`, `rate_limit_hit`, `signal_stop`) ไปที่ stdout เหมาะสำหรับ monitoring pipeline | off |
| `--workspace <WORKSPACE>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**หมายเหตุ.** รัน `bwoc supervise` ใน background process หรือ tmux window เฉพาะ เมื่อ status ของ agent เปลี่ยนเป็น `stopped` supervisor จะ exit อย่าง clean

**ตัวอย่าง.**

```bash
bwoc supervise sage                            # supervise ด้วย restart limit default
bwoc supervise sage --max-restarts-per-min 3   # limit เข้มงวดขึ้น
bwoc supervise sage --json | tee supervisor.log  # log events ลงไฟล์
```

---

## 6. หน่วยความจำ (Memory)

BWOC มีระบบ memory สองระดับ Tier 1 คือ file-based memory ใน `.bwoc/memory/` — entry สั้น ๆ แบบ YAML/markdown ที่คุณหรือ agent เขียน Tier 2 คือ deep memory store เสริม ขับเคลื่อนโดย external command (config ใน manifest ของ agent เป็น `deepMemoryCmd`) สำหรับ past decisions, session transcript, และความรู้ที่สะสมเกิน 200-line cap ของ `MEMORY.md`

`MEMORY.md` ของแต่ละ agent มีขีดจำกัดที่ 200 บรรทัด ข้อจำกัดนี้ตั้งใจให้ context ของ agent คมและป้องกันไม่ให้ความรู้ล้าสมัยสะสม คำบาลี *vaya* (ดู [`../glossary.th.md`](../glossary.th.md)) บรรยายวินัยการ prune นี้

**รูปแบบ (top-level).**

```
bwoc memory [OPTIONS] <COMMAND>
```

| Flag | ทำอะไร | Default |
|---|---|---|
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

### 6.1 `bwoc memory list` — แสดงรายการ memory entry

**จุดประสงค์.** แสดงรายการ memory entry ทั้งหมดที่ user เขียนใน `.bwoc/memory/`

```bash
bwoc memory list
```

### 6.2 `bwoc memory show` — แสดงเนื้อหา memory entry

**จุดประสงค์.** แสดงเนื้อหา memory entry หนึ่งอัน ไปที่ stdout

```bash
bwoc memory show <KEY>           # แสดง entry เดียว
bwoc memory show --all           # แสดงทุก entry ต่อกัน
```

### 6.3 `bwoc memory put` — เขียน memory entry

**จุดประสงค์.** เขียนหรือ overwrite memory entry ลำดับการเลือก source: inline `[content]` argument > `--file <path>` > stdin

```bash
bwoc memory put <KEY> "เนื้อหา inline"
bwoc memory put <KEY> --file notes.md
echo "เนื้อหา" | bwoc memory put <KEY>
```

### 6.4 `bwoc memory search` — ค้นหาแบบ substring

**จุดประสงค์.** ค้นหา substring ข้ามทุก memory entry (case-insensitive)

```bash
bwoc memory search "database schema"
bwoc memory search "auth token"
```

### 6.5 `bwoc memory rm` — ลบ memory entry

**จุดประสงค์.** ลบ memory entry จะ prompt ยืนยันบน TTY

```bash
bwoc memory rm <KEY>             # prompt ยืนยัน
bwoc memory rm <KEY> --yes       # ข้าม confirmation (scripted use)
```

### 6.6 Tier 2: `bwoc memory wake-up`

**จุดประสงค์.** Emit prior context เมื่อเริ่ม session โดยเรียก `deepMemoryCmd wake-up` Agent ใช้สิ่งนี้เพื่อ preload past decisions ที่เกี่ยวข้องเมื่อเริ่ม session ใหม่

```bash
bwoc memory wake-up
```

### 6.7 Tier 2: `bwoc memory t2-search`

**จุดประสงค์.** ค้นหา past decisions และ notes ใน Tier 2 store โดยเรียก `deepMemoryCmd search "<query>"` ชื่อ `t2-search` เพื่อไม่ชนกับ `search` subcommand ของ Tier 1

```bash
bwoc memory t2-search "ทำไมถึงเปลี่ยนมาใช้ async processing"
```

### 6.8 Tier 2: `bwoc memory mine`

**จุดประสงค์.** Persist session learnings เมื่อสิ้นสุด session โดยเรียก `deepMemoryCmd mine <path> --mode <mode>` Agent รันสิ่งนี้เมื่อสิ้นสุดแต่ละ session เพื่อดึงและเก็บ insight ลงใน Tier 2 store

```bash
bwoc memory mine
```

---

## 7. ทีมและ task list ร่วม

ทีม — เรียกว่า *Saṅgha* ในคำศัพท์ของ framework (ความหมายเรียบง่าย: กลุ่มเอเจนต์ที่ตั้งชื่อไว้ แชร์ task list เดียว และประสานงานเป็นหน่วยเดียว — ดู [`../glossary.th.md`](../glossary.th.md)) — รวมเอเจนต์ไว้รอบ task list ร่วม ใช้ทีมเมื่องานต้องถูกรับ ส่งต่อ และเสร็จโดยเอเจนต์ต่าง ๆ ในวงจร

ไฟล์ทีมบนดิสก์:
- `.bwoc/teams/<ทีม>.toml` — สมาชิกและ metadata (แก้ตรง ๆ เพื่อเพิ่ม/ลบสมาชิก)
- `.bwoc/teams/<ทีม>/tasks.jsonl` — shared task list

### 7.1 `bwoc team create` — สร้างทีม

**จุดประสงค์.** สร้างทีมใหม่พร้อมรายชื่อสมาชิกเริ่มต้น และ register ใน workspace

**รูปแบบ.**

```
bwoc team create <ID> [OPTIONS]
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<ID>` (จำเป็น) | Team identifier (kebab-case เช่น `security-ops`) | — |
| `--members <a,b,c>` | รายชื่อ agent คั่นด้วยจุลภาคที่จะเป็นสมาชิกตอนสร้าง ตั้งสมาชิกได้เลยตอน create | (ว่าง) |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**หมายเหตุ.** ไม่มี subcommand `member-add` หากต้องการเพิ่ม/ลบสมาชิกหลังสร้างแล้ว ให้แก้ `.bwoc/teams/<ทีม>.toml` ตรง ๆ

**ตัวอย่าง.**

```bash
bwoc team create security-ops --members sage,zhongkui,erlang
bwoc team create research-team --members sage
```

### 7.2 `bwoc team list` — แสดงรายชื่อทีมทั้งหมด

**จุดประสงค์.** แสดงทีมใน workspace พร้อมจำนวนสมาชิกและ task

```bash
bwoc team list
```

### 7.3 `bwoc team retire` — ยุบทีม

**จุดประสงค์.** ลบ membership file และ task list ของทีมออกจาก workspace

```bash
bwoc team retire security-ops
```

### 7.4 `bwoc task` — จัดการ task list ร่วม

**จุดประสงค์.** เพิ่ม แสดง claim เสร็จ plan approve และ reject งานบน shared task list ของทีม มีเจ็ด subcommand

**รูปแบบ (top-level).**

```
bwoc task [OPTIONS] <COMMAND>
```

| Flag | ทำอะไร | Default |
|---|---|---|
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

#### `bwoc task add` — เพิ่มงาน

เพิ่มงานใหม่ลงใน task list ของทีม

```
bwoc task add <TEAM> "<DESCRIPTION>" [--requires-plan] [--deps <a,b>]
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<TEAM>` (จำเป็น) | ชื่อทีม | — |
| `<DESCRIPTION>` (จำเป็น) | คำอธิบายงาน (quote string หลายคำ) | — |
| `--requires-plan` | กำหนดให้ agent ที่ claim ต้องส่ง plan และรอ approve ก่อนจะ complete งานได้ ดูขั้นตอน plan-approval ด้านล่าง | off |
| `--deps <a,b>` | รายการ task ID คั่นด้วยจุลภาคที่ต้องเสร็จก่อนงานนี้จะถูก claim ได้ | ไม่มี |

```bash
bwoc task add security-ops "Audit authentication endpoints"
bwoc task add security-ops "Pen-test API gateway" --requires-plan
bwoc task add security-ops "เขียนรายงานสรุป" --deps task-001,task-002
```

#### `bwoc task list` — แสดงรายการงาน

แสดงงานทั้งหมดบน task list ของทีมพร้อม state และผู้ claim ปัจจุบัน

```bash
bwoc task list security-ops
```

#### `bwoc task claim` — claim งาน

ทำเครื่องหมายงานที่ pending และไม่ถูก block ว่า in-progress และมอบหมายให้ agent

```
bwoc task claim <TEAM> <TASK_ID> --as <AGENT>
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<TEAM>` (จำเป็น) | ชื่อทีม | — |
| `<TASK_ID>` (จำเป็น) | Task ID จาก `bwoc task list` | — |
| `--as <AGENT>` (จำเป็น) | Agent ที่ claim งาน | — |

```bash
bwoc task claim security-ops task-001 --as sage
```

งานที่มี `--deps` ที่ยังไม่เสร็จไม่สามารถ claim ได้ — คำสั่งจะรายงานว่า blocked

#### `bwoc task plan` — ส่งหรือดู plan (Pavāraṇā)

ส่ง plan สำหรับงานที่ claim ไว้ แก้ไข plan ที่มีอยู่ หรือดู plan ปัจจุบัน หลักการ *Pavāraṇā* (ดู [`../glossary.th.md`](../glossary.th.md)) กำหนดให้ agent ที่ claim ต้องเชิญการ review ก่อนดำเนินงานที่มีผลกระทบสูง

```
bwoc task plan <TEAM> <TASK_ID> --as <AGENT> [--text "<PLAN>"]
bwoc task plan <TEAM> <TASK_ID>       # แสดง plan ปัจจุบัน (ไม่มี --as หรือ --text)
```

```bash
bwoc task plan security-ops task-001 --as sage --text "1. Map endpoints. 2. Fuzz inputs. 3. รายงาน"
bwoc task plan security-ops task-001        # ดู plan ปัจจุบัน
```

#### `bwoc task approve` — lead approve plan

Team lead (agent อื่นหรือ operator) approve plan ที่ส่งมา เมื่อ approve แล้ว ผู้ claim สามารถ complete งานได้

```
bwoc task approve <TEAM> <TASK_ID> --as <LEAD_AGENT>
```

```bash
bwoc task approve security-ops task-001 --as zhongkui
```

#### `bwoc task reject` — lead reject plan

Team lead reject plan ที่ส่งมา ผู้ claim ต้องแก้ไขและส่งใหม่ผ่าน `bwoc task plan`

```
bwoc task reject <TEAM> <TASK_ID> --as <LEAD_AGENT> [--reason "<WHY>"]
```

```bash
bwoc task reject security-ops task-001 --as zhongkui --reason "Scope แคบเกินไป — รวม auth headers ด้วย"
```

#### `bwoc task complete` — ปิดงาน

ทำเครื่องหมายงาน in-progress ว่าเสร็จแล้ว agent ต้องเป็นผู้ claim ปัจจุบัน

```
bwoc task complete <TEAM> <TASK_ID> --as <AGENT>
```

```bash
bwoc task complete security-ops task-001 --as sage
```

ถ้างานถูกสร้างด้วย `--requires-plan` และ plan ยังไม่ถูก approve คำสั่ง `complete` จะถูกปฏิเสธจนกว่าจะได้รับ approval

### 7.5 ขั้นตอน plan-approval — ตัวอย่างจริง

Flow plan-approval ออกแบบมาสำหรับงานที่มีความเสี่ยงสูง ที่ team lead ต้องตรวจแนวทางก่อนดำเนินการ

```
operator เพิ่มงานด้วย --requires-plan
        |
        v
agent claim งาน ("claim")
        |
        v
agent ส่ง plan ("plan --text ...")
        |
        v
  lead review
    /         \
approve       reject
    |             |
    v             v
agent complete  agent แก้ plan
งานได้         ("plan --text ..." อีกครั้ง)
        |
        v
    complete
```

ลำดับ shell จริง:

```bash
# 1. Operator เพิ่มงานที่ต้อง plan
bwoc task add audit-team "Audit สิทธิ์เข้าถึง production database" --requires-plan

# 2. Agent claim
bwoc task claim audit-team task-003 --as sage

# 3. Agent ส่ง plan
bwoc task plan audit-team task-003 --as sage \
  --text "ขั้น 1: Export access log ขั้น 2: Cross-reference กับ ACL ขั้น 3: Flag anomaly ขั้น 4: รายงาน"

# 4. Lead review และ approve
bwoc task approve audit-team task-003 --as zhongkui

# 5. Agent complete งาน
bwoc task complete audit-team task-003 --as sage
```

---

## 8. วงจรชีวิตเอเจนต์

### 8.1 `bwoc new` — สร้างเอเจนต์ใหม่

**จุดประสงค์.** Incarnate (สร้าง) agent ใหม่จาก template copy template กรอก placeholder register agent ใน `agents.toml` และสร้าง backend symlink คำบาลี *uppāda* (เกิด/อุบัติ — ดู [`../glossary.th.md`](../glossary.th.md)) ตั้งชื่อ phase นี้

**รูปแบบ.**

```
bwoc new [OPTIONS] <NAME>
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<NAME>` (จำเป็น) | ชื่อ agent แบบ kebab-case (เช่น `database-schema`) | — |
| `--target <TARGET>` | Target directory สำหรับ agent ใหม่ | `../agent-<name>/` relative to template |
| `--template <TEMPLATE>` | Path ไปยัง template directory | auto-detect `modules/agent-template/` จาก cwd ancestors |
| `--backend <BACKEND>` | Backend หลักที่ record ใน workspace registry ค่าที่รับ: `claude`, `antigravity`, `codex`, `kimi`, `ollama`, `openai-compatible` | `claude` |
| `--role <ROLE>` | คำอธิบาย role หนึ่งบรรทัด จะ prompt บน TTY ถ้าไม่ระบุ | — |
| `--primary-model <MODEL>` | Primary LLM model identifier จะ prompt ถ้าไม่ระบุบน TTY | — |
| `--fallback-model <MODEL>` | Fallback LLM model identifier ไม่บังคับ | ไม่มี |
| `--memory-path <PATH>` | Directory สำหรับ file-based memory | `memories/` |
| `--sessions-path <PATH>` | Session data directory สำหรับ Tier 2 mining ไม่บังคับจริง ๆ | ไม่มี |
| `--deep-memory-cmd <CMD>` | Tier 2 memory CLI command ไม่บังคับจริง ๆ | ไม่มี |
| `--lint-cmd <CMD>` | Lint command สำหรับ verification gate จะ prompt บน TTY ถ้าไม่ระบุ | — |
| `--format-cmd <CMD>` | Format command สำหรับ verification gate จะ prompt บน TTY ถ้าไม่ระบุ | — |
| `--test-cmd <CMD>` | Test command สำหรับ verification gate จะ prompt บน TTY ถ้าไม่ระบุ | — |
| `--build-cmd <CMD>` | Build command สำหรับ verification gate จะ prompt บน TTY ถ้าไม่ระบุ | — |
| `--worktree-base <PATH>` | Base directory สำหรับ worktree ไม่บังคับจริง ๆ | `/tmp` |
| `--scope <SCOPE>` | Persona scope หนึ่งบรรทัดว่า "agent นี้ทำอะไร" เติม `{{scopeDescription}}` | — |
| `--out-of-scope <TEXT>` | Persona anti-scope หนึ่งบรรทัดว่า "agent นี้ไม่ทำอะไร" | — |
| `--primary-capability <TEXT>` | คำอธิบายยาวขึ้นว่า agent นี้เชี่ยวชาญอะไร เติม `{{primaryCapability}}` ค่า default คือ role ถ้าไม่ระบุ | (ค่า role) |
| `--mindsets <LIST>` | Mindset เริ่มต้นที่จะ seed — ชื่อ kebab-case คั่นด้วยจุลภาค (เช่น `verify-before-act,right-amount`) หนึ่ง stub `.md` ต่อชื่อ | ไม่มี |
| `--skills <LIST>` | Skill เริ่มต้นที่จะ seed — ชื่อ kebab-case คั่นด้วยจุลภาค หนึ่ง stub `.md` ต่อชื่อ | ไม่มี |
| `--json` | Emit `{ agent_id, target, registered_in, symlinks, mindset_stubs, skill_stubs, persona_filled }` แทนรายงานที่มนุษย์อ่าน มีประโยชน์สำหรับ scripted multi-agent setup | off |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**หมายเหตุ.** Template ถูก auto-detect จาก cwd ancestors — ถ้ารัน `bwoc new` จากภายใน framework clone ไหนก็ได้ มันจะหา `modules/agent-template/` เจออัตโนมัติ ระบุ `--template` เฉพาะเมื่อทำงานนอก clone หลัง `bwoc new` รัน `bwoc check <agent>` เพื่อยืนยันว่า agent เป็น backend-neutral

คู่มือการสร้าง agent เต็ม ๆ: [`../agents/HANDBOOK.th.md`](../agents/HANDBOOK.th.md) และ template spec ที่ [github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md)

**ตัวอย่าง.**

```bash
# minimal — interactive prompts กรอกส่วนที่เหลือ
bwoc new sage --role "research assistant" --target agents/agent-sage

# non-interactive เต็ม (CI/scripted)
bwoc new sage \
  --role "research assistant" \
  --backend ollama \
  --primary-model llama3.2 \
  --lint-cmd "cargo clippy" \
  --format-cmd "cargo fmt" \
  --test-cmd "cargo test" \
  --build-cmd "cargo build" \
  --target agents/agent-sage

# สร้างพร้อม mindsets และ skills เริ่มต้น
bwoc new sage --role "analyst" \
  --mindsets "verify-before-act,right-amount" \
  --skills "data-analysis,report-writing" \
  --target agents/agent-sage

# JSON output สำหรับ scripted multi-agent setup
bwoc new sage --role "assistant" --json --target agents/agent-sage
```

### 8.2 `bwoc check` — ตรวจ backend neutrality

**จุดประสงค์.** ตรวจว่า `AGENTS.md` และ manifest ของ agent ไม่มี binding เฉพาะ backend ใด (ไม่มี YAML frontmatter, wikilinks, model ID แบบ hardcode, ชื่อ vendor) นอกจากนี้ยัง validate `config.manifest.json` (valid JSON) และ `MEMORY.md` (≤ 200 บรรทัด)

**รูปแบบ.**

```
bwoc check [OPTIONS] [PATH]
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `[PATH]` | Path ไปยัง agent หรือ template ที่จะ audit ใช้ร่วมกับ `--all` ไม่ได้ | current directory |
| `--all` | Audit ทุก agent ที่ incarnate แล้วใน workspace (fleet-wide audit) ใช้ร่วมกับ `[PATH]` ไม่ได้ | off |
| `--workspace <WORKSPACE>` | Workspace root (ใช้กับ `--all`) ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--json` | Emit JSON แทนรายงานที่มนุษย์อ่าน | off |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**Exit codes.** Exit `0` ถ้า agent ผ่านทุกการตรวจ Exit non-zero ถ้าพบ violation ใช้ใน CI เพื่อ gate agent promotion

**ตัวอย่าง.**

```bash
bwoc check agents/agent-sage          # audit agent เดียว
bwoc check --all                      # audit ทุก agent ใน workspace
bwoc check agents/agent-sage --json   # machine-readable report
```

### 8.3 `bwoc stop` — หยุด agent ชั่วคราว

**จุดประสงค์.** ตั้ง status ของ agent ใน registry เป็น `stopped` โดยไม่ลบไฟล์ daemon process จะถูก terminate ถ้ากำลังรันอยู่ ใช้เมื่อต้องการหยุด agent ชั่วคราว

**รูปแบบ.**

```
bwoc stop [OPTIONS] <NAME|--all>
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<NAME>` | Agent ที่จะหยุด รับ `agent-foo` หรือ `foo` ใช้ร่วมกับ `--all` ไม่ได้ | — |
| `--all` | หยุดทุก agent ที่ไม่ได้ stopped ใน workspace | off |
| `--yes` | ข้าม confirmation interactive จำเป็นสำหรับ non-TTY (scripted) | off |
| `--json` | Emit `{ workspace, agent, daemon_outcome, registry_updated }` ต้องใช้ `--yes` เฉพาะ single-agent | off |
| `--workspace <WORKSPACE>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**ตัวอย่าง.**

```bash
bwoc stop sage                        # หยุด agent เดียว (prompt ยืนยัน)
bwoc stop sage --yes                  # หยุดโดยไม่ prompt
bwoc stop --all --yes                 # หยุดทุก agent ที่กำลังรัน
bwoc stop sage --yes --json           # scripted stop พร้อม JSON output
```

### 8.4 `bwoc start` — เปิด agent ที่หยุดอยู่กลับมา

**จุดประสงค์.** ตั้ง status ของ agent ที่ stopped กลับเป็น `active` และ spawn daemon ถ้าต้องการ

**รูปแบบ.**

```
bwoc start [OPTIONS] <NAME|--all>
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<NAME>` | Agent ที่จะเปิด รับ `agent-foo` หรือ `foo` ใช้ร่วมกับ `--all` ไม่ได้ | — |
| `--all` | เปิดทุก agent ที่ stopped ใน workspace | off |
| `--no-daemon` | แค่ flip status ใน registry เท่านั้น ไม่ spawn `bwoc-agent --serve` | off |
| `--yes` | ข้าม confirmation interactive จำเป็นสำหรับ non-TTY | off |
| `--json` | Emit `{ workspace, agent, daemon_spawned, daemon_pid, ... }` ต้องใช้ `--yes` เฉพาะ single-agent | off |
| `--workspace <WORKSPACE>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**ตัวอย่าง.**

```bash
bwoc start sage                        # เปิด agent เดียว
bwoc start sage --no-daemon            # flip status เท่านั้น ไม่ spawn daemon
bwoc start --all --yes                 # เปิดทุก agent ที่ stopped
bwoc start sage --yes --json           # scripted start พร้อม JSON output
```

### 8.5 `bwoc retire` — ปลด agent ถาวร

**จุดประสงค์.** ลบ agent ออกจาก workspace registry (*vaya* — phase สุดท้ายของ lifecycle ดู [`../glossary.th.md`](../glossary.th.md)) ค่าเริ่มต้นจะลบ agent directory ออกจากดิสก์ด้วย

**รูปแบบ.**

```
bwoc retire [OPTIONS] <NAME>
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `<NAME>` (จำเป็น) | Agent ที่จะปลด รับ `agent-foo` หรือ `foo` | — |
| `--keep-files` | เก็บ agent directory บนดิสก์ไว้ ลบแค่ registry entry ใช้ร่วมกับ `--keep-memory` ไม่ได้ | off |
| `--keep-memory` | เก็บ `memories/` ไว้ในขณะที่ลบส่วนอื่นของ agent directory ให้ปลด agent แต่ยังคงความรู้ที่สะสมไว้ ใช้ร่วมกับ `--keep-files` ไม่ได้ | off |
| `--yes` | ข้าม confirmation interactive จำเป็นสำหรับ non-TTY | off |
| `--json` | Emit `{ workspace, agent, path, mode, registry_updated }` ต้องใช้ `--yes` (scripted destructive ops ต้อง acknowledge ชัดเจน) | off |
| `--workspace <WORKSPACE>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**หมายเหตุ.** ถ้าไม่มี `--keep-files` หรือ `--keep-memory` agent directory ทั้งหมดจะถูกลบ การนี้ไม่สามารถย้อนกลับได้ ใช้ `--keep-memory` เมื่อต้องการให้ความรู้อยู่รอดแต่ identity ของ agent ถูกลบ

**ตัวอย่าง.**

```bash
bwoc retire sage                          # ปลดและลบไฟล์ทั้งหมด (prompt)
bwoc retire sage --yes                    # ไม่ prompt
bwoc retire sage --keep-files             # ลบแค่ registry entry
bwoc retire sage --keep-memory --yes      # ลบไฟล์แต่เก็บ memories/ ไว้
bwoc retire sage --yes --json             # scripted JSON output
```

### 8.6 `bwoc trust` — ดูหรือสร้าง trust credentials

**จุดประสงค์.** อ่าน Kalyāṇamitta-7 trust profile ของ agent (trust claims ที่ประกาศและ `requiredTrust` thresholds จาก manifest) หรือสร้าง ed25519 signing keypair สำหรับ agent

*Kalyāṇamitta* (ดู [`../glossary.th.md`](../glossary.th.md)) หมายถึงกลไก trust-scoring ที่ควบคุมว่า agent ใดสามารถสื่อสารกับใคร

**รูปแบบ.**

```
bwoc trust [OPTIONS] [AGENT]
```

| Argument / Flag | ทำอะไร | Default |
|---|---|---|
| `[AGENT]` | ชื่อ agent (`agent-foo` หรือ `foo`) ไม่บังคับเฉพาะเมื่อใช้ `--keygen --all` | — |
| `--keygen` | สร้าง ed25519 signing keypair แทนการอ่าน profile private key เขียนลง `<agent>/.bwoc/agent.key` (permissions 0600) public key เขียนลง manifest field `signingPublicKey` | off |
| `--all` | กับ `--keygen`: สร้าง keypair สำหรับทุก agent ที่ register (backfill) | off |
| `--force` | กับ `--keygen`: overwrite key ที่มีอยู่ (rotate identity ของ agent) | off |
| `--workspace <WORKSPACE>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--json` | Emit JSON แทน table ที่มนุษย์อ่าน | off |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**ไฟล์ที่กระทบ.** กับ `--keygen`: เขียน `<agent>/.bwoc/agent.key` (0600) และ update `signingPublicKey` ใน `config.manifest.json`

**ตัวอย่าง.**

```bash
bwoc trust sage                           # อ่าน trust profile
bwoc trust sage --keygen                  # สร้าง ed25519 keypair
bwoc trust sage --keygen --force          # rotate key ที่มีอยู่
bwoc trust --keygen --all                 # สร้าง key ทุก agent (backfill)
bwoc trust sage --json                    # machine-readable trust profile
```

---

## 9. Fleet และ cross-workspace

### 9.1 `bwoc fleet health` — ตรวจสุขภาพ fleet governance

**จุดประสงค์.** ตรวจสัญญาณ Aparihāniya-dhamma fleet-governance ทั้ง 7 ตัวใน workspace อ่านอย่างเดียว ไม่เปลี่ยนอะไร *Aparihāniya-dhamma* (หลักไม่เสื่อม — ดู [`../glossary.th.md`](../glossary.th.md)) คือสัญญาณ 7 ตัวที่ BWOC ใช้วัดว่า fleet ทำงานดีหรือเปล่า

**รูปแบบ.**

```
bwoc fleet health [OPTIONS]
```

| Flag | ทำอะไร | Default |
|---|---|---|
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**ตัวอย่าง.**

```bash
bwoc fleet health                  # ตรวจสัญญาณ governance ทั้ง 7
```

### 9.2 `bwoc peer` — ดู peer workspace

**จุดประสงค์.** ดู peer workspace ที่ประกาศใน `routes.toml` ของ workspace นี้ เป็น read-only cross-workspace view สำหรับ monitoring และ collaboration

**รูปแบบ.**

```
bwoc peer [OPTIONS] <COMMAND>
```

| Flag | ทำอะไร | Default |
|---|---|---|
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

#### `bwoc peer list`

แสดง peer ทั้งหมดที่ประกาศใน `routes.toml` ของ workspace นี้

```bash
bwoc peer list
```

#### `bwoc peer status <KEY>`

ดู agent และ open team task ของ peer workspace

```bash
bwoc peer status my-peer-workspace
```

#### `bwoc peer learn <KEY>`

แสดงหรือดูเอกสารที่แชร์จาก allowlist ของ peer (`<peer>/.bwoc/interconnect/shared.toml`)

```bash
bwoc peer learn my-peer-workspace
```

#### `bwoc peer feedback <KEY> <AGENT>`

ให้ feedback ข้าม workspace แบบ signed แก่ peer agent review ถูก deliver เป็น signed envelope (`kind: feedback`) เข้า inbox ของ peer agent; ผู้รับตรวจ signature ของผู้ส่งก่อนรับ

```bash
bwoc peer feedback my-peer-workspace sage
```

### 9.3 `bwoc dashboard` — interactive TUI dashboard

**จุดประสงค์.** Launch full-screen TUI แบบ interactive แสดงรายชื่อ agent พร้อม live status กด `r` เพื่อ refresh

**รูปแบบ.**

```
bwoc dashboard [OPTIONS]
```

| Flag | ทำอะไร | Default |
|---|---|---|
| `--workspace <WORKSPACE>` | Workspace root ดูกฎการ resolve ที่ [หัวข้อ 2.1](#21-การ-resolve-workspace) | auto |
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

**ตัวอย่าง.**

```bash
bwoc dashboard
```

---

## 10. เอกสารใน workspace

BWOC มีระบบจัดการเอกสารเบา ๆ สำหรับ notes, retrospective, และ research doc ภายใน workspace มี alias สามตัว (`notes`, `retro`, `research`) ที่เป็น thin wrapper บน `doc` command สามารถประกาศประเภทเอกสาร custom ได้ใน `.bwoc/doc-kinds.toml`

ทุก document command แชร์ subcommand สามตัวเหมือนกัน: `new`, `list`, `view`

### 10.1 `bwoc notes` — บันทึก workspace

```
bwoc notes new "<TITLE>"          # สร้าง YYYY-MM-DD_<slug>.md ใน notes/
bwoc notes list                   # แสดงรายการ notes ล่าสุดก่อน
bwoc notes view <DATE_PREFIX>     # แสดง note ที่ match date prefix หรือชื่อไฟล์
```

### 10.2 `bwoc retro` — retrospective

```
bwoc retro new "<TITLE>"          # สร้าง YYYY-MM-DD_<slug>.md ใน retrospectives/
bwoc retro list
bwoc retro view <DATE_PREFIX>
```

### 10.3 `bwoc research` — เอกสารวิจัย

```
bwoc research new "<TITLE>"       # สร้าง YYYY-MM-DD_<slug>.md ใน research/
bwoc research list
bwoc research view <DATE_PREFIX>
```

### 10.4 `bwoc doc` — ประเภทเอกสาร custom

**จุดประสงค์.** จัดการเอกสารทุกประเภท — built-in หรือ custom ที่ประกาศใน workspace (ผ่าน `.bwoc/doc-kinds.toml`) ใช้สำหรับ custom kinds; alias ที่มีชื่อด้านบนคือ thin wrapper บน command นี้

**รูปแบบ.**

```
bwoc doc [OPTIONS] <COMMAND>
```

| Flag | ทำอะไร | Default |
|---|---|---|
| `--lang <LANG>` | ภาษาสำหรับ output | ค่าจากระบบ |

```bash
bwoc doc new <KIND> "<TITLE>"         # สร้างเอกสารของประเภทที่กำหนด
bwoc doc list <KIND>                  # แสดงรายการ ล่าสุดก่อน
bwoc doc view <KIND> <DATE_PREFIX>    # แสดงเอกสารที่ match date prefix หรือชื่อไฟล์
```

**ตัวอย่าง.**

```bash
bwoc notes new "Sprint 12 planning"
bwoc notes list
bwoc notes view 2026-06

bwoc retro new "Q2 retrospective"
bwoc retro list

bwoc research new "OAuth2 alternatives"
bwoc research view 2026-05-15

bwoc doc new incident "Database failover 2026-06-05"
bwoc doc list incident
```

---

## 11. แก้ปัญหาที่พบบ่อย

| อาการ | ทำอะไร |
|---|---|
| คำสั่ง `bwoc` หาไม่เจอ | ตรวจว่า `~/.cargo/bin` อยู่ใน `$PATH` รัน `cargo install --path crates/bwoc-cli` จาก framework clone อีกครั้ง |
| `error: workspace not found` หรือคล้ายกัน | รัน `bwoc doctor` ตรวจว่ามี `.bwoc/` ใน/เหนือ directory ปัจจุบัน ตั้ง `BWOC_WORKSPACE` หรือใช้ `--workspace` |
| Agent ไม่ปรากฏใน `bwoc list` | ต้องอยู่ใน workspace เดียวกัน ตรวจว่า `.bwoc/agents.toml` มี agent นั้น ถ้าใช้ `bwoc new` ตรวจว่า `--target` ถูกต้อง |
| `bwoc new` สร้าง agent ผิดที่ | ใช้ `--target agents/agent-<name>` ชัดเจน auto-detect resolve relative ต่อ template ไม่ใช่ cwd ของคุณ |
| `bwoc run` ไม่รับ task text | Task ต้องส่งเป็น `--task "<text>"` ไม่ใช่ positional argument |
| Agent ผูกกับ vendor เดียว | รัน `bwoc check <agent>` neutrality audit จะชี้ model ID แบบ hardcode, ชื่อ vendor, wikilinks, หรือ YAML frontmatter |
| `bwoc spawn` fail ด้วย `baseUrl is required` | Agent ใช้ backend `openai-compatible` ซึ่งต้องการ field `baseUrl` ใน `config.manifest.json` เพิ่มเข้าไป |
| Daemon ค้างหรือตาย | รัน `bwoc log <agent> -f` เพื่ออ่าน error ใช้ `bwoc supervise <agent>` เพื่อให้ restart อัตโนมัติ |
| `bwoc ping` ไม่ได้รับการตอบ | ตรวจว่า daemon กำลังรัน (`bwoc sessions`, `bwoc status <agent>`) daemon ต้องเริ่มด้วย `bwoc-agent --serve` |
| `bwoc workspace validate` exit 2 | มี structural violation อ่าน output แก้ปัญหาที่ระบุ รันใหม่ |
| `bwoc workspace prune` แสดง phantom entries | รัน `bwoc workspace prune --apply` เพื่อลบ phantom registry entries หรือ restore orphan directory |
| `bwoc task complete` ถูกปฏิเสธ | ถ้างานสร้างด้วย `--requires-plan` ต้อง approve plan ก่อน (`bwoc task approve`) |
| Trust gate reject `bwoc send --from <agent>` | Sender agent ต้องมีอยู่ใน workspace registry รัน `bwoc list` เพื่อยืนยัน |
| MEMORY.md ยาวกว่า 200 บรรทัด | จะ fail `bwoc check` prune ไฟล์ให้เหลือ ≤ 200 บรรทัดก่อนรัน fleet-wide audit |
| `bwoc update --run` ไม่ทำอะไร | มัน delegate ให้ package manager ของคุณ (เช่น `brew upgrade bwoc`) ตรวจว่า `bwoc` ติดตั้งผ่าน package manager นั้น |

เจอปัญหาที่คู่มือนี้ตอบไม่ได้ → ดู FAQ ที่ [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md) (ฉบับไทย: [FAQ.th.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/th/FAQ.th.md))

---

## 12. อยากรู้ลึกต่อ

| หัวข้อ | ลิงก์ |
|---|---|
| คำศัพท์เฉพาะ/บาลีทั้งหมด | [`../glossary.th.md`](../glossary.th.md) |
| สร้างและปรับแต่งเอเจนต์ | [`../agents/HANDBOOK.th.md`](../agents/HANDBOOK.th.md) |
| Framework source และ developer guide | [`../developer/HANDBOOK.th.md`](../developer/HANDBOOK.th.md) |
| Agent template spec (AGENTS.md) | [github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md) |
| โครงสร้าง workspace เต็ม | [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/WORKSPACE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/WORKSPACE.en.md) |
| Fleet governance docs | [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FLEET-GOVERNANCE.en.md) |
| Skills system docs | [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) |
| Plugins system docs | [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) |
| FAQ | [github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/FAQ.en.md) |
| VERSION.md (CalVer ปัจจุบัน) | [github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) |
| Framework repo root | [github.com/bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) |
