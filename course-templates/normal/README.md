# Normal Course Template

适用于**单一课程**的仓库，每门课程一个独立仓库。

## 目录结构

```
COURSE001/
├── readme.toml    # 课程数据
├── README.md      # 自动生成 (可选)
└── materials/     # 课程资料 (可选)
    └── ...
```

## 字段说明

| 字段 | 必填 | 说明 |
|------|------|------|
| `course_name` | ✅ | 课程中文名称 |
| `course_code` | ✅ | 课程代码 (唯一标识) |
| `repo_type` | ✅ | 固定为 `"normal"` |
| `description` | ❌ | 课程简介 |
| `sections` | ❌ | 章节列表 |
| `lecturers` | ❌ | 教师评价 |

## sections 字段结构

```toml
[[sections]]
title = "章节标题"

[[sections.items]]
content = "章节内容"
```

## lecturers 字段结构 (新 schema)

顺序: `lecturers.intro` → `lecturers.items` → `lecturers.summary`

```toml
[lecturers]

# 教师简介 (在教师列表之前)
[[lecturers.intro]]
content = "教师团队介绍"

[[lecturers.items]]
name = "教师姓名"

[[lecturers.items.reviews]]
content = "评价内容"
author = { name = "昵称", link = "链接", date = "2025-03" }

# 教师总结 (在教师列表之后)
[[lecturers.summary]]
content = "教师团队总结"
```

**注意**: 使用 `[[lecturers.items]]` 而非旧的 `[[lecturers]]`

## 示例

```toml
course_name = "计算机视觉"
course_code = "COMP3015"
repo_type = "normal"

description = """
计算机视觉课程介绍。
"""

[[sections]]
title = "课程大纲"
[[sections.items]]
content = """
1. 图像处理基础
2. 特征提取
3. 目标检测
4. 图像分割
"""

[lecturers]

[[lecturers.items]]
name = "张三"

[[lecturers.items.reviews]]
content = """
老师讲得很清楚！
"""
author = { name = "学生A", date = "2025-03" }
```

## 使用方式

1. 复制 `readme.toml` 到新仓库
2. 修改 `course_name`, `course_code` 等字段
3. 使用 `/pr` 命令添加内容
