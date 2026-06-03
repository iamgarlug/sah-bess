# Intelligent Controller — Architecture Overview

The Intelligent Controller (IC) is the microservice-based brain of SAH-BESS. It accepts commands from
**SCADA and custom interfaces**, controls field devices through **protocol abstractions over
DNP3/Modbus**, runs **onsite**, ages operational data to a **cloud data lake**, emits
**OpenTelemetry**, prefers **open-source libraries**, and **conforms to MESA** — governed by the
[IEEE/MESA standards register](../standards/ieee-register.md).

## Architecture style — hexagonal (ports & adapters)

**Decision:** the IC core uses hexagonal architecture. See [ADR-0001](../adr/0001-hexagonal-architecture.md).

**Why.** The durable asset is the **domain logic** — dispatch, the MESA-ESS operational state machine,
and IEEE 1547 grid-support functions — which must remain stable across a 10–20-year asset lifecycle.
Almost everything around it is volatile or undecided:

- the **DNP3 stack already churned** (opendnp3 went end-of-life; we move to stepfunc/dnp3),
- the **database / cache** is undecided,
- the **cloud backend** is deliberately vendor-neutral/undecided,
- **devices span many vendors** with differing register maps.

Hexagonal keeps the domain core dependency-free and pushes every volatile concern behind an interface
(a *port*) with swappable *adapters*. Swapping NModbus→another Modbus lib, TimescaleDB→InfluxDB, a real
device→Typhoon HIL, or onboarding a new vendor becomes an adapter change, not a core change. It also
makes the core unit-testable without hardware (serving the 80% coverage goal) and gives a clean seam to
attach regulatory-conformance indicators.

**Cost.** More projects and indirection up front — accepted given the lifecycle, the amount of
undecided infrastructure, and the multi-vendor / HIL requirements.

## Services

Each service is a .NET worker and/or minimal API.

| Service | Responsibility |
|---|---|
| **Gateway** | Ingress for **SCADA**, **custom**, and (future) **SEP2** commands (REST + gRPC); normalizes external requests into domain commands. *SEP2 = Smart Energy Profile 2.0 (IEEE 2030.5): an IP/HTTP(S) utility↔DER protocol, e.g. California Rule 21 / DERMS.* |
| **Dispatch** | Control/optimization orchestration; owns the **MESA-ESS operational state machine**; calls Analytics for model-driven decisions. |
| **DeviceGateway** | Protocol I/O plus the **multi-vendor DER driver registry** (below). |
| **Historian** | Operational time-series **sink** + tiered retention + cloud export. |
| **Analytics** | Invokes MATLAB models (**SoC/SoH**, forecasting, optimization) behind `IModelRunner`. *SoC = State of Charge (%, how full); SoH = State of Health (%, capacity/degradation vs. new).* |

## Ports (core interfaces, in `Sah.Ic.Abstractions`)

- `IDevicePort` — read/write to a field device, protocol-agnostic.
- `IPointMap` — maps raw protocol points ⇄ canonical domain signals.
- `IEssFunctions` — the IEEE 1547 + IEEE 1815.2/MESA-ESS function set.
- `ICommandSource` — inbound command channels (SCADA, custom, SEP2).
- `IHistorianStore` — write/query operational time-series.
- `IColdArchiveExporter` — age data to the cloud data lake.
- `IModelRunner` — execute MATLAB/analytical models.

## Multi-vendor DER driver model

A device-abstraction layer is valuable even with SunSpec/MESA, because vendors implement partial or
variant register maps, scaling factors, and quirks. `DeviceGateway` hosts a
**driver/profile registry**: per-manufacturer/model profiles map raw protocol points → canonical domain
signals via `IPointMap`.

Benefits:
- multi-vendor support without forking the domain,
- **config-driven device onboarding** (add a profile, no recompile),
- one normalized model for Dispatch to reason about,
- per-device fault isolation.

## Protocol abstraction

Adapters implement the ports without leaking into the core:

- `Sah.Ic.Protocols.Dnp3` — **stepfunc/dnp3** (actively maintained; replaces EOL opendnp3). See
  [ADR-0002](../adr/0002-protocol-libraries.md).
- `Sah.Ic.Protocols.Modbus` — **NModbus** + SunSpec 700/800/200 point maps.

## Typhoon HIL — interchangeable with real devices, via config

HIL exposes Modbus/DNP3, so the existing adapters reach it simply by pointing at the HIL endpoint
(`appsettings.HIL.json`). **No production code is HIL-specific.** The only HIL-specific logic is
optional **test-harness orchestration** (load schematic, inject faults, run IEEE 1547.1 scenarios) via
Typhoon's Python Test/SCADA API, which lives in the test layer — not a production adapter.

## Observability vs. Historian — two separate concerns

Observability and operational data are distinct concerns, kept separate to minimize coupling.

- **Observability (OpenTelemetry).** Traces/metrics/logs for *running the software*, exported via
  **OTLP to a Collector**. This is **infrastructure**, not a microservice — no service depends on
  another service for it.
- **Historian.** Operational *measurement data* from devices/dispatch. The Historian is a **one-way
  async sink**: producers **publish** measurement events to a bus (MQTT/queue, Sparkplug-B-friendly)
  and the Historian **subscribes**. Producers never call it synchronously and do not depend on its
  availability → coupling stays low.

## Data, caching & cloud (recommendation; captured as ADRs)

- **Redis** for hot/cache.
- **TimescaleDB** as the warm historian store (recommended) — see [ADR-0005](../adr/0005-database-historian.md).
- An age-out job writes **Parquet to S3-compatible storage** (works with ADLS/S3/MinIO) for the cold
  data lake — see [ADR-0003](../adr/0003-cloud-vendor-neutral.md).
- All behind `IHistorianStore` / `IColdArchiveExporter`, so the technology stays swappable.

## MATLAB integration

Analytics consumes **MATLAB-exported C# assemblies** compiled as ordinary .NET libraries — **not** the
MATLAB Compiler SDK runtime and **not** MATLAB Production Server. Invoked in-process behind
`IModelRunner`. See [ADR-0006](../adr/0006-matlab-integration.md).

## Regulatory conformance is built in

**ERCOT (Nodal Protocols + Operating Guides) is the mandatory compliance regime; NERC Reliability
Standards are proposed mandatory (not yet confirmed); IEEE/MESA conformance is recommended/optional.**
Code that implements a tracked requirement carries a `[Conformance(...)]` attribute validated against
the [standards register](../standards/register.yaml); a Requirements Traceability Matrix is generated
per release. Decision recorded in [ADR-0010](../adr/0010-regulatory-conformance.md); mechanism in
[`../sdlc/traceability.md`](../sdlc/traceability.md).