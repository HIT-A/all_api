# Future Roadmap - HITA API Development

## Current Status (2025-03-15) ✅ ALL CONCURRENT

### ✅ Implemented (25 Skills) - Full Concurrency Support
- **Data:** rag.query, rag.ingest (async + concurrent workers)
- **Files:** files.upload, files.download (batch + concurrent I/O)
- **COS:** cos.save_file, cos.delete_file, cos.list_files, cos.get_presigned_url, cos.get_quota (batch operations)
- **Crawling:** crawl4ai.page, crawl4ai.site (async), crawl4ai.status (NEW)
- **Aggregation:** aggregator.search, aggregator.summarize (async)
- **PR:** pr.preview (async), pr.submit (async), pr.lookup (sync)
- **MCP:** mcp.list_servers, mcp.list_tools, mcp.call_tool (async)
- **Test:** echo, sleep_echo (async)

---

## Phase 1 - Enhanced RAG ✅ COMPLETED (2025-03-15)

### ✅ Implemented
- [x] `rag.ingest` - Document ingestion pipeline with concurrent processing
  - ✅ Fetches from GitHub repositories
  - ✅ Parses Markdown/HTML/Text
  - ✅ Chunks text (1400 chars) with paragraph awareness
  - ✅ Embeds using ZHIPU BigModel API (2048-dim)
  - ✅ Stores source files in COS (optional)
  - ✅ Upserts to Qdrant with configurable worker pool
  - ✅ Concurrent processing (default 4 workers, configurable)
  - ✅ Retry logic with exponential backoff
  - **Status:** ✅ COMPLETE

### 📋 Planned
- [ ] `rag.delete` - Remove documents from index
- [ ] `rag.update` - Update existing documents
- [ ] `rag.search_advanced` - Advanced search with filters

---

## Phase 2 - PR Workflow Enhancement (Q1-Q2 2025)

### 📋 Planned
- [ ] `pr.list` - List all PRs for a course
- [ ] `pr.batch_submit` - Batch create PRs for multiple courses
- [ ] `pr.merge` - Auto-merge approved PRs
- [ ] `pr.comment` - Add comments to PRs
- [ ] `pr.close` - Close PRs without merging

### 🔮 Future Ideas
- [ ] `pr.sync` - Sync fork with upstream
- [ ] `pr.conflict_check` - Check for merge conflicts

---

## Phase 3 - File Management ✅ COMPLETED (2025-03-15)

### ✅ Implemented
- [x] `files.upload` - Multi-source file upload
  - ✅ Upload from base64 content
  - ✅ Upload from URL (download and store)
  - ✅ Upload from text content
  - ✅ Configurable content type
- [x] `files.download` - Flexible file download
  - ✅ Get presigned URL for downloads
  - ✅ Download and return base64 content
  - ✅ Configurable expiration

### 📋 Planned
- [ ] `files.stream` - Streaming upload for large files
- [ ] `files.preview` - Generate file previews (PDF, images)
- [ ] `files.convert` - Convert between formats (DOC → PDF)

### 🔮 Future Ideas
- [ ] `files.virus_scan` - Virus scanning
- [ ] `files.watermark` - Add watermarks to documents

---

## Phase 4 - GitHub Integration (Q2 2025)

### 📋 Planned
- [ ] `github.read_file` - Read files from GitHub repos
- [ ] `github.list_repos` - List organization repositories
- [ ] `github.search_code` - Search code across repos
- [ ] `github.get_repo_info` - Get repository metadata
- [ ] `github.list_commits` - List commit history

### 🔮 Future Ideas
- [ ] `github.webhook` - Webhook event handling
- [ ] `github.actions` - Trigger GitHub Actions

---

## Phase 4.5 - Web Crawling (Crawl4AI) ✅ COMPLETED (2025-03-15)

### ✅ Implemented
- [x] `crawl4ai.page` - Single page crawling with AI content filtering
  - ✅ Multiple output formats (markdown, HTML, text)
  - ✅ AI-powered noise removal and content extraction
  - ✅ Configurable wait conditions for dynamic content
- [x] `crawl4ai.site` (async) - Full website crawling
  - ✅ BFS-based site traversal with configurable depth
  - ✅ Sitemap.xml support for faster discovery
  - ✅ URL pattern filtering (include/exclude regex)
  - ✅ Rate limiting and robots.txt compliance
  - ✅ Concurrent requests (configurable)
  - ✅ Auto-store to COS and ingest to RAG
- [x] `crawl4ai.status` - Crawl job status monitoring

### Architecture
- MCP Server wrapper for Crawl4AI (Python)
- Docker containerization for easy deployment
- Async job queue for long-running crawls
- Integration with RAG pipeline for automatic ingestion

---

## Phase 5 - Campus Integration (Q2-Q3 2025)

