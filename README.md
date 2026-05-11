# Proposed AFP-CA-ETM8 — Digital Asset Treasury Operations

> **PROPOSED APPROACH** — Not an official AFP publication. Not reviewed or endorsed by
> the Association for Financial Professionals. This series proposes how existing AFP/CTP
> principles from *Essentials of Treasury Management, 8th Edition* apply to digital asset
> treasury operations, and is offered for professional discussion only.
> **Author:** Garzaro &nbsp;|&nbsp; **Status:** Proposed — subject to revision

---

## Series Identification

**Repository:** proposed-afp-ca-etm8
**Series prefix:** AFP-CA-ETM8
**Base standard:** *Essentials of Treasury Management*, 8th Edition (ETM8). AFP, 2025.
**Examination domains:** Cash and Liquidity Management; Short-Term Investing and Borrowing;
Financial Risk Management; Technology in Treasury; Financial Accounting and Reporting
**Prerequisite:** C001 is the required entry point for all subsequent addenda.

---

## Author Statement

This series was developed by Edgar Garzaro, a former commercial banking professional whose
direct experience in treasury and cash management operations forms the practical foundation
for every analytical conclusion presented here.

At Harris Bank / AMCORE Bank (AVP, Sales), the author managed deposit relationships
exceeding $16MM, worked in direct partnership with commercial lenders to identify and
execute treasury cross-sell opportunities, and operated within the same cash management
disciplines — float schedules, account analysis, investable balance optimization — that
ETM8 codifies. At Banco Popular North America, direct exposure to Account Reconciliation
and Positive Pay implementations provided hands-on experience with the payment control and
audit trail functions that underpin C009 (Payment Rail Assessment) and C013 (Accounting
and Audit Trail) in this series. These are not academic references. They are the operational
context from which the conclusions in this series were drawn.

The CTP credential and the AFP body of knowledge represent the formalization of standards
that serious treasury practitioners have always held — not because regulation requires them,
but because the governance of money demands them. The regulation followed the practice.
The practice is what this series is grounded in.

The motivation for this work is straightforward: the digital asset space is operating
treasury functions without the framework that governs treasury functions. That gap is not
theoretical. It produces real harm — to market participants, to counterparties, and to the
broader financial system. The practices that fill the gap where professional standards are
absent are, in many documented cases, the same practices that monetary policy and securities
regulation were designed to prevent. The framework that prevents them already exists. It
is the AFP/CTP body of knowledge. The only work required is the mapping.

This series adopts AFP/CTP methodology exclusively. Where ETM8 has a method, that method
is applied. Where ETM8 has a gap — because the instrument class did not exist when the
chapter was written — the gap is filled by the smallest possible extension of the existing
method, using the same analytical logic, the same decision criteria, and the same
professional standards that govern every other ETM8 application. No alternative frameworks
are introduced. No proprietary methods are employed. The discipline is the discipline.

The ethical commitment behind this work is the same commitment that produced the CTP
examination: money governance is a professional responsibility, and professional
responsibility requires a standard. This series is an argument that the standard already
exists, and that applying it to digital asset treasury operations is not a choice but an
obligation for any practitioner who takes that responsibility seriously.

---

## 1. Background and Rationale

ETM8 governs treasury management across the full lifecycle of the cash conversion cycle:
collections, disbursements, concentration, short-term investing, risk management, technology
integration, and financial reporting. The framework is grounded in U.S. and international
regulatory requirements — Federal Reserve Regulation D (reserve requirements), NACHA
Operating Rules (ACH origination and settlement), Federal Reserve operating circulars
(Fedwire Funds Service), FASB accounting standards, and the Bank Secrecy Act (BSA) /
FinCEN compliance framework — and is designed to produce treasury operations that are
auditable, compliant, and operationally sound.

Digital asset treasury operations are treasury operations. The instrument class is new.
The regulatory and operational framework that governs money movement is not. The Bank
Secrecy Act applies to digital asset transactions. FinCEN guidance (FIN-2013-G001 and
subsequent) classifies convertible virtual currency exchangers and administrators as money
services businesses (MSBs) subject to AML/CFT program requirements. FASB ASC 350-60,
effective for fiscal years beginning after December 15, 2024, requires fair value
measurement of digital assets under U.S. GAAP. The SEC, CFTC, and applicable state money
transmission licensing regimes assert jurisdiction over digital asset activities that
constitute regulated financial activity under existing definitions.

The regulatory framework already covers this instrument class. What has been absent is a
systematic mapping of that coverage onto the operational and analytical framework that CTP
practitioners apply in daily treasury practice. Practitioners entering the digital asset
space without this mapping are operating treasury functions subject to the same regulatory
requirements as their traditional portfolios, without the analytical structures ETM8
provides for meeting those requirements.

