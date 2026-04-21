---
title: Pandas
tags:
  - Series
  - DataFrame
categories:
  - Python
  - 数据分析
date: 2026-04-11 13:55:00
---

# 一、Pandas简介

## 1\.1 什么是Pandas

Pandas（Python Data Analysis Library）是基于NumPy开发的Python数据分析库，专门用于处理结构化数据（如表格数据、CSV文件、Excel文件等），提供了直观、高效的数据读取、清洗、筛选、分组、聚合等功能，是数据分析、数据挖掘的核心工具。

<!--more-->

Pandas与NumPy的关系：

- Pandas基于NumPy实现，底层使用ndarray存储数据，继承了NumPy的高效运算能力；

- NumPy专注于多维数组运算，而Pandas专注于结构化数据处理，提供了更贴近实际业务的API；

- 两者协同使用，可高效完成从数据读取到分析的全流程。

核心数据结构：Series（一维结构化数据）、DataFrame（二维结构化数据，类似Excel表格）。

## 1\.2 安装Pandas

使用pip命令安装，建议同时安装openpyxl（用于读取Excel文件）：

```python
# 安装最新版本Pandas和openpyxl
pip install pandas openpyxl

# 安装指定版本（适配NumPy 1.24.3，避免版本冲突）
pip install pandas==1.5.3 openpyxl==3.1.2
```

验证安装是否成功：

```python
import pandas as pd
print(pd.__version__)  # 输出版本号，如1.5.3即安装成功
```

# 二、Pandas核心数据结构：Series

## 2\.1 什么是Series

Series是Pandas中用于存储一维结构化数据的核心结构，类似“带标签的数组”——由两部分组成：

- values：数据本身（底层是NumPy的ndarray）；

- index：数据的标签（索引），可自定义，默认是0、1、2\.\.\.的整数索引。

与NumPy ndarray的区别：Series有明确的索引，可通过标签索引数据，更适合处理带标识的一维数据（如某一列数据）。

## 2\.2 创建Series

```python
import pandas as pd
import numpy as np

# 1. 从列表创建Series（默认整数索引）
s1 = pd.Series([10, 20, 30, 40, 50])
print("默认索引的Series：")
print(s1)
print("Series的值：", s1.values)  # 输出[10 20 30 40 50]（ndarray类型）
print("Series的索引：", s1.index)  # 输出RangeIndex(start=0, stop=5, step=1)

# 2. 自定义索引创建Series
s2 = pd.Series([100, 200, 300], index=["语文", "数学", "英语"])
print("\n自定义索引的Series：")
print(s2)
print("索引为'数学'的值：", s2["数学"])  # 输出200（通过标签索引）

# 3. 从字典创建Series（字典的key作为索引，value作为数据）
dict_data = {"北京": 1100, "上海": 900, "广州": 700, "深圳": 800}
s3 = pd.Series(dict_data)
print("\n从字典创建的Series：")
print(s3)

# 4. 从NumPy数组创建Series
arr = np.array([1.1, 2.2, 3.3, 4.4])
s4 = pd.Series(arr, index=["a", "b", "c", "d"])
print("\n从NumPy数组创建的Series：")
print(s4)
```

## 2\.3 Series的索引与切片

Series支持两种索引方式：标签索引（label\-based）和位置索引（position\-based），切片语法与NumPy类似，但更灵活。

```python
import pandas as pd

s = pd.Series([10, 20, 30, 40, 50], index=["a", "b", "c", "d", "e"])
print("原始Series：")
print(s)

# 1. 标签索引（单个标签/多个标签）
print("\n单个标签索引（'c'）：", s["c"])  # 输出30
print("多个标签索引（['a','c','e']）：", s[["a", "c", "e"]])  # 输出3个元素的Series

# 2. 位置索引（使用iloc方法，类似NumPy的下标）
print("\n位置索引（下标2）：", s.iloc[2])  # 输出30（对应标签'c'）
print("位置切片（0:3）：", s.iloc[0:3])  # 输出前3个元素（下标0、1、2）

# 3. 布尔索引（筛选数据）
print("\n筛选大于30的元素：", s[s > 30])  # 输出d:40、e:50
print("筛选索引包含'a'或'e'的元素：", s[s.index.isin(["a", "e"])])
```

## 2\.4 Series的常用操作与函数

