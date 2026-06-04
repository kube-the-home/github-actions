# github-actions

Reusable GitHub Actions workflows for kube-the-home repositories.

## Workflows

### `helm-lint.yaml`

Lints a Helm chart using `helm lint`.

**Inputs**

| Name | Required | Default | Description |
|---|---|---|---|
| `chart_path` | yes | — | Path to the Helm chart directory (e.g. `charts/my-chart`) |
| `helm_version` | no | `latest` | Helm version to install |
| `runs_on` | no | `ubuntu-latest` | Runner OS to use |

**Example**

```yaml
jobs:
  lint:
    uses: hankeln/github-actions/.github/workflows/helm-lint.yaml@main
    with:
      chart_path: charts/my-chart
```

---

### `semantic-release.yaml`

Runs [semantic-release](https://github.com/semantic-release/semantic-release) and outputs the new version. Uses a GitHub App for authentication so that the release commit can trigger downstream workflows.

**Secrets**

| Name | Required | Description |
|---|---|---|
| `APP_ID` | yes | GitHub App client ID |
| `APP_PRIVATE_KEY` | yes | GitHub App private key |

**Inputs**

| Name | Required | Default | Description |
|---|---|---|---|
| `runs_on` | no | `ubuntu-latest` | Runner OS to use |

**Outputs**

| Name | Description |
|---|---|
| `new_version` | Version created by semantic-release (without `v` prefix). Empty if no release was made. |

**Example**

```yaml
jobs:
  release:
    uses: hankeln/github-actions/.github/workflows/semantic-release.yaml@main
    secrets:
      APP_ID: ${{ secrets.APP_ID }}
      APP_PRIVATE_KEY: ${{ secrets.APP_PRIVATE_KEY }}
```

---

### `helm-release.yaml`

Updates `Chart.yaml`, regenerates Helm docs, and publishes the chart via [chart-releaser](https://github.com/helm/chart-releaser-action). Designed to run after `semantic-release.yaml`, consuming its `new_version` output.

**Secrets**

| Name | Required | Description |
|---|---|---|
| `APP_ID` | yes | GitHub App client ID |
| `APP_PRIVATE_KEY` | yes | GitHub App private key |

**Inputs**

| Name | Required | Default | Description |
|---|---|---|---|
| `chart_path` | yes | — | Path to the Helm chart directory (e.g. `charts/my-chart`) |
| `version` | yes | — | Chart version to release (without `v` prefix, e.g. `1.2.3`) |
| `helm_version` | no | `latest` | Helm version to install |
| `helm_dependency_repos` | no | `''` | Newline-separated list of Helm repos to add (`<name> <url>` per line) |
| `runs_on` | no | `ubuntu-latest` | Runner OS to use |

**Example**

```yaml
jobs:
  helm:
    uses: hankeln/github-actions/.github/workflows/helm-release.yaml@main
    with:
      chart_path: charts/my-chart
      version: 1.2.3
      helm_dependency_repos: |
        opentelemetry-collector https://open-telemetry.github.io/opentelemetry-helm-charts
    secrets:
      APP_ID: ${{ secrets.APP_ID }}
      APP_PRIVATE_KEY: ${{ secrets.APP_PRIVATE_KEY }}
```

---

## Full release pipeline example

Chain `semantic-release` and `helm-release` so that the Helm chart is only published when a new version is actually created.

```yaml
name: Release

on:
  push:
    branches: [main]

jobs:
  release:
    uses: hankeln/github-actions/.github/workflows/semantic-release.yaml@main
    secrets:
      APP_ID: ${{ secrets.APP_ID }}
      APP_PRIVATE_KEY: ${{ secrets.APP_PRIVATE_KEY }}

  helm:
    needs: release
    if: needs.release.outputs.new_version != ''
    uses: hankeln/github-actions/.github/workflows/helm-release.yaml@main
    with:
      chart_path: charts/my-chart
      version: ${{ needs.release.outputs.new_version }}
      helm_dependency_repos: |
        opentelemetry-collector https://open-telemetry.github.io/opentelemetry-helm-charts
    secrets:
      APP_ID: ${{ secrets.APP_ID }}
      APP_PRIVATE_KEY: ${{ secrets.APP_PRIVATE_KEY }}
```
