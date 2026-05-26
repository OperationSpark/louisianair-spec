# Architecture

## Executive Summary

Louisianair delivers a unified TypeScript monorepo that orchestrates community-built IoT sensors, a NestJS ingestion API, and a Next.js dashboard so fenceline neighborhoods can trust real-time air quality insights. PostgreSQL with the Timescale extension powers sub-two-second trend queries, while Fly.io deployment and Grafana Cloud observability keep the stack reliable within the <$200/month budget. Decision records emphasize transparency, compliance with Louisiana SB503 guidelines, and developer productivity for Operation Spark apprentices.

## Project Initialization

First implementation story: initialize the Nx workspace that anchors the API, dashboard, and shared libraries.

```bash
npx create-nx-workspace@latest louisianair --preset=apps --style=css --nxCloud=false
```

## Project Context Understanding

- Documentation reviewed: PRD (2025-11-05), epic breakdown, UX specification scaffold
- Scale snapshot: 7 epics / 33 stories, Level 3 complex system spanning IoT, API, and dashboard
- Core focus: DIY sensor kits streaming ppb readings, resilient ingestion, public APIs, trust messaging, workforce enablement
- Critical NFRs: <2s regional queries, <3s dashboard load on 3G, data freshness <5 minutes, 99.5% uptime, strong TLS + key security, 48h edge buffering
- Unique challenges: Gulf Coast field durability, SB503-compliant transparency, budget-capped cloud scaling, apprentice-friendly workflows
- UX status: template placeholders—coordinate with UX workflow before dashboard implementation

## Decision Summary

| Category          | Decision                                            | Version                                            | Affects Epics                                           | Rationale                                                                                 |
| ----------------- | --------------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Starter Workflow  | Nx-managed TypeScript monorepo                      | Verify latest Nx CLI at setup                      | All                                                     | Unifies API, dashboard, and shared libs with dependency graph enforcement for apprentices |
| Data Persistence  | PostgreSQL + TimescaleDB                            | Confirm Fly.io Timescale combo before provisioning | Sensor Network, Cloud Ingestion, Public APIs, Dashboard | Hypertables handle time-series ingest/query while staying in familiar Postgres ecosystem  |
| API Pattern       | REST + OpenAPI with generated SDKs                  | Confirm NestJS Swagger package versions            | Cloud Ingestion, Public APIs, Dashboard, Compliance     | Simple contracts for sensors, typed clients for partners, rate limiting matches FR-4      |
| Authentication    | Hashed per-device API keys, Auth0 for human users   | Select bcrypt/argon2 + Auth0 SDK versions          | Cloud Ingestion, Compliance, Workforce Enablement       | Meets FR-2.3 with approachable provisioning, leaves room for HMAC/mTLS upgrades           |
| Deployment Target | Fly.io machines + managed Postgres via Terraform    | Pin Terraform + flyctl versions                    | All backend epics                                       | Cost-effective regional deployment, IaC discipline, Timescale support                     |
| Observability     | Grafana Cloud (Loki + Prometheus) via OpenTelemetry | Confirm Grafana agent + OTel SDK versions          | Reliability, Compliance, Workforce Enablement           | Unified metrics/log correlation, lightweight operations, aligns with NFR-R4               |

## Project Structure

```
louisianair/
├─ apps/
│  ├─ api/                     # NestJS ingestion + public REST API
│  ├─ dashboard/               # Next.js community dashboard (App Router)
│  └─ provisioning-cli/        # (Future) device key rotation & provisioning CLI
├─ libs/
│  ├─ domain/                  # Domain models, value objects, Timescale projections
│  ├─ shared/                  # Cross-cutting utilities (logging adapters, config)
│  └─ infrastructure/          # DB access layer, Fly.io adapters, Grafana exporters
├─ hardware/
│  ├─ firmware/                # Arduino sketches, build scripts
│  └─ pi-services/             # Edge buffering service, systemd units, docs
├─ infra/
│  ├─ terraform/               # Fly.io + Postgres provisioning and secrets
│  └─ grafana/                 # Observability dashboards & alert definitions
├─ docs/
│  ├─ architecture.md
│  ├─ PRD.md
│  ├─ epics.md
│  └─ playbooks/
└─ tools/
   └─ scripts/                 # Developer tooling (lint hooks, scaffolding)
```

