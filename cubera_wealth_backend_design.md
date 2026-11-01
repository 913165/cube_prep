# Cubera Wealth Management Platform — Backend Design

**Document type:** Backend and platform solution design  
**Version:** 0.1 — Draft for discovery and stakeholder review

## Document purpose

This document describes the Spring Boot backend and platform: module boundaries, domain model, database, APIs, financial calculations, workflows, integrations, security, audit, deployment and operational controls. Financial metrics and compliance workflows require validation with Cubera stakeholders before production implementation.

## Backend and Frontend Solution Design

**Version:** 0.1 --- Draft for discovery and stakeholder review\
**Audience:** Cubera stakeholders, product owner, architects,
engineering, QA, security and operations.

------------------------------------------------------------------------

## 1. Executive summary

This document proposes a technical design for a web-based Wealth
Management Platform for Cubera Advisors. The initial scope covers:

1.  Client and family management
2.  Portfolio and asset management
3.  Wealth dashboards and reporting
4.  Compliance and audit

The platform will provide a unified view of clients, family
relationships, investment accounts, assets, holdings, valuations,
compliance status and operational activities. The recommended first
release is an internal platform; a client-facing portal can be added
later.

The first deliverable should be a high-fidelity React prototype using
synthetic data. Backend services and integrations should follow once
Cubera validates workflows, metric definitions, source systems,
jurisdictional requirements and MVP scope.

### Design principles

-   Financial calculations must be deterministic, reproducible and
    independently tested.
-   Material changes and approvals must be attributable to a user, time
    and reason.
-   Apply least privilege, encryption, strong authentication and
    data-scope authorization.
-   Preserve source, effective date and verification status for imported
    or manually entered data.
-   Start with a modular monolith; extract services only when justified.
-   Keep integrations behind adapters rather than coupling domain logic
    to vendors.
-   AI may assist with extraction or summaries later, but must not make
    autonomous financial or compliance decisions.

### Assumptions to validate

Confirm with Cubera before implementation: - Legal entities and
regulatory jurisdictions in scope. - Internal users only or client
access as well. - Whether the service is advisory, discretionary
management or another model. - Whether the product is reporting-only or
includes order execution. - Asset classes, markets, currencies and
valuation sources. - Custodians, brokers, market-data, FX and KYC/AML
providers. - Data residency, privacy, retention, audit and reporting
obligations. - Existing CRM, portfolio accounting and document
systems. - Expected users, clients, families, accounts, data volumes and
service levels.

This is a proposed design, not a description of Cubera's existing
systems or regulatory status. Dashboard figures and names in the
prototype are fictional.

------------------------------------------------------------------------

## 2. Product scope

### 2.1 Initial functional scope

  -----------------------------------------------------------------------
  Capability                          Scope
  ----------------------------------- -----------------------------------
  Client management                   Individual/entity profiles,
                                      contacts, identifiers, RM
                                      assignment and lifecycle

  Family management                   Family groups, members,
                                      relationships, ownership interests
                                      and consolidated view

  KYC and documents                   Document metadata, secure storage
                                      reference, expiry and review status

  Risk profile                        Questionnaire, classification,
                                      assessment date, reviewer and next
                                      review

  Portfolio management                Accounts, instruments,
                                      transactions, positions and
                                      valuations

  Wealth dashboard                    Approved AUM/AUA metrics, client
                                      counts, allocation, trends and
                                      attention items

  Reporting                           Client/family statements,
                                      valuation, allocation and
                                      performance reports

  Compliance                          Review tasks, configurable checks,
                                      exceptions and maker-checker
                                      approvals

  Audit                               Append-only business activity
                                      history

  Administration                      Users, roles, permissions and
                                      reference data
  -----------------------------------------------------------------------

### 2.2 Out of scope until separately approved

-   Trade execution, order routing or movement of client assets.
-   Automated recommendations or portfolio rebalancing.
-   Tax filing, legal/estate advice or general ledger.
-   Automated regulatory submissions.
-   AI-driven compliance decisions.
-   Client mobile application.

These capabilities materially change risk, integrations, testing and
regulatory scope.

### 2.3 User personas

  -----------------------------------------------------------------------
  Persona                             Responsibilities
  ----------------------------------- -----------------------------------
  Relationship Manager                Manage assigned client
                                      relationships and reviews

  Investment Analyst                  Review holdings, allocation and
                                      performance

  Operations Executive                Maintain client, account and
                                      document data

  Compliance Officer                  Review KYC/AML status, exceptions
                                      and evidence

  Approver/Supervisor                 Approve sensitive changes and
                                      exceptions

  Management                          Review aggregate business and
                                      wealth metrics

  System Administrator                Manage users, roles and
                                      configuration

  Client (future)                     View explicitly authorized
                                      personal/family data
  -----------------------------------------------------------------------

