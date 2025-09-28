# 因子 1 - 从自然语言到工具调用

[← 返回 README](README.md)

### 1. 从自然语言到工具调用

一个常见模式是将自然语言转换为结构化工具调用。

#### Stripe 示例

代理可以处理 Stripe API 调用，通过自然语言解析用户意图并生成工具调用。

#### 代理循环代码

```python
# 代理循环
def agent_loop(thread: Thread) -> str:
    while True:
        next_step = determine_next_step(thread)

        if next_step.intent == "done_for_now":
            return next_step.message

        # 处理工具调用...
        thread.events.append({"type": "tool_call", "data": next_step})

        # 执行工具并添加响应
        result = execute_tool(next_step)
        thread.events.append({"type": "tool_response", "data": result})
```

这允许代理逐步处理复杂任务，同时保持上下文。