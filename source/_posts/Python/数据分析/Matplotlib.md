---
title: Matplotlib
tags: Matplotlib
categories:
  - Python
  - 数据分析
date: 2026-04-12 14:12:33
---

# 一、Matplotlib简介

## 1\.1 什么是Matplotlib

Matplotlib是Python中最常用的数据分析可视化库，可快速绘制各类静态、高质量的图表（如折线图、柱状图、散点图、直方图等），支持自定义图表样式、标签、颜色等细节，是数据分析中“数据呈现”的核心工具。

<!--more-->

Matplotlib与NumPy、Pandas的关系：

- Matplotlib可直接接收NumPy数组和Pandas的Series、DataFrame数据，无需额外数据格式转换；

- Pandas的绘图功能（如df\.plot\(\)）底层就是基于Matplotlib实现的，可快速生成基础图表；

- 三者协同使用，可完成“数据读取（Pandas）→ 数据处理（NumPy\+Pandas）→ 数据可视化（Matplotlib）”的完整数据分析流程。

核心模块：matplotlib\.pyplot（通常简写为plt），是绘制图表的主要接口，提供了简洁的函数式绘图方法。

## 1\.2 安装Matplotlib

使用pip命令安装，与NumPy、Pandas兼容，建议安装最新稳定版：

```python
# 安装最新版本Matplotlib
pip install matplotlib

# 安装指定版本（适配之前的NumPy 1.24.3、Pandas 1.5.3）
pip install matplotlib==3.7.1
```

验证安装是否成功：

```python
import matplotlib.pyplot as plt
print(plt.__version__)  # 输出版本号，如3.7.1即安装成功

# 绘制简单折线图，验证绘图功能
plt.plot([1, 2, 3, 4])
plt.show()  # 显示图表，若能正常弹出图表窗口则安装无误
```

# 二、Matplotlib绘图基础

## 2\.1 核心绘图流程

Matplotlib绘图遵循“创建画布→绘制图表→设置样式→显示/保存图表”的固定流程，无论绘制哪种图表，核心逻辑一致，流程如下：

1. 导入模块：import matplotlib\.pyplot as plt（固定简写）；

2. 创建画布（可选）：plt\.figure\(\)，用于控制画布大小、分辨率等；

3. 绘制图表：调用plt\.plot\(\)、plt\.bar\(\)等函数，传入数据；

4. 设置样式：添加标题、坐标轴标签、图例、网格等；

5. 显示/保存图表：plt\.show\(\)（显示图表）、plt\.savefig\(\)（保存图表）。

```python
import matplotlib.pyplot as plt
import numpy as np

# 1. 准备数据（NumPy数组）
x = np.arange(1, 6)  # x轴数据：[1,2,3,4,5]
y = x ** 2  # y轴数据：[1,4,9,16,25]

# 2. 创建画布（可选，默认会自动创建）
plt.figure(figsize=(8, 5), dpi=100)  # figsize：画布大小（宽,高），dpi：分辨率

# 3. 绘制图表（折线图）
plt.plot(x, y)

# 4. 设置样式
plt.title("简单折线图")  # 图表标题
plt.xlabel("X轴")  # X轴标签
plt.ylabel("Y轴")  # Y轴标签
plt.grid(True, linestyle="--", alpha=0.5)  # 显示网格，线型为虚线，透明度0.5

# 5. 显示图表
plt.show()

# 补充：保存图表（需在plt.show()之前，否则保存为空）
# plt.savefig("line_chart.png", dpi=300, bbox_inches="tight")  # 保存为png格式，高清，避免标签被截断
```

## 2\.2 核心概念：画布与子图

在复杂可视化场景中，常需要在一个画布上绘制多个图表（子图），核心函数：plt\.subplot\(\) 或 plt\.subplots\(\)。

### 2\.2\.1 plt\.subplot\(\)：单个子图创建

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.arange(1, 6)
y1 = x ** 2
y2 = x ** 3

