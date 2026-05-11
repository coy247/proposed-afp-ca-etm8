> **PROPOSED APPROACH** — Not an official AFP publication. Not reviewed or endorsed by
> the Association for Financial Professionals. This document proposes how existing AFP/CTP
> principles from *Essentials of Treasury Management, 8th Edition* may apply to digital
> asset treasury operations, and is offered for professional discussion only.
> **Author:** Garzaro &nbsp;|&nbsp; **Status:** Proposed — subject to revision

# AFP-CA-ETM8-C013 — Accounting and Audit Trail Standards

> Base standard: *Essentials of Treasury Management*, 8th Edition (ETM8). AFP, 2025.
> Document ID: AFP-CA-ETM8-C013
> Series index: AFP-CA-ETM8-INDEX (C000-series-index.md)
> Prerequisite: AFP-CA-ETM8-C001 (Practitioner's Bridge — vocabulary bridge and delta concepts)

---

## 1. Scope and ETM8 Alignment

This addendum extends ETM8 Chapter 8 (Financial Accounting and Reporting) to digital asset treasury accounting and audit trail standards. ETM8 Chapter 8 covers the application of U.S. GAAP to treasury transactions, financial statement presentation, and the documentation standards underlying treasury audit trails. It addresses cash and cash equivalents, short-term investments, and the reconciliation (BLID chain / thorn anchor, C001 vocabulary bridge) of bank statements (float register snapshots, C001 vocabulary bridge) to general ledger balances.

Digital asset accounting has been materially affected by recent accounting standard updates. FASB ASC 350-60, effective for fiscal years beginning after December 15, 2024, requires fair value measurement for digital assets, replacing the prior impairment-only model. This is the most significant change to digital asset accounting since the AICPA's 2019 practice aid on the topic.

This document establishes:

- The four minimum requirements of the C013 audit trail for digital asset transactions
- The accounting treatment of FASB ASC 350-60 fair value measurement
- Transaction purpose classification requirements for yield and conversion transactions
- GL account mapping for float shield categories and HALT reserve
- Integration requirements with the BLID chain as the primary evidence source

All terms follow the C001 vocabulary bridge.

---

## 2. Applicable Accounting Standards

### 2.1 FASB ASC 350-60 — Digital Assets (Effective Fiscal Years After December 15, 2024)

FASB ASC 350-60 was issued in December 2023 and became effective for fiscal years beginning after December 15, 2024. Under ASC 350-60:

1. **Measurement basis**: Digital assets within the scope of ASC 350-60 are measured at **fair value** at each reporting date, with changes in fair value recognized in net income for the period.

2. **Scope**: ASC 350-60 applies to digital assets that meet all of the following: (a) the asset is an intangible asset, (b) the asset is created or reside on a distributed ledger, and (c) the asset is secured through cryptography. Tokenized T-bills (C001 vocabulary bridge: money market fund → tokenized T-bill) and protocol staking positions (C001 vocabulary bridge: interest-bearing deposit → protocol staking / yield instrument) may fall under ASC 350-60 or under specific financial instrument standards depending on their legal structure; practitioners must obtain legal and accounting counsel on the classification of each instrument.

3. **Level of fair value hierarchy**: Digital assets traded on active markets typically qualify as Level 1 (quoted prices in active markets). Less liquid digital assets may require Level 2 or Level 3 measurement. CTP conservative valuation floors (C012 Section 7.2) are not GAAP fair values — they are internal risk management floors and must not be substituted for GAAP fair value in financial statements.

4. **Balance sheet presentation**: Digital assets measured at fair value under ASC 350-60 are presented separately from cash and cash equivalents. Digital assets may not be classified as cash equivalents unless they meet the definition of a cash equivalent (original maturity of three months or less AND readily convertible to a known amount of cash).

5. **Stablecoin classification**: Regulated stablecoins held primarily as a medium of exchange and denominated in a stable reference currency may be classified as cash equivalents if they are redeemable at par through an established primary redemption mechanism. This classification requires ongoing assessment; a peg break event (C012 Section 7.3) would require immediate reclassification.

### 2.2 Relationship to ETM8 Chapter 8

ETM8 Chapter 8's bank reconciliation framework maps to digital asset treasury as follows:

| ETM8 Chapter 8 Concept | Digital Asset Treasury Equivalent |
|---|---|
| Bank statement | Float register snapshot (C001 vocabulary bridge) |
| Book balance | GL balance at custodial wallet level |
| Bank reconciliation | BLID chain thorn anchor comparison (C001 vocabulary bridge) |
| Outstanding checks | On-chain transactions broadcast but not yet confirmed (Z=1 pool) |
| Deposits in transit | Inbound on-chain transactions broadcast but not yet confirmed |
| Wire trace number | Transaction hash (primary reference key — see Section 3.1) |
| Settlement finality | Confirmation depth (n-block probabilistic) *(C001 D1 — Transaction Irreversibility)* |

---

## 3. C013 Audit Trail — Four Minimum Requirements

The C013 audit trail is the minimum set of data elements that must be captured and preserved for every on-chain transaction (C001 vocabulary bridge: wire transfer → on-chain transaction, protocol rail) in a CTP-eligible digital asset treasury. These four requirements are not optional and may not be substituted.

### 3.1 Requirement 1 — Transaction Hash as Primary Reference Key

The **transaction hash** (the cryptographic hash of the on-chain transaction, broadcast and recorded by the consensus protocol) is the primary reference key for all audit trail entries.

This is the direct analog of the wire trace number in ETM8 Chapter 8. Just as a wire trace number is the immutable, unique identifier of a wire transfer (on-chain transaction) assigned by the clearing house (consensus protocol), the transaction hash is assigned by the protocol at broadcast and cannot be changed.

**Implementation requirements**:

- Every GL journal entry, float register snapshot entry, and BLID transaction ID record involving an on-chain transaction must include the transaction hash as a field
- Transaction hashes must be stored in their full native format (typically hexadecimal, 64 characters for SHA-256-based hashes, 88 characters for ed25519-based signatures on Solana)
- No abbreviation, truncation, or aliasing of transaction hashes is permitted in audit trail records
- The transaction hash field must be indexed and searchable in the TMS audit log

### 3.2 Requirement 2 — Block Timestamp as Authoritative Event Anchor

The **block timestamp** from the confirming block is the authoritative timestamp for recognition of all on-chain events. The system timestamp (the time recorded by the treasury system when it detected or processed the event) must not be used as the authoritative event timestamp *(C001 D1 — Transaction Irreversibility)*.

This requirement arises from the nature of the consensus protocol (C001 vocabulary bridge: clearing house → consensus protocol) as the source of truth for on-chain events. The ledger does not accept corrections. If the treasury's system detected a confirmation one minute after the block was produced, the event occurred at the block timestamp, not at the detection time.

**Implication for SoD**: The block timestamp requirement means that retrospective recognition manipulation — recording an event with a favorable system timestamp — is detectable by comparing the GL entry date to the on-chain block timestamp retrieved from the BLID chain (C001 vocabulary bridge: reconciliation → BLID chain / thorn anchor).

**Implementation requirements**:

- The TMS must retrieve the block timestamp from the on-chain data (not from a local clock) at the time of confirmation detection
- Block timestamps must be stored in UTC with millisecond precision
- A separate "system detection timestamp" field may be included in the audit trail for operational purposes but must be clearly labeled as distinct from the authoritative block timestamp
- The corrected_friction_ms (C001 vocabulary bridge: float period → corrected_friction_ms / clearing epoch) computation uses block timestamps, not system timestamps, to measure clearing epoch elapsed time

### 3.3 Requirement 3 — Native Precision Maintained

Digital assets are divisible to extremely small units that have economic significance at scale. The audit trail must maintain **native asset precision** — the full decimal representation of the asset as defined by the protocol — throughout all records.

**Minimum precision requirements**:

| Asset / Protocol | Native Unit | Precision Required |
|---|---|---|
| BTC | Satoshi | 8 decimal places (0.00000001 BTC = 1 satoshi) |
| XMR | Piconero | 12 decimal places (0.000000000001 XMR = 1 piconero) |
| ETH | Wei | 18 decimal places |
| SOL | Lamport | 9 decimal places |
| XRPL (XRP) | Drop | 6 decimal places |
| Stellar (XLM) | Stroop | 7 decimal places |
| USDC (all chains) | Micro-USDC | 6 decimal places |

The minimum precision standard for C013 purposes is **≥15 decimal places** for satoshi/piconero precision, ensuring that BTC and XMR audit trails can represent values without rounding error at any reasonable treasury scale.

**Implementation requirements**:

- All amounts in the audit trail, float register, and GL must be stored as fixed-point or high-precision decimal numbers, not floating-point numbers
- Floating-point (IEEE 754 double precision) representation is prohibited for transaction amounts in the audit trail due to rounding error at high precision
- Fiat equivalent valuations (for ASC 350-60 fair value purposes) are stored separately from native asset amounts and clearly labeled as derived values

### 3.4 Requirement 4 — Channel Identifier on All Register Entries

Every entry in the float register snapshot and every GL journal entry related to a digital asset on-chain transaction must carry a **channel identifier** that specifies:

1. The custodial wallet address (or custodial wallet ID as maintained in the TMS)
2. The protocol class governing the transaction
3. The liquidity tier (hot / warm / cold per C010)

The channel identifier is the minimum data element required to associate a GL entry with:
- Its CECR computation (C008) via the custodial wallet × protocol class pairing
- Its SoD domain assignment (C010) via the custody tier
- Its risk category classification (C012) via protocol class and instrument type
- Its yield bucket assignment (C011) via custody tier and clearing epoch

**Implementation requirements**:

- Channel identifiers must be assigned in the TMS before any on-chain transaction is broadcast; a transaction without an assigned channel identifier must not be broadcast
- Channel identifiers are immutable once assigned to a transaction; if the channel assignment is incorrect, a correcting journal entry with a new BLID transaction ID references the original but does not alter the original's channel identifier

---

## 4. Transaction Purpose Classification

### 4.1 Purpose Classification Gap

A known schema gap exists between the current C013 audit trail requirements and the full set of yield and conversion transaction types introduced by C011 instruments. Until this gap is closed — by completing the instrument classification schema per the requirement below — the GL cannot correctly segregate yield earned from return of principal, or classify conversion transactions for ASC 350-60 purposes.

**This gap must be closed before any C011 instruments are added to the active portfolio.** Adding tokenized T-bills, protocol staking positions, or overcollateralized lending to the portfolio without purpose classification creates financial statement risk under ASC 350-60.

### 4.2 Required Purpose Classifications

Each on-chain transaction in the audit trail must be classified with one of the following purpose codes. This list is mandatory and must be extended (through the treasury policy change management process) when new instrument types are introduced.

| Purpose Code | Description | CECR Impact | ASC 350-60 Treatment |
|---|---|---|---|
| `TRANSFER_OPERATIONAL` | On-chain transfer for operational payment or disbursement | Z=1 in-transit; full CECR inclusion | Decrease in digital asset; recognize at confirmation |
| `TRANSFER_INTERNAL` | Custodial wallet-to-custodial wallet sweep (same beneficial owner) | Z=1 in-transit; CECR neutral (same portfolio) | No gain/loss; reclassification only |
| `YIELD_DEPLOYMENT` | Capital deployed to a yield instrument (staking, T-bill purchase) | Z=1; CECR credit on deployment channel | Investment; no immediate P&L |
| `YIELD_RECEIPT` | Yield income received from a staking or lending position | Z=1; CECR credit | Revenue — Yield Earned; recognize at block timestamp |
| `YIELD_REDEMPTION` | Return of principal from a yield instrument | Z=1; full CECR inclusion | Return of investment; not revenue |
| `PROTOCOL_FEE` | Protocol fee paid on a transaction | Z=0 (fee is cost, not in-transit) | Operating expense — Protocol Fee Cost |
| `CONVERSION` | Exchange of one digital asset for another | Z=1 on outbound; new Z=1 on inbound | Disposition at fair value; recognize gain/loss under ASC 350-60 |
| `HALT_FREEZE` | Transaction associated with a HARD_HALT event | Z=0 (HALT state) | HALT Float Reserve — accrue opportunity cost per C011 S5 |
| `SoD_CORRECTION` | Correction entry for a prior misclassification | Z=0 (correcting entry) | Correcting journal entry; reference original BLID transaction ID |

### 4.3 Yield and Conversion — Detailed Treatment

**Yield earned** (purpose code `YIELD_RECEIPT`): Under ASC 350-60, yield income received from a protocol staking / yield instrument (C001 vocabulary bridge: interest-bearing deposit → protocol staking / yield instrument) is recognized at the block timestamp when the inbound on-chain transaction achieves pool confirmation per C010 Bill of Rights III. Early recognition (before pool confirmation) creates speculative_collection_float and is prohibited.

**Conversion transactions** (purpose code `CONVERSION`): When digital assets are exchanged — for example, converting BTC to USDC — the disposition of the outbound asset must be recognized at fair value (ASC 350-60), with any difference between book value and fair value recognized in net income. The inbound asset is recorded at its fair value at the block timestamp of confirmation.

---

## 5. GL Account Mapping

The following GL account structure is the minimum required for CTP-eligible digital asset treasury. Accounts marked as mandatory must be established before any digital asset treasury operations begin.

| GL Account | Description | Feeds From | Mandatory |
|---|---|---|---|
| Digital Assets — Hot Tier | Fair value of hot tier custodial wallet positions (ASC 350-60) | Float register snapshot; pool confirmations | Yes |
| Digital Assets — Warm Tier | Fair value of warm tier positions | Float register snapshot; thorn anchors | Yes |
| Digital Assets — Cold Tier / Staking | Fair value of cold tier and yield instrument positions | Thorn anchors; yield instrument valuations | Yes |
| Float Cost Receivable | Accrued receivable for confirmed but not-yet-received yield income (protocol staking rewards accruing within an epoch but not yet paid to the custodial wallet) | Yield instrument terms; BLID chain | Yes |
| HALT Float Reserve | Accrued opportunity cost from HARD_HALT cycles per C011 Section 5 | Opportunity cost computation; C012 R1 events | Yes |
| Yield Earned | Revenue account for protocol staking yield and tokenized T-bill income received | `YIELD_RECEIPT` transactions at pool confirmation | Yes |
| Protocol Fee Cost | Expense account for deterministic protocol fees | `PROTOCOL_FEE` transactions | Yes |
| Unrealized Gain/Loss — Digital Assets | Fair value adjustment account (ASC 350-60 mark-to-market) | Period-end fair value remeasurement | Yes |
| Digital Asset — Conversion Gain/Loss | Realized gain or loss on conversion transactions | `CONVERSION` transactions at confirmation | Yes |
| CECR Float Adjustment | Adjustment account for CECR restatements following stale_canonical_float resolution | Thorn anchor corrections; CECR restatements | Recommended |

### 5.1 Period-End Close Procedures

The digital asset treasury period-end close must include:

1. Fair value remeasurement of all digital asset positions per ASC 350-60, using exchange pricing at the period-end block height
2. Thorn anchor issuance for all custodial wallets, confirming BLID chain reconciliation (C001 vocabulary bridge: reconciliation → BLID chain / thorn anchor) is current
3. CECR report for the period, with sub-float category balances at period end
4. HALT Float Reserve computation and accrual for any HALT cycles in the period
5. Float Cost Receivable reconciliation against confirmed yield receipts

### 5.2 Float Register Snapshot as the Source Document

The float register snapshot (C001 vocabulary bridge: bank statement → float register snapshot) is the source document for digital asset treasury GL entries, in the same way that bank statements serve as source documents for traditional treasury. The float register snapshot must:

- Be producible on demand for any historical date, at any block height
- Match the BLID chain thorn anchor state for the corresponding block height
- Be retained as an audit document for a minimum of 7 years (consistent with ETM8 Chapter 8 document retention guidance)

---

## 6. Integration with Other C-Series Addenda

- **C008** (Float Management): Block timestamps (Requirement 2) are the authoritative anchor for CECR epoch computations; corrected_friction_ms uses block timestamp differentials
- **C009** (Payment Rails): Transaction hash (Requirement 1) is required on all pre-broadcast gate records; EVM gas exposure flag appears as a protocol fee cost entry with the CTP-ineligible notation
- **C010** (Custody Architecture): Bill of Rights III (C010 Section 5) governs the recognition timing enforced by Requirement 2; BLID transaction ID is the primary GL reference key
- **C011** (Yield Policy): Purpose classification schema (Section 4) must be complete before any C011 instruments are deployed; HALT Float Reserve quantifies the C011 opportunity cost obligation
- **C012** (Risk Taxonomy): HALT Float Reserve is the GL expression of C012 R1 opportunity cost; float shield categories map to specific GL adjustment accounts
- **C014** (TMS Integration): TMS must automate all four audit trail requirements; block timestamp retrieval must occur within the TMS loop cycle, not on a deferred batch basis

---

## Standard Reference and Submission Information

> *Essentials of Treasury Management, 8th Edition (ETM8).*
> Association for Financial Professionals (AFP). Rockville, MD, 2025.
> © 2025 Association for Financial Professionals. All rights reserved.
> All proprietary rights to the examination, including copyright, are held by AFP.

Association for Financial Professionals
12345 Parklawn Drive, Suite 200, PMB 1001
Rockville, MD 20852
certification@AFPonline.org · +1 301.907.2862
