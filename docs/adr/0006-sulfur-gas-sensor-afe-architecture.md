# ADR-0006: Sulfur Gas Sensor and AFE Architecture

## Status

Proposed

**Implementation update (October 23, 2026):** Five Smart Citizen 8-channel ASB PCBs have been ordered for a Pico W / Alphasense development build. That board uses two ADS1115 ADCs over I²C, not the SPI/ADS1220 interface proposed below. See the [October hardware update](../funder-reports/oct-2026.md) for the illustrated node, BOM, rationale, and validation gates. This ADR remains a proposal; its ADC performance assumptions and 2.54 mm interconnect specification are not a wiring guide for the ASB build, whose J5 connection uses 2 mm pitch.

## Context

The project requires outdoor air quality monitoring in urban and near-industrial environments (outside industrial fence lines), with a focus on sulfur gases.

Primary constraints:

- **High data quality and accuracy** suitable for environmental monitoring (ppb to low-ppm resolution, stable baseline)
- **Per-node sensor cost under ~$300**
- Outdoor operation with large temperature and humidity variation
- Networked, unattended deployments
- Ability to scale from an initial single-node design to small networks

Platform constraints:

- MCU: RP2040 (Pico W)
- Firmware: TinyGo
- Preference for robust, deterministic interfaces (SPI over I2C where possible)

ADC constraints:

- High resolution and low noise for small signal changes (ppb to low-ppm)
- Low sample rate is acceptable; prioritize stability over speed
- For production, the ideal target is at least 6 usable channels (WE/AE x2 + Pt1000 differential + spare)
- For near-term prototyping, breakout-board availability is a hard constraint
- The prototype may temporarily instrument only a subset of the ideal AFE signals
- SPI preferred for deterministic behavior and clearer failure modes than I2C
- TinyGo support or straightforward driver effort

## Decision

### Sensors

Use **Alphasense A-series 4-electrode electrochemical sensors** to meet accuracy and cost constraints.

Selected sensors:

- **SO2:** SO2-A4+ (or SO2-A41+ if mechanical constraints require)
- **H2S:** H2S-A4+ (or H2S-A41+ if mechanical constraints require)

Rationale:

- Electrochemical sensors provide true redox-current measurements with known sensitivity (nA/ppm)
- A-series sensors are optimized for low-level (ppb to low-ppm) environmental monitoring
- 4-electrode design (WE + AE) improves baseline stability and drift compensation
- \"+\" variants offer improved long-term stability
- Total sensor cost remains within the target budget

A41 variants are mechanically optimized but electrically identical to A4 variants.

### Context Gas

Use **NO2** as the primary context gas.

Rationale:

- NO2 is stable in ambient air and representative of urban atmospheric conditions
- Useful for interpreting sulfur sensor behavior and identifying interference
- Provides more actionable context than NO or CO for outdoor air quality monitoring
- Fits cleanly within Alphasense A-series AFE slot constraints

### AFE Architecture

Use **two identical 2-way Alphasense A-series AFEs**, each with:

- One **oxidizing gas slot** populated with NO2
- One **reducing gas slot** populated with a sulfur sensor

Configuration:

- **AFE 1:** NO2 + SO2
- **AFE 2:** NO2 + H2S

Rationale:

- 2-way A-series AFEs support only one reducing-gas channel
- SO2 and H2S cannot coexist on a single 2-way AFE
- Duplicating NO2 provides consistent context for both sulfur measurements
- Avoids larger, higher-cost 3-way AFEs

### AFE-MCU Interface

- AFE outputs are **analog voltages**
- Working Electrode (WE) and Auxiliary Electrode (AE) outputs are read as **single-ended ADC inputs**
- Onboard Pt1000 temperature sensor is exposed as **Pt1000+ / Pt1000-** and read **differentially**

No digital protocol (I2C/SPI) is used between the AFE and MCU.

### ADC

Use an **external SPI delta-sigma ADC**.

Primary choice:

- **ADS1220** (one ADC per AFE for initial prototyping)

Rationale:

