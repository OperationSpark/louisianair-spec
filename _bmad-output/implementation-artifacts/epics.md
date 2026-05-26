# louisianair - Epic Breakdown

**Author:** OpSpark  
**Date:** 2025-11-05  
**Project Level:** Level 3 - Complex System (API + IoT + Dashboard)  
**Target Scale:** Pilot deployment: 5-10 community sensors with public API & dashboard

---

## Overview

This document provides the detailed epic breakdown for louisianair, expanding on the high-level epic list in the [PRD](./PRD.md).

Each epic includes:

- Expanded goal and value proposition
- Complete story breakdown with user stories
- Acceptance criteria for each story
- Story sequencing and dependencies

**Epic Sequencing Principles:**

- Epic 1 establishes foundational infrastructure and initial functionality
- Subsequent epics build progressively, each delivering significant end-to-end value
- Stories within epics are vertically sliced and sequentially ordered
- No forward dependencies - each story builds only on previous work

---

## Epic Overview

The PRD clusters naturally into the following epics. Foundational work (Epic 1) establishes the sensor-to-cloud pipeline before layering public access, user experiences, trust/compliance, and open program enablement.

1. **Sensor Network Foundation** – Stand up reproducible hardware kits, firmware, and Pi services so community-built monitors capture calibrated readings and survive field realities.
2. **Cloud Ingestion & Data Reliability** – Deliver the secure ingestion API, storage schema, and resilience features that keep data trustworthy and fresh for downstream use.
3. **Public Data Access & Partner APIs** – Expose actionable endpoints and developer tooling so residents, partners, and apprentices can query, analyze, and build on the dataset.
4. **Community Dashboard Experience** – Give fenceline residents an accessible, plain-language dashboard that visualizes live and historical trends across neighborhoods.
5. **Trust, Access Control & Compliance** – Enforce authentication, authorization, legal disclaimers, and transparency measures that uphold SB 503 requirements and data integrity.
6. **Open Program & Workforce Enablement** – Package curriculum, documentation, and contribution pathways that let students, apprentices, and external partners replicate and extend Louisianair.

---

## Epic 1 – Sensor Network Foundation

**Goal:** Enable community teams to assemble, configure, and field Raspberry Pi + Arduino sensor kits that capture methane and hydrogen sulfide readings reliably, buffer locally, and survive common failure scenarios.

**Value Proposition:** Puts real sensors in the field quickly, proving the hardware blueprint, creating trustworthy data sources, and unlocking all downstream cloud and visualization work.

### Stories

**Story E1.1: Publish Field-Ready Hardware Assembly Blueprint**

As an Operation Spark instructor,  
I want a step-by-step assembly guide for the Raspberry Pi + Arduino sensor kit,  
So that students can build a working monitor with minimal expert intervention.

**Acceptance Criteria:**

1. Guide lists exact bill of materials (Pi model, Arduino, gas sensors, power components) with sourcing links.
2. Includes annotated wiring diagrams for I²C and analog connections, plus photos of critical wiring steps.
3. Documents power requirements, enclosure considerations, and expected build time (~4 hours with support).

**Prerequisites:** None.

---

**Story E1.2: Flash Standardized Firmware Bundle**

As an apprentice responsible for hardware bring-up,  
I want a repeatable process to flash validated Arduino + Raspberry Pi firmware,  
So that every kit ships with consistent sensor acquisition and edge software.

**Acceptance Criteria:**

1. Provide scripts or commands for flashing the Arduino sketch and installing Pi services, with verification steps.
2. Firmware defaults to reading H₂S and methane sensors at a configurable interval (5 minutes by default).
3. Post-flash smoke test confirms serial link, sensor readiness, and logging of sample readings on the Pi.

**Prerequisites:** Story E1.1 completed.

---

**Story E1.3: Implement Edge Data Buffering & Retry Pool**

As an edge services maintainer,  
I want the Pi to queue readings locally with retry logic,  
So that temporary outages or API failures never lose sensor data.

**Acceptance Criteria:**

