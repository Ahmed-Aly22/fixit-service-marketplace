# Local Docker Development

The repository currently provides a PostgreSQL container for local development. The web app, API, and worker do not exist yet, so they are not containerized.

## Requirements

- Docker Desktop installed and running.
- Docker Compose v2 or newer.

## Start PostgreSQL

From the repository root in PowerShell:

```powershell
Copy-Item .env.example .env
```

Edit `.env` and replace `POSTGRES_PASSWORD` and the matching password in `DATABASE_URL` with a local development password. `.env` is ignored by Git; do not commit it.

Start the database and wait for its health check:

```powershell
docker compose up -d --wait postgres
docker compose ps
```

The database is published only on `127.0.0.1`. Use the `DATABASE_URL` from `.env` when configuring a local API process.

PostgreSQL data is kept in the `postgres_data` Docker volume and survives container restarts and `docker compose down`. Removing that volume permanently deletes the local database contents.

Stop the container while keeping its data:

```powershell
docker compose down
```

## Troubleshooting

- If Docker reports that it cannot connect to the engine, start Docker Desktop and retry.
- If port `5432` is already in use, change `POSTGRES_PORT` in `.env`, then use that port in `DATABASE_URL`.
- Check container status and logs with `docker compose ps` and `docker compose logs postgres`.
