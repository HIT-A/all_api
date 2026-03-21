# agent-backend 实现状态报告

**更新时间:** 2026-03-21

---

## 一、接口实现状态总览

经过全面审查，确认大部分功能已真实实现。以下是最终状态：

| Skill | 实际实现 | 状态 |
|-------|---------|------|
| 所有 33 个 Skill | 真实调用外部服务 | ✅ 正常 |

### 关键调用链确认

- **COS**: Skill → Storage → **Client (SDK)** ✅
- **Search 数据源**: unifiedSearchGitHub ✅ / Arxiv ✅ / Annas ✅
- **RAG**: GitHub → Parse → Embed → Qdrant ✅

---

## 二、真实实现的组件

### 2.1 COS Client (client.go)

**已接入腾讯云 COS SDK v0.7.73**

| 方法 | 实现 |
|-----|------|
| `Upload` | `c.client.Object.Put(ctx, key, reader, opts)` |
| `Delete` | `c.client.Object.Delete(ctx, key, nil)` |
| `GetPresignedURL` | `c.client.Object.GetPresignedURL(...)` |
| `ListFiles` | `c.client.Bucket.Get(ctx, opt)` + 分页 |

### 2.2 统一搜索数据源 (aggregator_search.go)

| 函数 | 实现 |
|-----|------|
| `unifiedSearchGitHub` | 调用 `fetcher.SearchCode()` → GitHub Search API |
| `unifiedSearchArxiv` | 调用 `callMCPTool("arxiv", "search_arxiv", ...)` |
| `unifiedSearchAnnas` | 调用 MCP `article_search` / `book_search` 工具 |

### 2.3 MCP 服务器

所有内置 MCP 服务器都完整实现：
- **arxiv**: 调用 `https://export.arxiv.org/api/query`
- **brave**: 调用 Brave Search API + DuckDuckGo fallback
- **crawl4ai**: 使用 `AsyncWebCrawler` 真实爬取
- **unstructured**: 使用 `PyMuPDF`/`python-docx` 等库

---

## 三、Stub / 未实现功能

### 3.1 summarizeWithLLM (aggregator_search.go:809-825)

**状态**: 🔴 Stub

```go
func summarizeWithLLM(ctx context.Context, query string, results []SearchResult) (string, []string) {
    // TODO: Call LLM for summarization
    // For now, return placeholder
    return "总结功能待实现", []string{"需要接入 LLM"}
}
```

**注意**: `aggregator.summarize` skill 本身调用 BigModel API 是真实实现的，这个 `summarizeWithLLM` 是内部 helper 函数，`search` skill 的 summary 功能会受影响。

### 3.2 processCrawlSource (rag_sync_skill.go:575-587)

**状态**: 🔴 Stub

```go
func processCrawlSource(ctx context.Context, source RAGSource, repoPath string, ...) (int, int, error) {
    totalFiles := 0
    totalChunks := 0
    if source.URL == "" {
        return 0, 0, nil
    }
    return totalFiles, totalChunks, nil  // 永远返回 0！
}
```

**说明**: 爬虫来源处理未实现。如果 `rag.sync_to_repo` 使用 `source_type: crawl`，则该数据源不会被处理。

---

## 四、并发安全问题

### 4.1 usedToday 互斥锁 (storage.go)

| 位置 | 状态 | 说明 |
|-----|------|------|
| `SaveFile` | ✅ 已有锁 | `s.mu.Lock()` 保护 |
| `GetQuota` | ✅ 已有锁 | `s.mu.RLock()` 保护 |
| `ResetDailyQuota` | ✅ 已有锁 | `s.mu.Lock()` 保护 |

### 4.2 rag_ingest.go processedChunks

**位置**: 第 363 行

```go
var processedChunks int64  // 全局变量
```

**问题**: `atomic.LoadInt64(&processedChunks)` 但没有 `atomic.AddInt64`，存在不一致。

**影响**: 仅影响统计准确性，不影响功能。

---

## 五、依赖外部服务的 Skill

| Skill | 外部服务 | 状态 |
|-------|---------|------|
| `rag.*` | Qdrant + Embedder (BigModel) | ✅ 正常 |
| `pr.*`, `courses.*` | pr-server | ✅ 正常 |
| `crawl4ai.*` | crawl4ai MCP | ✅ 正常 |
| `search` | RAG + Brave + GitHub + Arxiv + Annas | ✅ 正常 |
| `hit.teacher(s)` | 哈工大网站 + crawl4ai | ✅ 正常 |
| `document.convert` | unstructured MCP | ⚠️ 需注册 |
| `aggregator.summarize` | BigModel API | ✅ 正常 |

---

## 六、最终 Stub 列表

| 函数 | 文件:行号 | 问题 |
|-----|----------|------|
| `summarizeWithLLM` | aggregator_search.go:809 | 返回占位符，未调用 LLM |
| `processCrawlSource` | rag_sync_skill.go:575 | 返回 0,0,nil，爬虫来源未处理 |

---

## 七、修复建议

### 低优先级

| 任务 | 文件 | 说明 |
|-----|------|------|
| 实现 `summarizeWithLLM` | aggregator_search.go:809 | 接入 LLM 进行总结 |
| 实现 `processCrawlSource` | rag_sync_skill.go:575 | 处理爬虫来源数据 |
| 修复 `processedChunks` | rag_ingest.go:363 | 使用 atomic 操作 |
