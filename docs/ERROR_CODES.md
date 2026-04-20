# BSP — Error Codes (Single Source of Truth)

> **Canonical error catalog for the entire BSP stack.**
>
> Every BSP implementation — Move contracts, Registry API, SDKs (TypeScript & Python), CLI, and MCP server — MUST emit the codes defined here.
>
> This document is the **single source of truth**. Downstream repos (`bsp-contracts`, `bsp-registry-api`, `bsp-sdk-typescript`, `bsp-sdk-python`, `bsp-cli`, `bsp-mcp`) link to this file and do **not** duplicate the catalog.

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

## Stability guarantee

- Error codes are **append-only** within a major version.
- A code MAY change its `message` without notice. Never parse messages.
- Deprecated codes will be announced one minor version before removal.
- Numeric `abort_code` values in Move modules are stable — ranges documented below are reserved per module and MUST NOT collide.

---

# Part I — High-level Protocol Codes

Protocol-level identifiers used by SDKs, CLI, MCP, and public APIs. These are string codes, stable across implementations.

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

## Relayer / Network

| Code | Source | Meaning |
|---|---|---|
| `RELAYER_UNREACHABLE` | sdk, cli, mcp | Cannot connect to the registry endpoint |
| `RELAYER_GAS_FAILED` | api | Relayer could not submit Aptos transaction (insufficient gas) |
| `NETWORK_MISMATCH` | sdk | Payload network (mainnet/testnet) does not match relayer |
| `RATE_LIMITED` | api | Too many requests from this origin |
| `CHAIN_TX_FAILED` | api | Aptos transaction reverted |
| `CHAIN_TIMEOUT` | api | Aptos transaction did not confirm within window |

## Configuration / Client

| Code | Source | Meaning |
|---|---|---|
| `PRIVATE_KEY_MISSING` | cli, sdk | No private key configured |
| `PRIVATE_KEY_INVALID` | cli, sdk | Private key not a valid 128-char hex Ed25519 key |
| `CONFIG_NOT_FOUND` | cli | Config file does not exist at `~/.bsp/config.json` |
| `CONFIRM_REQUIRED` | cli | Destructive operation invoked without `--confirm` |
| `ENV_VAR_MISSING` | mcp | Required env var not set (e.g. `BSP_BEO_DOMAIN`) |

---

# Part II — Registry API HTTP Codes

HTTP semantics and machine codes for `bsp-registry-api`. All error responses follow:

```json
{ "error": "Human-readable message", "code": "OPTIONAL_MACHINE_CODE" }
```

Some consent-verification endpoints return `{ "valid": false, "reason": "..." }` inside a 200 OK — those are domain reasons, not HTTP errors.

## HTTP Status Codes

| Status | Meaning | When it happens |
|---|---|---|
| **200 OK** | Success | Request completed. For verify/cert endpoints, check `valid`/`certified` field. |
| **201 Created** | Resource created | BEO/IEO/ConsentToken successfully written on-chain. |
| **400 Bad Request** | Validation error | Missing/invalid field, malformed nonce/timestamp, schema mismatch. |
| **401 Unauthorized** | Auth failed | Invalid Ed25519 signature, invalid/expired JWT, revoked/expired consent token. |
| **403 Forbidden** | Not authorized | Consent token valid but intent/category not in scope. Institute-only endpoints without API key. |
| **404 Not Found** | Resource missing | BEO, IEO, or ConsentToken does not exist on-chain. |
| **409 Conflict** | Nonce reuse | `nonce` already consumed (replay protection). |
| **422 Unprocessable Entity** | Business rule | e.g. `TOO_MANY_RECORDS` — payload schema is valid but violates a policy. |
| **429 Too Many Requests** | Rate limited | Per-route quotas exceeded (10–60 req/min depending on route group). |
| **500 Internal Server Error** | Unexpected failure | Uncaught exception. Logs contain the correlation id. |
| **503 Service Unavailable** | Upstream flaky | Aptos RPC circuit breaker is open — retry in ~30s. |

## API Machine Codes

### Authentication / Signatures

