# ADR-0002: Hexagonal (ports & adapters) for the IC core

**Status:** Accepted

## Context

The Intelligent Controller's durable value is its domain logic (dispatch, the MESA-ESS state machine,
IEEE 1547 grid-support functions), which must remain stable across a 10–20-year asset lifecycle. The
surrounding technology is volatile or undecided: the DNP3 stack already churned (opendnp3 EOL), the
database/cache is undecided, the cloud backend is deliberately vendor-neutral, and field devices span
many vendors with differing register maps. The system also targets 80% test coverage and must run
conformance tests without hardware (Typhoon HIL).

## Decision

Adopt **hexagonal architecture (ports & adapters)**. The domain core (`Sah.Ic.Domain`,
`Sah.Ic.Abstractions`) depends on nothing external. All volatile concerns — protocols, persistence,
cloud, devices, MATLAB — sit behind ports and are implemented by swappable adapters.

## Consequences

- **Positive:** swapping a protocol lib, database, cloud target, or device vendor is an adapter change,
  not a core change; the core is unit-testable without hardware; a clean seam exists for `[Conformance]`
  indicators; real devices and Typhoon HIL are interchangeable by configuration.
- **Negative:** more projects and indirection up front. Accepted given the lifecycle, undecided
  infrastructure, and multi-vendor/HIL requirements.
- An architecture test should enforce the inward dependency rule.