Job title alone must not grant access. Permissions require roles and
data scope.

------------------------------------------------------------------------

## 3. High-level architecture

### 3.1 Recommended architecture

Start with a **modular monolith**: one Spring Boot deployment with clear
module boundaries, separate packages and explicit ownership of domain
data. This reduces early distributed-system complexity while preserving
future extraction options.

``` mermaid
flowchart TB
    U[Internal Users / Future Clients] --> FE[React + TypeScript]
    FE --> GW[HTTPS API / Reverse Proxy]
    GW --> APP[Spring Boot Modular Monolith]
    APP --> DB[(PostgreSQL)]
    APP --> OBJ[(Encrypted Object Storage)]
    APP --> JOB[Scheduled / Batch Jobs]
    JOB --> EXT[Custodian, Broker, Market Data, KYC]
    APP --> MSG[Email / Notification Provider]
```

### 3.2 Logical layers

1.  **Presentation:** React pages, reusable components, routing and API
    state.
2.  **API:** REST controllers, validation, response models, versioning
    and error handling.
3.  **Application:** use cases, authorization, transaction boundaries
    and orchestration.
4.  **Domain:** client, family, portfolio, instrument, valuation,
    compliance and audit rules.
5.  **Infrastructure:** PostgreSQL repositories, object storage,
    provider adapters, messaging and jobs.

Controllers should remain thin. Business rules must not live in React
components or controller methods.

### 3.3 Technology choices

  -----------------------------------------------------------------------
  Layer                   Proposed technology     Notes
  ----------------------- ----------------------- -----------------------
  Frontend                React + TypeScript      Typed, component-driven
                                                  UI

  Build                   Vite                    Frontend build tooling

  UI foundation           MUI, customized         Establish Cubera-owned
                                                  design tokens and
                                                  components

  Charts                  Apache ECharts          Allocation and
                                                  performance
                                                  visualizations

  Data grids              AG Grid or MUI Data     Confirm licensing
                          Grid                    before selection

  Routing                 React Router            Nested and protected
                                                  routes

  Server state            TanStack Query          Fetching, caching and
                                                  invalidation

  Forms                   React Hook Form + Zod   Form handling and
                                                  validation

  Backend                 Java 21 LTS + Spring    Confirm version against
                          Boot                    project standards

  Security                Spring Security + OIDC  Approved identity
                                                  provider

  Database                PostgreSQL              Relational data and
                                                  transactions

  Migrations              Flyway                  Versioned schema
                                                  changes

  Documents               S3-compatible object    Keep document bytes out
                          storage                 of normal DB tables

  Batch                   Spring Batch or         Imports, snapshots and
                          scheduled jobs          expiry checks

  API docs                OpenAPI                 Frontend/integration
                                                  contract

  Tests                   JUnit 5,                Unit through end-to-end
                          Testcontainers,         
                          Playwright              

  Observability           Micrometer,             Metrics, traces and
                          OpenTelemetry,          logs
                          Prometheus/Grafana      

  Deployment              Docker; Kubernetes if   Environment-specific
                          justified               deployment
  -----------------------------------------------------------------------

MUI is a component foundation, not a purchased wealth-management
product. Cubera's information architecture, styling, workflows and
interaction design remain custom.

------------------------------------------------------------------------

## 5. Backend solution design

### 5.1 Backend module boundaries

``` text
com.cubera.wealth
  identity/
    api/
    application/
    domain/
    infrastructure/
  client/
  family/
  kyc/
  document/
  risk/
  portfolio/
  instrument/
  transaction/
  valuation/
  performance/
  dashboard/
  reporting/
  compliance/
  approval/
  audit/
  notification/
  shared/
```

Each module should expose application-level interfaces to other modules.
Avoid unrestricted cross-module repository access.

### 5.2 Module responsibilities

**Identity and access** - Users, roles, permissions and data-scope
assignments. - Authentication federation and authorization policies. -
Privileged-action controls.

**Client and family** - Individual and legal-entity profiles. -
Contacts, addresses and identifiers. - Family membership and
relationship types. - Relationship manager assignments and lifecycle.

**KYC and documents** - Document metadata, type, issue/expiry dates and
verification status. - Secure object-storage references. - Review
history, evidence and expiry tasks. - No raw document bytes in normal
relational tables.

