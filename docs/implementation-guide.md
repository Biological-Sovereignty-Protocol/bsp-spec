# BSP Implementation Guide

> Version 0.2 — Practical guide for building on the Biological Sovereignty Protocol.

This document is for people who want to **build**, not just read the spec. Pick your persona below and follow the steps in order.

---

## Persona 1: The User App Developer

**Goal:** Build an app where users create their BSP identity and manage their biological data.

### Step 1 — Set up bsp-id-web

Fork the reference implementation or build your own UI. The reference app lives at `bsp-id-web/`.

```bash
git clone https://github.com/Biological-Sovereignty-Protocol/bsp-id-web
cd bsp-id-web
npm install
cp .env.example .env.local
```

### Step 2 — Configure your relayer URL

The relayer is the off-chain API that bridges your app to Arweave SmartWeave contracts. Point your app at it:

```env
# .env.local
NEXT_PUBLIC_BSP_RELAYER_URL=https://api.yourdomain.com
```

For local development, you can run the relayer from `bsp-registry-api/`:

```bash
cd bsp-registry-api
npm install
npm run dev
# Runs on http://localhost:3001
```

### Step 3 — Implement BEO creation flow

A user's identity starts with a seed phrase that generates a keypair, which then creates their Biological Entity Object (BEO).

```typescript
import { BSPClient, generateKeypair } from 'bsp-sdk';

const client = new BSPClient({ relayerUrl: process.env.NEXT_PUBLIC_BSP_RELAYER_URL });

// 1. Generate seed phrase (show this to the user — they must save it)
const { mnemonic, keypair } = await generateKeypair();

// 2. Create the BEO on-chain
const beo = await client.identity.createBEO({
  publicKey: keypair.publicKey,
  metadata: {
    alias: 'user-chosen-alias',  // optional, not PII
    created: Date.now(),
  }
});

console.log('BEO ID:', beo.id); // store this locally
```

### Step 4 — Implement ConsentToken UI

Before any institution can access a user's data, the user must issue a ConsentToken. Show them what they're authorizing.

```typescript
// Display the consent request to the user
function ConsentDialog({ request }) {
  return (
    <div>
      <h2>Data Access Request</h2>
      <p>Institution: {request.ieo.name}</p>
      <p>Scope: {request.scope.join(', ')}</p>
      <p>Valid until: {new Date(request.period.end).toLocaleDateString()}</p>
      <button onClick={() => issueConsent(request)}>Authorize</button>
      <button onClick={() => denyConsent(request)}>Deny</button>
    </div>
  );
}

// Issue the token
async function issueConsent(request) {
  const token = await client.consent.issue({
    beoId: currentUser.beoId,
    ieoId: request.ieo.id,
    scope: request.scope,
    period: request.period,
    keypair: currentUser.keypair,
  });
  return token; // share this token with the institution
}
```

### Step 5 — Implement key export and backup

Users must be able to export their private key. If they lose it, their data is permanently inaccessible.

```typescript
// Export encrypted key backup
const backup = await client.identity.exportKey({
  keypair: currentUser.keypair,
  password: userEnteredPassword, // encrypt with user-chosen password
});

// Offer download
const blob = new Blob([JSON.stringify(backup)], { type: 'application/json' });
downloadFile(blob, `bsp-key-backup-${Date.now()}.json`);
```

Present this step as mandatory during onboarding, not optional.

---

## Persona 2: The Institution (IEO) Developer

**Goal:** You run a lab, clinic, or health platform and want to receive BSP-authorized health data from users.

### Step 1 — Create your IEO

An Institutional Entity Object (IEO) is your institution's on-chain identity. Create it once.

```typescript
import { BSPClient } from 'bsp-sdk';

const client = new BSPClient({ relayerUrl: 'https://api.bsp.protocol' });

const ieo = await client.institutions.createIEO({
  name: 'Acme Clinical Lab',
  type: 'laboratory',   // laboratory | clinic | platform | research
  publicKey: yourInstitutionKeypair.publicKey,
  metadata: {
    jurisdiction: 'BR',
    contact: 'tech@acmelab.com',
  }
});

console.log('IEO ID:', ieo.id); // store this — it identifies your institution
```

### Step 2 — Request consent from users

Direct users to authorize your IEO. You can deep-link into bsp-id-web or implement the consent flow in your own app.

```typescript
// Generate a consent request link
const consentUrl = client.consent.buildRequestUrl({
  ieoId: yourIeo.id,
  scope: ['BSP-HM', 'BSP-LF'],  // biomarker taxonomy category codes
  period: {
    start: Date.now(),
    end: Date.now() + 90 * 24 * 60 * 60 * 1000, // 90 days
  },
  callbackUrl: 'https://yourapp.com/bsp/callback',
});

// Send this URL to the user (email, SMS, in-app link)
// They approve in their BSP identity app, then your callback receives the ConsentToken
```

### Step 3 — Verify ConsentToken before accepting data

Never accept data submissions without verifying the token. This is not optional.

```typescript
async function handleDataSubmission(payload) {
  const { consentToken, biorecords } = payload;

  // Verify the token is valid, not expired, and covers the submitted scope
  const verification = await client.consent.verify({
    token: consentToken,
    ieoId: yourIeo.id,
    requiredScope: biorecords.map(r => r.biomarkerCode),
  });

  if (!verification.valid) {
    throw new Error(`Consent invalid: ${verification.reason}`);
  }

  // Safe to proceed
  await storeRecords(biorecords, verification.beoId);
}
```

### Step 4 — Submit BioRecords using ExchangeClient