```python
import pandas as pd
import numpy as np

s = pd.Series([10, 20, np.nan, 40, 50, np.nan])  # 包含缺失值（NaN）
print("原始Series（含缺失值）：")
print(s)

# 1. 基本属性
print("\nSeries的长度：", len(s))  # 输出6
print("Series的描述性统计：")
print(s.describe())  # 输出计数、均值、标准差、最值、分位数

# 2. 缺失值处理（常用）
print("\n缺失值判断（True表示缺失）：", s.isnull())
print("非缺失值判断：", s.notnull())
print("删除缺失值（dropna）：", s.dropna())  # 删除含NaN的元素
print("填充缺失值（fillna）：", s.fillna(s.mean()))  # 用均值填充缺失值

# 3. 数值运算（与NumPy类似，支持广播）
print("\nSeries + 5：", s + 5)  # 每个元素加5（NaN加任何数仍为NaN）
print("Series * 2：", s * 2)  # 每个元素乘2
print("Series的均值：", s.mean())  # 输出30.0（自动忽略NaN）
print("Series的总和：", s.sum())  # 输出120.0（自动忽略NaN）

# 4. 索引相关操作
s.index = ["x", "y", "z", "m", "n", "p"]  # 重新设置索引
print("\n重新设置索引后的Series：")
print(s)
s_reset = s.reset_index()  # 将索引转为一列，生成新的DataFrame
print("\n索引转列后的结果：")
print(s_reset)
```

# 三、Pandas核心数据结构：DataFrame

## 3\.1 什么是DataFrame

DataFrame是Pandas中用于存储二维结构化数据的核心结构，类似Excel表格、SQL数据表，由行索引（index）和列索引（columns）组成，每个列可以是不同的数据类型（如int、float、str）。

核心特点：

- 二维表格结构，行表示样本，列表示特征；

- 每一列都是一个Series对象，列名就是Series的索引；

- 支持行、列的增删改查，以及复杂的筛选、分组、聚合操作；

- 可轻松处理缺失值、重复数据，适配多种数据格式。

## 3\.2 创建DataFrame

```python
import pandas as pd
import numpy as np

# 1. 从字典创建DataFrame（最常用）
# 字典的key作为列名，value作为列数据（需保证各列长度一致）
dict_data = {
    "姓名": ["张三", "李四", "王五", "赵六"],
    "年龄": [20, 21, 19, 22],
    "性别": ["男", "女", "男", "女"],
    "成绩": [85, 92, 78, 90]
}
df1 = pd.DataFrame(dict_data)
print("从字典创建的DataFrame：")
print(df1)
print("DataFrame的行索引：", df1.index)  # 默认整数索引
print("DataFrame的列索引：", df1.columns)  # 输出['姓名','年龄','性别','成绩']

# 2. 自定义行索引创建DataFrame
df2 = pd.DataFrame(dict_data, index=["学生1", "学生2", "学生3", "学生4"])
print("\n自定义行索引的DataFrame：")
print(df2)

# 3. 从NumPy数组创建DataFrame
arr = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
df3 = pd.DataFrame(arr, columns=["A", "B", "C"], index=["X", "Y", "Z"])
print("\n从NumPy数组创建的DataFrame：")
print(df3)

# 4. 从CSV/Excel文件读取（实战常用，后续详细讲解）
# df4 = pd.read_csv("data.csv")  # 读取CSV文件
# df5 = pd.read_excel("data.xlsx", sheet_name="Sheet1")  # 读取Excel文件
```

## 3\.3 DataFrame的索引与切片

DataFrame支持行、列的单独索引和组合索引，核心方法：

- \[\]：默认对列进行操作（筛选列）；

- loc：标签索引（行标签\+列标签）；

- iloc：位置索引（行下标\+列下标）；

- 布尔索引：按条件筛选行。

```python
import pandas as pd

# 准备数据
data = {
    "姓名": ["张三", "李四", "王五", "赵六"],
    "年龄": [20, 21, 19, 22],
    "性别": ["男", "女", "男", "女"],
    "成绩": [85, 92, 78, 90]
}
df = pd.DataFrame(data, index=["学生1", "学生2", "学生3", "学生4"])
print("原始DataFrame：")
print(df)

# 1. 筛选列（[]默认操作列）
print("\n筛选单个列（'姓名'）：")
print(df["姓名"])  # 返回Series
print("\n筛选多个列（['姓名','成绩']）：")
print(df[["姓名", "成绩"]])  # 返回DataFrame

# 2. 标签索引（loc）：行标签 + 列标签
print("\nloc筛选（行：'学生2'，列：'成绩'）：", df.loc["学生2", "成绩"])  # 输出92
print("\nloc筛选（行：['学生1','学生3']，列：['姓名','年龄']）：")
print(df.loc[["学生1", "学生3"], ["姓名", "年龄"]])

# 3. 位置索引（iloc）：行下标 + 列下标
print("\niloc筛选（行下标1，列下标3）：", df.iloc[1, 3])  # 输出92（对应学生2的成绩）
print("\niloc筛选（行下标0:2，列下标1:3）：")
print(df.iloc[0:2, 1:3])  # 前2行，第2-3列（下标1、2）

# 4. 布尔索引（筛选行）
print("\n筛选成绩大于85的行：")
print(df[df["成绩"] > 85])
print("\n筛选性别为'男'且年龄小于21的行：")
print(df[(df["性别"] == "男") & (df["年龄"] < 21)])  # 注意用&表示且，|表示或
```

