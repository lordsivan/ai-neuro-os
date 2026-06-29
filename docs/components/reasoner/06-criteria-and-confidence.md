# Reasoner — Criteria & Confidence (L5)

L5 adjudicates the differential (L4): it applies formal **criteria deterministically**,
**calibrates** each candidate's confidence, and chooses the **next discriminating test**.
This is the auditable half of the hybrid method — no LLM here, only explicit logic over
explicit evidence.

## Criteria application — deterministic

For each `DifferentialItem` with an applicable `CriteriaSet`, L5 evaluates **every
`Criterion`** against the `EvidenceBundle`:

| Result | Meaning |
|---|---|
| `met` | the evidence satisfies the criterion (refs recorded) |
| `not-met` | the evidence contradicts / fails it |
| `indeterminate` | required evidence is **missing** (drives the next-test choice) |

The `CriteriaSet`'s own logic (e.g. McDonald's "dissemination in space **and** time")
combines the per-criterion results into a set-level verdict. Because each result points at
the exact evidence (or names what's missing), the conclusion is fully **auditable** — a
reviewer can see why MS was or wasn't met, criterion by criterion.

> The LLM proposed the candidate; the rules decide whether the formal criteria are met.
> The two never blur.

## Confidence calibration

Each `DifferentialItem` gets a **calibrated** `confidence` (0–1), combining:

- **Criteria verdict** — met sets > indeterminate > not-met.
- **Evidence weight** — quantity/quality of `supporting` vs. `refuting` refs, weighted by
  each finding's own provenance/confidence (a histology-confirmed finding counts more than
  a candidate embedding match).
- **Precedent** — concordance with similar cases (a weak prior, not a decider).
- **Calibration** — mapped so a `confidence` of 0.8 means the same across diseases; the
  differential's confidences are **comparable and ranked**, not raw scores.

Output is a **ranked differential with calibrated uncertainty** — explicitly *not* a single
overconfident answer.

## Next discriminating test

L5 selects the test that would most reduce uncertainty — typically by resolving the
`indeterminate` criteria or separating the top candidates:

```
for each candidate next-test T:
    expectedInfoGain(T) ≈ how much T's likely results would
                          separate the top DifferentialItems
                          / resolve indeterminate criteria
choose argmax expectedInfoGain
```

The result is a `DiscriminatingTest` naming the test (e.g. **biopsy/histology**, contrast
MRI, lumbar puncture), **which candidates it separates**, and a grounded rationale. This is
the "what would confirm/refute" guidance — diagnostic, not treatment (treatment is
Pathways, C5).

## Conflict & safety handling

| Situation | L5 behavior |
|---|---|
| Two candidates close in confidence | keep both top-ranked; next-test targets their separation |
| Criteria met but weak imaging evidence | confidence reflects the tension; flagged for review |
| High-trust finding (histology) contradicts a candidate | candidate down-ranked decisively |
| All top candidates `indeterminate` | low confidence; next-test is the headline output |

L5 never forces a single answer when the evidence doesn't support one — it ranks and points
to the test that would.

## Output to L6

A scored, criteria-adjudicated differential + a chosen `DiscriminatingTest`, ready to be
packaged into a `DiagnosisProposal` (`07`): the top candidate becomes the proposed
**candidate** `Diagnosis`, the rest remain the visible, evidence-cited differential.
