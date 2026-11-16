# Module 5 — Part 3: Multi-Language Microservices, CI/CD & Vagrant Deployment

Pixel River Financial Bank Services — banking app as microservices, fronted by Nginx.

## Services

| Service | Language / image | Port | What it does |
| --- | --- | --- | --- |
| mongo | mongo:6 | 27017 (internal) | MongoDB, database `bank_app`, named volume `mongo_data` |
| backend | Python 3.10 / Flask | 5000 | Register, login, dashboard, deposit, withdraw; `/api/transactions` (logged-in user's transactions grouped by month) |
| transactions | Node 18 / Express | 3000 | JavaScript microservice; `GET /api/transactions/<userId>` returns that user's transactions grouped by month and year |
| studentportfolio | nginx:alpine (static) | 80 (internal) | Student portfolio, served through Nginx |
| nginx | nginx:alpine | 80 | Reverse proxy and single entry point at http://localhost/ |

## How Nginx routes requests

- `GET /api/transactions` (exact) → backend — the grouped monthly JSON for the logged-in user; the dashboard "View Transactions" button fetches this URL.
- `GET /api/transactions/<userId>` → transactions service (JavaScript).
- All other `/api/...`, `/login`, `/register`, `/dashboard`, `/deposit`, `/withdraw`, `/logout` → backend.
- Everything else (`/`) → studentportfolio.

## Run it

```powershell
cd "E:\rrc_polytech\Term 4\devops\module5\part3"
docker compose up --build -d
docker compose ps
```

Then open http://localhost/ in the browser. Stop Part 1 and Part 2 first — they use the same ports (80, 3000, 5000) and cannot run at the same time.

```powershell
docker compose down -v
```

Do not run `docker rmi $(docker images -aq)` until the whole assignment is finished.

## Tests and CI

Backend tests live in `backend/tests/` and use mongomock, so no real database is needed:

```powershell
cd backend
pip install -r requirements.txt
pytest -q tests/
```

`.github/workflows/ci.yml` runs the same tests on GitHub Actions on every push and pull request to `main`: it checks out the code, sets up Python 3.13.3, starts a MongoDB service container, installs the backend dependencies, and runs `pytest -q tests/`.

## Repository

GitHub repo: `module5-part3-rdobariya2` (created when the local run is verified — see the Part 3 document).
