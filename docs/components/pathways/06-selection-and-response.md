# Pathways — Selection & Response (L5)

L5 adjudicates both tracks. For **planning** it selects and ranks the plan, finalizes
eligibility, builds the rationale, and calibrates confidence. For **monitoring** it applies
**response criteria deterministically** to produce an auditable verdict. This is the
decision layer; L6 packages and hands off.

## Planning track — selection

From L4's grounded options + alternatives, L5 produces the recommended `TreatmentPlan`:

- **Rank alternatives** where options conflict — by evidence grade, precedent outcomes, and
  patient factors (performance status, comorbidities, biomarkers).
- **Finalize trial eligibility** — confirm per-criterion eligibility for any trial option;
  surface near-misses with the unmet criterion named.
- **Build rationale** — for each chosen element, why it's in (guideline + grade, trial
  eligibility, precedent), and note the trade-offs/alternatives not taken.
- **Calibrate confidence** — a calibrated score on the plan reflecting evidence strength and
  consensus; lower when guidelines are weak or the case is atypical.

Output: a sequenced, justified candidate plan with alternatives visible — **not** a single
opaque recommendation. The MDT (L6 → human) makes the call.

## Monitoring track — response assessment (deterministic)

For the comparison set (L4), L5 applies the response **criteria** (RANO/McDonald) item by
item — exactly the deterministic-checklist discipline Reasoner uses, here over time:

| Result | Meaning |
|---|---|
| `met` | the measured change satisfies the criterion (refs recorded) |
| `not-met` | it does not |
| `indeterminate` | required measurement missing/unreliable |

The scheme's logic combines per-criterion results into a **verdict**:

| Verdict | Typical basis (e.g. RANO) |
|---|---|
| `response` | ≥ threshold decrease in measurable disease, no new lesions |
| `stable` | neither response nor progression thresholds met |
| `progression` | ≥ threshold increase, or new lesion |
| `recurrence` | progression after a disease-free interval |
| `pseudo-progression` | apparent progression within the post-treatment confounder window → flagged, not called progression |

Each verdict records the `deltaMeasurements`, the criteria results, and the
`comparedStudies` — fully auditable, like Reasoner's criteria output.

## Confounder & safety handling

| Situation | L5 behavior |
|---|---|
| Apparent progression soon after chemoradiation | flag **pseudo-progression** risk; verdict cautious; recommend short-interval re-scan |
| Missing/unreliable measurement | criterion `indeterminate`; verdict reflects uncertainty |
| New lesion elsewhere | progression regardless of target change |
| Plan: weak guideline + atypical case | plan confidence lowered; alternatives emphasized for MDT |

L5 never overstates: monitoring verdicts and plan confidence both carry calibrated
uncertainty.

## Output to L6

- **Planning:** the selected candidate `TreatmentPlan` (sequence + rationale + alternatives +
  calibrated confidence).
- **Monitoring:** a `ResponseAssessment` (verdict + criteria results + deltas + compared
  studies) ready to become a `ProgressionAssessment`.
