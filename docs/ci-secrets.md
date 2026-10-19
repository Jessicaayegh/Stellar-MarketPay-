# CI and Deployment Secrets

This document distinguishes GitHub Actions secrets from credentials used by
operators and services at runtime.

## GitHub Actions

The checked-in GitHub Actions workflows run CI, end-to-end, security, and
validation jobs. None deploys the application or requires deployment
credentials. `GITHUB_TOKEN` is supplied automatically by GitHub Actions where
needed.

There are currently **no GitHub Actions deployment workflows**. In
particular, do not create repository or environment secrets for SSH deployment,
AWS, databases, Discord, or Slack on behalf of a deploy workflow that does not
exist. Staging and production deployments are initiated manually on their
respective hosts; see the [deployment runbook](deployment.md).

## Production Compose Runtime

[`docker-compose.prod.yml`](../docker-compose.prod.yml) expects the following
external Docker secrets to be made available to the Compose deployment on the
host:

| Secret name | Used by | Value |
|---|---|---|
| `database_url` | Blue/green backend services | PostgreSQL connection URL |
| `next_public_api_url` | Blue/green frontend services | Public API base URL |
| `grafana_admin_user` | Grafana | Grafana administrator username |
| `grafana_admin_password` | Grafana | Grafana administrator password |

The frontend and backend Compose services also read `NEXT_PUBLIC_API_URL` and
`DATABASE_URL` from the deployment host's environment for Compose
interpolation. Set those to the same corresponding values used by the
`next_public_api_url` and `database_url` secrets. Image references are selected
with `IMAGE_TAG` and, optionally, `BACKEND_IMAGE_TAG` (which defaults to
`IMAGE_TAG`).

Keep secret values in the host's secret-management facility; do not commit
them, put them in image tags, or paste them into deployment logs.

## Database Backup and Restore

[`scripts/backup-db.sh`](../scripts/backup-db.sh) and
[`scripts/restore-db.sh`](../scripts/restore-db.sh) run on the host where the
scripts are invoked. They read credentials from that process's environment;
they do not read GitHub Actions secrets.

| Variable | Required by script | Purpose |
|---|---|---|
| `AWS_ACCESS_KEY_ID` | Yes | S3-compatible storage access key |
| `AWS_SECRET_ACCESS_KEY` | Yes | S3-compatible storage secret |
| `S3_BUCKET` | Yes | Bucket to read/write backups |
| `PGHOST` | Yes | PostgreSQL host |
| `PGUSER` | Yes | PostgreSQL user |
| `PGPASSWORD` | Yes | PostgreSQL password |
| `AWS_ENDPOINT_URL` | No | S3-compatible endpoint; defaults to AWS S3 |
| `AWS_REGION` | No | Storage region; defaults to `auto` |
| `PGPORT` | No | PostgreSQL port; defaults to `5432` |
| `PGDATABASE` | No | Database name; defaults to `stellarwork` |
| `S3_PREFIX` | No | Object prefix; defaults to `db-backups` |

Provide these variables using the deployment host's secure environment or
secret manager. The restore script is destructive: it drops and recreates the
target database after interactive confirmation. Review
[database backup and restore](database-backup-restore.md) before using it.

## Optional Deployment Notification

[`deploy/scripts/deploy.sh`](../deploy/scripts/deploy.sh) can post rollback
alerts to Slack when `SLACK_WEBHOOK_URL` is set in its host process
environment. The main documented entry point,
[`scripts/blue-green-deploy.sh`](../scripts/blue-green-deploy.sh), does not
require a notification secret. Neither script uses `DISCORD_WEBHOOK_URL`.
