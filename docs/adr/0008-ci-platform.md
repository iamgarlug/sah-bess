# ADR-0008: CI/CD platform — GitHub Actions + Environments

**Status:** Proposed

## Context

The parent SDLC program requires automation-first delivery with manual approval gates and ephemeral
environments. The repository is hosted on GitHub. The pipeline must integrate coverage tracking
(dashboard, not a gate), SBOM/license scanning, the conformance-attribute check, and progressive
promotion across environments.

## Decision

Use **GitHub Actions** for CI/CD and **GitHub Environments with required reviewers** to implement the
manual approval gates. Provision environments (including ephemeral ones) with **Terraform**, keeping
infrastructure vendor-neutral.

## Consequences

- **Positive:** native to the existing GitHub repo; Environments give first-class required-reviewer
  gates; Terraform keeps infra portable and reproducible; ephemeral previews/clients fit naturally.
- **Negative:** ties CI orchestration to GitHub. Mitigated by keeping build/test/deploy logic in
  scripts and Terraform so the platform layer is thin and replaceable.
- **Revisit if:** the organization standardizes on a different CI platform (e.g. Azure DevOps,
  GitLab) — the underlying scripts/IaC would largely carry over.
