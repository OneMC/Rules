# Rules - iOS 项目通用开发规范

本项目作为**中央规则仓库**，通过**硬链接**将规范分发到各个 iOS 项目中，实现统一管理和同步更新。

---

## 使用方式

### 1. 初始化项目（创建硬链接）

将本仓库的规则文件硬链接到目标项目：

```bash
# 进入目标项目目录
cd /Users/mc/Project/Your-iOS-Project

# 创建硬链接（示例）
ln /Users/mc/Desktop/Rules/dev_rules.md ./dev_rules.md
ln /Users/mc/Desktop/Rules/refactor.md ./refactor.md
```

### 2. 批量创建脚本

```bash
#!/bin/bash
# setup-rules.sh - 在项目初始化时运行

RULES_DIR="/Users/mc/Desktop/Rules"
PROJECT_DIR="$(pwd)"

# 创建硬链接
ln -f "${RULES_DIR}/dev_rules.md" "${PROJECT_DIR}/dev_rules.md"
ln -f "${RULES_DIR}/refactor.md" "${PROJECT_DIR}/refactor.md"

echo "Rules linked successfully!"
ls -li ${PROJECT_DIR}/*.md
```

### 3. 更新规则

修改中央仓库的规则文件，所有硬链接的项目自动同步：

```bash
# 在中央仓库修改规则
cd /Users/mc/Desktop/Rules
vim dev_rules.md
git commit -am "update: xxx"

# 所有项目的 dev_rules.md 自动更新（因为是硬链接）
```

---

## 硬链接管理命令

### 查看硬链接状态

```bash
# 查看文件的 inode 和链接数
ls -li dev_rules.md

# 输出示例：
# 12345678 -rw-r--r-- 3 user staff 4316 Mar 22 23:30 dev_rules.md
#         ↑ 链接数为 3，表示有 3 个位置在使用此规则

# 查找所有指向同一 inode 的文件
find ~ -inum 12345678 2>/dev/null
```

### 删除项目中的规则（取消链接）

```bash
# 在目标项目中删除硬链接（不影响其他项目）
cd /Users/mc/Project/Your-iOS-Project
rm dev_rules.md refactor.md

# 注意：这不会删除中央仓库的文件
```

### 彻底删除规则文件

```bash
# 只有当所有硬链接都删除后，文件才真正删除
# 1. 先找到所有链接位置
ls -li /Users/mc/Desktop/Rules/dev_rules.md
# 记录 inode 号，如：12345678

# 2. 查找并删除所有硬链接
find ~ -inum 12345678 -exec rm {} \;
```

---

## 项目结构

```
Rules/                          # 中央规则仓库
├── dev_rules.md               # iOS 开发规范（AI 执行手册）
├── refactor.md                # 重构规则
└── document_rules.md          # 文档规范

Your-iOS-Project/              # 具体项目（通过硬链接引用规则）
├── Your-iOS-Project.xcworkspace
├── Podfile
├── dev_rules.md  →  (硬链接) →  Rules/dev_rules.md
└── refactor.md   →  (硬链接) →  Rules/refactor.md
```

---

## 注意事项

- **不要直接修改项目中的规则文件**（虽然会同步到中央仓库，但容易混淆）
- **推荐在中央仓库修改规则**，然后 `git commit`
- **硬链接不能跨文件系统**（如外接硬盘）
