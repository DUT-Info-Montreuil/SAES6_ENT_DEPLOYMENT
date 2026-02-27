# ENT (private) Deployment

Docker Compose setup for the ENT backend (Flask + Gunicorn), frontend (Angular build served by Nginx), and the Postgres database.

## Assumptions

- Single-host Docker deployment
- No HTTPS termination provided by this stack
- No horizontal scaling

## Prerequisites

- Docker Engine 24+ and Docker Compose v2
- Git (with submodule support)
- Open ports: 8080 (frontend), 5000 (backend)

## Installation

1) Clone the repository and submodules (on the `main` branch):
```bash
git clone -b main --recurse-submodules [https://github.com/DUT-Info-Montreuil/SAES6_ENT_DEPLOYMENT.git](https://github.com/DUT-Info-Montreuil/SAES6_ENT_DEPLOYMENT.git)
cd SAES6_ENT_DEPLOYMENT

# Force all submodules to checkout their main branch (avoids detached HEAD)
git submodule foreach git checkout main
git submodule foreach git pull origin main
```

2) Configure the backend:

Create or edit `backend/SAES6_ENT_BACKEND/.env` with values for your environment. See the backend README to get a full list of variables.

`DATABASE_URL` and `SERVER_PORT` are set by `docker-compose.yml`. If you want to change the port or database settings, update the `docker-compose.yml` accordingly.

3) Configure the frontend:

Edit `frontend/SAES6_ENT_PRIVATEFRONT/src/environments/environment.production.ts`
and set `apiUrl` to the backend base URL.

4) Configure the database:

The Postgres container is preconfigured with a database, user, and password via environment variables in `docker-compose.yml`. If you want to change these settings, update the `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `POSTGRES_DB` variables in the `database` service section.
You will also need to update the `DATABASE_URL` in the backend `.env` file accordingly.

Database data is persisted in a Docker volume named `db-data`. To access it, use:
```
docker compose exec database psql -U user -d db
```

5) Build and start the containers:
```
docker compose up -d --build
```

6) Open the app:
- Frontend: http://localhost:8080
- Backend: http://localhost:5000

Or wherever you configured the ports to be.

## Operations guide

Start and stop:
```
docker compose up -d
docker compose down
```

Check status and container health:
```
docker compose ps
```

View logs (containers log to stdout/stderr):
```
docker compose logs -f
docker compose logs -f back
docker compose logs -f front
docker compose logs -f database
```

Restart after a crash:
- Containers are set to `restart: unless-stopped` and will auto-restart.
- Manual restarts:
```
docker compose restart back
docker compose restart front
```

Apply configuration changes:
- Backend `.env` changes: restart the backend container.
```
docker compose restart back
```
- Frontend `apiUrl` changes: rebuild the frontend image.
```
docker compose up -d --build front
```

Data and persistence:
- Backend logs are stored in the `logs` Docker volume.
- The database data is persisted in a Docker volume named `db-data`.

Database access and backups:
```
docker compose exec database psql -U user -d db
docker compose exec database pg_dump -U user db > backup.sql
```

To restore a backup:
```
cat backup.sql | docker compose exec -T database psql -U user -d db
```

Update and redeploy:
```
git pull
git submodule update --init --recursive
docker compose up -d --build
```

## Logging

- Logs are written to the `logs` volume.
- You are responsible for log rotation.

## Failure scenarios

### Database container crash
- Backend will fail to connect
- Once database restarts, backend auto-recovers

### Backend crash
- Frontend remains available but API calls fail
- Backend auto-restarts due to restart policy

## Troubleshooting

- Port already in use: change the host port mappings in `docker-compose.yml`.
- Backend cannot reach database: wait for Postgres to finish starting or check
  `docker compose logs database`.
- Frontend shows blank/failed API calls: verify `apiUrl` and rebuild the frontend.

## Security considerations

Secrets are stored in plaintext `.env` files.
This setup should NOT be exposed directly to the public internet without HTTPS termination and proper secret management.

We assume you will run this behind a secure reverse proxy and use docker secrets in production.
