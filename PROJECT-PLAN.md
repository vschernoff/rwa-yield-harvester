# RWA Yield Harvester — Project Plan

**Version:** 0.1 · **Date:** October 2026 · **Status:** pre-build

This is a high-level plan. It states what will be built, in what order, and what the
design will be based on. It deliberately omits detailed architecture, interface
specifications and delivery estimates — those belong in design documents written once
the open questions below are closed.

---

## 1. What is being built

A vault on Solana that holds a regulated, real-world-asset-backed token on behalf of
depositors, plus a monitoring agent that watches the position and warns about
conditions that change what a depositor can expect.

Two distribution shapes are in scope, in this order:

1. **Embedded** — the vault sits behind a regulated application. Its users see a
   familiar earn product in their own app; the application keeps custody and the
   customer relationship. Users never see a wallet, a chain or a protocol name.
2. **Direct** — self-custody users interact with the same vault themselves.

The embedded shape comes first because it is the one with a distribution path.

## 2. The yield source

The first asset class is **reinsurance-backed RWA**: tokens whose return originates in
insurance premium income and the yield on the collateral float, rather than in crypto
lending, token emissions or trading.

Two properties make this class suitable where tokenised equities and high-yield
lending platforms are not: the issuer is a named, regulated entity publishing a net
asset value, and the return has no structural dependence on crypto prices rising.

Two categories were evaluated and explicitly ruled out:

- **Tokenised equities** — a tokenised share remains a transferable security and
  therefore a financial instrument under MiFID II. A crypto-asset authorisation does
  not permit placing or distributing one, which puts the asset outside the perimeter
  of the regulated applications this product is built for.
- **High-yield centralised lending** — the spread over market rates is payment for
  unsecured credit risk against an operator, usually without an audited loan book.
  Structurally the same shape as the 2022 failures.

## 3. Phases

**Phase 0 — Discovery and diligence.** *Complete.* Issuer and venue analysis, live
on-chain verification of yields, borrow rates, collateral parameters and exit depth,
rejection of unsuitable asset categories, and the regulatory reading that determines
which wrappers a regulated distributor can actually hold.

**Phase 1 — Vault core.** Deposit, accounting, share issuance and redemption against a
single RWA position. Risk parameters and caps enforced in the program. Governance of
those parameters held by a multisig. Third-party audit before any live capital.

**Phase 2 — Monitoring agent.** The checks in §4(c), running continuously against
public chain state, with alerting into a partner's own notification channel. Read-only
by construction.

**Phase 3 — Partner integration.** The reporting, reconciliation and support surface a
regulated application needs in order to present the product to its own users under its
own brand.

**Phase 4 — Second issuer.** No single-issuer dependency before capacity is materially
expanded. Adding a second RWA source is a precondition for scale, not an enhancement.

Each phase gates the next. Leverage-bearing strategies, if they ship at all, come after
the unlevered product has an operating record.

## 4. What the architecture will be based on

### (a) Token format

**Underlying position.** The RWA tokens targeted on Solana are standard SPL tokens that
accrue value by **exchange rate** rather than by rebasing — the holder's balance is
constant and the redemption value per unit rises. This matters more than it sounds:
exchange-rate accrual means integrating venues price the asset through an oracle rather
than tracking balance changes, and it makes vault share accounting a straightforward
ratio rather than a rebase reconciliation.

**Vault shares.** Issued under **Token-2022** (Token Extensions) rather than the legacy
SPL token program, for three reasons:

- *Metadata* — share tokens carry their own on-chain metadata without a side registry.
- *Transfer control* — where a distributor's compliance perimeter requires it, the
  extensions provide the means to constrain who may hold a share: default account
  state, allowlisting, and transfer hooks for programmatic checks at transfer time.
- *Optional non-transferability* — in the embedded shape the end user holds a book
  entry with their provider, not a token in a wallet. A share that is deliberately
  non-transferable is a legitimate configuration, and a simpler compliance story.

Whether shares are transferable at all is a product decision, not a technical one, and
it is listed among the open questions.

### (b) Vault standard

**There is no ratified Solana equivalent of ERC-4626.** The design will not pretend
otherwise, and will not invent a private interface where a public one exists. Three
reference points shape the approach:

- **Solana Vault Standard (SVS)** — a community, Anchor-based implementation of
  ERC-4626-style tokenised vaults: SPL deposits, Token-2022 shares, and the same core
  deposit/mint/withdraw/redeem semantics. It also covers tranched RWA vaults.
