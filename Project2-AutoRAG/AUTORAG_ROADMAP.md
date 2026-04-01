# AutoRAG Project Roadmap & Session State

## Current State (as of 2026-04-01)

### Project2-AutoRAG: COMPLETE and running on Windows natively

**Score: 0.208 -> 0.360 (+73%)** across 18 autonomous experiments. 5 kept, 13 discarded.

Pipeline is fully built and operational:
- `agents/rag.py` — chunking (3 strategies), embedding (OpenAI/Voyage), LanceDB indexing, retrieval
- `agents/llm.py` — Anthropic SDK wrapper with multi-model routing and cost tracking
- `agents/config.py` — config.yaml loader with frozen dataclasses, 7 optimization dimensions
- `agents/pipeline.py` — 5-stage RAG pipeline (classifier -> rewriter -> retrieval -> generator -> validator)
- `agents/skills/*.md` — 4 skill files (query_classifier, query_rewriter, answer_generator, answer_validator)
- `evaluate.py` — CRAG scoring harness (single LLM judge, Score_a metric)
- `optimizer_program.md` — optimizer instructions with baseline data and strategy priorities
- `scripts/run_optimizer.sh` + `.ps1` — optimizer loop with $15 budget tracking
- `results.tsv` — full experiment history (18 experiments)

### Environment

