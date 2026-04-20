# BSP — Glossary

> Canonical definitions of every term used across the BSP specification, SDK, CLI, MCP server, and website.
> Terms are normative — implementations MUST use these meanings.

---

## Core Entities

### BEO — Biological Entity Object
The sovereign biological identity of a human being. A BEO is the on-chain anchor that ties together a person's public key, `.bsp` domain, recovery configuration, and status. One human → one BEO. Lifelong. Non-transferable. Stored as an Aptos Move resource.

**Fields:** `beoId`, `domain`, `publicKey`, `keyVersion`, `status` (`ACTIVE` | `LOCKED` | `DESTROYED`), `recoveryConfig`, `createdAt`.

**Spec:** `spec/beo.md`

---

### IEO — Institutional Entity Object
The on-chain identity of an institution that interacts with BEOs — laboratories, hospitals, wearable companies, physicians, insurers, research institutions, and platforms. An IEO must be registered to issue or receive records. Every BioRecord is signed by an IEO.

**Types:** `LAB`, `HOSPITAL`, `WEARABLE`, `PHYSICIAN`, `INSURER`, `RESEARCH`, `PLATFORM`.

**Fields:** `ieoId`, `domain`, `type`, `name`, `publicKey`, `certificationLevel`, `status`, `createdAt`.

**Spec:** `spec/ieo.md`

---

### BioRecord
A single biological measurement expressed in the BSP data schema. Every biomarker value — from a blood test to a wearable reading — is a BioRecord. BioRecords are permanent, signed by the issuing IEO, and stored on Arweave with a pointer on-chain.

**Required fields:** `recordId`, `beoId`, `ieoId`, `categoryCode` (e.g. `BSP-LA`), `biomarkerCode`, `value`, `unit`, `referenceRange`, `collectedAt`, `issuedAt`, `signature`, `arweaveTxId`.

**Spec:** `spec/biorecord.md`

---

### ConsentToken
A cryptographically signed on-chain authorization issued by a BEO holder to an IEO. Every data access in BSP is gated by a ConsentToken. The token defines exactly what the IEO may do, for how long, and over which data categories.

**Fields:** `tokenId`, `beoId`, `ieoId`, `intents[]`, `categories[]` (optional), `expiresAt` (optional), `issuedAt`, `revokedAt` (nullable), `status`.

**Lifecycle:** issued → active → (optional) revoked / expired.

---

## Consent Model

### Scope
The set of data categories a ConsentToken covers. Represented as an array of BSP category codes (e.g. `BSP-LA`, `BSP-CV`, `BSP-HM`). A token with scope `[BSP-LA, BSP-HM]` cannot be used to read `BSP-CV` data even if that category exists in the BEO. If `categories` is empty or omitted, the token grants access to **all** categories under the authorized intents.

---

### Intent
The specific action a ConsentToken authorizes. Intents are discrete and additive — a single token may carry multiple intents.

**Enumerated intents:**

| Intent | Meaning |
|---|---|
| `SUBMIT_RECORD` | IEO may submit new BioRecords to the BEO |
| `READ_RECORDS` | IEO may read existing BioRecords |
| `ANALYZE_VITALITY` | IEO may compute derived analyses (never modifies records) |
| `REQUEST_SCORE` | IEO may request longevity / vitality scores |
| `EXPORT_DATA` | IEO may export data in FHIR / JSON / CSV on behalf of holder |
| `SYNC_PROTOCOL` | IEO may synchronize protocol updates (e.g. wearable sync) |

---

### Domain (`.bsp`)
The human-readable BSP identifier. A `.bsp` domain is cryptographically bound to a BEO (for humans) or an IEO (for institutions). Examples: `alice.bsp`, `fleury.bsp`. Resolves via the Aptos contract, not via DNS. Unique. Released when the BEO is destroyed.

**Spec:** `spec/bsp-domain.md`

---

### Guardian
A designated party pre-authorized to initiate BEO recovery if the holder loses access to their private key. Guardians are defined at BEO creation time in the `recoveryConfig` and stored on-chain as public keys. Recovery typically requires an N-of-M multi-signature from guardians (configurable).

---

### Recovery
The process of restoring access to a BEO when the holder has lost their private key. Recovery rotates the BEO's public key (incrementing `keyVersion`) after the configured guardian threshold signs the recovery payload. Recovery is **not** erasure — all existing BioRecords and ConsentTokens continue unchanged.

---

### Relayer
A service that receives signed BSP payloads from clients and submits them as Aptos transactions, paying the gas. The relayer is trustless from the protocol's perspective: it cannot forge signatures, modify payloads, or produce state the chain would not otherwise accept. Any party can run a relayer. The official relayer is the Registry API.

---

## Operational Terms

### Cryptographic Erasure
The protocol-native implementation of LGPD Art. 18 (Brazil) and GDPR Art. 17 (EU) "right to erasure". When a BEO is destroyed, its public key is nullified on-chain. Arweave records remain permanently stored but are no longer associated with a verifiable identity — effectively unreadable. All ConsentTokens are simultaneously revoked and the `.bsp` domain is released.

---

### Nonce
A unique value (minimum 16 characters) included in every signed payload to prevent replay attacks. Combined with a timestamp (max 5 minutes old), the chain rejects any payload that has been seen before.

---

### Key Version
A monotonically increasing counter tracking how many times a BEO's public key has been rotated. Starts at `1`. Increments on every `rotate-key` and on recovery. Old signatures remain valid for past records (verified against historic key versions).

---

### BIP — BSP Improvement Proposal
The formal governance mechanism for changing the protocol. Anyone can draft a BIP. Accepted BIPs become part of the canonical specification. Process defined in `spec/governance.md` and `bip/BIP-0000-template.md`.

---

## Taxonomy

### Category Code
A short identifier for a family of biomarkers (e.g. `BSP-LA` for Lab Advanced, `BSP-CV` for Cardiovascular, `BSP-HM` for Hematology). 25 categories across 4 levels.

### Biomarker Code
A unique identifier for a single measurable biomarker within a category. Format `BSP-{category}-{marker}`. Example: `BSP-HM-HGB` for hemoglobin.

### Taxonomy Levels
- **Level 1 — Core** (9 categories): advanced longevity biomarkers
- **Level 2 — Standard** (9 categories): routine laboratory biomarkers
- **Level 3 — Extended** (6 categories): specialized biomarkers
- **Level 4 — Device** (1 category): continuous wearable data

**Spec:** `spec/taxonomy/`

---

## Related

- **Architecture** — `docs/ARCHITECTURE.md`
- **Error Codes** — `docs/ERROR_CODES.md`
- **Implementation Guide** — `docs/implementation-guide.md`
- **Website Glossary** — `https://biologicalsovereigntyprotocol.com/glossary`
