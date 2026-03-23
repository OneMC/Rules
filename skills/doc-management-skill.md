---
name: doc-management
description: 文档管理规范执行。当需要创建、修改、删除项目文档，或检查文档规范性时激活。负责判定文档类别、执行文档操作流程、确保符合 document_rules.md 规范。
---

# 文档管理 Skill

执行项目文档标准化管理流程，确保文档分类正确、操作合规。

详见 `document_rules.md` 完整规范。

---

## 快速参考：Frontmatter 标准

所有文档必须在开头包含 YAML frontmatter：

```yaml
---
title: 文档标题
type: prd                          # prd | arc | dev | next | todo | architecture | 自定义
created: 2026-03-23
updated: 2026-03-23
description: 简短描述核心重点
---
```

### 常用 Type

| Type | 用途 | 存放路径 |
|------|------|---------|
| `prd` | 产品需求文档 | `Doc/{功能名}_prd.md` |
| `arc` | 架构设计文档 | `Doc/{功能名}_arc.md` |
| `dev` | 开发实现文档 | `Doc/{功能名}_dev.md` |
| `next` | 下一步计划 | `Doc/{功能名}_next.md` |
| `todo` | 全局待办 | `Doc/_todo.md` |
| `architecture` | 基础架构文档 | `Doc/Architecture/*.md` |

---

## 快速参考：操作流程

### 步骤 1：判定文档类别

| 判定条件 | Type | 路径 |
|---------|------|------|
| 影响全局基础设施 | architecture | `Doc/Architecture/` |
| 描述"要做什么" | prd | `Doc/` |
| 描述技术方案 | arc | `Doc/` |
| 记录实现问题 | dev | `Doc/` |
| 记录未完成工作 | next | `Doc/` |
| 全局待办 | todo | `Doc/` |

### 步骤 2：扫描文档库

1. 根据路径扫描文件夹及其子文件夹
2. 读取 frontmatter（通过 `title`、`type`、`description` 筛选）
3. 识别相关文档和潜在冲突

### 步骤 3：新建/修改/删除

**新建**：
1. 添加标准 frontmatter
2. 在正文建立 Obsidian 双向链接 `[[文件名]]`

**修改**：
1. 更新 `updated` 时间
2. 执行内容修改
3. 评估影响范围

**删除**：
1. 确认无用或已被替代
2. 检查并更新引用链接
3. 直接删除（历史在 Git 中）

---

## 关键检查点

- [ ] frontmatter 完整（4 个必需字段）？
- [ ] `type` 正确？
- [ ] 正文有 `[[文件名]]` 双向链接？
- [ ] 修改时评估了影响范围？

---

## 关联文档

- **document_rules.md** —— 完整规范
- **prd-skill.md** —— PRD 编写流程
