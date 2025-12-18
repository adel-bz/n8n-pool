# Full Node Sync Monitor (BSC)

A lightweight n8n workflow that polls a Binance Smart Chain full node via `eth_syncing` and alerts you if the node is still syncing or unhealthy.

## What it does
- Runs every 10 minutes by default (Schedule Trigger).
- Calls your node's JSON-RPC endpoint with `eth_syncing`.
- If the response is not `false`, the node is still syncing or degraded.
- Sends an alert to Telegram; HTTP errors can also alert via Discord.

## Requirements
- An accessible BSC full node URL (e.g., `http://YOUR_NODE_IP:8545`).
- Telegram bot token + chat ID (for the `Send Alert` node) or Discord webhook (for the `Failed Request` node).
- n8n instance where you can import and run the workflow JSON.

## Setup
1) In n8n, import `Blockchain/Full_Node_Sync_Monitor_BSC.json`.
2) Set the HTTP Request URL to your node's RPC endpoint.
3) Configure credentials:
   - Telegram: chat ID + API credential on `Send Alert` (or swap to Discord only).
   - Discord: webhook credential on `Failed Request` (optional if you only use Telegram).
4) Adjust schedule if needed (e.g., every 1 minute for tighter monitoring).
5) Activate the workflow.

## Alerts
- Telegram message: "The BSC full node is not synced with the blockchain".
- Discord embed (optional): "[Firing] BSC Node – The BSC node is not up".

## Tips
- Keep the node endpoint local/private; avoid exposing public RPC without protection.
- If you run multiple nodes, duplicate the workflow with different URLs and chat IDs.

