# Docker Swarm Stack Releases

## Setup Portainer

### Adding a stack

In the environment, add a new stack following these steps:

1. Name the stack according to the docker-compose YAML file name in this repo.
1. Configure the stack to pull from a git repository.
1. Enter in the details for this repo.
   - Repository URL: `<url>`
   - Repository reference: `refs/heads/<branch>`
1. Enter the name of the docker-compose YAML file.
1. Enable GitOps updates.
1. Configure `Fetch interval` with an interval of `5m`.
1. Configure `Environment variables`. Refer to the `*.env` files published with the release assets. Copy their contents into Portainer's **Environment variables** section (toggled to "Advanced mode") and edit as required.

## Configuration options

Use the selected template's header for its environment variables and data-directory setup.

### Object storage

For OpenObserve, choose `ZO_S3_PROVIDER=minio` (default), `aws` or `azure`. Create the bucket or container before starting the stack.

- **MinIO:** set `ZO_S3_SERVER_URL` to the S3 API URL, e.g. `http://minio:9000`, and set the bucket, region and access keys.
- **AWS S3:** leave `ZO_S3_SERVER_URL` empty and set the bucket, AWS region and access keys.
- **Azure Blob:** leave `ZO_S3_SERVER_URL`, `ZO_S3_ACCESS_KEY` and `ZO_S3_SECRET_KEY` empty, and set the container name, `AZURE_STORAGE_ACCOUNT_NAME` and `AZURE_STORAGE_ACCOUNT_KEY`.

### MinIO / S3 access policy

OpenObserve needs bucket read/write/delete permissions. Replace `openobserve-data` with your bucket name in this policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::openobserve-data",
        "arn:aws:s3:::openobserve-data/*"
      ]
    }
  ]
}
```

### OpenObserve metadata database

| Mode                | `ZO_META_STORE` | `POSTGRES_REPLICAS` | `ZO_META_POSTGRES_DSN`                                        |
| ------------------- | --------------- | ------------------- | ------------------------------------------------------------- |
| SQLite (default)    | `sqlite`        | `0`                 | Empty                                                         |
| Stack PostgreSQL    | `postgres`      | `1`                 | `postgres://openobserve:<password>@postgres:5432/openobserve` |
| External PostgreSQL | `postgres`      | `0`                 | External database connection URL                              |

For stack PostgreSQL, match the DSN to `POSTGRES_USER`, `POSTGRES_PASSWORD` and `POSTGRES_DB`, and prepare `POSTGRES_DATA_PATH` as described in the header. URL-encode reserved characters in DSN credentials.

### OpenObserve Traefik

Enable Traefik routing with `TRAEFIK_ROUTER_ENABLED=true` and set `TRAEFIK_DOMAIN`; otherwise use the published host ports. Traefik should use `--providers.swarm.network=swarm-public`.

### OpenObserve Dex

Dex is disabled by default. Enable it with `DEX_ENABLED=true` and `DEX_REPLICAS=1`, then configure the client and connector credentials from the header. Dex requires public HTTPS and the upstream callback `https://<TRAEFIK_DOMAIN>/dex/callback`.

## Upgrading existing stacks

Back up OpenObserve metadata and persistent data before upgrading. This template targets `0.60.5`, the first step from `0.50` in the [staged upgrade path](https://github.com/openobserve/openobserve/discussions/12356). Database schema migrations run during startup; rehearse this step on a restored backup and validate it before advancing to the next release line. Changing `ZO_META_STORE` also requires a separate metadata/file-list migration.

PostgreSQL stays on major version 17. The [17.12 release notes](https://www.postgresql.org/docs/release/17.12/) require no dump/restore for an existing 17.x database, but describe conditional reindexing, C-extension rebuilds and taking a new full backup if using incremental backups. Review those notes against the deployed database before updating.

The MinIO template still uses the legacy community image. The community repository is archived and even its final release has [unfixed authentication-bypass advisories](https://github.com/minio/minio/security/advisories/GHSA-9c4q-hq6p-c237). A supported replacement or AIStor migration is required; changing to the last community tag does not resolve these findings.
