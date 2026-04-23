---
title: Numpy
tags: ndarray
categories:
  - Python
  - 数据分析
date: 2026-04-10 08:53:18
---

# 一、NumPy简介

## 1\.1 什么是NumPy

NumPy（Numerical Python）是Python中用于科学计算的核心库，主要用于处理高维数组（ndarray），并提供了大量的数学函数、线性代数、傅里叶变换、随机数生成等功能，是Pandas、Matplotlib等数据分析工具的基础。

<!--more-->

核心优势：

- 高效的数组运算：基于C语言实现，运算速度远快于Python原生列表

- 简洁的API：提供直观的函数，简化数据处理流程

- 多维度支持：轻松处理1维、2维及更高维度的数组

- 与其他库无缝衔接：是数据分析、机器学习的必备工具

## 1\.2 安装NumPy

使用pip命令安装（最常用）：

```python
# 安装最新版本
pip install numpy

# 安装指定版本（如1.24.3，适配多数数据分析环境）
pip install numpy==1.24.3
```

验证安装是否成功：

```python
import numpy as np
print(np.__version__)  # 输出版本号，如1.24.3即安装成功
```

# 二、NumPy核心：ndarray数组

## 2\.1 什么是ndarray

ndarray（N\-dimensional array）是NumPy的核心数据结构，是一个多维的、同类型元素的数组（类比Python列表，但更高效、更强大）。

与Python列表的区别：

- 列表可以存储不同类型的数据（如int、str、float），ndarray默认存储同类型数据

- ndarray支持广播机制、向量运算，运算效率远高于列表

- ndarray有明确的维度（shape）和数据类型（dtype）

## 2\.2 创建ndarray数组

### 2\.2\.1 从Python列表/元组创建（最常用）

```python
import numpy as np

# 1. 创建1维数组（从列表）
arr1 = np.array([1, 2, 3, 4, 5])
print("1维数组：", arr1)
print("数组维度：", arr1.ndim)  # 输出1（ndim查看维度）
print("数组形状：", arr1.shape)  # 输出(5,)（shape查看各维度元素个数）
print("数据类型：", arr1.dtype)  # 输出int64（dtype查看数据类型）

# 2. 创建2维数组（从嵌套列表）
arr2 = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
print("\n2维数组：", arr2)
print("数组维度：", arr2.ndim)  # 输出2
print("数组形状：", arr2.shape)  # 输出(3, 3)（3行3列）
print("数组元素总数：", arr2.size)  # 输出9（size查看元素总数）

# 3. 指定数据类型创建
arr3 = np.array([1.1, 2.2, 3.3], dtype=np.int32)  # 强制转为int32类型
print("\n指定dtype的数组：", arr3)  # 输出[1 2 3]
print("数据类型：", arr3.dtype)  # 输出int32
```

### 2\.2\.2 快速创建特殊数组

```python
import numpy as np

# 1. 全0数组（常用作初始化）
zero_arr = np.zeros((2, 3))  # 形状为(2行3列)的全0数组，默认dtype为float64
print("全0数组：\n", zero_arr)

# 2. 全1数组
one_arr = np.ones((3, 2), dtype=np.int32)  # 形状为(3行2列)的全1数组，指定int32
print("\n全1数组：\n", one_arr)

# 3. 单位矩阵（对角线为1，其余为0，仅2维有效）
eye_arr = np.eye(3)  # 3x3的单位矩阵
print("\n单位矩阵：\n", eye_arr)

# 4. 等差数列（类似range，但支持浮点数）
arange_arr = np.arange(1, 10, 2)  # 从1开始，到10结束（不包含10），步长2
print("\n等差数列：", arange_arr)  # 输出[1 3 5 7 9]

# 5. 均匀分布的等差数列（指定个数）
linspace_arr = np.linspace(0, 10, 5)  # 从0到10，分成5个均匀的数
print("\n均匀等差数列：", linspace_arr)  # 输出[ 0.  2.5  5.  7.5 10. ]

# 6. 随机数组（0-1之间的随机浮点数）
random_arr = np.random.random((2, 2))  # 2x2的随机数组
print("\n随机数组（0-1）：\n", random_arr)
```

## 2\.3 数组的索引与切片

ndarray的索引和切片与Python列表类似，但支持多维索引，语法更灵活。

