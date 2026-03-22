# agent-backend 2026-03-22 架构变更说明

## 变更概述

agent-backend 的数据入仓链路已重构为**统一 pipeline**，所有入口共用同一套 dedup + COS + Qdrant 流程。

**核心变化：** 移除 GitHub Raw 队列中转，改用 SQLite SHA256 去重 + COS 直接存储 + Qdrant 直接向量化。

---

## 旧架构（已废弃）

```
data.ingest / files.upload → COS → GitHub Raw (incoming/*_raw/)
                                            ↓
                              rag.intake_manual_folder 监听 → 转换 → Qdrant
```

问题：GitHub Raw 队列冗余、命名丢失原始信息、COS 和 GitHub 重复存储。

---

## 新架构

```
┌──────────────────────────────────────────────────────────────┐
│  所有入口 共用: SHA256 dedup → COS(raw) → Qdrant            │
├──────────────────────────────────────────────────────────────┤
│ data.ingest         → dedup → COS → Qdrant                  │
│ files.upload        → dedup → COS → Qdrant (可选)            │
│ crawl4ai.page       → dedup → COS → Qdrant                  │
│ annas-archive dl    → dedup → COS → Qdrant                  │
│ github.batch_dl     → dedup → COS → Qdrant                  │
│ rag.sync_to_repo    → dedup → COS → Qdrant (同步 GitHub 仓) │
└──────────────────────────────────────────────────────────────┘
         ↑
  GitHub 仅保留 audit trail（raw/ + converted/），不参与数据流
```

---

## 去重机制

- **依据：** SHA256(content) 内容哈希
- **存储：** `jobs.db` 中的 `dedup_records` 表（SQLite）
- **行为：** 相同内容自动跳过，返回已存在的 COS key
- **强制覆盖：** `data.ingest` 可用 `overwrite: true`

---

## COS 路径命名

| 入口 | COS Key 格式 | 示例 |
|------|------------|------|
| `data.ingest` | `{prefix}/{type}/{sha}_{name}.md` | `rag-content/manual/abc123_notes.md` |
| `files.upload` | 用户指定 key | `uploads/slides/ch01.pdf` |
| `crawl4ai.page` | `{prefix}/crawl/{sha}_{name}.md` | `crawls/crawl/abc123_page.md` |
| Anna's Archive | `annas/{sha}/{filename}` | `annas/abc123/book.pdf` |
| `github.batch_download` | `{prefix}/github/{sha}/{path}` | `rag-content/github/abc123/slides/ch01.pdf` |
| `rag.sync_to_repo` | `{prefix}/{owner-repo}/{sha}/{path}` | `rag-content/HIT-A-PPT/abc123/slides/ch01.pdf` |

---

## 废弃/删除

- **`rag.intake_manual_folder`**：已删除注册，不再需要
- **GitHub Raw 队列**：`incoming/manual_raw` 等目录不再写入

---

## 新增依赖

- **`dedup_store.go`**：SQLite SHA256 去重表
- **`rag_ingest_direct.go`**：直接将 markdown 内容 ingest 到 Qdrant
- **`MINIMAX_API_KEY`** / **`MINIMAX_MODEL`**：环境变量已配置，优先用于 LLM 总结

---

## 文件变更清单

| 文件 | 变更 |
|------|------|
| `internal/skills/dedup_store.go` | **新增** - SQLite 去重存储 |
| `internal/skills/rag_ingest_direct.go` | **新增** - 直接 Qdrant ingest |
| `internal/skills/data_ingest_skill.go` | 重构 - 移除 GitHub Raw，推崇 dedup+cos+rag |
| `internal/skills/file_skills.go` | 重构 - 同上 + SHA256 去重 |
| `internal/skills/crawl4ai_skills.go` | 重构 - `QueueToIntake` → `StoreInCOS` + `AutoIngestRAG` |
| `internal/skills/mcp_skills.go` | 重构 - Anna's Archive download 改用新 pipeline |
| `internal/skills/github_batch_skills.go` | 重构 - 同上 |
| `internal/skills/rag_sync_skill.go` | 重构 - 加入 dedup + COS + Qdrant ingest |
| `internal/skills/registry.go` | 删除 - `rag.intake_manual_folder` 注册 |
