# Jupyter Notebook 测试框架

此文档描述了验证 Jupyter Notebook 中任何功能的通用测试框架，并以测试 BAML 日志捕获为例。

## 通用框架

### 概述

测试框架为测试 Notebook 实现提供完整的迭代循环：

1. **生成**具有特定功能的测试 Notebook
2. **执行** Notebook 在模拟的 Google Colab 环境中
3. **分析** 执行的 Notebook 以获取预期输出和行为
4. **报告** 清晰的通过/失败结果

### 核心组件

#### Notebook 模拟器 (`test_notebook_colab_sim.sh`)

模拟脚本为任何 Notebook 创建真实的 Google Colab 环境：

**环境设置：**
- 创建时间戳测试目录：`./tmp/test_YYYYMMDD_HHMMSS/`
- 设置新的 Python 虚拟环境
- 安装 Jupyter 依赖 (`notebook`, `nbconvert`, `ipykernel`)

**Notebook 执行：**
- 将测试 Notebook 复制到干净环境
- 使用 `ExecutePreprocessor` 运行所有单元格（模拟 Colab 执行）
- **关键**：在执行前激活虚拟环境
- **关键**：将执行的 Notebook 与单元格输出保存回磁盘

**使用：**
```bash
./test_notebook_colab_sim.sh your_notebook.ipynb
```

模拟器将：
- 执行 Notebook 中的所有单元格
- 保留测试目录以供检查
- 显示最终目录结构
- 报告成功/失败

#### 输出检查器 (`inspect_notebook.py`)

用于详细检查 Notebook 单元格输出的调试实用程序：

**功能：**
- 显示单元格源代码和执行计数
- 显示所有输出类型 (stream, execute_result, error)
- 在输出文本中突出模式
- 显示执行错误与回溯
- 通过关键字过滤单元格以进行专注调试

**使用：**
```bash
# 检查所有单元格
python3 inspect_notebook.py path/to/notebook.ipynb

# 过滤特定内容
python3 inspect_notebook.py path/to/notebook.ipynb "keyword"

# 查找错误
python3 inspect_notebook.py path/to/notebook.ipynb "error"
```

**示例输出：**
```
🔍 CELL 0 (code)
📝 SOURCE:
import sys
print("Hello!")
print("Error!", file=sys.stderr)

📤 OUTPUTS (2 outputs):
  Output 0: type=stream
    Text length: 7 chars
    > Hello!...
  Output 1: type=stream
    Text length: 7 chars
    > Error!...
    🎯 Found patterns: ['Error']
```

### Notebook 测试的关键见解

#### 执行环境
1. **虚拟环境激活至关重要** - 没有它，执行会静默失败
2. **输出持久化必须明确** - `ExecutePreprocessor` 只在内存中修改 Notebook
3. **检查执行计数** - `execution_count=None` 表示单元格从未执行
4. **处理不同输出类型** - stream, execute_result, error, display_data

#### 常见调试步骤
1. **验证基本执行：**
   ```bash
   python3 -c "
   import json
   nb = json.load(open('path/to/notebook.ipynb'))
   print('Execution counts:', [cell.get('execution_count') for cell in nb['cells'] if cell['cell_type']=='code'])
   "
   ```

2. **检查执行错误：**
   ```bash
   python3 inspect_notebook.py path/to/notebook.ipynb "error"
   ```

3. **查找特定输出模式：**
   ```bash
   python3 inspect_notebook.py path/to/notebook.ipynb "your_pattern"
   ```

### 创建自定义测试

#### 1. 最小测试模板

创建一个简单的 Notebook 测试基本功能：

```json
{
  "cells": [
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {},
      "outputs": [],
      "source": [
        "# Test basic execution\n",
        "print('Hello from notebook!')\n",
        "\n",
        "# Test file creation\n",
        "with open('test.txt', 'w') as f:\n",
        "    f.write('Test successful\\n')\n",
        "\n",
        "# Test error handling\n",
        "try:\n",
        "    result = your_function_to_test()\n",
        "    print(f'Result: {result}')\n",
        "except Exception as e:\n",
        "    print(f'Error: {e}')"
      ]
    }
  ],
  "metadata": {
    "kernelspec": {
      "display_name": "Python 3",
      "language": "python",
      "name": "python3"
    }
  },
  "nbformat": 4,
  "nbformat_minor": 4
}
```

