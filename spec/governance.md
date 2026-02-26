# BSP Governance — Protocol Changes and BIPs

> Version 0.2 | Ambrósio Institute

---

## Overview

The BSP specification evolves through a public improvement process — **BSP Improvement Proposals (BIPs)**.

Any individual, company, or institution can propose changes to the protocol. The Ambrósio Institute reviews all proposals and coordinates community discussion. Critical protocol changes require multi-signature authorization.

---

## Governance Principles

1. **Openness** — Anyone can propose a BIP. No institutional affiliation required.
2. **Transparency** — All proposals, discussions, and decisions are public.
3. **Conservative change** — Protocol changes have a high bar. Stability is a feature.
4. **Backward compatibility** — Accepted changes must not break existing implementations, unless the break is clearly necessary and the migration path is defined.
5. **Institute guardianship** — The Ambrósio Institute is the guardian of the specification, not its owner. The protocol serves the ecosystem.

---

## BIP Categories

| Category | Description | Examples |
|---|---|---|
| **BSP-BIP-TAXONOMY** | Add or modify biomarker codes | New biomarker, unit correction |
| **BSP-BIP-SCHEMA** | Changes to BEO, IEO, or BioRecord schema | New field, field type change |
| **BSP-BIP-EXCHANGE** | Changes to the Exchange Protocol | New intent, error code |
| **BSP-BIP-GOVERNANCE** | Changes to the governance process itself | BIP template, review timeline |
| **BSP-BIP-INFRA** | Smart contract upgrades | New contract, parameter change |

---

## BIP Status Flow

```
DRAFT → REVIEW → ACCEPTED | REJECTED
                    │
                  FINAL (after implementation)
```

| Status | Description |
|---|---|
| `DRAFT` | Author is drafting — not yet submitted for review |
| `REVIEW` | Submitted — open for community discussion (30 days) |
| `ACCEPTED` | Approved by Institute — scheduled for implementation |
| `REJECTED` | Not accepted — with explanation |
| `FINAL` | Implemented and active in a released BSP version |

---

## Submitting a BIP

1. Fork this repository
2. Copy `bip/BIP-0000-template.md` to `bip/BIP-XXXX-your-title.md`
3. Fill in the template completely
4. Open a Pull Request

The PR opens the 30-day public review period. The Ambrósio Institute will schedule a review call for proposals that reach community consensus.

---

## Critical Parameter Changes

Changes to smart contract parameters or the Governance contract itself require **multi-signature authorization** from Institute keyholders.

This prevents unilateral changes — including by the Institute's founder. The protocol is protected from any single actor.

---

## BIP Index

| BIP | Title | Status |
|---|---|---|
| BIP-0000 | BIP Template | FINAL |

---

*Ambrósio Institute · ambrosio.health · bsp.dev*
