# Environment Deployment Active Check Action

Reusable GitHub Action to determine whether a given GitHub environment already has an **active deployment** for a target ref (typically a branch like `main`).

When the active deployment SHA for the environment matches the ref's current commit, the action reports `should_deploy = 'false'`, which you can use to skip redundant deployments.

## Inputs

- `environment` (required):
  - The GitHub environment name to inspect (for example, `qa`, `stage`, or `production`).
- `ref` (optional, default: `main`):
  - Branch name or full ref (e.g. `main` or `refs/heads/main`) whose HEAD commit will be compared to the active deployment.
- `token` (optional):
  - GitHub token with `repo` scope.
  - Defaults to `github.token` when omitted.

## Outputs

- `should_deploy`:
  - `'true'` if **no active deployment** exists for the target environment **or** the active deployment SHA differs from the target ref SHA.
  - `'false'` if there *is* an active deployment whose SHA matches the target ref SHA.
- `current_sha`:
  - The SHA of the target ref used for comparison.
- `active_sha`:
  - The SHA of the current active deployment in the target environment (empty string if none).

## Example Usage

```yaml
jobs:
  check:
    runs-on: ubuntu-latest
    outputs:
      should_deploy: ${{ steps.env-check.outputs.should_deploy }}
    steps:
      - name: Check if QA needs deploy
        id: env-check
        uses: remedyproduct/environment-deployment-active-check-action@master
        with:
          environment: qa
          ref: main

  deploy:
    needs: check
    if: needs.check.outputs.should_deploy == 'true'
    runs-on: ubuntu-latest
    steps:
      - name: Do deployment
        run: echo "Deploying because QA is not on current main"
```
