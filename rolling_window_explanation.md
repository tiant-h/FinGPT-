# 滚动窗口（Rolling Window）预测流程详解

## 你的理解是正确的！

**是的，因为需要4年训练 + 1年测试，所以第一个预测结果只能从第5年（第20个季度）开始。**

---

## 1. 滚动窗口的基本设置

根据代码中的配置：

```python
# 从 fundamental_run_model.py 第56-59行
# train: 4 years = 16 quarters  (训练数据：4年 = 16个季度)
# test: 1 year = 4 quarters     (测试数据：1年 = 4个季度)
# so first trade date = #20 quarter  (所以第一个交易日期是第20个季度)
```

**关键参数：**
- `first_trade_date_index = 20`：第一个交易日期索引
- `testing_windows = 4`：测试窗口大小（4个季度 = 1年）
- `max_rolling_window_index = 44`：最大滚动窗口索引（44个季度 = 11年）

---

## 2. 数据时间线可视化

假设你有20年的数据（80个季度，Q1-Q80）：

```
时间轴（季度）：
Q1  Q2  Q3  Q4  Q5  Q6  Q7  Q8  Q9  Q10 Q11 Q12 Q13 Q14 Q15 Q16 Q17 Q18 Q19 Q20 Q21 Q22 ...
│   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │
└─────────── 训练数据（16个季度） ───────────┘ └─ 测试（4个季度） ─┘ │
                                                                      ↑
                                                              第一个预测日期（Q20）
```

### 第一个预测周期（i=20）：

**训练数据：** Q1 到 Q16（16个季度 = 4年）
```python
# prepare_rolling_train 函数逻辑
train = df[(df[date_column] >= unique_datetime[0])  # Q1开始
         & (df[date_column] < unique_datetime[20-4])]  # Q16结束（Q20-4=Q16）
```

**测试数据：** Q17 到 Q19（3个季度，用于验证模型）
```python
# prepare_rolling_test 函数逻辑
test = df[(df[date_column] >= unique_datetime[20-4])  # Q17开始
        & (df[date_column] < unique_datetime[20])]     # Q19结束（Q20之前）
```

**交易预测：** Q20（第一个预测日期）
```python
# prepare_trade_data 函数逻辑
trade = df[df[date_column] == unique_datetime[20]]  # Q20的数据
```

---

## 3. 完整的滚动窗口流程

### 循环从 `first_trade_date_index` 开始：

```python
# 从 ml_model.py 第309行
for i in range(first_trade_date_index, len(unique_datetime)):
    # i = 20, 21, 22, ..., 79
```

### 每个时间点的数据划分：

#### **当 i = 20（第一个预测点）：**
```
训练：Q1-Q16  (16个季度)
测试：Q17-Q19 (3个季度，用于验证)
预测：Q20     (第一个交易预测)
```

#### **当 i = 21（第二个预测点）：**
```
训练：Q2-Q17  (16个季度，窗口向前滚动)
测试：Q18-Q20 (3个季度)
预测：Q21     (第二个交易预测)
```

#### **当 i = 22（第三个预测点）：**
```
训练：Q3-Q18  (16个季度)
测试：Q19-Q21 (3个季度)
预测：Q22     (第三个交易预测)
```

#### **当 i = 44（第25个预测点，达到最大窗口）：**
```
训练：Q29-Q44  (16个季度，使用固定窗口大小)
测试：Q41-Q43  (3个季度)
预测：Q44      (交易预测)
```

#### **当 i = 45（第26个预测点，使用固定窗口）：**
```
训练：Q30-Q45  (16个季度，窗口大小固定为16)
测试：Q42-Q44  (3个季度)
预测：Q45      (交易预测)
```

---

## 4. 代码逻辑详解

### `prepare_rolling_train` 函数（第34-44行）：

```python
def prepare_rolling_train(..., current_index):
    if current_index <= max_rolling_window_index:  # i <= 44
        # 使用从开始到 current_index-testing_windows 的所有数据
        train = df[(df[date_column] >= unique_datetime[0])  # 从Q1开始
                 & (df[date_column] < unique_datetime[current_index-testing_windows])]
    else:  # i > 44
        # 使用固定大小的滚动窗口（16个季度）
        train = df[(df[date_column] >= unique_datetime[current_index-max_rolling_window_index])
                 & (df[date_column] < unique_datetime[current_index-testing_windows])]
```

