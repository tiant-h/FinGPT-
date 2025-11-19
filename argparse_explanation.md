# argparse 详解

## 1. argparse 是什么？

**argparse** 是 Python 标准库中的一个模块，用于**解析命令行参数**。

### 为什么需要 argparse？

想象一下，你写了一个程序，但每次运行都需要修改代码来改变参数，这很不方便。argparse 让你可以通过命令行直接传入参数，就像这样：

```bash
# 不需要修改代码，直接在命令行传入参数
python script.py --name "Alice" --age 25
```

---

## 2. 基本概念

### 核心组件：

1. **ArgumentParser**：参数解析器对象
2. **add_argument()**：添加参数定义
3. **parse_args()**：解析命令行参数

---

## 3. 基本使用步骤

### 步骤1：创建解析器
```python
import argparse
parser = argparse.ArgumentParser()
```

### 步骤2：添加参数
```python
parser.add_argument('--name', type=str, help='用户姓名')
parser.add_argument('--age', type=int, default=18, help='用户年龄')
```

### 步骤3：解析参数
```python
args = parser.parse_args()
print(args.name)  # 访问参数值
print(args.age)
```

---

## 4. 代码中的实际例子分析

让我们看看 `fundamental_run_model.py` 中是如何使用 argparse 的：

### 例子1：必需参数（Required）
```python
# 第19行
parser.add_argument('-sector_name','--sector_name_input', 
                    type=str,  
                    required=True,
                    help='sector name: i.e. sector10')
```

**解释：**
- `-sector_name`：短参数名（可以用 `-sector_name`）
- `--sector_name_input`：长参数名（可以用 `--sector_name_input`）
- `type=str`：参数类型是字符串
- `required=True`：**必须提供**，不提供会报错
- `help='...'`：帮助信息

**使用方式：**
```bash
python script.py -sector_name sector10
# 或
python script.py --sector_name_input sector10
```

### 例子2：可选参数（Optional，有默认值）
```python
# 第27行
parser.add_argument("-first_trade_index", 
                    default=20, 
                    type=int)
```

**解释：**
- `-first_trade_index`：参数名
- `default=20`：**默认值是 20**，如果不提供就用默认值
- `type=int`：参数类型是整数
- 没有 `required=True`：所以是**可选的**

**使用方式：**
```bash
# 使用默认值（20）
python script.py -sector_name sector10 ...

# 自定义值
python script.py -sector_name sector10 -first_trade_index 25 ...
```

### 例子3：列表参数
```python
# 第33-34行
parser.add_argument("-no_feature_column_names", 
                    default=['gvkey', 'tic', 'datadate', ...], 
                    type=list,
                    help='column names that are not fundamental features')
```

**解释：**
- `default=[...]`：默认值是一个列表
- `type=list`：参数类型是列表

---

## 5. 参数类型详解

### 5.1 参数名称

```python
# 短参数名（一个减号）
parser.add_argument('-f', ...)

# 长参数名（两个减号）
parser.add_argument('--file', ...)

# 同时定义短参数和长参数
parser.add_argument('-f', '--file', ...)  # 两种方式都可以用
```

### 5.2 参数类型（type）

```python
parser.add_argument('--age', type=int)        # 整数
parser.add_argument('--name', type=str)       # 字符串
parser.add_argument('--price', type=float)    # 浮点数
parser.add_argument('--flag', type=bool)      # 布尔值
```

### 5.3 必需 vs 可选

```python
# 必需参数（必须提供）
parser.add_argument('--input', required=True)

# 可选参数（可以不提供，使用默认值）
parser.add_argument('--output', default='result.txt')
```

### 5.4 位置参数 vs 可选参数

```python
# 位置参数（不需要 - 或 --）
parser.add_argument('filename')  
# 使用：python script.py data.txt

# 可选参数（需要 - 或 --）
parser.add_argument('--output')  
# 使用：python script.py --output result.txt
```

---

## 6. 完整示例

### 示例1：简单程序
```python
import argparse

parser = argparse.ArgumentParser(description='处理数据的程序')
parser.add_argument('--input', type=str, required=True, help='输入文件')
parser.add_argument('--output', type=str, default='output.txt', help='输出文件')
parser.add_argument('--verbose', action='store_true', help='显示详细信息')

args = parser.parse_args()

print(f"输入文件: {args.input}")
print(f"输出文件: {args.output}")
if args.verbose:
    print("详细模式已开启")
```

