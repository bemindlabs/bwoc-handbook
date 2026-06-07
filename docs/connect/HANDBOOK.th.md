# Chat Connectors — Telegram & Discord

🇬🇧 [English](HANDBOOK.en.md)

สำหรับผู้ที่ต้องการคุยกับ BWOC agent โดยตรงจาก Telegram หรือ Discord โดยไม่ต้องเปิด terminal บทนี้ครอบคลุมวิธีการทำงานของ `bwoc-connect` การตั้งค่า secrets การรันและ supervise connector รวมถึงขอบเขตด้านความปลอดภัย

ค้นหาคำศัพท์: [`../glossary.th.md`](../glossary.th.md) · source code crate: [bwoc-connect](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-connect) · framework repo: [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)

---

## สารบัญ

1. [ทำไมต้องใช้ Chat Connector?](#1-ทำไมต้องใช้-chat-connector)
2. [Platform ที่รองรับ](#2-platform-ที่รองรับ)
   - [2.1 Telegram (DM และ group)](#21-telegram-dm-และ-group)
   - [2.2 Discord (DM และ server channel)](#22-discord-dm-และ-server-channel)
3. [การตั้งค่า](#3-การตั้งค่า)
   - [3.1 สร้าง bot และรับ token](#31-สร้าง-bot-และรับ-token)
   - [3.2 เขียน connector config](#32-เขียน-connector-config)
   - [3.3 เก็บ token อย่างปลอดภัย](#33-เก็บ-token-อย่างปลอดภัย)
   - [3.4 รัน connector](#34-รัน-connector)
   - [3.5 Daemon supervision ผ่าน bwoc-agent --serve](#35-daemon-supervision-ผ่าน-bwoc-agent---serve)
4. [วิธีการไหลของข้อความ](#4-วิธีการไหลของข้อความ)
   - [4.1 DM path](#41-dm-path)
   - [4.2 Group / server-channel path](#42-group--server-channel-path)
5. [ความปลอดภัย](#5-ความปลอดภัย)
6. [ข้อจำกัดและหมายเหตุเวอร์ชัน](#6-ข้อจำกัดและหมายเหตุเวอร์ชัน)
7. [ดูเพิ่มเติม](#7-ดูเพิ่มเติม)

---

## 1. ทำไมต้องใช้ Chat Connector?

Agent ที่รันผ่าน vendor CLI หรือ `bwoc-harness` ปกติจะเข้าถึงได้จาก terminal: พิมพ์ `bwoc chat` เปิด TUI แล้วแลกเปลี่ยนข้อความในนั้น วิธีนี้สะดวกสำหรับนักพัฒนา แต่ไม่สะดวกสำหรับคนอื่นในทีมที่ไม่ได้ทำงานใน terminal

`bwoc-connect` คือ bridge ที่ย้าย conversation surface ของ agent ไปอยู่ใน chat app ที่ทุกคนเปิดอยู่อยู่แล้ว คุณได้ BWOC agent ตัวเดียวกัน — memory เดิม skills เดิม tool access เดิม — แต่เข้าถึงได้จาก Telegram DM หรือ Discord server channel ไม่ต้องเรียนรู้ interface เพิ่มเติม ไม่ต้องใช้ terminal สำหรับผู้ใช้ปลายทาง

connector เป็น binary แยกต่างหาก ไม่ได้เป็นส่วนหนึ่งของ `bwoc-cli` หรือ `bwoc-agent` network dependencies (reqwest, tokio-tungstenite) ถูก quarantine ไว้ที่นั่นเพื่อให้ core framework เบาและกระชับ

---

## 2. Platform ที่รองรับ

### 2.1 Telegram (DM และ group)

Telegram เป็น platform หลัก connector ใช้ **Bot API long-poll** (`getUpdates` timeout 25 วินาที): ส่ง request รอข้อความใหม่ ประมวลผล เลื่อน offset แล้วทำซ้ำ ไม่ต้องการ webhook ไม่ต้องเปิด inbound port

**DMs.** ผู้ใช้ส่งข้อความหา bot โดยตรง แต่ละ DM conversation ถูก isolate: connector ถือ agent session หนึ่งตัวต่อหนึ่ง `chat_id` ดังนั้น thread การสนทนาของคุณจะไม่ปนกับของคนอื่น

**Groups และ supergroups.** Agent สามารถเข้าร่วม group และมีส่วนร่วมใน Saṅgha team chat ได้ พฤติกรรมของ group ขึ้นอยู่กับ `mention_only` ใน config:

- **Mention-gated (ค่าเริ่มต้น `mention_only = true`).** ข้อความที่ @mention bot จะเริ่ม agent turn จริงๆ agent อ่าน peer context ของห้อง (ข้อความของคนอื่น) แล้วตอบ ข้อความที่ไม่ได้ mention bot จะถูก log เงียบๆ ลงใน team's `chat.jsonl` เป็น peer context สำหรับ turn ถัดไปของ agent โดยไม่มีการตอบกลับ
- **Open (`mention_only = false`).** ทุกข้อความจากผู้ส่งที่อยู่ใน allow-list จะ trigger agent turn

connector resolve `@username` ของ bot ตอน startup (ผ่าน `getMe`) เพื่อให้ตรวจจับ mention ได้แม่นยำ Channels (broadcast-only) และ anonymous posts จะถูกข้ามไป

### 2.2 Discord (DM และ server channel)

Discord ไม่มี long-poll API แทนที่ connector จะเชื่อมต่อกับ **Discord Gateway websocket** (v10) authenticate ด้วย IDENTIFY และส่ง heartbeat เป็นระยะ background task เป็นเจ้าของ connection push incoming `MESSAGE_CREATE` events เข้า queue และ reconnect อัตโนมัติหลัง disconnect ใดๆ

**Intents ที่ต้องการบน bot application ของคุณ:** `GUILD_MESSAGES`, `DIRECT_MESSAGES`, `MESSAGE_CONTENT` (privileged)

**DMs.** ทำงานเหมือนกับ Telegram DMs — agent session แยกต่างหากหนึ่งตัวต่อ channel

**Server channels (guilds).** พฤติกรรมเหมือน Telegram groups: `mention_only = true` ค่าเริ่มต้น การตรวจจับ mention ใช้ `mentions[]` array ใน Discord payload (robust กว่าการ match substring) ข้อความจาก bot จะถูก ignore อัตโนมัติเพื่อป้องกัน reply loop

routing และ allow-list logic เหมือนกันทั้งสอง platform มีแค่ transport layer ที่ต่างกัน

---

## 3. การตั้งค่า

### 3.1 สร้าง bot และรับ token

**Telegram.** เปิด [@BotFather](https://t.me/BotFather) ส่ง `/newbot` ทำตามขั้นตอน copy token (`123456789:ABCdef...`)

**Discord.** เปิด [Discord Developer Portal](https://discord.com/developers/applications) สร้าง application เพิ่ม Bot เปิด `MESSAGE CONTENT` privileged intent และ copy bot token นำ bot เข้า server ด้วย `bot` scope และ permission `Send Messages` + `Read Message History`

### 3.2 เขียน connector config

สร้าง `connectors/<platform>.toml` ไว้ใน agent directory (เช่น `agents/agent-sage/connectors/telegram.toml`)

Config ขั้นต่ำ (DM เท่านั้น):

```toml
enabled = true

# Platform user ids ที่อนุญาตให้เข้าถึง agent นี้
# Empty list = ไม่มีใคร ปิดค่าเริ่มต้น — เพิ่ม id ของคุณเอง
allow_from = [123456789]
```

Config พร้อม group/server-channel support:

```toml
enabled = true
allow_from = [123456789, 987654321]

[group]
# Saṅgha team ที่มี shared chat.jsonl สำรอง group room นี้
team = "my-team"
# ตอบเฉพาะเมื่อถูก @mention (ค่าเริ่มต้น true) ตั้งเป็น false เพื่อตอบทุกข้อความ
mention_only = true
```

ค่า `team` ต้องเป็น token ธรรมดา (ตัวอักษร ตัวเลข `-` `_` เท่านั้น) map ไปที่ `.bwoc/teams/<team>/chat.jsonl` ใน workspace root

ทั้ง `telegram.toml` และ `discord.toml` ใช้ config shape เดียวกันนี้

### 3.3 เก็บ token อย่างปลอดภัย

**macOS และ Windows.** connector อ่าน token จาก OS native keyring ก่อน keyring service คือ `bwoc/<platform>` และ account name คือ basename ของ agent directory

ตัวอย่าง (macOS, Telegram, agent ชื่อ `agent-sage`):

```sh
security add-generic-password -a agent-sage -s bwoc/telegram -w "YOUR_TOKEN"
```

**Linux และ fallback ทุก platform.** ตั้ง environment variable ก่อนรัน connector:

```sh
export TELEGRAM_BOT_TOKEN="YOUR_TOKEN"
# หรือสำหรับ Discord:
export DISCORD_BOT_TOKEN="YOUR_TOKEN"
```

ถ้าไม่มีทั้ง keyring entry และ environment variable `bwoc-connect` จะออกพร้อม error message ที่ระบุชื่อ variable ที่ต้องการ

### 3.4 รัน connector

```sh
bwoc-connect telegram --agent agents/agent-sage
# หรือ
bwoc-connect discord --agent agents/agent-sage
```

connector log ไปที่ stderr: platform ที่ใช้ agent ที่ใช้ allow list และ (ถ้าตั้งค่าไว้) team chat log path main loop รันจนกว่าจะถูก interrupt

สำหรับ smoke-testing แบบ manual `--max-polls N` จำกัด polling loop ให้ทำ N iteration แล้วออกอย่าง clean flag นี้ไม่ใช่สำหรับ production

### 3.5 Daemon supervision ผ่าน bwoc-agent --serve

การรัน `bwoc-connect` เป็น foreground process ยาวนานก็ใช้ได้สำหรับ development สำหรับ production ใช้ `bwoc-agent --serve` จาก agent directory:

```sh
bwoc-agent --serve --workdir agents/agent-sage
```

`ConnectorSupervisor` ใน `bwoc-agent` ตรวจจับว่า connector config ตัวไหน enabled locate binary `bwoc-connect` (sibling ของ process ปัจจุบัน แล้วค่อย PATH) spawn มัน และ keep alive:

- ถ้า `bwoc-connect` ออกด้วยเหตุผลใดก็ตาม จะถูก respawn หลัง backoff 5 วินาทีเพื่อป้องกัน crash-loop หมุน hot
- เมื่อ `bwoc-agent --serve` shutdown มันจะ kill connector child และเขียน status marker `stopped`
- health state (`running`, `exited`, `stopped`) ถูกเขียนแบบ atomic ลง `<agent>/.bwoc/connector.status` เพื่อให้ `bwoc status` แสดงได้

ถ้า `bwoc-connect` ไม่อยู่บน PATH และไม่ใช่ sibling binary `bwoc-agent --serve` จะ log warning แล้วรันต่อโดยไม่มี connector ไม่ abort

---

## 4. วิธีการไหลของข้อความ

### 4.1 DM path

```
Platform (Telegram/Discord)
        |
        |  [poll / gateway push]
        v
  bwoc-connect bridge
        |
        |  allow-list check (from_user_id อยู่ใน allow_from?)
        |    ถูก reject → silent drop
        v
  per-chat AgentSession (bwoc-harness --chat subprocess)
        |
        |  ChatInput::User เขียนไปที่ harness stdin
        |  ChatEvent stream อ่านจาก harness stdout
        |    PermissionRequest → auto-denied (remote user อนุมัติ tool ไม่ได้)
        |    ChatEvent::Message → reply text
        v
  Platform send (ตอบกลับไปที่ chat_id เดิม)
```

`bwoc-harness --chat` subprocess หนึ่งตัวถูก spawn ต่อ `chat_id` ในข้อความแรก และนำมาใช้ซ้ำตลอดอายุของ conversation ถ้า harness ออกกลางเซสชัน connector จะ drop session ที่ตายแล้วและ respawn ใหม่ในข้อความถัดไปจาก chat นั้น

### 4.2 Group / server-channel path

incoming group messages ผ่าน allow-list check ก่อนเหมือนกัน

หลังจากนั้น connector ตรวจ `mentions_bot`:

- **ถูก mention (หรือ `mention_only = false`).** ข้อความนี้คือ user turn `--team-chat` session (หนึ่งตัวต่อ group `chat_id`) ถูก spawn หรือนำมาใช้ซ้ำ ส่งข้อความเป็น `ChatInput::User` และ reply ของ agent ถูกส่งกลับไปยัง group
- **ไม่ได้ mention (`mention_only = true` ค่าเริ่มต้น).** ข้อความถูก append ลงใน team's `chat.jsonl` เป็น peer context (`TeamChatMessage` ที่มี `from: "tg:<user_id>"` หรือ `from: "dc:<user_id>"`) ไม่มีการสร้าง harness session ไม่มีการส่ง reply agent จะเห็น context นี้ครั้งถัดไปที่ถูก address

path ของ team `chat.jsonl` ถูก resolve โดยการเดินขึ้นจาก agent directory จนพบ `.bwoc/workspace.toml` marker แล้วต่อด้วย `.bwoc/teams/<team>/chat.jsonl` directory จะถูกสร้างอัตโนมัติถ้ายังไม่มี

---

## 5. ความปลอดภัย

**ปิดค่าเริ่มต้น.** `allow_from` list ว่างเปล่าหมายความว่าไม่มีใครเข้าถึง agent ได้ แม้ bot จะถูก list สาธารณะบน Telegram หรือถูกเพิ่มเข้า Discord server คุณต้องระบุ numeric platform user ids อย่างชัดเจนเพื่อเปิด access นี่ไม่สามารถ configure ออกไปได้ — มันคือ design

**Tool calls ไม่สามารถอนุมัติจากระยะไกลได้.** harness session รันในโหมด non-TTY `PermissionRequest` event ใดๆ จาก agent จะถูก deny อัตโนมัติ คนที่คุยผ่าน Telegram หรือ Discord ไม่สามารถอนุมัติ tool call ที่ destructive ได้

**การเก็บ token.** บน macOS และ Windows token อยู่ใน native keyring (Keychain / Windows Credential Manager) ไม่ใช่ใน config file บน Linux เป็น environment variable อย่า hardcode ลงใน shell scripts ที่ check in ไปใน source control

**Group allow-list.** ใน group หรือ server channel การตรวจ `allow_from` ใช้กับ user id ของผู้ส่งแต่ละคน ไม่ใช่กับ group เอง ใครที่ไม่อยู่ใน list ไม่สามารถเข้าถึง agent ได้แม้จะเป็นสมาชิกของ group ข้อความของพวกเขาจะถูก drop เงียบๆ (ไม่แม้แต่ log เป็น peer context)

---

## 6. ข้อจำกัดและหมายเหตุเวอร์ชัน

- `bwoc-connect` จัดการ **connector ที่ enabled หนึ่งตัวต่อ agent** ใน daemon supervision model ปัจจุบัน (`bwoc-agent --serve` supervise config แรกที่ enabled ที่หาเจอ)
- **Media (รูปภาพ ไฟล์ voice messages) ไม่ได้รับการจัดการ.** เฉพาะ text messages เท่านั้นที่ถูก relay non-text updates จะถูกข้ามไปเงียบๆ
- **Slash commands ยังไม่ได้ implement.** bot ตอบสนองต่อข้อความ plain text เท่านั้น
- **Multi-group binding** (agent หนึ่งตัว bridge กับหลาย group room พร้อมกัน) ยังไม่รองรับ
- **Discord gateway integration** ยังไม่ได้ verify กับ live bot ใน CI edge ที่ live ถูก review แบบ eyeball ถือว่าเป็น beta
- crate `bwoc-connect` version ตาม framework `Software-Version` ตรวจสอบ [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) สำหรับ release ปัจจุบัน

---

## 7. ดูเพิ่มเติม

- [Self-Hosting with bwoc-harness](../harness/HANDBOOK.th.md) — วิธีการทำงานของ harness subprocess และ `chat_proto` คืออะไร
- [Cross-Workspace & Protocols](../cross-workspace/HANDBOOK.th.md) — peer protocol, MQTT, multi-machine agents
- [Fleet Operations](../fleet-ops/HANDBOOK.th.md) — การรันและ supervise agents หลายตัวด้วย `bwoc-agent --serve`
- [Security & the Tianting Team](../security/HANDBOOK.th.md) — threat model, Sīla 5, signed trust
- [Glossary](../glossary.th.md) — Saṅgha, chat_proto, allow-list และคำศัพท์อื่นๆ
- Crate source: [bwoc-connect](https://github.com/bemindlabs/BWOC-Framework/tree/main/crates/bwoc-connect)
