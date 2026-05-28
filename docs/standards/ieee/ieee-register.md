# IEEE / MESA Standards Register — Intelligent Controller

This register is the **governing list of standards** for the SAH-BESS Intelligent Controller (IC). It
is the human-readable companion to the machine-readable [`register.yaml`](register.yaml), which is the
single source of truth used by the build-time conformance checks and the Requirements Traceability
Matrix (see [`../sdlc/traceability.md`](../sdlc/traceability.md)).

> Standard editions/years below reflect the latest known at time of writing. Confirm the exact edition
> at procurement; `register.yaml` carries the authoritative IDs.

**Compliance levels.** **ERCOT (Nodal Protocols + Operating Guides) is the mandatory compliance
regime**; **NERC Reliability Standards are proposed mandatory** (not yet confirmed for this
deployment); **IEEE/MESA conformance is recommended/optional.** Each entry in
[`register.yaml`](../register.yaml) carries a `compliance` field
(`mandatory` | `proposed-mandatory` | `recommended-optional` | `organizational`). See
[ADR-0010](../../adr/0010-regulatory-conformance.md).

## Proposed Mandatory — NERC Reliability Standards (CIP subset)

FERC-enforced cybersecurity compliance for the bulk electric system. Mandatory status not yet
confirmed for this deployment — tracked as `proposed-mandatory` in `register.yaml`. Representative
CIP subset below; confirm the standards in scope per deployment based on BES Cyber System applicability.

| ID | Standard | Scope |
|---|---|---|
| `NERC-CIP-002` | CIP-002 | BES Cyber System categorization |
| `NERC-CIP-003` | CIP-003 | Security management controls |
| `NERC-CIP-004` | CIP-004 | Personnel and training |
| `NERC-CIP-005` | CIP-005 | Electronic security perimeter(s) |
| `NERC-CIP-006` | CIP-006 | Physical security of BES Cyber Systems |
| `NERC-CIP-007` | CIP-007 | System security management |
| `NERC-CIP-008` | CIP-008 | Incident reporting and response planning |
| `NERC-CIP-009` | CIP-009 | Recovery plans for BES Cyber Systems |
| `NERC-CIP-010` | CIP-010 | Configuration change management and vulnerability assessments |
| `NERC-CIP-011` | CIP-011 | Information protection |
| `NERC-CIP-013` | CIP-013 | Supply chain risk management |
| `NERC-CIP-014` | CIP-014 | Physical security |

## Tier 1 — core, directly shapes the IC (recommended/optional)

| ID | Standard | Title / scope | Why it matters here |
|---|---|---|---|
| `IEEE-1547-2018` | IEEE 1547-2018 | Interconnection & Interoperability of DER with associated electric power systems interfaces | Anchor standard. Defines required grid-support / smart-inverter functions, ride-through, and the interoperability information model the IC must implement. |
| `IEEE-1547.1-2020` | IEEE 1547.1-2020 | Conformance test procedures for equipment interconnecting DER | Drives the conformance / Typhoon HIL test harness. |
| `IEEE-1815-2012` | IEEE 1815-2012 | DNP3 (Distributed Network Protocol) | Base protocol the DNP3 adapter implements; MESA-ESS is a DNP3 profile layered on this. |
| `IEEE-1815.2-2026` | IEEE 1815.2 | DNP3 **application profile for DER** (the published form of **MESA-DER**) | Exact DER point/function mappings the DNP3 adapter must speak for MESA conformance. |
| `IEEE-2030.5-2018` | IEEE 2030.5-2018 | Smart Energy Profile 2.0 (SEP2) | IP/HTTP(S) utility/aggregator↔DER path (e.g. California Rule 21, DERMS), behind `ICommandSource`. |
| `IEEE-2030.2.1-2019` | IEEE 2030.2.1-2019 | Design, operation & maintenance of BESS | Guidance for the BESS domain model and operational lifecycle. |
| `IEEE-2800-2022` | IEEE 2800-2022 | Interconnection & interoperability of inverter-based resources (IBR) on **transmission** systems | Future-proofing for grid-scale and wind/solar expansion. |
| `IEEE-2686-2024` | IEEE 2686-2024 | Battery Management Systems (BMS) for stationary ESS | BMS data models / comms for battery integration. |

## Tier 2 — supporting guidance