### ⚠️ Privacy Considerations
These skills require careful privacy handling. They should:
- Only access public data
- Never store credentials
- Use on-device authentication where possible

### 📋 Planned
- [ ] `school.search_courses` - Search course catalog (public)
- [ ] `school.get_course_info` - Get public course information
- [ ] `school.list_professors` - List professors by department

### 🔮 Future Ideas (Deferred)
- [ ] `school.login_*` - Campus login proxies
  - **Status:** Deferred due to privacy concerns
  - **Alternative:** Keep login on-device

---

## Phase 6 - Aggregation & Search ✅ IN PROGRESS (2025-03-15)

### ✅ Implemented
- [x] `aggregator.search` - Multi-source concurrent search (RAG + GitHub + COS)
  - ✅ Concurrent queries across all sources with timeout support
  - ✅ Result aggregation and ranking by relevance score
  - ✅ Configurable source selection and top_k
- [x] `aggregator.summarize` (async) - AI-powered search result summarization
  - ✅ ZHIPU BigModel API integration for intelligent summaries
  - ✅ Multiple summary styles: concise, detailed, bullet_points, qa
  - ✅ Multi-language support (Chinese/English auto-detection)
  - ✅ Key points extraction
  - ✅ Fallback to extractive summary when AI unavailable

### 📋 Planned
- [ ] `aggregator.rank` - Advanced ranking and sorting algorithms
- [ ] `search.semantic` - Semantic search across all sources

### 🔮 Future Ideas
- [ ] `search.federated` - Federated search with external APIs
- [ ] `search.recommend` - Recommendation engine

---

## Phase 7 - Advanced Features (Q3-Q4 2025)

### 📋 Planned
- [ ] `cache.clear` - Clear server cache
- [ ] `config.get` - Get service configuration
- [ ] `metrics.get` - Get service metrics
- [ ] `logs.query` - Query service logs

### 🔮 Future Ideas
- [ ] `analytics.dashboard` - Usage analytics
- [ ] `billing.get_usage` - Billing and usage tracking
- [ ] `alerts.configure` - Configure alerts and notifications

---

## Phase 8 - AI/ML Features (Q4 2025+)

### 🔮 Future Ideas
- [ ] `ai.summarize` - AI-powered document summarization
- [ ] `ai.translate` - Multi-language translation
- [ ] `ai.chat` - Conversational AI for course Q&A
- [ ] `ai.recommend` - Personalized course recommendations
- [ ] `ai.grade_predict` - Grade prediction (privacy-sensitive)

---

## Technical Debt & Improvements

### Performance ✅
- [x] **Full concurrency support** - All services now support concurrent processing
  - [x] Async skills with job queue (pr.preview, pr.submit, mcp.call_tool, rag.ingest)
  - [x] Batch operations with worker pools (files.*, cos.*)
  - [x] Atomic counters for thread-safe progress tracking
  - [x] Context-aware cancellation
- [ ] Implement caching layer (Redis)
- [ ] Add rate limiting per user
- [ ] Optimize database queries
- [ ] CDN integration for COS

### Security
- [ ] API key rotation
- [ ] OAuth2 integration
- [ ] Audit logging
- [ ] Data encryption at rest

### Monitoring
- [ ] Langfuse integration for tracing
- [ ] Prometheus metrics
- [ ] Grafana dashboards
- [ ] Alerting rules

### Testing
- [ ] Integration test suite
- [ ] Load testing
- [ ] Chaos engineering
- [ ] E2E tests

---

## API Versioning Strategy

### Current: v1
- Base endpoints: `/v1/skills`, `/v1/jobs`
- All current skills

### Future: v2 (2026+)
- Breaking changes will bump to `/v2`
- Deprecation warnings in v1
- Migration guide provided

---

## Priority Matrix

| Feature | Impact | Effort | Priority |
|---------|--------|--------|----------|
| rag.ingest | High | Medium | P0 |
| pr.list | Medium | Low | P1 |
| files.upload | Medium | Medium | P1 |
| github.read_file | Low | Low | P2 |
| ai.summarize | High | High | P3 |
| analytics | Medium | Medium | P2 |

---

## Success Metrics

### Phase 1 (Q1)
- [ ] rag.ingest processes 1000+ documents
- [ ] Average ingestion time < 30s per document

### Phase 2 (Q2)
- [ ] PR workflow handles 100+ submissions/month
- [ ] 95% success rate for pr.submit

### Phase 3 (Q3)
- [ ] COS storage reaches 50GB
- [ ] 99.9% uptime for all skills

### Phase 4 (Q4)
- [ ] 500+ active users
- [ ] < 100ms average response time

---

## Contributing

To propose new features:
1. Create issue in GitHub
2. Tag with `feature-request`
3. Include use case and expected behavior
4. Review in weekly sync

---

**Last Updated:** 2025-03-15
**Next Review:** 2025-04-01
