# ADR-0003: Protocol libraries — stepfunc/dnp3 + NModbus

**Status:** Accepted

## Context

The IC speaks DNP3 (MESA-ESS / IEEE 1815.2 profiles) and Modbus (SunSpec models). The previously common
open-source DNP3 library, opendnp3, is **end-of-life**. We prefer actively maintained, managed
(C#-friendly) open-source libraries.

## Decision

- **DNP3:** **stepfunc/dnp3** — actively maintained, ships a managed .NET binding via NuGet, supports
  the DNP3 features required for MESA profiles.
- **Modbus:** **NModbus** — established, managed, open-source.

Both are wrapped behind `IDevicePort` / `IPointMap` in `Sah.Ic.Protocols.Dnp3` and
`Sah.Ic.Protocols.Modbus`.

## Consequences

- **Positive:** maintained dependencies; managed bindings simplify the .NET integration; abstraction
  keeps the choice swappable.
- **Negative:** stepfunc/dnp3 ships native shared libraries — the onsite runtime (Linux/Windows) must
  include them; the solution blueprint notes this.
