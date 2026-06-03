# .NET Solution Blueprint — Intelligent Controller

> **Status: BLUEPRINT ONLY.** This document describes the solution to be created in a later,
> separately-approved round. **Nothing here has been scaffolded, built, or run.** The `dotnet`
> commands below are the intended steps for that future round.

## Target framework

**.NET 10 (LTS, released Nov 2025).** Rationale: .NET 9 is STS and nearing end-of-life; .NET 10 gives
the longest support runway for an onsite, long-lived deployment. All key dependencies are compatible:
stepfunc/dnp3 (.NET Standard 2.0), NModbus, and OpenTelemetry .NET.

**Test framework: xUnit.** Chosen alongside the framework for first-class parallelism, the `[Trait]`
mechanism used to link tests to conformance requirement IDs (feeds the Requirements Traceability
Matrix), and broad ecosystem support (coverlet, Testcontainers). See
[ADR-0004](../adr/0004-target-framework.md).

## Solution layout

There is no top-level solution file. Libraries ship as NuGet packages; each library and service is
its own project folder. The internal layout of each project (e.g. `src/` / `tests/` subfolders,
file organization) is left to the developer.

```
libraries/
  Sah.Ic.Abstractions/              # ports: IDevicePort, IPointMap, IEssFunctions, ICommandSource,
                                    #        IHistorianStore, IColdArchiveExporter, IModelRunner
                                    # dedicated shared-abstractions assembly; boundary enforced
                                    # by an architecture test (ADR-0009)
  Sah.Ic.Domain/                    # dispatch logic, MESA-ESS state machine, 1547 functions (no I/O)
  Sah.Ic.Contracts/                 # DTOs / gRPC + REST contracts shared across services
  Sah.Ic.Observability/             # OpenTelemetry wiring (OTLP exporter), shared logging/metrics
  Sah.Ic.Conformance.Abstractions/  # [Conformance] attribute only (runtime; annotates prod code)
  Sah.Ic.Protocols.Dnp3/            # adapter over stepfunc/dnp3
  Sah.Ic.Protocols.Modbus/          # adapter over NModbus + SunSpec 700/800/200 maps
services/
  Sah.Ic.Gateway/                   # SCADA/custom/(SEP2) ingress (REST + gRPC)
  Sah.Ic.Dispatch/                  # control/optimization orchestration
  Sah.Ic.DeviceGateway/             # protocol I/O + multi-vendor driver/profile registry
  Sah.Ic.Historian/                 # one-way async measurement sink + retention + cloud export
  Sah.Ic.Analytics/                 # MATLAB model execution behind IModelRunner
build/
  Sah.Ic.Conformance.Analyzer/      # build-time only: Roslyn analyzer + RTM generator (ADR-0010)
docs/                               # (this documentation)
deploy/
  docker-compose.yml                # local stack: Redis, TimescaleDB, MinIO, OTel Collector
  terraform/                        # environment definitions (see ../sdlc/environments.md)
Directory.Build.props               # central target framework, analyzers, conformance analyzer ref
```

### Dependency direction

`Domain` and `Abstractions` depend on nothing external. Protocol adapters depend on `Abstractions`.
Services depend on `Domain` + `Abstractions` + the adapters they need + `Observability`. Nothing
depends inward on a service. This is the hexagonal rule, enforced by an architecture test
([ADR-0008](../adr/0008-shared-abstractions-assembly.md)).

## Intended `dotnet` commands (for the later creation round)

```bash
# --- libraries (NuGet packages) ---
dotnet new classlib -n Sah.Ic.Abstractions             -o libraries/Sah.Ic.Abstractions
dotnet new classlib -n Sah.Ic.Domain                   -o libraries/Sah.Ic.Domain
dotnet new classlib -n Sah.Ic.Contracts                -o libraries/Sah.Ic.Contracts
dotnet new classlib -n Sah.Ic.Observability            -o libraries/Sah.Ic.Observability
dotnet new classlib -n Sah.Ic.Conformance.Abstractions -o libraries/Sah.Ic.Conformance.Abstractions
dotnet new classlib -n Sah.Ic.Protocols.Dnp3           -o libraries/Sah.Ic.Protocols.Dnp3
dotnet new classlib -n Sah.Ic.Protocols.Modbus         -o libraries/Sah.Ic.Protocols.Modbus

# --- build-time tooling (not shipped at runtime) ---
dotnet new classlib -n Sah.Ic.Conformance.Analyzer     -o build/Sah.Ic.Conformance.Analyzer

# --- services ---
dotnet new worker -n Sah.Ic.Gateway       -o services/Sah.Ic.Gateway
dotnet new worker -n Sah.Ic.Dispatch      -o services/Sah.Ic.Dispatch
dotnet new worker -n Sah.Ic.DeviceGateway -o services/Sah.Ic.DeviceGateway
dotnet new worker -n Sah.Ic.Historian     -o services/Sah.Ic.Historian
dotnet new worker -n Sah.Ic.Analytics     -o services/Sah.Ic.Analytics

# Test-project layout (xunit project per library/service plus any cross-cutting
# integration/HIL harnesses) is left to the developer — see "Testing & coverage" below.
```

### Key NuGet packages (added during creation)

| Project | Packages |
|---|---|
| `Protocols.Dnp3` | stepfunc/dnp3 (`dnp3` managed binding) |
| `Protocols.Modbus` | `NModbus` |
| `Observability` | `OpenTelemetry`, `OpenTelemetry.Exporter.OpenTelemetryProtocol`, `OpenTelemetry.Extensions.Hosting` |
| `Historian` | TimescaleDB via `Npgsql`; messaging client (MQTT/queue) |
| `IntegrationTests` | `Testcontainers`, `Testcontainers.PostgreSql`, `Testcontainers.Redis`, MinIO container |
| All test projects | `coverlet.collector`, `Microsoft.NET.Test.Sdk`, `xunit`, `FluentAssertions` |

## Testing & coverage

- One test project per component, aiming at the **80% line-coverage goal** (coverlet + ReportGenerator).
  80% is a goal tracked on a dashboard, **not a blocking gate** — emergency fixes may temporarily dip
  below it, and the dashboard flags prominently when coverage is under 80%.
- Coverage **excludes** generated gRPC stubs and thin `Program.cs` bootstrap.
- An **architecture test** asserts the dependency direction above.
- Tests carry xUnit traits linking them to standard requirement IDs (see traceability).

## Build governance

- `Directory.Build.props` centralizes the target framework, analyzer set, nullable/warnings-as-errors,
  and references the **conformance analyzer** so the regulation↔code linkage is enforced on every build.

## Future verification (when the solution is created)

`dotnet build` on .NET 10 succeeds; `dotnet test` passes (Modbus/DNP3 loopbacks, domain state machine,
integration via Testcontainers); coverage tracked (80% goal; dashboard flags when below); service
`/health` healthy; OTel flows Gateway→Dispatch→DeviceGateway via the Collector; HIL reachable by config
only; the conformance analyzer reports green and emits the RTM artifact.