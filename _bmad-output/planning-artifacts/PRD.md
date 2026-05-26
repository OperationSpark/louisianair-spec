# louisianair - Product Requirements Document

**Author:** OpSpark
**Date:** 2025-11-05
**Version:** 1.0

---

## Executive Summary

Louisianair empowers fenceline communities to assemble low-cost air monitors, stream trusted readings into a shared open database, and interpret local air trends through accessible dashboards while apprentices and youth build real-world IoT and software skills that strengthen environmental justice capacity.

This open-source initiative combines DIY sensor hardware (Raspberry Pi + Arduino + industrial gas sensors), a cloud-backed data API, and community-friendly dashboards to make air quality monitoring accessible to New Orleans neighborhoods most impacted by pollution. By pairing technical infrastructure with workforce development, Louisianair transforms environmental monitoring into a platform for civic action and youth opportunity.

### What Makes This Special

**"Democratizing air quality insights for environmental justice through open data, accessible technology, and youth-powered community capacity building."**

The magic happens when a fenceline resident checks the dashboard and understands what's in their air _right now_, when a student builds their first sensor and sees real data flowing to the cloud, and when community organizers use trustworthy local data to advocate for their neighborhoods - all while apprentices gain career-ready skills strengthening environmental justice infrastructure.

This isn't just another air quality API - it's a community empowerment platform that transforms pollution monitoring into workforce development and civic action.

---

## Project Classification

**Technical Type:** API/Backend + IoT System

**Domain:** Environmental/Civic Technology (Compliance-Aware)

**Complexity:** Level 3 - Complex System

**Primary Components:**

- RESTful API for air quality data ingestion and public consumption
- IoT edge nodes (Raspberry Pi + Arduino + ppb-sensitive gas sensors)
- Web dashboard for community visualization and exploration
- Educational playbooks and starter projects for replication

### Domain Context

While Louisianair is not seeking EPA certification for the MVP pilot, the system is designed with awareness of Louisiana's regulatory landscape:

- **Louisiana Community Air Monitoring Reliability Act (SB 503):** Restricts enforcement to EPA-certified methods, but communities still need affordable tools for monitoring, education, and organizing around local conditions.
- **Compliance-Ready Design:** Data outputs include sensor certification status, QA/QC metadata, and legal usage disclaimers to support transparency requirements.
- **Non-Enforcement Framing:** MVP explicitly marks datasets as "non-enforcement" when sensors are unapproved, with architecture that could accommodate certified instrumentation in future phases.
- **EPA Threshold Alerting:** System flags readings exceeding EPA or local exposure thresholds with severity labels for educational and advocacy purposes.

The domain requires thoughtful handling of data trust, sensor transparency, and community communication - but not the intensive validation or certification pathways of medical or financial software.

---

## Success Criteria

Louisianair's pilot succeeds when it simultaneously empowers community partners with actionable data, produces reusable open tooling, and delivers measurable workforce development outcomes.

### Community Empowerment Success

**Core Measure:** Fenceline residents and community organizations have reliable access to local air quality insights they trust and can act on.

**Specific Indicators:**

- **5-10 sensor deployments** operational in target New Orleans fenceline neighborhoods, streaming continuous methane and hydrogen sulfide readings
- **Community partners (LEAN, local organizations) actively query the public API** for real-time and historical data in their advocacy and education work
- **Accessible dashboard** enables non-technical residents to understand air quality trends without specialized training
- **Data transparency achieved:** All readings include sensor metadata, QA/QC status, and usage disclaimers that build trust in unapproved monitoring equipment

### Workforce Development Success

**Core Measure:** Operation Spark apprentices and students gain production-ready technical skills through meaningful contributions to deployed infrastructure.

**Specific Indicators:**

- **15-25 students/apprentices** actively participate across IoT assembly, firmware development, API implementation, and dashboard creation
- **Apprentices ship production code** - meaningful commits to the API, data pipeline, or dashboard that serve real community users
- **Students complete end-to-end builds** - assemble sensor kits, deploy units, validate data flow, and troubleshoot field issues
- **Portfolio outcomes:** Participants can demonstrate IoT, cloud API, and open-source collaboration experience to future employers or educational programs

### Open Source Replication Success

**Core Measure:** External partners can independently replicate or extend the Louisianair network without Operation Spark involvement.

**Specific Indicators:**

- **Repeatable playbook published** documenting hardware assembly, sensor calibration, deployment procedures, and operational maintenance
- **Starter projects** (Node.js/Python) demonstrate how to consume the API, build dashboards, or contribute new sensor nodes
- **Public API promoted** with clear documentation, query examples, and transparent data schemas
- **Open infrastructure:** All code, hardware specs, and deployment guides available under permissive licenses (MIT/Apache)

### Technical Reliability Success

**Core Measure:** The system "just works" for community users - data flows reliably, sensors stay online, and queries respond quickly.

**Specific Indicators:**

- **Data integrity:** Sensors→Pi→Cloud pipeline maintains accurate timestamps, GPS coordinates, and concentration values without systematic corruption
- **Uptime:** ≥99.5% availability for API queries with graceful degradation during sensor outages (stale data flagged clearly)
- **Query performance:** Regional requests covering up to 50 sites return within 2 seconds
- **30-day historical access:** All deployed sensors expose continuous time-series data for trend analysis and validation

---

## Product Scope

### MVP - Minimum Viable Product (Pilot Phase: Feb-May 2026)

**Core Value:** Prove the end-to-end sensing→cloud→dashboard→education loop with real deployments that deliver actionable air quality insights to fenceline communities.

**What Must Work:**

**Hardware & Edge:**

- DIY sensor kit blueprint using Raspberry Pi (4/5), Arduino Nano Every, and gas sensors with ppb sensitivity (H₂S, methane) that interface with microcontrollers
- Edge processing: Arduino reads sensor data over I²C, forwards to Pi via USB serial
- Local buffering: Raspberry Pi appends readings to CSV/SQLite with timestamp, GPS, ppm values
- Batch upload: Hourly HTTPS POST to cloud API with local retry queue for failed uploads

**Cloud API & Data:**

- RESTful JSON API with authenticated access (role-based scopes for teams/partners)
- Endpoints: ingest sensor readings, query latest values, fetch 30-day historical time-series, geo-scoped searches (radius or named zones)
- Data normalization: consistent units (ppb), sampling methodology metadata, calibration timestamps
- Alert flagging: readings exceeding EPA/local thresholds marked with severity labels
- Sensor transparency: every payload includes certification status, QA/QC metadata, legal disclaimers per SB 503

**Dashboard & Visualization:**

- Web UI for community access: map view of sensor locations, real-time readings, time-series charts
- Plain-language explanations: "What does 15 ppb of H₂S mean for my health?"
- Accessible design: mobile-responsive, screen reader compatible, low bandwidth-tolerant
- Optional alerting concepts: email/SMS notifications for threshold exceedances (experimental)

