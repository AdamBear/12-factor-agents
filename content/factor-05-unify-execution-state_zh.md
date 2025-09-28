# 因子 5 - 统一执行状态

[← 工具作为结构化输出](factor-04-tools-as-structured-outputs.md)

### 5. 统一执行状态

将执行状态作为线程事件列表统一管理。

#### 线程类

```typescript
export class Thread {
    events: Event[] = [];

    constructor(events: Event[]) {
        this.events = events;
    }

    serializeForLLM() {
        return this.events.map(e => this.serializeOneEvent(e)).join("\n");
    }
}
```

事件包括用户输入、工具调用和响应，确保状态一致性。