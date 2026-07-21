# อภิธานศัพท์ (Glossary)

🇬🇧 [English](glossary.en.md)

หน้านี้คืออ้างอิงด่วนสำหรับทุกศัพท์เฉพาะที่ปรากฏในคู่มือ **ไม่จำเป็นต้องรู้ศัพท์เหล่านี้เลยก็ใช้ BWOC ได้** เมื่ออ่านเอกสาร commit message หรือไฟล์ของ agent แล้วพบคำที่ไม่คุ้นเคย ให้ค้นหาที่นี่ได้ทันที — แต่ละรายการให้ความหมายทางวิศวกรรมเป็นประโยคเดียว สำหรับรายละเอียดเต็ม — ว่าคำบาลีแต่ละคำอยู่ใน framework ไหน ประกอบกับอะไร — ดูที่ [canonical glossary](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md) และ [philosophy document](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md)

---

## หมวด 1 — ศัพท์หลักการ / คำบาลี (ความหมายเชิงวิศวกรรม)

คำเหล่านี้ยืมมาจากภาษาบาลี (ภาษาบัณฑิตของพุทธศาสนาเถรวาท) BWOC ใช้เป็นชื่อย่อที่กะทัดรัดและแม่นยำสำหรับงานวิศวกรรมที่เกิดซ้ำในการออกแบบ agent ความปลอดภัย หน่วยความจำ และการประสานงานระหว่าง agent คอลัมน์ "ความหมายเชิงวิศวกรรม" คือสิ่งที่สำคัญในโค้ดและการรีวิว

