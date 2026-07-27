---
name: Run a browser agent to completion
description: Launch an H Company computer-use agent session on a task, watch it run, steer it, and read how it ended.
api: openapi/h-company-computer-use-agents-openapi-original.json
operations:
  - create_session_api_v2_sessions_post
  - get_session_status_api_v2_sessions__id__status_get
  - get_session_changes_api_v2_sessions__id__changes_get
  - send_session_messages_api_v2_sessions__id__messages_post
  - force_session_answer_api_v2_sessions__id__force_answer_post
  - cancel_session_api_v2_sessions__id__delete
---

# Run a browser agent to completion

Authenticate every request with `Authorization: Bearer hk-...` (keys are minted on Portal-H). Default host is the EU region `https://agp.eu.hcompany.ai`; use `https://agp.hcompany.ai` for US. Requests stay in-region.

## Steps

1. **Create the session** — `create_session_api_v2_sessions_post` (POST /api/v2/sessions). Put the *task* in `messages` (a `user_message`), not in `instructions`. Prefer a pre-built `h/` agent (e.g. `h/web-surfer-flash`); never guess an agent id — list the catalog first with `list_agents_api_v2_agents_get`. Send an `Idempotency-Key` header so a retried create doesn't launch a duplicate (a duplicate in-flight returns 409). Over-quota creates are accepted as `queued` unless you set `queue: false`.
2. **Poll status** — `get_session_status_api_v2_sessions__id__status_get` for a lightweight progress check, or long-poll `get_session_changes_api_v2_sessions__id__changes_get` for real-time updates.
3. **Steer if needed** — `send_session_messages_api_v2_sessions__id__messages_post` to chat with the running agent, or `force_session_answer_api_v2_sessions__id__force_answer_post` to make it emit a final answer next step.
4. **Stop early if needed** — `cancel_session_api_v2_sessions__id__delete` frees the concurrency slot immediately.
5. **Read the outcome** — a terminal session reports `error_code` (`environment_error`, `no_answer`, `answer_validation`, `timeout`, `internal`) and each answer an `outcome` (`success`, `partial`, `infeasible`, `blocked`). Build retry logic against these codes, not message strings.

## Rules

- Retry only `408`, `429`, and `5xx` (with backoff, honoring `Retry-After`). Never retry `400/401/402/403/404/422`.
- `402 Payment Required` means the org's monthly token budget is exhausted — the `detail` carries `limit`, `used`, `window_end`.
- Error envelope is `{ message, detail[] }` (custom, not RFC 9457). See errors/h-company-problem-types.yml and conventions/h-company-conventions.yml.
