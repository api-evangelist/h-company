---
name: Define reusable agents, skills, and environments
description: Build a reusable H Company agent — its environment, skills, and configuration — so sessions launch consistently.
api: openapi/h-company-computer-use-agents-openapi-original.json
operations:
  - create_environment_api_v2_environments_post
  - create_skill_api_v2_skills_post
  - create_agent_api_v2_agents_post
  - list_agents_api_v2_agents_get
  - patch_agent_api_v2_agents__agent_name__patch
---

# Define reusable agents, skills, and environments

An **agent** is a reusable configuration (environment + model + instructions + skills) that sessions run. Build the pieces bottom-up. Authenticate with `Authorization: Bearer hk-...`.

## Steps

1. **Create an environment** — `create_environment_api_v2_environments_post` (POST /api/v2/environments). Define the surface the agent acts on, e.g. a Browser (`mode` visual/text, `headless`, optional `vault_id`, `browser_profile_id`, managed proxy).
2. **Create reusable skills** — `create_skill_api_v2_skills_post` (POST /api/v2/skills). A skill is an instruction fragment an agent can draw on.
3. **Create the agent** — `create_agent_api_v2_agents_post` (POST /api/v2/agents) referencing your `environments`, `skills`, `model`, `instructions`, `answer_format`, and optional `subagents`. Send an `Idempotency-Key`; a duplicate agent name returns 409. Take the `model` id from the Models docs — the spec types it as a free string and won't validate it, so a wrong id is accepted at create and fails at run.
4. **List to confirm** — `list_agents_api_v2_agents_get` returns your org's agents plus the public `h/` catalog. Reserved `h/` agents can't be modified (403).
5. **Iterate** — `patch_agent_api_v2_agents__agent_name__patch` changes individual fields without resending the full object.

## Rules

- Instructions define *who* the agent is and its guardrails; the *task* always goes in a session's `messages`, never in `instructions`.
- Idempotency-Key on every create; see conventions/h-company-conventions.yml.