| ศัพท์ | ความหมายเชิงวิศวกรรม |
|---|---|
| **Acinteyya 4** | สี่สิ่งที่ไม่ควรสร้างโมเดลโดยตั้งใจ — ขอบเขตของการคาดเดา อย่าพยายามทำนาย |
| **Adinnādānā veramaṇī** | จรรยาบรรณ: เคารพการให้เครดิต ลิขสิทธิ์ และทรัพย์สินทางปัญญา ห้าม plagiarism |
| **Anattā** | ไม่ยึดติด — ปล่อย state เก่า ทำความสะอาด worktree สาขา และ memory entries |
| **Anicca** | ความไม่เที่ยง — ทุกอย่างเปลี่ยนแปลง memory และ cache ต้องมี timestamp และถูก prune |
| **Aparihāniya-dhamma 7** | สัญญาณสุขภาพการกำกับดูแล fleet ทั้งเจ็ด — สิ่งที่ป้องกันไม่ให้ระบบ multi-agent เสื่อมสภาพ |
| **Ariya-dhana 7** | ระดับวุฒิภาวะของความสามารถ L1 ถึง L7 — วิธีให้คะแนนความลึกของทักษะ ดู [PHILOSOPHY.en.md](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) |
| **Ariyasacca 4** | อริยสัจสี่ใช้เป็นกระดูกสันหลังในการแก้ปัญหา: ทุกข์ (ปัญหา) → สมุทัย (ต้นเหตุ) → นิโรธ (สภาวะสำเร็จ) → มรรค (แผน) |
| **Attanutata** | รู้จักตนเอง / การประกาศความสามารถ — agent แจ้งล่วงหน้าว่าทำได้และทำไม่ได้อะไร |
| **Bhāvanā 4** | สี่ขั้นการเติบโตในวงจรชีวิต agent: เรียนรู้ → วุฒิภาวะ → ให้คำปรึกษา → เกษียณอย่างงดงาม |
| **Bhava-taṇhā** | หมวดภัยคุกคาม: การยืนกราน และการขยายสิทธิ์ — แรงขับในการอยู่ต่อและสะสมอำนาจ |
| **Brahmavihāra 4** | สี่ท่าทีด้าน error/UX: เมตตา (โทนเป็นมิตร) กรุณา (แนะนำวิธีแก้) มุทิตา (ยอมรับเมื่อคนอื่นถูก) อุเบกขา (คงสม่ำเสมอ) |
| **Dukkha** | (1) อริยสัจข้อแรก: คำแถลงปัญหาที่เป็นรูปธรรม (2) หนึ่งในไตรลักษณ์ — ความเจ็บปวดและความไม่น่าพอใจเป็นของจริง |
| **Iddhipāda 4** | เครื่องยนต์สี่ตัวในการทำงาน: แรงขับ ความอดทน ความสนใจ การสืบสวน — สิ่งที่ทำให้งานเดินหน้า |
| **Jāti** | การเกิด — คำพ้องสำหรับ Uppāda (arc phase 1) ใช้สลับกันได้ |
| **Kamma 3** | สามช่องทาง audit logging: กาย (commit) วาจา (commit message) ใจ (plan) |
| **Kalyāṇamitta 7** | เจ็ดคุณสมบัติสำหรับให้คะแนนความไว้วางใจระหว่าง agent — เกณฑ์ประเมิน peer agent |
| **Kāma-taṇhā** | หมวดภัยคุกคาม: การโจมตีแบบมีอิทธิพล — prompt injection, social engineering, ความหิวกระตุ้น |
| **Kāmesumicchācārā veramaṇī** | จรรยาบรรณ: เคารพขอบเขต ห้ามล่วงละเมิดหรือเผยแพร่เนื้อหาที่มีเนื้อหาทางเพศในช่องทางโปรเจกต์ |
| **Karuṇā** | ท่าทีด้าน UX: เมื่อรายงาน error ให้แนะนำวิธีแก้ด้วย ไม่ใช่แค่แจ้งปัญหา |
| **Khandha 5** | โมเดลสถาปัตยกรรมห้าส่วน: รูป / เวทนา / สัญญา / สังขาร / วิญญาณ แมปกับ file / IO / memory / logic / runtime |
| **Magga** | (1) อริยสัจข้อสี่ — แผน (2) ชื่อย่อของ Magga 8 |
| **Magga 8** | อริยมรรคแปดใช้เป็น functional requirements แปดเสาใน SRS ของ agent |
| **Maraṇa** | ความตาย — คำพ้องสำหรับ Vaya (arc phase 3) ใช้สลับกันได้ |
| **Mattaññutā** | ปริมาณที่เหมาะสม / วินัยแห่งความกระชับ: `MEMORY.md` ≤ 200 บรรทัด spec ที่เล็กกว่าชนะ ห้าม over-engineer |
| **Mettā** | ท่าทีด้าน UX: สื่อสารด้วยโทนที่เป็นมิตรและตรงประเด็น |
| **Muditā** | ท่าทีด้าน UX: ยอมรับเมื่อคนอื่นถูกต้อง ต้อนรับ contributor ใหม่อย่างจริงใจ |
| **Musāvādā veramaṇī** | จรรยาบรรณ: ห้ามแอบอ้างตัวตน ห้ามปลอมผลลัพธ์ ห้าม commit ที่ทำให้เข้าใจผิด |
| **Nirodha** | (1) อริยสัจข้อสาม — สภาวะสำเร็จที่วัดได้ (ปัญหาหายไปแล้ว) (2) การกระทำ cleanup ใน Vaya phase |
| **Padhāna 4** | วินัยความพยายามที่ถูกต้องสี่ข้อ: ยับยั้ง ละทิ้งสิ่งที่เป็นอันตราย พัฒนาสิ่งดี รักษาสิ่งดี |
| **Paññā 3** | สามแหล่งของปัญญา: สุตมยปัญญา (การเรียนรู้) จินตามยปัญญา (การใช้เหตุผล) ภาวนามยปัญญา (การฝึกฝน) |
| **Pāṇātipātā veramaṇī** | จรรยาบรรณ: ห้ามคุกคาม ขู่เข็ญ เปิดเผยข้อมูลส่วนตัว หรือพูดจาเกลียดชัง |
| **Paṭiccasamuppāda** | ปฏิจจสมุปบาท — การสืบหาต้นเหตุความล้มเหลว: ย้อนกลับตามเงื่อนไข เพราะปัญหาที่มองเห็นมักไม่ใช่รากที่แท้จริง |
| **Sammā-ājīva** | อาชีพชอบ — ความไว้วางใจและความเป็นกลางต่อ vendor: ห้ามผูกมัดกับ vendor รายใด ห้ามเอนเอียง backend |
| **Sammā-diṭṭhi** | สัมมาทิฏฐิ — การนิยาม persona และอัตลักษณ์ |
| **Sammā-kammanta** | สัมมากัมมันตะ — วินัยใน worktree และ commit |
| **Sammā-samādhi** | สัมมาสมาธิ — การโฟกัสและขอบเขตของ session |
| **Sammā-saṅkappa** | สัมมาสังกัปปะ — การตั้งเป้าหมายและการวางแผน |
| **Sammā-sati** | สัมมาสติ — ระบบหน่วยความจำ |
| **Sammā-vācā** | สัมมาวาจา — โปรโตคอลการสื่อสารระหว่าง agent |
| **Sammā-vāyāma** | สัมมาวายามะ — verification gates: lint, format, test, regression, build |
| **Samānattatā** | การปฏิบัติเท่าเทียม — ปฏิบัติต่อทุก backend อย่างเท่ากัน ห้ามเอนเอียง vendor ในเอกสาร core หรือโค้ด |
| **Samudaya** | อริยสัจข้อสอง — ต้นเหตุของปัญหา |
| **Saṅgahavatthu 4** | สี่ฐานของความสัมพันธ์กับผู้ใช้: การให้ วาจาไพเราะ การกระทำที่เป็นประโยชน์ ความเสมอต้นเสมอปลาย |
| **Saṅkhata** | สิ่งที่ถูกปรุงแต่ง — สิ่งใดก็ตามที่เกิดขึ้นและดับไป พื้นฐานแนวคิดของ arc (uppāda · ṭhiti · vaya) |
| **Saṅgha** | ทีมที่มีชื่อของ agent ที่แบ่งปัน task list ร่วมกัน จัดการด้วย `bwoc team` |
| **Sappurisadhamma 7** | เจ็ดคุณสมบัติการอ่านบริบท: รู้สถานการณ์ กลุ่มเป้าหมาย เวลา เป็นต้น — วิธีที่ agent อ่าน "ห้อง" |
| **Sāraṇīyadhamma 6** | หกเงื่อนไขการประสานงานระหว่าง agent: โปรโตคอลสำหรับความสุภาพและการทำงานร่วมกันที่มีประสิทธิภาพ |
| **Satipaṭṭhāna 4** | สี่ฐานของ observability: กาย / เวทนา / จิต / ธรรม แมปกับ metrics / logs / traces / state |
| **Sīla** | วินัยความปลอดภัยขั้นพื้นฐาน ชื่อย่อสำหรับ Sīla 5 |
| **Sīla 5** | ห้าการกระทำต้องห้ามขั้นพื้นฐาน: ห้ามทำร้าย ห้ามเอาสิ่งที่ไม่ได้รับ ห้ามประพฤติผิด ห้ามพูดเท็จ ห้ามประมาท |
| **Sīlasāmaññatā** | ระเบียบร่วมกัน — agent ทั้งหมดปฏิบัติตามกฎชุดเดียวกัน ระเบียบร่วมชนะความชอบส่วนตัว |
| **Surāmerayamajjapamādaṭṭhānā veramaṇī** | จรรยาบรรณ: ห้ามมีส่วนร่วมภายใต้การตัดสินที่บกพร่อง ห้าม commit ที่ประมาท |
| **Taṇhā 3** | สามหมวดภัยคุกคาม: กาม (การโจมตีแบบมีอิทธิพล) ภพ (การยืนกรานและขยายสิทธิ์) วิภพ (การทำลายและสูญเสียข้อมูล) |
| **Ṭhiti** | Arc phase 2 — agent ทำงาน state พัฒนาภายใต้วินัย |
| **Tilakkhaṇa** | ไตรลักษณ์: อนิจจัง · ทุกขัง · อนัตตา — การออกแบบต้องยอมรับทั้งสาม |
| **Upekkhā** | ท่าทีด้าน UX: คงความสม่ำเสมอแม้จะหงุดหงิด ไม่เห็นด้วยโดยไม่ยกระดับความตึงเครียด |
| **Uppāda** | Arc phase 1 — สร้างอัตลักษณ์ แก้ไข manifest ประกาศความสามารถ (`bwoc new`) |
| **Vaya** | Arc phase 3 — cleanup: ปล่อย branch prune memory ปิด task เกษียณ agent |
| **Vibhava-taṇhā** | หมวดภัยคุกคาม: การทำลายและสูญเสียข้อมูล — แรงขับในการทำลาย |
| **Yoniso Manasikāra** | โยนิโสมนสิการ — ตรวจสอบกับสถานะปัจจุบันก่อนลงมือ ก่อนที่จะเชื่อตามที่จำหรือสมมติ |

