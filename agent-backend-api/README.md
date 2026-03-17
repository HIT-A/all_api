# agent-backend API Bundle (SSOT-aligned)

This folder is the **client-facing contract bundle** for **Server A: `agent-backend`**.

**SSOT reference (source of truth):**
- `HIT-A/agent-backend` → `docs/plans/2026-03-13-agentic-architecture-design.md`

This bundle should:
- summarize the **/v1 HTTP contract** in a copy/paste friendly way for client implementers
- stay aligned with SSOT semantics (especially: **invoke is always HTTP 200**)
- avoid embedding any private/EAS-related contracts (those stay on-device)

## Client hard rules (must follow)
- Clients should depend on `new_api/agent-backend-api/` and **must not copy SSOT全文** into app docs.
- `POST /v1/skills/{name}:invoke` **ALWAYS returns HTTP 200**.
  - failures use `{ "ok": false, "error": {"code","message","retryable"} }`
- App should call **agent-backend only**. It must not call `pr-server` directly.

## Files
- `v1.md`: readable contract summary + JSON examples
- `openapi.yaml`: machine-readable contract

## What's implemented today ✅

### Core Endpoints
- `/health` - Health check
- `/v1/skills` - List available skills
- `/v1/skills/{name}:invoke` - Invoke any skill (ALWAYS HTTP 200)
- `/v1/jobs/{id}` - Get async job status

### Public Skills (33 total)

#### Data Layer
- ✅ `rag.query` - RAG vector search (Qdrant)
- ✅ `rag.ingest` (async) - Document ingestion pipeline with concurrent processing
  - Fetches from GitHub repositories
  - Stores source files in COS (optional)
  - Chunks and embeds using ZHIPU API
  - Upserts to Qdrant with configurable workers

#### Web Crawling (NEW - Crawl4AI)
- ✅ `crawl4ai.page` - Crawl single web page with AI content filtering
  - Supports multiple output formats (markdown, HTML, text)
  - AI-powered noise removal and content extraction
  - Configurable wait conditions for dynamic content
- ✅ `crawl4ai.site` (async) - Full website crawling
  - BFS-based site traversal with configurable depth
  - Sitemap.xml support for faster discovery
  - URL pattern filtering (include/exclude)
  - Rate limiting and robots.txt compliance
  - Auto-store to COS and ingest to RAG
- ✅ `crawl4ai.status` - Check crawl job status and get results

#### File Management
- ✅ `files.upload` - Upload files from base64/URL/text to COS
- ✅ `files.download` - Download files as URL or base64 content
- ✅ `cos.save_file` - Upload file to Tencent Cloud COS
- ✅ `cos.delete_file` - Delete file from COS
- ✅ `cos.list_files` - List COS files
- ✅ `cos.get_presigned_url` - Get temporary access URL
- ✅ `cos.get_quota` - Query storage quota

#### PR Workflow
- ✅ `pr.preview` (async) - Preview course material changes with concurrent processing
- ✅ `pr.submit` (async) - Submit PR via pr-server with async execution
- ✅ `pr.lookup` - Query PR status (fast lookup, remains sync)

PR runtime updates (2026-03-17):
- `pr.preview` / `pr.submit` / `pr.lookup` now forward `PR_SERVER_TOKEN` as Bearer to pr-server.
- `pr.submit` supports `idempotency_key`; if omitted, agent-backend auto-generates one.
- `pr.submit` response mapping is aligned with pr-server `data.pr` schema.
- `pr.lookup` accepts both `number` (preferred) and `pr` (legacy) fields.

#### RAG Sync runtime updates
- ✅ `rag.sync_to_repo` supports both `repo`/`branch` aliases and `target_repo`/`target_branch`.
- Git clone/pull/push for sync uses `GITHUB_TOKEN` auth (optional `GITHUB_USERNAME`, default `x-access-token`).
- Sync disables interactive git prompts (`GIT_TERMINAL_PROMPT=0`) for reliable background jobs.
- Sync auto-configures commit identity via `GIT_AUTHOR_NAME` / `GIT_AUTHOR_EMAIL` (with defaults).

#### MCP Management
- ✅ `mcp.list_servers` - List registered MCP servers
- ✅ `mcp.list_tools` - List available MCP tools
- ✅ `mcp.call_tool` - Execute MCP tool