## 3\.4 DataFrame的常用操作

### 3\.4\.1 行、列的增删改

```python
import pandas as pd

data = {
    "姓名": ["张三", "李四", "王五"],
    "年龄": [20, 21, 19],
    "成绩": [85, 92, 78]
}
df = pd.DataFrame(data)
print("原始DataFrame：")
print(df)

# 1. 新增列
df["班级"] = ["一班", "二班", "一班"]  # 新增一列，长度与行数一致
df["排名"] = [3, 1, 4]  # 新增数值列
print("\n新增列后：")
print(df)

# 2. 修改列
df["成绩"] = df["成绩"] + 5  # 成绩列所有元素加5
df.rename(columns={"班级": "所在班级"}, inplace=True)  # 重命名列（inplace=True表示修改原DataFrame）
print("\n修改列后：")
print(df)

# 3. 删除列
df.drop(columns=["排名"], inplace=True)  # 删除"排名"列
print("\n删除列后：")
print(df)

# 4. 新增行
new_row = pd.DataFrame({"姓名": ["赵六"], "年龄": [22], "成绩": [90], "所在班级": ["二班"]})
df = pd.concat([df, new_row], ignore_index=True)  # 新增行，重置索引
print("\n新增行后：")
print(df)

# 5. 删除行
df.drop(index=[2], inplace=True)  # 删除下标为2的行（王五）
print("\n删除行后：")
print(df)
```

### 3\.4\.2 缺失值与重复值处理

```python
import pandas as pd
import numpy as np

# 准备含缺失值、重复值的数据
data = {
    "姓名": ["张三", "李四", "张三", "赵六", np.nan],
    "年龄": [20, np.nan, 20, 22, 19],
    "成绩": [85, 92, 85, np.nan, 78]
}
df = pd.DataFrame(data)
print("原始DataFrame（含缺失值、重复值）：")
print(df)

# 1. 缺失值处理
print("\n缺失值统计（每列缺失值个数）：")
print(df.isnull().sum())  # 统计每列NaN的数量
df1 = df.dropna()  # 删除含任何缺失值的行
print("\n删除缺失值后：")
print(df1)
df2 = df.fillna({"年龄": df["年龄"].mean(), "成绩": df["成绩"].median()})  # 按列填充缺失值
print("\n填充缺失值后：")
print(df2)

# 2. 重复值处理
print("\n重复值判断：")
print(df.duplicated())  # True表示重复行（除第一行外）
df3 = df.drop_duplicates()  # 删除重复行（保留第一行）
print("\n删除重复值后：")
print(df3)
df4 = df.drop_duplicates(subset=["姓名"], keep="last")  # 按"姓名"去重，保留最后一行
print("\n按姓名去重（保留最后一行）后：")
print(df4)
```

### 3\.4\.3 描述性统计与排序

```python
import pandas as pd

data = {
    "姓名": ["张三", "李四", "王五", "赵六"],
    "年龄": [20, 21, 19, 22],
    "成绩": [85, 92, 78, 90]
}
df = pd.DataFrame(data)
print("原始DataFrame：")
print(df)

# 1. 描述性统计
print("\n数值列的描述性统计：")
print(df.describe())  # 仅对数值列（年龄、成绩）统计
print("每列的均值：", df.mean(numeric_only=True))  # 仅计算数值列均值
print("每列的最大值：", df.max(numeric_only=True))
print("成绩列的中位数：", df["成绩"].median())

# 2. 排序
df_sorted1 = df.sort_values(by="成绩", ascending=False)  # 按成绩降序排序
print("\n按成绩降序排序：")
print(df_sorted1)
df_sorted2 = df.sort_values(by=["年龄", "成绩"], ascending=[True, False])  # 按年龄升序、成绩降序排序
print("\n按年龄升序、成绩降序排序：")
print(df_sorted2)
df_sorted3 = df.sort_index()  # 按行索引排序（默认升序）
print("\n按索引排序：")
print(df_sorted3)
```

