# AstroCrown — ACS & Monte Carlo Design Specification

**Version:** 1.1.3 (standalone)  
**Status:** DESIGN ONLY — economic and policy rules locked for v1; decision-grade simulation criteria remain BLOCKED  
**Simulator code:** NOT implemented in this document  
**Authority:** This file is the sole source of truth. Prior chats are historical only.  
Legacy materials (old internal Monte Carlo “35% corridor”, “1.67% tax optimal”, drawdown percentage claims, Sepolia transfer-tax token drafts, diversified yield rebuy-basket notes) are NOT evidence for v1 decisions.

---

## HOW TO USE THIS DOCUMENT

- Read the GLOSSARY first, then sections A through H in this version.
- For adversarial audit: use this complete file; do not assume that sections absent from this version exist elsewhere.
- For implementation: implement against this file only; bump the version on any material rule change and record a changelog entry.
- **Decision-grade Monte Carlo is BLOCKED in v1.1.3.** This version contains no section K and defines no complete decision-grade evidence checklist or simulation results. Any simulation output remains EXPLORATORY until a later version defines and verifies the missing criteria. Do not infer or invent the absent section.

---

## Version history

- **v1.1.3:** Corrected the document's section inventory and made the absent decision-grade section explicit. No economic or policy rules changed; simulation results remain exploratory.
- **v1.1.2:** Prior standalone specification; preserved unchanged in the repository.

---

## GLOSSARY (read first)

**ACS**  
Anti-Crash System. The set of rules that define the floor price, redeem-at-floor, and official floor–roof liquidity so that, under those rules, circulating tokens are not exited through an unbacked floor.

**FPRF**  
Floor Price Reserve Funds. The reserve of USD-referenced stable value that backs the floor. In equations this quantity is called V. Only V enters F = V / C.

**V**  
Same as FPRF: the balance of the floor reserve, measured in USD-stable units.

**F (floor)**  
Floor price in stable units per 1 ACROWN.  
When C > 0: F = V / C.

**C (circulating)**  
Quantity of ACROWN counted as user-held and redeemable at the floor.  
Platform inventory is not included in C.

**I (inventory)**  
ACROWN held by the platform (not yet released, or returned via redeem or fees).  
Source for primary sales and roof sales. Not part of C.

**q**  
A non-negative quantity of ACROWN in a single action (for example, amount sold, amount redeemed, or amount paid as a Paymaster fee in ACROWN). When several quantities appear in one flow, they may be written q_sale, q_redeem, q_fee, etc.  
Unless stated otherwise, q is measured in ACROWN token units (human-readable or base units must be fixed in the implementation config and used consistently).

**R (roof)**  
Upper official platform sale price for inventory.  
R = (1 + m) * F, where m is the corridor width  
(example: m = 0.30 means R is 30% above F).

**m (corridor width)**  
Fraction above the floor that defines the roof. Not pre-chosen; tested on a grid.  
The old figure “35%” is brainstorm only, not proof.

**G (gas float)**  
Balance used to pay network transaction fees (for example SOL) when the Paymaster sponsors user transactions. G is not part of V and does not enter F.

**Paymaster**  
Mechanism that sponsors chain gas so users can pay fees in ACROWN instead of holding the native gas token for sponsored actions. Users still pay a fee in ACROWN under published rules.

**T0**  
Minimum recurring survival operating cost (hosting, critical API/RPC access, essential tooling). “Small margin” means a modest buffer above T0 only.  
T0 does not mean full GPU, R&D, or aerospace budget on day one.

**LCR (coverage idea)**  
Under pure F = V / C, redeeming all of circulating C at price F consumes V exactly (100% coverage of C at the floor under that accounting).

**P_ACS**  
Official price used inside the ACS model band [F, R] for base simulation paths and official platform execution semantics.

**P_EXT**  
Price observed on external venues. May trade outside [F, R]. The ACS guarantee is program/platform liquidity rules, not every global market print.

**MC / Monte Carlo**  
Method of running many simulated paths under stated rules to compare parameters (corridor m, fee settings, stresses). This document locks DESIGN; it does not contain simulation result tables.

**GIGO**  
Garbage In, Garbage Out. If assumptions, inputs, or scenarios are wrong, hidden, or self-fulfilling, simulation outputs are not reliable evidence.

**alpha (α)**  
Share of Paymaster fee surplus (after gas is covered) allocated to V (floor residual). Share (1 − α) goes to operations (T0 + small margin).