```typescript
import { ExchangeClient } from 'bsp-sdk';

const exchange = new ExchangeClient({
  ieoId: yourIeo.id,
  keypair: yourInstitutionKeypair,
  relayerUrl: 'https://api.bsp.protocol',
});

const result = await exchange.submitRecords({
  consentToken: verifiedToken,
  records: [
    {
      biomarkerCode: 'BSP-HM-HGB',  // Hemoglobin
      value: 14.2,
      unit: 'g/dL',
      collectedAt: '2024-01-15T09:30:00Z',
      method: 'venous_blood',
    },
    {
      biomarkerCode: 'BSP-HM-HCT',  // Hematocrit
      value: 42.1,
      unit: '%',
      collectedAt: '2024-01-15T09:30:00Z',
      method: 'venous_blood',
    }
  ],
});

console.log('Transaction ID:', result.txId); // Arweave transaction — permanent
```

### Step 5 — Query authorized records

```typescript
// Fetch all records your IEO is authorized to read for a specific BEO
const records = await exchange.queryRecords({
  beoId: patientBeoId,
  scope: ['BSP-HM'],
  dateRange: {
    from: '2024-01-01',
    to: '2024-12-31',
  }
});

records.forEach(record => {
  console.log(`${record.biomarkerCode}: ${record.value} ${record.unit}`);
});
```

---

## Persona 3: The Protocol Integrator

**Goal:** Run your own BSP infrastructure — your own node/relayer for a private network or regional deployment.

### Step 1 — Deploy the SmartWeave contracts

BSP uses four SmartWeave contracts on Arweave. Deploy them in this order (each depends on the previous).

```bash
cd bsp-spec/contracts  # or obtain from the registry repository

# 1. BEORegistry — stores all biological entity identities
arweave deploy BEORegistry --wallet ./wallet.json

# 2. IEORegistry — stores all institutional identities
arweave deploy IEORegistry --wallet ./wallet.json

# 3. AccessControl — manages consent tokens and access permissions
arweave deploy AccessControl --wallet ./wallet.json \
  --init '{"beoRegistry":"<BEO_TX_ID>","ieoRegistry":"<IEO_TX_ID>"}'

# 4. DomainRegistry — manages the .bsp namespace
arweave deploy DomainRegistry --wallet ./wallet.json \
  --init '{"accessControl":"<ACCESS_CONTROL_TX_ID>"}'
```

Save all four transaction IDs. They are permanent and identify your deployment.

### Step 2 — Set up bsp-registry-api (the relayer)

The relayer is the HTTP API that your apps talk to. It handles batching, caching, and translating REST calls into SmartWeave interactions.

```bash
git clone https://github.com/Biological-Sovereignty-Protocol/bsp-registry-api
cd bsp-registry-api
npm install
```

Configure it to point at your contracts:

```env
# .env
ARWEAVE_HOST=arweave.net
ARWEAVE_PORT=443
ARWEAVE_PROTOCOL=https
ARWEAVE_WALLET_PATH=./wallet.json

BSP_BEO_REGISTRY_TX=<YOUR_BEO_TX_ID>
BSP_IEO_REGISTRY_TX=<YOUR_IEO_TX_ID>
BSP_ACCESS_CONTROL_TX=<YOUR_ACCESS_CONTROL_TX_ID>
BSP_DOMAIN_REGISTRY_TX=<YOUR_DOMAIN_REGISTRY_TX_ID>

PORT=3001
```

Start the relayer:

```bash
npm run build
npm start
# or with PM2: pm2 start ecosystem.config.js
```

### Step 3 — Configure authorized callers

Restrict which origins and API keys can write to your relayer.

```env
# .env — add these
ALLOWED_ORIGINS=https://yourapp.com,https://youradminpanel.com
API_KEYS=key_abc123,key_def456   # generate strong random keys
RATE_LIMIT_WRITES=100            # write requests per minute per key
RATE_LIMIT_READS=1000            # read requests per minute per key
```

For production, put the relayer behind a reverse proxy (nginx, Caddy) with TLS.

### Step 4 — Point bsp-id-web at your relayer

Any BSP-compatible app can point at your relayer by setting the relayer URL:

```env
# In bsp-id-web or any SDK consumer
NEXT_PUBLIC_BSP_RELAYER_URL=https://relayer.yourdomain.com
```

Apps and SDKs are relayer-agnostic. The same `bsp-sdk` code works against any compliant relayer.

---

## Common Pitfalls

### 1. Losing the private key

The user's keypair is the only access credential. There is no password reset, no recovery email, no support ticket that can restore it. Make key backup a hard requirement during onboarding, not a suggestion.

### 2. Skipping ConsentToken verification

If your institution accepts data without verifying the token, you have no proof of consent. This is a compliance risk and a BSP protocol violation. Always call `client.consent.verify()` before processing submissions.

### 3. Using mutable metadata for identity

BEO and IEO records on Arweave are permanent. Do not store PII (name, date of birth, email) in the on-chain object. Store only the public key and a stable alias. Keep sensitive metadata in your own database, linked by BEO ID.

### 4. Deploying contracts in the wrong order

The Consent Registry needs the BEO and IEO registry transaction IDs at deployment time. The BioRecord Index needs the Consent Registry TX ID. Deploy in sequence: BEO → IEO → Consent → Index.

### 5. Scope creep in ConsentTokens

Request only the biomarker taxonomy codes you actually need. Users see the scope during the consent dialog. Requesting `BSP.*` (all data) will cause users to decline. Request the minimum scope for your use case.

### 6. Not handling Arweave latency

Arweave transactions are not instant. After submitting a record, the `txId` is returned immediately but the record may take 10-20 minutes to be fully confirmed. Design your UX to show a "pending" state rather than assuming immediate availability.

---

*Questions or corrections? Open an issue or PR. This guide tracks the spec at version 0.2.*
