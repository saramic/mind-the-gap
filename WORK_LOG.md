# WORK_LOG — mind-the-gap

> A presence-aware household routine agent: DFRobot C4002 mmWave + FireBeetle 2
> ESP32-C6 feeding live room occupancy into a home MCP server.
> "Mind the gap": the gap between what the schedule _says_ happened and what
> _actually_ happened. And there's a train in it.

**Repo:** `github.com/saramic/mind-the-gap`
**Blog (GitHub Pages):** `saramic.github.io/mind-the-gap`

Alternative names if taken or not liked: `who-goes-there`, `room-for-thought`,
`present-tense`.

---

## 1. The brief (as submitted to DFRobot)

- **Kit:** Kit 1 (C4002 mmWave presence sensor + FireBeetle 2 ESP32-C6 DFR1075)
- **Idea:** Give an AI agent, via a home MCP server, live awareness of where
  people actually are in the house. The C4002 handles stationary and
  micro-movement presence in key rooms (kitchen, living room, kids' rooms).
- **Routines the agent already has as tools:** dog feeding, cat feeding,
  dishes/dishwasher cycles, vacuuming, door/lock status.
- **What presence adds:**
  - Don't run the vacuum while someone is stationary in that room.
  - Confirm the dog was actually fed by checking that someone was near the
    kitchen at the expected time.
  - Hold off on dish reminders while the kitchen is occupied.
  - Overall, it turns a dumb scheduler into a presence-aware routine agent.
- **Professional angle:** this is a testbed for MCP-based sensor integration patterns I can bring back to real agentic/API work.
- **Positioning:** strong on software and AI, still "old but limited" on hands-on electronics. Keep the barrier to entry low (more code than wiring), lean on DFRobot docs and the community, and write up the rough edges honestly.
- **Format:** both written and video.

### Commitments

| Milestone                                                                 | Due                    | Status |
| ------------------------------------------------------------------------- | ---------------------- | ------ |
| Single-room prototype: one C4002 → MCP server → agent handles dog feeding | Kit received + 2 weeks | ⬜     |
| Multi-room + fuller routines (dishes, vacuuming, cat feeding)             | Kit received + 30 days | ⬜     |
| Published on DFRobot Makelog                                              | With final write-up    | ⬜     |

**Kit received:** `YYYY-MM-DD` ← fill in, then derive both dates.

---

## 2. Links

### Where it gets published

- **Primary (required):** DFRobot Community Makelog.
  https://community.dfrobot.com/makelog-all-1-3.html
- **Repo + progress blog:** https://github.com/saramic/mind-the-gap →
  https://saramic.github.io/mind-the-gap/
- **Long-form / cross-post:** https://failure-driven.com/

### Social (building the failure-driven brand, linked from saramic)

- Instagram: https://www.instagram.com/failuredriven/
- X: https://x.com/failure_driven
- YouTube: https://www.youtube.com/@failure-driven
- TikTok: https://www.tiktok.com/@failure_driven
- LinkedIn: https://www.linkedin.com/company/failure-driven/

### Existing profiles and track record (link from README and posts)

- element14: https://community.element14.com/members/saramic
- GitHub: https://github.com/saramic
- Green Brain (element14): https://community.element14.com/challenges-projects/design-challenges/on-the-line/b/projects/posts/green-brain---distributed-greenhouse-control-platform
- Sentinel Box (element14): https://community.element14.com/challenges-projects/design-challenges/smart-security-and-surveillance/b/projects/posts/sentinel-box---consensus-based-lock-box
- https://saramic.github.io/vape-cell-EV/
- https://saramic.github.io/green-brain/
- https://saramic.github.io/sentinel-box/
- https://saramic.github.io/ham-radio/
- https://saramic.github.io/paws-and-wait/

---

## 3. Architecture (target)

```
 [C4002 mmWave] --UART--> [FireBeetle 2 ESP32-C6] --Wi-Fi (WebSocket/HTTP or MQTT)--> [Home MCP server]
                               |   \                                                     |  tools: get_room_presence,
                               |    \--BLE GATT--> phone / Web Bluetooth                 |  presence_history, ...
                               |                                                         v
                               \--PWM--> motor driver --> N-scale train (demo)     [AI agent + routine tools]
```

- **One event shape everywhere:** `{ room, type: "presence", state, distance?,
ts }`. The fake button, the real C4002, the web UI, BLE, and MCP all speak
  it.
- **The device stays dumb:** it reports state and the MCP server owns the
  history and the reasoning.
- Confirm the C4002 interface and pinout against the DFRobot wiki before wiring.

---

## 4. Plan of attack

### Phase 0: Scaffolding (day 0)

- [ ] Create the `mind-the-gap` repo with a README, this WORK_LOG.md,
      `firmware/`, `web/`, `mcp-server/`, and `docs/` (GitHub Pages).
- [ ] Set up the Arduino core for ESP32 3.x (needed for the C6). Select the
      FireBeetle 2 ESP32-C6 board and enable USB CDC On Boot.
- [ ] Write a blog post template, and a "how I'll measure things" page so the
      benchmarks can be reproduced.

### Phase 1: FireBeetle 2 ESP32-C6 bench review (laptop only, no wiring)

- [ ] **Identify the board:** run `esptool.py chip_id` / `flash_id` (or
      esptool-js in the browser) and record the chip revision, MAC, and flash size.
- [ ] **Hello hardware:** blink the onboard LED, read the BOOT button, and
      print a Serial chip report (CPU MHz, heap, flash, internal temperature).
- [ ] **Compute:** time integer and float loops, run CoreMark, and measure
      flash read/write speed and boot-to-`setup()` time.
- [ ] **Wi-Fi:**
  - Do an RSSI scan and check for an 802.11ax (Wi-Fi 6) link.
  - Measure throughput with the ESP-IDF `iperf` example (iperf2 on the laptop):
    TCP and UDP, both directions.
  - Measure ping jitter, cold boot → connected time, and reconnect time after a
    router reboot.
  - Do a house walk on a power bank, logging RSSI per room. That map becomes
    input for sensor placement.
- [ ] **BLE:**
  - Inspect a GATT service with nRF Connect.
  - Measure notify throughput with and without a larger MTU and 2M PHY.
  - Test Web Bluetooth from Chrome. iOS Safari doesn't support it, so note
    that.
- [ ] **Optional:** timer-wake deep sleep and battery voltage via the onboard
      divider (check the wiki for the pin). Defer Thread/Zigbee, since they need a
      second 802.15.4 device.
- 📝 **Post #1:** "Reviewing the FireBeetle 2 ESP32-C6 with nothing but a
  laptop"

### Phase 2: Mini app, "Hallway Station Board" (still no wiring)

- [ ] The BOOT button emits a fake `presence: approaching` event.
- [ ] An async web server and WebSocket dashboard (train-station
      departure-board style) shows uptime, RSSI, temperature, heap, and the
      presence state.
- [ ] A BLE GATT service mirrors the same state and adds a writable command
      characteristic. A Web Bluetooth page controls it from laptop or Android.
- [ ] A "Depart" button on the web page and the phone pulses the LED, which
      stands in for the train.
- [ ] **Stretch:** Improv Wi-Fi provisioning over BLE.
- 📝 **Post #2:** "One firmware, two radios: Wi-Fi + BLE dashboard on the
  ESP32-C6", plus a short video of the button driving the departure board.

### Phase 3: C4002 single room → MCP (**2-week commitment**)

- [ ] Wire the C4002 to the FireBeetle and replace the button with a real
      presence event.
- [ ] Calibrate in the kitchen: range, sensitivity, the stationary-person hold
      time, and false positives from the dog.
- [ ] MCP server (TypeScript/Node):
  - Ingest the event stream.
  - Expose the `get_room_presence(room)` and `presence_history(room, window)`
    tools.
- [ ] Agent routine: dog feeding is confirmed by kitchen presence near feeding
      time, and reminders are suppressed or escalated accordingly.
- 📝 **Post #3:** "Giving my AI agent eyes: mmWave presence as an MCP tool".
  This is the main DFRobot Makelog draft.

### Phase 4: Multi-room + routines (**30-day commitment**)

- [ ] Extend presence to the living room and kids' rooms, using the Phase 1
      RSSI map for placement.
- [ ] Add the routines:
  - Vacuum blocked while someone is stationary in the room.
  - Dish reminders held while the kitchen is occupied.
  - Cat feeding confirmation.
- [ ] Write up the integration patterns: debouncing, staleness/TTL, privacy,
      and how an agent should treat "probably present".
- 📝 **Final write-up:** publish to DFRobot Makelog and cross-post to
  failure-driven.com.

### Phase 5: Bonus demo, the hallway N-scale train

- [ ] The C4002 detects someone approaching the hallway layout and the train
      departs. PWM goes to a small motor driver on a separate track supply, never
      powered from the ESP32.
- [ ] Phone and web "Depart" controls carry over from Phase 2.
- 📹 This is the made-for-shorts piece for TikTok, Instagram, YouTube Shorts, and X.

---

## 5. Content plan

| Channel                   | What goes there                                                                           |
| ------------------------- | ----------------------------------------------------------------------------------------- |
| DFRobot Makelog           | Final project write-up (required), plus optionally the C6 review                          |
| GitHub repo + Pages       | Everything: code, this log, each post                                                     |
| failure-driven.com        | Cross-posts and "what went wrong" reflections                                             |
| YouTube                   | Phase 1 review walkthrough and final demo                                                 |
| TikTok / Instagram / X    | 15–60s clips: departure board, the train departing, the sensor "seeing" through stillness |
| LinkedIn (failure-driven) | Professional angle: MCP + sensor integration patterns for agentic systems                 |

Post from saramic and link to failure-driven to build that audience.

---

## 6. Log

### 2026-10-08

- Original brief recovered. Repo name chosen: `mind-the-gap`.
- Plan: start with the laptop-only FireBeetle review (Phase 1), then the
  Station Board mini app (Phase 2) before the C4002 arrives on the bench.
- **Next:** create the repo, install the ESP32 Arduino core 3.x, and run Phase
  1 identify + hello hardware.
