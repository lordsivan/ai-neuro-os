# Conductor — Reference (diagrams + attributes)

The visual model and attribute-level reference. Pairs with `04` (entities) and `05`–`07`
(planning / execution / serving).

## Pipeline (end to end)

```mermaid
flowchart TD
    subgraph inputs[L2 inputs]
      REG[(MCP registry)]:::ext
      TRANS[Transport]:::s
      LLMP[Planning LLM]:::s
      SIB[[Component descriptors]]:::ext
    end
    inputs --> L2[L2 · Route catalog<br/>capabilities + routes + health]
    L2 --> L4[L4 · Task planning<br/>decompose → DAG → resolve routes]
    L4 --> L5[L5 · Routing & execution<br/>dispatch · retry/fallback · health-aware · collect]
    L5 --> L6[L6 · Orchestration API + event wiring]
    L6 -->|authorize action| SENT[[Sentinel C8]]
    L6 -->|route candidate write| CA[[Connectome adapter]]
    classDef s fill:#eee,stroke:#999;
    classDef ext fill:#ffe9b3,stroke:#b8860b;
```

## Data shapes

```mermaid
classDiagram
    class Task { id kind trigger subject status }
    class Plan { id strategy steps edges status }
    class Step {
      id
      capability
      route        // resolved at L4
      dependsOn    // DAG edges
      conditional  // predicate on prior result
      state
      attempts
    }
    class Route { id target endpoint version healthState policyRef }
    class CapabilityDescriptor { id inputs outputs sideEffectClass replicas }
    class Invocation { id attempt outcome authorized auditRef }
    class RunContext { id results decisions auditTrail status }

    Task --> Plan : decomposes_into
    Plan --> Step : contains
    Step --> CapabilityDescriptor : needs
    CapabilityDescriptor --> Route : resolved_to
    Step --> Invocation : executed_as
    Plan --> RunContext : run_as
```

## Attribute reference

### Step
| Attribute | Type | Notes |
|---|---|---|
| `capability` | id | what the step needs (resolves to a route) |
| `route` | Route | chosen target (component **or** MCP server) |
| `dependsOn` | id[] | predecessor steps (DAG edges) |
| `conditional` | predicate | gate on a prior result; false → `skipped` |
| `state` | enum | `pending`→`authorizing`→`dispatched`→`succeeded`/`failed`/`skipped` |
| `attempts` | int | retry count consumed |

### Route
| Attribute | Type | Notes |
|---|---|---|
| `target` | enum/ref | component endpoint or MCP server |
| `endpoint` | string | where to invoke |
| `version` | string | **pinned per run**, recorded on every Invocation |
| `healthState` | enum | `healthy` / `degraded` / `unhealthy` |
| `policyRef` | ref | retry/fallback/timeout/breaker policy (`08`) |

### Invocation
| Attribute | Type | Notes |
|---|---|---|
| `attempt` | int | which try (retries/fallbacks each record one) |
| `outcome` | enum | `ok` / `timeout` / `error` / `unavailable` |
| `authorized` | ref | Sentinel decision (action-class steps) |
| `auditRef` | ref | audit-trail entry |

### Shared provenance
`id`, `source`, `asserted_by` (`conductor` — for the **routing decision**, never the clinical
fact), `timestamp`.

## Enumerations

| Enum | Values |
|---|---|
| `Task.kind` | `event`, `request` |
| `Task.status` | `planned`, `running`, `succeeded`, `failed`, `partial` |
| `Step.state` | `pending`, `authorizing`, `dispatched`, `succeeded`, `failed`, `skipped` |
| `CapabilityDescriptor.sideEffectClass` | `read`, `compute`, `action` |
| `Route.healthState` | `healthy`, `degraded`, `unhealthy` |
| `Invocation.outcome` | `ok`, `timeout`, `error`, `unavailable` |
| `Plan.strategy` | `template`, `llm-decomposed` |

## Worked instance

A concrete run — `study-ingested` for `pat-001` / `les-001` orchestrated end to end
(Perception → Connectome persist → Pathways.assessResponse → conditional re-plan), with a
retry/fallback on a flaky segmentation server, health-aware dispatch, and a Sentinel
authorize+audit hook on each action — is in `samples/conductor/`.