**Documentation & Replication:**

- Repeatable playbook: hardware assembly, sensor calibration, deployment procedures, troubleshooting
- Starter projects: Node.js/Python code examples for API consumption, dashboard extensions, new sensor node setup
- Open licensing: Code (MIT/Apache), hardware specs (CERN OHL), documentation (CC-BY)

**Workforce Integration:**

- IoT class curriculum: students build 5-10 units during Feb-Mar 2026 session
- Apprentice contributions: API development, data pipeline engineering, dashboard implementation
- Documentation outputs: student-created assembly guides, troubleshooting wikis, field deployment reports

---

**Explicitly OUT of MVP Scope:**

- ❌ EPA-certified instrumentation or accredited lab analysis
- ❌ Manufacturing or distributing turnkey hardware kits beyond pilot cohort
- ❌ Compliance-grade enforcement tooling or legal advocacy workflows
- ❌ Real-time streaming (MQTT/WebSocket) - batch uploads sufficient for MVP
- ❌ Advanced analytics: multi-pollutant correlations, predictive models, health impact scoring
- ❌ Multi-sensor hub orchestration with automated calibration drift detection
- ❌ Weatherproof enclosure design (defer until field experience gathered)

### Growth Features (Post-MVP: Summer/Fall 2026)

**After proving the pilot works, expand capabilities and reach:**

**Network Expansion:**

- Scale beyond New Orleans through LEAN's Community Air Monitoring Network partnerships
- Add 20-50 additional sensor nodes across Louisiana EJ communities
- Support for additional gas sensors: SO₂, NO₂, O₃ (co-pollutants)

**Advanced Analytics:**

- Multi-pollutant correlation analysis and trend visualization
- Comparative dashboards: "How does my neighborhood compare to citywide averages?"
- Health context layering: overlay air quality data with asthma rates, hospital visits (if accessible)

**Enhanced Reliability:**

- Migrate from CSV to lightweight time-series database (InfluxDB, TimescaleDB)
- Dual watchdogs for sensor/Pi health monitoring
- Automated calibration drift detection and alerting for maintenance needs

**Community Engagement:**

- Hackathons and workshops: extend tooling, build community apps, onboard new contributors
- Partner integrations: export data to external research platforms, policy dashboards
- Mobile app: iOS/Android access for on-the-go monitoring

**Developer Experience:**

- GraphQL endpoint alongside REST for flexible querying
- SDK packages: JavaScript, Python, Go clients for the API
- Webhook subscriptions: real-time notifications when readings exceed thresholds

### Vision Features (Future: 2027+)

**Long-term aspirations that require sustained funding and partnerships:**

**Certified Data Pathways:**

- Collaborate with Louisiana DEQ and EPA to explore pathways for certified data collection without losing accessibility
- Hybrid model: community sensors for awareness + certified stations for enforcement

**AI-Powered Insights:**

- Predictive air quality forecasting: "Next 24 hours likely elevated H₂S based on wind patterns"
- Natural language querying: "Show me all readings above dangerous levels last week in the Lower Ninth Ward"
- Anomaly detection: automated alerts for unusual sensor patterns that warrant investigation

**Policy & Advocacy Tools:**

- Storytelling dashboards tailored for policymakers, healthcare advocates, journalists
- Regulatory reporting packages: automated EPA/DEQ format exports (if legal frameworks evolve)
- Public health integration: API connections to state health departments for epidemiological research

**Global Replication:**

- International deployment guides: adapt for different regulatory contexts (EU, Latin America, Asia)
- Translated interfaces and documentation for non-English speaking communities
- Franchise model: city/region-specific Louisianair instances with shared codebase

**Hardware Innovation:**

- Custom PCB sensor hats for simplified Raspberry Pi integration
- Solar-powered outdoor deployments with cellular connectivity
- Mesh networking: sensors relay data through peer nodes in areas with poor internet

---

## API/Backend Specific Requirements

### API Architecture

- **Style:** RESTful JSON API
- **Versioning:** URI-based versioning (e.g., `/v1/...`) with backwards compatibility guarantees within major versions
- **Protocol:** HTTPS only, HTTP requests automatically upgraded
- **Future Expansion:** Architecture must support adding GraphQL or gRPC endpoints post-MVP without breaking REST clients

### Core API Endpoints

#### **Sensor Data Ingestion**

**POST `/v1/readings`**

- **Purpose:** Accept batch uploads of sensor readings from edge devices
- **Authentication:** Device-specific API keys with write-only scope
- **Request Body:**

```json
{
  "sensor_id": "string (unique device identifier)",
  "batch": [
    {
      "timestamp": "ISO 8601 UTC",
      "location": { "lat": "number", "lon": "number" },
      "readings": {
        "h2s_ppb": "number",
        "ch4_ppb": "number",
        "so2_ppb": "number (optional)",
        "no2_ppb": "number (optional)",
        "o3_ppb": "number (optional)"
      },
      "metadata": {
        "calibration_date": "ISO 8601",
        "sensor_status": "operational|calibrating|maintenance",
        "qa_qc_flags": ["string array"]
      }
    }
  ]
}
```

- **Response:** `201 Created` with receipt confirmation, `400 Bad Request` for invalid data, `429 Too Many Requests` if rate limit exceeded
- **Idempotency:** Duplicate timestamps from same sensor_id are deduplicated

#### **Real-Time & Historical Queries**

**GET `/v1/readings/latest`**

- **Purpose:** Fetch most recent readings from specified sensors or geographic area
- **Authentication:** Read-scoped API key or OAuth token
- **Query Parameters:**
  - `sensor_ids` (comma-separated) OR `location` (lat,lon,radius_km) OR `zone` (named area like "Lower_Ninth_Ward")
  - `gases` (optional filter: h2s,ch4,so2)
  - `include_metadata` (boolean, default true)
- **Response:** Array of latest readings with sensor metadata

**GET `/v1/readings/timeseries`**

- **Purpose:** Retrieve historical data for trend analysis
- **Query Parameters:**
  - `sensor_ids` OR `location` OR `zone` (required)
  - `start_time`, `end_time` (ISO 8601, max 30-day window for MVP)
  - `interval` (optional aggregation: raw, 15min, hourly, daily)
  - `gases` (optional filter)
- **Response:** Time-series array with optional statistical aggregations (min/max/avg)

**GET `/v1/readings/alerts`**

- **Purpose:** Query readings that exceeded EPA/local thresholds
- **Query Parameters:**
  - `location` OR `zone` (required)
  - `start_time`, `end_time`
  - `severity` (warning, hazardous)
- **Response:** Array of flagged readings with threshold context and health guidance

