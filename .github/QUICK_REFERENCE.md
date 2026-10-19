# GitHub Actions Quick Reference

## Workflows and secrets

The CI, end-to-end, and security workflows do not need deployment secrets.
There is no GitHub Actions workflow for staging or production deployment, so
do not add SSH, AWS, database, Discord, or Slack credentials to repository or
environment secrets for deployment.

Production deployments are operator-run on the target host with
[`scripts/blue-green-deploy.sh`](../scripts/blue-green-deploy.sh). The
[deployment runbook](../docs/deployment.md) describes the full procedure and
the [secrets reference](../docs/ci-secrets.md) lists the host/runtime values
used by Compose and database backup/restore.

## References

- [GitHub Actions overview](./README.md)
- [Deployment runbook](../docs/deployment.md)
- [CI and deployment secrets](../docs/ci-secrets.md)
- [Database backup and restore](../docs/database-backup-restore.md)
