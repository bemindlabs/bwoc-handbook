---
title: การเชื่อมต่อ Workspace และ Agent ข้ามเครื่อง
nav_order: 12
---

# การเชื่อมต่อ Workspace และ Agent ข้ามเครื่อง

> English version: [HANDBOOK.en.md](HANDBOOK.en.md) · Framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) · คำศัพท์: [../glossary.th.md](../glossary.th.md) · คู่มือ Agent: [../agents/HANDBOOK.th.md](../agents/HANDBOOK.th.md) · คู่มือความปลอดภัย: [../security/HANDBOOK.th.md](../security/HANDBOOK.th.md)

**แหล่งข้อมูลที่ถูกต้อง.** สเปคอยู่ใน framework repo เสมอ ถ้าเนื้อหาขัดแย้งกัน repo คือคำตอบ และหมายความว่า handbook นี้มีบั๊ก — กรุณาแก้ไข crate ที่เกี่ยวข้อง: [`bwoc-a2a`](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-a2a) · [`bwoc-mqtt`](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-mqtt) · docs: [`A2A.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/A2A.en.md) · [`SIGNING.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) · [`PEERS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PEERS.en.md)

---

## สารบัญ

1. [ทำไมต้องข้าม Workspace?](#1-ทำไมต้องข้าม-workspace)
2. [รากฐานความน่าเชื่อถือ — Signed Envelope](#2-รากฐานความน่าเชื่อถือ--signed-envelope)
3. [Peer และคำสั่ง `bwoc peer`](#3-peer-และคำสั่ง-bwoc-peer)
   - [3.1 ประกาศ Peer ใน routes.toml](#31-ประกาศ-peer-ใน-routestoml)
   - [3.2 bwoc peer list](#32-bwoc-peer-list)
   - [3.3 bwoc peer status](#33-bwoc-peer-status)
   - [3.4 bwoc peer learn](#34-bwoc-peer-learn)
   - [3.5 bwoc peer feedback](#35-bwoc-peer-feedback)
4. [โปรโตคอล A2A — Agent2Agent Interop](#4-โปรโตคอล-a2a--agent2agent-interop)
   - [4.1 สิ่งที่ bwoc-a2a มีให้](#41-สิ่งที่-bwoc-a2a-มีให้)
   - [4.2 Agent Card](#42-agent-card)
   - [4.3 JSON-RPC Message และ Task Handler](#43-json-rpc-message-และ-task-handler)
   - [4.4 HTTP Listener](#44-http-listener)
   - [4.5 Dependency Quarantine](#45-dependency-quarantine)
5. [MQTT Transport](#5-mqtt-transport)
   - [5.1 การ Publish Envelope](#51-การ-publish-envelope)
   - [5.2 การ Subscribe — ส่งเข้า inbox.jsonl](#52-การ-subscribe--ส่งเข้า-inboxjsonl)
6. [เลือก Transport ไหนดี?](#6-เลือก-transport-ไหนดี)
7. [ดูเพิ่มเติม](#7-ดูเพิ่มเติม)

---

## 1. ทำไมต้องข้าม Workspace?

Workspace เดียวก็พอสำหรับทีมส่วนใหญ่ แต่บางปัญหาต้องให้ agent จากเครื่องต่างกัน — หรือต่างองค์กร — ทำงานร่วมกัน:

- Agent ใน workspace หนึ่งต้องดู task list หรือเอกสารความรู้ของ agent ผู้เชี่ยวชาญที่อยู่ที่อื่น
- Pipeline บนเครื่องระยะไกลสร้าง output ที่ agent ในเครื่องต้องนำไปใช้ต่อ
- สองทีมต้องการแลก verified feedback เกี่ยวกับ agent ของกันโดยไม่ต้องรวม workspace
- ระบบอัตโนมัติเต็มรูปแบบที่ไม่มีคนคอยส่งข้อความระหว่าง service

BWOC แก้ทั้งสี่กรณีด้วยแนวคิดเดียวกัน: **signed envelope ที่ route ผ่าน transport หนึ่งในสาม**. Transport ต่างกันที่ latency, topology, และต้นทุนการตั้งค่า แต่ trust model เหมือนกันทุกตัว

---

## 2. รากฐานความน่าเชื่อถือ — Signed Envelope

ทุกข้อความข้าม workspace ใน BWOC คือ **signed envelope**: โครงสร้าง JSON ที่บรรจุ payload, public key ของผู้ส่ง, และลายเซ็น [ed25519](https://ed25519.cr.yp.to/) ของ payload นั้น workspace ผู้รับจะไม่ประมวลผลข้อความข้าม workspace จนกว่าจะตรวจสอบลายเซ็นผ่าน

สร้าง keypair สำหรับ workspace ของคุณ:

```bash
bwoc trust --keygen
```

คำสั่งนี้เขียน private key ลงใน workspace credential store และบันทึก public key ไว้ใน `.bwoc/identity.toml` แชร์ public key ไปยัง workspace ที่ต้องการสื่อสารด้วย แล้วให้ฝั่งนั้นเพิ่มลงใน peer declaration (ดู [หัวข้อ 3.1](#31-ประกาศ-peer-ใน-routestoml))

framework ใช้โมเดล **Kalyanamitta-7** ในการให้คะแนน trust ระหว่าง agent — คุณสมบัติ 7 ข้อที่บอกว่า peer agent เป็น "กัลยาณมิตร" (ผู้ร่วมงานที่ไว้ใจได้) หรือไม่ trust ข้าม workspace คือหนึ่งในสัญญาณเหล่านั้น ดูรูบริกเต็มได้ที่ [คู่มือความปลอดภัย](../security/HANDBOOK.th.md) และ [SIGNING.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md)

> **กฎ.** ถ้าข้อความมาโดยไม่มีลายเซ็นที่ถูกต้อง หรือ signing key ไม่อยู่ใน peer allowlist ของผู้รับ — envelope จะถูก drop เงียบๆ และบันทึก log ไว้ ไม่มีข้อยกเว้น

---

## 3. Peer และคำสั่ง `bwoc peer`

**Peer** คือมุมมองแบบ read-only ข้าม workspace คุณประกาศว่า workspace ระยะไกลมีอยู่; workspace ของคุณสามารถดู agent, task, และเอกสารที่แชร์ได้ — แต่ไม่สามารถเขียนเข้าไปใน workspace นั้นโดยตรง การเขียนทำผ่าน signed feedback ([หัวข้อ 3.5](#35-bwoc-peer-feedback)) หรือ A2A/MQTT

Peer declaration อยู่ใน `.bwoc/routes.toml` ที่ root ของ workspace

### 3.1 ประกาศ Peer ใน routes.toml

```toml
# .bwoc/routes.toml

[[peer]]
key        = "oracle"                        # alias สั้นๆ ที่ใช้ในคำสั่ง bwoc
name       = "Oracle Workspace"              # ชื่อแสดงผล
url        = "https://oracle.example.com"   # URL ของ A2A listener ฝั่ง peer
# หรือถ้าเป็น local / LAN:
# path     = "/Users/shared/oracle-workspace"
pubkey     = "ed25519:<base64-encoded-public-key>"
allowlist  = true    # ยอมรับ doc allowlist จาก shared.toml ของ peer หรือไม่
```

`key` คือ alias ที่ใช้ในทุกคำสั่ง `bwoc peer` `pubkey` ต้องตรงกับ key ที่ peer workspace ประกาศด้วย `bwoc trust --keygen` framework ตรวจสอบสิ่งนี้ตอน connect

### 3.2 bwoc peer list

```bash
bwoc peer list
```

แสดง peer ทุกรายที่ประกาศใน `routes.toml` ของ workspace นี้ พร้อม key, ชื่อ, URL หรือ path, และสถานะการเข้าถึง (reachable / unreachable / unknown)

```
KEY       NAME                STATUS
oracle    Oracle Workspace    reachable
staging   Staging Fleet       unreachable (last seen 2h ago)
```

### 3.3 bwoc peer status

```bash
bwoc peer status oracle
```

แสดงสถานะปัจจุบันของ workspace นั้น: agent ที่ลงทะเบียนอยู่, team ที่มี, และ task ที่เปิดอยู่บน task list ของแต่ละทีม คำสั่งนี้เป็นแบบ **read-only** — ไม่สามารถแก้ไขอะไรได้

```
Peer: oracle  (Oracle Workspace)
Agents:        agent-wenchang, agent-guanyin, agent-nezha
Teams:         research (3 tasks open), ops (1 task open)
Last verified: 2026-06-07T10:04:33Z
```

### 3.4 bwoc peer learn

```bash
bwoc peer learn oracle           # แสดง doc ที่ peer แชร์ไว้
bwoc peer learn oracle DESIGN    # ดู doc เฉพาะตาม ชื่อ
```

workspace ฝั่ง peer กำหนดเองว่าจะแชร์เอกสารอะไรบ้าง โดยระบุใน `.bwoc/interconnect/shared.toml` ของตัวเอง workspace ของคุณอ่านได้เฉพาะสิ่งที่อยู่ใน allowlist นั้น — ไฟล์ส่วนตัวของ peer ไม่ถูกเปิดเผยเลย

`bwoc peer learn` โดยไม่ระบุชื่อเอกสารจะพิมพ์ allowlist: ชื่อเอกสาร, คำอธิบายหนึ่งบรรทัด, และวันที่แก้ไขล่าสุด ระบุชื่อเอกสารเพื่อดึงและแสดงเนื้อหาเต็มในเทอร์มินัล

นี่คือวิธีที่ความรู้แพร่กระจายระหว่างองค์กรโดยไม่ต้องรวม repo: ต้นทางกำหนดขอบเขต workspace ของคุณดึงข้อมูลได้ตามต้องการ

### 3.5 bwoc peer feedback

```bash
bwoc peer feedback oracle agent-wenchang \
  --score 4 \
  --comment "วิเคราะห์ root cause ของปัญหา deploy ได้ดีมาก"
```

ส่ง envelope แบบ `kind: feedback` ที่มีลายเซ็นไปยัง workspace ปลายทาง โดยระบุ inbox ของ agent ที่ต้องการ envelope ถูกเซ็นด้วย private key ของ workspace คุณ workspace ผู้รับตรวจสอบลายเซ็นกับ public key ที่ประกาศไว้ก่อนยอมรับ ถ้าตรวจสอบไม่ผ่านจะ drop ทิ้ง

เมื่อรับแล้ว feedback จะตกลงใน `memories/inbox.jsonl` ของ agent ที่ระบุ operator ของ agent สามารถ review, คัดกรอง, หรือยก level ขึ้นได้ตาม [วงจรการพัฒนาตนเอง](../self-improvement/HANDBOOK.th.md)

> **ทำไมลายเซ็นถึงสำคัญ.** feedback ที่ไม่มีลายเซ็นหรือตรวจสอบไม่ได้ปลอมแปลงง่ายมาก ลายเซ็น ed25519 ทำให้ agent ผู้รับรู้ว่า workspace ไหนส่งคะแนนมา และไม่สามารถถูกหลอกด้วยผู้แอบอ้างได้

---

## 4. โปรโตคอล A2A — Agent2Agent Interop

Peer ให้มุมมอง read-only แบบที่มนุษย์ริเริ่มข้าม workspace A2A คือ **โปรโตคอลเครื่องต่อเครื่อง** สำหรับ agent ที่ต้องแลก task และข้อความโดยอัตโนมัติ รวมถึง agent จากองค์กรอื่นที่ใช้ซอฟต์แวร์ต่างกัน

crate `bwoc-a2a` implement [Agent2Agent protocol](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-a2a) โดยผูกกับ **A2A spec v1.0.0**

### 4.1 สิ่งที่ bwoc-a2a มีให้

| ความสามารถ | ทำอะไร |
|---|---|
| **Agent Card** | เอกสารระบุตัวตนที่เครื่องอ่านได้ serve ที่ URL ที่รู้จักกัน เพื่อให้ peer ค้นหา capability ได้ |
| **JSON-RPC handlers** | รับ call `message/send` และ `task/send`; dispatch ไปยัง agent ที่ถูกต้อง; ส่งผลลัพธ์กลับ |
| **HTTP listener** | server ที่ใช้ [axum](https://github.com/tokio-rs/axum) เชื่อมทุกอย่างเข้าด้วยกัน |
| **Envelope signing/verification** | ทุก request และ response คือ signed BWOC envelope |

### 4.2 Agent Card

Agent Card คือเอกสาร JSON ที่ serve ที่ URL คงที่ (ตามแบบแผน `/.well-known/agent.json` บน A2A listener) ประกอบด้วย:

- ชื่อและเวอร์ชันของ agent
- A2A spec version ที่ implement (`"1.0.0"`)
- capability ที่รับได้ (ประเภท message, schema ของ task)
- public key สำหรับยืนยันลายเซ็น

agent ระยะไกลดึง Agent Card ของคุณเพื่อตัดสินใจว่าจะ interoperate ได้ไหมก่อนส่งข้อความแม้แต่ข้อเดียว ไม่ต้องตั้งค่าเองด้วยมือ — `bwoc-a2a` สร้าง card จาก `config.manifest.json` ของ agent และ workspace identity อัตโนมัติ

### 4.3 JSON-RPC Message และ Task Handler

การสื่อสาร A2A ใช้ [JSON-RPC 2.0](https://www.jsonrpc.org/specification) method หลักสองตระกูลคือ:

**`message/send`** — ส่งข้อความแบบ fire-and-forget หรือ request-reply ใช้เมื่อ agent ต้องการแจ้งเหตุการณ์, แชร์ผลลัพธ์, หรือถามคำถามโดยไม่ต้องการ task context ที่ persistent

**`task/send`** — สร้างหรืออัปเดต task ใน context ของ agent ผู้รับ ผู้รับสามารถ accept, reject, หรือส่งผลลัพธ์ย่อยกลับ ใช้สำหรับการประสานงานระยะยาวที่ต้องการ state ข้ามหลาย turn

ทั้งสอง method บรรจุ signed BWOC envelope เป็น payload ใน `params` handler ตรวจสอบลายเซ็นก่อนทำงานใดๆ

### 4.4 HTTP Listener

axum listener ของ `bwoc-a2a` เปิด endpoint ดังนี้:

```
GET  /.well-known/agent.json   → Agent Card
POST /a2a                      → JSON-RPC endpoint (message/send, task/send, ...)
```

เริ่ม listener จาก CLI (subcommand เต็มระบุใน [`A2A.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/A2A.en.md)):

