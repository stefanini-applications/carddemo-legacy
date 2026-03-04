# System CardDemo - Overview for User Stories
**Version:** 2026-03-04  
**Purpose:** Single source of truth for writing user stories that span CardDemo core flows, batch operations, and its optional extensions.

---

## 📊 Platform Statistics
- **Technology Stack:** COBOL (IBM z/OS), CICS BMS maps, JCL batch orchestration, VSAM KSDS, optional DB2 12/13, IMS DB HIDAM, IBM MQ, RACF security and assembler utilities.
- **Architecture Pattern:** Transaction-driven CICS front end with batch back-end, optional DB2/IMS/MQ integrations for data persistence and messaging, and shared copybooks for data contracts.
- **Key Capabilities:** Account/card lifecycle management, transaction posting, bill payments, admin metadata management, MQ-based integrations (authorizations, account extractions), batch job orchestration for daily processing, optional fraud analytics and DB2-backed metadata.
- **Supported Languages/Notations:** COBOL, JCL, BMS macros, Assembler in `app/maclib`, embedded SQL, IMS DBD/PSB definitions, MQ message descriptors (EBCDIC CSV).

---

## 🏗️ High-Level Architecture
### Technology Stack
- **Backend:** COBOL programs running inside CICS for online work, batch COBOL launched through JCL and CA7/Control-M schedules.
- **Data Stores:** VSAM KSDS files for accounts, cards, transactions, statements; optional DB2 tables (e.g., `AUTHFRDS`, `TRANSACTION_TYPE*`) for relational metadata; IMS HIDAM segments for pending authorizations.
- **Messaging:** IBM MQ queues for external authorization/parameter exchanges and account inquiries.
- **Security & Infrastructure:** RACF for user authentication, CICS resource definitions via CSD, BMS screen layouts targeting 3270 terminals.
- **Others:** Shared copybooks in `app/cpy`, map definitions in `app/bms`, scheduler artifacts in `app/scheduler`.

### Architectural Patterns
- **Transaction Layer:** Every CICS transaction (CC00/CM00/CT02/etc.) routes through a COBOL program, references BMS mapsets, and follows message-area + `EXEC CICS` logic for validation/feedback.
- **Batch Jobs:** JCL jobs (e.g., `TRANEXTR`, `POSTTRAN`, `CREASTMT`, `COMBTRAN`) operate on VSAM datasets and can be chained via CA7/Control-M definitions.
- **Optional Integrations:** Authorization extension adds MQ-triggered transactions and two-phase commits spanning VSAM/IMS/DB2; transaction type module pairs DB2 CRUD transactions with VSAM extraction jobs; MQ account extraction extension is pure MQ request/response.
- **Authentication:** Sample users (admin `ADMIN001`, cardholder `USER0001`) authenticate through `CC00` and are routed to menu transactions such as `CA00`, `CM00`, or `CPVS`.

### 📋 Actors and Journeys
- **Regular User:** Logs in, uses `CM00` main menu, updates account/card data (`CAUP`, `CCUP`), posts transactions (`CT02`), sees pending authorizations (`CPVS/CPVD` when enabled), and initiates bill payments (`CB00`). Journeys emphasize sub-second online responses and immediate VSAM updates.
- **Admin User:** Accesses `CA00` menu, manages user records (`CU00-CU03`), updates transaction metadata via `CTTU`/`CTLI`, reviews MQ authorizations and fraud data, and monitors nightly batch jobs such as `POSTTRAN`, `CBPAUP0J`, and `TRANEXTR`.

---

## 📚 Module Catalog

<!-- MODULE_LIST_START -->
**Modules:** core-platform, authorization, transaction-type-management, mq-account-extractions
<!-- MODULE_LIST_END -->