**Risk** - Versioned questionnaires and scoring rules. - Answers,
classification, rationale and reviewer. - Effective date and next review
date. - Traceable changes and approvals where required.

**Portfolio and assets** - Investment accounts and ownership. -
Instrument master and asset classification. - Transactions and position
records. - Source system, import batch and reconciliation status.

**Valuation and performance** - Price and FX observations with provider,
timestamp and quality status. - Position valuation snapshots. -
Portfolio aggregation and performance calculations. - Versioned
methodology and treatment of fees, taxes, cash flows and corporate
actions.

**Dashboard and reporting** - Read-optimized queries and aggregates. -
Permission-aware reports. - Asynchronous report jobs and secure
downloads. - Reproducible reports with source/as-of metadata.

**Compliance and approvals** - Review checklists, exceptions and
evidence. - Screening-provider integration where approved. -
Maker-checker workflow and segregation of duties.

**Audit** - Append-only business events. - Actor, action, entity,
timestamp, correlation ID and change summary. - Restricted access and
tamper-evident retention controls appropriate to policy.

### 5.3 Conceptual domain entities

  -----------------------------------------------------------------------
  Entity                              Purpose
  ----------------------------------- -----------------------------------
  `User`, `Role`, `Permission`        Identity and access

  `Client`                            Person or legal entity

  `ClientContact`, `ClientIdentifier` Contact and identity details

  `FamilyGroup`                       Family/household grouping

  `FamilyMembership`                  Effective-dated membership

  `Relationship`                      Relationship between parties

  `OwnershipInterest`                 Account/asset ownership and share
                                      where applicable

  `KycCase`                           KYC review instance

  `Document`                          Metadata and secure storage
                                      reference

  `RiskAssessment`                    Versioned risk assessment

  `InvestmentAccount`                 Custodian/broker/bank account

  `Instrument`                        Tradable or reportable asset

  `Transaction`                       Investment activity

  `Position`                          Holding at an effective date

  `PriceObservation`                  Instrument price and provenance

  `FxRate`                            Currency conversion observation

  `ValuationSnapshot`                 Calculated value at a point in time

  `ComplianceCase`                    Review or exception

  `ApprovalRequest`                   Maker-checker action

  `AuditEvent`                        Business activity history

  `ImportBatch`                       External/manual data import
  -----------------------------------------------------------------------

### 5.4 Important data-model decisions

**Client versus family:** a family group is not a client record.
Membership can be time-bound. Relationships and ownership must be
explicit; family membership does not imply asset ownership.

**Account ownership:** support individual, joint and entity ownership as
required. Model beneficial-interest percentages and effective dates if
needed. Do not assume every family member owns every family asset.

**Transactions versus positions:** transactions are source activity;
positions are holdings at a point in time. Derive positions from a
complete transaction ledger or store source positions according to the
approved source model. Reconcile them when both are available.

**Valuations:** retain snapshots and input provenance. A dashboard total
should be reproducible from positions, prices, FX rates, ownership rules
and valuation timestamp.

**Currency:** store amount and currency together. Never aggregate
mixed-currency values without an explicit reporting currency, FX source
and conversion timestamp.

**History:** retain effective dates and recorded timestamps. Corrections
should preserve prior values and reasons instead of silently overwriting
material financial records.

### 5.5 Suggested database schema grouping

``` text
identity
  app_user, role, permission, user_role, role_permission, user_scope

client
  client, client_contact, client_identifier, client_status_history

family
  family_group, family_membership, party_relationship, ownership_interest

kyc
  kyc_case, kyc_document, document_metadata, risk_questionnaire,
  risk_questionnaire_version, risk_assessment, risk_answer

portfolio
  custodian, investment_account, account_owner, instrument, asset_class,
  transaction, position, import_batch, import_row_error

valuation
  price_observation, fx_rate, valuation_snapshot, valuation_line,
  performance_result

compliance
  compliance_case, compliance_check, compliance_evidence,
  approval_request, approval_action

reporting
  report_definition, report_job, report_artifact

audit
  audit_event, outbox_event
```

Use UUIDs or a centrally governed identifier strategy. Add uniqueness
constraints for external identifiers within their source/provider scope.
Index real query patterns: client search, account lookup, effective-date
holdings and compliance queues.

### 5.6 Financial calculation controls

Financial calculations must be deterministic and independently testable.

-   Use Java `BigDecimal` and database `NUMERIC` for monetary values;
    never use binary floating point for money.
