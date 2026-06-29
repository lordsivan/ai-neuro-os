# Console — MCP Capability Map (L1)

The capabilities Console orchestrates. All are **external MCP servers** — Console runs no
language model and no renderer of its own. Only **L2** calls these. (Its other inputs —
Connectome, Reasoner, Pathways, Recall — are **sibling components**, not L1; see `03`.)

## Capability catalog

| MCP server | Provides | Inputs | Outputs | Feeds |
|---|---|---|---|---|
| **LLM / NLP** | intent parsing **and** NL generation | clinician utterance (parse); composed, cited view model (generate) | structured `Query` intent; grounded NL explanation / report prose | intent adapter (L2), explanation & reports (L5) |
| **Rendering / report** | turn a view model into a rendered view or document | `ViewModel` + template | rendered panels / a formatted `ReportDraft` (e.g. PDF/HTML/structured doc) | view serving (L6), reports (L5/`08`) |
| **Notification** *(optional)* | deliver alerts / hand-offs | recipient + event (e.g. new progression, pending confirmation) | dispatched notification | event surfacing (L6) |
| **Terminology / ontology** *(shared)* | code & relate clinical concepts | term / code | SNOMED CT / ICD-11 codes | grounding NL ↔ graph entities |

> The **LLM/NLP** server is Console's defining capability: it is where *all* natural-language
> understanding and generation lives. Console itself maps the parsed intent to requests and
> the gathered facts to a view model — it never "reasons" in free text outside the grounded,
> cited material it was handed.
> **Terminology** is the same shared server the rest of the stack uses
> (`docs/components/pathways/02-mcp-capability-map.md`,
> `docs/components/connectome/02-mcp-capability-map.md`).

## Per-capability notes

### LLM / NLP — understand in, word out
Two directions, one server. **In:** parse the clinician's free-text question into a
structured `Query` (intent + subject + scope + trust profile hint) the intent adapter (L2)
can act on. **Out:** given a *composed, cited* view model, generate prose for explanations
and reports. Generation is **constrained to the supplied evidence** — the grounding comes
from L4/L5, not from the model's own knowledge (`06`). The model proposes wording; it never
asserts a clinical fact that isn't already in the view model.

### Rendering / report — present
Turns a `ViewModel` (or a `ReportDraft` skeleton) into something a human consumes: rendered
dashboard panels, a timeline, or a formatted clinical document. The renderer applies the
**trust display patterns** (`08`) — confirmed vs candidate styling/badges — from flags the
view model already carries; it makes no trust decisions itself.

### Notification *(optional)* — surface events
Delivers asynchronous hand-offs: a new `ProgressionAssessment` landed for a watched lesion,
or a candidate is awaiting confirmation. Event wiring/scheduling belongs to **Conductor**
(C6); Console just dispatches through this capability when asked.

### Terminology / ontology — grounding
Normalizes the terms in a question and in the answer to shared codes, so an utterance like
"the left frontal mass" binds to `les-001` and "glioma" to the coded `dx-001`. Keeps
Console's NL aligned with Connectome's domain model
(`docs/components/connectome/04-domain-model.md`).

## Contract stability

Higher layers code against `Query` / `ViewModel` / `Explanation` / `ReportDraft` (`04`),
never these payloads. Swap the LLM provider or the renderer → only its L2 adapter changes;
view composition (L4), trust application and explanation (L5) are unaffected. (Model & safety
governance and grounding/eval of the LLM are **Sentinel** (C8); multi-step routing and event
scheduling are **Conductor** (C6). Console just calls.)
