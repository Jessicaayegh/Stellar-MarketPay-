# Deployment Runbook

## Current deployment model

Deployments are **operator-run on the target host**. GitHub Actions currently
runs CI, end-to-end, and security checks; there is no automated staging or
production deployment workflow. GitHub environment approvals and GitHub
deployment secrets are therefore not part of the current deployment path.

The production Docker Compose deployment uses blue-green services defined in
[`docker-compose.prod.yml`](../docker-compose.prod.yml). An operator updates
the image(s) in the registry, then runs
[`scripts/blue-green-deploy.sh`](../scripts/blue-green-deploy.sh) from the
repository root on the deployment host. The script pulls the standby color,
starts it, health-checks it, changes the NGINX upstream, and removes the old
color after the rollback window. If a pre-switch check fails, it restores the
previous NGINX configuration and exits unsuccessfully.

The Helm chart under [`deploy/helm/`](../deploy/helm/) is a deployment
configuration, not a GitHub Actions deploy workflow.

## Before deploying

1. Confirm you have operator access to the target host, Docker Engine with the
   Compose plugin, and the repository checkout at the intended release.
2. Build and publish the frontend and backend image(s) to a registry the host
   can access. Use image tags that identify the release; the deploy script
   pulls the tags supplied to Compose and does not build or publish releases.
3. On the deployment host, make the external Compose secrets
   `database_url`, `next_public_api_url`, `grafana_admin_user`, and
   `grafana_admin_password` available. Set `DATABASE_URL` and
   `NEXT_PUBLIC_API_URL` in the host's deployment environment for Compose
   interpolation. Keep secret values out of the repository and logs.
4. Ensure the production Compose services and NGINX are already initialized,
   the active upstream is represented by `nginx/nginx.conf`, and the host can
   reach the configured registry and database.
5. Select an `IMAGE_TAG` for the frontend. Set `BACKEND_IMAGE_TAG` too if the
   backend uses a different tag; otherwise it defaults to `IMAGE_TAG`.

The exact names and purposes of runtime variables and secrets are in
[CI and deployment secrets](ci-secrets.md). No GitHub deployment secrets are
needed for this procedure.

## Deploy

From the repository root on the deployment host, set the release values in
that shell's environment, then invoke the script:

```bash
export IMAGE_TAG=registry.example.com/marketpay-frontend:release-tag
export BACKEND_IMAGE_TAG=registry.example.com/marketpay-backend:release-tag
export DATABASE_URL='postgresql://user:password@db-host:5432/database'
export NEXT_PUBLIC_API_URL='https://api.example.com'
bash scripts/blue-green-deploy.sh
```

Use the real image references and runtime configuration for the target host.
The script uses `docker-compose.prod.yml`, chooses the standby color based on
`nginx/nginx.conf`, and waits for the standby backend's `/api/health` endpoint
before switching traffic. Its defaults are 30 health-check attempts at
5-second intervals and a 600-second rollback window; `MAX_RETRIES`,
`RETRY_INTERVAL`, `ROLLBACK_WINDOW`, `HEALTH_ENDPOINT`, and `BACKEND_PORT` can
be overridden in the host process environment.

Watch the script's exit status and output. A zero exit status means the traffic
switch and post-switch health verification completed. A non-zero status means
the deploy did not complete successfully; inspect the NGINX configuration and
container status before attempting another deployment.

## Rollback and cleanup

The script restores the original NGINX configuration when standby health
checks or post-switch verification fail. After a successful switch, it leaves
the former active services running for the rollback window, then removes those
containers automatically. For a later rollback, deploy the previously
published known-good image tags by following the deploy procedure again.

## Backup credentials

Database backup and restore are separate host-run operations. Their S3
credentials and PostgreSQL variables are listed in
[CI and deployment secrets](ci-secrets.md); they are not read from GitHub
Actions. See [database backup and restore](database-backup-restore.md) for the
backup and restore procedures.
