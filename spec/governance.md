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

## Emergency Operations

Certain conditions require immediate protocol-level action outside the normal BIP review cycle. These operations are authorized by the multi-signature governance threshold and take effect on-chain without waiting for community review.

### Trigger Conditions

| Condition | Examples |
|---|---|
| Security incident | Active exploit draining BEO consent tokens, unauthorized mass data read |
| Key compromise | Institute keyholder private key leaked or suspected stolen |
| Critical bug | Smart contract logic error with potential for data loss or consent bypass |
| Regulatory emergency | Court order or regulatory injunction requiring immediate suspension |

### Suspension Action

To suspend an IEO or freeze exchange operations, submit a governance transaction using the `ACTION_SUSPEND_IEO` action type via the Governance contract.

```typescript
GovernanceAction {
  action_type: 'ACTION_SUSPEND_IEO'
  target_ieo:  string       // ieo_id of the institution to suspend
  reason:      string       // Mandatory — recorded on-chain permanently
  duration:    number | null // Seconds until auto-expiry, or null for indefinite
  evidence:    string       // Arweave tx ID of the incident evidence package
}
```

For protocol-wide freezes (all exchange halted), use `ACTION_FREEZE_EXCHANGE`. This should be reserved for catastrophic scenarios only.

### Approval Threshold

Emergency actions require a **2-of-3 multi-signature** from the current Institute keyholders. This is the same threshold as critical parameter changes. No single person — including the Institute founder — can unilaterally trigger a suspension.

If a keyholder is unavailable (e.g. the compromise is of their key), a **1-of-2 emergency override** is available with a mandatory 24-hour on-chain delay, after which any remaining keyholder can countermand the action.

### Reactivation Process

1. Incident confirmed resolved (internal post-mortem document published on Arweave).
2. Submit `ACTION_REINSTATE_IEO` (or `ACTION_UNFREEZE_EXCHANGE`) via Governance contract.
3. Same 2-of-3 multi-signature threshold applies.
4. A BIP documenting the incident and any spec-level mitigations is filed within 30 days and fast-tracked to `REVIEW` status.
5. Reinstated IEO status returns to `ACTIVE`. All suspended consent tokens that were valid at the time of suspension are automatically revalidated unless individually revoked during the suspension window.

---

## BIP Index

| BIP | Title | Status |
|---|---|---|
| BIP-0000 | BIP Template | FINAL |

---

*Ambrósio Institute · ambrosioinstitute.org · biologicalsovereigntyprotocol.com*
