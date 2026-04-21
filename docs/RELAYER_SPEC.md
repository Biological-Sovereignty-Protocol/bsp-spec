# BSP Relayer Specification

> Version 0.2 — April 2026
> Ambrósio Institute

---

## Purpose

A BSP Relayer is an **optional infrastructure component** that submits Aptos transactions on behalf of users who do not hold APT. Relayers are **infrastructure, not authority**. The BSP protocol operates independently of any specific relayer — users can interact directly with Move contracts if they choose.

The relayer layer exists as a convenience for gasless onboarding and for integrators who prefer not to manage wallets. It carries no privileged role in the protocol.

---

## Non-Normative Role

Relayers MUST NOT:

- Modify payloads signed by users
- Forge consent tokens or BEO operations
- Retain user private keys
- Block BEO portability between relayers

Relayers MAY:

- Refuse to relay any transaction (economic or policy reasons)
- Set their own rate limits, authentication policies, or eligibility criteria
- Charge fees in their own billing system (off-chain)
- Implement additional anti-abuse measures (captchas, proofs, KYC, etc.)

---

## Minimum Interface

Any relayer implementation SHOULD expose the following endpoints. Additional endpoints and authentication layers are permitted.

### POST /relayer/submit

Accepts a signed payload, relays to Aptos, returns transaction result.

**Request:**

```json
{
  "operation": "create_beo" | "submit_biorecord" | "grant_consent" | "...",
  "payload": { "...": "signed payload" },
  "signature": "0x...",
  "publicKey": "0x...",
  "nonce": "unique_per_operation",
  "timestamp": 1234567890
}
```

**Response (success):**

```json
{
  "tx_hash": "0x...",
  "block_height": 12345,
  "gas_used": 100
}
```

**Response (refusal):** HTTP `403`, `429`, or `503` with a reason code.

### GET /relayer/status

Returns relayer health and policy info.

```json
{
  "operator": "ambrosio-institute",
  "accepts": ["create_beo", "submit_biorecord"],
  "policies": {
    "requires_api_key": true,
    "rate_limit_per_key": "60/hour",
    "captcha": false
  },
  "aptos_network": "mainnet",
  "available": true
}
```

---

## Trust Model

A compromised or malicious relayer can:

- **Refuse service** — users migrate to another relayer
- **Delay transactions** — users retry elsewhere
- **Log request metadata** — users who want privacy should choose relayers accordingly

A compromised relayer CANNOT:

- Forge signatures (Ed25519 verified on-chain)
- Modify signed payloads (rejected by Move contract)
- Steal private keys (never transmitted)
- Destroy or censor on-chain state (Aptos consensus protects this)

See `docs/THREAT_MODEL.md` — section "Compromised Relayer" — for the full adversary analysis.

---

## Running Your Own Relayer

The reference implementation is open-source at [`github.com/Ambrosio-Institute/bsp-registry-api`](https://github.com/Biological-Sovereignty-Protocol/bsp-registry-api). To operate an independent relayer:

1. Deploy the reference implementation (or build your own conforming to this spec)
2. Fund an Aptos wallet to pay gas for users
3. Optionally fund an Arweave wallet for BioRecord storage
4. Configure your anti-abuse policies (rate limits, authentication, etc.)
5. Publish your `/relayer/status` endpoint
6. Announce your relayer via your own channels — there is no central registry

BEOs created through any conforming relayer are **fully portable**. A user whose BEO was created via Relayer A can later have operations submitted via Relayer B by signing a new payload — the protocol does not distinguish.

---

## Multi-Relayer Future

The BSP protocol is designed for a diverse relayer ecosystem:

| Type | Description |
|---|---|
| **Institutional** | Operated by hospitals, research orgs, ministries. May require institutional credentials. |
| **Commercial** | Charge users or third parties in fiat/stablecoin for premium service. |
| **Community** | Free or donation-funded, often region-specific. |
| **Self-relay** | Advanced users running their own node for maximum sovereignty. |

No relayer is privileged. The Ambrósio Institute operates a reference relayer as a public good during the protocol's early phase, but it is not the canonical relayer and carries no special authority in the Move contracts.

---

## Economic Sustainability

Relayers bear operational cost (Aptos gas, Arweave storage, infrastructure). Sustainable models include:

- **Subsidy** — operator absorbs cost (common for early adoption or institutional deployment)
- **Metered billing** — operator charges users or integrators off-chain
- **Self-relay** — user provides their own wallet; operator only facilitates
- **Treasury grants** — funded by protocol treasury or community DAO

The protocol does not enforce any specific economic model. This is intentional — relayer economics are a market layer, not a protocol layer.

---

## Reference Implementation Policies (Ambrósio Institute)

The Ambrósio Institute's reference relayer (`registry.bsp.protocol`) currently operates under these policies:

- `POST /beo`: requires `Authorization: Bearer <INSTITUTE_API_KEY>`
- Rate limit: 60 BEO creations per hour per API key
- Daily APT spend cap: 10 APT (configurable)
- Free tier for verified institutional partners

These are **operational decisions of this specific relayer**, not protocol rules. Other relayers SHOULD publish their own policy documents.

---

## Related

- **Architecture** — `docs/ARCHITECTURE.md`
- **Threat Model** — `docs/THREAT_MODEL.md`
- **Intents Catalog** — `docs/INTENTS_CATALOG.md`
- **Governance** — `spec/governance.md`
- **Implementation Guide** — `docs/implementation-guide.md`