## Epic to Architecture Mapping

| Epic                                | Architectural Home                                                                   | Notes                                                     |
| ----------------------------------- | ------------------------------------------------------------------------------------ | --------------------------------------------------------- |
| Sensor Network Foundation           | `hardware/firmware`, `hardware/pi-services`, `apps/api` ingestion module             | Edge firmware, buffering, and ingestion endpoints         |
| Cloud Ingestion & Data Reliability  | `apps/api` ingestion module, `libs/domain`, `libs/infrastructure`, `infra/terraform` | Validation, Timescale storage, retry orchestration        |
| Public Data Access & Partner APIs   | `apps/api` public module, `libs/domain`, generated SDK pipeline                      | REST queries, OpenAPI docs, partner enablement            |
| Community Dashboard Experience      | `apps/dashboard`, `libs/shared`, `infra/grafana`                                     | Map/timeline visuals, status overlays, observability link |
| Trust, Access Control & Compliance  | `apps/api` compliance module, `infra/grafana`, Terraform secrets                     | SB503 labeling, auth, legal disclaimers                   |
| Open Program & Workforce Enablement | `docs/playbooks`, `tools/scripts`, `hardware` documentation                          | Curriculum, provisioning guides, training artifacts       |
| Alerts & Notifications (FR-6)       | `apps/api` alerting module, `libs/infrastructure` queues, Grafana alert rules        | Threshold detection, notification dispatch                |

## Technology Stack Details

### Core Technologies

- Nx-managed monorepo (TypeScript) hosting NestJS API, Next.js dashboard, and shared libraries
- PostgreSQL 15 with TimescaleDB hypertables for time-series storage and geospatial filtering
- BullMQ + Redis queue for ingestion retries and alert evaluation jobs
- Grafana Cloud (Loki, Prometheus, Alerting) with OpenTelemetry SDK integration
- Terraform + flyctl for infrastructure automation, Resend for initial alert notifications

### Integration Points

- Edge buffering service posts HTTPS batches to `/v1/readings`, including QA metadata and API key headers
- Public API exposes endpoints for latest readings, regional aggregates, sensor status, and metadata lookups
- Dashboard fetches via generated TypeScript SDK plus SWR caching; Grafana dashboards surface ops telemetry
- Alerting pipeline consumes Timescale continuous aggregates, pushes notifications, and syncs severity to dashboard overlays
- Workforce enablement scripts interface with Auth0 and Fly APIs for account provisioning and deployment exercises

## Novel Pattern Designs

### Fenceline Transparency Pipeline

- **Purpose:** Maintain trust by attaching certification status, QA/QC metadata, and legal disclaimers to every reading while keeping data accessible.
- **Components:** Ingestion compliance filter (API), trust service (domain/infrastructure libs), dashboard annotation layer, and public API decorators.
- **Data Flow:** Sensor → compliance filter enriches payload → Timescale stores trust metadata → public API exposes fields → dashboard renders banners and severity legend.
- **Implementation Guide:** Enumerate certification states, require compliance metadata on DTOs, seed disclaimer content, regression-test trust label propagation.
- **Epic Coverage:** Trust & Compliance, Community Dashboard, Public Data Access.

## Implementation Patterns

- **Naming conventions:**
  - REST endpoints `/v1/<plural-resource>`; path params kebab-case; query params snake_case
  - Database schemas/tables snake_case, plural tables, foreign keys `<table>_id`
  - Frontend components PascalCase filenames grouped by feature directories
- **Structure:**
  - Tests colocated as `.spec.ts`; shared utilities in `libs/shared`, domain logic in `libs/domain`; API modules import via DI providers
  - Edge scripts and systemd units housed under `hardware/pi-services`
