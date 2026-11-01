# Cubera Wealth Management Platform — Frontend Design

**Document type:** Frontend solution design  
**Version:** 0.1 — Draft for discovery and stakeholder review

## Document purpose

This document describes the React frontend: application structure, navigation, screens, dashboard experience, API consumption, accessibility, browser-side security and delivery considerations. The prototype should use synthetic data only.

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

## 4. Frontend solution design

### 4.1 Information architecture

``` text
Workspace
  Overview
  Clients
  Family Groups
  Portfolios
  Asset Allocation
  Reports

Governance
  Compliance
  Approvals
  Audit Trail

Administration
  Users & Roles
  Reference Data
  System Settings
```

Menu visibility is not a security control. The backend must enforce
every permission.

### 4.2 Screen inventory

  -----------------------------------------------------------------------
  Screen                  Main content            Key interactions
  ----------------------- ----------------------- -----------------------
  Executive dashboard     AUM/AUA cards,          Date, currency, scope
                          client/family counts,   filters and drill-down
                          trend, allocation,      
                          action list             

  Client list             Search, RM, status,     Search, filter, create,
                          risk, review date       export if permitted

  Client detail           Overview, contacts,     Edit, upload, submit
                          KYC, risk, family,      for review
                          accounts, documents,    
                          activity                

  Family list             Family, primary         Search, filter, open
                          contact, RM, members,   
                          consolidated wealth     

  Family detail           Members, relationships, Add member, record
                          ownership, accounts and relationship,
                          consolidated assets     drill-down

  Portfolio list          Account, owner,         Filter and open
                          custodian, currency,    
                          value and date          

  Portfolio detail        Holdings, transactions, Date filter, import and
                          allocation and          reconciliation
                          valuation history       

  Instrument master       Identifier, name,       Search and maintain
                          class, exchange and     reference data
                          currency                

  Reports                 Templates, report       Generate, preview and
                          history and status      download

  Compliance              KYC status, exceptions, Assign, review and
                          expiry and review queue escalate

  Approvals               Pending maker-checker   Approve, reject or
                          requests                request changes

  Audit trail             Actor, action, entity,  Search and filter
                          timestamp and           
                          correlation ID          

  Administration          Users, roles and        Manage access
                          permissions             
  -----------------------------------------------------------------------

### 4.3 Executive dashboard

The first dashboard should include: - Reporting date, selected currency
and data freshness. - Summary cards for AUM/AUA (as defined), client
relationships, family groups and open compliance actions. - Historical
assets trend. - Asset allocation by asset class. - Priority
client/family relationships. - Attention list for expiring KYC, pending
approvals and reviews. - Drill-down to the records supporting each
metric.

**AUM, AUA, net worth and total portfolio value are not
interchangeable.** Cubera must approve each metric's definition,
inclusion rules, source and as-of date before production display.

### 4.4 Frontend project structure

``` text
cubera-web/
  public/
  src/
    app/
      App.tsx
      router.tsx
      providers/
      layouts/
    api/
      client.ts
      generated/
      errors.ts
    components/
      common/
      data-grid/
      charts/
      forms/
      feedback/
    features/
      dashboard/
      clients/
      families/
      portfolios/
      instruments/
      reports/
      compliance/
      approvals/
      audit/
      administration/
    auth/
    theme/
      tokens.ts
      theme.ts
    utils/
    test/
  package.json
  vite.config.ts
```

Each feature should own its pages, domain components, hooks and types.
Put only genuinely reusable components in `components/common`.

### 4.5 Frontend standards

-   Enable TypeScript strict mode.
-   Keep API DTOs separate from view-model types.
-   Generate or validate API types from OpenAPI where practical.
-   Never calculate official valuations or returns in the browser.
-   Display server-provided totals and calculation metadata.
-   Handle loading, empty, stale-data and error states.
-   Use server-side pagination/filtering for large datasets.
-   Format money by currency; never assume INR for every account.
-   Display reporting/as-of date beside financial figures.
-   Do not store sensitive KYC data in browser local storage.
-   Support keyboard navigation, accessible labels and adequate
    contrast.
-   Confirm high-impact actions and display their outcomes.

### 4.6 Frontend security

-   Use OIDC Authorization Code with PKCE through an approved identity
    provider.
-   Prefer a secure BFF/session-cookie design where feasible; otherwise
    use the identity provider's approved SPA pattern.
-   Cookies must be `HttpOnly`, `Secure` and appropriately `SameSite`.
-   Protect against CSRF when using cookie-based authentication.
-   Do not store tokens or KYC documents in local storage.
-   Enforce authorization on the server for every request.
-   Apply session timeout, MFA and re-authentication for sensitive
    actions as policy requires.
-   Hidden buttons and route guards are usability features, not security
    controls.

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
