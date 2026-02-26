# BSP Taxonomy — Level 4: Device Biomarkers

> Version 0.2 | Ambrósio Institute

Continuous biometric data from wearable devices and monitoring equipment. Submitted as daily consolidated BioRecords — statistical summaries of continuous measurements.

**Note:** Wearable IEOs submit daily consolidated records per biomarker — not second-by-second raw data. This prevents infrastructure overload while preserving clinical utility.

---

## Category 25 — Device & Continuous Monitoring (BSP-DV)

### Cardiac & Cardiovascular

| BSP Code | Biomarker | Unit | Consolidation |
|---|---|---|---|
| `BSP-DV-001` | Heart Rate Variability (RMSSD) | ms | Daily mean + SD |
| `BSP-DV-002` | Resting Heart Rate | bpm | Daily mean + min |
| `BSP-DV-003` | HRV (SDNN) | ms | Daily mean |
| `BSP-DV-004` | ECG — Rhythm Classification | categorical | Continuous file (compressed) |
| `BSP-DV-005` | Blood Pressure (continuous) | mmHg | Daily mean + peak |

### Respiratory & Oxygenation

| BSP Code | Biomarker | Unit | Consolidation |
|---|---|---|---|
| `BSP-DV-006` | SpO2 (blood oxygen) | % | Daily mean + nadir |
| `BSP-DV-007` | Respiratory Rate | breaths/min | Daily mean + nocturnal |

### Metabolic

| BSP Code | Biomarker | Unit | Consolidation |
|---|---|---|---|
| `BSP-DV-008` | Continuous Glucose (CGM) | mg/dL | Daily mean, SD, TIR, TAR, TBR |
| `BSP-DV-009` | Skin Temperature | °C | Daily mean + nocturnal delta |
| `BSP-DV-010` | Core Body Temperature | °C | Daily mean |

### Activity & Movement

| BSP Code | Biomarker | Unit | Consolidation |
|---|---|---|---|
| `BSP-DV-011` | Daily Steps | steps | Daily total |
| `BSP-DV-012` | Active Energy Expenditure | kcal | Daily total |
| `BSP-DV-013` | VO2 Max (estimated) | mL/kg/min | Weekly (stable metric) |
| `BSP-DV-014` | Activity Intensity Distribution | minutes | Daily (sedentary/light/moderate/vigorous) |

### Sleep Architecture

| BSP Code | Biomarker | Unit | Consolidation |
|---|---|---|---|
| `BSP-DV-015` | Total Sleep Duration | minutes | Nightly |
| `BSP-DV-016` | REM Sleep | minutes | Nightly |
| `BSP-DV-017` | Deep Sleep (N3) | minutes | Nightly |
| `BSP-DV-018` | Sleep Efficiency | % | Nightly |
| `BSP-DV-019` | Sleep Latency | minutes | Nightly |
| `BSP-DV-020` | Sleep Regularity Index | score | Weekly |

### Recovery & Readiness

| BSP Code | Biomarker | Unit | Consolidation |
|---|---|---|---|
| `BSP-DV-021` | Recovery Score | 0–100 | Daily |
| `BSP-DV-022` | Readiness Score | 0–100 | Daily |
| `BSP-DV-023` | Stress Level Index | 0–100 | Daily |

---

## Device SDK Requirement

Wearable IEOs must use the official BSP Device SDK to ensure:
- Correct daily consolidation format
- Proper biomarker coding
- Cryptographic signing of daily batches
- Automated consent verification before submission

```python
# BSP Device SDK — daily consolidation example
from bsp_sdk.device import DeviceRecordBuilder

record = (DeviceRecordBuilder()
    .beo_id("550e8400-...")
    .biomarker("BSP-DV-001")           # HRV RMSSD
    .date("2026-02-24")
    .consolidated({
        "mean": 47.3,
        "sd": 12.1,
        "samples": 1440,
        "method": "overnight-5min-segments"
    })
    .unit("ms")
    .device({ "model": "Oura Ring Gen4", "firmware": "2.8.1" })
    .build())
```

---

*Ambrósio Institute · ambrosioinstitute.org · biologicalsovereigntyprotocol.com*
