# คู่มือ BWOC + Google Workspace

🇬🇧 [English](HANDBOOK.en.md)

เชื่อมต่อ workspace ของ BWOC เข้ากับ Google Workspace — Drive, Gmail และ Calendar — ผ่านกลุ่มปลั๊กอิน `gws` ที่ติดมากับ framework
บทนี้ครอบคลุมสิ่งที่ปลั๊กอินมอบให้ วิธีติดตั้งและเปิดใช้ การดำเนินการที่ทำได้ และวิธีจัดการ credential อย่างปลอดภัย

ดูคำศัพท์: [`../glossary.en.md`](../glossary.en.md) · ซอร์สโค้ดปลั๊กอิน: [bemindlabs/BWOC-Framework — modules/plugins/gws](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/gws) · สเปกปลั๊กอิน: [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md)

---

## สารบัญ

1. [สิ่งที่ปลั๊กอิน gws มอบให้คุณ](#1-สิ่งที่ปลั๊กอิน-gws-มอบให้คุณ)
2. [ติดตั้งและเปิดใช้งาน](#2-ติดตั้งและเปิดใช้งาน)
3. [Credential และ secrets](#3-credential-และ-secrets)
4. [อ้างอิง operations](#4-อ้างอิง-operations)
5. [ตัวอย่างการใช้งาน](#5-ตัวอย่างการใช้งาน)
6. [ความปลอดภัยและ OAuth scopes](#6-ความปลอดภัยและ-oauth-scopes)
7. [ดูเพิ่มเติม](#7-ดูเพิ่มเติม)

---

## 1. สิ่งที่ปลั๊กอิน gws มอบให้คุณ

กลุ่ม `gws` ประกอบด้วยปลั๊กอิน framework สี่ตัว — ทั้งหมดมี `kind = "gws"` — ที่ให้ agents และ CLI `bwoc gws` อ่านข้อมูล Google Workspace ได้โดยไม่ต้องเขียน credential handling เอง

| ปลั๊กอิน | ครอบคลุม |
|---|---|
| `gws-auth` | ฐาน OAuth2: การ resolve token, Bearer header, refresh อัตโนมัติ และ verb `status` ที่รายงานสถานะ auth โดยไม่เปิดเผยค่า token เลย ปลั๊กอินบริการทุกตัว source auth มาจากที่นี่ |
| `gws-drive` | Google Drive: แสดงรายการไฟล์ และอ่าน metadata ของไฟล์เดียว |
| `gws-gmail` | Gmail: ค้นหา thread, ดู thread เดียว และแสดงรายการ label |
| `gws-calendar` | Google Calendar: แสดงรายการ calendar และ events |

ทั้งสี่ตัวเป็น **read-mostly** ไม่มี verb ใดส่งเมล สร้าง event หรืออัปโหลดไฟล์ การเขียนถูกเลื่อนไปยัง release ในอนาคตอย่างชัดเจน และทุก verb ที่เขียนในอนาคตจะต้องผ่าน operator confirmation ก่อนดำเนินการ

กลุ่ม `gws` แยกจากกลุ่ม `gcloud` `gcloud` เข้าถึง Google Cloud Platform infrastructure ผ่าน CLI `gcloud` ด้วย service-account credential `gws` เข้าถึงแอปพลิเคชัน productivity (Drive, Gmail, Calendar) ผ่าน Workspace REST API ด้วย OAuth2 user-consent scopes ซึ่งเป็น authentication family และ surface คนละชนิดกัน

---

## 2. ติดตั้งและเปิดใช้งาน

### 2.1 ติดตั้งปลั๊กอิน

ปลั๊กอิน `gws` มาพร้อมกับ framework source ติดตั้งแต่ละตัวด้วย `bwoc plugin install` โดยชี้ไปที่ local path ใน framework clone หรือจาก Git URL สาธารณะ:

```bash
# จาก local framework clone ที่ ./bwoc-framework
bwoc plugin install ./bwoc-framework/modules/plugins/gws/gws-auth
bwoc plugin install ./bwoc-framework/modules/plugins/gws/gws-drive
bwoc plugin install ./bwoc-framework/modules/plugins/gws/gws-gmail
bwoc plugin install ./bwoc-framework/modules/plugins/gws/gws-calendar
```

หลังติดตั้ง แต่ละปลั๊กอินจะอยู่ที่ `modules/plugins/gws-auth/`, `modules/plugins/gws-drive/` และต่อๆ ไป การติดตั้งไม่ได้เปิดใช้อัตโนมัติ — นั่นเป็นขั้นตอนแยกต่างหากโดยเจตนา

### 2.2 เปิดใช้งานใน workspace.toml

เพิ่ม entry สำหรับแต่ละปลั๊กอินที่ต้องการเปิดใช้ ต้องเปิด `gws-auth` เสมอเมื่อเปิดใช้ปลั๊กอินบริการใดก็ตาม เพราะปลั๊กอินบริการ source OAuth helpers มาจากมัน

```toml
# workspace.toml

[plugins.gws-auth]
enabled = true

[plugins.gws-drive]
enabled = true

[plugins.gws-gmail]
enabled = true

[plugins.gws-calendar]
enabled = true
```

key `enabled` คือ config field เดียวระดับ workspace ที่ปลั๊กอิน `gws` ใช้ ไม่มี `[config.schema]` สำหรับปลั๊กอินเหล่านี้ — การ resolve credential ขับเคลื่อนด้วย environment ไม่ใช่ config (ดู [หัวข้อ 3](#3-credential-และ-secrets))

หรือจะสลับด้วย CLI:

```bash
bwoc plugin enable gws-auth
bwoc plugin enable gws-drive
```

ตรวจสอบสิ่งที่ติดตั้งและเปิดใช้:

```bash
bwoc plugin list --kind gws
```

### 2.3 Dependencies ที่จำเป็นในขณะ runtime

ปลั๊กอินบริการแต่ละตัวต้องการ `jq` และ `curl` ใน `PATH` `gws-auth` เพียงอย่างเดียวต้องการแค่ `jq` ปลั๊กอินจะตรวจสอบเมื่อถูกเรียก และหากขาดสิ่งใดจะแสดงข้อความที่ชัดเจน

---

## 3. Credential และ secrets

### 3.1 Token ถึงปลั๊กอินอย่างไร

ปลั๊กอิน `gws-auth` resolve OAuth2 access token จากสองแหล่ง ตรวจสอบตามลำดับนี้:

1. **Environment variable `BWOC_GWS_TOKEN`** — ตั้งค่า token โดยตรงใน environment เป็นวิธีที่แนะนำสำหรับ CI และ context ชั่วคราว Token จาก env ไม่มี metadata (scopes, expiry, account)
2. **Token file** ที่ `<workspace>/.bwoc/secrets/gws-token.json` — ไฟล์ JSON ที่เก็บ access token และ optionally refresh token, expiry, scopes และข้อมูล account ไฟล์ต้องเป็น `chmod 600` (owner อ่านได้เท่านั้น) ปลั๊กอินปฏิเสธไฟล์ที่ group หรือ world อ่านได้

รูปแบบ token file:

```json
{
  "access_token": "ya29.TOKEN_ของคุณ",
  "refresh_token": "1//REFRESH_TOKEN_ของคุณ",
  "client_id": "CLIENT_ID_ของคุณ.apps.googleusercontent.com",
  "client_secret": "CLIENT_SECRET_ของคุณ",
  "expiry": "2026-06-08T12:00:00Z",
  "scopes": [
    "https://www.googleapis.com/auth/drive.readonly",
    "https://www.googleapis.com/auth/gmail.readonly",
    "https://www.googleapis.com/auth/calendar.readonly"
  ],
  "account": "you@example.com"
}
```

มีเพียง `access_token` ที่จำเป็น เมื่อมี `refresh_token`, `client_id` และ `client_secret` ครบทั้งสามและ token หมดอายุ `gws-auth` จะ refresh อัตโนมัติและเขียนไฟล์ใหม่ใน place — ไม่ต้อง rotate token เอง

directory `.bwoc/secrets/` ถูก gitignore ในระดับ workspace อย่าเด็ดขาดที่ commit token file

### 3.2 ขอรับ token

การขอ token — OAuth consent flow — เป็น action ของ operator นอก BWOC วิธีทั่วไป:

1. สร้าง OAuth 2.0 Client ID ใน Google Cloud Console (application type: Desktop)
2. ทำ consent flow ด้วยเครื่องมือเช่น `oauth2l` หรือ script สั้นๆ ที่ใช้ Google Auth Library โดยขอเฉพาะ scope ที่ต้องการ (ดู [หัวข้อ 6](#6-ความปลอดภัยและ-oauth-scopes))
3. เขียน `access_token` ที่ได้ (และ optionally refresh trio) ลงใน `.bwoc/secrets/gws-token.json` พร้อม `chmod 600`

หรือตั้ง `BWOC_GWS_TOKEN` เป็น access token อายุสั้นสำหรับการรันครั้งเดียว

### 3.3 ตรวจสอบสถานะ auth

```bash
bwoc gws status
```

คำสั่งนี้เรียก verb `status` ของ `gws-auth` รายงานว่ามี token อยู่ไหม มาจากแหล่งไหน scopes ที่ได้รับ account และ token หมดอายุหรือ refresh ได้หรือไม่ — แต่ **ไม่เคยพิมพ์ค่า token เลย**

ตัวอย่าง output:

```json
{
  "ok": true,
  "plugin": "gws-auth",
  "operation": "status",
  "active_source": "secrets-file",
  "has_token": true,
  "account": "you@example.com",
  "scopes": [
    "https://www.googleapis.com/auth/drive.readonly"
  ],
  "expiry": "2026-06-08T12:00:00Z",
  "expired": false,
  "refreshable": true
}
```

---

## 4. อ้างอิง operations

### 4.1 gws-auth

| Verb | สิ่งที่ทำ |
|---|---|
| `status` | รายงานการมีอยู่ของ token, แหล่งที่ active, scopes ที่ได้รับ, account, expiry และ refresh ได้หรือไม่ Read-only ไม่เคยพิมพ์ค่า token |

### 4.2 gws-drive

| Verb | สิ่งที่ทำ |
|---|---|
| `list` | แสดงรายการไฟล์ใน Drive รับ `query` (Drive search syntax) และ `max` (default 100) แบบ optional paginate อัตโนมัติ |
| `get` | คืน metadata ของไฟล์เดียวตาม `file_id` ไม่เคย download เนื้อหา |

แต่ละผลลัพธ์ถูก project เป็น Drive file shape มาตรฐาน: `file_id`, `name`, `mime_type`, `modified_time` และ optionally `owners` กับ `web_view_link`

### 4.3 gws-gmail

| Verb | Alias | สิ่งที่ทำ |
|---|---|---|
| `search` | `threads` | ค้นหา Gmail thread รับ `query` (Gmail search syntax เช่น `from:me is:unread`) และ `max` (default 100) แบบ optional เติม subject, sender และ timestamp ให้แต่ละผลลัพธ์ |
| `show` | `message`, `messages` | คืน metadata ของ thread เดียวตาม `thread_id` |
| `labels` | — | แสดงรายการ label ทั้งหมดของ account (ระบบและ user สร้าง) |

แต่ละ thread ถูก project เป็น Gmail thread shape มาตรฐาน: `thread_id`, `subject`, `from`, `last_message_time` และ optionally `snippet` กับ `labels`

### 4.4 gws-calendar

| Verb | Alias | สิ่งที่ทำ |
|---|---|---|
| `calendars` | `list` | แสดงรายการ calendar ที่ token เห็นได้ คืน `calendar_id`, `summary` และ optionally `primary` กับ `access_role` |
| `events` | — | แสดงรายการ events ใน calendar รับ `calendar_id` (default `primary`) และ `max` (default 100) paginate และ expand recurring events |

แต่ละ event ถูก project เป็น Calendar event shape มาตรฐาน: `event_id`, `calendar_id`, `summary`, `start`, `end` และ optionally `attendees_count` event ตลอดวันใช้วันที่ event ที่มีเวลาใช้ datetime

### 4.5 รหัส error (ปลั๊กอิน gws ทั้งหมด)

ปลั๊กอิน `gws` ทุกตัวใช้ exit code ชุดเดียวกัน:

| Code | ความหมาย |
|---|---|
| `0` | สำเร็จ — JSON object หนึ่งชิ้นบน stdout |
| `1` | ขาด dependency (`jq` หรือ `curl` ไม่อยู่ใน PATH) |
| `2` | Usage error — operation ไม่รู้จัก, field จำเป็นขาดหาย (เช่น `file_id`, `thread_id`) หรือไม่พบ token |
| `3` | Auth หรือ scope error — HTTP 401 (token ไม่ valid/หมดอายุ) หรือ 403 (token ขาด scope ที่ต้องการ) |
| `4` | Rate limited — HTTP 429 หลังพยายาม backoff สี่ครั้ง |
| `5` | ไม่พบ — HTTP 404 |
| `6` | Transport error หรือ HTTP status ที่ไม่คาดคิด |

---

## 5. ตัวอย่างการใช้งาน

ตัวอย่างต่อไปนี้สมมติว่า `gws-auth` และ `gws-drive` เปิดใช้งานและมี token ที่ valid อยู่แล้ว

ตรวจสอบสถานะ auth ก่อน:

```bash
bwoc gws status
```

แสดงรายการ Google Docs 10 อันดับแรกที่แก้ไขล่าสุดใน Drive:

```bash
echo '{"operation":"list","query":"mimeType='\''application/vnd.google-apps.document'\''","max":10}' \
  | bwoc gws drive
```

หรือส่ง operation ผ่าน environment:

```bash
BWOC_GWS_OPERATION=list bwoc gws drive \
  <<< '{"query":"mimeType='\''application/vnd.google-apps.document'\''","max":10}'
```

ดึง metadata ของไฟล์ที่ระบุ:

```bash
echo '{"operation":"get","file_id":"1AbC_dEfGhIjKlMnOpQrStUvWxYz"}' | bwoc gws drive
```

ค้นหา Gmail เพื่อหาข้อความที่ยังไม่อ่านจาก sender ที่ระบุ:

```bash
echo '{"operation":"search","query":"from:alice@example.com is:unread","max":5}' | bwoc gws gmail
```

แสดงรายการ events ที่กำลังจะมาถึงใน primary calendar:

```bash
echo '{"operation":"events","calendar_id":"primary","max":10}' | bwoc gws calendar
```

ทุก verb ส่ง JSON object หนึ่งชิ้นบน stdout เมื่อ error ข้อความ plain-text จะไปที่ stderr และ process exit ด้วย non-zero code

---

## 6. ความปลอดภัยและ OAuth scopes

**Scope เป็นรายบริการและผูกกับ consent** Token ที่ได้รับ `drive.readonly` ไม่สามารถอ่าน Gmail หรือ Calendar ได้ แต่ละปลั๊กอินต้องการ scope เดียว:

| ปลั๊กอิน | Scope ที่จำเป็น |
|---|---|
| `gws-drive` | `https://www.googleapis.com/auth/drive.readonly` |
| `gws-gmail` | `https://www.googleapis.com/auth/gmail.readonly` |
| `gws-calendar` | `https://www.googleapis.com/auth/calendar.readonly` |

ขอเฉพาะ scope ที่ agents ของคุณต้องใช้จริงๆ หากการเรียกไปยังบริการที่ token ไม่มี scope ปลั๊กอินจะแสดง `token lacks the required scope for <service>` แทนที่จะเป็น HTTP 403 เปล่าๆ เพื่อให้ชัดว่าต้องเพิ่ม scope อะไร

**Token ไม่เคยถูก log หรือพิมพ์** helpers ของ `gws-auth` ส่ง access token เฉพาะในรูป `Authorization: Bearer` header ใน curl outbound calls เท่านั้น ไม่เคยปรากฏใน plugin output, JSON envelope หรือ log ใดๆ verb `status` รายงาน metadata (แหล่ง, scopes, account, expiry) — ไม่ใช่ค่า token

**Token file ต้อง owner-only** ปลั๊กอินปฏิเสธการอ่าน `.bwoc/secrets/gws-token.json` หากไฟล์ group หรือ world อ่านได้ รัน `chmod 600 .bwoc/secrets/gws-token.json` หลังสร้างหรืออัปเดตไฟล์

**Directory `.bwoc/secrets/` ถูก gitignore** อย่าเด็ดขาดที่ commit token file `auth.toml` ในซอร์สปลั๊กอินประกาศเฉพาะ shape ของ contract (ชื่อ field, ชื่อ env var, path ไฟล์) ไม่มีค่าใดๆ `bwoc check` ปฏิเสธ `auth.toml` ที่มี token value ที่ไม่ว่างเปล่า

**Token refresh อัตโนมัติ** เมื่อ token file มี expiry ที่ผ่านไปแล้วและมี refresh trio ครบ (`refresh_token` + `client_id` + `client_secret`) `gws-auth` จะ refresh token อัตโนมัติก่อนแต่ละ request และเขียนไฟล์ใหม่แบบ atomic (ผ่าน temp file ที่ `chmod 600` + `mv`) หากจำเป็นต้อง refresh แต่ทำไม่ได้ (env token หรือขาด refresh trio) request จะดำเนินต่อด้วย token เดิมและ HTTP 401 ที่ได้จะแสดงพร้อมข้อความ "re-authorize" ที่ชัดเจน

---

## 7. ดูเพิ่มเติม

- [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) — plugin spec ครบถ้วน: kinds, รูปแบบ manifest, lifecycle hooks, การ load และ Workspace Resource Schema ที่ปลั๊กอิน `gws` ส่งออกมา
- [ซอร์สโค้ดปลั๊กอิน gws](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/gws) — `manifest.toml`, `SPEC.md` และ entry scripts ของปลั๊กอินทั้งสี่
- [`../backends/HANDBOOK.en.md`](../backends/HANDBOOK.en.md) — การตั้งค่า backend เกี่ยวข้องหากคุณรัน agent ที่จะเรียก verbs ของ `bwoc gws`
- [`../security/HANDBOOK.en.md`](../security/HANDBOOK.en.md) — threat model ของ workspace และกฎ secret-store ที่ควบคุม `.bwoc/secrets/`
- [`../glossary.en.md`](../glossary.en.md) — ค้นหาคำศัพท์
