# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Helm chart repository published to `https://denysvitali.github.io/charts`. Currently contains one chart: `happy` (Happy Server — a backend for Claude Code clients with PostgreSQL, Redis, S3, and optional UI).

## Common Commands

```bash
# Lint a chart strictly
helm lint --strict --with-subcharts charts/<chart-name>

# Lint with chart-testing (uses ct.yaml config)
ct lint --config ct.yaml --charts charts/<chart-name>

# Render templates locally
helm template my-release charts/<chart-name>

# Render with a specific values overlay
helm template my-release charts/<chart-name> -f charts/<chart-name>/values.prod.yaml

# Validate rendered templates against K8s schemas
helm template my-release charts/<chart-name> | kubeconform -strict -kubernetes-version 1.29.0

# Security lint rendered templates
helm template my-release charts/<chart-name> | kube-linter lint -

# YAML-aware diff between two renders
dyff between <(helm template rel charts/foo -f old.yaml) <(helm template rel charts/foo -f new.yaml)
```

## CI Pipeline (PR to main)

1. **helm lint --strict** + **ct lint** (yamllint + yamale)
2. **kubeconform** against K8s 1.29.0 JSON schemas
3. **kube-linter** static security analysis (config: `.kube-linter.yaml`)
4. **dyff** template diff posted as a PR comment for each changed chart across all values overlays

## Release Process

Releases are triggered manually via `workflow_dispatch` on the release workflow. It uses **git-cliff** to auto-calculate semantic versions from conventional commits, tags, packages, and publishes to GitHub Pages via chart-releaser. To release a chart: ensure your commits follow conventional commit format, then trigger the release workflow.

## Conventions

- **Conventional commits required**: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `perf:`, `test:`, `ci:`, `build:` — git-cliff parses these for changelog generation and version bumping.
- **Values overlays**: Charts may have `values.prod.yaml`, `values.dev.yaml` etc. alongside `values.yaml`. CI validates all overlays.
- **CI test values**: Place minimal/mock values for CI linting in `charts/<name>/ci/test-values.yaml` (e.g., disable external dependencies, use fake URLs).
- **YAML lint rules**: Max line length 160 (warning), truthy restricted to `true`/`false` only. Config: `.yamllint.yaml`.
- **kube-linter**: Default checks plus `no-anti-affinity` and `privilege-escalation-container`. CPU/memory requirements are excluded. Config: `.kube-linter.yaml`.
- **Chart version**: Do NOT manually bump `version` in `Chart.yaml` — the release workflow handles this via git-cliff.

## Architecture: happy chart

The `happy` chart (`charts/happy/`) deploys:
- **Happy Server** deployment with a Prisma migration init container, env vars for DB/S3/Redis, liveness/readiness probes on `/healthz`
- **PostgreSQL** via CloudNativePG `Cluster` CRD (optional, `postgres.enabled`)
- **Redis** deployment (optional, `redis.enabled`)
- **Happy UI** deployment (optional, `ui.enabled`)
- **ExternalSecret** for secrets via External Secrets Operator (optional, `externalSecret.enabled`)
- **PodDisruptionBudget** (enabled by default)
- Standard Service + optional Ingress with TLS
