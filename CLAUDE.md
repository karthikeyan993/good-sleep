# Sleep tracker — project brief for Claude Code

Personal sleep tracker. I own the data end to end: sensor → my phone app → my storage → my sleep analysis.
Full cost and build plan: https://claude.ai/artifact/X8z2KFZJ2zPk3WX2EABFsM

## Phases

1. **Polar Verity Sense + my own app (current phase).** Upper-arm optical band, bought in India (₹9,024).
   - Use the official Polar BLE SDK: https://github.com/polarofficial/polar-ble-sdk (Android and iOS).
   - Keep the band in its **default mode**. It can record heart rate, PPI (beat-to-beat intervals), raw PPG at 55 Hz
     and accelerometer at 52 Hz to its own memory overnight (offline recording). "SDK mode" allows custom sample
     rates but turns off HR and PPI, which we need for HRV. Product docs:
     https://github.com/polarofficial/polar-ble-sdk/blob/master/documentation/products/PolarVeritySense.md
   - Morning flow: phone connects, downloads last night's recording, saves it locally, optionally uploads to my server.
2. **Sleep algorithm.** Python. Split nights into 30-second epochs. Features per epoch: movement, mean HR,
   HRV (RMSSD from PPI), breathing rate from PPG, time of night. Start with Cole-Kripke-style sleep/wake from motion,
   then train wake / light / deep / REM on public labelled data (PhysioNet has wearable heart-rate + accelerometer
   datasets with EEG-scored stages). Outputs: hypnogram, total sleep time, time to fall asleep, sleep efficiency,
   wake after sleep onset, resting HR, overnight HRV.
3. **Optional DIY wristband.** Seeed XIAO nRF52840 Sense (onboard 6-axis IMU, 2 MB flash, LiPo charging) +
   MAX30102 PPG + 150 mAh LiPo, optional MAX30205 skin temperature. Log to flash overnight, bulk-download over BLE
   in the morning. Validate against the Polar band worn on the same night.
4. **Optional custom PCB + 3D-printed case** (KiCad, JLCPCB).

## Repo layout (create as needed)

- `app/` — phone app (platform: **TBD, confirm with me before scaffolding**)
- `analysis/` — Python package: loaders, epoching, features, models, reports; tests in `analysis/tests/`
- `server/` — optional backend for uploads (keep simple: one API + Postgres or SQLite)
- `firmware/` — phase 3, PlatformIO project for the XIAO nRF52840
- `hardware/` — phase 4, case CAD and KiCad files
- `docs/` — decisions and notes
- `data/` — local recordings, **git-ignored**; commit only small synthetic or anonymised samples under `analysis/tests/fixtures/`

## Working rules

- This repo often runs in a Claude Code cloud session. The cloud VM has **no Bluetooth or USB access to my devices**.
  Anything that talks to the band or flashes firmware gets tested on my machine. In the cloud, test with recorded or
  synthetic fixture files instead.
- My health data is private: never commit real recordings, tokens or keys. Keep secrets in environment variables.
- Prefer small, reviewable steps: one feature per branch/PR, with tests for the analysis code.
- Not a medical device: label sleep stages and SpO2 as estimates in any UI text.
- Ask me before choosing the phone platform or app framework, adding a paid service, or changing the data format.
