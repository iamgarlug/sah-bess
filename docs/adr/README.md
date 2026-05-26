# Architecture Decision Records

Each ADR captures one decision: its context, the decision, status, and consequences. ADRs marked
**Proposed** are recommendations to be confirmed; **Accepted** are settled; **Deferred** are
deliberately left open with a default recommendation recorded.

| ADR | Title | Status |
|---|---|---|
| [0000](0000-adr-process.md) | ADR process — tracked in GitHub | Proposed |
| [0001](0001-hexagonal-architecture.md) | Hexagonal (ports & adapters) for the IC core | Accepted |
| [0002](0002-protocol-libraries.md) | Protocol libraries: stepfunc/dnp3 + NModbus | Accepted |
| [0003](0003-cloud-vendor-neutral.md) | Vendor-neutral cloud (OTLP + S3-compatible + Parquet) | Accepted |
| [0004](0004-target-framework.md) | Target framework .NET 10 (LTS) + xUnit | Proposed |
| [0005](0005-database-historian.md) | Historian store: TimescaleDB (recommended) | Deferred |
| [0006](0006-matlab-integration.md) | MATLAB integration: MATLAB-exported C# assemblies | Accepted |
| [0007](0007-ci-platform.md) | CI/CD platform: GitHub Actions + Environments | Proposed |
| [0008](0008-shared-abstractions-assembly.md) | Shared abstractions in a dedicated assembly | Accepted |
| [0009](0009-regulatory-conformance.md) | Regulatory conformance approach (NERC CIP mandatory, IEEE optional) | Proposed |
