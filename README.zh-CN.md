# read-later-skill

![read-later-skill cover](./assets/read-later-skill-cover.png)

[English](./README.md) | [中文](./README.zh-CN.md)

这是一个“稍后阅读”技能仓库，用来定义如何把用户在聊天或 Agent 环境中发来的文章链接进行摘要、分类、去重，并写入 MongoDB，方便后续检索和追问。

当前仓库主要承载的是技能说明与参考文档，适合作为 Agent Skill / Prompt 规范仓库使用。

## 项目目标

- 接收用户在聊天或 Agent 环境中发送的文章链接
- 提取并整理文章摘要、标签、主题、备注等结构化信息
- 基于 `canonical_url` 和 `url_hash` 做去重
- 将文章保存到固定集合 `readlater.articles`
- 支持后续按时间、主题、站点、等关键词进行查询与追问

## 仓库结构

```text
.
├── SKILL.md
└── references/
    ├── mongodb-collection-schema.md
    ├── reply-templates.md
    └── smoke-test.md
```

### 文件说明

- `SKILL.md`：主技能说明，定义适用场景、写入流程、检索流程、返回格式与安全规则
- `references/mongodb-collection-schema.md`：MongoDB 集合结构、索引建议、查询约束
- `references/reply-templates.md`：保存成功、回答问题、低置信度回退时的回复模板
- `references/smoke-test.md`：在本地对话窗口或任意 Agent 聊天环境中验证技能可用性的 5 步冒烟测试

## 核心工作流

### 1. 收录文章

当用户发送文章链接后，技能应按以下顺序处理：

1. 规范化 URL，移除明显追踪参数
2. 生成 `url_hash`
3. 先按 `canonical_url`、再按 `url_hash` 去重
4. 生成结构化文档
5. 写入 MongoDB 或更新允许更新的字段
6. 返回简短确认信息、标签、原文链接和记录 ID

建议写入字段包括：

- `title`
- `source`
- `original_url`
- `canonical_url`
- `summary`
- `key_points`
- `tags`
- `topic`
- `language`
- `user_note`
- `saved_at`
- `last_updated_at`

### 2. 查询与追问

当用户问“之前那篇文章说了什么”这类问题时，技能应：

1. 解析时间范围、主题关键词、来源站点等条件
2. 在 `readlater.articles` 中过滤候选记录
3. 按主题匹配度与时间排序
4. 优先基于已存储的 `summary`、`key_points`、`evidence_snippets` 作答
5. 如果命中不确定，返回候选项并追问一个澄清条件

## MongoDB 约定

本项目固定使用：

- 数据库：`readlater`
- 集合：`articles`

核心字段示例见 `references/mongodb-collection-schema.md`，其中几个关键约束如下：

- 写入对象固定为 `readlater.articles`
- 去重优先级：`canonical_url` -> `url_hash`
- 建议使用 `saved_at` 作为时间过滤字段
- 建议使用 `topic` + `tags` 做主题检索
- 写入安全过滤条件建议带上 `created_by = "read-later-skill"`

推荐索引：

```javascript
db.articles.createIndex({ url_hash: 1 }, { unique: true })
db.articles.createIndex({ canonical_url: 1 })
db.articles.createIndex({ saved_at: -1 })
db.articles.createIndex({ topic: 1, saved_at: -1 })
db.articles.createIndex({ tags: 1 })
db.articles.createIndex({ created_by: 1, saved_at: -1 })
```

## 回复约定

为了让交互输出稳定一致，仓库提供了回复模板：

- 保存成功：返回一句话总结、标签、原文链接、记录 ID
- 问答响应：返回结论、依据、原文链接、记录 ID
- 低置信度回退：返回候选文章列表，并引导用户补充条件

详见 `references/reply-templates.md`。

## 冒烟测试

可以直接在本地对话窗口或其他 Agent 聊天环境中按 `references/smoke-test.md` 进行 5 步验证：

1. 搜索最近保存的文章
2. 拉取某条记录详情
3. 创建一条测试文章记录
4. 更新测试记录状态与备注
5. 反查这条测试文章的摘要与来源

如果以上流程都能跑通，说明这个技能的基础收录与检索链路可用。

## 适用场景

- 个人稍后读文章收集
- 聊天式 Agent 的文章收藏能力
- 基于 MongoDB 的轻量知识留存与检索
- 为后续“文章问答”提供结构化数据基础
