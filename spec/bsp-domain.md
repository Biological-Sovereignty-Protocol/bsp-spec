# BSP Domain System — .bsp

> Version 0.2 | Ambrósio Institute

---

## Overview

Every BEO and IEO in the BSP ecosystem is identified by a human-readable `.bsp` domain — a permanent, sovereign biological address registered on the Arweave blockchain via the DomainRegistry smart contract.

The `.bsp` namespace is managed by the Ambrósio Institute. Domain assignment and uniqueness are enforced on-chain — not by a central server.

---

## Domain Types

| Type | Format | Example | Cost | Transferable |
|---|---|---|---|---|
| Individual | `firstname.bsp` | `andre.bsp` | Free | Never |
| Individual (privacy) | `initials-year.bsp` | `am1985.bsp` | Free | Never |
| Individual (anonymous) | `[random].bsp` | `b7k3m.bsp` | Free | Never |
| Professional | `dr.name.bsp` | `dr.carlos.bsp` | Paid, permanent | Never |
| Institutional | `institution.bsp` | `fleury.bsp` | Paid, annual | Yes |
| Research | `org.topic.bsp` | `usp.longevity.bsp` | Paid | No |
| Sub-institutional | `name@institution.bsp` | `dr.silva@hcor.bsp` | Under hospital fee | No |

---

## Domain Rules

### Permanence
Individual domains are permanent and non-transferable. Once `andre.bsp` is claimed, it exists forever on Arweave and cannot be transferred to another person or deleted.

### Uniqueness
The DomainRegistry smart contract guarantees that each `.bsp` domain can only exist once. No two entities can hold the same domain.

### Case Insensitivity
All `.bsp` domains are stored and resolved in lowercase. `Andre.bsp` and `andre.bsp` refer to the same domain.

### Reserved Namespaces
The following prefixes are reserved by the Ambrósio Institute:
- `bsp.*` — Protocol infrastructure
- `institute.*` — Institute operations
- `registry.*` — Registry services
- `test.*` — Testing environments

---

## Domain Resolution

```typescript
// Resolve a .bsp domain to a BEO or IEO
import { DomainResolver } from 'bsp-sdk'

const resolver = new DomainResolver()

// Resolve individual domain
const beo = await resolver.resolve('andre.bsp')
// Returns: { type: 'BEO', beo_id: '550e8400-...', public_key: 'ed25519:...' }

// Resolve institutional domain
const ieo = await resolver.resolve('fleury.bsp')
// Returns: { type: 'IEO', ieo_id: '9f1a2b3c-...', ieo_type: 'LABORATORY', ... }

// Check availability
const available = await resolver.isAvailable('newname.bsp')
// Returns: true | false
```

---

## Social Recovery Protocol

If a BEO holder loses their private key, the guardian network enables recovery.

### Setup

```typescript
// At BEO creation or anytime after
const recovery = new RecoveryManager(beoId)

await recovery.addGuardian({
  name: 'Maria',
  contact: 'maria@example.com',   // Encrypted and stored on-chain
  public_key: 'ed25519:...'
})

// Set recovery threshold
await recovery.setThreshold({ required: 2, total: 3 })
```

### Recovery Flow

1. Holder loses device and/or private key
2. Holder contacts 2 of 3 guardians
3. Each guardian submits a signed confirmation on-chain
4. After 2 confirmations: new key pair can be registered
5. Old key is revoked; new key is associated with the domain

```typescript
// Guardian submits confirmation
const recovery = new RecoveryManager(beoId)
await recovery.confirmRecovery({
  guardian_key: guardianPrivateKey,
  new_public_key: recoveryPublicKey,  // Holder's new key
  request_id: recoveryRequestId
})
```

### Security Properties
- No single guardian can restore access alone
- No central server is involved
- The recovery protocol executes on-chain
- The request is time-limited (72 hours default)
- All recovery events are permanently auditable

---

## Seed Phrase Backup

As a last resort (all guardians unavailable), the 24-word seed phrase provides emergency recovery.

The seed phrase is generated at BEO creation and must be written down and stored offline by the holder. The BSP system stores only a cryptographic hash — never the phrase itself.

> **Warning:** If the seed phrase is lost and no guardians are available, access to the BEO cannot be recovered by anyone — including the Ambrósio Institute. Store it safely.

---

*Ambrósio Institute · ambrosioinstitute.org · biologicalsovereigntyprotocol.com*
