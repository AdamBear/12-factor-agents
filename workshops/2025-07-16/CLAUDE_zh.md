# 研讨会 2025-07-16: Python/Jupyter Notebook 实现

• **主要工具**：`walkthroughgen_py.py` - 将 TypeScript 演练转换为 Jupyter Notebook
• **配置**：`walkthrough.yaml` - 定义 Notebook 结构和内容
• **输出**：`workshop_final.ipynb` - 生成的 Notebook 包含第 0-7 章
• **测试**：`test_notebook_colab_sim.sh` - 模拟 Google Colab 环境

## 关键实现学习

• **Notebook 中无 async/await** - 所有 BAML 调用必须是同步的，移除所有异步模式
• **无 sys.argv** - 主函数直接接受参数：`main("hello")` 而非命令行参数
• **全局命名空间** - 单元格中定义的函数全局持久化，无需单元格间模块导入
• **BAML 设置可选** - 仅在引入 BAML 时使用 `baml_setup: true` 步骤（第 1 章+）
• **get_baml_client() 模式** - Google Colab 导入缓存问题的必需变通方法
• **来自 GitHub 的 BAML 文件** - 使用 curl 获取，因为 Colab 无法显示本地 BAML 文件
• **重新生成 BAML** - 在 BAML 文件更改时在 run_main 中使用 `regenerate_baml: true`
• **导入移除** - 从 Python 文件移除 `from baml_client import get_baml_client` 导入
• **IN_COLAB 检测** - 使用 try/except 在 google.colab 导入上检测环境
• **人类输入处理** - get_human_input() 在 Colab 中使用真实 input()，本地自动响应

## 实现模式

• **walkthroughgen_py.py 增强** - 为 run_main 步骤添加 kwargs 支持
• **测试模拟** - test_notebook_colab_sim.sh 创建带有所有依赖的干净 venv
• **调试工件** - 测试运行保存在 ./tmp/test_TIMESTAMP/ 目录中
• **BAML 测试支持** - baml-cli test 在 Notebook 中工作良好，与初始假设相反
• **工具执行** - 代理循环中的所有计算器操作（加/减/乘/除）
• **澄清流** - ClarificationRequest 工具用于处理模糊输入
• **序列化格式** - 线程历史使用 JSON vs XML（XML 更令牌高效）
• **渐进复杂性** - 从 Hello World 开始，逐步添加 BAML、工具、循环、测试

## 章节实现状态

• **第 0 章**：Hello World - 简单 Python 程序，无 BAML ✅
• **第 1 章**：CLI 和代理 - BAML 介绍，基本代理 ✅
• **第 2 章**：计算器工具 - 无执行的工具定义 ✅
• **第 3 章**：工具循环 - 带有工具执行的完整代理循环 ✅
• **第 4 章**：BAML 测试 - 带有断言的测试用例 ✅
• **第 5 章**：人类工具 - 带有输入处理的澄清请求 ✅
• **第 6 章**：改进提示 - 提示中的推理步骤 ✅
• **第 7 章**：上下文序列化 - JSON/XML 线程格式 ✅
• **第 8-12 章**：跳过 - 服务器基于的功能不适合 Notebook ⚠️

## 避免的常见陷阱

• **导入错误** - Notebook 中 baml_client 导入失败，使用全局 get_baml_client
• **异步模式** - Notebook 无法处理 async/await，一切必须同步
• **文件路径** - 使用 Notebook 目录的绝对路径，处理 ./ 前缀
• **BAML 文件冲突** - 每个章节更新相同文件 (agent.baml) 而非章节特定
• **工具注册** - 确保代理循环 switch 语句中处理所有工具类型
• **测试期望** - BAML 测试可能有变化输出，断言验证关键属性
• **环境差异** - 代码必须在 Colab 和本地测试环境中工作

## 测试命令

• 生成 Notebook：`uv run python walkthroughgen_py.py walkthrough.yaml -o test.ipynb`
• 完整 Colab 模拟：`./test_notebook_colab_sim.sh`
• 运行 BAML 测试：`baml-cli test`（从带有 baml_src 的目录）

## 文件结构

• `walkthrough/*.py` - 每个章节代码的 Python 实现
• `walkthrough/*.baml` - Notebook 执行期间从 GitHub 获取的 BAML 文件
• `walkthroughgen_py.py` - 主要转换工具
• `walkthrough.yaml` - 所有章节的 Notebook 定义
• `test_notebook_colab_sim.sh` - 完整 Colab 环境模拟
• `workshop_final.ipynb` - 最终生成的工作坊就绪 Notebook