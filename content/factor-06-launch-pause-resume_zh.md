# 因子 6 - 使用简单 API 启动/暂停/恢复

[← 统一执行状态](factor-05-unify-execution-state.md)

### 6. 使用简单 API 启动/暂停/恢复

代理应通过 REST API 或 Webhook 管理生命周期。

#### 示例端点

```javascript
// POST /thread - 启动新线程
app.post('/thread', async (req, res) => {
    const thread = new Thread([{ type: "user_input", data: req.body.message }]);
    const result = await agentLoop(thread);
    res.json(result);
});

// GET /thread/:id - 获取线程状态
app.get('/thread/:id', (req, res) => {
    const thread = store.get(req.params.id);
    res.json(thread);
});
```

这允许外部系统暂停和恢复代理执行。