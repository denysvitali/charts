# Helm Charts

A curated collection of Helm charts for Kubernetes, published at:

```
https://denysvitali.github.io/charts
```

## Adding the Repository

```bash
helm repo add denysvitali https://denysvitali.github.io/charts
helm repo update
```

## Available Charts

| Chart | Description | Version |
|-------|-------------|---------|
| *(charts will appear here as they are added)* | | |

## Contributing

Contributions are welcome! Please read the contribution guidelines before submitting a PR.

### Development Prerequisites

- [Helm](https://helm.sh/docs/intro/install/) >= 3.x
- [kubeconform](https://github.com/yannh/kubeconform) (for schema validation)
- [kube-linter](https://github.com/stackrox/kube-linter) (for security checks)
- [chart-testing (ct)](https://github.com/helm/chart-testing) (for lint/test)
- [dyff](https://github.com/homeport/dyff) (for YAML-aware template diffs)

### Workflow Overview

Every pull request to `main` triggers:

1. **Helm Lint** — `helm lint --strict --with-subcharts` across all changed charts and their values overlays
2. **Chart Testing** — `ct lint` for yamllint + yamale validation
3. **kubeconform** — Validates rendered templates against Kubernetes JSON schemas and the CRDs catalog
4. **kube-linter** — Static security analysis on rendered templates
5. **Template Diff** — A [dyff](https://github.com/homeport/dyff)-powered YAML-aware diff is posted as a PR comment for each changed chart and all its values files (default + overlays like `values.prod.yaml`, `values.dev.yaml`)

On merge to `main`, if the chart version was bumped in `Chart.yaml`, the chart is automatically packaged and published to the GitHub Pages Helm repository via `chart-releaser`.

### Chart Structure

```
charts/
└── my-chart/
    ├── Chart.yaml          # chart metadata — bump version here to release
    ├── values.yaml         # default values
    ├── values.prod.yaml    # (optional) production overlay
    ├── values.dev.yaml     # (optional) development overlay
    ├── templates/
    │   ├── _helpers.tpl
    │   ├── deployment.yaml
    │   └── ...
    └── ci/                 # extra values files for ct install tests
        └── test-values.yaml
```

### Releasing a Chart

1. Make your changes in `charts/<chart-name>/`.
2. Bump the `version` field in `charts/<chart-name>/Chart.yaml`.
3. Open a PR — CI will validate your changes and show a template diff.
4. Merge to `main` — the chart is automatically released.

## License

MIT — see [LICENSE](LICENSE).