**关键点：**
- 当 `i <= 44` 时：使用**累积数据**（从Q1开始，逐渐增加）
- 当 `i > 44` 时：使用**固定窗口**（只使用最近16个季度）

### `prepare_rolling_test` 函数（第46-51行）：

```python
def prepare_rolling_test(..., current_index):
    # 测试数据：current_index-testing_windows 到 current_index-1
    test = df[(df[date_column] >= unique_datetime[current_index-testing_windows])
            & (df[date_column] < unique_datetime[current_index])]
```

**测试窗口：** 总是使用预测日期之前的4个季度（实际是3个季度，因为不包含预测日期本身）

---

## 5. 为什么前4年没有预测？

### 原因分析：

1. **需要训练数据：** 模型需要至少16个季度（4年）的历史数据来训练
2. **需要测试数据：** 在预测之前，需要4个季度（1年）的数据来验证模型性能
3. **时间序列要求：** 不能使用未来数据预测过去（数据泄露）

### 数据使用情况：

```
年份1-4（Q1-Q16）：    → 仅用于训练（没有预测）
年份5（Q17-Q20）：     → Q17-Q19用于测试，Q20是第一个预测
年份6-20（Q21-Q80）：  → 滚动窗口预测
```

---

## 6. 实际例子：20年数据

假设你有20年的季度数据（80个季度）：

| 季度范围 | 用途 | 说明 |
|---------|------|------|
| Q1-Q16 | 训练数据 | 前4年，仅用于训练第一个模型 |
| Q17-Q19 | 测试数据 | 第5年的前3个季度，用于验证第一个模型 |
| **Q20** | **第一个预测** | **第5年第4个季度，第一个交易预测** |
| Q21-Q79 | 滚动预测 | 后续所有季度的预测 |
| Q80 | 最后一个预测 | 第20年最后一个季度 |

**结果：**
- ✅ **有预测的季度：** Q20-Q80（61个季度，约15.25年）
- ❌ **没有预测的季度：** Q1-Q19（19个季度，约4.75年）

---

## 7. 如何调整以获得更多预测？

如果你想从更早的时间开始预测，可以：

### 选项1：减少训练窗口大小
```python
# 修改 max_rolling_window_index
# 例如：使用2年训练（8个季度）而不是4年（16个季度）
# 这样第一个预测可以从第3年开始
```

### 选项2：减少测试窗口大小
```python
# 修改 testing_windows
# 例如：使用2个季度测试而不是4个季度
# 这样第一个预测可以提前2个季度
```

### 选项3：调整 first_trade_date_index
```python
# 直接修改 first_trade_date_index
# 但要注意：如果训练数据不足，模型性能会下降
```

**权衡：**
- ⚠️ 训练数据太少 → 模型性能下降
- ⚠️ 测试数据太少 → 模型验证不充分
- ⚠️ 过早预测 → 可能过拟合早期数据

---

## 8. 总结

### 你的理解完全正确！

1. ✅ **前4年（Q1-Q16）**：仅用于训练，**没有预测**
2. ✅ **第5年（Q17-Q20）**：
   - Q17-Q19：用于测试验证
   - **Q20：第一个预测结果**
3. ✅ **第6-20年（Q21-Q80）**：使用滚动窗口进行预测

### 关键代码位置：

- **训练数据准备：** `ml_model.py` 第34-44行
- **测试数据准备：** `ml_model.py` 第46-51行
- **预测数据准备：** `ml_model.py` 第53-58行
- **循环开始：** `ml_model.py` 第309行
- **参数设置：** `fundamental_run_model.py` 第56-60行

### 数据损失：

- **损失的前4.75年**是时间序列预测的**必要成本**
- 这是为了确保：
  - 有足够的训练数据
  - 有足够的验证数据
  - 避免数据泄露（使用未来预测过去）

---

## 9. 可视化总结

```
20年数据（80个季度）的预测流程：

Q1-Q16:  [训练数据] ──────────────┐
Q17-Q19: [测试数据] ────┐          │
Q20:     [预测1] ←──────┘          │
Q21:     [预测2] ←─── 滚动窗口 ────┤
Q22:     [预测3] ←─── 滚动窗口 ────┤
...                                │
Q44:     [预测25] ←── 达到最大窗口 │
Q45:     [预测26] ←── 固定窗口 ────┤
...                                │
Q80:     [预测61] ←────────────────┘

结果：61个预测（Q20-Q80），约15.25年
```
