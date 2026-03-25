---
name: weave-eval
description: "Evaluate and benchmark RAG solutions using weave-cli. Use when: (1) creating evaluation datasets, (2) running evals against agents, (3) benchmarking multiple agents, (4) creating custom evaluators (LLM judge or human), (5) analyzing eval results and recommending improvements. NOT for: initial setup (use weave-setup), data ingestion (use weave-ingest), querying (use weave-search), or deployment (use weave-stack)."
homepage: https://github.com/maximilien/weave-cli
tags: ["rag", "data", "weave", "vector-database"]
metadata:
  {
    "openclaw":
      {
        "emoji": "🎯",
        "requires": { "bins": ["weave"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "maximilien/tap/weave-cli",
              "bins": ["weave"],
              "label": "Install weave-cli (brew)"
            },
            {
              "id": "go",
              "kind": "go",
              "package": "github.com/maximilien/weave-cli",
              "bins": ["weave"],
              "label": "Install weave-cli (go install)"
            }
          ]
      }
  }
---

# weave-eval

Evaluate and benchmark RAG solutions — create datasets, run evals, compare agents, and iterate on quality.

## When to Use

- Creating baseline and test evaluation datasets
- Running evaluations against RAG/QA/summarize agents
- Benchmarking multiple agents or configurations
- Creating custom evaluators (LLM judge, human review)
- Analyzing results and recommending improvements

## When NOT to Use

- Initial setup -> `weave-setup` skill
- Data ingestion -> `weave-ingest` skill
- Querying and search -> `weave-search` skill
- Stack deployment -> `weave-stack` skill

## Evaluation Datasets

### Dataset Format (YAML)

```yaml
# evals/datasets/baseline.yaml
- id: test-001
  description: Basic factual lookup
  query: "What is the return policy for damaged items?"
  expected_answer: "Items damaged in transit can be returned within 30 days"
  required_concepts:
    - return policy
    - damaged items
    - 30 days
  must_cite: true
  min_relevance_score: 0.75

- id: test-002
  description: Multi-document synthesis
  query: "Compare pricing between Q1 and Q2"
  expected_answer: "Q1 average was $150, Q2 rose to $180"
  required_concepts:
    - Q1 pricing
    - Q2 pricing
    - comparison
  min_relevance_score: 0.70
```

### Dataset Management

```bash
weave eval datasets list
weave eval datasets create baseline --dataset baseline.yaml
```

## Running Evaluations

### Single Agent Eval

```bash
weave eval run \
  --agent rag-agent \
  --dataset baseline \
  --collection MyDocs
```

### Benchmark Multiple Agents

```bash
weave eval benchmark \
  --agents rag-agent,qa-agent,summarizer \
  --dataset baseline
```

### View Results

```bash
weave eval list                          # List all eval runs
weave eval show run-20260325-140530      # Show specific run
```

## Evaluators

### Built-in Evaluators

| Evaluator | What It Measures |
|-----------|-----------------|
| AccuracyEvaluator | Semantic similarity between response and expected answer |
| CitationEvaluator | Citation quality — are sources properly referenced? |

### Custom Evaluators

```bash
# List available evaluators
weave eval list-evaluators

# Validate a custom evaluator
weave eval validate-evaluator ./my-evaluator.yaml

# Create new evaluator
weave eval create-evaluator
```

### LLM Judge Pattern

Create an LLM-as-judge evaluator for automated quality assessment:

```yaml
# evaluators/llm-judge.yaml
name: llm-quality-judge
type: llm
model: gpt-4o
temperature: 0.1
criteria:
  - relevance: "Does the answer address the question?"
  - accuracy: "Are the facts correct based on source documents?"
  - completeness: "Are all key points covered?"
  - citation_quality: "Are sources properly cited?"
scoring: 1-5
```

## Eval Thresholds (from auctionsmax-ai reference)

| Metric | Minimum | Target |
|--------|---------|--------|
| Accuracy | >= 0.70 | >= 0.85 |
| Citation | >= 0.75 | >= 0.90 |
| Relevance | >= 0.60 | >= 0.80 |

## Iteration Workflow

When eval results are below threshold:

1. **Low relevance?** Try hybrid search, increase top-k, different embedding model
2. **Low accuracy?** Switch to QA agent (lower temperature), add system prompt constraints
3. **Low citation?** Enable strict mode, require must_cite in agent config
4. **Inconsistent?** Check chunking — try larger chunks or more overlap
5. **All low?** Review data quality, re-ingest with better schema

## Parallel Eval Pattern (for ClawMax multiagent)

```bash
# Agent 1: Baseline eval
weave eval run --agent rag-agent --dataset baseline

# Agent 2: Multimodal eval
weave eval run --agent rag-agent --dataset multimodal

# Agent 3: Image quality eval
weave eval run --agent rag-agent --dataset image-search
```

## Decision Workflow

1. Create baseline dataset: 5-10 representative questions with expected answers
2. Run first eval: `weave eval run --agent rag-agent --dataset baseline`
3. Analyze results: Check accuracy, citation, relevance scores
4. If below threshold: adjust (search mode, agent type, embeddings, chunking)
5. Re-run eval to measure improvement
6. When satisfied: create production dataset (50+ questions)
7. Run comprehensive benchmark across agent types
8. Hand off to `weave-stack` for production deployment
