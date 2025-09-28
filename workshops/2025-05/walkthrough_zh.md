# 从零构建 12 因子代理模板

从裸 TypeScript 仓库开始构建 12 因子代理的步骤。此指南将指导你创建遵循 12 因子方法的 TypeScript 代理。

### 清理

确保从干净状态开始

```bash
rm -rf baml_src/ && rm -rf src/
```

### 第 0 章 - Hello World

复制初始 package.json

```bash
cp ./walkthrough/00-package.json package.json
npm install
cp ./walkthrough/00-tsconfig.json tsconfig.json
cp ./walkthrough/00-.gitignore .gitignore
mkdir -p src
cp ./walkthrough/00-index.ts src/index.ts
npx tsx src/index.ts
```

你应该看到：hello, world!

### 第 1 章 - CLI 和代理循环

安装 BAML

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

你应该看到模型响应。

### 第 2 章 - 添加计算器工具

```bash
cp ./walkthrough/02-tool_calculator.baml baml_src/tool_calculator.baml
cp ./walkthrough/02-agent.baml baml_src/agent.baml
npx baml-cli generate
npx tsx src/index.ts 'can you add 3 and 4'
```

### 第 3 章 - 在循环中处理工具调用

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

### 第 4 章 - 将测试添加到 agent.baml

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

### 第 5 章 - 多个人类工具

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

### 第 6 章 - 使用推理自定义你的提示

```bash
export BAML_LOG=debug
cp ./walkthrough/06-agent.baml baml_src/agent.baml
npx baml-cli generate
npx tsx src/index.ts 'can you multiply 3 and 4'
```

#### 可选挑战
在你的工具输出格式中添加一个字段，包括推理步骤！

### 第 7 章 - 自定义你的上下文窗口

```bash
cp ./walkthrough/07-agent.ts src/agent.ts
BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'
cp ./walkthrough/07b-agent.ts src/agent.ts
BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'
cp ./walkthrough/07c-agent.baml baml_src/agent.baml
npx baml-cli test
```

### 第 8 章 - 添加 API 端点

```bash
export BAML_LOG=off
npm install express && npm install --save-dev @types/express supertest
cp ./walkthrough/08-server.ts src/server.ts
npx tsx src/server.ts
curl -X POST http://localhost:3000/thread -H "Content-Type: application/json" -d '{"message":"can you add 3 and 4"}'
```

### 第 9 章 - 内存状态和异步澄清

```bash
cp ./walkthrough/09-state.ts src/state.ts
cp ./walkthrough/09-server.ts src/server.ts
npx tsx src/server.ts
curl -X POST http://localhost:3000/thread -H "Content-Type: application/json" -d '{"message":"can you multiply 3 and xyz"}'
```

### 第 10 章 - 添加人类批准

```bash
cp ./walkthrough/10-server.ts src/server.ts
cp ./walkthrough/10-agent.ts src/agent.ts
npx tsx src/server.ts
curl -X POST http://localhost:3000/thread -H "Content-Type: application/json" -d '{"message":"can you divide 3 by 4"}'
curl -X POST 'http://localhost:3000/thread/{thread_id}/response' -H "Content-Type: application/json" -d '{"type": "approval", "approved": false, "comment": "I dont think thats right, use 5 instead of 4"}'
curl -X POST 'http://localhost:3000/thread/{thread_id}/response' -H "Content-Type: application/json" -d '{"type": "approval", "approved": true}'
```

### 第 11 章 - 通过电子邮件的人类批准

```bash
npm install humanlayer
cp ./walkthrough/11-cli.ts src/cli.ts
npx tsx src/index.ts 'can you divide 4 by 5'
cp ./walkthrough/11b-cli.ts src/cli.ts
npx tsx src/index.ts 'can you multiply 4 and xyz'
cp ./walkthrough/11c-cli.ts src/cli.ts
npx tsx src/index.ts 'can you divide 4 by 5'
```

### 第 XX 章 - HumanLayer Webhook 集成

添加初始化 HumanLayer 的代码到服务器

```bash
cp ./walkthrough/12-1-server-init.ts src/server.ts
cp ./walkthrough/12a-server.ts src/server.ts
npx tsx src/server.ts
```

现在服务器运行，发送负载到 '/thread' 端点

__ 执行响应步骤

__ 现在处理 divide 的批准

__ 现在也处理 done_for_now