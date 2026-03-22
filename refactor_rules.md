# 重构规则

> **适用前提**：项目处于开发阶段，无线上用户和历史数据负担。

## 核心原则

| 原则 | 说明 |
|------|------|
| **零兼容负担** | 不需要兼容旧数据、旧接口、旧格式 |
| **直接修正** | 发现设计缺陷直接改，不叠补丁 |
| **批量更新** | 改接口时同步更新所有调用方（同一 PR） |

## AI 执行规则

### 禁止事项
- ❌ 保留旧接口做兼容（`@available(*, deprecated)`）
- ❌ 写数据库迁移脚本（`ALTER TABLE`）
- ❌ 保留冗余代码层

### 必须事项
- ✅ 直接按最优方案修改
- ✅ 直接修改 `CREATE TABLE` 语句，删表重建
- ✅ 直接改接口签名，批量替换调用方
- ✅ 直接删除冗余代码

## 示例

```swift
// ❌ 禁止：保留旧接口
@available(*, deprecated)
func oldMethod() { newMethod() }

// ✅ 正确：直接替换，批量更新调用方
func newMethod() { }
```

```sql
-- ❌ 禁止：迁移脚本
ALTER TABLE users ADD COLUMN age INTEGER;

-- ✅ 正确：直接修改建表语句
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name TEXT,
    age INTEGER  -- 直接加字段
);
```

## 检查清单

重构后验证：
- [ ] 已删除所有旧代码，无冗余保留
- [ ] 所有调用方已同步更新
- [ ] 无 `@available(*, deprecated)`
- [ ] 符合 dev_rules.md 规范