#### **Sensor & Network Metadata**

**GET `/v1/sensors`**

- **Purpose:** List all active sensors with location and status
- **Authentication:** Public endpoint (no auth required)
- **Query Parameters:** `status` (operational, offline, maintenance), `zone`
- **Response:** Sensor registry with certification status, deployment date, last seen timestamp

**GET `/v1/sensors/{sensor_id}`**

- **Purpose:** Detailed information about specific sensor
- **Response:** Full sensor profile including hardware specs, calibration history, QA/QC status, Louisiana DEQ/EPA approval status

**GET `/v1/zones`**

- **Purpose:** List predefined geographic zones (e.g., neighborhoods, industrial corridors)
- **Response:** GeoJSON boundaries for named zones

### Authentication & Authorization

**Model:** Token-based authentication with role-based access control (RBAC)

**Roles:**

- **Public Consumer:** Read-only access to anonymized aggregate data (no auth required for limited endpoints)
- **Partner Organization:** Read access to full sensor network data (OAuth2 or API key)
- **Device/Sensor:** Write-only access to ingest readings (device-specific API key)
- **Admin/Maintainer:** Full read/write access including sensor management (OAuth2 with elevated scope)

**Implementation:**

- API keys for programmatic access (sensors, partner scripts)
- OAuth2 for interactive applications (dashboards, mobile apps)
- JWT tokens with expiration and refresh capability
- Rate limits per role: Public (100 req/hour), Partner (1000 req/hour), Device (unlimited ingest)

### Data Schemas & Validation

**Core Reading Schema:**

```typescript
interface Reading {
  id: string; // UUID
  sensor_id: string; // Device identifier
  timestamp: DateTime; // UTC, ISO 8601
  location: {
    lat: number; // Decimal degrees
    lon: number;
    altitude_m?: number;
  };
  readings: {
    [gas: string]: number; // ppb concentration
  };
  metadata: {
    calibration_date: DateTime;
    sensor_certification: "none" | "louisiana_deq" | "epa_approved";
    qa_qc_flags: string[]; // e.g., ["drift_detected", "calibration_overdue"]
    legal_disclaimer: string; // SB 503 compliance text
  };
  processed?: {
    rolling_avg_1h?: number;
    rolling_avg_24h?: number;
  };
}
```

**Validation Rules:**

- Timestamps must be within 24 hours of upload time (reject stale data)
- GPS coordinates must fall within configured deployment region (Louisiana for MVP)
- Gas concentration values must be non-negative and within sensor range (0-1000 ppb typical)
- Sensor_id must be pre-registered in system

### Error Codes & Handling

**Standard HTTP Status Codes:**

- `200 OK` - Successful query
- `201 Created` - Reading batch accepted
- `400 Bad Request` - Invalid parameters or malformed data (include detailed error field)
- `401 Unauthorized` - Missing or invalid authentication
- `403 Forbidden` - Valid auth but insufficient permissions
- `404 Not Found` - Sensor or resource doesn't exist
- `429 Too Many Requests` - Rate limit exceeded (include Retry-After header)
- `500 Internal Server Error` - Server-side failure (logged for investigation)
- `503 Service Unavailable` - Maintenance mode or degraded service

**Error Response Format:**

```json
{
  "error": {
    "code": "invalid_timerange",
    "message": "Time window exceeds 30-day maximum for this endpoint",
    "details": {
      "requested_days": 45,
      "max_days": 30
    }
  }
}
```

### Rate Limiting & Quotas

**MVP Approach:** Simple token bucket per API key

- Public endpoints: 100 requests/hour
- Partner keys: 1000 requests/hour
- Sensor ingestion: No rate limit (but batch size max 1000 readings)
- Burst allowance: 2x normal rate for 1 minute

**Headers:**

- `X-RateLimit-Limit`: Total quota
- `X-RateLimit-Remaining`: Remaining in current window
- `X-RateLimit-Reset`: Unix timestamp when quota resets

### API Documentation & Developer Experience

**OpenAPI Specification:**

- Maintain OpenAPI 3.0 schema for all endpoints
- Auto-generate API reference docs from schema
- Interactive API explorer (Swagger UI or similar)

**Starter Code Examples:**

- Node.js/JavaScript: Fetch readings, display on map
- Python: Data analysis notebook, fetch historical data, export to CSV
- Go: Sensor upload client with retry logic

**HTTP Requests Collection:**

- Pre-configured requests for all endpoints (shareable without cloud dependency)
- Environment variables for API keys and base URL
- Export for other API clients (Bruno, Postman, Insomnia) if needed

### Technology Stack Decisions (From Technical Preferences)

**API Framework:** Go (preferred) or Node.js

- Go: Strong typing, performance, straightforward testing, low resource footprint
- Consideration: Node.js if entry-level developer onboarding outweighs Go benefits

**Hosting:** Lightweight managed (Fly.io preferred), AWS/GCP acceptable

- Priority: Low maintenance operations, observability, deployment simplicity

**Database:** To be determined during architecture phase

- Options: Managed PostgreSQL with time-series extensions (TimescaleDB), InfluxDB, or hybrid approach
- Requirements: 5-10 sensors × hourly readings, 30-day retention minimum, support for geo-queries

**Observability:**

- System health endpoint: `GET /health` (readiness, liveness checks)
- Structured logging: JSON logs with trace IDs for request correlation
- Metrics: API latency, error rates, ingestion throughput, database query performance

---

## Dashboard User Experience Principles

### Visual Personality & Feel

**Core Vibe:** Trustworthy, accessible, community-focused

- **NOT:** Corporate dashboard with jargon and complex charts
- **BUT:** Neighborhood tool that speaks plainly and invites exploration

**Design Language:**

- Clean, uncluttered layouts that work on mobile phones and library computers
- High-contrast colors for readability in various lighting conditions
- Friendly but serious tone - air quality matters, but the tool shouldn't intimidate
- Visual hierarchy that prioritizes "What's happening now?" over technical details

**Accessibility First:**

- WCAG 2.1 AA compliance minimum (AAA for text contrast)
- Screen reader compatible with semantic HTML and ARIA labels
- Keyboard navigable for users who can't use a mouse
- Low bandwidth tolerant: <500KB initial page load, progressive enhancement for richer features
- MVP: English only (Spanish, Vietnamese support deferred post-MVP; users can rely on browser translation)

### Key Interactions & User Flows

#### **Primary Flow: "What's my air quality right now?"**

1. User opens dashboard (no login required for public view)
2. Map shows New Orleans with sensor markers colored by current status:
   - Green: All readings normal
   - Yellow: One or more gases approaching concern levels
   - Red: Readings exceed EPA thresholds
   - Gray: Sensor offline or stale data
