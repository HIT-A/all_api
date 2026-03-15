# Multi-Project Course Template

适用于**多门相关课程**放在同一个仓库的场景，如系列课程、模块化课程。

## 目录结构

```
MultiCourseRepo/
├── readme.toml    # 所有课程数据
├── README.md      # 自动生成
└── materials/     # 课程资料 (可选)
    ├── course1/
    └── course2/
```

## 适用场景

- 同一老师的系列课程
- 同一门课的不同学期/班级
- 模块化课程 (如 "计算机基础" 包含多个子模块)
- 归类在一起的 相关课程

## 字段说明

| 字段 | 必填 | 说明 |
|------|------|------|
| `course_name` | ✅ | 仓库/系列名称 |
| `course_code` | ✅ | 主课程代码 (父级) |
| `repo_type` | ✅ | 固定为 `"multi-project"` |
| `description` | ❌ | 仓库整体简介 |
| `courses` | ✅ | 子课程列表 |
| `sections` | ❌ | 仓库级别通用章节 |

## courses 子课程结构

```toml
[[courses]]
code = "COURSE001"       # 子课程代码 (唯一)
name = "课程名称"         # 子课程名称
description = """..."   # 可选

# 子课程章节 (与 normal 相同)
[[courses.sections]]
title = "章节标题"
[[courses.sections.items]]
content = "内容"

# 子课程教师 (与 normal 相同)
[[courses.teachers]]
name = "教师名"

[[courses.teachers.reviews]]
content = "评价"
author = { name = "昵称", date = "2025-03" }
```

## 与 normal 的区别

| 特性 | normal | multi-project |
|------|--------|---------------|
| 课程数 | 1 | 多 |
| 查询方式 | `/查 CODE` | `/查 子课程名` |
| repo_type | `"normal"` | `"multi-project"` |
| 章节层级 | 顶层 `sections` | `courses[].sections` |
| 教师层级 | 顶层 `lecturers` | `courses[].teachers` |

## 示例

```toml
course_name = "计算机基础系列"
course_code = "CS1000"
repo_type = "multi-project"

description = """
计算机基础系列课程仓库。
"""

# 子课程 1
[[courses]]
code = "CS1001"
name = "程序设计基础"

[[courses.sections]]
title = "内容"
[[courses.sections.items]]
content = "C 语言基础"

[[courses.teachers]]
name = "张老师"

[[courses.teachers.reviews]]
content = "讲得很清楚！"
author = { name = "学生A", date = "2025-03" }

# 子课程 2
[[courses]]
code = "CS1002"
name = "数据结构"

[[courses.sections]]
title = "内容"
[[courses.sections.items]]
content = "链表、树、图"

[[courses.teachers]]
name = "李老师"

[[courses.teachers.reviews]]
content = "作业有点多"
author = { name = "学生B", date = "2025-03" }
```

## 使用方式

1. 复制 `readme.toml` 到新仓库
2. 修改 `course_name`, `course_code`
3. 添加 `[[courses]]` 块定义子课程
4. 使用 `/查 <子课程名>` 查询
5. 使用 `/pr` 命令添加内容
