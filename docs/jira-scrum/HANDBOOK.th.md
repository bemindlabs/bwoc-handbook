# Jira และ Scrum กับ BWOC

> Language toggle: [English (canonical)](HANDBOOK.en.md) | **ภาษาไทย**
>
> บทนี้ครอบคลุมสองส่วนที่ทำให้ agent BWOC ทำงานร่วมกับ Jira ได้อย่างเป็นธรรมชาติ ได้แก่ plugin `jira-cloud-rest` และ skill `scrum-via-jira` อ่านส่วน plugin เพื่อเข้าใจชั้น integration อ่านส่วน skill เพื่อดูว่า agent ของคุณทำอะไรได้บ้าง

---

## สารบัญ

1. [สองส่วนและความสัมพันธ์](#1-สองส่วนและความสัมพันธ์)
2. [Plugin jira-cloud-rest — ชั้น integration](#2-plugin-jira-cloud-rest--ชั้น-integration)
   - [2.1 ติดตั้งและเปิดใช้งาน](#21-ติดตั้งและเปิดใช้งาน)
   - [2.2 Config — project key](#22-config--project-key)
   - [2.3 Credentials — BWOC_JIRA_* และ secrets.toml](#23-credentials--bwoc_jira_-และ-secretstoml)
   - [2.4 Verb ที่ plugin เปิดเผย](#24-verb-ที่-plugin-เปิดเผย)
   - [2.5 Bidirectional sync และ gated transitions](#25-bidirectional-sync-และ-gated-transitions)
3. [Skill scrum-via-jira — คำศัพท์ scrum สำหรับ agent](#3-skill-scrum-via-jira--คำศัพท์-scrum-สำหรับ-agent)
   - [3.1 เปิดใช้ skill บน agent](#31-เปิดใช้-skill-บน-agent)
   - [3.2 Scrum verb ทั้งหก](#32-scrum-verb-ทั้งหก)
4. [โมเดล kind-dependency — เปลี่ยน adapter โดยไม่แตะ skill](#4-โมเดล-kind-dependency--เปลี่ยน-adapter-โดยไม่แตะ-skill)
5. [ตัวอย่าง Sprint workflow](#5-ตัวอย่าง-sprint-workflow)
6. [ดูเพิ่มเติม](#6-ดูเพิ่มเติม)

---

## 1. สองส่วนและความสัมพันธ์

การรัน scrum workflow ที่ใช้ Jira เป็น backend ใน BWOC ต้องการสองโมดูลที่ทำงานคนละระดับกัน:

| โมดูล | ประเภท | ระดับ | ใครเปิดใช้ |
|---|---|---|---|
| `jira-cloud-rest` | Plugin (`kind = "jira"`) | Framework integration — รับผิดชอบ REST, auth, JQL, rate limits, sync ledger | Operator ตั้งค่าใน `workspace.toml` |
| `scrum-via-jira` | Skill (maturity L1) | Agent capability — รับผิดชอบ scrum semantics (sprints, stories, backlog) | Agent author ตั้งค่าใน `config.manifest.json` |

Plugin คือ HTTP adapter มันรู้จัก Atlassian Cloud REST v3, วิธี authenticate, วิธี scope JQL ให้อยู่ใน project ของคุณ, วิธีจัดการ `429` backoff, และวิธีเขียน sync ledger ที่ `.scrum/jira-sync.json` มันไม่รู้เรื่อง scrum เลย

Skill คือชั้น scrum vocabulary มันรู้ว่า "open a sprint" หรือ "transition a story" หมายถึงอะไร และมอบงานเชิงกล — การเรียก HTTP จริง — ให้กับ plugin ที่มี `kind = "jira"` ที่เปิดใช้งานใน workspace Skill ไม่ถือ credentials และไม่แตะ sync ledger โดยตรง

การพึ่งพาเป็นทิศทางเดียว: skill เรียก verb ของ plugin; plugin ไม่รู้ว่า skill มีอยู่ Plugin ที่มี `kind = "jira"` ทำงานได้อย่างสมบูรณ์แม้ไม่มี skill — operator ขับเคลื่อนทุกอย่างผ่าน `bwoc jira` CLI โดยตรงก็ได้

---

## 2. Plugin jira-cloud-rest — ชั้น integration

`jira-cloud-rest` คือ reference implementation ของ `jira` plugin kind มันเป็น write-capable plugin ตัวแรกของ framework: ต่างจาก audit หรือ reporting plugin ที่อ่านอย่างเดียว ตัวนี้ **อ่านและเขียน** external system of record — Jira Cloud — นี่คือเหตุผลที่การเขียนต้องผ่าน operator confirmation

Source: [modules/plugins/jira-cloud-rest](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/jira-cloud-rest)

### 2.1 ติดตั้งและเปิดใช้งาน

ถ้า plugin ยังไม่ได้อยู่ใน `modules/plugins/jira-cloud-rest/` ของ workspace ให้ติดตั้ง:

```bash
bwoc plugin install https://github.com/bemindlabs/BWOC-Framework.git#main
# หรือจาก local path ถ้า clone framework ไว้แล้ว:
bwoc plugin install ./projects/bwoc-framwork/modules/plugins/jira-cloud-rest/
```

จากนั้นเปิดใช้งานโดยเพิ่ม block ใน `workspace.toml`:

```toml
[plugins.jira-cloud-rest]
enabled = true
project = "MYPROJ"   # Jira project key ของคุณ — field config เดียวที่ต้องกำหนด
```

ตรวจสอบว่า manifest ผ่าน neutrality และ schema check:

```bash
bwoc check --all
```

### 2.2 Config — project key

Config key ที่ประกาศไว้มีเพียงตัวเดียวคือ `project` — Jira project key ที่ทุก read จะถูก scope ไว้ (เช่น `"BWOC"`, `"MYTEAM"`) ไม่สามารถละเว้นได้ ทุก JQL query ที่ plugin ออกจะถูกผูกไว้กับ project นี้โดยอัตโนมัติ; query ที่พยายาม escape ออกไปจะถูกปฏิเสธหรือเพิ่ม constraint

ไม่มีอะไรอื่นใน config โดยเจตนา Credentials ไม่ใช่ config — ควรอยู่ใน environment หรือไฟล์ secrets ที่ gitignore (ดูด้านล่าง)

### 2.3 Credentials — BWOC_JIRA_* และ secrets.toml

Plugin authenticate กับ Atlassian Cloud ด้วย HTTP Basic: email บัญชีของคุณเป็น username และ API token เป็น password ไปยัง base URL ของ site

สาม environment variable ส่งข้อมูลเหล่านี้ Plugin อ่านที่ runtime — อันแรกที่เจอชนะ:

| Variable | ที่เก็บ |
|---|---|
| `BWOC_JIRA_EMAIL` | Email บัญชี Atlassian ของคุณ |
| `BWOC_JIRA_TOKEN` | Atlassian API token ของคุณ — นี่คือ secret |
| `BWOC_JIRA_BASE_URL` | Site URL เช่น `https://myteam.atlassian.net` |

ถ้าไม่ได้ set environment variable plugin จะ fallback ไปที่ `.bwoc/secrets.toml` — ไฟล์ที่ gitignore แล้ว มี `chmod 600` อยู่ใน workspace root ตั้งค่าแบบนี้:

```toml
# .bwoc/secrets.toml — ห้าม commit ไฟล์นี้
[jira]
email    = "you@example.com"
token    = "your-api-token"
base_url = "https://myteam.atlassian.net"
```

Token ถูกส่งตรงไปที่ `curl -u` โดยไม่เคย echo, ไม่เคยเขียนลงไฟล์ output ใด และไม่เคยรวมใน JSON response ถ้าขาด input ใดใดหนึ่งใน 3 ตัว plugin จะ fail fast พร้อม diagnostic ระบุชัดว่าขาด variable ไหน — ไม่มีทางดำเนินต่อด้วย auth ที่ไม่ครบ

หากต้องการสร้าง API token ไปที่ id.atlassian.com > Security > API tokens

### 2.4 Verb ที่ plugin เปิดเผย

`bwoc jira` CLI ส่งต่อ live verb สามตัวไปยัง plugin adapter:

| Verb | ทิศทาง | ทำอะไร | Gate |
|---|---|---|---|
| `query` | read | รัน project-scoped JQL ต่อ `/rest/api/3/search/jql`; คืน paginated issues ตาม Issue Mapping Schema | ไม่มี — reads ไม่มี gate |
| `transition` | write | อ่าน current status ของ issue จากนั้น POST ไปที่ `/transitions` เพื่อย้ายไปยัง target status; idempotent (ถ้า status ตรงอยู่แล้วคือ no-op) | ต้องการ operator confirmation |
| `sync` | read/write | อ่าน sync ledger และ reconcile กับ Jira; `--dry-run` แสดงแผนโดยไม่ apply | apply ต้องผ่าน gate |

Verb เพิ่มเติมอีกสองตัว (`link`, `unlink`, `status`) เป็น offline — แตะเฉพาะ local sync ledger ไม่ถึง adapter

Result set จาก `query` จำกัดที่ 100 items ต่อ page Response มี `next_page_token` และ flag `is_last` สำหรับ paging ในอนาคต

### 2.5 Bidirectional sync และ gated transitions

`jira` kind เป็น plugin kind แรกของ BWOC ที่เขียนไปยัง external system ทำให้มีการควบคุมเชิงโครงสร้างสองอย่าง:

**Operator-confirmation gate.** ทุก write verb (`transition`, `sync` apply) ต้องการ confirmation ชัดเจนที่ `bwoc jira` CLI ก่อน adapter จะถูก invoke Gate แสดง effect ที่แน่นอน — target issue, current status, target status — ก่อนลงมือ Default คือ No Agent headless ต้องส่ง `--yes` อย่างชัดเจน และเฉพาะเมื่อ operator authorize action นั้นแล้วเท่านั้น

**Sync ledger.** Plugin ดูแล `.scrum/jira-sync.json` — mapping ระหว่าง scrum stories ใน local กับ Jira issues แต่ละ entry บันทึก `issue_key`, `project`, `summary`, `status`, และ watermark `last_synced` Watermark ขับ per-field last-writer-wins conflict detection การได้รับ `401`/`403` จาก Jira แสดงขึ้นว่า "หมุน token" — ไม่สร้าง sync conflict ที่เป็นเท็จ

Sync ใน v0.1.0 สร้าง contract, auth, และ read path; full bidirectional resolution engine เลื่อนไปเป็น release ถัดไป Envelope shape ยืนยันแล้ว CLI contract จึง stable ตั้งแต่ตอนนี้

---

## 3. Skill scrum-via-jira — คำศัพท์ scrum สำหรับ agent

`scrum-via-jira` คือ skill-on-plugin skill ตัวแรกของ framework มอบ operation scrum ระดับสูง 6 ตัวให้ agent แต่ละตัว delegate งาน HTTP ให้กับ `jira`-kind plugin ผ่าน `bwoc jira` verb Skill รับผิดชอบ scrum semantics; plugin รับผิดชอบ integration

Source: [modules/skills/scrum-via-jira](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/skills/scrum-via-jira)

### 3.1 เปิดใช้ skill บน agent

เพิ่ม skill ใน `config.manifest.json` ของ agent:

```json
{
  "skills": {
    "framework": [
      { "name": "scrum-via-jira", "version": ">=0.1.0", "enabled": true }
    ]
  }
}
```

ตอน spawn framework จะตรวจว่ามี enabled `jira`-kind plugin อยู่ใน workspace หรือไม่ ถ้าไม่มี spawn จะ fail ทันทีพร้อม diagnostic ระบุ kind ที่ขาดหาย Agent ไม่มีทาง wired บางส่วน

รัน verify gate ก่อน spawn เพื่อ catch ปัญหานี้ได้เร็วขึ้น:

```bash
bwoc skill verify scrum-via-jira
```

### 3.2 Scrum verb ทั้งหก

Skill เปิดเผย operation 6 ตัว สองตัวเป็น read-only ไม่มี gate; สี่ตัวเป็น write ที่รับ operator-confirmation gate จาก plugin:

| Verb | Scrum intent | Plugin verb ที่ใช้ใต้ hood | ทิศทาง | Gate |
|---|---|---|---|---|
| `propose-sprint` | รวบรวม backlog stories และ emit sprint ที่เสนอ | `bwoc jira query` | read | ไม่มี |
| `open-sprint` | เปิดใช้ sprint ที่ตกลงแล้ว; assign stories | `bwoc jira link` แล้ว `bwoc jira sync` | write | operator confirmation |
| `transition-story` | ย้าย story หนึ่งตัวใน scrum lifecycle (`backlog → in_progress → done`) | `bwoc jira transition` | write | operator confirmation |
| `sync-backlog` | Reconcile backlog กับ Jira; ดึง external changes และ push local ones | `bwoc jira sync` (มี `--dry-run` preview ก่อน) | read/write | apply ต้องผ่าน gate |
| `close-sprint` | Finalize sprint; transition stories ที่เหลือและบันทึก close | `bwoc jira transition` + `bwoc jira sync` | write | operator confirmation |
| `list-active-sprints` | แสดง sprint ที่เปิดอยู่สำหรับ project ที่ config ไว้ | `bwoc jira query` | read | ไม่มี |

Skill ไม่เพิ่ม gate ที่สองซ้อนบน gate ของ plugin และไม่ลบ gate ใด Write verb มีจุด confirmation เพียงหนึ่งจุด — ที่ boundary ของ `bwoc jira` CLI

Skill ไม่แตะ `.scrum/jira-sync.json` โดยตรงและไม่ถือ credentials มันเข้าถึง sync ledger เฉพาะผ่าน plugin เท่านั้น Credentials resolve ภายใน plugin จาก `BWOC_JIRA_*` env หรือ `.bwoc/secrets.toml` ไม่ผ่าน skill

ตอน `init` skill resolve `jira`-kind plugin dependency และ cache dispatch handle ไม่มี REST call ตอน init — agent ไม่มีทาง half-wired และ init รวดเร็วเสมอ Teardown เป็น no-op; skill ไม่มี external state

---

## 4. โมเดล kind-dependency — เปลี่ยน adapter โดยไม่แตะ skill

`scrum-via-jira` ประกาศ dependency ใน `manifest.toml` ดังนี้:

```toml
[contract]
requires         = []
requires_plugins = ["jira"]
```

ค่า `"jira"` คือ plugin **kind** ไม่ใช่ชื่อ plugin Skill ไม่รู้จัก `jira-cloud-rest` โดยเฉพาะ มันพึ่งพาเฉพาะว่ามี plugin ที่ enabled อยู่ซึ่งประกาศ `kind = "jira"` ใน manifest

ผลลัพธ์:

- วันนี้ workspace รัน `jira-cloud-rest` (Atlassian Cloud REST v3)
- พรุ่งนี้มีคนสร้าง `jira-server-rest` สำหรับ Jira Server แบบ self-hosted — หรือ `jira-mock` สำหรับ testing
- การ swap adapter คือการเปลี่ยน config ของ workspace: disable plugin เก่า, enable plugin ใหม่ ไม่ต้องแตะ skill หรือ agent ใดที่ใช้มัน

การแยกถูก enforce เชิงโครงสร้าง `requires_plugins` ตั้งชื่อ kind (จาก [Plugin Kinds](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md)); `requires` ตั้งชื่อ skill names `"jira"` แบบ bare จึงไม่คลุมเครือ — สอง field มี namespace ต่างกัน `bwoc check` validate ที่ manifest-read time ว่าทุกค่าใน `requires_plugins` คือ valid kind enum โดยไม่ต้องการให้ plugin enabled ตอนนั้น การ enablement ตรวจสอบตอน agent spawn

โมเดลเต็มรูปแบบ — ว่าทำไม dedicated field ดีกว่าการ overload `requires` — อยู่ใน [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md)

---

## 5. ตัวอย่าง Sprint workflow

ตัวอย่างนี้แสดง agent ที่รัน sprint เดียวตั้งแต่เสนอจนปิด Operator ยืนยันแต่ละ write

**ก่อนเริ่ม** ตรวจสอบ:

1. `jira-cloud-rest` เปิดใช้งานใน `workspace.toml` พร้อม `project` key ที่กำหนดแล้ว
2. `BWOC_JIRA_EMAIL`, `BWOC_JIRA_TOKEN`, และ `BWOC_JIRA_BASE_URL` อยู่ใน environment (หรือใน `.bwoc/secrets.toml`)
3. `scrum-via-jira` เปิดใช้งานใน `config.manifest.json` ของ agent
4. `bwoc skill verify scrum-via-jira` exit 0

```bash
# 1. ตรวจดู sprint ที่กำลังรันอยู่
bwoc jira query --jql "sprint in openSprints()"

# 2. เสนอ sprint — agent อ่าน backlog และร่างโครงสร้าง
#    (propose-sprint → bwoc jira query, read-only, ไม่มี gate)
bwoc skill invoke scrum-via-jira propose-sprint

# 3. เปิด sprint — assign stories ที่เลือกไว้ให้ sprint
#    (open-sprint → bwoc jira link + sync, write, มี gate)
bwoc skill invoke scrum-via-jira open-sprint --stories MYPROJ-10,MYPROJ-11,MYPROJ-12

# 4. ระหว่าง sprint — ย้าย story ไปที่ in-progress
#    (transition-story → bwoc jira transition, write, มี gate)
bwoc skill invoke scrum-via-jira transition-story --story MYPROJ-10 --to in_progress

# 5. ดึง status change ที่ teammate แก้โดยตรงใน Jira
#    (sync-backlog → bwoc jira sync --dry-run ก่อน แล้ว apply, มี gate)
bwoc skill invoke scrum-via-jira sync-backlog

# 6. ปิด sprint เมื่อ stories เสร็จครบ
#    (close-sprint → bwoc jira transition + sync, write, มี gate)
bwoc skill invoke scrum-via-jira close-sprint
```

ที่ขั้นตอน 3, 4, 5 (apply), และ 6 `bwoc jira` CLI แสดงแผนการที่ชัดเจนและ prompt `y/N` ก่อนแตะ Jira ใน headless agent flow operator pre-authorize ขั้นตอนเหล่านี้ด้วย `--yes` ใน invocation context

**ถ้า transition อยู่ที่ target status แล้ว** (เช่น story ถูกย้ายใน Jira ก่อนที่ `transition-story` จะรัน) adapter detect สิ่งนี้และคืน no-op success — ไม่มี duplicate transition, ไม่มี error

**ถ้า Jira คืน `429 Too Many Requests`** adapter ทำตาม `Retry-After` header และ retry ได้ถึง 4 ครั้งก่อน surface retryable error Write verb ทั้งหมดเป็น idempotent ดังนั้น retry หลัง transient failure จึงปลอดภัย

---

## 6. ดูเพิ่มเติม

**Framework source (public GitHub):**

- [jira-cloud-rest plugin](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/plugins/jira-cloud-rest) — manifest, SPEC, และ entry point `jira.sh`
- [scrum-via-jira skill](https://github.com/bemindlabs/BWOC-Framework/tree/main/modules/skills/scrum-via-jira) — manifest และ SPEC
- [PLUGINS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/PLUGINS.en.md) — `jira` kind row, Issue Mapping Schema, write-verb operator-confirm gate contract
- [SKILLS.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/SKILLS.en.md) — skill-on-plugin dependency model; `requires_plugins`

**บทอื่นในคู่มือ:**

- [../slots/HANDBOOK.en.md](../slots/HANDBOOK.en.md) — วิธีเพิ่มและ configure skill ในไฟล์ slot ของ agent
- [../glossary.en.md](../glossary.en.md) — ค้นหาคำศัพท์ รวมถึง plugin, skill, kind, maturity

---

*คู่มือนี้ดูแลควบคู่กับ framework เมื่อ behavior ที่อธิบายที่นี่ขัดแย้งกับ [framework repo](https://github.com/bemindlabs/BWOC-Framework) repo คือคำตอบที่ถูก — กรุณาเปิด fix*