3. Click sensor marker → Popup shows:
   - Current H₂S and methane levels in ppb
   - Plain language: "Air quality is GOOD" or "H₂S levels are ELEVATED"
   - Last updated timestamp
   - Link to detailed view
4. Detailed sensor view:
   - 24-hour trend chart showing how levels changed today
   - 7-day mini-chart for weekly context
   - Educational sidebar: "What is hydrogen sulfide?" with health context
   - Option to sign up for alerts when levels rise

#### **Secondary Flow: "How has air quality changed over time?"**

1. From map view, user selects "Historical Data" tab
2. Choose sensor(s) from dropdown or click map
3. Select date range (default: last 7 days, max 30 days for MVP)
4. View overlaid line charts with:
   - Clear legend distinguishing H₂S, methane, any co-pollutants
   - Shaded regions showing EPA threshold zones
   - Annotations for notable events if community adds them
5. Export as CSV or PNG for sharing

#### **Tertiary Flow: "Compare my neighborhood to others"**

1. Zone selector: Choose from predefined areas (Lower Ninth Ward, Industrial Canal, etc.)
2. Dashboard shows aggregate stats:
   - Average levels across all sensors in that zone
   - Comparison to city-wide averages
   - Days with elevated readings in past month
3. Optional: Overlay zones on map to visualize spatial patterns

### Critical Design Elements

**Map Component:**

- Mobile-first: Touch-friendly markers, zoom/pan gestures
- Satellite or street map toggle (some users navigate by landmarks)
- "Locate me" button to center map on user's current location
- Boundary overlays for named zones (neighborhoods)

**Data Visualization:**

- Simple line charts (avoid complex multi-axis plots)
- Color-coded to match EPA Air Quality Index categories where applicable
- Tooltips on hover/tap showing exact values and timestamps
- Option to toggle between ppb units and categorical labels ("Low", "Moderate", "High")

**Plain Language Translations:**

- Every technical term has a tooltip: "ppb = parts per billion"
- Health context always present: "15 ppb H₂S may cause odor complaints but is below health concern levels"
- Sensor limitations explained: "This is a community sensor, not EPA-certified. Use for awareness, not enforcement."

**Alert Signup (Optional for MVP):**

- Email/SMS registration form
- Choose alert threshold: any exceedance, only hazardous levels, daily summaries
- Unsubscribe link in every notification

### Framework Considerations

**From Technical Preferences:**

- Prefer framework with strong DX and flexible deployment (not locked to React/Vercel)
- Candidates: SvelteKit (modern, performant), Astro (content-focused), Next.js with Fly.io/other hosting
- Must support server-side rendering for SEO and low-bandwidth users
- Easy integration with mapping libraries (Mapbox GL JS, Leaflet)

**Responsive Breakpoints:**

- Mobile (320-767px): Single column, map full-width, charts stack vertically
- Tablet (768-1023px): Two-column where appropriate, larger charts
- Desktop (1024px+): Multi-column dashboard, side-by-side comparisons

### User Roles in Dashboard

**Public Viewer (No Login):**

- See current map and recent data
- Export CSVs
- Read educational content

**Partner Organization (Login Required):**

- Access admin view showing all sensors and their health status
- Download bulk historical data
- Manage alert subscriptions for their organization

**Maintainer (Admin):**

- Sensor management: Add/edit/remove sensors from registry
- Annotate events: "Construction near sensor A on 3/15"
- View system health metrics

---

## Functional Requirements

Requirements organized by capability, with acceptance criteria for implementation validation.

### FR Dependencies

Critical dependencies between functional requirements inform implementation sequencing:

- **FR-2 (Data Ingestion) depends on FR-1 (Sensor Hardware):** API ingestion endpoints require operational sensors producing data
- **FR-3 (Data Consumption) depends on FR-2 (Data Ingestion):** Query endpoints require stored sensor readings in database
- **FR-4.3 (Device Authentication) is prerequisite for FR-2:** Sensors must authenticate before uploading data
- **FR-4.1 (Public Read Access) is prerequisite for FR-3:** Query endpoints require rate limiting and access control
- **FR-5 (Dashboard) depends on FR-3 (Data Consumption):** UI queries data via FR-3 API endpoints
- **FR-6 (Alerts) depends on FR-2 (Data Ingestion):** Alert system monitors incoming readings for threshold exceedances
- **FR-7.2 (API Starter Projects) depends on FR-3:** Code examples require functional query endpoints
- **FR-7.1 (Hardware Playbook) depends on FR-1:** Documentation requires validated sensor assembly process
- **FR-8 (Workforce Development) runs parallel:** Students/apprentices contribute to FR-1, FR-2, FR-3, FR-5 implementation

**Implementation Note:** FR-1 → FR-2 → FR-3 → FR-5 represents the critical path for end-to-end system functionality.

---

### FR-1: Sensor Hardware & Edge Processing

**Capability:** DIY sensor kits capture gas concentration readings and forward data to cloud infrastructure.

**Requirements:**

**FR-1.1 Sensor Kit Assembly**
**Phase:** MVP

- System provides hardware blueprint specifying Raspberry Pi 4/5, Arduino Nano Every, and gas sensors with ppb-level sensitivity that connect via I²C or analog interfaces (specific sensor models determined during hardware spike)
- Assembly documentation includes wiring diagrams, power requirements, and component sourcing guides
- Kit supports I²C communication between Arduino and gas sensors
- **Acceptance:** Student can assemble functional unit from blueprint in <4 hours with instructor support

**FR-1.2 Sensor Data Acquisition**
**Phase:** MVP

- Arduino firmware reads H₂S and methane concentrations from sensors via I²C protocol
- Readings captured at configurable intervals (default: every 5 minutes)
- Raw sensor values converted to ppb units using manufacturer calibration formulas or empirical calibration curves
- **Acceptance:** Arduino successfully reads and converts sensor values with <5% deviation from reference measurements

**FR-1.3 Edge-to-Cloud Data Pipeline**
**Phase:** MVP

- Arduino forwards readings to Raspberry Pi via USB serial connection
- Raspberry Pi service validates and appends readings to local storage
- Readings include: timestamp (UTC), GPS coordinates, gas concentrations (ppb), sensor metadata
- **Acceptance:** 95% of readings captured by Arduino reach cloud storage within 2 hours

**FR-1.4 Batch Upload & Retry Logic**
**Phase:** MVP

- Raspberry Pi uploads reading batches to cloud API hourly via HTTPS POST
- Failed uploads queued locally and retried
- Local storage maintains 48-hour buffer before pruning oldest data
- **Acceptance:** Zero data loss during 24-hour network outage test; all buffered readings uploaded upon reconnection

**FR-1.5 Sensor Health Monitoring**
**Phase:** Growth

