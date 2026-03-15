# HIT-Course 部署指南

## 项目概述

| 仓库 | 说明 | 地址 |
|------|------|------|
| bot-backend | AstrBot 插件（课程+RAG+PR） | https://github.com/HIT-A/bot-backend |
| repos-management | 威海/本部课程仓库工作流 | https://github.com/HIT-A/repos-management |
| data-backend | Go 后端服务（pr-server） | https://github.com/HIT-A/data-backend |
| api_bundle | API 文档集合 | https://github.com/HIT-A/api_bundle |

## 架构

```
┌─────────────────────────────────────────────────┐
│                  服务器                          │
│  ┌─────────────────────────────────────────┐  │
│  │              AstrBot                     │  │
│  │  ├── hit_course (课程/PR/RAG)           │  │
│  │  ├── mnemosyne (记忆库)                 │  │
│  │  ├── parser (链接解析)                  │  │
│  │  └── portrayal (人物画像)                │  │
│  └─────────────────────────────────────────┘  │
│                      │                          │
│  ┌──────────────────┼──────────────────┐      │
│  ▼                  ▼                    ▼      │
│ Milvus Lite    agent-backend      pr-server   │
│ (记忆库)        (RAG/AI)          (PR提交)    │
│                                             │
└─────────────────────────────────────────────┘
         │                  │              │
         ▼                  ▼              ▼
       Qdrant        GitHub API      GitHub
    (知识库向量)                        (课程仓库)
```

## 部署步骤

### 1. 安装 AstrBot

```bash
# 方式一：Docker (推荐)
docker run -d --name astrbot -p 8080:8080 -v astrbot_data:/app/data astrbot/astrbot:latest

# 方式二：pip
pip install astrbot
astrbot init
astrbot run
```

### 2. 安装 bot-backend 插件

```bash
cd /path/to/astrbot/data/plugins
git clone https://github.com/HIT-A/bot-backend.git hit_course

# 安装依赖
cd hit_course
pip install -e .

# 重启 AstrBot
```

### 3. 安装 mnemosyne (可选)

```bash
cd /path/to/astrbot/data/plugins
git clone https://github.com/lxfight/astrbot_plugin_mnemosyne

# 安装依赖
pip install pymilvus milvus-lite
```

### 4. 安装 parser (可选)

```bash
cd /path/to/astrbot/data/plugins
git clone https://github.com/Zhalslar/astrbot_plugin_parser
```

### 5. 安装 portrayal (可选)

```bash
cd /path/to/astrbot/data/plugins
git clone https://github.com/Zhalslar/astrbot_plugin_portrayal
```

### 6. 配置环境变量

创建 `.env` 文件：

```bash
# Agent Backend (必填)
HITSZ_COURSE_AGENT_BACKEND_URL=http://your-agent-backend:8080
HITSZ_COURSE_AGENT_BACKEND_API_KEY=your_api_key

# GitHub (用于拉取课程数据)
HITSZ_COURSE_GITHUB_ORG=HITSZ-OpenAuto
HITSZ_COURSE_GITHUB_TOKEN=your_github_token

# LLM (可选)
HITSZ_COURSE_AI_API_KEY=your_llm_key
HITSZ_COURSE_AI_BASE_URL=https://api.n1n.ai/v1
HITSZ_COURSE_AI_MODEL=gemini-2.5-pro
```

### 7. 配置 pr-server (Go 服务)

```bash
# 克隆
git clone https://github.com/HIT-A/data-backend.git
cd data-backend/services/pr-server

# 配置 config.yaml
cp config.example.yaml config.yaml
# 编辑 config.yaml，设置 campus、org、cache_org 等

# 运行
go run cmd/pr-server/main.go
# 或编译运行
go build -o pr-server cmd/pr-server/main.go
./pr-server
```

### 8. 配置 agent-backend (可选)

```bash
git clone https://github.com/HIT-A/agent-backend
# 按文档配置
```

## 课程仓库支持

| 校区 | 组织 | 模式 | 工作流 |
|------|------|------|--------|
| 深圳 | HITSZ-OpenAuto | 单课程单仓库 | HITSZ-OpenAuto/repos-management |
| 威海 | HIT-A | Mono-repo | HIT-A/repos-management |
| 本部 | HIT-A | Mono-repo | HIT-A/repos-management |

## 常用命令

```bash
# 搜课程
/搜 Python

# 查课程
/查 COMP1011

# 查老师
/查老师 张三

# RAG 问答
/问 这门课怎么样

# PR 提交
/pr start COMP1011
/pr add 考核方式 平时30%期末70%
/pr addreview 张三 老师很nice
/pr submit
```

## 维护

- **更新插件**: `git pull` 在插件目录
- **重启 AstrBot**: `astrbot restart` 或重启容器
- **查看日志**: `astrbot logs` 或 `docker logs astrbot`
