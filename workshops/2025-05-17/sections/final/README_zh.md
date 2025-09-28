# 第 0 章 - Hello World

让我们从基本 TypeScript 设置和 Hello World 程序开始。

此指南使用 TypeScript 编写（是的，Python 版本即将推出）

在每个文件编辑之间有许多检查点，即使你不熟悉 TypeScript，也应该能够跟上并运行每个示例。

要运行此指南，你需要安装相对较新版本的 Node.js 和 npm

### 安装

```bash
brew install node@20
node --version
cp ./walkthrough/00-package.json package.json
npm install
cp ./walkthrough/00-tsconfig.json tsconfig.json
cp ./walkthrough/00-.gitignore .gitignore
mkdir -p src
cp ./walkthrough/00-index.ts src/index.ts
npx tsx src/index.ts
```

你应该看到：hello, world!

# 第 1 章 - CLI 和代理循环

现在让我们添加 BAML 并创建第一个带有 CLI 接口的代理。

首先，我们需要安装 [BAML](https://github.com/boundaryml/baml)，这是一个用于提示和结构化输出的工具。

```bash
npm install @boundaryml/baml
npx baml-cli init
rm baml_src/resume.baml
cp ./walkthrough/01-agent.baml baml_src/agent.baml
npx baml-cli generate
export BAML_LOG=debug
cp ./walkthrough/01-cli.ts src/cli.ts
cp ./walkthrough/01-index.ts src/index.ts
cp ./walkthrough/01-agent.ts src/agent.ts
export OPENAI_API_KEY=...
npx tsx src/index.ts hello
```

你应该看到模型的熟悉响应

# 第 2 章 - 添加计算器工具

让我们为我们的代理添加一些计算器工具。

让我们从添加计算器工具定义开始

这些是简单的结构化输出，我们将要求模型作为“下一步”返回

```bash
cp ./walkthrough/02-tool_calculator.baml baml_src/tool_calculator.baml
cp ./walkthrough/02-agent.baml baml_src/agent.baml
npx baml-cli generate
npx tsx src/index.ts 'can you add 3 and 4'
```

你应该看到工具调用到计算器

# 第 3 章 - 在循环中处理工具调用

现在让我们添加一个真实的代理循环，可以运行工具并从 LLM 获取最终答案。

首先，让我们更新代理来处理工具调用

```bash
cp ./walkthrough/03-agent.ts src/agent.ts
npx tsx src/index.ts 'can you add 3 and 4'
export BAML_LOG=off
npx tsx src/index.ts 'can you add 3 and 4, then add 6 to that result'
npx tsx src/index.ts 'can you multiply 3 and 4'
cp ./walkthrough/03b-agent.ts src/agent.ts
npx tsx src/index.ts 'can you subtract 3 from 4'
npx tsx src/index.ts 'can you multiply 3 and 4'
npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'
```

# 第 4 章 - 将测试添加到 agent.baml

让我们为我们的 BAML 代理添加一些测试。

要开始，请启用 baml 日志

```bash
export BAML_LOG=debug
cp ./walkthrough/04-agent.baml baml_src/agent.baml
npx baml-cli test
cp ./walkthrough/04b-agent.baml baml_src/agent.baml
npx baml-cli test
export BAML_LOG=off
cp ./walkthrough/04c-agent.baml baml_src/agent.baml
npx baml-cli test
```

# 第 5 章 - 多个人类工具

在本节中，我们将添加支持多个用于联系人类的工具。

```bash
export BAML_LOG=off
cp ./walkthrough/05-agent.baml baml_src/agent.baml
npx baml-cli generate
cp ./walkthrough/05-agent.ts src/agent.ts
cp ./walkthrough/05-cli.ts src/cli.ts
npx tsx src/index.ts 'can you multiply 3 and FD*(#F&& '
cp ./walkthrough/05b-agent.baml baml_src/agent.baml
npx baml-cli test
cp ./walkthrough/05c-agent.baml baml_src/agent.baml
npx baml-cli test
```

# 第 6 章 - 使用推理自定义你的提示

在本节中，我们将探索如何使用推理步骤自定义代理的提示。

这是 [因子 2 - 拥有你的提示](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-2-own-your-prompts.md) 的核心

有关于推理的深入探讨在 AI That Works [推理模型与推理步骤](https://github.com/hellovai/ai-that-works/tree/main/2025-04-07-reasoning-models-vs-prompts)

```bash
export BAML_LOG=debug
cp ./walkthrough/06-agent.baml baml_src/agent.baml
npx baml-cli generate
npx tsx src/index.ts 'can you multiply 3 and 4'
```

#### 可选挑战
在你的工具输出格式中添加一个字段，包括推理步骤！

# 第 7 章 - 自定义你的上下文窗口

在本节中，我们将探索如何自定义代理的上下文窗口。

这是 [因子 3 - 拥有你的上下文窗口](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-3-own-your-context-window.md) 的核心

```bash
cp ./walkthrough/07-agent.ts src/agent.ts
BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'
cp ./walkthrough/07b-agent.ts src/agent.ts
BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'
cp ./walkthrough/07c-agent.baml baml_src/agent.baml
npx baml-cli test
```

# 第 8 章 - 添加 API 端点

通过 HTTP 暴露代理的 Express 服务器。

```bash
export BAML_LOG=off
npm install express && npm install --save-dev @types/express supertest
cp ./walkthrough/08-server.ts src/server.ts
npx tsx src/server.ts
curl -X POST http://localhost:3000/thread -H "Content-Type: application/json" -d '{"message":"can you add 3 and 4"}'
```

你应该从代理获取答案，包括代理跟踪，以消息结束，如：

    {"intent":"done_for_now","message":"The sum of 3 and 4 is 7."}

# 第 9 章 - 内存状态和异步澄清

添加状态管理和异步澄清支持。

```bash
cp ./walkthrough/09-state.ts src/state.ts
cp ./walkthrough/09-server.ts src/server.ts
npx tsx src/server.ts
curl -X POST http://localhost:3000/thread -H "Content-Type: application/json" -d '{"message":"can you multiply 3 and xyz"}'
```

# 第 10 章 - 添加人类批准

添加对操作的人类批准支持。

```bash
cp ./walkthrough/10-server.ts src/server.ts
cp ./walkthrough/10-agent.ts src/agent.ts
npx tsx src/server.ts
curl -X POST http://localhost:3000/thread -H "Content-Type: application/json" -d '{"message":"can you divide 3 by 4"}'
```

你应该看到：

    {
  "thread_id": "2b243b66-215a-4f37-8bc6-9ace3849043b",
  "events": [
    {
      "type": "user_input",
      "data": "can you divide 3 by 4"
    },
    {
      "type": "tool_call",
      "data": {
        "intent": "divide",
        "a": 3,
        "b": 4,
        "response_url": "/thread/2b243b66-215a-4f37-8bc6-9ace3849043b/response"
      }
    }
  ]
}

使用另一个 curl 调用拒绝请求，更改线程 ID

```bash
curl -X POST 'http://localhost:3000/thread/{thread_id}/response' -H "Content-Type: application/json" -d '{"type": "approval", "approved": false, "comment": "I dont think thats right, use 5 instead of 4"}'
```

你应该看到：最后一个工具调用现在是 `"intent":"divide","a":3,"b":5"`

现在你可以批准操作

```bash
curl -X POST 'http://localhost:3000/thread/{thread_id}/response' -H "Content-Type: application/json" -d '{"type": "approval", "approved": true}'
```

你应该看到最终消息包括工具响应和最终结果！

# 第 11 章 - 通过电子邮件的人类批准

在本节中，我们将添加通过电子邮件的人类批准支持。

```bash
npm install humanlayer
cp ./walkthrough/11-cli.ts src/cli.ts
npx tsx src/index.ts 'can you divide 4 by 5'
cp ./walkthrough/11b-cli.ts src/cli.ts
npx tsx src/index.ts 'can you multiply 4 and xyz'
cp ./walkthrough/11c-cli.ts src/cli.ts
npx tsx src/index.ts 'can you divide 4 by 5'
```

这就是了 - 在下一章中，我们将构建一个完全通过电子邮件驱动的工作流代理，使用 Webhook 进行人类批准

# 第 XX 章 - HumanLayer Webhook 集成

前面的部分使用了 humanlayer SDK 的“同步模式” - 这意味着每次等待人类批准时，我们都在循环中轮询直到收到人类响应。

这显然不是理想的，尤其是对于生产工作负载，
所以在本节中我们将实现 [因子 6 - 使用简单 API 启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-6-launch-pause-resume.md)
通过更新服务器在联系人类后结束处理，并使用 Webhook 接收结果。

添加初始化 humanlayer 的代码到服务器

```bash
cp ./walkthrough/12-1-server-init.ts src/server.ts
cp ./walkthrough/12a-server.ts src/server.ts
npx tsx src/server.ts
```

现在服务器运行，发送负载到 '/thread' 端点

__ 执行响应步骤

__ 现在处理 divide 的批准

__ 现在也处理 done_for_now