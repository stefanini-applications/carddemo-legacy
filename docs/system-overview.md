# System CardDemo - Overview for User Stories

**Version:** 2026-03-11  
**Purpose:** Single source of truth for Mainframe-focused user stories spanning the core CardDemo experience and its optional extensions.

---

## 📊 Platform Statistics
- **Technology Stack:** IBM Enterprise COBOL & Assembler programs running inside CICS TS with VSAM KSDS datasets, batch JCL, optional Db2, IMS DB, and MQ adapters.  
- **Architecture Pattern:** Transaction-driven mainframe architecture with BMS-screened CICS programs as the online API layer and batched JCL jobs for back-office data refresh.  
- **Key Capabilities:** Account/card management, transaction reporting, statement generation, authorization tracking, DB2-based reference management, MQ/VSAM extraction experiments.  
- **Supported Languages:** English (texts live inside BMS map definitions under `app/bms/` and copybooks rather than an external locale registry).

---

## 🏗️ High-Level Architecture

### Technology Stack
**Backend:** COBOL/Assembler (CICS & Batch), JCL, Db2 static SQL, IMS DB PSBs, MQ channels.  
**Frontend:** CICS BMS maps rendered by the CICS TS terminal interface.  
**Database:** VSAM KSDS (AIX/TRANSACT/ACCTDAT etc.), Db2 tables (`TRANSACTION_TYPE`, `AUTHFRDS`), IMS DB HIDAM.  
**Cache & Messaging:** VSAM buffers inside CICS, MQ request/response queues for asynchronous services.  
**Others:** CICS Resource definitions (CSD), CICS-DB2, CICS-IMS, CICS-MQ / JCL utilities in `app/jcl/`, `app/csd/`, `app/ctl/`.

### Architectural Patterns
- **Transaction Boundary Enforcement:** Each CICS transaction (e.g., `CC00`, `CP00`, `CTTU`) scopes VSAM/Db2/IMS interactions within a single unit of work with conditional COMMIT/ROLLBACK.  
- **Service Encapsulation:** BMS maps invoke dedicated COBOL programs per screen; shared copybooks (`app/cpy/`, `app/cpy-bms/`) centralize record definitions.  
- **Integration Layers:** Optional modules attach to MQ (`app/app-vsam-mq/`), Db2 (`app/app-transaction-type-db2/`), and IMS (`app/app-authorization-ims-db2-mq/`), keeping the core VSAM pipeline untouched until extensions are enabled.  
- **Security & Sessions:** Signon transaction `CC00` validates credentials against `AWS.M2.CARDDEMO.USRSEC.PS`, establishing session state before opening navigation screens.

---

## 📚 Module Catalog

<!-- MODULE_LIST_START -->
**Modules:** carddemo-core, authorization-ims-db2-mq, transaction-type-db2, vsam-mq-extractions
<!-- MODULE_LIST_END -->

### 1. Carddemo Core
**ID:** `cardemo-core`  
**Purpose:** Host the baseline credit-card lifecycle: user sign-on, account/card CRUD, transaction capture, billing, and report generation along with the necessary batch jobs that seed VSAM datasets.  
**Key Components:** `app/cbl/` online COBOL programs, `app/bms/` CICS mapsets, `app/cpy/` & `app/cpy-bms/` copybooks, `app/jcl/` batch jobs, `app/data/` sample datasets, `app/csd/` CICS resource definitions.  
**Public APIs:**  
- `CICS CC00` – Sign-on entry point feeding `COSGN00C`.  
- `CICS CAUP/CA00/CAVW` – Account view/update screens backed by VSAM `ACCTDATA`, `CARDDATA`.  
- `CICS CT00/CT01/CT02` – Transaction list/view/add; writes to `AWS.M2.CARDDEMO.TRANSACT.VSAM.KSDS`.  
- `JCL POSTTRAN`, `CREASTMT`, `TRANIDX` – Batch jobs that rebuild matrices, statements, and indexes nightly.  
- `CICS CB00` – Bill payment flow that updates VSAM balances and generates report records.  
**User Story Examples:**  
- As a cardholder, I want to view my latest transactions (CT00/CT01) so that I can reconcile spending before the next statement.  
- As an admin, I want to suspend a card via CAUP so that suspected fraud stops transaction posting immediately.  

