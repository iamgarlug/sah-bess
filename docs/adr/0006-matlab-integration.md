# ADR-0006: MATLAB integration — MATLAB-exported C# assemblies

**Status:** Accepted

## Context

Heavy math (SoC/SoH estimation, forecasting, optimization) is authored in MATLAB. The Analytics service
must execute these models from .NET.

## Decision

Analytics consumes **MATLAB-exported C# assemblies**, compiled and referenced as ordinary .NET
libraries. **We will not use the MATLAB Compiler SDK runtime, and will not use MATLAB Production
Server.** Models are invoked in-process behind `IModelRunner`, so Analytics stays agnostic to how a
model was produced.

## Consequences

- **Positive:** no MATLAB Runtime or model-hosting server to operate onsite; models ship as in-process
  .NET assemblies; `IModelRunner` keeps callers decoupled and allows reimplementing a model in pure
  .NET later without touching callers.
- **Negative:** model updates require re-exporting and redeploying the assembly (versioned with the
  service); the MATLAB export toolchain is part of the model build pipeline.