```bash
bwoc a2a serve --agent agents/agent-wenchang --port 8080
```

ในสภาพแวดล้อม production วาง TLS-terminating proxy (nginx, Caddy, Cloudflare) ไว้ข้างหน้า listener เองพูด plain HTTP เท่านั้น; TLS เป็นเรื่องของ network layer

### 4.5 Dependency Quarantine

`bwoc-a2a` ถูกแยกออกจาก `bwoc-core` โดยตั้งใจ กฎคือ: **HTTP และ network dependency ทั้งหมดอยู่ใน `bwoc-a2a` เท่านั้น ไม่เข้า `bwoc-core`** ข้อนี้ทำให้ crate หลักเบาและป้องกันไม่ให้ transitive dependency จาก network stack รั่วเข้าไปใน workload ที่ไม่ต้องการ

ถ้าคุณเขียน workspace tool ที่ต้องการแค่ local agent operation ให้ depend เฉพาะ `bwoc-core` ดึง `bwoc-a2a` เข้ามาเฉพาะเมื่อต้องการ A2A HTTP capability

---

## 5. MQTT Transport

MQTT เหมาะเมื่อ agent สื่อสารผ่าน shared message broker แทน HTTP แบบ point-to-point เหมาะกับ:

- สถานการณ์ IoT หรือ embedded fleet ที่ device publish event ไปยัง broker
- เครือข่าย local ที่ latency ต่ำและมี Mosquitto broker รันอยู่แล้ว
- รูปแบบ fan-out ที่ผู้ส่งหนึ่งรายต้องการผู้รับหลายราย

