# skill: pr.preview

## 适用场景
预览课程改动。深圳支持 multi-project ops。

## 操作路由规则（2026-03-18）
- 不再生成或转发 `append_course_review`。
- 用户表达“对子课程评价”时，统一改发 `append_course_section_item`。
- `section_title` 默认建议使用 `课程评价`（若上游显式给出则按显式值）。
- 用户表达“评价某位老师”时，发 `add_course_teacher_review`。
- `harbin` / `weihai` 的 multi-project 操作会被拦截并返回 `INVALID_OPS`。
- normal 操作保持不变（`add_lecturer_review`、`add_section_item`）。

## 请求定义
- Method: POST
- Path: /v1/skills/pr.preview:invoke
- Headers:
  - Content-Type: application/json
  - Accept: application/json
- Body:
  {
    "input": {"campus":"harbin","course_code":"CS101","ops":[{"op":"add_section_item","title":"更新","content":"内容"}]}
  }

## ReAct 模板
- Thought: 我需要 pr.preview 来完成当前子任务。
- Action: 调用 /v1/skills/pr.preview:invoke 并传入最小参数。
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
