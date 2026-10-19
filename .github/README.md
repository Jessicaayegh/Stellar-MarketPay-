# GitHub Actions Workflows

The checked-in workflows automate CI, end-to-end testing, and security checks.
They do not deploy the application.

## Workflows

- `ci.yml` — linting, tests, builds, contract tests, and secret scanning.
- `e2e.yml` — browser end-to-end and accessibility tests.
- `security.yml` — code and dependency security scans.
- `automerge.yml`, `certora.yml`, `fuzz.yml`, and `deploy-storybook.yml` —
  automation for their specific repository tasks.
- `validate-secrets.yml` — reusable workflow for callers that need to validate
  secrets; it does not initiate a deployment.

See each workflow's triggers and jobs for its exact behavior. CI and testing
workflows use mock configuration and do not require deployment credentials.

## Deployment

There is no active GitHub Actions workflow for staging or production
deployment. Deployments are currently performed manually on the target host
with [`scripts/blue-green-deploy.sh`](../scripts/blue-green-deploy.sh).
Follow the [deployment runbook](../docs/deployment.md) and use the
[runtime secrets reference](../docs/ci-secrets.md). Do not copy or enable
sample `.yml.example` deployment workflows.