- Edge service logs sensor status: operational, calibrating, maintenance, error
- Watchdog process detects and logs Arduino communication failures
- System flags stale data (>2 hours since last reading)
- **Acceptance:** Dashboard correctly displays sensor offline within 15 minutes of Arduino failure

---

### FR-2: Cloud API - Data Ingestion

**Capability:** Secure API accepts authenticated sensor uploads and validates data integrity.

**Dependencies:** Requires FR-1.3 (edge devices uploading data) and FR-4.3 (device authentication).

**FR-2.1 Reading Ingestion Endpoint**
**Phase:** MVP

- POST `/v1/readings` accepts JSON batches up to 1000 readings per request
- Validates: sensor_id exists, timestamps within 24h, GPS within Louisiana bounds, concentrations non-negative
- Returns 201 Created with receipt confirmation or 400 Bad Request with specific error
- **Acceptance:** API correctly rejects invalid data (bad timestamps, out-of-range GPS) and accepts valid batches

**FR-2.2 Duplicate Detection**
**Phase:** MVP

- System deduplicates readings with identical sensor_id + timestamp combinations
- Idempotency allows sensors to retry uploads without creating duplicate records
- **Acceptance:** Re-uploading same batch 5 times results in single set of stored readings

**FR-2.3 Device Authentication**
**Phase:** MVP

- Each sensor assigned unique API key with write-only scope
- Keys rotatable without hardware reconfiguration (via secure bootstrap process)
- Invalid or revoked keys return 401 Unauthorized
- **Acceptance:** Sensor with valid key uploads successfully; sensor with revoked key fails with 401

**FR-2.4 Metadata Enrichment**
**Phase:** MVP

- System attaches sensor certification status, calibration date, QA/QC flags to every reading
- Louisiana SB 503 legal disclaimer added automatically to non-certified sensor data
- **Acceptance:** All stored readings include complete metadata per schema definition

---

### FR-3: Cloud API - Data Consumption

**Capability:** Public and partner users query air quality data for current conditions, historical trends, and geographic analysis.

**Dependencies:** Requires FR-2 (ingested data in database) and FR-4.1 (public access controls).

**FR-3.1 Latest Readings Query**
**Phase:** MVP

- GET `/v1/readings/latest` returns most recent reading from each specified sensor or geographic zone
- Supports filtering by sensor IDs, lat/lon radius, or named zones
- Optional gas type filtering (H₂S only, methane only, all)
- **Acceptance:** Query for "all sensors in Lower Ninth Ward" returns correct latest readings in <2 seconds

**FR-3.2 Historical Time-Series Query**
**Phase:** MVP

- GET `/v1/readings/timeseries` returns readings within specified time window (max 30 days for MVP)
- Supports raw data or aggregated intervals (15min, hourly, daily)
- Returns statistical summaries (min/max/avg) when aggregation requested
- **Acceptance:** 7-day hourly average query for 10 sensors completes in <3 seconds

**FR-3.3 Threshold Alert Query**
**Phase:** MVP

- GET `/v1/readings/alerts` returns readings that exceeded EPA or local thresholds
- Filters by location, date range, and severity (warning, hazardous)
- Includes threshold context and health guidance in response
- **Acceptance:** Query correctly identifies all readings above EPA H₂S threshold (10 ppb 24h avg, 70 ppb 1h peak)

**FR-3.4 Sensor Registry**
**Phase:** MVP

- GET `/v1/sensors` lists all sensors with status, location, deployment date, last seen timestamp
- Public endpoint requires no authentication
- Includes certification status and hardware specifications
- **Acceptance:** Registry accurately reflects sensors deployed in field; offline sensors marked within 15 minutes

**FR-3.5 Geographic Zone Support**
**Phase:** Growth

- GET `/v1/zones` returns GeoJSON boundaries for predefined neighborhoods
- Queries support zone names (e.g., "Lower_Ninth_Ward") in addition to lat/lon
- **Acceptance:** System correctly maps sensor GPS coordinates to appropriate zones

---

### FR-4: Access Control & Security

**Capability:** Role-based authentication protects sensitive operations while keeping public data accessible.

**FR-4.1 Public Read Access**
**Phase:** MVP

- Sensor registry, latest readings, and time-series queries available without authentication
- Rate limited to 100 requests/hour per IP address
- **Acceptance:** Unauthenticated user can query current air quality and view historical trends

**FR-4.2 Partner API Keys**
**Phase:** Growth

- Organizations receive API keys with elevated rate limits (1000 req/hour)
- Keys include read access to full dataset and bulk export capabilities
- **Acceptance:** Partner key enables historical data export for all sensors across 30-day window

**FR-4.3 Device Write-Only Keys**
**Phase:** MVP

- Sensors authenticate with device-specific API keys scoped to ingestion only
- Keys cannot query data or access other sensors' information
- **Acceptance:** Device key successfully uploads data but receives 403 Forbidden when attempting to query

**FR-4.4 Admin Access**
**Phase:** MVP

- Admin users authenticate with API keys or basic token authentication with elevated privileges
- Can manage sensor registry (add/edit/remove), annotate events, view system health
- **Note:** OAuth2 implementation with JWT/refresh tokens deferred to Growth phase
- **Acceptance:** Admin can add new sensor to registry and it appears in public API within 1 minute

---

### FR-5: Community Dashboard - Visualization

**Capability:** Web interface enables residents to understand local air quality without technical expertise.

**Dependencies:** Requires FR-3 (query API endpoints for data retrieval).

**FR-5.1 Interactive Map View**
**Phase:** MVP

- Dashboard displays New Orleans map with sensor markers color-coded by current air quality status
- Clicking marker shows popup with current readings, plain-language status, last update time
- Mobile-responsive with touch-friendly controls
- **Acceptance:** User on mobile phone can view current air quality across all sensors and tap for details

**FR-5.2 Sensor Detail View**
**Phase:** MVP

- Individual sensor page shows 24-hour and 7-day trend charts for H₂S and methane
- Charts include EPA threshold shading and plain-language interpretation
- Educational sidebar explains what each gas is and health implications
- **Acceptance:** Non-technical user can describe whether air quality improved or worsened over past week

**FR-5.3 Historical Comparison**
**Phase:** MVP

- Users select date range (up to 30 days) and overlay multiple sensors on single chart
- Export data as CSV or chart as PNG image
- **Acceptance:** Community organizer exports 30-day H₂S data for advocacy presentation

**FR-5.4 Neighborhood Zone Comparison**
**Phase:** Growth

- Zone selector aggregates sensors within predefined neighborhoods
- Shows average levels, comparison to citywide stats, days with elevated readings
- **Acceptance:** User compares Industrial Canal zone to Lower Ninth Ward for past month

**FR-5.5 Plain Language Translation**
**Phase:** MVP

