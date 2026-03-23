---
name: doc-management
description: 文档管理规范执行。当需要创建、修改、删除项目文档，或检查文档规范性时激活。负责判定文档类别、执行文档操作流程、维护双向链接、确保符合 document_rules.md 规范。
---

# 文档管理 Skill

执行项目文档标准化管理流程，确保文档分类正确、操作合规、关联完整。

---

## 标准文档 Frontmatter

所有文档必须在开头包含 YAML frontmatter：

```yaml
---
title: 文档标题                    # 简短明确的功能/模块名称
type: prd                          # prd | arc | dev | next | todo | architecture
category: feature                  # feature | infrastructure | process
status: draft                      # draft | active | deprecated | archived
created: 2026-03-23
updated: 2026-03-23
tags: [标签1, 标签2]               # 用于快速检索和分类
related: [相关文档1, 相关文档2]     # Obsidian 双向链接，不含.md后缀
description: 一句话描述本文档的核心内容
---
```

### Type 定义

| Type | 用途 | 存放路径 |
|------|------|---------|
| `prd` | 产品需求文档 | `Doc/{功能名}_prd.md` |
| `arc` | 架构设计文档 | `Doc/{功能名}_arc.md` 或 `Doc/Architecture/*.md` |
| `dev` | 开发实现文档 | `Doc/{功能名}_dev.md` |
| `next` | 下一步计划 | `Doc/{功能名}_next.md` |
| `todo` | 全局待办 | `Doc/_todo.md` |
| `architecture` | 基础架构文档 | `Doc/Architecture/*.md` |

### Category 定义

| Category | 说明 |
|----------|------|
| `feature` | 功能相关文档 |
| `infrastructure` | 基础架构文档 |
| `process` | 流程/规范类文档 |

---

## 操作流程（强制顺序执行）

### 步骤 1：判定文档类别

根据用户意图，确定文档类别：

| 判定条件 | Category | Type | 路径 |
|---------|----------|------|------|
| 影响全局基础设施（数据库、网络、存储等） | infrastructure | architecture | `Doc/Architecture/` |
| 描述"要做什么"、需求背景 | feature | prd | `Doc/` |
| 描述技术方案、模块设计 | feature | arc | `Doc/` |
| 记录实现问题和解决方案 | feature | dev | `Doc/` |
| 记录未完成工作 | feature | next | `Doc/` |
| 全局待办 | process | todo | `Doc/` |

**基础架构先行原则**：功能设计前，必须先查阅相关基础架构文档。

---

### 步骤 2：扫描文档库

**目的**：发现相关文档、检查冲突、建立上下文。

**执行**：
1. 根据确定的路径，扫描目标文件夹及其子文件夹
2. 读取所有文档的 frontmatter（不读正文）
3. 通过 `title`、`tags`、`description` 筛选相关文档
4. 检查 `related` 字段，识别关联关系

**输出**：相关文档清单（含 type、status、description）

---

### 步骤 3A：新建文档流程

**必须回答**："为什么该文档不存在？"

有效理由：
- 该功能首次进入此阶段
- 前期未文档化，现补建
- 功能拆分后独立出来

**执行**：
1. 创建文件，添加标准 frontmatter
2. 根据 type 选择模板填充内容
3. 建立 Obsidian 双向链接（`related` 字段 + 正文 `[[文件名]]`）
4. 如有影响其他功能，添加到 `related` 并准备同步更新

---

### 步骤 3B：修改文档流程

**必须回答**："为什么需要修改？当前存在什么问题？"

有效理由：
- 需求变更：新增验收标准
- 架构调整：接口方案变化
- 开发踩坑：记录问题解决方案

**执行**：
1. 读取现有文档 frontmatter
2. 更新 `updated` 时间
3. 如 status 变化（draft → active），同步更新
4. 如影响范围变化，更新 `related` 字段
5. 执行内容修改
6. **评估影响范围**：如涉及基础架构变更，列出所有受影响功能

---

## 关键检查点

任何文档操作后，执行自检：

- [ ] frontmatter 是否完整（6 个必需字段）？
- [ ] `type` 和 `category` 是否正确？
- [ ] `related` 是否包含所有关联文档？
- [ ] 正文是否有对应的 `[[文件名]]` 双向链接？
- [ ] 如涉及修改，是否评估了影响范围？

---

## 关联 Skill

- **prd**：产品需求编写（调用本 skill 进行文档创建/更新）
- **arc**：架构设计编写（调用本 skill 进行文档创建/更新）
- **dev**：开发文档编写（调用本 skill 进行文档创建/更新）
