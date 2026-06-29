# Conductor — Overview (C6)

**Conductor** is the **orchestration service** of `ai-neuro-os` — the "kernel". It plans
multi-step tasks, **routes** calls across components and MCP servers, runs the **agent
loop**, and manages the **MCP registry** (capability discovery, health, versioning). It is
how a single external trigger — "a study landed" — becomes a sequenced, fault-tolerant,
audited run across Perception, Connectome, Pathways and Reasoner.

Like every component it follows the stack rules: **no low-level code** (the registry,
transport, and planning LLM run in **MCP servers**), **layered** (each layer depends only on
the one below), **domain + aggregate code only**.

> In OS terms Conductor is the kernel — but in our **knowledge-first** stack it is a
> *supporting service*, not the center. **Connectome (C1) holds the state; Conductor moves
> work around it.**

## Why this exists

The components each do one thing well, but **someone has to sequence them**. When a
follow-up MRI is ingested, Perception must run, Connectome must persist, Pathways must
re-assess response — in that order, with dependencies, retries when a server flakes, and a
clean skip when one is unhealthy. Without a conductor that logic leaks into every UI and
event handler, untraceably. Conductor centralizes it: **one place that plans, routes,
recovers, and records** — so orchestration is itself auditable.

## What it does (and does not)

| Conductor **is** | Conductor **is not** |
|---|---|
| The task planner + router ("kernel") | The MCP registry / LLM (those are MCP servers) |
| The agent loop that drives multi-step runs | A graph store (that's Connectome, C1) |
| Health-aware dispatch with retry/fallback | A domain reasoner (Reasoner/Pathways own that) |
| The enforcer of step **ordering** and routing | The **confirmer** of any clinical fact (a human is) |
| The recorder of an action audit trail | The governance authority (that's Sentinel, C8) |

> **Conductor routes; it does not decide.** It enforces *that* Perception runs before
> Connectome persists; it never overrides *what* Perception found or *whether* a diagnosis
> is confirmed. Clinical assertions stay **candidate** until a human confirms (`07`).

## How it works

Conductor runs **one scaffold** (L1–L6, `01`) for every task:

- **Plan** — decompose a request/event into a **step DAG**, then resolve each step to a
  **route**: a sibling-component endpoint (Perception, Connectome, Pathways, Reasoner) or an
  MCP server. LLM-assisted decomposition for open-ended tasks; templated DAGs for known
  events (`05`).
- **Route & execute** — dispatch steps respecting dependencies, with **retries, fallbacks,
  timeouts, concurrency, and circuit-breaking** on unhealthy servers; collect results into a
  `RunContext` (`06`).
- **Serve & wire** — expose `runTask`, `scheduleOnEvent`, `status`; wire stack events such as
  *study-ingested → Perception → Connectome persist → Pathways.assessResponse* (`07`).

## Boundaries (route / authorize / audit)

> **Conductor plans and routes; Sentinel authorizes; Connectome persists; a human confirms.**

- Conductor **does not own domain knowledge** — it calls components that do.
- Before any **action** (a write, an external effect) Conductor asks **Sentinel (C8)** to
  **authorize**, and records an **audit** entry for the action and its provenance (`07`,
  `08`). Sentinel governs *whether*; Conductor governs *order and routing*.
- Conductor never confirms a clinical fact: it routes a **candidate** diagnosis / plan /
  assessment to Connectome flagged `candidate`; confirmation is a human step (Console, C7).

```
event / request ──▶ CONDUCTOR (plan DAG → resolve routes)
                         │  per step: Sentinel authorize → dispatch (retry/fallback/health) → audit
        Perception ─▶ Connectome persist ─▶ Pathways.assessResponse ─▶ (Reasoner / re-plan)
                         │
                   RunContext + audit trail (results, routes, retries, decisions)
```

## Scope (this phase)

- **Inputs:** a request or a stack **event** (e.g. `study-ingested`), the **MCP registry**
  (servers, health, versions), and **component/endpoint descriptors**.
- **Output:** an executed **Plan** (step DAG) with collected results, full **routing +
  retry/fallback** record, Sentinel **authorize** checks, and an **audit** trail.
- **Deliverable:** design specification + a worked sample. **No code.**

## Design principles

1. **No low-level code** — registry, transport, planning LLM are MCP servers; Conductor
   orchestrates.
2. **Strict layering** — L1→L6, each depends only on the one below (`01`).
3. **Plan as a DAG** — explicit steps + dependencies, never an opaque script (`05`).
4. **Health-aware, fault-tolerant** — retries, fallbacks, timeouts, circuit-breaking (`06`).
5. **Authorize before acting** — Sentinel gates every action; every action is audited (`07`,
   `08`).
6. **Route, never confirm** — clinical assertions stay candidate; a human confirms.

## Requirements traceability

| Requirement | Where |
|---|---|
| MCP registry (discovery / health / versions) | `02`, `08-registry-and-policies.md` |
| Routing adapters → uniform Route catalog | `03-routing-adapters.md` |
| Domain model (Task/Plan/Step/Route/Invocation) | `04-domain-model.md` |
| Task planning (decompose → DAG → resolve routes) | `05-task-planning.md` |
| Routing & execution (retry/fallback/health/concurrency) | `06-routing-and-execution.md` |
| Orchestration API + event wiring | `07-orchestration-api.md` |
| Sentinel authorize + audit on actions | `07`, `08` |
| Routing policies, circuit-breaking, versioning | `08` |

## Reading order

`01` layers → `02` capabilities → `03` routing adapters → `04` domain model →
`05` task planning → `06` routing & execution → `07` orchestration API →
`08` registry & policies → `09` reference. Then the worked example in `samples/conductor/`.