### 1. Core Platform
**ID:** `core-platform`  
**Purpose:** Deliver the mandatory CardDemo runtime: 3270 sign-on, role-aware menu routing, account/card lifecycle operations, transaction capture, bill payment, and report triggering; plus nightly/monthly batch cycles that reconcile VSAM transaction state and generate statements. This module is the dependency base for all optional extensions.  
**Business Context:** Core-platform is the operational backbone for both cardholder flows (self-service updates, posting, bill pay) and back-office controls (security maintenance, reporting, batch closure/reopen windows). If this module is degraded, optional DB2/IMS/MQ features cannot provide end-to-end business value.  
**Key Components:**  
- **Online COBOL programs:** `COSGN00C` (auth + role routing), `COMEN01C` (user menu), `COADM01C` (admin menu), `COACTVWC`/`COACTUPC` (account view/update), `COCRDUPC` (card update), `COTRN00C`/`COTRN01C`/`COTRN02C` (transaction list/view/add), `COBIL00C` (bill pay), `CORPT00C` (report submission).  
- **BMS mapsets:** `COSGN00`, `COMEN01`, `COACTVW`, `COACTUP`, `COCRDUP`, `COTRN00`, `COTRN01`, `COTRN02`, `COBIL00`, `CORPT00`.  
- **CICS resource definitions:** `app/csd/CARDDEMO.CSD` defines files (`ACCTDAT`, `CARDDAT`, `CCXREF`, `CXACAIX`, `TRANSACT`, `USRSEC`), mapsets, programs, and transactions (`CC00`, `CM00`, `CA00`, `CAUP`, `CAVW`, `CCUP`, `CT00`, `CT01`, `CT02`, `CB00`, `CR00`, `CU00-CU03`).  
- **Batch/JCL orchestration:** `POSTTRAN`, `INTCALC`, `TRANBKP`, `COMBTRAN`, `TRANIDX`, `CREASTMT`, `OPENFIL`, `CLOSEFIL`; supported by CA7 (`app/scheduler/CardDemo.ca7`) and Control-M (`app/scheduler/CardDemo.controlm`) chains.  
- **Shared contracts:** copybooks `COCOM01Y` (COMMAREA), `CVACT01Y`/`CVACT02Y`/`CVACT03Y` (account/card/xref), `CVTRA05Y` (transaction), `CSUSR01Y` (security user).  
**Public APIs:**  
- **CICS transactions:** `CC00`, `CM00`, `CA00`, `CAVW`, `CAUP`, `CCLI`, `CCDL`, `CCUP`, `CT00`, `CT01`, `CT02`, `CB00`, `CR00`, `CU00`, `CU01`, `CU02`, `CU03`.  
- **Online interfaces:** BMS map input/output contracts with PF-key controls (`ENTER`, `PF3`, `PF4`, `PF5`, `PF7`, `PF8`, `PF12` depending on screen).  
- **Batch interfaces:** `POSTTRAN`, `CREASTMT`, `INTCALC`, `TRANBKP`, `COMBTRAN`, `TRANIDX`, `OPENFIL`, `CLOSEFIL` (plus optional weekly jobs coordinated by scheduler definitions).  
- **Operational scripts:** `scripts/run_full_batch.sh`, `scripts/run_posting.sh`, `scripts/run_interest_calc.sh` for JES submission order examples.  
**Dependencies:**  
- **Internal:** Feeds optional `authorization`, `transaction-type-management`, and `mq-account-extractions` modules through shared COMMAREA, VSAM entities, and menu integration points (`COMEN02Y`, `COADM02Y`).  
- **External runtime:** CICS TS, VSAM KSDS/AIX, JES/JCL, IDCAMS/SORT, RACF model, CA7/Control-M; assembler utilities `MVSWAIT` and `COBDATFT` support timing/date conversion patterns in batch ecosystems.  
**Data Models and Structures:**  
- **Security (`CSUSR01Y`):** user ID, password, user type (`A`/`U`) for CC00 gatekeeping.  
- **Account (`CVACT01Y`):** account ID, active flag, current/cycle balances, credit limits, lifecycle dates, group ID.  
- **Card (`CVACT02Y`):** card number, account ID, CVV, embossed name, expiry date, active status.  
- **Card XREF (`CVACT03Y`):** card number ↔ customer ID ↔ account ID bridge used by account/card/transaction workflows.  
- **Transaction (`CVTRA05Y`):** transaction ID, type/category/source, description, amount, merchant fields, card number, orig/proc timestamps.  
- **Application context (`COCOM01Y`):** FROM/TO program+transaction, signed-on user, selected account/card/customer IDs, map context.  
**Module-Specific Business Rules:**  
- Sign-on requires non-blank credentials and exact password match in `USRSEC`; admin users route to `CA00`, standard users route to `CM00`.  
- Menu options are validated against copybook-defined counts (`COMEN02Y`, `COADM02Y`); non-numeric or out-of-range options return map errors.  
- Account/card update transactions require valid account/card keys and cross-reference integrity through `CCXREF`/`CXACAIX`.  
- `CT02` enforces required fields, numeric code constraints, signed amount format (`-99999999.99`), and calendar-valid `YYYY-MM-DD` dates (`CSUTLDTC`).  
- `CB00` refuses payment when current balance is zero/negative; on confirm=`Y`, it writes a bill-payment transaction and rewrites account balance.  
- `CT00` pagination uses VSAM browse semantics (`STARTBR`, `READNEXT`, `READPREV`) with explicit top/bottom boundary messaging.  
- Batch windows require `CLOSEFIL` before mutating transaction files and `OPENFIL` after reload/index steps to restore online access.  
**User Story Examples:**  
- As a cardholder, I want to update my mailing address via `CAUP` so that statements route to my new location.  
- As a payments analyst, I want nightly `POSTTRAN` and `INTCALC` to finish before 4:00 AM so downstream reporting jobs can start.  
- As an auditor, I want `CT01` to display transactions sorted by posting date so I can verify customer activity before statement generation.

