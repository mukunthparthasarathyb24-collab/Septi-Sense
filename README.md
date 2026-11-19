# SeptiSense

**A wearable vital-sign monitor with a 6 / 12 / 24-hour sepsis early-warning outlook.**
Team project (4 members) for the **Utkrista 2026 Connect & Sync** hackathon, IIIT Sri City — **1st place**.

> Educational / research prototype. **Not a medical device.** Not clinically validated, and it does not diagnose sepsis or any other condition.

Forked from the team repository [Dharun01prasad/Septi-Sense](https://github.com/Dharun01prasad/Septi-Sense). The full technical write-up is in [REPORT.md](REPORT.md).

---

## How it works

```
Body sensors ──► ESP32 node ──(Wi-Fi, 1 record/s)──► Laptop backend ──► Web dashboard + SMS alerts
ECG, PPG/SpO2,    filter, R-peaks,                    store, breathing     plain-language status,
skin temperature  quality checks,                     rate, hourly         Low / Raised / High
                  15-min buffer                       features, ML model   outlook
```

1. **Wearable node (ESP32 DevKit V1).** Reads the AD8232 ECG (analog, 250 Hz, on its own core), the MAX30102 pulse/SpO₂ sensor and the MLX90614 skin thermometer (both I²C). Each second it filters the signals, detects heartbeats, scores the signal quality and sends a timestamped record over Wi-Fi. If Wi-Fi drops, up to 15 minutes of records are kept and re-sent in order.
2. **Backend (Python, FastAPI, SQLite).** Stores each patient's data, estimates breathing rate from the ECG and pulse signals (there is no breathing sensor), discards poor-quality seconds and builds hourly features.
3. **Machine-learning model (LightGBM).** Uses only what the wearable can measure (heart rate, SpO₂, temperature, breathing rate, age, sex) to estimate the chance of meeting Sepsis-3 within 6, 12 and 24 hours, with an uncertainty band and the main factors behind each estimate.
4. **Dashboard and alerts.** A single-file web dashboard that works offline shows the vitals and outlook in plain language. Alarm-level alerts are sent by SMS through an Android phone used as a local gateway, with escalation to a caregiver.

## Results (from REPORT.md)

The model was trained on one hospital system and tested once on 20,000 patients from a second hospital it never saw (PhysioNet/CinC Challenge 2019 data):

| 6-hour model, wearable inputs only | Value |
|---|---|
| AUROC (95 % CI) | 0.685 (0.672–0.698) |
| NEWS2 score on the same vitals | 0.633 |
| Septic patients warned within 6 h of onset | 42.4 % (random alarms: 18.6 %) |
| False-alarm episodes per patient-day | 0.35 |

It works, but modestly: most alarms are false (hour-level PPV 4–9 %), and a model that also has lab results reaches AUROC 0.772. That gap is the cost of using only wearable data. These numbers come from ICU data. The model's performance on real wearable data has not been measured, and the end-to-end run on the hardware still needs to be verified (see REPORT.md §12 and §14).

## Repository layout

| Folder | Contents |
|---|---|
| `00_Brief/` | Hackathon participant brief |
| `01_Hardware/` | Wiring diagrams, sensor placement, I²C hardware-test sketch |
| `02_ML/` | Data download, feature builder, training and evaluation, trained models, result plots |
| `03_Firmware/SeptiSense_Node/` | ESP32 firmware |
| `04_Backend/` | FastAPI service, inference, breathing-rate estimation, SMS, device simulator |
| `05_Dashboard/` | Web dashboard (`index.html`) |
| `samples/` | Example recordings |

## Quick start

```bash
pip install -r requirements.txt
cd 02_ML && python download_data.py && python train.py data && cd ..
python 04_Backend/app.py                           # open http://localhost:8000
python 04_Backend/simulator.py --deteriorate 120   # optional: simulated device instead of hardware
```

Hardware setup, firmware configuration and SMS gateway setup are in REPORT.md §15.

## Team and my role

Built by a team of four. **My part (Mukunth Parthasarathy):** the wearable sensing hardware and the firmware's signal acquisition and filtering (ECG, PPG/SpO₂, temperature, I²C sensor bus). My teammates built the machine-learning model, the backend and the dashboard.
