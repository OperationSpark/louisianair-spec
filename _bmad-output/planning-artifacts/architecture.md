# Aethernet Architecture

**Author:** OpSpark
**Date:** 2025-11-06
**Version:** 1.0
**Architecture Type:** Multi-Component IoT + Cloud + Web Experience System

---

## Executive Summary

Aethernet's architecture is designed as a distributed system with three primary components: IoT edge devices (Raspberry Pi + Arduino), a cloud-based REST API (Go), and a community-facing web experience (SvelteKit) that powers dashboards, admin panels, and device portals. The system prioritizes data integrity, operational simplicity, and accessibility for New Orleans fenceline communities while supporting workforce development through clear architectural patterns.

**Key Architectural Decisions:**

- **Go standard library** for API (zero framework dependencies, maximum control)
- **TimescaleDB + PostGIS** for time-series geospatial data storage
- **SQLite edge buffering** with retry logic for network resilience
- **SvelteKit + Sveaflet** for lightweight, accessible web surfaces
- **Fly.io deployment** with managed Postgres in `iad` region
- **Multi-component monorepo** for atomic cross-component changes

This architecture supports scaling from 5-10 MVP sensors to 50+ sensors in Growth phase without major redesign, maintains <$20/month operational costs for MVP, and enables 15-25 apprentices to contribute meaningfully to production infrastructure.

---

## Project Initialization

### Component Initialization Commands

**API Service:**

```bash
mkdir -p aethernet/api
cd aethernet/api
go mod init github.com/operationspark/aethernet/api
mkdir -p cmd/server internal/{handlers,middleware,models,db,types,config} testutil
```

**Web Frontend:**

```bash
cd aethernet
npm create svelte@latest web
# Select: SvelteKit demo app, TypeScript, Prettier, ESLint, Vitest
cd web
npm install
npm install sveaflet leaflet
```

**Edge Service:**

```bash
mkdir -p aethernet-edge-firmware/edge-service/{src,tests,systemd}
cd aethernet-edge-firmware/edge-service
python3 -m venv venv
source venv/bin/activate
pip install requests pytest pytest-cov
```

**Firmware (MCU/SoC):**

```bash
mkdir -p aethernet-edge-firmware/firmware/reference-sensor
# Use Arduino IDE, PlatformIO, or board-specific SDK to create sketches
```

**Database:**

```bash
mkdir -p aethernet/db/migrations
# Migrations written in SQL, applied via migration tool or manually
```

---

## Decision Summary

| Category            | Decision                                                                              | Version                                                      | Affects Epics | Rationale                                                                                                          |
| ------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------ | ------------- | ------------------------------------------------------------------------------------------------------------------ |
| API Framework       | Go standard library (net/http)                                                        | Go 1.25                                                      | Epic 2, 3, 5  | Zero dependencies, 1.22+ routing enhancements, aligns with "boring technology" philosophy, straightforward testing |
| Authentication      | Stateful Go auth: bcrypt-hashed users + signed session cookies + team/device API keys | Go 1.23+, bcrypt v4.0.1, SvelteKit 2.x (verified 2025-11-07) | Epic 2, 5     | Implement in-house auth now, reuse team context + API keys while retaining exit path to hosted IdP                 |
| Database            | PostgreSQL 16+ with TimescaleDB 2.23.0 + PostGIS                                      | PG 16, TimescaleDB 2.23.0                                    | Epic 2, 3     | Time-series + geospatial queries, automatic compression, continuous aggregates, 30-day retention                   |
| Deployment Platform | Fly.io (iad region), Managed Postgres, GitHub Actions CI/CD                           | N/A                                                          | All epics     | Low maintenance, automatic HTTPS, ~$10-20/month MVP cost, scales to $120-150/month at 50 sensors                   |
| IoT Edge Protocol   | SQLite local buffering + JSON batch HTTPS upload                                      | SQLite 3, Python 3.9+                                        | Epic 1, 2     | ACID guarantees, retry logic simplicity, 48-hour buffer with <500KB storage per sensor                             |
| API Response Format | Envelope pattern with data/error/meta fields                                          | N/A                                                          | Epic 2, 3     | Consistent shape across all endpoints, request ID tracing, extensible metadata                                     |
| Error Handling      | Structured JSON logging, predefined error codes, panic recovery                       | Go slog (stdlib)                                             | All epics     | Request correlation, debuggable production issues, apprentice-friendly patterns                                    |
| Date/Time Handling  | UTC everywhere, RFC3339 format, display local in browser                              | N/A                                                          | All epics     | Prevents timezone bugs, DST issues, cross-component timestamp inconsistencies                                      |
| Testing Strategy    | 70% API coverage, 50% edge coverage, CI enforcement                                   | Go testing, Vitest, pytest                                   | All epics     | Quality gates in PRs, apprentice code review checkpoints                                                           |
| Rate Limiting       | In-memory token bucket (100/1000/unlimited by tier)                                   | golang.org/x/time/rate                                       | Epic 3, 5     | Zero Redis dependency for MVP, 13-17M ops/sec performance, migrate to Redis in Growth for multi-region             |

---

## Project Structure

### Repository Layout: Multi-Repo Overview

- `aethernet-core` – Web/API platform and shared documentation that product teams actively deploy.
- `aethernet-edge-firmware` – Field devices (edge gateway runtimes plus interchangeable MCU/SoC firmware) packaged for tinkerers and community hardware experiments.
- **Split rationale:** Keeps CI/CD focused on production web/API services while allowing the hardware repo to version maker-ready releases (toolchains, flashing guides, adapter docs) at its own cadence without forcing DIY contributors to pull server code.