- **Format:**
  - API responses `{ data, meta }`; errors `{ error: { code, message, details? } }`
  - Exports deliver UTC timestamps (ISO-8601) with numeric ppm/ppb readings
- **Communication:**
  - BullMQ queues (`ingest-retry`, `alert-evaluator`) with payload schema in `libs/domain/queues`
  - Alerts start with email (Resend) and optional Slack webhook; templates in `libs/shared/notifications`
- **Lifecycle:**
  - Dashboard standardizes loading states via shared hooks; backend retries max five attempts with exponential backoff
  - Provisioning playbooks document sensor tracing from key issuance through first heartbeat
- **Location:**
  - API route modules in `apps/api/src/app/<module>`; DTOs in `libs/domain/dto`
  - Terraform modules under `infra/terraform/modules/*`; Grafana dashboards versioned alongside alert rules

## Consistency Rules

### Naming Conventions

- Endpoint URLs pluralized; enums and DTOs use PascalCase; config keys SCREAMING_SNAKE_CASE in environment files.
- Redis/BullMQ queue names prefixed with `louisianair-` to avoid collisions.

### Code Organization

- Enforce Nx dependency constraints: `apps` may depend on `libs/*`, while `libs/infrastructure` cannot import from `libs/domain`.
- Firmware and edge services maintain separate versioning to allow independent release cadence.

### Error Handling

- Global NestJS exception filter returns `{status, code, message, details?}` with correlation IDs.
- Dashboard displays severity-aligned toasts and fallback states for stale sensor data.

### Logging Strategy

- Use `pino` with OTel context to emit JSON logs enriched with trace IDs and sensor identifiers.
- Edge services write structured logs to local journal and forward summaries during successful uploads.

## Data Architecture

- Timescale hypertables partition readings by sensor and time, with continuous aggregates for hourly and daily rollups.
- Sensor metadata stored in relational tables with geospatial indices (PostGIS) for regional queries.
- Compliance tables track certification status, QA/QC audits, and disclaimer content versioning.
- Redis cache stores recent readings per sensor to support low-latency dashboard fetches and alert evaluation.

## API Contracts

- `/v1/readings` (POST) ingests up to 1,000 readings, validates timestamps, geofence, QA flags, idempotency keys.
- `/v1/readings/latest`, `/v1/readings/aggregate`, `/v1/sensors`, `/v1/alerts` expose public data with pagination and filter parameters.
- OpenAPI spec generated via Nest decorators, published under `/docs`, and drives TypeScript & Python SDK generation.
- Contract tests (Prism/Dredd) ensure API & SDK stay aligned; response metadata always includes trust fields.

## Security Architecture

- Device ingestion secured with hashed API keys (bcrypt/argon2) stored in Postgres; rotation handled by provisioning CLI.
- Human-facing endpoints will integrate Auth0 (JWT RS256) with role-based permissions for administrators vs public data consumers.
- TLS 1.2+ enforced end-to-end; Fly.io certificates auto-managed; sensitive secrets injected via Fly secrets + Terraform.
- Rate limiting per IP/key, anomaly detection for out-of-bounds readings, and signed audit logs for compliance changes.

## Performance Considerations

- Regional queries optimized through Timescale continuous aggregates and geospatial indices to meet <2s target.
- Dashboard prefetches using SWR with background revalidation; skeleton states ensure <3s first meaningful paint on 3G.
- Ingestion pipeline batches hourly with 48-hour edge buffer; backpressure via BullMQ prevents overload.
- Load testing planned with k6 scripts to validate 1,000 requests/hour sustained traffic and ingestion spikes.

## Deployment Architecture

- Fly.io app per runtime (`louisianair-api`, `louisianair-dashboard`), each scaled to dual regions for redundancy near Gulf Coast users.
- Managed Fly Postgres (primary + read replica) with Timescale extension enabled; backups retained 30 days.
- Terraform modules codify apps, machines, secrets, and Grafana integrations; CI triggers plan/apply for staging/production.
- Grafana Agent sidecar streams logs/metrics to Grafana Cloud; alert rules notify Slack/email on threshold breaches.