MCP policy update (2026-03-17):
- Dynamic MCP registration is disabled in production-facing skill surface.
- `mcp.register_server` and `mcp.unregister_server` are removed from `/v1/skills`.

#### Aggregation & Search (NEW)
- ✅ `aggregator.search` - Multi-source concurrent search (RAG + GitHub + COS)
  - Concurrently queries multiple data sources
  - Aggregates and ranks results by relevance score
  - Configurable timeout and source selection
- ✅ `aggregator.summarize` (async) - AI-powered search result summarization
  - Uses ZHIPU BigModel API for intelligent summaries
  - Multiple styles: concise, detailed, bullet_points, qa
  - Supports Chinese and English
  - Extracts key points automatically

#### Test Skills
- ✅ `echo` - Echo test
- ✅ `sleep_echo` - Async delay test

## Architecture Highlights

### Concurrent Processing
All services now support **concurrent processing** for maximum performance:

#### Async Skills (Long-running operations)
| Skill | Workers | Description |
|-------|---------|-------------|
| `rag.ingest` | Configurable (default: 4) | Concurrent file fetching, parsing, chunking, embedding |
| `pr.preview` | Job queue | Async PR preview generation |
| `pr.submit` | Job queue | Async PR submission to GitHub |
| `mcp.call_tool` | Job queue | Async external tool execution |
| `sleep_echo` | Job queue | Test async skill |

#### Batch Operations (Concurrent I/O)
| Skill | Batch Support | Workers | Description |
|-------|--------------|---------|-------------|
| `files.upload` | ✅ Yes | 4 | Concurrent multi-file upload |
| `files.download` | ✅ Yes | 4 | Concurrent multi-file download |
| `cos.save_file` | ✅ Yes | 4 | Concurrent COS uploads |
| `cos.delete_file` | ✅ Yes | 4 | Concurrent COS deletions |
| `cos.get_presigned_url` | ✅ Yes | 4 | Concurrent URL generation |

#### Concurrency Features
- **Worker pools**: Configurable goroutine pools for parallel processing
- **Atomic counters**: Thread-safe progress tracking
- **Channel-based communication**: Safe data passing between goroutines
- **Context cancellation**: Proper cleanup on timeout/cancellation
- **Error aggregation**: Collect and report all errors from concurrent operations
- **Retry logic**: Exponential backoff for transient failures

### Storage Strategy

### Storage Strategy
- **Source files**: Can be stored in COS for backup/persistence
- **Vectors**: Stored in Qdrant for semantic search
- **Embeddings**: Generated via ZHIPU BigModel API (2048-dim)

### Data Flow
```
GitHub Repository
       ↓
   (fetch) ←── Concurrent workers
       ↓
   [COS Storage] (optional backup)
       ↓
   (parse) ←── Markdown/HTML/Text
       ↓
   (chunk) ←── 1400 char chunks
       ↓
   (embed) ←── ZHIPU BigModel API
       ↓
   (upsert) ←── Qdrant vector DB
```

## Quick Start

### List all skills
```bash
curl http://agent-backend:8080/v1/skills
```

### RAG Ingest (async)
```bash
curl -X POST http://agent-backend:8080/v1/skills/rag.ingest:invoke \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "repo": "hit/hita-docs",
      "ref": "main",
      "path_prefix": "courses/COMP1011",
      "source": "COMP1011-spring-2025",
      "workers": 8,
      "store_in_cos": true,
      "cos_prefix": "rag-source"
    }
  }'
# Returns: {"job_id": "xxx"}

# Check status:
curl http://agent-backend:8080/v1/jobs/xxx
```

### Upload File
```bash
curl -X POST http://agent-backend:8080/v1/skills/files.upload:invoke \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "key": "materials/lecture1.pdf",
      "content_base64": "SGVsbG8g...",
      "content_type": "application/pdf"
    }
  }'
```

### Download File
```bash
# Get download URL
curl -X POST http://agent-backend:8080/v1/skills/files.download:invoke \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "key": "materials/lecture1.pdf",
      "format": "url",
      "expires_seconds": 3600
    }
  }'

# Or get content as base64
curl -X POST http://agent-backend:8080/v1/skills/files.download:invoke \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "key": "materials/lecture1.pdf",
      "format": "base64"
    }
  }'
```