### 2. Authorization IMS & Db2 MQ Bridge
**ID:** `authorization-ims-db2-mq`  
**Purpose:** Provide end-to-end pending authorization handling: MQ-triggered authorizations, IMS persistence, DB2 fraud logging, and batch purging for expired holds.  
**Key Components:** `app/app-authorization-ims-db2-mq/cbl/` COBOL programs `COPAU*`, IMS DBD/PSB definitions, DB2 DDL (`AUTHFRDS`), MQ configs, and BMS screens (`CPVS`, `CPVD`).  
**Public APIs:**  
- `MQ Authorization Request/Response` – Cloud clients post to `CARDDEMO.REQUEST.QUEUE`, COBOL program `COPAUA0C` responds via `CARDDEMO.RESPONSE.QUEUE`.  
- `CICS CPVS/CPVD` – Pending authorization summary/details with IMS reads and DB2 inserts.  
- `Batch CBPAUP0J` – Purges expired IMS authorizations and reconciles credit.  
**User Story Examples:**  
- As an analyst, I want to mark suspicious authorizations (CPVD) so that fraud flags are stored in DB2.  
- As a cloud integrator, I want MQ messages formatted according to the provided request/response schema so that my service can simulate real card swipes.

### 3. Transaction Type Management with Db2
**ID:** `transaction-type-db2`  
**Purpose:** Replace VSAM reference tables for transaction types with Db2 persistence, exposing admin CRUD screens and batch synchronization jobs.  
**Key Components:** `app/app-transaction-type-db2/cbl/` programs `COTRTU*`, `COTRTL*`, embedded Db2 SQL, `ddl/` DDL scripts for `TRANSACTION_TYPE` and `TRANSACTION_TYPE_CATEGORY`, `jcl/` jobs (`CREADB21`, `TRANEXTR`, `MNTTRDB2`).  
**Public APIs:**  
- `CICS CTTU/CTLI` – Add/edit and list/delete transaction types with Db2 cursors.  
- `JCL TRANEXTR` – Extracts latest Db2 rows into VSAM files for the core pipeline.  
- `JCL MNTTRDB2` – Batch maintenance job that syncs VSAM/Db2 reference data.  
**User Story Examples:**  
- As an admin, I want to add a new transaction type via CTTU so that the billing code can appear in statements.  
- As a batch operator, I want TRANEXTR to refresh VSAM reference files overnight so that online transactions see the latest codes.

### 4. VSAM + MQ Account Extraction
**ID:** `vsam-mq-extractions`  
**Purpose:** Demonstrate MQ request/response patterns for on-demand queries: system-date (CDRD) and account detail (CDRA) lookups by bridging VSAM with MQ.  
**Key Components:** `app/app-vsam-mq/cbl/` programs `CODATE01`, `COACCT01`, MQ definitions (`DEFINE MQCONN`, `MQQUEUE`), request/response structures, optional listeners that translate VSAM records for MQ clients.  
**Public APIs:**  
- `CICS CDRD/CDRA` – MQ-laced inquiry screens triggered by user input, retrieving VSAM records and correlating replies.  
- `MQ Request/Response` – Date/account message formats documented in the module README.  
**User Story Examples:**  
- As an integration tester, I want CDRA to return account details over MQ so that an MQ consumer can reconcile remote systems.  
- As an operator, I want CDRD to confirm the host date through MQ before running time-sensitive batch jobs.

---

## 🔄 Architecture Diagram

