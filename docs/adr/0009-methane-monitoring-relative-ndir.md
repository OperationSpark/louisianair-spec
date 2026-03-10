# ADR-0009: Methane Monitoring via Relative NDIR Enhancements with Meteorological Context

## Status

Proposed

## Context

The project needs a methane monitoring approach that is credible for community and fence-line deployments without pretending low-cost nodes provide laboratory-grade ambient CH4 truth.

Key constraints:

- Outdoor methane monitoring is strongly affected by temperature, humidity, airflow, and plume transport
- The project needs unattended, networked nodes that can be deployed across multiple community sites
- Per-node hardware must remain materially cheaper and simpler than regulatory-grade analyzers
- The system must produce defensible signals that support pattern detection, community reporting, and follow-on investigation

Key credibility risks:

- Claiming high absolute accuracy from a low-cost methane node
- Treating a single methane sensor as ground truth
- Publishing raw CH4 numbers with no local baseline or wind context
- Using generic reducing-gas MOX signals and labeling them as methane

For this project, methane value comes primarily from detecting repeated relative enhancements and correlating them with wind direction, time of day, and site activity, not from asserting highly accurate absolute background concentration.

## Decision

Adopt a **relative-enhancement methane monitoring architecture** built around a **mid-tier NDIR methane sensor module** plus required environmental context sensors.

### Methane Sensor

Use an **NDIR CH4 module** that meets these minimum selection criteria:

- Stated resolution of **0.1 ppm or better**
- **Internal temperature compensation**
- **Continuous measurement** mode
- **Digital output** over UART or I2C

The methane channel is to be treated as an indicator of **local methane enhancement above baseline**, not as a regulatory or reference-grade absolute concentration measurement.

### Required Context Sensors

Each methane node must include:

- **Ambient temperature**
- **Relative humidity**
- **Wind speed**
- **Wind direction**

Pressure is recommended when practical, but is not mandatory for the first deployable design.

CO2 is recommended as a context gas for later revisions because it can help distinguish combustion-related events from fugitive methane leaks.

### Data Interpretation Model

Methane reporting must be based on **baseline-relative behavior**:

- Maintain a rolling local baseline window for each node
- Compute and store **delta methane** relative to that baseline
- Apply aggressive averaging and smoothing, typically in the **30 second to 5 minute** range
- Use cloud-side event detection that incorporates wind direction and cross-node agreement where available

The primary interpretation target is:

- repeated methane enhancements above local background
- associated with winds from a direction of interest
- observed over time, or across more than one node

### Product and Scientific Messaging

External product language must avoid framing methane output as precise ground-truth ambient concentration.

Preferred framing:

- "methane enhancement above local background"
- "repeated methane events associated with winds from the industrial corridor"
- "pattern detection for community monitoring and follow-up investigation"

Avoid:

- hard claims of high absolute CH4 accuracy
- single-node source attribution claims
- health-threshold claims based only on methane node output

## Consequences

### Positive

- Produces a more defensible methane story than low-cost absolute-number claims
- Fits a networked community-monitoring deployment model better than reference-instrument assumptions
- Wind correlation materially improves the interpretability of spikes and plume events
- Baseline-relative analytics are more robust to drift than raw-number dashboards
- Leaves room for future cross-node anomaly detection and event scoring

### Negative

- Small methane enhancements may still be missed
- False positives remain possible, especially with poor siting or weak wind data
- Useful pattern detection may require weeks of accumulated data
- Output is less intuitive than a single published "current methane concentration" number
- This decision constrains UI, API, and reporting language that currently implies direct absolute methane readings

## Mitigations

- Prefer deployment of multiple nodes over overclaiming single-node certainty
- Co-locate wind sensing on every methane node rather than treating meteorology as optional
- Surface baseline, delta, smoothing window, and confidence context in downstream data products
- Document uncertainty and drift limits explicitly in dashboards, reports, and public-facing materials
- Use cross-node comparison and repeated-event logic before elevating claims about persistent site behavior

## Alternatives Considered

### Absolute ambient methane concentration as the primary product output

Rejected for current scope. Low-cost distributed nodes are unlikely to sustain the credibility needed for strong absolute ambient CH4 claims without reference-grade hardware, calibration workflow, and tighter environmental control than this project targets.

### Single-node methane monitoring without wind instrumentation

Rejected. Without wind speed and direction, methane spikes are much harder to distinguish from drift, noise, or transient local artifacts, which weakens both scientific and community credibility.

### MOX or generic reducing-gas methane proxy sensors

Rejected. These sensors do not provide methane-specific enough behavior for a defensible CH4 claim in mixed outdoor air and are too vulnerable to humidity, temperature, and cross-gas interference.

### Reference-grade methane analyzers

Rejected for initial deployment. They offer stronger absolute performance but are outside the project's practical cost, power, and deployment envelope for a small community sensor network.

## Assumptions to Confirm

- Suitable mid-tier NDIR methane modules can be sourced at a cost compatible with node budgets
- A deployable wind sensor can be integrated without unacceptable power, maintenance, or enclosure penalties
- Downstream dashboards and APIs can represent methane as baseline-relative events rather than only raw concentration values
