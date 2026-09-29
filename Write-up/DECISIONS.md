# Engineering Decisions

## 1. Architecture Chosen

I chose a layered Analyst + Independent Auditor design:

```text
Question
    ↓
Planning
    ↓
Memory
    ↓
Parallel Search
    ↓
Entity / Source Filtering
    ↓
Semantic Relevance
    ↓
Source Selection
    ↓
Page Fetch
    ↓
Evidence Extraction
    ↓
Provenance
    ↓
Claim Validation
    ↓
Sufficiency / Freshness
    ↓
Citations / Answer
    ↓
Trace
```

The notebook contains an integrated pipeline function that follows these stages, but that function stops after fetching and returns intermediate state. Later benchmark cells connect evidence extraction, validation, and answers separately. I therefore treat the architecture as demonstrated stage-by-stage rather than as one verified end-to-end callable pipeline.

The Auditor is deliberately separate: it re-fetches cited URLs instead of reusing Analyst-fetched evidence.

---

## 2. Why I Chose This Architecture

I chose separation because search relevance is not the same as evidence quality. A result can mention the correct entity without directly supporting a claim.

To address this, I added:

- entity and source filtering
- semantic relevance evaluation
- page fetching
- evidence extraction
- provenance tracking
- claim validation
- sufficiency and freshness checks

When semantic relevance fails, the pipeline preserves `RELEVANCE_UNAVAILABLE` rather than silently discarding the source.

I used concurrent search because individual queries are independent. The notebook implements this using:

```python
asyncio.to_thread()
asyncio.gather()
```

around Tavily search operations.

I also replaced fixed character-window evidence extraction with sentence- and paragraph-level context after observing that fixed windows could break relationships between entities, roles, organizations, and dates.

---

## 3. My Decisions on Memory and Evidence

I chose SQLite for persistent entity memory.

Stored records include:

- aliases
- facts
- source URLs
- document identifiers
- temporal scope
- retrieval timestamps

Freshness-sensitive questions trigger fresh web verification instead of blind memory reuse.

Two memory experiments produced different outcomes.

The earlier experiment failed to demonstrate a benefit:

- no Titan/Ajoy memory hit detected
- both conditions used four searches
- estimated evidence tokens remained 2,225
- warm run took 2.336 s
- cold run took 1.919 s

A later controlled experiment explicitly created reusable Titan memory and compared:

| Metric | Cold | Warm |
|----------|----------|----------|
| Queries | 4 | 1 |
| Time | 2.299 s | 0.946 s |

Results:

- 75% query reduction
- 58.9% elapsed-time reduction
- two facts reused

I treat this as evidence that memory reuse can reduce redundant search in a controlled setting. I do not interpret it as API-cost reduction or proof that SQLite always yields the same savings.

Evidence records retain:

- source URLs
- timestamps
- passages
- provenance metadata
- temporal metadata

The notebook distinguishes:

- direct evidence
- supporting evidence
- weak contextual evidence

---

## 4. How I Handled Conflict Resolution

I treated the Titan leadership example as a temporal and provenance problem before treating it as a contradiction.

The resolver examines:

- role descriptions
- dates
- source authority

while prioritizing primary company sources.

The recorded outcome was:

```text
TEMPORAL_DIFFERENCE_RESOLVED_BY_PRIMARY_SOURCE
```

Jewellery Division leadership refers to the pre-transition period, while Managing Director references describe the post-transition period.

I do not present this as a general contradiction solver. It is a deterministic date-and-authority resolver demonstrated on this specific case.

---

## 5. What I Tried and Rejected

I initially used fixed character windows around keywords.

I later replaced them with sentence- and paragraph-based extraction after identifying evidence-boundary failures.

I also built:

1. a per-source Gemini relevance evaluator
2. a batched Gemini relevance evaluator

The batched version evaluates multiple candidates together and verifies that every source ID appears exactly once.

My first feedback experiment was confounded:

- baseline used 1 query
- feedback-aware version used 8 queries

Because the search budgets differed, I did not treat the results as a fair comparison.

I therefore reran the experiment with equal budgets.

Results:

| Metric | Baseline | Feedback-Aware |
|----------|----------|----------|
| Queries | 8 | 8 |
| Unique Sources | 10 | 14 |
| Primary Sources | 2 | 2 |

Conclusion:

**No primary-source coverage improvement under equal search budget.**

I retained the Gemini path but used the deterministic planner fallback after Gemini quota exhaustion. I did not represent fallback outputs as Gemini-planner results.

---

## 6. What Broke and What I Learned

### Unavailable citations

The Auditor found four cited URLs that could not be independently retrieved.

### Unsupported evidence

Four claims had retrievable sources but insufficiently direct supporting evidence.

### Q5 benchmark case

The final audit input explicitly marks the Analyst output as unconfirmed, so Q5 contributed zero audited claims.

### Memory

The earlier warm-memory experiment produced no memory hit and increased elapsed time.

### Feedback evaluation

The initial unequal-budget comparison was not a fair causal evaluation. The equal-budget rerun corrected this issue.

### Gemini quota

Quota exhaustion forced use of the deterministic fallback path.

### Notebook execution state

Static inspection found:

- a cell referencing `entity_classified_sources` before it is created
- later cells referencing `test_plan`, which is never defined

Because of this, I cannot guarantee fresh-runtime execution without notebook cleanup.

### Contradiction detection

The Auditor includes a `CONTRADICTED` category, but the implemented verifier does not demonstrate a true contradiction detector. The recorded contradicted count remained zero.

### Main lesson

Relevant retrieval does not automatically produce verifiable evidence.

---

## 7. What I Would Do With Two More Weeks

My first priority would be making the notebook fully fresh-runtime safe:

- remove duplicate cells
- remove stale state
- fix execution order
- resolve undefined variables
- consolidate benchmarking into a single callable pipeline

Second, I would replace heuristic claim-evidence matching with a verifier capable of distinguishing support from contradiction.

Third, I would make document identity a first-class concept so mirrored copies of the same document are not treated as independent evidence.

Fourth, I would add provider telemetry:

- Gemini request counts
- Tavily request counts
- billed token usage
- latency metrics
- failure statistics
- monetary cost tracking

This would allow actual cost measurements instead of proxies.

Finally, I would enforce the 120-second benchmark requirement using explicit timeout assertions.

---

## 8. Final Decision

I chose an architecture where the Analyst can search broadly while the Auditor is allowed to distrust its citations.

The main engineering lesson is:

> Retrieval quality and evidence quality are different problems.

Memory, filtering, provenance, temporal reasoning, and feedback can reduce redundant work or alter search behavior, but independent re-fetching is what reveals whether evidence is both retrievable and sufficiently direct.

If I continued developing this project, I would focus future improvements on that verification boundary rather than treating search relevance as proof.