#### 2. 测试脚本模板

```bash
#!/bin/bash
set -e

echo "🧪 Testing [Your Feature]..."

# 清理之前的测试
rm -f test_notebook.ipynb

# 生成或复制你的测试 Notebook
cp your_test_notebook.ipynb test_notebook.ipynb

# 在模拟器中运行
echo "🚀 Running test in sim..."
./test_notebook_colab_sim.sh test_notebook.ipynb

# 查找执行的 Notebook
NOTEBOOK_DIR=$(ls -1dt tmp/test_* | head -1)
NOTEBOOK_PATH="$NOTEBOOK_DIR/test_notebook.ipynb"

# 分析结果
echo "📋 Analyzing results..."
python3 inspect_notebook.py "$NOTEBOOK_PATH" "your_search_term"

# 添加你的自定义分析
python3 -c "
import json
with open('$NOTEBOOK_PATH') as f:
    nb = json.load(f)

# 你的自定义分析逻辑在这里
success = check_for_expected_outputs(nb)

if success:
    print('✅ PASS: Test succeeded!')
else:
    print('❌ FAIL: Test failed!')
    exit(1)
"

echo "🧹 Cleaning up..."
rm -f test_notebook.ipynb
```

---

## 用例：BAML 日志捕获测试

此部分演示如何使用通用框架进行特定用例：测试 Notebook 中的 BAML 日志捕获。

### 问题陈述

BAML（一个语言模型框架）使用 FFI 绑定到 Rust 二进制文件，并将日志输出到 stderr。我们需要测试不同的日志捕获方法是否能在 Jupyter Notebook 单元格中成功捕获这些日志。

### 测试实现

#### 测试配置 (`simple_log_test.yaml`)

```yaml
title: "BAML Log Capture Test"
text: "Simple test for log capture"

sections:
  - title: "Log Capture Test"
    steps:
      - baml_setup: true
      - fetch_file:
          src: "walkthrough/01-agent.baml"
          dest: "baml_src/agent.baml"
      - file:
          src: "./simple_main.py"
      - text: "Testing log capture with show_logs=true:"
      - run_main:
          args: "What is 2+2?"
          show_logs: true
```

#### 测试函数 (`simple_main.py`)

```python
def main(message="What is 2+2?"):
    """Simple main function that calls BAML directly"""
    client = get_baml_client()

    # Call the BAML function - this should generate logs
    result = client.DetermineNextStep(f"User asked: {message}")

    print(f"Input: {message}")
    print(f"Result: {result}")
    return result
```

#### 日志捕获实现

当前在 `walkthroughgen_py.py` 中的工作实现：

```python
def run_with_baml_logs(func, *args, **kwargs):
    """Test log capture using IPython capture_output"""
    # Ensure BAML_LOG is set
    if 'BAML_LOG' not in os.environ:
        os.environ['BAML_LOG'] = 'info'

    print(f"[LOG CAPTURE TEST] Running with BAML_LOG={os.environ.get('BAML_LOG')}...")

    # Capture both stdout and stderr
    with capture_output() as captured:
        result = func(*args, **kwargs)

    # Display captured outputs
    if captured.stdout:
        print("=== Captured Stdout ===")
        print(captured.stdout)

    if captured.stderr:
        print("=== Captured BAML Logs ===")
        print(captured.stderr)
    else:
        print("=== No BAML Logs Captured ===")

    print("=== Function Result ===")
    print(result)

    return result
```

### 测试执行

#### 主测试脚本 (`test_log_capture.sh`)

```bash
#!/bin/bash
set -e

echo "🧪 Testing BAML Log Capture..."

# 从 YAML 配置生成测试 Notebook
echo "📝 Generating test notebook..."
uv run python walkthroughgen_py.py simple_log_test.yaml -o test_capture.ipynb

# 在模拟器中运行
echo "🚀 Running test in sim..."
./test_notebook_colab_sim.sh test_capture.ipynb

# 查找执行的 Notebook
NOTEBOOK_DIR=$(ls -1dt tmp/test_* | head -1)
NOTEBOOK_PATH="$NOTEBOOK_DIR/test_notebook.ipynb"

echo "📋 Analyzing results from $NOTEBOOK_PATH..."

# 调试输出
echo "🔍 Dumping debug info..."
python3 inspect_notebook.py "$NOTEBOOK_PATH" "run_with_baml_logs"

# 分析 BAML 日志模式
echo "📊 Running log capture analysis..."
python3 analyze_log_capture.py "$NOTEBOOK_PATH"

echo "🧹 Cleaning up..."
rm -f test_capture.ipynb
```

