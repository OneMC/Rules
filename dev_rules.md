# 开发规范

## 技术栈约束

| 技术 | 使用 | 禁止 |
|------|------|------|
| UI 框架 | UIKit | SwiftUI |
| 依赖管理 | CocoaPods | SPM |
| 布局 | SnapKit | Storyboard / XIB |

## 工程文件禁令

- **禁止修改 `.pbxproj`** - 创建文件后由开发者手动添加到 Xcode
- **禁止修改 `Podfile.lock`** - 只能改 `Podfile`，由开发者执行 `pod install`
- **禁止修改 `Pods/` 目录**

## 编译命令

项目使用 **CocoaPods** 管理依赖，比如进入到`.xcworkspace` 或 `.xcodeproj`的项目夹中进行编译，编译前必须在项目执行 `pod install`，

### 查看可用模拟器

```bash
xcrun simctl list devices
```

## Logger 规范

> **说明**：本节阐述的是 **Subsystem 与 Category 的使用思想**，用于区分大功能模块和具体流程。
>
> - 如果项目中已有的 Logger 库**支持** `subsystem` 和 `category` 参数，请直接使用项目封装
> - 如果项目中 Logger 库**不支持**，建议开发者评估是否采用系统原生的 `os.log`，以获得 subsystem/category 的日志分类能力

### Subsystem vs Category 规范

**核心原则**：
- **Subsystem**：标识一个大的功能模块（如 `com.app.module`）
- **Category**：标识该模块内的具体流程或子功能（首字母大写）

#### 1. Subsystem 定义

每个独立的大功能模块定义一个统一的 Subsystem：

```swift
import os.log

// ✅ 正确：整个模块使用统一的 Subsystem
extension Logger {
    private static let subsystem = "com.app.network"
    
    static let networkRequest = Logger(subsystem: subsystem, category: "Request")
    static let networkResponse = Logger(subsystem: subsystem, category: "Response")
    static let networkCache = Logger(subsystem: subsystem, category: "Cache")
    static let networkError = Logger(subsystem: subsystem, category: "Error")
}
```

**禁止**：将流程拆分成多个 Subsystem。

```swift
// ❌ 错误：不要把流程拆成多个 Subsystem
extension Logger {
    static let requestLogger = Logger(subsystem: "com.app.network.request", category: "Request")
    static let responseLogger = Logger(subsystem: "com.app.network.response", category: "Response")
}
```

#### 2. Category 定义

使用 Category 区分同一模块内的不同流程：

```swift
import os.log

// ✅ 正确：用 Category 区分流程
extension Logger {
    private static let networkSubsystem = "com.app.network"
    
    static let networkRequest  = Logger(subsystem: networkSubsystem, category: "Request")
    static let networkResponse = Logger(subsystem: networkSubsystem, category: "Response")
    static let networkCache    = Logger(subsystem: networkSubsystem, category: "Cache")
    static let networkError    = Logger(subsystem: networkSubsystem, category: "Error")
}

// 使用
Logger.networkRequest.log("发起请求: \(url)")
Logger.networkResponse.log("收到响应: \(status)")
Logger.networkCache.log("缓存命中: \(key)")
Logger.networkError.error("请求失败: \(error)")
```

#### 3. 层级设计示例

| 模块 | Subsystem | Category | 说明 |
|------|-----------|----------|------|
| 网络 | `com.app.network` | Request / Response / Cache / Error | 请求/响应/缓存/错误 |
| 数据库 | `com.app.database` | Query / Migration / Sync / Error | 查询/迁移/同步/错误 |
| 播放器 | `com.app.player` | Load / Buffer / Render / Control | 加载/缓冲/渲染/控制 |

### 日志详细程度规范

**原则**：日志从上到下逐级详细，确保能快速定位问题。

#### 1. 入口层日志

记录方法调用和关键参数：

```swift
import os.log

extension Logger {
    private static let subsystem = "com.app.user"
    static let userProfile = Logger(subsystem: subsystem, category: "Profile")
}

// ✅ 正确：入口记录调用信息
func fetchUserProfile(userId: String) {
    Logger.userProfile.log("[fetchUserProfile] start, userId: \(userId)")
    // ...
}
```

#### 2. 流程节点日志

记录流程关键节点和状态变化：

```swift
import os.log

extension Logger {
    private static let subsystem = "com.app.payment"
    static let payment = Logger(subsystem: subsystem, category: "Payment")
}

// ✅ 正确：记录流程节点
func processPayment(orderId: String) {
    Logger.payment.log("[processPayment] start, orderId: \(orderId)")
    
    validateOrder(orderId)
    Logger.payment.log("[processPayment] order validated")
    
    chargeCustomer()
    Logger.payment.log("[processPayment] charged success")
    
    updateOrderStatus()
    Logger.payment.log("[processPayment] finished")
}
```

#### 3. 分支判断日志

记录分支选择和判断条件：

```swift
import os.log

extension Logger {
    private static let subsystem = "com.app.data"
    static let dataLoad = Logger(subsystem: subsystem, category: "Data")
}

// ✅ 正确：记录分支选择
if cache.isValid {
    Logger.dataLoad.log("[loadData] use cache, age: \(cache.age)s")
    return cache.data
} else {
    Logger.dataLoad.log("[loadData] cache expired, fetch from network")
    return fetchFromNetwork()
}
```

#### 4. 异常路径日志

错误和异常必须有详细日志：

