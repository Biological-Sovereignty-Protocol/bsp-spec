# BSP — Architecture

> How the Biological Sovereignty Protocol fits together, end to end.
> Version 0.2 — April 2026

---

## Overview

BSP is layered. The user holds the keys. The SDK and CLI sign operations locally. A gasless relayer submits transactions to Aptos Move contracts. Payloads are persisted on Arweave for permanence. No component in the chain can forge or modify a user's actions without the user's key.

---

## System Diagram

```mermaid
flowchart LR
    User([User / Patient]) -->|opens| Frontend[bsp-id-web<br/>Frontend]
    User -->|runs| CLI[bspctl<br/>CLI]
    User -->|delegates to| AI[AI Assistant]

    AI -->|via MCP| MCP[bsp-mcp<br/>MCP Server]

    Frontend -->|uses| SDK[bsp-sdk<br/>TS / Python]
    CLI -->|uses| SDK
    MCP -->|uses| SDK

    SDK -->|signed payload<br/>Ed25519| API[Registry API<br/>Gasless Relayer]
    API -->|submits TX<br/>pays gas| Contracts[Aptos Move<br/>Contracts]
    API -->|persists| Arweave[(Arweave<br/>Permanent Storage)]

    Contracts -->|verifies signature<br/>on-chain| Contracts
    Contracts -->|emits events| Indexer[Indexer]
    Indexer -->|reads| API

    style User fill:#fff2cc,stroke:#d6b656
    style Contracts fill:#d5e8d4,stroke:#82b366
    style Arweave fill:#dae8fc,stroke:#6c8ebf
```

### Layers

| Layer | Component | Role |
|---|---|---|
| **Presentation** | `bsp-id-web`, `bspctl`, AI assistants | User-facing surfaces |
| **Client SDK** | `bsp-sdk-typescript`, `bsp-sdk-python` | Ed25519 signing, type definitions, payload builders |
| **Transport** | Registry API (relayer) | Gasless submission of signed payloads to chain |
| **Protocol** | Aptos Move contracts | Source of truth — BEO, IEO, ConsentToken, AccessControl |
| **Storage** | Arweave | Permanent BioRecord persistence |

---

## Trust Boundaries

| Boundary | Who trusts what |
|---|---|
| User ↔ Client (SDK/CLI/Web) | User trusts their machine and the signing code they run |
| Client ↔ Relayer | Client trusts nothing — payloads are signed. Relayer cannot forge. |
| Relayer ↔ Chain | Chain verifies every signature. Relayer is only a gas payer. |
| Chain ↔ Arweave | Chain pins the Arweave TX ID. Arweave guarantees permanence. |

The relayer is **infrastructure, not authority**. A compromised relayer cannot modify consent, forge records, or destroy a BEO. The worst a rogue relayer can do is refuse to relay — in which case any other relayer (or the user directly) can take over.

See `docs/RELAYER_SPEC.md` for the full relayer specification, minimum interface, and the permissionless multi-relayer model.

---

## Sequence: Create a BEO

```mermaid
sequenceDiagram
    actor User
    participant App as Client<br/>(SDK / CLI / Web)
    participant API as Registry API<br/>(Relayer)
    participant Chain as Aptos Move<br/>Contract
    participant AR as Arweave

    User->>App: Choose domain (e.g. alice.bsp)
    App->>App: Generate Ed25519 keypair locally
    App->>App: Build BEO payload + sign
    App->>API: POST /beo (signed payload)
    API->>Chain: submit_create_beo(payload, sig)
    Chain->>Chain: Verify Ed25519 signature
    Chain->>Chain: Check domain availability
    Chain->>AR: Pin BEO metadata
    AR-->>Chain: tx_id
    Chain-->>API: BEO created (beoId, txId)
    API-->>App: Response
    App-->>User: Show keys (ONCE) + BEO ID

    Note over User: User stores private key<br/>OFFLINE. Never transmitted.
```

---

## Sequence: Grant ConsentToken

```mermaid
sequenceDiagram
    actor User
    participant App as Client (holder)
    participant API as Registry API
    participant Chain as AccessControl<br/>Contract
    participant IEO as Institution

    User->>App: Grant consent to IEO<br/>(intents + categories + expiry)
    App->>App: Build token payload
    App->>App: Sign with BEO private key
    App->>API: POST /consent/grant
    API->>Chain: issue_token(payload, sig)
    Chain->>Chain: Verify holder signature
    Chain->>Chain: Validate IEO exists + active
    Chain->>Chain: Mint ConsentToken (tokenId)
    Chain-->>API: ConsentToken created
    API-->>App: tokenId + details
    App-->>User: Show tokenId
    User->>IEO: Share tokenId<br/>(out of band)

    Note over Chain: Token state is on-chain.<br/>Revocation is immediate.
```