- **ERC-7540 (asynchronous deposit and redemption)**, which SVS also reflects. This is
  the single most important borrowing from the EVM world for this product. RWA
  redemption is *not* synchronous: the issuer meets redemptions in windows, subject to
  caps and to collateral actually being released. A vault that models redemption as a
  **request** followed later by a **claim** describes reality; one that models it as an
  instant swap will mislead its users at exactly the wrong moment.
- **sRFC 37 (Token ACL)** and the canonical-vault work the Solana Foundation has
  sponsored — relevant because permissioned RWAs need token-level access control, and
  because aligning with an emerging standard is cheaper than diverging from it.

The working assumption is ERC-4626-equivalent semantics for the synchronous path,
ERC-7540-style request/claim semantics for the redemption path, and Token-2022 shares
throughout.

### (c) What the monitoring agent verifies on chain

The agent exists because the honest risks in this product are **liquidity and
counterparty conditions**, not price volatility — and those conditions are observable
on chain before they become visible to a user.

**Exit conditions — the headline check.**

- *Approaching the queue threshold.* The issuer's redemption capacity is capped per
  period. The agent tracks outstanding redemption demand against that cap and warns as
  the vault approaches the point at which a withdrawal **stops being immediate and
  starts queueing**. This is the single most user-visible state change in the product.
- *Secondary-market depth.* Continuous quoting of realistic exit sizes against the
  on-chain markets for the RWA token, alerting when the price impact of a given clip
  crosses a threshold, and force-reducing exposure before depth is exhausted rather
  than after.
- *Buffer adequacy.* The unlevered reserve held for ordinary withdrawals, measured
  against recent outflow, so the buffer is refilled on a schedule rather than in a panic.

**Issuer and valuation conditions.**

- *NAV freshness* — alert when the issuer's published value has not been refreshed
  within its expected cadence.
- *NAV plausibility* — alert on abnormal period-over-period moves in the reported rate,
  and on implausibly smooth reported performance, which is a sign of a modelled rather
  than a marked value.
- *Market price versus NAV* — divergence between the secondary price and the issuer's
  published value is the earliest public signal that the market disagrees with the
  issuer.

**Venue conditions** (relevant wherever the vault borrows or supplies).

- *Utilisation and free liquidity* on the borrow venue, with escalating thresholds —
  because a market past its rate kink can turn a positive carry negative without any
  price moving.
- *Rate-of-change on borrow cost*, not just its level.
- *Carry adequacy* — the spread between what the RWA position earns and what the
  borrow costs, measured against the minimum spread at which a leveraged strategy is
  permitted to be open at all. Below it, the strategy stays shut.

**Protocol and oracle conditions.**

- Oracle staleness, confidence intervals, and divergence between providers.
- Collateral parameter changes, curator actions and loss-socialisation events on any
  venue holding vault assets.
- Collateralisation headroom where borrowing is active, with banded warnings well
  before any protocol-level enforcement.

**Hard boundaries.** The agent reads public state and emits notifications. It holds no
keys, signs nothing, moves no funds, and gives no personalised recommendation. Any
action it suggests is taken by the vault's own governed logic or by a human.

## 5. What this project will not do

- Pay a return on a token that is itself a stablecoin, where doing so would conflict
  with the rules governing that instrument.
- Promise instant liquidity it cannot evidence.
- Ship leverage as a default setting.
- Depend on a single issuer at scale.

## 6. Open questions

1. **Share transferability** — transferable Token-2022 shares, or non-transferable
   book-entry only? Determines the compliance surface and the integration shape.
2. **Holder of record** — does an integrating application custody the RWA position
   itself, or hold a claim on the vault? This is as much a safeguarding question as a
   technical one, and it determines which chain even matters to a partner.
3. **Issuer eligibility** — the permitted-jurisdiction and KYC constraints attached to
   each candidate RWA token, and whether a vault entity can be the holder of record on
   behalf of downstream users.
4. **Second issuer** — which asset joins the first, and whether it sits on the same chain.
5. **Governance** — composition and powers of the multisig controlling risk parameters.

---

*Verified inputs behind this plan — yields, borrow rates, collateral parameters, exit
depth and redemption terms — were read directly from public chain state and issuer
documentation during August and September 2026, and are dated in the underlying
research. Rates move; the plan does not depend on any particular level.*

© 2026 Vitaliy Chernov. All rights reserved.