```swift
import os.log

extension Logger {
    private static let subsystem = "com.app.parser"
    static let parse = Logger(subsystem: subsystem, category: "Parse")
}

// ✅ 正确：详细记录错误信息
do {
    let result = try parseResponse(data)
    Logger.parse.log("[parseResponse] success, items: \(result.count)")
} catch {
    Logger.parse.error("[parseResponse] failed, dataSize: \(data.count), error: \(error)")
    return nil
}
```

#### 5. 日志内容规范

每条日志应包含：
- **上下文标识**：方法名/流程名
- **关键数据**：影响流程的变量值
- **状态信息**：当前执行的阶段或结果

```swift
import os.log

extension Logger {
    private static let subsystem = "com.app.sync"
    static let sync = Logger(subsystem: subsystem, category: "Sync")
}

// ✅ 良好示例
Logger.sync.log("[syncData] start, lastSync: \(lastSyncTime), pending: \(pendingItems.count)")
Logger.sync.log("[syncData] batch uploaded, count: \(uploaded), failed: \(failed)")
Logger.sync.log("[syncData] completed, nextSync: \(nextSyncTime)")

// ❌ 不良示例：信息不足
Logger.sync.log("sync started")
Logger.sync.log("sync done")
```

### Guard 语句规范

**原则**：guard 语句需要返回时，必须分行写，方便后续添加 debug 日志。

```swift
import os.log

extension Logger {
    private static let subsystem = "com.app.module"
    static let error = Logger(subsystem: subsystem, category: "Error")
}

// ✅ 正确：分行写，便于添加日志
guard let self = self else {
    Logger.error.log("self 已释放")
    return false
}

guard !items.isEmpty else {
    return
}

// ❌ 错误：不要写成一行
guard let self = self else { return false }
guard !items.isEmpty else { return }
```

## assertionFailure 规则

### 核心原则

**开发期间应尽可能添加 `assertionFailure`**，用于在 Debug 模式下快速暴露问题，避免潜在 bug 流入生产环境。

### 必须添加 assertionFailure 的场景

| 场景 | 说明 | 推荐方式 |
|------|------|----------|
| try-catch 异常 | 捕获到异常时立即触发 | `catch { assertionFailure("...") }` |
| 严重 guard 失败 | 不应该发生的状态 | `assertionFailure("...")` |
| 意外的 nil | 关键对象必须存在 | `assertionFailure("...")` |
| 状态前置检查 | 条件不满足时触发 | `assert(condition, "...")` 或 `if !condition { assertionFailure("...") }` |

### 示例场景

#### try-catch 场景

```swift
import os.log

extension Logger {
    static let parser = Logger(subsystem: "com.app.parser", category: "Parse")
}

do {
    let result = try parseResponse(data)
    Logger.parser.log("[parseResponse] success, items: \(result.count)")
} catch {
    // ✅ 正确：触发 assertionFailure 并记录详细错误
    assertionFailure("[parseResponse] 解析失败, dataSize: \(data.count), error: \(error)")
    Logger.parser.error("[parseResponse] failed, dataSize: \(data.count), error: \(error)")
    return nil
}
```

#### guard 场景

```swift
import os.log

extension Logger {
    static let dataAccess = Logger(subsystem: "com.app.data", category: "DataAccess")
}

// ✅ 正确：严重错误添加 assertionFailure
guard let config = loadConfig() else {
    assertionFailure("[loadConfig] 配置文件加载失败，路径: \(configPath)")
    Logger.dataAccess.error("[loadConfig] config 加载失败，使用默认配置")
    return defaultConfig
}

// ✅ 正确：数据有效性检查
func processItems(_ items: [Item]) {
    if items.isEmpty {
        assertionFailure("[processItems] items 不应为空数组")
    }
    if items.count >= 10000 {
        assertionFailure("[processItems] items 数量异常: \(items.count)")
    }
    
    Logger.dataAccess.log("[processItems] start, count: \(items.count)")
    // ...
}
```

#### 状态检查场景

```swift
import os.log

extension Logger {
    static let player = Logger(subsystem: "com.app.player", category: "Player")
}

func play() {
    // ✅ 正确：前置状态检查
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

### assertionFailure 内容规范

**每条 assertionFailure 必须包含**:
1. **方法名/上下文标识**：`[methodName]`
2. **具体错误原因**：失败的具体原因
3. **关键数据**：有助于定位问题的变量值

```swift
// ✅ 良好示例
assertionFailure("[fetchUser] response 为空, userId: \(userId), statusCode: \(statusCode)")
assertionFailure("[selectItem] index 越界: \(index), count: \(count)")
assertionFailure("[updateUI] 必须在主线程调用，当前线程: \(Thread.current)")

// ❌ 不良示例：信息不足
assertionFailure("")
assertionFailure("result 为 nil")
assertionFailure("出错了")
```

### assertionFailure vs fatalError

| 场景 | 推荐方式 | 说明 |
|------|---------|------|
| 开发期快速暴露问题 | `assertionFailure("...")` | Debug 触发，Release 忽略 |
| 严重不可恢复错误 | `fatalError("...")` | 任何环境都崩溃 |
| 理论上不可能发生 | `assertionFailure("...")` + 日志 | 开发期暴露，生产期优雅降级 |

```swift
// ✅ 正确：开发期暴露，生产期记录错误并降级
guard let data = cache[key] else {
    assertionFailure("[getCache] 缓存数据缺失, key: \(key)")
    Logger.cache.error("[getCache] cache miss, key: \(key)")
    return fetchFromNetwork(key)
}
```
