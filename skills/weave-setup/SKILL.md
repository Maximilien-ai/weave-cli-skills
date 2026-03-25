---
name: weave-setup
description: "Set up and configure weave-cli for vector database RAG solutions. Use when: (1) installing or updating weave-cli, (2) creating or updating config.yaml and .env files, (3) running weave doctor diagnostics, (4) selecting and configuring vector databases, (5) verifying health of VDB connections. NOT for: data ingestion (use weave-ingest), querying (use weave-search), evaluations (use weave-eval), or stack deployment (use weave-stack)."
homepage: https://github.com/maximilien/weave-cli
tags: ["rag", "data", "weave", "vector-database"]
metadata:
  {
    "openclaw":
      {
        "emoji": "⚙️",
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

# weave-setup

Set up and configure weave-cli for building multimodal RAG solutions across 10 vector databases.

## When to Use

- Installing weave-cli for the first time
- Creating or updating `config.yaml` and `.env` for a project
- Running diagnostics to verify everything works
- Selecting which VDB(s) to use based on project requirements
- Checking VDB connectivity and health

## When NOT to Use

- Data ingestion, collections, documents -> `weave-ingest` skill
- Searching, querying, agents -> `weave-search` skill
- Evaluations and benchmarks -> `weave-eval` skill
- Stack deployment (k8s/podman) -> `weave-stack` skill
- End-to-end planning -> `weave-planner` skill

## Installation

```bash
# Clone and build from source
git clone https://github.com/maximilien/weave-cli.git
cd weave-cli && ./build.sh
# Binary at: bin/weave

# Verify
weave --version
weave doctor
```

## Configuration

### Interactive Setup

```bash
weave config create --env       # Create config.yaml + .env interactively
weave config update --env       # Update existing configuration
```

### Manual config.yaml

```yaml
quiet-config: true
logging:
  level: info

databases:
  default: weaviate-cloud
  vector_databases:
    - name: weaviate-cloud
      type: weaviate-cloud
      url: ${WEAVIATE_URL}
      api_key: ${WEAVIATE_API_KEY}
      openai_api_key: ${OPENAI_API_KEY}
      timeout: 10
      collections:
        - name: MyDocs
          type: text
          description: Main document collection
```

### .env File

```bash
OPENAI_API_KEY=sk-proj-...
WEAVIATE_URL=https://your-cluster.weaviate.cloud
WEAVIATE_API_KEY=your-key
# Add credentials for whichever VDBs you configure
```

## Diagnostics

```bash
weave doctor                     # Full system diagnostic
weave doctor --fix               # With auto-fix suggestions
weave doctor --section config    # Check specific section
weave doctor --section vdb       # VDB connectivity only
weave doctor --verbose           # Show passing checks too
weave doctor --json              # Machine-readable output
```

Doctor sections: `system`, `config`, `env`, `vdb`, `llm`, `embeddings`, `stack`, `opik`

## Health & VDB Info

```bash
weave health check               # Quick VDB health
weave health check --weaviate-cloud
weave vdb list                   # List all configured databases
weave vdb info weaviate-cloud    # Details on specific VDB
```

## Config Management

```bash
weave config show                # Display current config
weave config list                # List configured databases
weave config sync                # Sync to ~/.weave-cli
weave config fix --errors-only   # Auto-fix issues
weave config show-schema         # Show config schema
```

## Supported Vector Databases (10)

| VDB | Local | Cloud | Best For |
|-----|-------|-------|----------|
| Weaviate | weaviate-local | weaviate-cloud | Production, all features |
| Qdrant | qdrant-local | qdrant-cloud | Rust performance, filtering |
| Milvus | milvus-local | milvus-cloud | Horizontal scaling |
| Chroma | chroma-local | chroma-cloud | Simple setup (macOS) |
| Supabase | supabase-local | supabase-cloud | PostgreSQL users |
| Neo4j | neo4j-local | neo4j-cloud | Graph + vector hybrid |
| MongoDB | mongodb-local | mongodb-cloud | Atlas Vector Search |
| Pinecone | — | pinecone | Auto-scaling managed |
| Elasticsearch | elasticsearch-local | elasticsearch-cloud | Hybrid search |
| OpenSearch | opensearch-local | opensearch-cloud | AWS, k-NN + BM25 |

### VDB Selection Guide

- **Quick prototyping:** Weaviate Cloud (free tier) or Chroma local
- **Production:** Weaviate, Qdrant, or Milvus
- **Existing infra:** Match your current DB
- **Graph + vector:** Neo4j
- **AWS native:** OpenSearch

### VDB Override Flags

```bash
weave --weaviate-cloud <command>
weave --milvus-local <command>
weave --qdrant-cloud <command>
weave --all <command>          # All configured VDBs
weave --mock <command>         # Mock for testing
```

## Global Flags

```bash
--config FILE       # Config path (default: ./config.yaml)
--env FILE          # Env path (default: ./.env)
-v, --verbose       # Verbose output
-q, --quiet         # Minimal messages
--json              # JSON output
--no-confirm        # Skip confirmations
--timeout DURATION  # e.g., 5s, 30s
```

## Decision Workflow

1. Check prerequisites: `weave --version`, `weave doctor`
2. Understand goals: What data? What queries? What scale?
3. Recommend VDB based on goals, infra, budget
4. Create config: `weave config create --env`
5. Verify: `weave health check`, `weave doctor --fix`
6. Hand off to `weave-ingest` skill for data loading
