# ADR-0005: MQTT Topic Structure for Louisianair Devices

## Status

Accepted

## Context

Louisianair devices publish sensor and status data over MQTT. Devices may vary in hardware and architecture (e.g. single MCUs like Pico W-class devices, MCU + companion systems, or future platforms). Devices may be owned by different users and connect through shared gateways or directly to a shared MQTT broker.

Devices can be deployed in arbitrary and changing physical locations. MQTT topics form a long-lived contract between device firmware, gateways, and backend services, so the topic structure must be stable, hardware-agnostic, and support multi-tenant isolation.

## Decision

Use a simplified, tenant-centric, device-oriented MQTT topic hierarchy. Physical location and device hardware type are not encoded in topics. Sensor identity and values are carried in message payloads.

The canonical topic structure is:

```
louisianair/<env>/<tenant>/<device_id>/<channel>
```

Where:
- `<env>` is a deployment environment (e.g. `prod`, `dev`)
- `<tenant>` is an opaque owner/account identifier
- `<device_id>` is a stable unique device identifier
- `<channel>` describes the message category

Devices publish the following channels:

| Topic          | Retained | Purpose                                        |
| -------------- | -------- | ---------------------------------------------- |
| `telemetry`    | no       | Periodic sensor readings                       |
| `state`        | yes      | Last known state (online/offline, last sample) |
| `event/<type>` | no       | Discrete events (boot, fault, calibration)     |
| `meta`         | yes      | Device metadata (fw version, hw rev, capabilities) |

Example topics:

- `louisianair/prod/t_8f3a/device-0017/telemetry`
- `louisianair/prod/t_8f3a/device-0017/state`
- `louisianair/prod/t_8f3a/device-0017/event/boot`
- `louisianair/prod/t_8f3a/device-0017/meta`

This structure enables clean multi-tenant ACLs on shared brokers and prevents cross-tenant interference. Excluding physical location avoids topic churn when devices move or when location is unknown at publish time. Excluding device hardware type keeps the topic contract stable as implementations evolve.

Bundling sensor data in payloads reduces publish overhead on constrained devices and avoids topic changes when sensors are added or removed.

## Alternatives Considered

### Location- or hardware-based topic hierarchy

Rejected. Encoding physical location or device type in topics causes churn when devices move or hardware evolves, and complicates multi-tenant isolation.

### Per-sensor topics

Rejected. Publishing each sensor on its own topic increases topic churn as sensors change, and adds overhead on constrained devices.

## Consequences

This structure provides a stable and extensible topic contract that supports device mobility, shared infrastructure, and future expansion (commands, configuration, fleet management). Consumers must parse payloads to extract individual metrics, and all telemetry for a device shares the same QoS and retention behavior.

## Related Decisions

- Builds on [ADR-0004](0004-protocol-mqtt.md) (MQTT protocol choice)