# 四、Pandas数据读取与保存

实战中，数据分析的第一步是读取外部数据（如CSV、Excel、TXT），处理完成后需保存结果，Pandas提供了简洁的API实现。

## 4\.1 读取常见数据格式

```python
import pandas as pd

# 1. 读取CSV文件（最常用）
df_csv = pd.read_csv("data.csv")  # 读取当前目录下的data.csv
# 可选参数：sep（分隔符，默认逗号）、header（表头行，默认第0行）、index_col（指定索引列）
df_csv2 = pd.read_csv("data.csv", sep=",", header=0, index_col="姓名")

# 2. 读取Excel文件
df_excel = pd.read_excel("data.xlsx", sheet_name="Sheet1")  # 读取Sheet1工作表
# 可选参数：header（表头）、index_col（索引列）、usecols（指定读取的列）
df_excel2 = pd.read_excel("data.xlsx", sheet_name=0, usecols=["姓名", "成绩"])

# 3. 读取TXT文件（需指定分隔符）
df_txt = pd.read_table("data.txt", sep="\t")  # 制表符分隔的TXT文件

# 4. 查看读取的数据
print("CSV文件数据：")
print(df_csv.head())  # head()默认显示前5行，避免数据过多
print("\nExcel文件数据：")
print(df_excel.tail(3))  # tail()显示后3行
```

## 4\.2 保存数据到外部文件

```python
import pandas as pd

# 准备数据
data = {
    "姓名": ["张三", "李四", "王五", "赵六"],
    "年龄": [20, 21, 19, 22],
    "成绩": [85, 92, 78, 90]
}
df = pd.DataFrame(data)

# 1. 保存为CSV文件
df.to_csv("result.csv", index=False)  # index=False表示不保存行索引（避免多余列）
# 可选参数：sep（分隔符）、header（是否保存表头，默认True）、encoding（编码，如utf-8）
df.to_csv("result2.csv", sep=",", header=True, encoding="utf-8")

# 2. 保存为Excel文件
df.to_excel("result.xlsx", sheet_name="结果", index=False)  # 指定工作表名称

# 3. 保存为TXT文件
df.to_csv("result.txt", sep="\t", index=False)  # 用制表符分隔，保存为TXT

print("数据保存完成！")
```

# 五、Pandas分组与聚合（核心实战）

分组（groupby）与聚合是数据分析的核心操作，用于按指定条件对数据分组，然后对每组进行统计（如求和、均值、计数），类似SQL的GROUP BY。

## 5\.1 基本分组操作

```python
import pandas as pd

# 准备数据（学生成绩表）
data = {
    "班级": ["一班", "一班", "二班", "二班", "一班", "二班"],
    "姓名": ["张三", "李四", "王五", "赵六", "孙七", "周八"],
    "性别": ["男", "女", "男", "女", "男", "女"],
    "成绩": [85, 92, 78, 90, 88, 82]
}
df = pd.DataFrame(data)
print("原始数据：")
print(df)

# 1. 按单个列分组（按班级分组）
group_by_class = df.groupby("班级")
# 查看分组结果（返回分组对象，需结合聚合函数使用）
print("\n按班级分组后的各组数据：")
for name, group in group_by_class:
    print(f"班级：{name}")
    print(group)
    print("-" * 20)

# 2. 按多个列分组（按班级+性别分组）
group_by_class_gender = df.groupby(["班级", "性别"])
print("\n按班级+性别分组的结果：")
print(group_by_class_gender.size())  # size()统计每组的人数

# 3. 分组后聚合（常用聚合函数：sum、mean、count、max、min）
print("\n按班级分组，计算成绩的均值和总和：")
print(group_by_class["成绩"].agg(["mean", "sum"]))  # 对成绩列聚合
print("\n按班级+性别分组，计算成绩的最大值和最小值：")
print(group_by_class_gender["成绩"].agg(["max", "min"]))

# 4. 分组后自定义聚合函数
def range_func(x):
    return x.max() - x.min()  # 计算成绩范围（最高分-最低分）

print("\n按班级分组，计算成绩范围：")
print(group_by_class["成绩"].agg(range_func))
```

## 5\.2 分组后的过滤（filter）

filter方法用于按分组后的条件，筛选出符合要求的组的所有数据。

```python
import pandas as pd

data = {
    "班级": ["一班", "一班", "二班", "二班", "一班", "二班"],
    "姓名": ["张三", "李四", "王五", "赵六", "孙七", "周八"],
    "成绩": [85, 92, 78, 90, 88, 82]
}
df = pd.DataFrame(data)

# 筛选出“班级平均成绩大于85分”的所有学生数据
group_by_class = df.groupby("班级")
filtered_df = group_by_class.filter(lambda x: x["成绩"].mean() > 85)
print("班级平均成绩大于85分的学生：")
print(filtered_df)
```