crate `bwoc-mqtt` จัดการทั้งสองทิศทาง ดูอ้างอิง crate ได้ที่ [`bwoc-mqtt`](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-mqtt)

### 5.1 การ Publish Envelope

```bash
bwoc mqtt publish \
  --broker mqtt://192.168.1.113:1883 \
  --topic  bwoc/oracle/agent-wenchang/inbox \
  --agent  agents/agent-guanyin \
  --payload '{"kind":"message","body":"Deploy complete."}'
```

CLI เซ็น envelope ด้วย key ของ agent ผู้ส่ง แล้ว publish ไปยัง topic ที่ระบุ broker เก็บข้อความไว้จนกว่า subscriber จะมารับ

Topic ใช้แบบแผน `bwoc/<workspace-key>/<agent-name>/<slot>` — สามารถ subscribe ที่ระดับไหนของ hierarchy ก็ได้

### 5.2 การ Subscribe — ส่งเข้า inbox.jsonl

```bash
bwoc mqtt subscribe \
  --broker mqtt://192.168.1.113:1883 \
  --topic  bwoc/my-workspace/agent-wenchang/inbox \
  --agent  agents/agent-wenchang
```

subscriber ทำงานต่อเนื่อง ข้อความที่มาถึงแต่ละข้อจะถูกตรวจสอบ (ตรวจลายเซ็น) และถ้าถูกต้องจะ append เข้า `memories/inbox.jsonl` ของ agent ที่ระบุ envelope ที่ตรวจสอบไม่ผ่านจะถูก log และ drop

