# ADR-0010: Regulatory conformance approach

**Status:** Proposed

## Context

SAH-BESS operates on the electric power system and must satisfy mandatory regulatory compliance, while
also conforming to engineering interoperability standards. These are not the same kind of obligation,
and auditors need to see which requirements are implemented and tested.

## Decision

- **ERCOT (Nodal Protocols + Nodal Operating Guides) is the mandatory compliance regime.**
  **NERC Reliability Standards are proposed mandatory** — mandatory status is not yet confirmed for
  this deployment and is tracked as `proposed-mandatory` in the standards register.
  **IEEE/MESA conformance is recommended/optional.** The standards register records these distinctions
  via a `compliance` field (`mandatory` | `proposed-mandatory` | `recommended-optional`).
- Conformance is tracked with:
  - the machine-readable register [`docs/standards/register.yaml`](../standards/register.yaml) as the
    single source of truth,
  - a `[Conformance(Standard, Clause, Requirement)]` attribute on implementing code,
  - a generated **Requirements Traceability Matrix** (standard → code symbol → covering test),
  - a tamper-evident audit log for runtime evidence.
- **Tooling Placement**
  - the `[Conformance]` attribute is a small **runtime** assembly, **`Sah.Ic.Conformance.Abstractions`**
    in `src/` (production code carries the annotations),
  - the **Roslyn analyzer + RTM generator** are **build-time tooling** in **`build/Sah.Ic.Conformance.Analyzer`**,
    not shipped at runtime.

## Consequences

- **Positive:** the mandatory vs. optional distinction is explicit and auditable; build-time tooling is
  separated from runtime code; the RTM gives auditors a single artifact.
- **Negative:** two conformance assemblies instead of one; the register must be kept in sync as scope
  grows.
- **Revisit:** confirm whether NERC Reliability Standards become mandatory for this deployment as
  bulk-electric-system applicability is determined; promote `proposed-mandatory` entries to `mandatory`
  in `register.yaml` at that time.
