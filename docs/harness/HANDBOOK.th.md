# Self-Hosting ด้วย bwoc-harness

🇬🇧 [English](HANDBOOK.en.md)

สำหรับคนที่ต้องการรัน BWOC agent บน **Ollama หรือ endpoint ที่ compatible กับ OpenAI** โดยไม่ต้องติดตั้ง vendor CLI บทนี้อธิบายว่า `bwoc-harness` คืออะไร, configure และ launch อย่างไร, และ harness เพิ่มความสามารถอะไรให้เหนือกว่าการเรียก model ตรงๆ

ดูศัพท์เฉพาะ: [`../glossary.th.md`](../glossary.th.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)

---

## สารบัญ

1. [ทำไมต้อง Self-Host?](#1-ทำไมต้อง-self-host)
2. [Vendor-CLI Backend vs Harness Backend](#2-vendor-cli-backend-vs-harness-backend)
3. [ตั้งค่า Ollama Backend](#3-ตั้งค่า-ollama-backend)
4. [ตั้งค่า OpenAI-Compatible Endpoint](#4-ตั้งค่า-openai-compatible-endpoint)
5. [TUI Chat Client](#5-tui-chat-client)
6. [สิ่งที่ Harness มอบให้](#6-สิ่งที่-harness-มอบให้)
   - [6.1 Agentic loop และ core tools](#61-agentic-loop-และ-core-tools)
   - [6.2 Tool authorization](#62-tool-authorization)
   - [6.3 Task queue](#63-task-queue)
   - [6.4 Telemetry](#64-telemetry)
   - [6.5 Safety guardrails](#65-safety-guardrails)
   - [6.6 Tier-2 deep memory](#66-tier-2-deep-memory)
   - [6.7 chat_proto event stream](#67-chat_proto-event-stream)
7. [Dependency Quarantine](#7-dependency-quarantine)
8. [ดูเพิ่มเติม](#8-ดูเพิ่มเติม)

---

## 1. ทำไมต้อง Self-Host?

Vendor CLI (Claude Code, Codex, AGY, Kimi, Copilot) ดูแลโดยผู้ให้บริการแต่ละราย มันจัดการ model API, agentic loop, และ tool surface ให้คุณ สะดวกเมื่อมี subscription แต่ก็มีข้อจำกัด: ต้องพึ่งพา external service, ไม่สามารถเลือก model ที่ vendor ไม่ได้ให้บริการ, และรันแบบ air-gapped ไม่ได้

การ self-host ด้วย `bwoc-harness` ขจัดข้อจำกัดเหล่านั้น:

- **ใช้ model ใดก็ได้** รัน Ollama ใน local, ชี้ไปยัง Ollama server ระยะไกล, หรือเชื่อมต่อกับ endpoint ใดก็ตามที่พูดภาษา OpenAI Chat Completions API ไม่ว่าจะเป็น open-weight model, private fine-tune, หรือ cloud inference provider ที่ไม่ใช่ native BWOC backend
- **ไม่ต้องมี vendor CLI บน PATH** — `bwoc-harness` มาพร้อมกับ BWOC ไม่ต้องติดตั้งอะไรเพิ่ม
- **Agentic runtime เต็มรูปแบบ** — harness ไม่ใช่แค่ proxy บางๆ มันมี agentic loop, core tools, task queue, telemetry, safety guardrails, และ memory integration ครบเหมือนกัน เพียงแต่คุยกับ endpoint ของคุณแทน
- **Air-gap friendly** — agent บน Ollama local ที่ไม่มี outbound network ก็ยังได้รับ BWOC runtime เต็มรูปแบบ

ข้อแลกเปลี่ยน: คุณรับผิดชอบการรัน model endpoint และทำให้มันเข้าถึงได้ harness จะรายงาน error ชัดเจนถ้าเชื่อมต่อไม่ได้

---

## 2. Vendor-CLI Backend vs Harness Backend

BWOC รองรับ backend สองประเภท การเข้าใจความต่างบอกว่าควรใช้ตัวไหนและทำไม launch flags ถึงต่างกัน

| | Vendor-CLI backend | Harness backend |
|---|---|---|
| **ตัวอย่าง** | `claude`, `antigravity`, `codex`, `kimi`, `copilot` | `ollama`, `openai-compatible` |
| **ค่า `--backend`** | `claude` · `antigravity` · `codex` · `kimi` · `copilot` | `ollama` · `openai-compatible` |
| **อะไรรัน agent** | Binary CLI ของ vendor (`claude`, `agy`, `codex`, `kimi`, `copilot`) | `bwoc-harness` (มาพร้อม BWOC) |
| **ต้องมี binary บน PATH** | ใช่ — BWOC ไม่ bundle vendor CLI | ไม่ |
| **Agentic loop** | Vendor กำหนด | BWOC กำหนด (สม่ำเสมอทุก harness backend) |
| **รองรับ `bwoc chat --tui`** | ไม่ — vendor CLI render interface เอง | ใช่ |
| **`baseUrl` ใน manifest** | ไม่ใช้ | optional สำหรับ `ollama`; required สำหรับ `openai-compatible` |
| **Tool authorization** | Vendor กำหนด policy | BWOC safety guardrail layer |

ไฟล์ instruction เหมือนกันทุก backend — `OLLAMA.md` และ `OPENAI.md` เป็น symlink ชี้ไปยัง `AGENTS.md` เหมือนกับ `CLAUDE.md` ทุกประการ สิ่งที่ต่างกันระหว่าง vendor session กับ harness session คือใครรัน loop และ model traffic ไปที่ไหน

---

## 3. ตั้งค่า Ollama Backend

**เงื่อนไขก่อนเริ่ม:** Ollama ติดตั้งและรันอยู่ มี model อย่างน้อยหนึ่งตัวที่ pull มาแล้ว (เช่น `ollama pull gemma3:27b`) agent incarnate แล้วและผ่าน `bwoc check`

### ขั้นที่ 1 — ตั้ง model ใน manifest

เปิด (หรือแก้ผ่าน prompt) `config.manifest.json` ใน agent directory แล้วตั้ง `primaryModel` เป็น Ollama model tag ที่ต้องการ:

```json
{
  "primaryModel": "gemma3:27b",
  "fallbackModel": "gemma3:4b"
}
```

ฟิลด์ `fallbackModel` optional แต่แนะนำให้ใส่ harness จะสลับไปใช้มันอัตโนมัติถ้า primary ตอบสนองไม่ได้

### ขั้นที่ 2 — ตั้ง `baseUrl` (เฉพาะเมื่อ Ollama ไม่ได้อยู่บน localhost)

ถ้า Ollama รันบน host ระยะไกล ให้เพิ่ม `baseUrl` ใน manifest:

```json
{
  "primaryModel": "gemma3:27b",
  "baseUrl": "http://192.168.1.113:11434/v1"
}
```

ถ้าไม่มี `baseUrl` `bwoc-harness` จะใช้ `http://localhost:11434/v1` เป็นค่า default สำหรับ Ollama ที่ติดตั้ง local ทั่วไปไม่ต้องใส่

### ขั้นที่ 3 — Spawn agent

```bash
bwoc spawn <agent> --backend ollama
# หรือจากใน agent directory
bwoc spawn --backend ollama
```

สามารถตั้ง `"backend": "ollama"` ใน manifest เพื่อให้ `bwoc spawn <agent>` ใช้ Ollama โดยไม่ต้องพิมพ์ `--backend` ทุกครั้ง

### ตรวจสอบว่า endpoint เข้าถึงได้

```bash
curl http://localhost:11434/v1/models
# ต้องคืน JSON list ของ model ที่ pull มาแล้ว
```

ถ้า curl ล้มเหลว `bwoc spawn --backend ollama` ก็จะล้มเหลวเช่นกัน — แก้ Ollama ก่อน

---

## 4. ตั้งค่า OpenAI-Compatible Endpoint

HTTP server ใดก็ตามที่ implement OpenAI Chat Completions API (`POST /v1/chat/completions`) ใช้เป็น harness backend ได้ ครอบคลุมถึง hosted inference service, local inference server (vLLM, llama.cpp server, LM Studio เป็นต้น), และ cloud provider ที่ไม่ใช่ native BWOC backend

**`baseUrl` เป็นสิ่งจำเป็น** ไม่มีค่า default ถ้าไม่มี `bwoc spawn --backend openai-compatible` จะออกทันทีพร้อม error ชัดเจน

### ขั้นที่ 1 — ตั้ง `baseUrl` และ `primaryModel`

```json
{
  "primaryModel": "my-custom-model",
  "baseUrl": "http://localhost:8080/v1"
}
```

### ขั้นที่ 2 — ตั้ง API key ถ้า endpoint ต้องการ

ถ้า endpoint ต้องการ API key ให้ตั้งใน environment variable ที่ harness อ่าน pattern ทั่วไป:

```bash
export OPENAI_API_KEY="your-key-here"
bwoc spawn <agent> --backend openai-compatible
```

harness จะส่ง key ใน header `Authorization: Bearer` ถ้าไม่ต้องการ key env var จะว่างหรือไม่มีก็ได้

### ขั้นที่ 3 — Spawn

```bash
bwoc spawn <agent> --backend openai-compatible
```

### Checklist ความ compatible ของ endpoint

- Implement `POST /v1/chat/completions` พร้อมฟิลด์ `messages`, `model`, และ `stream`
- คืน server-sent events เมื่อ `stream: true`
- คืน `choices[].delta.content` ในแต่ละ streaming chunk
- รับ `tools` และ `tool_calls` สำหรับ function-calling (จำเป็นสำหรับ core tools)

ถ้า endpoint ไม่รองรับ tool calls, core tools จะไม่ทำงาน agent ยังรันได้แต่จะจำกัดอยู่กับ text response ล้วนๆ

---

## 5. TUI Chat Client

`bwoc chat <agent> --tui` เปิด terminal chat interface แบบ full-screen ที่สร้างด้วย ratatui ใช้ได้เฉพาะกับ harness backend (`ollama` และ `openai-compatible`) เท่านั้น vendor CLI backend render interface เอง flag `--tui` ไม่มีผลกับพวกมัน

```bash
# Full-screen TUI กับ backend ที่ configure ไว้ (ต้องเป็น harness backend)
bwoc chat sage --tui

# ระบุ backend ชัดเจน
bwoc chat sage --backend ollama --tui
bwoc chat sage --backend openai-compatible --tui
```

สิ่งที่ TUI มอบให้:

- **Streaming token display** — token ของ model ปรากฏเมื่อมาถึง ไม่ต้องรอ response เต็ม
- **Tool call visibility** — เมื่อ harness เรียก core tool, TUI แสดงว่าเรียก tool ไหนและผลลัพธ์ก่อนที่ model จะดำเนินต่อ
- **Session history** — บทสนทนา scroll ด้วย keyboard
- **Status bar** — model ปัจจุบัน, backend, และ latency มองเห็นตลอดเวลา

ภายใต้ประทุน `bwoc chat --tui` เริ่ม `bwoc-harness --chat` และ render `chat_proto` event stream ที่มันปล่อยออกมา stream เดียวกันนี้คือสิ่งที่ app `bwoc-chat` desktop consume — ทั้งสองใช้ wire format เดียวกัน

ออกจาก TUI: `Ctrl+C` หรือ `q` จาก input field

---

## 6. สิ่งที่ Harness มอบให้

`bwoc-harness` ไม่ได้เป็นแค่ HTTP bridge — agent ทุกตัวที่รันบน harness backend จะได้รับ BWOC agentic runtime เต็มรูปแบบตามที่อธิบายด้านล่าง

### 6.1 Agentic loop และ core tools

harness implement BWOC agentic loop: ส่ง context ของ agent ไปยัง model, รับ response, ตรวจจับ tool call, execute tool ที่ผ่านการ authorize, append ผลลัพธ์, และ loop ต่อจนกว่า model จะสร้าง final response โดยไม่มี pending tool call

Core tools ที่มาพร้อม harness:

| Tool | หน้าที่ |
|---|---|
| `read_file` | อ่านไฟล์จาก filesystem |
| `write_file` | เขียนหรือ overwrite ไฟล์ |
| `edit_file` | apply diff แบบ targeted ให้ไฟล์ |
| `run_command` | execute shell command และ capture output |
| `search` | grep หรือ glob ใน workspace |
| `memory_get` / `memory_put` | อ่านและเขียน Tier-1 memory entry |
| `http_fetch` | fetch URL (อยู่ภายใต้ safety guardrails) |

การใช้งาน tool อยู่ภายใต้ authorization layer ที่อธิบายใน Section 6.2

### 6.2 Tool authorization

ก่อนที่ tool call จะ execute harness จะส่งผ่าน tool-authorization check คุณ configure ได้ว่า tool ไหน enable, disable, หรือต้องยืนยันก่อนใน `config.manifest.json` หรือผ่าน prompt ในระหว่าง session

สิ่งนี้แตกต่างจาก safety guardrail layer (Section 6.5) ซึ่งทำงานในระดับที่สูงกว่าและ agent ไม่สามารถ override ได้ tool authorization คือ operator-level control สำหรับ tune ว่า agent เฉพาะตัวทำอะไรได้บ้าง

### 6.3 Task queue

เมื่อรันภายใต้ `bwoc run` หรือใน fleet ที่มีการ supervise harness จะเชื่อมต่อกับ workspace task queue agent สามารถ dequeue task, รายงานความคืบหน้า, และ mark completion ได้ สิ่งนี้คือสิ่งที่ทำให้ multi-agent pipeline และ asynchronous task dispatch เป็นไปได้ — agent หนึ่ง post task อีกตัวรับไปทำ

### 6.4 Telemetry

harness ปล่อย structured telemetry ในแต่ละ session: token count (prompt และ completion), tool call count, latency ต่อ turn, และ exit status ข้อมูลนี้พร้อมใช้งานสำหรับ `bwoc supervise` และ monitoring tooling ใดก็ตามที่คุณเชื่อมต่อกับ workspace

Telemetry อยู่ใน local ไม่ส่งออกไปยัง external service เว้นแต่คุณ configure exporter

### 6.5 Safety guardrails

harness มี safety guardrail layer ที่ทำงานก่อน tool execution ออกแบบตาม framework principles สองข้อ:

**Sīla 5 (ศีลห้า) — forbidden actions** harness ปฏิเสธการ execute tool call ที่ map ไปยังหนึ่งในห้า forbidden category โดยไม่คำนึงว่า model จะร้องขออะไร สิ่งเหล่านี้คือ hard stop ไม่ใช่ soft warning agent ไม่สามารถ override ผ่าน prompt ได้

**Taṇhā 3 (ตัณหาสาม) — threat detection** guardrail layer classify tool call แต่ละตัวกับสามประเภทของ risky intent (destructive intent, privacy violation, deception) call ที่ถูก classify จะถูก block และ log ไม่ใช่แค่ skip แบบเงียบๆ

Guardrail อยู่นอกเหนือจาก tool authorization ไม่ใช่ replacement action ต้องผ่านทั้งสอง layer จึงจะ execute ได้

### 6.6 Tier-2 deep memory

เมื่อตั้ง `deepMemoryCmd` และ `sessionsPath` ใน `config.manifest.json` harness จะเปิดใช้งาน Tier-2 deep memory ซึ่งเป็น persistent, cross-session, cross-agent memory ที่ agent เรียกคืนได้เมื่อเริ่ม session และ mine ได้เมื่อสิ้นสุด session

ฟิลด์ manifest ที่เกี่ยวข้อง:

```json
{
  "deepMemoryCmd": "bwoc memory",
  "sessionsPath": "memories/sessions/"
}
```

เมื่อตั้งค่าเหล่านี้ harness จะพูด Tier-2 contract อัตโนมัติ:

- `wake-up` — เรียกเมื่อเริ่ม session เพื่อเรียกคืน context ก่อนหน้าที่เกี่ยวข้อง
- `search` — เรียกเมื่อ agent ต้องค้นหา decision หรือ feedback ในอดีต
- `mine` — เรียกเมื่อสิ้นสุด session เพื่อ persist สิ่งที่เรียนรู้จาก session

ถ้าไม่มีฟิลด์เหล่านี้ Tier-2 จะ inactive และ agent ทำงานด้วย Tier-1 (`MEMORY.md`) เท่านั้น ดู [`../self-improvement/HANDBOOK.th.md`](../self-improvement/HANDBOOK.th.md) เพื่อทำความเข้าใจว่า memory tier เข้าไปอยู่ใน learning loop อย่างไร

### 6.7 chat_proto event stream

harness backend ทุกตัวปล่อย event บน `chat_proto` stream ซึ่งเป็น structured wire format ที่บรรจุ:

- `token` event เมื่อ model streaming output
- `tool_call` / `tool_result` event สำหรับแต่ละ tool invocation
- `turn_start` / `turn_end` event พร้อม latency metadata
- `error` event พร้อม structured error code

นี่คือ stream เดียวกับที่ `bwoc chat --tui` render และที่ app `bwoc-chat` desktop consume ถ้าคุณกำลังสร้าง tooling บน BWOC (dashboard, logging pipeline, test harness) ให้ consume stream นี้โดยตรง

---

## 7. Dependency Quarantine

`bwoc-harness` compile เป็น binary แยก ไม่ได้ link เข้า `bwoc` core นี่คือ design choice โดยตั้งใจที่เรียกว่า **dependency quarantine**: network stack, HTTP client, streaming parser, และ runtime dependency ทั้งหมดที่แตะ external endpoint อยู่ใน harness binary ไม่ใช่ `bwoc` core

ผลในทางปฏิบัติสำหรับ operator:

- **`bwoc` core เบา** — การ build หรือ audit `bwoc` core ไม่ดึง HTTP หรือ async runtime dependency เข้ามา
- **Harness update ได้อิสระ** — model API version ใหม่หรือ security patch ใน HTTP client ship ใน harness update โดยไม่แตะ core CLI
- **Security boundary ชัดเจน** — ทุกอย่างที่คุยกับ network ถูก isolate ใน binary เดียว การ audit harness binary คือการ audit code ทั้งหมดที่ facing network

คุณไม่ต้องจัดการ harness binary แยกต่างหาก `bwoc spawn` ค้นหาและ exec มันอัตโนมัติ ถ้า harness binary หายไปหรือ outdated `bwoc` จะรายงาน version mismatch

---

## 8. ดูเพิ่มเติม

| แหล่งข้อมูล | สิ่งที่ได้ |
|---|---|
| [HARNESS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/HARNESS.en.md) | Harness spec ฉบับ authoritative — protocol details, guardrail rules, manifest schema |
| [bwoc-harness crate](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-harness) | Source code ของ harness binary |
| [`../backends/HANDBOOK.th.md`](../backends/HANDBOOK.th.md) | Backend ทุกตัวเคียงกัน — launch command, manifest fields, troubleshooting |
| [`../self-improvement/HANDBOOK.th.md`](../self-improvement/HANDBOOK.th.md) | Memory tier, learning loop, และ Tier-2 deep memory ตั้งแต่ต้นจนจบ |
| [`../glossary.th.md`](../glossary.th.md) | Sīla, Taṇhā, และทุก term เฉพาะที่ใช้ข้างต้น |
| [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) | Framework repo root |

---

*ถ้ามีข้อขัดแย้งระหว่าง handbook นี้กับ [framework repo](https://github.com/bemindlabs/BWOC-Framework) ให้ถือว่า repo ถูก — handbook นี้มี bug โปรดช่วยแก้ไข*
