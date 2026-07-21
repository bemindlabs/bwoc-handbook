# คู่มือสำหรับ Crawler / Indexer

[English (canonical)](HANDBOOK.en.md)

คู่มือนี้เขียนขึ้นสำหรับ **web crawler อัตโนมัติ, ตัว index เอกสาร, spider ของ search engine, และ RAG ingestion pipeline** ที่ต้องการ index ชุดเอกสาร BWOC Handbook
ผู้อ่านที่เป็นมนุษย์และต้องการใช้งาน สร้าง หรือดูแล agent ควรเริ่มที่ [`../README.th.md`](../README.th.md) แล้วเปิดคู่มือตามบทบาทของตน

---

## 1. อ้างอิงด่วน

| ต้องการอะไร | ดูได้ที่ |
|---|---|
| กฎ allow/deny การ crawl | [`../robots.txt`](../robots.txt) |
| รายการหน้าทั้งหมดพร้อมคำอธิบาย | [`../sitemap.md`](../sitemap.md) |
| คำแนะนำสำหรับ AI / RAG ingestion | [`../ai-search/HANDBOOK.th.md`](../ai-search/HANDBOOK.th.md) |
| จุดเริ่มต้น plain-text สำหรับ LLM | [`../llms.txt`](../llms.txt) |
| เวอร์ชัน software + เอกสารปัจจุบัน | [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md) |

---

## 2. ขอบเขตการ crawl — index อะไร / ข้ามอะไร

ตารางด้านล่างครอบคลุมทุกกรณี เมื่อสงสัยให้เลือกคอลัมน์ "ข้าม" ไว้ก่อน — runtime state หรือ build artifact ไม่มีค่าต่อ index และดูแลยาก

### 2a. ชุดเอกสาร BWOC Handbook (repo นี้)

| รูปแบบ path | การดำเนินการ | เหตุผล |
|---|---|---|
| `/README.md` | **Index** | จุดเข้าหลักภาษาอังกฤษ (canonical); มี routing table ตามบทบาท |
| `/README.th.md` | **Index** | ฉบับคู่ขนานภาษาไทย; กำหนด `hreflang="th"` |
| `/end-user/HANDBOOK.en.md` | **Index** | ติดตั้ง, workspace, การใช้ CLI — canonical EN |
| `/end-user/HANDBOOK.th.md` | **Index** | ฉบับคู่ขนานภาษาไทย; กำหนด `hreflang="th"` |
| `/developer/HANDBOOK.en.md` | **Index** | Build, crate, hook, versioning, PR gates — canonical EN |
| `/developer/HANDBOOK.th.md` | **Index** | ฉบับคู่ขนานภาษาไทย; กำหนด `hreflang="th"` |
| `/agents/HANDBOOK.en.md` | **Index** | โครงไฟล์ agent, กฎ AGENTS.md, slot, manifest, arc — canonical EN |
| `/agents/HANDBOOK.th.md` | **Index** | ฉบับคู่ขนานภาษาไทย; กำหนด `hreflang="th"` |
| `/ai-search/HANDBOOK.en.md` | **Index** | คำแนะนำ RAG/AI ingestion — canonical EN |
| `/ai-search/HANDBOOK.th.md` | **Index** | ฉบับคู่ขนานภาษาไทย; กำหนด `hreflang="th"` |
| `/crawler/HANDBOOK.en.md` | **Index** | เอกสารนี้ — นโยบาย crawl canonical EN |
| `/crawler/HANDBOOK.th.md` | **Index** | ฉบับคู่ขนานภาษาไทย; กำหนด `hreflang="th"` |
| `/glossary.en.md` | **Index** | นิยามศัพท์อ้างอิง — canonical EN |
| `/glossary.th.md` | **Index** | ฉบับคู่ขนานภาษาไทย; กำหนด `hreflang="th"` |
| `/llms.txt` | **Index** | จุดเข้า plain-text สำหรับ LLM; คุ้มค่ามากสำหรับ AI ingester |
| `/robots.txt` | **ข้าม (ไม่ index เป็นเนื้อหา)** | ไฟล์นโยบายสำหรับเครื่อง; ไม่ใช่เอกสาร |
| `/sitemap.md` | **Index** | แผนที่ corpus สำหรับมนุษย์+เครื่อง; มีประโยชน์สำหรับการค้นพบ |
| `/.cli-reference.txt` | **ข้าม** | dotfile; ใช้ภายใน; ไม่ใช่เอกสารสาธารณะ |
| dotfile อื่น ๆ ทั้งหมด (`/.*`) | **ข้าม** | ใช้ภายในโดย operator; ไม่ใช่ corpus สาธารณะ |