---

## Sequence: Submit BioRecord (IEO → BEO)

```mermaid
sequenceDiagram
    participant IEO as Institution<br/>(Lab / Hospital)
    participant API as Registry API
    participant Chain as Contracts
    participant AR as Arweave

    IEO->>IEO: Build BioRecord<br/>(category, code, value, refs)
    IEO->>IEO: Sign with IEO private key
    IEO->>API: POST /records/submit<br/>(beoId, tokenId, record, sig)
    API->>Chain: verify_consent(tokenId, intent=SUBMIT_RECORD)
    Chain-->>API: OK (scope matches)
    API->>Chain: record_submission(beoId, recordId, sig)
    Chain->>AR: Persist full BioRecord JSON
    AR-->>Chain: arweave_tx_id
    Chain->>Chain: Link recordId → arweave_tx_id
    Chain-->>API: Submission confirmed
    API-->>IEO: recordId + arweaveTxId
```

---

## Sequence: Destroy BEO (Cryptographic Erasure)

```mermaid
sequenceDiagram
    actor User
    participant App as Client
    participant Chain as Contracts
    participant AC as AccessControl
    participant Dom as Domain Registry

    User->>App: bsp destroy <beoId> --confirm
    App->>App: Build destruction payload + sign
    App->>Chain: submit_destroy_beo(payload, sig)
    Chain->>Chain: Verify holder signature
    Chain->>Chain: Nullify public key<br/>(cryptographic erasure)
    Chain->>AC: Revoke ALL ConsentTokens
    AC-->>Chain: All tokens revoked
    Chain->>Dom: Release .bsp domain
    Dom-->>Chain: Domain released
    Chain->>Chain: Wipe recovery config
    Chain->>Chain: Set status = DESTROYED
    Chain-->>App: Destruction confirmed
    App-->>User: BEO destroyed — no undo

    Note over Chain: Arweave records remain<br/>but are now unreadable<br/>(key nullified).
    Note over User: Implements LGPD Art. 18<br/>& GDPR Art. 17.
```

---

## Component Responsibilities

### `bsp-sdk` (TypeScript / Python)
- Ed25519 keypair generation and signing
- Canonical payload serialization
- Type definitions for BEO, IEO, ConsentToken, BioRecord
- `ExchangeClient` — fetches records with on-chain consent verification

### `bspctl` (CLI)
- 22 commands covering full protocol lifecycle
- Local config at `~/.bsp/config.json`
- Delegates signing to `bsp-sdk`
- No server-side state

### `bsp-mcp` (MCP Server)
- stdio transport for AI assistants
- `ConsentGuard` gates every data-access tool
- Enforces intent + expiry + on-chain revocation state

### `bsp-id-web` (Frontend)
- React + Vite web app
- Generates keys in-browser (never sent to server)
- Issues ConsentTokens through a visual flow

### `bsp-registry-api` (Relayer)
- Receives signed payloads
- Submits Aptos transactions
- Pays gas on behalf of users
- Cannot modify or forge — signatures are verified on-chain

### Aptos Move Contracts
- `BEO` — sovereign biological identity
- `IEO` — institutional entity
- `AccessControl` — ConsentToken issuance, revocation, scope checks
- `DomainRegistry` — `.bsp` namespace

### Arweave
- Permanent storage for BioRecord JSON
- Chain stores only the Arweave TX ID pointer
- Economic incentive ensures 200+ year persistence

---

## Extensibility

New biomarkers, intents, and IEO types are added through the BIP (BSP Improvement Proposal) process. See `../bip/BIP-0000-template.md` and `spec/governance.md`.

---

## Related

- **Glossary** — `docs/GLOSSARY.md`
- **Error Codes** — `docs/ERROR_CODES.md`
- **Relayer Specification** — `docs/RELAYER_SPEC.md`
- **Threat Model** — `docs/THREAT_MODEL.md`
- **Implementation Guide** — `docs/implementation-guide.md`
- **Specification** — `spec/overview.md`
