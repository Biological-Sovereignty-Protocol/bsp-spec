# BIP-XXXX — [Title]

| Field | Value |
|---|---|
| **BIP** | XXXX |
| **Title** | [Short descriptive title] |
| **Type** | T (Taxonomy) \| P (Protocol) \| G (Governance) \| I (Informational) |
| **Author(s)** | [Full Name — Institution or Independent] |
| **Contact** | [email@example.com] |
| **Conflict of Interest** | [None \| Describe any financial interest] |
| **Status** | DRAFT |
| **Created** | YYYY-MM-DD |
| **BSP Version** | 1.0 |
| **Comment Period** | 30 days (BIP-T) \| 90 days (BIP-P) \| 120 days (BIP-G) |

---

## Abstract

> Maximum 200 words describing what this BIP proposes.

---

## Motivation

Why is this change needed now? What problem does it solve?

---

## Specification

Precise technical description of the proposed change.

### Current State

What exists today, if applicable.

### Proposed Change

Exactly what would change — use tables, schemas, and examples.

#### For BIP-T (Taxonomy): Biomarker Specification

```yaml
proposed_code: BSP-XX-NNN
name:          "Biomarker Scientific Name"
display_name:  "Human-readable Name"
category:      BSP-XX
level:         CORE | STANDARD | EXTENDED | DEVICE

unit:          "Standard unit (e.g. nmol/L)"
method:        "Measurement method (e.g. ELISA, PCR, LC-MS)"

ref_range:
  optimal:    "Lower-Upper or >X or <X"
  functional: "Lower-Upper"
  deficiency: "<X"
  toxicity:   ">X | null"
  unit:       "Same as above"
  population: "adult-general | specify if narrower"

cost_tier:    LOW | MEDIUM | HIGH | RESEARCH_ONLY

evidence:
  - citation:    "Author et al. (Year). Title. Journal, Vol(Issue), pages."
    doi:         "10.xxxx/xxxxxx"
    year:        YYYY
    study_type:  RCT | Meta-analysis | Cohort | Case-control | Expert-consensus
    n_participants: N
    finding:     "How this study supports inclusion of this biomarker"
    quality:     HIGH | MODERATE | LOW
```

#### For BIP-P (Protocol): Schema Change

```typescript
// Before
interface ExistingType {
  field_name: string
}

// After
interface UpdatedType {
  field_name:     string
  new_field:      string    // Description of new field and its purpose
}
```

### Examples

```json
// Working example of the proposed format or behavior
{
  "example": "value"
}
```

---

## Rationale

Why this specific approach? What alternatives were considered and rejected?

| Alternative | Reason Rejected |
|-------------|----------------|
| Alternative A | ... |
| Alternative B | ... |

---

## Backward Compatibility

- [ ] This change is fully backward compatible
- [ ] This change requires a migration (describe migration path below)
- [ ] This change breaks backward compatibility (provide strong justification)

### Migration Path (if applicable)

---

## Evidence Summary

> Required for BIP-T and BIP-P. At least 2 peer-reviewed references required.

| Reference | Study Type | N | Key Finding | Quality |
|-----------|-----------|---|-------------|---------|
| Author et al. (Year) | Meta-analysis | 50,000 | ... | HIGH |

---

## Implementation Notes

Notes for implementors of this BIP if accepted:
- SDK changes required
- Taxonomy version bump
- Smart contract update required? (Yes / No)

---

## References

- Author et al. (Year). [Title](https://doi.org/...). Journal.
- Additional references...

---

*[Author Name] · [Date] · BIP Status: DRAFT*