| Code | HTTP | Description | Action |
|---|---|---|---|
| `INVALID_SIGNATURE` | 401 | Ed25519 verification failed against the BEO/IEO on-chain public key. | Re-sign the canonical payload with the correct key; verify nonce+timestamp are echoed back. |
| `TOKEN_NOT_FOUND` | 401 | ConsentToken id not present on-chain. | Check the token was created and the id is correct. |
| `TOKEN_REVOKED` | 401 | Token was revoked by the BEO holder. | Request a fresh ConsentToken. |
| `TOKEN_EXPIRED` | 401 | Token `expires_at` is past. | Request a fresh ConsentToken with a new TTL. |
| `INTENT_NOT_AUTHORIZED` | 403 | Required intent (`SUBMIT_RECORD`, `READ_RECORDS`, `EXPORT_DATA`) not in token scope. | Ask the holder to grant a new token with the required intent. |
| `IEO_NOT_FOUND` | 401 | IEO referenced by the token no longer exists / is destroyed. | Re-issue from a live IEO. |

### Consent Exchange

| Code | HTTP | Description | Action |
|---|---|---|---|
| `RECORDS_REQUIRED` | 400 | `records` must be a non-empty array. | Send at least one record. |
| `TOO_MANY_RECORDS` | 422 | More than 50 records in a single `submitRecords` call. | Split into multiple batches of <=50. |
| `ENCRYPTION_SECRET_MISSING` | 500 | Server was not configured with `BIOSTORE_ENCRYPTION_SECRET` or `JWT_SECRET`. | Operator: set the env var and restart. |

### Nonce / Replay Protection

| Code | HTTP | Description | Action |
|---|---|---|---|
| `NONCE_REQUIRED` | 400 | Missing `nonce` field. | Send a fresh >=16-char random nonce. |
| `NONCE_TOO_SHORT` | 400 | Nonce has fewer than 16 characters. | Use a >=16-char cryptographic random nonce. |
| `NONCE_REUSED` | 409 | Nonce was already seen (replay detected). | Generate a new nonce per request. |
| `TIMESTAMP_SKEW` | 400 | Timestamp more than 5 minutes from server clock. | Sync client clock (NTP) and retry. |

### Guardians

| Code | HTTP | Description | Action |
|---|---|---|---|
| `JWT_INVALID` | 401 | Invitation/recovery JWT signature invalid. | Request a new invitation. |
| `JWT_EXPIRED` | 401 | JWT past 72h TTL. | Ask the BEO holder to resend the invitation. |
| `INVITATION_REPLAYED` | 401 | Invitation JWT was already accepted. | A guardian can only accept once. |

### Arweave

| Code | HTTP | Description | Action |
|---|---|---|---|
| `ARWEAVE_UPLOAD_FAILED` | 500 | Upload rejected after 3 retries (exponential backoff 1s->4s). | Retry later; if persistent, check `ARWEAVE_WALLET_KEY` balance. |
| `ARWEAVE_TXID_INVALID` | 400 | Referenced Arweave tx id is malformed. | Verify the id is 43 base64url chars. |

### Aptos / Circuit Breaker

| Code | HTTP | Description | Action |
|---|---|---|---|
| `APTOS_RPC_UNAVAILABLE` | 503 | Circuit breaker is OPEN after >50% error rate over the rolling window. | Retry after ~30s — the breaker will half-open automatically. |
| `APTOS_SUBMIT_FAILED` | 500 | Transaction submit/wait errored (non-retryable path). | Inspect logs; verify relayer account has APT for gas. |

### Institute / Admin

| Code | HTTP | Description | Action |
|---|---|---|---|
| `INSTITUTE_API_KEY_MISSING` | 401 | `X-Institute-Api-Key` header absent. | Provide the key (admin only). |
| `INSTITUTE_API_KEY_INVALID` | 401 | Provided key does not match `INSTITUTE_API_KEY`. | Confirm the key; rotate if compromised. |

## API Handling Guidance

- **Retries:** Safe to retry on `5xx` and `APTOS_RPC_UNAVAILABLE`. Never retry on `4xx` without changing the payload (nonce must be fresh).
- **Correlation id:** Every response includes `X-Request-Id`; log it and include it in support tickets.
- **Backoff:** When you hit `429` or `503`, honor `Retry-After` when present; otherwise exponential backoff with jitter (1s, 2s, 4s, cap at 30s).
- **Consent verification:** `GET /api/consent/:tokenId` returns **200** with `{ valid: false, reason }` for domain-level failures. Only transport/server errors surface as 4xx/5xx here.

