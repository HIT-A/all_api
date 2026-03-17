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
