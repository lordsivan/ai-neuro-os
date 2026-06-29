# Conductor — MCP Capability Map (L1)

The capabilities Conductor orchestrates. All are **external MCP servers** — Conductor hosts
no registry database, runs no transport stack, and runs no model. Only **L2** calls these.
(Its other routing targets — Perception, Connectome, Pathways, Reasoner — are **sibling
components**, not L1; see `03`.)

## Capability catalog

| MCP server | Provides | Inputs | Outputs | Feeds |
|---|---|---|---|---|
| **MCP registry** | service discovery, health, versions | query (capability / id) | server entries: endpoint, capabilities, `health`, `version` | route catalog (L2), policies (`08`) |
| **Transport** | invoke any registered MCP server / endpoint | route + payload | invocation result / error | execution (L5) |
| **LLM / planning** | decompose an open-ended request into steps | task + capability catalog | a proposed step DAG (steps + deps) | task planning (L4) |

> The registry and transport are Conductor's **distinctive** capabilities (no other
> component owns the device-manager role). The **LLM/planning** server is the *same vendor
> class* Reasoner uses for hypothesis generation — reused here for **task decomposition**,
> not clinical reasoning.

## Per-capability notes

### MCP registry — the device manager
The single source of truth for "what servers exist, are they up, and which version". Each
entry exposes the server's **capabilities** (what it can do), **endpoint** (where), **health**
(live status), and **version**. Conductor queries it to discover routes and to make
**health-aware** dispatch decisions (skip / circuit-break unhealthy servers, `06`, `08`). The
registry covers the *whole stack's* MCP servers — the imaging-AI servers Perception fronts,
the embedding/vector servers Recall fronts, the KBs Reasoner/Pathways use — so Conductor sees
one catalog.

> Conductor **reads** the registry and reacts to it; it does not *own* server lifecycle.
> Registration/health-probing is the registry server's job — Conductor consumes the signal
> and applies **routing policy** on top (`08`).

### Transport — the uniform call
One mechanism to invoke any registered MCP endpoint with a payload and get a result or a
typed error (timeout, unavailable, bad-request). Transport makes a route to *any* MCP server
look the same to L5, so retry/fallback/timeout logic (`06`) is written once and works for all.

### LLM / planning — task decomposition
For requests whose step shape isn't known ahead of time, the planning LLM proposes a **step
DAG** given the task and the available **capability catalog** (`03`). It is used only to
*structure* work — propose steps and dependencies — **not** to perform any clinical step. For
known stack events, Conductor uses **templated DAGs** instead and never calls the LLM (`05`).

> The LLM proposes *structure*, never *facts*. Every proposed step must resolve to a real
> route in the catalog; steps that don't are rejected at L4. Clinical judgment stays inside
> the components the steps route to.

## Contract stability

Higher layers code against `Route` / `CapabilityDescriptor` / `Invocation` (`04`), never these
raw payloads. Swap the registry implementation → only its L2 adapter changes; swap the
planning LLM → only L4's decomposition adapter + prompt change; the execution core (L5) is
unaffected because it speaks `Route`/`Invocation`.

> LLM/model governance — approval, eval, drift, safety — is **Sentinel** (C8), not Conductor;
> Conductor *schedules and routes* model calls, it does not certify them. See
> `docs/components/reasoner/02-mcp-capability-map.md` for the same split on the diagnostic LLM.