**运行：**
```bash
python script.py --input data.csv --output result.txt --verbose
```

### 示例2：代码中的实际使用
```python
# 在 fundamental_run_model.py 中
args = parser.parse_args()  # 解析所有参数

# 使用参数
inputfile_fundamental = args.fundamental_input  # 获取参数值
sector_name = args.sector_name_input
first_trade_index = args.first_trade_index  # 如果没有提供，使用默认值 20
```

---

## 7. 常用参数选项

| 选项 | 说明 | 示例 |
|------|------|------|
| `type` | 参数类型 | `type=int`, `type=str` |
| `default` | 默认值 | `default=20` |
| `required` | 是否必需 | `required=True` |
| `help` | 帮助信息 | `help='输入文件名'` |
| `choices` | 可选值列表 | `choices=['a', 'b', 'c']` |
| `action` | 动作 | `action='store_true'`（用于布尔标志） |

### action 的常见用法：

```python
# 布尔标志（不需要值，存在就是 True）
parser.add_argument('--verbose', action='store_true')
# 使用：python script.py --verbose  # args.verbose = True
# 不使用：python script.py          # args.verbose = False

# 计数（出现次数）
parser.add_argument('--count', action='count')
# 使用：python script.py --count --count --count  # args.count = 3
```

---

## 8. 获取帮助信息

argparse 自动生成帮助信息：

```bash
# 查看所有参数的帮助
python fundamental_run_model.py -h
# 或
python fundamental_run_model.py --help
```

**输出示例：**
```
usage: fundamental_run_model.py [-h] -sector_name SECTOR_NAME_INPUT 
                                -fundamental FUNDAMENTAL_INPUT 
                                -sector SECTOR_INPUT 
                                [-first_trade_index FIRST_TRADE_INDEX]
                                ...

optional arguments:
  -h, --help            show this help message and exit
  -sector_name SECTOR_NAME_INPUT, --sector_name_input SECTOR_NAME_INPUT
                        sector name: i.e. sector10
  ...
```

---

## 9. 实际运行示例

基于代码中的参数，完整的运行命令：

```bash
python3 fundamental_run_model.py \
  -sector_name sector10 \
  -fundamental Data/fundamental_final_table.xlsx \
  -sector Data/1-focasting_data/sector10_clean.xlsx \
  -first_trade_index 20 \
  -testing_window 4 \
  -label_column y_return \
  -date_column date \
  -tic_column tic
```

**或者使用长参数名：**
```bash
python3 fundamental_run_model.py \
  --sector_name_input sector10 \
  --fundamental_input Data/fundamental_final_table.xlsx \
  --sector_input Data/1-focasting_data/sector10_clean.xlsx
```

---

## 10. 总结

### argparse 的优势：

1. ✅ **用户友好**：自动生成帮助信息
2. ✅ **类型检查**：自动验证参数类型
3. ✅ **灵活配置**：支持必需/可选参数、默认值
4. ✅ **标准化**：Python 标准库，无需安装
5. ✅ **错误处理**：自动处理参数错误并提示

### 核心流程：

```
定义参数 → 解析参数 → 使用参数
   ↓           ↓           ↓
add_argument  parse_args  args.参数名
```

### 类比理解：

- **argparse** 就像餐厅的菜单
- **add_argument** 就像在菜单上添加菜品
- **parse_args** 就像服务员记录你点的菜
- **args** 就像你的订单，可以查看你点了什么

---

## 11. 常见错误和解决方案

### 错误1：缺少必需参数
```bash
python script.py  # 缺少 -sector_name
# 错误：error: the following arguments are required: -sector_name/--sector_name_input
```

**解决：** 提供所有必需参数

### 错误2：类型不匹配
```bash
python script.py -first_trade_index abc  # 应该是整数
# 错误：invalid int value: 'abc'
```

**解决：** 提供正确类型的值

### 错误3：未知参数
```bash
python script.py --unknown_param value
# 错误：unrecognized arguments: --unknown_param
```

**解决：** 只使用已定义的参数
