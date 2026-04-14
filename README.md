# select-runner

Prefer a runner label you configure when at least one matching runner is online; fall back to another label otherwise (typical setup: `self-hosted` with a GitHub-hosted fallback).

Supports runners registered at both the **repo level** and **org level**.

## Usage

### Repo-level runners (default)

If your self-hosted runners are registered directly on the repository, the default `github.token` is sufficient:

```yaml
jobs:
  select-runner:
    runs-on: ubuntu-latest
    permissions:
      actions: read
    outputs:
      runner: ${{ steps.select.outputs.runner }}
    steps:
      - uses: lyskdev/select-runner@v1
        id: select
        with:
          preferred-runner-label: self-hosted

  build:
    needs: select-runner
    runs-on: ${{ needs.select-runner.outputs.runner }}
    steps:
      - run: echo "Running on ${{ needs.select-runner.outputs.runner }}"
```

### Org-level runners

If your runners are registered at the **organization level** (common when sharing runners across multiple repos), the repo-level API cannot see them. Pass a `runner-token` with org-level read access:

```yaml
jobs:
  select-runner:
    runs-on: ubuntu-latest
    permissions:
      actions: read
    outputs:
      runner: ${{ steps.select.outputs.runner }}
    steps:
      - uses: lyskdev/select-runner@v1
        id: select
        with:
          preferred-runner-label: self-hosted
          runner-token: ${{ secrets.RUNNER_TOKEN }}

  build:
    needs: select-runner
    runs-on: ${{ needs.select-runner.outputs.runner }}
    steps:
      - run: echo "Running on ${{ needs.select-runner.outputs.runner }}"
```

#### Creating the `RUNNER_TOKEN` secret

1. Create a **fine-grained Personal Access Token** at [github.com/settings/tokens](https://github.com/settings/tokens?type=beta):
   - **Resource owner:** your organization
   - **Organization permissions:** "Self-hosted runners" &rarr; **Read**
   - No repository permissions needed
2. Add it as an **org-level secret** named `RUNNER_TOKEN` at `github.com/organizations/{org}/settings/secrets/actions`

> A classic PAT with `admin:org` scope also works but grants broader access than necessary.

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `preferred-runner-label` | No | `self-hosted` | Label of the preferred runner when at least one online runner has it |
| `fallback` | No | `ubuntu-latest` | Runner label when no online runners match the preferred runner label |
| `token` | No | `github.token` | Token for the repo-level Actions API (needs `actions: read`) |
| `runner-token` | No | `""` | Token with org-level runner read access. Enables org-level runner discovery. Not required when runners are registered at the repo level |

## Outputs

| Name | Description |
|---|---|
| `runner` | Resolved runner label to pass to `runs-on` |

## How it works

1. Queries the **repo-level** GitHub REST API to list runners for the current repository
2. Counts runners that are **online** and carry the preferred runner label
3. If at least one is found, outputs that label
4. If none found **and** `runner-token` is provided, queries the **org-level** API
5. If at least one org runner is online with the label, outputs that label
6. Otherwise, outputs the fallback label

```
repo runners online? ──yes──> use preferred label
        │
        no
        │
runner-token provided? ──no──> use fallback label
        │
       yes
        │
org runners online? ──yes──> use preferred label
        │
        no
        │
use fallback label
```

## Why org-level runners need a separate token

The default `github.token` in GitHub Actions is scoped to the repository. It can list repo-level runners via `/repos/{owner}/{repo}/actions/runners`, but **cannot** access the org-level endpoint `/orgs/{org}/actions/runners` — even if the org's runner group grants access to all repositories.

This is a GitHub API limitation: the runner group controls which repos can **use** the runners (job scheduling), but the repo-scoped token still cannot **query** org-level runner status.

## License

MIT
