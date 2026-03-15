# HITSZ Course Bot

HITSZ 课程助手 for [AstrBot](https://github.com/AstrBotDevs/AstrBot) - 课程查询、RAG 问答、PR 提交闭环。

## 功能

| 命令 | 功能 |
|------|------|
| `/搜 <关键词>` | 模糊搜索课程 |
| `/查 <课程代码/名称/昵称>` | 查询课程详情 |
| `/查老师 <教师名>` | 查询教师评价 |
| `/设置昵称 <昵称> <课程代码>` | 设置课程昵称 |
| `/刷` | 从 GitHub 同步课程数据 |
| `/问 <问题>` | RAG 知识库问答 |
| `/重构知识库` | 重建 RAG 索引 |
| `/pr start <课程>` | 开始 PR 提交流程 |
| `/pr submit` | 提交 PR |

## 快速开始

### 1. 安装

```bash
# 克隆仓库
git clone https://github.com/HIT-A/bot-backend.git
cd bot-backend

# 安装依赖
pip install -e .
```

### 2. 配置

复制 `.env.example` 为 `.env` 并配置：

```bash
# Agent Backend (必填)
HITSZ_COURSE_AGENT_BACKEND_URL=http://localhost:8080
HITSZ_COURSE_AGENT_BACKEND_API_KEY=

# GitHub 配置
HITSZ_COURSE_GITHUB_ORG=HITSZ-OpenAuto
HITSZ_COURSE_GITHUB_TOKEN=

# LLM 配置 (可选)
HITSZ_COURSE_AI_API_KEY=
HITSZ_COURSE_AI_BASE_URL=
HITSZ_COURSE_AI_MODEL=
```

### 3. 使用

```bash
# 启动 AstrBot
astrbot run
```

## 架构

```
┌─────────────────────────────────────────┐
│              AstrBot                     │
│  ┌───────────────────────────────────┐  │
│  │     hitsz_course 插件              │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────┐ │  │
│  │  │ course_  │ │  pr_    │ │rag_ │ │  │
│  │  │ manager  │ │ entry   │ │engine│ │  │
│  │  └────┬────┘ └────┬────┘ └──┬──┘ │  │
│  └───────┼───────────┼──────────┼────┘  │
│          │           │          │        │
│  ┌───────┴──────────┴──────────┴────┐  │
│  │         data/ (共享课程数据)        │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
           │              │
    ┌──────┴──────┐  ┌───┴────────┐
    │ agent-      │  │ GitHub Org  │
    │ backend     │  │ (课程仓库)  │
    │ (REST API)  │  └────────────┘
    └──────┬──────┘
           │
    ┌──────┼──────┐
    ▼      ▼      ▼
 Qdrant  COS   pr-server
```

## 依赖

- Python 3.10+
- aiohttp
- tomlkit
- pygithub

## License

AGPL-3.0
