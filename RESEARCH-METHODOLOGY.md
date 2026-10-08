# Research Methodology

The project plan rests on diligence that is **not published here**. This note describes
how that work was done and the standard it was held to, so a reader can judge whether
the plan is grounded — without the research itself becoming public.

## Principles

**Primary sources only.** Figures are read directly from program APIs and on-chain
account state, and from issuer documentation. Aggregators and dashboards are used to
*discover* candidates; nothing from them is treated as true until verified at source.
Marketing pages are never a source of a number.

**Everything is dated.** Every figure carries the date it was read. Anything more than
about two weeks old is treated as stale and re-read before it is used in a
counterparty-facing document. Rates, utilisation and depth all move; a plan that
depends on a particular level is a plan with a short shelf life.

**Nothing depends on a level.** Where the design needs a number — a minimum spread, a
depth threshold, a utilisation ceiling — it is a *parameter*, not an assumption baked
into the architecture.

**Falsification first.** Each diligence pass ends with an explicit list of claims that
could **not** be verified. Those claims are excluded or labelled unverified; they are
never softened into the narrative. Several otherwise attractive candidates were
rejected on the strength of that list alone.

## What is verified about a candidate RWA

- **Issuer** — the legal entity, its regulator and licence, and whether assets are
  genuinely segregated from the operator's own balance sheet.
- **Yield source** — the actual cash flow that produces the return, and what it
  depends on. A return whose source cannot be described in one sentence is a return
  that has not been understood.
- **Collateral** — what backs the position, including any crypto-native component
  whose risk may be correlated with the rest of a user's portfolio.
- **Valuation** — who publishes net asset value, on what cadence, who attests to it
  independently, and how it reaches the chain.
- **Redemption** — primary mechanics in detail: windows, periodic caps, eligibility
  gates, fees, whether price is struck at request or execution, and what happens to an
  unfilled request. This is the most commonly misunderstood property of an RWA.
- **Secondary liquidity** — quoted price impact at realistic exit sizes against live
  markets. Pool headline size is not depth, and the difference between the two has
  decided more than one go/no-go here.
- **Audits** — read for *scope*. A smart-contract or penetration-test audit says
  nothing about solvency; a reserve attestation says nothing about code. Conflating
  the two is the most common error in this asset class.
- **Track record** — length of the published series, and whether its smoothness is
  plausible for the underlying business.

## What is verified about an execution venue

Supply and borrow rates with the utilisation that produces them; free liquidity and
caps; collateral parameters and how enforcement actually works; oracle providers,
staleness behaviour and cross-provider divergence; and the powers governance or a
curator holds over parameters — including the ability to socialise losses onto
depositors.

## Category screening

Whole asset categories are screened out before any issuer-level work begins. Two were
rejected during this project and are named in the plan, on regulatory-perimeter and
credit-risk grounds respectively. Screening at the category level is cheaper than
discovering the same problem one issuer at a time.

## What is held privately

The dated datasets, the per-issuer analyses, the venue comparisons, and all
partner-specific and commercial material. This is the substantive asset of the project,
and publishing it would hand over the integration map alongside it.

It is available to counterparties under NDA during evaluation.

## The standard

Any figure that reaches a counterparty is a figure read at source and dated. Anything
that cannot be verified is either stated as unverified or left out entirely — including
when leaving it out makes the product look less impressive than a competitor willing to
quote it.

---

© 2026 Vitaliy Chernov. All rights reserved.
