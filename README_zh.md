# 12 因子代理宣言

## 12 因子代理是什么？

一个 12 因子代理是一个使用 LLM 作为推理引擎的软件代理，遵循 12 因子应用的 12 个原则，但针对代理进行了调整。

这些原则帮助代理在极端条件下运行良好，并具有良好的开发/生产对齐。

## 为什么 12 因子代理？

构建代理的困难在于它们在简单和复杂之间摇摆不定。

一些代理是“无状态”的：从头开始运行并结束。

其他代理需要管理变量状态：这些工作，但它们变得混乱。

12 因子代理方法旨在让代理既简单又强大：它们是“无状态”的，但看起来是有状态的。

## 12 因子代理原则

### 1. 从自然语言到工具调用

将用户的自然语言查询转换为结构化工具调用。

#### 代理循环伪代码

```python
# 代理循环
def agent_loop(thread: Thread) -> str:
    while True:
        next_step = determine_next_step(thread)

        if next_step.intent == "done_for_now":
            return next_step.message

        if next_step.intent == "add":
            thread.events.append({"type": "tool_call", "data": next_step})
            result = next_step.a + next_step.b
            thread.events.append({"type": "tool_response", "data": result})
            continue

        # 处理其他工具...
```

### 2. 拥有你的提示

将系统提示作为代码的一部分，而不是硬编码字符串。

### 3. 拥有你的上下文窗口

将上下文作为代码的一部分，使用自定义序列化格式（如 XML）。

### 4. 工具作为结构化输出

使用结构化输出来定义工具，而不是函数调用 API。

### 5. 统一执行状态

将执行状态作为线程事件列表管理。

### 6. 使用简单 API 启动/暂停/恢复

通过 API 端点启动、暂停和恢复代理。

### 7. 使用工具联系人类

使用人类工具来请求澄清或批准。

### 8. 拥有你的控制流

在代码中实现代理循环，而不是依赖框架。

### 9. 拥有你的部署依赖

明确声明所有部署依赖，如 HumanLayer 用于人类联系。

### 10. 拥有你的后台任务

使用队列或调度器处理后台任务。

### 11. 拥有你的可观察性

记录代理事件和人类交互。

### 12. 拥有你的持久化

使用数据库或文件系统持久化线程状态。

## 快速入门

### 安装

```bash
npm install @boundaryml/baml humanlayer
npx baml-cli init
```

### 构建代理

遵循 [工作坊](workshops/) 中的步骤构建基本代理。

### 运行测试

```bash
npx baml-cli test
```

## 贡献

- 阅读 [所有因素](content/)
- 运行 [工作坊](workshops/)
- 加入 [Discord](https://discord.gg/humanlayer)

## 许可证

此项目使用 [MIT 许可证](LICENSE)。