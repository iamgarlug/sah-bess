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

```
Sah.Ic.sln
src/
  Sah.Ic.Abstractions        # ports: IDevicePort, IPointMap, IEssFunctions,
                             #        ICommandSource, IHistorianStore,
                             #        IColdArchiveExporter, IModelRunner
                             # dedicated shared-abstractions assembly; boundary enforced
                             # by an architecture test (ADR-0008)
  Sah.Ic.Domain              # dispatch logic, MESA-ESS state machine, 1547 functions (no I/O)
  Sah.Ic.Contracts           # DTOs / gRPC + REST contracts shared across services
  Sah.Ic.Observability       # OpenTelemetry wiring (OTLP exporter), shared logging/metrics
  Sah.Ic.Conformance.Abstractions  # [Conformance] attribute only (runtime; annotates prod code)
  Sah.Ic.Protocols.Dnp3      # adapter over stepfunc/dnp3
  Sah.Ic.Protocols.Modbus    # adapter over NModbus + SunSpec 700/800/200 maps
  services/
    Sah.Ic.Gateway           # SCADA/custom/(SEP2) ingress (REST + gRPC)
    Sah.Ic.Dispatch          # control/optimization orchestration
    Sah.Ic.DeviceGateway     # protocol I/O + multi-vendor driver/profile registry
    Sah.Ic.Historian         # one-way async measurement sink + retention + cloud export
    Sah.Ic.Analytics         # MATLAB model execution behind IModelRunner
tests/
  Sah.Ic.Domain.Tests
  Sah.Ic.Abstractions.Tests
  Sah.Ic.Protocols.Tests     # Modbus/DNP3 loopback adapters
  Sah.Ic.Gateway.Tests
  Sah.Ic.Dispatch.Tests
  Sah.Ic.Historian.Tests
  Sah.Ic.Analytics.Tests
  Sah.Ic.IntegrationTests    # Testcontainers: Redis / TimescaleDB / MinIO
  Sah.Ic.HilHarness          # OPTIONAL: Typhoon HIL orchestration (Python Test/SCADA API)
build/
  Sah.Ic.Conformance.Analyzer  # build-time only: Roslyn analyzer + RTM generator (ADR-0009)
docs/                        # (this documentation)
deploy/
  docker-compose.yml         # local stack: Redis, TimescaleDB, MinIO, OTel Collector
  terraform/                 # environment definitions (see ../sdlc/environments.md)
Directory.Build.props        # central target framework, analyzers, conformance analyzer ref
```

### Dependency direction

`Domain` and `Abstractions` depend on nothing external. Protocol adapters depend on `Abstractions`.
Services depend on `Domain` + `Abstractions` + the adapters they need + `Observability`. Nothing
depends inward on a service. This is the hexagonal rule, enforced by an architecture test
([ADR-0008](../adr/0008-shared-abstractions-assembly.md)).

## Intended `dotnet` commands (for the later creation round)

```bash
# --- solution + core libraries ---
dotnet new sln -n Sah.Ic
dotnet new classlib  -n Sah.Ic.Abstractions   -o src/Sah.Ic.Abstractions
dotnet new classlib  -n Sah.Ic.Domain         -o src/Sah.Ic.Domain
dotnet new classlib  -n Sah.Ic.Contracts      -o src/Sah.Ic.Contracts
dotnet new classlib  -n Sah.Ic.Observability  -o src/Sah.Ic.Observability
dotnet new classlib  -n Sah.Ic.Conformance.Abstractions -o src/Sah.Ic.Conformance.Abstractions
dotnet new classlib  -n Sah.Ic.Protocols.Dnp3   -o src/Sah.Ic.Protocols.Dnp3
dotnet new classlib  -n Sah.Ic.Protocols.Modbus -o src/Sah.Ic.Protocols.Modbus

# --- build-time tooling (not shipped at runtime) ---
dotnet new classlib  -n Sah.Ic.Conformance.Analyzer -o build/Sah.Ic.Conformance.Analyzer

# --- services ---
dotnet new worker  -n Sah.Ic.Gateway       -o src/services/Sah.Ic.Gateway
dotnet new worker  -n Sah.Ic.Dispatch      -o src/services/Sah.Ic.Dispatch
dotnet new worker  -n Sah.Ic.DeviceGateway -o src/services/Sah.Ic.DeviceGateway
dotnet new worker  -n Sah.Ic.Historian     -o src/services/Sah.Ic.Historian
dotnet new worker  -n Sah.Ic.Analytics     -o src/services/Sah.Ic.Analytics

# --- tests (xUnit) ---
dotnet new xunit -n Sah.Ic.Domain.Tests       -o tests/Sah.Ic.Domain.Tests
dotnet new xunit -n Sah.Ic.Abstractions.Tests -o tests/Sah.Ic.Abstractions.Tests
dotnet new xunit -n Sah.Ic.Protocols.Tests    -o tests/Sah.Ic.Protocols.Tests
dotnet new xunit -n Sah.Ic.Gateway.Tests      -o tests/Sah.Ic.Gateway.Tests
dotnet new xunit -n Sah.Ic.Dispatch.Tests     -o tests/Sah.Ic.Dispatch.Tests
dotnet new xunit -n Sah.Ic.Historian.Tests    -o tests/Sah.Ic.Historian.Tests
dotnet new xunit -n Sah.Ic.Analytics.Tests    -o tests/Sah.Ic.Analytics.Tests
dotnet new xunit -n Sah.Ic.IntegrationTests   -o tests/Sah.Ic.IntegrationTests

# --- add everything to the solution ---
dotnet sln Sah.Ic.sln add $(find src build tests -name '*.csproj')
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