---

# Part III — Move Contract Abort Codes

Numeric `abort_code` values emitted by each Move module in `bsp-contracts/src/move/sources/`. SDKs and relayers MUST map `vm_status.abort_code` back to these constants by module range.

## Code ranges per module

| Range | Module | File |
|-------|--------|------|
| 1 – 99 | `beo_registry` | `src/move/sources/beo_registry.move` |
| 100 – 199 | `ieo_registry` | `src/move/sources/ieo_registry.move` |
| 200 – 299 | `domain_registry` | `src/move/sources/domain_registry.move` |
| 300 – 399 | `access_control` | `src/move/sources/access_control.move` |
| 400 – 499 | `governance` | `src/move/sources/governance.move` |

## `beo_registry` (1 – 99)

| Código | Constante | Descrição | Como tratar |
|--------|-----------|-----------|-------------|
| 1 | `E_BEO_NOT_FOUND` | BEO id inexistente | Checar `beo_id`; oferecer criar BEO novo |
| 2 | `E_BEO_DESTROYED` | BEO foi destruído (LGPD) | Estado terminal — criar novo BEO |
| 3 | `E_BEO_LOCKED` | BEO bloqueado (emergência) | Pedir unlock ou recovery |
| 4 | `E_DOMAIN_REQUIRED` | Domínio vazio | Preencher domínio `.bsp` |
| 5 | `E_DOMAIN_TOO_LONG` | Domínio acima do limite | Reduzir tamanho (<64 chars) |
| 6 | `E_DOMAIN_INVALID_SUFFIX` | Sufixo não é `.bsp` | Usar sufixo `.bsp` |
| 7 | `E_DOMAIN_INVALID_FORMAT` | Caracteres inválidos | Apenas `[a-z0-9-]` |
| 8 | `E_DOMAIN_TAKEN` | Domínio já registrado | Escolher outro domínio |
| 9 | `E_PUBLIC_KEY_TAKEN` | Pubkey já usada | Gerar novo par de chaves |
| 10 | `E_FIELD_REQUIRED` | Campo obrigatório ausente | Validar input antes do envio |
| 11 | `E_INVALID_SIGNATURE` | Assinatura Ed25519 inválida | Conferir chave + mensagem canônica |
| 12 | `E_TIMESTAMP_TOO_OLD` | Timestamp expirado (>5min) | Reassinar com `now()` |
| 13 | `E_TIMESTAMP_FUTURE` | Timestamp no futuro | Conferir clock do cliente |
| 14 | `E_RECOVERY_NO_CONFIG` | Nenhum guardião configurado | `update_recovery` primeiro |
| 15 | `E_RECOVERY_IN_PROGRESS` | Recovery já em andamento | Aguardar expiração ou conclusão |
| 16 | `E_RECOVERY_COOLDOWN` | Cooldown de 24h ativo | Aguardar janela de cooldown |
| 17 | `E_RECOVERY_EXPIRED` | Janela de 72h expirou | Reiniciar recovery |
| 18 | `E_GUARDIAN_NOT_FOUND` | Contato não é guardião | Confirmar lista de guardians |
| 19 | `E_GUARDIAN_ALREADY_ACTIVE` | Guardião já aceitou | Nada a fazer |
| 20 | `E_GUARDIAN_NO_KEY` | Guardião sem pubkey | Guardião precisa aceitar primeiro |
| 21 | `E_GUARDIAN_ALREADY_CONFIRMED` | Guardião já confirmou recovery | Usar outro guardião |
| 22 | `E_KEY_SAME` | Nova chave igual à antiga | Gerar chave diferente |
| 23 | `E_KEY_INVALID_LENGTH` | Pubkey com tamanho inválido | Usar chave Ed25519 de 32 bytes |
| 24 | `E_RECOVERY_GUARDIANS_MIN` | <2 guardiões | Adicionar mais guardiões |
| 25 | `E_RECOVERY_THRESHOLD_INVALID` | Threshold fora de range | `1 <= threshold <= len(guardians)` |
| 26 | `E_GUARDIAN_ROLE_INVALID` | Role fora de `{0,1,2}` | Usar PRIMARY/SECONDARY/TERTIARY |
| 27 | `E_GUARDIAN_DUPLICATE_CONTACT` | Mesmo contato duas vezes | Contatos devem ser únicos |
| 28 | `E_ALREADY_LOCKED` | BEO já bloqueado | Nada a fazer |
| 29 | `E_NOT_LOCKED` | BEO não está bloqueado | Nada a fazer |
| 30 | `E_ALREADY_DESTROYED` | BEO já destruído | Estado terminal |
| 31 | `E_NOT_INITIALIZED` | Módulo não inicializado | Chamar `initialize` primeiro |
| 32 | `E_ALREADY_INITIALIZED` | Módulo já inicializado | Uma inicialização apenas |

