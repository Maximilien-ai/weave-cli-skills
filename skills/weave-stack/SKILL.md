---
name: weave-stack
description: "Deploy and operate RAG infrastructure using weave-cli stacks. Use when: (1) initializing weave-stack.yaml for k8s or podman, (2) deploying VDB + ingestion + dashboard, (3) monitoring stack health, (4) day-2 operations (backup, scaling, security), (5) managing the weave dashboard UI. NOT for: initial setup (use weave-setup), data ingestion (use weave-ingest), querying (use weave-search), or evaluations (use weave-eval)."
homepage: https://github.com/maximilien/weave-cli
tags: ["rag", "data", "weave", "vector-database"]
metadata:
  {
    "openclaw":
      {
        "emoji": "📦",
        "requires": { "bins": ["weave", "kubectl"] },
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

# weave-stack

Deploy and operate complete RAG infrastructure — VDB, ingestion pipeline, dashboard, and day-2 ops.

## When to Use

- Initializing stack configuration (weave-stack.yaml)
- Deploying RAG infrastructure to k8s or podman
- Monitoring stack health and logs
- Day-2 operations: backup, scaling, security patching
- Managing the weave dashboard UI

## When NOT to Use

- Initial weave-cli setup -> `weave-setup` skill
- Data ingestion commands -> `weave-ingest` skill
- Search and querying -> `weave-search` skill
- Evaluations -> `weave-eval` skill

## Stack Lifecycle

### Initialize

```bash
weave stack init                     # Creates weave-stack.yaml
weave stack validate                 # Validate configuration
```

### weave-stack.yaml Format

```yaml
runtime:
  type: kubernetes
  provider: kind              # kind | minikube | eks | gke
  nodes: 1
  container_runtime: podman   # podman | docker

vectordb:
  type: milvus
  version: "2.3.0"
  resources:
    requests: { memory: "2Gi" }
    limits: { memory: "4Gi" }

llm:
  provider: openai
  embedding_model: text-embedding-3-small
  chat_model: gpt-4o

image_storage:
  type: minio
  endpoint: minio:9000
  bucket: weave-images

ingestion:
  workers: 2
  batch_size: 10
  retry:
    strategy: exponential
    max_retries: 3
  checkpoint: true
```

### Deploy

```bash
# Local Kubernetes (Kind)
weave stack up --runtime kind

# Minikube
weave stack up --runtime minikube

# Cloud
weave stack up --runtime eks      # AWS EKS
weave stack up --runtime gke      # Google GKE
```

### Monitor

```bash
weave stack status                   # Component health
weave stack logs                     # All logs
weave stack logs milvus              # Specific component
weave stack logs --follow            # Stream logs
```

### Stack Operations

```bash
# Ingest data in deployed stack
weave stack ingest ./data/ --collection StackDocs

# Manage collections in stack
weave stack collections list
weave stack collections create NewCollection

# Backup stack data
weave stack backup

# Direct kubectl access
weave stack kubectl get pods
weave stack kubectl describe pod milvus-0

# Port forwarding
weave stack port-forward
```

### Dashboard

```bash
weave stack dashboard                # Open web dashboard
```

Dashboard features: search interface, collection browser, health monitoring, agent queries.

### Shutdown

```bash
weave stack down                     # Stop all services
```

## Local Development Stacks

### Docker Compose (Milvus)

```yaml
# local/milvus/docker-compose.yml
services:
  milvus:
    image: milvusdb/milvus:v2.4.0-standalone
    ports:
      - "19530:19530"
      - "9091:9091"
    deploy:
      resources:
        limits:
          memory: 12G
```

```bash
cd local/milvus && docker compose up -d
```

### Docker Compose (MinIO for images)

```bash
cd docker && docker compose -f docker-compose.minio.yml up -d
# API: localhost:9000, Console: localhost:9001
```

## Day-2 Operations

### Regular Updates
```bash
weave stack status                   # Check current state
weave stack backup                   # Backup before changes
# Update weave-stack.yaml version numbers
weave stack up                       # Rolling update
```

### Scaling
```bash
# Edit weave-stack.yaml
# Increase nodes, workers, memory limits
weave stack validate
weave stack up                       # Apply changes
```

### Security
```bash
# Rotate credentials in .env
weave config update --env
weave stack up                       # Redeploy with new creds
```

### Backup & Recovery
```bash
weave stack backup                   # Backup all collections
weave backup validate backup.weavebak
# In case of failure:
weave backup restore backup.weavebak
```

## Metrics & Monitoring

```bash
# Start metrics server
weave serve --metrics-port 9090

# Prometheus endpoint
curl http://localhost:9090/metrics
```

## MCP Integration

```bash
weave mcp list                       # Available tools
weave mcp call tool-name             # Call MCP tool
weave mcp test                       # Test connection
weave mcp list --server http://localhost:8030
```

## Decision Workflow

1. Choose runtime: `kind` for local dev, `eks`/`gke` for production
2. Initialize: `weave stack init`
3. Configure VDB, LLM, storage in weave-stack.yaml
4. Validate: `weave stack validate`
5. Deploy: `weave stack up --runtime kind`
6. Verify: `weave stack status`
7. Ingest data: `weave stack ingest`
8. Open dashboard: `weave stack dashboard`
9. Set up monitoring and backups for production
