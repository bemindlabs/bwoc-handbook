🇬🇧 [English](HANDBOOK.en.md)

# ระบบนิเวศ BWOC — ภาพรวมโปรเจกต์

ครอบครัว BWOC ประกอบด้วย framework หลักและแอปพลิเคชันเสริมรวมถึงเป้าหมายฮาร์ดแวร์ต่าง ๆ ที่โคจรรอบ `bwoc` CLI ในฐานะแหล่งความจริงเดียว framework เป็นเจ้าของ identity, lifecycle, memory และโปรโตคอลระหว่าง agent; โปรเจกต์อื่น ๆ ทุกตัวในครอบครัวนี้รับข้อมูลเหล่านั้นผ่าน `bwoc <cmd> --json` หรือ event stream `bwoc_core::chat_proto` โดยไม่มีโปรเจกต์ใดอ่านไฟล์ workspace โดยตรง หน้านี้แสดงแผนที่สมาชิกทุกตัวในครอบครัว: แต่ละตัวทำอะไร สร้างด้วยอะไร เชื่อมต่อกับ core อย่างไร และหาได้จากที่ไหน

---

## ภาพรวม

| โปรเจกต์ | บทบาท | Stack | ลิงก์ |
|---|---|---|---|
| **BWOC-Framework** | Core framework + `bwoc` CLI | Rust (9 crate), macOS / Linux / Windows | [bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework) |
| **bwoc-handbook** | เอกสารสองภาษาแบ่งตามบทบาท | Markdown (EN + TH) | ชุดเอกสารนี้ |
| **bwoc-chat** | Native desktop chat สำหรับ agent | Rust + egui | [bemindlabs/bwoc-chat](https://github.com/bemindlabs/bwoc-chat) |
| **bwoc-devices-app** (BWOC Monitor) | macOS dashboard แสดงสถานะ fleet แบบ read-only | Rust + Tauri 2 + tokio + static HTML/JS | ภายใน / ยังไม่เผยแพร่สาธารณะ |
| **bwoc-llm-pm** (LLM Provider Monitor) | macOS menu-bar: สถานะ auth + quota ของผู้ให้บริการ LLM | Swift, SwiftUI, SwiftPM, macOS 13+ | [bemindlabs/LLMProviderMonitor](https://github.com/bemindlabs/LLMProviderMonitor) |
| **bwoc-mcc** | macOS menu-bar control center สำหรับ fleet | Swift 5.9, SwiftUI, macOS 13+ | [bemindlabs/bwoc-mcc](https://github.com/bemindlabs/bwoc-mcc) |
| **bwoc-devices** | Monorepo อุปกรณ์ — Rust core + firmware หลายบอร์ด | Rust (4 crate), Arduino/PlatformIO C++ | [bemindlabs/bwoc-devices](https://github.com/bemindlabs/bwoc-devices) |

---

## Core

### BWOC-Framework

**คืออะไร.** รากฐานที่ทุกอย่างพึ่งพา BWOC-Framework คือ specification ที่ไม่ผูกกับ backend ใด และ implementation ใน Rust สำหรับการสร้าง รัน และประสานงาน AI coding agent มาในรูป Rust workspace เก้า crate ได้แก่ `bwoc-cli`, `bwoc-harness`, `bwoc-core`, `bwoc-agent`, `bwoc-mqtt`, `bwoc-deep-memory` และอื่น ๆ พร้อม binary `bwoc` CLI และ scaffold `modules/agent-template` ที่ clone ได้

**Stack.** Rust 1.85+ (edition 2024), รองรับหลายแพลตฟอร์ม (macOS / Linux / Windows), MIT. เวอร์ชันปัจจุบัน: v2.24.0.

**สิ่งที่มอบให้ระบบนิเวศ.** โปรเจกต์อื่น ๆ ทุกตัวในครอบครัวเป็นผู้บริโภค ไม่ใช่คู่แข่ง:

- `bwoc list --json`, `bwoc sessions --json`, `bwoc fleet health` — JSON API ที่แอปบนเดสก์ท็อปและ firmware ของอุปกรณ์ใช้ polling
- `bwoc-harness --chat` — subprocess ที่ `bwoc-chat` spawn; เป็นเจ้าของการเรียก model, tool, guardrail และ permission
- `bwoc_core::chat_proto` — typed event stream (`token`, `tool_call`, `tool_result`, `permission_request`, `turn_end`) ที่ `bwoc-chat` render
- Agent registry, manifest schema, team protocol, inbox และ scrum state ที่ `bwoc-mcc` อ่าน

**ลิงก์.** [github.com/bemindlabs/BWOC-Framework](https://github.com/bemindlabs/BWOC-Framework)

---

### bwoc-handbook

**คืออะไร.** ชุดเอกสารที่คุณกำลังอ่านอยู่นี้ คู่มือสองภาษา (EN/TH) แบ่งตามบทบาท สำหรับผู้ใช้งาน นักพัฒนา ผู้สร้าง agent, AI search agent และ crawler ทำหน้าที่สรุปและอ้างอิงข้ามไปยัง framework โดยไม่ซ้ำซ้อนกับแหล่งความจริง — หากขัดแย้งกัน repo ของ framework ชนะเสมอ

**Stack.** Markdown ธรรมดา คู่ไฟล์ `*.en.md` / `*.th.md` ต่อหน้า

**เชื่อมต่ออย่างไร.** คู่มือนี้อ้างอิง framework repo สำหรับข้อเท็จจริงและลิงก์ไปยัง GitHub สาธารณะเสมอ ไม่ฝัง path ของ workspace ใน local

**ลิงก์.** ชุดเอกสารนี้ — ดู [`../README.md`](../README.md) สำหรับ index เต็ม

---

## Desktop และ Menu-Bar Apps

### bwoc-chat

**คืออะไร.** หน้าต่าง chat บนเดสก์ท็อปสำหรับ BWOC agent แบบ native — agent เดียวหรือทั้งทีมในหน้าต่างเดียว มี reply แบบ streaming, panel แสดงกิจกรรม tool แบบ real-time ต่อ agent และ prompt Allow / Deny แบบ inline เมื่อ tool อยู่ในโหมด `ask`

**Stack.** Rust (edition 2024), [egui](https://github.com/emilk/egui) / eframe — GUI immediate-mode ใน Rust ล้วน ไม่มี webview พึ่งพา `bwoc-core` (by path) สำหรับโปรโตคอล chat และการ resolve agent; ไม่พึ่งพา `bwoc-cli` หรือ `bwoc-harness` ตอน build รองรับ backend `ollama` และ `openai-compatible`

**เชื่อมต่อกับ framework อย่างไร.** `bwoc-chat` spawn `bwoc-harness --chat` เป็น child process สำหรับแต่ละ agent อ่าน `bwoc_core::chat_proto` event จาก stdout ของ child (token, tool call, permission request) และเขียน `ChatInput` line กลับไปยัง stdin harness เป็นเจ้าของ session; หน้าต่างทำหน้าที่เพียง render และส่งต่อ ชื่อ agent ถูก resolve จาก workspace registry — `.bwoc/agents.toml` เดียวกันกับที่ CLI อ่าน

**โหมดทีม.** ส่งชื่อ agent หลายตัวใน command line; แต่ละตัวกลายเป็น harness subprocess ของตัวเอง ทั้งหมด multiplex รวมอยู่ใน transcript เดียว ข้อความ broadcast ไปทุก agent; `@name` ระบุ agent เดียว

**ทำไมต้องแยก repo.** เพื่อแยก dependency tree ของ GUI ที่หนัก (winit, glow) ออกจาก framework workspace ที่เบา `bwoc-chat` ดึงเฉพาะ `bwoc-core` เท่านั้น

**สถานะ.** ใช้งานได้ Backend แบบ vendor-CLI (claude / codex / kimi / agy) ไม่ได้ render ที่นี่ — ใช้ `bwoc spawn` สำหรับอันเหล่านั้น

**ลิงก์.** [github.com/bemindlabs/bwoc-chat](https://github.com/bemindlabs/bwoc-chat)

---

### bwoc-devices-app (BWOC Monitor)

**คืออะไร.** Dashboard บนเดสก์ท็อป macOS แบบ read-only สำหรับ BWOC workspace shell ออกไปยัง `bwoc` CLI ตาม timer ที่กำหนดได้ และ render สามแผง: **Fleet Health** (สัญญาณ governance เจ็ดตัว), **Agents** (registry + status) และ **Sessions** (daemon / tmux pane ที่กำลังรัน)

**Stack.** Rust + Tauri 2 + tokio + static HTML/JS ไม่มี npm, Node หรือ Vite ฝั่ง Rust เปิด Tauri IPC command (`list_agents`, `list_sessions`, `fleet_health`, `fleet_snapshot`) ให้ webview polling คำสั่ง `fleet_snapshot` รวมทั้งสามเข้าเป็น canonical BWOC fleet JSON payload — รูปแบบเดียวกับที่ firmware อุปกรณ์รับผ่าน WebSocket

**เชื่อมต่อกับ framework อย่างไร.** CLI คือแหล่งความจริงเดียว; แอปไม่อ่านไฟล์ workspace โดยตรง spawn `bwoc --workspace <path> <cmd> --json` และ parse output output ของ `fleet health` (ไม่มี flag `--json` ใน v1) ถูก parse จาก block structure โดยฝั่ง Rust

**สถานะ.** ภายใน / ยังไม่เผยแพร่สาธารณะ ไม่มี URL สาธารณะ

---

### bwoc-llm-pm (LLM Provider Monitor)

**คืออะไร.** แอป macOS menu-bar แบบ lightweight ที่แสดงสถานะ auth และการใช้ token / credit ของ LLM provider CLI ที่ติดตั้งบนเครื่องมองเห็นได้ทันที พร้อม action สำหรับ login / switch แต่ละตัว

**ผู้ให้บริการที่รองรับ.** Antigravity, Claude, Codex, Kimi, Ollama

**Stack.** Swift, SwiftUI, SwiftPM, macOS 13+ สาม target: `LLMProviderMonitorCore` (logic ล้วน, unit-tested), `LLMProviderMonitor` (SwiftUI menu-bar app), `CoreChecks` (test runner แบบ plain executable — ไม่ต้อง XCTest) Activation policy `.accessory` — ไม่มี icon บน Dock

**เชื่อมต่อกับ framework อย่างไร.** probe provider CLI โดยอิสระจาก BWOC; ไม่เรียก `bwoc` โดยตรง ความสัมพันธ์กับ ecosystem BWOC เป็นแบบเสริมกัน: `bwoc-mcc` ดูแล fleet operations (agent, session, inbox) ในขณะที่ `bwoc-llm-pm` ดูแล credential และ quota ของผู้ให้บริการ ทั้งสองออกแบบมาให้อยู่ side-by-side บน menu bar

**การติดตาม credit.** Codex: อ่าน rollout file จาก `~/.codex/sessions/`. Claude: อ่าน `~/.claude/usage-cache.json` ที่ถูกเขียนโดย `statusLine` hook ระหว่าง Claude Code session ที่ active. Ollama: แสดง model ที่กำลัง load จาก `ollama ps`; ไม่มี quota. Antigravity: quota ไม่สามารถเข้าถึงได้แบบ headless แสดงเป็น em dash

**สถานะ.** ใช้งานได้ ยังไม่ได้ package เป็น `.app` bundle แบบ signed — รันเป็น SwiftPM executable สำหรับการใช้งาน local

**ลิงก์.** [github.com/bemindlabs/LLMProviderMonitor](https://github.com/bemindlabs/LLMProviderMonitor)

---

### bwoc-mcc

**คืออะไร.** SwiftUI macOS menu-bar control center สำหรับ BWOC agent fleet แสดงสถานะ real-time — agent, session, inbox — และ quick action สำหรับ spawn, chat, stop และ supervise agent โดยไม่ต้องออกจาก menu bar

**Stack.** Swift 5.9, SwiftUI `MenuBarExtra`, SwiftPM, macOS 13+ (Ventura+) สาม target: `BwocMccCore` (library), `BwocMcc` (SwiftUI app), `CoreChecks` (test runner) ไม่มี Electron ไม่มี background daemon test รันผ่าน `swift run CoreChecks` ไม่ใช่ XCTest

**เชื่อมต่อกับ framework อย่างไร.** shell-out ทั้งหมดผ่าน `bwoc <cmd> --json` — แอปถูกแยกออกจาก Rust internals ของ BWOC และพึ่งพาเพียง CLI surface ที่เสถียร อ่าน `bwoc list --json` และ `bwoc sessions --json` เดียวกันกับ `bwoc-devices-app`; polling ทุกห้าวินาที; ปุ่ม refresh บังคับ poll Quick action สำหรับ chat เปิดหน้าต่าง `bwoc-chat` egui สำหรับ agent แบบ `ollama` / `openai-compatible` และ fallback ไปที่ `bwoc chat` ใน Terminal สำหรับ vendor-CLI agent

**ขอบเขต.** Fleet operations เท่านั้น — agent, session, inbox และ (วางแผน) scrum state LLM provider auth และ quota อยู่นอกขอบเขต; นั่นคืองานของ `bwoc-llm-pm`

**สถานะ.** Alpha โครงสร้างพื้นฐาน build ได้ launch ได้ และ render fleet จาก `bwoc list --json` Quick action, sessions view, inbox preview และ scrum integration อยู่ระหว่างพัฒนาภายใต้ `BWOC-EPIC-5`

**ลิงก์.** [github.com/bemindlabs/bwoc-mcc](https://github.com/bemindlabs/bwoc-mcc)

---

## อุปกรณ์และ Firmware

### bwoc-devices

**คืออะไร.** Monorepo standalone ที่นำ dashboard fleet ของ BWOC ไปแสดงบนฮาร์ดแวร์จริง Rust core แบบพกพาหนึ่งตัว abstract ฮาร์ดแวร์ไว้หลัง HAL trait; firmware หลายบอร์ดแต่ละตัว implement display และ transport อุปกรณ์เข้าร่วม fleet และ render dashboard แบบ real-time

**Crate.**

| Crate | บทบาท |
|---|---|
| `bwoc-device-proto` | JSON wire contract ระหว่าง host และอุปกรณ์ — แหล่งความจริงสำหรับทุก target |
| `bwoc-device-hal` | HAL trait: Net, Store, Input, Display — ไม่ต้อง `std` |
| `bwoc-device-core` | Runtime แบบพกพา: source arbitration (`SourceArbiter`), fleet state, render |
| `bwoc-device-linux` | Linux / Raspberry Pi HAL impl + binary `bwoc-device`; transport แบบ feature-gated |

**บอร์ดเป้าหมาย.**

| บอร์ด | Stack | สถานะ |
|---|---|---|
| WT32-SC01 Plus (ESP32-S3, 3.5" ST7796) | Arduino / PlatformIO C++ | Production (รับมาจาก bwoc-penlee-sc01-plus) |
| Raspberry Pi 5 / Linux | Rust core, `--features pi` (MQTT + WS + fbdev + evdev) | ใช้งานได้; deploy ผ่าน `deploy/rpi5/` |
| ESP32-S3 ball (1.28" round LCD, XiaoZhi voice) | ESP-IDF C++, LovyanGFX | Scaffolded ยังไม่ได้ build |

**Stack.** Rust workspace (`no_std + alloc` สำหรับ `proto` / `hal` / `core`; `std` สำหรับ `bwoc-device-linux`), Arduino / PlatformIO C++ Transport: MQTT (`rumqttc`), WebSocket (`tungstenite`), stdin (USB), framebuffer, evdev touch Feature flag: `mqtt`, `ws`, `sim`, `fbdev`, `evdev`, `full`, `pi`

**ขอบเขตโปรโตคอล.** ESP32 C++ firmware ไม่ link Rust crate implement การ render เอง (LovyanGFX) และ conform กับ `bwoc-device-proto` ผ่าน header `firmware/<board>/gen/bwoc_proto.h` ที่ generate จาก tool crate `gen-cpp-proto` เท่านั้น โปรโตคอลคือ contract ร่วม; HAL code ไม่ใช่

**เชื่อมต่อกับ framework อย่างไร.** Host push fleet snapshot — JSON canonical เดียวกับที่คำสั่ง `fleet_snapshot` ของ `bwoc-devices-app` produce — ผ่าน MQTT หรือ WebSocket ไปยังอุปกรณ์ runtime `bwoc-device-core` ใช้ `SourceArbiter` (USB มีลำดับความสำคัญสูงกว่า WS; WS สูงกว่า MQTT) และ re-render dashboard แต่ละ tick อุปกรณ์ไม่เรียก `bwoc` CLI โดยตรง; รับ payload ที่ compose ไว้แล้วจาก pusher

**สถานะ.** ใช้งานได้ Raspberry Pi และ ESP32-SC01-Plus target พร้อมใช้งาน

**ลิงก์.** [github.com/bemindlabs/bwoc-devices](https://github.com/bemindlabs/bwoc-devices)

---

### bwoc-penlee-sc01-plus (ประวัติศาสตร์)

**คืออะไร.** โปรเจกต์เดิมที่เป็นต้นกำเนิดแนวคิด fleet display บน SC01 Plus: firmware ESP32-S3 (BLE สำหรับ config, WebSocket สำหรับข้อมูล) และ Tauri host app ที่ push BWOC fleet JSON ไปยังอุปกรณ์ firmware และแนวคิดโปรโตคอลถูก generalize และรับเข้าสู่ **bwoc-devices** (`firmware/esp32-sc01plus/`) directory เดิมยังคงอยู่เป็น regression reference ระหว่างที่ firmware ที่ migrate กำลังสร้างความเสถียร ไม่มี URL สาธารณะแยกต่างหาก — อยู่ภายใน history ของ bwoc-devices

---

## ความสัมพันธ์ระหว่างทุกโปรเจกต์

`bwoc` CLI และ wire schema `bwoc-device-proto` คือ contract ร่วมที่ทำให้โปรเจกต์เหล่านี้ composable โดยไม่ผูกกันแน่น

```
bwoc CLI  ──────────────────────────────────────────────────────────────┐
  │  bwoc list --json            ใช้โดย: bwoc-mcc, bwoc-devices-app    │
  │  bwoc sessions --json        ใช้โดย: bwoc-mcc, bwoc-devices-app    │
  │  bwoc fleet health           ใช้โดย: bwoc-mcc, bwoc-devices-app    │
  │                                                                      │
  │  bwoc-harness --chat  ──────► bwoc-chat (egui renderer)             │
  │                          ▲                                           │
  │                          └── เปิดโดย bwoc-mcc (quick action)       │
  │                                                                      │
fleet snapshot (JSON)  ──────► bwoc-devices-app ──► (push) bwoc-devices │
                                                                         │
bwoc-llm-pm  ── อิสระจาก bwoc CLI; เสริม bwoc-mcc ─────────────────────┘
```

แอปทุกตัวบนเดสก์ท็อปหรือ menu bar เรียก `bwoc` ด้วยชื่อและ parse output แบบ `--json` ไม่มีแอปใดอ่านไฟล์ `.bwoc/` โดยตรง ไม่มีแอปใด depend on `bwoc-cli` หรือ `bwoc-harness` ตอน build (ยกเว้น `bwoc-chat` ซึ่ง depend on `bwoc-core` เฉพาะสำหรับ protocol type) Firmware อุปกรณ์รับเพียง fleet JSON payload ไม่ใช่ CLI นี่หมายความว่า CLI surface แบบ `--json` และ `bwoc-device-proto` คือ stability boundary ของครอบครัวทั้งหมด: เปลี่ยนด้วยความระมัดระวัง version อย่างชัดเจน

---

*กลับไปที่ index คู่มือ: [`../README.md`](../README.md) — คำศัพท์: [`../glossary.th.md`](../glossary.th.md)*