## `ieo_registry` (100 – 199)

| Código | Constante | Descrição | Como tratar |
|--------|-----------|-----------|-------------|
| 100 | `E_IEO_NOT_FOUND` | IEO inexistente | Checar `ieo_id` |
| 101 | `E_IEO_DESTROYED` | IEO destruído | Estado terminal |
| 102 | `E_IEO_LOCKED` | IEO bloqueado | Unlock via governance |
| 103 | `E_IEO_SUSPENDED` | IEO suspenso por compliance | Resolver compliance |
| 104 | `E_IEO_REVOKED` | IEO revogado | Criar novo IEO |
| 105 | `E_DOMAIN_REQUIRED` | Domínio vazio | Preencher domínio |
| 106 | `E_DOMAIN_TOO_LONG` | Domínio muito longo | Reduzir tamanho |
| 107 | `E_DOMAIN_TAKEN` | Domínio já em uso | Escolher outro |
| 108 | `E_FIELD_REQUIRED` | Campo obrigatório ausente | Validar input |
| 109 | `E_INVALID_SIGNATURE` | Assinatura Ed25519 inválida | Reassinar com chave correta |
| 110 | `E_TIMESTAMP_TOO_OLD` | Timestamp expirado | Reassinar com `now()` |
| 111 | `E_TIMESTAMP_FUTURE` | Timestamp no futuro | Conferir clock |
| 112 | `E_NOT_KEYHOLDER` | Caller não é keyholder | Usar conta autorizada |
| 113 | `E_INVALID_IEO_TYPE` | Tipo IEO desconhecido | Usar tipo cadastrado |
| 114 | `E_INVALID_CERT_LEVEL` | Nível de certificação inválido | Consultar lista |
| 115 | `E_KEY_INVALID_LENGTH` | Pubkey tamanho inválido | Ed25519 32 bytes |
| 116 | `E_KEY_SAME` | Nova chave igual à antiga | Gerar chave diferente |
| 117 | `E_RECOVERY_NO_CONFIG` | Sem config de recovery | Configurar guardiões |
| 118 | `E_RECOVERY_IN_PROGRESS` | Recovery em andamento | Aguardar |
| 119 | `E_RECOVERY_COOLDOWN` | Cooldown ativo | Aguardar |
| 120 | `E_RECOVERY_EXPIRED` | Recovery expirado | Reiniciar |
| 121 | `E_GUARDIAN_NOT_FOUND` | Guardião não encontrado | Confirmar lista |
| 122 | `E_GUARDIAN_ALREADY_ACTIVE` | Guardião já ativo | Nada a fazer |
| 123 | `E_GUARDIAN_NO_KEY` | Guardião sem chave | Precisa aceitar |
| 124 | `E_GUARDIAN_ALREADY_CONFIRMED` | Guardião já confirmou | Usar outro |
| 125 | `E_RECOVERY_GUARDIANS_MIN` | <2 guardiões | Adicionar mais |
| 126 | `E_RECOVERY_THRESHOLD_INVALID` | Threshold fora de range | Corrigir |
| 127 | `E_GUARDIAN_ROLE_INVALID` | Role inválido | Usar PRIMARY/SECONDARY/TERTIARY |
| 128 | `E_GUARDIAN_DUPLICATE_CONTACT` | Contato duplicado | Contatos únicos |
| 129 | `E_ALREADY_LOCKED` | Já bloqueado | Nada a fazer |
| 130 | `E_NOT_LOCKED` | Não está bloqueado | Nada a fazer |
| 131 | `E_ALREADY_DESTROYED` | Já destruído | Estado terminal |
| 132 | `E_NOT_INITIALIZED` | Módulo não inicializado | Chamar `initialize` |
| 133 | `E_ALREADY_INITIALIZED` | Já inicializado | Uma vez apenas |
| 139 | `E_ALREADY_KEYHOLDER` | Já é keyholder | Nada a fazer |
| 140 | `E_LAST_KEYHOLDER` | Último keyholder, não pode remover | Adicionar outro primeiro |
| 141 | `E_IEO_TYPE_EXISTS` | Tipo IEO duplicado | Escolher outro nome |
| 142 | `E_IEO_TYPE_NOT_FOUND` | Tipo IEO não cadastrado | Registrar primeiro |
| 143 | `E_LAST_IEO_TYPE` | Último tipo IEO | Manter ao menos um |
| 144 | `E_CANNOT_DESTROY_REVOKED` | IEO revogado não pode destruir | Desrevogar primeiro |
| 145 | `E_NOT_GOVERNANCE` | Caller não é governance | Usar módulo governance |