- All technical terms have tooltips explaining meaning (e.g., "ppb = parts per billion")
- Every reading includes health context ("15 ppb H₂S may cause odor but is below health concern")
- Sensor limitations clearly stated ("Community sensor, not EPA-certified. Use for awareness, not enforcement.")
- **Acceptance:** User testing shows 80% comprehension of health implications without prompting

**FR-5.6 Core Accessibility Compliance**
**Phase:** MVP

- WCAG 2.1 AA compliant: keyboard navigable, screen reader compatible, high contrast text
- Page loads in <3 seconds on 3G connection with initial content visible in <1 second
- **Acceptance:** Screen reader successfully navigates and announces all key information; passes automated accessibility audit

**FR-5.7 Multi-Language Support**
**Phase:** Growth

- Interface supports language toggle between English and Spanish
- All dashboard text, labels, tooltips, and educational content translatable
- **Acceptance:** User can switch to Spanish language and all UI elements display correctly

---

### FR-6: Alert & Notification System

**Capability:** Users subscribe to notifications when air quality exceeds thresholds.

**Dependencies:** Requires FR-2 (real-time data ingestion for threshold monitoring) and FR-3.3 (alert query logic).

**FR-6.1 Alert Subscription**
**Phase:** Growth

- Web form collects email/SMS, sensor selection, and threshold preferences
- Options: any EPA exceedance, only hazardous levels, daily summaries
- **Acceptance:** User successfully subscribes and receives test notification within 5 minutes

**FR-6.2 Alert Delivery**
**Phase:** Growth

- System checks new readings against subscriber thresholds in real-time
- Sends notifications within 15 minutes of threshold exceedance
- Includes unsubscribe link in every message
- **Acceptance:** Subscriber receives alert when H₂S exceeds 10 ppb at monitored sensor

---

### FR-7: Open Source Documentation & Replication

**Capability:** External partners can independently replicate Louisianair deployments.

**Dependencies:** FR-7.1 requires validated FR-1 (sensor assembly); FR-7.2 requires functional FR-3 (API endpoints); FR-7.3 requires deployed FR-2 and FR-3 (cloud infrastructure).

**FR-7.1 Hardware Assembly Playbook**
**Phase:** MVP

- Step-by-step guide: component sourcing, wiring, firmware installation, calibration
- Includes photos, diagrams, troubleshooting tips, estimated costs
- **Acceptance:** External team with no prior IoT experience builds functional sensor using only playbook

**FR-7.2 API Integration Starter Projects**
**Phase:** MVP

- Node.js example: Fetch and display readings on web page
- **Note:** Python and Go examples deferred to Growth phase
- **Acceptance:** Developer implements functional dashboard using Node.js starter code in <2 hours

**FR-7.3 Deployment Guide**
**Phase:** MVP

- Cloud hosting setup (Fly.io or AWS), database initialization, API deployment
- Environment configuration, secrets management, monitoring setup
- **Acceptance:** DevOps engineer deploys complete Louisianair instance in new region using guide

**FR-7.4 API Reference Documentation**
**Phase:** MVP

- OpenAPI 3.0 specification with interactive Swagger UI explorer
- Request/response examples for every endpoint
- Postman collection with pre-configured environments
- **Acceptance:** Developer successfully queries API using only reference docs, no additional support

---

### FR-8: Workforce Development Integration

**Capability:** Students and apprentices gain hands-on experience through meaningful contributions.

**Dependencies:** Runs parallel to FR-1 through FR-5 implementation; students contribute to hardware assembly (FR-1), apprentices contribute to API/dashboard development (FR-2, FR-3, FR-5).

**FR-8.1 Educational Curriculum Materials**
**Phase:** MVP

- IoT class lesson plans: sensor assembly, Arduino programming, data flow understanding
- Lab exercises with grading rubrics and learning outcomes
- **Acceptance:** Instructor delivers full IoT unit resulting in 5-10 deployed sensors

**FR-8.2 Apprentice Code Contributions**
**Phase:** MVP

- Apprentices commit production code to API, data pipeline, or dashboard repositories
- Pull request reviews provide mentorship and skill development
- **Acceptance:** At least 3 apprentices have merged commits in deployed codebase

**FR-8.3 Student Documentation Outputs**
**Phase:** MVP

- Students create assembly guides, troubleshooting wikis, deployment reports based on field experience
- Outputs integrated into public playbook for community benefit
- **Acceptance:** Student-authored content comprises ≥30% of final playbook documentation

---

## Non-Functional Requirements

### Performance

**NFR-P1: API Query Response Time**

- Regional queries covering up to 50 sensors return within 2 seconds (95th percentile)
- Single sensor latest reading queries return within 500ms (95th percentile)
- Historical time-series queries (7-day window, 10 sensors, hourly aggregation) complete within 3 seconds
- **Rationale:** Community users expect responsive dashboards; slow queries discourage exploration

**NFR-P2: Dashboard Load Time**

- Initial page load completes in <3 seconds on 3G connection (Slow 3G profile: 400ms RTT, 400kbps down)
- First meaningful paint (map visible) within 1 second
- Interactive (user can click markers) within 2 seconds
- **Rationale:** Many fenceline communities have limited broadband access; mobile data is primary internet source

**NFR-P3: Data Freshness**

- Sensor readings ingested and available for querying within 5 minutes of API upload
- Dashboard displays "last updated" timestamps accurate to within 1 minute
- **Rationale:** Hourly batch uploads + processing time = acceptable latency for community awareness (not emergency response)

### Security

**NFR-S1: Data in Transit**

- All API communications over HTTPS with TLS 1.2+ (prefer TLS 1.3)
- HTTP requests automatically upgraded to HTTPS
- Certificate pinning not required (sensor firmware simplicity prioritized)
- **Rationale:** Sensor data is public, but authentication tokens and device keys must be protected

**NFR-S2: Authentication & Secrets**

- API keys stored hashed (bcrypt or Argon2) in database
- JWT tokens signed with RS256 (asymmetric keys), rotatable without downtime
- Device API keys transmitted only once during provisioning; not logged or displayed in UI after initial generation
- **Rationale:** Compromise of device key allows fake data injection; must treat as sensitive

**NFR-S3: Input Validation**

- All API endpoints validate and sanitize inputs (prevent SQL injection, XSS)
- GPS coordinates validated against reasonable bounds (reject Antarctica, middle of ocean)
- Gas concentration values validated against sensor physical limits (reject negative or impossibly high readings)
- **Rationale:** Community sensors may malfunction or be tampered with; reject obvious anomalies

**NFR-S4: Rate Limiting & DoS Protection**

- Per-IP rate limiting on public endpoints (100 req/hour)
- Device authentication required for ingestion (prevent spam uploads)
- No sensitive operations (sensor deletion, admin actions) exposed without OAuth
- **Rationale:** Open API is attack surface; must prevent abuse without hindering legitimate use

