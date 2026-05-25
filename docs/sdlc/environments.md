# SAH-BESS Environments

The minimum environment set for the SAH-BESS delivery pipeline. All environments are defined as
**Terraform** so they are reproducible and vendor-neutral; ephemeral ones are created and destroyed on
demand.

| Environment | Purpose | Devices | Lifecycle |
|---|---|---|---|
| **Dev** | Fast inner loop; per-PR preview deploys | Mocked / loopback adapters | Always-on shared + ephemeral per-PR |
| **QA** | Integration testing + MESA conformance smoke tests; stable build | Simulated / a few real | Persistent |
| **Testing (Load/Perf)** | Load, soak, and performance testing at scaled data volumes | Simulated at scale | On-demand (scale up for runs, down after) |
| **Typhoon HIL** | Controller-in-the-Loop against the HIL rig; IEEE 1547.1 conformance runs | Typhoon HIL (via Modbus/DNP3) | Persistent, rig-attached |
| **Sales / Demo** | Curated, stable demo data for prospect demos | Curated/simulated | Persistent or scheduled |
| **Client testing** | Per-client isolated validation | Per-client config | **Ephemeral**, TTL-bounded, spun up/down per engagement |

## Notes

- **Dev → ephemeral PR previews** let reviewers exercise a change without touching shared QA.
- **Typhoon HIL** is reached purely by configuration (`appsettings.HIL.json`) — the same protocol
  adapters used against real devices point at the HIL endpoint. No HIL-specific production code.
- **Load/Perf** scales the historian + messaging backbone to validate throughput and soak behavior
  before onsite release.
- **Client testing** environments are isolated per client and **torn down on a TTL** to control cost
  and data residency; provisioning them is behind a manual approval gate (see
  [`pipeline.md`](pipeline.md)).
- Promotion order and the gates between environments are defined in [`pipeline.md`](pipeline.md).
