# Echo Do Assistant — Companion App PRD

**Status:** Draft v0.1
**Author:** Product (Santosh)
**Last updated:** 2026-06-13
**Surfaces:** Web app (responsive dashboard) + Mobile app (iOS/Android)
**Companion to:** Echo Do Assistant hardware (ESP32-WROOM-32D · mic · speaker · servo · Li-Po)

---

## 1. Product Context

The Echo Do Assistant is a **utility + IoT hybrid**:

- **Utility / assistant side** — captures voice (MAX9814 mic), responds with audio (PAM8403 speaker). It *understands and talks*.
- **IoT / physical side** — Wi-Fi + BLE connected (ESP32), and unlike a pure smart speaker it has an **actuator** (SG90 servo, 180°) that performs a real-world "**do**" action — flip a switch, push a button, dispense, point, wave, lock/unlock, etc.
- **Battery-powered & rechargeable** — Li-Po + TP4056 charger + MT3608 5V boost. Portable, with a real battery-health story.
- **Maker-grade & 3D-printed** — open hardware, hackable, accessory-friendly.

The hardware alone is "dumb" without a brain to configure it. **The companion app is the product experience**: provisioning, control, telemetry, lifecycle, skills, and the community that keeps a maker device alive.

### Why an app is mandatory (not optional)
A headless box with a mic, speaker, and servo cannot tell you its battery health, let you define what the servo *does*, push firmware, or show you that the servo has actuated 40,000 times and is wearing out. All of that lives in the app.

---

## 2. Personas

| Persona | Description | Primary needs |
|---|---|---|
| **The Maker (builder)** | Built it from the BOM, comfortable with firmware/wiring | Provisioning, OTA, GPIO/skill scripting, telemetry, community |
| **The Household user** | Received/bought an assembled unit | Simple control, routines, voice setup, "it just works" |
| **The Fleet owner** | Runs many units (classroom, makerspace, small biz) | Multi-device dashboard, bulk OTA, health alerts, roles |
| **The Developer** | Extends the platform | Public API, SDK, custom skills, accessory definitions |

---

## 3. Goals & Success Metrics

| Goal | Metric |
|---|---|
| Frictionless setup | < 3 min median time from unbox → first voice command (TTFV) |
| Reliable connectivity | > 99% device-online uptime reported via heartbeat |
| Lifecycle visibility | 100% of units report battery SoH + servo actuation count |
| Extensibility adoption | > 40% of active users create ≥1 custom routine/skill in 30 days |
| Community stickiness | > 25% of users view a KB article or community post monthly |
| Update hygiene | > 80% of fleet on latest stable firmware within 30 days of release |

---

## 4. Feature Areas (Epics)

### EPIC 1 — Onboarding & Provisioning
The first-run experience. Hardware is BLE + Wi-Fi, so use BLE for setup, Wi-Fi for operation.
- **BLE pairing / discovery** — app finds the device over Bluetooth on first boot.
- **Wi-Fi provisioning** — SoftAP or BLE-based credential handoff (SSID + password) to the ESP32.
- **Guided setup wizard** — name the device, choose a room/zone, pick a wake word, calibrate mic gain (MAX9814 has AGC), test the speaker, home the servo.
- **Servo "action" calibration** — define the servo's 0°/90°/180° physical end-stops and what each represents ("off / neutral / on"). Critical because the servo is the device's reason-to-exist.
- **QR/serial claim** — bind the physical unit to the user account (anti-theft, warranty, support).
- **Build-from-BOM mode** — for makers: link the on-device app straight to the build guide/BOM (reuse the existing project files) and a firmware-flash helper.

### EPIC 2 — Device Dashboard & Real-Time Telemetry
The "is it alive and what's it doing" view.
- **Live status tile** — online/offline, Wi-Fi RSSI, IP, firmware version, uptime.
- **Real-time telemetry stream** (MQTT/WebSocket): battery voltage & %, charging state (TP4056), boost-rail health (5V stability), CPU temp, free heap/memory, mic input level (VU meter), last command, servo position.
- **Servo activity monitor** — current angle, last actuation timestamp, actuation in progress.
- **Audio meters** — live mic level + last TTS/audio played.
- **Connectivity quality** — RSSI history, disconnect events, reconnect attempts.

