# Rules

iOS 项目开发规则文档

---

## 硬链接管理

本项目文档可能需要在多个位置使用硬链接同步，以下是常用操作：

### 创建硬链接

```bash
# 基本语法
ln <源文件> <目标路径>

# 示例：将 dev_rules.md 硬链接到项目目录
ln /Users/mc/Desktop/Rules/dev_rules.md /Users/mc/Project/iOS-Project/dev_rules.md

# 示例：硬链接 refactor.md
ln /Users/mc/Desktop/Rules/refactor.md /Users/mc/Project/iOS-Project/refactor.md
```

### 查看硬链接

```bash
# 查看文件的 inode 和链接数
ls -li <文件路径>

# 示例
ls -li dev_rules.md
# 输出示例：
# 12345678 -rw-r--r-- 2 user staff 4316 Mar 22 23:30 dev_rules.md
#         ↑ 链接数为 2，表示有两个硬链接指向同一文件

# 查找所有指向同一 inode 的文件
find <搜索目录> -inum <inode号>

# 示例
find /Users/mc -inum 12345678
```

### 删除硬链接

```bash
# 删除硬链接（语法与普通文件相同）
rm <文件路径>

# 示例：删除硬链接（不影响其他链接）
rm /Users/mc/Project/iOS-Project/dev_rules.md

# 注意：当所有硬链接都被删除后，文件内容才真正删除
# 只要还有一个硬链接存在，文件内容就保留
```

### 检查清单

- [ ] 创建后使用 `ls -li` 验证 inode 和链接数
- [ ] 修改任一硬链接，验证其他链接是否同步更新
- [ ] 确认删除的是链接而非源文件（除非都要删除）
