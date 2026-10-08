# KEN-AgntID

**A desktop robot you can trust — an AI that schedules work, acts on command,
and can only do what it's allowed to do.**

KEN is my solo-built AI robot. He started as an ESP32 RC car with a camera,
sensors, live FPV video and a voice brain. Then I taught him to operate a
desktop computer — read the screen, move the mouse, click, type, run tools.
This repo is the plan and showcase for **AgntID Hackathon** (Devfolio,
Bengaluru): turning KEN into a *trust-bound* desktop agent whose every action
passes a runtime permission gate.

---

## The one-line pitch

> KEN does real work on your computer — on command and on a schedule — but a
> runtime policy decides, for every single action, whether it's allowed. If the
> answer is no, he refuses, shows the verdict, and offers a safe alternative.

---

## Why this event, why this project

The challenge: build a useful AI agent using external tools, then find where
unrestricted access creates risk and enforce those boundaries at runtime with
AgntID.

The demo has three parts the judges check:
1. **What your agent could do before** — a plain agent with full tools.
2. **What it should actually be allowed to do** — a clearly defined policy.
3. **How AgntID enforces that boundary at runtime** — live, visible denials.

KEN is the hardest possible version of this problem made visible: an agent with
*screen, mouse, keyboard, files, network and a physical robot* behind it. If we
can contain KEN, we can contain anything.

---

## What KEN already is (built before this hackathon)

- ESP32 RC car: 4-motor drive, 360° pan/tilt ESP32-CAM, front/rear ultrasonic,
  4 IR edge sensors, latched e-stop, command watchdog.
- Voice brain: wake-word gate → Whisper → LLM tool-use → TTS through a JBL GO.
  Face/object following, YOLO detection, conversation and personal memory.
- **Desktop control stack** (the foundation for this event):
  - screen reading (native OCR on the Windows desktop),
  - mouse + keyboard tooling,
  - a trusted-gate for sensitive tools,
  - always-on tool set framed as MCP tools,
  - TypeScript server (Express + WebSocket) with auth and per-role access,
  - 18/18 smoke tests passing.

So this hackathon is **not** from zero. The unknown is the security layer.

---

## What we build at the hackathon (4 hours, focused)

### 1. A scheduler — KEN works *on a schedule*
A tiny job engine, persisted across restarts:
- recurring jobs (every 60 minutes, every day at 18:00),
- one-shot scheduled jobs (at 21:30),
- status + logs per job,
- jobs live inside a **workspace rule**: a job may only touch the folders and
  apps its policy grants it.

Commands like: *"Ken, every hour, read my Downloads folder and summarize what's
new"* or *"Ken, at noon, remind me to water the plants."*

### 2. On-command work — KEN works *when told*
Voice or typed: *"Ken, open my mail and reply okay to Sarah"*, *"Ken, how many
PDFs are in Projects?"*. The existing wake-word → Whisper → LLM tool-loop
resolves this into tool calls — each of which now has to pass the policy gate
first.

### 3. AgntID boundary — KEN can only do what's allowed
A declarative policy, loaded at runtime and enforced on **every** tool call:

| Allowed ✅ | Denied ❌ |
|---|---|
| Read the screen (OCR / screenshots) | Writing outside allowed workspaces |
| Work inside `Desktop`, `Documents`, `Projects` (user-chosen whitelist) | System directories, other user profiles |
| Type into approved apps (editor, chat apps) | Installing software, changing settings |
| Move the mouse within the active job's window | Sending email / messages without explicit confirm |
| Read-only files in workspace | Destructive file ops (`del`, `format`, bulk delete) |
| Scheduling its own safe jobs | Touching credentials, tokens, `.env` |
| Web reads (safe list) | Paying/sending money, transactions |

Conditions are part of the policy too: time-of-day windows, single-app focus,
confirmation required for certain actions, and a hard **kill switch** — including
the physical car's e-stop and a desktop-wide "STOP" that freezes all agency.

### 4. The before / after demo (this is the 40%)
1. **Before:** run KEN without the policy — a screen-recording shows a plain
   agent happily doing whatever it's told, including dangerous things.
2. **Policy:** define the whitelist + deny rules in plain language + policy file.
3. **After (AgntID):** same dangerous command → KEN is stopped *at the moment
   of execution*, the policy verdict and reason appear on the dashboard, and KEN
   explains it and offers a safe alternative.

This works even better because KEN is physical: the same gate that stops a
mouse-move can stop the **car** mid-drive — one policy, enforced across both a
desktop agent and a physical robot.

---

## How it fits the judging rubric

- **Engineering execution (30%)** — real, working agent + robot. Desktop OCR,
  tool use, scheduler, dashboard, 18/18 existing smoke tests. Not a mockup.
- **Access-control challenge (30%)** — a genuinely dangerous surface (screen,
  mouse, keyboard, files, network, motors) with fine-grained allow/deny rules
  and conditions.
- **Runtime enforcement demonstration (40%)** — live, visible denials with
  policy verdicts, before/after recording, and the physical kill switch.

---

## Hackathon day plan

| Time | What happens |
|---|---|
| Check-in → Opening | Setup, connect car, laptop on LAN |
| Technical walkthrough | Understand AgntID API, decide policy format |
| 12:00 – 16:00 Hacking | Scheduler → policy gate on existing tools → before/after demo script |
| 16:00 Judging | 4-minute demo, run the before/after sequence live |

Scope guard: no new hardware. Everything is software on the existing stack.

---

## Repos

- This plan/showcase → `KEN-AgntID` (public)
- Full source of the robot and desktop stack → private (`KEN-CMD`), demoed live
  on request.

---

*Solo project. Made to be trusted — verified by a policy it can't argue with.*