### EPIC 3 — Usage Analytics & Insights
Turn raw telemetry into stories.
- **Command history & logs** — every voice command, intent matched, action taken, success/fail.
- **Usage heatmaps** — by hour/day/week; "most active times," "quiet periods."
- **Top intents/skills** — what users actually ask for.
- **Servo actuation counter & trends** — total + per-day actuations (this is the wear-driver).
- **Energy usage** — discharge curves, charge cycles, "runtime per charge" trend.
- **Weekly digest** — auto-generated "Your Echo Do this week" summary (commands, actuations, uptime, battery health).
- **Anomaly flags** — sudden drop in commands recognized, frequent disconnects, abnormal current draw.

### EPIC 4 — Health, Decay & Lifecycle Management ⭐
The differentiator for a battery + actuator device. Hardware *wears out*; the app makes that visible and actionable.
- **Battery State of Health (SoH)** — track capacity fade across charge cycles; estimate remaining battery lifespan; "replace battery soon" prediction. Li-Po degrades — surface it.
- **Charge-cycle counter** — count full-equivalent cycles via the TP4056/coulomb estimate.
- **Servo wear & lifecycle** — SG90 nylon gears have a finite actuation life. Track total actuations vs. a rated budget; show a **wear gauge** and "servo end-of-life forecast"; recommend the **MG90S metal-gear** upgrade (already noted as an alternative in the BOM).
- **Predictive maintenance** — "Your servo is at 78% of rated life; order a replacement." One-tap reorder via the accessory/part links already in the BOM.
- **Component health board** — per-component status (mic, speaker, servo, battery, charger) with green/amber/red.
- **Decay timeline / device age** — visualize the device's life: install date, cumulative runtime, components replaced, firmware history. A "digital logbook."
- **Calibration drift detection** — mic gain or servo end-stop drift over time → prompt recalibration.
- **End-of-life & recycling** — guidance for safe Li-Po disposal and 3D-print part recycling.

### EPIC 5 — OTA Firmware & Software Updates
ESP32 supports OTA — make it first-class.
- **Over-the-air firmware updates** — push signed firmware to one device or the whole fleet.
- **Release channels** — Stable / Beta / Nightly (makers opt into beta).
- **Changelogs & release notes** in-app.
- **Staged rollout & auto-rollback** — canary a % of fleet; auto-revert on boot-loop/health regression.
- **Scheduled updates** — "update overnight while charging."
- **Version pinning & history** — see and roll back to prior firmware.
- **Config/skill sync** — push routine/skill changes independently of firmware.

### EPIC 6 — Skills, Routines & Extensibility ⭐
The "Do" in Echo Do. This is where the servo + voice become programmable.
- **Skill library** — installable capabilities (timers, weather, reminders, "knock-knock," servo-action skills).
- **Visual automation builder** — IF (trigger) THEN (action): trigger = voice phrase, time, telemetry threshold (e.g., battery < 20%), webhook, geofence; action = move servo to angle X, play audio, send notification, call webhook.
- **Custom wake words / phrases** — map a phrase → an action profile.
- **Servo action presets** — named physical actions ("Press button," "Flip switch," "Wave") with angle/speed/sequence; sharable.
- **Macros & sequences** — multi-step (speak → wait → actuate → confirm).
- **Third-party integrations** — IFTTT, Home Assistant, Matter/HomeKit bridge, Zapier, MQTT.
- **Webhooks & REST triggers** — fire device actions from anything.
- **Local/offline rules** — basic rules run on-device even without cloud.

