# Connectome — Navigation & Queries (L6)

L6 is the entry point: it turns the lesion-centric, trust-annotated graph (built by
L1–L5) into the **journeys** clinicians and agents actually take. Queries below are
expressed as **pseudo-Cypher / plain language** — design illustrations, **not runnable
code**. Each journey notes the lower layers / MCP capabilities it relies on.

> Convention: traversals can be filtered by trust, e.g. `WHERE e.confidence >= τ` and/or
> `e.asserted_by IN [...]`, and by `status IN [confirmed, candidate]`.

## A. Cross-modal: "find this finding in another modality" *(the core journey)*

From an MRI finding, reach its CT, US and histology counterparts.

```cypher
MATCH (f:Finding {id: $anchor})-[:manifestation_of]->(l:Lesion)
MATCH (l)<-[m:manifestation_of]-(g:Finding)-[:has_finding]-(:Series)
       -[:contains_series]-(s:ImagingStudy)-[:acquired_with]->(mod:Modality)
RETURN mod.name, g, m.confidence, m.status
ORDER BY mod.name
```
Relies on: L5 correlation (manifestation links), L4 graph. Returns the same entity grouped
by modality, each tagged confirmed/candidate + confidence.

## B. Patient history → imaging → diagnosis

```cypher
MATCH (p:Patient {id: $pid})-[:has_history]->(h:PatientHistory)
MATCH (p)-[:has_encounter]->(:Encounter)-[:has_study]->(:ImagingStudy)
       -[:contains_series]-(:Series)-[:has_finding]->(f:Finding)
MATCH (f)-[:manifestation_of]->(l:Lesion)-[:supports]->(d:Diagnosis)
RETURN h, collect(distinct f) AS findings, d
```
Relies on: clinical adapter (FHIR), finding adapter, correlation. Shows how the history
frames the findings that support the diagnosis.

## C. Diagnosis → treatment plan

```cypher
MATCH (d:Diagnosis {id: $dx})-[:treated_by]->(t:TreatmentPlan)
OPTIONAL MATCH (d)<-[:confirmed_by]-(:Finding)   // histologic confirmation
RETURN d, t
```
Relies on: clinical adapter (FHIR CarePlan/Procedure), pathology adapter.

## D. Lesion → progression timeline

```cypher
MATCH (l:Lesion {id: $lid})-[:assessed_by]->(a:ProgressionAssessment)
OPTIONAL MATCH path = (a)-[:progresses_to*]->(:ProgressionAssessment)
RETURN a, path ORDER BY a.timepoint
```
Plus the underlying findings over time:
```cypher
MATCH (l:Lesion {id: $lid})<-[:manifestation_of]-(f:Finding)
       -[:has_finding]-(:Series)-[:contains_series]-(s:ImagingStudy)
RETURN s.acquiredAt, f.measurements ORDER BY s.acquiredAt
```
Relies on: temporal model (L4), correlation (L5). Drives growth/response/recurrence views.

## E. Histology → which radiology finding it confirms

```cypher
MATCH (hf:Finding {modality: 'Histology', id: $hid})-[:manifestation_of]->(l:Lesion)
MATCH (l)<-[:manifestation_of]-(rf:Finding)
WHERE rf.modality IN ['MRI','CT','Ultrasound']
OPTIONAL MATCH (l)-[:supports]->(d:Diagnosis)<-[:confirmed_by]-(hf)
RETURN rf, d
```
Relies on: pathology adapter, registration (mechanism 3 for spatial confirmation),
correlation. Answers "the biopsy proved *which* imaging lesion is what".

## Discovery journeys *(embedding-backed)*

These use the **discovery role** (`02`, `06` mechanism 5). They go beyond *known* edges.

### F. This finding → visually/semantically similar findings (any modality)
```
discover_similar(finding $anchor, k=20, scope='cohort')
  → ANN over multimodal embeddings (L1 vector-search via L5)
  → ranked Findings across modalities & patients, each a CANDIDATE corresponds_to
```
Use: "what does this unusual lesion look like elsewhere?" Results are candidates the
clinician can confirm.

### G. This case → similar prior patients / cohort
```
discover_similar_cases(lesion $lid | patient $pid, k=10)
  → embed the lesion's manifestations + history → ANN over cohort case-vectors
  → ranked prior cases with their diagnoses/outcomes
```
Use: precedent-based reasoning ("how did similar cases progress / respond?").

### H. Free-text / history query → relevant findings & reports
```
search("tumefactive demyelination, steroid-responsive", filters)
  → hybrid lexical + vector retrieval (retrieval MCP via L5)
  → ranked findings/reports
```
Use: open-ended exploration grounded in the cohort.

## Trust filtering everywhere

Any journey above accepts a trust profile:
- **Strict** — `confirmed` links only, `asserted_by = human`. For clinical decisions.
- **Exploratory** — include `candidate` links and embedding discoveries. For hypothesis
  generation.

The UI (future **C6 Console**) chooses the profile; Connectome enforces it in the
traversal. Candidate/discovered results are always visually distinct from confirmed ones.

## Layer dependency recap

L6 calls **only L5**. It never queries an MCP server or the adapters directly — discovery
and correlation are surfaced *through* L5, preserving the dependency rule (`01`).