### RAG Query
```bash
curl -X POST http://agent-backend:8080/v1/skills/rag.query:invoke \
  -H "Content-Type: application/json" \
  -d '{"input": {"query": "线性代数", "top_k": 10}}'
```

### Batch Operations (Concurrent)

#### Upload Multiple Files Concurrently
```bash
curl -X POST http://agent-backend:8080/v1/skills/files.upload:invoke \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "files": [
        {"key": "file1.pdf", "content_base64": "...", "content_type": "application/pdf"},
        {"key": "file2.jpg", "content_base64": "...", "content_type": "image/jpeg"},
        {"key": "file3.txt", "text": "Hello World", "content_type": "text/plain"}
      ]
    }
  }'
# Returns: {"success": true, "total": 3, "success_count": 3, "fail_count": 0, "results": [...]}
```

#### Download Multiple Files Concurrently
```bash
curl -X POST http://agent-backend:8080/v1/skills/files.download:invoke \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "keys": ["file1.pdf", "file2.jpg", "file3.txt"],
      "format": "url",
      "expires_seconds": 3600
    }
  }'
```

#### Batch COS Upload
```bash
curl -X POST http://agent-backend:8080/v1/skills/cos.save_file:invoke \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "files": [
        {"key": "backup/file1.txt", "content_base64": "..."},
        {"key": "backup/file2.txt", "content_base64": "..."}
      ]
    }
  }'
```

#### Batch COS Delete
```bash
curl -X POST http://agent-backend:8080/v1/skills/cos.delete_file:invoke \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "keys": ["old/file1.txt", "old/file2.txt", "old/file3.txt"]
    }
  }'
```

### Web Crawling (Crawl4AI)

#### Crawl Single Page
```bash
curl -X POST http://agent-backend:8080/v1/skills/crawl4ai.page:invoke \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "url": "https://example.com/article",
      "content_filter": true,
      "output_format": "markdown"
    }
  }'
# Returns: {"url": "...", "title": "...", "content": "...", "word_count": 1200}
```

#### Full Site Crawl (async)
```bash
curl -X POST http://agent-backend:8080/v1/skills/crawl4ai.site:invoke \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "start_url": "https://docs.python.org/3",
      "max_pages": 100,
      "max_depth": 3,
      "include_patterns": ["/3/tutorial/", "/3/library/"],
      "exclude_patterns": ["/3/whatsnew/"],
      "use_sitemap": true,
      "store_in_cos": true,
      "auto_ingest_to_rag": true,
      "rag_source": "python-docs"
    }
  }'
# Returns: {"crawl_id": "xxx", "status": "started"}

# Check status:
curl http://agent-backend:8080/v1/jobs/xxx
# Returns: {"status": "completed", "pages_crawled": 50, "results": [...]}
```

### Aggregation & Search

#### Multi-Source Search
```bash
curl -X POST http://agent-backend:8080/v1/skills/aggregator.search:invoke \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "query": "machine learning algorithms",
      "sources": ["rag", "cos", "github"],
      "top_k": 10,
      "timeout_seconds": 15
    }
  }'
# Returns: {"query": "...", "total": 10, "results": [...], "sources_count": {"rag": 5, "cos": 3, "github": 2}}
```

#### AI Summarization (async)
```bash
# First, get search results, then summarize
curl -X POST http://agent-backend:8080/v1/skills/aggregator.summarize:invoke \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "query": "machine learning algorithms",
      "results": [
        {"source": "rag", "title": "ML Basics", "content": "...", "score": 0.95},
        {"source": "cos", "title": "lecture1.pdf", "content": "...", "score": 0.88}
      ],
      "style": "bullet_points",
      "max_length": 300,
      "language": "zh"
    }
  }'
# Returns: {"job_id": "xxx"}

# Check status:
curl http://agent-backend:8080/v1/jobs/xxx
# Returns: {"summary": "...", "key_points": [...], "confidence": 0.92}
```

## Environment Configuration

### Required for RAG Ingest
```bash
# Qdrant
QDRANT_URL=https://qdrant.example.com
QDRANT_COLLECTION=documents
QDRANT_API_KEY=your-api-key

# ZHIPU BigModel (for embeddings)
EMBEDDING_PROVIDER=bigmodel
BIGMODEL_API_KEY=your-api-key
BIGMODEL_EMBEDDING_MODEL=embedding-3
BIGMODEL_DIMENSIONS=2048

# GitHub
GITHUB_TOKEN=your-token
GITHUB_API_BASE_URL=https://api.github.com

# COS (optional, for source file storage)
COS_SECRET_ID=your-secret-id
COS_SECRET_KEY=your-secret-key
COS_REGION=ap-guangzhou
COS_BUCKET=hita-courses
```