### 2. Authorization
**ID:** `authorization`  
**Purpose:** Add an MQ-driven credit card authorization service that stores pending requests in IMS and tracks fraud in DB2.  
**Key Components:** COBOL programs (`COPAUA0C`, `COPAUS0C`, `COPAUS1C`, `COPAUS2C`, `CBPAUP0C`), MQ queues (`AWS.M2.CARDDEMO.PAUTH.REQUEST`, `AWS.M2.CARDDEMO.PAUTH.REPLY`), IMS DBDs/PSBs (`DBPAUTP0`, `DBPAUTX0`, `PSBPAUTB`, `PSBPAUTL`), DB2 table/index (`AUTHFRDS`, `XAUTHFRD`), BMS screens (`COPAU00`, `COPAU01`).  
**Public APIs:** CICS transactions `CP00` (MQ trigger processing), `CPVS` (summary), `CPVD` (details and fraud marking). MQ interface for request/response, DB2 writes to `AUTHFRDS`, batch job `CBPAUP0J` purges expired IMS records.  
**User Story Examples:**  
- As a partner system, I want to submit a CSV-style authorization request via `AWS.M2.CARDDEMO.PAUTH.REQUEST` and receive a decision in under two seconds.  
- As a fraud analyst, I want `CPVD` to flag suspicious responses, persist `AUTH_FRAUD='Y'` in `AUTHFRDS`, and surface the reason code to the terminal.  
- As a platform engineer, I want `CBPAUP0J` to purge authorizations older than 30 days so the IMS HIDAM remains performant.

### 3. Transaction Type Management
**ID:** `transaction-type-management`  
**Purpose:** Provide DB2-backed CRUD for transaction types with VSAM-sync batch jobs for the core engine.  
**Key Components:** CICS transactions `CTTU` (add/edit) and `CTLI` (list/update/delete), DB2 precompiler artifacts (`dcl`, `ddl` directories), DB2 tables `CARDDEMO.TRANSACTION_TYPE`, `CARDDEMO.TRANSACTION_TYPE_CATEGORY`, batch jobs `TRANEXTR`, `MNTTRDB2`, copybooks for host variables/SQLCA.  
**Public APIs:** `CTTU`, `CTLI`, `TRANEXTR`, `MNTTRDB2`.  
**User Story Examples:**  
- As an admin, I want `CTTU` to create a new transaction type with description so cardholders can post it immediately.  
- As an integration engineer, I want `TRANEXTR` to export DB2 transaction metadata nightly so `CT00`/`CT02` can rely on VSAM copies.  
- As a QA lead, I want `CTLI` to prevent deletes when `TRANSACTION_TYPE_CATEGORY` references a type so data integrity stays intact.