```
aethernet-core/
├── .github/workflows/         # CI for API/Web deployments
├── api/                       # Go REST API (unchanged layout)
├── web/                       # SvelteKit web app (dashboard, admin, portals)
├── docs/                      # PRD, architecture, guides
├── db/                        # Shared SQL migrations
├── .gitignore
├── README.md
└── LICENSE

aethernet-edge-firmware/
├── edge-service/              # Gateway runtime (Python or similar)
│   ├── src/                   # Serial ingestion, local buffering, upload
│   ├── tests/                 # Device-level test harnesses
│   └── systemd/               # Example service definitions
├── firmware/                  # MCU/SoC sketches + board configs
│   ├── reference-sensor/      # Generic I²C/CAN sensor readers
│   └── adapters/              # Board-specific wiring + bootstraps
├── docs/                      # Maker guides, wiring diagrams, flashing
├── toolchains/                # Build scripts, flashing utilities
└── README.md
```

---

## Epic to Architecture Mapping

### Epic 1: Sensor Network Foundation

**Components:** `aethernet-edge-firmware/edge-service`, `aethernet-edge-firmware/firmware`, `docs/hardware-playbook.md`

**Architectural Support:**

- SQLite local buffering (`edge-service/src/database.py`)
- Generic MCU/SoC sensor acquisition sketches (`firmware/reference-sensor/`)
- Edge gateway upload service with retry logic (`edge-service/src/uploader.py`)
- 48-hour buffer retention with <500KB storage per sensor

**Key Patterns:**

- Serial/field bus communication between MCU/SoC boards and gateway devices (USB, UART, CAN, etc.) with adapters documented in `firmware/adapters/`
- Batch size: Up to 1000 readings per upload
- Retry strategy: Exponential backoff (1h, 2h, 4h, 8h, 16h)

---

### Epic 2: Cloud Ingestion & Data Reliability

**Components:** `api/internal/handlers/readings.go`, `api/internal/db/`, `db/migrations/`

**Architectural Support:**

- Ingestion endpoint: `POST /v1/readings`
- TimescaleDB hypertable with 7-day chunks
- PostGIS for geospatial data (lat/lon → GEOGRAPHY(POINT))
- Device authentication middleware enforcing API keys + session policy
- Metadata enrichment: SB 503 disclaimers, certification status

**Key Patterns:**

- Deduplication: sensor_id + timestamp uniqueness
- Validation: Timestamps within 24 hours, GPS within Louisiana bounds
- Observability: Structured logging with request IDs

---

### Epic 3: Public Data Access & Partner APIs

**Components:** `api/internal/handlers/`, `docs/api-reference.md`

**Architectural Support:**

- Query endpoints:
  - `GET /v1/readings/latest`
  - `GET /v1/readings/timeseries`
  - `GET /v1/readings/alerts`
  - `GET /v1/sensors`, `GET /v1/sensors/{id}`
- TimescaleDB continuous aggregates for performance
- PostGIS spatial indexes for radius queries
- OpenAPI 3.0 specification

**Key Patterns:**

- Envelope responses with data/error/meta
- Rate limiting: 100 req/hour public, 1000 req/hour partner
- Performance: <2s for 50 sensors, <500ms single sensor

---

### Epic 4: Community Web Experience

**Components:** `web/src/`

**Architectural Support:**

- SvelteKit SSR for <3s load on 3G
- Sveaflet for Leaflet map integration
- Map view: `routes/+page.svelte`
- Sensor detail: `routes/sensors/[id]/+page.svelte`
- Historical comparison: `routes/history/+page.svelte`

**Key Patterns:**

- API client: Fetch with envelope response parsing
- Accessibility: WCAG 2.1 AA compliance, screen reader compatible
- Performance: <500KB bundle, progressive enhancement

---

### Epic 5: Trust, Access Control & Compliance

**Components:** `api/internal/middleware/`, cross-cutting concerns

**Architectural Support:**

- Rate limiting middleware (token bucket)
- Authentication middleware (session + API key enforcement)
- SB 503 disclaimers in all responses
- Audit logging with request correlation
- Security headers (HTTPS, X-Content-Type-Options, etc.)

**Key Patterns:**

- Predefined error codes prevent agent inconsistency
- Structured JSON logging for observability
- CORS whitelist for web origin

---

### Epic 6: Open Program & Workforce Enablement

**Components:** `docs/`, educational materials

**Architectural Support:**

- Hardware playbook: Assembly, calibration, deployment
- API integration examples: Node.js starter projects
- Contribution guide: Git conventions, PR requirements
- Testing standards: 70% API, 50% edge coverage

**Key Patterns:**

- Documentation in monorepo for atomic updates
- Student-friendly READMEs with setup scripts
- Portfolio artifacts for apprentice outcomes

---

## Technology Stack Details

### Core Technologies

**Backend API:**

- **Language:** Go 1.25
- **HTTP Framework:** Standard library `net/http`
- **Database Driver:** `github.com/lib/pq` (PostgreSQL wire protocol)
- **Logging:** `log/slog` (Go standard library, structured JSON)
- **Testing:** `testing` package + `github.com/stretchr/testify/assert`
- **Rate Limiting:** `golang.org/x/time/rate` (token bucket)

**Database:**

- **Primary:** PostgreSQL 16
- **Extensions:**
  - TimescaleDB 2.23.0 (time-series optimization, compression, continuous aggregates)
  - PostGIS 3.x (geospatial queries, GEOGRAPHY(POINT) type)
