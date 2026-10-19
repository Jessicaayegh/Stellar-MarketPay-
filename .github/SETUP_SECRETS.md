# Deployment and Secrets Setup

GitHub Actions currently runs CI, end-to-end, and security checks only. There
is no automated staging or production deployment workflow, and deployment
credentials should not be added to GitHub Actions for this setup.

Deployments are run manually by an operator on the deployment host using
[`scripts/blue-green-deploy.sh`](../scripts/blue-green-deploy.sh) and
[`docker-compose.prod.yml`](../docker-compose.prod.yml). Follow the
[deployment runbook](../docs/deployment.md) for the complete procedure.

The current runtime secret names and their uses are listed in
[`docs/ci-secrets.md`](../docs/ci-secrets.md). In short, production Compose
requires the external secrets `database_url`, `next_public_api_url`,
`grafana_admin_user`, and `grafana_admin_password`. Database backup and restore
also require S3-compatible storage credentials and PostgreSQL connection
variables on the host where those scripts run. These are host/runtime secrets,
not GitHub Actions secrets.

Do not copy or enable a sample deployment workflow. If GitHub Actions
deployment automation is introduced later, add a real workflow only after its
runner, environment protections, secret requirements, and deployment commands
are implemented and documented together.