**NFR-S5: Dependency Security**

- Automated dependency scanning (Dependabot, Snyk) for known vulnerabilities
- Critical CVEs patched within 7 days; high severity within 30 days
- Frontend dependencies (npm/yarn) regularly audited and updated
- **Rationale:** Open source project depends on community trust; security negligence undermines mission

### Reliability & Availability

**NFR-R1: API Uptime**

- Target ≥99.5% uptime (allows ~3.6 hours/month downtime for maintenance)
- Planned maintenance communicated 48 hours in advance via status page
- Graceful degradation: if database read replicas fail, serve stale data with warning rather than hard error
- **Rationale:** Community users rely on data for advocacy; excessive downtime erodes trust

**NFR-R2: Data Integrity**

- Zero data loss during sensor uploads (retries with exponential backoff until success or 48h expiration)
- Database backups daily with 30-day retention
- Point-in-time recovery capability for last 7 days
- **Rationale:** Historical data is primary product value; loss is unacceptable

**NFR-R3: Sensor Offline Handling**

- Dashboard clearly indicates sensors offline >2 hours (gray marker, "Last seen: X hours ago")
- API returns last known reading with `stale: true` flag when sensor offline
- No automatic data interpolation or estimation (preserve transparency)
- **Rationale:** Community sensors lack professional maintenance; outages expected but must be communicated honestly

**NFR-R4: Operational Observability**

- Structured logging (JSON format) with correlation IDs for request tracing
- Health endpoints expose service status, database connectivity, recent error rates
- Alerting on critical failures: database down, API error rate >5%, disk space <10%
- **Rationale:** Small team needs efficient troubleshooting; observability prevents extended outages

### Scalability

**NFR-SC1: Sensor Network Growth**

- System architecture supports scaling from 5-10 MVP sensors to 50 sensors without major redesign
- Database schema and API queries optimized for 50 concurrent sensors × hourly readings
- Infrastructure costs remain <$200/month at 50-sensor scale (Fly.io or similar)
- **Rationale:** Post-MVP expansion likely if pilot succeeds; avoid costly rewrites

**NFR-SC2: Query Load**

- API handles 1000 requests/hour sustained load without degradation
- Database connection pooling prevents exhaustion under burst traffic
- Read replicas or caching layer added if query load exceeds single-instance capacity
- **Rationale:** Public API may attract external developers; must handle moderate traffic spikes

**NFR-SC3: Data Retention**

- MVP: 30-day minimum retention, 90-day storage if resources allow
- Post-MVP: Consider archival strategy for multi-year time-series (cold storage, data warehousing)
- Database size projections: 10 sensors × 12 readings/hour × 30 days × ~500 bytes = ~1.7MB/month (negligible)
- **Rationale:** Gas sensors produce low data volume; storage is cheap, design for long-term retention

### Maintainability & Operations

**NFR-M1: Deployment Automation**

- Infrastructure-as-code (Terraform or similar) for reproducible cloud deployments
- CI/CD pipeline: automated tests on PR, deploy to staging on merge to main, manual promotion to production
- Zero-downtime deploys for API updates (blue-green or rolling deploys)
- **Rationale:** Small team cannot afford manual deployment toil; automation reduces errors

**NFR-M2: Code Quality**

- Automated linting and formatting (gofmt/prettier) enforced in CI
- Test coverage ≥70% for API business logic, ≥50% for edge firmware
- Documentation: README, API reference, architecture decision records (ADRs)
- **Rationale:** Student and apprentice contributors need clear standards; code quality enables long-term maintenance

**NFR-M3: Operational Runbooks**

- Playbooks for common scenarios: sensor offline, database restore, API key rotation, scaling up infrastructure
- Incident response procedures with on-call rotation (if team grows)
- **Rationale:** Knowledge must transfer beyond founding team; runbooks prevent single points of failure

**NFR-M4: Dependency Management**

- Minimal dependency footprint (prefer standard library where reasonable)
- Pin major versions, allow minor/patch updates for security fixes
- Document rationale for complex or unusual dependencies
- **Rationale:** Every dependency is maintenance burden; justify trade-offs explicitly

### Compliance & Data Governance

**NFR-C1: Louisiana SB 503 Compliance**

- Every API response includes sensor certification status ("none", "louisiana_deq", "epa_approved")
- Non-certified sensor data includes legal disclaimer text in metadata
- Dashboard prominently displays: "Community sensor data for awareness, not enforcement"
- **Rationale:** Louisiana law restricts enforcement to certified methods; must communicate sensor limitations clearly

**NFR-C2: Open Data Licensing**

- API data available under Open Database License (ODbL) or CC BY 4.0
- Code repositories use MIT or Apache 2.0 license
- Hardware designs use CERN OHL or CC BY-SA
- Documentation uses CC BY 4.0
- **Rationale:** Open licensing enables replication and builds trust in data integrity

**NFR-C3: Privacy & PII**

- Sensor locations (GPS coordinates) are public by design (consent obtained during deployment)
- No personally identifiable information collected from dashboard viewers (no login required for public access)
- Partner API keys and admin accounts: email addresses not shared publicly
- **Rationale:** Air quality monitoring serves public interest; data transparency prioritized over individual privacy

**NFR-C4: Data Provenance**

- Every reading includes metadata: sensor hardware specs, calibration date, deployment location
- QA/QC flags document known sensor limitations (drift detected, calibration overdue)
- Audit log for admin actions (sensor added/removed, configuration changes)
- **Rationale:** Data trustworthiness depends on understanding sensor quality and maintenance history

### Accessibility

**NFR-A1: Web Accessibility Standards**

- WCAG 2.1 Level AA compliance for dashboard
- Automated accessibility testing in CI pipeline (axe, pa11y)
- Manual screen reader testing (NVDA, JAWS) quarterly
- **Rationale:** Environmental justice requires inclusive tools; exclude no one due to disability

**NFR-A2: Low-Bandwidth Optimization**

- Dashboard loads core functionality on <500KB initial transfer
- Images optimized (WebP format, lazy loading)
- Map tiles cached aggressively; offline map fallback considered post-MVP
- **Rationale:** Many target users have limited data plans or slow connections

**NFR-A3: Multi-Language Support (Future)**

- MVP: English only, rely on browser translation (Google Translate)
- Post-MVP: Consider Spanish localization if community partners prioritize it
- Architecture: i18n-ready (externalize strings, support locale switching)
- **Rationale:** Resource constraints defer translation, but design should not preclude future support

### Educational & Workforce Development

**NFR-E1: Student-Friendly Documentation**

