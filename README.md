# RWA Yield Harvester

A yield product that brings **regulated real-world-asset (RWA) returns** to users of
custodial crypto applications — without those users ever touching a wallet, a bridge,
or a protocol interface.

The product has two halves:

- **A vault** that holds a position in a regulated, real-world-asset-backed token and
  handles entry, exit, accrual and distribution on behalf of depositors.
- **A monitoring agent** that watches the position continuously and raises warnings —
  while holding no keys, submitting no transactions, and making no discretionary
  decisions over anyone's funds.

First implementation targets **Solana**.

## Status

Pre-build. Discovery and issuer diligence are complete; no contracts are deployed and
no capital is live. The current artefact is the project plan.

- 📄 [Project plan](./PROJECT-PLAN.md)
- 🔬 [Research methodology](./RESEARCH-METHODOLOGY.md) — how the diligence behind the
  plan was done. The research itself is held privately and available under NDA.

## Why this exists

Custodial crypto apps can offer their users staking on the coins they already list.
They generally cannot offer anything whose return comes from outside crypto — and the
tokenised real-world assets that do pay such a return are, in practice, out of retail
reach: permissioned, KYC-gated, jurisdiction-restricted, or simply impossible to reach
without self-custody and several chains of plumbing.

This project closes that gap at the infrastructure layer, so a regulated application
can present real-world yield as one more line in a product its users already understand.

## Principles

1. **Never custody user keys.** The agent observes and notifies; it never signs.
2. **Say what the yield is made of.** A named issuer and a documented cash flow, or it
   does not ship.
3. **Honest liquidity.** Capacity is sized to verified exit depth, not to ambition.
4. **No hidden leverage.** Any borrowing is explicit, bounded, and opt-in.

---

© 2026 Vitaliy Chernov. All rights reserved.