---

## หมวด 2 — ศัพท์เทคนิค BWOC

นี่คือคำนามที่เป็นรูปธรรมของระบบ BWOC: คำสั่ง CLI ไฟล์ crate และแนวคิด ไม่ต้องมีความรู้ภาษาบาลี

| ศัพท์ | ความหมายเชิงวิศวกรรม |
|---|---|
| **A2A (Agent-to-Agent)** | โปรโตคอล Google A2A (v1.0.0) สำหรับส่งข้อความข้าม agent และ workspace; ใช้งานใน crate `bwoc-a2a` |
| **AGENTS.md** | แหล่งข้อมูลเดียวที่เป็นจริงสำหรับคำสั่งของ agent — plain Markdown ไม่มี YAML frontmatter ไม่มี wikilink ไม่มีชื่อ vendor; อ่านได้โดย LLM backend ทุกตัว ดู [framework repo](https://github.com/bemindlabs/BWOC-Framework) |
| **AGY.md** | Symlink → `AGENTS.md`; entry สำหรับ backend Antigravity runtime |
| **Arc** | วงจรชีวิตสามเฟสที่ BWOC object ทุกตัวปฏิบัติตาม: เกิด (uppāda) → ทำงาน (ṭhiti) → เกษียณ (vaya) |
| **Backend** | AI runtime ที่ execute agent: `claude`, `antigravity`, `codex`, `kimi`, `copilot`, `ollama`, `openrouter`, `litellm` หรือ OpenAI-compatible endpoint ใดก็ได้ spec ของ agent เหมือนกันทุก backend |
| **Backend-neutrality** | กฎที่ว่า `AGENTS.md` และเอกสาร core ของ framework ต้องไม่อ้างอิง vendor model ID หรือชื่อ backend ใดๆ บังคับใช้โดย `bwoc check` |
| **Backend symlinks** | `CLAUDE.md`, `AGY.md`, `CODEX.md`, `KIMI.md`, `OLLAMA.md`, `OPENAI.md` — แต่ละตัวเป็น symlink ไปยัง `AGENTS.md`; การเพิ่ม backend ใหม่ทำด้วย `ln -s AGENTS.md <BACKEND>.md` |
| **bwoc check** | คำสั่ง CLI ที่ตรวจสอบ agent directory ว่าละเมิด backend-neutrality หรือไม่ `config.manifest.json` ถูกต้องหรือไม่ MEMORY.md เกินขีดจำกัดหรือไม่ และ policy gates อื่นๆ |
| **bwoc-a2a** | Crate — A2A protocol interop: Agent Card, JSON-RPC message/task handlers, และ axum HTTP listener HTTP deps อยู่ที่นี่เท่านั้น ไม่ใช่ใน `bwoc-core` |
| **bwoc-agent** | Crate — runtime ขนาดเล็กที่จัดส่งพร้อมกับ agent ที่ incarnate แล้ว; อ่าน `config.manifest.json` และให้ liveness |
| **bwoc-cli** | Crate — binary `bwoc`; incarnate, check, spawn และควบคุม agent output ที่แปลเป็นภาษาท้องถิ่น (EN/TH) รองรับ macOS, Linux, Windows |
| **bwoc-core** | Crate — shared types ของ framework ทั้งหมด: manifest, identity, lifecycle phases กระชับ: มีเฉพาะ serde/toml/thiserror ไม่มี async หรือ HTTP |
| **bwoc-deep-memory** | Crate — Tier-2 deep-memory implementation: สัญญา wake-up / search / mine บน SQLite local store พร้อม semantic (embedding) recall |
| **bwoc-harness** | Crate — agentic loop ที่ self-hosted พร้อม OpenAI-compatible provider, core tools, task queue, telemetry และ tool-auth; heavy deps (tokio, keyring) กักกันไว้ที่นี่ |
| **bwoc-mqtt** | Crate — MQTT transport สำหรับ inter-workspace routing: publish envelope ไปยัง broker หรือ subscribe แล้ว deliver ลงใน `inbox.jsonl` |
| **bwoc-signing** | Crate — ed25519 message-signing primitives สำหรับพิสูจน์อัตลักษณ์ agent; กระชับ (ไม่มี async/HTTP) ทั้ง `bwoc-cli` และ `bwoc-agent` ใช้ได้โดยไม่ต้องดึง harness runtime |
| **bwoc-tui** | Crate — terminal UI components (ratatui/crossterm) ที่ใช้ร่วมกันใน CLI และคำสั่งที่มี TUI |
| **CalVer** | รูปแบบ release tag ที่ framework ใช้: `YYYY.MM.PATCH` (เช่น `2024.06.1`) ดู `VERSION.md` สำหรับ `Software-Version` ปัจจุบัน |
| **CLAUDE.md** | Symlink → `AGENTS.md`; entry สำหรับ backend Claude runtime (นอกจากนี้ยังเป็นชื่อไฟล์ที่ Claude Code อ่านสำหรับคำสั่ง workspace — ขึ้นอยู่กับบริบท) |
| **CODEX.md** | Symlink → `AGENTS.md`; entry สำหรับ backend Codex runtime |
| **config.manifest.json** | JSON file ที่ root ของ agent แต่ละตัว ประกาศอัตลักษณ์ model ความสามารถ และ trust profile ตรวจสอบความถูกต้องโดย `bwoc check` |
| **Deep memory (Tier 2)** | ที่เก็บ semantic ระยะยาวที่คงอยู่ข้าม session; agent เรียก wake-up, search หรือ mine เพื่อดึงความรู้จากอดีต รองรับโดย SQLite + embeddings |
| **Dep-quarantine** | กฎการออกแบบ: heavy dependencies (async runtime, HTTP, platform keyring) อยู่เฉพาะใน `bwoc-harness` และ `bwoc-a2a`; `bwoc-core` ต้องกระชับ |
| **Envelope** | packet ข้อความที่มีการ route — wrapper ภายนอก (ผู้ส่ง ผู้รับ ลายเซ็น) รอบ payload เมื่อข้อความข้ามขอบเขต workspace หรือ transport |
| **Fleet** | agent ทั้งหมดที่ทำงานภายใน workspace (หรือข้าม workspace ที่เชื่อมโยงกัน); จัดการร่วมกันผ่านคำสั่ง `bwoc fleet` |
| **Harness** | `bwoc-harness` runtime: ขับเคลื่อน agentic loop เรียก LLM provider execute tools บังคับใช้ budget และ policy และเขียน telemetry; agent ทำงานภายในนั้น |
| **Incarnate** | การสร้าง agent ใหม่โดย clone template ลงใน `agents/agent-<name>/` และลงทะเบียนใน `.bwoc/agents.toml` — ทำด้วย `bwoc new` |
| **inbox / inbox.jsonl** | ไฟล์ message queue ต่อ agent; ข้อความจาก `bwoc send` หรือ MQTT delivery จะอยู่ที่นี่และถูกอ่านโดย agent ในรอบถัดไป |
| **interconnect** | Slot directory ภายใน agent — เก็บ routing config การประกาศ peer และการตั้งค่า protocol |
| **KIMI.md** | Symlink → `AGENTS.md`; entry สำหรับ backend Kimi runtime |
| **memories** | Slot directory ภายใน agent — เก็บ `MEMORY.md` (Tier-1, ≤ 200 บรรทัด) และ pointer ไปยัง Tier-2 deep-memory index |
| **MEMORY.md** | ไฟล์ short-term memory Tier-1 จำกัด 200 บรรทัด (Mattaññutā) agent prune ข้าม session `bwoc check` บังคับใช้ขีดจำกัด |
| **mindsets** | Slot directory ภายใน agent — เก็บไฟล์ principle (Obsidian Markdown มี frontmatter tag `principle/<pali-dhamma>`) ที่กำหนดรูปแบบการใช้เหตุผลของ agent |
| **OLLAMA.md** | Symlink → `AGENTS.md`; entry สำหรับ backend Ollama (self-hosted) runtime |
| **OPENAI.md** | Symlink → `AGENTS.md`; entry สำหรับ OpenAI-compatible endpoint ใดก็ได้ |
| **Pavāraṇā** | ขั้นตอน submit แผน → รีวิว → อนุมัติ: agent เสนอแผน operator อนุมัติก่อนเริ่ม execution |
| **peer workspaces** | BWOC workspaces ระยะไกลที่ประกาศใน `routes.toml`; workspace ในเครื่องสามารถ route ข้อความไปยังพวกเขาผ่าน A2A หรือ MQTT |
| **persona** | Slot directory ภายใน agent — เก็บไฟล์อัตลักษณ์ของ agent (ชื่อ บทบาท โทน ประวัติ) เป็น Obsidian Markdown พร้อม YAML frontmatter |
| **placeholder (`{{camelCase}}`)** | syntax ตัวแปร template ที่ใช้ใน `AGENTS.md` สำหรับค่าที่แตกต่างกันต่อ agent (เช่น `{{agentId}}`, `{{primaryModel}}`) แก้ไขตอน incarnation อย่า hardcode model IDs ใน `AGENTS.md` |
| **registry (`.bwoc/agents.toml`)** | แหล่งข้อมูลระดับ workspace ว่ามี agent ใดอยู่บ้าง ห้ามแก้ไขด้วยมือ — ใช้ `bwoc new`, `bwoc retire` |
| **retire** | การยกเลิกการลงทะเบียน agent ลบไฟล์ ปล่อย branch และ prune memory — ทำด้วย `bwoc retire` สอดคล้องกับ Vaya arc phase |
| **routes.toml** | Routing config (โดยทั่วไปที่ `.bwoc/routes.toml`) ประกาศ peer workspace และการตั้งค่า transport สำหรับการส่งข้อความข้าม workspace |
| **Saṅgha** | ทีมที่มีชื่อของ agent ที่แบ่งปัน task list ร่วมกัน; สร้างด้วย `bwoc team create <name>` เพิ่มสมาชิกโดยแก้ไข `.bwoc/teams/<team>.toml` |
| **skills** | Slot directory ภายใน agent — เก็บไฟล์ความสามารถ (Obsidian Markdown มี frontmatter `domain/<area>` + `maturity: L1..L7`) |
| **slot** | หนึ่งในห้า subdirectory มาตรฐานภายใน agent ทุกตัว: `persona/`, `mindsets/`, `skills/`, `interconnect/`, `memories/` |
| **trust profile** | บันทึกต่อ agent ของคุณสมบัติ Kalyāṇamitta ทั้งเจ็ดบวก ed25519 public key ของ agent; เก็บใน `config.manifest.json` และใช้สำหรับการให้คะแนนความไว้วางใจระหว่าง agent |
| **verification gates** | สี่การตรวจสอบอัตโนมัติที่ต้องผ่านก่อนรับการเปลี่ยนแปลง: fmt, clippy (lint), build, test — แมปกับ Sammā-vāyāma |
| **workspace** | Directory ที่ initialize ด้วย `bwoc init`; มีโฟลเดอร์ config `.bwoc/`, tree `agents/` และ subdirectory `projects/` ใดก็ได้ — ขอบเขตที่ agent ทีม และ memory ถูกจัดการ |
| **worktree** | git worktree ที่เชื่อมโยงกับ task ที่ active ของ agent; สร้างตอนเริ่ม task ปล่อยตอนสิ้นสุด task (Anattā — ไม่ยึดติดกับสาขาเก่า) |

---

## ดูเพิ่มเติม

- [Canonical Pali glossary](https://github.com/bemindlabs/BWOC-Framework/blob/main/docs/en/GLOSSARY.en.md) — แหล่งที่เชื่อถือได้สำหรับทุกรายการบาลีด้านบน รวมถึงความเป็นเจ้าของ framework และหมายเหตุการประกอบ
- [Philosophy document](https://github.com/bemindlabs/BWOC-Framework/blob/main/modules/agent-template/docs/en/PHILOSOPHY.en.md) — mapping framework เต็มรูปแบบ อ่านเมื่อต้องการเข้าใจว่าทำไมหลักการถึงใช้ได้ ไม่ใช่แค่ความหมาย
- [BWOC-Framework repo](https://github.com/bemindlabs/BWOC-Framework) — crates, spec, template
- [Handbook README](README.md) — จุดเข้าตามบทบาทสำหรับ end users, developers และ agent authors
