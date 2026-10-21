# 建模规则

## 元素

- 按照 C4 Model 为理论指导进行建模

- 系统上下文: 定义 `system`，为用户或其他系统提供有价值功能的软件系统，由一个或多个 Container 构成
- 容器: 定义 `backend`、`frontend`、`message` 等，运行应用程序或存储数据的单元，具有明确的运行时或数据存储边界。
- 组件: 定义 `component`，优先按业务职责划分，技术职责作为补充，不按 Controller、Service、Repository 机械划分。
- 代码: 不定义

### 组件判断准则

判断问题,满足时的含义

1. 是否属于某个明确的 Container？ Component 必须有容器边界
2. 是否有独立、清晰的职责？ 能解释它负责什么
3. 是否对其他模块提供明确接口？ 有可识别的交互边界
4. 是否封装了一组相关功能？ 不只是一个普通工具函数
5. 是否与所属 Container 一起部署？ 独立部署通常应考虑 Container

怎么划分很重要，比如是按业务语义划分，还是按照技术划分？(controller、service、repository)

### 示例

```c4
model {
    // 模型
    creditplatform = system "交易系统" {
        // 建模
        backend = backend "后端" {
            // 模型
            credit = component "授信"
            loan = component "借款"
            repay = component "还款"
            pay = component "支付"
            close = component "关户"

            // 关系
            crane.schedule .trigger loan "定时任务1触发" {
                description "每天凌晨执行借款状态同步"
            }
            crane.schedule .trigger loan "定时任务2触发" {
                description "每小时执行借款超时检查"
            }
            loan .publish mafka.queue "发布"
            pay .subscribes mafka.queue "订阅" {
                description "订阅 pay.notify 主题"
            }
        }
        db = database "数据库"
        // 关系
        backend .read-write db "读写"
    }

    // 关系
    creditplatform.backend.loan -> xiaodai-gateway.backend.loan
    creditplatform.backend -> portal.backend "依赖"
    creditplatform -> xiaodai
}
```

## 关系

- 调用，定义 `call`
- 异步调用，定义 `async`
- 依赖，定义 `depend`
- 订阅, 定义 `subscribe`
- 发布，定义 `publish`
- 触发，定义 `trigger`
- 读写，定义 `read-write`
- ...

## 视图

### L1 系统上下文视图

```c4
view l1-system-context-{system-name} {
    title "xxx"
    include -> {system-name}->
    include {system-name}
}
```

### L2 容器视图

```c4
view l2-container-{system-name} of {system-name} {
    title "xxx"
    include *
}
```

### L3 组件视图

```c4
view l3-component-{system-name}-{container-name} of {system-name}.{container-name} {
    title "xxx"
    include *
}
```

### L4 代码视图

不建模

## 总结

架构建模基本原则

1. 以 C4 Model 为理论基础，采用 L1 System、L2 Container、L3 Component 三层建模，不维护 L4。
2. System 表示具有独立职责的软件系统。
3. Container 表示应用运行单元或数据存储单元。
4. Component 表示 Container 内部封装相关功能、具有明确接口的逻辑单元。
5. Component 优先按业务职责划分，允许独立技术组件，不直接按照 Maven Module 或技术分层划分。
6. 关系连接已知的最准确交互端点，允许跨层级关系。
7. 关系类型必须具有明确语义，方向必须与真实交互一致。
8. 同一架构事实只维护一次，不为视图重复定义关系。
9. 视图按 L1/L2/L3 分层，必要时提供依赖扩展视图。
10. AI 以完整 Model 为事实来源，按需检索元素和关系，不依赖视图的可见内容。
