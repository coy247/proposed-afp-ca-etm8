> **PROPOSED APPROACH — EXPOSURE DRAFT.** Not an official AFP publication. Not reviewed or
> endorsed by the Association for Financial Professionals. This exposure draft proposes conforming
> amendments to how existing AFP/CTP principles from *Essentials of Treasury Management, 8th Edition*
> may apply to digital asset treasury operations, and is issued for professional discussion only.
> **Author:** Garzaro &nbsp;|&nbsp; **Status:** Exposure Draft — issued for comment; not adopted

# AFP-CA-ETM8-C008-ED1 — Exposure Draft: Realignment of CECR to the Crypto Earnings Credit Rate

> Base standard: *Essentials of Treasury Management*, 8th Edition (ETM8). AFP, 2025.
> Document ID: AFP-CA-ETM8-C008-ED1
> Series index: AFP-CA-ETM8-INDEX (C000-series-index.md)
> Prerequisite: AFP-CA-ETM8-C001 (Practitioner's Bridge)
> Amends: AFP-CA-ETM8-C008 (Float Management Standards), Section 3
> Disposition on adoption: supersedes C008 §3.1–§3.2 as revised herein

---

## 1. Purpose and Status

This exposure draft proposes conforming amendments to Section 3 of AFP-CA-ETM8-C008. It is issued
for professional comment and is not adopted. Upon adoption, the amendments in Sections 4–6 would be
incorporated into C008 and the superseded text retired.

The proposal originates in a definitional inconsistency: the identifier **CECR**, as presently
defined in C008 §3.1, denotes a *Clearing Epoch Coverage Ratio* — a time-in-state coverage fraction.
That construct is not an earnings-credit rate, and is inconsistent with the intended meaning of CECR
as the **Crypto Earnings Credit Rate** and with the account-analysis framework of ETM8 from which the
concept derives.

## 2. ETM8 Basis — The Earnings Credit and Account Analysis

Under ETM8's treatment of account analysis, the **earnings credit** is a *soft* (notional) credit
extended on a customer's **investable balance** to offset charges for compensable services. It is not
cash interest and is not equivalent to hard interest. The investable balance is derived from the
account balance by removing amounts not available for the bank to invest:

```
ledger balance  −  float (uncollected funds)      =  collected balance
collected balance  −  reserve requirement          =  investable balance
earnings credit    =  investable balance × ECR × (days in period / 360)
```

The **Earnings Credit Rate (ECR)** is an administered, dynamic rate. It moves with short-term rate
conditions and competitive posture; where a bank references an external index, prevailing practice
uses a short government reference (e.g., the 91-day Treasury bill) or an overnight funds rate, often
applied as a stated percentage of that reference. It is not, in general, a pure formula pegged to a
single index.

> *Editorial note (RFC-04):* the precise ETM8, 8th Edition chapter and section for account analysis
> and the earnings credit allowance is to be confirmed against the reorganized 8th-edition table of
> contents before adoption. The construct above is stated from AFP's published description of the
> earnings credit; the cross-reference is pending editorial verification.

## 3. Issue Identified

C008 §3.1 currently defines:

```
CECR = (sum of Z=1 position-milliseconds) / (total observed position-milliseconds in period)     [0,1]
```

This is a coverage fraction measuring the proportion of a period a position spends in the cleared
(Z=1) state. It is a legitimate and useful operational quantity, but it is **not** an earnings-credit
rate: it is dimensionless, is not applied to a balance, carries no time factor, and offsets no
service charge. Retaining the CECR identifier for this quantity overloads the term and misrepresents
the account-analysis lineage from which "CECR" (Crypto Earnings Credit Rate) is intended to descend.

## 4. Proposed Amendment 1 — Redefinition of CECR (revises §3.1)

**CECR — Crypto Earnings Credit Rate.** CECR is the digital-asset analog of the earnings credit rate:
an administered, dynamic soft-credit rate applied to a channel's **investable position balance** to
generate a credit that offsets **protocol fees** (the compensable-service analog in digital asset
operations).

```
investable position balance = settled position value − in-transit (float) value
crypto earnings credit       = investable position balance × CECR × (T / 360)
```

where:
- **In-transit (float) value** is the digital-asset analog of collection float — position value whose
  clearing epoch has not yet resolved. The friction/staleness fraction defined in C008 §4 (Z-State)
  is the basis for this deduction.
- **Reserve analog.** Digital assets carry no fractional-reserve requirement. Where a protocol imposes
  a mandatory lockup or unbonding period, that encumbered value is not investable and is deducted on
  the same basis as a reserve requirement (see RFC-03).
- **T** is the period expressed in day-equivalents (clearing epochs normalized to a 360-day
  convention, consistent with the ETM8 account-analysis convention above).

**CECR is a rate, not a coverage fraction.** Any efficiency floor or tiering applied to CECR must be
expressed in rate terms; the [0,1] tier ladder presently in §3.2 does not apply to a rate (see §6).

## 5. Proposed Amendment 2 — Undefined Measure on an Empty Observation Window (adds §3.4)

Where an observation window contains no settled-position exposure — i.e., no investable balance is
formed — **CECR is undefined (absent), not zero.** An absent measure denotes "no surface to measure,"
which is materially distinct from "an investable balance existed and generated no qualifying credit."

- An **absent** CECR is not a RED-tier condition and does not, by itself, trigger a HARD_HALT
  assessment under C008 §5.
- Only a window in which an investable balance existed and the resulting credit fell below the
  applicable floor constitutes an efficiency deficiency subject to tiering and escalation.

This distinction parallels C008 §6, under which `canonical_gap_float` of **zero** is the conformant
state — establishing that, within this series, a zero-valued measure is context-dependent and must not
be uniformly interpreted as a deficiency.

## 5a. Proposed Amendment 3 — Absent-Input State for a Floor (adds §3.2 clause; general)

**A floor without a defined absent-input state is not a gate.** Where a measured input to a floor test can
be absent and an implementation supplies a default in its place, that default — not the measurement —
determines the verdict. Any default at or above the floor renders the gate **unfalsifiable**: the floor can
never be breached, regardless of the underlying condition, because no real measurement is ever consulted. A
standard that specifies a floor must therefore also specify the treatment of an absent input; leaving it to
implementation default is equivalent to leaving the floor undefined. This holds for any implementation of
C008, independent of how a given system happens to compute or store the measure.

The case is live for **CEC** (§6). The CTP efficiency floor (§3.2, CEC ≥ 0.60) is a floor test, and a
coverage measure with no observation window has genuinely no value — the same "no surface to measure"
condition §5 establishes for CECR. The standard must specify which disposition governs an absent CEC input,
and should select one rather than leave it to the reporting system:

- **(a) Fail closed** — an absent input is treated identically to a floor breach (RED tier / HARD_HALT
  assessment per §3.2).
- **(b) Absent ≠ zero, ≠ pass** — an absent input is a third state, reported as **unmeasured**, and is not
  classified into a tier. **Recommended**, consistent with the CECR treatment in §5: a coverage measure
  with no observation window has no value rather than a bad one, and classifying it as either pass or fail
  asserts a measurement that was never taken.
- **(c) Fail open with disclosure** — a default is permitted, but any tier derived from a defaulted (rather
  than measured) CEC must be flagged as such in the float register entry, so a defaulted verdict is never
  indistinguishable from a measured one.

Under the recommended disposition (b), a channel or portfolio whose CEC cannot be computed for a period is
reported **unmeasured** for that period and excluded from tier-based deployment gating until a measurement
exists — never silently classified as conformant by a default that sits above the floor.

## 6. Disposition of the Clearing Epoch Coverage Measure

The time-in-state fraction presently defined as CECR remains operationally useful for measuring
clearing efficiency independent of the credit economics. To resolve the identifier collision, this
exposure draft proposes to **retain that quantity under a distinct designation — Clearing Epoch
Coverage (CEC)** — and to reserve **CECR** exclusively for the Crypto Earnings Credit Rate:

```
CEC = (sum of Z=1 position-milliseconds) / (total observed position-milliseconds in period)     [0,1]
```

The existing §3.2 tier ladder (≥0.90 GREEN; 0.75–<0.90 YELLOW; 0.60–<0.75 ORANGE; <0.60 RED) and the
0.60 efficiency floor are **correctly scaled to CEC** (a [0,1] fraction) and would carry forward
unchanged under the CEC designation. They do **not** transfer to CECR, which is rate-denominated.

## 7. Conforming Amendments to Related Addenda

Adoption requires conforming updates where "CECR" is presently referenced as a coverage fraction:

- **AFP-CA-ETM8-C011 (Yield Policy)** — the deployment gate "no deployment while CECR < 0.60" is a
  *coverage* condition and should reference **CEC ≥ 0.60**. Any separate rate-based condition on the
  Crypto Earnings Credit Rate is to be stated distinctly.
- **AFP-CA-ETM8-C014 (TMS Integration)** — "real-time CECR computation" should specify **which**
  measure the TMS computes (CEC, CECR, or both) and at what cadence.
- **AFP-CA-ETM8-C012 (Risk Taxonomy)** — confirm that HARD_HALT triggers reference CEC (coverage), and
  incorporate the empty-window (absent) case from §5 so that "no surface" is not risk-classified.

## 8. Requests for Comment

- **RFC-01 — Benchmark anchor for CECR.** CECR is administered/dynamic. Candidate anchors: (a) **Term
  SOFR** — successor to USD LIBOR, whose panels ceased June 2023 — as a TradFi-canonical, auditable
  reference; (b) a crypto-native short reference rate. If an external index is adopted, specify the
  applied percentage (haircut), consistent with account-analysis practice.
- **RFC-02 — Disposition of CEC. RESOLVED (2026-08-02): RETAIN.** The coverage measure is retained under
  the CEC designation (§6); the §3.2 [0,1] tier ladder + 0.60 floor carry forward to CEC. Basis: the
  ladder is scaled to a [0,1] coverage measure and is meaningless against a rate, so retiring CEC would
  orphan it. This unblocks the C011/C014/C012 conforming repoint (§7).
- **RFC-03 — Reserve analog.** Confirm whether protocol lockup/unbonding periods reduce the investable
  position balance, and how encumbered value is measured.
- **RFC-04 — ETM8 cross-reference.** Confirm the precise 8th-edition chapter/section for account
  analysis and the earnings credit allowance (§2).

## 9. References

- *Essentials of Treasury Management*, 8th Edition (ETM8). AFP, 2025 — account analysis; earnings
  credit allowance (chapter reference per RFC-04).
- AFP, "What Is the Earnings Credit?" — definition of the earnings credit and investable balance.
- AFP, "Earnings Credit Rate: Predicting the Cloudy Future of ECR" — ECR as an administered, dynamic
  rate.
- AFP-CA-ETM8-C008 (Float Management Standards) — Sections 3, 4 (Z-State), 5 (HARD_HALT), 6.