รัน subscriber เป็น background service (systemd unit, launchd plist, หรือ tmux session) บนเครื่องที่ host agent

---

## 6. เลือก Transport ไหนดี?

| สถานการณ์ | วิธีที่แนะนำ |
|---|---|
| ต้องการดู agent และ task ของ peer workspace โดยไม่ต้องอัตโนมัติ | `bwoc peer status` / `bwoc peer learn` |
| ต้องการให้คะแนน feedback แก่ peer agent | `bwoc peer feedback` |
| สอง agent บนเครื่องต่างกันต้องแลก task โดยอัตโนมัติ | A2A (`bwoc-a2a`, JSON-RPC over HTTP) |
| ต้องการ endpoint คงที่ที่ระบบภายนอกค้นหา agent ได้ | A2A — serve Agent Card ที่ `/.well-known/agent.json` |
| Agent สื่อสารผ่าน shared broker (IoT fleet, LAN, fan-out) | MQTT (`bwoc-mqtt`) |
| ปริมาณข้อความสูง, latency ต่ำ, หรือแบบ many-to-many | MQTT บน dedicated broker |
| แจ้งเตือนแบบ async จาก script ระยะไกลเข้า inbox ของ agent | MQTT publish (คำสั่งเดียว ไม่ต้อง listener ฝั่งผู้ส่ง) |