# 创建画布
plt.figure(figsize=(10, 4))

# 第一个子图：2行1列，第1个位置（行优先）
plt.subplot(2, 1, 1)  # （行数，列数，当前子图索引）
plt.plot(x, y1, color="red", label="y=x²")  # 设置颜色和图例标签
plt.title("y=x² 折线图")
plt.legend()  # 显示图例（需指定label参数）

# 第二个子图：2行1列，第2个位置
plt.subplot(2, 1, 2)
plt.plot(x, y2, color="blue", label="y=x³")
plt.title("y=x³ 折线图")
plt.legend()

# 调整子图间距（避免标题重叠）
plt.tight_layout()

plt.show()
```

### 2\.2\.2 plt\.subplots\(\)：批量创建子图（推荐）

plt\.subplots\(\)会返回“画布对象（figure）”和“子图数组（axes）”，可通过索引直接操作子图，更适合批量处理。

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.arange(1, 6)
y1 = x ** 2
y2 = x ** 3
y3 = np.sin(x)
y4 = np.cos(x)

# 批量创建子图：2行2列，返回画布和子图数组
fig, axes = plt.subplots(2, 2, figsize=(10, 8))

# 操作子图（axes是2x2的数组，通过索引访问）
axes[0, 0].plot(x, y1, color="red")  # 第1行第1列子图
axes[0, 0].set_title("y=x²")  # 子图标题（用set_title，区别于plt.title）
axes[0, 0].set_xlabel("X轴")  # 子图X轴标签
axes[0, 0].grid(True)

axes[0, 1].plot(x, y2, color="blue")  # 第1行第2列子图
axes[0, 1].set_title("y=x³")

axes[1, 0].plot(x, y3, color="green")  # 第2行第1列子图
axes[1, 0].set_title("y=sin(x)")

axes[1, 1].plot(x, y4, color="orange")  # 第2行第2列子图
axes[1, 1].set_title("y=cos(x)")

# 调整子图间距
plt.tight_layout()
plt.show()
```

## 2\.3 常用样式设置

图表样式直接影响可读性，重点掌握以下常用设置，可让图表更规范、美观：

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.arange(1, 6)
y = x ** 2

plt.figure(figsize=(8, 5))

# 1. 线条样式：color（颜色）、linestyle（线型）、linewidth（线宽）、marker（数据点标记）
plt.plot(x, y, 
         color="purple",  # 颜色（可写英文、十六进制如#800080）
         linestyle="-.",  # 线型：-（实线）、--（虚线）、-.（点划线）、:（点线）
         linewidth=2,     # 线宽
         marker="o",      # 数据点标记：o（圆形）、s（方形）、^（三角形）、*（星号）
         markersize=8,    # 标记大小
         markerfacecolor="white")  # 标记填充色

# 2. 标题与坐标轴
plt.title("样式设置示例", fontsize=14, fontweight="bold")  # 标题字体大小、加粗
plt.xlabel("X轴（单位：个）", fontsize=12)  # 坐标轴标签
plt.ylabel("Y轴（单位：平方）", fontsize=12)

# 3. 坐标轴范围（避免自动缩放导致的偏差）
plt.xlim(0, 6)  # X轴范围：0~6
plt.ylim(0, 30)  # Y轴范围：0~30

# 4. 图例（需指定plot的label参数）
plt.legend(loc="upper left", fontsize=10)  # loc：图例位置（upper left=左上角）

# 5. 网格
plt.grid(True, linestyle="--", alpha=0.3)  # alpha：透明度（0~1）

# 6. 坐标轴刻度（自定义刻度）
plt.xticks([0, 2, 4, 6], labels=["0", "2", "4", "6"])  # X轴刻度及标签
plt.yticks([0, 10, 20, 30])  # Y轴刻度

