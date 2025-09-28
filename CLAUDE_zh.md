# CLAUDE.md

此文件为 Claude Code (claude.ai/code) 在此仓库中处理代码时提供指导。

## 仓库概述

这是一个专注于 AI/ML 的项目集合，主要围绕对话 AI、角色扮演和神经网络应用。仓库包含多个独立项目，使用不同的技术栈，包括 Web 应用、桌面应用、Python ML 框架和 Go 后端。

## 项目类别和技术

### 对话 AI / 角色扮演
- **SillyTavern**：Node.js/Express 后端，支持 MongoDB 的 AI 角色聊天
- **RisuAI**：多平台 Svelte 5 + Tauri + Capacitor 应用，支持 MongoDB/MinIO 存储
- **RolePlayAI**：基于 Svelte 的角色扮演应用，支持 Tauri 桌面版和 Web 版
- **Narratium.ai**：Next.js AI 叙事平台，集成 LangChain

### AI 开发平台
- **coze-studio**：全栈 AI 代理平台（React/TypeScript 前端 + Go 后端）
- **Second-Me-Org**：AI 身份平台，支持分层内存建模（Flask + Next.js）

### ML/AI 研究与工具
- **ComfyUI**：基于 Python 的 AI 工作流框架，PyTorch 后端
- **CharacterEval**：Python 评估框架，用于基于角色的对话系统
- **super-agent-party**：Python 代理编排框架

## 常见开发命令

### Node.js/TypeScript 项目（大多数前端项目）
```bash
# 包管理（使用项目特定管理器）
npm install          # 或 pnpm install / yarn install
npm run dev          # 开发服务器
npm run build        # 生产构建
npm run lint         # 代码 linting
npm run test         # 运行测试

# 对于 Rush.js 单体仓库 (coze-studio)
rush update          # 安装依赖
rush build           # 构建所有包
rush test            # 运行所有测试
```

### Python 项目 (ComfyUI, CharacterEval 等)
```bash
# 环境设置
pip install -r requirements.txt
# 或
poetry install

# 常见 Python 命令
python main.py       # 运行主应用
python -m pytest    # 运行测试
python -m ruff check # Linting（配置处）
```

### 多平台桌面应用 (RisuAI, RolePlayAI)
```bash
# 开发
pnpm dev             # Web 开发服务器
pnpm tauri dev       # 桌面应用开发
pnpm runserver       # 后端 API 服务器

# 构建
pnpm build           # Web 构建
pnpm tauribuild      # 桌面应用构建
pnpm buildandroid    # Android 构建
pnpm buildios        # iOS 构建（需要 Xcode）
```

### Docker 基于的项目 (coze-studio, Second-Me-Org)
```bash
# 环境设置
docker compose up -d # 启动所有服务
make middleware      # 启动数据库/基础设施服务
make server          # 开发模式启动后端
make debug           # 完整开发环境

# 构建与部署
make build           # 构建应用
make docker-up       # 完整 Docker 部署
```

## 架构模式

### 多平台应用
几个项目遵循跨平台模式：
- **Web**：Vite/SvelteKit 或 Next.js 用于浏览器部署
- **桌面**：Tauri (Rust) 用于原生桌面应用
- **移动**：Capacitor 用于 iOS/Android 部署
- **后端**：Node.js/Express 或 Python Flask 用于 API 服务

### 存储架构
- **传统**：浏览器存储使用 LocalForage
- **现代**：MongoDB + MinIO 用于可扩展应用
- **混合**：项目通常同时支持传统和现代存储

### AI 集成模式
- **多提供商支持**：OpenAI、Anthropic、Ollama、本地模型
- **流式响应**：实时 AI 响应处理
- **令牌管理**：使用跟踪和优化
- **内存系统**：角色一致性的上下文管理

## 数据库系统

### MongoDB（主要用于角色/聊天应用）
```bash
# 连接测试
node test-mongodb-connection.js

# 数据迁移
npm run migrate      # 适用处
```

### SQLite（用于本地/桌面应用）
```bash
# 数据库管理通过应用 API 处理
# 通常不需要直接 SQL 命令
```

### 向量数据库
- **Milvus**：coze-studio 中的嵌入使用
- **Vectra**：SillyTavern 中的向量相似性搜索

## 关键开发模式

### 状态管理
- **Svelte 5**：基于 Runes 的响应系统
- **React**：Zustand 用于全局状态管理
- **TypeScript**：项目全面类型安全

### API 设计
- **REST API**：Express.js 后端，综合中间件
- **WebSocket**：聊天应用的实时通信
- **流式**：AI 响应流式的服务器发送事件

### 测试策略
- **前端**：现代项目使用 Vitest，遗留使用 Jest
- **后端**：Python 使用 pytest，Go 使用内置测试
- **E2E**：按项目需求单独配置

## AI 模型配置

许多项目支持多个 AI 提供商。典型配置位置：
- `backend/conf/model/` (coze-studio)
- `server/config/` (RolePlayAI, RisuAI)
- 环境变量（`.env` 文件）

支持的提供商通常包括：
- OpenAI GPT 模型
- Anthropic Claude
- 通过 Ollama 的本地模型
- Hugging Face 变换器
- 自定义端点

## 常见问题与解决方案

### 包管理
- 使用每个项目的特定包管理器 (npm, pnpm, yarn, poetry)
- 对于单体仓库，使用单体工具 (Rush.js) 而非单个包管理器
- 检查 `package.json` 脚本以获取项目特定命令

### 跨平台开发
- Tauri 应用需要 Rust 工具链
- 移动构建需要 Android Studio/Xcode
- 桌面构建可能需要平台特定依赖

### AI 集成
- 确保正确配置 API 密钥
- 检查速率限制和令牌使用
- 验证自定义提供商的模型兼容性
- 单独测试流式端点与标准请求

### 存储迁移
- 许多项目正从 LocalForage 迁移到 MongoDB
- 迁移工具通常在 `migration/` 目录中提供
- 运行迁移前备份数据

## 安全考虑

- API 密钥应存储在环境变量中
- 绝不提交秘密到版本控制
- Web 应用使用 CSRF 保护
- 为多源部署实施正确的 CORS 策略
- 验证并清理用户输入，尤其是 AI 提示

## 性能优化

### 前端
- 大型应用使用代码拆分
- AI 模型实现延迟加载
- 使用适当的树摇优化捆绑大小
- 使用服务工作者实现离线功能

### 后端
- 实施正确的缓存策略
- 使用数据库连接池
- 优化 AI 模型加载和推理
- 监控长运行进程的内存使用
- 将现有工作总结到.claude\note.md中，包括现在已经完成部分和下一步的工作