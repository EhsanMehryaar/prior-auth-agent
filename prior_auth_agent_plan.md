# Prior-Authorization Agent: Project Plan

**Goal:** A portfolio project that shows agentic AI engineering for regulated decisions: multi-step reasoning, grounded retrieval, critique loops, human-in-the-loop review, and evaluation of whole agent runs.

**Timeline:** ~8 weeks of evenings, with a working demo by week 4.

---

## 1. Scope

**What it does:** takes a synthetic prior-authorization request (patient summary + requested service) and returns **approve / deny / needs human review**. Every criterion in the decision cites the exact policy text it relied on.

**First version: three services with clear, public Medicare criteria**
- CPAP for sleep apnea (NCD 240.4)
- Home oxygen (NCD 240.2)
- Bariatric surgery (NCD 100.1)

Three is enough to show it works across different policies. Add more only after the evaluation numbers are good.

---

## 2. Architecture (LangGraph)

```
intake → extract_facts → retrieve_policy → check_criteria → critic → judge
                                                ↑______________|  (revise if critic fails, max 2 loops)
judge → [confident]  → final_decision
      → [low conf / conflict] → human_review (interrupt) → final_decision
```

| Node | Role |
|---|---|
| **intake / extract_facts** | Uses the LLM with structured output (Pydantic) to pull diagnoses, test results (AHI, SpO2, BMI), dates and prior treatments from the case |
| **retrieve_policy** | Combines keyword (BM25) and vector search over the coverage rules, reranks the results, and returns criteria with section IDs |
| **check_criteria (proposer)** | Goes through criteria one by one: met / not met / missing evidence, with a quote from the patient record and a quote from the policy |
| **critic** | Looks for unsupported claims, wrong thresholds and missed criteria; if it finds a problem, sends the case back to the proposer |
| **judge** | Makes the final call and sets a confidence level; missing evidence routes to a human instead of a denial |
| **human_review** | LangGraph `interrupt()` with a SQLite checkpointer, so a run can pause, wait for a reviewer and resume |

**Reliability:** tool calls retry with backoff, any malformed JSON goes back to the model for repair, and every node has a step limit.

---

## 3. Stack

- **Core:** Python, LangGraph, Pydantic, one LLM API (Claude or OpenAI)
- **Retrieval:** Chroma or pgvector, rank_bm25, a reranker (a cross-encoder or Cohere)
- **Observability:** Langfuse
- **App:** FastAPI (backend), Streamlit (reviewer UI)
- **Engineering:** pytest, GitHub Actions (CI + eval gate), Docker

---

## 4. Repo Layout

```
prior-auth-agent/
  data/policies/        # NCD text, chunked + metadata
  data/cases/           # synthetic cases + gold labels (JSON)
  src/agent/            # graph.py, nodes/, state.py, prompts/
  src/retrieval/        # ingest.py, hybrid.py, rerank.py
  src/schemas/          # Pydantic models: Case, Criterion, Decision
  src/api/              # FastAPI
  app/                  # Streamlit reviewer UI
  evals/                # run_evals.py, metrics.py, reports/
  tests/
  .github/workflows/    # CI + eval gate
```

---

## 5. Week-by-Week Plan

| Week | Build | Done when |
|---|---|---|
| 1 | Get the three NCDs, chunk them by criterion, add metadata (section, criterion ID). Write the Pydantic schemas. | You can search the policies and get back the right criterion text |
| 2 | Write 30 synthetic cases with gold labels: approve, deny, missing evidence, edge cases near thresholds. Generate them with an LLM, then check every label by hand. | Gold set committed (the most important asset in the project) |
| 3 | Build the linear graph: extract_facts → retrieve → check_criteria → decision | It runs from start to finish on all 30 cases |
| 4 | Add hybrid retrieval with reranking, plus the Streamlit UI showing the decision, per-criterion evidence and citations | **Demo-able.** Deploy it |
| 5 | Add the critic and judge loop, confidence scores and the human-review step (interrupt + checkpointer) | A low-confidence case pauses, you approve it in the UI, and the run resumes |
| 6 | Add reliability: retries, repair of malformed output, step limits, fault injection (make a tool fail and confirm the run recovers) | It runs through injected failures without crashing |
| 7 | Add evaluation and tracing: decision accuracy, per-criterion accuracy, citation correctness, retrieval recall@k, failure-mode tagging, Langfuse traces. CI fails if accuracy drops. Expand the gold set to ~100 cases. | Eval report + CI gate |
| 8 | Write the README with an architecture diagram and eval results, a blog post on what failed and how you fixed it, and a short demo video | Public repo + live demo + post |

### Weekly checklist

- [ ] Week 1: Policy ingestion, chunking and schemas
- [ ] Week 2: 30 gold-labeled synthetic cases
- [ ] Week 3: Linear graph runs on all cases
- [ ] Week 4: Hybrid retrieval + reranking + Streamlit UI, deployed
- [ ] Week 5: Critic/judge loop + human-in-the-loop checkpoint
- [ ] Week 6: Retries, output repair, step limits, fault injection
- [ ] Week 7: Eval harness + Langfuse tracing + CI regression gate (~100 cases)
- [ ] Week 8: README, blog post, demo video

---

## 6. Metrics to Report

- Decision accuracy, and how often missing evidence correctly goes to a human instead of a wrong denial
- Per-criterion accuracy
- Citation precision (does the quoted text actually support the claim?)
- Retrieval recall@5
- Average steps, latency and cost per case
- **Before/after numbers for adding the critic** (the strongest resume bullet)

---

## 7. Pitfalls to Avoid

- Don't let the LLM write the cases and grade them without you checking. Your gold labels are the credibility of the whole project.
- Use only synthetic data, and put a clear "not for clinical use" note in the README.
- Get one service working end to end before adding a second.
- Don't add multiple agents or memory before week 5. A simple graph with good evaluation beats a complicated one without it.

---

## 8. Job Posting Coverage

| Posting area | Covered by |
|---|---|
| Multi-step reasoning, planning, tool use | Graph nodes, structured extraction, retrieval tool |
| Stateful workflows, retries, self-correction, human-in-the-loop | Critic loop, retries, `interrupt()` + checkpointer |
| Proposer/critic/judge review, auditable rationales | check_criteria → critic → judge with per-criterion citations |
| End-to-end RAG (chunking, hybrid retrieval, reranking) | Retrieval module |
| Observability and tracing | Langfuse |
| Trajectory/task-level evaluation, regression analysis | Eval harness + CI gate |
| Guardrails, escalation, auditability | Confidence routing, missing evidence → human, citation checks |
| Testing, CI/CD, documentation | pytest, GitHub Actions, README + blog post |
