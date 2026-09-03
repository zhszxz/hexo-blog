---
title: HTML
tags:
  - HTML
categories:
  - 前端
  - html
date: 2026-09-03 21:02:55
---

# HTML基础知识与常用标签学习笔记

HTML 是前端学习的第一块地基。它不负责“好不好看”，也不负责“能不能交互”，它最核心的任务是：用标签把网页内容组织成浏览器、人、搜索引擎、屏幕阅读器都能理解的结构。

这篇笔记参考了 `D:\front\code\01-html` 目录下的示例代码，覆盖基础结构、文本、图片、路径、音视频、链接、锚点、网页结构、列表、表格、表单和特殊字符等常见知识点。

<!--more-->

## 一、HTML是什么

HTML 全称是 HyperText Markup Language，中文叫“超文本标记语言”。

- 超文本：网页中不只有普通文字，还可以有图片、音频、视频、链接等内容。
- 标记语言：通过一对一对的标签告诉浏览器“这段内容是什么”。
- HTML 文件通常以 `.html` 作为后缀，可以直接用浏览器打开。

一个最小的 HTML 页面如下：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>我的第一个页面</title>
</head>
<body>
  <h1>Hello World</h1>
</body>
</html>
```

上面这段代码中：

| 代码 | 作用 |
| --- | --- |
| `<!DOCTYPE html>` | 声明当前文档使用 HTML5 标准 |
| `<html>` | HTML 文档的根标签，所有内容都写在它里面 |
| `lang="zh-CN"` | 声明页面主要语言是中文 |
| `<head>` | 存放页面配置，不直接显示在页面主体中 |
| `<meta charset="UTF-8">` | 设置字符编码，避免中文乱码 |
| `<meta name="viewport">` | 让页面在移动端按设备宽度显示 |
| `<title>` | 浏览器标签页标题 |
| `<body>` | 页面中真正显示给用户看的内容 |

## 二、标签、元素和属性

HTML 的基本写法是标签。大多数标签成对出现：

```html
<p>这是一个段落</p>
```

其中 `<p>` 是开始标签，`</p>` 是结束标签，中间的文字是内容。开始标签、内容、结束标签合起来叫一个元素。

有些标签是单标签，不需要结束标签，例如：

```html
<hr>
<br>
<img src="./img/pig.png" alt="示例图片">
```

标签还可以带属性。属性写在开始标签里，用来补充说明这个标签：

```html
<img src="./img/pig.png" alt="猪爸爸" width="500" title="猪爸爸">
```

这里的 `src`、`alt`、`width`、`title` 都是属性。

## 三、标签关系

HTML 标签之间常见的关系有两种：

- 父子关系：一个标签直接包住另一个标签。
- 兄弟关系：两个标签在同一级。

示例：

```html
<div>
  <p>
    <span>文字</span>
  </p>
</div>
```

在这段代码中：

- `div` 是 `p` 的父元素。
- `p` 是 `span` 的父元素。
- `span` 是 `p` 的子元素。

写 HTML 时一定要注意嵌套顺序，先打开的标签通常后关闭：

```html
<!-- 正确 -->
<p><strong>重要文字</strong></p>

<!-- 错误 -->
<p><strong>重要文字</p></strong>
```

## 四、文本类标签

### 1. 标题标签

标题标签从 `h1` 到 `h6`，数字越小，级别越高。

```html
<h1>一级标题：页面最重要的标题</h1>
<h2>二级标题：章节标题</h2>
<h3>三级标题：小节标题</h3>
<h4>四级标题</h4>
<h5>五级标题</h5>
<h6>六级标题</h6>
```

使用建议：

- 一个页面通常只写一个 `h1`。
- 标题不要只为了“字体变大”而乱用，应该表达内容层级。
- `h2`、`h3` 等按层级递进，方便阅读和搜索引擎理解。

### 2. 段落、换行和分割线

```html
<p>这是第一段文字。</p>
<p>这是第二段文字。</p>

第一行<br>
第二行

<hr>
```

| 标签 | 作用 |
| --- | --- |
| `<p>` | 表示一个段落，段落之间默认会换行并有间距 |
| `<br>` | 强制换行 |
| `<hr>` | 添加一条水平分割线 |

注意：`p` 标签里面通常不要再放 `h1`、`div`、`p` 这类块级元素。下面这种写法不推荐：

```html
<p>
  <h1>标题</h1>
