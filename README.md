# RF-Infra (RadioFind Infrastructure)

Docker Compose setup that runs the whole **RadioFind** stack locally: PostgreSQL, the Spring Boot backend, the Angular SSR frontend and an nginx reverse proxy in front of them.

## 🧱 Services

| Service    | Image / build context            | Port (container) | Exposed on host | Purpose                          |
| ---------- | -------------------------------- | ---------------- | --------------- | -------------------------------- |
| `db`       | `postgres:17`                    | 5432             | `5433`          | Main database (`radiofind`)      |
| `backend`  | built from `../rf-back`          | 8080             | —               | REST API (Spring Boot)           |
| `frontend` | built from `../rf-front`         | 4000             | —               | Angular SSR server               |
| `nginx`    | `nginx:alpine`                   | 80               | `80`            | Reverse proxy, single entrypoint |
| `mailpit`  | `axllent/mailpit`                | 1025 / 8025      | `8025`          | Local SMTP server and mail inbox |

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
| `JWT_SECRET`    | Secret used to sign JWT tokens, e.g. `openssl rand -base64 64`             |
| `FRONTEND_URL`  | Public URL of the frontend, used in links sent by the backend (`http://localhost`) |

## 📬 Email

The backend sends every email (two-factor codes, password reset links) to Mailpit, never to a real mailbox. Read them at <http://localhost:8025>.

The SMTP settings are set in [docker-compose.yml](docker-compose.yml) and override any `MAIL_*` value in `.env`. To send through a real SMTP server instead, change `MAIL_HOST`, `MAIL_PORT`, `MAIL_USERNAME`, `MAIL_PASSWORD`, `MAIL_SMTP_AUTH` and `MAIL_SMTP_STARTTLS` there.

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
