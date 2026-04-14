# select-runner

Prefer self-hosted GitHub Actions runners when at least one is online; fall back to a GitHub-hosted runner otherwise.

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
          self-hosted-label: self-hosted # optional, default: self-hosted
          fallback: ubuntu-latest        # optional, default: ubuntu-latest

  build:
    needs: pick-runner
    runs-on: ${{ needs.pick-runner.outputs.runner }}
    steps:
      - run: echo "Running on ${{ needs.pick-runner.outputs.runner }}"
```

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `self-hosted-label` | No | `self-hosted` | Label to match against online self-hosted runners |
| `fallback` | No | `ubuntu-latest` | Runner label to use when no self-hosted runners are online |
| `token` | No | `github.token` | Token used to query the Actions API (needs `actions: read`) |

## Outputs

| Name | Description |
|---|---|
| `runner` | Resolved runner label to pass to `runs-on` |

## How it works

1. Calls the GitHub REST API to list runners for the current repository
2. Counts runners that are **online** and carry the requested label
3. If at least one is online, outputs that label
4. Otherwise, outputs the fallback label

## License

MIT