</p>
```

### 3. 强调和重要性标签

HTML 中有一些标签会改变文字样式，也会表达语义。

```html
<strong>重要内容，通常显示为加粗</strong>
<em>强调内容，通常显示为倾斜</em>
<ins>新增或插入内容，通常显示为下划线</ins>
<del>删除内容，通常显示为删除线</del>
```

也有一组只偏向样式、语义较弱的标签：

```html
<b>加粗文字</b>
<i>倾斜文字</i>
<u>下划线文字</u>
<s>删除线文字</s>
```

初学阶段可以记住：如果内容真的有“重要、强调、插入、删除”的含义，优先使用 `strong`、`em`、`ins`、`del`。

### 4. 注释

HTML 注释不会显示在页面上，常用于解释代码。

```html
<!-- 这是一段注释，浏览器页面中看不到 -->
```

## 五、块级元素和行内元素

HTML 元素大致可以理解成两类：

| 类型 | 特点 | 常见标签 |
| --- | --- | --- |
| 块级元素 | 独占一行，可以看作一整块内容 | `h1`、`p`、`div`、`ul`、`ol`、`table`、`form` |
| 行内元素 | 不独占一行，只占自身内容宽度 | `span`、`strong`、`em`、`a`、`ins`、`del` |

示例：

```html
<h1>我是块级元素</h1>
<p>我也是块级元素</p>

<strong>我是行内元素</strong>
<strong>我会和前面的 strong 在同一行显示</strong>
<em>我也是行内元素</em>
```

特殊提醒：

- `div` 是常用的块级容器，本身没有明确语义。
- `span` 是常用的行内容器，本身没有明确语义。
- 容器标签常配合 CSS 使用，用来包裹和控制一组内容。

## 六、图片标签和路径

### 1. 图片标签

图片使用 `img` 标签：

```html
<img src="./img/pig.png" alt="猪爸爸" width="500" title="猪爸爸">
```

常见属性：

| 属性 | 作用 |
| --- | --- |
| `src` | 图片路径 |
| `alt` | 图片无法显示时的替代文字，也有利于无障碍阅读 |
| `width` | 图片宽度 |
| `height` | 图片高度 |
| `title` | 鼠标悬停时显示的提示文字 |

建议：只设置 `width` 或 `height` 中的一个，另一个会等比例缩放，图片不容易变形。

常见图片格式：

- `jpg/jpeg`：适合照片。
- `png`：适合透明背景图片、图标。
- `gif`：可做简单动图。
- `webp`、`avif`：体积更小，适合现代网页。

### 2. 相对路径

相对路径是相对于当前 HTML 文件的位置来查找资源。

```html
<!-- 当前 HTML 文件同级的 pig.png -->
<img src="./pig.png" alt="">

<!-- 当前 HTML 文件下一级 img 文件夹中的 pig.png -->
<img src="img/pig.png" alt="">
<img src="./img/pig.png" alt="">

<!-- 当前 HTML 文件上一级目录中的 img 文件夹 -->
<img src="../img/pig.png" alt="">
```

常见符号：

| 写法 | 含义 |
| --- | --- |
| `./` | 当前目录 |
| `../` | 上一级目录 |
| `img/pig.png` | 当前目录下的 `img` 文件夹里的文件 |

### 3. 绝对路径

绝对路径可以是本机盘符路径，也可以是网络路径：

```html
<!-- 本机路径，不适合发布到网站后使用 -->
<img src="E:/HTML5/代码/pig.png" alt="">

<!-- 网络路径，适合引用线上资源 -->
<img src="https://www.itcast.cn/2018czgw/images/logo2.png" alt="">
```

网页项目中更推荐使用相对路径，因为项目换电脑、换服务器后更容易保持正常。

## 七、音频和视频标签

### 1. 视频

```html
<video src="./media/yu7.mp4" width="400" controls></video>
```

常见属性：

| 属性 | 作用 |
| --- | --- |
| `src` | 视频路径 |
| `width` / `height` | 视频宽高 |
| `controls` | 显示浏览器默认播放控件 |
| `autoplay` | 自动播放 |
| `muted` | 静音 |
| `loop` | 循环播放 |
| `poster` | 视频未播放前的封面图 |

很多浏览器为了避免打扰用户，会限制非静音视频自动播放。因此如果想自动播放，常见写法是同时加上 `autoplay muted`：

```html
<video
  src="./media/yu7.mp4"
  width="400"
  controls
  autoplay
  muted
  loop
  poster="./media/yu7.jpg">
