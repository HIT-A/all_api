# GET /v1/skills

## 适用场景
用于让 Agent 动态发现技能名与是否异步。ReAct 的第一步建议先调用它。

## 请求定义
- Method: GET
- Path: /v1/skills
- Headers:
  - Accept: application/json
- Body: 无

## 成功响应样例
{"skills":[{"name":"...","is_async":false}]}

## ReAct 调用建议
1. Thought: 判断是否需要调用该端点。
2. Action: 发起 HTTP 请求。
3. Observation: 解析返回并决定下一步。
4. 如果失败: 记录 error.code/error.message，再决定重试或降级。
