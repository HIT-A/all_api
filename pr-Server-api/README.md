# pr-server API (internal)

This document describes the **internal** API for the Go-based `pr-server`.

## Scope

`pr-server` is responsible for writing course review content into GitHub repositories via Pull Requests:

- Fetch current `readme.toml`
- Apply **structured ops** (add/update/delete review items)
- Render `README.md` from TOML
- Create branch/commit/PR
- Support **multiple campuses** (Shenzhen/Harbin/Weihai) via configuration-driven repo mapping

Non-goals:

- RAG/embedding/crawlers/aggregation (handled by `hoa-agent-backend`)
- Public client API.

## Contract statement (IMPORTANT)

- **pr-server is internal REST only**. It is never called directly by the mobile/web app.
- **agent-backend is the skills gateway** (SSOT `/v1/skills`, `:invoke`, `/v1/jobs`).
- agent-backend will **adapt/forward** to pr-server and normalize outputs to the skills protocol.

## 2026-03-18 完整修复（先后差异）

| 能力 | 修复前 | 修复后 |
|---|---|---|
| `POST /v1/courses:search` | 主要依赖 GitHub repo 名称命中（`in:name`），老师名/别名命中率低 | 同时匹配 `code/name/repo/teachers/aliases`，支持“老师 -> 课程”检索 |
| `POST /v1/courses:search` 返回字段 | `code/name/org/repo` | 新增 `teachers[]`、`aliases[]`，便于上游直接展示和二次匹配 |
| `POST /v1/course:read` 返回语义 | 结果中固定包含 `readme_toml + readme_md` | README-first：默认仅返回 `readme_md`；仅当 `include_toml=true` 时返回 `readme_toml` |
| 多地区操作兼容 | 三地区仅支持 normal 两类操作 | 深圳（HOA）支持 normal + multi-project；哈工大本部/威海仅支持 normal |

### course:read 新请求参数

```json
{
  "target": { "campus": "weihai", "course_code": "MATH1001" },
  "include_toml": false
}
```

### courses:search 新返回字段示例

```json
{
  "ok": true,
  "data": {
    "results": [
      {
        "code": "CS101",
        "name": "计算机导论",
        "org": "HITSZ-OpenAuto",
        "repo": "CS101",
        "teachers": ["张三", "李四"],
        "aliases": ["计算机基础", "COMP101"]
      }
    ]
  }
}
```

### courses:search 行为补充（实现细节）

- 优先路径：仓库名检索 + code search 命中仓库后解析 `readme.toml`。
- 回退路径：当上述路径无结果时，会进行有限量仓库扫描并本地匹配教师/别名。
- 性能边界：回退扫描有上限（受服务端配置控制），用于平衡召回率与响应延迟。

## 多格式操作兼容（按校区）

### 深圳（HOA）

- 支持 normal 操作：
  - `add_lecturer_review`
  - `add_section_item`
- 支持 multi-project 操作：
  - `append_course_review`
  - `add_course_teacher_review`
  - `append_course_section_item`

### 哈工大本部 / 威海

- 仅支持 normal 操作：
  - `add_lecturer_review`
  - `add_section_item`
- 如果传入 multi-project 操作，返回 `400 INVALID_OPS`。

### multi-project 操作示例

```json
{
  "target": { "campus": "shenzhen", "course_code": "HOA-MULTI" },
  "ops": [
    {
      "op": "append_course_review",
      "course_name": "AUTO1001",
      "topic": "课程评价",
      "content": "整体不错",
      "author": { "name": "A", "link": "", "date": "2026-03" }
    },
    {
      "op": "add_course_teacher_review",
      "course_name": "AUTO1001",
      "teacher_name": "王老师",
      "content": "讲得很好",
      "author": { "name": "A", "link": "", "date": "2026-03" }
    },
    {
      "op": "append_course_section_item",
      "course_name": "AUTO1001",
      "section_title": "exam",
      "item": {
        "content": "考试经验",
        "author": { "name": "A", "link": "", "date": "2026-03" }
      }
    }
  ]
}
```

## Authentication

All endpoints require:

- `Authorization: Bearer <PR_SERVER_TOKEN>`

`pr-server` should be deployed on an internal network (or behind a gateway). Token should be managed by infra.

## Versioning

- Base path: `/v1`
- Breaking changes bump major version in path (e.g. `/v2`).

## Core concepts

### Target (course locator)

Requests locate a course by `campus + course_code` (and optional `course_name`).

```json
{
  "target": { "campus": "shenzhen", "course_code": "COMP1011", "course_name": "Intro" }
}
```

`pr-server` maps it to a GitHub repo and TOML path using config (recommended):

- `org` - primary GitHub organization
- `cache_org` - fallback organization for read-only queries (default: same as `org`)
- `repo`
- `ref` (default `main`)
- `toml_path` (default `readme.toml` for per-course repos; may be `courses/<code>/readme.toml` for mono repos)

#### Cache org (fallback)

When querying course data, pr-server first tries the primary `org`. If the repository doesn't exist there, it falls back to `cache_org` for read-only operations.

- **Preview**: Reads from cache_org if not found in primary org
- **Submit**: Always writes to primary org (not cache_org)

#### Missing target bootstrap

- **Shenzhen**: missing target falls back to `hoa-cache`, path pattern `<course_code>-<course_name>/readme.toml`; if still missing, pr-server bootstraps a minimal normal TOML in-memory for preview/submit.
- **Weihai / Harbin**: mono repos `DATA_WH` / `DATA_HB` themselves act as cache; when `courses/<code>/readme.toml` is missing, pr-server bootstraps minimal normal TOML in-memory on the resolved path.

