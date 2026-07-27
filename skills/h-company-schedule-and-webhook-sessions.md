---
name: Schedule sessions and receive webhook notifications
description: Run H Company agent sessions on a cron cadence and get signed webhook notifications as they change status.
api: openapi/h-company-computer-use-agents-openapi-original.json
operations:
  - create_schedule_api_v2_schedules_post
  - trigger_schedule_api_v2_schedules__schedule_id__trigger_post
  - list_schedule_runs_api_v2_schedules__schedule_id__runs_get
  - create_webhook_api_v2_webhooks_post
  - list_webhook_events_api_v2_webhooks_events_get
  - ping_webhook_api_v2_webhooks__webhook_id__ping_post
  - rotate_webhook_secret_api_v2_webhooks__webhook_id__rotate_post
---

# Schedule sessions and receive webhook notifications

Automate recurring agent work and react to session lifecycle events without polling. Authenticate with `Authorization: Bearer hk-...`.

## Schedule recurring sessions

1. **Create a schedule** — `create_schedule_api_v2_schedules_post` (POST /api/v2/schedules) with a five-field cron `timing` (evaluated in an IANA timezone) and a `session_request` template. Each fire creates a session.
2. **Trigger on demand** — `trigger_schedule_api_v2_schedules__schedule_id__trigger_post` fires once, immediately.
3. **Audit** — `list_schedule_runs_api_v2_schedules__schedule_id__runs_get` shows the fire history and whether each fire created a session or was skipped. Pause/resume without deleting.

## Receive webhooks

1. **Discover event types** — `list_webhook_events_api_v2_webhooks_events_get` (GET /api/v2/webhooks/events) lists subscribable types: `session.completed`, `session.failed`, `session.timed_out`, `session.idle`, `session.awaiting_tool_results`, `session.status_updated`.
2. **Register an endpoint** — `create_webhook_api_v2_webhooks_post` with your `url` and `enabled_events`. Deliveries are signed; verify the signature with the webhook secret.
3. **Test** — `ping_webhook_api_v2_webhooks__webhook_id__ping_post` sends a signed test event.
4. **Rotate the secret** — `rotate_webhook_secret_api_v2_webhooks__webhook_id__rotate_post` replaces the signing secret without missing a verification.

## Rules

- Failed deliveries retry with backoff; endpoints that keep failing are auto-disabled — monitor `consecutive_failures` / `last_delivery_status`.
- Subscribe to specific events rather than the `session.status_updated` firehose. See asyncapi/h-company-webhooks.yml.