- **Hosting:** Fly.io Managed Postgres (Basic plan MVP, Launch plan Growth)

**Web Frontend:**

- **Framework:** SvelteKit 2.x (SSR, file-based routing)
- **Language:** TypeScript
- **Mapping:** Sveaflet + Leaflet.js
- **Styling:** TailwindCSS (low-bandwidth optimization)
- **Testing:** Vitest (unit), Playwright (E2E)

**IoT Edge:**

- **Language:** Python 3.9+
- **Local Storage:** SQLite 3 (ACID compliant buffering)
- **HTTP Client:** `requests` library
- **Service Management:** systemd

**Arduino Firmware:**

- **Language:** C/C++ (Arduino framework)
- **Hardware:** Arduino Nano Every
- **Communication:** I²C (sensors), USB Serial (Pi)

**Deployment:**

- **Platform:** Fly.io
- **Region:** `iad` (Ashburn, Virginia) - closest to New Orleans
- **CI/CD:** GitHub Actions
- **Containers:** Docker (multi-stage builds)
- **Secrets:** Fly.io secrets manager (`flyctl secrets`)

---

### Integration Points

**Edge → API:**

- **Protocol:** HTTPS POST
- **Endpoint:** `/v1/readings`
- **Authentication:** Bearer token (device API key)
- **Format:** JSON batch (max 1000 readings)
- **Retry Logic:** Exponential backoff, 48-hour buffer
- **Example:**

```json
POST /v1/readings
Authorization: Bearer <device_api_key>
{
  "sensor_id": "sensor-001",
  "batch": [
    {
      "timestamp": "2026-02-15T14:00:00Z",
      "location": {"lat": 29.9511, "lon": -90.0701},
      "readings": {"h2s_ppb": 15.2, "ch4_ppb": 450.3},
      "metadata": {
        "calibration_date": "2026-02-01T00:00:00Z",
        "sensor_status": "operational"
      }
    }
  ]
}
```

**Web → API:**

- **Protocol:** HTTPS GET
- **Authentication:** None (public endpoints) or Bearer token (partner)
- **Format:** JSON envelope responses
- **Caching:** 1-minute client-side cache
- **Example:**

```typescript
const response = await fetch("/v1/readings/latest?sensor_ids=sensor-001");
const json = await response.json();
if (json.error) {
  // Display error.message to user
} else {
  // Render json.data
}
```

**API → Database:**

- **Protocol:** PostgreSQL wire protocol
- **Connection Pool:** Max 20 connections
- **Query Timeout:** 10 seconds
- **Transaction Usage:** Multi-table operations only
- **Prepared Statements:** Always (SQL injection prevention)
- **Example:**

```go
const query = `
  INSERT INTO readings (sensor_id, timestamp, location, h2s_ppb, ch4_ppb, ...)
  VALUES ($1, $2, ST_SetSRID(ST_MakePoint($3, $4), 4326), $5, $6, ...)
`
db.Exec(query, sensorID, timestamp, lon, lat, h2s, ch4)
```

---

## Novel Patterns

### Community Alert Fan-out

**Purpose:** Convert hazardous readings into rapid, multi-channel notifications for New Orleans fenceline communities while preserving legal disclaimers and auditability.

**When to Apply:** Any ingestion that exceeds PRD thresholds for H₂S/CH₄ or produces SB 503 reportable events across Epics 2, 3, and 5.

```
[Edge Sensor]
    │ batch upload
    ▼
[API /v1/readings] ──> [Alert Classifier (api/internal/alerts/)]
                                │ queue payload
                                ▼
                       [Fan-out Worker]
           ┌──────────────┬───────────────┬────────────────┐
           ▼              ▼               ▼                ▼
      Web Dashboard   SMS/Email hooks  Partner Webhooks  Audit Log
 (web/src/routes/…)    (queue job)     (api/internal/)   (db/logs)
```

**Implementation Notes:**
- `api/internal/alerts/classifier.go` emits `AlertEvent{sensor_id, zone, severity, readings, disclaimer}` for downstream workers.
- `/v1/readings/alerts` exposes the same payload to the web experience, which renders disclaimers verbatim in `web/src/routes/history/+page.svelte`.
- Background fan-out jobs persist alert metadata in `db/alerts` and enqueue deliveries via the async runner defined in Deployment Architecture.

**Edge Cases:**
- *Stale data:* Classifier rejects readings where `now - timestamp > 15 minutes`.
- *Channel failures:* Delivery retries at 1m, 5m, 15m, then raises an ops alert via GitHub Issues or PagerDuty.
- *Rate spikes:* Alerts queue is bounded; overflow triggers a `rate_limited` response and writes to the audit log.

**States & Transitions:** `detected → queued → delivered → acknowledged`. Unacknowledged alerts remain highlighted on dashboards for 24h.

**Scalability:** Workers are stateless; scale horizontally by running additional Fly Machines pointing at the same queue/topic.

### Resilient Edge Buffering & Sync

**Purpose:** Guarantee no data loss for up to 48 hours of network outages while keeping hardware approachable for apprentices.

**Flow:**
1. Firmware writes readings to local storage via Arduino → Pi serial bridge.
2. `edge-service/src/database.py` persists rows into SQLite (WAL enabled) with `state=pending`.
3. `edge-service/src/uploader.py` batches pending rows, calls `POST /v1/readings` with device API keys, then marks rows `delivered`.
4. Retry cadence: 15s → 60s → 5m → 30m → 2h; after 2h the service raises an operator alert (feeding the Fan-out pattern for visibility).

