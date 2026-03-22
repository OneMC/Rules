# iOS 开发规范 (AI 执行手册)

> **重要提示**：本文件提供**通用 iOS 开发规范**。
> 
> **优先级规则**：
> - 如果项目有自定义日志规范（如 `AI_RULES/dev_rules/dev.md` 中的 Logger 规范），**优先使用项目自定义规范**
> - 本文件的 `extension Logger` 方式仅在项目无自定义日志时使用

---

## 快速决策清单

生成代码前检查：
- [ ] 使用 UIKit + SnapKit，非 SwiftUI/Storyboard/XIB
- [ ] 不修改 `.pbxproj` / `Podfile.lock` / `Pods/`
- [ ] **Logger 优先使用项目自定义规范**（如有），否则使用本文件的 `extension Logger` 方式
- [ ] Guard 语句分行写
- [ ] try-catch 包含 `assertionFailure` + 日志

---

## 一、技术栈与工程约束

| 项目 | 要求 |
|------|------|
| UI | UIKit + SnapKit |
| 依赖 | CocoaPods，只改 `Podfile` |
| 禁止 | SwiftUI, Storyboard, XIB, `.pbxproj`, `Podfile.lock` |

---

## 二、日志规范（通用版）

> **⚠️ 兼容性说明**：
> - **如果项目已有自定义日志库**（如 `AI_RULES/dev_rules/dev.md` 中的 Logger），**优先使用项目自定义规范**
> - 本章节仅作为项目无自定义日志时的**备选方案**

### 2.1 核心思想

- **Subsystem**：大功能模块（如 `com.app.network`）
- **Category**：模块内流程（如 `Request`, `Response`, `Error`）

### 2.2 标准用法（项目无自定义 Logger 时使用）

```swift
import os.log

// 扩展 OSLog.Logger
extension Logger {
    private static let subsystem = "com.app.network"
    
    static let networkRequest  = Logger(subsystem: subsystem, category: "Request")
    static let networkResponse = Logger(subsystem: subsystem, category: "Response")
    static let networkError    = Logger(subsystem: subsystem, category: "Error")
}

// 使用
Logger.networkRequest.log("[fetchUser] start, userId: \(userId)")
```

### 2.3 项目自定义 Logger 适配示例

如果项目使用自定义 Logger（如通过 `import Utils`），按以下方式适配：

```swift
import Utils  // 或项目日志模块

private let subsystem = "com.app.network"

// 使用项目 Logger API（示例，以项目实际 API 为准）
Logger.log("request", subsystem: subsystem) { "[fetchUser] start, userId: \(userId)" }
```

### 2.4 禁止事项

```swift
// ❌ 禁止：多个 Subsystem
Logger(subsystem: "com.app.network.request", category: "Request")

// ✅ 正确：统一 Subsystem，不同 Category
Logger(subsystem: "com.app.network", category: "Request")
```

### 2.5 日志内容要求

每条日志必须包含：`[方法名] 描述, 关键数据`

```swift
// ✅ 正确（os.log 方式）
Logger.network.log("[fetchUser] start, userId: \(userId)")

// ✅ 正确（项目自定义 Logger 方式）
Logger.log("network", subsystem: subsystem) { "[fetchUser] start, userId: \(userId)" }

// ❌ 禁止
Logger.network.log("start")  // 缺少方法名和数据
```

### 2.6 Guard 规范

```swift
// ✅ 必须分行，便于加日志
guard let self = self else {
    // os.log 方式：
    Logger.network.error("[<methodName>] self 已释放")
    // 或项目 Logger 方式：
    // Logger.log("error", subsystem: subsystem) { "[<methodName>] self 已释放" }
    return false
}

// ❌ 禁止
guard let self = self else { return false }
```

---

## 三、assertionFailure 规范（强制）

### 3.1 必须使用场景

| 场景 | 处理方式 |
|------|----------|
| try-catch | catch 中 `assertionFailure` + 日志 |
| 严重 guard 失败 | `assertionFailure` + 降级处理 |
| 状态不一致 | `assertionFailure` 暴露问题 |

### 3.2 标准模式

```swift
// try-catch 场景
do {
    try parse()
} catch {
    assertionFailure("[parse] 失败, data: \(data), error: \(error)")
    
    // os.log 方式：
    Logger.parse.error("[parse] failed: \(error)")
    
    // 或项目自定义 Logger 方式：
    // Logger.log("parse", subsystem: subsystem) { "[parse] failed: \(error)" }
    
    return nil
}

// guard 场景
guard let config = loadConfig() else {
    assertionFailure("[loadConfig] 失败, path: \(path)")
    return defaultConfig
}
```

### 3.3 内容要求

必须包含：`[方法名] 错误原因, 关键数据`

```swift
// ✅ 正确
assertionFailure("[fetchUser] response 为空, userId: \(userId)")

// ❌ 禁止
assertionFailure("出错了")
```

---

## 四、代码生成模板

### 模板 A：使用项目自定义 Logger（推荐，如果项目有自定义 Logger）

```swift
import UIKit
import Utils  // 项目日志模块

// MARK: - Constants
private let subsystem = "com.app.<模块>"

// MARK: - Class
final class <Class>: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        Logger.log("<topic>", subsystem: subsystem) { "[viewDidLoad] start" }
        setupUI()
    }
    
    private func setupUI() {
        // SnapKit layout
    }
    
    func <method>(param: Type) {
        Logger.log("<topic>", subsystem: subsystem) { "[<method>] start, param: \(param)" }
        
        guard let self = self else {
            Logger.log("error", subsystem: subsystem) { "[<method>] self 已释放" }
            return
        }
        
        do {
            try operation()
            Logger.log("<topic>", subsystem: subsystem) { "[<method>] success" }
        } catch {
            assertionFailure("[<method>] failed: \(error)")
            Logger.log("error", subsystem: subsystem) { "[<method>] error: \(error)" }
        }
    }
}
```

### 模板 B：使用标准 os.log（项目无自定义 Logger 时使用）

```swift
import UIKit
import os.log

// MARK: - Logger
extension Logger {
    private static let subsystem = "com.app.<模块>"
    static let <category> = Logger(subsystem: subsystem, category: "<Category>")
}

// MARK: - Class
final class <Class>: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        Logger.<category>.log("[viewDidLoad] start")
        setupUI()
    }
    
    private func setupUI() {
        // SnapKit layout
    }
    
    func <method>(param: Type) {
        Logger.<category>.log("[<method>] start, param: \(param)")
        
        guard let self = self else {
            Logger.<category>.error("[<method>] self 已释放")
            return
        }
        
        do {
            try operation()
            Logger.<category>.log("[<method>] success")
        } catch {
            assertionFailure("[<method>] failed: \(error)")
            Logger.<category>.error("[<method>] error: \(error)")
        }
    }
}
```

---

## 五、检查清单

生成代码后验证：
- [ ] UIKit + SnapKit
- [ ] **Logger 使用项目自定义规范**（如有），否则使用 `extension Logger`
- [ ] 统一 Subsystem，不同 Category
- [ ] 日志格式：`[方法名] 描述, 数据`
- [ ] Guard 分行写，包含日志
- [ ] try-catch 有 `assertionFailure` + 日志
- [ ] `assertionFailure` 包含 `[方法名]` 和详细数据