</video>
```

更兼容的写法：

```html
<video width="400" controls muted loop poster="./media/yu7.jpg">
  <source src="./media/yu7.mp4" type="video/mp4">
  <p>您的浏览器不支持 mp4 视频播放</p>
</video>
```

### 2. 音频

```html
<audio src="./media/ldh.mp3" controls muted autoplay></audio>
```

兼容性写法：

```html
<audio controls>
  <source src="./media/ldh.mp3" type="audio/mp3">
  <p>您的浏览器不支持音频播放</p>
</audio>
```

布尔属性可以省略属性值，例如 `controls="controls"` 可以简写为 `controls`。

## 八、链接和锚点

链接使用 `a` 标签，核心属性是 `href`。

### 1. 常见链接类型

```html
<!-- 内部链接：跳转到当前项目中的另一个页面 -->
<a href="./11-音视频标签.html">音视频</a>

<!-- 外部链接：跳转到其他网站 -->
<a href="https://www.deepseek.com/" title="探索未至之境" target="_blank">deepseek</a>

<!-- 空链接：常用作临时占位 -->
<a href="#">产品介绍</a>

<!-- 下载链接：链接到压缩包等文件时通常会触发下载 -->
<a href="./download.zip">下载软件</a>

<!-- 邮件链接：点击后尝试打开邮件客户端 -->
<a href="mailto:123@qq.com">联系我们</a>
```

常用属性：

| 属性 | 作用 |
| --- | --- |
| `href` | 链接地址 |
| `title` | 鼠标悬停提示 |
| `target="_blank"` | 在新标签页打开 |

### 2. 锚点链接

锚点可以让页面跳转到某个指定位置。步骤是：

1. 给目标元素设置 `id`。
2. 链接的 `href` 写成 `#id名`。

```html
<a href="#section1">跳转到第一部分</a>

<h2 id="section1">第一部分</h2>
<p>这里是第一部分的内容。</p>
```

如果想让滚动更平滑，可以配合 CSS：

```html
<style>
  html {
    scroll-behavior: smooth;
  }
</style>
```

综合案例中的“返回内容顶部”就是锚点链接：

```html
<header id="top">
  <h1>科技日报</h1>
</header>

<footer>
  <p><a href="#top">返回内容顶部</a></p>
</footer>
```

## 九、网页结构语义化标签

语义化标签的作用是：让页面结构更清楚，让人和机器都更容易理解内容。

```html
<header>网页头部</header>
<nav>导航栏</nav>
<main>
  <aside>侧边栏</aside>
  <article>主要文章内容</article>
</main>
<section>页面中的一个区块</section>
<footer>页面底部</footer>
```

常见结构标签：

| 标签 | 作用 |
| --- | --- |
| `header` | 页眉，通常放标题、Logo、简介 |
| `nav` | 导航区域 |
| `main` | 页面主要内容，一个页面通常只有一个 |
| `aside` | 侧边栏、补充信息 |
| `article` | 独立文章、新闻、博客正文 |
| `section` | 有主题的一块内容 |
| `footer` | 页脚，通常放版权、备案、联系方式 |
| `div` | 无语义块级容器 |
| `span` | 无语义行内容器 |

对比一下：

```html
<!-- 语义不清楚 -->
<div>
  <div>导航</div>
  <div>正文</div>
  <div>底部</div>
</div>

<!-- 语义更清楚 -->
<header>网站标题</header>
<nav>导航菜单</nav>
<main>主要内容</main>
<footer>版权信息</footer>
```

## 十、列表标签

### 1. 无序列表

无序列表适合表示没有先后顺序的一组内容。

```html
<ul>
  <li>佩奇</li>
  <li>猪爸爸</li>
  <li>猪妈妈</li>
  <li>乔治</li>
</ul>
```

### 2. 有序列表

有序列表适合表示步骤、排名、流程。

```html
<ol>
  <li>看视频</li>
  <li>写代码</li>
  <li>做笔记</li>
  <li>多复习</li>
</ol>
```

### 3. 描述列表