# 六、Pandas实战小案例

案例：分析学生成绩数据，完成数据读取、清洗、筛选、分组统计全流程。

```python
import pandas as pd
import numpy as np

# 1. 读取Excel数据（假设数据存在data.xlsx中，Sheet1为成绩表）
df = pd.read_excel("data.xlsx", sheet_name="Sheet1")
print("原始数据：")
print(df.head())

# 2. 数据清洗（处理缺失值、重复值）
print("\n数据清洗前：")
print(f"缺失值统计：\n{df.isnull().sum()}")
print(f"重复值数量：{df.duplicated().sum()}")

# 填充缺失值（年龄用均值，成绩用中位数）
df["年龄"] = df["年龄"].fillna(df["年龄"].mean())
df["成绩"] = df["成绩"].fillna(df["成绩"].median())
# 删除重复值
df = df.drop_duplicates()
print("\n数据清洗后：")
print(f"缺失值统计：\n{df.isnull().sum()}")
print(f"重复值数量：{df.duplicated().sum()}")

# 3. 数据筛选（筛选出成绩大于80分且年龄小于22岁的学生）
filtered_df = df[(df["成绩"] > 80) & (df["年龄"] < 22)]
print("\n成绩大于80分且年龄小于22岁的学生：")
print(filtered_df)

# 4. 分组统计（按班级分组，计算各项成绩指标）
group_stats = df.groupby("班级")["成绩"].agg([
    "count",  # 人数
    "mean",   # 平均分
    "max",    # 最高分
    "min",    # 最低分
    "std"     # 标准差
]).round(2)  # 保留2位小数
group_stats.columns = ["人数", "平均分", "最高分", "最低分", "标准差"]
print("\n按班级分组统计成绩：")
print(group_stats)

# 5. 新增列（成绩等级：优秀≥90，良好80-89，及格60-79，不及格<60）
def grade_level(score):
    if score >= 90:
        return "优秀"
    elif score >= 80:
        return "良好"
    elif score >= 60:
        return "及格"
    else:
        return "不及格"

df["成绩等级"] = df["成绩"].apply(grade_level)
print("\n新增成绩等级列后：")
print(df[["姓名", "班级", "成绩", "成绩等级"]].head())

# 6. 保存处理后的结果
df.to_excel("processed_data.xlsx", sheet_name="处理后数据", index=False)
print("\n处理后的数据已保存到processed_data.xlsx！")
```

# 七、常见问题与注意事项

- 数据类型问题：读取数据时，Pandas会自动推断数据类型，若需指定类型（如将“年龄”设为int），可使用dtype参数（如pd\.read\_csv\(\&\#34;data\.csv\&\#34;, dtype=\{\&\#34;年龄\&\#34;: int\}\)）。

- 缺失值处理：dropna\(\)会删除含缺失值的行/列，fillna\(\)可填充缺失值，需根据业务场景选择（如数值型用均值/中位数，分类型用众数）。

- inplace参数：大部分修改操作（如drop、rename、fillna）都有inplace参数，设为True表示修改原DataFrame，False表示返回新的DataFrame（默认False）。

- 分组聚合时的列选择：groupby后若只对某一列聚合，需指定列名（如group\_by\_class\[\&\#34;成绩\&\#34;\]\.mean\(\)），否则会对所有数值列聚合。

- 索引问题：重置索引用reset\_index\(\)，设置索引用set\_index\(\)，避免索引混乱导致的数据筛选错误。

- 中文编码问题：读取/保存含中文的文件时，需指定encoding=\&\#34;utf\-8\&\#34;，避免中文乱码。

# 八、总结

Pandas是Python数据分析的核心工具，重点掌握以下内容：

1. 核心数据结构：Series（一维）、DataFrame（二维）的创建与基本操作；

2. 索引与切片：loc（标签）、iloc（位置）、布尔索引的灵活使用；

3. 数据清洗：缺失值、重复值的处理方法；

4. 数据读取与保存：CSV、Excel文件的读写操作；

5. 分组与聚合：groupby的使用，以及常见聚合函数的应用；

6. 实战能力：结合案例，完成从数据读取到分析、保存的全流程。

Pandas的功能远不止于此，后续可深入学习合并（merge）、拼接（concat）、透视表（pivot\_table）等高级功能，结合Matplotlib可实现数据可视化，为数据分析提供更直观的呈现。
