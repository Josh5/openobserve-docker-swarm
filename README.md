# OpenObserve on Docker Swarm

Docker Swarm templates for OpenObserve, with optional PostgreSQL and Dex services. One OpenObserve template supports SQLite, bundled PostgreSQL or external PostgreSQL, with or without Traefik and Dex.

## Deployment

Follow the [deployment instructions](docker-swarm-templates/README.md). MinIO is deployed separately using the [MinIO template](docker-swarm-templates/docker-compose.minio.yml), or use an existing MinIO, AWS S3 or Azure Blob service.
