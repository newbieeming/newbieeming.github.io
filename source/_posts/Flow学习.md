---
title: Flow学习
date: 2025-07-19 00:20
index_img: https://cdn.jsdelivr.net/gh/newbieeming/image@main/img20260414234656120.png
categories:
  - Android
  - Flow
  - Kotlin
---
### Flow

> Flow中存在冷流和热流 ，**冷流（Cold Flow）**和**热流（Hot Flow）**的核心区别在于**数据生产的触发时机**和**消费者关系**，二者分别适用于不同的异步数据流场景。

#### **一、冷流（Cold Flow）**

##### **定义与特性**

- **惰性执行**：仅在调用 `collect()` 时开始生产数据，每次收集都会重新执行流构建逻辑（如 `flow { ... }` 块内的代码）。
- **一对一关系**：每个消费者独立获取数据流，互不干扰。
- **无状态共享**：不保留历史数据，新消费者从头开始接收数据。
- **线程安全**：默认在调用者的协程上下文中运行，需配合 `flowOn` 切换线程。

##### **使用场景**

1. **一次性任务**

   - 如网络请求、数据库查询，每次收集需重新执行逻辑。
     **示例**：

   ```kotlin
   val coldFlow = flow {
       delay(1000) // 模拟网络请求
       emit("Data from network")
   }
   runBlocking {
       coldFlow.collect { println(it) } // 第一次收集
       coldFlow.collect { println(it) } // 第二次收集，重新执行网络请求
   }
   ```

2. **需要重复触发的数据生成**

   - 如分页加载、传感器数据融合，每次收集需重新生成数据。
     **示例**：

   ```kotlin
   val pageFlow = flow {
       for (page in 1..3) {
           delay(500)
           emit("Page $page data")
       }
   }
   runBlocking {
       pageFlow.collect { println(it) } // 每次收集重新加载分页数据
   }
   ```

3. **固定数据集的响应式处理**

   - 通过 `flowOf()` 或集合转换（如 `listOf(1, 2, 3).asFlow()`）创建静态数据流。
     **示例**：

   ```kotlin
   val colors = flowOf("Red", "Green", "Blue")
   runBlocking {
       colors.collect { println(it) } // 输出固定颜色列表
   }
   ```

#### **二、热流（Hot Flow）**

##### **定义与特性**

- **主动执行**：数据生产独立于 `collect()` 调用，即使无消费者也会发射数据。
- **一对多关系**：多个消费者共享同一数据流，可接收相同或历史数据。
- **状态共享**：通过 `StateFlow` 或 `SharedFlow` 保留最新值或历史数据。
- **背压支持**：通过缓冲区策略（如 `replay`、`extraBufferCapacity`）控制数据积压。

##### **使用场景**

1. **实时状态管理（StateFlow）**

   - 适用于 UI 状态更新（如 Jetpack Compose 或 ViewModel），始终持有最新状态值。
     **示例**：

   ```kotlin
   val stateFlow = MutableStateFlow(0) // 初始值为 0
   runBlocking {
       launch {
           delay(1000)
           stateFlow.value = 1 // 更新状态
       }
       stateFlow.collect { println("State: $it") } // 立即收到最新值
   }
   ```

2. **事件广播（SharedFlow）**

   - 适用于一次性事件（如 Toast 提示、导航跳转），避免重复消费问题。
     **示例**：

   ```kotlin
   val eventFlow = MutableSharedFlow<String>()
   runBlocking {
       launch {
           eventFlow.emit("Show Toast: Hello") // 发送事件
       }
       eventFlow.collect { println(it) } // 消费者接收事件
   }
   ```

3. **WebSocket/实时数据推送**

   - 多个订阅者共享同一数据流，新订阅者可获取历史数据（通过 `replay` 参数）。
     **示例**：

   ```kotlin
   val sharedFlow = MutableSharedFlow<Int>(replay = 2) // 回放最后 2 条数据
   runBlocking {
       launch {
           repeat(5) {
               delay(500)
               sharedFlow.emit(it)
           }
       }
       delay(1500)
       sharedFlow.collect { println("Subscriber 1: $it") } // 收到 1, 2, 3, 4
       delay(500)
       sharedFlow.collect { println("Subscriber 2: $it") } // 收到 3, 4（若 replay=2）
   }
   ```

#### **三、冷流 vs 热流：关键对比**

| **特性**         | **冷流（Flow）**               | **热流（StateFlow/SharedFlow）**    |
| ---------------- | ------------------------------ | ----------------------------------- |
| **数据生产时机** | 调用 `collect()` 时触发        | 独立于 `collect()`，主动发射数据    |
| **消费者关系**   | 一对一，每次收集重新执行       | 一对多，共享同一数据流              |
| **状态共享**     | 无状态，新消费者从头接收       | 保留最新值或历史数据（通过配置）    |
| **典型场景**     | 网络请求、分页加载、固定数据集 | UI 状态管理、事件广播、实时数据推送 |