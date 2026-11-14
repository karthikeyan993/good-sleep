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
- Optional: **MAX30205** skin-temperature sensor (I2C, default address 0x48).
- Worn as a wristband (20 mm strap); may also be worn on the upper arm with a longer strap.

## Phases

1. **Bench prototype.** Wire XIAO + MAX30101 + battery; stream raw green/red/IR and accelerometer data to a laptop.
   Done when a clean pulse wave shows while sitting still.
2. **Overnight firmware** (PlatformIO, Arduino framework or Zephyr — propose and ask me). Sample accelerometer at
   25–50 Hz and PPG at 50–100 Hz with timestamps (time synced from the phone), buffer to flash all night, low-power
   between samples, bulk download over a custom BLE GATT service in the morning. Done when 8 hours log on one charge.
3. **Phone app and storage.** Morning sync over BLE, save to local SQLite, optional upload to my server, charts of
   heart rate, HRV and movement. Platform: **TBD, confirm with me before scaffolding.**
4. **Case and wear tests.** 3D-printed two-part case with a clear window over the sensor, an opaque light barrier
   between LEDs and photodetector, 20 mm strap lugs, USB-C reachable for charging.
5. **Sleep algorithm** (Python). Split nights into 30-second epochs. Features: movement, mean HR, HRV (RMSSD from
   beat-to-beat intervals detected in green PPG), breathing rate, time of night. Start with Cole-Kripke-style
   sleep/wake from motion, then train wake / light / deep / REM on public labelled data (PhysioNet has wearable
   heart-rate + accelerometer datasets with EEG-scored stages). Outputs: hypnogram, total sleep time, time to fall
   asleep, sleep efficiency, wake after sleep onset, resting HR, overnight HRV.
   Validation: HR vs a fingertip pulse oximeter; sleep times vs my sleep diary.
6. **Optional custom PCB** (KiCad, JLCPCB, one-sided assembly) for a thinner device.

## Repo layout (create as needed)

- `firmware/` — PlatformIO project for the XIAO nRF52840
- `app/` — phone app
- `analysis/` — Python package: loaders, epoching, PPG beat detection, features, models, reports; tests in `analysis/tests/`
- `server/` — optional upload backend (keep simple)
- `hardware/` — case CAD, wiring diagram, later KiCad files
- `docs/` — decisions, data format spec, notes
- `data/` — local recordings, **git-ignored**; commit only small synthetic or anonymised samples under `analysis/tests/fixtures/`

## Working rules

- This repo often runs in a Claude Code cloud session. The cloud VM has **no Bluetooth or USB access to my board**.
  Flashing, serial monitoring and BLE tests happen on my machine. In the cloud, make firmware compile
  (`pio run`) and test analysis code against recorded or synthetic fixture files.
- Define the on-device log format and BLE transfer protocol in `docs/` before writing both ends, and version it.
- My health data is private: never commit real recordings, tokens or keys. Keep secrets in environment variables.
- Prefer small, reviewable steps: one feature per branch/PR, with tests for the analysis code.
- Not a medical device: label sleep stages and SpO2 as estimates in any UI text.
- Ask me before choosing the phone platform, app framework or firmware RTOS, adding a paid service,
  or changing the data format.
