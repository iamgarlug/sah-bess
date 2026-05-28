# SAH-BESS Documentation

Next-generation Battery Energy Storage System (BESS) with expansions for wind and solar power.

This directory is **documentation only**. No application code or .NET solution exists yet — the
solution is *blueprinted* here and will be created in a later, separately-approved round.

## Layout

| Path | Contents |
|---|---|
| [`standards/`](standards/) | The **IEEE/MESA standards register** that governs the Intelligent Controller, plus the machine-readable `register.yaml` used for regulatory traceability. |
| [`architecture/`](architecture/) | Intelligent Controller architecture overview and the **.NET solution blueprint** (project layout + the exact `dotnet` commands to run later). |
| [`sdlc/`](sdlc/) | The parent SDLC program: delivery **pipeline**, **environments**, and regulation→code **traceability**. |
| [`adr/`](adr/) | Architecture Decision Records for decisions still open or deliberately deferred. |

## Program structure

- **Parent program — SDLC & delivery pipeline.** How SAH-BESS is built, tested, promoted, and
  audited. See [`sdlc/`](sdlc/).
- **Sub-project A — Intelligent Controller (IC).** The microservice-based controller itself. See
  [`architecture/`](architecture/) and [`standards/`](standards/).

## Reading order

1. [`standards/ieee-register.md`](standards/ieee-register.md) — what governs the system (top priority).
2. [`architecture/overview.md`](architecture/overview.md) — how the IC is shaped and why.
3. [`architecture/solution-blueprint.md`](architecture/solution-blueprint.md) — the planned code layout.
4. [`sdlc/pipeline.md`](sdlc/pipeline.md), [`sdlc/environments.md`](sdlc/environments.md),
   [`sdlc/traceability.md`](sdlc/traceability.md) — how it ships and stays auditable.
5. [`adr/`](adr/) — decisions and their status.