```python
import numpy as np

# 1. 1维数组的索引与切片（和列表完全一致）
arr1 = np.arange(10)  # [0 1 2 3 4 5 6 7 8 9]
print("索引第3个元素（下标2）：", arr1[2])  # 输出2
print("切片[2:6]（左闭右开）：", arr1[2:6])  # 输出[2 3 4 5]
print("切片[::2]（步长2）：", arr1[::2])  # 输出[0 2 4 6 8]
print("反向切片[::-1]（倒序）：", arr1[::-1])  # 输出[9 8 7 6 5 4 3 2 1 0]

# 2. 2维数组的索引与切片（行优先）
arr2 = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
print("\n2维数组：\n", arr2)

# 索引单个元素（行下标, 列下标）
print("索引(1,2)位置元素：", arr2[1, 2])  # 输出6（第2行第3列，下标从0开始）

# 切片：取第1行（下标0）的所有列
print("第1行所有元素：", arr2[0, :])  # 输出[1 2 3]

# 切片：取所有行的第2列（下标1）
print("所有行的第2列：", arr2[:, 1])  # 输出[2 5 8]

# 切片：取前2行、前2列（切片区间左闭右开）
print("前2行前2列：\n", arr2[:2, :2])  # 输出[[1 2],[4 5]]

# 3. 布尔索引（常用於筛选数据）
arr3 = np.array([1, 2, 3, 4, 5, 6])
mask = arr3 > 3  # 生成布尔数组[False False False  True  True  True]
print("\n筛选出大于3的元素：", arr3[mask])  # 输出[4 5 6]
```

# 三、NumPy核心运算

## 3\.1 数组与标量的运算

NumPy数组支持与标量（单个数字）的加减乘除、幂运算等，会自动对数组中每个元素进行运算（广播机制）。

```python
import numpy as np

arr = np.array([1, 2, 3, 4, 5])

# 加减乘除
print("数组+2：", arr + 2)  # 输出[3 4 5 6 7]
print("数组-1：", arr - 1)  # 输出[0 1 2 3 4]
print("数组*3：", arr * 3)  # 输出[3 6 9 12 15]
print("数组/2：", arr / 2)  # 输出[0.5 1.  1.5 2.  2.5]

# 幂运算
print("数组的2次方：", arr ** 2)  # 输出[1 4 9 16 25]

# 取余
print("数组对2取余：", arr % 2)  # 输出[1 0 1 0 1]
```

## 3\.2 数组与数组的运算

同形状的数组之间，会对应元素进行运算；不同形状的数组会触发广播机制（需满足广播条件）。

```python
import numpy as np

# 1. 同形状数组运算（对应元素运算）
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])

print("arr1 + arr2：", arr1 + arr2)  # 输出[5 7 9]
print("arr1 * arr2：", arr1 * arr2)  # 输出[4 10 18]
print("arr1 / arr2：", arr1 / arr2)  # 输出[0.25 0.4  0.5]

# 2. 广播机制（不同形状数组运算）
# 条件：从后往前比较维度,相等或其中一个为1可广播,维度数不同时,少的前面补1
arr3 = np.array([[1, 2, 3], [4, 5, 6]])  # 形状(2,3)
arr4 = np.array([10, 20, 30])  # 形状(3,)
print("\n广播运算 arr3 + arr4：\n", arr3 + arr4)
# 输出：[[11 22 33],[14 25 36]]（arr4自动扩展为(2,3)，与arr3对应元素相加）
```

## 3\.3 常用数学函数

NumPy提供了大量内置数学函数，无需循环，直接作用于整个数组，效率极高。

```python
import numpy as np

arr = np.array([1, 2, 3, 4, 5])

# 1. 基础数学函数
print("绝对值：", np.abs(arr))  # 输出[1 2 3 4 5]（负数才会变化）
print("平方根：", np.sqrt(arr))  # 输出[1.  1.4142 1.732  2.  2.2361]
print("正弦值：", np.sin(arr))  # 输出各元素的正弦值（弧度制）
print("余弦值：", np.cos(arr))  # 输出各元素的余弦值
print("指数值（e^x）：", np.exp(arr))  # 输出[e^1, e^2, ..., e^5]
print("对数（ln）：", np.log(arr))  # 输出各元素的自然对数

# 2. 统计类函数
print("\n数组求和：", np.sum(arr))  # 输出15
print("数组均值：", np.mean(arr))  # 输出3.0
print("数组中位数：", np.median(arr))  # 输出3.0
print("数组最大值：", np.max(arr))  # 输出5
print("数组最小值：", np.min(arr))  # 输出1
print("最大值索引：", np.argmax(arr))  # 输出4（最大值5的下标）
print("最小值索引：", np.argmin(arr))  # 输出0（最小值1的下标）

# 3. 2维数组的统计（可指定轴axis）
arr2 = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
print("\n2维数组求和（axis=0，按列求和）：", np.sum(arr2, axis=0))  # 输出[12 15 18]
print("2维数组求和（axis=1，按行求和）：", np.sum(arr2, axis=1))  # 输出[6 15 24]
```

# 四、数组的形状操作

在数据分析中，经常需要调整数组的形状（如将1维数组转为2维，或合并、拆分数组）。