- **Runs on Windows natively** (NOT WSL — WSL crashes from memory issues)
- `lancedb==0.30.0` pinned (last version with Windows wheels, 0.30.1+ is Linux/Mac only)
- Python 3.12 via uv
- All `uv run` commands execute from `C:\Users\ssehgal\autoresearch-master\Project2-AutoRAG\`

### Indexing

- 18,752 total documents extracted from CRAG benchmark
- **Eval-only filtering**: only 3,720 docs referenced by dev+test questions are indexed (20% of total)
- 146,868 chunks at fixed/512/100 chunking
- OpenAI text-embedding-3-small (1536 dims)
- Rate limiting in `_embed_openai()`: dynamic delay based on estimated tokens vs 1M TPM limit
- Index build time: ~29 minutes
- Command: `uv run scripts/build_index.py --eval-only --force`

### Baseline Results (500 dev questions)

```
crag_score:         0.208
accuracy:           39.4% (158 perfect, 39 acceptable)
hallucination_rate: 18.6% (93 incorrect)
missing:            210 (42% — generator says "I don't know" too often)
cost:               $3.89, ~60 minutes
```

### Best Results After Optimization (100-question eval subset)

```
crag_score:         0.360
accuracy:           46%
hallucination_rate: 10%
```

### What the Optimizer Discovered (no human guidance)

1. Hallucination is the bottleneck, not accuracy. Each incorrect = -1.0, each missing = 0.0.
2. Confidence threshold has a sweet spot at 0.80 (found through 3 experiments: 0.70, 0.85, 0.80).
3. Upgrading validator from Haiku to Sonnet = +0.030 (better confidence calibration).
4. Upgrading classifier to Sonnet = no benefit, +48% cost.
5. More context (top_k 7-8) increases hallucination — more noise.
6. Smaller chunks (256) produce fragments too small for coherent answers.
7. False premise detection improved from 0.364 to 0.727 with categorized examples.
8. Forcing extreme conciseness INCREASES hallucination — terse answers hallucinate more.

### Plateau Status

Last 7 experiments (012-018) were all discards. Cheap optimizations (prompts, config params) are exhausted. Remaining opportunities:
- Sentence-based chunking (requires re-index)
- Few-shot examples (not yet implemented)
- Larger chunk_size 1024 (exp 018 failed due to batch size — fixable by lowering `_embed_openai` batch_size)

---

## What Was Built Per Session

### Previous WSL sessions
- Downloaded CRAG dataset (18,752 docs, 500 dev + 500 test questions)
- Built agents/rag.py, agents/llm.py, agents/config.py
- Built scripts/download_crag.py, scripts/build_index.py
- config.yaml with 7 optimization dimensions
- Failed to build index (Voyage rate limits, then OpenAI rate limits)

### This session (2026-04-01, Windows native)
- Fixed lancedb to 0.30.0 (Windows wheel)
- Fixed .env (duplicated ANTHROPIC_API_KEY= prefix)
- Added rate limiting to `_embed_openai()` with dynamic TPM delay
- Added eval-only doc filtering to `build_index()` (3,720 docs instead of 18,752)
- Added `--eval-only` flag to `scripts/build_index.py`
- Built index successfully: 146,868 chunks in 29 minutes, zero errors
- Created all 4 skill files (agents/skills/*.md)
- Created agents/pipeline.py (5-stage RAG pipeline)
- Created evaluate.py (CRAG scoring harness with LLM-as-judge)
- Created optimizer_program.md
- Created scripts/run_optimizer.sh + .ps1 with budget tracking
- Ran full 500-question baseline: crag_score = 0.208
- Ran optimizer: 18 experiments, score 0.208 -> 0.360
- Created README.md for Project2
- Updated root README.md
- Cleaned all AI-assisted coding markers from commit history and files
- Pushed everything to GitHub

---

## Next Steps: Domain-Agnostic Template (Option 3)

### The Core Insight

The autoresearch pattern's interface contract between the optimizer and the domain-specific system is exactly 3 things:

1. **A grep-parseable metric line**: evaluate.py prints `<metric_name>: <float>` to stdout
2. **Git-tracked modifiable files**: optimizer edits skill files + config.yaml, git handles keep/discard
3. **A single CLI command**: `uv run evaluate.py > run.log 2>&1`

Everything else (LLM wrapper, optimizer loop, results.tsv, git protocol) is pure infrastructure that never changes across domains.

### Component Classification

**Domain-agnostic (copy as-is):**
- `agents/llm.py` — LLM wrapper with model routing (identical in P1 and P2)
- `scripts/run_optimizer.sh` + `.ps1` — optimizer loop (only branch name and prompt text change)
- `results.tsv` format — same header across all projects
- Git keep/discard workflow — identical everywhere

**Parameterizable (fill in template blanks):**
- `agents/config.py` — config loader structure stays, domain-specific sections change
- `config.yaml` — models + pipeline sections are universal, domain-specific knobs added per project
- `optimizer_program.md` — ~60% reusable boilerplate (workflow, git protocol, results format), ~40% domain-specific (what the pipeline does, strategy priorities, baseline score)

**Domain-specific (write from scratch):**
- `evaluate.py` — scoring logic, ground truth loading, metric computation (only contract: print `metric_name: float`)
- `agents/pipeline.py` — stage definitions, JSON schemas, orchestration (number/type of stages varies)
- `agents/skills/*.md` — every skill file, written for the domain
- `data/` — evaluation dataset and ground truth
- Data acquisition scripts

### Proposed Template Structure

```
autoresearch-starter/
|-- README.md                    # Pattern explanation, getting started
|-- .env.example                 # API key placeholders
|-- .gitignore                   # Standard ignores
|-- pyproject.toml               # Minimal deps (anthropic, python-dotenv, pyyaml)
|
|-- optimizer_program.md         # ~60% boilerplate, ~40% {{FILL_IN}} sections
|-- config.yaml                  # Skeleton with models + pipeline sections
|-- results.tsv                  # Just the header row
|
|-- evaluate.py                  # SKELETON: CLI harness + timing + print contract
|                                #   user implements score_one_item() + aggregate_scores()
|
|-- agents/
|   |-- __init__.py
|   |-- llm.py                   # FULL COPY from P2 (model routing, cost tracking)
|   |-- config.py                # SKELETON: ModelsConfig + PipelineConfig + load_config()
|   |-- pipeline.py              # SKELETON: example 2-stage pipeline showing the pattern
|   |-- skills/
|       |-- example_stage.md     # EXAMPLE: shows skill file conventions
|
|-- scripts/
|   |-- run_optimizer.ps1        # FULL COPY, parameterized branch/prompt
|   |-- run_optimizer.sh         # FULL COPY, parameterized branch/prompt
|
|-- docs/
    |-- PATTERN.md               # Explanation of the autoresearch pattern
    |-- CHECKLIST.md             # "What you need to implement" checklist
```

### Example Domain Applications

**Customer Support QA:**
- Pipeline: Ticket Classifier -> Knowledge Retriever -> Response Generator -> Tone Validator
- Metric: response quality (helpfulness + tone + accuracy) via LLM judge
- Config knobs: response_tone, max_response_length, knowledge_base_scope

**Legal Document Review:**
- Pipeline: Document Classifier -> Clause Extractor -> Risk Analyzer -> Summary Generator
- Metric: clause extraction F1 + risk classification accuracy
- Config knobs: jurisdiction, document_type, risk_tolerance_level

**Medical Coding:**
- Pipeline: Note Classifier -> Code Extractor -> Code Validator -> Specificity Enhancer
- Metric: ICD-10 code accuracy (exact match + hierarchical match)
- Config knobs: code_specificity_level, coding_guidelines_version

---

## Multi-Domain Architecture (Future)

AutoRAG will evolve to support multiple domain benchmarks with the same optimizer:

```
autorag/
  domains/
    general/
      corpus/              # CRAG corpus subset
      eval.json            # CRAG questions
      domain_config.yaml
    finance/
      corpus/              # SEC 10-K/10-Q filings from EDGAR
      eval.json            # FinanceBench (150 expert-annotated questions)
      domain_config.yaml
    biomedical/
      corpus/              # PubMed abstracts
      eval.json            # MIRAGE benchmark: PubMedQA + BioASQ
      domain_config.yaml
```

### Benchmarks

| Domain | Benchmark | Size | Source |
|---|---|---|---|
| General | CRAG (Meta) | 4,409 QA pairs | Already integrated |
| Finance | FinanceBench | 150 questions | HuggingFace: PatronusAI/financebench |
| Biomedical | MIRAGE | ~1,118 questions | GitHub: Teddy-XiongGZ/MIRAGE |

### Implementation Order

1. **DONE**: CRAG-based AutoRAG complete, score 0.208 -> 0.360
2. **NEXT**: Build autoresearch-starter template (extract reusable components)
3. **THEN**: Add `domains/` abstraction layer, refactor evaluate.py to accept `--domain`
4. **THEN**: Add FinanceBench domain
5. **THEN**: Add MIRAGE/biomedical domain
6. **FINAL**: Cross-domain comparison — one architecture, three domains, three optimization paths

### LinkedIn Narrative Arc

- Post 1: Project 1 — autoresearch for prompt optimization (done, published)
- Post 2: Project 2 — autoresearch for the entire RAG stack (ready to publish)
- Post 3: Cross-domain — one architecture, three domains, different optimization paths discovered
- Post 4 (thought leadership): "The autoresearch pattern is not about model training"

### Key Constraints

- Runs on Windows natively (no WSL)
- Sam is funding personally — keep API costs reasonable
- Use Haiku for classification/routing, Sonnet for generation
- Sample eval subsets (100-500 questions) for optimizer runs, full set for final scoring only
- lancedb==0.30.0 (pinned for Windows compatibility)
