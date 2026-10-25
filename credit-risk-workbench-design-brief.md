# Corporate Credit Risk and P&L Workbench

## Product design and implementation brief

| Document field | Value |
| --- | --- |
| Version | 1.0 |
| Date | 25 October 2026 |
| Audience | Product design, trading, quantitative research, risk and engineering |
| Status | Design handoff; proposed implementation defaults require desk confirmation |
| Portfolio context | Long–short high-yield corporate bonds with ETFs used to manage excess broad-market beta |
| Primary objective | Make market movements beyond broad beta observable, connect them to portfolio exposures, and explain their P&L and risk implications |

## Contents

1. [Product purpose and scope](#1-product-purpose-and-scope)
2. [Core concepts and design principles](#2-core-concepts-and-design-principles)
3. [User workflow and information architecture](#3-user-workflow-and-information-architecture)
4. [Screen requirements](#4-screen-requirements)
5. [Progressive delivery plan](#5-progressive-delivery-plan)
6. [Reference basket specification](#6-reference-basket-specification)
7. [Factor catalogue and promotion rules](#7-factor-catalogue-and-promotion-rules)
8. [Calculation and attribution contracts](#8-calculation-and-attribution-contracts)
9. [Data model and provenance](#9-data-model-and-provenance)
10. [Quality controls and acceptance scenarios](#10-quality-controls-and-acceptance-scenarios)
11. [Operating process and implementation architecture](#11-operating-process-and-implementation-architecture)
12. [Initial backlog and release checklist](#12-initial-backlog-and-release-checklist)
13. [Decisions to resolve](#13-decisions-to-resolve)
14. [Glossary](#14-glossary)
15. [Supporting references](#15-supporting-references)

---

## 1. Product purpose and scope

### 1.1 Problem

The book can have limited broad HY beta after ETF hedging while retaining substantial exposure to quality decompression, sector rotation, credit-curve changes, liquidity differences, ETF basis and issuer-specific moves.

A single beta number does not explain these behaviours. The desk needs observable reference baskets and market measures, a consistent mapping to the actual positions, and a daily explanation that reconciles to official P&L.

### 1.2 Intended outcome

The workbench should support a statement such as:

> The ETF hedge offset most of the common credit move. CCC credit weakened relative to B, and telecom widened relative to comparable credit. The cash longs carried both exposures; the ETF shorts offset part of them. Separately, three issuers accounted for most of the residual loss.

This is an illustrative explanation, not a finding about the actual book.

The product must answer four questions:

1. **Market:** What moved outside the portfolio?
2. **P&L:** How did those movements affect the positions held during the period?
3. **Exposure:** What risks does the current book carry?
4. **Scenarios:** What would happen under a defined continuation or reversal of those moves?

### 1.3 Confirmed scope

- Descriptive market-risk analysis, P&L decomposition and risk attribution.
- Cash corporate bonds and ETF hedges in one consistent exposure framework.
- Custom reference baskets are available as a way to observe market movements.
- Incremental delivery: every layer must provide a usable output.
- Detailed drilldown from market movement to factor, issuer and bond.

### 1.4 Scope boundaries

- No alpha forecasts, security-selection scores or buy/sell signals.
- Residual P&L must not be labelled alpha.
- No automated trade execution.
- Hedge comparison may be added after measurement and attribution are reliable.
- Generic dispersion is a market-state diagnostic; it does not automatically receive a signed P&L contribution.

### 1.5 Proposed initial assumptions

These are implementation defaults, not confirmed facts about the desk:

- End-of-day reporting first, with trade timestamps retained for correct P&L.
- One currency and reference-curve convention in the first model; additional currencies require explicit rates, FX and regional credit treatment.
- Existing official P&L and vendor analytics are used where available.
- A proportional-spread/DTS convention provides the initial common-credit baseline.
- Historical definitions, prices, weights and model versions are preserved.
- Callable, distressed and event-driven instruments remain visible and receive explicit exception treatment where ordinary spread approximations are inadequate.

## 2. Core concepts and design principles

### 2.1 Three distinct objects

| Object | Purpose | Examples | Treatment |
| --- | --- | --- | --- |
| Market indicator | Describe a directly observed or reproducibly constructed market series | OAS, rating gaps and ratios, sector basket returns | Available for monitoring once data quality is adequate |
| Attribution factor | Define a directional market movement and its mapping to securities | Common proportional widening, controlled quality contrast, conditional sector move | May contribute to additive P&L and scenario calculations |
| Risk-state diagnostic | Describe the distribution or reliability of market outcomes | Residual dispersion, tail breadth, quote staleness, rolling correlation | Informs risk interpretation and model review; no automatic directional P&L |

One indicator can become an attribution factor after its shock, mapping and overlap controls are defined. Not every indicator needs to become a factor.

### 2.2 Persistent product principles

1. **Market measurements are independent of the current book.** Reference baskets use an external universe. The portfolio is overlaid on those measurements.
2. **Every number has a unit, sign and time basis.** A widening-loss exposure is different from a positive P&L sensitivity.
3. **Raw and controlled movements remain visible.** Users can inspect how a raw sector move becomes a sector-relative attribution factor.
4. **Actual P&L remains the accounting reference.** More factors change the explanation, not actual profit or loss.
5. **Cash and ETF exposures are comparable.** Show cash longs, cash shorts, ETF hedges and net risk separately.
6. **Missing data is not zero.** Unavailable or stale readings have distinct visual states.
7. **Attribution conventions are versioned.** Historical comparisons use compatible definitions.
8. **Measured risk and hedge availability are separate.** A missing sector ETF does not remove the sector from the model.
9. **The interface supports investigation.** Every aggregate can be traced to the securities, marks and weights behind it.
10. **Each release remains useful on its own.** Unbuilt factors are not presented as zero-risk categories.

## 3. User workflow and information architecture

### 3.1 Primary daily workflow

1. Review official cash, ETF and net P&L.
2. Check market moves and data freshness.
3. Identify the largest explained contributions and residuals.
4. Drill into the relevant factor, issuer and bonds.
5. Compare the exposures held during the period with current exposures.
6. Run defined scenarios on the current book.
7. Review exceptions and archive the explanation.

### 3.2 Navigation

Primary screens:

- **Market**
- **P&L**
- **Exposure**
- **Scenarios**

Supporting views:

- **Basket library**
- **Factor definitions**
- **Data quality and reconciliation**
- **Model versions**

### 3.3 Shared context

Persist the selected book, reporting currency, date/range and attribution version across primary screens.

Always distinguish:

- The interval being explained.
- The position timestamp used for historical attribution.
- The current-position timestamp used for prospective risk.
- Market-data timestamps and source.
- Whether a result is observed, calculated, estimated or unavailable.

Changing the historical date must not silently combine historical factor moves with current positions.

## 4. Screen requirements

### 4.1 Market screen

**Question:** What happened outside the book?

Required components:

- Broad HY movement and BB/B/CCC observations.
- Quality gaps and spread ratios in aligned panels.
- Sector-by-rating heatmap with a tenor filter.
- Credit-curve movements, separate from government-rate movements.
- Dispersion and tail measures once available.
- ETF price/NAV observations.
- Coverage, issuer-count and stale-mark indicators.

Each market measure should expose:

| Field | Requirement |
| --- | --- |
| Identity | Stable name, indicator/factor ID and definition version |
| Level | Current value and unit where meaningful |
| Movement | 1-day, 5-day and 20-day change with stated convention |
| Context | Historical range or percentile, with its window |
| Interpretation | One sentence explaining what an increase means |
| Quality | Coverage, source, timestamp and relevant caveats |
| Drilldown | Constituents, weights and largest contributions |
| Status | Monitor only, attribution enabled, scenario enabled or unavailable |

The heatmap must allow separate views of absolute spread changes, proportional changes and controlled relative moves. Do not blend these into one unexplained measure.

### 4.2 P&L screen

**Question:** How did market movements affect the positions held?

Required components:

- Actual cash-bond, ETF and net P&L.
- An additive attribution waterfall or signed contribution bars.
- A contribution table with factor-to-issuer-to-bond drilldown.
- Daily and cumulative residual P&L.
- Separate accounting reconciliation and data-quality indicators.

Core table:

| Component | Observed move | Cash-long exposure | Cash-short exposure | ETF exposure | Net exposure | Attributed P&L |
| --- | --- | --- | --- | --- | --- | --- |
| Common credit | Defined unit | Signed | Signed | Signed | Signed | Currency amount |
| Quality contrasts | Defined unit | Signed | Signed | Signed | Signed | Currency amount |
| Sector effects | Defined unit | Signed | Signed | Signed | Signed | Currency amount |
| Credit curve | Defined unit | Signed | Signed | Signed | Signed | Currency amount |
| Other implemented factors | Defined unit | Signed | Signed | Signed | Signed | Currency amount |

Additional P&L rows must include applicable carry/roll, rates, FX, ETF basis/tracking, issuer/bond residuals, identified events, nonlinear effects, trading, funding, borrow, fees and reconciliation.

Do not use a percentage-of-total-P&L chart when large offsetting gains and losses make the denominator misleading.

### 4.3 Exposure screen

**Question:** What risk is held now?

Show:

- Cash longs, cash shorts, ETF hedges and net exposures.
- Gross and net CS01, dollar DTS and key-rate sensitivities.
- Exposures to implemented quality, sector and curve factors.
- Issuer concentrations and overlap through ETFs.
- Standard-shock losses.
- Covariance-based risk contributions once implemented.
- Intended versus incidental exposure, if the desk supplies that classification.

A factor row must state whether its value is dollars per 1 bp, dollars per 1% proportional move, or dollars per another defined shock.

Where available, explain exposure changes from trades, market-driven sensitivity changes, ETF composition and model updates separately.

### 4.4 Scenarios screen

**Question:** What would happen under a specified market move?

Each scenario must show:

- Scenario name, definition and version.
- Shock size, unit, sign and what remains fixed.
- Current portfolio timestamp.
- Cash, ETF and net projected P&L.
- Largest factor and issuer contributors.
- Linear sensitivity versus full-repricing method.
- Coverage and instrument exceptions.

Scenario controls must apply only implemented mappings. A missing factor should be unavailable with an explanation, never a silent zero.

The first release of this screen uses predefined scenarios. A later release can allow user-defined combinations with validation against redundant or incompatible shocks.

### 4.5 Basket and factor detail

Provide a detail drawer or dedicated page containing:

- Economic question.
- Universe and eligibility.
- Legs, weights, issuer caps and matching controls.
- Reference market and shock anchor.
- Return and spread-movement definitions.
- Unit, normalization and positive direction.
- Rebalance and constituent-exit treatment.
- Residual unwanted exposures.
- Raw versus controlled series.
- Data coverage and latest update.
- Whether it is enabled for monitoring, attribution or scenarios.

### 4.6 Interaction and visual behaviour

- Preserve user context when drilling between factor, issuer and bond.
- Keep P&L sign colours separate from the interpretation of market moves. Widening can hurt a long and help a short.
- Show positive and negative risk contributions without forcing them into a positive-only composition chart.
- Use explicit states for loading, missing, stale, thin sample, reconciliation break and model-version mismatch.
- Show observed values in native units; offer standardized views as an additional option with the normalization disclosed.
- Distinguish market-representative weights from the portfolio's own exposure weights.
- Keep important definitions accessible without filling the main screen with implementation text.

## 5. Progressive delivery plan

### Layer 0 — Measurement foundation

**Build**

- Book perimeter, reporting currency, valuation time and price conventions.
- Stable security and issuer identifiers.
- Position, trade, cash-flow and official P&L feeds.
- Daily marks, analytics and source timestamps.
- Historical ratings, classifications and universe membership.
- Initial signed CS01, dollar DTS and rates exposures.

**Usable output**

A position and issuer report showing actual P&L, gross/net sensitivities and data coverage.

**Acceptance gate**

- Positions roll forward from known trades/events.
- Signed quantities and price units are correct.
- Official P&L differences are identified within an agreed tolerance.
- Coupon, short-position and ETF distribution examples behave correctly.
- Historical reference data is not overwritten with current classifications.

### Layer 1 — Baseline dashboard

**Build**

- A basic market diary: HY, BB/B/CCC, rates and ETF observations.
- Carry/roll and rates explanation using existing analytics where possible.
- Common proportional-credit exposure and P&L.
- ETF P&L and a separately identified basis/tracking or unexplained component.
- Explicit residual and reconciliation lines.

**Usable output**

An answer to: “Did the broad hedge offset the common credit move, and where is the remaining P&L?”

Quality and sector observations can appear as monitor-only series before their attribution is implemented.

**Acceptance gate**

- Common widening produces sensible cash and ETF contributions.
- Rates-only moves and distribution dates do not create false credit factors.
- Intraday trading is accounted for explicitly.
- Large residuals can be traced to positions and marks.

### Layer 2 — Reference basket library

**Build**

- Shared basket calculation engine and definition registry.
- B/BB and CCC/B pairs.
- Matched sector baskets for the largest gross sector risks.
- A maturity/credit-curve pair where useful.
- Optional liquidity comparisons only when inputs are credible.

**Usable output**

A market monitor with raw levels, relative moves, controlled credit returns and constituent contribution drilldown.

**Acceptance gate**

- Basket weights are known before the return interval.
- Rebalances and exits do not create unlabelled jumps.
- Controls and residual broad/rates exposures are visible.
- Issuer concentration and stale-mark coverage are reported.

### Layer 3 — Additive factor attribution

**Build**

- A defined common-market coordinate.
- Two independent quality contrasts.
- Conditional sector and then credit-curve effects.
- Joint estimation or a documented fixed decomposition convention.
- Instrument-level contributions and aggregate drilldowns.
- Historical attribution versioning.

**Usable output**

Observed market move → opening exposure → dollar contribution → issuer/bond explanation.

**Acceptance gate**

- Factor overlap and identification are resolved.
- A pure market move is not charged again as quality/sector P&L.
- New factors improve interpretation beyond development dates or individual issuers.
- Daily and cumulative contributions reconcile.
- Old/new attribution comparisons use compatible model definitions.

### Layer 4 — Dispersion and issuer diagnostics

**Build**

- Within-peer spread-level dispersion.
- Dispersion of daily residual issuer moves.
- Tail breadth, tail severity and concentration.
- Long/short portfolio overlays on the market distribution.
- Issuer-common versus bond-specific residual structure.

**Usable output**

An explanation of which parts of market divergence the book owned.

**Acceptance gate**

- Generic dispersion has no automatic signed P&L charge.
- Multiple bonds of an issuer do not appear as independent diversification.
- Issuer events, stale marks and accounting breaks remain distinguishable.

### Layer 5 — Forward risk

**Build first**

Defined current-position scenarios for broad credit, quality, sectors, credit curve, rates, ETF basis and issuer default/recovery.

**Build next**

Factor covariance, issuer residual risk, risk contributions and normal/stressed estimates.

**Usable output**

Current dollar exposure, standard-shock loss, scenario contributors and later covariance-aware risk.

**Acceptance gate**

- Scenario anchors, signs and units are explicit.
- Significant optionality and large shocks use appropriate repricing.
- Issuer overlap includes ETF holdings where available.
- Risk contribution arithmetic reconciles.
- Forecasts are checked against realized outcomes at the intended horizon.

### Layer 6 — Daily operating process

**Build**

- Scheduled processing, exception review and archived runs.
- Historical reproducibility.
- Exposure-change explanations.
- Consistent screen context and exportable results.
- Optional hedge comparison after measurement is reliable.

**Usable output**

A repeatable daily process with traceable inputs and stable definitions.

**Acceptance gate**

- A historical report can be reproduced from its archived inputs and versions.
- Failed or incomplete runs cannot silently appear complete.
- Changing a definition does not overwrite earlier meaning.

## 6. Reference basket specification

### 6.1 Required basket fields

| Field | Requirement |
| --- | --- |
| ID and version | Stable identifier and explicit definition version |
| Economic question | One clear market behaviour being measured |
| Universe | Currency, instrument types, eligibility and data requirements |
| Legs | Long and short/reference groups, if paired |
| Weighting | Weight convention, issuer cap and treatment of sparse groups |
| Controls | Market beta, rates, rating, sector, tenor, seniority and optionality as applicable |
| Market convention | Proportional spread, calibrated spread beta or another documented convention |
| Observables | Spread levels, gap/ratio, raw return and controlled movement |
| Positive direction | What an increase means economically |
| Normalization | Fixed gross or risk unit; never an unstable near-zero net-value denominator |
| Maintenance | Membership review, rebalance, weight timing and hedge-ratio updates |
| Events | Defaults, migrations, calls, maturities and new issues |
| Coverage | Issuer count, missing/stale weights and concentration |
| Permissions | Monitor only, attribution enabled and/or scenario enabled |

### 6.2 Initial basket example: B versus BB

- **Question:** Is B credit weakening relative to BB beyond the initial common proportional-spread convention?
- **Leg A:** B-rated corporate bonds in the chosen universe.
- **Leg B:** BB-rated corporate bonds in the same currency.
- **Controls:** Matched sector and tenor mix; relevant seniority/optionality controls.
- **Weights:** Issuer-capped within each leg; equal dollar DTS between legs initially.
- **Observed levels:** OAS for each leg, B-minus-BB gap and log spread ratio.
- **Movement:** Paired credit-price return after carry/roll and rates treatment.
- **Direction:** Positive decompression means B weakens relative to BB; a long-B/short-BB return has the opposite sign.
- **Attribution:** The controlled quality factor is mapped jointly with the other implemented factors.
- **Quality display:** Coverage, issuer concentration, stale-mark share and residual market/rates exposure.

### 6.3 Construction rules

- Use an external universe rather than only the desk's positions.
- Use starting weights and classifications for each measurement interval.
- Cap issuer influence; multiple bonds from one issuer do not create independent observations.
- Match sectors within rating/tenor groups; aggregate over a documented common mix.
- Do not invent missing rating/tenor cells. Broaden, restrict or flag the comparison.
- Separate changing constituent-level spread averages from composition-controlled movement indices.
- Retain economic outcomes of defaults and exits in P&L history.
- Record the exact rebalancing and risk-normalization convention.
- Label fixed-constituent baskets and constant-maturity curves distinctly.

## 7. Factor catalogue and promotion rules

### 7.1 Initial catalogue

| Series | Primary observation | Intended role |
| --- | --- | --- |
| Broad HY | Common proportional spread change | Baseline credit factor |
| B/BB | Spread gap, ratio and controlled pair return | Quality indicator and factor |
| CCC/B | Spread gap, ratio and controlled pair return | Tail-quality indicator and factor |
| Sector relative moves | Sector versus comparable non-sector credit | Conditional sector factors |
| Credit curve | Short-versus-long credit movement | Curve factor where material |
| Liquidity | Bid–ask/quote measures and matched credit comparison | Indicator first; factor if supported |
| ETF basis | Market-price versus NAV return and premium/discount | Separate ETF contribution and risk |
| Issuer dispersion | Width of peer-adjusted issuer levels/moves | Risk-state diagnostic |
| Tail breadth/severity | Counts/weights and magnitude of extreme moves | Risk-state diagnostic |
| Residual correlation | Rolling co-movement of issuer residuals | Estimated risk diagnostic; display window |

### 7.2 Promotion to attribution

A candidate may become an additive factor only when:

1. It is measured independently of the current portfolio.
2. Its economic interpretation is clear and reproducible.
3. Its unit, sign, anchor and security mapping are defined.
4. Coverage is sufficient for the intended use.
5. Overlap with existing factors is explicitly controlled.
6. Relevant exposures can be determined before the period being explained.
7. It improves residual interpretation or stress coverage beyond isolated development examples.
8. Its definition and comparison history are versioned.

Monitor-only status is a valid permanent outcome for useful diagnostics.

## 8. Calculation and attribution contracts

These contracts define the intended mathematical meaning. Implementation may use existing pricing infrastructure, but units and interpretation must remain consistent.

### 8.1 Units and time

| Quantity | Convention |
| --- | --- |
| Signed market value, V | Positive for a long, negative for a short; explicit reporting currency |
| Spread duration, D^s | Sensitivity to the selected spread convention |
| Spread, s | Decimal in formulas; a separate display conversion produces bp |
| Absolute spread change | Basis points when applied to CS01 |
| Proportional spread change, z | Decimal: 0.10 means 10% widening |
| Historical exposure | Beginning-of-period or correctly timestamped subperiod exposure |
| Forward exposure | Current portfolio and market snapshot |
| Cumulative P&L | Sum of period dollar contributions, using each period's exposures |

Do not normalize a leveraged long–short book by a near-zero net market value. If returns are displayed, use a documented NAV or capital denominator.

### 8.2 Bond sensitivity and spread P&L

For an ordinary small spread move:

$$
CS01_i = V_i D_i^s 10^{-4}
$$

$$
\Delta P_i^{credit} \approx -CS01_i\,\Delta s_i^{bp}
$$

For a proportional spread move:

$$
K_i = V_i D_i^s s_i,
\qquad
z_i = \frac{\Delta s_i}{s_{i,0}}
$$

$$
\Delta P_i^{credit} \approx -K_i z_i
$$

Thus a long position with positive K loses approximately 0.01 K for a 1% proportional widening.

DTS is the initial structural convention for common credit exposure. Empirical adjustments must use a documented estimation method and information available before attribution. An empirical return beta already contains price sensitivity; it must not be multiplied by DTS again. See [Reference 1](#reference-1).

### 8.3 Common market movement

An initial basket-based common move is:

$$
m_t = \sum_i w_{i,t-1}
\frac{s_{i,t}-s_{i,t-1}}{s_{i,t-1}}
$$

Weights are predetermined, normalized and subject to the agreed issuer controls.

The first-order portfolio contribution is:

$$
P\&L_{market,t} \approx -\left(\sum_i K_{i,t-1}\right)m_t
$$

The market coordinate must remain anchored to a documented reference universe and weighting convention.

### 8.4 Pair normalization

For market-value weights within each leg:

$$
\overline{DTS}_A = \sum_{i\in A} w_i D_i^s s_i
$$

Let N_A and N_B be the positive market values allocated to the two legs, not their bond face amounts. The initial leg market-value matching rule is:

$$
N_B = N_A\frac{\overline{DTS}_A}{\overline{DTS}_B}
$$

This matches dollar DTS. It does not automatically eliminate all rate, sector, maturity or empirical-beta differences. Remaining exposures must be measured.

Calculate basket returns using prior-period weights. Store raw total return and the return attributable to credit-price changes separately. Carry/roll, rates, events and nonlinear effects require explicit treatment.

### 8.5 Indicators versus factor coordinates

Examples of useful raw quality indicators are:

$$
G_{B/BB}=S_B-S_{BB},
\qquad
R_{B/BB}=\log(S_B/S_{BB})
$$

The gap can widen under a common proportional selloff while the ratio remains unchanged. These are different views of the same underlying prices, not two independent P&L factors.

A ratio shock is incomplete without an anchor. For example, a two-group coordinate system can be defined as:

$$
M=w\log S_B+(1-w)\log S_{BB},
\qquad
Q=\log S_B-\log S_{BB}
$$

Then:

$$
\Delta\log S_B=\Delta M+(1-w)\Delta Q
$$

$$
\Delta\log S_{BB}=\Delta M-w\Delta Q
$$

This is one explicit way to define quality movement while holding the common level fixed. Hold w fixed over the measurement interval.

The log-spread coordinates in this example are an alternative convention to the arithmetic proportional changes in Sections 8.3 and 8.6. Do not interchange their shocks without conversion. For a log-spread scenario, the corresponding arithmetic change is z = exp(delta log s) - 1; a linear approximation must be labelled as such. The implemented multi-group model must document one consistent normalization and conversion rule.

### 8.6 Joint attribution model

A compact proportional-spread specification is:

$$
z_{i,t}
=\beta_{i,t-1}m_t
+q_{i,t-1}^{\top}Q_t
+a_{i,t-1}^{\top}S_t
+c_{i,t-1}^{\top}C_t
+u_{i,t}
$$

Here Q, S and C represent quality, sector and credit-curve movements. Use beta equal to one for the initial structural version, with any later refinements frozen before each attribution interval.

Requirements:

- Resolve factor redundancy with explicit constraints or reference groups.
- Estimate rating and sector effects jointly, or use a documented fixed decomposition order.
- Keep raw basket observations available beside controlled factors.
- Do not add unrestricted issuer fixed effects that absorb the systematic categories. Model issuer-common and issue-specific residual structure after the systematic fit.
- Cross-sectional controls do not imply that factor histories are uncorrelated.
- A controlled statistical description does not prove an economic cause.

For a factor with loading x:

$$
E_{k,t-1}=\sum_i K_{i,t-1}x_{ik,t-1}
$$

$$
P\&L_{k,t}\approx-E_{k,t-1}f_{k,t}
$$

E is a widening-loss exposure under this sign convention. If the interface instead shows signed P&L sensitivity, it must show the corresponding negative sign.

Calculate instrument contributions first, then aggregate. Published fixed-income methodologies use cross-sectional estimation with spread and key-rate sensitivities; the exact proposed factor set remains a design choice for this book. See [Reference 2](#reference-2).

### 8.7 P&L reconciliation

The accounting identity is:

**Actual P&L = mutually exclusive accounting and market contributions + explicit reconciliation difference.**

Required distinctions:

- Cash-flow carry versus spread-price movement.
- Rates versus credit-curve movement.
- Systematic credit versus issuer/bond residual.
- Recognized events versus remaining residual.
- Nonlinear/pricing effects versus accounting breaks.
- Opening-position P&L versus trading effects.
- Funding, borrow and fees versus costs already embedded in a return series.

Use dirty prices and cash flows consistently. Do not add coupon or ETF distribution income again when it is already included in total return.

For full revaluation, fix the order of rate, spread and other updates, or use a documented averaged allocation where interactions are material. Nonlinear attribution is not uniquely determined without such a convention.

### 8.8 ETF accounting and look-through

Maintain two position views:

1. Actual ETF shares for official accounting and traded P&L.
2. Synthetic underlying holdings for risk and explanation.

Synthetic holdings must not be added to official assets or P&L a second time.

Monitor three return layers with their normalization made explicit:

1. Modeled underlying holdings return.
2. NAV return minus modeled holdings return.
3. Market-price return minus NAV return.

For the accounting bridge, use dollars or an explicitly common return base. NAV-normalized and market-price-normalized returns must not be scaled interchangeably.

For a fixed holding interval, let q be signed ETF shares, P the market price per share, N the NAV per share, and d the per-share distribution recognized consistently during the interval:

$$
P\&L_{actual}=q(P_1-P_0+d)
$$

$$
P\&L_{NAV}=q(N_1-N_0+d)
$$

$$
P\&L_{market/NAV}
=q[(P_1-N_1)-(P_0-N_0)]
$$

Therefore actual holding-period P&L equals NAV P&L plus the market/NAV dollar difference. The modeled-holdings-to-NAV bridge uses NAV-based dollars. Apply the bridge to the appropriate subperiods when shares change, and handle execution and external costs separately.

Scale underlying risk to the actual ETF share position using holdings/per-share or NAV-based exposure information. Recognize cash and derivatives where material.

The NAV-versus-model difference can include holdings changes, expenses, marks and model error. Market-price/NAV divergence is a separate observation and can reflect asynchronous information.

Use actual holdings where available, rather than treating creation/redemption baskets as the complete fund. These can differ materially. See [Reference 3](#reference-3).

### 8.9 Dispersion and issuer diagnostics

Use issuer-capped or issuer-level observations after systematic effects are removed.

Display:

- Spread-level dispersion relative to matched peers.
- Dispersion of daily residual moves.
- Tail breadth and severity.
- Long-book and short-book residual outcomes.
- Largest issuer contributors.

Do not allocate the same loss to both “dispersion” and issuer residual P&L. Distressed and defaulted instruments also require price/recovery views where spread statistics become unstable.

### 8.10 Forward risk

For a return-factor model:

$$
r=Bf+\epsilon,
\qquad
a=B^\top v
$$

where v contains signed dollar holdings.

Under the model's factor/residual orthogonality convention:

$$
\operatorname{Var}(P\&L)=a^\top\Sigma_f a+v^\top\Omega v
$$

Preserve issuer-common residual covariance, issue-specific differences and relevant cash/ETF overlap. Persistent factor/residual relationships require model review rather than an automatic independence assumption.

Factor variance contributions use:

$$
RC_k=a_k(\Sigma_f a)_k
$$

They can be negative because exposures offset each other. Standalone risks do not add to total portfolio risk.

## 9. Data model and provenance

The implementation is platform-agnostic. Tables may initially be files or database tables, provided identifiers, history and versioning are preserved.

| Entity | Representative fields |
| --- | --- |
| Security reference | security_id, issuer_id, risk_parent_id, type, currency, price_multiplier, sector, rating, maturity, seniority, coupon/call terms, effective_at, available_at |
| Position snapshot | book_id, position_at, security_id, signed_quantity, quantity_unit, position_basis |
| Trades and events | event_id, revision, event_time, security_id, signed_quantity/cash_amount, price, accrued, fees, event_type, cancellation/correction status |
| Official P&L | book_id, interval_start/end, security_id or aggregate ID, component, amount, currency, source_version |
| Market marks | security_id, observed_at, available_at, source, clean/dirty price, accrued, bid/ask, quality flags, revision |
| Analytics | security_id, as_of, mark_version, curve/model versions, OAS, spread duration, key-rate vector, optionality measures, quality |
| Universe membership | universe_id, security_id, effective_from/to, available_at, eligibility and classification version |
| ETF holdings | ETF_id, holdings_date, available_at, underlying_id, quantity/weight, cash/derivative fields, source |
| ETF market data | ETF_id, observed_at, price, NAV, distributions, valuation conventions |
| Basket definition | basket_id, definition_version, eligibility, weights, controls, sign, normalization, maintenance |
| Basket weights | basket_id, version, effective_at, available_at, security_id, weight and leg |
| Market series | series_id, version, observed_date, level, movement, units, coverage, contributor lineage |
| Exposures | portfolio/issuer/security ID, as_of, factor_id, model_version, signed exposure, unit, quality |
| Attribution | interval, security_id, component/factor_id, contribution, currency, model_version, input_run_id |
| Scenarios | scenario_id/version, position_at, shock definition, security_id, projected P&L, method and coverage |
| Run manifest | run_id, inputs/versions, timestamps, status, exceptions, output versions |

Store consistent timestamps and display the relevant desk timezone. Preserve economic effective time separately from information availability.

Coverage should be reported using gross exposure measures where feasible; signed net weights can conceal missing long and short positions that offset.

## 10. Quality controls and acceptance scenarios

These are required checks against concrete risks, not targets for cosmetic fit. Numeric tolerances should reflect accounting precision, price quality and economic materiality.

| Test | Expected behaviour |
| --- | --- |
| Common proportional widening | Initial DTS-neutral pair has approximately zero first-order credit P&L; no extra quality/sector charge is invented |
| Rates-only move | Rates explain responds; controlled credit factor remains stable within pricing tolerance |
| Pure quality move | Appropriate quality contrast responds; a CCC-heavy sector does not absorb the move solely through its composition |
| Sector-relative move | Matched sector factor responds after common and quality effects |
| Flat prices with basket rebalance | No unexplained market-return jump |
| Rating migration | Composition/classification change is visible and historical exposure uses the correct lagged classification |
| Issuer default or exit | Economic P&L is retained and not removed by dropping the constituent |
| Coupon/ETF distribution | No artificial loss or duplicate income |
| Intraday position change | Contribution uses opening/subperiod holdings and trade economics |
| Stale cash marks versus live ETF | Quality flags and multi-day diagnostics expose the timing mismatch |
| Multiple bonds and ETF overlap | Issuer exposure is aggregated consistently without false diversification |
| Model update | Historical comparison uses one version or an explicit bridge |
| Cumulative attribution | Sum of daily/subperiod contributions equals cumulative attributed dollars |
| Missing factor or sensitivity | Display is unavailable/estimated, not zero |

### 10.1 Illustrative quality-pair check

Use synthetic values solely to test units and signs:

| Position | Starting spread | Signed CS01 |
| --- | --- | --- |
| Long CCC basket | 800 bp | +$1,000/bp |
| Short BB basket | 200 bp | -$4,000/bp |

Dollar DTS is +$800,000 and -$800,000 respectively.

- Under 1% proportional widening, CCC widens 8 bp and BB 2 bp: P&L is -$8,000 + $8,000 = $0.
- If CCC instead widens 18 bp while BB widens 2 bp: P&L is -$18,000 + $8,000 = -$10,000.

The additional relative weakness is visible without attributing a loss to common beta.

### 10.2 Investigation rules

- Review difficult historical days and separate held-out dates.
- Review long and short books separately.
- Investigate repeated residual clustering by issuer, sector, quality, tenor and liquidity.
- Check daily results against multi-day observations where marks are asynchronous.
- Downweighting an observation for factor estimation must not erase its actual P&L.
- Do not target a fixed explained-P&L percentage at the expense of transparent residuals.

## 11. Operating process and implementation architecture

### 11.1 Daily run

1. Ingest positions, transactions, events, marks, curves and ETF information.
2. Apply identifier, timing, completeness and staleness checks.
3. Reconcile official accounting and position changes.
4. Calculate or ingest instrument analytics and mechanical P&L.
5. Calculate reference-basket observations and controlled factors.
6. Attribute the period's P&L using the appropriate historical exposures.
7. Calculate current exposures, scenarios and available risk estimates.
8. Publish results with exceptions and coverage.
9. Review the largest residuals and archive the run.

An incomplete run must be visibly incomplete. Late data should create a revised, traceable result.

### 11.2 Weekly review

- Review residual patterns and thin/stale market groups.
- Check basket concentration and maintenance.
- Review current versus historical sensitivities.
- Assess whether additional factors are justified.
- Review factor definition changes before deployment.
- Compare forecast risk and scenarios with realized outcomes.

### 11.3 Logical components

| Component | Responsibility |
| --- | --- |
| Historical data store | Preserve raw observations, positions, events and versions |
| Basket engine | Apply shared eligibility, weighting, controls and movement calculations |
| Exposure and attribution engine | Map observations to securities and reconcile dollar contributions |
| Scenario and risk engine | Reprice defined shocks and estimate covariance-aware risk |
| Reporting interface | Present results, context, drilldowns, definitions and exceptions |

Keep calculations outside presentation code. A spreadsheet prototype and a later application should consume the same calculated outputs.

### 11.4 Optional hedge comparison

After the earlier layers are reliable, represent each allowed hedge as an exposure vector:

$$
a_{after}=a_{before}+Hh
$$

Show current, proposed and remaining exposures; scenario effects; basis risk; and transaction/financing constraints.

Minimum variance and broad-market neutrality are different objectives. Required neutrality conditions must be explicit. Several similar ETFs may provide limited additional independent hedge directions.

This capability remains a comparison tool unless execution is separately requested and authorized.

## 12. Initial backlog and release checklist

### 12.1 First build tasks

| Priority | Task | Deliverable |
| --- | --- | --- |
| P0 | Select difficult historical days and reserve separate validation dates | Review set with known questions |
| P0 | Agree perimeter, price/time, P&L and beta conventions | Measurement specification |
| P0 | Assemble minimum historical inputs and start preserving daily snapshots | Versioned source tables |
| P0 | Reconcile positions and official P&L | Accounting/coverage report |
| P0 | Calculate signed CS01, dollar DTS and rates exposure | Cash/ETF/net exposure table |
| P0 | Implement common-credit baseline and market diary | First daily dashboard |
| P1 | Build and validate B/BB reference pair | Basket definition, history and contribution view |
| P1 | Add CCC/B and largest sector comparisons | Expanded market monitor |
| P1 | Implement joint quality and sector attribution | Additive P&L bridge |
| P2 | Add curve, dispersion, tails and issuer residual detail | Detailed market/book analysis |
| P2 | Implement defined current-position scenarios | Scenario-loss report |
| P2 | Add covariance risk and optional hedge comparison | Forward risk extensions |

A short initial history can validate plumbing and accounting. Risk calibration requires history appropriate to the horizon and market regimes; the same short sample should not automatically be treated as sufficient.

### 12.2 First usable release

- [ ] Actual cash, ETF and net P&L is available.
- [ ] Major accounting breaks are identified.
- [ ] Signed cash/ETF/net CS01, dollar DTS and rates exposures are visible.
- [ ] Carry/rates/common-credit contributions and residual are shown.
- [ ] BB/B/CCC market observations are visible as diagnostics.
- [ ] ETF observations include their valuation timing and current mapping quality.
- [ ] Largest residual issuer contributors can be inspected.
- [ ] Missing and stale inputs cannot appear as zero.
- [ ] Inputs, outputs and model definitions are archived.

### 12.3 Design handoff deliverables

The design team should produce:

- Navigation and shared context model.
- Market, P&L, Exposure and Scenarios screen layouts.
- Factor/issuer/bond drilldown interaction.
- Basket/factor detail view.
- Loading, unavailable, stale, thin-sample, reconciliation and version-mismatch states.
- Clear sign and unit conventions for tables and charts.
- Progressive-disclosure behaviour as later layers become available.
- A reusable table/heatmap/chart system suitable for dense analytical work.

## 13. Decisions to resolve

These items should not be silently assumed by design or engineering:

| Decision | Why it matters |
| --- | --- |
| Currency and regional scope | Rates, FX, reference curves and comparable universes |
| End-of-day versus intraday use | Timestamp requirements and holding-period attribution |
| Official P&L and price source | Reconciliation and valuation conventions |
| Available bond analytics | Use of existing sensitivities versus a pricing build |
| Reference universe | Representativeness and factor coverage |
| Initial market-beta convention | Meaning of neutrality and quality residuals |
| ETF holdings availability | Precision of look-through and issuer overlap |
| Callable/distressed coverage | Appropriate pricing and event treatment |
| Sector, rating and tenor taxonomy | Matching, sparsity and interpretation |
| Basket maintenance schedule | Stability versus responsiveness |
| Reporting capital denominator | Meaningful portfolio return/risk percentages |
| Risk horizon and limits | Covariance estimation, scenario calibration and controls |
| Target platform | Integration, dashboard delivery and export requirements |
| Ownership of definitions and changes | Consistent governance of the model's meaning |

## 14. Glossary

| Term | Meaning in this product |
| --- | --- |
| OAS | Option-adjusted spread under the selected pricing and reference-curve convention |
| CS01 | Signed dollar exposure to a 1 bp spread change |
| Key-rate sensitivity | Dollar exposure to a movement at a specified point on the rates curve |
| DTS | Spread duration multiplied by spread |
| Dollar DTS | Signed market value multiplied by DTS; sensitivity to a unit proportional spread change |
| Decompression | Greater separation between stronger and weaker credit; the precise gap/ratio/factor definition must be stated |
| Sector-relative move | Sector movement after the agreed common, quality and other controls |
| Credit-curve move | Change in the term structure of credit spreads, separate from rates |
| Dispersion | Width of a distribution of issuer levels or moves; the chosen distribution must be stated |
| ETF basis | Defined difference between ETF traded performance and NAV/underlying performance |
| Issuer residual | Issuer movement remaining after systematic factors |
| Reconciliation difference | Remaining mismatch between the accounting P&L and the calculated explanation |
| Attribution version | The exact factor definitions, mappings and conventions used for a result |

## 15. Supporting references

The sources below support general modeling principles. The product scope, screens, sequencing and acceptance requirements above are proposed design decisions for this workbench.

### Reference 1

Robeco, **Duration Times Spread: measuring credit risk**. Supports the relative-spread sensitivity interpretation of DTS.

[Robeco research](https://www.robeco.com/en-us/insights/2024/09/duration-times-spread-measuring-credit-risk)

### Reference 2

MSCI, **Developments for Portfolio Managers**, fixed-income model section, November 2023. Describes cross-sectional factor estimation, key-rate and DTS exposures, and factor/specific covariance.

[MSCI presentation](https://www.msci.com/documents/1296102/41604730/Developments%2Bfor%2BPortfolio%2BManagers.pdf/54de75fd-00be-2d74-f871-315ef89b849a?t=1699368987962)

### Reference 3

Bank for International Settlements, **The anatomy of bond ETF arbitrage**. Discusses differences between ETF holdings and creation/redemption baskets, and valuation timing in ETF/NAV comparisons.

[BIS research](https://www.bis.org/publications/anatomy-bond-etf-arbitrage)

### Reference 4

Elizaveta Krylova, ECB Working Paper 1912, **Determinants of euro-denominated corporate bond spreads**, June 2016. Background on extracting rating, sector, maturity and other effects from individual-bond data. Its empirical universe should not be treated as validation of the proposed HY specification.

[ECB working paper](https://www.ecb.europa.eu/pub/pdf/scpwps/ecbwp1912.en.pdf)


