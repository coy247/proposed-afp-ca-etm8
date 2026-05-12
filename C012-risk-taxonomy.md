> **PROPOSED APPROACH** — Not an official AFP publication. Not reviewed or endorsed by
> the Association for Financial Professionals. This document proposes how existing AFP/CTP
> principles from *Essentials of Treasury Management, 8th Edition* may apply to digital
> asset treasury operations, and is offered for professional discussion only.
> **Author:** Garzaro &nbsp;|&nbsp; **Status:** Proposed — subject to revision

# AFP-CA-ETM8-C012 — Risk Taxonomy for Digital Asset Treasury

> Base standard: *Essentials of Treasury Management*, 8th Edition (ETM8). AFP, 2025.
> Document ID: AFP-CA-ETM8-C012
> Series index: AFP-CA-ETM8-INDEX (C000-series-index.md)
> Prerequisite: AFP-CA-ETM8-C001 (Practitioner's Bridge — vocabulary bridge and delta concepts)

---

## 1. Scope and ETM8 Alignment

This addendum extends ETM8 Chapter 16 (Enterprise Risk Management) and Chapter 17 (Financial Risk Management) to establish a structured risk taxonomy for digital asset treasury operations. ETM8 Chapter 16 defines the ERM framework — risk identification, assessment, response, and monitoring — as applied to treasury functions. ETM8 Chapter 17 addresses specific financial risk categories: market risk, credit risk, liquidity risk, and operational risk.

Digital asset treasury introduces risk categories that have no direct analog in ETM8's frameworks and materially transforms the risk profile of categories that do have analogs. This document defines seven risk categories (R1–R7) specific to digital asset treasury, maps each to the C001 delta concepts that introduce or amplify the risk, identifies the float shield categories associated with each, and establishes the monitoring and escalation standards for each category.

Two status designations govern risk event response:

- **Active shield**: an active risk event requiring operational remediation. Treasury operations proceed with restrictions per the governing risk category.
- **Thorn anchor blocked**: a structural R4/R5 finding requiring immediate action. No on-chain transactions from the affected custodial wallet (C001 vocabulary bridge: bank account → custodial wallet) may proceed until the thorn anchor is cleared.

All terms follow the C001 vocabulary bridge.

---

## 2. Risk Category R1 — Protocol Risk

### 2.1 Definition

Protocol risk is the risk that the consensus protocol (C001 vocabulary bridge: clearing house → consensus protocol) governing one or more custodial wallets in the portfolio fails to perform normally, resulting in delayed, failed, or abnormally expensive on-chain transactions (C001 vocabulary bridge: wire transfers → on-chain transactions, protocol rail).

Protocol risk arises from conditions internal to the protocol layer and is not directly controllable by the treasury *(C001 D3 — Protocol-Layer Risk)*.

### 2.2 Sub-Categories

**Validator concentration risk**: A small number of validators control a disproportionate share of block production or consensus, creating a single point of failure or censorship risk *(C001 D6 — Validator Concentration Risk)*. For proof-of-stake and delegated proof-of-stake protocols, validator concentration is measured by the Nakamoto coefficient (minimum number of validators required to compromise the network). A Nakamoto coefficient below 10 is proposed as an R1 flag for treasury purposes; this threshold is not derived from ETM8 (the instrument class did not exist when ETM8 was written) and may be adjusted by treasury policy based on the specific protocol's consensus architecture.

**Network stall risk**: The consensus protocol halts or produces no blocks for a period exceeding the nominal clearing epoch (C001 vocabulary bridge: clearing window → clearing epoch, protocol-defined). Network stalls increment the antinodeIndex and, at antinodeIndex ≥ 13, trigger HARD_HALT per C008 Section 5.

**Fee spike risk**: Protocol fee (deterministic) (C001 vocabulary bridge: payment network fee → protocol fee (deterministic)) spikes to a level that materially exceeds the budgeted transaction cost assumption. For CTP-eligible rails, a fee spike means the observed fee is materially above the protocol floor (e.g., XRPL drops spiking due to a traffic surge). For EVM rails, fee spike risk is continuous and is a primary reason EVM gas is CTP-ineligible.

### 2.3 Associated Float Shield

**sensor_blackout_float**: Value in custodial wallet positions for which the treasury's monitoring system cannot confirm on-chain state due to protocol connectivity loss. A sensor blackout is not the same as a stale position — the position may be Z=1 on-chain but cannot be confirmed by the treasury system, preventing CECR computation.

Sensor_blackout_float is classified as Z=0 for CECR purposes until connectivity is restored and the position is confirmed. Upon restoration, the CECR for the affected period is recomputed using the actual on-chain confirmation timestamps from the float register snapshot (C001 vocabulary bridge: bank statement → float register snapshot).

### 2.4 Monitoring Standard

R1 is monitored at each TMS loop cycle (≤90 seconds per C014 Section 2.7). A Hermes event queue entry is generated for:

- antinodeIndex increment on any active protocol class
- Network stall exceeding 2× the nominal clearing epoch
- Nakamoto coefficient drop below 10 for any protocol in the active portfolio
- Protocol fee spike exceeding 300% of the 30-day average for the relevant protocol

---

## 3. Risk Category R2 — Operational Risk

### 3.1 Definition

Operational risk is the risk of loss resulting from inadequate or failed internal processes, systems, or people, applied to digital asset treasury operations. This category extends the ETM8 Chapter 16 operational risk concept to the specific failure modes of treasury management system (TMS) automation and event queue infrastructure.

### 3.2 Sub-Categories

**System downtime risk**: The TMS or monitoring infrastructure is unavailable, preventing float register updates, CECR computation, or BLID chain (C001 vocabulary bridge: reconciliation → BLID chain / thorn anchor) anchor issuance. During TMS downtime, all positions default to Z=0 for CECR purposes until the system restores and reconciles.

**Event queue failure**: The Hermes event queue (C014 Section 2.7) fails to deliver risk flag notifications. This is a systemic operational failure that eliminates the treasury's ability to detect R1–R7 events in real time. Hermes queue failure is automatically an active shield condition for R2.

### 3.3 Associated Float Shields

**fabric_blackout_float**: Value in positions for which the external data feed required to compute yield or CECR metrics is unavailable. The term "fabric" refers to the external connectivity layer of the treasury monitoring system, not the internal BLID chain. Fabric blackout does not affect on-chain state; it affects the treasury's ability to observe and report on that state.

**triage_float** *(C001 D3 — Protocol-Layer Risk)*: Value stranded in positions that cannot be classified (Z=1 or Z=0) because the protocol-layer data required for classification is unavailable or corrupted. Triage_float is distinct from sensor_blackout_float: sensor blackout is a connectivity failure; triage_float is a data quality failure (the data is present but cannot be trusted).

### 3.4 Monitoring Standard

R2 is monitored via heartbeat check at each TMS loop cycle. If the Hermes event queue produces no events for a period exceeding 3× the loop interval, a system health alert must be triggered. R2 active shield status is triggered automatically by TMS downtime exceeding one clearing epoch on any protocol class in the active portfolio.

---

## 4. Risk Category R3 — Smart Contract and Bridge Risk

### 4.1 Definition

Smart contract and bridge risk is the risk that a smart contract governing a treasury position behaves contrary to the treasury's intent, whether due to a code defect, an exploit, an unauthorized upgrade, or an oracle manipulation *(C001 D5 — Smart Contract Counterparty)*.

This risk category applies to any position where the treasury's custodial wallet has an on-chain transaction directed at a smart contract address — including yield instruments (tokenized T-bills (C001 vocabulary bridge: money market fund → tokenized T-bill), overcollateralized lending, liquid staking), multi-signature wallet contracts, and bridge contracts used for cross-chain transfers.

### 4.2 Code-as-Counterparty Principle

The central governance principle for R3 is **code-as-counterparty**: the smart contract is the counterparty, not the human entity that deployed it. This distinction matters because:

- The deploying entity may not control the contract if ownership has been renounced or transferred
- Contract code can be exploited by any party, not just the deployer
- Upgrade mechanisms (proxy patterns) may allow the contract logic to change without the treasury's knowledge or consent

Every float register snapshot entry involving a smart contract custodial wallet must carry the "code-as-counterparty" flag.

### 4.3 Associated Float Shield

**coupling_float** (from C010 Section 3.2, cross-referenced here from R3 perspective): When the bridge or smart contract creates a coupling between two positions such that a failure in the contract propagates to the treasury's balance unexpectedly. The most common example: a bridge contract failure that leaves assets permanently locked in transit.

Bridge coupling_float is distinct from the SoD-origin coupling_float in C010. Both must be tracked separately in the float register, with the source flag indicating "bridge/contract" or "SoD" origin.

### 4.4 Monitoring Standard

R3 is monitored at each BLID anchor cycle. Any smart contract position without a current audit (audit older than 12 months) is flagged as an active shield condition. Any smart contract position governed by an upgradeable proxy without a multi-sig time-lock on the upgrade function is flagged as an active shield condition.

Bridge positions are assessed at each anchor cycle for bridge liveness (the bridge protocol is operational) and bridge balance integrity (assets sent to the bridge are confirmed as accessible on the destination chain).

---

## 5. Risk Category R4 — Key Management Risk

### 5.1 Definition

Key management risk is the risk of loss of access to, or unauthorized use of, the private keys governing custodial wallets in the portfolio *(C001 D2 — Private Key Custody)*. Key management risk is existential: loss of private keys is permanent loss of the associated assets. There is no recovery mechanism.

### 5.2 Risk Scenarios

**Key loss**: Private keys are lost due to hardware failure, forgotten passphrases, or destruction of backup media. Without the key, the custodial wallet balance is permanently inaccessible.

**Key compromise**: Private keys are accessed by an unauthorized party. Unlike bank account fraud, unauthorized on-chain transactions cannot be reversed after confirmation *(D1 — Transaction Irreversibility)*. All assets in the compromised custodial wallet must be treated as potentially lost.

**Key custody failure**: The operational procedures for key access fail — e.g., the required multi-sig quorum is unavailable (death, incapacitation, or departure of a keyholder without proper succession planning).

### 5.3 Associated Float Shield

**stale_canonical_float**: Key custody failure that prevents thorn anchor verification produces stale_canonical_float — the difference between the float register snapshot balance and the on-chain confirmed balance. If the thorn anchor is blocked due to key custody failure, the entire position is classified as stale_canonical_float until the anchor is cleared *(D2 — Private Key Custody)*.

### 5.4 Monitoring Standard — Thorn Anchor Blocked

A blocked thorn anchor is the primary R4 indicator. When detected:

1. Immediately classify all positions on the affected custodial wallet as stale_canonical_float
2. Declare thorn anchor blocked status — a structural R4 finding
3. Suspend all on-chain transactions from the affected custodial wallet
4. Escalate to treasury management and CFO within one clearing epoch
5. Initiate key recovery procedures per the documented recovery plan (C010 Section 2.3)

**Thorn anchor blocked requires immediate action, not a standard remediation schedule.** There is no acceptable grace period for a structural R4 finding.

---

## 6. Risk Category R5 — Custody Counterparty Risk

### 6.1 Definition

Custody counterparty risk is the risk that the qualified digital asset custodian (C001 vocabulary bridge: custodian bank → qualified digital asset custodian) fails operationally, financially, or legally, resulting in loss of or inability to access the treasury's assets *(C001 D2 — Private Key Custody)*.

This is the digital asset analog of the ETM8 Chapter 3 counterparty risk for bank relationships, but it is more severe in several respects:

- Digital asset custodians typically operate without deposit insurance (unlike FDIC-insured bank deposits)
- Custodian insolvency may result in assets being treated as custodian property in bankruptcy proceedings, depending on jurisdiction
- A custodian's operational failure can prevent access to assets even if the assets themselves are on-chain and nominally intact

### 6.2 Associated Float Shield

**speculative_collection_float** (cross-referenced from C010 Section 3.2 in R5 context): When a custodian reports an expected balance that has not been confirmed at pool confirmation, the treasury's float register reflects speculative_collection_float. Under R5, this arises when the custodian's reporting systems are unreliable or when the custodian has commingled client assets and the reported balance reflects an allocation rather than a confirmed on-chain position.

### 6.3 Monitoring Standard

R5 is monitored through:

1. Monthly reconciliation of all custodian-reported balances against independent on-chain verification. The custodian's float register snapshot must agree with the BLID chain anchor to within the instrument's native precision (C013 Section 3.3).

2. Quarterly review of custodian financial health, regulatory status, and insurance coverage per the due diligence criteria in C010 Section 6.

3. Any custodian balance discrepancy that cannot be explained within one anchor cycle triggers an immediate thorn anchor blocked classification for the affected custodian's custodial wallets.

---

## 7. Risk Category R6 — Market Risk

### 7.1 Definition

Market risk is the risk of loss due to adverse price movements in digital assets held in the portfolio. This extends ETM8 Chapter 17's market risk framework to the high-volatility, 24/7 trading environment of digital assets.

### 7.2 Conservative Valuation Standards

For risk assessment and scenario analysis purposes, conservative valuation floors must be established per asset class and reviewed quarterly. The methodology for computing floors is mandatory; specific values are updated by the treasury team based on current market data and must not be treated as static constants embedded in policy.

**Floor computation methodology**:

| Asset Class | Methodology |
|---|---|
| Volatile digital assets (BTC, XMR, ETH, SOL, etc.) | Lower of: (a) 30-day trailing minimum price on a primary exchange, and (b) 40% peak-to-trough drawdown applied to the most recent 12-month closing price |
| Regulated stablecoins (USDC, USDT) | USD 1.00 (par), subject to immediate peg break escalation per Section 7.3 if deviation exceeds 0.50% |
| Tokenized T-bills | Current NAV less a 1% secondary market liquidity haircut |

These floors are minimum scenario values for stress testing — not price predictions and not GAAP fair values (see C013 Section 2.1 for the distinction). A treasury that is solvent at these floors is demonstrating adequate capital conservation under stress.

**Update requirement**: Conservative floor values must be updated quarterly, or within 5 business days of any adverse price movement exceeding 20% on a relevant asset. Stale floors that no longer reflect current conditions are a C012 R6 governance failure.

**Conservative valuations are mandatory for**: CECR computation when positions are marked to market, Baumol floor computation, and any scenario analysis submitted under C012 or C011.

### 7.3 Peg Break Risk

Stablecoin positions carry peg break risk — the risk that the stablecoin loses its 1:1 peg to the referenced fiat currency. Peg break risk is a market risk event that can convert what appeared to be a zero-volatility position into a high-volatility loss.

Peg break monitoring: Any stablecoin position that trades at a discount of more than 0.50% to peg on a primary exchange triggers an active shield R6 condition. The treasury must evaluate redemption through the issuer's direct redemption mechanism (not secondary market) within one clearing epoch of the peg break signal.

### 7.4 Monitoring Standard

R6 market data feeds must run at each TMS loop cycle, comparing current market prices to:
- Conservative floor values (for risk exposure computation)
- Stablecoin peg (for peg break monitoring)
- Tokenized T-bill NAV (for yield instrument valuation)

---

## 8. Risk Category R7 — Liquidity Risk

### 8.1 Definition

Liquidity risk is the risk that the treasury cannot convert a position to the required medium (typically regulated stablecoin or fiat equivalent) within the required time horizon at an acceptable cost. This extends ETM8 Chapter 17's liquidity risk framework to digital asset portfolio concentration risk.

### 8.2 Concentration Risk Criteria

Digital asset liquidity risk applies the same concentration principles as ETM8 Chapter 3's bank concentration risk:

- No single protocol class should represent more than 40% of total portfolio AUM
- No single custodial wallet (or custodian) should hold more than 60% of total portfolio AUM
- No single yield instrument should represent more than 25% of the relevant capital stack bucket

Concentration above these thresholds requires a documented risk acceptance signed by the CFO or equivalent, reviewed quarterly.

### 8.3 Clearing Epoch Liquidity Ladder

The liquidity ladder for digital asset treasury maps to clearing epochs rather than business days:

- **Immediate liquidity** (≤5,000 ms clearing epoch): hot tier stablecoin positions
- **Same-clearing-epoch liquidity** (≤60 minutes): hot tier BTC, SOL, XRPL/Stellar native positions
- **Next-day liquidity** (≤24 hours): warm tier positions and liquid staking with short redemption queues
- **Extended liquidity** (>24 hours): cold tier staking positions and long-dated tokenized T-bills

The liquidity ladder must be maintained in the TMS and reviewed at each BLID anchor cycle. Any bucket that falls below the C011 capital stack allocation minimum by more than 15 percentage points triggers an R7 active shield condition.

### 8.4 Monitoring Standard

R7 is monitored at each TMS loop cycle. Persistent deployment gate failure (three or more consecutive failures per C011 Section 3.3) triggers an automatic R7 active shield classification.

---

## 9. Active Shield and Thorn Anchor Blocked — Escalation Matrix

| Status | Trigger | Required Actions | Escalation Level |
|---|---|---|---|
| Active shield — R1 | antinodeIndex increment, network stall, fee spike | Log Hermes event; restrict affected channel; monitor for HARD_HALT threshold | Operations lead |
| Active shield — R2 | TMS downtime, Hermes queue failure | Default all positions to Z=0; investigate; restore within 1 clearing epoch | Operations lead + IT |
| Active shield — R3 | Smart contract audit gap, proxy upgrade | Flag code-as-counterparty; suspend new deployments to affected contract | Treasury manager |
| Active shield — R6 | Price below conservative floor or peg break | Evaluate liquidation; freeze new positions in affected asset | Treasury manager + CFO |
| Active shield — R7 | Concentration breach, persistent gate failure | Rebalance plan required within 5 business days | Treasury manager |
| Thorn anchor blocked — R4 | Key custody failure, key recovery required | Suspend all transactions; immediate key recovery; CFO notification | CFO + external counsel |
| Thorn anchor blocked — R5 | Custodian balance discrepancy unresolved | Suspend custodian; independent on-chain verification; regulatory notification if required | CFO + legal |
| HARD_HALT | antinodeIndex ≥ 13 or tier = HALT | Suspend all deployment; compute opportunity cost; resolution per C008 S5.3 | CFO |

---

## 10. Integration with Other C-Series Addenda

- **C008** (Float Management): All sub-float categories in this taxonomy originate in or feed back to C008 CECR computation; HARD_HALT is the operational expression of R1 at threshold
- **C009** (Payment Rails): R3 assessment requirements apply to every smart contract counterparty in rail assessments; EVM gas exposure is a standing R1 (Protocol Risk) flag for any EVM-rail position
- **C010** (Custody Architecture): R4 and R5 monitoring depend on C010 thorn anchor and BLID chain infrastructure; SoD failures (coupling_float) are the R3/R4 bridge
- **C011** (Yield Policy): Deployment gate failure triggers R7; validator concentration (D6) in 90-day bucket feeds R1 and R6 inputs
- **C013** (Accounting): GL accounts for each float shield category per C013 Section 4; HALT float reserve quantifies R1 opportunity cost
- **C014** (TMS Integration): Hermes event queue is the primary delivery mechanism for R1–R7 flag events; TMS loop interval (≤90s) determines the latency of risk detection

---

## Source Reference

> *Essentials of Treasury Management, 8th Edition (ETM8).*
> Association for Financial Professionals (AFP). Rockville, MD, 2025.
> © 2025 Association for Financial Professionals. All rights reserved.
> All proprietary rights to the examination, including copyright, are held by AFP.