- Hardware assembly guides use plain language, avoid jargon
- Code examples include extensive comments explaining "why" not just "what"
- Video tutorials for complex steps (sensor calibration, firmware flashing)
- **Rationale:** Students and apprentices are primary contributors; lower barriers to entry

**NFR-E2: Onboarding Time**

- New apprentice can set up local development environment in <2 hours following README
- New IoT student can assemble sensor kit in <4 hours with instructor support
- **Rationale:** Limited instructional time; efficient onboarding maximizes learning impact

**NFR-E3: Code Review Process**

- All apprentice PRs receive mentorship-focused review within 48 hours
- Feedback balances "teach best practices" with "ship working code"
- Apprentices document learning in PR descriptions or retrospectives
- **Rationale:** Workforce development is mission-critical; code review is teaching opportunity

---

## References & Context Documents

**Product Foundation:**

- Product Brief: `docs/product-brief-louisianair-2025-11-03.md`
- Brainstorming Session: `docs/bmm-brainstorming-session-2025-11-04.md`
- Technical Research: `docs/research-technical-2025-11-04.md` (Section 1: Requirements)

**Key External Resources:**

- [Louisiana Community Air Monitoring Reliability Act (SB 503): Legal framework for sensor data usage](https://www.legis.la.gov/legis/BillInfo.aspx?s=24rs&b=SB503&sbi=y)
- [EPA Air Quality Thresholds: H₂S (10 ppb 24h avg, 70 ppb 1h peak)](https://www.epa.gov/aegl/hydrogen-sulfide-results-aegl-program)
  - [NAAQS Table](https://www.epa.gov/criteria-air-pollutants/naaqs-table)
- DFRobot Sensor Documentation: https://wiki.dfrobot.com/SKU_SEN0465toSEN0476_Gravity_Gas_Sensor_Calibrated_I2C_UART (reference example, not required)
- [LEAN Community Air Monitoring Network: Partner organization context](https://leanweb.org/about-lean/projects/community-air-monitoring-network)
  - [Advocate - Can these new pollution monitors show the way forward in Louisiana?](https://www.theadvocate.com/baton_rouge/news/environment/with-new-air-sensors-lean-seeks-to-spur-pollution-gains/article_b239409a-ea0a-4b05-918f-95506d9719ae.html)

---

## Project Timeline

**Phase 1: Planning & Design (Oct-Dec 2025)**

- ✅ Product brief completed
- ✅ Technical brainstorming session
- ✅ PRD creation (this document)
- 🔜 Architecture design
- 🔜 Component procurement

**Phase 2: Prototyping (Jan 2026)**

- Hardware spike: sensor-to-cloud pipeline validation
- Database schema design
- API contracts definition
- IoT class preparation and instructor staffing

**Phase 3: Implementation (Feb-Mar 2026)**

- IoT students build and deploy 5-10 sensor units
- Apprentices develop API and data pipeline
- Database integration and testing
- Dashboard initial development

**Phase 4: Integration & Deployment (Mar-Apr 2026)**

- API completion and sensor connection
- Dashboard finalization
- Field debugging and operational readiness
- Documentation and playbook creation

**Phase 5: Public Launch (Apr/May 2026)**

- Open API promotion
- Playbook and starter projects publication
- Community partner training
- Initial advocacy and educational use cases

**Phase 6 (Optional): Expansion (Summer-Fall 2026)**

- Hackathons and workshops (funding dependent)
- Network expansion beyond New Orleans
- Partner onboarding and replication support

---

## Next Steps

### Immediate (Within 2 Weeks)

**For Product Team:**

1. **Review this PRD** - Gather feedback from Operation Spark, shift7, and LEAN partners
2. **Hardware Spike Execution** - Validate sensor→Pi→cloud pipeline with real hardware (Priority #1 from brainstorming)
3. **API Contract Definition** - Draft OpenAPI spec based on FR-2 and FR-3 requirements

**For Architecture:**

1. **Run architecture workflow** - Create technical architecture document
2. **Database selection** - Evaluate PostgreSQL + TimescaleDB vs InfluxDB vs hybrid
3. **Hosting plan** - Finalize Fly.io vs AWS decision with cost projections

### Near-Term (2-4 Weeks)

**For Development:**

1. **Repository setup** - Initialize monorepo or multi-repo structure with CI/CD
2. **API scaffold** - Implement health endpoints, authentication, basic CRUD
3. **Edge firmware** - Arduino code for sensor reading and serial communication

**For Education:**

1. **IoT curriculum finalization** - Lesson plans, lab exercises, grading rubrics
2. **Component ordering** - Procure Raspberry Pis, Arduinos, sensors for 10 kits
3. **Instructor training** - Brief IoT class instructors on hardware and objectives

### Medium-Term (1-3 Months)

**Milestone: IoT Class Deployment**

- Students assemble and deploy 5-10 functional sensor units
- Real data flowing from field sensors to cloud API
- Dashboard showing live map with actual readings

**Milestone: API Public Beta**

- External partners can query API with documentation
- Starter projects published for replication
- Performance and reliability validated under real load

### Epic Breakdown Required

This PRD must be decomposed into implementable epics and bite-sized stories (200k context limit for dev agents).

**Next Workflow:** Run `/bmad:bmm:workflows:create-epics-and-stories` to generate:

- Epic definitions with acceptance criteria
- Story breakdown with task granularity
- Implementation sequencing and dependencies

---

## Document Summary

**What We're Building:**
An open-source air quality monitoring platform that empowers New Orleans fenceline communities with DIY sensors, shared data infrastructure, and accessible dashboards while Operation Spark apprentices and students build production-ready IoT and software skills.

**Why It Matters:**
Louisianair democratizes air quality insights for environmental justice through open data, accessible technology, and youth-powered community capacity building. It transforms pollution monitoring into workforce development and civic action.

**Key Success Metrics:**

- 5-10 operational sensor deployments in target communities
- 15-25 students/apprentices gaining hands-on technical experience
- Public API with external partner adoption
- Repeatable playbook enabling independent replication

**What Makes It Special:**
The magic happens when fenceline residents understand their air quality in real-time, students build their first IoT system and see data flow to the cloud, and community organizers use trustworthy local data to advocate - all while apprentices gain career-ready skills strengthening environmental justice infrastructure.

**Project Classification:** Level 3 Complex System - API/Backend + IoT + Dashboard
**Timeline:** Oct 2025 - May 2026 pilot phase
**Domain:** Environmental/Civic Technology with Louisiana regulatory awareness

---

_This PRD captures the complete vision, requirements, and success criteria for Louisianair._

_Created through collaborative discovery between OpSpark and PM facilitator John._

_Next Step: Architecture design → Epic/Story breakdown → Implementation_

**Document Version:** 1.0
**Last Updated:** 2025-11-05
**Status:** Ready for Architecture Phase
**Workflow:** bmad/bmm/workflows/prd
