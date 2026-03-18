# GET /health

## 适用场景
用于探活、启动前检查、失败后快速判断服务可用性。

## 请求定义
- Method: GET
- Path: /health
- Headers:
  - Accept: application/json
- Body: 无

## 成功响应样例
{"status":"ok"}

## ReAct 调用建议
1. Thought: 判断是否需要调用该端点。
2. Action: 发起 HTTP 请求。
3. Observation: 解析返回并决定下一步。
4. 如果失败: 记录 error.code/error.message，再决定重试或降级。
