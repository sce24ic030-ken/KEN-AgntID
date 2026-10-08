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

## Who KEN is (the full picture)

KEN isn't a weekend toy — he's an on-going, working project with hardware,
vision, sensing, voice, memory and personality, all built and tested before
this hackathon.

### Hardware (the body)

| Part | What it is |
|---|---|
| ESP32 DevKit | drive controller, 4-motor drivetrain |
| ESP32-CAM (OV2640) | live JPEG video on a **360° pan/tilt** mount |
| Sensor node | **ultrasonic** front + rear, **4 IR edge sensors** (one per corner) |
| JBL GO speaker | voice in/out, paired to the laptop over Bluetooth |
| Laptop server | Node + TypeScript brain, WebSocket hub, Cloudflare tunnel for remote |

### Driving (the wheels)

- 4-motor differential drive, browser-controlled from keyboard (WASD/arrows) or
  an on-screen thumbpad.
- Speed limiter slider — hard-capped at the server, re-clamped at the car.
- Command watchdog on the car (default 1000 ms, 500 ms on LAN): if commands stop
  arriving, the wheels stop. Network failure = stopped car.
- No drive commands at boot; wheels stay still until told to move.
- E-stop is latched in firmware and only released deliberately.
- `TEST MODE` — motors simulated for safe bench testing.

### Vision (the eyes)

- Live FPV canvas — raw MJPEG pushed over WebSocket (low-latency, not slides).
- 360° pan / tilt camera mount, drag-to-aim from the dashboard, camera "peek".
- YOLO object detection — ~80 common object classes for scene context.
- Live vision overlay — face and object bounding boxes drawn on the video, with
  the current follow target highlighted.
- Optional multimodal vision — a vision LLM describes what the car sees so KEN
  can talk about his environment.

### Sensing (the instincts)

- Front and rear ultrasonic range; four IR edge sensors with live edge dots.
- `SENSORS` card on the dashboard: live front/rear cm + 4 edge dots +
  obstacle/edge tags.
- Sensor safety gate in the AI: obstacle/edge readings **override** AI drive
  commands before they are sent — always, no exceptions.
- Camera/stepper clicks, headlight control (1/0), all driven from software.

### Voice (the ears and mouth)

- Wake-word gated voice loop (default wake word "Ken"). KEN only acts when you
  start with "Ken…"; calling his name wakes him, then he auto-sleeps after
  `WAKE_IDLE_TIMEOUT_S` of silence.
- VAD → Whisper → LLM → TTS through the JBL GO. Barge-in: talking over him stops
  playback instantly.
- Type-to-talk — same LLM+TTS pipeline from the dashboard's AI CHAT text box;
  voice and typing share one conversation.
- Full live language switch — "speak Hindi" / "Spanish" / "Tamil" mid-talk, KEN
  hot-swaps reply language and STT. "go back to auto" returns to auto-detect.

### Memory (the mind)

- Conversation memory — chat history persists across restarts.
- Personal memory per person — new faces are enrolled on the fly
  ("remember my face as Sarah") and each known face gets their own profile:
  photo, facts, and the *full* conversation history since you first met. KEN
  greets you by face and talks to you about what only you two talked about,
  remembering it from the very first time.

### Autonomous behavior (the will)

- **Autonomous face-follow** — in AUTO mode KEN follows your face; switch
  targets by name ("follow Sarah"). Always gated by the safety layer.
- **Object-follow autonomy** — follow a detected object class ("Ken, follow the
  dog") or steer clear ("Ken, stay away from people"). Priority:
  **avoid > follow > face-follow**.
- **Patrol mode** — "start patrol" drives a sensor-guided pattern while
  narrating; "stop patrol" halts it.
- **Proactive conversation** — while awake, KEN occasionally speaks up based on
  what he sees (a face, an object, a nearby obstacle).
- **Wake on face** — he wakes himself the moment a face appears in frame
  (`WAKE_ON_FACE`).

### Personality (the character)

- Structured personality output: KEN replies as `{speech, action}` so he not
  only talks but **acts** — spins, does his happy wiggle-dance, zooms, cheats
  backward, winks his headlights, or peeks the camera. All gated by the safety
  layer, always.
- Voice + personality switching by voice: "use a British voice", "be more
  sarcastic".
- LLM tool-use for natural-language control: "back up", "follow Sarah", "turn
  the lights on", "open the camera" — no fixed regexes.
- Multi-turn tool refinement: the LLM chains tools (sense → react → report) to
  build richer behaviours.

### The cockpit (the dashboard)

- Video canvas, virtual thumbpad + keyboard, speed limiter slider.
- `AUTO` button flips driving authority to the AI; E-stop works in both modes.
- AI CHAT panel with live transcript, timestamps, awake/asleep pill, typing
  indicator and quick-command chips.
- Ping meters (browser↔server, server↔vehicle) and status pills for
  car / camera / sensor / audio / AI.

### Safety (the rules)

- Configurable command watchdog; sensor gate the AI can't bypass.
- Driving authority by mode: manual = dashboard, auto = AI. E-stop from anywhere,
  mid-voice-response included.
- Estop latched at firmware; no drive commands at boot; remote use only with
  real secrets (`AUTH_TOKEN` + per-device tokens), WSS only.

### Build status

Server, dashboard, all three firmware projects and the AI service are built;
smoke suite passes **18/18**; remaining checks need physical hardware on the
bench.

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