# iOS 开发规范 (AI 执行手册)

> **文档用途**：本规范用于指导 AI 代码生成，AI 必须严格遵守以下规则。

---

## 快速决策清单

在生成/修改代码前，AI 必须按此清单检查：

- [ ] 技术栈约束检查（UIKit/CocoaPods/SnapKit）
- [ ] 工程文件禁令检查（.pbxproj/Podfile.lock/Pods/）
- [ ] Logger 使用检查（subsystem/category 正确性）
- [ ] Guard 格式检查（分行写）
- [ ] assertionFailure 检查（异常路径必须添加）

---

## 一、技术栈约束（强制）

| 技术 | 使用 | 禁止 |
|------|------|------|
| UI 框架 | UIKit | SwiftUI |
| 依赖管理 | CocoaPods | SPM |
| 布局 | SnapKit | Storyboard / XIB |

**AI 执行规则**：
- 生成 UI 代码必须使用 UIKit
- 布局代码必须使用 SnapKit
- 禁止生成 SwiftUI、Storyboard、XIB 相关代码

---

## 二、工程文件禁令（强制）

**绝对禁止修改**：
1. `.pbxproj` - 创建文件后由开发者手动添加到 Xcode
2. `Podfile.lock` - 只能改 `Podfile`，由开发者执行 `pod install`
3. `Pods/` 目录 - 完全禁止修改

**AI 执行规则**：
- 创建新文件时，只生成 `.swift` 文件，不修改项目配置文件
- 需要添加依赖时，只修改 `Podfile`，不触碰 `Podfile.lock`

---

## 三、编译命令

```bash
# 编译前必须在项目目录执行
cd <项目目录> && pod install

# 然后使用 .xcworkspace 打开/编译
open *.xcworkspace
# 或
xcodebuild -workspace *.xcworkspace -scheme <SchemeName>

# 查看可用模拟器
xcrun simctl list devices
```

---

## 四、日志规范（强制）

### 4.1 核心思想

使用 **Subsystem + Category** 两层结构组织日志：
- **Subsystem**：大功能模块（如 `com.app.network`）
- **Category**：模块内的具体流程（如 `Request`, `Response`, `Error`）

> **兼容性说明**：如果项目已有 日志 库支持 subsystem/category，直接使用；否则建议采用系统 `os.log`。

### 4.2 标准使用模式（os.log）

**前提**：必须导入 `os.log`，`Logger` 是 `OSLog.Logger` 的扩展

```swift
import os.log

// 扩展的是 os.log 的 Logger，不是自定义 Logger 类
extension Logger {
    // Subsystem = 模块标识
    private static let networkSubsystem = "com.app.network"
    
    // Category = 流程标识（首字母大写）
    static let networkRequest  = Logger(subsystem: networkSubsystem, category: "Request")
    static let networkResponse = Logger(subsystem: networkSubsystem, category: "Response")
    static let networkCache    = Logger(subsystem: networkSubsystem, category: "Cache")
    static let networkError    = Logger(subsystem: networkSubsystem, category: "Error")
}
```

**使用方式**：
```swift
import os.log

// 使用扩展中定义的静态属性
Logger.networkRequest.log("发起请求: \(url)")
Logger.networkResponse.log("收到响应: \(status)")
Logger.networkError.error("请求失败: \(error)")
```

### 4.3 禁止事项

```swift
// ❌ 禁止：将流程拆成多个 Subsystem
static let requestLogger = Logger(subsystem: "com.app.network.request", category: "Request")
static let responseLogger = Logger(subsystem: "com.app.network.response", category: "Response")

// ✅ 正确：统一 Subsystem，用 Category 区分
static let networkRequest  = Logger(subsystem: "com.app.network", category: "Request")
static let networkResponse = Logger(subsystem: "com.app.network", category: "Response")
```

### 4.4 日志详细程度（强制）

**每条日志必须包含**：
1. **[方法名]**：上下文标识
2. **关键数据**：影响流程的变量值
3. **状态信息**：当前阶段或结果

**日志层级**：

| 层级 | 记录内容 | 示例 |
|------|---------|------|
| 入口 | 方法调用 + 参数 | `[fetchUser] start, userId: xxx` |
| 节点 | 流程关键节点 | `[fetchUser] cache hit` |
| 分支 | 分支选择 + 条件 | `[fetchUser] use network, cache expired` |
| 异常 | 详细错误信息 | `[fetchUser] failed, error: xxx` |

**完整示例**：

```swift
import os.log

extension Logger {
    private static let subsystem = "com.app.payment"
    static let payment = Logger(subsystem: subsystem, category: "Payment")
}

func processPayment(orderId: String) {
    // 1. 入口日志
    Logger.payment.log("[processPayment] start, orderId: \(orderId)")
    
    // 2. 节点日志
    validateOrder(orderId)
    Logger.payment.log("[processPayment] order validated")
    
    // 3. 分支日志
    if !hasEnoughBalance {
        Logger.payment.log("[processPayment] insufficient balance, required: \(amount)")
        return .failure(.insufficientBalance)
    }
    
    // 4. 异常日志
    do {
        try chargeCustomer()
        Logger.payment.log("[processPayment] charged success")
    } catch {
        Logger.payment.error("[processPayment] charge failed, error: \(error)")
        return .failure(.chargeFailed)
    }
    
    Logger.payment.log("[processPayment] finished")
}
```

**不良示例对比**：

```swift
// ❌ 信息不足
Logger.payment.log("start")
Logger.payment.log("done")

// ✅ 详细完整
Logger.payment.log("[processPayment] start, orderId: \(orderId)")
Logger.payment.log("[processPayment] finished, transactionId: \(txId)")
```

### 4.5 Guard 语句规范（强制）

**必须分行写**，便于添加日志：

