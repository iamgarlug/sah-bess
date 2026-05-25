# ADR-0006: MATLAB integration — Compiler SDK → .NET assembly

**Status:** Deferred (recommendation recorded)

## Context

Heavy math (SoC/SoH estimation, forecasting, optimization) is authored in MATLAB. The Analytics service
must execute these models from .NET. Options: MATLAB Compiler SDK (compile models to a .NET assembly),
MATLAB Production Server (scalable model-hosting service), or reimplementation in .NET.

## Decision

**Defer** the final choice. **Recommended default: MATLAB Compiler SDK** → a .NET assembly consumed by
Analytics (uses the MATLAB Runtime, no per-node MATLAB license). **MATLAB Production Server** is the
scalable alternative when model load grows. Both sit behind `IModelRunner`, so Analytics is agnostic.

## Consequences

- **Positive:** keeps model authorship in MATLAB; `IModelRunner` lets us switch hosting modes or even
  reimplement specific models in .NET without touching callers.
- **Negative:** MATLAB Runtime must be deployed onsite for the Compiler SDK path; licensing terms to be
  confirmed.
- **Revisit when:** model count/latency requirements are known, or licensing constraints surface.