**State Machine:** `pending → in_flight → delivered → expired`. Expired rows (older than 48h) are exported to `edge-service/systemd/offline-dump/` for manual upload.

**Failure Modes:**
- Disk full: uploader pauses and emits local warnings; apprentices swap SD cards per `docs/hardware-playbook.md`.
- Clock skew: uploader normalizes timestamps via GPS/ntp offsets before transmission.
- Power loss: systemd unit restarts the service; WAL ensures no corruption.

**Scalability:** Each edge node remains self-contained; central services only see deduplicated batches tagged by `sensor_id` + timestamp to avoid double counting.

### Apprentice Co-Development Loop

**Purpose:** Support 15-25 apprentices contributing safely across hardware, API, and web surfaces without regressing production quality.

**Pattern Elements:**
- **Workspace Seeds:** `docs/hardware-playbook.md`, `docs/api-reference.md`, and `docs/stories/<epic>/` give scripted exercises tied to epics.
- **Guard Rails:** GitHub Actions enforce 70% API coverage, 50% edge coverage, linting, and docs updates before merge.
- **Handoff Checklist:** Each epic requires schema diffs, UI screenshots, and alert scenarios stored under `docs/stories/<epic>/artifacts/`.

```
[Apprentice Pair]
    │ plan + document
    ▼
[docs/stories/<epic>/] ──> [Implementation (api/, web/, edge-service/)]
                                 │
                                 ▼
                        [CI + Validation]
                                 │
                                 ▼
                     [Fly.io Deploy / Release]
```

**Edge Cases:**
- Missing documentation triggers CI failure (docs lint job) so every change ships with updated README or runbook entries.
- If CI is flaky, apprentices run `make verify` locally (future script) before re-queuing jobs.

**Scalability:** Pods of apprentices clone the same story templates; as cohorts expand, new pods can spin up by copying `docs/stories/template/` and maintaining separation via branch naming conventions.

---

## Data Architecture

### Core Schema: Readings Table

```sql
CREATE TABLE readings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sensor_id VARCHAR(50) NOT NULL,
  timestamp TIMESTAMPTZ NOT NULL,

  -- Geospatial (PostGIS)
  location GEOGRAPHY(POINT, 4326) NOT NULL,

  -- Gas concentrations (ppb - parts per billion)
  h2s_ppb NUMERIC(10,2),    -- Hydrogen sulfide
  ch4_ppb NUMERIC(10,2),    -- Methane
  so2_ppb NUMERIC(10,2),    -- Sulfur dioxide (Growth phase)
  no2_ppb NUMERIC(10,2),    -- Nitrogen dioxide (Growth phase)
  o3_ppb NUMERIC(10,2),     -- Ozone (Growth phase)

  -- Metadata
  calibration_date TIMESTAMPTZ NOT NULL,
  sensor_certification VARCHAR(20) DEFAULT 'none',
  qa_qc_flags JSONB,
  legal_disclaimer TEXT DEFAULT 'Community sensor data for awareness, not enforcement. Not EPA-certified.',

  -- Computed/aggregated values
  rolling_avg_1h NUMERIC(10,2),
  rolling_avg_24h NUMERIC(10,2),

  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Convert to TimescaleDB hypertable (7-day chunks)
SELECT create_hypertable('readings', 'timestamp',
  chunk_time_interval => INTERVAL '7 days');

-- Add space partitioning for future scaling
SELECT add_dimension('readings', 'sensor_id',
  number_partitions => 4);

-- Indexing strategy
CREATE INDEX idx_sensor_time ON readings (sensor_id, timestamp DESC);
CREATE INDEX idx_location ON readings USING GIST(location);
CREATE INDEX idx_time_geo ON readings USING GIST(timestamp, location);
CREATE INDEX idx_alerts ON readings (timestamp) WHERE h2s_ppb > 10 OR ch4_ppb > 1000;

-- Compression policy (compress data older than 7 days)
SELECT add_compression_policy('readings', INTERVAL '7 days');

-- Retention policy (drop data older than 90 days)
SELECT add_retention_policy('readings', INTERVAL '90 days');
```

### Continuous Aggregates (Pre-computed Rollups)

```sql
-- Hourly aggregates for web experience performance
CREATE MATERIALIZED VIEW readings_hourly
WITH (timescaledb.continuous) AS
SELECT
  sensor_id,
  time_bucket('1 hour', timestamp) AS hour,
  AVG(h2s_ppb) AS h2s_avg,
  MAX(h2s_ppb) AS h2s_max,
  MIN(h2s_ppb) AS h2s_min,
  AVG(ch4_ppb) AS ch4_avg,
  MAX(ch4_ppb) AS ch4_max,
  MIN(ch4_ppb) AS ch4_min
FROM readings
GROUP BY sensor_id, hour;

-- Refresh policy: update every hour
SELECT add_continuous_aggregate_policy('readings_hourly',
  start_offset => INTERVAL '3 hours',
  end_offset => INTERVAL '1 hour',
  schedule_interval => INTERVAL '1 hour');
```

### Sensor Registry Schema

```sql
CREATE TABLE sensors (
  sensor_id VARCHAR(50) PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  location GEOGRAPHY(POINT, 4326) NOT NULL,
  zone VARCHAR(100),

  -- Hardware details
  manufacturer VARCHAR(100),
  model VARCHAR(100),

  -- Operational metadata
  deployment_date TIMESTAMPTZ NOT NULL,
  calibration_date TIMESTAMPTZ NOT NULL,
  last_seen TIMESTAMPTZ,
  status VARCHAR(20) DEFAULT 'operational',

  -- Certification & compliance
  certification_status VARCHAR(20) DEFAULT 'none',
  qa_qc_notes TEXT,

  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_sensors_location ON sensors USING GIST(location);
CREATE INDEX idx_sensors_zone ON sensors (zone);
CREATE INDEX idx_sensors_status ON sensors (status);
```