### Ops (structured edits)

`ops` is an ordered list. Each op is applied to TOML before rendering README.

#### 1. add_lecturer_review

Add a lecturer review to the course:

```json
{
  "op": "add_lecturer_review",
  "lecturer_name": "郑宜峰",
  "content": "挺好的老师，交流时感觉很亲切。",
  "author": { "name": "xxx", "link": "xxx", "date": "2026-03" }
}
```

#### 2. add_section_item

Add a section item (e.g., exam info, materials):

```json
{
  "op": "add_section_item",
  "title": "关于考试",
  "content": "考试难度不大，平均分80左右。",
  "author": { "name": "lmh", "link": "https://github.com/lmh12138", "date": "" }
}
```

All `author` fields are optional:
- `name`: Author name
- `link`: Author link (e.g., GitHub profile)
- `date`: Date in YYYY-MM format

#### Lecturers schema (breaking constraint)

TOML **MUST** use the new lecturers schema:

```toml
[lecturers]

[[lecturers.items]]
name = "郑宜峰"

[[lecturers.items.reviews]]
content = "..."
# author = { name = "...", link = "...", date = "..." }
```

Legacy TOML `[[lecturers]]` / `[[lecturers.reviews]]` is **NOT** supported.

## Agent-backend adapter mapping

agent-backend should expose these as **skills** (client-facing), and forward to the following pr-server REST endpoints:

| Client-facing skill (agent-backend) | Async | pr-server endpoint | Notes |
|---|---:|---|---|
| `pr.preview` | no | `POST /v1/course:preview` | Preview TOML/README without PR |
| `pr.submit` | yes | `POST /v1/course:submit` | Create branch/commit/PR; agent-backend wraps into jobs |
| `pr.lookup` | no | `GET /v1/pr:lookup` | Optional PR state lookup |

 The app/client should **only** call agent-backend’s skills API.

## Endpoints

### Health

`GET /health`

Response:

```json
{ "ok": true, "data": { "status": "ok" } }
```

### Preview (no PR)

`POST /v1/course:preview`

Use this to validate ops and preview the resulting `readme.toml` and `README.md` without creating a PR.

Request:

```json
{
  "target": { "campus": "shenzhen", "course_code": "COMP1011" },
  "ops": []
}
```

Response (example):

```json
{
  "ok": true,
  "data": {
    "base": {
      "org": "HITSZ-OpenAuto",
      "repo": "COMP1011",
      "ref": "main",
      "toml_path": "readme.toml"
    },
    "result": {
      "readme_toml": "...",
      "readme_md": "..."
    },
    "summary": {
      "changed_files": ["readme.toml", "README.md"],
      "warnings": []
    }
  }
}
```

### Submit (create branch/commit/PR)

`POST /v1/course:submit`

This endpoint performs the write workflow and opens a GitHub PR.

409 conflict covers two server-side cases:

- `REQUEST_IN_PROGRESS`: same `campus + course_code + idempotency_key` is still running.
- `BRANCH_EXISTS`: generated head branch already exists.

Request:

```json
{
  "target": { "campus": "shenzhen", "course_code": "COMP1011" },
  "ops": [],
  "pr": {
    "title": "docs: update course review",
    "body": "Automated update.",
    "labels": ["auto"],
    "draft": false
  },
  "idempotency_key": "<uuid-or-content-hash>"
}
```

Response:

```json
{
  "ok": true,
  "data": {
    "base": { "org": "HITSZ-OpenAuto", "repo": "COMP1011", "ref": "main" },
    "commit": { "sha": "<sha>" },
    "pr": {
      "url": "https://github.com/<org>/<repo>/pull/123",
      "number": 123,
      "head_branch": "auto/COMP1011/20260314-abcdef",
      "base_branch": "main"
    }
  }
}
```

### Lookup PR (optional)

`GET /v1/pr:lookup?org=...&repo=...&number=123`

Response:

```json
{
  "ok": true,
  "data": {
    "state": "open",
    "url": "...",
    "merged": false,
    "checks": {
      "status": "in_progress",
      "conclusion": null
    }
  }
}
```

## Error model

All errors use:

```json
{
  "ok": false,
  "error": {
    "code": "TOML_SCHEMA_ERROR",
    "message": "...",
    "details": {}
  }
}
```

Common error codes currently returned by pr-server:

- `INVALID_JSON`, `MISSING_TARGET`, `MISSING_KEYWORD`, `MISSING_PARAMS`, `INVALID_NUMBER`
- `MISSING_IDEMPOTENCY_KEY`, `REQUEST_IN_PROGRESS`, `BRANCH_EXISTS`
- `REPO_NOT_FOUND`, `TOML_NOT_FOUND`, `PR_NOT_FOUND`
- `TOML_SCHEMA_ERROR`, `RENDER_FAILED`, `CONFIG_ERROR`, `GITHUB_ERROR`, `INVALID_OPS`

Suggested error codes:

- `INVALID_INPUT`
- `COURSE_NOT_FOUND`
- `GITHUB_AUTH_FAILED`
- `GITHUB_NOT_FOUND`
- `TOML_PARSE_ERROR`
- `TOML_SCHEMA_ERROR` (e.g. legacy lecturers)
- `RENDER_FAILED`
- `PR_CREATE_FAILED`
- `IDEMPOTENCY_CONFLICT`

## OpenAPI

See `openapi.yaml`.
