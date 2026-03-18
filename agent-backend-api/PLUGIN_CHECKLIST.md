# agent-backend 第三方插件清单

用途：给“应用端 Agent”作为固定接口清单，避免它不知道如何发包。

基准：
- Base URL: http(s)://<host>:8080
- 调用约束：POST /v1/skills/{name}:invoke 永远返回 HTTP 200，成败看响应体 ok 字段。
- 当前技能数：33

## Header 模板（键值对）

- H_JSON
  - Content-Type: application/json
  - Accept: application/json
- H_GET
  - Accept: application/json

说明：当前默认只使用 H_JSON/H_GET。

---

## 平台 API（4个）

| 插件名称 | 插件描述 | 插件 URL | Header 列表 |
|---|---|---|---|
| backend.health | 健康检查 | GET http(s)://<host>:8080/health | H_GET |
| backend.skills.list | 拉取技能目录 | GET http(s)://<host>:8080/v1/skills | H_GET |
| backend.skills.invoke | 统一技能调用入口 | POST http(s)://<host>:8080/v1/skills/{name}:invoke | H_JSON |
| backend.jobs.get | 异步任务查询 | GET http(s)://<host>:8080/v1/jobs/{job_id} | H_GET |

---

## Skill API（33个）

统一 URL 规则：
- POST http(s)://<host>:8080/v1/skills/{name}:invoke

统一 Header：
- H_JSON

| 插件名称 | 插件描述 | 插件 URL | Header 列表 |
|---|---|---|---|
| skill.search | 聚合搜索 | POST .../v1/skills/search:invoke | H_JSON |
| skill.data.ingest | 数据接入 | POST .../v1/skills/data.ingest:invoke | H_JSON |
| skill.data.ingest_batch | 批量数据接入 | POST .../v1/skills/data.ingest_batch:invoke | H_JSON |
| skill.rag.query | 向量检索 | POST .../v1/skills/rag.query:invoke | H_JSON |
| skill.rag.ingest | RAG 入库（异步） | POST .../v1/skills/rag.ingest:invoke | H_JSON |
| skill.rag.ingest_from_github | GitHub 入库（异步） | POST .../v1/skills/rag.ingest_from_github:invoke | H_JSON |
| skill.rag.intake_manual_folder | 手动目录 intake（异步） | POST .../v1/skills/rag.intake_manual_folder:invoke | H_JSON |
| skill.rag.sync_to_repo | 同步到仓库（异步） | POST .../v1/skills/rag.sync_to_repo:invoke | H_JSON |
| skill.pr.preview | PR 预览（异步） | POST .../v1/skills/pr.preview:invoke | H_JSON |
| skill.pr.submit | PR 提交（异步） | POST .../v1/skills/pr.submit:invoke | H_JSON |
| skill.pr.lookup | PR 查询 | POST .../v1/skills/pr.lookup:invoke | H_JSON |
| skill.courses.search | 课程搜索（返回可包含 teachers/aliases） | POST .../v1/skills/courses.search:invoke | H_JSON |
| skill.course.read | 课程读取（README-first；include_toml=true 才返回 TOML） | POST .../v1/skills/course.read:invoke | H_JSON |
| skill.hit.teacher | 单老师查询 | POST .../v1/skills/hit.teacher:invoke | H_JSON |
| skill.hit.teachers | 批量老师查询 | POST .../v1/skills/hit.teachers:invoke | H_JSON |
| skill.crawl4ai.page | 网页单页抓取 | POST .../v1/skills/crawl4ai.page:invoke | H_JSON |
| skill.crawl4ai.site | 全站抓取（异步） | POST .../v1/skills/crawl4ai.site:invoke | H_JSON |
| skill.crawl4ai.status | 抓取状态 | POST .../v1/skills/crawl4ai.status:invoke | H_JSON |
| skill.github.batch_download | GitHub 批量下载（异步） | POST .../v1/skills/github.batch_download:invoke | H_JSON |
| skill.document.convert | 文档转换 | POST .../v1/skills/document.convert:invoke | H_JSON |
| skill.cos.save_file | COS 保存 | POST .../v1/skills/cos.save_file:invoke | H_JSON |
| skill.cos.delete_file | COS 删除 | POST .../v1/skills/cos.delete_file:invoke | H_JSON |
| skill.cos.list_files | COS 列表 | POST .../v1/skills/cos.list_files:invoke | H_JSON |
| skill.cos.get_presigned_url | COS 预签名 | POST .../v1/skills/cos.get_presigned_url:invoke | H_JSON |
| skill.cos.get_quota | COS 配额 | POST .../v1/skills/cos.get_quota:invoke | H_JSON |
| skill.files.upload | 文件上传 | POST .../v1/skills/files.upload:invoke | H_JSON |
| skill.files.download | 文件下载 | POST .../v1/skills/files.download:invoke | H_JSON |
| skill.mcp.list_servers | MCP 服务器列表 | POST .../v1/skills/mcp.list_servers:invoke | H_JSON |
| skill.mcp.list_tools | MCP 工具列表 | POST .../v1/skills/mcp.list_tools:invoke | H_JSON |
| skill.mcp.call_tool | MCP 工具调用（异步） | POST .../v1/skills/mcp.call_tool:invoke | H_JSON |
| skill.aggregator.summarize | 结果总结（异步） | POST .../v1/skills/aggregator.summarize:invoke | H_JSON |
| skill.echo | 回显测试 | POST .../v1/skills/echo:invoke | H_JSON |
| skill.sleep_echo | 延时回显（异步） | POST .../v1/skills/sleep_echo:invoke | H_JSON |

---

