# Autoresearch for Agentic Systems

Applying Karpathy's [autoresearch](https://github.com/karpathy/autoresearch) pattern to agentic AI systems. Instead of autonomously optimizing LLM training code, we autonomously optimize agent prompts and configurations to improve measurable output quality.

The core insight: **the autoresearch pattern is not about model training — it's a general framework for autonomous self-improvement of any system with a measurable quality metric.**

## Projects

### [Project 1: Financial AutoAgent](Project1-Financial_Autoagentic_System/)

A self-improving financial research pipeline that analyzes SEC 10-K filings. Three agents (extractor, analyst, synthesizer) process filings end-to-end, while an optimizer agent autonomously iterates on their skill prompts to maximize a composite quality score evaluated against XBRL ground truth.

- **Best composite score:** 0.7340 (from 0.7235 baseline, +1.5%)
- **Optimizable surface:** 3 skill file prompts (1 dimension)
- **Evaluation set:** 13 companies across 6 sectors
- **Metric:** Weighted blend of extraction accuracy, analysis quality, and cost efficiency

### [Project 2: AutoRAG](Project2-AutoRAG/)

A self-improving RAG system tested against Meta's CRAG benchmark across 5 domains (finance, sports, music, movie, open). The optimizer tunes not just prompts but the entire RAG stack: chunking strategy, embedding model, retrieval parameters, model routing, and pipeline topology.

- **Best CRAG score:** 0.360 (from 0.208 baseline, +73%)
- **Optimizable surface:** 7 dimensions (config.yaml + 4 skill files)
- **Evaluation set:** 500 questions, 5 domains, 8 question types
- **Metric:** CRAG Score_a (accuracy - hallucination_rate)
- **Key finding:** The optimizer learned to prioritize hallucination reduction over accuracy, found optimal confidence thresholds through experimentation, and discovered that more expensive models only help at specific pipeline stages.

## Architecture Pattern

```
Karpathy's autoresearch          This repo
========================         ========================
prepare.py (fixed eval)    →     evaluate.py (fixed eval)
train.py (agent edits)     →     agents/skills/*.md (optimizer edits)
val_bpb (metric)           →     composite_score (metric)
program.md (strategy)      →     optimizer_program.md (strategy)
5-min GPU budget            →     fixed API cost budget
git keep/discard            →     git keep/discard
```

## License

MIT
