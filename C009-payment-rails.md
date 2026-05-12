> **PROPOSED APPROACH** — Not an official AFP publication. Not reviewed or endorsed by
> the Association for Financial Professionals. This document proposes how existing AFP/CTP
> principles from *Essentials of Treasury Management, 8th Edition* may apply to digital
> asset treasury operations, and is offered for professional discussion only.
> **Author:** Garzaro &nbsp;|&nbsp; **Status:** Proposed — subject to revision

# AFP-CA-ETM8-C009 — Payment Rail Assessment Standards

> Base standard: *Essentials of Treasury Management*, 8th Edition (ETM8). AFP, 2025.
> Document ID: AFP-CA-ETM8-C009
> Series index: AFP-CA-ETM8-INDEX (C000-series-index.md)
> Prerequisite: AFP-CA-ETM8-C001 (Practitioner's Bridge — vocabulary bridge and delta concepts)

---

## 1. Scope and ETM8 Alignment

This addendum extends ETM8 Chapter 4 (Payment Instruments and Systems) to digital asset payment rails. ETM8 Chapter 4 evaluates payment instruments across dimensions of availability, cost, certainty, finality, and operational risk. That evaluation framework was designed for bank-operated payment networks in which fees are negotiated, clearing is reversible, and settlement finality is legally defined by the operating rules of the clearing house (consensus protocol).

Digital asset payment rails require a materially different evaluation methodology because:

1. Finality is probabilistic and irreversible from the moment of broadcast *(C001 D1 — Transaction Irreversibility)*
2. Fees may be deterministic (CTP-eligible) or auction-based (CTP-ineligible), depending on protocol class
3. The consensus protocol (C001 vocabulary bridge: clearing house → consensus protocol) is software, not an institution; it has no appeals process and no reversal mechanism
4. Settlement occurs 24/7 without banking-hours dependency *(C001 D4 — 24/7 Clearing)*

This document provides:

- A CTP-standard evaluation framework for each payment rail dimension
- CTP eligibility classifications (eligible / ineligible) per rail
- The pre-broadcast authorization gate, required for all CTP-eligible operations
- A priority selection order for payment rail deployment
- EVM gas exposure flag requirements

All terms follow the C001 vocabulary bridge. "Payment network fee" as used in ETM8 maps to "protocol fee (deterministic)" for CTP-eligible rails in this addendum.

---

## 2. Payment Rail Evaluation Framework

Each payment rail is assessed across five dimensions. The assessment produces a CTP eligibility determination that governs whether the rail may be used in CTP-certified treasury operations.

### 2.1 Dimension 1 — Protocol Class

The protocol class determines the clearing epoch derivation rule (C008 Section 2.2) and the nature of finality. Protocol class is a fixed attribute of the rail, not adjustable by the practitioner.

Assessment question: *What consensus mechanism governs settlement?*

Acceptable protocol class values: `pow`, `pos`, `poh`, `dpos`, `l2-optimistic`, `l2-zk`, `payment-channel`, `stablecoin`, `contract`, `cbdc`

### 2.2 Dimension 2 — Fee Model

The fee model determines CTP eligibility. Only deterministic fee models are CTP-eligible.

| Fee Model | CTP Status | Description |
|---|---|---|
| Fixed minimum | Eligible | A protocol-defined floor fee (e.g., XRPL 10 drops) that does not vary with congestion |
| Rate-on-amount | Eligible | A fixed rate applied to a known transfer amount, computable at instruction time |
| Path-bounded variable | Eligible | A variable fee bounded by path-find result at instruction time (Lightning Network) |
| Auction-based | **Ineligible** | Fee is set by competitive bidding at broadcast time; cost is unknown at instruction *(C001 D3 — Protocol-Layer Risk)* |

EVM gas is the primary example of an auction-based fee model. All EVM-rail assessments must carry an explicit EVM gas exposure flag (see Section 6).

### 2.3 Dimension 3 — Confirmation Depth

**Confirmation depth** is the number of blocks that must accumulate after the block containing an on-chain transaction (C001 vocabulary bridge: wire transfer → on-chain transaction, protocol rail) before the transaction is treated as settled for treasury purposes *(C001 vocabulary bridge: settlement finality → confirmation depth, n-block probabilistic)*.

Unlike bank settlement finality, which is legally defined at a specific moment, confirmation depth is probabilistic: each additional block reduces the probability of chain reorganization, but no finite depth provides absolute finality *(C001 D1 — Transaction Irreversibility)*.

**CTP confirmation depth standards:**

| Protocol Class | Minimum Confirmation Depth | Approximate Time at Minimum |
|---|---|---|
| `pow` — BTC | 6 blocks | ~60 minutes |
| `pos` — ETH | 1 finalized epoch (2 epochs ≈ 12.8 min) | ~13 minutes |
| `poh` — SOL | 32 slots (optimistic) | ~13 seconds |
| `dpos` | 2/3 supermajority round | Protocol-dependent |
| `payment-channel` | Path-find confirmed + HTLC expiry clear | Seconds (off-chain routing) |
| `stablecoin` — XRPL | 1 ledger close | ~4 seconds |
| `stablecoin` — Stellar | 1 ledger close | ~5 seconds |
| `l2-zk` | 1 L1 proof finalization | Minutes to hours (proof generation) |
| `l2-optimistic` | Post-challenge-period | 7 days (rarely CTP-eligible) |

### 2.4 Dimension 4 — Reversibility Profile

All on-chain transactions are irreversible upon broadcast and confirmation. This is not a risk to be mitigated after the fact — it is an architectural property of all digital asset rails without exception *(C001 D1 — Transaction Irreversibility)*.

The reversibility profile assessment confirms that the pre-broadcast authorization gate (Section 4) is in place and that operational controls for on-chain transactions are categorically different from wire transfer controls.

Assessment question: *Are all authorization controls applied before broadcast? Are no post-broadcast correction mechanisms assumed?*

### 2.5 Dimension 5 — Private Key Exposure

Every on-chain transaction requires signing with a private key associated with the originating custodial wallet (C001 vocabulary bridge: bank account → custodial wallet). Key exposure during signing is the primary operational risk unique to digital asset rails *(C001 D2 — Private Key Custody)*.

Assessment question: *Does the rail's operational workflow require the private key to be present in a hot environment? Does key exposure duration exceed the minimum required for signing?*

---

## 3. CTP-Eligible Payment Rails

The following rails meet all five assessment dimensions for CTP-eligible operations when operated within the specified parameters.

### 3.1 XRPL (XRP Ledger)

- **Protocol class**: `stablecoin` — applies to both USDC/USDT issued on XRPL and native XRP transactions. XRPL uses Federated Byzantine Agreement (FBA) with a published Unique Node List (UNL), not proof-of-stake. FBA/UNL produces ledger-close-equivalent finality consistent with the `stablecoin` protocol class derivation rule in C008 Section 2.2.
- **Fee model**: Fixed minimum — **10 drops** (0.00001 XRP). Deterministic at instruction time.
- **Confirmation depth**: 1 ledger close, approximately 3,000–5,000 ms clearing epoch
- **Reversibility profile**: Irreversible at ledger close; pre-broadcast gate required *(D1)*
- **Key exposure**: Signing occurs locally; key does not transit the network
- **CTP status**: **Eligible**
- **Protocol fee (deterministic)** (C001 vocabulary bridge: payment network fee → protocol fee (deterministic)): 10 drops per transaction

**Practitioner note**: XRPL's consensus protocol is not proof-of-work; it relies on a unique node list (UNL) mechanism. Validator concentration risk *(C001 D6 — Validator Concentration Risk)* applies; practitioners should assess UNL diversity before allocating material transaction volume.

### 3.2 Stellar

- **Protocol class**: `stablecoin` (for USDC/USDT on Stellar)
- **Fee model**: Fixed minimum — **100 stroops** (0.00001 XLM). Deterministic at instruction time.
- **Confirmation depth**: 1 ledger close, approximately 5,000 ms clearing epoch
- **Reversibility profile**: Irreversible at ledger close; pre-broadcast gate required *(D1)*
- **Key exposure**: Local signing; key does not transit
- **CTP status**: **Eligible**
- **Protocol fee (deterministic)**: 100 stroops per operation

### 3.3 Solana

- **Protocol class**: `poh` (Proof of History)
- **Fee model**: Fixed base — **5,000 lamports per signature**. Additional priority fee exists (auction-based) and is CTP-ineligible if used. Base fee only is CTP-eligible.
- **Confirmation depth**: 32 slots (optimistic), approximately 12,800 ms clearing epoch
- **Reversibility profile**: Irreversible at slot confirmation; pre-broadcast gate required *(D1)*
- **Key exposure**: Local signing
- **CTP status**: **Eligible** — base fee only. Priority fee usage triggers EVM gas exposure flag treatment.
- **Protocol fee (deterministic)**: 5,000 lamports per signature (base fee)

**Practitioner note**: Solana's `poh` architecture provides extremely short clearing epochs, making it suitable for high-frequency treasury operations. However, the network has experienced periodic stalls; practitioners should monitor antinodeIndex per C008 Section 5 and maintain fallback rail designation.

### 3.4 Lightning Network (BTC Payment Channel)

- **Protocol class**: `payment-channel`
- **Fee model**: `base_fee + (fee_rate × forwarded_amount)` — bounded at path-find time. The path-find result establishes an upper bound that does not change between path-find and payment execution.
- **Confirmation depth**: Off-chain routing; HTLC (Hash Time-Locked Contract) expiry provides on-chain fallback. For treasury purposes, treat confirmed path-find + routing success as settlement.
- **Reversibility profile**: Payments are irreversible once the HTLC preimage is revealed; pre-broadcast gate required *(D1)*. Failed payments are not irreversible — failed HTLCs time out.
- **Key exposure**: Hot channel keys are required for routing operations; this is an elevated *(D2 — Private Key Custody)* exposure relative to cold-custody rails
- **CTP status**: **Eligible** — within path-find-bounded fee and with hot-key controls per C010
- **Protocol fee (deterministic)**: base_fee + (fee_rate × forwarded_amount), bounded at path-find

---

## 4. Pre-Broadcast Authorization Gate

### 4.1 Requirement

**All controls governing an on-chain transaction must be applied before broadcast. No post-broadcast correction is possible** *(C001 D1 — Transaction Irreversibility)*.

This is the most operationally significant difference from ETM8 Chapter 4's wire transfer (on-chain transaction) control framework. ETM8 wire transfer controls include recall procedures and same-day reversal capabilities. These do not exist for on-chain transactions on any CTP-eligible rail.

### 4.2 Gate Components

The pre-broadcast authorization gate must include, at minimum:

1. **Beneficiary verification**: On-chain address verified against an approved counterparty register before transaction construction. Address verification must occur in the same session as signing; a previously verified address is not automatically re-verified for a new transaction *(D2 — Private Key Custody)*.

2. **Amount validation**: Transaction amount validated against authorization limits, Baumol floor (C011 Section 2), and net-30 forecast. The gate must reject transactions that would cause post-transaction balance to fall below the Baumol floor.

3. **Protocol fee confirmation**: Protocol fee (deterministic) confirmed as CTP-eligible (not auction-based) before construction. If the fee model cannot be confirmed as deterministic, the transaction must not proceed on a CTP-eligible basis.

4. **Dual-control sign-off**: Per C010 Section 3, the signing key and the transaction authorization must not reside in the same control domain. The person or system authorizing the payment must be distinct from the entity holding the signing key *(D2 — Private Key Custody)*.

5. **Channel CECR check**: The originating custodial wallet's channel CECR must be ≥ 0.60 (C008 CTP efficiency floor). A channel in RED tier or HALT status may not originate new on-chain transactions.

### 4.3 Gate Failure Handling

If any gate component fails, the on-chain transaction must not be broadcast. Gate failures must be logged in the float register snapshot with:

- The BLID transaction ID (C001 vocabulary bridge: SoD audit record → BLID transaction ID) of the failed authorization attempt
- The specific gate component that failed
- The timestamp of failure (system timestamp acceptable for gate logs; block timestamp applies only to confirmed on-chain events per C013)

---

## 5. Payment Rail Selection Order

When a treasury operation could be executed on multiple rails, the following priority order applies. Lower-numbered rails should be exhausted before advancing to higher-numbered rails.

| Priority | Rail | Rationale |
|---|---|---|
| 1 | Regulated stablecoin on XRPL or Stellar | Shortest clearing epoch; lowest deterministic fee; highest CECR potential |
| 2 | Lightning Network (BTC payment channel) | Sub-second routing for established channels; CTP-eligible within path bounds |
| 3 | XRPL native / Stellar native | Direct ledger settlement; CTP-eligible fee model |
| 4 | On-chain BTC | Established, deeply liquid; longer clearing epoch (600,000 ms `pow`); higher fee relative to amount |
| 5 | On-chain Solana (base fee only) | Short clearing epoch; network stall risk requires antinodeIndex monitoring |
| 6 | EVM-rail (last resort) | Auction-based gas fee; CTP-ineligible; EVM gas exposure flag required; use only when no CTP-eligible alternative exists |

**Selection documentation requirement**: When a rail other than Priority 1 is selected, the float register snapshot entry for that transaction must include a selection rationale field documenting why higher-priority rails were not used.

---

## 6. EVM Gas Exposure Flag

### 6.1 Definition

The **EVM gas exposure flag** is a mandatory field on all float register snapshot entries involving on-chain transactions on EVM-compatible networks (Ethereum mainnet, Polygon PoS, BNB Chain, Avalanche C-Chain, and equivalent EVM-compatible L1/L2 networks operating with auction-based gas pricing).

Flag values:
- `present` — the on-chain transaction is subject to auction-based gas pricing *(D3 — Protocol-Layer Risk)*
- `none` — the on-chain transaction uses a deterministic, CTP-eligible fee model

### 6.2 Implications of `present` Flag

A float register entry with EVM gas exposure = `present`:

- May not be included in CECR numerator calculations as a Z=1 (in-transit) position for CTP efficiency reporting
- Must be reported separately from CTP-eligible positions in any audit or certification submission
- Does not automatically disqualify the transaction from operational use, but does disqualify it from CTP float efficiency claims
- Must carry a protocol fee notation using the observed gas cost (post-confirmation) rather than a pre-instruction estimate, clearly labeled as "non-deterministic, CTP-ineligible"

### 6.3 EVM Gas and CECR Exclusion

EVM-rail positions are excluded from CECR calculations. A treasury that conducts significant on-chain activity on EVM rails will see its effective CECR diluted when EVM positions are correctly excluded. This dilution is not a measurement error — it accurately reflects the CTP-ineligibility of those positions.

Practitioners managing mixed portfolios (CTP-eligible and EVM-rail positions) must maintain separate register segments and report CECR on the CTP-eligible segment only.

---

## 7. Smart Contract Counterparty on Payment Rails

On-chain transactions that route through or settle against a smart contract (rather than a direct custodial wallet transfer) introduce additional risk that does not exist on traditional payment rails *(C001 D5 — Smart Contract Counterparty)*.

For payment rail assessment purposes, a smart contract counterparty exists when:

- The on-chain transaction's recipient address is a contract, not an externally-owned address (EOA)
- Settlement depends on contract-defined logic (e.g., atomic swaps, DEX routing, bridge contracts)
- The fee or settlement amount is determined by contract state at execution time rather than pre-broadcast

Smart contract counterparty assessments must include:

1. Identification of the governing smart contract address and version
2. Audit status of the contract (third-party audit date and scope)
3. Upgrade/proxy risk: can the contract logic be changed without the treasury's consent?
4. Oracle dependency: does the contract rely on an external price or data feed that could be manipulated?

All smart contract counterparty positions must carry the flag "code-as-counterparty" in the float register entry and are subject to the C012 R3 risk category (Smart Contract / Bridge Risk).

---

## 8. Integration with Other C-Series Addenda

- **C008** (Float Management): clearingMs values for this document's rail table feed directly into C008 protocol class clearing epoch derivations
- **C010** (Custody Architecture): Pre-broadcast authorization gate (Section 4.2) depends on C010 SoD controls for dual-control sign-off and key custody segregation
- **C011** (Yield Policy): Payment rail selection order (Section 5) applies to capital deployment transactions; Baumol floor check (Section 4.2, gate component 2) links to C011 Section 2
- **C012** (Risk Taxonomy): EVM gas exposure (Section 6) feeds R1 (Protocol Risk); smart contract counterparty (Section 7) feeds R3 (Smart Contract / Bridge Risk)
- **C013** (Accounting): EVM gas exposure flag and selection rationale fields are minimum audit trail requirements per C013 Section 3
- **C014** (TMS Integration): TMS must implement pre-broadcast authorization gate enforcement and EVM gas exposure flag automation

---

## Source Reference

> *Essentials of Treasury Management, 8th Edition (ETM8).*
> Association for Financial Professionals (AFP). Rockville, MD, 2025.
> © 2025 Association for Financial Professionals. All rights reserved.
> All proprietary rights to the examination, including copyright, are held by AFP.
