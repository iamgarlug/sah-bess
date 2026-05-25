# ADR-0004: Target framework — .NET 10 (LTS)

**Status:** Proposed

## Context

The IC is a long-lived, onsite .NET system. Framework support lifetime matters. As of writing, **.NET 10
is LTS (released Nov 2025)** while **.NET 9 is STS** and nearing end-of-life. Key dependencies —
stepfunc/dnp3 (.NET Standard 2.0), NModbus, OpenTelemetry .NET — are compatible with .NET 10.

## Decision

Target **.NET 10 (LTS)** for all projects, centralized in `Directory.Build.props`.

## Consequences

- **Positive:** longest support runway; current tooling and performance; compatible with all planned
  dependencies.
- **Negative:** none significant. Revisit if a critical dependency lags .NET 10 support at creation
  time — confirm before scaffolding.
