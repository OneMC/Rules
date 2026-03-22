# Refactor Prompt

## 作用
指导 AI 在项目重构时做出正确决策：直接修正 vs 兼容旧方案。

## 配置方式
在 AI 工具中配置为重构场景下的上下文提示，或与 `dev_rules.prompt.md` 同时使用。

## 何时激活
- 发现代码设计缺陷时
- 需要调整数据库表结构时
- 修改接口/方法签名时
- 删除冗余代码层时

## 核心指令

### 1. 零兼容负担原则
**项目处于开发阶段，无线上用户，不需要兼容旧方案。**

### 2. 决策矩阵

| 场景 | 正确做法 | 禁止做法 |
|------|----------|----------|
| 发现设计缺陷 | 直接修正最优方案 | 叠补丁兼容旧代码 |
| 数据库结构调整 | 直接修改 CREATE TABLE | 写 ALTER TABLE 迁移脚本 |
| 接口签名变更 | 直接修改，批量更新调用方 | 保留 @available 旧接口 |
| 冗余代码层 | 直接删除/合并 | 保留过渡期 |

### 3. 执行流程

**步骤 1：识别重构范围**
```
分析影响范围 → 列出所有调用点 → 准备批量修改
```

**步骤 2：直接修改（不兼容）**
```swift
// ❌ 禁止：保留旧接口
@available(*, deprecated)
func oldMethod() { newMethod() }

// ✅ 正确：直接替换
func newMethod() { }
// 然后批量替换所有调用点（同一 PR）
```

**步骤 3：同步更新所有依赖**
- 同一 PR 内完成所有调用点修改
- 确保编译通过
- 无需 deprecated 标记

### 4. 数据库重构示例

```swift
// ❌ 禁止：迁移脚本
ALTER TABLE users ADD COLUMN age INTEGER;

// ✅ 正确：直接修改建表语句
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name TEXT,
    age INTEGER  // 直接添加
);
// 重新安装 App 即可
```

### 5. 检查清单
重构完成后验证：
- [ ] 已删除所有旧代码，无冗余保留
- [ ] 所有调用方已同步更新（同一 PR）
- [ ] 无 `@available(*, deprecated)` 标记
- [ ] 符合 dev_rules.md 编码规范

## 参考文档
详细规则见同目录 `refactor.md`
