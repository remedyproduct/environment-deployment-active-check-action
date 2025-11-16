# Check Active Environment Deployment

GitHub Action that checks whether a GitHub environment already has an **active deployment** for a target ref (typically a branch like `main`) and tells you if a new deploy is needed.

A common use case is a scheduled (cron) workflow: for example, a nightly job that redeploys `qa` only if `main` has new commits compared to the currently active deployment. When the active deployment SHA matches the ref's current commit, the action reports `should_deploy = 'false'`, so the job can skip the deployment step.

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

## Required permissions

Minimal permissions:

```yaml
permissions:
  contents: read
  deployments: read
```

If you pass a custom `token`, it must be able to read refs and deployments for the repository.

## Example Usage

```yaml
jobs:
  check:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      deployments: read
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