## Development Environment

### Prerequisites

- Node.js 20 LTS, pnpm 9+, Nx CLI (global optional), Docker, Terraform 1.7+, flyctl, Grafana Agent CLI, Python 3.11 for tooling scripts.
- Access to Grafana Cloud workspace, Auth0 tenant (when human auth added), and Resend account for email alerts.

### Setup Commands

```bash
npx create-nx-workspace@latest louisianair --preset=apps --style=css --nxCloud=false
cd louisianair
pnpm install
pnpm nx generate @nrwl/nest:application api
pnpm nx generate @nrwl/next:application dashboard
flyctl auth login
terraform init infra/terraform
```

## Architecture Decision Records (ADRs)

- ADR-001: Adopt Nx TypeScript monorepo to align apprentice workflows across API, dashboard, and tooling.
- ADR-002: Store time-series data in PostgreSQL with Timescale to balance performance and familiarity.
- ADR-003: Deliver REST APIs with OpenAPI-driven SDKs for community partners and automation agents.
- ADR-004: Secure device ingestion with hashed API keys, reserving HMAC/mTLS for future hardening.
- ADR-005: Deploy on Fly.io with Terraform automation to meet budget and reliability constraints.
- ADR-006: Standardize observability on Grafana Cloud with OpenTelemetry instrumentation.
- ADR-007 (Future Spike): Evaluate Go-based backend path post-MVP for broader apprentice exposure.

## Validation Summary

- Architecture Completeness: Mostly Complete
- Version Specificity: Many Missing (needs explicit version numbers and verification dates for Nx CLI, NestJS, Timescale, Grafana agent, Node.js, Redis, BullMQ, Terraform, flyctl, etc.)
- Pattern Clarity: Clear
- AI Agent Readiness: Mostly Ready (blocked on version details)

**Critical Issues**

1. Missing concrete version numbers and verification dates for every technology choice.
2. Starter command lacks explicit Nx version pin once verified.
3. Observability stack requires Grafana Agent configuration details and version alignment.

**Recommended Actions Before Implementation**

1. Run WebSearch or official release notes to capture current LTS versions for all core tools and update Decision Summary accordingly.
2. Document Grafana Agent deployment steps (sidecar vs standalone) with configuration snippet and version in observability section.
3. Confirm Fly.io-managed Timescale quotas and capture resource sizing assumptions in Deployment Architecture.

---

_Generated by BMAD Decision Architecture Workflow v1.0_  
_Date: 2025-11-05_  
_For: OpSpark_

<!-- TEMPLATE_OUTPUT: project_context_understanding -->

## Project Context Understanding

- Documentation reviewed: PRD (2025-11-05), epics/stories breakdown, UX spec scaffold
- Scale snapshot: 7 epics covering 33 implementation stories; Level 3 complex system with API + IoT + dashboard scope
- Core functional focus: DIY sensor kits streaming ppb gas readings, cloud ingestion & validation, public data APIs, community dashboards, alerting, compliance transparency, and workforce enablement content
- Critical non-functional drivers: sub-2s community queries, <3s dashboard loads on 3G, data freshness within 5 min, 99.5% API uptime, strict TLS + key security, 48h edge buffer with zero-loss retries
- Unique challenges flagged: field hardware resilience in Gulf Coast conditions, regulatory SB503 transparency requirements, community trust messaging, cost-capped cloud scaling to 50 sensors, workforce training integration
- UX spec status: template placeholders only—requires decisions before influencing architecture
<!-- /TEMPLATE_OUTPUT -->

<!-- TEMPLATE_OUTPUT: starter_template_decision -->

## Starter Template Decision

