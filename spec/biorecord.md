# BioRecord — Biological Measurement Object

> Version 0.2 | Ambrósio Institute

---

## Overview

Every biological measurement in the BSP ecosystem is represented as a **BioRecord**.

BioRecords are the atomic units of biological data. Every blood test result, wearable reading, genomic marker, or clinical assessment that enters the BSP ecosystem is structured as a BioRecord.

---

## BioRecord Schema

```typescript
BioRecord {
  // ─── IDENTITY ──────────────────────────────────────────────────
  record_id:    string      // Unique identifier for this record (UUID v4)
  beo_id:       string      // The biological entity this record belongs to
  version:      semver      // BSP version of this record

  // ─── TIMING ────────────────────────────────────────────────────
  timestamp:    ISO8601     // When this measurement was taken (not submitted)
  submitted_at: ISO8601     // When this record was written to Arweave

  // ─── SOURCE ────────────────────────────────────────────────────
  source:       SourceMeta  // Who produced this measurement

  // ─── TAXONOMY ──────────────────────────────────────────────────
  category:     string      // BSP taxonomy category code (e.g. "BSP-LA")
  biomarker:    string      // Standardized BSP biomarker code (e.g. "BSP-LA-004")
  level:        BioLevel    // CORE | STANDARD | EXTENDED | DEVICE

  // ─── MEASUREMENT ───────────────────────────────────────────────
  value:        number | string  // The measured value
  unit:         string           // Standardized unit (SI or BSP-defined)
  ref_range:    RangeObject      // Optimal and functional reference ranges

  // ─── QUALITY ───────────────────────────────────────────────────
  confidence:   float       // 0.0 to 1.0 measurement confidence
  status:       RecordStatus // ACTIVE | SUPERSEDED | PENDING

  // ─── CORRECTIONS ───────────────────────────────────────────────
  supersedes:   string | null  // record_id of corrected record, if applicable

  // ─── CRYPTOGRAPHY ──────────────────────────────────────────────
  signature:    string      // Cryptographic signature from submitting entity
}

SourceMeta {
  ieo_id:      string      // The submitting institution's IEO identifier
  ieo_domain:  string      // e.g. "fleury.bsp"
  method:      string      // Measurement method (e.g. "HPLC", "immunoassay")
  equipment:   string      // Equipment identifier (optional)
  operator:    string      // Lab operator reference (optional, anonymized)
}

RangeObject {
  optimal_low:    number   // Lower bound of optimal range
  optimal_high:   number   // Upper bound of optimal range
  functional_low: number   // Lower bound of functional range (no immediate risk)
  functional_high: number  // Upper bound of functional range
  critical_low:   number   // Below this = immediate clinical attention
  critical_high:  number   // Above this = immediate clinical attention
  unit:           string   // Same as parent record unit
  population:     string   // Reference population for this range
}
```

---

## BioLevel Values

| Level | Code | Description |
|---|---|---|
| Core | `CORE` | Advanced longevity and biological aging biomarkers (L1) |
| Standard | `STANDARD` | Routine laboratory biomarkers performed worldwide (L2) |
| Extended | `EXTENDED` | Specialized clinical and research biomarkers (L3) |
| Device | `DEVICE` | Continuous biometric data from wearable devices (L4) |

---

## RecordStatus Values

| Status | Description |
|---|---|
| `ACTIVE` | This is the current, valid record |
| `SUPERSEDED` | This record has been corrected by a newer record |
| `PENDING` | Submitted but awaiting confirmation (e.g. device sync) |

---

## Immutability and Corrections

BioRecords are **immutable once written**. They cannot be altered or deleted after submission to Arweave.

When a correction is necessary (e.g. a lab transcription error), the corrected value is submitted as a **new BioRecord** that supersedes the original:

```typescript
// Corrected record
{
  record_id:    "new-uuid-...",
  beo_id:       "550e8400-...",
  status:       "ACTIVE",
  supersedes:   "original-uuid-...",   // References the incorrect record
  biomarker:    "BSP-GL-001",
  value:        94,
  // ...
}

// Original record (automatically updated on-chain)
{
  record_id:    "original-uuid-...",
  status:       "SUPERSEDED",          // Status updated to SUPERSEDED
  // ... original data preserved
}
```

The complete audit trail — including the original incorrect value — is preserved permanently on Arweave.

---

## Submitting a BioRecord

Any system can submit a BioRecord to a BEO — subject to the holder's consent via AccessControl.

```typescript
// TypeScript SDK
import { BioRecordBuilder, ExchangeClient } from 'bsp-sdk'

const record = new BioRecordBuilder()
  .beoId('550e8400-e29b-41d4-a716-446655440000')
  .biomarker('BSP-GL-001')           // Fasting glucose
  .value(94)
  .unit('mg/dL')
  .timestamp('2026-02-24T08:30:00Z')
  .refRange({
    optimal_low: 70,
    optimal_high: 90,
    functional_low: 60,
    functional_high: 100,
    critical_low: 40,
    critical_high: 180,
    unit: 'mg/dL',
    population: 'adult-general'
  })
  .confidence(0.99)
  .build()

const client = new ExchangeClient({ ieoId: 'my-lab.bsp' })
const result = await client.submit(record)
```

```python
# Python SDK
from bsp_sdk import BioRecordBuilder, ExchangeClient

record = (BioRecordBuilder()
    .beo_id("550e8400-e29b-41d4-a716-446655440000")
    .biomarker("BSP-GL-001")
    .value(94)
    .unit("mg/dL")
    .timestamp("2026-02-24T08:30:00Z")
    .confidence(0.99)
    .build())

client = ExchangeClient(ieo_id="my-lab.bsp")
result = client.submit(record)
```

---

## Biomarker Codes

Every biomarker in the BSP taxonomy has a standardized code in the format:

```
BSP-[CATEGORY]-[NUMBER]
```

Examples:
- `BSP-LA-004` — NAD+ (Longevity & Aging category, marker #004)
- `BSP-GL-001` — Fasting Glucose (Glycemia & Metabolic category, marker #001)
- `BSP-DV-001` — Heart Rate Variability (Device category, marker #001)

→ See [`taxonomy/`](taxonomy/) for the complete biomarker reference.

---

## Example BioRecord (JSON)

```json
{
  "record_id": "7b3f9a12-4c8e-4d21-b6f0-1a9e8c7d5b2a",
  "beo_id": "550e8400-e29b-41d4-a716-446655440000",
  "version": "0.2.0",
  "timestamp": "2026-02-24T08:30:00Z",
  "submitted_at": "2026-02-24T09:15:00Z",
  "source": {
    "ieo_id": "9f1a2b3c-4d5e-6f7a-8b9c-0d1e2f3a4b5c",
    "ieo_domain": "fleury.bsp",
    "method": "colorimetric-enzymatic",
    "equipment": "Roche Cobas 8000"
  },
  "category": "BSP-GL",
  "biomarker": "BSP-GL-001",
  "level": "STANDARD",
  "value": 94,
  "unit": "mg/dL",
  "ref_range": {
    "optimal_low": 70,
    "optimal_high": 90,
    "functional_low": 60,
    "functional_high": 100,
    "critical_low": 40,
    "critical_high": 180,
    "unit": "mg/dL",
    "population": "adult-general"
  },
  "confidence": 0.99,
  "status": "ACTIVE",
  "supersedes": null,
  "signature": "ed25519:9xK2Lm..."
}
```

→ Full example: [`../examples/biorecord-example.json`](../examples/biorecord-example.json)

---

*Ambrósio Institute · ambrosioinstitute.org · biologicalsovereigntyprotocol.com*
