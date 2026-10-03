# CapacityLab Load Test

A GitHub Action that starts a [CapacityLab](https://capacitylab.dev) Load Run
from a CI template for the release you are building and, by default, fails the
job when the Result verdict is not attained.

The template (a pre-defined Load Spec, zone and budget) and the CI key are
created in the CapacityLab Console. The action downloads the CapacityLab CLI
from your API (`<api>/api/v1/ci/cli.mjs`) and runs it with the runner's Node
(Node 22 or newer; add `actions/setup-node` first if your runner lacks it).

## Usage

1. In the Console, create a CI template and a CI key.
2. Store the key as the repository (or environment) secret `CAPACITYLAB_CI_KEY`.
3. Add a job:

```yaml
name: Load gate
on:
  pull_request:

jobs:
  load-gate:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write # only needed for github-comment
    steps:
      - uses: CapacityLove/load-test-action@v1
        with:
          template: checkout
          api-url: <api> # the API origin shown on the Console CI page
          ci-key: ${{ secrets.CAPACITYLAB_CI_KEY }}
          github-comment: "true"
```

## Inputs

| Input                    | Required | Default             | Description                                                                          |
| ------------------------ | -------- | ------------------- | ------------------------------------------------------------------------------------ |
| `template`               | yes      |                     | CI template name configured in the Console.                                          |
| `ci-key`                 | yes      |                     | CapacityLab CI key. Pass it from `secrets.CAPACITYLAB_CI_KEY`.                       |
| `api-url`                | yes      |                     | CapacityLab API origin.                                                              |
| `release`                | no       | `github.sha`        | Release the run is attributed to (commit SHA or tag). One Load Run per release.      |
| `wait`                   | no       | `true`              | Wait for the Result verdict and gate on it; `false` returns once the run is admitted. |
| `timeout-minutes`        | no       | `90`                | Longest wait for the verdict.                                                        |
| `max-queue-wait-minutes` | no       | `30`                | Longest time to keep retrying a retryable refusal, for example no capacity.          |
| `github-comment`         | no       | `false`             | Create or update one pull request comment per template with the result.              |
| `github-token`           | no       | `github.token`      | Token used for the pull request comment.                                             |

With `github-comment`, outside a pull request the comment is skipped and the
gate is unaffected. A job summary is always written to `GITHUB_STEP_SUMMARY`.

## Secrets

`CAPACITYLAB_CI_KEY` is a narrow credential: it can trigger and read only the
templates it names, with no Console, MCP or other API access. The action passes
it to the CLI through the environment only, never on the command line. Revoke
it in the Console at any time.

## Exit codes

| Code | Meaning                                                                                    |
| ---- | ------------------------------------------------------------------------------------------ |
| 0    | Gate passed (Result verdict attained), or the run was admitted with `wait: "false"`.       |
| 1    | Gate failed (verdict not attained, for example a No Regression regression).                |
| 2    | Inconclusive Result.                                                                       |
| 3    | Trigger refused (budget, template not allowed or not found, capacity after the queue wait). |
| 4    | Usage, authentication, network, protocol or timeout error.                                 |

The action fails the step on any non-zero code. To treat exit code 2 differently, run the CLI directly and inspect its exit status.

## Versioning

Pin `@v1` for compatible updates, or a full tag such as `@v1.0.0` for an
immutable reference.

## License

[MIT](LICENSE). Learn more at [capacitylab.dev](https://capacitylab.dev).