```swift
// ✅ 正确：分行写
guard let self = self else {
    Logger.error.error("[methodName] self 已释放")
    return false
}

guard !items.isEmpty else {
    return
}

// ❌ 禁止：写成一行
guard let self = self else { return false }
guard !items.isEmpty else { return }
```

---

## 五、assertionFailure 规则（强制）

### 5.1 核心原则

**开发期间必须添加 assertionFailure**，用于 Debug 模式快速暴露问题。

### 5.2 必须使用场景

| 场景 | 处理方式 |
|------|---------|
| try-catch 异常 | catch 块中必须调用 assertionFailure |
| 严重 guard 失败 | 理论上不应该发生的情况 |
| 意外的 nil | 关键对象必须存在 |
| 状态不一致 | 内部状态错误 |

### 5.3 标准模式

**try-catch 场景**：
```swift
do {
    let result = try parseResponse(data)
    Logger.parse.log("[parseResponse] success, items: \(result.count)")
} catch {
    // ✅ 必须：触发 assertionFailure + 记录详细错误
    assertionFailure("[parseResponse] 解析失败, dataSize: \(data.count), error: \(error)")
    Logger.parse.error("[parseResponse] failed, dataSize: \(data.count), error: \(error)")
    return nil
}
```

**guard 场景**：
```swift
guard let config = loadConfig() else {
    // ✅ 理论上不应该发生
    assertionFailure("[loadConfig] 配置文件加载失败，路径: \(configPath)")
    Logger.dataAccess.error("[loadConfig] config 加载失败，使用默认配置")
    return defaultConfig
}
```

**状态检查场景**：
```swift
func play() {
    // ✅ 前置状态检查
    if playerState != .ready {
        assertionFailure("[play] 播放器状态错误: \(playerState)，必须先调用 prepare()")
    }
    if currentItem == nil {
        assertionFailure("[play] currentItem 为空，必须先设置播放内容")
    }
    
    Logger.player.log("[play] start, item: \(currentItem?.id ?? "nil")")
    // ...
}
```

### 5.4 内容规范

**每条 assertionFailure 必须包含**：
1. **[方法名]**：上下文标识
2. **错误原因**：具体失败原因
3. **关键数据**：有助于定位的变量值

```swift
// ✅ 良好示例
assertionFailure("[fetchUser] response 为空, userId: \(userId), statusCode: \(statusCode)")
assertionFailure("[selectItem] index 越界: \(index), count: \(count)")
assertionFailure("[updateUI] 必须在主线程调用，当前线程: \(Thread.current)")

// ❌ 不良示例
assertionFailure("")
assertionFailure("result 为 nil")
assertionFailure("出错了")
```

### 5.5 assertionFailure vs fatalError

| 方式 | 使用场景 | 生产环境行为 |
|------|---------|-------------|
| `assertionFailure("...")` | 理论上不应发生的情况 | 忽略，继续执行 |
| `fatalError("...")` | 严重不可恢复错误 | 崩溃 |

```swift
// ✅ 开发期暴露，生产期优雅降级
guard let data = cache[key] else {
    assertionFailure("[getCache] 缓存数据缺失, key: \(key)")
    Logger.cache.error("[getCache] cache miss, key: \(key)")
    return fetchFromNetwork(key)
}
```

---

## 六、AI 代码生成模板参考

### 6.1 新功能文件模板

```swift
import UIKit
import os.log 

// MARK: - Logger Definition (扩展 os.log.Logger)
extension Logger {
    private static let subsystem = "com.app.<模块名>"
    static let <category> = Logger(subsystem: subsystem, category: "<Category>")
}

// MARK: - Class/Struct Definition
final class <ClassName>: UIViewController {
    
    // MARK: - Properties
    
    // MARK: - Lifecycle
    override func viewDidLoad() {
        super.viewDidLoad()
        Logger.<category>.log("[<method>] start")
        setupUI()
    }
    
    // MARK: - Setup
    private func setupUI() {
        // SnapKit layout code
    }
    
    // MARK: - Methods
    func <method>() {
        Logger.<category>.log("[<method>] start, param: \(value)")
        
        // guard 必须分行
        guard let self = self else {
            Logger.<category>.error("[<method>] self 已释放")
            return
        }
        
        // 分支记录
        if <condition> {
            Logger.<category>.log("[<method>] branch A, reason: xxx")
        } else {
            Logger.<category>.log("[<method>] branch B, reason: xxx")
        }
        
        // 异常处理
        do {
            try <operation>()
            Logger.<category>.log("[<method>] success")
        } catch {
            assertionFailure("[<method>] failed: \(error)")
            Logger.<category>.error("[<method>] error: \(error)")
        }
    }
}
```

### 6.2 检查清单（AI 自检用）

生成代码后，AI 必须验证：

- [ ] 使用 UIKit，非 SwiftUI
- [ ] 布局使用 SnapKit，非 Storyboard/XIB
- [ ] Logger 已定义（extension Logger）
- [ ] Subsystem 和 Category 正确区分
- [ ] 每条日志包含 [方法名] 前缀
- [ ] Guard 语句分行写（非单行）
- [ ] try-catch 包含 assertionFailure
- [ ] assertionFailure 包含详细上下文

---

## 七、常见错误速查

| 错误类型 | 错误示例 | 正确示例 |
|---------|---------|---------|
| Logger 定义错误 | 多个 Subsystem | 统一 Subsystem + 不同 Category |
| 日志信息不足 | `log("start")` | `log("[method] start, param: x")` |
| Guard 写成一行 | `guard x else { return }` | `guard x else {\n  log()\n  return\n}` |
| 缺少 assertionFailure | catch 只记录日志 | catch + assertionFailure + 日志 |
| 使用错误技术栈 | `import SwiftUI` | `import UIKit` |