-   Define precision, scale and rounding policy by currency/instrument.
-   Record price and FX source and timestamps.
-   Define treatment of stale/missing prices, suspended securities and
    unavailable FX rates.
-   Define whether returns are time-weighted, money-weighted, or both.
-   Define treatment of external cash flows, fees, taxes, dividends,
    interest, splits and corporate actions.
-   Keep realized and unrealized gains/losses distinct where required.
-   Define reporting currency and ownership/consolidation rules.
-   Mark incomplete or estimated valuations clearly.

------------------------------------------------------------------------

## 6. API design

### 6.1 API conventions

-   Base path: `/api/v1`
-   JSON request and response bodies.
-   OpenAPI published with the application.
-   Consistent pagination, sorting and filtering.
-   ISO-8601 timestamps with an agreed timezone/UTC convention.
-   ISO 4217 currency codes where applicable.
-   Stable machine-readable error codes.
-   Correlation ID propagated through logs and provider calls.
-   Idempotency keys for retryable import/command operations.
-   Optimistic locking/version fields for concurrent edits.
-   Do not return sensitive identifiers or KYC data unless necessary and
    authorized.

Example paginated response:

``` json
{
  "items": [],
  "page": 0,
  "size": 25,
  "totalElements": 0,
  "totalPages": 0
}
```

### 6.2 Initial endpoint catalogue

  --------------------------------------------------------------------------------------------------
  Method                  Endpoint                                          Purpose
  ----------------------- ------------------------------------------------- ------------------------
  GET                     `/api/v1/dashboard/summary`                       Summary metrics for
                                                                            authorized scope

  GET                     `/api/v1/dashboard/aua-trend`                     Historical assets trend

  GET                     `/api/v1/dashboard/asset-allocation`              Allocation by asset
                                                                            class

  GET                     `/api/v1/clients`                                 Search/paginate clients

  POST                    `/api/v1/clients`                                 Create client

  GET                     `/api/v1/clients/{clientId}`                      Client detail

  PATCH                   `/api/v1/clients/{clientId}`                      Update permitted fields

  GET                     `/api/v1/clients/{clientId}/documents`            Authorized document
                                                                            metadata

  POST                    `/api/v1/clients/{clientId}/kyc-cases`            Start KYC review

  GET                     `/api/v1/families`                                Search family groups

  POST                    `/api/v1/families`                                Create family group

  GET                     `/api/v1/families/{familyId}`                     Family detail

  POST                    `/api/v1/families/{familyId}/members`             Add member

  GET                     `/api/v1/portfolios`                              Search authorized
                                                                            portfolios

  GET                     `/api/v1/portfolios/{portfolioId}`                Portfolio detail

  GET                     `/api/v1/portfolios/{portfolioId}/holdings`       Holdings as of date

  GET                     `/api/v1/portfolios/{portfolioId}/transactions`   Transaction history

  GET                     `/api/v1/instruments`                             Instrument lookup

  POST                    `/api/v1/imports`                                 Submit approved import

  GET                     `/api/v1/compliance/cases`                        Compliance work queue

  POST                    `/api/v1/compliance/cases/{caseId}/actions`       Record review action

  GET                     `/api/v1/approvals`                               Pending approval queue

  POST                    `/api/v1/approvals/{approvalId}/decision`         Approve/reject/request
                                                                            changes

  GET                     `/api/v1/audit-events`                            Search authorized audit
                                                                            events

  POST                    `/api/v1/reports`                                 Request report
                                                                            generation

  GET                     `/api/v1/reports/{reportId}`                      Report status and
                                                                            metadata
  --------------------------------------------------------------------------------------------------

Contracts, permissions and request/response schemas must be finalized
before implementation.

### 6.3 Dashboard API example

`GET /api/v1/dashboard/summary?asOf=2026-10-30&currency=INR`

Illustrative response only:

``` json
{
  "asOf": "2026-10-30",
  "reportingCurrency": "INR",
  "assetsUnderAdvisory": {
    "amount": "8504000000.00",
    "currency": "INR",
    "changePercent": "8.60"
  },
  "clientCount": 248,
  "familyGroupCount": 86,
  "openComplianceActions": 12,
  "dataStatus": "ILLUSTRATIVE"
}
```

Production metrics must be calculated from authorized, validated source
data and include meaningful data-quality/as-of status.

### 6.4 Error response example

``` json
{
  "code": "VALIDATION_ERROR",
  "message": "The request contains invalid fields.",
  "correlationId": "generated-correlation-id",
  "fieldErrors": [
    {
      "field": "reportingCurrency",
      "message": "A supported currency is required."
    }
  ]
}
```