plt.show()
```

# 三、Matplotlib常见图表绘制（核心实战）

结合数据分析场景，重点掌握以下6类常见图表，覆盖80%的可视化需求，所有示例均兼容NumPy数组和Pandas数据。

## 3\.1 折线图（plot）

用途：展示数据随时间/连续变量的变化趋势（如销量变化、气温变化）。

```python
import matplotlib.pyplot as plt
import pandas as pd

# 用Pandas准备数据（模拟月度销量数据）
data = {
    "月份": [1, 2, 3, 4, 5, 6],
    "销量": [120, 150, 130, 180, 160, 200],
    "利润": [20, 25, 22, 30, 28, 35]
}
df = pd.DataFrame(data)

plt.figure(figsize=(8, 5))

# 绘制两条折线（同一画布）
plt.plot(df["月份"], df["销量"], label="销量", color="blue", marker="o")
plt.plot(df["月份"], df["利润"], label="利润", color="red", marker="s")

# 样式设置
plt.title("月度销量与利润变化趋势", fontsize=14)
plt.xlabel("月份", fontsize=12)
plt.ylabel("金额（万元）", fontsize=12)
plt.legend()
plt.grid(True, alpha=0.3)
plt.xticks(df["月份"])  # X轴刻度与月份一致

plt.show()
```

## 3\.2 柱状图（bar/barh）

用途：对比多个类别数据的差异（如不同产品销量、不同班级平均分），分为垂直柱状图（bar）和水平柱状图（barh）。

```python
import matplotlib.pyplot as plt
import pandas as pd

# 准备数据（模拟不同产品销量）
df = pd.DataFrame({
    "产品": ["A", "B", "C", "D"],
    "销量": [350, 280, 420, 300]
})

plt.figure(figsize=(8, 5))

# 1. 垂直柱状图（bar）
plt.bar(df["产品"], df["销量"], 
        color=["red", "blue", "green", "orange"],  # 每个柱子不同颜色
        width=0.6,  # 柱子宽度
        edgecolor="black")  # 柱子边框颜色

# 2. 水平柱状图（barh，可选）
# plt.barh(df["产品"], df["销量"], color=["red", "blue", "green", "orange"])

# 在柱子上添加数值标签（关键：提升可读性）
for i, v in enumerate(df["销量"]):
    plt.text(i, v + 10, str(v), ha="center", fontsize=10)  # ha="center"：水平居中

# 样式设置
plt.title("不同产品销量对比", fontsize=14)
plt.xlabel("产品", fontsize=12)
plt.ylabel("销量（件）", fontsize=12)
plt.ylim(0, 450)  # Y轴范围略高于最大销量，避免数值标签被截断

plt.show()
```

## 3\.3 散点图（scatter）

用途：展示两个变量之间的相关性（如身高与体重、学习时间与成绩），点的颜色/大小可映射第三个变量。

```python
import matplotlib.pyplot as plt
import numpy as np

# 准备数据（模拟学习时间与成绩的关系）
np.random.seed(1)  # 固定随机种子，让结果可复现
study_time = np.random.randint(1, 10, 50)  # 50个1~10小时的学习时间
score = 60 + study_time * 3 + np.random.randn(50) * 2  # 成绩=60+3*学习时间+随机波动

plt.figure(figsize=(8, 5))

# 绘制散点图，用颜色映射成绩高低
scatter = plt.scatter(study_time, score, 
                      c=score,  # 颜色映射成绩
                      cmap="RdYlGn",  # 颜色映射方案（红→黄→绿，成绩从低到高）
                      s=50,  # 点的大小
                      alpha=0.7)  # 透明度，避免点重叠

# 添加颜色条（解释颜色含义）
plt.colorbar(scatter, label="成绩")

# 样式设置
plt.title("学习时间与成绩相关性", fontsize=14)
plt.xlabel("学习时间（小时）", fontsize=12)
plt.ylabel("成绩（分）", fontsize=12)
plt.grid(True, alpha=0.3)

plt.show()
```

## 3\.4 直方图（hist）

用途：展示数据的分布情况（如成绩分布、身高分布），将数据分组，统计每组的频数/频率。

```python
import matplotlib.pyplot as plt
import numpy as np