### 4. MQ Account Extractions
**ID:** `mq-account-extractions`  
**Purpose:** Offer MQ-driven system-date and account-data inquiries demonstrating asynchronous VSAM extraction patterns.  
**Key Components:** Transactions `CDRD` (`CODATE01`) and `CDRA` (`COACCT01`), MQ definitions (`MQCONN`, `MQQUEUE` resources), request/response message formats, BMS/CICS resource definitions.  
**Public APIs:** `CDRD`, `CDRA`, MQ queues `CARDDEMO.REQUEST.QUEUE`, `CARDDEMO.RESPONSE.QUEUE`.  
**User Story Examples:**  
- As a scheduler, I want to push a date request and calibrate nightly jobs with the host date returned through `CARDDEMO.RESPONSE.QUEUE`.  
- As an aggregator, I want to retrieve account details via `CDRA` so a downstream portal can mirror VSAM account snapshots.

---

## 🔄 Architecture Diagram
```mermaid
flowchart TD
    User[User/Admin via 3270] -->|CC00/CA00| CICS[COBOL programs + BMS in CICS]
    CICS --> VSAM[VSAM KSDS: account/card/transaction datasets]
    CICS --> DB2[DB2: AUTHFRDS, TRANSACTION_TYPE* tables]
    CICS --> IMS[IMS HIDAM: PAUTSUM0/PAUTDTL1]
    CICS --> MQ[IBM MQ queues]
    MQ --> External[External systems / Scheduling jobs]
    CICS --> Batch[Batch jobs via JCL/CA7/Control-M]
    Batch --> VSAM
    Batch --> DB2
```

## 🔗 Module Dependency Diagram
```mermaid
flowchart LR
    core-platform --> batch-processing[Batch Operations (JCL/Control-M)]
    core-platform --> authorization
    core-platform --> transaction-type-management
    core-platform --> mq-account-extractions
    authorization --> IMS
    authorization --> DB2
    authorization --> MQ
    transaction-type-management --> DB2
    transaction-type-management --> VSAM
    mq-account-extractions --> MQ
    mq-account-extractions --> VSAM
```

---

## 🔌 Public Interfaces
- **CICS transactions:** `CC00`, `CM00`, `CAUP`/`CCUP`, `CT00`/`CT01`/`CT02`, `CB00`, `CR00`, `CP00`, `CPVS`, `CPVD`, `CTTU`, `CTLI`, `CDRD`, `CDRA`.  
- **Batch jobs:** `POSTTRAN`, `CREASTMT`, `TRANEXTR`, `TRANBKP`, `OPENFIL`, `CLOSEFIL`, `CBPAUP0J`, `INTCALC`, `TRANIDX`, `COMBTRAN`.  
- **MQ queues:** `AWS.M2.CARDDEMO.PAUTH.REQUEST`, `AWS.M2.CARDDEMO.PAUTH.REPLY`, `CARDDEMO.REQUEST.QUEUE`, `CARDDEMO.RESPONSE.QUEUE`.  
- **DB2 tables:** `CARDDEMO.TRANSACTION_TYPE`, `CARDDEMO.TRANSACTION_TYPE_CATEGORY`, `AUTHFRDS`.  
- **IMS database:** HIDAM `DBPAUTP0` with segments `PAUTSUM0`, `PAUTDTL1`, and index `PAUTINDX`.

**Sample MQ authorization request (CSV fields):**
```
AUTH-DATE,AUTH-TIME,CARD-NUM,AUTH-TYPE,CARD-EXPIRY-DATE,MESSAGE-TYPE,MESSAGE-SOURCE,PROCESSING-CODE,TRANSACTION-AMT,MERCHANT-CATAGORY-CODE,ACQR-COUNTRY-CODE,POS-ENTRY-MODE,MERCHANT-ID,MERCHANT-NAME,MERCHANT-CITY,MERCHANT-STATE,MERCHANT-ZIP,TRANSACTION-ID
```

