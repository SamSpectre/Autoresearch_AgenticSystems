# Autoresearch for Agentic Systems

Applying Karpathy's [autoresearch](https://github.com/karpathy/autoresearch) pattern to agentic AI systems. Instead of autonomously optimizing LLM training code, we autonomously optimize agent prompts and configurations to improve measurable output quality.

The core insight: **the autoresearch pattern is not about model training — it's a general framework for autonomous self-improvement of any system with a measurable quality metric.**

## Projects

### [Project 1: Financial AutoAgent](Project1-Financial_Autoagentic_System/)

A self-improving financial research pipeline that analyzes SEC 10-K filings. Three agents (extractor, analyst, synthesizer) process filings end-to-end, while an optimizer agent autonomously iterates on their skill prompts to maximize a composite quality score evaluated against XBRL ground truth.

- **Baseline composite score:** 0.7235
- **Evaluation set:** 15 companies across 6 sectors
- **Metric:** Weighted blend of extraction accuracy, analysis quality, and cost efficiency

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
