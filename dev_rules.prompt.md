# Dev Rules Prompt

## 作用
指导 AI 在 iOS 项目开发中严格遵守编码规范，包括技术栈选择、日志规范、错误处理等。

## 配置方式
在 AI 工具（如 Cursor、Claude Code、Kimi 等）的设置中，将本文件添加为：
- **Context Rules** / **系统提示词** 
- 或放在项目根目录的 `.cursorrules`、`.claude.md` 等位置

## 何时激活
- 生成新代码时
- 修改现有代码时
- 代码审查/重构时

## 核心指令

### 1. 强制前置检查
每次生成/修改代码前，必须在内心确认：
```
□ 使用 UIKit + SnapKit，非 SwiftUI/Storyboard/XIB
□ 不修改 .pbxproj / Podfile.lock / Pods/
□ Logger 使用 extension Logger 定义（os.log）
□ Guard 语句分行写
□ try-catch 包含 assertionFailure + Logger.error
```

### 2. 代码生成规范

**文件头部必须包含：**
```swift
import UIKit
import os.log

// MARK: - Logger
extension Logger {
    private static let subsystem = "com.app.<模块名>"
    static let <category> = Logger(subsystem: subsystem, category: "<Category>")
}
```

**日志格式要求：**
- 每条日志：`[方法名] 描述, 关键数据`
- 示例：`Logger.network.log("[fetchUser] start, userId: \(userId)")`

**Guard 语句：**
```swift
// ✅ 必须分行
guard let self = self else {
    Logger.<category>.error("[<method>] self 已释放")
    return false
}
```

**异常处理：**
```swift
do {
    try operation()
} catch {
    assertionFailure("[<method>] 失败: \(error)")
    Logger.<category>.error("[<method>] error: \(error)")
}
```

### 3. 禁止事项
- 禁止生成 SwiftUI、Storyboard、XIB 代码
- 禁止修改工程配置文件
- 禁止多个 Subsystem（统一 Subsystem，不同 Category）
- 禁止单行 Guard 语句
- 禁止 catch 块只记录日志而不调用 assertionFailure

### 4. 自检清单
生成代码后，AI 必须自我验证：
- [ ] UIKit + SnapKit
- [ ] extension Logger 定义正确
- [ ] 日志包含 [方法名] 和数据
- [ ] Guard 分行且有日志
- [ ] try-catch 有 assertionFailure

## 参考文档
详细规则见同目录 `dev_rules.md`
