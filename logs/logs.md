# AI Development Logs

These logs summarize the AI-assisted development process used while building the Analyst + Auditor project.

---

# AI Coding Session 01 — Assignment Analysis and System Definition

**Purpose:** Understand the Analyst and Auditor assignment and identify what would distinguish a minimum-compliance solution from a stronger engineering submission.

## Developer

I need to build the Analyst and Auditor assignment. The evaluator explicitly says that simply completing the requirements is not enough. I need to investigate failures, test assumptions, and demonstrate meaningful engineering decisions.

The system needs an Analyst that performs live research and an independent Auditor that verifies the Analyst's claims.

## AI Assistant

The architecture should be treated as a research/evidence system rather than simply a search-and-answer pipeline.

Important requirements identified:

- research questions should require live web evidence
- the Analyst should plan before searching
- web search and page fetching should be real
- sensible work should be parallelized
- claims appearing in only one source should be cross-checked
- learning should carry across questions
- every answer should contain citations
- the system must explicitly state when evidence cannot be found
- at least eight questions should be tested
- at least two questions should reuse previously researched entities
- the Auditor should independently verify cited evidence

The take-it-further requirements should also be treated as experiments rather than merely implementation checkboxes.

## Engineering Decision

I decided to build a two-agent architecture:

**Analyst**

Question → Plan → Entity Resolution → Parallel Research → Evidence Processing → Validation → Conflict Resolution → Sufficiency Check → Answer

**Auditor**

Analyst Answer → Claim Extraction → Citation Check → Independent Source Retrieval → Evidence Verification → Verdict

The Auditor must not simply trust evidence already collected by the Analyst.

---

# AI Coding Session 02 — Analyst Research Architecture

**Purpose:** Design the research pipeline and live web-search execution.

## Developer

The Analyst needs to answer increasingly difficult research questions using live evidence. It also needs to plan before searching and use parallel search where appropriate.

## AI Assistant

The Analyst should separate planning from execution.

The research flow should be:

1. receive question
2. identify entities and required claims
3. generate a research plan
4. generate search queries
5. execute independent searches in parallel
6. collect sources
7. fetch relevant pages
8. extract evidence
9. compare evidence
10. resolve conflicts
11. check whether research is sufficient
12. answer with citations

Parallelism should be used for independent search queries rather than serially searching every query.

## Engineering Decision

I created benchmark questions around Titan Company Limited and later Infosys.

The Titan sequence deliberately reuses Ajoy Chawla and Titan Company Limited across multiple questions so that memory can potentially reduce repeated research.

The final benchmark contains eight questions ranging from straightforward leadership lookup to cross-source conflict analysis and a fresh Infosys research task.

---

# AI Coding Session 03 — Independent Auditor

**Purpose:** Build an Auditor that independently checks Analyst claims.

## Developer

The Auditor must verify every claim and should not simply reuse the Analyst's retrieved evidence.

## AI Assistant

The Auditor should operate independently:

Analyst answer  
→ extract atomic claims  
→ identify citation for each claim  
→ independently fetch cited URL  
→ inspect returned content  
→ determine whether evidence directly supports the claim  
→ assign verdict

Suggested verdicts:

- SUPPORTED
- UNSUPPORTED
- CONTRADICTED
- NO_CITATION

A citation that exists is not automatically evidence that supports the claim.

## Engineering Decision

I implemented independent source re-fetching.

The Auditor records:

- claim
- citation URL
- whether source could be fetched
- HTTP status
- source type
- evidence
- verdict
- reason
- elapsed time

This allows citation availability failures to be distinguished from evidence failures.

## Key Insight

A research system should verify the relationship between:

**claim → citation → source → evidence**

rather than only checking whether a URL is present.

---

# AI Coding Session 04 — Memory and Evidence Model

**Purpose:** Design memory so that previous research can accelerate later questions without becoming unquestioned truth.

## Developer

The assignment requires the Analyst to learn across questions. I need to show that later questions about an already researched entity become faster or cheaper.

## AI Assistant

Memory should not be treated as permanent truth.

A stored fact should retain:

- entity
- fact
- source
- retrieval information
- freshness information

The Analyst can reuse a previous fact while still performing a freshness check when necessary.

## Developer

I want to test this quantitatively.

## Engineering Decision

I created a controlled Titan memory-reuse experiment.

The stored memory included:

- Ajoy Chawla is Managing Director of Titan Company Limited.
- Ajoy Chawla previously led Titan's Jewellery Division.

### Result

Cold:

- 4 queries
- 2.299 seconds

Warm:

- 1 freshness-check query
- 0.946 seconds

Query reduction: **75.0%**

Latency reduction: **58.9%**