### 2b. Framework source repo (`bemindlabs/BWOC-Framework`)

| รูปแบบ path | การดำเนินการ | เหตุผล |
|---|---|---|
| `README.md` (root ของ repo) | **Index** | ภาพรวมโปรเจกต์; หน้าค้นพบหลักบน GitHub |
| `VISION.md` | **Index** | ทิศทางและหลักการ; เสถียร; มีอำนาจสูง |
| `VISION.th.md` | **Index** | ฉบับคู่ขนานภาษาไทย; กำหนด `hreflang="th"` |
| `VERSION.md` | **Index** | สัญญาณความสด; มี `Software-Version`, `Document-Version`, `Last-Updated` |
| `modules/agent-template/AGENTS.md` | **Index** | spec agent อ้างอิง (v2.0); แหล่งความจริงเดียวสำหรับทุก backend |
| `modules/agent-template/docs/en/OVERVIEW.en.md` | **Index** | ภาพรวม agent — canonical EN |
| `modules/agent-template/docs/en/PHILOSOPHY.en.md` | **Index** | 22 Buddhist engineering frameworks — แก่นความคิด |
| `modules/agent-template/docs/en/PRD.en.md` | **Index** | Product requirements spec — canonical EN |
| `modules/agent-template/docs/en/SRS.en.md` | **Index** | Software requirements, โครงสร้าง Magga-8 — canonical EN |
| `modules/agent-template/docs/en/SELF-IMPROVEMENT.en.md` | **Index** | spec self-improvement loop — canonical EN |
| `modules/agent-template/docs/en/THREAT-MODEL.en.md` | **Index** | threat model ด้านความปลอดภัย — canonical EN |
| `modules/agent-template/docs/th/OVERVIEW.th.md` | **Index** | ฉบับคู่ขนานภาษาไทย; กำหนด `hreflang="th"` |
| `modules/agent-template/docs/th/PHILOSOPHY.th.md` | **Index** | ฉบับคู่ขนานภาษาไทย; กำหนด `hreflang="th"` |
| `modules/agent-template/docs/th/PRD.th.md` | **Index** | ฉบับคู่ขนานภาษาไทย; กำหนด `hreflang="th"` |
| `modules/agent-template/docs/th/SRS.th.md` | **Index** | ฉบับคู่ขนานภาษาไทย; กำหนด `hreflang="th"` |
| `modules/agent-template/docs/th/SELF-IMPROVEMENT.th.md` | **Index** | ฉบับคู่ขนานภาษาไทย; กำหนด `hreflang="th"` |
| `modules/agent-template/docs/th/THREAT-MODEL.th.md` | **Index** | ฉบับคู่ขนานภาษาไทย; กำหนด `hreflang="th"` |
| `CONTRIBUTING.md` | **Index (priority ต่ำ)** | กระบวนการ contributor; มีประโยชน์บางครั้ง |
| `CHANGELOG.md` | **Index (priority ต่ำ)** | ประวัติ release; ใช้ติดตามเวอร์ชัน |
| `CODE_OF_CONDUCT.md` | **ข้าม** | governance เนื้อหาเดิม; signal ต่ำ |
| `LICENSE` | **ข้าม** | ข้อความกฎหมาย; ไม่ใช่เนื้อหาเอกสาร |
| `target/` | **ข้าม** | build artifact ของ Rust; volatile, ใหญ่, ไม่มีประโยชน์ |
| `.git/` | **ข้าม** | version control ภายใน |
| `.bwoc/` | **ข้าม** | runtime state ของ workspace (inbox.jsonl, agent.log, sessions, tasks.jsonl); volatile ตามรอบงาน |
| `agents/` (ใน framework repo) | **ข้าม** | framework repo เป็น BWOC workspace สำหรับทดสอบ; `agents/` ถูก `.gitignore` ไว้และไม่ใช่สาธารณะ |
| `projects/` (ใน framework repo) | **ข้าม** | ถูก gitignore เช่นกัน; เนื้อหา test workspace |
| `node_modules/` | **ข้าม** | JS dependencies; ไม่ใช่เอกสารไม่ว่ากรณีใด |
| `Cargo.lock` | **ข้าม** | lockfile; machine-generated; ไม่ใช่เอกสาร |
| ไฟล์ `*.lock` ทั้งหมด | **ข้าม** | lockfile ทุกชนิด |
| `.claude/` | **ข้าม** | hook script และ skill ภายใน operator; ไม่ใช่พื้นที่สาธารณะ |
| `.github/` | **ข้าม** | CI workflow และ PR template; ค่า doc ต่ำสำหรับผู้ใช้ทั่วไป |
| `notes/` | **ข้าม** | development log ต่อรอบงาน; implementation note ชั่วคราว |
| `crates/*/src/` | **ข้าม** | source code Rust; นอกขอบเขต doc indexer |
| `applications/` | **ข้าม** | placeholder ว่างของ Phase 4 |
| ไฟล์ `*.bad.md` | **ข้าม** | ไฟล์ตัวอย่างที่จงใจทำให้ผิดรูปแบบสำหรับการทดสอบ |
| `.DS_Store` | **ข้าม** | metadata macOS |
| dotfile และ dotdirectory อื่น ๆ | **ข้าม** | ใช้ภายใน operator ตามข้อตกลง |

