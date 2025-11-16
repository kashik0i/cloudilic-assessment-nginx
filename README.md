# Cloudilic Assessment — NGINX Orchestration

This repository contains the docker-compose setup and NGINX configuration that orchestrate the Cloudilic assessment stack:
- PostgreSQL with pgvector (database)
- Backend API (Node + pnpm)
- Frontend (Vite/React) built and served by NGINX
- NGINX reverse proxy/gateway

## Repository structure
- `docker-compose.yml` — Orchestrates DB, backend, frontend build, and NGINX
- `nginx/` — All NGINX configs (gateway, upstreams, client and backend locations)
- `scripts/clone.sh` — Helper script to clone the frontend and backend repositories into this workspace
- `cloudilic-assessment-frontend/` — Frontend app (cloned or present)
- `cloudilic-assessment-backend/` — Backend app (cloned or present)

## Prerequisites
- Docker and Docker Compose v2
- Git
- Open local ports: 9015 (NGINX), 4000 (backend), 9898 (Postgres host mapping)

## Quick start (local)
1) Clone this repo

2) Clone the application repos using the helper script. If you see a permission error, make the script executable first:

```bash
chmod +x scripts/clone.sh
./scripts/clone.sh
```

3) Start the stack

```bash
docker compose up --build
```

4) Access the services
- Frontend (served by NGINX): http://localhost:9015/
- API via NGINX gateway: http://localhost:9015/api
- Database: host port 9898 → container 5432 (DB: `ragdb`, user: `postgres`, password: `postgres`)

## How it works
- Frontend container builds the app and copies the output to a shared path: `./.repository/build/cloudilic-assessment-frontend`
- NGINX serves those built assets from `/usr/share/nginx/html/cloudilic-assessment-frontend`
- Backend listens on port `4000` inside the network and is proxied by NGINX under `/api`
- Postgres uses the `ankane/pgvector` image; data is persisted in the `pgdata` volume

## Configuration notes
- NGINX listens on port `9015` by default (see `nginx/conf.d/cloudilic-assessment.conf`)
- An environment variable `CERTBOT_EMAIL` is set in `docker-compose.yml` for the `jonasal/nginx-certbot` image. For local development on `localhost`, TLS issuance isn’t used; this image works fine in HTTP-only mode on a custom port.

## Common issues
- Permission denied when running the clone script:
  - Run: `chmod +x scripts/clone.sh`
- Frontend not updating after changes:
  - Rebuild the frontend container or rerun `docker compose up --build`
- Port conflicts:
  - Change host ports in `docker-compose.yml` (e.g., `9015`, `9898`) if they are already in use

## Cleaning up
To stop and remove containers (keeping volumes):
```bash
docker compose down
```
To remove volumes as well (this deletes DB data):
```bash
docker compose down -v
```

## Todo
- Automate frontend/backend cloning on first run
- Configure domains and TLS if deploying beyond localhost
- Add CI to build and validate NGINX configuration changes