The consequence of that gap is not merely operational. Treasury practices that constitute
BSA violations, unlicensed money transmission, custody control failures, or unsegregated
client fund handling in a traditional treasury context do not become permissible because
the instrument is digital. The ETM8 framework, applied correctly to digital asset
operations, produces a treasury function that satisfies existing regulatory requirements.
The absence of that framework produces a treasury function that, in many operating patterns
common in the digital asset space, does not.

This series closes the analytical gap. It does not introduce new regulatory requirements.
It applies the framework that governs money — and that CTP practitioners are credentialed
to administer — to the instrument class that requires it.

---

## 2. Methodology

The conclusions in this series were reached through direct operational engagement with
digital asset treasury functions: active management of settlement streams across multiple
payment rails, application of float management disciplines to clearing epoch measurement,
payment rail cost analysis using the AFP disbursement cost framework, investable balance
deployment under Investment Policy Statement constraints, and audit trail construction
using content-addressed transaction identifiers in place of wire trace numbers.

Each ETM8 chapter was evaluated against real operational scenarios in digital asset
treasury, not against theoretical descriptions of how digital assets work. The analytical
process followed the ETM8 method in each domain:

**Float management (C008):** The AFP float taxonomy (friction float, strategic float,
net float position) was applied to digital asset settlement streams. The key finding was
that the taxonomy transfers without modification; the measurement unit changes from banking
days to protocol-defined clearing epochs, and the measurement instrument changes from bank
statement availability schedules to on-chain confirmation data. The float management
discipline is identical.

**Payment rail assessment (C009):** AFP payment rail evaluation criteria (availability,
cost, certainty, finality, operational risk) were applied to digital asset payment rails.
The key finding was that auction-based fee mechanisms (EVM gas) fail the cost certainty
criterion that ETM8 requires for treasury-eligible payment rails, and that protocol-defined
deterministic fee schedules satisfy it. The evaluation framework is identical; the
instrument characteristics determine the outcome.

**Custody controls (C010):** ETM8 Chapter 18 SoD requirements were mapped to digital
asset custody architecture. The key finding was that private key custody is a treasury
control function, not an IT function — it carries the same authorization matrix, dual-
control requirements, and audit trail obligations as wire release authority under traditional
treasury operations. The control framework is identical; the control mechanism is different.

**Yield policy (C011):** The Baumol-Tobin optimal balance model and IPS deployment gate
were applied to digital asset investable balance deployment. The key finding was that the
model transfers directly; the instrument tier table requires extension to cover tokenized
instruments, and the liquidity premium embedded in each tier must account for clearing
epoch duration rather than banking-day availability. The analytical framework is identical.

**Risk taxonomy (C012):** ETM8 ERM categories were evaluated for coverage of digital asset
risk characteristics. The key finding was that six risk characteristics (D1–D6, defined in
C001) require explicit classification because existing ETM8 risk categories do not fully
capture them. Seven risk categories (R1–R7) were defined to provide complete coverage.
The ERM monitoring and escalation framework transfers without modification.

**Accounting and audit trail (C013):** ETM8 Chapter 8 audit trail requirements were mapped
to digital asset transaction records. The key finding was that block timestamp — not system
timestamp — is the authoritative event anchor for digital asset transactions, consistent
with ETM8's requirement that audit evidence be independent of the party being audited.
FASB ASC 350-60 provides the accounting treatment. The audit trail discipline is identical.

**TMS integration (C014):** ETM8 Chapter 15 TMS evaluation criteria were applied to
digital asset treasury system requirements. The key finding was that continuous clearing —
the absence of banking-hours cutoff windows — is the primary architectural difference,
and that TMS platforms serving digital asset treasury must support time-continuous position
updates, protocol-native transaction import, and clearing epoch-based float scheduling.
The evaluation framework is identical; the capability requirements differ.

In each domain, the analytical conclusion was the same: the ETM8 framework applies. The
instrument is different. The discipline is not.

---

## 3. Regulatory Grounding

The following regulatory frameworks are directly applicable to digital asset treasury
operations and are referenced throughout this series:

