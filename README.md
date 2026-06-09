# github-actions

Reusable GitHub Actions workflows for kube-the-home repositories.

## Workflows

### `helm-lint.yaml`

Lints a Helm chart using `helm lint`.

**Inputs**

| Name           | Required | Default         | Description                                               |
| -------------- | -------- | --------------- | --------------------------------------------------------- |
| `chart_path`   | yes      | —               | Path to the Helm chart directory (e.g. `charts/my-chart`) |
| `helm_version` | no       | `latest`        | Helm version to install                                   |
| `runs_on`      | no       | `ubuntu-latest` | Runner OS to use                                          |

**Example**

```yaml
jobs:
  lint:
    uses: hankeln/github-actions/.github/workflows/helm-lint.yaml@main
    with:
      chart_path: charts/my-chart
```

---

### `dotnet-build-test.yaml`

Restores, builds, and tests a .NET solution or project.

**Inputs**

| Name                | Required | Default         | Description                                       |
| ------------------- | -------- | --------------- | ------------------------------------------------- |
| `working_directory` | no       | `.`             | Directory containing the solution or project file |
| `dotnet_version`    | no       | `8.0.x`         | .NET SDK version to use (e.g. `8.0.x`, `9.0.x`)   |
| `configuration`     | no       | `Release`       | Build configuration (`Debug` or `Release`)        |
| `runs_on`           | no       | `ubuntu-latest` | Runner OS to use                                  |

**Example**

```yaml
jobs:
  build:
    uses: hankeln/github-actions/.github/workflows/dotnet-build-test.yaml@main
    with:
      working_directory: src
      dotnet_version: 9.0.x
```

---

### `semantic-release.yaml`

