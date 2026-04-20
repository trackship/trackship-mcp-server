# TrackShip MCP — Tool Definitions

This file is the spec for building `POST https://api.trackship.com/v1/mcp`.

---

## How It Works

One endpoint handles all tools. PHP routes internally by the `tool` field.

```
POST https://api.trackship.com/v1/mcp
trackship-api-key: <user api key>
app-name: <user app name>
Content-Type: application/json

{
  "tool": "get_shipment",
  "input": {
    "tracking_number": "9400111899223397867254"
  }
}
```

---

## What Needs to Be Built

| # | What | Type | Notes |
|---|---|---|---|
| 1 | `POST /v1/mcp` | New endpoint | MCP router — routes to internal logic by `tool` name |
| 2 | `POST /v1/order/tracking/` | New endpoint | Replaces deprecated `/v1/shipment/by-order-id` |
| 3 | `POST /v1/shipments/` | New endpoint | List shipments with filters + pagination |
| 4 | `list_stores` logic | New logic | MCP only for now |
| 5 | `get_account_plan` logic | New logic | MCP only for now |

`get_shipment` reuses existing `/v1/shipment/get/` logic internally — no rebuild needed.

---

## PHP Router Structure

```php
// POST /v1/mcp
$tool  = $request['tool'];
$input = $request['input'];

switch ( $tool ) {
    case 'get_shipment':       return mcp_get_shipment( $input );
    case 'list_shipments':     return mcp_list_shipments( $input );
    case 'get_order_tracking': return mcp_get_order_tracking( $input );
    case 'list_stores':        return mcp_list_stores( $input );
    case 'get_account_plan':   return mcp_get_account_plan( $input );
    default:
        return [ 'status' => 'error', 'message' => 'Unknown tool: ' . $tool ];
}
```

---

## Tool 1 — `get_shipment`

Reuses existing `/v1/shipment/get/` logic internally.

**Input:**
```json
{
  "tracking_number": "9400111899223397867254",
  "tracking_provider": "usps"
}
```

| Parameter | Type | Required | Notes |
|---|---|---|---|
| tracking_number | String | Yes | |
| tracking_provider | String | No | Auto-detected if omitted |

**Response (success):**
```json
{
  "status": "success",
  "data": {
    "order_id": "24",
    "tracking_number": "9212490276905104629392",
    "tracking_provider": "usps",
    "tracking_event_status": "in_transit",
    "tracking_est_delivery_date": null,
    "shipping_service": "Priority Mail",
    "last_event_time": "2023-02-09 10:38:00",
    "events": [
      {
        "message": "Picked Up by Shipping Partner",
        "status": "pre_transit",
        "datetime": "2023-02-09 06:24:00",
        "source": "USPS",
        "tracking_location": {
          "city": "LOS ANGELES",
          "state": "CA",
          "country": "",
          "zip": "90045"
        }
      }
    ]
  }
}
```

**Response (error):**
```json
{ "status": "error", "message": "tracking information is not exist" }
```

---

## Tool 2 — `list_shipments`

New endpoint: `POST /v1/shipments/`

**Input:**
```json
{
  "tracking_provider": "usps",
  "status": "in_transit",
  "date_from": "2025-04-01",
  "date_to": "2025-04-15",
  "page": 1,
  "per_page": 20
}
```

| Parameter | Type | Required | Notes |
|---|---|---|---|
| tracking_provider | String | No | e.g. usps, fedex, ups |
| status | String | No | delivered, in_transit, exception, pending, pre_transit |
| date_from | String | No | YYYY-MM-DD |
| date_to | String | No | YYYY-MM-DD |
| page | Integer | No | Default: 1 |
| per_page | Integer | No | Default: 20, max: 100 |

**Response (success):**
```json
{
  "status": "success",
  "data": {
    "total": 142,
    "page": 1,
    "per_page": 20,
    "shipments": [
      {
        "order_id": "24",
        "tracking_number": "9400111899223397867254",
        "tracking_provider": "usps",
        "tracking_event_status": "delivered",
        "last_event_time": "2025-04-15 14:32:00"
      }
    ]
  }
}
```

---

## Tool 3 — `get_order_tracking`

New endpoint: `POST /v1/order/tracking/`
Replaces deprecated: `POST /v1/shipment/by-order-id`

**Input:**
```json
{
  "order_id": "141"
}
```

| Parameter | Type | Required | Notes |
|---|---|---|---|
| order_id | String | Yes | The order ID |

**Response (success):**
```json
{
  "status": "success",
  "data": [
    {
      "order_id": "141",
      "tracking_number": "463805514319",
      "tracking_provider": "fedex",
      "tracking_event_status": "delivered",
      "last_event_time": "2025-07-05 12:08:08",
      "events": [
        {
          "message": "Delivered",
          "status": "delivered",
          "datetime": "2025-07-05 12:08:08",
          "source": "Fedex",
          "tracking_location": {
            "city": "Swan Lake",
            "state": "MB",
            "country": "",
            "zip": "R0G2S0"
          }
        }
      ]
    }
  ]
}
```

**Response (error):**
```json
{ "status": "error", "message": "Order ID not found" }
```

---

## Tool 4 — `list_stores`

New logic, MCP only for now.

**Input:** none

**Response (success):**
```json
{
  "status": "success",
  "data": {
    "stores": [
      {
        "store_id": 7,
        "store_name": "My Store",
        "store_url": "https://mystore.com",
        "platform": "woocommerce",
        "connected": true
      }
    ]
  }
}
```

---

## Tool 5 — `get_account_plan`

New logic, MCP only for now.

**Input:** none

**Response (success):**
```json
{
  "status": "success",
  "data": {
    "plan_name": "Growth",
    "shipments_used": 3840,
    "shipments_limit": 5000,
    "renewal_date": "2025-05-01"
  }
}
```

---

## Auth

Every request to `/v1/mcp` must include:

```
trackship-api-key: <user api key>
app-name: <user app name>
```

Validate `trackship-api-key` against the existing API key field in the DB. Return `401` if invalid.

---

## Error Format

All tools return:

```json
{ "status": "error", "message": "Human readable message" }
```
