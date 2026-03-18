# Agent-Backend Agent API 索引（ReAct 版）

## 这套文档是否适合 ReAct
- 适合。原因: 统一调用入口、明确同步/异步分流、错误结构稳定。
- ReAct 推荐流程: 先查技能目录 -> 选 skill -> 调用 -> 观测 -> 重试或切换。

## 快速入口
- [GET /health](endpoint-health.md)
- [GET /v1/skills](endpoint-skills-list.md)
- [POST /v1/skills/{name}:invoke](endpoint-skills-invoke.md)
- [GET /v1/jobs/{job_id}](endpoint-jobs-get.md)

## 功能索引（按任务）
- 内容分享入 RAG: [skill-data-ingest](skill-data-ingest.md), [skill-data-ingest-batch](skill-data-ingest-batch.md), [skill-rag-intake-manual-folder](skill-rag-intake-manual-folder.md)
- 统一搜索: [skill-search](skill-search.md), [skill-courses-search](skill-courses-search.md), [skill-rag-query](skill-rag-query.md)
- 课程 PR: [skill-pr-preview](skill-pr-preview.md), [skill-pr-submit](skill-pr-submit.md), [skill-pr-lookup](skill-pr-lookup.md)
- 爬虫: [skill-crawl4ai-page](skill-crawl4ai-page.md), [skill-crawl4ai-site](skill-crawl4ai-site.md), [skill-crawl4ai-status](skill-crawl4ai-status.md)
- 文件/COS: [skill-files-upload](skill-files-upload.md), [skill-files-download](skill-files-download.md), [skill-cos-save-file](skill-cos-save-file.md)
- MCP: [skill-mcp-list-servers](skill-mcp-list-servers.md), [skill-mcp-list-tools](skill-mcp-list-tools.md), [skill-mcp-call-tool](skill-mcp-call-tool.md)

## 全部技能接口清单
- [search](skill-search.md)
- [data.ingest](skill-data-ingest.md)
- [data.ingest_batch](skill-data-ingest_batch.md)
- [rag.query](skill-rag-query.md)
- [rag.ingest](skill-rag-ingest.md)
- [rag.ingest_from_github](skill-rag-ingest_from_github.md)
- [rag.intake_manual_folder](skill-rag-intake_manual_folder.md)
- [rag.sync_to_repo](skill-rag-sync_to_repo.md)
- [pr.preview](skill-pr-preview.md)
- [pr.submit](skill-pr-submit.md)
- [pr.lookup](skill-pr-lookup.md)
- [courses.search](skill-courses-search.md)
- [course.read](skill-course-read.md)
- [hit.teacher](skill-hit-teacher.md)
- [hit.teachers](skill-hit-teachers.md)
- [crawl4ai.page](skill-crawl4ai-page.md)
- [crawl4ai.site](skill-crawl4ai-site.md)
- [crawl4ai.status](skill-crawl4ai-status.md)
- [github.batch_download](skill-github-batch_download.md)
- [document.convert](skill-document-convert.md)
- [cos.save_file](skill-cos-save_file.md)
- [cos.delete_file](skill-cos-delete_file.md)
- [cos.list_files](skill-cos-list_files.md)
- [cos.get_presigned_url](skill-cos-get_presigned_url.md)
- [cos.get_quota](skill-cos-get_quota.md)
- [files.upload](skill-files-upload.md)
- [files.download](skill-files-download.md)
- [mcp.list_servers](skill-mcp-list_servers.md)
- [mcp.list_tools](skill-mcp-list_tools.md)
- [mcp.call_tool](skill-mcp-call_tool.md)
- [aggregator.summarize](skill-aggregator-summarize.md)
- [echo](skill-echo.md)
- [sleep_echo](skill-sleep_echo.md)

## ReAct 系统提示建议（最短版）
1. 先 GET /v1/skills 获取能力。
2. 所有业务都走 POST /v1/skills/{name}:invoke。
3. 不看 HTTP 200 断定成功，必须看 ok 字段。
4. 有 job_id 就轮询 /v1/jobs/{job_id}。
5. INVALID_INPUT 按错误提示补参后重试。
6. 连续失败两次后切换策略或回退。
