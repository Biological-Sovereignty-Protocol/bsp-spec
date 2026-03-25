# IEO — Institutional Entity Object

> Version 0.2 | Ambrósio Institute

---

## Overview

The **Institutional Entity Object (IEO)** represents any organization, system, or professional that interacts with biological data on behalf of, or with the consent of, a BEO holder.

Every institution in the BSP ecosystem — from a national hospital chain to a wearable device manufacturer to an independent physician — is represented as an IEO.

> **Creating an IEO is open to any institution.**
> BSP Certification is voluntary — but unlocks meaningful benefits within the ecosystem.

---

## IEO Object Schema

```typescript
IEO {
  // ─── IDENTITY ──────────────────────────────────────────────────
  ieo_id:       string     // Universally unique institutional identifier
  domain:       string     // .bsp address — e.g. "fleury.bsp"
  display_name: string     // Full legal name of the institution
  ieo_type:     IEOType    // LABORATORY | HOSPITAL | WEARABLE | PHYSICIAN |
                           // INSURER | RESEARCH | PLATFORM
  country:      ISO3166    // Country of primary operation
  jurisdiction: string     // Regulatory jurisdiction
  legal_id:     string     // CNPJ (BR) / EIN (US) / VAT (EU) etc.
  public_key:   string     // Institutional public key for signed submissions
  created_at:   ISO8601    // IEO registration timestamp
  version:      semver     // BSP version at time of creation

  // ─── CERTIFICATION ─────────────────────────────────────────────
  certification: {
    level:        CertLevel    // BASIC | ADVANCED | FULL | RESEARCH
    granted_at:   ISO8601
    expires_at:   ISO8601      // Annual renewal
    categories:   string[]     // Authorized BSP categories (e.g. ["BSP-LA", "BSP-HM"])
    intents:      BSPIntent[]  // Authorized exchange intents
    restrictions: string[]     // Explicit prohibitions if any
    certified_by: string       // Institute auditor reference
    audit_ref:    string       // Audit transaction ID on Arweave
  }

  // ─── OPERATIONAL RECORD ────────────────────────────────────────
  operations: {
    biorecords_submitted: number   // Total BioRecords submitted to date
    last_submission:      ISO8601
    compliance_rate:      float    // Schema compliance rate (0.0–1.0)
    active_consents:      number   // Current open consent tokens
    complaints:           number
    audits:               Audit[]
  }

  // ─── CONTACTS ──────────────────────────────────────────────────
  contacts: {
    technical_lead:   ContactRef
    compliance_lead:  ContactRef
    api_endpoint:     string     // Primary BSP API endpoint URL
    webhook_url:      string     // Notification webhook (optional)
  }

  // ─── STATUS ────────────────────────────────────────────────────
  status:             IEOStatus  // ACTIVE | SUSPENDED | REVOKED | PENDING
  suspension_reason:  string | null
  revocation_reason:  string | null
}
```

---

## IEO Types

### LAB — Clinical & Diagnostic Laboratory (`IEOType.LABORATORY`)

Clinical and diagnostic laboratories — the primary source of BioRecords in the BSP ecosystem.

| Property | Rule |
|---|---|
| Default authorized levels | L2 Standard |
| Advanced authorized levels | L1 Core, L3 Extended on certification |
| Can READ BEOs | Never — submission is write-only |
| Domain format | `institutionname.bsp` (e.g. `fleury.bsp`) |
| Renewal | Annual compliance audit |

### HSP — Hospital & Health System (`IEOType.HOSPITAL`)

Hospitals and health systems. Authorized across multiple taxonomy levels simultaneously. May credential physicians as sub-IEOs.

| Property | Rule |
|---|---|
| Default authorized levels | L1 Core + L2 Standard |
| Advanced authorized levels | L3 Extended, L4 Device on certification |
| Can READ BEOs | With active consent token, time-limited |
| Physician sub-IEOs | `dr.silva@hcor.bsp` format |

### WRB — Wearable & Device (`IEOType.WEARABLE`)

Hardware and software companies producing wearable devices and continuous monitoring systems.

| Property | Rule |
|---|---|
| Authorized levels | L4 Device (BSP-DV) **exclusively** |
| Submission frequency | Daily consolidated records per user |
| Can READ BEOs | **Never** — permanent restriction, cannot be overridden by user consent |
| SDK requirement | Must use official BSP Device SDK |

> **Critical:** Wearable IEOs are the only IEO type that may never be granted READ_RECORDS access under any circumstances.

### PHY — Physician & Practitioner (`IEOType.PHYSICIAN`)

Licensed physicians and health practitioners. Unique in that READ_RECORDS is their primary function.

| Property | Rule |
|---|---|
| Primary function | READ_RECORDS with consent token |
| Submission rights | Clinical assessments only (BSP-CL) |
| Consent model | Time-limited and scope-limited tokens |
| Credential verification | Medical license number required at registration |

### INS — Health Insurer (`IEOType.INSURER`)

Health insurance companies. Operate under the most restrictive access model.

| Property | Rule |
|---|---|
| Read access | Aggregate anonymized data only |
| Individual access | Only with explicit ongoing consent, never automatic |
| **PROHIBITED** | Cannot use BSP data for underwriting, coverage denial, or premium calculation based on individual biological risk |

> This restriction is encoded in the IEO contract and enforced at the smart contract level.

### RES — Research Institution (`IEOType.RESEARCH`)

Universities, research centers, and clinical trial organizations.

| Property | Rule |
|---|---|
| Data access | Anonymized aggregate with explicit opt-in |
| Individual identification | Structurally impossible — BEO references stripped |
| Commercial restriction | Cannot commercialize raw BSP data access |

### PLT — Digital Health Platform & AI System (`IEOType.PLATFORM`)

Digital health platforms, telemedicine services, and AI systems. This is the category for AVA/SVA implementations.

| Property | Rule |
|---|---|
| Primary function | ANALYZE_VITALITY and REQUEST_SCORE intents |
| Read access | With persistent user consent — refreshable |
| Write access | Cannot — platforms interpret, not measure |

---

## Certification Levels

| Level | Requirements | What it unlocks |
|---|---|---|
| **BSP-Compliant Basic** | L2 Standard biomarkers only | Diretory listing, basic badge |
| **BSP-Compliant Advanced** | L1 Core + L2 Standard | AVA data feed, advanced badge |
| **BSP-Compliant Full-Spectrum** | All levels | Full ecosystem integration |
| **BSP Research Partner** | Research IEO with BIP contribution | Anonymized research data access |

**Certification is voluntary.** Any institution can participate in the BSP ecosystem without certification — but certified institutions:
- Appear in the official directory
- Display a verifiable on-chain badge
- Have their data feed into AVA analysis
- Are recommended by the Ambrósio OS

Non-certified institutions can operate but the Ambrósio app displays a "unverified source" notice.

---

## Exchange Intents

Each IEO type is authorized for specific exchange intents:

| Intent | Description | Authorized IEO Types |
|---|---|---|
| `SUBMIT_RECORD` | Submit biological measurements to a BEO | LAB, HSP, WRB, PHY |
| `READ_RECORDS` | Read BioRecords from a BEO | PHY, PLT, INS (aggregate only) |
| `REQUEST_CERTIFICATION` | Apply for BSP certification | All types |
| `ANALYZE_VITALITY` | Submit BioRecords for AVA analysis | PLT |
| `REQUEST_SCORE` | Request SVA score generation | PLT |
| `SUBMIT_BIP` | Submit a BSP Improvement Proposal | All types |

---

*Ambrósio Institute · ambrosioinstitute.org · biologicalsovereigntyprotocol.com*
