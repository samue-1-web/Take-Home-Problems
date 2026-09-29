# Analyst + Auditor

## Clean-Machine Quick Start

**Intended execution path: Google Colab.**

The notebook reads `GEMINI_API_KEY` and `TAVILY_API_KEY` from `google.colab.userdata`, installs its external Python dependencies in the first setup cell, and executes cells in notebook order.

### Run in under 5 minutes

1. Open `Analyst_Auditor_FINAL.ipynb` in Google Colab.
2. In Colab Secrets/Userdata, create:
   - `GEMINI_API_KEY`
   - `TAVILY_API_KEY`
3. Run the notebook from the first cell downward.
4. The setup cell installs:
   - `google-genai`
   - `tavily-python`
   - `pydantic`
   - `requests`
   - `beautifulsoup4`
   - `lxml`
   - `pymupdf`
5. The notebook performs:
   - Tavily web search
   - HTML page retrieval using Requests + BeautifulSoup
   - PDF extraction using PyMuPDF
6. Review:
   - Benchmark section
   - Independent Auditor section
   - Feedback section
   - Conflict Resolution section
   - Sufficiency/Freshness section
   - Final Dashboard section

---

## What the System Does

This notebook implements an Analyst + Independent Auditor workflow for answering open-web research questions.

### Analyst

The Analyst:

- Plans research tasks
- Searches the web
- Retrieves pages and PDFs
- Extracts evidence
- Performs claim validation
- Maintains reusable memory
- Resolves source conflicts
- Checks evidence sufficiency and freshness
- Produces cited answers

### Auditor

The Auditor independently:

- Re-fetches cited URLs
- Extracts evidence again
- Evaluates claim support
- Assigns claim-level verdicts:

  - `SUPPORTED`
  - `UNSUPPORTED`
  - `CONTRADICTED`
  - `SOURCE_UNAVAILABLE`

The Auditor does not reuse Analyst-fetched evidence.

---

## Repository Structure

```text
Analyst_Auditor_FINAL.ipynb   Main notebook
README.md                     Setup and execution guide
requirements.txt             Python dependencies
DECISIONS.md                 Engineering decisions and lessons learned
logs/                        AI-assisted development session logs
```

---

## Dependencies

```text
google-genai
tavily-python
pydantic
requests
beautifulsoup4
lxml
pymupdf
```

Standard-library modules used:

```text
sqlite3
json
re
time
asyncio
uuid
datetime
typing
urllib.parse
collections
```

---

## API Keys

The notebook requires:

```text
GEMINI_API_KEY
TAVILY_API_KEY
```

Keys are read through:

```python
google.colab.userdata
```

No API keys are hard-coded.

---

## System Architecture

```text
Question
    ↓
Planner
    ↓
Memory
    ↓
Web Search
    ↓
Entity / Source Filtering
    ↓
Semantic Relevance
    ↓
Source Selection
    ↓
Page Fetching
    ↓
Evidence Extraction
    ↓
Claim Validation
    ↓
Sufficiency + Freshness Gate
    ↓
Citations + Answer
    ↓
Independent Auditor
    ↓
Feedback Rules
```

---

## Memory

Memory is stored in:

```text
memory/analyst_memory.db
```

Stored information includes:

- aliases
- facts
- source URLs
- document identifiers
- timestamps
- temporal scope

Freshness-sensitive questions trigger revalidation instead of blind memory reuse.

### Recorded Memory Experiment

| Metric | Cold | Warm |
|----------|----------|----------|
| Queries | 4 | 1 |
| Time | 2.299 s | 0.946 s |
| Query Reduction | 75% | |
| Time Reduction | 58.9% | |

---

## Auditor Results

Recorded audit summary:

| Metric | Value |
|----------|----------|
| Questions Processed | 8 |
| Claims Audited | 16 |
| Supported | 8 |
| Unsupported | 4 |
| Source Unavailable | 4 |
| Contradicted | 0 |

The audit exposed both unavailable citations and claims whose evidence was not sufficiently direct.

---

## Experiments

### 1. Memory Reuse

Compared cold search against warm memory-assisted retrieval.

### 2. Feedback Loop

Audit findings were converted into Analyst research rules.

### 3. Fair-Budget Feedback Evaluation

Both systems used equal search budgets.

Results:

- Baseline unique sources: 10
- Feedback unique sources: 14
- Primary sources: 2 vs 2

Conclusion:

```text
No primary-source coverage improvement under equal search budget.
```

### 4. Conflict Resolution

Resolved leadership-role disagreements using:

- source authority
- temporal reasoning
- primary-source prioritization

### 5. Adversarial Analyst

| Metric | Normal | Audit-Aware |
|----------|----------|----------|
| Queries | 3 | 5 |
| Primary Sources | 0 | 11 |
| Unique Sources | 13 | 21 |
| Time | 0.936 s | 2.522 s |

### 6. Sufficiency + Freshness Gate

Observed outputs:

```text
SUFFICIENT
RESEARCH_MORE
RESEARCH_MORE
```

---

## Final Dashboard Summary

### Memory Efficiency

- 75% query reduction
- 58.9% elapsed-time reduction

### Auditor

- 16 claims audited
- 8 supported
- 4 unsupported
- 4 source unavailable

### Feedback Loop

- 5 automatic Analyst rules generated

### Conflict Resolution

- Temporal difference resolved using primary sources

### Adversarial Analyst

- +11 primary sources when audit-aware

### Sufficiency Gate

- Prevents answering when evidence is missing or stale

---

## Limitations

1. Requires Google Colab userdata for API keys.
2. Notebook execution depends on notebook cell order.
3. Some preserved outputs originate from earlier runs.
4. Monetary API cost was not measured.
5. Contradiction detection is not fully implemented.
6. Q5 produced no confirmed Analyst output.
7. The integrated pipeline is demonstrated stage-by-stage rather than as a single verified end-to-end function.

---

## Future Work

- End-to-end integrated pipeline execution
- Stronger contradiction detection
- Better document identity handling
- Provider-level token and cost tracking
- Enforced benchmark timeout checks
- Additional robustness evaluation

---