1. Service captures timestamp, GPS, gas concentrations, and sensor metadata, storing batches with a 48-hour retention window.
2. Failed uploads are retried with exponential backoff; successful retries purge local copies.
3. Supports configurable batch size (default hourly) and logs retry outcomes for observability.

**Prerequisites:** Story E1.2 completed.

---

**Story E1.4: Add Sensor Health Monitoring & Alerts**

As a community operations lead,  
I want the kit to surface sensor health and offline states,  
So that we can respond quickly when field units fail.

**Acceptance Criteria:**

1. Watchdog detects Arduino disconnects or stale readings (>2 hours) and records state transitions (operational, maintenance, error).
2. Health status is exposed via local log/API for ingestion (ties into dashboard alerts later).
3. Documentation includes troubleshooting flow for the most common failure modes (power, wiring, sensor drift).

**Prerequisites:** Story E1.3 completed.

---

**Story E1.5: Pilot Deployment Checklist & Field QA Procedure**

As a deployment coordinator,  
I want a pre-flight checklist and QA routine for installing a kit on-site,  
So that each community deployment starts with validated, trustworthy readings.

**Acceptance Criteria:**

1. Checklist covers physical mounting, power verification, GPS capture, and first-upload validation.
2. Defines acceptance thresholds (e.g., sensor tolerance within ±5% reference reading) and remediation actions.
3. Captures post-deployment handoff steps (documentation updates, contact info, maintenance cadence).

**Prerequisites:** Story E1.4 completed.

---

## Epic 2 – Cloud Ingestion & Data Reliability

**Goal:** Provide the secure ingestion pipeline, storage model, and operational safeguards that turn field sensor uploads into trustworthy, queryable datasets.

**Value Proposition:** Guarantees data integrity and availability so public APIs and dashboards can rely on fresh, compliant readings without manual babysitting.

### Stories

**Story E2.1: Define Reading Persistence Schema & Validation Rules**

As a backend engineer,  
I want a normalized storage schema and validation checklist for incoming readings,  
So that every sensor upload lands in a consistent, query-friendly structure.

**Acceptance Criteria:**

1. Document schema covering readings table, metadata tables, and indices for geo/time queries.
2. Validation rules (timestamp bounds, GPS region, sensor registration) captured and codified as automated checks.
3. Migration or infrastructure script prepared to apply schema in development/staging environments.

**Prerequisites:** Epic 1 stories completed (data expectations clarified).

---

**Story E2.2: Implement `/v1/readings` Batch Ingestion Endpoint**

As a device integration engineer,  
I want a secure batch ingestion endpoint with deduplication,  
So that sensors can retry uploads without creating double records.

**Acceptance Criteria:**

1. Endpoint accepts up to 1,000 readings per request, validates payload, and enforces schema from E2.1.
2. Deduplication prevents duplicate sensor_id + timestamp combinations; idempotency verified by integration test.
3. Returns actionable error responses (400/401) and success receipts consistent with PRD expectations.

**Prerequisites:** Story E2.1 completed.

---

**Story E2.3: Build Device Authentication & Key Rotation Service**

As a platform administrator,  
I want write-only API keys with rotation support,  
So that compromised devices can be revoked without disrupting the network.

**Acceptance Criteria:**

1. Provisioning flow issues unique ingestion keys hashed at rest; revocation prevents further uploads.
2. Rotation procedure documented and verified via test (old key rejected, new key accepted without firmware changes).
3. Unauthorized attempts logged with IP, sensor_id, and reason code for audit.

**Prerequisites:** Story E2.2 completed.

---

**Story E2.4: Attach Metadata Enrichment & SB 503 Disclaimers**

As a compliance steward,  
I want every stored reading annotated with calibration, certification, and legal disclaimers,  
So that downstream consumers always understand sensor quality and legal posture.

**Acceptance Criteria:**

1. Enrichment layer injects calibration date, certification status, and legal disclaimer strings onto each persisted record.
2. Automatically flags non-certified sensors with “community sensor, not enforcement” labeling.
3. Automated tests confirm metadata present for new and replayed uploads.

**Prerequisites:** Story E2.2 completed.