描述列表适合表示“术语 + 解释”。

```html
<dl>
  <dt>家电</dt>
  <dd>电视</dd>
  <dd>洗衣机</dd>
  <dd>冰箱</dd>
</dl>
```

| 标签 | 作用 |
| --- | --- |
| `ul` | 无序列表 |
| `ol` | 有序列表 |
| `li` | 列表项 |
| `dl` | 描述列表 |
| `dt` | 被描述的名称 |
| `dd` | 描述内容 |

注意：`ul` 和 `ol` 的直接子元素通常应该是 `li`。

## 十一、表格标签

表格用于展示二维数据，例如成绩表、数据统计表。

```html
<table border="1">
  <tr>
    <th>姓名</th>
    <th>年龄</th>
    <th>成绩</th>
  </tr>
  <tr>
    <td>张三</td>
    <td>18</td>
    <td>100</td>
  </tr>
  <tr>
    <td>李四</td>
    <td>20</td>
    <td>90</td>
  </tr>
</table>
```

| 标签 | 作用 |
| --- | --- |
| `table` | 表格 |
| `tr` | 表格中的一行 |
| `td` | 普通单元格 |
| `th` | 表头单元格，默认加粗居中 |

### 合并单元格

`rowspan` 表示跨行合并，`colspan` 表示跨列合并。

```html
<table border="1">
  <tr>
    <th>姓名</th>
    <th>年龄</th>
    <th>居住地</th>
  </tr>
  <tr>
    <td>李四</td>
    <td>20</td>
    <td rowspan="2">深圳</td>
  </tr>
  <tr>
    <td>王五</td>
    <td>22</td>
  </tr>
  <tr>
    <td>日期</td>
    <td colspan="2"></td>
  </tr>
</table>
```

合并后，被占掉的位置不要再额外写一个 `td`，否则表格会错位。

## 十二、表单标签

表单用于收集用户输入，例如登录、注册、搜索、评论。

### 1. 表单容器

```html
<form action="">
  <!-- 表单控件写在这里 -->
</form>
```

`form` 是表单容器，常见属性有：

| 属性 | 作用 |
| --- | --- |
| `action` | 表单提交到哪里 |
| `method` | 提交方式，常见值有 `get` 和 `post` |

### 2. input 输入框

```html
<label>
  账号：
  <input type="text" placeholder="请输入账号" name="username" autocomplete="off">
</label>

<label>
  密码：
  <input type="password" placeholder="请输入密码" name="pwd" maxlength="6">
</label>
```

常见属性：

| 属性 | 作用 |
| --- | --- |
| `type` | 输入框类型 |
| `placeholder` | 输入提示 |
| `name` | 字段名，提交表单时很重要 |
| `value` | 字段值 |
| `maxlength` | 最大输入长度 |
| `autocomplete="off"` | 关闭浏览器自动补全 |
| `checked` | 单选框或复选框默认选中 |

常用 `type`：

| 类型 | 作用 |
| --- | --- |
| `text` | 普通文本 |
| `password` | 密码输入 |
| `radio` | 单选框 |
| `checkbox` | 复选框 |
| `file` | 文件上传 |

### 3. 单选框和复选框

单选框：同一组选项的 `name` 必须相同，这样才能做到只能选一个。

```html
性别：
<label>
  <input type="radio" name="gender" value="1" checked> 男
</label>
<label>
  <input type="radio" name="gender" value="0"> 女
</label>
```

复选框：同一组选项也可以使用相同的 `name`，表示多个爱好值。

```html
爱好：
<label>
  <input type="checkbox" name="hobby" value="0"> 足球
</label>
<label>
  <input type="checkbox" name="hobby" value="1"> 篮球
</label>
```

### 4. 文件上传

```html
<input type="file" name="avatar" multiple accept=".jpg,.png">
```

| 属性 | 作用 |
| --- | --- |
| `multiple` | 允许选择多个文件 |
| `accept` | 限制可选择的文件类型 |

### 5. 文本域、下拉列表和按钮

```html
<label>
  留言：
  <textarea name="msg" cols="30" rows="3" placeholder="请输入留言"></textarea>
</label>

城市：
<select name="city">
  <option value="bj">北京</option>
  <option value="sh" selected>上海</option>
  <option value="gz">广州</option>
  <option value="sz">深圳</option>
</select>

<button>提交</button>
```