## `domain_registry` (200 – 299)

| Código | Constante | Descrição | Como tratar |
|--------|-----------|-----------|-------------|
| 200 | `E_DOMAIN_REQUIRED` | Domínio vazio | Preencher |
| 201 | `E_DOMAIN_TOO_LONG` | Domínio muito longo | Reduzir |
| 202 | `E_DOMAIN_INVALID_SUFFIX` | Sufixo inválido | Usar `.bsp` |
| 203 | `E_DOMAIN_ALREADY_REGISTERED` | Já registrado | Escolher outro |
| 204 | `E_DOMAIN_NOT_FOUND` | Domínio não encontrado | Verificar spelling |
| 205 | `E_NOT_AUTHORIZED` | Caller não autorizado | Usar institute ou registry addr |
| 206 | `E_ENTITY_ID_REQUIRED` | entity_id=0 | Preencher id válido |
| 207 | `E_INVALID_ENTITY_TYPE` | Tipo inválido | `BEO` ou `IEO` |
| 208 | `E_BEO_NON_TRANSFERABLE` | BEO não transferível | Criar novo BEO |
| 209 | `E_NOT_INITIALIZED` | Não inicializado | `initialize` primeiro |
| 210 | `E_ALREADY_INITIALIZED` | Já inicializado | Uma vez apenas |

## `access_control` (300 – 399)

| Código | Constante | Descrição | Como tratar |
|--------|-----------|-----------|-------------|
| 300 | `E_TOKEN_NOT_FOUND` | ConsentToken inexistente | Emitir token |
| 301 | `E_TOKEN_REVOKED` | Token revogado | Emitir novo |
| 302 | `E_FIELD_REQUIRED` | Campo ausente | Validar input |
| 303 | `E_INVALID_SIGNATURE` | Assinatura inválida | Reassinar |
| 304 | `E_TIMESTAMP_TOO_OLD` | Timestamp velho | Usar `now()` |
| 305 | `E_TIMESTAMP_FUTURE` | Timestamp futuro | Conferir clock |
| 306 | `E_BEO_NOT_ACTIVE` | BEO inativo | Checar status do BEO |
| 307 | `E_IEO_NOT_ACTIVE` | IEO inativo | Checar status do IEO |
| 308 | `E_INVALID_INTENT` | Intent não cadastrado | Usar intent válido |
| 309 | `E_DUPLICATE_TOKEN` | Token duplicado | Usar token existente |
| 310 | `E_INVALID_EXPIRES` | `expires_at` inválido | Futuro ou 0 |
| 311 | `E_SCOPE_REQUIRED` | Scope vazio | Incluir ao menos 1 intent |
| 312 | `E_PERIOD_INVALID` | `period_from >= period_to` | Corrigir ordem |
| 313 | `E_NOT_AUTHORIZED` | Caller não autorizado | Institute ou holder |
| 314 | `E_NOT_INITIALIZED` | Módulo não inicializado | `initialize` primeiro |
| 315 | `E_ALREADY_INITIALIZED` | Já inicializado | Uma vez apenas |

### Códigos de resultado de `verify_token`

`verify_token` retorna `(valid: bool, reason: u8)` em vez de abortar.