---

**Story E2.5: Instrument Ingestion Observability & Alerting**

As an ops engineer,  
I want metrics, logs, and alert thresholds for ingestion health,  
So that the team spots failures before communities notice missing data.

**Acceptance Criteria:**

1. Structured logs capture request IDs, validation failures, retry outcomes, and dedupe events.
2. Metrics surfaced for throughput, error rate, backlog size, and latency with alert thresholds (e.g., error rate >5%).
3. Runbook section added detailing responses to ingestion outage, backlog growth, or database failure.

**Prerequisites:** Stories E2.2 and E2.3 completed.

---

## Epic 3 – Public Data Access & Partner APIs

**Goal:** Deliver performant, well-documented endpoints and tooling that let residents, community partners, and apprentices access live and historical air quality data.

**Value Proposition:** Converts raw readings into actionable insights and enables external innovation, fulfilling the transparency promise that differentiates Louisianair.

### Stories

**Story E3.1: Implement `/v1/readings/latest` Endpoint**

As a community data consumer,  
I want a fast endpoint that returns the latest reading per sensor or zone,  
So that I can see current conditions without filtering raw data.

**Acceptance Criteria:**

1. Supports filters by sensor IDs, lat/lon radius, and named zones with <2s response for 50 sensors.
2. Includes plain-language status fields (normal/elevated/hazard) derived from thresholds.
3. Unit tests cover filter combinations and stale sensor handling (`stale: true`).

**Prerequisites:** Epic 2 completed.

---

**Story E3.2: Implement `/v1/readings/timeseries` Endpoint**

As an analyst or educator,  
I want access to historical readings with optional aggregation,  
So that I can study trends and build lesson plans from real data.

**Acceptance Criteria:**

1. Supports time window filtering (max 30 days), interval aggregation (15min/hour/day), and basic stats (min/max/avg).
2. Performance tested to deliver 7-day hourly averages for 10 sensors within 3 seconds.
3. Handles empty result sets gracefully and documents query parameter validation errors.

**Prerequisites:** Story E3.1 completed.

---

**Story E3.3: Implement `/v1/readings/alerts` Endpoint**

As a community advocate,  
I want to query threshold exceedances with severity context,  
So that I can escalate pollution events backed by data.

**Acceptance Criteria:**

1. Flags readings above EPA/local thresholds with severity labels and health guidance text.
2. Filters by geographies, date ranges, and severity levels; supports pagination for long windows.
3. Includes metadata references (sensor certification, disclaimer) from Epic 2 enrichment.

**Prerequisites:** Stories E3.1 and E3.2 completed.

---

**Story E3.4: Publish OpenAPI Schema & Interactive Explorer**

As a developer partner,  
I want accurate API documentation and an interactive explorer,  
So that I can integrate without reading service code or guessing parameters.

**Acceptance Criteria:**

1. OpenAPI 3.0 document generated and versioned alongside the service; includes all endpoints and error models.
2. Bundle Swagger UI (or equivalent) with environment-aware base URL for staging/production.
3. Documentation section in repo README links to explorer, auth instructions, and sample queries.

**Prerequisites:** Stories E3.1-E3.3 completed.

---

**Story E3.5: Provide Starter Projects & SDK Samples**

As an apprentice or community developer,  
I want ready-to-run examples for querying and visualizing data,  
So that I can build dashboards or analyses within hours, not days.

**Acceptance Criteria:**

1. Publish Node.js, Python, and Go samples demonstrating latest readings display, CSV export, and ingestion client.
2. Include instructions for configuring API keys, environment variables, and running locally.
3. Integrate code samples into documentation site/playbook with screenshots or output examples.

**Prerequisites:** Story E3.4 completed.

---

## Epic 4 – Community Dashboard Experience

**Goal:** Deliver an accessible, mobile-friendly dashboard that translates live readings into plain-language insights for residents and community partners.

**Value Proposition:** Empowers non-technical users to understand what’s in their air right now, compare trends, and advocate using trustworthy visuals.

### Stories

**Story E4.1: Build Map Overview with Real-Time Status**

