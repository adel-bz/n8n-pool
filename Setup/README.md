# Setup

Deployment recipes for n8n, either single-node or a master/worker cluster with runners.

## Layout
- `Singel-Node/` — Simple Postgres + n8n on one host.
- `Worker-Node/master/` — Master stack: Postgres + Redis + n8n (queue mode, external runners enabled).
- `Worker-Node/worker/` — Worker stack: two n8n workers plus two runners, pointing to the master.

## Prereqs
- Docker + docker-compose.
- Static IP planning if you keep the default `172.110.0.0/24` bridge and the sample master IP `192.168.1.2` used by workers.
- Values for DB credentials, `ENCRYPTION_KEY`, and `RUNNERS_TOKEN` (must match across master and workers).
- A public `WEBHOOK_URL` (https) for production use.

## Single-node (Singel-Node)
1) Copy `.env` and fill: `POSTGRES_USER/PASSWORD/DB`, `POSTGRES_NON_ROOT_USER/PASSWORD`, `N8N_ENDPOINT_WEBHOOK_TEST`.
2) (Optional) edit `docker-compose.yml` ports if `5678` conflicts.
3) Start: `docker compose up -d`.
4) Access n8n: http://localhost:5678 (or your host binding).

## Master/worker cluster
Master (`Worker-Node/master`):
1) Fill `.env` with Postgres creds, `ENCRYPTION_KEY`, `RUNNERS_TOKEN`, `WEBHOOK_URL`.
2) If ports conflict, adjust `5433` (host), `6380` (Redis host), `5663` (n8n UI bind), and/or static IPs in `docker-compose.yml`.
3) Start: `docker compose up -d` (brings up Postgres, Redis, n8n master).

Workers (`Worker-Node/worker`):
1) Fill `.env` with the same DB creds, `ENCRYPTION_KEY`, and `RUNNERS_TOKEN` used on the master.
2) In `docker-compose.yml`, set `DB_POSTGRESDB_HOST` and `QUEUE_BULL_REDIS_HOST` to the master’s reachable IP (sample uses `192.168.1.2`). Adjust host ports if your master uses different ones.
3) Ensure the worker subnet (`172.110.0.0/24`) does not clash locally; change if needed along with the static IPs.
4) Start: `docker compose up -d` (spins two workers + two runners).

## Notes
- The init scripts (`init-data.sh`) create the non-root Postgres user on first boot if env vars are set.
- Master n8n runs in queue mode; workers process jobs; runners are enabled externally with the shared `RUNNERS_TOKEN`.
- Bind UI ports to localhost or secure with firewall/ingress if exposed.
- Enable TLS/HTTPS at your reverse proxy for production webhooks and UI.