| Código | Constante | Significado |
|--------|-----------|-------------|
| 0 | `VERIFY_VALID` | Token válido |
| 1 | `VERIFY_TOKEN_NOT_FOUND` | Token não encontrado |
| 2 | `VERIFY_TOKEN_MISMATCH` | `ieo_id` não bate com o token |
| 3 | `VERIFY_TOKEN_REVOKED` | Token revogado |
| 4 | `VERIFY_TOKEN_EXPIRED` | Token expirado |
| 5 | `VERIFY_INTENT_NOT_AUTHORIZED` | Intent fora do scope |
| 6 | `VERIFY_CATEGORY_NOT_AUTHORIZED` | Categoria fora do scope |
| 7 | `VERIFY_PERIOD_OUT_OF_SCOPE` | Período fora do range |
| 8 | `VERIFY_IEO_NOT_ACTIVE` | IEO está inativo |

## `governance` (400 – 499)

| Código | Constante | Descrição | Como tratar |
|--------|-----------|-----------|-------------|
| 400 | `E_NOT_INITIALIZED` | Governance não inicializada | `initialize` primeiro |
| 401 | `E_ALREADY_INITIALIZED` | Já inicializada | Uma vez apenas |
| 402 | `E_NOT_KEYHOLDER` | Caller não é keyholder | Usar keyholder ativo |
| 403 | `E_PROPOSAL_NOT_FOUND` | Proposta inexistente | Checar `proposal_id` |
| 404 | `E_PROPOSAL_EXECUTED` | Proposta já executada | Criar nova |
| 405 | `E_PROPOSAL_EXPIRED` | Prazo vencido | Criar nova |
| 406 | `E_ALREADY_APPROVED` | Já aprovada pelo caller | Nada a fazer |
| 407 | `E_INVALID_PROPOSAL_ACTION` | Ação inválida | Checar tipo |

## Como tratar abort_code no SDK / relayer

1. Capture o `vm_status` do `UserTransaction` falho.
2. Extraia `abort_code` e identifique o módulo pela origem do erro.
3. Mapeie para a constante pelo range (1–99 = BEO, 100–199 = IEO, etc.).
4. Exiba a mensagem amigável da coluna "Descrição".
5. Execute a ação sugerida em "Como tratar".

Para novos códigos, abra PR editando este arquivo junto com o módulo Move. As tabelas acima são a fonte única — a documentação em `bsp-contracts/docs/ERROR_CODES.md` apenas aponta para cá.

---

# Part IV — SDK & Client Error Surfaces

## TypeScript SDK (`bsp-sdk-typescript`)

- Location: throws generic `Error` with the protocol `code` from Part I in the `message` or a `.code` property.
- Consumers SHOULD branch on the string code, not the message text.
- For contract-level failures, the SDK surfaces the Move `abort_code` from Part III wrapped with the corresponding protocol code (e.g. `BEO_NOT_FOUND`).

## Python SDK (`bsp-sdk-python`)

- Location: raises exceptions whose `args[0]` contains the protocol code from Part I.
- On HTTP errors, the SDK includes the machine code from Part II when returned by the API.

## CLI (`bsp-cli`)

- All errors printed to stderr with the protocol `code` prefix.
- Exit codes:
  - `0` — success
  - `1` — generic error
  - `2` — validation / bad usage
  - `3` — missing config / private key
  - `4` — confirm required for destructive op

## MCP Server (`bsp-mcp`)

- Errors surfaced through MCP error channel with the protocol code as `code` and human message.
- `ConsentGuard` rejections use `CONSENT_REQUIRED`, `INTENT_NOT_AUTHORIZED`, `TOKEN_EXPIRED`, `TOKEN_REVOKED`.

---

## Cross-reference — where each surface emits these codes

| Canonical source | Location |
|---|---|
| **Contract abort codes** | `ambrosio-institute/bsp-contracts` → `src/move/sources/*.move` |
| **API machine codes** | `ambrosio-institute/bsp-registry-api` → `src/errors/catalog.ts` |
| **TypeScript SDK** | `bsp-sdk-typescript` → thrown from `src/**/*.ts` |
| **Python SDK** | `bsp-sdk-python` → raised from `bsp_sdk/**/*.py` |
| **CLI** | `bsp-cli` → `src/lib/errors.ts` |
| **MCP server** | `bsp-mcp` → `src/index.ts` (ConsentGuard + tool handlers) |

---

## Related

- **Architecture** — `docs/ARCHITECTURE.md`
- **Glossary** — `docs/GLOSSARY.md`
- **Governance** — `spec/governance.md`
