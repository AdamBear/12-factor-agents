# 第 0 章 - Hello World

让我们从基本 TypeScript 设置和 Hello World 程序开始。

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