---
name: weave-planner
description: "Plan and orchestrate end-to-end RAG solutions using weave-cli. Use when: (1) gathering user requirements for a RAG project, (2) designing the solution architecture (VDB, embeddings, agents, evals), (3) creating execution plans across multiple agents, (4) coordinating the RAG lifecycle from setup through production, (5) recommending improvements after eval results. This is the orchestrator skill — it coordinates work across weave-setup, weave-ingest, weave-search, weave-eval, and weave-stack."
homepage: https://github.com/Maximilien-ai/weave-cli
tags: ["rag", "data", "weave", "vector-database"]
metadata:
  {
    "openclaw":
      {
        "emoji": "📋",
        "requires": { "bins": ["weave"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "Maximilien-ai/tap/weave-cli",
              "bins": ["weave"],
              "label": "Install weave-cli (brew)"
            },
            {
              "id": "go",
              "kind": "go",
              "package": "github.com/Maximilien-ai/weave-cli",
              "bins": ["weave"],
              "label": "Install weave-cli (go install)"
            }
          ]
      }
  }
---

# weave-planner

Plan and orchestrate end-to-end RAG solutions — from requirements gathering through production deployment.

## Role

The planner is the **orchestrator** for the RAG lifecycle. It coordinates work across the other weave skills:

```
weave-planner (you are here)
  ├── weave-setup   — environment configuration
  ├── weave-ingest  — data loading (parallelizable)
  ├── weave-search  — query and agent tuning (parallelizable)
  ├── weave-eval    — quality measurement (parallelizable)
  └── weave-stack   — infrastructure deployment
```

## When to Use

- User wants to build a RAG solution but doesn't know where to start
- Need to design architecture: which VDB, embeddings, agents, evals
- Coordinating multiple agents working in parallel
- Recommending next steps after eval results
- Managing the iteration loop (eval → tune → re-eval)

## RAG Lifecycle Phases

### Phase 1: Requirements Gathering

Ask the user:

1. **Data:** What documents? Format (PDF, text, images)? Volume (100s, 1000s, millions)?
2. **Queries:** What will users ask? Factual lookup? Synthesis? Summarization?
3. **Quality:** What accuracy is needed? Is citation important?
4. **Scale:** Dev/test only? Production? Expected QPS?
5. **Budget:** Cloud VDB ok? OpenAI embeddings ok? Or need OSS/local?
6. **Interface:** CLI? Dashboard? API? MCP?
7. **Existing infra:** Already using a database? Cloud provider preference?

### Phase 2: Architecture Design

Based on requirements, decide:

| Decision | Options | Default |
|----------|---------|---------|
| VDB | 10 options (see weave-setup) | Weaviate Cloud (prototyping), Milvus (production) |
| Embeddings | OpenAI, sentence-transformers, Ollama | text-embedding-3-small |
| Agent type | RAG, QA, summarize, custom | RAG for general, QA for precise |
| Chunking | semantic, fixed, paragraph | semantic, 500 tokens, 50 overlap |
| Stack | local (kind), cloud (eks/gke), podman | kind for dev |
| Eval strategy | LLM judge, human, hybrid | LLM judge (automated) |

### Phase 3: Execution Plan

Create a phased plan that maximizes parallelism:

```
Sequential: Setup (1 agent)
  └── weave-setup: install, config, doctor, VDB verify

Parallel Fork (after setup):
  ├── Data Team (N agents): weave-ingest
  │   ├── Agent 1: Collection A (PDFs batch 1)
  │   ├── Agent 2: Collection A (PDFs batch 2)
  │   └── Agent 3: Collection B (images)
  │
  └── Search Team (N agents): weave-search
      ├── Agent 4: Create RAG agent, test queries
      └── Agent 5: Create QA agent, test queries

After Ingest Complete:
  ├── Eval Team (N agents): weave-eval
  │   ├── Agent 6: Baseline eval
  │   ├── Agent 7: Multimodal eval
  │   └── Agent 8: Image search eval
  │
  └── Backup: weave backup create

Decision Point: Review eval results with user
  ├── Iterate: Adjust and re-run (back to ingest/search)
  └── Ship: Deploy to production

Sequential: Deploy (1 agent)
  └── weave-stack: init, validate, up, dashboard
```

### Phase 4: Iteration Loop

After eval results come back:

1. Review scores against thresholds (accuracy >= 0.70, citation >= 0.75)
2. If **below threshold**, recommend changes:
   - Low relevance → Try hybrid search, different embedding model
   - Low accuracy → Switch to QA agent, lower temperature
   - Low citation → Enable strict mode, must_cite
   - Inconsistent → Adjust chunking strategy
3. Coordinate re-ingestion or re-embedding if needed
4. Re-run evals to measure improvement
5. Repeat until user is satisfied

### Phase 5: Production

1. Create production stack config (weave-stack.yaml)
2. Deploy: `weave stack up --runtime eks` (or gke, kind)
3. Full data ingestion with backup
4. Final eval run on production
5. Set up day-2 ops: monitoring, backup schedule, scaling plan

### Phase 6: Day-2 Operations

Plan for ongoing operations:
- **Updates:** Schedule regular re-ingestion for new documents
- **Scaling:** Monitor QPS, scale VDB nodes as needed
- **Security:** Rotate API keys, patch VDB versions
- **Backup:** Automated backup schedule
- **Monitoring:** Prometheus metrics, health checks

## ClawMax Team Mapping

When deploying as a ClawMax template:

| Role | Skills | Count | Group |
|------|--------|-------|-------|
| Planner | weave-planner, weave-setup | 1 | Planning |
| Data Engineer | weave-ingest | 1-N | Data |
| Search Engineer | weave-search | 1-N | Search & QA |
| Eval Engineer | weave-eval | 1-N | Quality |
| Ops Engineer | weave-stack | 1 | Ops |

Workflows coordinate the phases. The planner monitors progress and triggers transitions between phases.

## Quick Start Plan (Minimal MVP)

For the fastest path to a working RAG solution:

1. `weave config create --env` (2 min)
2. `weave doctor --fix` (1 min)
3. `weave collection create MyDocs` (30 sec)
4. `weave pipeline ingest ./data/ --collection MyDocs --workers 4` (varies)
5. `weave agents create my-rag --type rag` (30 sec)
6. `weave query MyDocs "test question"` (instant)
7. `weave eval run --agent my-rag --dataset baseline` (2 min)
8. Review results, iterate if needed

Total: ~10 minutes + ingestion time for a basic working RAG.
