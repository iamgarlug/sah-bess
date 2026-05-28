# ADR-0009: Shared abstractions in a dedicated assembly

**Status:** Accepted

## Context

The hexagonal design ([ADR-0002](0002-hexagonal-architecture.md)) depends on a stable core that does not
take dependencies on protocols, persistence, cloud, or services. To make that boundary real (not just a
convention), the shared ports and contracts need a defined home and an enforcement mechanism.

## Decision

- Shared **ports** live in a dedicated assembly **`Sah.Ic.Abstractions`** (with **`Sah.Ic.Contracts`**
  for cross-service DTOs). These assemblies depend on nothing external.
- The dependency rule — services and adapters may depend inward on Abstractions/Domain, but nothing may
  depend outward on a service — is **enforced by an architecture test** (e.g. NetArchTest) that runs in
  CI and fails the build on violation.

## Consequences

- **Positive:** the core boundary is machine-enforced, not just documented; adapters/services are
  swappable without leaking into the core; the test gives fast feedback on accidental coupling.
- **Negative:** more projects up front; the architecture test must be kept current as new
  assemblies are added.
