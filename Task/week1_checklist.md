# Week 1 Checklist: Policy Ingestion, Chunking and Schemas

**Done when:** you can search the three policies and get back the right criterion text.
(A simple search is enough this week; hybrid retrieval and reranking come in Week 4.)

## 1. Repo setup
- [x] Create the folder layout: `data/policies/`, `data/cases/`, `src/agent/`, `src/retrieval/`, `src/schemas/`, `src/api/`, `app/`, `evals/`, `tests/`, `.github/workflows/`
- [x] Add `__init__.py` to each Python package
- [ ] Set up the environment (Python 3.11+, `uv` or `venv`) and `pyproject.toml` with `pydantic`, `rank_bm25`, `chromadb`, `pytest`
- [ ] Add `.gitignore`, `.env.example`, and a placeholder README with a "Synthetic data only, not for clinical use" note
- [ ] Make the first commit and push to GitHub

## 2. Get the policy text
- [ ] Download NCD 240.4 (CPAP for obstructive sleep apnea) from the CMS Medicare Coverage Database
- [ ] Download NCD 240.2 (Home oxygen)
- [ ] Download NCD 100.1 (Bariatric surgery for morbid obesity)
- [ ] Save each as clean text/markdown in `data/policies/raw/`, keeping section headings
- [ ] Record source URL, effective date and version for each in `data/policies/sources.json`

## 3. Chunk by criterion
- [ ] List each NCD's coverage criteria by hand (thresholds, conditions, exclusions)
- [ ] Make one chunk per criterion (not fixed-size chunks), keeping each criterion's conditions together
- [ ] Add metadata to each chunk: `ncd_id`, `service`, `section`, `criterion_id` (e.g. `240.4-C2`), `criterion_type` (requirement / exclusion / documentation), `threshold`, `source_url`, `effective_date`
- [ ] Write `src/retrieval/ingest.py` to produce `data/policies/chunks.jsonl`
- [ ] Check every chunk by eye against the source text

## 4. Pydantic schemas (`src/schemas/`)
- [ ] `PolicyChunk`: metadata fields + `text`
- [ ] `Case`: case ID, demographics, diagnoses (ICD-10), test results (AHI, SpO2, PaO2, BMI, with dates), prior treatments, requested service, notes
- [ ] `CriterionResult`: `criterion_id`, status (`met` / `not_met` / `missing_evidence`), patient-record quote, policy quote
- [ ] `Decision`: outcome (`approve` / `deny` / `needs_human_review`), list of `CriterionResult`, confidence, rationale
- [ ] Write one example JSON each for `Case` and `Decision` and confirm they validate (sets the format for Week 2 cases)

## 5. Simple search
- [ ] Write `src/retrieval/search.py`: BM25 over chunks, metadata filter by `service`/`ncd_id`, `search(query, service=None, k=5)` returning `PolicyChunk` objects
- [ ] (Optional) Embed chunks into Chroma to get a head start on Week 4

## 6. Verify ("done when")
- [ ] Write 10–15 test queries, each with its expected `criterion_id`
- [ ] pytest: schemas validate
- [ ] pytest: every chunk has complete metadata
- [ ] pytest: expected criterion appears in the top 3 results for each query
- [ ] Commit everything and tick off Week 1 in the plan

> Tip: finish CPAP (NCD 240.4) end to end first, then add the other two.