- Primary technology focus: unified TypeScript monorepo covering NestJS API, Next.js dashboard, and shared domain libs
- Selected starter: Nx workspace (TypeScript preset)
- Initialization command: `npx create-nx-workspace@latest louisianair --preset=apps --style=css --nxCloud=false` (confirm latest Nx release before running)
- Starter-provided decisions:
  - Language & tooling: TypeScript + Nx-managed workspace
  - Linting/formatting: ESLint + Prettier configs generated by Nx
  - Testing harness: Jest configured per app/lib with Nx runners
  - Project structure: apps/ (per runtime) + libs/ (shared modules) with enforced dependency graph
  - Build tooling: Nx task runner with cached `build`, `test`, and `lint` targets
- First implementation story: initialize repo by running the command above to scaffold the workspace
- Future consideration: capture appetite for an alternate Go-backed architecture in a follow-up spike once baseline is established
<!-- /TEMPLATE_OUTPUT -->

<!-- TEMPLATE_OUTPUT: decision_record -->

## Decision Record

- Category: Data Persistence
- Decision: PostgreSQL with TimescaleDB extension for time-series storage
- Verified Version: Pending (confirm latest Timescale/PG combo before provisioning)
- Affects Epics: Sensor Network Foundation, Cloud Ingestion & Data Reliability, Public Data Access & Partner APIs, Community Dashboard Experience
- Rationale: Keeps us in familiar Postgres ecosystem for apprentices, hypertables deliver efficient ingest + geospatial queries to meet sub-2s dashboard targets, Fly.io managed Postgres supports Timescale while staying within <$200/month budget
- Provided by Starter: No
<!-- /TEMPLATE_OUTPUT -->

<!-- TEMPLATE_OUTPUT: decision_record -->

## Decision Record

- Category: API Pattern
- Decision: REST-first API with OpenAPI contract and generated SDKs
- Verified Version: Pending (confirm latest NestJS + OpenAPI packages before implementation)
- Affects Epics: Cloud Ingestion & Data Reliability, Public Data Access & Partner APIs, Community Dashboard Experience, Trust & Compliance
- Rationale: Keeps ingestion endpoints simple for sensors, supports community clients with predictable resources, allows rate limiting per FR-4, and generated SDKs help apprentices build consumers without GraphQL overhead
- Provided by Starter: No
<!-- /TEMPLATE_OUTPUT -->

<!-- TEMPLATE_OUTPUT: decision_record -->

## Decision Record

- Category: Authentication
- Decision: Hashed per-device API keys with rotation workflow
- Verified Version: Pending (select NestJS auth modules + hashing library versions later)
- Affects Epics: Cloud Ingestion & Data Reliability, Trust & Compliance, Sensor Network Foundation
- Rationale: Meets FR-2.3 requirements with minimal edge complexity, aligns with workforce training goals, supports key rotation without reflashing devices, and keeps future HMAC/mTLS upgrades open
- Provided by Starter: No
<!-- /TEMPLATE_OUTPUT -->

<!-- TEMPLATE_OUTPUT: decision_record -->

## Decision Record

- Category: Deployment Target
- Decision: Fly.io machines + managed Postgres (Timescale-enabled) orchestrated via Terraform
- Verified Version: Pending (confirm Flyctl + Terraform provider versions before provisioning)
- Affects Epics: Cloud Ingestion & Data Reliability, Public Data Access & Partner APIs, Trust & Compliance, Deployment & Operations
- Rationale: Provides geographically close deployment, managed Postgres with Timescale support, container-based scaling within <$200/mo budget, and an approachable DX for apprentices while keeping IaC best practices through Terraform
- Provided by Starter: No
<!-- /TEMPLATE_OUTPUT -->

<!-- TEMPLATE_OUTPUT: decision_record -->

## Decision Record

- Category: Observability & Logging
- Decision: Grafana Cloud (Loki + Prometheus) with OpenTelemetry structured logging
- Verified Version: Pending (confirm Grafana agent versions and OTel SDK versions)
- Affects Epics: Cloud Ingestion & Data Reliability, Trust & Compliance, Operations & Workforce Enablement
- Rationale: Provides unified dashboards for metrics/logs, minimal ops overhead, aligns with time-series focus, and exposes apprentices to industry-standard observability practices while supporting alerting per NFR-R4
- Provided by Starter: No
<!-- /TEMPLATE_OUTPUT -->