## Important Limitation

An earlier naive memory experiment failed to demonstrate an improvement because the memory was not actually hit.

This led to the decision to use a controlled entity-reuse experiment instead.

---

# AI Coding Session 05 — Conflict Resolution

**Purpose:** Test how the system handles apparently conflicting information.

## Engineering Decision

The research found Titan Company's official succession-plan document.

The official source states that Ajoy Chawla would succeed C.K. Venkataraman as Managing Director effective **1 January 2026**, while he had previously been CEO of the Jewellery Division.

The conflict resolver therefore classified the apparent disagreement as:

**TEMPORAL_DIFFERENCE_RESOLVED_BY_PRIMARY_SOURCE**

## Limitation

The current implementation is rule-based rather than a full temporal knowledge model.

---

# AI Coding Session 06 — Research Sufficiency and Freshness

**Purpose:** Prevent the Analyst from producing answers before sufficient evidence exists.

## Engineering Decision

I implemented a deterministic Research Sufficiency + Freshness Gate.

Possible outcomes:

- SUFFICIENT
- RESEARCH_MORE

### Tests

Fresh evidence → SUFFICIENT

Missing evidence → RESEARCH_MORE

Stale evidence → RESEARCH_MORE

---

# AI Coding Session 07 — Automatic Auditor Feedback

**Purpose:** Feed Auditor findings back into future Analyst research.

## Engineering Decision

I implemented automatic Auditor → Analyst feedback.

Generated rules:

1. Prefer independently retrievable citations.
2. Prefer primary company, regulatory, exchange, or official sources.
3. Match atomic claims to direct evidence.
4. Target exact financial statements.
5. Verify temporal context for leadership transitions.

### Equal-Budget Experiment

Baseline:

- 8 queries
- 10 unique sources
- 2 primary sources

Feedback-aware:

- 8 queries
- 14 unique sources
- 2 primary sources

Conclusion:

Feedback increased evidence breadth but did not improve primary-source coverage under equal search budget.

---

# AI Coding Session 08 — Adversarial Analyst

**Purpose:** Test whether audit awareness changes research behaviour.

### Normal Analyst

- 3 queries
- 0 primary sources
- 13 unique sources
- 0.936 seconds

### Audit-Aware Analyst

- 5 queries
- 11 primary sources
- 21 unique sources
- 2.522 seconds

### Conclusion

Audit awareness increased primary-source usage but also increased latency and search cost.

---

# AI Coding Session 09 — Auditor Evaluation and Failure Classification

**Purpose:** Evaluate the Auditor over the benchmark.

### Final Audit Results

- 8 questions processed
- 16 claims audited
- 8 supported
- 4 unsupported
- 4 source unavailable
- 0 contradicted

### Lesson

A failed citation fetch is not equivalent to a false claim.

Different failure modes must remain separate.

---

# AI Coding Session 10 — Cost, Quota, and Experimental Honesty

**Purpose:** Handle cost reporting without fabricating measurements.

## Engineering Decision

I did not invent monetary cost numbers.

Instead I reported measurable quantities:

- search-query reduction
- latency reduction
- source counts
- primary-source coverage

### Limitation

The requested monetary cost-halving target was not demonstrated.

---

# AI Coding Session 11 — Final Dashboard and Validation

**Purpose:** Assemble and validate final evaluation results.

### Dashboard Summary

Memory:

- 75% query reduction
- 58.9% latency reduction

Auditor:

- 8 questions
- 16 claims
- 8 supported
- 4 unsupported
- 4 source unavailable
- 0 contradicted

Feedback:

- 5 automatic rules generated

Conflict Resolution:

- temporal role conflict resolved

Sufficiency:

- sufficient vs research-more behavior validated

### Validation

Final dashboard classification:

8 + 4 + 4 + 0 = 16 claims

Validation passed.

---

# AI Coding Session 12 — Final Notebook Cleanup and Submission Packaging

**Purpose:** Prepare final submission notebook.

### Final Notebook

`Analyst_Auditor_FINAL.ipynb`

### Cleanup Summary

- Original notebook cells retained: 111
- Original notebook cells removed: 41
- Final notebook cells: 125
- New documentation cells: 14

### Validation

Checked:

- notebook structure
- execution dependencies
- benchmark variables
- memory experiments
- auditor results
- feedback experiments
- conflict resolution
- sufficiency gate
- dashboard
- API-key leakage

No secrets or API keys were included.

### Final Engineering Position

Evidence verification should be treated as a first-class problem.

The system should identify when evidence is:

- missing
- stale
- unavailable
- indirect
- unsupported
- apparently conflicting

rather than assuming that retrieval alone guarantees correctness.