**Sample MQ response:**
```
CARD-NUM,TRANSACTION-ID,AUTH-ID-CODE,AUTH-RESP-CODE,AUTH-RESP-REASON,APPROVED-AMT
```

---

## 📊 Data Models
### AUTHFRDS (DB2)
```sql
CREATE TABLE <schema>.AUTHFRDS (
  CARD_NUM CHAR(16) NOT NULL,
  AUTH_TS TIMESTAMP NOT NULL,
  AUTH_RESP_CODE CHAR(2),
  PROCESSING_CODE CHAR(6),
  TRANSACTION_AMT DECIMAL(12,2),
  MATCH_STATUS CHAR(1),
  AUTH_FRAUD CHAR(1),
  FRAUD_RPT_DATE DATE,
  PRIMARY KEY(CARD_NUM, AUTH_TS)
);
```
### IMS segments
- `PAUTSUM0`: root authorization summary segment.  
- `PAUTDTL1`: child authorization detail segment.  
- `PAUTINDX`: HIDAM index for quick lookups.
### MQ message formats (COBOL-style)
```cobol
01 AUTH-REQUEST.
   05 AUTH-DATE        PIC X(8).
   05 AUTH-TIME        PIC X(6).
   05 CARD-NUM         PIC X(16).
   05 TRANSACTION-AMT  PIC 9(12)V99.
   05 MERCHANT-ID      PIC X(15).
   05 TRANSACTION-ID   PIC X(15).
```

---

## 📋 Business Rules by Module
### Core Platform
- Transactions require active records in VSAM datasets (`ACCTDATA`, `CARDDATA`, `TRANSACT`).  
- Bill payments (`CB00`) validate available balance before posting and prompt PF keys for confirmation.  
- Cross-reference lookups (`CARDXREF`) ensure card/account pair consistency during updates.
### Authorization
- Every MQ request must persist to IMS and DB2 before replying; the MQ response echoes the same `TRANSACTION-ID`.  
- Fraud marking (`CPVD` PF5) records `AUTH_FRAUD='Y'` in `AUTHFRDS` and annotates the IMS detail segment.  
- `CBPAUP0J` removes IMS authorizations older than 30 days and updates available credit for the associated account/card.
### Transaction Type Management
- DB2 delete operations are blocked when `TRANSACTION_TYPE_CATEGORY` references a type (`DELETE RESTRICT`).  
- Batch `TRANEXTR` exports DB2 rows into VSAM-friendly files every run so online transactions keep their local reference data.  
- Static SQL uses host variables and SQLCA checks; every non-zero SQLCODE triggers rollback.
### MQ Account Extractions
- Each MQ request carries a correlation ID that must appear in the response queue payload.  
- Account detail requests verify VSAM account presence before populating the 300-byte `ACCOUNT-DATA` response.  
- Date requests simply return the host date and correlation ID regardless of payload content.

---

## 🌐 Internationalization and Translation
CardDemo does not rely on filesystem-based i18n catalogs. All screen text lives inside BMS mapsets (`COSGN00`, `COACTUP`, `COPAU00`, etc.) compiled into the CICS map library, and translations (if needed) are handled by deploying alternate mapsets or copybooks. There is no `locales/` or JSON/YAML locale folder.

---

## 📋 Form and Listing Patterns
### Form Pattern
- CICS BMS maps define fields and PF keys; every form ties to a COBOL copybook with PIC clauses.  
- Validation occurs in COBOL routines that set `MSGSET/MSGID`/`MESSAGE` and `RETURN` to redisplay the map.  
- Forms may run inside modals only within the 3270 terminal (PF keys), so there are no reusable base UI components beyond BMS macros.
### Listing Pattern
- Lists such as `CT00`, `CPVS`, and `CTLI` read VSAM sequentially with manual paging (`READ NEXT`, `READ PREV`).  
- Actions (edit, delete) are PF key-driven, and the message area displays results or warnings.  
- Notifications are embedded in the map (PF messages) instead of toast/snackbar widgets.