Never expose stack traces, SQL details, secrets or sensitive KYC values
in API errors.

------------------------------------------------------------------------

## 7. Core business workflows

### 7.1 Client onboarding

``` mermaid
flowchart TD
    A[Create client profile] --> B[Capture identity and contact details]
    B --> C[Create KYC case and collect documents]
    C --> D[Perform required verification and screening]
    D --> E{Checks complete?}
    E -- No --> F[Request missing evidence / create exception]
    F --> C
    E -- Yes --> G[Reviewer records outcome]
    G --> H{Approval required?}
    H -- Yes --> I[Independent approver review]
    I --> J{Approved?}
    J -- No --> F
    J -- Yes --> K[Activate client]
    H -- No --> K
```

Exact KYC checks, evidence and approval rules must be configured for
applicable jurisdictions and Cubera policies.

### 7.2 Family creation and consolidation

1.  Create a family group.
2.  Add existing client records as members.
3.  Record relationship type and effective dates.
4.  Record account ownership/beneficial interests where applicable.
5.  Validate that accounts are not duplicated in consolidation.
6.  Calculate family wealth using approved ownership and valuation
    rules.
7.  Display consolidated values with drill-down to member, account and
    asset.

Family membership must not automatically imply asset ownership.

### 7.3 Portfolio ingestion and reconciliation

1.  Receive file/API payload from an approved source.
2.  Validate format, source, account and instrument identifiers.
3.  Store raw import securely where policy permits.
4.  Parse into staging records.
5.  Report rejected rows and validation errors.
6.  Detect duplicates and apply idempotency rules.
7.  Reconcile source positions/transactions against existing records.
8.  Approve/release the batch according to policy.
9.  Publish accepted data and valuation updates.
10. Record batch outcome, source, actor and audit events.

Do not overwrite accepted position history without retaining source and
effective-date information.

### 7.4 Maker-checker approval

A maker submits a sensitive change. The request enters a pending state
and a different authorized checker reviews it. The same user must not
approve their own request where segregation of duties is required. Store
submitted values, decision, reason and timestamps.

### 7.5 Dashboard data flow

Use optimized query services or precomputed snapshots. Do not trigger
expensive full-portfolio calculations on every page load. Each response
should identify reporting date, currency, data freshness and incomplete
valuation status.

------------------------------------------------------------------------

## 8. Security, privacy and compliance

Requirements must be confirmed with Cubera's compliance/legal
stakeholders and qualified advisers in each jurisdiction. This is a
technical proposal, not legal or regulatory advice.

### 8.1 Security controls

-   Central identity provider, MFA and user lifecycle management.
-   RBAC combined with record-level/data-scope authorization.
-   Least privilege for users, services and database accounts.
-   TLS in transit and encryption at rest for database, backups and
    object storage.
-   Managed secrets; no credentials in source code or frontend bundles.
-   Secure uploads with file type/size validation and malware scanning.
-   API rate limiting and abuse protection.
-   Input validation and output encoding.
-   Dependency, container, static-code and secret scanning.
-   Central security logs and alerts.
-   Periodic access reviews and privileged-user monitoring.
-   Backup, restore and disaster-recovery testing.

### 8.2 Authorization model

Example permissions:

``` text
CLIENT_READ
CLIENT_CREATE
CLIENT_UPDATE
CLIENT_SENSITIVE_READ
FAMILY_READ
FAMILY_MANAGE
PORTFOLIO_READ
PORTFOLIO_IMPORT
REPORT_GENERATE
KYC_REVIEW
COMPLIANCE_CASE_MANAGE
APPROVAL_DECIDE
AUDIT_READ
USER_ADMIN
```

Data scopes may include assigned RM book, legal entity, business unit,
family/client group or explicit client entitlement. Cubera must approve
the actual scope model.

### 8.3 Audit requirements

Capture: - Actor/user or service identity. - Action and outcome. -
Entity type and identifier. - Timestamp and correlation ID. -
Before/after values for approved auditable fields, masking or excluding
sensitive data. - Reason/comment for sensitive changes. - Approval
reference. - Source channel (UI, import, API or scheduled job).

Application users should not be able to edit or delete audit records.
Consider restricted storage, database permissions, cryptographic
chaining or immutable retention where policy requires. Avoid copying
unnecessary personal data into logs.

### 8.4 Privacy and retention

