# POST /v1/skills/{name}:invoke

## 适用场景
所有业务能力统一入口。

## 请求定义
- Method: POST
- Path: /v1/skills/{name}:invoke
- Headers:
  - Content-Type: application/json
  - Accept: application/json
- Body: {"input": { ... }}

## 成功响应样例
同步: {"ok":true,"output":{...}}
异步: {"ok":true,"job_id":"..."}
失败: {"ok":false,"error":{"code":"...","message":"...","retryable":true}}

## ReAct 调用建议
1. Thought: 判断是否需要调用该端点。
2. Action: 发起 HTTP 请求。
3. Observation: 解析返回并决定下一步。
4. 如果失败: 记录 error.code/error.message，再决定重试或降级。