### Authentication & Team Schema

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email CITEXT UNIQUE NOT NULL,
  password_hash BYTEA NOT NULL,
  activated_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE teams (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE team_users (
  team_id UUID REFERENCES teams(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  role TEXT CHECK (role IN ('owner','maintainer','viewer')),
  PRIMARY KEY (team_id, user_id)
);

CREATE TABLE team_devices (
  team_id UUID REFERENCES teams(id) ON DELETE CASCADE,
  sensor_id VARCHAR(50) REFERENCES sensors(sensor_id) ON DELETE CASCADE,
  PRIMARY KEY (team_id, sensor_id)
);

CREATE TABLE device_api_keys (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sensor_id VARCHAR(50) REFERENCES sensors(sensor_id) ON DELETE CASCADE,
  secret_hash BYTEA NOT NULL,
  last_used_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE sessions (
  token CHAR(64) PRIMARY KEY,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  csrf_token CHAR(64) NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE tokens (
  token CHAR(64) PRIMARY KEY,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  scope TEXT CHECK (scope IN ('activation','password_reset','developer_api')),
  secret_hash BYTEA NOT NULL,
  metadata JSONB DEFAULT '{}',
  last_used_at TIMESTAMPTZ,
  expires_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

## API Contracts

### REST Endpoints

**Base URL:** `https://api.aethernet.app` (production), `http://localhost:8080` (development)

**Versioning:** `/v1/...` (URI-based, backwards compatible within major version)

**Authentication:** `Authorization: Bearer <api_key>` header

---

#### POST /v1/readings

**Purpose:** Batch ingestion from IoT edge devices

**Authentication:** Device API key (write-only scope)

**Request:**

```json
{
  "sensor_id": "sensor-001",
  "batch": [
    {
      "timestamp": "2026-02-15T14:00:00Z",
      "location": { "lat": 29.9511, "lon": -90.0701 },
      "readings": { "h2s_ppb": 15.2, "ch4_ppb": 450.3 },
      "metadata": {
        "calibration_date": "2026-02-01T00:00:00Z",
        "sensor_status": "operational"
      }
    }
  ]
}
```

**Response (201 Created):**

```json
{
  "data": {
    "receipt_id": "rcpt_abc123",
    "accepted": 12,
    "rejected": 0
  },
  "meta": {
    "timestamp": "2026-02-15T14:30:00Z",
    "request_id": "req_xyz789"
  }
}
```

---

#### GET /v1/readings/latest

**Purpose:** Fetch most recent readings from specified sensors or geographic area

**Query Parameters:**

- `sensor_ids` (comma-separated) OR `location` (lat,lon,radius_km) OR `zone`
- `gases` (optional filter: h2s,ch4,so2)

**Response (200 OK):**

```json
{
  "data": [
    {
      "sensor_id": "sensor-001",
      "timestamp": "2026-02-15T14:25:00Z",
      "location": { "lat": 29.9511, "lon": -90.0701 },
      "readings": { "h2s_ppb": 15.2 },
      "status": "normal",
      "stale": false,
      "metadata": {
        "certification_status": "none",
        "legal_disclaimer": "Community sensor data for awareness, not enforcement."
      }
    }
  ],
  "meta": {
    "count": 1,
    "timestamp": "2026-02-15T14:30:00Z",
    "request_id": "req_abc123"
  }
}
```

---

#### GET /v1/readings/timeseries

**Purpose:** Retrieve historical data for trend analysis

**Query Parameters:**

- `sensor_ids` OR `location` OR `zone` (required)
- `start_time`, `end_time` (ISO 8601, max 30-day window)
- `interval` (optional: raw, 15min, hourly, daily)

---

#### GET /v1/readings/alerts

**Purpose:** Query readings that exceeded thresholds

**Query Parameters:**

- `location` OR `zone` (required)
- `severity` (warning, hazardous)

---

#### GET /v1/sensors

**Purpose:** List all active sensors

---

#### GET /v1/sensors/{id}

**Purpose:** Detailed information about specific sensor

---

#### GET /health

**Purpose:** Health check for monitoring

**Response (200 OK):**

```json
{
  "status": "healthy",
  "checks": {
    "database": "ok",
    "disk_space": "ok"
  },
  "timestamp": "2026-02-15T14:30:00Z"
}
```

---

### Error Responses

**Standard Format:**

```json
{
  "error": {
    "code": "predefined_error_code",
    "message": "Human-readable error message",
    "details": {}
  },
  "meta": {
    "timestamp": "2026-02-15T14:30:00Z",
    "request_id": "req_abc123"
  }
}
```

**Predefined Error Codes:**

- `invalid_request` - Malformed JSON or missing fields
- `invalid_timestamp` - Timestamp out of range
- `invalid_location` - GPS coordinates invalid
- `invalid_timerange` - Time window exceeds 30 days
- `sensor_not_found` - Sensor ID not registered
- `unauthorized` - Missing/invalid authentication
- `forbidden` - Valid auth, insufficient permissions
- `rate_limited` - Rate limit exceeded
- `database_error` - Server-side database failure
- `internal_error` - Unexpected server error

**HTTP Status Codes:**

- `200 OK`, `201 Created`, `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `429 Too Many Requests`, `500 Internal Server Error`, `503 Service Unavailable`

---

## Security Architecture

### Authentication & Authorization

**Decision:** Implement in-house, stateful authentication immediately using bcrypt-hashed credentials, Go session middleware, and SvelteKit-aware cookies while preserving the ADR-009 exit path to a hosted IdP.

**User Login Flow (Go 1.23.1 + `golang.org/x/crypto/bcrypt` v4.0.1):**

1. `POST /v1/auth/login` validates email/password against the `users` table (bcrypt cost=12, per-hash salt).
2. On success, the API creates a `sessions` row with `id`, `user_id`, `csrf_token`, `expires_at`, `roles`, and `team_ids` columns.
3. Response sets an `HttpOnly`, `Secure`, `SameSite=Lax` cookie `aethernet_session=<opaque-token>` signed with HMAC-SHA256 using `SESSION_SIGNING_KEY`.
4. SvelteKit loads the cookie in `hooks.server.ts`, calls `GET /v1/auth/session`, and injects the resolved user + team context into `locals` for downstream routes.
5. Logout (manual or idle timeout) deletes the session row and instructs the browser to clear the cookie.

**Session Policy & States:**

- Idle timeout: 12 hours; absolute lifetime: 7 days. Renewal occurs on requests after 30 minutes of activity.
- State transitions: `provisioned → active → suspended → revoked`. Suspended sessions remain in the table but are excluded by middleware.
- CSRF tokens are stored per session and required on mutating requests (Go middleware enforces header `X-CSRF-Token` for browser contexts).

**Authorization:**

- RBAC roles: `admin`, `team:owner`, `team:maintainer`, `team:viewer`, `device`.
- Middleware loads the session, hydrates `contexts.TeamMemberships`, and checks handler-specific policies (e.g., only owners can rotate device keys).
- Admins bypass team scoping but still require explicit device grants for destructive actions.

**Developer & Device API Keys:**

- Developer keys live in `tokens` with `scope='developer_api'`, SHA-256 + pepper hashing, optional metadata to restrict teams or actions, and 90-day rotation reminders.
- Device ingestion keys live in `device_api_keys` (one per sensor) and are flashed via the hardware provisioning scripts. Edge agents send `Authorization: Bearer <device_key>`; API middleware verifies hash, enforces IP/rate heuristics, and injects the owning team ID.
- Rotation occurs quarterly or on compromise; dual-key mode (old + new) prevents ingestion gaps.

**SvelteKit Integration:**

- `hooks.server.ts` reads `aethernet_session`, calls the session endpoint, and redirects to `/login` on invalid/expired tokens.
- Client-side fetch helpers automatically include CSRF tokens and attach the latest `request_id` for observability.

**Future Hosted IdP Path:** Encapsulate auth logic behind an interface; when moving to Zitadel/Auth0, swap the credential issuance portion while retaining session + team enforcement code. A migration checklist lives in ADR-009.

---

### Data Security

**Encryption in Transit:**

- All API communication over HTTPS with TLS 1.3 (Fly.io automatic)
- HTTP requests automatically upgraded to HTTPS

**Encryption at Rest:**

- Database encryption via Fly.io Managed Postgres
- API keys hashed with SHA-256 + salt
- Secrets stored in Fly.io secrets manager

**Input Validation:**

- SQL injection prevention via prepared statements
- GPS coordinates validated against reasonable bounds
- Gas concentration values validated against sensor limits

---

### Rate Limiting & DoS Protection

**Implementation:** In-memory token bucket algorithm

**Tiers:**

- **Public:** 100 requests/hour per IP
- **Partner:** 1000 requests/hour per key
- **Device:** Unlimited (batch max 1000 readings)
- **Admin:** 1000 requests/hour per key

**Growth Migration:** Redis-based rate limiting when deploying multi-region

---

### Security Headers

**Standard Headers:**

```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

**CORS Configuration:**

- Whitelist: `https://dashboard.aethernet.app`, `http://localhost:5173`
- Methods: GET, POST, OPTIONS
- Headers: Authorization, Content-Type

---

## Performance Considerations

### API Performance Targets

**From PRD:**

- Regional queries (50 sensors): <2s (95th percentile)
- Single sensor latest: <500ms (95th percentile)
- Historical (7-day, 10 sensors, hourly): <3s

**Optimization Strategies:**

1. TimescaleDB continuous aggregates
2. PostGIS spatial indexes
3. Connection pooling (max 20 connections)
4. Query timeout (10 seconds)

---

### Web Performance

**From PRD:**

- Initial page load: <3s on 3G
- First meaningful paint: <1s
- Interactive: <2s

**Optimization Strategies:**

1. Bundle size: <500KB
2. Server-Side Rendering (SSR)
3. Progressive enhancement
4. API caching (1-minute client-side)

---

## Deployment Architecture

### Fly.io Infrastructure

**Region:** `iad` (Ashburn, Virginia)

**API Service:**

- App: `aethernet-api`
- VM: `shared-cpu-1x` (256MB)
- Cost: ~$5-10/month

**Web Service:**

- App: `aethernet-web`
- VM: `shared-cpu-1x` (256MB)
- Cost: ~$5-10/month

**Database:**

- Service: Fly.io Managed Postgres
- Plan: Basic (MVP) → Launch (Growth)
- Storage: 10GB MVP
- Cost: ~$2-10/month MVP, ~$82-100/month Growth

**Total MVP Cost:** ~$12-30/month

---

### CI/CD Pipeline

**GitHub Actions:**

- `aethernet-core`: API tests, web tests, auto-deploy on merge to `main`, path-based triggers, coverage gates (70% API, unit tests web app)
- `aethernet-edge-firmware`: build + hardware regression checks on tagged releases; publish firmware binaries/toolchains without auto-deploy

---

### Secrets Management

**Local Development (`.env`):**

```bash
DATABASE_URL=postgres://localhost/aethernet_dev
API_KEY_SECRET=dev-secret-key
LOG_LEVEL=debug
```

**Production (Fly.io Secrets):**

```bash
flyctl secrets set -a aethernet-api \
  DATABASE_URL=postgres://... \
  API_KEY_SECRET=<generated> \
  LOG_LEVEL=info
```

---

## Implementation Patterns

### Naming Conventions

**API Endpoints:**

- `POST /v1/readings` (plural, lowercase)
- `GET /v1/sensors/{id}` (plural resource, `{param}`)

**Database:**

- `readings` (plural, snake_case)
- `sensor_id` (snake_case columns)

**Go Code:**

- `type Reading struct` (PascalCase types)
- `func HandleReadings` (PascalCase functions)
- `requestID` (camelCase variables)

**TypeScript:**

- `interface Reading` (PascalCase types)
- `function fetchReadings` (camelCase functions)

**Files:**

- Go: `readings.go`, `readings_test.go`
- Svelte: `SensorMap.svelte`
- Python: `serial_reader.py`
- Firmware adapters: `firmware/adapters/<board>/<sensor>.ino`

**Python Edge Services:**

- Modules live under `edge-service/src/` with snake_case filenames (`buffer_store.py`, `fanout_bridge.py`).
- Shared helpers go in `edge-service/src/common/` to prevent circular imports.
- Tests mirror module names under `edge-service/tests/test_<module>.py`.

**Firmware Naming:**

- PlatformIO-style folders per board, sketches using PascalCase (`CommunitySensor.ino`).
- Adapter-specific configs sit in `firmware/adapters/<board>/config.json`.

---

### Error Handling Pattern

```go
func HandleReadings(w http.ResponseWriter, r *http.Request) {
    requestID := getRequestID(r)

    // 1. Parse & validate input
    var payload BatchPayload
    if err := json.NewDecoder(r.Body).Decode(&payload); err != nil {
        writeError(w, "invalid_request", "Invalid JSON", 400, requestID)
        return
    }

    // 2. Validate business rules
    if err := validateTimestamps(payload); err != nil {
        writeError(w, "invalid_timestamp", err.Error(), 400, requestID)
        return
    }

    // 3. Database operation
    if err := db.InsertReadings(payload); err != nil {
        log.Error("insert failed", "request_id", requestID, "error", err)
        writeError(w, "database_error", "Unable to store", 500, requestID)
        return
    }

    // 4. Success
    writeSuccess(w, data, 201, requestID)
}
```

---

### Logging Pattern

```go
log.Info("reading ingested",
    "request_id", requestID,
    "sensor_id", "sensor-001",
    "count", 12)
```

**Levels:** ERROR, WARN, INFO, DEBUG

---

### Cross-Cutting Standards

**Every AI agent MUST:**

1. Log structured JSON with `request_id`
2. Return envelope responses
3. Use UTC timestamps (RFC3339)
4. Include security headers
5. Handle panics gracefully
6. Include SB 503 disclaimers
7. Store secrets in env vars
8. Expose `/health` endpoint

---

### Lifecycle & Background Patterns

- **Alert Fan-out:** Follow `detected → queued → delivered → acknowledged`; retries at 1m/5m/15m before surfacing to ops.
- **Edge Sync:** `pending → in_flight → delivered → expired`; expired data exported for manual recovery.
- **Apprentice Workflow:** `planned → in_progress → code_review → validated`; CI enforces progression by blocking merges lacking artifacts.
- Background jobs run hourly via Fly Machines (or `flyctl scale` workers) and must log `job_id`, `pattern`, and `result`.

---

## Architecture Decision Records

### ADR-001: Go Standard Library Over Framework

**Status:** Accepted
**Date:** 2025-11-06

**Decision:** Use Go standard library (net/http) with 1.22+ enhanced routing.

**Rationale:**

- Go 1.22+ includes method matching and wildcards
- Zero external dependencies
- Apprentices learn foundational HTTP concepts
- Standard library is stable and backwards-compatible

---

### ADR-002: TimescaleDB + PostGIS

**Status:** Accepted
**Date:** 2025-11-06

**Decision:** PostgreSQL 16 with TimescaleDB 2.23.0 and PostGIS.

**Rationale:**

- Automatic chunk management, compression
- Native geospatial queries (radius, zones)
- Familiar SQL interface
- Managed by Fly.io

---

### ADR-003: SQLite Edge Buffering

**Status:** Accepted
**Date:** 2025-11-06

**Decision:** SQLite for local buffering with Python upload service.

**Rationale:**

- ACID compliance prevents data corruption
- Retry logic simplicity via SQL queries
- Student-friendly debugging
- <500KB for 48-hour buffer

---

### ADR-004: In-Memory Rate Limiting (MVP) → Redis (Growth)

**Status:** Accepted
**Date:** 2025-11-06

**Decision:** In-memory token bucket for MVP, Redis for Growth.

**Rationale:**

- Zero dependencies for MVP
- 13-17M ops/sec performance
- Clean migration path to Redis

---

### ADR-005: SvelteKit Over Next.js

**Status:** Accepted
**Date:** 2025-11-06

**Decision:** SvelteKit 2.x with Sveaflet.

**Rationale:**

- <500KB bundle size
- Not locked to Vercel
- Modern DX for apprentices
- SSR for low-bandwidth users

---

### ADR-006: Monorepo

**Status:** Accepted
**Date:** 2025-11-06

**Decision:** Single monorepo with component directories.

**Rationale:**

- Easier apprentice onboarding
- Atomic cross-component changes
- Shared documentation

---

### ADR-007: In-House Stateful Authentication

**Status:** Accepted
**Date:** 2025-11-07

**Decision:** Ship Go-based authentication immediately using bcrypt-hashed credentials, stateful session cookies, per-session CSRF tokens, and team/device API keys managed in Postgres while keeping an exit path to a hosted IdP (see ADR-009).

**Rationale:**

- Unblocks Epic 5 security scope without waiting on external tooling.
- Keeps apprentices close to the auth implementation for learning and rapid iteration.
- Session cookies + server-side storage align with Fly.io constraints and simplify SvelteKit integration.
- Device and developer API keys share hashing, rotation, and auditing infrastructure.

---

### ADR-008: Split Edge/Firmware Repo from Core

**Status:** Accepted
**Date:** 2025-11-06

**Decision:** Move the edge gateway runtime and MCU/SoC firmware into a dedicated `aethernet-edge-firmware` repository while keeping the Go API, web experience, shared docs, and database migrations inside `aethernet-core`.

**Rationale:**

- Hardware contributors no longer need to clone server/web dependencies when building DIY sensors.
- Allows firmware/toolchain releases to be tagged and documented at their own cadence without triggering production deployments.
- Reduces cross-toolchain conflicts (Go, Node, Python, PlatformIO) within a single repo.

**Implementation Notes:**

- Create `aethernet-edge-firmware` via subtree split, preserving history for `edge/` and `arduino/` directories.
- Update `aethernet-core` CI to drop hardware paths and reference the new repo in docs.
- Publish GitHub Releases from `aethernet-edge-firmware` containing binaries/toolchains for workshops and maker kits.

---

### ADR-009: MVP User Auth In-House, Plan for Hosted IdP

**Status:** Draft
**Date:** 2025-11-06

**Decision:** Implement the initial user-authentication stack inside `aethernet-core` with stateful sessions (server-side tokens + signed cookies), password reset flows, and per-team access control. Model devices under teams/projects so users gain privileges by belonging to a team, and define an exit path to a hosted IdP (e.g., Zitadel/Auth0) once external tenant needs or compliance triggers arise.

**Rationale:**

- Keeps MVP lightweight: no extra infrastructure, faster onboarding for apprentices who can read/modify the full auth flow.
- Aligns with current scope (internal admins + team-based device managers) where storing credentials in our Postgres DB is manageable.
- Builds foundational understanding of secure password handling, email confirmation, and session management before outsourcing.
- Introduces the team/ownership concept early so device permissions map cleanly to future requirements.
- Provides a clean migration story by issuing JWTs and normalizing RBAC + team claims now, so a future IdP can replace the credential issuance step without rewriting downstream middleware.

**Exit Criteria for Hosted IdP:**

- Need for external organization self-service (community cohorts, device operators).
- Requirement for advanced auth factors (WebAuthn, enterprise SSO) or compliance mandates we don’t want to implement ourselves.
- Team capacity constraints (no appetite to maintain password resets, audit logging, SCIM automation).

- Use bcrypt password hashing (per-hash salts inherent to bcrypt), per-user tokens for activation/reset, and a stateful session store (Postgres + signed cookies) carrying role + team identifiers.
- Model `teams`, `team_users`, and `team_devices` tables; enforce device CRUD through middleware that loads memberships from the session context.
- Issue developer API keys (hashed at rest) so automated tooling can authenticate without hijacking browser sessions; keys inherit the user’s team scopes.
- Provide short-lived API session tokens when authorized access is required (still backed by server state rather than self-contained JWTs).
- Encapsulate auth logic behind an interface so we can swap the provider with an IdP SDK later.
- Document the migration plan: introduce OIDC middleware, map IdP claims (org/projects) to team records, and disable local credential storage once adoption completes.

---

## Next Steps

### Immediate

1. **Resolve Authentication Design** (before Epic 5)
2. **Run Solutioning Gate Check**
3. **Sprint Planning** (Phase 4)

### Implementation Sequencing

**Phase 1 (Weeks 1-2): Epic 1**

- Hardware blueprint, Arduino firmware, Pi service
- Deploy 1-2 prototype sensors

**Phase 2 (Weeks 3-4): Epics 2-3**

- Database schema, ingestion endpoint, query endpoints
- OpenAPI specification

**Phase 3 (Weeks 5-6): Epics 4-5**

- Web app (dashboard, admin, device portal surfaces)
- Rate limiting, authentication, accessibility

**Phase 4 (Weeks 7-8): Epic 6 + Launch**

- Documentation, playbooks, examples
- Public launch

### Success Criteria

**Community:**

- ✅ 5-10 sensors operational
- ✅ Public API queries
- ✅ Accessible web experience
- ✅ Data transparency

**Workforce:**

- ✅ 15-25 apprentices participate
- ✅ Production code shipped
- ✅ Portfolio artifacts

**Technical:**

- ✅ Zero data loss (24-hour outage)
- ✅ 99.5% uptime
- ✅ <2s queries, <500ms single sensor
- ✅ 30-day historical access

---

_This architecture document was created through collaborative decision facilitation with OpSpark._

_Generated: 2025-11-06_
_For: OpSpark - Aethernet Project_
_Ready for: Sprint Planning and Implementation_