| 标签 | 作用 |
| --- | --- |
| `textarea` | 多行文本输入 |
| `select` | 下拉列表 |
| `option` | 下拉列表中的选项 |
| `button` | 按钮 |
| `label` | 表单说明文字，包裹控件后点击文字也能选中控件 |

## 十三、特殊字符

如果想在网页中显示一些容易被 HTML 误解的字符，需要使用字符实体。

```html
&nbsp;  空格
&lt;    小于号 <
&gt;    大于号 >
&copy;  版权符号
&reg;   注册商标
&yen;   人民币符号
&cent;  分
&pound; 英镑符号
&euro;  欧元符号
&amp;   与符号 &
&quot;  双引号
```

例如，要在页面上显示 `<p>` 是一个段落标签，应该写成：

```html
&lt;p&gt; 是一个段落标签
```

如果直接写 `<p>`，浏览器会把它当成真正的 HTML 标签解析。

## 十四、综合案例：科技日报页面

下面是一个简化后的综合案例，整合了网页结构、标题段落、导航、图片、表格、表单、视频、邮件链接和锚点返回顶部。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>科技日报-首页</title>
</head>
<body>
  <header id="top">
    <h1><a href="./index.html" title="科技日报">科技日报</a></h1>
    <p>探索未来科技，传递创新力量</p>
    <nav>
      <ul>
        <li><a href="./index.html">首页</a></li>
        <li><a href="#">科技动态</a></li>
        <li><a href="#">国际科技</a></li>
        <li><a href="#">科技论坛</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <h2>今日要闻</h2>

    <article>
      <h3>1. 量子计算机取得突破性进展</h3>
      <p>量子计算正在推动人工智能、芯片、材料等领域的发展。</p>
      <img src="./img/1.webp" alt="人工智能在医疗领域的最新应用" width="500" title="人工智能在医疗领域的最新应用">
    </article>

    <section>
      <h3>2. 行业数据</h3>
      <table border="1">
        <tr>
          <th>领域</th>
          <th>投资额（亿美元）</th>
          <th>增长率</th>
        </tr>
        <tr>
          <td>人工智能</td>
          <td>580</td>
          <td>+42%</td>
        </tr>
        <tr>
          <td>量子计算</td>
          <td>120</td>
          <td>+215%</td>
        </tr>
      </table>
    </section>

    <section>
      <h3>3. 读者互动</h3>
      <form action="">
        <p>您最关注的科技领域是？</p>
        <label><input type="radio" name="domain" value="ai"> 人工智能</label>
        <label><input type="radio" name="domain" value="qc"> 量子计算</label>

        <p>留下您的观点：</p>
        <textarea name="msg" cols="30" rows="5"></textarea>
        <br>
        <button>提交</button>
      </form>
    </section>

    <section>
      <h3>4. 科技大讲堂</h3>
      <video src="./img/video.mp4" poster="./img/poster.jpg" controls width="500" autoplay muted loop></video>
    </section>
  </main>

  <footer>
    <p>&copy; 2025 科技日报社 版权所有</p>
    <p>联系我们：<a href="mailto:123@qq.com">123@qq.com</a></p>
    <p><a href="#top">返回内容顶部</a></p>
  </footer>
