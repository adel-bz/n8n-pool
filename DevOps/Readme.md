# Invoice Check (Domain Expiry Monitor)

Daily n8n workflow that pulls a CSV from GitLab, filters for domains expiring within 1 day via an LLM, and posts the JSON result to Telegram. Uses the Jalali (Shamsi) calendar for "today".

## What it does
- Runs daily at 09:00 (Asia/Tehran).
- Fetches `Domain-Payment.csv` from GitLab repo `devops/domains-payment` (branch `main`).
- Parses rows, renders a Markdown table for the LLM prompt, and provides today’s Jalali date (Latin digits).
- OpenAI node (`gpt-5` in sample) returns strict JSON for domains expiring in 1 day.
- If node sanity-checks the JSON (looks for "Domain 1" and "Handler Account"), then sends to Telegram.

## Requirements
- n8n (self-hosted or Cloud).
- GitLab API credential with read access to `devops/domains-payment`.
- OpenAI API credential for `@n8n/n8n-nodes-langchain.openAi` (model `gpt-5` by default).
- Telegram bot token + chat ID.
- CSV schema should include the fields you want returned (e.g., Domain 1, Expire Date, Handler, Handler Account).

## Setup
1) Import `DevOps/Invoice-Check.json` into n8n.
2) Set credentials:
   - GitLab on “Get a file” (repo: `domains-payment`, owner: `devops`, file: `Domain-Payment.csv`, ref: `main`).
   - OpenAI on “Check Domains”.
   - Telegram on “Send Result”.
3) Adjust schedule/timezone if needed (default 09:00 Asia/Tehran).
4) Run once to verify end-to-end.

## Workflow nodes (in order)
1. Schedule Trigger
2. Current Time (Jalali)
3. Change Date Format (optional)
4. Get a file (GitLab CSV)
5. Extract CSV File
6. Make a Table (Code)
7. Merge (date + table)
8. Check Domains (OpenAI)
9. Validate Output (If)
10. Send Result (Telegram)

## Output shape (LLM instruction)
```
[
  {
    "Domain 1": "example.com",
    "Expire Date": "1404/06/12",
    "Days Left to Expiration": 1,
    "Handler": "Namecheap",
    "Handler Account": "example@gmail.com"
  }
]
```

## Hardening tips
- Replace the If node with a Code validator that parses JSON and checks required fields.
- Add retry/backoff on the OpenAI node if you expect rate limits.
- Add Slack/Email nodes after Telegram for multi-channel alerts.