# 准备数据（模拟500名学生的成绩分布，正态分布：均值80，标准差5）
scores = np.random.normal(80, 5, 500)

plt.figure(figsize=(8, 5))

# 绘制直方图
plt.hist(scores, 
         bins=15,  # 分组数量（bins越多，分组越细）
         color="lightblue", 
         edgecolor="black",
         alpha=0.7)

# 样式设置
plt.title("500名学生成绩分布", fontsize=14)
plt.xlabel("成绩（分）", fontsize=12)
plt.ylabel("学生人数", fontsize=12)
plt.grid(True, alpha=0.3, axis="y")  # 只显示Y轴网格

# 添加均值线
plt.axvline(scores.mean(), color="red", linestyle="--", label=f"均值：{scores.mean():.1f}")
plt.legend()

plt.show()
```

## 3\.5 饼图（pie）

用途：展示各部分占总体的比例（如各产品销量占比、各科目成绩占比），注意：数据总和需为100%或可归一化。

```python
import matplotlib.pyplot as plt
import pandas as pd

# 准备数据（模拟各产品销量占比）
df = pd.DataFrame({
    "产品": ["A", "B", "C", "D"],
    "销量": [350, 280, 420, 300]
})
# 计算占比
df["占比"] = df["销量"] / df["销量"].sum() * 100

plt.figure(figsize=(7, 7))  # 饼图建议用正方形画布，避免变形

# 绘制饼图
plt.pie(df["销量"], 
        labels=df["产品"],  # 各部分标签
        autopct="%1.1f%%",  # 显示占比，保留1位小数
        colors=["red", "blue", "green", "orange"],
        explode=[0.05, 0, 0.05, 0],  # 突出显示A、C产品（远离圆心）
        shadow=True,  # 显示阴影，提升立体感
        startangle=90)  # 起始角度（90度为正上方）

# 样式设置
plt.title("各产品销量占比", fontsize=14)
plt.axis("equal")  # 保证饼图为正圆形

plt.show()
```

## 3\.6 箱线图（boxplot）

用途：展示数据的分布特征（中位数、四分位数、异常值），常用于异常值检测（如成绩中的极端值、销量中的异常数据）。

```python
import matplotlib.pyplot as plt
import numpy as np

# 准备数据（模拟3个班级的成绩数据）
class1 = np.random.normal(80, 5, 50)
class2 = np.random.normal(75, 8, 50)
class3 = np.random.normal(85, 6, 50)
data = [class1, class2, class3]

plt.figure(figsize=(8, 5))

# 绘制箱线图
box = plt.boxplot(data, 
                  labels=["一班", "二班", "三班"],  # 标签
                  patch_artist=True,  # 允许填充颜色
                  boxprops={"facecolor": "lightblue"},  # 箱体颜色
                  flierprops={"color": "red", "marker": "o"})  # 异常值样式

# 样式设置
plt.title("3个班级成绩分布箱线图", fontsize=14)
plt.xlabel("班级", fontsize=12)
plt.ylabel("成绩（分）", fontsize=12)
plt.grid(True, alpha=0.3, axis="y")

# 箱线图解读：箱体上下沿为四分位数（Q1、Q3），中间线为中位数，须线为最大值/最小值，红点为异常值
plt.show()
```

# 四、Matplotlib与Pandas结合实战

Pandas的DataFrame自带plot\(\)方法，可快速调用Matplotlib绘制图表，语法更简洁，适合快速可视化，底层仍依赖Matplotlib，可结合Matplotlib的样式设置优化图表。

```python
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

# 准备数据（模拟学生成绩表）
df = pd.DataFrame({
    "姓名": ["张三", "李四", "王五", "赵六", "孙七", "周八"],
    "班级": ["一班", "一班", "二班", "二班", "一班", "二班"],
    "成绩": [85, 92, 78, 90, 88, 82],
    "年龄": [20, 21, 19, 22, 20, 19]
})

