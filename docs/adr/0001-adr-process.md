# ADR-0001: ADR process — tracked in GitHub

**Status:** Proposed

## Context

The project needs a lightweight, durable way to record architecture decisions so the rationale travels
with the code and is reviewable.

## Decision

Architecture Decision Records are authored as Markdown files in `docs/adr/`, numbered sequentially, and
**tracked in Git and reviewed through GitHub pull requests**. Each ADR records context, decision,
status, and consequences. Status values: **Proposed**, **Accepted**, **Deferred**, **Superseded**.

## Consequences

- **Positive:** decisions are versioned alongside code, diffable, and reviewed in the normal PR flow;
  no extra tooling required.
- **Negative:** no automated ADR indexing/visualisation beyond the manual `README.md` index.
- **Revisit:** if a dedicated ADR creation/management tool (e.g. Log4brains, adr-tools) provides enough
  added value (templating, indexing, graphing) to justify adopting it.