Define data classification, collection purpose, notice/consent,
retention, deletion/anonymization, legal hold, masking, export
restrictions, data residency and cross-border transfer rules with
counsel. Establish vendor due diligence and data-processing agreements.

------------------------------------------------------------------------

## 9. Reporting and analytics

### 9.1 Initial reports

-   Client profile and KYC status.
-   Family group and relationship summary.
-   Portfolio holdings statement.
-   Transaction activity.
-   Asset allocation.
-   Portfolio valuation.
-   Performance (after methodology approval).
-   Expiring-document and compliance exception report.
-   Pending approvals.
-   Audit activity.

### 9.2 Report controls

Every report should include generation time, underlying as-of date,
reporting currency, valuation/performance methodology version,
source/completeness status, generating user, confidentiality marking and
unique report ID.

Generate large reports asynchronously. Store artifacts securely,
authorize downloads at download time and expire links.

------------------------------------------------------------------------

## 10. External integrations

Use adapters with normalized internal contracts rather than coupling
domain logic to vendor APIs.

  -----------------------------------------------------------------------
  Integration             Purpose                 Considerations
  ----------------------- ----------------------- -----------------------
  Identity provider       Authentication/MFA      OIDC, lifecycle
                                                  management

  Custodian/broker        Accounts, transactions, Reconciliation and
                          positions               source IDs

  Market data             Prices and corporate    Licensing, timestamps,
                          actions                 stale data

  FX provider             Currency conversion     Base/quote convention
                                                  and provenance

  KYC/AML provider        Verification and        Jurisdiction and
                          screening               false-positive review

  Object storage          KYC/report files        Encryption, malware
                                                  scan and retention

  Email/SMS               Notifications           Approved vendor and
                                                  delivery tracking

  CRM/accounting          Existing operational    System of record and
                          data                    conflict resolution
  -----------------------------------------------------------------------

For each integration, document system of record, owner, authentication,
fields, frequency, rate limits, retry strategy, error queue,
reconciliation, SLA and commercial licensing.

------------------------------------------------------------------------

## 11. Non-functional requirements

Targets are initial proposals and must be validated against expected
volumes and business criticality.

  -----------------------------------------------------------------------
  Area                                Initial requirement
  ----------------------------------- -----------------------------------
  Availability                        Agree service target and graceful
                                      dependency failure behavior

  Dashboard latency                   Target p95 under 3 seconds for
                                      normal prepared queries

  Search/list latency                 Target p95 under 2 seconds for
                                      normal paginated queries

  Scalability                         Horizontal application scaling;
                                      avoid in-memory session dependency

  Integrity                           ACID transactions, constraints and
                                      reconciliation

  Security                            MFA, least privilege, encryption
                                      and security testing

  Accessibility                       Aim for WCAG 2.2 AA on core
                                      journeys

  Browser support                     Current supported Chrome, Edge,
                                      Firefox and Safari

  Recovery                            Define RPO/RTO; test restore and
                                      disaster recovery

  Observability                       Central logs, metrics, traces and
                                      actionable alerts

  Maintainability                     Automated tests, migrations,
                                      reviews and module boundaries

  Localization                        Currency, date, number and timezone
                                      handling
  -----------------------------------------------------------------------

Targets are not guarantees until tested using representative data and
load profiles.

------------------------------------------------------------------------

## 12. Testing strategy

### Backend

-   Domain unit tests for ownership, consolidation, valuation and
    performance.
-   PostgreSQL integration tests using Testcontainers.
-   API tests for validation, pagination, authorization and errors.
-   Migration tests from clean and supported previous schemas.
-   Import tests for duplicate, partial, malformed and out-of-order
    data.
-   Concurrency tests for approvals and simultaneous updates.
-   Calculation tests with independently reviewed golden datasets.

### Frontend

-   Component tests for forms, grids, charts and status states.
-   API contract tests against OpenAPI examples.
-   Playwright end-to-end tests for onboarding, portfolio review and
    approvals.
-   Accessibility and responsive-layout checks.
-   Permission tests confirming unauthorized data/actions are not
    exposed.

### Security and resilience

-   Dependency/container vulnerability scanning.
-   SAST and secret scanning in CI.
-   DAST and penetration testing before production.
-   Upload/download security tests.
-   Rate-limit, timeout and retry tests for integrations.
-   Backup restoration and disaster-recovery exercises.

Calculation test cases should cover rounding, missing data, FX
conversion, corporate actions, cash flows and date boundaries.

------------------------------------------------------------------------

## 13. Deployment and environments

### 13.1 Environments

``` text
Local Development
       |
Development / Integration
       |
QA / UAT
       |
Production
```