<!-- TEMPLATE_OUTPUT: cross_cutting_decisions -->

## Cross-Cutting Decisions

- Error handling: NestJS global exception filter returning `{status, code, message, details?}` envelopes with correlation IDs
- Logging: Pino JSON logger wired via Nest, enriched with OpenTelemetry trace IDs and sensor identifiers
- Date & time: All services read/write UTC ISO-8601; dashboard localizes for display only
- Authentication/authorization: Device ingestion via hashed API keys; human-facing endpoints integrate with Auth0 when required
- API responses: Standard `{data, meta}` structure with pagination metadata; errors standardize to `{error}` envelope
- Testing approach: Jest unit tests per module, OpenAPI-driven contract tests in CI, Fly staging integration smoke tests pre-release
<!-- /TEMPLATE_OUTPUT -->

<!-- TEMPLATE_OUTPUT: project_structure -->

## Project Structure

```
louisianair/
├─ apps/
│  ├─ api/                     # NestJS ingestion + public REST API
│  ├─ dashboard/               # Next.js community dashboard (App Router)
│  └─ provisioning-cli/        # (Future) CLI for device key rotation & provisioning
├─ libs/
│  ├─ domain/                  # Domain models, value objects, Timescale projections
│  ├─ shared/                  # Cross-cutting utilities (logging adapters, config)
│  └─ infrastructure/          # DB access layer, Fly.io adapters, Grafana exporters
├─ hardware/
│  ├─ firmware/                # Arduino sketches, build scripts
│  └─ pi-services/             # Edge buffering service, systemd units, docs
├─ infra/
│  ├─ terraform/               # Fly.io + Postgres provisioning and secrets
│  └─ grafana/                 # Observability dashboards & alert definitions
├─ docs/
│  ├─ architecture.md          # Generated architecture document
│  ├─ PRD.md
│  ├─ epics.md
│  └─ playbooks/
└─ tools/
   └─ scripts/                 # Developer tooling (lint hooks, scaffolding)
```

<!-- /TEMPLATE_OUTPUT -->

<!-- TEMPLATE_OUTPUT: epic_mapping_table -->

| Epic                                | Architectural Home                                                                        | Notes                                                 |
| ----------------------------------- | ----------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| Sensor Network Foundation           | `hardware/firmware`, `hardware/pi-services`, `apps/api` ingestion module                  | Edge firmware, buffering, and ingestion endpoints     |
| Cloud Ingestion & Data Reliability  | `apps/api` ingestion module, `libs/domain`, `libs/infrastructure`, `infra/terraform`      | Handles ingest validation, Timescale storage, retries |
| Public Data Access & Partner APIs   | `apps/api` public module, `libs/domain`, generated SDK pipeline                           | REST queries, OpenAPI, partner tooling                |
| Community Dashboard Experience      | `apps/dashboard`, `libs/shared`, Grafana dashboards for internal ops                      | Map visualization, real-time status                   |
| Trust, Access Control & Compliance  | `apps/api` auth/compliance module, `infra/grafana`, Terraform secrets                     | Device auth, rate limiting, legal disclosures         |
| Open Program & Workforce Enablement | `docs/playbooks`, `tools/scripts`, `hardware` documentation                               | Curriculum, scripts for apprentices                   |
| Alerts & Notifications (FR-6)       | `apps/api` alerting module, background jobs in `libs/infrastructure`, Grafana alert rules | Threshold detection, notification plumbing            |

<!-- /TEMPLATE_OUTPUT -->

<!-- TEMPLATE_OUTPUT: novel_pattern_designs -->

## Novel Pattern Designs

### Pattern: Fenceline Transparency Pipeline