---

## 🎯 User Story Patterns
### Templates
- **Cardholder:** As a cardholder, I want [feature] so that [benefit].  
- **Admin:** As an administrator, I want [control] to keep [metadata/operations] reliable.  
- **Integrator:** As an integration developer, I want [MQ/DB2 contract] so that [external system] can consume CardDemo data.
### Complexity Guidelines
- **Simple (1-2 pts):** Update a BMS message or extend an existing VSAM field.  
- **Medium (3-5 pts):** Add a new validation path involving DB2 or MQ, or extend a batch job with new dataset handling.  
- **Complex (6-8 pts):** Add a new MQ-driven transaction that spans VSAM → IMS → DB2 or rework nightly batch dependencies.
### Acceptance Criteria Patterns
- **Core Platform:** Must authenticate via `CC00`, update VSAM, and show success/failure via BMS message area; invalid input must refresh the map with error messaging.  
- **Authorization:** MQ requests must write to IMS before responding; fraud marking updates DB2 and refreshes the detail screen.  
- **Transaction Type Management:** CRUD operations must run against DB2 with SQLCA validation and propagate to VSAM via `TRANEXTR`.  
- **MQ Account Extractions:** Responses must include correlated IDs, proper field lengths (10+ chars for IDs, 300 for account data), and handle MQ timeouts gracefully.

---

## ⚡ Performance Budgets
- **Online transactions:** Target <1 second for approval UI flows (`CAUP`, `CT02`, `CPVS`).  
- **MQ authorizations:** Aim for <2 seconds end-to-end queue round-trip plus IMS/DB2 persistence.  
- **Batch jobs:** `POSTTRAN`, `CREASTMT`, `TRANEXTR`, and `TRANIDX` should complete within 15–30 minutes nightly; `CBPAUP0J` should finish within maintenance window.  
- **Database operations:** DB2 operations should stay under 500 ms (P95) for admin screens; IMS reads/writes under 250 ms for pending authorizations.  
- **Data loads:** VSAM dataset refreshes (`ACCTFILE`, `CARDFILE`) should finish within 20 minutes.

---

## 🚨 Readiness Considerations
### Technical Risks
1. **Missing datasets:** Required VSAM and DB2/IMS objects must be provisioned before online/batch jobs run; failure halts most flows.  
2. **Optional subsystem dependencies:** MQ, IMS, and DB2 must all be wired before `CP00`, `CTTU`, `CDRD`, and `CDRA` can operate.  
3. **SQLCA/Binding drift:** Precompiled DB2 programs depend on correct DB2 plan/bind; mismatched SQLCA causes immediate CICS abends.
### Tech Debt
- **Copybook drift:** Shared copybooks (e.g., `CVACT02Y`) require synchronized changes; inconsistencies risk data corruption.  
- **Scheduler visibility:** CA7/Control-M definitions in `app/scheduler` need better documentation for new team members.
### Sequencing for User Stories
- **Prerequisites:** Provision VSAM datasets, define CICS resources, register RACF users, load optional DB2/IMS/MQ definitions.  
- **Recommended order:** 1) Core platform baseline (authentication → transaction posting → batch), 2) Transaction type DB2 integration, 3) Authorization MQ stack, 4) MQ account extractions.

---

## 📈 Success Metrics
### Adoption
- **Target:** 100% of pilot teams run the nightly batch pipeline within 3 hours of the scheduled start.  
- **Engagement:** Admins execute `TRANEXTR` weekly and assess MQ-authorization results through `CPVS/CPVD`.  
- **Retention:** Teams continue leveraging CardDemo as a regression bench when optional extensions are enabled.
### Business Impact
- **Metric 1:** Cut manual authorization testing time by 50% by routing requests through the MQ-based `CP00`/`CPVS` screens.  
- **Metric 2:** Ensure new transaction types appear in VSAM within a single nightly `TRANEXTR` run so postings stay accurate.

*Last updated: 2026-03-04*
