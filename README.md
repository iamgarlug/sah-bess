# SAH-BESS — Smart & Adaptive Hybrid Battery Energy Storage System

Next-generation Battery Energy Storage System (BESS) Intelligent Controller with expansions for wind and solar power.

## What this is

SAH-BESS is a microservice-based **Intelligent Controller (IC)** for grid-connected energy storage. It manages dispatch logic, the MESA-ESS state machine, IEEE 1547 grid-support functions, and multi-vendor DER device I/O — designed for a 10–20-year asset lifecycle across multiple client sites and cloud environments.

## Technology

- **Language / framework:** C# / .NET 10 LTS
- **Architecture:** Hexagonal (ports & adapters)
- **Protocols:** DNP3 (stepfunc/dnp3), Modbus (NModbus + SunSpec)
- **Observability:** OpenTelemetry / OTLP
- **Storage:** Redis (hot cache), TimescaleDB (warm historian), Parquet → S3-compatible (cold archive)
- **Testing:** xUnit with requirements traceability via `[Trait]`

## Compliance

| Body | Status | Register |
|---|---|---|
| ERCOT Nodal Protocols + Operating Guides | **Mandatory** | [`ercot-register.yaml`](docs/standards/ercot/ercot-register.yaml) |
| NERC Reliability Standards | Proposed mandatory | [`nerc-register.yaml`](docs/standards/nerc/nerc-register.yaml) |
| IEEE / MESA / IEC / SunSpec | Recommended / optional | [`ieee-register.yaml`](docs/standards/ieee/ieee-register.yaml) |

All compliance requirements are governed by [`docs/standards/register.yaml`](docs/standards/register.yaml) and enforced at build time via a Roslyn analyzer. See [`docs/sdlc/traceability.md`](docs/sdlc/traceability.md) for the full regulation → code → test traceability mechanism.

## Documentation

| Path | Contents |
|---|---|
| [`docs/architecture/`](docs/architecture/) | IC architecture overview and .NET solution blueprint |
| [`docs/standards/`](docs/standards/) | Standards registers (ERCOT, NERC, IEEE/MESA) |
| [`docs/adr/`](docs/adr/) | Architecture Decision Records |
| [`docs/sdlc/`](docs/sdlc/) | Delivery pipeline, environments, and regulatory traceability |
