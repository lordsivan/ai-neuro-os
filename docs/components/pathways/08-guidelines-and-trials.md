# Pathways — Guidelines, Trials & Response Schemes (cross-cutting)

The three knowledge sources Pathways plans and monitors against, served by L1 KBs and
applied at L4/L5. All are **pluggable**: a case selects the applicable items; new ones are
added to the KB without changing the pipeline.

## Treatment guidelines

Machine-readable guideline entries keyed by diagnosis + factors.

```
GuidelineEntry {
  appliesTo:    condition + grade/stage + factors (age, biomarkers, performance status)
  recommends:   [ regimen elements with ordering ]
  conditions:   checkable prerequisites
  evidenceGrade:e.g. high | moderate | low
  version:      <guideline release>
}
```

- Selected at L2 by the coded diagnosis + patient factors.
- Drive `TreatmentOption`s at L4; their `evidenceGrade` weights selection at L5.
- Examples are domain-agnostic; the model spans general neurology (neuro-onc therapy,
  MS disease-modifying therapy, stroke secondary prevention, etc.).

## Clinical trials

Trial records + an eligibility evaluator.

```
Trial {
  id, title, phase
  eligibility: [ criterion: { statement, logic, evidenceNeeded } ]
}
TrialMatch { trialId, eligibility: eligible|ineligible|unknown, criteriaResults[] }
```

- Eligibility is evaluated per-criterion against the patient profile (diagnosis, prior
  treatment, biomarkers, performance status) — the same checkable-criterion shape used for
  diagnosis/response criteria.
- A `TrialMatch` becomes a trial `TreatmentOption` at L4, carrying *why* the patient
  qualifies (or which criterion fails).

## Response / progression schemes

The criteria Pathways applies for **monitoring** (shared with Reasoner's diagnosis use).

| Scheme | Domain | Verdicts |
|---|---|---|
| **RANO** | neuro-oncology | response / stable / progression / pseudo-progression |
| **McDonald** | MS activity | new/active lesions → relapse/activity |
| **mRS** | functional outcome | 0–6 disability score (outcome) |

```
ResponseScheme {
  id:        "RANO"
  criteria:  [ { statement, logic, threshold, evidenceNeeded } ]
  combine:   how per-criterion results → verdict
  confounders:[ e.g. pseudo-progression window after chemoradiation ]
}
```

- Applied **deterministically** at L5 (`06`) over the baseline↔current comparison.
- `confounders` encode safety windows (e.g. don't call progression within the
  pseudo-progression window) — surfaced as flags, not silent suppression.

## Selection (what applies)

L2 (`03`) selects per case:

| Case signal | Guidelines | Trials | Response scheme |
|---|---|---|---|
| high-grade glioma | neuro-onc therapy | glioma trials | RANO |
| MS | disease-modifying therapy | MS trials | McDonald |
| stroke | secondary prevention | — | mRS (outcome) |

## Versioning & governance

- Guidelines, trial data, and response schemes are **versioned**; the version used is
  recorded on every plan/assessment for reproducibility.
- Currency and approval are governed by **Sentinel** (C8) — clinical knowledge changes are
  controlled and audited; prior versions stay available to re-derive past plans/assessments.

## Why pluggable matters

A new guideline, trial, or scheme is a new KB entry + selection rule — **no pipeline
change**. The deterministic evaluators (eligibility at L4, response at L5) work for any
criterion in the standard checkable shape, keeping Pathways extensible across neurology.