- Low-SPS, long-interval sampling lets averaging suppress random noise (e.g., 20 SPS over 30 s = 600 samples, ~24.5x noise reduction)
- In this regime, effective resolution is dominated by input-referred noise and stability, not headline "bits"
- ADS1220 is a low-noise, 24-bit delta-sigma ADC with SPI, PGA, and a mux that supports either four single-ended inputs or two differential pairs
- MCP3564 was initially selected because it better matched the ideal channel-count target, and a breakout board could be found early in evaluation
- That path turned out to be less practical than expected because additional MCP3564 breakout boards were not readily obtainable
- Readily available ADS1220 breakout boards make bench prototyping materially easier than the previously selected MCP3564 path
- The most available alternative breakout boards found during selection were either single-ended only or too low resolution for this use
- For prototype work, breakout availability is more important than preserving the ideal channel count on day one
- SPI is robust and well-supported in TinyGo
- RP2040 internal ADC lacks sufficient channel count and effective resolution

Each ADS1220 services one AFE prototype path:

- One differential pair is used for the selected gas-sensor channel on that AFE
- The second differential pair is used for the AFE Pt1000 measurement
- This leaves the other gas-sensor channel on that AFE uninstrumented in the initial prototype
- A later revision may add another ADC to recover the deferred gas-sensor channel

### Interconnect

- **10-pin IDC ribbon cables**
- 2.54 mm (0.1") pitch, 2x5 connectors
- IDC breakouts or mating headers on the MCU carrier PCB

Rationale:

- Standard interface used by Alphasense A-series AFEs
- Simple, robust, and well-documented

### Humidity and Ambient Temperature

Add a **separate digital RH/T sensor** (e.g. Sensirion SHT3x or SHT4x) via I2C.

Rationale:

- AFE Pt1000 measures sensor temperature only
- Relative humidity is required to interpret sulfur sensor artifacts
- DHT-11-class sensors are insufficient for outdoor air quality monitoring

## Consequences

### Positive

- Meets accuracy and stability goals within cost constraints
- Architecture scales cleanly from 1 to 2 AFEs
- Provides necessary atmospheric context (NO2, RH, temperature)
- Uses a low-noise ADC available on readily obtainable breakout boards, reducing prototype friction
- Deterministic firmware behavior with SPI-based ADCs

### Negative

- Higher BOM cost than MQ/MOX-based designs
- ADC channel count is tighter than the original ideal
- Initial prototype cannot instrument every gas-sensor channel from each AFE at once
- Recovering the deferred gas-sensor channel likely requires an additional ADC in a later revision
- Redundant NO2 sensors across AFEs
- Requires a new TinyGo ADS1220 driver

This architecture prioritizes data quality, interpretability, and long-term stability while remaining within a realistic per-node cost target. It is intended as a robust baseline for v1 deployments and future network expansion.

## Alternatives Considered

### Metal-oxide (MOX) sensors (MQ-x class)

Rejected. Used as a low-cost prototype exploration path, but MOX sensors have poor selectivity in mixed gases, require elevated operating temperatures with heater power, and exhibit strong temperature/humidity dependence that can drive response drift. These trade-offs conflict with stability and power goals for outdoor monitoring.

### Electrochemical VOC sensors (combo VOC + sulfur)

Rejected. VOC sensors were considered for broader atmospheric context, but combining VOC outputs with sulfur targets adds calibration and interpretation complexity beyond the current project scope.

### SGX Sensortech EC4 H2S/SO2 sensors

Rejected. The EC4 H2S and SO2 variants target ppm-scale ranges and resolution (e.g., 0-1000 ppm with 1 ppm resolution for H2S; 0-2000 ppm with 5 ppm resolution for SO2), which does not align with ppb-level targets for environmental monitoring.

### MCP3564 as prototype ADC

Rejected for the current prototype iteration. MCP3564 remained attractive on technical grounds because its channel count better fits the full AFE signal set, but breakout-board availability was worse than expected in practice. One board could be sourced during evaluation, but additional boards were not readily available, which made it a weak choice for a repeatable prototype path.
