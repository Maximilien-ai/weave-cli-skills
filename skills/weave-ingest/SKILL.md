---
name: weave-ingest
description: "Ingest data into vector databases using weave-cli. Use when: (1) creating collections and schemas, (2) ingesting documents (PDF, text, images), (3) batch pipeline ingestion with workers, (4) getting AI schema/chunking suggestions, (5) backing up and restoring collections. NOT for: initial setup (use weave-setup), querying data (use weave-search), evaluations (use weave-eval), or deployment (use weave-stack)."
homepage: https://github.com/maximilien/weave-cli
tags: ["rag", "data", "weave", "vector-database"]
metadata:
  {
    "openclaw":
      {
        "emoji": "🗄️",
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

# weave-ingest

Ingest documents into vector databases — create collections, manage schemas, run batch pipelines, and backup data.

## When to Use

- Creating collections with schemas
- Ingesting PDFs, text files, images into VDBs
- Running batch ingestion pipelines with parallel workers
- Getting AI-powered schema and chunking recommendations
- Backing up and restoring collection data

## When NOT to Use

- Initial setup and config -> `weave-setup` skill
- Querying and searching -> `weave-search` skill
- Evaluations -> `weave-eval` skill
- Stack deployment -> `weave-stack` skill

## Collections

### Create

```bash
weave collection create MyDocs
weave collection create MyDocs --schema schema.json
weave collection create MyDocs --weaviate-cloud
```

### Manage

```bash
weave collection list
weave collection show MyDocs
weave collection count MyDocs
weave collection delete MyDocs
weave collection delete-all          # Dangerous! All collections
weave collection delete-schema MyDocs
```

## AI-Powered Suggestions

### Schema Suggest

```bash
# Analyze files and recommend schema
weave schema suggest ./data/
weave schema suggest ./data/sample.pdf
```

### Chunking Suggest

```bash
# Recommend chunk size and strategy based on data
weave chunking suggest ./data/sample.pdf
```

## Document Ingestion

### Single Documents

```bash
weave document create MyDocs ./data/file.pdf
weave document create MyDocs ./data/file.txt
weave document create ProductImages ./images/photo.jpg \
  --image-storage s3 --s3-bucket my-bucket
```

### Batch Documents

```bash
# From a batch file (JSON/YAML list of documents)
weave document batch ./batch-manifest.json
```

### Pipeline Ingestion (Recommended for Large Datasets)

```bash
# Basic pipeline
weave pipeline ingest ./data/ --collection MyDocs

# With glob patterns and parallelism
weave pipeline ingest ./data/ \
  --collection MyDocs \
  --glob "**/*.pdf" \
  --exclude "**/draft-*" \
  --batch-size 100 \
  --workers 4 \
  --recursive

# Dry run first
weave pipeline ingest ./data/ \
  --collection MyDocs \
  --glob "**/*.pdf" \
  --dry-run

# Resume interrupted ingestion
weave pipeline ingest ./data/ \
  --collection MyDocs \
  --resume

# JSON output for automation
weave pipeline ingest ./data/ \
  --collection MyDocs \
  --output json
```

### Document Management

```bash
weave document list MyDocs --limit 50
weave document show MyDocs doc-id-123
weave document count MyDocs
weave document delete MyDocs doc-id-123
weave document delete-all MyDocs
weave document update MyDocs doc-id-123
```

### PDF & Image Processing

```bash
# Convert PDF for inspection
weave document pdf-convert ./file.pdf

# Inspect document metadata
weave document inspect ./file.pdf
```

## Embedding Strategies

```bash
# List available embedding models
weave embeddings list
```

| Provider | Model | Dimensions | Cost |
|----------|-------|-----------|------|
| OpenAI | text-embedding-3-small | 1536 | $0.02/1M tokens |
| OpenAI | text-embedding-3-large | 3072 | Higher |
| OSS | sentence-transformers/all-mpnet-base-v2 | 768 | Free |
| OSS | sentence-transformers/all-MiniLM-L6-v2 | 384 | Free |
| Ollama | nomic-embed-text | 768 | Free (local) |
| Ollama | mxbai-embed-large | 1024 | Free (local) |

Use OSS embeddings for 90%+ cost savings:
```bash
weave document create MyDocs data.pdf \
  --embedding sentence-transformers/all-mpnet-base-v2
```

## Backup & Restore

```bash
# Create backup
weave backup create MyDocs --output backup.weavebak --compress

# Validate backup integrity
weave backup validate backup.weavebak

# Restore from backup
weave backup restore backup.weavebak --collection RestoredDocs

# List backups in directory
weave backup list ./backups/
```

Backup preserves: embeddings, metadata, images. Compressed ~50-60MB for 2600+ docs.

## Chunking Strategies

Configure in config.yaml:
```yaml
chunking:
  strategy: semantic    # semantic | fixed | paragraph
  tokens: 500           # chunk size
  overlap: 50           # overlap between chunks
```

## Parallel Ingestion Pattern (for ClawMax multiagent)

Split large datasets across N agents:
```bash
# Agent 1: PDFs A-M
weave pipeline ingest ./data/ --collection MyDocs --glob "**/[A-M]*.pdf" --workers 4

# Agent 2: PDFs N-Z
weave pipeline ingest ./data/ --collection MyDocs --glob "**/[N-Z]*.pdf" --workers 4
```

## Decision Workflow

1. Get AI schema suggestion: `weave schema suggest ./data/`
2. Get chunking recommendation: `weave chunking suggest ./data/sample.pdf`
3. Create collection with recommended schema
4. Dry-run pipeline: `weave pipeline ingest ... --dry-run`
5. Run ingestion with workers
6. Verify: `weave collection count`, `weave document list`
7. Backup: `weave backup create`
8. Hand off to `weave-search` and `weave-eval` skills
