> **PROPOSED APPROACH** — Not an official AFP publication. Not reviewed or endorsed by
> the Association for Financial Professionals. This document proposes how existing AFP/CTP
> principles from *Essentials of Treasury Management, 8th Edition* may apply to digital
> asset treasury operations, and is offered for professional discussion only.
> **Author:** Garzaro &nbsp;|&nbsp; **Status:** Proposed — subject to revision

# AFP-CA-ETM8-C010 — Custody Architecture and Internal Controls

> Base standard: *Essentials of Treasury Management*, 8th Edition (ETM8). AFP, 2025.
> Document ID: AFP-CA-ETM8-C010
> Series index: AFP-CA-ETM8-INDEX (C000-series-index.md)
> Prerequisite: AFP-CA-ETM8-C001 (Practitioner's Bridge — vocabulary bridge and delta concepts)

---

## 1. Scope and ETM8 Alignment

This addendum extends ETM8 Chapter 3 (Banks and Financial Institutions) and Chapter 18 (Treasury Policies and Procedures) to digital asset custody architecture and internal controls. ETM8 Chapter 3 establishes standards for bank relationship management, account structure, and counterparty due diligence. ETM8 Chapter 18 governs the design of treasury policies, segregation of duties (SoD), and audit trail requirements.

Both chapters assume that custody of funds is held by an institutional depository, that transactions are reversible pending settlement, and that audit evidence is documentary. Digital asset custody operates on fundamentally different principles:

- Custody of assets is equivalent to custody of cryptographic private keys *(C001 D2 — Private Key Custody)*
- Transaction irreversibility means control failures cannot be corrected after broadcast *(C001 D1 — Transaction Irreversibility)*
- The custodian bank (C001 vocabulary bridge: custodian bank → qualified digital asset custodian) may be an institutional service, an on-premises hardware solution, or a protocol-managed key management system
- Audit evidence is on-chain and cryptographically verifiable; paper records are secondary

This document defines:

- The three-tier custody architecture (hot / warm / cold) and the asset allocation rules for each tier
- The extension of SoD principles to include the cryptographic dimension of key custody
- Float shield categories that arise from SoD failures specific to digital asset custody
- The BLID chain as the authoritative SoD audit trail, replacing paper-based SoD logs
- Recognition rules under the Bill of Rights III standard

---

## 2. Three-Tier Custody Architecture

Digital asset custody must be structured across three tiers, each with defined operational purpose, key custody requirements, and balance limits. The three-tier architecture is not optional — it is the minimum structure required for CTP-eligible digital asset treasury operations.

### 2.1 Tier 1 — Hot Custody (Operational)

**Purpose**: Funds required for same-day or intra-day on-chain transactions (C001 vocabulary bridge: wire transfers → on-chain transactions, protocol rail). Hot custody custodial wallets (C001 vocabulary bridge: bank accounts → custodial wallets) are online continuously and signing keys are accessible to automated or semi-automated systems.

**Key custody requirement**: Signing keys reside in a hardware security module (HSM) or equivalent key management service. Keys must not be stored in plaintext at any time. Hot custody does not qualify for "qualified cold custody" designation *(D2 — Private Key Custody)*.

**Balance limit**: Hot tier allocation should not exceed operational requirements for a 48-hour cycle. Excess balances above the 48-hour operational requirement must be swept to warm or cold tier per the treasury policy.

**CECR applicability**: All on-chain transactions originating from hot custodial wallets are subject to full CECR reporting (C008). Hot tier is the primary source of Z=1 in-transit positions.

**SoD requirement**: The individual or system authorized to initiate on-chain transactions must not have unilateral access to the signing key. At minimum, transaction construction (defining recipient and amount) and transaction signing (private key operation) must be separated across control domains *(D2 — Private Key Custody)*.

### 2.2 Tier 2 — Warm Custody (Reserve)

**Purpose**: Reserve balances that may be needed within a 1–30 day window. Warm custody custodial wallets are not continuously online; signing keys are accessible through a deliberate activation process requiring human intervention.

**Key custody requirement**: Signing keys are stored in hardware (hardware wallets or HSM in offline state) and require a manual unlock process. Multi-signature (multi-sig) schemes requiring 2-of-3 or 3-of-5 keyholders are standard for warm custody.

**Balance limit**: Warm tier holds the balance designated for the 30-day capital stack bucket per C011 (40% allocation). Balances that are not expected to be needed within 30 days should be migrated to cold tier.

**CECR applicability**: Warm tier positions are typically Z=0 (stale) in CECR terms because they are not in active clearing epochs. Warm tier contributes to canonical_gap_float if balance snapshots are not reconciled (BLID chain anchored) within each anchor cycle.

**SoD requirement**: Warm tier signing requires a quorum of keyholders. No single keyholder may sign for warm tier without the required co-signers. The warm tier activation process must be logged in the BLID chain with the identity (or role designation) of each signing participant.

### 2.3 Tier 3 — Cold Custody (Strategic)

**Purpose**: Long-term strategic reserves and the 90-day yield tier per C011. Cold custody custodial wallets are air-gapped; signing keys are physically isolated from networked systems.

**Key custody requirement**: Signing keys meet the **qualified cold custody** standard, defined as:

1. Private keys generated on an air-gapped device that has never been connected to a network
2. Keys stored on hardware wallets or metal backup media, never on networked storage
3. Physical access to cold custody hardware is controlled by dual-person integrity (DPI) — no single person can access cold custody keys without a designated second party
4. Cold custody key recovery procedures are documented, tested at least annually, and stored separately from the keys themselves

**Qualified cold custody is required** for any position designated to the 90-day yield tier under C011. Unqualified cold custody (e.g., software wallets held by a single custodian in an offline environment but without DPI) does not satisfy the qualified cold custody standard *(D2 — Private Key Custody)*.

**Balance limit**: Cold tier holds the 90-day capital stack allocation (30% by C011) plus any strategic reserve above operational and reserve requirements.

**SoD requirement**: Cold tier transactions require DPI sign-off for key access, plus a separate authorization from treasury management for the transaction itself. The authorization must precede key access; key access must not be used to construct and authorize a transaction simultaneously.

---

## 3. Segregation of Duties — Cryptographic Extension

ETM8 Chapter 18 establishes SoD as a foundational control, requiring that no single individual can initiate, authorize, and record a transaction. Digital asset treasury extends this requirement to include the cryptographic dimension: the **signing key** represents a distinct control domain that must be separated from both authorization and recording.

### 3.1 The Four-Domain Model

Digital asset SoD requires separation across four control domains:

| Domain | Function | Who/What |
|---|---|---|
| Authorization | Approves the transaction (amount, recipient, purpose) | Treasury officer / approval workflow |
| Construction | Builds the on-chain transaction object | Treasury operations / TMS system |
| Signing | Applies the private key cryptographic signature | Key custodian (hardware) |
| Recording | Enters the transaction in the float register and GL | Finance / accounting system |

**Critical restriction**: No individual, role, or automated system may hold responsibility for more than two adjacent domains simultaneously. The most critical separation is Authorization ↔ Signing — an individual who can both authorize and sign can unilaterally move funds *(D2 — Private Key Custody)*.

### 3.2 SoD Failures and Float Shield Categories

ETM8 Chapter 18 identifies SoD failures as audit findings. In digital asset treasury, SoD failures produce quantifiable float shield categories that must be reported in the CECR framework (C008).

**coupling_float**: Arises when the GL recorder (Domain 4 — Recording) also holds signing authority (Domain 3 — Signing). The coupling between recording and signing creates an opportunity to record a transaction differently from how it was executed on-chain, without external detection.

- Detection: Compare BLID chain (C001 vocabulary bridge: reconciliation → BLID chain / thorn anchor) anchor hashes to GL entries for each clearing epoch
- Remediation: Immediately reassign signing authority to a role that has no GL recording access; recompute CECR excluding affected positions pending review

**stale_canonical_float**: Arises when the custodian-reported balance (from the float register snapshot, C001 vocabulary bridge: bank statement → float register snapshot) does not match the on-chain confirmed balance at the thorn anchor point. This is the digital asset equivalent of a bank reconciliation failure.

- Detection: Automated BLID chain anchor comparison at each anchor cycle
- Remediation: Suspend CECR reporting on the affected channel; initiate reconciliation; do not proceed to yield deployment (C011) until stale_canonical_float is resolved

**speculative_collection_float**: Arises when a collection is recognized in the GL before the on-chain transaction achieves the required confirmation depth (pool confirmation). This violates Bill of Rights III (Section 5 of this document) and creates financial statements that overstate available balances.

- Detection: Compare GL recognition timestamps to on-chain confirmation timestamps and block timestamps (C013 Section 3.2)
- Remediation: Reverse the premature recognition; re-record at the correct block timestamp

### 3.3 SoD Documentation Requirements

The treasury policy (ETM8 Chapter 18 analog) must document:

1. The specific role assignments for each of the four SoD domains
2. The process for temporary role assignment when a primary role-holder is unavailable (succession must not collapse SoD)
3. The multi-sig threshold and keyholder identities or roles for warm and cold tier custodial wallets
4. The escalation path for SoD exceptions, which must be approved by the CFO or equivalent officer and logged in the BLID chain

---

## 4. BLID Chain as SoD Audit Trail

### 4.1 Definition and Function

The **BLID chain** (C001 vocabulary bridge: reconciliation → BLID chain / thorn anchor) is the authoritative, tamper-evident audit trail for all digital asset treasury SoD events, replacing the paper SoD log. Each entry in the BLID chain is a **BLID transaction ID** (C001 vocabulary bridge: SoD audit record → BLID (booLangID_v2) transaction ID) — a hash-chained record that cannot be modified without invalidating all subsequent entries.

The BLID chain records:

- Each authorization event (Domain 1), with the authorizing role and timestamp
- Each signing event (Domain 3), with the signing key identifier (not the key itself) and timestamp
- Each CECR anchor cycle result (Z-state snapshot, CECR value, portfolio tier)
- Each HARD_HALT event and resolution (C008 Section 5)
- Each SoD exception approval

### 4.2 Thorn Anchor

A **thorn anchor** is a specific BLID chain entry that marks the result of a reconciliation cycle — the comparison of the float register snapshot balance to the on-chain confirmed balance. A healthy thorn anchor confirms that:

- The float register snapshot balance equals the on-chain balance at the block height of the anchor
- No stale_canonical_float exists on the anchored channel
- The CECR computation for the anchor period is validated

**Anchor frequency**: Thorn anchors must be issued at least once per the shorter of: (a) one clearing epoch, or (b) 90 seconds (the C014 loop interval). For cold tier custodial wallets where on-chain balance does not change frequently, a daily thorn anchor is the minimum.

**Thorn anchor blocked**: A thorn anchor is "blocked" when stale_canonical_float > 0 or when the on-chain balance cannot be confirmed due to protocol connectivity issues. A blocked thorn anchor is a structural R4/R5 finding under C012 and requires immediate action. No new on-chain transactions may originate from a custodial wallet with a blocked thorn anchor *(D2 — Private Key Custody)*.

### 4.3 BLID Chain Integrity Requirements

The BLID chain must satisfy:

1. **Hash chaining**: Each entry contains the hash of the previous entry, creating a chain where any modification to a historical entry invalidates all subsequent entries
2. **Immutability**: Entries cannot be deleted or overwritten; corrections are made by appending a correction entry that references the original by BLID transaction ID
3. **Independent verifiability**: The chain hash can be verified by any authorized party with read access, without requiring the system that produced it
4. **Off-chain backup**: The BLID chain must be backed up to a storage location independent of the primary treasury system at each anchor cycle

---

## 5. Bill of Rights III — Recognition at Pool Confirmation

### 5.1 Statement of the Rule

**Bill of Rights III**: Recognition of a digital asset receipt (collection) occurs at **pool confirmation** (the point at which an on-chain transaction reaches the required confirmation depth for the governing protocol class), not at submission (broadcast).

This rule has three corollaries:

1. **No early recognition**: A collection may not be recognized in the GL, float register, or management reports at any point before pool confirmation
2. **No system timestamp substitution**: Recognition timestamps must use the block timestamp from the confirming block, not the system clock at the time the treasury system detected confirmation *(C001 D1 — Transaction Irreversibility)* — see C013 for full accounting treatment
3. **No optimistic counting**: Unconfirmed on-chain transactions may not be included in the CECR numerator as Z=1 in-transit unless they are within the clearingMs window and the protocol's standard mempool acceptance has been confirmed

### 5.2 Rationale

ETM8 Chapter 3 treatment of bank account (custodial wallet) balances includes the concept of ledger balance versus collected balance — the difference being uncollected items. Bill of Rights III formalizes the analog for digital asset treasury: a broadcast-but-unconfirmed on-chain transaction is not a collected item and must not be treated as available balance.

### 5.3 Recognition Failure Remediation

When speculative_collection_float is detected (a collection recognized before pool confirmation):

1. Reverse the GL entry for the pre-confirmation recognition
2. Record a correction entry in the BLID chain citing the original BLID transaction ID
3. Re-record the recognition at the correct block timestamp of pool confirmation
4. Assess whether the error affected any CECR reports filed during the pre-confirmation period; if so, restate the affected CECR values in the next anchor cycle report

---

## 6. Qualified Digital Asset Custodian Due Diligence

ETM8 Chapter 3 provides a bank relationship management framework including financial strength assessment, service level evaluation, and counterparty risk review. The analog for digital asset treasury is a **qualified digital asset custodian** (C001 vocabulary bridge: custodian bank → qualified digital asset custodian) due diligence framework.

### 6.1 Eligibility Criteria

A qualified digital asset custodian must satisfy all of the following:

1. **Regulatory status**: Licensed as a digital asset custodian under applicable jurisdiction (examples: NYDFS BitLicense, Wyoming SPDI, OCC trust charter, or equivalent)
2. **Insurance**: Maintains crime/specie insurance covering digital asset theft, key compromise, and operational loss. Policy limits must be reviewed against portfolio AUM
3. **Audit**: Undergoes at least annual SOC 2 Type II or equivalent independent audit of custody controls, specifically including key management procedures
4. **Segregation**: Client assets held in segregated custodial wallets, not commingled with custodian proprietary holdings
5. **Key architecture**: Publishes a documented key architecture that demonstrates qualified cold custody capability for at-rest assets *(D2 — Private Key Custody)*

### 6.2 Disqualifying Conditions

An entity may not serve as qualified digital asset custodian for CTP-eligible operations if any of the following apply:

- Active regulatory enforcement action relating to custody, key management, or client fund segregation
- No independent audit of key management controls within the preceding 12 months
- Material balance discrepancy between custodian-reported balances and independently verifiable on-chain balances within the preceding 12 months
- Inability to produce a signed attestation of client asset segregation on demand

### 6.3 Custodian Concentration Risk

ETM8 Chapter 3 addresses bank concentration risk — the risk of overreliance on a single depository institution. The same principle applies to qualified digital asset custodians. A treasury should not hold more than 60% of total digital asset portfolio AUM with any single custodian. Concentration above this threshold requires a documented risk acceptance signed by the CFO or equivalent.

---

## 7. Integration with Other C-Series Addenda

- **C008** (Float Management): SoD failures (Section 3.2) map directly to float shield categories; BLID chain anchor cycles set the CECR reporting cadence
- **C009** (Payment Rails): Pre-broadcast authorization gate requires the SoD separation defined in Section 3.1; qualified cold custody (Section 2.3) is required for 90-day yield tier operations
- **C011** (Yield Policy): Qualified cold custody requirement (Section 2.3) is the prerequisite for 90-day capital stack allocation
- **C012** (Risk Taxonomy): stale_canonical_float, coupling_float, and speculative_collection_float (Section 3.2) map to R4, R5 risk categories; blocked thorn anchor is a structural R4/R5 finding
- **C013** (Accounting): BLID transaction ID is the primary reference key for all GL entries; Bill of Rights III governs the recognition timestamp requirement
- **C014** (TMS Integration): BLID chain issuance and thorn anchor generation are mandatory TMS functions (C014 Section 2)

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
