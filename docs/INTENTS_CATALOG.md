# BSP Intents Catalog

> Version 0.2 | Ambrósio Institute

This is the authoritative list of all BSP intents. Every ConsentToken must declare one or more intents from this list. Undeclared or unknown intents are rejected by the AccessControl smart contract.

---

## SUBMIT_RECORD

**Description:** Allows the token holder to submit BioRecords to the target BEO.

**Who can use:** IEOs of type `LABORATORY`, `HOSPITAL`, `PHYSICIAN`, `WEARABLE`.

**Data scope:** Write-only. The submitting IEO cannot read back the records it submitted — it only receives a confirmation receipt (`record_id` + `arweave_tx`).

**Restrictions:**
- Token must also specify authorized `categories`. Records outside those categories are rejected.
- `WEARABLE` IEOs are limited to the `BSP-DV` (Device) category only.
- Submission is final — records can be superseded but not deleted by the IEO.

---

## READ_RECORDS

**Description:** Allows the token holder to read BioRecords from the target BEO.

**Who can use:** IEOs of type `PHYSICIAN`, `PLATFORM`. `INSURER` may only receive aggregate anonymized output — never individual records.

**Data scope:** Read-only. Returns BioRecords filtered by the categories declared in the ConsentToken. Records outside the declared categories are invisible to the requester.

**Restrictions:**
- `WEARABLE` IEOs can never be granted this intent — the smart contract rejects any ConsentToken that combines `IEOType.WEARABLE` with `READ_RECORDS`.
- Tokens must be time-limited (no persistent `READ_RECORDS` tokens for `INSURER` type).
- Pagination is enforced server-side (max 100 records per request, `offset`-based).

---

## EXPORT_DATA

**Description:** Allows the token holder to perform a sovereign data export — a full structured dump of the BEO's records in a portable format.

**Who can use:** The BEO holder themselves (via the Ambrósio OS or self-sovereign tools). No third-party IEO may be granted this intent.

**Data scope:** All BioRecords in the BEO, across all categories. Formats: `JSON`, `CSV`, `FHIR_R4`.

**Restrictions:**
- This intent is exclusively self-sovereign. ConsentTokens granting `EXPORT_DATA` to a third-party IEO are invalid and rejected on-chain.
- Each export is logged on Arweave as an audit event.
- Rate-limited to prevent abuse: maximum 10 exports per 24-hour window.

---

## MANAGE_CONSENT

**Description:** Allows the actor to list, inspect, and revoke ConsentTokens associated with a BEO.

**Who can use:** The BEO holder only. Granted as part of the holder's self-sovereign administrative capabilities.

**Data scope:** ConsentToken metadata (token_id, ieo_id, intents, categories, granted_at, expires_at, revoked status). Does not expose BioRecord content.

**Restrictions:**
- Cannot be delegated to any IEO.
- Revocation via this intent is immediate and irreversible — the on-chain record is final.
- Listing returns all tokens (active, expired, revoked) so the holder has complete auditability.

---

## Additional Intents (IEO-facing)

The following intents are authorized for institutional actors and documented in full in `spec/ieo.md`:

| Intent | Authorized IEO Types | Description |
|---|---|---|
| `REQUEST_CERTIFICATION` | All types | Apply for BSP certification |
| `ANALYZE_VITALITY` | `PLATFORM` | Submit records for AVA analysis |
| `REQUEST_SCORE` | `PLATFORM` | Request SVA score generation |
| `SUBMIT_BIP` | All types | Submit a BSP Improvement Proposal |

---

*Ambrósio Institute · ambrosioinstitute.org · biologicalsovereigntyprotocol.com*
