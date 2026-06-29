# Conductor — MCP Registry & Routing Policies (cross-cutting)

The two cross-cutting concerns Conductor runs *across* every task: the **MCP registry** (what
servers exist, their health and versions) and the **routing policies** (how to retry, fall
back, time out, and circuit-break). Both are **pluggable**: a route selects the applicable
policy; new servers/policies are added without changing the pipeline.

## The MCP registry (device manager)

The registry server (L1, `02`) is the stack's single catalog of MCP servers. Conductor
**reads** it at L2 (`03`) and reacts to it; it does not own server lifecycle.

```
RegistryEntry {
  id, capabilities:[ capability ids ]
  endpoint:    where to invoke
  health:      healthy | degraded | unhealthy   (live)
  version:     <server release>
  replicas:    [ alternate endpoints for the same capabilities ]
}
```

- **Discovery** — resolve a step's capability to one or more routes (`05`).
- **Health** — drive health-aware dispatch and the circuit-breaker (below).
- **Versioning** — the resolved `version` is **pinned per run** and recorded on every
  `Invocation`, so a run is reproducible and the audit shows exactly what was hit.
- **Replicas** — alternate routes for one capability; the basis of **fallback**.

> The registry spans the whole stack — the imaging-AI servers Perception fronts, the
> embedding/vector servers Recall fronts, the KBs/LLMs Reasoner & Pathways use — so Conductor
> sees **one** catalog and routes uniformly.

## Routing policies

Bound to each route at L2 (`03`), applied at L5 (`06`).

| Policy | Field | Meaning |
|---|---|---|
| **Timeout** | `deadlineMs` | per-call deadline; exceed → treated as failure |
| **Retry** | `maxAttempts`, `backoff` | retry transient (`timeout`/`unavailable`) failures |
| **Fallback** | `replicaOrder` | which replica routes to try, in order |
| **Circuit-breaker** | `failureThreshold`, `cooldown` | trip on repeated failures / `unhealthy`; skip the route until cooldown |
| **Concurrency** | `maxParallel` | cap on simultaneous in-flight steps |

```
RoutingPolicy {
  appliesTo:    capability / route / target class
  timeout:      deadlineMs
  retry:        { maxAttempts, backoff }
  fallback:     { replicaOrder }
  breaker:      { failureThreshold, cooldown }
}
```

### Circuit-breaking

A route is broken when the registry reports it `unhealthy` **or** it trips the breaker on
repeated failures. While broken, dispatch **skips straight to a replica** (no wasted attempts)
and the trip is recorded as an `Invocation` event. After `cooldown`, the route is probed again
and restored if healthy. This is what made the worked run resilient: the flaky segmentation
server was broken, the run fell back to a healthy replica, and `find-mri-001` was still
produced (`06`).

## Audit format

Every action produces an audit entry; together they are the `RunContext.auditTrail` (`07`).

```
AuditEntry {
  taskId, stepId, invocationId
  route:        { target, endpoint, version }
  attempt, outcome
  authorization:{ decision, reason, by: "Sentinel" }   // action-class steps
  asserted_by:  "conductor"      // the ROUTING decision; domain provenance is the component's
  timestamp
}
```

- Conductor's provenance covers **routing and ordering**, never the clinical fact.
- The audit is the auditable record of *what was run, in what order, with what version, and
  whether it was authorized* — the orchestration counterpart to each component's domain
  provenance.

## Governance split

| Concern | Owner |
|---|---|
| Server registration / health probing | **registry server** (L1) |
| Retry / fallback / breaker / timeout | **Conductor** (policy, here) |
| Whether an action is permitted (consent/PHI/access) | **Sentinel** (C8) |
| Model approval / eval / drift / safety | **Sentinel** (C8) |
| The clinical assertion + candidate→confirmed | the routed **component** + a human |

## Why pluggable matters

A new MCP server is a new registry entry + a policy binding — **no pipeline change**. A new
event type is a new templated DAG (`05`) bound at L6 (`07`). The deterministic execution core
(L5) works for any route in the standard shape, keeping Conductor extensible as the stack
grows. Versioning and audit currency are governed with **Sentinel** (C8), mirroring
`docs/components/pathways/08-guidelines-and-trials.md`.
