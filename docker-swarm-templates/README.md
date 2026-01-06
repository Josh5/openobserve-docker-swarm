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
1. Configure Polling updates with an interval of `5m`.
1. Configure Environment Variables. Refer to the `*.env` files published with the release assets. Copy their contents into Portainer's **Environment variables** section (toggled to "Advanced mode") and edit as required.

## OpenObserve stack options

You can deploy OpenObserve using one of these stacks:

- `docker-compose.openobserve-stack.yml` (OpenObserve only)
- `docker-compose.openobserve-postgres-stack.yml` (OpenObserve + Postgres 17)

If you are using Postgres, set the metadata store values in the OpenObserve env file and point the DSN at the `postgres` service.

## Postgres notes

The combined OpenObserve + Postgres stack uses `postgres:17-bookworm` and stores data in `POSTGRES_DATA_PATH`. The Postgres port is published to the host for administration, but OpenObserve connects to Postgres through the internal `private-net` network.

## MinIO / S3 access policy

OpenObserve needs bucket read/write/delete permissions. A minimal bucket policy looks like this:

```
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