| ID | Standard | Note |
|---|---|---|
| `IEEE-1547.2-2023` | IEEE 1547.2 | Application guide for 1547. |
| `IEEE-1547.3-2023` | IEEE 1547.3-2023 | **DER cybersecurity** guidance. |
| `IEEE-1547.9-2022` | IEEE 1547.9-2022 | Interconnecting energy storage DER. |
| `IEEE-2030-2011` | IEEE 2030-2011 | Smart grid interoperability reference model. |
| `IEEE-2030.2-2015` | IEEE 2030.2-2015 | Interoperability of storage with the grid. |
| `IEEE-2030.3-2016` | IEEE 2030.3-2016 | Test procedures for storage equipment. |
| `IEEE-1815.1-2015` | IEEE 1815.1-2015 | Mapping IEC 61850 ⇄ DNP3. |
| `IEEE-1679-2020` | IEEE 1679-2020 (+1679.1 lithium) | Characterizing/evaluating storage technologies. |
| `IEEE-1881-2022` | IEEE 1881 | Battery terminology. |

## Cybersecurity & timing

| ID | Standard | Note |
|---|---|---|
| `IEEE-1686-2022` | IEEE 1686-2022 | IED cybersecurity capabilities — informs the tamper-evident audit log. |
| `IEEE-C37.240-2014` | IEEE C37.240-2014 | Substation/IED cybersecurity. |
| `IEEE-1588-2019` | IEEE 1588-2019 (PTP) | Time sync for distributed control, HIL timing, SOE/historian timestamps. |
| `IEEE-C37.118-2011` | IEEE C37.118.1/.2 | Synchrophasor measurement/transfer (optional). |
| `IEEE-519-2022` | IEEE 519-2022 | Harmonics / power-quality limits. |

## Adjacent (non-IEEE) — required for MESA / safety / comms completeness

| ID | Standard | Note |
|---|---|---|
| `MESA-ESS` | MESA-ESS specification | DNP3 application profile for ESS. The IC's MESA-ESS state machine targets this. |
| `MESA-DEVICE` | MESA-Device (= SunSpec) | Modbus models: **700** (PCS/inverter), **800** family incl. base **802** (battery), **200** (meter). |
| `SUNSPEC-MODELS` | SunSpec Modbus model definitions | Free machine-readable model defs; drive the Modbus adapter point maps. |
| `IEC-61850` | IEC 61850 / 61850-7-420 | Substation automation + DER logical nodes. |
| `IEC-62443` | IEC 62443 | Industrial automation & control system security. |
| `IEC-61400-25` | IEC 61400-25 | Communications for wind power plants. |
| `UL-9540` | UL 9540 / 9540A | ESS safety + thermal-runaway fire propagation test. Organizational/hardware scope. |
| `NFPA-855` | NFPA 855 | Installation of stationary ESS. Organizational/hardware scope. |

## Procurement priority (to refine implementation)

| Priority | Documents | Reason |
|---|---|---|
| **Buy first (become code)** | `IEEE-1815.2-2026` / MESA-DER, `IEEE-1547-2018`, `IEEE-1547.1-2020` | Directly define points, functions, and conformance tests. |
| **Buy next** | `IEEE-1815-2012`, `IEEE-2686-2024` | Base DNP3 + BMS integration. |
| **Conditional** | `IEEE-2030.5-2018` (if SEP2 near-term), `IEEE-2800-2022` (when transmission work starts) | Defer until the corresponding adapter/feature is scheduled. |
| **FREE — get now** | `SUNSPEC-MODELS` 700/800/802/200 + `MESA-DEVICE` / `MESA-ESS` | Downloadable with a SunSpec account; drive the Modbus adapter. Highest value-per-dollar. |
| **Guidance only** | 1547.2, 1547.9, 2030-2011, 2030.2-2015, 2030.2.1, 2030.3, 1679, 1881 | Reference, not directly implemented. |

## How this register is used

- Every `id` here also appears in [`register.yaml`](register.yaml).
- Code that implements a requirement is annotated with a `[Conformance(Standard="<id>", Clause="…")]`
  attribute. A build-time analyzer rejects any attribute whose `Standard` is not in `register.yaml`.
- The Requirements Traceability Matrix maps `standard → code symbol → covering test` for audit.

See [`../sdlc/traceability.md`](../sdlc/traceability.md) for the full mechanism.