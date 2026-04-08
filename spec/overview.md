# BSP Architecture — The Three Layers

> Version 0.2 | Ambrósio Institute

---

## Overview

The Biological Sovereignty Protocol is organized into three distinct layers. Each layer has a clearly defined responsibility and is designed to be independent — changes to one layer do not break implementations of another.

```
┌─────────────────────────────────────────────────────────┐
│                  INTELLIGENCE LAYER                      │
│         AVA · SVA · Third-party algorithms              │
│    (above the protocol — not defined by BSP)            │
└─────────────────────────────────────────────────────────┘
                           │
┌─────────────────────────────────────────────────────────┐
│              LAYER 3 — BSP-Exchange                      │
│           Communication Protocol                        │
│   How systems request and respond with biological data  │
└─────────────────────────────────────────────────────────┘
                           │
┌─────────────────────────────────────────────────────────┐
│              LAYER 2 — BSP-Data                          │
│           Biological Data Schema                        │
│   Structure of all biological measurements (BioRecord)  │
└─────────────────────────────────────────────────────────┘
                           │
┌─────────────────────────────────────────────────────────┐
│              LAYER 1 — BSP-Identity                      │
│           Biological Identity                           │
│   The sovereign identity object — BEO                   │
│   Stored on Arweave — permanent, decentralized          │
└─────────────────────────────────────────────────────────┘
```

---

## Layer 1 — BSP-Identity

**What it defines:** Who holds the data.

Every individual and every institution in the BSP ecosystem has a permanent, decentralized identity:
- **BEO** — Biological Entity Object (individual)
- **IEO** — Institutional Entity Object (laboratory, hospital, platform, etc.)

The BEO is the center of gravity of the entire protocol. Every BioRecord, every consent, every interaction is anchored to a BEO.

BEOs and IEOs are stored on the **Arweave** blockchain — permanent and decentralized. No company, government, or the Ambrósio Institute itself can delete or alter a registered identity.

→ See [`beo.md`](beo.md) and [`ieo.md`](ieo.md) for complete specifications.

---

## Layer 2 — BSP-Data

**What it defines:** What the data contains.

Every biological measurement — a blood test result, a genomic marker, a wearable reading, a clinical assessment — is represented as a **BioRecord**.

BioRecords are the atomic units of biological data in the BSP ecosystem. They are:
- **Immutable** — once written, they cannot be altered
- **Anchored** — every BioRecord belongs to a specific BEO
- **Classified** — every BioRecord carries a BSP taxonomy code
- **Signed** — every BioRecord carries a cryptographic signature from the submitting entity

→ See [`biorecord.md`](biorecord.md) for the complete BioRecord specification.
→ See [`taxonomy/`](taxonomy/) for the full biomarker taxonomy.

---

## Layer 3 — BSP-Exchange

**What it defines:** How data moves.

The BSP Exchange Protocol defines the format of requests and responses between systems:
- How any system submits data to a BEO
- How a platform requests read access
- How an AI engine queries a biological history
- How consent tokens are structured and verified

All exchange operations are subject to the **AccessControl** smart contract — the BEO holder's consent is required for every data transaction.

→ See [`exchange.md`](exchange.md) for the complete Exchange Protocol specification.

---

## Decentralized Infrastructure

BSP records are stored on **Arweave** — a permanent, decentralized storage protocol designed to preserve data for 200+ years.

AO processes managing BEO identities, domain registrations, and access permissions run on Arweave's hyper-parallel compute platform. This ensures:
- No single point of failure
- No company (including Ambrósio Institute) can alter the rules unilaterally
- Data written to BSP infrastructure exists permanently

The five smart contracts in the BSP infrastructure:

| Contract | Purpose |
|---|---|
| **BEORegistry** | Creates and manages biological identities — open to anyone |
| **IEORegistry** | Manages institutional identities and certification status |
| **DomainRegistry** | Controls the `.bsp` namespace — guarantees uniqueness |
| **AccessControl** | Manages consent tokens — the true gatekeeper of the protocol |
| **Governance** | Multi-signature authorization for critical protocol changes |

→ See [`bsp-domain.md`](bsp-domain.md) for the `.bsp` domain system.
→ See [`governance.md`](governance.md) for the governance model.

---

## The Intelligence Layer (Above BSP)

The intelligence layer is **not part of the BSP specification**.

BSP defines how data is structured and transported — not what conclusions to draw from it. Intelligence layers such as:
- Ambrósio Vitality Algorithm (AVA)
- Ambrósio Vitality Score (SVA)
- Any third-party analytics engine

...operate above the protocol, consuming standardized BSP data to produce insights. Any system in the world can implement the BSP. Only Ambrósio holds the AVA.

---

*Ambrósio Institute · ambrosioinstitute.org · biologicalsovereigntyprotocol.com*
