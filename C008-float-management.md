> **PROPOSED APPROACH** — Not an official AFP publication. Not reviewed or endorsed by
> the Association for Financial Professionals. This document proposes how existing AFP/CTP
> principles from *Essentials of Treasury Management, 8th Edition* may apply to digital
> asset treasury operations, and is offered for professional discussion only.
> **Author:** Garzaro &nbsp;|&nbsp; **Status:** Proposed — subject to revision

# AFP-CA-ETM8-C008 — Float Management Standards for Digital Asset Treasury Operations

> Base standard: *Essentials of Treasury Management*, 8th Edition (ETM8). AFP, 2025.
> Document ID: AFP-CA-ETM8-C008
> Series index: AFP-CA-ETM8-INDEX (C000-series-index.md)
> Prerequisite: AFP-CA-ETM8-C001 (Practitioner's Bridge — vocabulary bridge and delta concepts)

---

## 1. Scope and ETM8 Alignment

This addendum extends ETM8 Chapter 12 (Disbursements, Collections, and Concentration) to digital asset treasury operations. ETM8 Chapter 12 establishes the framework for measuring and managing float across disbursement and collection cycles. That framework assumes banking-hours clearing windows, predictable clearing house (consensus protocol) availability, and reversible wire transfers (on-chain transactions). Digital asset treasury operations invalidate each of those assumptions.

This document defines:

- The **clearing epoch** as the authoritative analog to the ETM8 clearing window, derived from protocol class
- The **Clearing Epoch Coverage Ratio (CECR)** as the primary efficiency metric, replacing legacy float efficiency ratios
- Portfolio tier classification (GREEN / YELLOW / ORANGE / RED) based on CECR thresholds and HALT event counts
- Sub-float categories and their individual coverage obligations
- The Z-state flag and HARD_HALT condition as operationally mandatory controls

All terms follow the C001 vocabulary bridge. Where legacy ETM8 vocabulary appears in quoted text or cross-references, the C001 bridge term is appended in brackets on first use.

---

## 2. The Clearing Epoch — Replacing the Clearing Window

### 2.1 Definition

The **clearing epoch** is the protocol-defined settlement cycle, measured in milliseconds, derived deterministically from the consensus protocol class governing the instrument. It is the precise analog of the ETM8 **clearing window** (C001 vocabulary bridge: clearing window → clearing epoch, protocol-defined).

Unlike a banking clearing window, which is a business-hours construct subject to cut-off times and operator discretion, the clearing epoch is:

- **Deterministic**: computable from published protocol parameters before any on-chain transaction (C001 vocabulary bridge: wire transfer → on-chain transaction, protocol rail) is broadcast
- **Continuous**: operates without banking-hours dependency (C001 D4 — 24/7 Clearing)
- **Non-negotiable**: the consensus protocol (C001 vocabulary bridge: clearing house → consensus protocol) does not accept clearing epoch override requests

### 2.2 Protocol Class Table

The following table defines clearing epoch derivation rules by protocol class. These are the only CTP-recognized values for clearing epoch computation. Practitioners must use the derivation rule for the protocol class governing each custodial wallet (C001 vocabulary bridge: bank account → custodial wallet) position.

| Protocol Class | Representative Instruments | clearingMs Derivation Rule | Notes |
|---|---|---|---|
| `pow` (Proof of Work) | BTC (600,000 ms), XMR (120,000 ms) | target_block_time_ms × 1 | Probabilistic; confirmation depth applies (C001 D1 — Transaction Irreversibility) |
| `pos` (Proof of Stake) | ETH post-merge, ADA | slot_time_ms × finality_slots | Finality slots are protocol-defined; use conservative published value |
| `poh` (Proof of History) | SOL | slot_time_ms × confirmation_slots | ~400 ms slot; use 400 × 32 = 12,800 ms for optimistic finality |
| `dpos` (Delegated Proof of Stake) | HIVE, EOS | round_time_ms × 2/3_supermajority | Validator concentration risk applies (C001 D6 — Validator Concentration Risk) |
| `l2-optimistic` | OP Mainnet, Arbitrum | L1_epoch_ms + challenge_period_ms | Challenge period (7 days = 604,800,000 ms) dominates; rarely CTP-eligible |
| `l2-zk` | zkSync Era, Polygon zkEVM | L1_epoch_ms + proof_gen_ms | Proof generation time is variable; use 95th-percentile observed value |
| `payment-channel` | Lightning Network (BTC) | htlc_timeout_ms (path-find bounded) | base_fee + fee_rate × amount; deterministic at path-find time |
| `stablecoin` | USDC (XRPL), USDT (Stellar) | ledger_close_ms × 1 | XRPL: 3,000–5,000 ms; Stellar: 5,000 ms nominal |
| `contract` | Protocol staking, OUSG, BUIDL | redemption_notice_ms | Defined in instrument terms; treat as clearing epoch for yield tier (C001 D5 — Smart Contract Counterparty) |
| `cbdc` | Wholesale CBDC pilots | operator_published_ms | Use operator SLA; treat as clearing window analog |

**Application rule**: When a treasury position spans multiple protocol classes (e.g., a custodial wallet holding both BTC and a tokenized T-bill (C001 vocabulary bridge: money market fund → tokenized T-bill)), compute CECR separately per instrument class. Do not blend clearing epochs across protocol classes.

### 2.3 Governor Ceiling

The **governor ceiling** constrains the maximum permissible clearing epoch for CTP-eligible instruments. It is defined as:

```
governor_ceiling = clearingMs × 0.90
```

The 10% margin represents the minimum buffer required for TMS detection and operational response — specifically, the time needed to detect an approaching Z=0 transition within the ≤90-second loop interval (C014 Section 5.1) and initiate remediation before the clearing epoch fully elapses. Instruments whose observed settlement duration exceeds the governor ceiling require immediate Z-state review and sub-float reclassification.

---

## 3. Clearing Epoch Coverage Ratio (CECR)

### 3.1 Definition

The **Clearing Epoch Coverage Ratio (CECR)** is the primary float efficiency metric for digital asset treasury operations, replacing legacy availability ratio and float efficiency ratio calculations in ETM8 Chapter 12.

```
CECR = (sum of Z=1 position-milliseconds) / (total observed position-milliseconds in period)
```

Where:
- **Z=1** designates an in-transit position within an active clearing epoch (see Section 4 — Z-State)
- **Z=0** designates a stale position; the clearing epoch has elapsed without confirmed settlement

CECR is computed per channel (custodial wallet × protocol class pairing) and then rolled up to portfolio level using position-value weighting.

### 3.2 CTP Efficiency Floor

The **CTP efficiency floor** is **CECR ≥ 0.60**.

| CECR Range | Portfolio Tier | Status |
|---|---|---|
| ≥ 0.90 | GREEN | Full CTP eligibility; no remediation required |
| 0.75 – < 0.90 | YELLOW | Marginal; reduction schedule required within 30 days |
| 0.60 – < 0.75 | ORANGE | Below target; reduction schedule required within 10 business days |
| < 0.60 | RED | CTP ineligible; HARD_HALT evaluation required |

A portfolio at RED tier must initiate a HARD_HALT assessment before any new deployment of idle capital. No yield-tier deployment (C011) is permitted while CECR < 0.60.

### 3.3 Channel-Level CECR Reporting

Each custodial wallet × protocol class channel must report CECR independently. A single high-performing channel cannot mask a failing channel through portfolio aggregation for regulatory reporting purposes. Portfolio-level CECR aggregation is permitted only for internal dashboards clearly labeled as "portfolio composite."

---

## 4. Z-State Classification

### 4.1 Z=1 — In-Transit

A position carries **Z=1** when all of the following conditions are satisfied:

1. An on-chain transaction has been broadcast and accepted by the mempool (or equivalent pre-consensus pool)
2. The elapsed time since broadcast is less than or equal to the clearing epoch (clearingMs) for the governing protocol class
3. No protocol-layer error signal has been received (D3 — Protocol-Layer Risk)
4. The antinodeIndex for the governing protocol is < 13

### 4.2 Z=0 — Stale

A position is reclassified to **Z=0** when any of the following occurs:

1. Elapsed time since broadcast exceeds clearingMs without confirmed settlement
2. An on-chain transaction is dropped from the mempool without confirmation
3. The antinodeIndex for the governing protocol reaches or exceeds 13 (see HARD_HALT, Section 5)
4. A confirmation depth (C001 vocabulary bridge: settlement finality → confirmation depth, n-block probabilistic) threshold is not reached within 2× the nominal clearing epoch
5. The custodial wallet reports a balance inconsistency between the BLID chain (C001 vocabulary bridge: reconciliation → BLID chain / thorn anchor) anchor and the on-chain state (C001 vocabulary bridge: bank statement → float register snapshot)

### 4.3 Z-State and the Float Register

Every Z-state transition must be recorded in the float register snapshot with:

- The BLID transaction ID (C001 vocabulary bridge: SoD audit record → BLID transaction ID) of the causative on-chain transaction
- The block timestamp at transition (not system timestamp — see C013 Section 3.2)
- The new Z value (0 or 1)
- The corrected_friction_ms (C001 vocabulary bridge: float period → corrected_friction_ms / clearing epoch) representing accumulated stale time

---

## 5. HARD_HALT

### 5.1 Trigger Conditions

A **HARD_HALT** is declared when either of the following conditions is met:

- `antinodeIndex ≥ 13` for any protocol class in the active portfolio
- Any channel carries `tier = HALT` as assigned by the C012 risk taxonomy

HARD_HALT is a binary state, not a warning tier. There is no partial HARD_HALT.

### 5.2 Operational Requirements Under HARD_HALT

Upon HARD_HALT:

1. **No new on-chain transactions** may be broadcast on the affected protocol until the antinodeIndex drops below 13 and a cleared thorn anchor is recorded
2. **No yield deployment** (C011) is permitted for instruments settling on the affected protocol
3. **Sub-float reclassification** is mandatory: all in-transit positions on the affected protocol are immediately reclassified to Z=0 pending individual confirmation review
4. A **BLID chain anchor** must be issued within one clearing epoch of HARD_HALT declaration to mark the event boundary in the audit trail
5. **Escalation to treasury management** is required within the shorter of: (a) one clearing epoch, or (b) four hours

### 5.3 HARD_HALT Resolution

HARD_HALT resolution requires all of the following:

- `antinodeIndex < 13` for the affected protocol
- A confirmed thorn anchor on the BLID chain showing post-HALT state
- CECR recomputed and confirmed ≥ 0.60 on the affected channel
- Treasury management sign-off per the SoD requirements in C010

---

## 6. Sub-Float Categories

ETM8 Chapter 12 identifies disbursement float and collection float as the primary sub-categories. Digital asset treasury operations require three additional sub-float categories, each demanding its own CECR computation and reduction schedule.

### 6.1 instrument_gap_float

**Definition**: Value stranded between instrument types due to protocol class mismatch or redemption notice delays.

**Example**: A tokenized T-bill (OUSG/BUIDL) position with a redemption notice period (the `contract` protocol class clearing epoch) that exceeds the operational liquidity requirement. The gap between redemption notice and fund availability constitutes instrument_gap_float.

**CECR requirement**: CECR ≥ 0.70 for instrument_gap_float channels. Channels below this threshold require a reduction schedule filed within 10 business days.

**Reduction schedule**: Redistribute to shorter-clearing-epoch instruments until CECR ≥ 0.70 or position is retired.

### 6.2 pool_stale_float

**Definition**: Value in the pre-confirmation pool (mempool or equivalent) that has exceeded the Z=1 window without confirmation. This is the digital asset analog of mail float in ETM8.

**Example**: An on-chain transaction broadcast to the BTC network that has remained unconfirmed beyond 600,000 ms (the `pow` clearing epoch) without reaching the required confirmation depth.

**CECR requirement**: CECR ≥ 0.65. Pool_stale_float above 5% of total portfolio value by position-weight requires immediate remediation.

**Reduction schedule**: Replace broadcast strategy with fee-accelerated re-broadcast (within protocol-floor fee schedule; EVM gas auction-based replacement is CTP-ineligible per Section 7).

### 6.3 canonical_gap_float

**Definition**: The difference between the balance reported in the float register snapshot (C001 vocabulary bridge: bank statement → float register snapshot) and the balance confirmed on-chain at the thorn anchor point.

**Example**: A custodial wallet reporting a balance that has not been reconciled (BLID chain anchored) within the standard anchor cycle, creating an unverified balance gap.

**CECR requirement**: canonical_gap_float must be zero at each BLID anchor cycle. Any non-zero canonical_gap_float triggers an immediate thorn anchor review and constitutes a potential R4/R5 risk finding under C012.

---

## 7. Fee Eligibility for CTP Purposes

### 7.1 Protocol-Floor Fee Schedules Only

For CTP certification purposes, only **protocol-floor (deterministic) fee schedules** are eligible for inclusion in clearing epoch and CECR computations. A fee schedule is CTP-eligible when:

- The fee is fully determinable at instruction time (before broadcast)
- The fee does not change based on network congestion between instruction and confirmation
- The fee is expressed as a fixed amount or a fixed rate applied to a known quantity

CTP-eligible fee schedules include:

- XRPL: 10 drops (fixed minimum)
- Stellar: 100 stroops (fixed minimum)
- Solana: 5,000 lamports per signature (deterministic at instruction)
- Lightning Network: base_fee + (fee_rate × forwarded_amount), computed at path-find time and bounded by the path

### 7.2 EVM Gas — CTP-Ineligible

**EVM gas (auction-based) fee schedules are CTP-ineligible** *(C001 D3 — Protocol-Layer Risk)*.

EVM gas cost is indeterminate at instruction time due to its auction mechanism. The fee can change materially between the time a transaction is constructed and the time it is included in a block. This indeterminacy:

- Prevents accurate clearing epoch computation (gas cost affects effective settlement economics)
- Creates unquantifiable protocol-layer risk exposure
- Is incompatible with the deterministic fee requirement of the CECR model

All EVM-rail assessments must carry an explicit "EVM gas exposure: present" flag in float register entries. EVM on-chain transactions may be used operationally but may not be used as the basis for CTP float efficiency claims. See C009 Section 4 for payment rail selection guidance.

---

## 8. Portfolio Tier Classification — Summary

| Tier | CECR | HALT Count (rolling 30d) | Permitted Actions |
|---|---|---|---|
| GREEN | ≥ 0.90 | 0 | Full operations; yield deployment permitted |
| YELLOW | 0.75 – < 0.90 | 0–1 | Operations permitted; reduction schedule required |
| ORANGE | 0.60 – < 0.75 | 1–2 | Restricted; no new deployment; remediation active |
| RED | < 0.60 | Any | HARD_HALT assessment required; no deployment |
| HALT | Any | Active HARD_HALT | All deployment suspended; escalation mandatory |

Portfolio tier is computed at each BLID anchor cycle and recorded in the float register snapshot. Tier downgrades take effect immediately. Tier upgrades require two consecutive anchor cycles at the upgraded tier before reclassification.

---

## 9. Integration with Other C-Series Addenda

- **C009** (Payment Rail Assessment): Provides the protocol class assignments and confirmation depth data used to derive clearingMs values in the protocol class table (Section 2.2)
- **C010** (Custody Architecture): Defines the SoD controls required for BLID chain anchor issuance and thorn anchor review procedures
- **C011** (Yield Policy): Uses CECR and portfolio tier as the primary gates for idle capital deployment; no deployment permitted below CTP efficiency floor
- **C012** (Risk Taxonomy): Maps sub-float categories to risk categories R1–R5; HARD_HALT events feed C012 active shield classification
- **C013** (Accounting): Defines the GL treatment for pool_stale_float and canonical_gap_float as distinct line items
- **C014** (TMS Integration): Specifies the minimum TMS capability for real-time CECR computation and HARD_HALT detection

---

## Source Reference

> *Essentials of Treasury Management, 8th Edition (ETM8).*
> Association for Financial Professionals (AFP). Rockville, MD, 2025.
> © 2025 Association for Financial Professionals. All rights reserved.
> All proprietary rights to the examination, including copyright, are held by AFP.
