> **PROPOSED APPROACH** — Not an official AFP publication. Not reviewed or endorsed by
> the Association for Financial Professionals. This document proposes how existing AFP/CTP
> principles from *Essentials of Treasury Management, 8th Edition* may apply to digital
> asset treasury operations, and is offered for professional discussion only.
> **Author:** Garzaro &nbsp;|&nbsp; **Status:** Proposed — subject to revision

# AFP-CA-ETM8-C014 — Treasury Management System Integration

> Base standard: *Essentials of Treasury Management*, 8th Edition (ETM8). AFP, 2025.
> Document ID: AFP-CA-ETM8-C014
> Series index: AFP-CA-ETM8-INDEX (C000-series-index.md)
> Prerequisite: AFP-CA-ETM8-C001 (Practitioner's Bridge — vocabulary bridge and delta concepts)

---

## 1. Scope and ETM8 Alignment

This addendum extends ETM8 Chapter 15 (Technology in Treasury) to define the minimum Treasury Management System (TMS) capabilities required for CTP-eligible digital asset treasury operations. ETM8 Chapter 15 establishes the framework for evaluating TMS platforms: functional coverage, integration architecture, data integrity, and the role of automation in treasury efficiency. It identifies the TMS as the operational center of the treasury function — the system of record for positions, cash flows, and reporting.

Digital asset treasury operations impose requirements on TMS architecture that are categorically different from those of traditional treasury:

- Settlement occurs continuously, 24 hours a day, 7 days a week, without banking-hours cutoffs *(C001 D4 — 24/7 Clearing)*
- The authoritative source of position truth is an on-chain ledger, not a bank's data feed
- Irreversible on-chain transactions (C001 vocabulary bridge: wire transfers → on-chain transactions, protocol rail) require pre-broadcast controls that cannot be retrofitted post-execution *(C001 D1 — Transaction Irreversibility)*
- Private key operations require integration with hardware custody systems *(C001 D2 — Private Key Custody)*
- Protocol-layer risk events require sub-minute detection and escalation *(C001 D3 — Protocol-Layer Risk)*

This document defines seven mandatory TMS requirements, the 24/7 clearing compatibility standard, the minimum TMS dashboard API specification, and the loop interval requirement governing TMS operational cadence.

---

## 2. The Seven Mandatory TMS Requirements

A TMS is CTP-eligible for digital asset treasury operations only if it satisfies all seven requirements in this section. Partial compliance does not constitute CTP eligibility. Each requirement may be satisfied by the core TMS platform or by a certified integration with a dedicated subsystem — but the integrated system as a whole must satisfy all seven.

### 2.1 Requirement 1 — Real-Time Float Register with CECR per Channel

The TMS must maintain a real-time float register (C001 vocabulary bridge: bank statement → float register snapshot) that computes the Clearing Epoch Coverage Ratio (CECR) for each active channel (custodial wallet (C001 vocabulary bridge: bank account → custodial wallet) × protocol class pairing) at each loop cycle.

**Specification**:

- CECR must be computed using the formula defined in C008 Section 3.1: `(sum of Z=1 position-milliseconds) / (total observed position-milliseconds in period)`
- CECR must be computed independently per channel; portfolio composite CECR must be derived from channel-level values, not computed from aggregated inputs
- The float register must maintain a rolling 30-day history of CECR values per channel for trend analysis and HALT opportunity cost computation (C011 Section 5.1)
- Position entries in the float register must satisfy all four C013 audit trail requirements: transaction hash, block timestamp, native precision, and channel identifier

**Acceptance test**: System can produce a channel-level CECR report as of any block height in the trailing 30 days, with source entries traceable to individual on-chain transactions via transaction hash.

### 2.2 Requirement 2 — Clearing Epoch Derivation by Protocol Class

The TMS must implement the protocol class table from C008 Section 2.2 and derive the clearing epoch (clearingMs) for each instrument held in the portfolio based on its protocol class. Manual entry of clearingMs values is permitted only for `contract` and `cbdc` protocol classes, where the clearing epoch is defined in instrument-specific terms rather than derived from protocol parameters.

**Specification**:

- Protocol class must be assigned to each custodial wallet at setup, not at transaction time
- clearingMs derivation must use the published protocol parameters for the nominal clearing epoch; for protocols with variable block times (e.g., `pow`), use the published target block time
- The governor ceiling (C008 Section 2.3) must be computed from clearingMs and stored as a derived parameter: `governor_ceiling = clearingMs × 0.90`
- Z-state assignment (Z=1 or Z=0) must be derived from the elapsed time since broadcast versus clearingMs, updated at each loop cycle

**Acceptance test**: Given a transaction broadcast timestamp and protocol class, system computes the correct Z-state at any subsequent loop cycle, including the Z=0 transition at clearingMs elapsed.

### 2.3 Requirement 3 — HARD_HALT Detection and Escalation

The TMS must detect HARD_HALT conditions and escalate within the timeframes defined in C008 Section 5.2 *(C001 D3 — Protocol-Layer Risk)*.

**HARD_HALT trigger conditions** (either is sufficient):
- `antinodeIndex ≥ 13` for any protocol class in the active portfolio
- Any channel assigned `tier = HALT` by the C012 risk taxonomy

**Specification**:

- antinodeIndex must be computed and updated at each loop cycle for every active protocol class
- antinodeIndex increment logic: each stall event or confirmation failure on a protocol increments the index; successful confirmation at nominal clearing epoch decrements the index by 1 (index floor = 0)
- Upon HARD_HALT trigger: immediately set affected channel Z-state to Z=0 for all positions, suspend deployment gate for affected instruments, generate a Hermes event queue entry with severity = HALT
- HARD_HALT state must persist until explicitly cleared per C008 Section 5.3 (clearing conditions including confirmed thorn anchor and antinodeIndex < 13)
- No automated HARD_HALT resolution is permitted; resolution requires human sign-off logged in the BLID chain (C001 vocabulary bridge: reconciliation → BLID chain / thorn anchor)

**Acceptance test**: Simulate antinodeIndex = 13 on a test protocol class; verify that within one loop cycle, all positions on the affected protocol are Z=0, deployment gate is suspended, and a Hermes event is generated.

### 2.4 Requirement 4 — Audit Trail with Block Timestamp Primary Key

The TMS audit trail must satisfy all four C013 minimum requirements (C013 Section 3), with block timestamp as the authoritative event timestamp *(C001 D1 — Transaction Irreversibility)*.

**Specification**:

- The TMS must retrieve block timestamps from the on-chain data source (node API, block explorer API, or custodial wallet API) at the time of confirmation detection, not from a local system clock
- Block timestamps must be stored in the audit log as a separate field from the system detection timestamp; both must be present for every confirmed on-chain event
- The audit log must be append-only (no modification of existing entries); corrections are made by appending a new entry referencing the original by transaction hash
- Transaction hashes must be stored at full native length; the audit log schema must not truncate any field of the transaction hash
- Native asset amounts must be stored with ≥15 decimal places of precision (no floating-point representation)

**Acceptance test**: Query the audit log for a confirmed on-chain transaction; verify that the block timestamp retrieved from the on-chain data source matches the block timestamp in the audit log for that transaction hash.

### 2.5 Requirement 5 — BLID Chain Issuance on Each Healthy Anchor Cycle

The TMS must issue a BLID transaction ID (C001 vocabulary bridge: SoD audit record → BLID transaction ID) at each anchor cycle where all of the following conditions are met: (a) the float register snapshot balance matches the on-chain confirmed balance, (b) no stale_canonical_float exists on the anchored channel, and (c) the CECR on the anchored channel is ≥ 0.60.

**Specification**:

- BLID chain entries must be hash-chained: each entry contains the cryptographic hash of the previous entry, creating a tamper-evident chain
- BLID chain entries must be generated automatically by the TMS at each anchor cycle; manual BLID issuance is permitted only for correction entries
- Anchor cycle frequency: at minimum, one anchor per 90-second loop cycle for channels with active positions; cold tier channels may anchor at a 24-hour cadence given infrequent on-chain activity
- The BLID chain must be exported to off-chain storage (separate from the primary TMS database) at each anchor cycle, or at minimum daily
- A thorn anchor entry must be included in the BLID chain at each anchor cycle, recording the channel balance, CECR, and Z-state snapshot

**Acceptance test**: Demonstrate hash chain integrity by modifying a historical BLID entry and verifying that the system detects the chain break at the next anchor cycle.

### 2.6 Requirement 6 — C011 Deployment Gate with Baumol Floor Computation

The TMS must implement the C011 deployment gate as an automated pre-transaction check, enforcing both the balance condition and the forecast condition before any yield instrument purchase or staking transaction is authorized *(C001 D2 — Private Key Custody for cold-tier deployments; C001 D5 — Smart Contract Counterparty for contract-tier instruments)*.

**Specification**:

- Baumol floor must be stored as a named parameter in the TMS, updatable only through the treasury policy change management process (not editable by operations personnel without change management approval)
- Baumol floor computation must reflect: 48-hour operational on-chain transaction volume at the current rolling average, plus a 20% buffer
- Net-30 forecast input must be sourced from the cash flow forecasting module or an approved external feed; manual override of the net-30 forecast requires a dual-control approval logged in the BLID chain
- When the deployment gate fails, the TMS must: block the transaction, log the gate failure event with both failing conditions and current values, and generate a Hermes event queue entry
- The gate must re-evaluate at each loop cycle; a previously failed gate may open automatically when conditions improve, but the opening must be logged as a gate-open event

**Acceptance test**: Set net-30 forecast to a negative value; verify that all yield deployment transaction attempts are blocked, gate failure is logged, and a Hermes event is generated.

### 2.7 Requirement 7 — Hermes Event Queue with C012 Risk Flag Classification

The TMS must maintain a Hermes event queue that receives, classifies, and escalates events from all C012 risk categories (R1–R7) in real time *(C001 D4 — 24/7 Clearing)*.

**Specification**:

- Hermes event queue must operate continuously; no scheduled downtime is permitted. Any unplanned queue downtime exceeding one loop interval must be treated as an R2 active shield event.
- Events must be classified by risk category (R1–R7) and severity (INFO, WARNING, ACTIVE_SHIELD, HALT) at the time of generation
- HALT-severity events must trigger immediate push notification to designated escalation contacts; the notification must occur within one loop cycle of the triggering condition
- The Hermes queue must maintain a persistent log of all events with timestamps, risk category, severity, and resolution status
- Events must remain in an "open" state until a human operator explicitly closes them with a resolution note logged in the BLID chain; automated event closure is not permitted for ACTIVE_SHIELD or HALT severity events
- Heartbeat monitoring: if no events are generated for a period exceeding 3× the loop interval, the TMS must generate a synthetic heartbeat event to confirm queue liveness. Absence of a heartbeat event triggers R2 monitoring (C012 Section 3.4)

**Acceptance test**: Simulate an antinodeIndex spike to 13 on a test protocol; verify that a HALT-severity Hermes event is generated within one loop interval, appears in the queue, and that the queue liveness heartbeat continues independently.

---

## 3. 24/7 Clearing Compatibility Standard

### 3.1 Definition

**24/7 clearing compatibility** means that no automated TMS process has a dependency on banking-hours availability, business-day calendars, or clearing house (consensus protocol) operating schedules *(C001 D4 — 24/7 Clearing)*.

This is a systemic requirement: the TMS must be capable of detecting, classifying, and responding to on-chain events at any time, including weekends, holidays, and outside of UTC business hours.

### 3.2 Prohibited Banking-Hours Dependencies

The following are prohibited in any automated TMS process:

- Settlement windows that reference "next business day" or "cut-off time" for digital asset clearing epoch computation (clearing epochs are protocol-defined, not business-day-defined)
- CECR computation pauses during off-hours (CECR must accumulate continuously; Z=0 time during weekends and holidays counts against CECR in the same way as weekday time)
- Hermes event queue suspension or reduced monitoring cadence during off-hours
- Baumol floor evaluation suspended overnight or on weekends
- BLID chain anchor cycles paused during off-hours (anchoring must occur at the specified cadence regardless of time of day or day of week)

### 3.3 Maintenance Window Handling

Scheduled TMS maintenance windows are permitted but must satisfy:

1. Maintenance windows must be announced at least 24 hours in advance in the treasury policy
2. During a maintenance window, no new on-chain transactions may be broadcast
3. All active Z=1 positions during a maintenance window continue to accumulate Z=1 or Z=0 time based on protocol clock, not TMS availability; the CECR computation upon maintenance window close must retroactively account for all Z-state transitions during the downtime using block timestamps from the on-chain data source
4. Maintenance windows may not extend beyond one clearing epoch for any CTP-eligible rail in the active portfolio
5. Maintenance window start and end must be logged in the BLID chain

---

## 4. Dashboard API — Minimum TMS Interface

### 4.1 Endpoint Specification

The minimum TMS interface for digital asset treasury is the float dashboard API, accessible at `/api/float`. This endpoint is the programmatic interface to the float register and CECR state, and is the integration point for external reporting, audit, and management dashboard tools.

**Base endpoint**: `GET /api/float`

**Minimum response schema** (JSON):

```json
{
  "timestamp_utc": "<ISO 8601 UTC timestamp of report generation>",
  "block_reference": {
    "protocol": "<protocol class>",
    "block_height": "<integer>",
    "block_timestamp_utc": "<ISO 8601 UTC>"
  },
  "portfolio_tier": "<GREEN | YELLOW | ORANGE | RED | HALT>",
  "portfolio_cecr": "<decimal, 4 decimal places>",
  "channels": [
    {
      "channel_id": "<custodial wallet ID>:<protocol class>",
      "custody_tier": "<hot | warm | cold>",
      "protocol_class": "<protocol class code>",
      "clearing_ms": "<integer>",
      "governor_ceiling_ms": "<integer>",
      "cecr": "<decimal, 4 decimal places>",
      "z_state": "<1 | 0>",
      "balance_native": "<string, native precision>",
      "balance_usd_floor": "<decimal, 2 decimal places>",
      "antinode_index": "<integer>",
      "last_thorn_anchor_blid": "<BLID transaction ID>",
      "last_anchor_timestamp_utc": "<ISO 8601 UTC>",
      "stale_canonical_float_native": "<string, native precision>",
      "evm_gas_exposure": "<present | none>"
    }
  ],
  "halt_events_rolling_30d": "<integer>",
  "deployment_gate_status": "<open | closed>",
  "deployment_gate_reason": "<string | null>",
  "baumol_floor_native": "<string, native precision>",
  "net30_forecast_usd": "<decimal, 2 decimal places>",
  "sub_float": {
    "instrument_gap_float_usd": "<decimal, 2 decimal places>",
    "pool_stale_float_usd": "<decimal, 2 decimal places>",
    "canonical_gap_float_usd": "<decimal, 2 decimal places>"
  }
}
```

### 4.2 Additional Required Endpoints

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/float/channels/:channel_id` | GET | Per-channel detail: full position list, Z-state history, CECR time series |
| `/api/float/blid/:blid_id` | GET | Retrieve specific BLID chain entry and its chain context |
| `/api/float/halt` | GET | Current HALT status, affected channels, trigger conditions, resolution status |
| `/api/float/gate` | GET | Current deployment gate status with condition values |
| `/api/float/register` | GET | Float register snapshot at a specified block height (query param: `block_height`) |
| `/api/float/hermes` | GET | Hermes event queue: open events, filtering by risk category and severity |
| `/api/float/cecr/history` | GET | Rolling CECR history per channel (query params: `channel_id`, `days`) |

### 4.3 API Authentication and Access Control

The `/api/float` interface must enforce:

- Authentication via API key or OAuth 2.0 token for all endpoints
- Read-only access for reporting and audit integrations; write operations (BLID corrections, gate overrides) require an elevated authorization scope subject to SoD review (C010 Section 3)
- All API access must be logged in the BLID chain audit trail with the requesting identity, endpoint, and timestamp
- No unauthenticated access to any float register or BLID chain data, regardless of network location

---

## 5. Loop Interval Requirement

### 5.1 Standard Loop Interval

The TMS loop interval — the cadence at which the system executes all mandatory monitoring, CECR computation, Z-state evaluation, deployment gate check, and Hermes event generation — is **≤90 seconds**.

This limit is derived from the operational requirement to detect Z-state transitions on the shortest CTP-eligible clearing epochs within a single clearing epoch window. XRPL ledger closes at 3,000–5,000 ms and Stellar at 5,000 ms; a 90-second full-cycle interval, combined with the protocol-specific override poll intervals in Section 5.2, ensures that the TMS can detect and escalate HARD_HALT conditions (C008 Section 5.1) before a second clearing epoch elapses on any CTP-eligible rail.

A TMS operating on a loop interval longer than 90 seconds cannot reliably detect Z-state transitions for short-clearing-epoch protocols (notably Stellar at ~5,000 ms and XRPL at ~3,000–5,000 ms clearing epochs) within a single clearing epoch, which would produce systematically overstated CECR values.

### 5.2 Protocol-Specific Override Intervals

For protocols with clearing epochs shorter than the standard loop interval, the TMS must implement protocol-specific override intervals for Z-state polling:

| Protocol Class | Nominal clearingMs | Required Z-State Poll Interval |
|---|---|---|
| `stablecoin` (XRPL) | 3,000–5,000 ms | ≤3,000 ms |
| `stablecoin` (Stellar) | 5,000 ms | ≤5,000 ms |
| `poh` (Solana) | 12,800 ms | ≤10,000 ms |
| `payment-channel` (Lightning) | Seconds | Event-driven (HTLC state change) |
| All others | > 90,000 ms | ≤90,000 ms (standard loop) |

Protocol-specific Z-state polling may use a lightweight endpoint (e.g., ledger close subscription via WebSocket rather than a full loop cycle computation) provided that:

- The Z-state result is immediately written to the float register
- A full loop cycle (including CECR recomputation) is triggered at the standard ≤90-second cadence regardless of protocol-specific poll results
- The protocol-specific poll is logged in the audit trail with its own timestamp (system timestamp acceptable for poll latency; block timestamp applies to the confirmed state observed)

### 5.3 Loop Latency Monitoring

The TMS must monitor its own loop execution time and generate a Hermes R2 event if any loop cycle exceeds the ≤90-second interval by more than 50% (i.e., exceeds 135 seconds total elapsed time). Persistent loop latency is an R2 operational risk indicator and may indicate system resource degradation that could impair HARD_HALT detection reliability.

---

## 6. TMS Selection and Certification Guidance

### 6.1 Build vs. Buy Considerations

The seven mandatory requirements (Section 2) may be satisfied by:

1. A purpose-built digital asset treasury platform that natively implements all seven requirements
2. A traditional TMS with certified integrations to: (a) a digital asset custodial wallet API provider, (b) a BLID chain implementation, and (c) a protocol monitoring service that provides antinodeIndex-equivalent data

In either case, the combined system must be validated against all seven acceptance tests before deployment in a production CTP-eligible treasury environment.

### 6.2 Vendor Due Diligence Requirements

For any TMS vendor or integration partner:

1. SOC 2 Type II audit with coverage of data integrity, availability, and confidentiality is the minimum acceptable assurance
2. The vendor must disclose the block timestamp source used in audit trail generation; acceptance requires that block timestamps are sourced from on-chain data, not from a centralized data aggregator that may introduce delays or errors
3. The vendor must confirm that the BLID chain implementation uses cryptographic hash chaining (not a sequential numbering scheme) and can demonstrate chain integrity verification
4. HARD_HALT detection logic must be documented and testable; vendor must provide test harness for antinodeIndex simulation

### 6.3 Legacy TMS — Migration Path

Existing treasury operations using a traditional TMS (not designed for digital assets) that wish to achieve CTP eligibility must:

1. Document current state versus the seven requirements: identify gaps per requirement
2. Prioritize Requirements 3 (HARD_HALT), 4 (audit trail), and 7 (Hermes queue) as the first phase of implementation, as these address the highest operational risk
3. Implement Requirements 1, 2, 5, and 6 in a second phase
4. Complete all seven requirements and run parallel operations (legacy + new system) for a minimum of one 30-day capital stack cycle before retiring legacy processes
5. Obtain a BLID chain anchor that specifically attests to the migration completion, referencing the last legacy audit trail entry

---

## 7. Integration with Other C-Series Addenda

- **C008** (Float Management): TMS is the execution platform for all C008 CECR computations, Z-state management, HARD_HALT logic, and portfolio tier classification
- **C009** (Payment Rails): Pre-broadcast authorization gate is a TMS function; EVM gas exposure flag assignment and payment rail selection order enforcement are TMS responsibilities
- **C010** (Custody Architecture): BLID chain issuance (Requirement 5) and thorn anchor generation are the primary custody SoD audit mechanisms implemented in TMS; hot/warm/cold tier assignment is a TMS configuration parameter
- **C011** (Yield Policy): Deployment gate (Requirement 6) and Baumol floor parameter are implemented in TMS; C011 capital stack bucket assignments are TMS-maintained configuration
- **C012** (Risk Taxonomy): Hermes event queue (Requirement 7) is the R1–R7 risk event delivery mechanism; all float shield categories are computed and flagged by TMS
- **C013** (Accounting): All four C013 audit trail requirements are implemented as TMS data capture requirements; the float register snapshot and block timestamp audit trail are TMS-generated artifacts

---

## Source Reference

> *Essentials of Treasury Management, 8th Edition (ETM8).*
> Association for Financial Professionals (AFP). Rockville, MD, 2025.
> © 2025 Association for Financial Professionals. All rights reserved.
> All proprietary rights to the examination, including copyright, are held by AFP.
