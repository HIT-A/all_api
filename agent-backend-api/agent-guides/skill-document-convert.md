# skill: document.convert

## 适用场景
文档转 markdown。

## 请求定义
- Method: POST
- Path: /v1/skills/document.convert:invoke
- Headers:
  - Content-Type: application/json
  - Accept: application/json
- Body:
  {
    "input": {"url":"https://example.com/a.pdf"}
  }

## ReAct 模板
- Thought: 我需要 document.convert 来完成当前子任务。
- Action: 调用 /v1/skills/document.convert:invoke 并传入最小参数。
- Observation: 读取 ok/output 或 job_id。
- Next Thought: 根据结果选择继续推理、补参数重试、或切换 skill。

## 异步处理
- 该 skill 为同步，直接读取 output。

## 常见错误恢复
- INVALID_INPUT: 根据 error.message 补必填字段。
- INTERNAL/NETWORK: 最多重试 1-2 次并指数退避。
- NOT_FOUND: 检查对象是否存在（repo/course/crawl_id 等）。
