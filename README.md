# BSP — Biological Sovereignty Protocol

> **"The protocol that connects every health system on earth."**
> Version 0.2 — Specification Draft
> Published by the Ambrósio Institute · [ambrosioinstitute.org](https://ambrosioinstitute.org) · [biologicalsovereigntyprotocol.com](https://biologicalsovereigntyprotocol.com)

---

## What is BSP?

The Biological Sovereignty Protocol (BSP) is an open standard that defines a universal language for exchanging health and longevity data.

BSP enables any wearable device, laboratory system, health platform, telemedicine service, or artificial intelligence engine to communicate using a shared, structured, and interoperable protocol.

**The BSP is the language. The AVA is the intelligence that speaks it.**

> The BSP defines the protocol, not the intelligence.
> Any system in the world can implement the BSP.
> Only Ambrósio holds the AVA.

---

## The Problem BSP Solves

### 1. Fragmentation
A patient's blood test from Laboratory A cannot be read by Platform B. A longevity protocol designed by a physician in São Paulo cannot be automatically executed by a telemedicine platform in Tokyo. Every health company builds proprietary silos — and patients pay the price.

### 2. Sovereignty Failure
Individuals do not hold sovereignty over their own biological data. Health data is owned by hospitals, insurance companies, and technology platforms — not by the person to whom it belongs.

### 3. AI Readiness Gap
Artificial intelligence has the potential to transform medicine — but only if it has access to structured, standardized, high-quality biological data. Today, most health data is unstructured, inconsistent, and trapped in proprietary formats.

---

## Design Principles

| Principle | Description |
|---|---|
| **Openness** | The protocol specification is fully open. Any individual, company, or institution may implement BSP without licensing fees or permission. |
| **Neutrality** | BSP does not favor any algorithm, scoring system, or health philosophy. It is a neutral transport layer for biological data. |
| **Sovereignty** | Every BSP record belongs to a biological entity — a human being. Implementations must support individual data ownership as a first-class right. |
| **Extensibility** | The protocol is versioned and extensible. New biomarkers and data types can be added without breaking backward compatibility. |
| **Intelligence Agnosticism** | BSP defines how data is structured in transit — not what conclusions to draw from it. Intelligence layers such as AVA operate above the protocol, not inside it. |
| **Biological Completeness** | The biomarker taxonomy covers the complete spectrum of human biology. Any measurement a laboratory can perform has a BSP code. |
| **Permanence** | Biological data is permanent by design. Built on Arweave, BSP records cannot be deleted, altered, or lost. |

---

## Core Architecture

BSP operates in three layers:

| Layer | Name | Responsibility |
|---|---|---|
| **BSP-Identity** | Biological Identity | Defines the sovereign biological identity object — the BEO — lifelong carrier of a person's health data |
| **BSP-Data** | Biological Data Schema | Defines the structure of all biological measurements and biomarker records |
| **BSP-Exchange** | Communication Protocol | Defines how systems request and respond with biological data |

---

## Repository Structure

```
bsp-spec/
├── spec/
│   ├── overview.md          # Architecture — the three layers
│   ├── beo.md               # Biological Entity Object — complete specification
│   ├── ieo.md               # Institutional Entity Object
│   ├── biorecord.md         # BioRecord format, fields, and validation
│   ├── exchange.md          # Exchange Protocol — request/response
│   ├── bsp-domain.md        # .bsp domain system
│   ├── governance.md        # Governance model and BIP process
│   └── taxonomy/
│       ├── level-1-core.md      # 9 categories — advanced longevity biomarkers
│       ├── level-2-standard.md  # 9 categories — routine laboratory biomarkers
│       ├── level-3-extended.md  # 6 categories — specialized biomarkers
│       └── level-4-device.md    # 1 category — continuous wearable data
├── bip/                     # BSP Improvement Proposals
│   └── BIP-0000-template.md
├── examples/                # BioRecords and BEOs as JSON examples
│   ├── beo-example.json
│   └── biorecord-example.json
└── LICENSE                  # Creative Commons CC BY 4.0
```

---

## Implementation Guides

**Already understand the spec and want to build?** Skip straight to the step-by-step guides:

| Guide | Who it's for |
|---|---|
| [Implementation Guide](docs/implementation-guide.md) | App developers, institution integrators, and protocol operators |

The guide covers three paths:
- **User App Developer** — build an app where users create their BSP identity
- **Institution (IEO) Developer** — receive BSP-authorized health data as a lab or clinic
- **Protocol Integrator** — run your own BSP node and relayer

It includes working code snippets and a Common Pitfalls section.

---

## Quick Start

### For Developers & Laboratories

1. Read [`spec/overview.md`](spec/overview.md) — understand the three layers
2. Read [`spec/beo.md`](spec/beo.md) — understand the sovereign identity object
3. Read [`spec/biorecord.md`](spec/biorecord.md) — understand the data format
4. Browse [`spec/taxonomy/`](spec/taxonomy/) — find the biomarker codes for your use case
5. Check [`examples/`](examples/) — working JSON examples

### For Implementors

Install an official SDK:
```bash
# TypeScript / JavaScript
npm install bsp-sdk

# Python
pip install bsp-sdk
```

→ SDK repositories: [bsp-sdk-typescript](https://github.com/Biological-Sovereignty-Protocol/bsp-sdk-typescript) · [bsp-sdk-python](https://github.com/Biological-Sovereignty-Protocol/bsp-sdk-python)

---

## The Biomarker Taxonomy

BSP defines the most comprehensive open biomarker taxonomy ever codified — covering the complete spectrum of measurable human biology.

| Level | Coverage | Categories |
|---|---|---|
| **L1 — Core** | Advanced longevity and biological aging biomarkers | 9 |
| **L2 — Standard** | Routine laboratory biomarkers performed worldwide | 9 |
| **L3 — Extended** | Specialized clinical and research biomarkers | 6 |
| **L4 — Device** | Continuous biometric data from wearable devices | 1 |

Full taxonomy: [`spec/taxonomy/`](spec/taxonomy/)

---

## Contributing

BSP is governed through an open improvement process. To propose changes to the specification:

1. Read [`spec/governance.md`](spec/governance.md)
2. Use the BIP template at [`bip/BIP-0000-template.md`](bip/BIP-0000-template.md)
3. Open a Pull Request

The Ambrósio Institute reviews all BIPs. Protocol changes require community discussion and multi-signature approval.

---

## License

This specification is published under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE).

Any individual, company, or institution may implement the BSP, build on it, or distribute it — without licensing fees or permission requirements.

---

*The protocol belongs to the world. The intelligence belongs to Ambrósio. The sovereignty belongs to the individual.*

**Ambrósio Institute** · [ambrosioinstitute.org](https://ambrosioinstitute.org) · [biologicalsovereigntyprotocol.com](https://biologicalsovereigntyprotocol.com)