### EPIC 7 — Accessories & Marketplace
A maker device thrives on add-ons.
- **Accessory catalog** — compatible add-ons: MG90S servo upgrade, larger Li-Po, mic windscreen, alternate enclosures, mounting brackets, button-pusher arms, etc.
- **3D-print accessory library** — downloadable STL/STEP for printable add-ons (servo arms, wall mounts, faceplates). Tie into the existing print-settings metadata.
- **One-tap reorder** — reuse BOM vendor links (Amazon/AliExpress/Adafruit/DigiKey) for consumables (battery, servo, screws, inserts).
- **Compatibility checker** — "Will this accessory fit my enclosure rev?"
- **Accessory auto-detect** — if an add-on exposes an ID over I2C/GPIO, app recognizes and configures it.
- **(Optional) Marketplace for community skills/accessories** — creators publish, others install.

### EPIC 8 — Knowledge Base & Self-Help
- **Searchable KB** — setup, wiring, troubleshooting, FAQs (seed from the existing GUIDE.md + connection maps).
- **Interactive build guide** — the existing 4-phase guide (Fabricate → Wire → Bring-up → Assemble) as a checklist, with the electrical/mechanical connection diagrams.
- **Wiring & pinout viewer** — render the ELECTRICAL/MECHANICAL connection JSON as interactive diagrams.
- **Contextual help** — error codes from telemetry deep-link to the matching KB fix.
- **Guided troubleshooters** — "Servo not moving?" decision-tree that reads live telemetry (checks 5V rail, PWM, servo current).
- **Video & 3D model viewer** — embed the turntable video + interactive 3D of the enclosure.
- **AI assistant / chatbot** — natural-language help grounded in the KB + device telemetry ("why did my battery drain fast yesterday?").

### EPIC 9 — Community & Social
What keeps a maker product alive after the novelty.
- **Community forum / feed** — Q&A, show-and-tell, mods.
- **Skill & routine sharing** — publish your automations; install others' with one tap (with ratings/reviews).
- **Build showcase** — photos/videos of custom enclosures and use-cases.
- **Leaderboards / badges** — gamify (most actuations, longest uptime, "100-day streak," "first to flash beta").
- **Challenges & events** — monthly "build a ___ with your Echo Do" contests.
- **Expert/maker profiles & following.**
- **Bug reports & feature voting** — public roadmap with upvotes.
- **Direct device-log attach to posts** — share a telemetry snapshot when asking for help.

### EPIC 10 — Voice, Audio & Action Configuration
Device-specific tuning the hardware demands.
- **Wake-word & language selection.**
- **Mic sensitivity / AGC tuning** (MAX9814) + noise-floor calibration.
- **Speaker volume, TTS voice, audio feedback sounds.**
- **Voice command trainer** — record/confirm custom phrases.
- **Servo motion tuning** — speed, easing, angle limits, hold-vs-release, jitter suppression.
- **Privacy mute** — hardware/software mic-off with clear indicator.

### EPIC 11 — Power & Energy Management
- **Battery dashboard** — %, voltage, time-to-empty/full, charging state.
- **Power profiles** — Performance / Balanced / Low-power (deep-sleep tuning for ESP32).
- **Charge optimization** — "charge to 80% to extend Li-Po life" toggle.
- **Low-battery automations** — at X%, mute servo / notify / enter sleep.
- **Runtime estimator** — predicted hours given current usage pattern.

### EPIC 12 — Notifications & Alerts
- **Push/email/SMS** for: device offline, low battery, servo wear threshold, firmware available, anomaly detected, charging complete.
- **Alert rules & thresholds** — user-configurable.
- **Quiet hours / Do-Not-Disturb.**
- **Critical vs. informational tiers.**

### EPIC 13 — Multi-Device & Fleet Management
- **Device groups / rooms / zones.**
- **Fleet dashboard** — all devices, health at a glance, bulk actions.
- **Bulk OTA & config push.**
- **Roles & sharing** — owner / admin / guest; share a device with family or teammates.
- **Cross-device routines** — "when device A hears X, device B actuates."

### EPIC 14 — Security, Privacy & Permissions
- **Account & auth** — SSO, 2FA.
- **Encrypted transport** — TLS to cloud, signed OTA images.
- **Voice-data privacy controls** — opt-in cloud processing vs. on-device; local-only mode; clear data-retention and one-tap "delete my voice history."
- **Device access tokens & revocation.**
- **Audit log** — who did what to which device.
- **GDPR-style data export & delete.**

