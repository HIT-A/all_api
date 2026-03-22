# agent-backend 实现状态报告

**更新时间:** 2026-03-22

---

## 一、架构概览

### 统一数据 Pipeline（2026-03-22 重大重构）

所有数据入口统一走：**SHA256 dedup → COS 原始存储 → Qdrant 向量**

```
data.ingest / files.upload / crawl4ai.page / annas-archive / github.batch_download / rag.sync_to_repo
    │
    ▼
SQLite dedup_records 表（SHA256 去重）
    │
    ├─→ [已存在] 跳过
    │
    ▼ [新内容]
COS 原始文件存储
    │
    ▼
直接 RAG ingest → Qdrant
    │
    ▼
可查询
```

**废弃：**
- `rag.intake_manual_folder` — 已删除注册
- GitHub Raw 队列（`incoming/*_raw/`）— 不再写入

---

## 二、Skill 列表（30 个）

| 分类 | Skills | 状态 |
|------|--------|------|
| **Test** | echo, sleep_echo | ✅ 正常 |
| **RAG** | rag.query, rag.ingest, rag.ingest_from_github, rag.sync_to_repo | ✅ 全部重构 |
| **PR** | pr.preview, pr.submit, pr.lookup | ✅ 正常 |
| **Course** | course.read, courses.search | ✅ 正常 |
| **HIT** | hit.teacher, hit.teachers | ✅ 正常 |
| **COS** | cos.save_file, cos.delete_file, cos.list_files, cos.get_presigned_url | ✅ 正常 |
| **Files** | files.upload, files.download | ✅ 已重构 |
| **Search** | search, aggregator.summarize | ✅ 正常 |
| **Data** | data.ingest, data.ingest_batch | ✅ 已重构 |
| **Crawl** | crawl4ai.page, crawl4ai.site, crawl4ai.status | ✅ page 已重构 |
| **GitHub** | github.batch_download, document.convert | ✅ 已重构 |
| **MCP** | mcp.list_servers, mcp.list_tools, mcp.call_tool | ✅ 正常 |

---

## 三、新增文件

| 文件 | 说明 |
|------|------|
| `internal/skills/dedup_store.go` | SQLite SHA256 去重存储 |
| `internal/skills/rag_ingest_direct.go` | 直接将 markdown ingest 到 Qdrant |

---

## 四、环境变量

| 变量 | 值 | 说明 |
|------|-----|------|
| `COS_SECRET_ID` | `***` | 腾讯云 COS |
| `COS_SECRET_KEY` | `***` | 腾讯云 COS |
| `COS_BUCKET` | `hita-001-1410950200` | 腾讯云 COS |
| `COS_REGION` | `ap-guangzhou` | 腾讯云 COS |
| `GITHUB_TOKEN` | `***` | GitHub |
| `MINIMAX_API_KEY` | `***` | MiniMax LLM（优先） |
| `MINIMAX_MODEL` | `MiniMax-M2.7` | MiniMax 模型 |
| `BIGMODEL_API_KEY` | `***` | BigModel（回退/嵌入） |
| `QDRANT_URL` | `http://127.0.0.1:6333` | Qdrant 向量库 |
| `QDRANT_COLLECTION` | `hit_courses_2048` | Qdrant collection |
| `ANNAS_SECRET_KEY` | `***` | Anna's Archive |

---

## 五、已解决问题

| 问题 | 修复方式 |
|------|---------|
| COS SDK 接入 | 腾讯云 COS SDK v0.7.73 |
| `usedToday` 并发安全 | RWMutex |
| COS 错误静默忽略 | 添加警告日志 |
| DownloadBytes 无限制 | io.LimitReader 100MB |
| 弱拼音转换 | mozillazg/go-pinyin 真实转换 |
| GitHub Raw 队列冗余 | 移除，改用 dedup + COS |
| COS 冗余存储转换后文件 | 删除 markdown 上传 COS |
| GitHub Converted 命名丢失原始信息 | 改用相对路径 |
| `cos.get_quota` 无用 | 已删除 |
| `summarizeWithLLM` stub | 接入 MiniMax API |
| `processCrawlSource` stub | 接入 crawl4ai.page |
| `processedChunks` 非原子 | 影响小，低优先级 |

---

## 六、Go 版本

当前环境：Go 1.24.11  
go.mod 要求：Go 1.24.11  
依赖 x/net 降级至 v0.34.0（兼容 1.24.11）

如需使用 Go 1.25.0，恢复 go.mod 中 `go 1.25.0` 和 `golang.org/x/net v0.52.0`。
