# OpenObserve on Docker Swarm

## Development setup

From the root of this project, run these commands:

1. Create the `***.env` files

   ```
   ./scripts/init-docker-compose-stack-env.sh ./docker-swarm-templates/docker-compose.minio.yml
   ./scripts/init-docker-compose-stack-env.sh ./docker-swarm-templates/docker-compose.openobserve-stack.yml
   ```

2. Modify any additional config options in the `***.env` files.

3. Run the dev compose stacks

   ```
   sudo docker compose \
       -f ./docker-swarm-templates/docker-compose.minio.yml \
       --env-file ./docker-swarm-templates/minio.env \
       up -d

   sudo docker compose \
       -f ./docker-swarm-templates/docker-compose.openobserve-stack.yml \
       --env-file ./docker-swarm-templates/openobserve-stack.env \
       up -d
   ```

## PostgreSQL setup (optional)

If you want to use PostgreSQL for the OpenObserve metadata store, create the env files
and start the Postgres stack:

```
./scripts/init-docker-compose-stack-env.sh ./docker-swarm-templates/docker-compose.postgres.yml

sudo docker compose \
    -f ./docker-swarm-templates/docker-compose.postgres.yml \
    --env-file ./docker-swarm-templates/postgres.env \
    up -d
```

Then update `docker-swarm-templates/openobserve-stack.env`:

```
ZO_META_STORE=postgres
ZO_META_POSTGRES_DSN=postgres://openobserve:supersecret@<stack>_postgres:5432/openobserve
```

## MinIO setup

Before starting OpenObserve with S3 storage, create a bucket in MinIO that matches
`ZO_S3_BUCKET_NAME` and create an access key that maps to `ZO_S3_ACCESS_KEY` and
`ZO_S3_SECRET_KEY` in `docker-swarm-templates/openobserve-stack.env`.