# 1. 用Pandas快速绘制折线图（按班级分组统计平均分）
plt.figure(figsize=(8, 5))
class_score = df.groupby("班级")["成绩"].mean()  # 按班级分组求平均分
class_score.plot(kind="line", marker="o", color="blue")  # kind指定图表类型

# 结合Matplotlib设置样式
plt.title("各班级平均成绩", fontsize=14)
plt.xlabel("班级", fontsize=12)
plt.ylabel("平均分", fontsize=12)
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()

# 2. 用Pandas快速绘制柱状图（各班级人数）
plt.figure(figsize=(8, 5))
class_count = df.groupby("班级")["姓名"].count()  # 按班级统计人数
class_count.plot(kind="bar", color=["red", "blue"])

# 添加数值标签
for i, v in enumerate(class_count):
    plt.text(i, v + 0.1, str(v), ha="center")

plt.title("各班级人数统计", fontsize=14)
plt.xlabel("班级", fontsize=12)
plt.ylabel("人数", fontsize=12)
plt.ylim(0, 4)
plt.show()

# 3. 用Pandas绘制散点图（年龄与成绩的关系）
plt.figure(figsize=(8, 5))
df.plot(kind="scatter", x="年龄", y="成绩", c="成绩", cmap="RdYlGn", s=50, alpha=0.7)

plt.title("年龄与成绩相关性", fontsize=14)
plt.grid(True, alpha=0.3)
plt.colorbar(label="成绩")
plt.show()
```

# 五、常见问题与注意事项

- 中文显示问题：默认情况下，Matplotlib不支持中文，会显示乱码，解决方案：添加字体设置（plt\.rcParams）。
  `\# 解决中文乱码（放在导入模块后、绘图前）
  plt\.rcParams\[\&\#34;font\.sans\-serif\&\#34;\] = \[\&\#34;SimHei\&\#34;\]  \# 中文支持（Windows系统）
  plt\.rcParams\[\&\#34;axes\.unicode\_minus\&\#34;\] = False  \# 解决负号显示异常`

- 图表保存问题：plt\.savefig\(\)必须放在plt\.show\(\)之前，否则会保存为空（plt\.show\(\)会清空画布）；若标签被截断，添加bbox\_inches=\&\#34;tight\&\#34;参数。

- 子图间距问题：多个子图叠加时，标题、标签可能重叠，使用plt\.tight\_layout\(\)自动调整子图间距，或用plt\.subplots\_adjust\(\)手动调整。

- 数据格式问题：Matplotlib优先接收NumPy数组和Pandas的Series、DataFrame，若传入Python列表，会自动转换，但效率较低，建议提前转换为NumPy数组。

- 样式一致性：同一批图表（如同一分析报告中的图表），需保持颜色、字体、线型的一致性，提升专业性。

- 图表可读性：避免过度装饰，重点突出数据；添加必要的标题、标签、图例，让读者快速理解图表含义；数值标签需清晰，避免重叠。

# 六、总结

Matplotlib是Python数据分析可视化的核心工具，重点掌握以下内容，即可满足大部分数据分析场景的可视化需求：

1. 绘图基础：核心流程（创建画布→绘制图表→设置样式→显示/保存）、画布与子图的使用；

2. 常见图表：折线图（趋势）、柱状图（对比）、散点图（相关性）、直方图（分布）、饼图（占比）、箱线图（异常值）；

3. 样式设置：标题、坐标轴、图例、网格、颜色、线型等细节优化；

4. 协同使用：与NumPy、Pandas结合，实现数据处理与可视化的无缝衔接；

5. 实战技巧：中文显示、图表保存、子图间距调整、数值标签添加等。

Matplotlib的功能十分强大，除了基础图表，还可绘制3D图表、动态图表等，后续可结合具体业务场景深入学习；同时，可了解Seaborn库（基于Matplotlib），其封装了更简洁的API，可快速绘制更美观的统计图表。
