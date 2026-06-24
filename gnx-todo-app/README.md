# GNX Todo App — Starter Repository

Welcome. This is the starter application for the **GNX Solutions DevOps Engineer Machine Test**.

Read `MACHINE_TEST.md` for the full instructions.

---

## App Architecture

```
Browser → Frontend (Nginx :80) → /api/* → Backend (Node.js :3000) → PostgreSQL (:5432)
```

## Services

| Service | Tech | Port | Description |
|---|---|---|---|
| `frontend` | Nginx + HTML/JS | 80 | Serves the UI, proxies /api to backend |
| `backend` | Node.js Express | 3000 | REST API for todos |
| `postgres` | PostgreSQL 15 | 5432 | Stores todos |

## API Endpoints

| Method | Path | Description |
|---|---|---|
| GET | /health | Health check |
| GET | /todos | List all todos |
| POST | /todos | Create a todo — body: `{ "title": "..." }` |
| DELETE | /todos/:id | Delete a todo by ID |

## Backend Environment Variables

| Variable | Default | Description |
|---|---|---|
| DB_HOST | localhost | Postgres hostname |
| DB_PORT | 5432 | Postgres port |
| DB_NAME | tododb | Database name |
| DB_USER | postgres | Database user |
| DB_PASSWORD | postgres | Database password |
| PORT | 3000 | Backend listen port |

## Run Locally

```bash
docker compose up --build
```

Open http://localhost:8080 — you should see the Todo app.

---

> Your task is in `MACHINE_TEST.md`. Good luck.
