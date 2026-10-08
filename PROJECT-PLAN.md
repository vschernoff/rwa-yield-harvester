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

**Chain scope: Solana first.** The vault, the RWA position and the monitoring agent are
designed for Solana. Other execution environments are out of scope until the Solana
implementation has an operating record — at which point adding one is a portability
exercise against the same design, not a redesign.

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

## 3. Holder of record

In the embedded shape, **the integrating institution is the holder of record.** The
vault position sits in that institution's own custody, under its own keys, inside the
safeguarding arrangements it already operates. End users hold book-entry claims against
their provider — exactly as they do for every other asset in the application.

The vault is execution and accounting infrastructure. It takes custody of nothing, holds
no keys, and does not become a counterparty to the end user.

Three consequences follow:

- **No new custodian enters the picture.** This is the property that makes the structure
  reviewable: the institution's existing custody, segregation and reconciliation
  controls continue to apply unchanged, and no client asset moves outside them.
- **Eligibility is assessed at institution level.** Where the underlying RWA is
  permissioned — restricted to eligible holders in permitted jurisdictions, subject to
  KYC at the token level — the eligible holder is the institution. Its users reach the
  asset through their provider's book entry, not by qualifying individually.
- **The chain is invisible to the end user, but not to the institution.** Whoever is
  holder of record must be able to custody the position on the chain where it lives,
  directly or through a sub-custodian. For a Solana-first implementation that is a
  concrete integration requirement, and it is the reason a partner's chain coverage
  still matters even though its users never see a network.

Withdrawals resolve behind that boundary. The user takes one action; the vault meets it
from its liquidity buffer, from the secondary market, or through the issuer's redemption
process, depending on size and conditions. §6(c) describes the conditions the monitoring
agent watches in order to keep that promise honest.

## 4. Regulatory basis (EU)

This product exists because of a specific rule, and a specific gap that rule leaves open.

**The wall.** MiCA **Article 50** prohibits granting any remuneration or benefit tied to
the length of time an **e-money token** is held — and extends the prohibition to benefits
provided through third parties, so it cannot be subcontracted away. **Article 40** does
the same for asset-referenced tokens. The practical effect is that **a custodian cannot
pay yield on a stablecoin balance it holds for a client.** Coinbase switched off USDC
rewards for EEA clients on 1 December 2024 for exactly this reason.

**The gap.** The prohibition attaches to *holding*. Where the client parts with the token
and receives a position in return, the return is income on an asset placed — not a
benefit for one held. The test counsel applies is a single question:

> *Would the client receive this benefit by simply keeping the balance?*

If the answer is no, the structure sits outside the prohibition. This is the mechanism by
which a regulated institution can offer its clients a return on stablecoin balances at
all, and it is the reason the product is built as a vault position rather than as an
interest-bearing account.

**What that requires of the design.** Three constraints, architectural rather than
procedural:

- Subscription is an **exchange**, not an accrual. The client's stablecoin is exchanged
  for a vault position.
- Reporting shows **a position and its income**, never a balance earning interest.
  Statement wording is part of the architecture, not a copywriting decision.
- Valuation is taken from the vault's own share price, read from chain on a daily
  snapshot. No external valuation provider sits in the loop for the vault layer.

This is architecture, not legal advice. Each institution confirms the treatment with its
own counsel against its own authorisation, and the European Commission's MiCA review —
consultation closed 31 August 2026 — may move the line on lending.

### Precedent

The structure is not theoretical. Regulated European institutions have shipped
position-based earn products, and the useful negative case is equally well documented.

- **Coinbase — the negative case.** Ended USDC rewards for EEA customers effective
  **1 December 2024**, stating that under MiCA it was required to terminate the
  programme. Accrued rewards were paid out by 10 December. This is the clearest public
  evidence of where the wall actually stands.