Keep separate databases, credentials and storage per environment. Do not
copy production KYC/financial data into development without an approved
masking process. Use CI/CD with automated tests, security scans and
controlled approvals. Maintain rollback/forward-fix procedures.

### 13.2 Initial topology

``` text
Users
  |
WAF / Ingress
  |
React static web app
  |
HTTPS API
  |
Spring Boot application
  |             |
PostgreSQL   Object Storage
  |
External provider adapters
```

Prefer managed PostgreSQL where possible. Kubernetes is optional: use it
only if operational support and scale/availability requirements justify
it. A managed container platform or simpler deployment may be preferable
initially.

------------------------------------------------------------------------

## 14. Delivery roadmap

This is a suggested sequence, not a committed schedule. Estimates
require discovery and team sizing.

### Phase 0 --- Discovery and design

-   Confirm business model, users and jurisdictions.
-   Define AUM/AUA/net-worth terms and calculation rules.
-   Confirm source systems, asset classes and integration vendors.
-   Agree permissions, maker-checker and compliance policies.
-   Approve wireframes, data model and API contracts.
-   Prepare synthetic demo data.

**Exit:** agreed scope, prioritized backlog, architecture decisions and
acceptance criteria.

### Phase 1 --- UI prototype

-   Cubera design system and application shell.
-   Executive dashboard using synthetic data.
-   Client/family list and detail screens.
-   Portfolio/holdings screens.
-   Compliance and approval screens.
-   Responsive layout and stakeholder feedback.

**Exit:** stakeholder approval of visual direction, navigation and
workflows. Prototype is marked non-production and contains no real
client data.

### Phase 2 --- Platform foundation

-   React application structure and API client.
-   Spring Boot modular monolith and module boundaries.
-   Identity integration, roles and authorization.
-   PostgreSQL schema and Flyway migrations.
-   Audit framework, error handling and observability.
-   CI/CD and development/QA environments.

**Exit:** authenticated users access authorized pages and audited APIs.

### Phase 3 --- Client, family and KYC

-   Client/family CRUD and workflows.
-   Relationships and ownership model.
-   Document metadata and secure storage.
-   KYC review and risk assessment.
-   Assignment and approval workflow.

**Exit:** onboarding and family maintenance pass UAT.

### Phase 4 --- Portfolio and valuation

-   Account and instrument master.
-   Transaction/position ingestion.
-   Import validation and reconciliation.
-   Price and FX provider adapters.
-   Valuation snapshots and allocation.
-   Reviewed calculation methodology.

**Exit:** agreed sample portfolios reconcile to independently verified
expected results.

### Phase 5 --- Dashboard, reporting and compliance

-   Production dashboard queries.
-   Client/family reports.
-   Compliance queues and evidence.
-   Audit search and exports.
-   Operational alerts and scheduled tasks.

**Exit:** reports reconcile, authorization tests pass and business users
complete UAT.

### Phase 6 --- Production readiness

-   Security assessment and penetration test.
-   Performance/resilience testing.
-   Backup/restore and recovery tests.
-   Data migration rehearsal.
-   Runbooks, support model and training.
-   Controlled rollout.

**Exit:** formal business, security, compliance and operations go-live
approval.

------------------------------------------------------------------------

## 15. Key risks and mitigations

  -------------------------------------------------------------------------------
  Risk                    Impact                  Mitigation
  ----------------------- ----------------------- -------------------------------
  Unclear AUM/AUA         Misleading              Agree metric definitions and
  definition              dashboards/reports      source rules first

  Inconsistent source     Incorrect               Provenance, staging,
  data                    holdings/valuations     reconciliation and quality
                                                  flags

  Incorrect family        Incorrect consolidated  Explicit effective-dated
  ownership assumptions   wealth                  ownership

  Unapproved return       Misleading performance  Agree and independently
  methodology                                     validate calculations

  Late regulatory         Rework and delay        Jurisdiction/legal/compliance
  discovery                                       discovery early

  Overly broad MVP        Cost and schedule       Prioritize core operations and
                          growth                  reporting

  Premature microservices Operational complexity  Modular monolith and documented
                                                  extraction criteria

  Sensitive-data exposure Legal and reputational  Least privilege, encryption,
                          harm                    masking and testing

  Vendor data/licensing   Missing or delayed data Validate contracts, APIs and
  constraints                                     permitted use early

  Unclear system of       Conflicting data        Define ownership and
  record                                          synchronization per domain
  -------------------------------------------------------------------------------

------------------------------------------------------------------------

