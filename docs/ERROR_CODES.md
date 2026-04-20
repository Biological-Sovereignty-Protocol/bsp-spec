# BSP — Error Codes

> Canonical error catalog. Every BSP implementation (contracts, API, SDK, CLI, MCP server) MUST emit these codes on failure.
> Codes are stable — they will not be renamed across minor versions.

---

## How error codes work

Every BSP error carries:

| Field | Type | Description |
|---|---|---|
| `code` | string | Stable machine-readable identifier (this catalog) |
| `message` | string | Human-readable description |
| `source` | string | Which layer raised it: `contract` \| `api` \| `sdk` \| `cli` \| `mcp` |
| `details` | object | Optional structured context (IDs, timestamps, etc.) |

Clients MUST NOT parse error messages. Always branch on `code`.

---

## Identity (BEO / IEO)

| Code | Source | Meaning |
|---|---|---|
| `BEO_NOT_FOUND` | contract, api | BEO ID does not exist on chain |
| `BEO_LOCKED` | contract | Operation rejected — BEO is in `LOCKED` status |
| `BEO_DESTROYED` | contract | Operation rejected — BEO has been permanently destroyed |
| `BEO_ALREADY_EXISTS` | contract | Domain already bound to an existing BEO |
| `DOMAIN_INVALID` | contract, sdk | `.bsp` domain format invalid (lowercase, `.bsp` suffix, allowed chars) |
| `DOMAIN_RESERVED` | contract | Domain is reserved (e.g. `admin.bsp`, `root.bsp`) |
| `IEO_NOT_FOUND` | contract, api | IEO ID does not exist |
| `IEO_INACTIVE` | contract | IEO is locked or destroyed — cannot receive or submit |
| `IEO_TYPE_INVALID` | sdk | IEO type not in enum (`LAB`, `HOSPITAL`, `WEARABLE`, ...) |
| `KEY_VERSION_MISMATCH` | contract | Signature was produced with an outdated key version |

---

## Consent

| Code | Source | Meaning |
|---|---|---|
| `CONSENT_REQUIRED` | mcp, api | No ConsentToken configured or passed for this operation |
| `TOKEN_NOT_FOUND` | contract, api | ConsentToken ID does not exist |
| `TOKEN_EXPIRED` | contract, mcp | Token `expiresAt` has passed |
| `TOKEN_REVOKED` | contract, mcp | Token was revoked by the holder |
| `INTENT_NOT_AUTHORIZED` | contract, mcp | Required intent not present in token.intents |
| `SCOPE_VIOLATION` | contract | Requested category is outside token.categories |
| `TOKEN_SIGNATURE_INVALID` | contract | Signature on token payload does not match BEO public key |

---

## Cryptography / Signatures

| Code | Source | Meaning |
|---|---|---|
| `SIGNATURE_INVALID` | contract, sdk | Ed25519 verification failed |
| `SIGNATURE_MISSING` | api | Request payload not signed |
| `NONCE_REPLAY` | contract, api | Nonce already used — replay attack detected |
| `NONCE_TOO_SHORT` | sdk | Nonce length below 16-char minimum |
| `TIMESTAMP_TOO_OLD` | contract, api | Request timestamp older than 5 minutes |
| `TIMESTAMP_IN_FUTURE` | contract | Request timestamp beyond acceptable skew |
| `PUBLIC_KEY_NULLIFIED` | contract | BEO key has been nullified (destroyed BEO) |

---

## BioRecord / Data

| Code | Source | Meaning |
|---|---|---|
| `RECORD_NOT_FOUND` | api, contract | BioRecord ID does not exist |
| `RECORD_SCHEMA_INVALID` | sdk, api | Record does not conform to BSP schema |
| `BIOMARKER_CODE_UNKNOWN` | sdk | Biomarker code not in taxonomy |
| `CATEGORY_CODE_UNKNOWN` | sdk | Category code not in taxonomy |
| `UNIT_MISMATCH` | sdk | Unit does not match expected unit for biomarker |
| `VALUE_OUT_OF_RANGE` | sdk | Value outside physiologically plausible range |
| `ARWEAVE_PERSIST_FAILED` | api | Record was validated but could not be persisted to Arweave |
| `ARWEAVE_TX_NOT_CONFIRMED` | api | Arweave TX not yet confirmed at read time |

---

## Relayer / Network

| Code | Source | Meaning |
|---|---|---|
| `RELAYER_UNREACHABLE` | sdk, cli, mcp | Cannot connect to the registry endpoint |
| `RELAYER_GAS_FAILED` | api | Relayer could not submit Aptos transaction (insufficient gas) |
| `NETWORK_MISMATCH` | sdk | Payload network (mainnet/testnet) does not match relayer |
| `RATE_LIMITED` | api | Too many requests from this origin |
| `CHAIN_TX_FAILED` | api | Aptos transaction reverted |
| `CHAIN_TIMEOUT` | api | Aptos transaction did not confirm within window |

---

## Configuration / Client

| Code | Source | Meaning |
|---|---|---|
| `PRIVATE_KEY_MISSING` | cli, sdk | No private key configured |
| `PRIVATE_KEY_INVALID` | cli, sdk | Private key not a valid 128-char hex Ed25519 key |
| `CONFIG_NOT_FOUND` | cli | Config file does not exist at `~/.bsp/config.json` |
| `CONFIRM_REQUIRED` | cli | Destructive operation invoked without `--confirm` |
| `ENV_VAR_MISSING` | mcp | Required env var not set (e.g. `BSP_BEO_DOMAIN`) |

---

## Cross-reference

| Canonical source | Location |
|---|---|
| **Contract errors** | `ambrosio-institute/bsp-contracts` → `sources/errors.move` |
| **API errors** | `ambrosio-institute/bsp-registry-api` → `src/errors/catalog.ts` |
| **SDK errors** | `bsp-sdk-typescript` → `src/errors.ts` / `bsp-sdk-python` → `bsp_sdk/errors.py` |
| **CLI errors** | `bsp-cli` → `src/lib/errors.ts` |
| **MCP errors** | `bsp-mcp` → `src/index.ts` (ConsentGuard + tool handlers) |

---

## Stability guarantee

- Error codes are **append-only** within a major version.
- A code MAY change its `message` without notice. Never parse messages.
- Deprecated codes will be announced one minor version before removal.

## Related

- **Architecture** — `docs/ARCHITECTURE.md`
- **Glossary** — `docs/GLOSSARY.md`
- **Governance** — `spec/governance.md`