```python
import numpy as np

# 1. reshape：调整数组形状（元素总数不变）
arr1 = np.arange(12)  # [0 1 2 3 4 5 6 7 8 9 10 11]
arr1_reshape = arr1.reshape(3, 4)  # 转为3行4列的2维数组
print("reshape后：\n", arr1_reshape)
print("reshape后的形状：", arr1_reshape.shape)  # 输出(3,4)

# 2. flatten：将多维数组转为1维数组（拷贝，修改新数组不影响原数组）
arr2 = np.array([[1, 2, 3], [4, 5, 6]])
arr2_flatten = arr2.flatten()
print("\nflatten后：", arr2_flatten)  # 输出[1 2 3 4 5 6]

# 3. ravel：将多维数组转为1维数组（视图，修改新数组会影响原数组）
arr2_ravel = arr2.ravel()
arr2_ravel[0] = 100  # 修改新数组
print("ravel后修改元素，原数组：\n", arr2)  # 原数组第1个元素变为100

# 4. 转置（行变列，列变行）
arr3 = np.array([[1, 2, 3], [4, 5, 6]])
arr3_transpose = arr3.T  # 或np.transpose(arr3)
print("\n转置后：\n", arr3_transpose)  # 输出[[1 4],[2 5],[3 6]]

# 5. 合并数组（concatenate）
arr4 = np.array([[1, 2], [3, 4]])
arr5 = np.array([[5, 6], [7, 8]])
# 按行合并（axis=0，默认）
merge_row = np.concatenate([arr4, arr5], axis=0)
print("\n按行合并：\n", merge_row)  # 输出[[1 2],[3 4],[5 6],[7 8]]
# 按列合并（axis=1）
merge_col = np.concatenate([arr4, arr5], axis=1)
print("按列合并：\n", merge_col)  # 输出[[1 2 5 6],[3 4 7 8]]

# 6. 拆分数组（split）
arr6 = np.arange(10)
# 拆分为3个部分，指定拆分位置
split_arr = np.split(arr6, [3, 7])
print("\n拆分后：", split_arr)  # 输出[array([0,1,2]), array([3,4,5,6]), array([7,8,9])]
```

# 五、NumPy实战小案例

用NumPy实现简单的数据分析场景：生成随机数据，计算统计指标，筛选异常值。

```python
import numpy as np

# 案例：分析学生成绩（生成50个学生的数学成绩，范围[60, 100]）
# 1. 生成随机成绩（整数）
scores = np.random.randint(60, 101, size=50)  # 50个[60,100]的随机整数
print("50个学生数学成绩：", scores)

# 2. 计算统计指标
print("\n成绩统计：")
print("平均分：", np.mean(scores))
print("最高分：", np.max(scores))
print("最低分：", np.min(scores))
print("成绩中位数：", np.median(scores))
print("成绩标准差：", np.std(scores))  # 标准差，反映成绩离散程度

# 3. 筛选异常值（假设低于70分为需要关注的成绩）
low_scores = scores[scores < 70]
print("\n需要关注的成绩（低于70分）：", low_scores)
print("需要关注的学生人数：", len(low_scores))

# 4. 成绩分组（70以下、70-80、80-90、90以上）
group1 = len(scores[scores < 70])
group2 = len(scores[(scores >= 70) & (scores < 80)])
group3 = len(scores[(scores >= 80) & (scores < 90)])
group4 = len(scores[scores >= 90])
print("\n成绩分组统计：")
print("70分以下：", group1, "人")
print("70-80分：", group2, "人")
print("80-90分：", group3, "人")
print("90分以上：", group4, "人")
```

# 六、常见问题与注意事项

- 数据类型问题：ndarray默认数据类型为float64/int64，若需指定类型（如int32、float32），创建时加上dtype参数，避免内存浪费。

- 广播机制：不同形状数组运算时，需确保满足广播条件（一个数组的维度是另一个的子集），否则会报错。

- 视图与拷贝：ravel返回视图（修改影响原数组），flatten返回拷贝（修改不影响原数组），按需选择。

- 索引越界：ndarray的索引从0开始，超出下标范围会报错，需注意数组形状。

- 与Python列表的区别：尽量使用ndarray进行数值运算，避免用列表循环，提升效率。

# 七、总结

NumPy是Python数据分析的基础，核心是ndarray数组，重点掌握：

1. ndarray的创建方法（列表创建、特殊数组创建）；

2. 数组的索引、切片、布尔索引（数据筛选的核心）；

3. 数组的运算（标量运算、数组运算、广播机制）；

4. 常用数学函数和统计函数；

5. 数组的形状操作（reshape、转置、合并、拆分）。

后续学习Pandas、Matplotlib时，会频繁用到NumPy的知识点，建议多练习代码，熟练掌握核心操作。