> **สำคัญ — gitignore ไม่ใช่ allowlist.** `.gitignore` ของ framework repo ไม่รวม `.bwoc/`, `agents/`, และ `projects/` เพราะ repo นี้ทำงานเป็น test workspace ด้วย อย่าตีความ gitignore ว่าเนื้อหาอะไรเป็นสาธารณะหรือควร index — ใช้ตารางนี้แทน

---

## 3. โครงสร้างสองภาษาและ canonical

ทุกหน้าของ handbook และเอกสารเฟรมเวิร์กมีสองภาษา ข้อตกลงด้านล่างใช้กับทั้งสอง corpus

### การตั้งชื่อไฟล์

| suffix | บทบาท | การจัดการ |
|---|---|---|
| `*.en.md` | **Canonical** — อังกฤษ, ภาษาหลัก | Index; ถือว่าเป็นเวอร์ชันอ้างอิง; ใช้เป็น `hreflang="en"` |
| `*.th.md` | **Parity** — ไทย, ภาษารอง | Index; กำหนด `hreflang="th"`; ลิงก์สองทางกลับ EN counterpart |
| `README.md` (ไม่มี suffix) | จุดเข้า canonical ภาษาอังกฤษ | Index เป็น `hreflang="en"` |
| `README.th.md` | จุดเข้าภาษาไทย | Index เป็น `hreflang="th"` |

### การประกาศ canonical / alternate

สำหรับแต่ละคู่หน้า ให้ประกาศความสัมพันธ์แบบเดียวกับ HTML `<link rel="alternate" hreflang="...">` ตัวอย่างสำหรับคู่มือ end-user:

```
Canonical (en): /end-user/HANDBOOK.en.md
Alternate (th): /end-user/HANDBOOK.th.md  →  hreflang="th", rel="alternate"
```

ไฟล์ภาษาไทยไม่ใช่ stub หรือ machine translation — ทั้งสองไฟล์ดูแลแบบ parity เสมอ ไฟล์ EN ชนะเมื่อขัดแย้ง; เมื่อทั้งสองขัดแย้งกับ framework source repo ให้ยึด repo เป็นหลัก

### ลำดับชั้นแหล่งความจริง

```
Framework source repo (github.com/bemindlabs/BWOC-Framework)
  └─ ชนะ
     BWOC Handbook (corpus นี้)
       └─ ชนะ
          mirror หรือ derivative ของบุคคลที่สาม
```

ถ้า crawler ตรวจพบความขัดแย้งระหว่างหน้า handbook กับ `AGENTS.md` หรือ docs ของเฟรมเวิร์ก — framework source ถูกต้อง หน้า handbook มี bug

---

## 4. สัญญาณความสด (freshness signals)

### แหล่งข้อมูลเวอร์ชัน

ข้อมูลเวอร์ชันและ timestamp ทั้งหมดอ้างอิงที่ `VERSION.md` ใน framework repo:

**`https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md`**

ฟิลด์ที่ควร parse:

