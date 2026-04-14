# select-runner

Prefer a runner label you configure when at least one matching runner is online; fall back to another label otherwise (typical setup: `self-hosted` with a GitHub-hosted fallback).

## Usage

```yaml
jobs:
  pick-runner:
    runs-on: ubuntu-latest
    permissions:
      actions: read
    outputs:
      runner: ${{ steps.select.outputs.runner }}
    steps:
      - uses: lyskdev/select-runner@v1
        id: select
        with:
          preferred-runner-label: self-hosted # optional, default: self-hosted
          fallback: ubuntu-latest        # optional, default: ubuntu-latest

  build:
    needs: pick-runner
    runs-on: ${{ needs.pick-runner.outputs.runner }}
    steps:
      - run: echo "Running on ${{ needs.pick-runner.outputs.runner }}"
```

## Inputs

The preferred runner input was renamed from `self-hosted-label` to `preferred-runner-label`; behavior and default (`self-hosted`) are unchanged.

| Name | Required | Default | Description |
|---|---|---|---|
| `preferred-runner-label` | No | `self-hosted` | Label of the preferred runner when at least one online runner has it |
| `fallback` | No | `ubuntu-latest` | Runner label when no online runners match the preferred runner label |
| `token` | No | `github.token` | Token used to query the Actions API (needs `actions: read`) |

## Outputs

| Name | Description |
|---|---|
| `runner` | Resolved runner label to pass to `runs-on` |

## How it works

1. Calls the GitHub REST API to list runners for the current repository
2. Counts runners that are **online** and carry the preferred runner label
3. If at least one is online, outputs that label
4. Otherwise, outputs the fallback label

## License

MIT
