# ADR-0003: Vendor-neutral cloud (OTLP + S3-compatible + Parquet)

**Status:** Accepted

## Context

The IC runs onsite and ages operational data to a cloud data lake, and emits OpenTelemetry. The cloud
provider is not fixed and may differ per client/site. We want to avoid lock-in.

## Decision

Use **vendor-neutral interfaces** at every cloud boundary:

- **Observability:** export via **OTLP** to an OpenTelemetry Collector (provider chosen at deploy time).
- **Cold storage / data lake:** write **Parquet** to **S3-compatible** object storage (works with AWS
  S3, Azure ADLS via S3 proxy, MinIO on-prem).
- Cloud access sits behind `IColdArchiveExporter`.

## Consequences

- **Positive:** the same build deploys against different clouds; on-prem MinIO works for air-gapped
  sites; Parquet is broadly queryable by analytics tooling.
- **Negative:** we forgo some provider-specific managed conveniences. Acceptable for portability.
