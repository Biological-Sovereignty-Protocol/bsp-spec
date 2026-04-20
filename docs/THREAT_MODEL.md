# BSP Threat Model

> Version 0.2 | Ambrósio Institute

---

## Adversary Models

### 1. Network Observer (Passive)

An adversary positioned on the network path between a client and the BSP Registry API — an ISP, cloud provider, or compromised router.

**Capabilities:** Observe encrypted traffic metadata (source, destination, timing, payload size). Cannot read payload content over TLS.

**Goal:** Correlate BEO holders with IEOs — infer which institutions a person uses, frequency of submissions.

**Mitigation implemented:**
- All API communication requires TLS 1.2+. Plaintext is rejected.
- Request IDs and audit logs do not leak BEO holder identity to the observer (no PII in HTTP headers).
- ConsentToken IDs are opaque UUIDs — they reveal nothing about the holder without access to the smart contract state.

**Residual risk:** Traffic analysis at scale (e.g. timing correlation between IEO submission spike and a known patient population) is out of scope for the protocol layer.

---

### 2. Compromised Relayer

An adversary who has gained control of a BSP Relayer node — the off-chain component that submits transactions to Aptos on behalf of clients.

**Capabilities:** Can see all transaction payloads passing through the relayer. Can delay, drop, or attempt to replay transactions. Cannot forge signatures.

**Goal:** Replay a past consent grant or data submission to re-use a previously authorized operation. Suppress a revocation to keep a token alive.

**Mitigation implemented:**
- Every transaction carries a unique `nonce` (≥16 bytes, hex-encoded). Nonces are stored in Redis with a 10-minute TTL and rejected on replay.
- Timestamps are validated: requests older than 5 minutes or more than 30 seconds in the future are rejected.
- Revocations are idempotent on-chain and reflected immediately in the AccessControl contract. A delayed revocation succeeds — it may be slightly late but cannot be permanently suppressed.
- All transactions are cryptographically signed by the BEO holder's Ed25519 key. The relayer cannot forge or modify payloads without invalidating the signature.

**Residual risk:** A compromised relayer can delay (not suppress) revocations within the nonce TTL window. Holders should use direct on-chain submission for time-critical revocations.

---

### 3. Malicious IEO

A certified institution acting outside its authorization — attempting to read data it was not consented to, or exceeding its declared intent scope.

**Capabilities:** Holds valid API credentials and one or more legitimate ConsentTokens. Attempts to query data outside token scope, or submits requests after token expiry/revocation.

**Goal:** Access BioRecord categories not covered by the consent token. Persist access after the holder revokes.

**Mitigation implemented:**
- Scope enforcement is double-checked: at the API layer (schema validation rejects queries outside declared categories) and at the Aptos AccessControl contract (on-chain enforcement is authoritative).
- Token expiry and revocation are checked on every request against the live contract state — there is no cached "token still valid" assumption.
- `WEARABLE` IEO type is structurally prevented from receiving `READ_RECORDS` intent at the contract level.
- Audit logs of every exchange operation are written to Arweave permanently. Malicious access attempts leave an irrefutable trail.

**Residual risk:** An IEO that has not yet been suspended can continue operating between the moment of detected misbehavior and the completion of the emergency suspension process (see `spec/governance.md` — Emergency Operations).

---

## Trust Assumptions

| Component | What we trust | Basis |
|---|---|---|
| Aptos validators | Honest majority of validators; BFT consensus is correct | Aptos network design; same assumption as all Aptos-based apps |
| Arweave permanence | Data written to Arweave is immutable and permanently retrievable | Arweave economic model and network history |
| Ed25519 security | Discrete log problem is hard; no feasible forgery without the private key | NIST / academic consensus as of BSP v0.2 |
| Redis nonce store | Redis instance is not compromised at the infrastructure level | Operational security of the BSP Registry deployment |
| TLS certificates | Certificate authorities for BSP API endpoints are trustworthy | Standard web PKI; same assumption as HTTPS everywhere |

---

## Attack Surfaces

| Surface | Attack vector | Mitigation |
|---|---|---|
| `POST /api/relayer/*` | Replay attack with captured signed payload | Nonce + timestamp validation; Redis atomic SETNX |
| `GET /api/exchange/records` | Unauthenticated listing of BioRecords | Schema validation requires `consentToken`, `signature`, `nonce`, and `timestamp` on every request |
| ConsentToken issuance | Token grant with forged holder signature | Ed25519 signature verified against on-chain BEO public key before any token is minted |
| Token revocation | Race condition — two revokes for the same token | Aptos Move resource model makes revocation atomic; second caller receives `E_ALREADY_REVOKED` |
| Governance contract | Single-actor suspension/parameter change | 2-of-3 multi-signature required; no unilateral action possible |
| IEO registration | Self-registration with false credentials | IEO creation is permissionless; certification (which unlocks higher access) requires Institute audit |

---

## Out of Scope

The following threat scenarios are explicitly not addressed by the BSP protocol layer. They may be addressed at the application or operational level by individual deployments.

**Client-side compromise.** If the BEO holder's device is compromised (malware, physical access), an attacker with access to the private key can perform any action the holder can. The protocol cannot distinguish legitimate from coerced requests. Holders are advised to use hardware key storage.

**Physical security.** Attacks on the data center or hardware running BSP Registry API nodes are outside protocol scope. Deployments must implement appropriate physical and cloud security controls.

**Social engineering of the Institute.** A malicious actor convincing Institute keyholders to co-sign a fraudulent governance action is out of scope. This is addressed by Institute operational security and the 2-of-3 threshold that requires collusion.

**Quantum adversaries.** Ed25519 is not post-quantum secure. A sufficiently advanced quantum computer could break holder signatures. Migration to post-quantum key schemes will be addressed in a future BIP when NIST PQC standards are finalized.

**BSP taxonomy correctness.** The protocol enforces that submitted biomarker codes exist in the taxonomy, but does not verify that the biological measurements themselves are accurate or clinically valid. Data quality is the responsibility of the submitting IEO.

---

*Ambrósio Institute · ambrosioinstitute.org · biologicalsovereigntyprotocol.com*