As a fenceline resident,  
I want a city map that shows each sensor’s current status,  
So that I can instantly understand local air quality.

**Acceptance Criteria:**

1. Map renders sensor markers color-coded by status (green/yellow/red/gray) with tooltip showing latest values and “last updated” timestamp.
2. Mobile layout ensures map loads under 3 seconds on 3G with accessible tap targets.
3. Integrates stale sensor indicator linked to Epic 2 metadata (`stale: true`).

**Prerequisites:** Epic 3 endpoints available.

---

**Story E4.2: Create Sensor Detail View with Trend Charts**

As a community organizer,  
I want detailed charts for a specific sensor,  
So that I can understand changes over the past day and week.

**Acceptance Criteria:**

1. Displays 24-hour and 7-day charts with EPA threshold shading and plain-language interpretations.
2. Includes educational sidebar explaining gas types, health implications, and disclaimers.
3. Supports screen-reader accessible descriptions and keyboard navigation.

**Prerequisites:** Story E4.1 completed.

---

**Story E4.3: Implement Historical Comparison Workspace**

As an educator or analyst,  
I want to compare multiple sensors or zones over time,  
So that I can highlight disparities and trends in class or advocacy presentations.

**Acceptance Criteria:**

1. Users select sensors/zones and date ranges (up to 30 days) to overlay charts with legends and export options (CSV/PNG).
2. Performance optimized to complete typical comparison queries within 3 seconds.
3. Provides annotation hooks for community-added notes (e.g., field events).

**Prerequisites:** Story E4.2 completed.

---

**Story E4.4: Ship Neighborhood Zone Comparison Cards**

As a resident evaluating my neighborhood,  
I want quick comparisons of my zone against others,  
So that I can contextualize readings without plotting graphs.

**Acceptance Criteria:**

1. Cards summarize average levels, days above thresholds, and variance across predefined neighborhoods.
2. Highlights significant deltas with plain-language callouts and recommended actions.
3. Respects accessibility guidelines (contrast, ARIA labels) and responsive layouts.

**Prerequisites:** Story E4.3 completed.

---

**Story E4.5: Harden Performance, Accessibility, and Low-Bandwidth Experience**

- As a UX quality lead,  
  I want guarantees that the dashboard runs fast and inclusively,  
  So that no community member is excluded due to device, bandwidth, or assistive technology.

  **Acceptance Criteria:**

1. Automated accessibility tests (axe/pa11y) integrated into CI; manual screen-reader audit documented.
2. Lighthouse or WebPageTest results show <500KB initial load and <3s first paint on 3G.
3. Adds offline-friendly fallback messaging for map tiles and ensures all content remains usable if tiles fail.

**Prerequisites:** Stories E4.1-E4.4 completed.

---

## Epic 5 – Trust, Access Control & Compliance

**Goal:** Bake in authentication, authorization, rate limiting, auditability, and legal disclaimers so the platform remains trustworthy, SB 503 compliant, and resilient to abuse.

**Value Proposition:** Maintains community and partner trust by guarding data integrity, clarifying sensor limitations, and preventing misuse or abuse of the open API.

### Stories

**Story E5.1: Implement Public & Partner Rate Limiting**

As a platform reliability engineer,  
I want tiered rate limits across public, partner, and device roles,  
So that the API stays responsive even under bursty traffic.

**Acceptance Criteria:**

1. Token bucket or similar limiter enforces 100 req/hour for public users and 1,000 req/hour for partner keys.
2. 429 responses include Retry-After headers and are logged with key scope information.
3. Rate limiting configuration documented for ops escalation and future tuning.

**Prerequisites:** Epic 3 endpoints live.

---

**Story E5.2: Deliver Partner/Admin Authentication Flows**

As an administrator,  
I want secure OAuth2 access for admins and scoped keys for partners,  
So that sensitive operations stay protected without blocking collaboration.

**Acceptance Criteria:**

