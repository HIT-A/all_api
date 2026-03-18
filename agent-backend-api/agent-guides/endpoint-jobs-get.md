# GET /v1/jobs/{job_id}

## 适用场景
轮询异步任务状态，直到 succeeded 或 failed。

## 请求定义
- Method: GET
- Path: /v1/jobs/{job_id}
- Headers:
  - Accept: application/json
- Body: 无

## 成功响应样例
{"ok":true,"job":{"status":"queued|running|succeeded|failed","output_json":{...}}}

## ReAct 调用建议
1. Thought: 判断是否需要调用该端点。
2. Action: 发起 HTTP 请求。
3. Observation: 解析返回并决定下一步。
4. 如果失败: 记录 error.code/error.message，再决定重试或降级。