- Purpose: Ensure community-facing data clearly communicates sensor certification status, QA metadata, and legal disclaimers while keeping unvalidated readings available for advocacy.
- Components:
  - Ingestion Compliance Filter (apps/api) attaches QA/QC metadata, certification flags, and disclaimer codes to each reading.
  - Trust & Compliance Service (libs/domain + libs/infrastructure) enforces SB503 labeling, tracks sensor verification state, and drives alert severity.
  - Dashboard Annotation Layer (apps/dashboard) overlays disclaimers, status banners, and severity legends on map/timeline visualizations.
  - Public API Decorator maps certification flags into response metadata so partners can filter/label responsibly.
- Data Flow: Sensor readings → Ingestion Compliance Filter enriches payload → Timescale hypertables store trust metadata → API decorator exposes trust fields → Dashboard renders trust overlays.
- Implementation Guide: Define enums for certification + QA status, require compliance metadata per ingestion DTO, seed disclaimer content and severity color palette, include regression tests for trust label propagation.
- Affects Epics: Trust, Access Control & Compliance; Community Dashboard Experience; Public Data Access & Partner APIs.
- Validation: Dashboard specimens demonstrate legal disclaimer banner for uncertified sensors; API schema ensures trust metadata present on every response.
<!-- /TEMPLATE_OUTPUT -->

<!-- TEMPLATE_OUTPUT: implementation_patterns -->

## Implementation Patterns

- Naming conventions
  - REST endpoints: `/v1/<resource>` with plural nouns (`/v1/readings`), path params kebab-case (`:sensor-id`); query params snake_case
  - Database objects: schemas lowercase snake_case; tables plural (`sensor_readings`); columns snake_case; foreign keys `<table>_id`
  - Frontend components: PascalCase filenames (`SensorStatusCard.tsx`); directories by feature (`/features/alerts/`)
- Structure patterns
  - Tests colocated next to implementation with `.spec.ts` files (Nx generates)
  - Shared utilities live in `libs/shared`, domain logic in `libs/domain`; API modules import via explicit providers only
  - Edge services scripts under `hardware/pi-services`; systemd units versioned in same folder
- Format patterns
  - API responses: `{ data: {...}, meta: {...} }`; errors: `{ error: { code, message, details? } }`
  - Time-series exports use CSV with UTC timestamps; JSON responses include ISO strings and numeric ppm/ppb values
- Communication patterns
  - Background jobs use BullMQ queues (`ingest-retry`, `alert-evaluator`); job payload schema defined in `libs/domain/queues`
  - Alert notifications initially via email (Resend) with Slack webhook optional; message templates stored in `libs/shared/notifications`
- Lifecycle patterns
  - Loading states standardized via `useLoadingState` hook (dashboard); backend retries limited to 5 attempts with exponential backoff
  - Sensor provisioning workflow documented in playbooks with step-by-step tracing to API key issuance
- Location patterns
  - API route definitions inside `apps/api/src/app/<module>`; DTOs in `libs/domain/dto`
  - Terraform modules under `infra/terraform/modules/*`; Grafana dashboards JSON in `infra/grafana/dashboards`
  <!-- /TEMPLATE_OUTPUT -->

<!-- TEMPLATE_OUTPUT: coherence_validation -->

## Coherence Validation

- Decisions align: Nx/TypeScript monorepo cleanly supports Nest API + Next dashboard, Timescale-backed Postgres, Fly.io deployment, and Grafana observability without conflicting stacks.
- Coverage check: Every epic has mapped components; trust/compliance pattern threads through ingestion, API, and dashboard; alerts leverage BullMQ + Grafana integration.
- Remaining risks: Confirm Fly.io Timescale hosting quotas for sustained ingest; UX spec still placeholder—coordinate with design workflow before dashboard build; version verification outstanding for Nx, NestJS, Timescale, Grafana agent.
- Follow-up: Schedule spike on potential Go backend variant post-MVP; plan for HMAC upgrade path once device provisioning matures.
<!-- /TEMPLATE_OUTPUT -->