## 给应用端 Agent 的系统提示单（可直接粘贴）

你是应用端 API 调用代理，只能调用 agent-backend。

固定规则：
1. Base URL 为 http(s)://<host>:8080。
2. 调用技能时统一请求：POST /v1/skills/{name}:invoke。
3. Header 固定：Content-Type=application/json，Accept=application/json。
4. 不能根据 HTTP 状态码判断业务成功，必须读取响应体 ok 字段。
5. 如果响应是 {ok:true, job_id:"..."}，则每 1-2 秒轮询 GET /v1/jobs/{job_id}。
6. 轮询结束条件：job.status in [succeeded, failed]。
7. job.status=succeeded 时读取 job.output_json。
8. job.status=failed 时读取 job.error（或 error 字段）并返回可读错误。
9. 参数名必须严格按 skill 文档发送，不可自行改名。
10. pr.lookup 优先使用 number 字段（兼容 pr）。
11. pr.submit 建议传 idempotency_key，避免重试重复创建。
12. rag.sync_to_repo 可使用 repo/branch（等价 target_repo/target_branch）。
13. pr.preview/pr.submit 的 ops：深圳可用 normal + multi-project；哈工大本部/威海仅支持 normal。

最小请求模板：

POST /v1/skills/{name}:invoke
Headers:
- Content-Type: application/json
- Accept: application/json
Body:
{
  "input": {
    "按下方最小Body清单填写": true
  }
}

最小 Body 清单（33个 skill，直接照抄，不要猜字段）：

| skill | 最小 Body（即 input 对象） |
|---|---|
| search | {"query":"机器学习"} |
| data.ingest | {"source_type":"manual","source_name":"note","content":"# hello"} |
| data.ingest_batch | {"items":[{"source_type":"manual","source_name":"note1","content":"# a"}]} |
| rag.query | {"query":"线性代数"} |
| rag.ingest | {"repo":"HIT-A/HITA_RagData","ref":"main"} |
| rag.ingest_from_github | {"repos":[{"org":"HIT-A","repo":"all_api","ref":"main"}]} |
| rag.intake_manual_folder | {"repo":"HIT-A/HITA_RagData","branch":"main","folder":"manual"} |
| rag.sync_to_repo | {"repo":"HIT-A/HITA_RagData","branch":"main","daily":false} |
| pr.preview | {"campus":"harbin","course_code":"CS101","ops":[{"op":"add_section_item","title":"更新","content":"内容"}]} |
| pr.submit | {"campus":"harbin","course_code":"CS101","ops":[{"op":"add_section_item","title":"更新","content":"内容"}],"idempotency_key":"submit-cs101-20260318"} |
| pr.lookup | {"org":"HIT-A","repo":"DATA_HB","number":8} |
| courses.search | {"keyword":"python","campus":"harbin"} |
| course.read | {"campus":"harbin","course_code":"CS101","include_toml":false} |
| hit.teacher | {"name":"张三"} |
| hit.teachers | {"names":["张三","李四"]} |
| crawl4ai.page | {"url":"https://example.com"} |
| crawl4ai.site | {"start_url":"https://example.com","max_pages":20} |
| crawl4ai.status | {"crawl_id":"<crawl_id>"} |
| github.batch_download | {"repos":[{"org":"HIT-A","repo":"all_api","ref":"main"}]} |
| document.convert | {"url":"https://example.com/a.pdf"} |
| cos.save_file | {"key":"tmp/hello.txt","content_base64":"aGVsbG8="} |
| cos.delete_file | {"key":"tmp/hello.txt"} |
| cos.list_files | {"prefix":"tmp/"} |
| cos.get_presigned_url | {"key":"tmp/hello.txt"} |
| cos.get_quota | {} |
| files.upload | {"key":"tmp/a.txt","text":"hello"} |
| files.download | {"key":"tmp/a.txt","format":"url"} |
| mcp.list_servers | {} |
| mcp.list_tools | {} |
| mcp.call_tool | {"server":"brave-search","tool":"brave_web_search","arguments":{"query":"OpenAI"}} |
| aggregator.summarize | {"query":"机器学习","results":[{"title":"A","content":"B"}]} |
| echo | {"text":"ping"} |
| sleep_echo | {"text":"ping","sleep_seconds":1} |

PR 多格式示例（深圳 HOA）：

```json
{
  "input": {
    "campus": "shenzhen",
    "course_code": "HOA-MULTI",
    "ops": [
      {
        "op": "append_course_review",
        "course_name": "高等数学",
        "topic": "作业",
        "content": "测试multi-project"
      }
    ]
  }
}
```

```json
{
  "input": {
    "campus": "shenzhen",
    "course_code": "HOA-MULTI",
    "ops": [
      {
        "op": "append_course_section_item",
        "course_name": "高等数学",
        "section_title": "学习资料",
        "item": {
          "title": "讲义",
          "content": "测试"
        }
      }
    ],
    "idempotency_key": "shenzhen-multi-20260318"
  }
}
```

注意：
- 在 `harbin` 或 `weihai` 传入上述 multi-project 操作会返回 `INVALID_OPS`。

兜底规则：
- 不确定参数时，先调用 GET /v1/skills 查看 input_schema，再发包。
- 如果返回 INVALID_INPUT，按 error.message 补字段后重试。

最小轮询模板：

GET /v1/jobs/{job_id}
Headers:
- Accept: application/json

返回处理模板：
- 若 ok=false：输出 error.code + error.message，并根据 retryable 决定是否建议重试。
- 若 ok=true 且有 output：直接返回 output。
- 若 ok=true 且有 job_id：进入轮询流程。