**epsilon (ε)**  
Elasticity parameter: how strongly higher effective fees reduce mid-band and wash activity in the model. Redeem demand is treated as less elastic.

**lambda (λ)**  
Intensity rates for synthetic event arrivals (buys, redeems, washes, etc.).

**PDA**  
Program-derived account (Solana terminology): a program-controlled sub-account, useful to separate V, G, and ops balances under one program.

**USD-stable**  
A token intended to track the US dollar. Used as the unit of account for V.  
The issuer may still freeze accounts, blacklist, or depeg; that residual risk must be disclosed. The project’s “no user freeze” policy for ACROWN does not control the stable issuer.

**Sponsorship refusal**  
The Paymaster declines to pay network gas (for example G is empty, or the fee required by policy would exceed q_max). This is not an admin freeze or blacklist of the user. ACS instructions remain callable; the user may need native gas to submit the transaction.

**Ops (operations balance)**  
Funds for survival operating spend (T0 + small margin), logically separated from V.  
Not part of the floor formula.

---

## A. AIMS

1. Non-custodial product posture: no user-deposit TVL pool as the core model.
2. Fixed-supply $ACROWN; no mint and no burn required for ACS mechanics.
3. Transparent floor F = V / C; users can redeem at F under the published rules.
4. Official inventory liquidity in band [F, R] while inventory and rules allow.
5. Paymaster so users need not hold native gas for normal sponsored actions; users pay fees in ACROWN under published rules.
6. Fee value allocation order of intent:
   (a) network gas and G health  
   (b) floor residual that can strengthen V relative to C  
   (c) ops survival: T0 + small margin only at the fee-policy target
7. Priority order:
   (1) Survival of G, T0, and ACS integrity  
   (2) ACS integrity (no dilution, honest F, no silent drain of V)  
   (3) Adoption and honest volume  
   (4) Later scale (extra GPU, R&D, aerospace) from larger absolute fee income if adoption succeeds — not as day-one hard fee requirements
8. Never dilute the floor. Maintain ethical standards in design, operations, and public communication (see dogma D2).
9. User sovereignty: no admin pause, freeze, or blacklist of users on ACS paths.
10. Auditability: F should be recomputable from published FPRF (V) and defined C.
11. Monte Carlo v1 tests parameters under explicit scenarios; it does not rubber-stamp a pre-chosen corridor percentage.

---

## B. DOGMAS

**D1** Never dilute F = V / C.  
No release, gift, or accounting trick may increase C without adequate addition to V (or equivalent protection) so that F would fall.

**D2** Ethical standards in communication and conduct.  
Includes, without limitation:
- no public profit guarantees or “profits from day one” style claims
- no misleading description of floor, reserves, risks, or guarantees
- no presenting exploratory simulation as proven performance
- no hiding material residual risks (including stable-issuer risk)
- honesty about what is official ACS liquidity versus external market prints
- alignment of public claims with rules actually implemented on-chain  
Ethical practice is broader than marketing copy alone; it constrains how the system is described, measured, and sold.

**D3** No admin pause, freeze, or blacklist of users on ACS paths.  
Redeem and official inventory sale instructions remain permissionlessly callable subject to chain inclusion and gas. Sponsorship refusal is not a user freeze (see Glossary).

**D4** Fixed supply; recycle via inventory; no mint required for ACS.

**D5** Users pay for sponsored usage through the published Paymaster fee path.  
Includes, without limitation:
- users pay fees in ACROWN under published rules for sponsored actions
- the protocol does not silently gift network gas by spending V outside those published rules
- fee accounting must not double-count value or disguise ops spend as floor
- gas sponsorship is funded from G and the gas slice of fees, not from arbitrary FPRF withdrawals
- when sponsorship is refused, users may still act with native gas if the chain allows; that does not create a right to free gas from V  
D5 is the whole user-pays / non-parasitic-gas principle, not only the single prohibition on silent V drains.

**D6** V is pure USD-stable only in v1.  
No yield strategies inside V; no volatile crypto as V; no privacy-coin or “crowd shielding” mixing of FPRF (breaks simple public recomputation of F).

**D7** Survival outranks adoption metrics when they conflict.  
Without survival of G, T0, and ACS integrity, adoption is not durable.

**D8** Legacy Monte Carlo and legacy token-tax marketing numbers are not proof.

---

## C. STATE VARIABLES

Every simulated path tracks at least:

- V — Floor Price Reserve Funds (USD-stable units only)
- C — Circulating redeemable ACROWN (user-held per membership rules)
- I — Platform ACROWN inventory
- G — Gas float (native gas units or equivalent)
- Ops — Operations balance (logical; not in V)
- P_ACS — Official ACS price
- P_EXT — Optional external price (for stress or comparison metrics)
- F, R — Floor and roof

Counters (examples): cumulative fee value to gas / to V / to ops; total redeemed quantity; sponsorship refusals; wash cost; time in G green/amber/red; dilution event count (must remain zero under correct rule execution).

No separate growth reserve inside v1 floor math.

---

## D. CIRCULATING MEMBERSHIP (C) — v1 DEFAULT

**IN C:**
- ACROWN balances held by end users for ordinary ownership and redeem rights.

**NOT IN C:**
- Platform inventory I
- Paymaster / fee intake balances controlled by the platform
- Ops-controlled ACROWN
- Stable FPRF balances (those are V, not circulating ACROWN)
- Wallets whose purpose is platform control rather than end-user redeem

Multisig, DAO, LP, bridge, or market-maker wallets:  
Default NOT IN C unless explicitly documented as end-user redeemable custody.  
Any exception must be written into the configuration of a simulation run.

---

## E. FLOOR, REDEEM, TERMINAL STATE

**Floor formula:**  
If C > 0: F = V / C  
If C = 0: F is mathematically undefined; the UI may display the last defined F.

**Redeem of quantity q** (q > 0, q ≤ user balance, q ≤ C):
1. User sends q ACROWN into inventory I (C decreases by q; I increases by q).
2. Protocol pays the user q * F in USD-stable from V (V decreases by q * F).
3. User also pays the standard Paymaster fee in ACROWN, as quantity q_fee, IN ADDITION to q (same fee rules as other sponsored actions). No extra “exit tax” in v1.
4. Under pure redeem with no fee residual credited to V, F is unchanged.
5. If fee accounting places residual stable value into V, F may increase.

No admin pause, freeze, or blacklist on redeem.  
No hard per-transaction redeem cap is required in the base Monte Carlo (optional sensitivity analysis later).

**Terminal state C = 0:**
- No further redeem.
- Last F may be shown for display only.
- Residual V remains platform FPRF property under FPRF rules.
- New circulating C appears only through sales from I under normal sale rules (stable proceeds to V).

---

## F. SALES, ROOF, LAUNCH, INVENTORY

**Official platform sale of quantity q from inventory I:**
- Price p must satisfy F ≤ p ≤ R for official inventory liquidity policy.
- 100% of sale proceeds (stable) go to V. No ops skim from sales in v1.
- I decreases by q; C increases by q.

**Launch window** (short promotional period, for example under 24 hours):
- Sell at p = F or slightly above F.

**After launch:**
- Sales allowed across the band as needed.
- If peer-to-peer offers are insufficient to meet demand, the platform may sell from I at p in [F, R].

**Roof:**
- Platform may sell from I at p = R.
- Proceeds 100% to V.

**Promotions / “free” distribution:**
- Forbidden if they raise C without protecting V (dilution).
- Allowed only with an ops-funded equivalent top-up of V (or an equivalent rule that does not reduce F).

**Inventory policy:**
- I is depletable.
- Large total token supply does NOT by itself prove I cannot be emptied.
- Release policy and sale rates determine I paths.
- Monte Carlo must include inventory depletion, delayed fill, and band-break cases.

---

## G. OFFICIAL BAND vs EXTERNAL PRICE

Base Monte Carlo paths may keep P_ACS inside [F, R] while redeem and roof offers function (support at F via redeem; supply at R via inventory sales).

P_EXT may move outside [F, R]. ACS does not claim control of all external venues.

Metrics that evaluate corridor behavior and official fees use P_ACS unless a scenario explicitly studies P_EXT.

**Stress scenarios required for scientific hygiene:**
- I depleted so roof cannot fill
- delayed fills / liquidity gaps
- external price dislocation versus the official band

**Corridor parameter m:**  
Coarse grid example: 10%, 20%, 30%, 40%, 50%, 60%, then finer steps.  
No pre-accepted “optimal” m. Old 35% is not proof.

---

## H. PAYMASTER FEE STRUCTURE

Fee is not a single fixed ACROWN amount forever.

**Conceptual calculation (policy-level):**

```text
gas_stable ≈ gas_in_native * price_native_per_stable
             * (1 + convert_friction)
             * (1 + buffer)

q_fee ≈ clamp(
          ceil( gas_stable / F * (1 + surplus_uplift) ),
          q_min,
          q_max
        )
