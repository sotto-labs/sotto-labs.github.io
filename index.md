---
layout: default
title: Sotto — Confidential B2B Settlement on Stellar
---

# Sotto

**Confidential B2B settlement on Stellar.**

Two businesses settle in USDC on a public ledger without publishing what they
paid each other — while their auditor, and only their auditor, sees everything.

[Contracts](https://github.com/sotto-labs/sotto-contracts) ·
[Core](https://github.com/sotto-labs/sotto-core) ·
[Console](https://github.com/sotto-labs/sotto-console)

---

## The problem

Cross-border B2B payment is the thing Stellar is commercially best at. Anchors,
SEP-24 on/off ramps, five-second finality, sub-cent fees. The rails work.

Businesses still won't use them at scale, for a reason that has nothing to do
with the rails: **a public ledger publishes your commercial relationships.**

If a manufacturer settles with its suppliers on Stellar today, anyone with a
block explorer can read:

- who its suppliers are, and in what order of importance
- what it pays per shipment, and therefore its input costs
- its total volume, and therefore its revenue within a rounding error
- when its balances dip, and therefore when it is cash-constrained

A competitor gets a live feed of your margins. In traditional banking this
information is confidential by default; on-chain it is public by default. No
treasurer signs off on that trade, no matter how good the settlement speed is.

The naive fix — hide everything — fails for the opposite reason. A business that
cannot show its auditor or its regulator what happened has traded one unusable
system for another.

**What is needed is selective confidentiality:** amounts hidden from the public,
fully visible to the parties and to a designated auditor, with disclosure
provable on demand.

---

## What Sotto is

Stellar shipped the cryptographic substrate. **Confidential Tokens** — the
OpenZeppelin contract suite wired to Nethermind's UltraHonk verifier — wrap any
SEP-41 token so that balances and transfer amounts are hidden while sender and
recipient addresses stay visible. That is precisely the right privacy shape for
B2B: you know your counterparty, the world doesn't know your numbers.

But a wrapper contract is not a settlement system.

| The primitive gives you | Sotto adds |
|---|---|
| A confidential transfer | An invoice it settles against |
| A hidden balance | A reconciled ledger position |
| One payment | Netting across many |
| An auditor view key | An auditor workflow and export |
| A proof of one transaction | A period-close disclosure pack |

**Sotto is the layer that turns a confidential transfer into a settled invoice.**

### Scope, stated honestly

Sotto does **not** invent cryptography. It does not fork the Confidential Token
contract or write new circuits. It consumes those primitives as a dependency, so
any upstream improvement is a free upgrade.

Sotto is also **not a mixer** and **not a privacy pool**. Counterparty addresses
are public by design. Sotto hides amounts, not relationships.

---

## Architecture

```
┌───────────────────────────────────────────────────────┐
│  sotto-console        Treasurer UI, auditor UI,       │  ← Sotto
│                       invoice inbox, disclosure view  │
├───────────────────────────────────────────────────────┤
│  sotto-core           Netting engine, proof service,  │  ← Sotto
│                       reconciliation, connectors      │
├───────────────────────────────────────────────────────┤
│  sotto-contracts      Invoice registry, settlement    │  ← Sotto
│                       orchestrator, policy adapter    │
├───────────────────────────────────────────────────────┤
│  Confidential Token   OpenZeppelin suite + Nethermind │  ← dependency
│  + UltraHonk verifier                                 │
├───────────────────────────────────────────────────────┤
│  Stellar base ledger  SEP-41 / SAC, BN254, BLS12-381, │  ← protocol
│                       Poseidon (P25 X-Ray / P26)      │
└───────────────────────────────────────────────────────┘
```

### Repositories

| Repo | Stack | Owns |
|---|---|---|
| [`sotto-contracts`](https://github.com/sotto-labs/sotto-contracts) | Rust / Soroban | Invoice registry, settlement orchestrator, netting attestation, policy adapter |
| [`sotto-core`](https://github.com/sotto-labs/sotto-core) | TypeScript / Node | Netting engine, proof service, reconciliation, disclosure, connectors |
| [`sotto-console`](https://github.com/sotto-labs/sotto-console) | Next.js / React | Treasurer, counterparty, and auditor views |

---

## Trust model

| Party | Can see |
|---|---|
| Public / competitor | That two addresses transacted. Not amounts. Not balances. |
| Buyer and supplier | Their own invoices, amounts, and settlement history. |
| Designated auditor | All amounts and balances for assets in the wrapper. |
| Sotto (the service) | Nothing it isn't given. No custody, no spending keys. |
| Asset issuer | Retains SAC-level freeze, cascading into the wrapper. |

The last row is deliberate. An enterprise treasurer *wants* the issuer freeze to
exist. It is a feature of this design, not a weakness.

---

## Why Stellar

- **The cryptography is native and cheap.** BN254, BLS12-381, and
  Poseidon/Poseidon2 are protocol-level host functions after X-Ray (P25) and
  Yardstick (P26). A pairing check that costs tens of millions of Wasm
  instructions elsewhere is a single host call here.
- **The compliance surface ships with the primitive.** Auditor view keys,
  selective disclosure, account-level freezing, and a configurable policy engine
  are part of Confidential Tokens rather than something Sotto has to invent and
  then convince institutions to trust.
- **The customers are already here.** Stellar's ecosystem is anchors, regulated
  payment rails, and B2B settlement companies across emerging markets. Sotto's
  users are the network's existing users.

---

## Roadmap

**Phase 1 — Foundation.** Repo scaffolding, contract interfaces, Confidential
Token integration spike on testnet, invoice registry, single bilateral
settlement end to end.

**Phase 2 — Netting.** Bilateral netting engine, netting attestation contract,
reconciliation output, treasurer console.

**Phase 3 — Compliance surface.** Auditor role and view-key workflow, selective
disclosure packs, policy adapter wired to identity registries, period-close
export.

**Phase 4 — Adoption.** Connectors for real invoice sources, multilateral
netting, design-partner pilot, mainnet readiness pending Confidential Token GA.

---

## Open questions

Stated plainly, because a project that pretends it has no unknowns is not
credible.

- Confidential Tokens are testnet-only and not mainnet-approved. Phase 4 depends
  on a timeline Sotto does not control.
- Proof generation cost and latency at realistic batch sizes is unmeasured.
- Multilateral netting across three or more parties without revealing the
  bilateral components is an open design problem.
- Soroban state archival: the invoice registry needs a TTL and rent strategy
  that survives an inactive quarter.

---

## Contributing

Sotto is open source and built in public across three repositories. Work is
scoped into self-contained tasks with acceptance criteria, named test cases, and
explicit boundaries. See `CONTRIBUTING.md` in each repo.

Contributions are coordinated through [GrantFox](https://grantfox.xyz).