- **Deblock (France)** — first MiCA CASP authorisation from the AMF (May 2025). Has run
  vaults since June 2025 accepting USDC, EURC and EURCV, curated by Steakhouse
  Financial. Deblock announced passing **$100M deposited in under a year**; the EURCV
  vault has been reported at roughly 3.7% on about €74M. The closest precedent to this
  project's shape: a licensed institution, its own clients, a position rather than a
  balance.
- **Société Générale — FORGE** — deployed its EURCV and USDCV stablecoins into onchain
  lending markets, with vaults curated by MEV Capital, and EURCV distribution
  integrations including Safe. A regulated bank, on the same structural pattern.
- **Trezor Suite** — USDC and USDT yield through curated vaults (Steakhouse Financial),
  announced May 2026. Notable as the rent-rather-than-build path, with every deposit,
  withdrawal and claim signed on the user's own device.
- **Bitpanda, Crypto.com, Gemini, Bitget** — all documented integrators of the same
  lending infrastructure for onchain earn products. Bitpanda's sits in its
  non-custodial wallet, with vaults curated by Steakhouse Financial and Gauntlet.

**What the precedent does and does not establish.** It establishes that the
position-based structure is one regulated institutions have been willing to ship, and
that at least one MiCA-authorised CASP (Deblock) and one bank (SG-FORGE) have done so in
production. It does **not** establish the asset class: every precedent above earns from
**crypto lending markets**, not from real-world assets. What carries across is the
wrapper — client parts with the token, holds a position, receives income on it — not the
yield source. And several of these products are non-custodial by design, which is a
different legal shape from a custodial institution placing client assets; the custodial
precedents are the narrower set.

Dates and figures above are as reported by the institutions and the infrastructure
provider; several are company announcements rather than independently audited, and are
recorded here as precedent rather than as benchmarks.

## 5. Phases

**Phase 0 — Discovery and diligence.** *Complete.* Issuer and venue analysis, live
on-chain verification of yields, borrow rates, collateral parameters and exit depth,
rejection of unsuitable asset categories, and the regulatory reading that determines
which wrappers a regulated distributor can actually hold.

**Phase 1 — Vault core.** Deposit, accounting, share issuance and redemption against a
single RWA position. Risk parameters and caps enforced in the program. Governance of
those parameters held by a multisig. Third-party audit before any live capital.

**Phase 2 — Monitoring agent.** The checks in §6(c), running continuously against
public chain state, with alerting into a partner's own notification channel. Read-only
by construction.

**Phase 3 — Partner integration.** The reporting, reconciliation and support surface a
regulated application needs in order to present the product to its own users under its
own brand.

**Phase 4 — Second issuer.** No single-issuer dependency before capacity is materially
expanded. Adding a second RWA source is a precondition for scale, not an enhancement.

Each phase gates the next. Leverage-bearing strategies, if they ship at all, come after
the unlevered product has an operating record.

## 6. What the architecture will be based on

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

## 7. What this project will not do

- Pay a return on a token that is itself a stablecoin, where doing so would conflict
  with the rules governing that instrument.
- Promise instant liquidity it cannot evidence.
- Ship leverage as a default setting.
- Depend on a single issuer at scale.

## 8. Research

The diligence behind this plan — issuer analyses, venue comparisons, dated on-chain
readings and redemption-terms research — is held privately and available to
counterparties under NDA. How it was conducted, and the standard applied, is described
in [RESEARCH-METHODOLOGY.md](./RESEARCH-METHODOLOGY.md).

## 9. Open questions

1. **Share transferability** — the embedded shape settles the user-facing side: end
   users hold book entries, not tokens. What remains open is whether shares held by an
   institution should be transferable between eligible holders at all, or deliberately
   non-transferable to keep the compliance surface minimal.
2. **Chain coverage at the partner** — a holder of record must custody the position on
   the chain it lives on. Which institutions can do that for Solana directly, and where
   a sub-custodian is the realistic answer.
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
