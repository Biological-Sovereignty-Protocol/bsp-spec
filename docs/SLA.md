# BSP Service Level Agreement

Public SLA for the Biological Sovereignty Protocol (BSP) reference services operated by the Ambrosio Institute. Defines the commitments on availability, latency, and communication between the Institute and BSP users (BEOs, Guardians, Keyholders, integrators).

> **Version**: 1.0 — published 2026-04-20.
> **Scope**: applies to production (`mainnet`) services only. Testnet environments are best-effort with no SLA.

## 1. Services Covered

| Service | Endpoint | Notes |
|---------|----------|-------|
| Registry API | `https://registry.biologicalsovereigntyprotocol.com` | Core read/write API for BEO identity, tokens, biorecords. |
| Identity Web | `https://id.biologicalsovereigntyprotocol.com` | User-facing BSP ID management. |
| Status Page | `https://status.biologicalsovereigntyprotocol.com` | Incident and maintenance communication. |
| Contracts | Aptos mainnet package at `<bsp.aptos>` | On-chain state is final and permanent by protocol design. |

External dependencies **not under our control** (best-effort only):
- Aptos mainnet availability (Aptos Foundation SLA).
- Arweave network availability.
- `arweave.net` gateway availability.

## 2. Availability Commitment

**99.5% monthly uptime** on the Registry API, measured as:

```
uptime = (total_minutes_in_month - downtime_minutes) / total_minutes_in_month
```

Downtime = cumulative minutes where synthetic probes against `/health` fail from at least 2 of 3 geographic monitoring regions.

Excluded from downtime:
- Scheduled maintenance announced > 72 hours in advance (up to 4 hours per month).
- Outage caused by external dependencies listed above when confirmed by their status pages.
- Force majeure (natural disaster, state action, internet-scale routing failure).
- Abuse mitigation or security incident response where exclusion is necessary to protect user data.

### 2.1 Service Credits

Not applicable at this phase (pre-commercial). The Institute publishes monthly uptime reports as a transparency commitment. Commercial SLA with credits is available to Institute Partner tier subscribers (out of scope of this document).

## 3. Latency Commitments

Measured on the Registry API at the 95th percentile over a 5-minute sliding window, from the same region as the operator.

| Operation Class | Examples | P95 Target |
|-----------------|----------|------------|
| **Read** (cached / no Aptos call) | `GET /v1/verify/:token`, `GET /v1/beo/:id/public` | **< 500 ms** |
| **Read** (on-chain, cache miss) | `GET /v1/beo/:id` first call | < 1.5 s |
| **Write** (Aptos tx required) | `POST /v1/tokens/issue`, `POST /v1/beo/rotate_key`, `POST /v1/biorecord/create` | **< 3 s** (includes Aptos finality) |
| **Write** (Arweave + Aptos) | Biorecord with payload upload | < 8 s |
| **Admin / governance** | Institute-only endpoints | Best-effort |

Latency SLA is suspended during an active dependency incident (Aptos or Arweave) provided such incident is declared on our status page.

## 4. Maintenance Windows

- **Preferred window**: Sunday 02:00–06:00 UTC (low global traffic).
- **Announcement**: at least 72 hours in advance via status page + email to registered integrators.
- **Budget**: up to 4 hours of scheduled downtime per calendar month, excluded from the uptime calculation.
- **Emergency maintenance**: may occur without advance notice when a security or data-integrity issue is being mitigated. Announced in real time on status page.

## 5. Communication Channels

| Purpose | Channel |
|---------|---------|
| Real-time service status | https://status.biologicalsovereigntyprotocol.com |
| Incident reports (post-mortem) | https://biologicalsovereigntyprotocol.com/incidents |
| Release notes | https://biologicalsovereigntyprotocol.com/releases |
| Security advisories | https://biologicalsovereigntyprotocol.com/security |
| Operator contact | `ops@biologicalsovereigntyprotocol.com` |
| Security vulnerability reports | `security@biologicalsovereigntyprotocol.com` (PGP key on SECURITY.md) |

> **Status page placeholder**: the `status.*` subdomain is currently being provisioned. During the interim, incidents are announced at the main domain `/status` path and cross-posted to the Ambrosio Institute GitHub org announcements.

## 6. Incident Classification (as exposed to users)

Matches our internal runbooks. User-visible severities:

| Level | Definition | Example user impact |
|-------|------------|---------------------|
| **Outage** | Service unavailable. | Cannot authenticate, cannot issue tokens. |
| **Degraded** | Slower than SLA or partial feature failure. | Biorecord uploads slow; read paths OK. |
| **Informational** | Maintenance, upcoming change, advisory. | Scheduled upgrade at T+48h. |

Each incident receives a dedicated status-page entry with timestamped updates. Post-mortems for "Outage" incidents are published within 7 calendar days.

## 7. Data Integrity Guarantees

BSP's design makes several properties protocol-level (not dependent on this SLA):

- **On-chain state** (Aptos): append-only, final after Aptos finality. No loss possible while Aptos operates.
- **Biorecord payloads** (Arweave): immutable once confirmed. Hash proofs allow third-party verification.
- **User keys**: held by users (hardware wallets or secure enclave); the Institute cannot access or move funds.
- **No silent mutation**: every state change produces a verifiable on-chain event.

The Registry API is a thin relay layer over these guarantees. Even if the API is entirely destroyed, user data remains accessible via direct Aptos and Arweave queries.

## 8. Changes to This SLA

- Material changes are announced ≥ 30 days in advance via the release notes channel.
- Minor clarifications may be applied retroactively; diff is visible via Git history of this file.
- Current version is always at `bsp-spec/docs/SLA.md` on the `main` branch.

## 9. Reporting a Breach

If you believe the Institute has breached this SLA:
1. Email `ops@biologicalsovereigntyprotocol.com` with subject `SLA BREACH — <date>`.
2. Include: timeframe, affected endpoints, evidence (probe logs, screenshots), impact on you.
3. Acknowledgement within 2 business days. Resolution within 10 business days.

The Institute commits to publishing a transparency report each quarter summarizing SLA performance, incidents, and resolutions.
