# Contributing to BSP

Thank you for considering a contribution to the Biological Sovereignty Protocol (BSP).

BSP is a cryptographic standard for biological data ownership. Changes to the protocol have consequences for every system that speaks BSP — so we move carefully, review in public, and document everything.

## Ways to contribute

- **Propose a protocol change** via a BIP (BSP Improvement Proposal).
- **File a bug** in the specification, examples, or reference implementations.
- **Improve examples, taxonomy, or documentation** via a regular PR.
- **Join a working group** (Taxonomy, Schema, Security, Ecosystem) — open an issue with the label `wg-interest`.

## Proposing a change (BIP process)

Any material change to the specification goes through the BIP process.

### 1. Fork

Fork `bsp-spec` on GitHub.

### 2. Draft

Copy the template:

```bash
cp bip/BIP-0000-template.md bip/BIP-XXXX-your-title.md
```

Fill in every field. BIP types:

- **BIP-T** — Taxonomy changes (biomarker codes, reference ranges).
- **BIP-P** — Protocol / schema changes (BEO, IEO, BioRecord, Exchange).
- **BIP-G** — Governance changes (multisig, process, charter).
- **BIP-I** — Informational (best practices, position statements).

### 3. Pull request

Open a PR against `main`. The PR opens a public comment period:

| Type | Comment period |
|---|---|
| BIP-T | 30 days |
| BIP-P | 90 days |
| BIP-G | 120 days |

### 4. Review

- Working group reviews the technical content.
- Community comments on the PR.
- For BIP-T and BIP-P, at least 2 peer-reviewed references are required.
- Breaking changes require a strong rationale and a migration path.

### 5. Decision

- **Accepted** — merged to `main`, status set to `FINAL`, version bumped.
- **Rejected** — kept in repo with status `REJECTED` and rationale.
- **Withdrawn** — author withdraws, status set to `WITHDRAWN`.

Acceptance of BIP-P and BIP-G changes requires signature from the governance multisig (2-of-3). See [BIP-0001](bip/BIP-0001.md).

## Code of Conduct

All contributors are expected to follow the [Contributor Covenant Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/). Report violations to `conduct@biologicalsovereigntyprotocol.com`.

## Running the spec tests

The specification ships with validation scripts that check schemas, examples, and taxonomy consistency.

```bash
# From repo root
npm install
npm test
```

The test suite validates:

- BEO / IEO / BioRecord JSON schemas
- Taxonomy YAML files against the taxonomy meta-schema
- All `examples/*.json` files against their declared schema
- Cross-references between BIPs and the spec

CI runs on every PR. PRs cannot merge with failing tests.

## Style

- Markdown: ATX headings (`#`), fenced code blocks with language tags.
- Prose: short sentences, present tense, no filler.
- Tables preferred to nested lists for reference material.
- English (US) for the canonical spec. Translations live in the website repo.

## Security

Do not open public issues for security vulnerabilities. See [SECURITY.md](SECURITY.md) for the private disclosure process.

## License

All contributions are MIT-licensed. By opening a PR you agree to license your contribution under the repository license.