```mermaid
flowchart LR
    subgraph CICS Region
        BMS(CICS BMS maps) --> COBOL(COBOL/Assembler programs)
        COBOL --> VSAM(VSAM KSDS files)
        COBOL --> DB2(Db2 tables)
        COBOL --> IMS(IMS DB HIDAM)
        COBOL --> MQ(IBM MQ queues)
    end
    subgraph Batch Layer
        JCL[JCL jobs (POSTTRAN, TRANEXTR, etc.)]
        JCL --> VSAM
        JCL --> Db2Sync(Db2 extraction)
    end
    subgraph Optional Extensions
        MQModule[Authorization + Account MQ flows] --> MQ
        Db2Module[Db2 Transaction Type BR] --> Db2
        IMSModule[IMS-based authorizations] --> IMS
    end
    BMS --> UserTerminals[3270/Cloud MQ clients]
    UserTerminals --> MQModule
``` 
```

---

## 🧭 Actors and User Journeys
- **Regular cardholder:** Signs on via `CC00`, navigates `CAUP`/`CAVW`, adds payments (`CB00`), lists transactions (`CT00`), and reviews statements generated by `CREASTMT`.  
- **Admin:** Uses the Admin Menu (`CA00`) to manage users (`CU00-03`), extend transaction catalog (`CTTU`, `CTLI`), and audit pending authorizations (`CPVS`, `CPVD`).  
- **Integration specialist:** Posts MQ requests to `CARDDEMO.REQUEST.QUEUE`, waits for replies, and bothers the VSAM/MQ bridges for synchronous system-date/account results.  
- **Batch operator:** Executes `ACCTFILE` → `CARDFILE` → `POSTTRAN` → `CREASTMT` sequences nightly, plus optional `TRANEXTR` and `CBPAUP0J` when Db2/MQ modules are active.

---

## 📊 Data Models

| Artifact | Description |
| --- | --- |
| `AUTHFRDS` (Db2) | Tracks fraud metadata per authorization; key columns include `CARD_NUM`, `AUTH_TS`, `AUTH_RESP_CODE`, `FRAUD_RPT_DATE`, `AUTH_FRAUD`. |
| `TRANSACTION_TYPE` / `TRANSACTION_TYPE_CATEGORY` | Db2 tables holding reference data that synchronizes into VSAM through `TRANEXTR` and `MNTTRDB2`. |
| VSAM datasets (`ACCTDATA`, `CARDDATA`, `TRANSACT`, `DALYTRAN`) | Store account/card/transaction records with lengths 300–350 bytes as described in the installation tables; copybooks such as `CVACT01Y`, `CVCUS01Y`, `CVTRA06Y` define their layouts. |

```sql
CREATE TABLE <<db2-schema>>.AUTHFRDS (
  CARD_NUM CHAR(16) NOT NULL,
  AUTH_TS TIMESTAMP NOT NULL,
  AUTH_RESP_CODE CHAR(2),
  AUTH_FRAUD CHAR(1),
  FRAUD_RPT_DATE DATE,
  PRIMARY KEY(CARD_NUM, AUTH_TS)
);
```

---

## 🛰️ Public Interfaces - Sample Payloads

### Authorization MQ request/response
- Input queue `AWS.M2.CARDDEMO.PAUTH.REQUEST` accepts comma-separated values such as `AUTH-DATE`, `CARD-NUM`, `TRANSACTION-AMT`, `MERCHANT-CITY`, `TRANSACTION-ID`.  
- Output queue `AWS.M2.CARDDEMO.PAUTH.REPLY` returns correlated values `CARD-NUM`, `TRANSACTION-ID`, `AUTH-RESP-CODE`, and `APPROVED-AMT`.  
```text
AUTH-DATE,AUTH-TIME,CARD-NUM,AUTH-TYPE,CARD-EXPIRY-DATE,MESSAGE-TYPE,MESSAGE-SOURCE,PROCESSING-CODE,TRANSACTION-AMT,MERCHANT-CATAGORY-CODE,ACQR-COUNTRY-CODE,POS-ENTRY-MODE,MERCHANT-ID,MERCHANT-NAME,MERCHANT-CITY,MERCHANT-STATE,MERCHANT-ZIP,TRANSACTION-ID
```
```text
CARD-NUM,TRANSACTION-ID,AUTH-ID-CODE,AUTH-RESP-CODE,AUTH-RESP-REASON,APPROVED-AMT
```

### MQ-backed inquiries (CDRD/CDRA)
- Date/account requests are submitted as fixed layouts and answered with report-sized payloads so MQ consumers can synchronize with host data.  
```text
DATE-REQUEST-MSG: REQUEST-TYPE ("DATE"), REQUEST-ID
DATE-RESPONSE-MSG: RESPONSE-TYPE ("DATE"), RESPONSE-ID, SYSTEM-DATE
ACCT-REQUEST-MSG: REQUEST-TYPE ("ACCT"), REQUEST-ID, ACCOUNT-NUMBER
ACCT-RESPONSE-MSG: RESPONSE-TYPE ("ACCT"), RESPONSE-ID, ACCOUNT-DATA (300 bytes)
```

---

## 📋 Business Rules by Module

### Carddemo Core - Rules
- Daily batch pipeline must refresh VSAM datasets before CICS opens (CLOSEFIL → OPENFIL).  
- Transactions (`CT00/CT02`) must validate sufficient credit and lock the VSAM `TRANSACT` record before posting.  
- User management (`CU01-03`) is limited to admin sessions established via `CC00`.

### Authorization IMS & Db2 MQ Bridge - Rules
- Authorization requests cannot be processed unless the MQ queues, IMS DB, and DB2 tables are all defined.  
- CPVS/CPVD screens must log each interaction in IMS and mirror fraud flags into `AUTHFRDS`.  
- CBPAUP0J purges expired authorizations and releases held credit every night.

### Transaction Type Management with Db2 - Rules
- Db2 transactions require embedded SQL host variables declared in `dcl/` and error handling via `SQLCA`.  
- Deleting a transaction type from Db2 fails if it is referenced by active VSAM transactions (TRANEXTR/JCL guard).  
- `TRANEXTR` must run after each `MNTTRDB2` to keep VSAM reference files in sync.

### VSAM + MQ Account Extraction - Rules
- MQ messages must include correlation IDs so CDRD/CDRA can match replies.  
- Account lookups (CDRA) only succeed if the provided account number exists in VSAM cross-reference tables (`CARDXREF`).  
- System date inquiries (CDRD) always respond with the host date from the same batch cycle to keep batch schedules aligned.

---

## 🌐 Internationalization and Translation
Text strings are embedded directly inside the BMS map definitions stored under `app/bms/` (for example, `COSGN00.A` for the signon map). Each map handles labels, messages, and validations; there is no external locale directory. Copybooks (`app/cpy/`, `app/cpy-bms/`) carry field-level descriptions for form labels, so translations require editing the BMS map segments and redistributing the load library.

---

## 📋 Form and Listing Patterns
### Component Structure Analysis
- **No reusable UI framework:** Each feature defines its own BMS map (`app/bms/COACTVW`, `COCRDUP`, `COPAU00`), and the corresponding COBOL program orchestrates the screen.  
- **Forms-on-BMS:** Input validation lives in COBOL `IF`/`EVALUATE` blocks; confirmation flows (e.g., `CB00` bill payment) rely on sequential screen swaps rather than modal overlays.  
- **Lists:** Transaction and authorization lists (`CT00`, `CPVS`) render by looping over VSAM index records and embedding them into map fields. Paging is manual (forward/backward statements keyed by `CTLI` and `CPVS`).

### Validation and Notifications
- Validation is embedded in COBOL: example `IF ACCOUNT-BALANCE < 0 THEN DISPLAY 'Insufficient funds'` within `COACTUP0C`.  
- Notifications manifest as next-screen messages in BMS maps; there is no global snackbar, the standard pattern is to set a conditional message field and redisplay the map.

### Listing Pattern
- Tables consist of BMS map rows repeated line-by-line; pagination is explicit via `CTLI`: COBOL reads the next/previous record set and rewrites the map, showing edit/delete actions tied to `COTRTLIC`.

---

## 📝 User Story Patterns
### Templates by Domain
- **Cardholder stories:** As a cardholder, I want to view/update my account (CAUP) so that I can keep my billing information current.  
- **Admin stories:** As an admin, I want to manage users, transaction types, and pending authorizations so that policy enforcement stays in sync with compliance.  
- **Integration stories:** As an MQ consumer, I want the account and date services to mirror VSAM and host time so that external services can orchestrate batch workflows.

### Story Complexity (Points)
1. **Simple (1–2 pts):** Add a new message to an existing BMS map (e.g., error text for CAUP).  
2. **Medium (3–5 pts):** Introduce a new transaction type CRUD path in CTTU + TRANEXTR sync.  
3. **Complex (5–8 pts):** Connect MQ authorization flows to Db2 fraud logging plus nightly purge (CBPAUP0J) and audit updates.

### Acceptance Criteria Patterns
- **Authentication:** Signon flow must reject invalid credentials and redirect to CC00 for re-entry.  
- **Validation:** Each COBOL program must check conditional flags before writing to VSAM/Db2/IMS and surface a BMS message on failure.  
- **Performance:** CICS transactions should finish within 2 seconds (P95) under test load.  
- **Error:** MQ replies must carry correlation IDs and supply human-readable errors when VSAM lookups fail.

---

## ⚡ Performance Budgets
- **Online Response (CICS SAS tokens):** < 2 seconds (P95) for interactive screens (`CAUP`, `CT00`, `CPVS`) when VSAM and Db2 are healthy.  
- **MQ Latency:** < 3 seconds for request/response roundtrip (CDRD/CDRA + Authorization queue).  
- **Batch Run Time:** Nightly jobs like `POSTTRAN`, `TRANIDX`, `CREASTMT`, `TRANEXTR` should finish within 5 minutes per dataset to keep the overnight window manageable.  
- **Cache/VSAM throughput:** Keep VSAM I/O waits below 100ms; maintain `CBPAUP0J` run under 4 minutes for purging expired authorizations.

---

## 🚨 Readiness Considerations
### Technical Risks
- **[RISK-1]:** Dependence on mainframe middleware (CICS/Db2/IMS/MQ) → Mitigation: Validate resource definitions in `app/csd/` and ensure CSD load libraries sync before deployment.  
- **[RISK-2]:** Manual BMS map editing is error-prone → Mitigation: Keep map sets under version control (`app/bms/`) and run NEWCOPY sequences via documented JCL.  
- **[RISK-3]:** MQ and Db2 optional modules bring schema drift → Mitigation: Gate deployment with installer `CREADB21` and `TRANEXTR` jobs, and maintain watchers on MQ queues.

### Tech Debt
- **BMS-centric UI:** No modern UI or locale framework → Resolution: Consider wrapping CICS screens with REST front-ends as a future modernization spike.  
- **Copybook sprawl:** Shared copybooks live in `app/cpy/`/`cpy-bms/` but lack formal linting → Resolution: Introduce code-generation or pattern enforcement for new copybooks.

### Sequencing for US
- **Prerequisites:** Ensure VSAM datasets (`ACCTDATA`, `CARDDATA`, `TRANSACT`, etc.) are built and seeded; install optional Db2/IMS/MQ resources before enabling corresponding transactions.  
- **Recommended order:** 1) Core dataset setup + CC00 signon, 2) Transaction screens (CT00/CAUP) + batch jobs, 3) Optional Db2/IMS/MQ features in the order of dependencies (Db2 → MQ → IMS + DB2 fraud integration) to minimize refactor.

---

## 📈 Success Metrics
### Adoption
- **Target:** 90% of active cardholders should execute at least one CAUP/CT00 flow per billing cycle.  
- **Engagement:** Track MQ service usage rate (CDRD/CDRA) and pending authorization lookups once the extension is enabled.  
- **Retention:** Ensure admin guidelines keep transaction type updates aligned with regulatory changes by measuring how often CTTU/CTLI runs per month.

### Business Impact
- **[METRIC-1]:** Reduce fraud-treatment turnaround by 30% when IMS/DB2 module is active (measured by CPVS/CPVD throughput).  
- **[METRIC-2]:** Improve transaction accuracy by ensuring Db2-managed reference data stays synchronized via TRANEXTR.

*Last updated: March 11, 2026*
