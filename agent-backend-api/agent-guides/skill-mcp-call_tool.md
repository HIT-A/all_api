# skill: mcp.call_tool

## 适用场景
调用 MCP 工具。

## 请求定义
- Method: POST
- Path: /v1/skills/mcp.call_tool:invoke
- Headers:
  - Content-Type: application/json
  - Accept: application/json
- Body:
  {
    "input": {"server":"brave-search","tool":"brave_web_search","arguments":{"query":"OpenAI"}}
  }

## ReAct 模板
- Thought: 我需要 mcp.call_tool 来完成当前子任务。
- Action: 调用 /v1/skills/mcp.call_tool:invoke 并传入最小参数。
- Observation: 读取 ok/output 或 job_id。
- Next Thought: 根据结果选择继续推理、补参数重试、或切换 skill。

## 异步处理
- 该 skill 为异步。
- 第一步: 调用 POST /v1/skills/{name}:invoke。
- 如果返回 job_id，轮询 GET /v1/jobs/{job_id}。
- 结束条件: job.status 为 succeeded 或 failed。

## 常见错误恢复
- INVALID_INPUT: 根据 error.message 补必填字段。
- INTERNAL/NETWORK: 最多重试 1-2 次并指数退避。
- NOT_FOUND: 检查对象是否存在（repo/course/crawl_id 等）。
