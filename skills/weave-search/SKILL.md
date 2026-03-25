---
name: weave-search
description: "Query vector databases and manage AI agents using weave-cli. Use when: (1) running semantic, BM25, or hybrid searches, (2) creating and configuring RAG/QA/summarize agents, (3) comparing embedding models, (4) re-embedding collections, (5) tuning search parameters. NOT for: initial setup (use weave-setup), data ingestion (use weave-ingest), evaluations (use weave-eval), or deployment (use weave-stack)."
homepage: https://github.com/maximilien/weave-cli
tags: ["rag", "data", "weave", "vector-database"]
metadata:
  {
    "openclaw":
      {
        "emoji": "🔍",
        "requires": { "bins": ["weave"] },
        "install":
          [
            {
              "id": "go",
              "kind": "go",
              "package": "github.com/maximilien/weave-cli",
              "bins": ["weave"],
              "label": "Install weave-cli (go install)",
            },
          ],
      },
  }
---

# weave-search

Query vector databases and manage AI agents for RAG, QA, and summarization.

## When to Use

- Running semantic, BM25, or hybrid searches against collections
- Creating and managing weave AI agents (RAG, QA, summarize, custom)
- Comparing embedding models across collections
- Re-embedding collections with different models
- Tuning search relevance and parameters

## When NOT to Use

- Initial setup and config -> `weave-setup` skill
- Data ingestion -> `weave-ingest` skill
- Evaluations and benchmarks -> `weave-eval` skill
- Stack deployment -> `weave-stack` skill

## Querying

### Direct Collection Query

```bash
# Semantic search (default)
weave collection query MyDocs "what is the return policy?"
weave collection query MyDocs "find documents about pricing" --top-k 10

# Shorthand
weave query MyDocs "search terms"
```

### Search Modes

```bash
# Semantic (vector similarity)
weave collection query MyDocs "query" --mode semantic

# BM25 (keyword matching)
weave collection query MyDocs "query" --mode bm25

# Hybrid (semantic + BM25 combined)
weave collection query MyDocs "query" --mode hybrid
```

### Metadata Filtering

```bash
weave collection query MyDocs "query" \
  --filter '{"category": "pricing", "year": 2026}'
```

## AI Agents

### Built-in Agent Types

| Type | Temperature | Best For |
|------|------------|----------|
| `rag` | 0.7 | General document Q&A, research |
| `qa` | 0.3 | Precise answers, customer support, compliance |
| `summarize` | 0.5 | Document summaries, bullet-point overviews |
| `custom` | User-defined | Specialized use cases |

### Agent Management

```bash
# Create agent
weave agents create my-rag --type rag
weave agents create my-qa --type qa
weave agents create my-summarizer --type summarize
weave agents create custom-agent --type custom

# List agents
weave agents list

# Show agent config
weave agents show my-rag

# Edit agent config
weave agents edit my-rag

# Copy agent
weave agents copy my-rag my-rag-v2

# Delete agent
weave agents delete my-rag
```

### Agent Configuration (weave-agents.yaml)

```yaml
agents:
  - name: auction-rag
    type: rag
    model: gpt-4o
    temperature: 0.1
    max_tokens: 2000
    collections:
      - AuctionListings
    system_prompt: |
      You are an auction catalog expert.
      Always cite sources with [1], [2] notation.
```

## Re-embedding & Comparison

### Re-embed Collection (20x faster than re-ingestion)

```bash
# Re-embed with OSS model
weave collection re-embed MyDocs \
  --new-embedding sentence-transformers/all-mpnet-base-v2 \
  --output MyDocs_OSS
```

### Compare Embedding Models

```bash
# Side-by-side comparison
weave collection compare MyDocs MyDocs_OSS \
  --query "search terms" \
  --report comparison.md
```

## Search Tuning

### Top-K Results

```bash
weave query MyDocs "query" --top-k 5    # Fewer, more relevant
weave query MyDocs "query" --top-k 20   # More results, broader
```

### JSON Output

```bash
weave query MyDocs "query" --json
```

## Parallel Search Pattern (for ClawMax multiagent)

Multiple search agents can test different query strategies concurrently:
```bash
# Agent 1: Semantic search tuning
weave query MyDocs "query" --mode semantic --top-k 5

# Agent 2: Hybrid search tuning
weave query MyDocs "query" --mode hybrid --top-k 10

# Agent 3: Different agent type
weave agents create qa-strict --type qa
```

## Decision Workflow

1. Start with semantic search: `weave query MyDocs "test query"`
2. Try hybrid if keyword precision needed
3. Create appropriate agent type (RAG for general, QA for precise)
4. Compare embedding models if quality isn't sufficient
5. Re-embed with better model if needed
6. Hand off to `weave-eval` for systematic quality measurement