Runs [semantic-release](https://github.com/semantic-release/semantic-release) and outputs the new version. Uses a GitHub App for authentication so that the release commit can trigger downstream workflows.

**Secrets**

| Name              | Required | Description            |
| ----------------- | -------- | ---------------------- |
| `APP_ID`          | yes      | GitHub App client ID   |
| `APP_PRIVATE_KEY` | yes      | GitHub App private key |

**Inputs**

| Name      | Required | Default         | Description      |
| --------- | -------- | --------------- | ---------------- |
| `runs_on` | no       | `ubuntu-latest` | Runner OS to use |

**Outputs**

| Name          | Description                                                                             |
| ------------- | --------------------------------------------------------------------------------------- |
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

| Name              | Required | Description            |
| ----------------- | -------- | ---------------------- |
| `APP_ID`          | yes      | GitHub App client ID   |
| `APP_PRIVATE_KEY` | yes      | GitHub App private key |

**Inputs**

| Name                    | Required | Default         | Description                                                           |
| ----------------------- | -------- | --------------- | --------------------------------------------------------------------- |
| `chart_path`            | yes      | —               | Path to the Helm chart directory (e.g. `charts/my-chart`)             |
| `version`               | yes      | —               | Chart version to release (without `v` prefix, e.g. `1.2.3`)           |
| `helm_version`          | no       | `latest`        | Helm version to install                                               |
| `helm_dependency_repos` | no       | `''`            | Newline-separated list of Helm repos to add (`<name> <url>` per line) |
| `runs_on`               | no       | `ubuntu-latest` | Runner OS to use                                                      |

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

### `git-conventions.yaml`

Validates that all commits on a pull request follow the [Conventional Commits](https://www.conventionalcommits.org) specification.

**Inputs**

| Name      | Required | Default         | Description      |
| --------- | -------- | --------------- | ---------------- |
| `runs_on` | no       | `ubuntu-latest` | Runner OS to use |

**Example**

```yaml
jobs:
  conventions:
    uses: hankeln/github-actions/.github/workflows/git-conventions.yaml@main
```

---

### `dotnet-nuget-release.yaml`

Builds, packs, and pushes a .NET project to NuGet.

**Secrets**

| Name            | Required | Description                        |
| --------------- | -------- | ---------------------------------- |
| `nuget_api_key` | yes      | NuGet API key for pushing packages |

**Inputs**

| Name                | Required | Default         | Description                                       |
| ------------------- | -------- | --------------- | ------------------------------------------------- |
| `working_directory` | no       | `.`             | Directory containing the solution or project file |
| `dotnet_version`    | no       | `8.0.x`         | .NET SDK version to use (e.g. `8.0.x`, `9.0.x`)   |
| `runs_on`           | no       | `ubuntu-latest` | Runner OS to use                                  |

**Example**

```yaml
jobs:
  nuget:
    uses: hankeln/github-actions/.github/workflows/dotnet-nuget-release.yaml@main
    with:
      working_directory: src
    secrets:
      nuget_api_key: ${{ secrets.NUGET_API_KEY }}
```

---

### `yamllint.yaml`

Lints YAML files using [yamllint](https://github.com/karancode/yamllint-github-action).

**Inputs**

| Name          | Required | Default         | Description               |
| ------------- | -------- | --------------- | ------------------------- |
| `file_or_dir` | no       | `.`             | File or directory to lint |
| `runs_on`     | no       | `ubuntu-latest` | Runner OS to use          |

**Example**

```yaml
jobs:
  yamllint:
    uses: hankeln/github-actions/.github/workflows/yamllint.yaml@main
```

---

### `gitops-linter.yaml`

Lints GitOps manifests using [glint](https://github.com/lukashankeln/glint). Optionally uploads a SARIF report to the GitHub Security tab.

**Inputs**

| Name           | Required | Default         | Description                                      |
| -------------- | -------- | --------------- | ------------------------------------------------ |
| `fail_on`      | no       | `error`         | Severity level to fail on (`error` or `warning`) |
| `upload_sarif` | no       | `false`         | Upload SARIF results to GitHub Security tab      |
| `runs_on`      | no       | `ubuntu-latest` | Runner OS to use                                 |

**Example**

```yaml
jobs:
  gitops-lint:
    uses: hankeln/github-actions/.github/workflows/gitops-linter.yaml@main
    permissions:
      contents: read
      security-events: write
```

---

### `helm-template.yaml`

Renders a Helm chart with `helm template`, optionally passing inline values.

**Inputs**

| Name           | Required | Default         | Description                                                          |
| -------------- | -------- | --------------- | -------------------------------------------------------------------- |
| `chart_path`   | yes      | —               | Path to the Helm chart directory (e.g. `charts/my-chart`)            |
| `values`       | no       | `''`            | Values to pass as a YAML string (written to a temporary values file) |
| `helm_version` | no       | `latest`        | Helm version to install                                              |
| `runs_on`      | no       | `ubuntu-latest` | Runner OS to use                                                     |

**Example**

```yaml
jobs:
  template:
    uses: hankeln/github-actions/.github/workflows/helm-template.yaml@main
    with:
      chart_path: charts/my-chart
      values: |
        replicaCount: 2
        image:
          tag: dev
```

---

### `release-please.yaml`

Runs [release-please](https://github.com/googleapis/release-please) to manage release PRs and changelog. Uses a GitHub App for authentication so that the release commit can trigger downstream workflows.

**Secrets**

| Name              | Required | Description            |
| ----------------- | -------- | ---------------------- |
| `APP_ID`          | yes      | GitHub App client ID   |
| `APP_PRIVATE_KEY` | yes      | GitHub App private key |

**Inputs**

| Name            | Required | Default                         | Description                                    |
| --------------- | -------- | ------------------------------- | ---------------------------------------------- |
| `release_type`  | no       | `simple`                        | Release type (e.g. `simple`, `node`, `python`) |
| `config_file`   | no       | `release-please-config.json`    | Path to release-please config file             |
| `manifest_file` | no       | `.release-please-manifest.json` | Path to release-please manifest file           |
| `runs_on`       | no       | `ubuntu-latest`                 | Runner OS to use                               |

**Outputs**

| Name              | Description                                         |
| ----------------- | --------------------------------------------------- |
| `release_created` | `true` if a release was created                     |
| `tag_name`        | Tag name of the created release                     |
| `version`         | Version of the created release (without `v` prefix) |

**Example**

```yaml
jobs:
  release:
    uses: hankeln/github-actions/.github/workflows/release-please.yaml@main
    secrets:
      APP_ID: ${{ secrets.APP_ID }}
      APP_PRIVATE_KEY: ${{ secrets.APP_PRIVATE_KEY }}
```

---

### `mkdocs.yaml`

Deploys a [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) site to GitHub Pages using `mkdocs gh-deploy`.

**Inputs**

| Name             | Required | Default         | Description           |
| ---------------- | -------- | --------------- | --------------------- |
| `python_version` | no       | `3.x`           | Python version to use |
| `runs_on`        | no       | `ubuntu-latest` | Runner OS to use      |

**Example**

```yaml
on:
  push:
    branches: [main]

jobs:
  docs:
    uses: hankeln/github-actions/.github/workflows/mkdocs.yaml@main
```

---

### `golang-build.yaml`

Builds a Go module using `go build ./...`.

**Inputs**

| Name                | Required | Default         | Description                                    |
| ------------------- | -------- | --------------- | ---------------------------------------------- |
| `working_directory` | no       | `.`             | Directory containing the Go module             |
| `go_version`        | no       | `stable`        | Go version to use (e.g. `stable`, `1.23.x`)    |
| `runs_on`           | no       | `ubuntu-latest` | Runner OS to use                               |

**Example**

```yaml
jobs:
  build:
    uses: hankeln/github-actions/.github/workflows/golang-build.yaml@main
    with:
      working_directory: src
      go_version: "1.23.x"
```

---

### `golang-test.yaml`

Runs Go tests using `go test -v ./...`.

**Inputs**

| Name                | Required | Default         | Description                                    |
| ------------------- | -------- | --------------- | ---------------------------------------------- |
| `working_directory` | no       | `.`             | Directory containing the Go module             |
| `go_version`        | no       | `stable`        | Go version to use (e.g. `stable`, `1.23.x`)    |
| `runs_on`           | no       | `ubuntu-latest` | Runner OS to use                               |

**Example**

```yaml
jobs:
  test:
    uses: hankeln/github-actions/.github/workflows/golang-test.yaml@main
    with:
      working_directory: src
```

---

### `golang-lint.yaml`

Lints a Go module using [golangci-lint](https://github.com/golangci/golangci-lint).

**Inputs**

| Name                   | Required | Default         | Description                                          |
| ---------------------- | -------- | --------------- | ---------------------------------------------------- |
| `working_directory`    | no       | `.`             | Directory containing the Go module                   |
| `go_version`           | no       | `stable`        | Go version to use (e.g. `stable`, `1.23.x`)          |
| `golangci_lint_version`| no       | `latest`        | golangci-lint version to use (e.g. `latest`, `v1.64.0`) |
| `runs_on`              | no       | `ubuntu-latest` | Runner OS to use                                     |

**Example**

```yaml
jobs:
  lint:
    uses: hankeln/github-actions/.github/workflows/golang-lint.yaml@main
    with:
      working_directory: src
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