### Required for Crawl4AI

**Option 1: Docker (Recommended)**
```bash
# Build Crawl4AI MCP Server
cd mcp-servers/crawl4ai
docker build -t crawl4ai-mcp .

# Register MCP server
export MCP_SERVERS='[
  {
    "name": "crawl4ai",
    "transport": "stdio",
    "command": ["docker", "run", "--rm", "-i", "crawl4ai-mcp"],
    "enabled": true
  }
]'
```

**Option 2: Local Installation**
```bash
# Install Crawl4AI
pip install crawl4ai mcp
playwright install chromium

# Register MCP server
export MCP_SERVERS='[
  {
    "name": "crawl4ai",
    "transport": "stdio",
    "command": ["python", "mcp-servers/crawl4ai/server.py"],
    "enabled": true
  }
]'
```

### Required for AI Summarization
```bash
# ZHIPU BigModel (for AI summaries)
BIGMODEL_API_KEY=your-api-key
BIGMODEL_SUMMARIZE_MODEL=glm-4  # Optional, defaults to glm-4
```

## Skill Categories

| Category | Skills | Purpose |
|----------|--------|---------|
| **Data** | `rag.query`, `rag.ingest` | Vector search & ingestion |
| **Files** | `files.upload`, `files.download`, `cos.*` | File storage & retrieval |
| **Crawling** | `crawl4ai.page`, `crawl4ai.site`, `crawl4ai.status` | Web crawling & scraping |
| **Aggregation** | `aggregator.search`, `aggregator.summarize` | Multi-source search & AI summaries |
| **PR** | `pr.preview`, `pr.submit`, `pr.lookup` | Course material workflows |
| **MCP** | `mcp.*` | External tool integration |
| **Test** | `echo`, `sleep_echo` | Development testing |

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    端侧 (HITA_L/HITA_X)               │
│  - 教务登录 (私有技能)                                        │
│  - 本地 RAG (文件索引)                                       │
│  - 缓存管理                                                 │
└────────────────────┬────────────────────────────────────────┘
                     │ HTTP POST /v1/skills/{name}:invoke
                     ▼
┌─────────────────────────────────────────────────────────────┐
│            agent-backend (云端公共技能)              │
│                                                             │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐   │
│  │  RAG Skills   │  │  File Skills  │  │  PR Skills    │   │
│  │  rag.query    │  │  files.*      │  │  pr.preview   │   │
│  │  rag.ingest   │  │  cos.*        │  │  pr.submit    │   │
│  │  (concurrent) │  │               │  │  pr.lookup    │   │
│  └───────┬───────┘  └───────┬───────┘  └───────┬───────┘   │
│          │                  │                  │           │
│          ▼                  ▼                  ▼           │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐   │
│  │  Qdrant       │  │  Tencent COS  │  │  pr-server    │   │
│  │  (vectors)    │  │  (objects)    │  │  (GitHub)     │   │
│  └───────────────┘  └───────────────┘  └───────────────┘   │
│                                                             │
│  ┌───────────────┐  ┌───────────────┐                      │
│  │  ZHIPU API    │  │  MCP Skills   │                      │
│  │  (embeddings) │  │  mcp.*        │                      │
│  └───────────────┘  └───────────────┘                      │
└─────────────────────────────────────────────────────────────┘
```

## Recent Updates

### 2025-03-15 - RAG Ingest + Files Skills
- ✅ Added `rag.ingest` with concurrent processing
- ✅ Added `files.upload` for multi-source uploads (base64/URL/text)
- ✅ Added `files.download` for flexible downloads (URL/base64)
- ✅ ZHIPU BigModel integration for high-quality embeddings
- ✅ COS integration for source file storage

### 2025-03-14 - COS Integration
- Added Tencent Cloud COS storage support
- 5 new COS skills for file management
- Supports large files (PDF, PPT, materials)

### 2025-03-13 - MCP Support
- Added Model Context Protocol (MCP) integration
- Support for external MCP servers (Playwright, Filesystem, etc.)

See `v1.md` for detailed examples of all skills.