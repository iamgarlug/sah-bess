# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project State

**SAH-BESS** (Smart & Adaptive Hybrid Battery Energy Storage System) is currently a **blueprint-only repository** — no `.NET` solution or source code exists yet. All scaffolding decisions are captured in `docs/architecture/solution-blueprint.md`, which contains the exact `dotnet new` commands to be run when implementation begins.

## Planned Build & Test Commands

```bash
dotnet build
dotnet test /p:CollectCoverage=true /p:CoverageFormat=opencover
```

Coverage target is 80% (tracked on a dashboard, not a blocking gate).

## Architecture

**Language:** C# / .NET 10 LTS  
**Style:** Hexagonal (ports & adapters)  
**Test framework:** xUnit with `[Trait]` for requirements traceability

### Planned Project Layout

Libraries ship as NuGet packages; each library and service is its own project folder. The internal
layout of each project (e.g. `src/` / `tests/` subfolders, file organization) is left to the developer.

```
libraries/
  Sah.Ic.Domain/                    # Dispatch logic, MESA-ESS state machine, IEEE 1547 grid-support — no dependencies
  Sah.Ic.Abstractions/              # Ports: IDevicePort, IPointMap, IEssFunctions, ICommandSource, IHistorianStore, IColdArchiveExporter, IModelRunner
  Sah.Ic.Contracts/                 # Cross-service DTOs
  Sah.Ic.Observability/             # OpenTelemetry wiring (OTLP export)
  Sah.Ic.Conformance.Abstractions/  # Runtime [Conformance] attribute
  Sah.Ic.Protocols.Dnp3/            # DNP3 adapter via stepfunc/dnp3
  Sah.Ic.Protocols.Modbus/          # Modbus adapter via NModbus + SunSpec 700/800/200 point maps
  Sah.Ic.Conformance.Analyzer/      # Roslyn build-time analyzer — validates [Conformance] attributes against docs/standards/register.yaml
services/
  Gateway/        # SCADA/SEP2 command ingress (REST + gRPC)
  Dispatch/       # Orchestration, MESA-ESS state machine
  DeviceGateway/  # Protocol I/O + multi-vendor DER driver/profile registry
  Historian/      # One-way async operational time-series sink + cold export (Parquet → S3-compatible)
  Analytics/      # MATLAB model execution (SoC/SoH, forecasting, optimization)
```

`Directory.Build.props` will enforce the target framework, enable analyzers, and wire up `Sah.Ic.Conformance.Analyzer` for every project.

### Multi-Vendor DER Model

Per-manufacturer/model profiles implement `IPointMap` to translate raw protocol points to canonical domain signals. New device onboarding is config-driven — no code changes required for supported protocols.

### Data Stores

| Store | Role |
|---|---|
| Redis | Hot cache |
| TimescaleDB (recommended) or InfluxDB | Warm historian (ADR-0005 deferred) |
| Parquet → S3-compatible (AWS S3 / Azure ADLS / MinIO) | Cold archive |

Observability (OpenTelemetry/OTLP) and the Historian are **separate concerns** — OTLP is infrastructure telemetry; Historian is a one-way async operational data sink via MQTT/queue.

### MATLAB Integration

MATLAB models are exported as C# assemblies and run **in-process** — no MATLAB Compiler SDK dependency at runtime (ADR-0006).

### Typhoon HIL

HIL test hardware is interchangeable with real devices via configuration only; no HIL-specific code enters production paths.

## Regulatory Conformance

Two sets of standards are **mandatory**:

1. **ERCOT** — both ERCOT Nodal Protocols and ERCOT Nodal Operating Guides. Source documents are in `docs/standards/ercot/nodal/protocols-2026-06/` and `docs/standards/ercot/nodal/operating-guides-2026-02/`.
2. **NERC Reliability Standards** — proposed mandatory; not yet confirmed for this deployment. Current published standards: https://www.nerc.com/standards/reliability-standards

IEEE 1547 and MESA-ESS conformance are recommended/optional.

All compliance-relevant code must carry `[Conformance(Standard="...", Clause="...", Requirement="...")]` attributes. The Roslyn analyzer (`Sah.Ic.Conformance.Analyzer`) validates these attribute IDs exist in `docs/standards/register.yaml` at build time. Tests link to requirements via `[Trait("Conformance", "IEEE-1547-2018:5.4.1")]`.

`docs/standards/register.yaml` is the single source of truth for all standard IDs. Before adding a new `[Conformance]` attribute, verify the clause exists there.

## Key Documentation

| Path | Purpose |
|---|---|
| `docs/adr/` | Architecture Decision Records — read before proposing changes to tech stack or architecture |
| `docs/architecture/solution-blueprint.md` | Scaffolding commands, project naming conventions, Central Package Management setup |
| `docs/sdlc/` | Delivery pipeline, environment promotion, CI/CD (GitHub Actions) |
| `docs/standards/register.yaml` | Machine-readable conformance register |

## CI/CD Pipeline

**CI (every PR):** build → unit + integration tests → static analysis → SBOM + license scan → conformance-attribute check.

**CD (progressive promotion):** `Dev (per-PR ephemeral) → QA → Load/Perf → HIL` *(manual gate: IEEE 1547.1 sign-off)* `→ Onsite` *(manual gate: production release)*.

Platform: GitHub Actions with Environments (ADR-0007).