คำสั่ง peer และ A2A เสริมกัน: `bwoc peer` คือ CLI ที่มนุษย์ใช้ตรวจสอบและให้ feedback; A2A คือ wire protocol เครื่องต่อเครื่องสำหรับการประสานงานอัตโนมัติ MQTT แทน A2A เมื่อ topology แบบ broker เหมาะกว่า HTTP โดยตรง

> **ทั้งสามใช้ trust model เดียวกัน.** signed ed25519 envelope ตรวจสอบก่อนรับเสมอ transport เปลี่ยน แต่การรับประกันความปลอดภัยไม่เปลี่ยน

---

## 7. ดูเพิ่มเติม

- [คู่มือความปลอดภัย](../security/HANDBOOK.th.md) — การสร้าง key, คะแนน trust แบบ Kalyanamitta-7, threat model
- [คู่มือ Agent](../agents/HANDBOOK.th.md) — โครงสร้าง agent, `interconnect/`, inbox
- [คู่มือการพัฒนาตนเอง](../self-improvement/HANDBOOK.th.md) — วิธีที่ feedback ใน `inbox.jsonl` ถูกคัดกรองและยก level
- [คำศัพท์](../glossary.th.md) — Kalyanamitta 7, Sammā-vācā, Saṅgha
- Framework source: [`bwoc-a2a` crate](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-a2a) · [`bwoc-mqtt` crate](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-mqtt)
- Framework docs: [`A2A.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/A2A.en.md) · [`SIGNING.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SIGNING.en.md) · [`PEERS.en.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PEERS.en.md)