### EPIC 15 — Developer Platform (SDK & API)
- **Public REST + WebSocket/MQTT API** — read telemetry, send actions.
- **Skill SDK** — author/package/publish skills.
- **Accessory definition spec** — declare a new accessory (pins, capabilities, 3D model) in JSON.
- **Webhook & OAuth app registration.**
- **Sandbox / simulator** — test skills against a virtual device.
- **API keys, rate limits, usage dashboard.**

### EPIC 16 — Monetization (optional / future)
- **Free tier** — core control, single device, basic telemetry, community read.
- **Pro / Maker+** — extended telemetry history, fleet, beta channel, advanced automations, AI help.
- **Marketplace revenue share** — paid skills/accessories.
- **Hardware bundles & consumables** — battery/servo reorder margins.

---

## 5. Cross-Cutting Non-Functional Requirements
- **Offline-tolerant** — core control works on LAN even if cloud is down; queue telemetry for later sync.
- **Real-time** — telemetry latency < 1s; servo command round-trip < 500ms on LAN.
- **Scalable ingestion** — time-series store for telemetry (per-device, retention by tier).
- **Accessibility** — WCAG AA, voice-first considerations, large-text/contrast modes.
- **Cross-platform parity** — web + iOS + Android share a design system.
- **Resilience** — OTA auto-rollback; no brick on failed update.

---

## 6. Suggested Architecture (high level)
- **Device firmware (ESP32):** MQTT client (telemetry up, commands down), OTA agent, local rule engine, BLE provisioning service.
- **Cloud:** MQTT broker + ingest → time-series DB (telemetry) + relational DB (users/devices/skills) + object store (firmware, STLs, media).
- **API layer:** REST + WebSocket; auth/identity service.
- **Apps:** React/Next.js web (deployable on Vercel — already wired up), React Native mobile sharing logic/design system.
- **Pipelines:** firmware signing + staged rollout service; analytics/digest jobs.

---

## 7. Roadmap (phased)

### MVP (V0) — "Control & See"
EPIC 1 (provisioning), EPIC 2 (live telemetry), EPIC 10 (voice/servo config basics), EPIC 11 (battery dashboard), EPIC 5 (basic OTA), EPIC 14 (auth + privacy basics).

### V1 — "Understand & Extend"
EPIC 3 (analytics), EPIC 4 (health/decay/lifecycle ⭐), EPIC 6 (skills/routines ⭐), EPIC 8 (KB + interactive guide), EPIC 12 (alerts).

### V2 — "Grow & Open"
EPIC 7 (accessories/marketplace), EPIC 9 (community), EPIC 13 (fleet), EPIC 15 (developer SDK/API), EPIC 16 (monetization).

---

## 8. Open Questions
1. Cloud-connected by default, or local-first with optional cloud? (privacy vs. features)
2. Is voice intent processing on-device, cloud, or hybrid? (ESP32 is limited — likely cloud STT/NLU.)
3. What is the SG90 servo's rated actuation life for the wear model? (define the budget; recommend MG90S upgrade path)
4. Coulomb counting hardware for accurate battery SoH, or voltage-curve estimation only?
5. Single-tenant maker product, or fleet/B2B (makerspaces, education) as a first-class segment?
6. Marketplace: curated vs. open submission? Revenue share model?

---

## 9. Appendix — Hardware → App Telemetry Map
| Component | Part | App surfaces |
|---|---|---|
| Controller | ESP32-WROOM-32D | Wi-Fi RSSI, IP, firmware, CPU temp, heap, uptime, OTA |
| Mic | MAX9814 | Input level (VU), AGC/gain tuning, noise floor |
| Speaker/amp | PAM8403 | Volume, last-played, TTS voice, audio test |
| Actuator | SG90 servo | Angle, actuation count, wear gauge, action presets, EOL forecast |
| Battery | 3.7V Li-Po | %, voltage, SoH, charge cycles, runtime estimate |
| Charger | TP4056 | Charging state, charge-complete alert |
| Boost | MT3608 | 5V rail stability/health |
| Enclosure | 3D-printed | Accessory/STL library, build guide, 3D viewer |