#### 分析脚本 (`analyze_log_capture.py`)

```python
#!/usr/bin/env python3
import json
import sys
import os

def check_logs(notebook_path):
    """Check if BAML logs were captured in the notebook"""

    with open(notebook_path) as f:
        nb = json.load(f)

    found_log_pattern = False
    found_capture_test = False

    for i, cell in enumerate(nb['cells']):
        if cell['cell_type'] == 'code' and 'outputs' in cell:
            source = ''.join(cell.get('source', []))
            if 'run_with_baml_logs' in source:
                found_capture_test = True
                print(f'Found log capture test in cell {i}')

                # Check outputs for BAML logs
                for output in cell['outputs']:
                    if output.get('output_type') == 'stream' and 'text' in output:
                        text = ''.join(output['text'])
                        # Look for the specific BAML log pattern
                        if '---Parsed Response (class DoneForNow)---' in text:
                            found_log_pattern = True
                            print(f'✅ FOUND BAML LOG PATTERN in cell {i} output!')

    return found_capture_test, found_log_pattern

# Run analysis and return pass/fail
capture_test_found, log_pattern_found = check_logs(sys.argv[1])

if not capture_test_found:
    print('❌ FAIL: No log capture test found in notebook')
    sys.exit(1)

if log_pattern_found:
    print('✅ PASS: BAML logs successfully captured in notebook output!')
    sys.exit(0)
else:
    print('❌ FAIL: BAML log pattern not found in captured output')
    sys.exit(1)
```

### 预期输出流

#### 成功测试运行：

```bash
$ ./test_log_capture.sh

🧪 Testing BAML Log Capture...
📝 Generating test notebook...
Generated notebook: test_capture.ipynb
🚀 Running test in sim...
🧪 Creating clean test environment in: ./tmp/test_20250716_191106
📁 Test directory will be preserved for inspection
🐍 Creating fresh Python virtual environment...
📦 Installing Jupyter dependencies...
🏃 Running notebook in clean environment...
✅ Notebook executed successfully!
💾 Executed notebook saved with outputs

📋 Analyzing results from tmp/test_20250716_191106/test_notebook.ipynb...
🔍 Dumping debug info...
Found log capture test in cell 11

📤 OUTPUTS (3 outputs):
  Output 0: type=stream
    Text length: 49 chars
    > [LOG CAPTURE TEST] Running with BAML_LOG=info......
  Output 1: type=stream
    Text length: 1272 chars
    > 2025-07-16T19:11:22.445 [BAML [92mINFO[0m] [35mFunction DetermineNextStep[0m...
    🎯 Found patterns: ['BAML', 'Parsed', 'Response']

📊 Running log capture analysis...
Found log capture test in cell 11
✅ FOUND BAML LOG PATTERN in cell 11 output!
✅ PASS: BAML logs successfully captured in notebook output!
🧹 Cleaning up...
```

### BAML 特定的关键见解

1. **BAML 日志输出到 stderr** - 由于 FFI 绑定到 Rust 二进制文件
2. **需要 `BAML_LOG=info`** - 环境变量控制详细程度
3. **日志包括 ANSI 颜色代码** - 需要处理终端格式
4. **模式匹配** - 查找 `---Parsed Response (class DoneForNow)---` 以确认成功执行
5. **IPython capture_output() 有效** - 成功在 Notebook 上下文中捕获 stderr

### 迭代循环好处

此框架启用快速测试不同日志捕获方法：

1. **修改** `walkthroughgen_py.py` 中的 `run_with_baml_logs` 函数
2. **运行** `./test_log_capture.sh`
3. **获取** 立即通过/失败反馈
4. **如需调试，使用** `inspect_notebook.py`
5. **重复** 直到找到工作实现

此相同模式可应用于测试任何 Notebook 功能：库集成、环境设置、输出格式、错误处理等。