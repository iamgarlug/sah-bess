# ADR-0005: Target framework — .NET 10 (LTS) + xUnit

**Status:** Proposed

## Context

The IC is a long-lived, onsite .NET system. Framework support lifetime matters. As of writing, **.NET 10
is LTS (released Nov 2025)** while **.NET 9 is STS** and nearing end-of-life. Key dependencies —
stepfunc/dnp3 (.NET Standard 2.0), NModbus, OpenTelemetry .NET — are compatible with .NET 10. A test
framework must also be chosen up front because conformance traceability depends on linking tests to
requirement IDs.

## Decision

- Target **.NET 10 (LTS)** for all projects, centralized in `Directory.Build.props`.
- Use **xUnit** as the test framework: first-class parallelism, the `[Trait]` mechanism used to link
  tests to conformance requirement IDs (see [ADR-0010](0010-regulatory-conformance.md)), and broad
  ecosystem/tooling support (coverlet, Testcontainers).

## Consequences

- **Positive:** longest support runway; current tooling/performance; compatible with all planned
  dependencies; xUnit traits feed the Requirements Traceability Matrix.
- **Negative:** none significant. Revisit the framework if a critical dependency lags .NET 10 support at
  creation time — confirm before scaffolding.