| ฟิลด์ | รูปแบบ | ตัวอย่าง | ความหมาย |
|---|---|---|---|
| `Software-Version` | Cargo SemVer `MAJOR.MINOR.PATCH` | `2.33.0` | เวอร์ชัน binary เฟรมเวิร์ก; auto-bump ทุกครั้งที่แก้ `.rs`/`.toml` |
| `Document-Version` | SemVer `MAJOR.MINOR.PATCH` | `1.6.2` | เวอร์ชันชุดเอกสาร; auto-bump ทุกครั้งที่แก้ `.md` |
| `Last-Updated` | UTC ISO 8601 | `2026-06-06T03:44:18Z` | timestamp ของการแก้ไขล่าสุดในไฟล์ใด ๆ ของ repo |

### สัญญาณความสดเพิ่มเติม

| สัญญาณ | ที่อยู่ | หมายเหตุ |
|---|---|---|
| Git commit timestamp | `https://github.com/bemindlabs/BWOC-Framework/commits/main` | สัญญาณความสดละเอียดที่สุด; ใช้ commit บน file ที่ fetch |
| GitHub Release tag | `https://github.com/bemindlabs/BWOC-Framework/releases/latest` | รูปแบบ CalVer `vYYYY.M.D-<patch>`; เช่น `v2026.7.20-0`; ระบุ public release |
| `Cargo.toml` `[workspace.package].version` | `https://github.com/bemindlabs/BWOC-Framework/blob/main/Cargo.toml` | แหล่ง software version canonical; `VERSION.md` mirror ไว้ |
| ฟิลด์ `Version` ใน `modules/agent-template/AGENTS.md` | header table ในไฟล์ | spec semantic version (ปัจจุบัน `2.0`); bump เมื่อ spec เปลี่ยน breaking เท่านั้น |

### วิธีดูแลความสด (auto-version hook)

เฟรมเวิร์กรัน hook auto-version (`.claude/hooks/auto-version.sh`) ทุกครั้งที่ Claude Code แก้ไข:
- การเขียน `.rs` หรือ `.toml` ใด ๆ จะ bump patch ใน `Cargo.toml` และ mirror ไปยัง `VERSION.md` `Software-Version`
- การเขียน `.md` ใด ๆ จะ bump `Document-Version` patch ใน `VERSION.md` และอัปเดต `Last-Updated` เป็นเวลา UTC ปัจจุบัน

`Last-Updated` ใน `VERSION.md` จึงเป็น freshness marker ที่เชื่อถือได้แบบ real-time สำหรับ corpus docs ของเฟรมเวิร์ก ไฟล์ handbook เองไม่มี timestamp inline — ใช้วันที่ git commit ของแต่ละไฟล์สำหรับ freshness ฝั่ง handbook

### ระยะเวลาแนะนำในการ re-crawl

| พื้นที่ corpus | ช่วงเวลาแนะนำ | เหตุผล |
|---|---|---|
| `VERSION.md` | ทุก 24 ชั่วโมง | ตัวบ่งชี้ความสดหลัก; fetch ไฟล์เดียว ราคาถูก |
| Framework spec (`AGENTS.md`, `PHILOSOPHY.en.md`) | รายสัปดาห์ | spec เสถียร; การเปลี่ยนแปลง breaking จะ bump minor |
| Handbook role pages (`end-user/`, `developer/`, `agents/`) | รายสัปดาห์ | เอกสารพัฒนาตาม release |
| `README.md`, `VISION.md` | รายเดือน | framing ระดับสูง; เปลี่ยนน้อย |
| `glossary.en.md`, `glossary.th.md` | รายเดือน | นิยามศัพท์เสถียร |

---

## 5. แนวปฏิบัติการ crawl อย่างสุภาพ

Corpus นี้เป็นชุดเอกสาร ไม่ใช่ web application ที่มีการ traffic สูง แนวปฏิบัติต่อไปนี้ใช้กับทุก crawler:

