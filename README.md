# X-Plane Local Web API — OpenAPI + AsyncAPI Specification

An unofficial [OpenAPI 3.0](https://www.openapis.org/) + [AsyncAPI 2.6](https://www.asyncapi.com/) specification for the [X-Plane local Web API](https://developer.x-plane.com/article/x-plane-web-api/).

Starting with X-Plane 12.1.1, the simulator includes a built-in web server with REST and WebSocket interfaces for communicating with a running instance.

## Spec Files

| File | Format | Covers |
|------|--------|--------|
| `openapi.json` | OpenAPI 3.0.3 | REST API endpoints |
| `asyncapi.json` | AsyncAPI 2.6.0 | WebSocket message types |

## REST Endpoints (openapi.json)

| Method | Endpoint | Description | Since |
|--------|----------|-------------|-------|
| `GET` | `/api/capabilities` | API versions and X-Plane version | v2 (12.1.4) |
| `GET` | `/api/v3/datarefs` | List datarefs (filter, paginate, field select) | v1 (12.1.1) |
| `GET` | `/api/v3/datarefs/count` | Get dataref count | v1 (12.1.1) |
| `GET` | `/api/v3/datarefs/{id}/value` | Read a dataref value | v1 (12.1.1) |
| `PATCH` | `/api/v3/datarefs/{id}/value` | Write a dataref value | v1 (12.1.1) |
| `GET` | `/api/v3/commands` | List commands | v2 (12.1.4) |
| `GET` | `/api/v3/commands/count` | Get command count | v2 (12.1.4) |
| `POST` | `/api/v3/command/{id}/activate` | Activate a command | v2 (12.1.4) |
| `POST` | `/api/v3/flight` | Start a new flight | v3 (12.4.0) |
| `PATCH` | `/api/v3/flight` | Update current flight | v3 (12.4.0) |

## WebSocket Messages (asyncapi.json)

**Client to server:**

| Message Type | Description | Since |
|-------------|-------------|-------|
| `dataref_subscribe_values` | Subscribe to dataref updates (10Hz) | v1 |
| `dataref_unsubscribe_values` | Unsubscribe from dataref updates | v1 |
| `dataref_set_values` | Set dataref values | v1 |
| `command_subscribe_is_active` | Subscribe to command status | v2 |
| `command_unsubscribe_is_active` | Unsubscribe from command status | v2 |
| `command_set_is_active` | Activate/deactivate commands | v2 |

**Server to client:**

| Message Type | Description |
|-------------|-------------|
| `result` | Success/failure response to any request |
| `dataref_update_values` | Real-time dataref value changes |
| `command_update_is_active` | Command activation status changes |

## Quick Start

### Use with curl

```bash
# Check API capabilities
curl http://localhost:8086/api/capabilities

# List first 5 datarefs
curl 'http://localhost:8086/api/v3/datarefs?limit=5'

# Get dataref count
curl http://localhost:8086/api/v3/datarefs/count

# Find the simulator zulu time dataref (writable)
curl 'http://localhost:8086/api/v3/datarefs?filter%5Bname%5D=sim/time/zulu_time_sec'

# Read the current zulu time in seconds since midnight UTC (use the ID from above)
curl http://localhost:8086/api/v3/datarefs/40003472032/value

# Set zulu time to noon (43200 seconds) — the sky and lighting change instantly!
curl -X PATCH http://localhost:8086/api/v3/datarefs/40003472032/value \
  -H 'Content-Type: application/json' \
  -d '{"data": 43200}'

# List commands
curl 'http://localhost:8086/api/v3/commands?limit=5'

# Toggle the landing lights on/off (press and release)
curl -X POST http://localhost:8086/api/v3/command/818/activate \
  -H 'Content-Type: application/json' \
  -d '{"duration": 0}'
```

> **Note:** Dataref IDs are assigned at runtime and may differ between sessions. Always look up the ID by name first.

### WebSocket (using websocat)

```bash
# Connect
websocat ws://localhost:8086/api/v3

# Subscribe to zulu time updates at 10Hz (type this after connecting)
{"req_id": 1, "type": "dataref_subscribe_values", "params": {"datarefs": [{"id": 40003472032}]}}
```

### Import into Postman

1. Open Postman
2. Click **Import** > select `openapi.json`
3. A collection with all REST endpoints will be created

### View in Swagger Editor

1. Go to [editor.swagger.io](https://editor.swagger.io)
2. Click **File** > **Import File** > select `openapi.json`

### View AsyncAPI in Studio

1. Go to [studio.asyncapi.com](https://studio.asyncapi.com)
2. Import `asyncapi.json`

## Configuration

The web server runs on `localhost:8086` by default. To change the port:

```bash
./X-Plane --web_server_port=8088
```

To disable the API entirely, set "Disable Incoming Traffic" in Settings > Network.

## Disclaimer

This is an **unofficial** community-maintained specification. It is not affiliated with or endorsed by Laminar Research. The API is provided by X-Plane and documented at [developer.x-plane.com](https://developer.x-plane.com/article/x-plane-web-api/).

## Contributing

Found an issue or want to improve the spec? PRs are welcome!

## License

This specification is provided as-is for community use. X-Plane is owned and operated by Laminar Research.
