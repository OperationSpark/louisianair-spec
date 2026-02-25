# Brainstorming Session Results

**Session Date:** 2025-11-04
**Facilitator:** Business Analyst Mary
**Participant:** OpSpark

## Executive Summary

**Topic:** Designing the Aethernet sensor-to-cloud data pipeline

**Session Goals:** Identify feasible technical pathways to convert raw sensor readings into calibrated concentration values, buffer them locally, and deliver reliable batches to the cloud API despite limited embedded-hardware experience.

**Techniques Used:** Morphological Analysis (deep system decomposition)

**Total Ideas Generated:** 5 idea clusters captured across immediate, future, and moonshot categories.

### Key Themes Identified:

- Commitment to a spike-first approach: settle on one reference architecture, gather real-world data, then iterate based on empirical results instead of debating hypotheticals.
- Reliance on field-ready documentation: DFrobot sensor materials and similar vendor guides anchor decision making because in-house hardware expertise is limited.
- Progressive hardening mindset: start with raw sampling and CSV storage, but keep future pathways open for reliability upgrades (watchdogs, structured queues, database migration).

## Technique Sessions

### Technique 1: Morphological Analysis

- Parameter A (Sensor interface layer): DFrobot Gravity I²C module | UART module (0–3 V) | Analog output via external ADC
- Parameter B (Microcontroller / edge processor): Arduino Nano Every | ESP32 DevKit | Raspberry Pi Pico
- Parameter C (Handoff to Pi): USB serial | GPIO UART | I²C bridge (Pi polls MCU)
- Parameter D (Local processing & storage): Append-only CSV | SQLite ingest | Lightweight time-series store
- Parameter E (Batch upload transport): Scheduled HTTPS POST | MQTT client | WebSocket stream
- Parameter F (Reliability measures): Local backlog + retry | Dual watchdogs | Calibration checksum + drift alert

## Idea Categorization

### Immediate Opportunities

_Ideas ready to implement now_

- Build a spike around Combination 1: DFrobot Gravity gas sensor over I²C feeding an Arduino Nano Every, forwarding raw samples via USB serial to the Raspberry Pi, appending to a single CSV with schema validation, and batching hourly HTTPS uploads with a local retry queue.
- Use the DFrobot reference documentation as the primary guide for sensor wiring, calibration, and acceptable operating conditions: https://wiki.dfrobot.com/SKU_SEN0465toSEN0476_Gravity_Gas_Sensor_Calibrated_I2C_UART

### Future Innovations

_Ideas requiring development/research_

- Determine optimal sampling cadence and whether pre-processing (averaging, filtering) should live on the microcontroller or the Pi once raw data characteristics are understood.
- Evaluate migration from flat CSV to a lightweight database or structured queue file once batching and backlog behaviors are characterized in the field.
- Design and test weatherproof enclosures with proper venting for home exterior installs, including power management (mains vs battery backup) and physical strain relief for USB connections.

### Moonshots

_Ambitious, transformative concepts_

- Extend the edge stack to support multi-sensor hubs with automated calibration drift detection, self-healing watchdog routines, and richer telemetry for maintenance insights.

### Insights and Learnings

_Key realizations from the session_

- Accepting uncertainty: acknowledging “cloudy” areas reframes the goal toward building confidence through experiments rather than over-analysis.
- Documentation-first grounding: linking to a definitive sensor guide gives the team a concrete starting point and reduces ambiguity in wiring, power, and calibration choices.
- Deferred complexity: intentionally postponing DB migrations, alerting, or multi-sensor orchestration keeps the focus on data integrity basics and throughput validation.

## Action Planning

### Top 3 Priority Ideas

#### #1 Priority: Map MVP Component Stack

- Rationale: Ensure the entire team shares a concrete understanding of what the MVP truly requires so workstreams stay aligned.
- Next steps: Draft the overarching component list (sensor hardware, edge processing, ingestion APIs, storage, analytics, admin tools) and validate with stakeholders.
- Resources needed: To be determined after initial draft (likely core engineering leads + partner input).
- Timeline: Complete by end of this week.

#### #2 Priority: Sensor-to-Cloud Pipeline Spike

- Rationale: Validate the sensing, local storage, and cloud upload loop so we can smooth the student UX/DX using real-world constraints instead of assumptions.
- Next steps: Assemble the DFrobot sensor + Arduino Nano + Pi prototype, log raw samples locally, push batches to the cloud API, document pain points for future tooling.
- Resources needed: Sensor hardware kit, Raspberry Pi (4/5), development time from IoT lead, temporary cloud endpoint for testing.
- Timeline: Execute over the next few weeks (target completion within current sprint window).

#### #3 Priority: Define External Data API Contracts

- Rationale: Grant requirement demands a public consumption API aligned with partner expectations; getting the contracts right ensures data usability and trust.
- Next steps: Review existing environmental data APIs for best practices, capture required fields (GPS, ppm, composition, timestamps, etc.), consult LEAD partners for priority metrics, and draft the contract + architecture.
- Resources needed: Access to reference APIs, input from LEAD/community partners, coordination with data/backend leads.
- Timeline: Develop contracts and architecture over the next few weeks in parallel with the spike.

## Reflection and Follow-up

### What Worked Well

Capturing a morphological matrix kept the discussion grounded in concrete architectural components without committing prematurely.
Referencing the DFrobot documentation early provided a shared baseline for sensor wiring and calibration, reducing ambiguity.
Having a structured flow (generation → convergence → insights) made it easier to acknowledge uncertainties while still landing actionable decisions.

### Areas for Further Exploration

Validate sampling cadence, noise behavior, and calibration routines once the prototype hardware is assembled.
Benchmark SQLite versus append-only CSV for durability and operational ergonomics on the Raspberry Pi.
Research proven enclosure designs from community air-monitoring projects to inform weatherproofing and maintenance plans.

### Recommended Follow-up Techniques

Revisit Morphological Analysis when new hardware components or communication protocols enter the design space.
Use Question Storming in future sessions to surface knowledge gaps around calibration, cloud ingestion resilience, and community upkeep.
Apply Failure Mode Analysis after the spike to anticipate reliability pitfalls in field deployments.

### Questions That Emerged

What sampling frequency balances sensor noise, power consumption, and data fidelity for methane/H₂S monitoring?
How should “sent vs pending” batches be tracked once we migrate from CSV to SQLite?
What environmental protections (venting, filters, heating) keep gas sensors accurate in Gulf Coast humidity?

### Next Session Planning

- **Suggested topics:** Evaluate SQLite-backed batching prototype; design weatherproof enclosure; outline calibration workflow for community maintainers.
- **Recommended timeframe:** Reconvene after the initial hardware spike (target 2–3 weeks).
- **Preparation needed:** Gather spike findings (sensor logs, storage notes), enclosure sketches/photos, and questions from pilot community partners.

---

_Session facilitated using the BMAD CIS brainstorming framework_
