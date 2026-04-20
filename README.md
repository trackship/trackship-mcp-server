# TrackShip MCP Server

Model Context Protocol (MCP) server for [TrackShip](https://trackship.com) — shipment tracking platform.

Connect AI assistants like Claude to your TrackShip account to query shipments, tracking checkpoints, orders, and store information through natural conversation.

---

## Features

- Get shipment status by tracking number
- Get full tracking checkpoints by tracking number
- List shipments with filters (carrier, status, date range, pagination)
- Get order tracking by order ID
- List connected stores
- Get account plan information

---

## Setup

### 1. Get your API Key

Log in to your TrackShip account → **Settings → API Key**

### 2. Add to Claude Desktop

Open your Claude Desktop config file:

- **Mac:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

Add the following:

```json
{
  "mcpServers": {
    "trackship": {
      "type": "http",
      "url": "https://api.trackship.com/v1/mcp",
      "headers": {
        "trackship-api-key": "YOUR_TRACKSHIP_API_KEY",
        "app-name": "YOUR_APP_NAME"
      }
    }
  }
}
```

Replace `YOUR_TRACKSHIP_API_KEY` with your actual key and `YOUR_APP_NAME` with your registered app name.

### 3. Restart Claude Desktop

Once restarted, TrackShip tools will be available in your Claude conversations.

---

## Available Tools

| Tool | Description | Input |
|---|---|---|
| `get_shipment` | Get status + checkpoints for a shipment | `tracking_number`, `tracking_provider` (optional) |
| `list_shipments` | List shipments with optional filters | `tracking_provider`, `status`, `date_from`, `date_to`, `page`, `per_page` |
| `get_order_tracking` | Get tracking info by order ID | `order_id` |
| `list_stores` | List all connected stores | — |
| `get_account_plan` | Get current plan and usage information | — |

---

## Example Prompts

Once connected, you can ask Claude things like:

- *"What's the status of tracking number 9400111899223397867254?"*
- *"Show me all delayed shipments from last week"*
- *"List all USPS shipments from April 1 to April 15"*
- *"What's the tracking status for order #1042?"*
- *"Which stores are connected to my TrackShip account?"*
- *"What plan am I on and how many shipments have I used?"*

---

## Authentication

All requests require the following headers:

```
trackship-api-key: YOUR_TRACKSHIP_API_KEY
app-name: YOUR_APP_NAME
```

Keys are scoped to your account and stores.

---

## Data Privacy

When using this MCP server with an AI assistant, shipment data including tracking numbers, order IDs, and delivery statuses are shared with your chosen AI provider (e.g., Anthropic for Claude). Please review your provider's data policy before use.

---

## MCP Endpoint

All tools are accessible via a single endpoint:

```
POST https://api.trackship.com/v1/mcp
```

---

## Links

- [TrackShip](https://trackship.com)
- [TrackShip Help Center](https://help.trackship.com)
- [TrackShip Docs](https://docs.trackship.com)
- [Get your API Key](https://my.trackship.com/settings/#api)
