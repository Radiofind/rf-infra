# RF-Infra (RadioFind Infrastructure)

Docker Compose setup that runs the whole **RadioFind** stack locally: PostgreSQL, the Spring Boot backend, the Angular SSR frontend and an nginx reverse proxy in front of them.

## 🧱 Services

| Service    | Image / build context            | Port (container) | Exposed on host | Purpose                          |
| ---------- | -------------------------------- | ---------------- | --------------- | -------------------------------- |
| `db`       | `postgres:17`                    | 5432             | `5433`          | Main database (`radiofind`)      |
| `backend`  | built from `../rf-back`          | 8080             | —               | REST API (Spring Boot)           |
| `frontend` | built from `../rf-front`         | 4000             | —               | Angular SSR server               |
| `nginx`    | `nginx:alpine`                   | 80               | `80`            | Reverse proxy, single entrypoint |

Routing in [nginx/nginx.conf](nginx/nginx.conf):

- `/api/*` → `backend:8080`
- everything else → `frontend:4000`

```
browser ──► nginx :80 ──┬── /api/ ──► backend :8080 ──► db :5432
                        └── /     ──► frontend :4000
```

## 📁 Expected directory layout

The backend and frontend are built from sibling repositories, so they must be checked out next to this one:

```
radiofind/
├── rf-back/     # git@github.com:Radiofind/rf-back.git
├── rf-front/
└── rf-infra/    # this repository
```

## ⚙️ Configuration

Copy the example env file and fill in the values:

```bash
cp .env.example .env
```

| Variable        | Description                                                                 |
| --------------- | --------------------------------------------------------------------------- |
| `MAIL_USERNAME` | SMTP login (Gmail account) used by the backend to send emails              |
| `MAIL_PASSWORD` | SMTP password — for Gmail use an [app password](https://myaccount.google.com/apppasswords) |
| `JWT_SECRET`    | Secret used to sign JWT tokens, e.g. `openssl rand -base64 64`             |
| `FRONTEND_URL`  | Public URL of the frontend, used in links sent by the backend (`http://localhost`) |

`MAIL_HOST` and `MAIL_PORT` are listed in `.env.example`, but the backend currently uses `smtp.gmail.com:587` from its own `application.yml`.

Database credentials (`postgres` / `postgres`, database `radiofind`) are set directly in [docker-compose.yml](docker-compose.yml) and are intended for local development only.

`.env` is git-ignored — never commit real secrets.

## 🚀 Running

Build and start everything:

```bash
docker compose up -d --build
```

Then open <http://localhost>. The API is available under <http://localhost/api/>.

Useful commands:

```bash
docker compose ps                  # service status
docker compose logs -f backend     # follow backend logs
docker compose up -d --build backend   # rebuild a single service after code changes
docker compose down                # stop the stack
docker compose down -v             # stop and delete the database volume
```

## 🗄️ Database access

PostgreSQL is published on host port **5433** (to avoid clashing with a local Postgres on 5432):

```bash
psql -h localhost -p 5433 -U postgres -d radiofind
```

Data is persisted in the `db-data` named volume. The schema is managed by Hibernate (`ddl-auto: update`) on backend startup.
