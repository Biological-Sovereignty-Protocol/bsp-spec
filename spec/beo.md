# BEO — Biological Entity Object

> Version 0.2 | Ambrósio Institute

---

## Overview

The **Biological Entity Object (BEO)** is the central unit of the BSP. Every piece of data in the BSP ecosystem is anchored to a BEO.

The BEO is **sovereign** — it belongs to the individual, not to any platform. It is stored on decentralized infrastructure (Arweave) and identified by a permanent `.bsp` domain name.

> **The difference between a BEO and a traditional medical record is fundamental:**
> The medical record belongs to the hospital. The BEO belongs to you.
> The hospital is only a contributor — not the owner.

---

## BEO Object Schema

```typescript
BEO {
  // ─── IDENTITY ──────────────────────────────────────────────────
  beo_id:      string     // Universally unique biological identifier (UUID v4)
  domain:      string     // Human-readable address — e.g. "andre.bsp"
  created_at:  ISO8601    // When this entity was first registered
  version:     semver     // BSP version of this record

  // ─── CRYPTOGRAPHY ──────────────────────────────────────────────
  public_key:  string     // Owner's public key (RSA-4096 or Ed25519)
  key_version: number     // Increments on key rotation (starts at 1)

  // ─── DATA ──────────────────────────────────────────────────────
  records:     BioRecord[]  // All biological measurements
  protocols:   Protocol[]   // Active health protocols

  // ─── SOVEREIGNTY ───────────────────────────────────────────────
  sovereignty: SovereigntyMeta  // Ownership, consent, and recovery metadata

  // ─── STATUS ────────────────────────────────────────────────────
  active_recovery: object | null  // Active recovery request metadata, or null
  locked_at:       string | null  // ISO8601 timestamp if BEO is locked, or null
}

SovereigntyMeta {
  guardians:       Guardian[]   // Social recovery network (3 recommended)
  recovery_scheme: string       // "2-of-3" threshold
  seed_phrase_hash: string      // Hashed verification (phrase stored offline by user)
  consent_log:     ConsentEntry[] // All access authorizations — on-chain
}

Guardian {
  contact:     string              // How to reach this guardian (encrypted)
  public_key:  string             // Guardian's public key for the recovery protocol
  role:        string             // 'primary' | 'secondary' | 'tertiary'
  status:      'PENDING' | 'ACTIVE'  // Whether they have accepted the guardian role
  accepted_at: string | null      // ISO8601 timestamp of acceptance, or null if pending
}
```

---

## Creating a BEO

BEO creation is **open to anyone**. No permission from the Ambrósio Institute or any authority is required.

```typescript
// Using the bsp-sdk-typescript
import { BEOClient } from 'bsp-sdk'

const client = new BEOClient()

const beo = await client.create({
  domain: 'andre.bsp',       // Desired .bsp domain
  guardians: [               // Optional at creation — can add later
    { contact: 'maria@example.com', public_key: '...' },
    { contact: 'joao@example.com',  public_key: '...' },
  ]
})

console.log(beo.beo_id)   // "550e8400-e29b-41d4-a716-446655440000"
console.log(beo.domain)   // "andre.bsp"
```

```python
# Using the bsp-sdk-python
from bsp_sdk import BEOClient

client = BEOClient()

beo = client.create(
    domain="andre.bsp",
    guardians=[
        {"contact": "maria@example.com", "public_key": "..."},
    ]
)

print(beo.beo_id)    # "550e8400-e29b-41d4-a716-446655440000"
print(beo.domain)    # "andre.bsp"
```

---

## The .bsp Domain

Every BEO is identified by a human-readable `.bsp` domain — a permanent, sovereign biological address.

| Domain Type | Example | Rules |
|---|---|---|
| Individual | `andre.bsp` | Free, permanent, non-transferable, tied to one BEO |
| Professional | `dr.carlos.bsp` | Paid, permanent, non-transferable, tied to practitioner IEO |
| Institutional | `fleury.bsp` | Paid, annual renewal, transferable, tied to IEO |
| Research | `usp.longevity.bsp` | Paid, tied to Research Partner certification |

→ See [`bsp-domain.md`](bsp-domain.md) for the complete domain system specification.

---

## BEO Properties

### Permanence
Once created, a BEO cannot be deleted — by the owner, by any institution, or by the Ambrósio Institute. The biological identity exists permanently on Arweave.

### Sovereignty
The individual holds the private key. No system — including the Ambrósio Institute — can access the BEO's data without explicit authorization from the key holder.

### Portability
All data within a BEO can be exported in BSP-standard format at any time. No lock-in.

### Immutability
BioRecords cannot be altered once written. Corrections are submitted as new BioRecords that supersede previous records — preserving the complete audit trail.

---

## Access Control

All third-party access to a BEO is governed by the **AccessControl** smart contract on Arweave.

Any system that wants to read from or write to a BEO must:
1. Request authorization from the BEO holder
2. Receive a signed consent token from the holder
3. Submit that token with every transaction

Without a valid token, the smart contract automatically rejects the transaction. The individual is the gatekeeper — not the Ambrósio Institute.

Consent tokens are:
- **Scoped** — limited to specific data categories and intents
- **Time-limited** — expire automatically unless renewed
- **Revocable** — the holder can revoke at any time
- **Auditable** — all grants and revocations are permanently recorded on-chain

→ See [`exchange.md`](exchange.md) for the complete consent token specification.

---

## Social Recovery

If a BEO holder loses their private key, recovery is possible through the guardian network.

**Recovery requires:** 2 of 3 guardians to confirm the holder's identity.

No single guardian can restore access alone. No central server is involved. The recovery protocol is executed on-chain.

**Guardian setup:**
1. At BEO creation (recommended), designate 3 trusted people
2. Each guardian accepts the role and provides a public key
3. Recovery threshold: 2-of-3 signatures required

→ See [`bsp-domain.md`](bsp-domain.md) for recovery protocol details.

---

## BEO vs IEO — Key Distinctions

| Property | BEO (Individual) | IEO (Institution) |
|---|---|---|
| Represents | A living human being | An organization, system, or professional |
| Created by | The individual | Any institution, directly |
| Transferable | Never | Yes — on acquisition or merger |
| Can read BEOs | Own data only | Only with valid consent token |
| Can write to BEOs | Cannot | Yes — with AccessControl authorization |
| Domain format | `firstname.bsp` | `institutionname.bsp` |
| Cost | Free — sovereignty is a right | Paid — annual certification fee |

→ See [`ieo.md`](ieo.md) for the complete IEO specification.

---

## Example BEO (JSON)

```json
{
  "beo_id": "550e8400-e29b-41d4-a716-446655440000",
  "domain": "andre.bsp",
  "created_at": "2026-02-24T14:32:00Z",
  "version": "0.2.0",
  "public_key": "ed25519:4K8Yg2...",
  "key_version": 1,
  "active_recovery": null,
  "locked_at": null,
  "sovereignty": {
    "guardians": [
      {
        "contact": "encrypted:3a7b9c...",
        "public_key": "ed25519:7xM2Pq...",
        "role": "primary",
        "status": "ACTIVE",
        "accepted_at": "2026-02-24T14:35:00Z"
      }
    ],
    "recovery_scheme": "2-of-3",
    "consent_log": []
  }
}
```

→ Full example: [`../examples/beo-example.json`](../examples/beo-example.json)

---

*Ambrósio Institute · ambrosioinstitute.org · biologicalsovereigntyprotocol.com*
