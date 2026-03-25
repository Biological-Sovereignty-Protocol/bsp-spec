# Changelog — BSP Specification

All notable changes to the BSP protocol specification are recorded here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

Specification versions follow [Semantic Versioning](https://semver.org/):
- MAJOR — breaking protocol changes (on-chain incompatibility)
- MINOR — new entities, fields, or intents (backward compatible)
- PATCH — clarifications, typos, editorial fixes

---

## [1.0.0] — 2026-03-24

Initial release of the BSP protocol specification.

### Added

**Entity specifications**
- BEO spec — Biological Entity Object: fields, key generation (Ed25519), domain system,
  lifecycle states (ACTIVE, LOCKED, SUSPENDED), Social Recovery model (threshold signatures)
- IEO spec — Institutional Entity Object: entity types (LAB, HOSPITAL, WEARABLE, PHYSICIAN,
  INSURER, RESEARCH, PLATFORM), certification levels (UNCERTIFIED, BASIC, STANDARD, TRUSTED),
  voluntary certification process via Ambrósio Institute
- BioRecord spec — record schema, required vs. optional fields, versioning via `supersedes`,
  confidence scoring (0.0–1.0), source attestation model
- ConsentToken spec — token structure, scope model, intent enumeration, expiry semantics,
  revocation finality, audit log requirements

**Exchange Protocol**
- Exchange Protocol v1 — submit, read, and sovereign export operations
- Double-authentication model: ConsentToken + IEO signature requirement
- Batch submission limits and error handling
- Pagination spec for `readRecords`
- `EXPORT_DATA` sovereign right — any BSP-compliant system must support this;
  blocking is a protocol violation
- Rate limiting headers and retry semantics

**Governance**
- BIP (BSP Improvement Proposal) process — proposal stages: DRAFT, REVIEW, ACCEPTED, FINAL, WITHDRAWN
- Governance roles: Core Contributors, Certified IEOs, Community
- Voting thresholds by change category (breaking vs. non-breaking)
- BIP-0001: BIP process definition (self-referential bootstrap BIP)

**Biomarker taxonomy**
- Taxonomy levels — Core (universal, well-validated), Standard (broadly used),
  Extended (specialized or emerging), Device (wearable-sourced)
- Core categories: BSP-HM (hematology), BSP-LA (laboratory), BSP-CV (cardiovascular)
- Standard categories: BSP-GN (genomics), BSP-NT (nutrition), BSP-MB (microbiome),
  BSP-MS (musculoskeletal), BSP-NR (neurology), BSP-HN (hormones/endocrinology)
- Extended categories: BSP-EP (epigenetics), BSP-PR (proteomics), BSP-MT (metabolomics)
- Device categories: BSP-WT (wearable/continuous), BSP-SL (sleep), BSP-AC (activity)
- Code format: `BSP-XX-NNN` (category 2 chars + sequence 3 digits)
- Initial taxonomy: 214 biomarker codes across all categories

**Domain system**
- `.bsp` namespace specification — registration rules, uniqueness constraints
- Domain resolution protocol — DomainRegistry → BEORegistry / IEORegistry lookup chain
- Reserved domains list and reservation policy

**Cryptography**
- Key generation spec: Ed25519 for all BEO and IEO keys
- Signature format for IEO-signed requests
- Social Recovery: Shamir Secret Sharing with configurable threshold (k-of-n)
- Encryption at rest: AES-256-GCM for BioRecord data fields

---

[1.0.0]: https://github.com/biological-sovereignty-protocol/bsp-spec/releases/tag/v1.0.0
