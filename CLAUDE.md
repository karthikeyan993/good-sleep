# Sleep tracker — project brief for Claude Code

Personal sleep tracker, built from my own hardware and my own software. I own everything end to end:
sensor → firmware → my phone app → my storage → my sleep analysis. No vendor SDKs or closed apps.
Full cost and build plan: https://claude.ai/artifact/X8z2KFZJ2zPk3WX2EABFsM

## Hardware (bought in India)

- **Seeed XIAO nRF52840 Sense** — nRF52840 (BLE 5.0), onboard 6-axis IMU (LSM6DS3TR-C), 2 MB QSPI flash,
  LiPo charging. 21 × 17.5 mm.
- **MAX30101 optical sensor** (SmartElex breakout) — green, red and infrared LEDs, 18-bit ADC, I2C address 0x57.
  Use **green** for heart rate / HRV on the wrist (stronger signal, less motion noise), **IR as a motion reference**,
  and **red + IR** for SpO2 estimates.
- **3.7 V 150 mAh LiPo** with protection board.
- **W25Q128JV 16 MB SPI NOR flash** (decided 14 Nov: raw nights; 3.3 V "JV" part, not the 1.8 V "JW").
- **DRV2605L haptic driver + 10 mm coin vibration motor** (I2C address 0x5A) for move reminders and a smart alarm.
- Optional later: skin temperature, **TMP117** preferred over MAX30205 (both I2C, default address 0x48).
- Worn as a wristband (20 mm quick-release hook-and-loop strap); may also be worn on the upper arm with a longer strap.
- Known hardware limits: the MAX30101's green LED wants a 4.5–5 V LED supply but the breakout runs at 3.3 V (test
  it at bench step 4; IR is the night fallback); cut the breakout's power-LED jumper (~1.4 mA).
- Parts, prices, wiring, power budget and safety rules: https://claude.ai/artifact/SA2F6snirbWCmEdQUTUNmn

## Phases

1. **Bench prototype.** Wire XIAO + MAX30101 + battery; stream raw green/red/IR and accelerometer data to a laptop.
   Done when a clean pulse wave shows while sitting still.
2. **Overnight firmware** (decided 14 Nov: **C on Zephyr**, built with `west`; a Zig rewrite is parked for later).
   Sample accelerometer at 25–50 Hz (the IMU's 26 or 52 Hz rates) and PPG at 50–100 Hz with timestamps (time synced
   from the Mac), buffer to the 16 MB SPI flash all night, low-power between samples, bulk download over a custom BLE
   GATT service in the morning. Done when 8 hours log on one charge.
3. **Sync, server and dashboard** (decided 14 Nov). A Python sync script on the MacBook (`bleak`) downloads the night
   over BLE each morning and uploads it unchanged to my server (**Rust** + PostgreSQL, decided 15 Nov: my server has
   1 GB RAM and runs other work; built after the Python course). The dashboard is a web app served by the server and saved to the iPhone Home
   Screen: charts of heart rate, HRV and movement. A native iPhone app (Swift) is parked; Safari has no Web Bluetooth.
4. **Case and wear tests.** 3D-printed two-part case with a clear window over the sensor, an opaque light barrier
   between LEDs and photodetector, 20 mm strap lugs, USB-C reachable for charging.
5. **Sleep algorithm** (Python). Split nights into 30-second epochs. Features: movement, mean HR, HRV (RMSSD from
   beat-to-beat intervals detected in green PPG), breathing rate, time of night. Start with Cole-Kripke-style
   sleep/wake from motion, then train wake / light / deep / REM on public labelled data (PhysioNet has wearable
   heart-rate + accelerometer datasets with EEG-scored stages). Outputs: hypnogram, total sleep time, time to fall
   asleep, sleep efficiency, wake after sleep onset, resting HR, overnight HRV.
   Validation: HR vs a fingertip pulse oximeter; sleep times vs my sleep diary.
6. **Optional custom PCB** (KiCad, JLCPCB, one-sided assembly) for a thinner device.
7. **Extras** (after Phase 2): vibration (DRV2605L), steps (the IMU's built-in step counter), move reminder,
   smart alarm with double-tap to stop. Settings travel from the dashboard to the band at the morning sync.

## Where this sits in my plan

- Roadmap (todoist repo, ROADMAP.md): every day touches this project (15 Nov). Firmware = weekday evenings 18:00–20:00
  (a King hour, then the board hour), Saturday 1h, Sunday 3h hands-on, with King / CS:APP / OSTEP as the concept
  source (hybrid). The server comes after the Python course, in Rust, in the morning Backend slot.
- Tasks: the **Good Sleep** board in Todoist. Build plan: https://claude.ai/artifact/V8tGNCdFt2NYx6RFtiKZuu ·
  3D concept: https://claude.ai/artifact/HyKwjZAfPRNs3kaQSsKwMd

## Repo layout (create as needed)

- `firmware/` — Zephyr application (`west`) for the XIAO nRF52840 Sense (`xiao_ble/nrf52840/sense`)
- `sync/` — Mac sync script (Python, `bleak`)
- `analysis/` — Python package: loaders, epoching, PPG beat detection, features, models, reports; tests in `analysis/tests/`
- `server/` — Rust backend + web dashboard
- `hardware/` — case CAD, wiring diagram, later KiCad files
- `docs/` — decisions, data format spec, notes
- `data/` — local recordings, **git-ignored**; commit only small synthetic or anonymised samples under `analysis/tests/fixtures/`

## Working rules

- This repo often runs in a Claude Code cloud session. The cloud VM has **no Bluetooth or USB access to my board**.
  Flashing, serial monitoring and BLE tests happen on my machine. In the cloud, make firmware compile
  (`west build`) and test analysis code against recorded or synthetic fixture files.
- Define the on-device log format and BLE transfer protocol in `docs/` before writing both ends, and version it.
- My health data is private: never commit real recordings, tokens or keys. Keep secrets in environment variables.
- Prefer small, reviewable steps, with tests for the analysis code.
- Always work on `main`: no feature branches, no PRs. A cloud session that starts on a `claude/...` branch switches to
  `main` and pushes `main`; any other branch gets rebased onto `main` and deleted. This line is my standing permission.
- Not a medical device: label sleep stages and SpO2 as estimates in any UI text.
- Ask me before changing the phone platform, app framework or firmware RTOS, adding a paid service,
  or changing the data format.

## Agents (from ROADMAP.md in the todoist repo)

- **Learning** (C / King, CS:APP, the course): the agent explains, never solves.
- Languages: firmware in C (a Rust port with Embassy is parked); Swift only drop-in, when an iOS app is needed, AI-native.
- **This project**: I decide the data model, API shape, log format and boundaries first, in writing. The agent writes;
  I review every diff; a line I can't explain doesn't go in. The first version of each new concept is by hand (my
  MAX30101 driver, the first endpoint), then the agent does the repetitive variants.
- README has a **Decisions** section: what I chose, the alternatives, and where I overruled the agent.
