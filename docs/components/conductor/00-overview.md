# Conductor (C6) — Overview *(stub — to brainstorm)*

> **Status:** placeholder. Named and scoped; not yet specified as a full L1–L6 component.

## Role

**Conductor** is the **orchestration service**: it plans multi-step tasks, **routes**
calls across components and MCP servers, runs the agent loop, and manages the **MCP
registry** (capability discovery, health, versioning). It is the "kernel" in OS terms —
but in our **knowledge-first** stack it is a *supporting service*, not the center:
Connectome holds the state; Conductor moves work around it.

## Depends on

- **All components** — it routes to them; it does not own domain knowledge.
- MCP servers (L1): the **registry** of all servers (health, capabilities, versions).

## Responsibilities

| Concern | Notes |
|---|---|
| Task planning | decompose a request into component / MCP calls |
| Routing | dispatch to the right component/server; handle retries, fallbacks |
| Agent loop | drive multi-step agentic workflows |
| MCP registry | discover, health-check, version MCP servers ("device manager") |

## Open questions (for the brainstorm)

- C0 vs. C6: is Conductor numbered last (supporting) or first (spine)? We chose
  **knowledge-first**, so it is a service — revisit only if orchestration becomes the
  product's center of gravity.
- Where workflow definitions live and how they stay auditable (coordinate with **Sentinel**).
- Sync request/response vs. long-running background jobs.

When promoted, this folder grows the same `01`–`09` layer docs as Connectome.