</body>
</html>
```

这个案例可以拆成几层理解：

- `header`：页面头部，包含站点标题、说明和导航。
- `nav + ul + li + a`：导航菜单的常见组合。
- `main`：页面主体内容。
- `article` / `section`：组织新闻文章和功能区块。
- `img`：展示新闻配图。
- `table`：展示行业数据。
- `form`：收集用户选择和留言。
- `video`：展示视频内容。
- `footer`：版权、联系方式、返回顶部。

## 十五、常用标签速查表

| 分类 | 标签 | 作用 |
| --- | --- | --- |
| 文档结构 | `html` | HTML 根元素 |
| 文档结构 | `head` | 页面配置区域 |
| 文档结构 | `body` | 页面可见内容 |
| 文档结构 | `meta` | 元信息，如字符编码、视口 |
| 文档结构 | `title` | 浏览器标签页标题 |
| 文本 | `h1` - `h6` | 标题 |
| 文本 | `p` | 段落 |
| 文本 | `br` | 换行 |
| 文本 | `hr` | 分割线 |
| 文本语义 | `strong` | 重要，加粗 |
| 文本语义 | `em` | 强调，倾斜 |
| 文本语义 | `ins` | 插入，下划线 |
| 文本语义 | `del` | 删除，删除线 |
| 容器 | `div` | 块级无语义容器 |
| 容器 | `span` | 行内无语义容器 |
| 媒体 | `img` | 图片 |
| 媒体 | `audio` | 音频 |
| 媒体 | `video` | 视频 |
| 链接 | `a` | 超链接、锚点、邮件链接、下载链接 |
| 结构 | `header` | 页眉 |
| 结构 | `nav` | 导航 |
| 结构 | `main` | 主体 |
| 结构 | `aside` | 侧边栏 |
| 结构 | `article` | 独立文章 |
| 结构 | `section` | 区块 |
| 结构 | `footer` | 页脚 |
| 列表 | `ul` | 无序列表 |
| 列表 | `ol` | 有序列表 |
| 列表 | `li` | 列表项 |
| 列表 | `dl` / `dt` / `dd` | 描述列表 |
| 表格 | `table` | 表格 |
| 表格 | `tr` | 行 |
| 表格 | `td` | 普通单元格 |
| 表格 | `th` | 表头单元格 |
| 表单 | `form` | 表单容器 |
| 表单 | `input` | 输入控件 |
| 表单 | `label` | 表单说明文字 |
| 表单 | `textarea` | 多行文本 |
| 表单 | `select` / `option` | 下拉列表 |
| 表单 | `button` | 按钮 |

## 十六、初学者容易混淆的点

### 1. HTML、CSS、JavaScript 的分工

- HTML：负责内容结构，比如标题、段落、图片、表格、表单。
- CSS：负责页面样式，比如颜色、大小、布局、间距。
- JavaScript：负责交互逻辑，比如点击按钮、表单校验、动态更新页面。

可以把网页理解成：

- HTML 是骨架。
- CSS 是外观。
- JavaScript 是动作。

### 2. 语义和样式不是一回事

`h1` 默认字体大，但它不是“大字标签”，而是“一级标题标签”。

`strong` 默认加粗，但它不是单纯的“加粗标签”，而是表示“这段内容很重要”。

如果只是想调整样式，后面应该交给 CSS；如果想表达内容含义，才应该选择对应的 HTML 标签。

### 3. 相对路径更适合项目开发

本机盘符路径只在你自己的电脑上有效：

```html
<img src="E:/HTML5/代码/pig.png" alt="">
```

项目中更推荐：

```html
<img src="./img/pig.png" alt="">
```

这样项目移动到其他电脑或服务器后，资源更容易正常显示。

### 4. 表单控件要重视 `name`

表单提交时，后端通常通过 `name` 获取对应数据。

```html
<input type="text" name="username">
<input type="password" name="pwd">
```

如果没有 `name`，控件即使能输入，提交时也可能拿不到对应字段。

### 5. 单选框分组靠相同的 `name`

```html
<input type="radio" name="gender" value="1"> 男
<input type="radio" name="gender" value="0"> 女
```

两个 radio 的 `name` 都是 `gender`，浏览器才知道它们属于同一组，只能选一个。

## 十七、复习清单

学习完这一章，可以用下面的问题自测：

- 能不能手写一个 HTML5 基础骨架？
- `head` 和 `body` 分别放什么？
- `h1` 到 `h6` 的层级有什么区别？
- `p`、`br`、`hr` 分别用在什么场景？
- `strong` 和 `b` 有什么区别？
- `div` 和 `span` 有什么区别？
- `img` 的 `src`、`alt`、`title` 分别是什么作用？
- `./` 和 `../` 在路径中分别代表什么？
- `audio` 和 `video` 如何显示播放控件？
- `a` 标签可以写哪些类型的链接？
- 锚点链接如何实现？
- `header`、`nav`、`main`、`footer` 分别表示什么？
- 无序列表、有序列表、描述列表分别怎么写？
- 表格中的 `table`、`tr`、`td`、`th` 分别是什么？
- `rowspan` 和 `colspan` 分别表示什么？
- 表单中的 `form`、`input`、`label`、`textarea`、`select`、`button` 分别有什么作用？
- 单选框为什么需要相同的 `name`？
- 如何在页面中显示 `<p>` 这几个字符本身？