| Framework | Applicability |
|---|---|
| Federal Reserve Regulation D | Reserve requirement computation; investable balance determination |
| NACHA Operating Rules | ACH origination windows, prenote requirements, return item handling |
| Federal Reserve Fedwire Operating Circular | Settlement finality; Fedwire Funds Service operating hours and cutoff |
| FASB ASC 350-60 | Fair value measurement of digital assets (effective fiscal years beginning after December 15, 2024) |
| Bank Secrecy Act / FinCEN MSB rules | AML/CFT program requirements; CTR and SAR filing obligations; recordkeeping |
| FinCEN FIN-2013-G001 and subsequent guidance | Classification of convertible virtual currency activity as MSB |
| SEC Regulation S-P; Rule 17a-4 | Customer asset protection; books and records requirements |
| State money transmission licensing | Jurisdiction-specific licensing requirements for digital asset transmission |
| AICPA 2019 Practice Aid on Virtual Currency | Accounting and audit guidance for periods prior to FASB ASC 350-60 |
| Investment Company Act of 1940 | Applicable to pooled digital asset vehicles meeting the definition of investment company |

Treasury functions operating in the digital asset space are subject to this regulatory
environment regardless of whether those functions have been evaluated against it. This
series provides the analytical framework for conducting that evaluation using ETM8 methods.

---

## 4. Application to CTP Examination Domains

| CTP Domain | Addenda |
|---|---|
| Cash and Liquidity Management | C008 (Float Management), C009 (Payment Rail Assessment) |
| Short-Term Investing and Borrowing | C011 (Yield Policy and Opportunity Cost) |
| Financial Risk Management | C012 (Risk Taxonomy) |
| Treasury Policies and Procedures | C010 (Custody Controls) |
| Financial Accounting and Reporting | C013 (Accounting and Audit Trail) |
| Technology in Treasury | C014 (TMS Integration) |
| Cross-domain foundation | C001 (Practitioner's Bridge — required for all domains) |

---

## 5. Series Index

C001 is the required entry point. It defines the vocabulary bridge between ETM8
terminology and digital asset equivalents, and introduces the six delta concepts (D1–D6)
referenced throughout the series. No other addendum should be read without first reading C001.

| Document | ETM8 Primary Chapter | Scope |
|---|---|---|
| [C001 — Practitioner's Bridge](C001-practitioner-bridge.md) | Cross-series | Vocabulary bridge; delta concepts D1–D6; identical, analogous, and new. **Required first.** |
| [C008 — Float Management](C008-float-management.md) | Ch. 12 | Clearing epoch framework; float register; friction vs. strategic float; float shields |
| [C009 — Payment Rail Assessment](C009-payment-rails.md) | Ch. 4 | CTP eligibility criteria; protocol fee assessment; gas exclusion; finality measurement |
| [C010 — Custody Controls](C010-custody-controls.md) | Ch. 3, Ch. 18 | Key custody as treasury control; SoD; dual-control authorization; audit trail |
| [C011 — Yield Policy](C011-yield-policy.md) | Ch. 13 | Baumol floor; instrument tier table; IPS deployment gate; opportunity cost |
| [C012 — Risk Taxonomy](C012-risk-taxonomy.md) | Ch. 16–17 | R1–R7 risk categories; float shield mapping; monitoring and escalation standards |
| [C013 — Accounting and Audit Trail](C013-accounting.md) | Ch. 8 | FASB ASC 350-60; block timestamp as authoritative anchor; reconciliation framework |
| [C014 — TMS Integration](C014-tms-integration.md) | Ch. 15 | Minimum TMS capabilities; continuous clearing integration; data integrity requirements |
| [Treasury in Practice](TREASURY-IN-PRACTICE.md) | Companion vignette | Illustrative aside: a fictional CTP practitioner applies the framework. AFP Exchange editorial register. |

---

## 6. How to Use This Series

**CTP practitioners evaluating a digital asset treasury operation:**
Begin with C001 to establish the vocabulary mapping. Apply the relevant addenda in the
order corresponding to the treasury functions under review. Each addendum produces a
structured assessment against the ETM8 standard for its domain.

**Treasury teams building or acquiring digital asset treasury capability:**
The addenda define what correct treasury practice looks like for each functional area.
A treasury function implemented to the standards these addenda describe satisfies the
same ETM8-grounded requirements that govern traditional treasury operations, and can be
reviewed, audited, and governed by CTP-credentialed professionals.

**Audit and compliance functions:**
The regulatory table in Section 3 identifies the specific regulatory requirements
applicable to each treasury domain. The addenda identify where digital asset treasury
practices align with or deviate from the controls framework that satisfies those
requirements under traditional treasury operations.

---

## Source Reference

> *Essentials of Treasury Management, 8th Edition (ETM8).*
> Association for Financial Professionals (AFP). Rockville, MD, 2025.
> © 2025 Association for Financial Professionals. All rights reserved.

This series proposes addenda to ETM8. It does not reproduce, replace, or supersede the
source standard. CTP practitioners should consult ETM8 directly for the base framework
these documents extend. All proprietary rights to the CTP examination and ETM8, including
copyright, are held by AFP.
