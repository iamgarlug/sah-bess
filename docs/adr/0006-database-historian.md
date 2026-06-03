# ADR-0006: Historian store — TimescaleDB (recommended)

**Status:** Deferred (recommendation recorded)

## Context

The Historian service stores operational time-series measurement data with tiered retention before
aging cold data to the lake. The specific store is not yet decided. Candidates: TimescaleDB,
InfluxDB, plain PostgreSQL, or a cloud-managed time-series service.

## Decision

**Defer** the final choice. **Recommended default: TimescaleDB** (PostgreSQL extension) — mature
time-series features, SQL familiarity, open-source, and runnable on-prem. Access is hidden behind
`IHistorianStore`, so the choice remains swappable; Redis is used for the hot/cache tier.

## Consequences

- **Positive:** no premature lock-in; integration tests use Testcontainers against the chosen store;
  the abstraction allows revisiting without touching the domain.
- **Negative:** the abstraction must avoid leaking store-specific query features into the core.
- **Revisit when:** performance/scale testing (Load/Perf environment) provides real throughput numbers.
