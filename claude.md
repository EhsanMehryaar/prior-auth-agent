# Prior-Auth Agent

LangGraph agent that reviews synthetic prior-authorization requests against
Medicare NCD criteria and returns approve / deny / needs human review, with
a policy citation for every criterion. Roadmap: @docs/PLAN.md

## Commands
- Install: `pip install -e ".[dev]"`
- Ingest policies: `python -m src.retrieval.ingest`
- Run app: `streamlit run app/main.py`
- Tests: `pytest`
- Evals: `python -m evals.run_evals --set gold`  (must not regress vs. last report)

## Layout
- src/agent/      graph.py (nodes + edges), state.py, prompts/
- src/retrieval/  ingest, hybrid search, rerank
- src/schemas/    Pydantic models: Case, Criterion, Decision
- evals/          harness, metrics, reports/
- data/cases/     gold-labeled synthetic cases

## Rules
- Synthetic data only. Never add real patient data.
- Never edit data/cases/*gold* labels without asking me.
- Never loosen the CI eval gate thresholds to make a build pass.
- All LLM outputs go through Pydantic schemas; no free-text parsing.
- Prompts live in src/agent/prompts/, not inline in code.

## Workflow
- Before calling a task done: run pytest and evals, report the numbers.
- Keep changes small; one feature per commit.

## Gotchas
- (add as you find them)