1. OAuth2 flow issues JWTs with 1-hour expiry and refresh tokens for admins; includes sensor registry management UI/API.
2. Partner keys generated with read-only scope and higher rate limits; key management dashboard or CLI documented.
3. Authentication failures audited with user/key identifiers and reason codes.

**Prerequisites:** Story E5.1 completed.

---

**Story E5.3: Surface Legal Disclaimers & Transparency in API and UI**

As a compliance officer,  
I want every touchpoint to communicate sensor certification status and legal usage constraints,  
So that SB 503 requirements are met and community trust is reinforced.

**Acceptance Criteria:**

1. API responses include certification status fields and standard disclaimer text for non-certified sensors.
2. Dashboard displays banner or inline messaging clarifying “Community sensor data for awareness, not enforcement.”
3. Documentation/playbook section added explaining legal context and proper data usage.

**Prerequisites:** Epic 4 UI and Epic 3 APIs available.

---

**Story E5.4: Implement Audit Logging & Tamper Detection**

As a security steward,  
I want immutable logs of key administrative and ingestion actions,  
So that we can trace issues and detect malicious tampering.

**Acceptance Criteria:**

1. Audit log records sensor registry changes, key provisioning, revocations, and suspicious ingestion attempts.
2. Logs stored in append-only medium with retention aligned to compliance guidance (≥90 days).
3. Alerts configured for unusual patterns (e.g., repeated failed logins, ingestion from unregistered sensors).

**Prerequisites:** Stories E5.1 and E5.2 completed.

---

**Story E5.5: Document Security & Compliance Runbooks**

As an operations lead,  
I want clear runbooks for security incidents and compliance reviews,  
So that the team responds consistently without institutional memory loss.

**Acceptance Criteria:**

1. Runbooks cover rate-limit exceedance handling, key revocation, data breach response, and compliance audit preparation.
2. Includes contact tree and escalation paths for community partners affected by outages or security incidents.
3. Linked from repository docs and shared with leadership/stakeholders for review.

**Prerequisites:** Stories E5.3 and E5.4 completed.

---

## Epic 6 – Open Program & Workforce Enablement

**Goal:** Package curriculum, documentation, and contribution workflows that empower students, apprentices, and external partners to replicate and evolve Louisianair.

**Value Proposition:** Extends impact beyond the pilot by building community capacity, supporting workforce development, and ensuring knowledge persists across cohorts.

### Stories

**Story E6.1: Finalize IoT Curriculum & Lab Materials**

As an Operation Spark instructor,  
I want a complete lesson plan and lab bundle for the IoT class,  
So that students can build and deploy 5-10 sensors during the session.

**Acceptance Criteria:**

1. Curriculum includes learning objectives, lesson outlines, lab exercises, and evaluation rubrics.
2. Materials reference Epic 1 guides and integrate troubleshooting checkpoints.
3. Instructor prep guide covers required equipment, timing, and common pitfalls observed during pilot.

**Prerequisites:** Epic 1 completed.

---

**Story E6.2: Publish Hardware & Deployment Playbook**

As an external partner,  
I want a repeatable playbook for assembling and deploying Louisianair sensors,  
So that I can replicate the network without live support from Operation Spark.

**Acceptance Criteria:**

1. Playbook consolidates hardware blueprint, flashing instructions, deployment checklist, and maintenance routines.
2. Includes photos/video walkthroughs where helpful plus cost estimates for a 5-10 sensor rollout.
3. Released under appropriate open hardware/documentation licenses (CERN OHL / CC BY).

**Prerequisites:** Story E6.1 completed.

---

**Story E6.3: Release Developer Starter Projects & Contribution Guide**

As an apprentice contributor,  
I want clear instructions for contributing code and using starter projects,  
So that I can make production-ready commits confidently.

**Acceptance Criteria:**

1. Contribution guide covers repo structure, coding standards, testing expectations, and PR review process.
2. Starter projects from Epic 3 packaged with setup scripts and linked in README/playbook.
3. Adds issue templates and project board columns for tracking story-level work.

**Prerequisites:** Story E3.5 completed.

---

**Story E6.4: Document Workforce Outcomes & Portfolio Artifacts**

