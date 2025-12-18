# n8n-template

Reusable n8n assets plus a Grafana dashboard for monitoring. Each workflow directory has its own README; this root file gives you the bird's-eye view.

## Contents
- DevOps/Invoice-Check.json — Daily CSV-driven domain expiry checker that posts JSON results to Telegram (Jalali date aware).
- Blockchain/Full_Node_Sync_Monitor_BSC.json — Monitors a BSC full node's sync status via eth_syncing, alerts on Telegram/Discord.
- n8n-Monitoring.json — Grafana dashboard for Prometheus-scraped n8n metrics (success rates, queue stats, CPU/RSS, GC, cache, etc.).
- Setup/ — Docker Compose recipes for single-node and master/worker n8n deployments.

## Quick start
1) Import workflows: In n8n, Import from File -> pick the JSON in DevOps or Blockchain.
2) Wire credentials: Provide OpenAI/Telegram (Invoice Check) and Telegram/Discord + node URL (BSC monitor).
3) Test with sample data: For Invoice Check, place domains.csv at /home/exfiles/domains.csv in the n8n runtime and run once.
4) Monitoring: Import n8n-Monitoring.json into Grafana, point the Prometheus datasource at your n8n metrics endpoint, and select job/instance variables.

## Setup (Docker)
- Single-node: `Setup/Singel-Node/` — Postgres + n8n on one host; fill `.env`, run `docker compose up -d`, access port 5678.
- Master/worker: `Setup/Worker-Node/master` for Postgres + Redis + n8n (queue mode), and `Setup/Worker-Node/worker` for workers/runners; share `ENCRYPTION_KEY` and `RUNNERS_TOKEN`, point worker DB/Redis hosts to the master.
- Full instructions live in `Setup/README.md`.

## n8n Monitoring dashboard (n8n-Monitoring.json)
- Import into Grafana; set the Prometheus datasource (default UID in the JSON is PBFA97CFB590B2093). Variables: `job` and `instance`; refresh: 10s.
- Overview stats: service up/down, n8n version, Node.js version, leader flag, active workflow count, last activity age, uptime, FD usage %.
- Throughput & reliability: workflow success/failed rates (5m), success rate %, queue completed/failed (5m), queue waiting/active sizes, HTTP request duration avg/p95.
- Runtime health: CPU rate, RSS, heap used vs total, event loop lag (p50/p90/p99), GC duration by kind (major/minor/incremental).
- Cache & I/O: cache hit ratio/hits/misses/updates, open file descriptors.
- Per-instance breakdown: tables for success/failed rate and CPU/RSS per instance, plus top error-rate instances (timeseries).

## Notes
- Schedules: Invoice Check runs daily at 09:00 Asia/Tehran; BSC monitor polls every 10 minutes by default.
- Timezone: Invoice Check uses Jalali (Shamsi) date in its LLM prompt; adjust if your locale differs.
- Feel free to fork workflows for additional transports (Slack/Email) or stricter validation.
