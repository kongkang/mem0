# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mem0 ("mem-zero") is an intelligent memory layer for AI agents. This is a monorepo containing:

- **`mem0/`** — Core Python SDK (v1.0.5, published as `mem0ai` on PyPI)
- **`mem0-ts/`** — TypeScript SDK (v2.3.0, published as `mem0ai` on npm)
- **`server/`** — FastAPI REST API server
- **`openmemory/`** — Personal memory layer with MCP server and UI
- **`embedchain/`** — Open-source RAG framework (separate project)
- **`vercel-ai-sdk/`** — Vercel AI SDK integration
- **`openclaw/`** — OpenClaw agent platform plugin
- **`docs/`** — Mintlify documentation site
- **`evaluation/`** — LOCOMO benchmark evaluation framework
- **`tests/`** — Python SDK test suite

## Common Commands

### Python SDK

```bash
# Install dev environment (uses hatch)
make install

# Run tests (default Python version)
make test

# Run tests for specific Python version
make test-py-3.9
make test-py-3.10
make test-py-3.11
make test-py-3.12

# Run a single test file
hatch run pytest tests/test_main.py

# Run a single test function
hatch run pytest tests/test_main.py::TestClassName::test_func -v

# Format code
make format       # ruff format
make sort         # isort with black profile

# Lint
make lint         # ruff check

# Build package
make build

# Activate dev shell for specific Python version
hatch shell dev_py_3_12
```

### TypeScript SDK (`mem0-ts/`)

```bash
cd mem0-ts
pnpm install
pnpm run build        # clean + prettier check + tsup
pnpm test             # jest
pnpm test -- --watch  # jest watch mode
pnpm run format       # prettier
```

### Server (Docker)

```bash
cd server
docker compose up     # starts postgres (pgvector), neo4j, and FastAPI server on :8888
```

### Docs

```bash
make docs   # runs mintlify dev in docs/
```

## Architecture

### Memory Flow

1. **Add**: Messages → LLM extracts facts → Embeddings → Stored in vector store + optional graph store
2. **Search**: Query → Embeddings → Vector similarity search → Optional reranking → Results
3. **Update/Delete**: Modify memories with SQLite-based edit history tracking

### Python SDK Structure (`mem0/`)

- **`memory/`** — Core `Memory` and `AsyncMemory` classes (main entry points for self-hosted)
- **`client/`** — `MemoryClient` for the hosted Mem0 platform API
- **`llms/`** — 22+ LLM provider implementations (OpenAI, Anthropic, Gemini, Groq, Ollama, etc.)
- **`embeddings/`** — 15+ embedding providers (OpenAI, HuggingFace, FastEmbed, etc.)
- **`vector_stores/`** — 25+ vector store backends (Qdrant default, Chroma, Pinecone, pgvector, Milvus, etc.)
- **`graphs/`** — Graph memory with Neo4j, Memgraph, Kuzu
- **`reranker/`** — Search result reranking
- **`configs/`** — Pydantic configuration models
- **`proxy/`** — OpenAI-compatible proxy layer

### TypeScript SDK Structure (`mem0-ts/src/`)

- **`client/`** — HTTP client for hosted Mem0 platform
- **`oss/`** — Open-source implementations (mirrors Python SDK capabilities)
- **`community/`** — Community integrations
- Dual export: `mem0ai` (platform client) and `mem0ai/oss` (self-hosted)

### Multi-Level Memory

Memories are scoped by `user_id`, `agent_id`, and `run_id`, supporting User, Session, and Agent-level memory.

## Code Style

- **Python**: Ruff (line-length 120), isort with black profile. Pre-commit hooks enforce both.
- **TypeScript**: Prettier. Node >= 18, pnpm as package manager.
- **Python versions**: 3.9–3.12 supported
- **Ruff excludes**: `embedchain/` and `openmemory/` directories

## Key Dependencies

- **Python core**: qdrant-client, pydantic, openai, sqlalchemy
- **Optional groups**: `graph`, `vector_stores`, `llms`, `extras` (install via `pip install mem0ai[graph,vector_stores]`)
- **TypeScript core**: axios, openai, zod, uuid

## Docker Dev Environment

The `server/docker-compose.yaml` runs:
- PostgreSQL with pgvector (port 8432)
- Neo4j with APOC plugin (ports 8474/8687)
- FastAPI server with hot-reload (port 8888)