As a workforce development lead,  
I want templates and examples for apprentices to showcase their contributions,  
So that participants capture portfolio-ready artifacts and career narratives.

**Acceptance Criteria:**

1. Provides template for project summaries, code highlights, and reflection prompts aligned with program goals.
2. Curates examples (screenshots, commit links) demonstrating end-to-end involvement (hardware, API, dashboard).
3. Shares guidance on presenting outcomes to employers/partners, tying back to success metrics in PRD.

**Prerequisites:** Stories E6.1-E6.3 completed.

---

**Story E6.5: Establish Community Support & Feedback Loop**

As a program manager,  
I want lightweight channels for community partners to report issues and request features,  
So that we keep the product aligned with real-world needs.

**Acceptance Criteria:**

1. Sets up issue intake form or forum, triage workflow, and response SLAs for partner requests.
2. Quarterly review cadence defined to feed requests back into product planning (link to workflow-status updates).
3. Summaries shared with stakeholders, highlighting resolved issues and upcoming priorities.

**Prerequisites:** Stories E6.2 and E6.3 completed.

---

## Story Guidelines Reference

**Story Format:**

```
**Story [EPIC.N]: [Story Title]**

As a [user type],
I want [goal/desire],
So that [benefit/value].

**Acceptance Criteria:**
1. [Specific testable criterion]
2. [Another specific criterion]
3. [etc.]

**Prerequisites:** [Dependencies on previous stories, if any]
```

**Story Requirements:**

- **Vertical slices** - Complete, testable functionality delivery
- **Sequential ordering** - Logical progression within epic
- **No forward dependencies** - Only depend on previous work
- **AI-agent sized** - Completable in 2-4 hour focused session
- **Value-focused** - Integrate technical enablers into value-delivering stories

---

**For implementation:** Use the `create-story` workflow to generate individual story implementation plans from this epic breakdown.

---

## Implementation Guidance

**Getting Started**

- Begin with Epic 1: Stories E1.1–E1.3 can run concurrently once the assembly blueprint draft exists.
- Prepare development environments by applying the ingestion schema (E2.1) and bootstrapping the API service with auth middleware.
- Capture early deliverables: validated hardware kit, ingestion service skeleton, and documentation repository sections for hardware and API.

**Parallelization & Phasing**

- Phase 1 (Weeks 1-2): Complete Epic 1, start ingestion core (E2.1/E2.2), and finish IoT curriculum (E6.1) so students can build prototypes in class.
- Phase 2 (Weeks 3-4): Build public endpoints (E3.1–E3.3) while dashboard team implements E4.1–E4.2; initiate trust work with E5.1.
- Phase 3 (Weeks 5-6): Harden dashboard and compliance (E4.3–E4.5, E5.2–E5.5) and ship replication playbooks, starter projects, and workforce enablement artifacts (E6.2–E6.5).

**Domain & Compliance Checkpoints**

- After E2.4: Review SB 503 disclaimers and metadata enrichment with legal advisors before beta release.
- Before E5.3: Validate UI messaging with community partners to ensure trust and transparency.
- During Epics 1 & 6: Feed field learnings into curriculum and playbook updates continuously.

**Technical Notes**

- Decide on time-series database (TimescaleDB vs InfluxDB) and hosting platform before deploying E2.2.
- Reuse logging and metrics middleware across ingestion and public APIs to streamline observability (E2.5).
- Feature-flag optional alerting (FR-6) so experimentation doesn’t block MVP readiness.

**Risk Mitigation**

- Hardware variability: log calibration outcomes during E1.5 to refine validation tolerances for ingestion.
- API abuse: deploy rate limiting (E5.1) and auth (E5.2) before promoting public beta endpoints.
- Dashboard performance: enforce bundle size budgets and run lighthouse/axe checks during Epic 4.

**Success Metrics by Phase**

- Phase 1: Sensor kit streams to staging ingestion API for 24 hours without data loss.
- Phase 2: Public endpoints and dashboard map/trend views live with real data.
- Phase 3: Compliance messaging shipped, curriculum published, and partner replication checklist validated.

---
