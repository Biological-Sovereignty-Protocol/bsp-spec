# BSP Exchange Protocol

> Version 0.2 | Ambrósio Institute

---

## Overview

The BSP Exchange Protocol defines how systems request and respond with biological data — the communication layer of the protocol.

All exchange operations are subject to the **AccessControl** smart contract. The BEO holder's consent is required for every data transaction involving their biological data.

---

## Core Principle

> No system — laboratory, platform, AI engine, or the Ambrósio Institute itself — can access a BEO without explicit authorization from the holder, enforced by the AccessControl smart contract on Arweave.

The consent requirement is **mathematical**, not institutional. The smart contract rejects unauthorized transactions automatically — no human review, no backend server, no institutional trust required.

---

## Consent Token

Before any exchange operation, the executing system must hold a valid **ConsentToken** signed by the BEO holder.

```typescript
ConsentToken {
  token_id:    string       // Unique token identifier
  beo_id:      string       // The BEO this token grants access to
  ieo_id:      string       // The institution this token is granted to
  intents:     BSPIntent[]  // Authorized operations (e.g. ["SUBMIT_BIORECORD"])
  categories:  string[]     // Authorized BSP categories (e.g. ["BSP-LA", "BSP-GL"])
  granted_at:  ISO8601      // When the holder granted this token
  expires_at:  ISO8601      // When this token expires (null = persistent)
  revoked:     boolean      // Whether this token has been revoked
  signature:   string       // Holder's cryptographic signature
  arweave_tx:  string       // Arweave transaction ID — the on-chain record
}
```

### Token Properties

| Property | Description |
|---|---|
| **Scoped** | Limited to specific intents and data categories |
| **Time-limited** | Expire at `expires_at` — automatically invalid after |
| **Revocable** | Holder can revoke at any time, recorded on-chain |
| **Auditable** | All grants, uses, and revocations permanently on Arweave |
| **Non-transferable** | Tokens are bound to a specific IEO — cannot be shared |

---

## Exchange Operations

### SUBMIT_BIORECORD

Submit a biological measurement to a BEO.

**Required consent:** `SUBMIT_BIORECORD` intent + authorized category matching the biomarker

```typescript
// Request
SubmitBioRecordRequest {
  token:    ConsentToken    // Valid, non-expired, non-revoked consent token
  record:   BioRecord       // The BioRecord to submit
}

// Response
SubmitBioRecordResponse {
  success:    boolean
  record_id:  string        // Assigned record_id
  arweave_tx: string        // Arweave transaction ID — permanent record
  timestamp:  ISO8601
}
```

**Validation rules:**
1. ConsentToken must be valid, not expired, not revoked
2. Token's `intents` must include `SUBMIT_BIORECORD`
3. Token's `categories` must include the record's `category`
4. Record's `beo_id` must match token's `beo_id`
5. Record must pass schema validation (all required fields present)
6. Biomarker code must exist in the BSP taxonomy
7. Value must be within plausible physiological range

---

### READ_RECORDS

Read BioRecords from a BEO.

**Required consent:** `READ_RECORDS` intent + authorized categories

```typescript
// Request
ReadRecordsRequest {
  token:      ConsentToken   // Valid consent token
  beo_id:     string
  filters: {
    categories:  string[]    // Filter by BSP category (optional)
    biomarkers:  string[]    // Filter by specific biomarker codes (optional)
    from:        ISO8601     // Records after this timestamp (optional)
    to:          ISO8601     // Records before this timestamp (optional)
    status:      RecordStatus // ACTIVE | SUPERSEDED | PENDING (default: ACTIVE)
    limit:       number      // Max records to return (default: 100)
    offset:      number      // Pagination offset
  }
}

// Response
ReadRecordsResponse {
  beo_id:   string
  records:  BioRecord[]
  total:    number
  has_more: boolean
}
```

---

### REQUEST_CONSENT

An institution requests a consent token from a BEO holder.

This operation is initiated by the institution but must be completed by the BEO holder — the token does not exist until the holder signs.

```typescript
// Consent Request (sent to holder)
ConsentRequest {
  request_id:  string
  ieo_id:      string
  ieo_domain:  string       // e.g. "fleury.bsp"
  ieo_name:    string       // e.g. "Fleury Laboratórios"
  intents:     BSPIntent[]  // What access is being requested
  categories:  string[]     // Which data categories
  expires_in:  number       // Requested duration in seconds (null = persistent)
  reason:      string       // Human-readable explanation
}

// Holder response — grant
ConsentGrant {
  request_id: string
  token:      ConsentToken  // Signed by holder's private key
}

// Holder response — deny
ConsentDenial {
  request_id: string
  reason:     string        // Optional
}
```

---

### REVOKE_CONSENT

The BEO holder revokes a previously granted consent token.

```typescript
// Revocation (signed by holder)
ConsentRevocation {
  token_id:   string        // The token being revoked
  beo_id:     string
  reason:     string        // Optional
  revoked_at: ISO8601
  signature:  string        // Holder's signature
  arweave_tx: string        // On-chain record
}
```

After revocation, any further use of the token is rejected by the AccessControl smart contract.

---

## Error Codes

| Code | Description |
|---|---|
| `BSP-E-001` | Invalid or missing consent token |
| `BSP-E-002` | Token expired |
| `BSP-E-003` | Token revoked |
| `BSP-E-004` | Intent not authorized by token |
| `BSP-E-005` | Category not authorized by token |
| `BSP-E-006` | BEO not found |
| `BSP-E-007` | IEO not found or suspended |
| `BSP-E-008` | Schema validation failure |
| `BSP-E-009` | Biomarker code not found in taxonomy |
| `BSP-E-010` | Value outside plausible physiological range |
| `BSP-E-011` | Arweave write failure — retry |
| `BSP-E-012` | Signature verification failure |

---

## SDK Usage

```typescript
// TypeScript — Submit a BioRecord
import { ExchangeClient, ConsentManager } from '@bsp/sdk'

// Check if we have a valid consent token
const consentManager = new ConsentManager({ ieoId: 'my-lab.bsp' })
const token = await consentManager.getToken(beoId)

if (!token || token.isExpired()) {
  // Request new consent from user
  const request = await consentManager.requestConsent(beoId, {
    intents: ['SUBMIT_BIORECORD'],
    categories: ['BSP-GL', 'BSP-HM'],
    expiresIn: 365 * 24 * 60 * 60  // 1 year
  })
  // User must approve — returns token when approved
  token = await consentManager.waitForApproval(request.request_id)
}

// Submit with valid token
const client = new ExchangeClient()
const result = await client.submit(record, token)
```

---

*Ambrósio Institute · ambrosioinstitute.org · biologicalsovereigntyprotocol.com*
