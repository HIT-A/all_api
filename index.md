# API Bundles Index

This folder is the **shared API documentation workspace** (human-readable norms + JSON examples + OpenAPI) for:

## Project Architecture (Updated 2025-03-16)

This project targets HIT students across three campuses with a **privacy-first, agentic** architecture:
- **On-device Orchestrator (HITA)** handles all private skills (login/session, EAS data fetch, local memory/cache).
- **Server Public Skills (agent-backend)** provides shared capabilities (RAG, PR workflows, public search, COS storage) via a unified Skills protocol.
- **Privacy-by-architecture:** no credentials/cookies/tokens are sent to the server.

Core flow:
1. Client performs private EAS actions on device.
2. Client calls agent-backend for public skills only.
3. Server executes async jobs with traceable, redacted observability.

## Recent Updates

### 2025-03-16 - Skills Expansion ✅
- Added RAG ingestion skills (rag.ingest, rag.ingest_from_github, rag.sync_to_repo)
- Added file management skills (files.upload, files.download)
- Added web crawling skills (crawl4ai.page, crawl4ai.site, crawl4ai.status)
- Added GitHub batch download and document conversion
- Added AI summarization skill (aggregator.summarize)
- **32 skills total now implemented**

### 2025-03-14 - COS Integration ✅
- Added Tencent Cloud COS storage support
- 5 new COS skills for file upload/management
- Supports large files (PDF, PPT, materials)
- Integrated with PR workflow

### 2025-03-13 - MCP Support ✅
- Added Model Context Protocol (MCP) integration
- Support for external MCP servers (Playwright, Filesystem, etc.)
- Dynamic tool discovery and execution
- 5 MCP management skills

### 2025-03-12 - PR Skills ✅
- Added pr.preview, pr.submit, pr.lookup
- Integrated with pr-server
- Support for multi-campus (Shenzhen/Harbin/Weihai)

## Responsibilities Summary
- Client app calls **agent-backend only** for public skills.
- agent-backend exposes **SSOT skills protocol**:
  - `GET /v1/skills`
  - `POST /v1/skills/{name}:invoke` (ALWAYS HTTP 200; failures are ok=false)
  - `GET /v1/jobs/{id}`
- pr-server exposes **internal REST** and never serves mobile/web directly.
- Campus APIs are **capture-based references** for on-device adapters (not server contracts).

## Bundles

### `agent-backend-api/`
**Client-facing API for public skills**
- `README.md` (rules + examples + skill catalog)
- `v1.md` (readable contract summary with all 32 skills)
- `openapi.yaml` (machine-readable contract)
- **Current Status:** 31 skills implemented, RAG ingestion + Files + Crawl newly added

### `pr-Server-api/`
**Internal REST API for GitHub operations**
- `README.md` (internal REST + adapter mapping)
- `openapi.yaml` (internal REST)
- **Note:** Not client-facing; agent-backend adapts these to skills

### `school-api/`
**Campus API captures for on-device adapters**
- `shenzhen.md` (HITSZ captures)
- `main.md` (HIT Harbin captures; login + portal + teaching system)
- `weihai.md` (HITWH captures; status: deferred)

### `bot-backend-api/`
**AstrBot 插件 - 课程助手**
- `README.md` (安装 + 配置 + 命令列表)
- **Status:** 课程查询、RAG 问答、PR 提交

### `course-templates/`
**课程仓库模板 (新 schema)**
- `normal/` - 单一课程模板
- `multi-project/` - 多课程仓库模板

## Skill Catalog

### ✅ Implemented (32 skills)

| Category | Skills | Count |
|----------|--------|-------|
| **RAG** | rag.query, rag.ingest, rag.ingest_from_github, rag.sync_to_repo | 4 |
| **PR** | pr.preview, pr.submit, pr.lookup | 3 |
| **Data** | data.ingest, data.ingest_batch | 2 |
| **Search** | search, courses.search, course.read | 3 |
| **Aggregation** | aggregator.summarize | 1 |
| **HIT** | hit.teacher, hit.teachers | 2 |
| **Crawl** | crawl4ai.page, crawl4ai.site, crawl4ai.status | 3 |
| **GitHub** | github.batch_download, document.convert | 2 |
| **COS** | cos.save_file, cos.delete_file, cos.list_files, cos.get_presigned_url, cos.get_quota | 5 |
| **Files** | files.upload, files.download | 2 |
| **MCP** | mcp.list_servers, mcp.list_tools, mcp.call_tool, mcp.register_server, mcp.unregister_server | 5 |
| **Test** | echo, sleep_echo | 2 |
| **Total** | | **32** |

### 🔄 Planned

See `FUTURE_ROADMAP.md` for detailed planning:

**Phase 1 (Q1 2025):**
- ✅ rag.ingest - Document ingestion (已完成)
- pr.list - List course PRs

**Phase 2 (Q2 2025):**
- ✅ files.upload - Local file upload (已完成)
- github.read_file - GitHub file access

**Phase 3 (Q3 2025):**
- ✅ search - Multi-source search (已完成)
- ✅ courses.search - Public course search (已完成)

**Phase 4 (Q4 2025+):**
- AI/ML features
- Advanced analytics

## SSOT Reference

- Protocol SSOT lives in `HIT-A/agent-backend/docs/plans/2026-03-13-agentic-architecture-design.md`

## Quick Links

- [API Roadmap](FUTURE_ROADMAP.md) - Future features and planning
- [agent-backend Skills](agent-backend-api/v1.md) - All 32 skills with examples
- [pr-server Internal](pr-Server-api/README.md) - Internal REST API docs

---

**Maintained by:** HITA Development Team
**Last Updated:** 2025-03-16
