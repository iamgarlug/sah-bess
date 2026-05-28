# Regulatory Traceability — Regulation → Code → Test

SAH-BESS must demonstrate, to governmental auditors, that the standards it claims to meet are actually
implemented and tested. The mechanism below creates a **strong, machine-checkable association between
regulations and the code (and tests) that satisfy them**.

## 1. Single source of truth

[`docs/standards/register.yaml`](../standards/register.yaml) lists every governing standard with a
stable `id` (e.g. `NERC-CIP-007`, `IEEE-1547-2018`), a `compliance` level, and — as implementation
proceeds — the specific `clauses` implemented. The human-readable companion is
[`ieee-register.md`](../standards/ieee/ieee-register.md).

**ERCOT (Nodal Protocols + Operating Guides) is the mandatory compliance regime** (`compliance: mandatory`);
**NERC Reliability Standards are proposed mandatory** (`compliance: proposed-mandatory`) — not yet
confirmed for this deployment; **IEEE/MESA conformance is recommended/optional**
(`compliance: recommended-optional`). This distinction is recorded in
[ADR-0010](../adr/0010-regulatory-conformance.md).

## 2. Code indicators — the `[Conformance]` attribute

Code that implements a requirement is annotated:

```csharp
[Conformance(
    Standard = "IEEE-1547-2018",
    Clause   = "5.4.1",
    Requirement = "Volt-VAR mode response within specified time")]
public Task ApplyVoltVarAsync(VoltVarCurve curve, CancellationToken ct) { ... }
```

- The attribute lives in `Sah.Ic.Conformance`.
- A **Roslyn analyzer** in the same project fails the build if `Standard` is not present in
  `register.yaml` (and warns if `Clause` is unknown for that standard) — so attributes cannot drift
  from the register.

## 3. Requirements Traceability Matrix (RTM)

A build step (source generator + a small CLI) walks all `[Conformance]` attributes and the
xUnit-trait links from tests, then emits a per-release **RTM artifact**:

```
standard (+clause)  →  code symbol(s)  →  covering test(s)  →  pass/fail
```

The RTM is published as a release artifact for audit. Gaps (a registered Tier-1 clause with no code,
or code with no covering test) are reported.

## 4. Tests linked to requirements

Tests tag the requirement they cover, so coverage maps to regulations rather than just lines:

```csharp
[Fact]
[Trait("Conformance", "IEEE-1547-2018:5.4.1")]
public async Task VoltVar_responds_within_required_time() { ... }
```

## 5. Tamper-evident audit log

Every SCADA/custom command and operational-state change is recorded (who / what / when), aligned with
**IEEE 1686** and **IEEE C37.240** cybersecurity expectations. The log is retained onsite and exported
to the cloud data lake alongside operational data. This provides the runtime evidence auditors need to
complement the build-time RTM.

## Summary of the audit story

| Question an auditor asks | Where the answer comes from |
|---|---|
| "Which standards do you claim to meet?" | `register.yaml` / `ieee-register.md` |
| "Show me the code that implements clause X." | `[Conformance]` attributes → RTM |
| "Show me the test that proves it." | xUnit `Conformance` traits → RTM |
| "Prove the system actually behaved that way in the field." | Tamper-evident audit log in the data lake |