## 16. Decisions required from Cubera

### Business and users

1.  Which legal entities and jurisdictions will use the platform?
2.  Internal-only initially, or client logins too?
3.  Expected users, clients, families, accounts and portfolios?
4.  What does Cubera mean by AUM, AUA and consolidated family wealth?

### Portfolio and data

5.  Which asset classes are required in release one?
6.  Which markets and currencies?
7.  Are transactions/positions sourced from custodians, Excel, manual
    entry or a combination?
8.  Which provider supplies prices and FX rates?
9.  Required valuation frequency and historical depth?
10. Which performance methodology and benchmarks?

### Compliance and operations

11. Which KYC/AML checks and vendors apply in each jurisdiction?
12. What are maker-checker and segregation-of-duties rules?
13. What retention, audit, privacy and data-residency policies apply?
14. Who may view, edit, approve and export each data category?

### Delivery and operations

15. Existing CRM, portfolio, accounting or document systems?
16. Preferred hosting environment and support model?
17. Target MVP date, budget range and delivery team?
18. Who owns product decisions, UAT and production operations?

------------------------------------------------------------------------

## 17. Recommended MVP boundary

**Include** - Internal login and role-based access. - Client profiles
and family groups. - KYC document metadata and review status. - Risk
profile capture and review. - Investment accounts, instrument master and
holdings. - Controlled CSV/Excel import with validation and
reconciliation. - Valuation using an approved price/FX source, or
clearly labelled sample/manual values during pilot. - Dashboard with
AUM/AUA only after definition and source approval. - Asset allocation
and basic client/family reports. - Compliance tasks, maker-checker
approvals and audit trail.

**Defer** - Trade execution and order management. - Automated
recommendations and rebalancing. - Complex alternative-asset
valuation. - Nonessential multi-provider integrations. - Client
self-service portal unless required for launch. - AI-driven workflows.

The MVP must not present incomplete, estimated or manually supplied
values as verified financial facts.

------------------------------------------------------------------------

## 18. Definition of done

A feature is ready for release only when: - Business acceptance criteria
are met. - API and UI authorization checks are implemented and tested. -
Validation, empty, loading and failure states are handled. - Required
business actions generate audit events. - Sensitive data is protected in
storage, logs and exports. - Automated tests pass in CI. - Database
migrations are reviewed and tested. - API and operational documentation
is updated. - Monitoring and support runbooks exist. - Product owner and
relevant control functions approve the feature.

------------------------------------------------------------------------

## 19. Final recommendation

Build Cubera as a secure, auditable, API-first wealth-management
application with a custom React experience and Spring Boot
modular-monolith backend.

The immediate next step is a focused discovery workshop: validate
dashboard metrics and workflows, confirm the MVP boundary, and identify
source systems and jurisdictional requirements. Then finalize the domain
model, API contracts, security model and delivery estimates.

### Architecture decision summary

-   **Frontend:** React + TypeScript + Vite.
-   **UI foundation:** customized MUI design system.
-   **Charts:** Apache ECharts.
-   **Backend:** Java + Spring Boot modular monolith.
-   **Database:** PostgreSQL + Flyway.
-   **Identity:** approved OIDC provider with MFA.
-   **Documents:** encrypted object storage.
-   **Integrations:** isolated provider adapters.
-   **Financial calculations:** deterministic backend domain services.
-   **Audit:** append-only business audit records with restricted
    access.
-   **Delivery:** synthetic-data UI prototype first, then phased backend
    implementation.

------------------------------------------------------------------------

## Appendix A --- Suggested repositories

``` text
cubera-wealth-web/
  React + TypeScript frontend

cubera-wealth-api/
  Spring Boot modular-monolith backend

cubera-wealth-deployment/
  Docker, environment configuration, CI/CD and infrastructure

cubera-wealth-docs/
  Requirements, architecture decisions, API contracts and runbooks
```

A monorepo is also viable if coordinated versioning is preferred. Choose
based on team ownership and release practices.

## Appendix B --- Architecture decisions to record

Maintain Architecture Decision Records (ADRs) for: - Modular monolith
versus service extraction. - Identity provider and browser
authentication pattern. - Client/family/party and ownership model. -
Source of truth for accounts, transactions and positions. - AUM/AUA and
valuation methodology. - Market-data and FX providers. - Audit
immutability and retention. - Document storage, malware scanning and
retention. - Deployment platform, data residency and recovery targets. -
Import format, reconciliation and correction policy.

**End of document --- Version 0.1**

**End of document — Version 0.1**