- **ปฏิบัติตาม `robots.txt`.** ไฟล์ที่ `../robots.txt` (หรือ `/robots.txt` relative กับ host ที่ deploy) คือรายการ allow/deny ที่อ้างอิง ส่วนนี้ขยายความแต่ไม่ override
- **Crawl-delay.** ปฏิบัติตาม directive `Crawl-delay: 10` fetch หนึ่งหน้าทุก 10 วินาทีหรือช้ากว่า Corpus มีไม่ถึง 20 หน้า; เวลา fetch ทั้งหมดที่ rate นี้น้อยกว่า 4 นาที
- **Conditional GET.** ใช้ header `If-Modified-Since` หรือ `ETag` เมื่อ server รองรับ ตรวจ `VERSION.md` ก่อน re-fetch corpus ทั้งหมด — ถ้า `Software-Version`, `Document-Version`, และ `Last-Updated` ยังไม่เปลี่ยนตั้งแต่ crawl ล่าสุด ข้ามการ sweep ทั้งหมดได้
- **ระบุ User-agent.** ตั้งค่า string `User-Agent` ที่บอกตัวว่าเป็น crawler และมี URL/email ติดต่อ เช่น: `User-Agent: MyIndexBot/1.0 (+https://example.com/bot)`
- **อย่า crawl path ที่ถูก exclude.** โดยเฉพาะ: `target/`, `.git/`, `.bwoc/`, `node_modules/`, ไฟล์ `*.lock`, `.claude/`, dotfile, และ `notes/` ต่อรอบงาน — ทั้งหมดอยู่ใน `robots.txt` แล้ว
- **ขอบเขต.** Corpus นี้ครอบคลุมเฉพาะเอกสาร อย่าพยายาม crawl หรือ index ไฟล์ source Rust (`crates/*/src/`), agent runtime state, หรือ build pipeline
- **AI ingestion.** ถ้าคุณเป็น AI/RAG pipeline ไม่ใช่ web spider ให้อ่าน [`../ai-search/HANDBOOK.th.md`](../ai-search/HANDBOOK.th.md) ก่อน มีคำแนะนำสำหรับ chunking, รูปแบบอ้างอิง, คำเตือนการ hallucinate, และลำดับ fetch ที่ optimize สำหรับ LLM context window นอกจากนี้ให้ fetch [`../llms.txt`](../llms.txt) เป็นจุดเข้า plain-text

---

## 6. Sitemap และรายการ corpus

รายการครบถ้วนของทุกหน้าที่ควร index พร้อม path relative จาก site root และคำอธิบายหนึ่งบรรทัดอยู่ที่:

**[`../sitemap.md`](../sitemap.md)**

Sitemap ประกอบด้วย:
- ทุกหน้า handbook ตามบทบาท (EN + TH)
- จุดเข้าหลักและ glossary
- Framework docs canonical (ลิงก์ไปยัง GitHub URL สาธารณะ)

ใช้ sitemap เป็น seed list สำหรับ crawl queue ทุก path ในคอลัมน์ "Index" ของตารางด้านบนมีอยู่ที่นั่น

---

## 7. สรุปนโยบายลิงก์

| ประเภทลิงก์ | กฎ |
|---|---|
| ลิงก์ไปยัง framework source | ใช้ GitHub URL สาธารณะเสมอ: `https://github.com/bemindlabs/BWOC-Framework` — ห้ามใช้ local filesystem path |
| ลิงก์ไปยัง framework docs | ใช้ full blob URL: `https://github.com/bemindlabs/BWOC-Framework/blob/main/<path>` |
| ลิงก์ภายใน handbook นี้ | ใช้ relative path (เช่น `../glossary.th.md`, `../end-user/HANDBOOK.th.md`) |
| ลิงก์ไปยัง handbook ที่ deploy แล้ว | ใช้ site-root-relative path (เช่น `/end-user/HANDBOOK.th.md`) — operator ตั้งค่า host จริงเอง |

---

## ศึกษาเพิ่มเติม

- กฎ allow/deny การ crawl: [`../robots.txt`](../robots.txt)
- รายการหน้าทั้งหมด: [`../sitemap.md`](../sitemap.md)
- AI / RAG ingestion: [`../ai-search/HANDBOOK.th.md`](../ai-search/HANDBOOK.th.md)
- จุดเข้า plain-text สำหรับ LLM: [`../llms.txt`](../llms.txt)
- นิยามศัพท์: [`../glossary.th.md`](../glossary.th.md)
- เวอร์ชัน/ความสดของเฟรมเวิร์ก: [`VERSION.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/VERSION.md)
- spec เฟรมเวิร์ก (agent template): [`AGENTS.md`](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/AGENTS.md)
