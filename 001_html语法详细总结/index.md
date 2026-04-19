# HTML 语法详细总结


# HTML语法详细总结
HTML（HyperText Markup Language，超文本标记语言）是构建网页结构的核心标记语言，**并非编程语言**，通过标签（Tag）定义网页的内容结构与语义，最新标准为HTML5，是前端开发三大核心技术（HTML、CSS、JavaScript）之一。

## 一、HTML文档基本结构
一个标准的HTML5文档必须包含以下核心结构，浏览器会按照该结构解析页面。

### 1. 完整标准示例
```html
<!-- 文档类型声明：告诉浏览器使用HTML5标准解析页面 -->
<!DOCTYPE html>
<!-- 根标签：页面所有内容都必须包裹在html标签内，lang属性声明页面语言 -->
<html lang="zh-CN">
<!-- 头部标签：存放页面元数据，不直接展示给用户，用于浏览器、搜索引擎解析 -->
<head>
    <!-- 字符编码声明：必须放在head最前面，避免乱码，通用UTF-8 -->
    <meta charset="UTF-8">
    <!-- 视口设置：移动端适配核心，保证页面在移动设备上正常显示 -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- 页面标题：浏览器标签栏显示，SEO核心 -->
    <title>HTML语法总结</title>
    <!-- 引入外部CSS样式表 -->
    <link rel="stylesheet" href="style.css">
    <!-- 内嵌CSS样式 -->
    <style>
        /* 样式代码 */
    </style>
</head>
<!-- 主体标签：页面所有可见内容都必须放在body内 -->
<body>
    <!-- 页面可见内容 -->
    <h1>HTML核心语法</h1>
    <p>这是一个标准的HTML5文档</p>

    <!-- 引入JavaScript脚本，推荐放在body末尾，避免阻塞页面渲染 -->
    <script src="script.js"></script>
</body>
</html>
```

### 2. 核心结构逐解
1.  **`<!DOCTYPE html>`**：文档类型声明，必须放在文档最顶部，无闭合标签。作用是触发浏览器的**标准模式**，避免怪异模式（Quirks Mode）导致的样式错乱，HTML5的声明极简，无需引用DTD。
2.  **`<html>`**：页面唯一根标签，所有内容必须包裹在此标签内。`lang`属性声明页面语言（`zh-CN`=简体中文，`en`=英文），用于SEO和无障碍阅读。
3.  **`<head>`**：页面元数据容器，内容不直接展示给用户，核心作用是配置页面、告知浏览器和搜索引擎页面信息。
4.  **`<body>`**：页面可视内容容器，所有用户可见的文本、图片、视频、按钮等内容都必须放在此标签内。

## 二、HTML核心语法规则
### 1. 标签（Tag）语法
HTML通过标签标记内容，标签名不区分大小写，但**行业规范强制使用小写**，分为双标签和单标签两类。

#### （1）双标签
绝大多数标签为双标签，由**开始标签**和**结束标签**组成，内容包裹在两者之间。
- 语法：`<标签名> 内容 </标签名>`
- 示例：`<p>这是一个段落</p>`、`<h1>一级标题</h1>`

#### （2）单标签（自闭合标签）
无需包裹内容，仅通过单个标签完成功能，HTML5中可省略闭合斜杠`/`，推荐极简写法。
- 语法：`<标签名>` 或 `<标签名/>`（XHTML规范）
- 常用单标签：`<br>`（换行）、`<hr>`（水平线）、`<img>`（图片）、`<input>`（输入框）、`<meta>`、`<link>`

### 2. 标签嵌套规则
HTML标签必须**正确嵌套**，不能交叉嵌套，最终形成树形的DOM结构。
- 正确示例：`<div><p>段落内容</p></div>`
- 错误示例：`<div><p>段落内容</div></p>`（交叉嵌套）

核心嵌套禁忌：
1.  行内元素不能嵌套块级元素（如`<span>`内不能放`<div>`、`<p>`）
2.  `<p>`标签不能嵌套任何块级元素（包括自身）
3.  `<a>`标签不能嵌套`<a>`标签
4.  `<ul>`/`<ol>`的直接子元素只能是`<li>`
5.  `<table>`的直接子元素只能是`<caption>`、`<thead>`、`<tbody>`、`<tfoot>`

### 3. 属性（Attribute）语法
属性用于给标签添加额外信息、配置功能，写在**开始标签**内，多个属性用空格分隔。

#### （1）基础语法
```html
<!-- 语法：属性名="属性值"，推荐双引号包裹属性值 -->
<标签名 属性名1="属性值1" 属性名2="属性值2"> 内容 </标签名>

<!-- 示例 -->
<a href="https://www.example.com" target="_blank" title="示例网站">跳转链接</a>
<img src="image.jpg" alt="示例图片" width="300" height="200">
```

#### （2）核心规则
1.  属性名不区分大小写，推荐小写
2.  属性值推荐用双引号包裹，单引号也可，无特殊字符时可省略（不推荐）
3.  **布尔属性**：无需写属性值，只要标签内存在该属性即生效，常用于表单。
    示例：`<input type="checkbox" checked disabled>`（checked=选中，disabled=禁用）
4.  同一标签内，同一属性不能重复定义

#### （3）全局属性
所有HTML标签都支持的通用属性，核心常用如下：
| 属性名 | 核心作用 | 示例 |
|--------|----------|------|
| `id` | 元素的唯一标识，一个页面内不能重复，用于锚点、JS/CSS选中元素 | `<div id="header">头部</div>` |
| `class` | 元素的类名，可重复、可多个（空格分隔），用于CSS批量设置样式、JS选中元素 | `<p class="text-red font-bold">段落</p>` |
| `style` | 元素的内联CSS样式，优先级最高 | `<div style="color: red; font-size: 16px;">内容</div>` |
| `title` | 鼠标悬浮在元素上时显示的提示文本 | `<a href="#" title="点击返回首页">首页</a>` |
| `data-*` | 自定义数据属性，用于给元素存储自定义数据，JS可读取 | `<div data-user-id="123" data-role="admin">用户</div>` |
| `hidden` | 布尔属性，隐藏元素，相当于`display: none` | `<p hidden>这段内容会被隐藏</p>` |
| `contenteditable` | 布尔属性，设置元素内容是否可编辑 | `<div contenteditable>这段内容可编辑</div>` |
| `tabindex` | 设置元素的Tab键聚焦顺序，值为数字 | `<input tabindex="1">` |

### 4. 注释语法
HTML注释用于给代码添加说明，浏览器不会解析注释内容，也不会展示给用户。
- 语法：`<!-- 注释内容 -->`
- 规则：注释可以单行、多行，**不能嵌套注释**
- 示例：
```html
<!-- 这是单行注释 -->
<!--
这是多行注释
用于说明代码的功能
-->
```

### 5. 字符实体（转义字符）
HTML中，部分特殊字符会被浏览器解析为标签或语法，必须通过字符实体转义才能正常显示。
常用核心字符实体：
| 原字符 | 含义 | 字符实体 |
|--------|------|----------|
| ` ` | 不间断空格（普通空格会被浏览器合并） | `&nbsp;` |
| `<` | 小于号/左尖括号 | `&lt;` |
| `>` | 大于号/右尖括号 | `&gt;` |
| `&` | 和号 | `&amp;` |
| `"` | 双引号 | `&quot;` |
| `'` | 单引号 | `&apos;` |
| `©` | 版权符号 | `&copy;` |
| `®` | 注册商标符号 | `&reg;` |

示例：`<p>HTML标签用 &lt; &gt; 包裹</p>`，页面会显示：HTML标签用 < > 包裹

## 三、HTML常用标签分类详解
### 1. 元数据标签（head内）
用于配置页面元信息，全部放在`<head>`标签内，核心如下：
| 标签 | 核心作用 | 常用属性/示例 |
|------|----------|---------------|
| `<meta>` | 定义页面元数据，配置字符编码、视口、SEO关键词、描述等 | 1. 字符编码：`<meta charset="UTF-8">`<br>2. 移动端视口：`<meta name="viewport" content="width=device-width, initial-scale=1.0">`<br>3. SEO关键词：`<meta name="keywords" content="HTML,前端,语法">`<br>4. 页面描述：`<meta name="description" content="HTML语法详细总结">` |
| `<title>` | 定义页面标题，显示在浏览器标签栏，SEO核心权重标签 | `<title>HTML语法总结</title>` |
| `<link>` | 引入外部资源，最常用引入CSS样式表、网站图标 | 1. 引入CSS：`<link rel="stylesheet" href="style.css">`<br>2. 网站图标：`<link rel="icon" href="favicon.ico">` |
| `<style>` | 内嵌CSS样式，直接在HTML内写样式 | `<style>body { margin: 0; }</style>` |
| `<script>` | 引入/编写JavaScript代码，可放在head或body末尾，推荐body末尾 | 1. 引入外部JS：`<script src="script.js"></script>`<br>2. 内嵌JS：`<script>console.log('hello')</script>` |
| `<base>` | 定义页面所有相对链接的基础URL，一个页面只能有一个 | `<base href="https://www.example.com/" target="_blank">` |
| `<noscript>` | 当浏览器禁用JavaScript时，显示的替代内容 | `<noscript>请开启JavaScript以正常访问页面</noscript>` |

### 2. 语义化结构标签（HTML5核心）
HTML5新增的语义化标签，替代传统的`<div>`堆砌，提升页面可读性、SEO、无障碍访问，每个标签都有明确的语义和使用场景。
| 标签 | 语义与使用场景 | 注意事项 |
|------|----------------|----------|
| `<header>` | 页面/区块的头部，通常包含logo、导航、标题 | 可多个，不能嵌套在`<footer>`内 |
| `<nav>` | 页面的主导航区域，存放核心导航链接 | 仅用于页面主要导航，不要把所有链接都放进去 |
| `<main>` | 页面的主体内容，包含页面核心的、唯一的内容 | 一个页面**只能有一个**`<main>`，不能放在header/footer/aside/article内 |
| `<article>` | 独立、完整的内容单元，可单独脱离页面使用 | 如：一篇文章、一条新闻、一个帖子、一条评论、一个商品卡片 |
| `<section>` | 页面的逻辑分区/章节，用于划分内容模块 | 必须有标题（h1-h6），是有主题的内容组，不要用来做纯样式容器（用div） |
| `<aside>` | 页面的侧边栏、附属内容，与主体内容相关性低 | 如：侧边栏、广告、相关推荐、引用内容 |
| `<footer>` | 页面/区块的底部，通常包含版权、备案信息、联系方式 | 可多个，一个页面可多个区块的footer |
| `<address>` | 定义联系信息，如作者、网站所有者的联系方式 | 内容通常为邮箱、电话、地址，浏览器默认斜体 |

示例：语义化页面结构
```html
<body>
    <header>
        <h1>网站Logo</h1>
        <nav>
            <ul>
                <li><a href="#">首页</a></li>
                <li><a href="#">文章</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <article>
            <h2>HTML语法详解</h2>
            <p>文章内容...</p>
            <section>
                <h3>基础语法</h3>
                <p>章节内容...</p>
            </section>
        </article>

        <aside>
            <h3>相关推荐</h3>
            <p>推荐内容...</p>
        </aside>
    </main>

    <footer>
        <p>© 2024 版权所有</p>
        <address>联系邮箱：example@xxx.com</address>
    </footer>
</body>
```

### 3. 文本内容与格式化标签
#### （1）标题与段落标签
| 标签 | 作用 | 规范 |
|------|------|------|
| `<h1>`-`<h6>` | 标题标签，h1是最高级，h6是最低级 | 1. 一个页面推荐只有一个h1（主标题）<br>2. 层级不能跳级（h1之后接h2，不能直接接h3）<br>3. 用于语义化标题，不要用来单纯设置字体大小 |
| `<p>` | 段落标签，用于包裹一段文本 | 浏览器会自动给段落添加上下外边距，不能嵌套块级元素 |
| `<br>` | 换行标签，强制文本换行 | 单标签，不要用多个<br>来设置间距，用CSS |
| `<hr>` | 水平线标签，生成一条水平分隔线 | 单标签，用于内容分隔，样式用CSS控制 |

#### （2）文本格式化标签（语义优先）
区分**语义化标签**（带强调、语义含义，SEO和无障碍友好）和**纯样式标签**（仅修改样式，无语义），优先使用语义化标签。
| 语义化标签 | 作用 | 纯样式标签 | 作用 |
|------------|------|------------|------|
| `<strong>` | 重要文本，强烈强调，浏览器默认加粗 | `<b>` | 单纯加粗文本，无语义 |
| `<em>` | 强调文本，语气加重，浏览器默认斜体 | `<i>` | 单纯斜体文本，无语义 |
| `<del>` | 删除的文本，浏览器默认加删除线 | `<s>` | 单纯加删除线，无语义 |
| `<ins>` | 新增/插入的文本，浏览器默认加下划线 | `<u>` | 单纯加下划线，无语义 |
| `<mark>` | 标记/高亮文本，浏览器默认黄色背景 | - | - |
| `<sub>` | 下标文本，如：H₂O | - | - |
| `<sup>` | 上标文本，如：X² | - | - |
| `<blockquote>` | 块级引用，长文本引用，自带缩进 | `<q>` | 行内引用，短文本引用，自动加引号 |
| `<code>` | 行内代码，等宽字体 | `<pre>` | 预格式化文本，保留空格和换行，用于大段代码 |

#### （3）通用容器标签
| 标签 | 类型 | 作用 |
|------|------|------|
| `<div>` | 块级元素 | 通用块级容器，无语义，用于布局、样式分组，是最常用的容器 |
| `<span>` | 行内元素 | 通用行内容器，无语义，用于包裹小段文本、设置行内样式 |

### 4. 列表标签
用于结构化展示列表内容，分为有序列表、无序列表、自定义列表三类，全部为块级元素。
#### （1）无序列表 `<ul>`
用于无顺序的列表，默认带实心圆点符号，直接子元素只能是`<li>`。
```html
<ul>
    <li>列表项1</li>
    <li>列表项2</li>
    <li>列表项3</li>
</ul>
```
常用属性：`type`，设置列表符号类型：`disc`（实心圆，默认）、`circle`（空心圆）、`square`（实心方块）、`none`（无符号）

#### （2）有序列表 `<ol>`
用于有顺序的列表，默认带数字序号，直接子元素只能是`<li>`。
```html
<ol start="1" type="1">
    <li>第一步</li>
    <li>第二步</li>
    <li>第三步</li>
</ol>
```
常用属性：
- `type`：序号类型：`1`（数字，默认）、`a`（小写字母）、`A`（大写字母）、`i`（小写罗马数字）、`I`（大写罗马数字）
- `start`：序号起始值，数字
- `reversed`：布尔属性，序号倒序

#### （3）自定义列表 `<dl>`
用于术语/名称+描述的列表，常用于名词解释、属性说明，由`<dt>`（术语/名称）和`<dd>`（描述/详情）组成。
```html
<dl>
    <dt>HTML</dt>
    <dd>超文本标记语言，用于构建网页结构</dd>
    <dt>CSS</dt>
    <dd>层叠样式表，用于设置网页样式</dd>
</dl>
```

### 5. 链接标签 `<a>`
超链接标签，用于实现页面跳转、锚点定位、文件下载、唤起邮箱/电话等，是行内元素。
#### 核心语法与属性
```html
<a href="跳转地址" target="打开方式" rel="安全属性">链接内容</a>
```
核心属性详解：
1.  **`href`（必填）**：链接目标地址，核心属性
    - 绝对地址：跳转外部网站，如`https://www.baidu.com`
    - 相对地址：跳转站内页面，如`./about.html`
    - 锚点地址：跳转到当前页面指定id的元素，如`#footer`
    - 功能地址：唤起邮箱`mailto:example@xxx.com`、唤起电话`tel:13800138000`、唤起短信`sms:13800138000`
    - 空链接：`href="#"`（点击返回顶部）、`href="javascript:;"`（无跳转，仅触发JS）
2.  **`target`**：链接打开方式
    - `_self`：当前窗口打开，默认值
    - `_blank`：新窗口打开（必须配合`rel="noopener noreferrer"`，避免安全风险）
    - `_parent`：父级窗口打开
    - `_top`：顶级窗口打开
3.  **`rel`**：链接与当前页面的关系，安全与SEO相关
    - `noopener noreferrer`：配合`_blank`使用，防止新窗口获取原页面权限，避免安全漏洞
    - `nofollow`：告诉搜索引擎不要抓取该链接，不传递权重
4.  **`download`**：布尔属性，点击链接直接下载href指定的文件，而不是打开
5.  **`title`**：鼠标悬浮提示文本

#### 锚点链接实现
```html
<!-- 点击跳转到id为footer的元素 -->
<a href="#footer">跳转到页脚</a>

<!-- 目标元素 -->
<div id="footer">页脚内容</div>
```

### 6. 媒体标签（HTML5原生支持）
用于在页面嵌入图片、音频、视频等媒体资源，无需依赖第三方插件。
#### （1）图片标签 `<img>`
单标签，行内块元素，用于嵌入图片，是页面最常用的媒体标签。
核心语法：
```html
<img src="图片地址" alt="图片替代文本" width="宽度" height="高度" loading="lazy">
```
核心属性详解：
1.  **`src`（必填）**：图片地址，支持绝对地址、相对地址、base64地址
2.  **`alt`（必填）**：图片替代文本，SEO和无障碍核心
    - 图片加载失败时，显示该文本
    - 屏幕阅读器会读取该文本，给视障用户
    - 搜索引擎通过该文本识别图片内容
3.  **`width`/`height`**：图片的宽度/高度，单位px，可省略（用CSS控制）
4.  **`loading`**：图片加载方式，`lazy`=懒加载（图片进入可视区域时才加载，提升性能），`eager`=立即加载（默认）

#### （2）音频标签 ``
HTML5新增，用于嵌入音频文件，原生支持播放。
核心语法：
```html
<audio src="audio.mp3" controls loop muted autoplay preload="auto">
    您的浏览器不支持音频播放，请升级浏览器

```
核心属性：
- `src`：音频文件地址，支持mp3、wav、ogg等格式
- `controls`：布尔属性，显示浏览器自带的播放控件（必须，否则用户无法播放）
- `autoplay`：布尔属性，自动播放（多数浏览器禁用，必须配合muted才能生效）
- `loop`：布尔属性，循环播放
- `muted`：布尔属性，静音播放
- `preload`：预加载策略，`auto`/`metadata`/`none`

兼容多格式：用`<source>`标签指定多个音频文件，浏览器自动选择支持的格式
```html
<audio controls>
    <source src="audio.mp3" type="audio/mpeg">
    <source src="audio.ogg" type="audio/ogg">
    您的浏览器不支持音频播放

```

#### （3）视频标签 `<video>`
HTML5新增，用于嵌入视频文件，原生支持播放。
核心语法：
```html
<video src="video.mp4" controls width="800" height="450" loop muted autoplay poster="cover.jpg">
    您的浏览器不支持视频播放，请升级浏览器
</video>
```
核心属性（和audio通用的属性不再重复）：
- `width`/`height`：视频播放器的宽度/高度
- `poster`：视频封面图片地址，播放前显示
- `playsinline`：布尔属性，移动端内联播放，不自动全屏

#### （4）媒体语义标签
- `<figure>`：用于包裹媒体资源（图片、音频、视频）和对应的说明文本
- `<figcaption>`：媒体资源的标题/说明文本，必须放在`<figure>`内
示例：
```html
<figure>
    <img src="example.jpg" alt="示例图片">
    <figcaption>图1：HTML语法示例图</figcaption>
</figure>
```

### 7. 表格标签
用于展示结构化的二维表格数据，禁止用于页面布局（布局用CSS）。
#### （1）完整表格结构
```html
<table border="1" cellpadding="0" cellspacing="0">
    <!-- 表格标题 -->
    <caption>学生成绩表</caption>
    <!-- 表头区域 -->
    <thead>
        <tr>
            <th>姓名</th>
            <th>语文</th>
            <th>数学</th>
        </tr>
    </thead>
    <!-- 表格主体区域 -->
    <tbody>
        <tr>
            <td>张三</td>
            <td>90</td>
            <td>95</td>
        </tr>
        <tr>
            <td>李四</td>
            <td>85</td>
            <td>98</td>
        </tr>
    </tbody>
    <!-- 表尾区域 -->
    <tfoot>
        <tr>
            <td>平均分</td>
            <td>87.5</td>
            <td>96.5</td>
        </tr>
    </tfoot>
</table>
```

#### （2）核心标签与属性
| 标签 | 核心作用 |
|------|----------|
| `<table>` | 表格根标签，包裹所有表格内容 |
| `<caption>` | 表格标题，默认居中显示，一个表格只能有一个 |
| `<thead>` | 表格表头容器，语义化，提升可读性 |
| `<tbody>` | 表格主体容器，存放核心数据行，一个表格可多个 |
| `<tfoot>` | 表格表尾容器，存放汇总行，语义化 |
| `<tr>` | 表格行，包裹单元格，必须放在thead/tbody/tfoot内 |
| `<th>` | 表头单元格，默认加粗、居中，用于表头 |
| `<td>` | 表格数据单元格，存放表格核心数据 |

核心属性：
- 单元格合并：`colspan`（横向合并列）、`rowspan`（纵向合并行）
- 表格样式：`border`（边框宽度）、`cellpadding`（单元格内边距）、`cellspacing`（单元格间距），推荐用CSS控制
- 无障碍：`scope`（表头单元格的作用范围，`col`=列、`row`=行）

### 8. 表单标签（用户交互核心）
表单用于收集用户输入的数据，提交给后端服务器，是前端与后端交互的核心，HTML5对表单做了大量增强。

#### （1）表单根标签 `<form>`
所有表单控件必须包裹在`<form>`标签内，用于定义表单的提交配置。
核心语法：
```html
<form action="提交地址" method="提交方式" enctype="数据编码格式" name="表单名称">
    <!-- 表单控件 -->
</form>
```
核心属性详解：
1.  **`action`**：表单数据提交的后端接口地址，为空则提交到当前页面
2.  **`method`**：HTTP提交方式，核心两种：
    - `get`：默认值，表单数据拼接在url后，明文传输，长度有限制，用于查询、无敏感数据的场景
    - `post`：表单数据放在请求体中，密文传输，长度无限制，用于提交敏感数据、文件上传
3.  **`enctype`**：表单数据的编码格式，仅method="post"时生效
    - `application/x-www-form-urlencoded`：默认值，普通表单提交
    - `multipart/form-data`：文件上传时必须使用该值
4.  **`novalidate`**：布尔属性，关闭浏览器自带的表单验证

#### （2）核心表单控件 `<input>`
单标签，最常用的表单控件，通过`type`属性切换不同的输入类型，核心必填属性`name`（提交给后端的参数名，否则后端无法获取数据）。

##### 基础type类型
| type值 | 控件类型 | 核心说明 |
|--------|----------|----------|
| `text` | 单行文本输入框 | 默认值，用于输入普通文本 |
| `password` | 密码输入框 | 输入内容自动隐藏为圆点 |
| `radio` | 单选框 | 同一组单选框必须有相同的name属性，只能选中一个 |
| `checkbox` | 复选框 | 同一组复选框name属性相同，可选中多个 |
| `submit` | 提交按钮 | 点击提交表单 |
| `reset` | 重置按钮 | 点击重置表单所有控件为初始值 |
| `button` | 普通按钮 | 无默认行为，用于配合JS |
| `file` | 文件选择框 | 用于上传文件，multiple属性支持多选 |
| `hidden` | 隐藏域 | 页面不可见，用于传递固定参数 |

##### HTML5新增type类型
| type值 | 控件类型 | 核心说明 |
|--------|----------|----------|
| `email` | 邮箱输入框 | 浏览器自动验证邮箱格式 |
| `tel` | 手机号输入框 | 移动端自动弹出数字键盘 |
| `number` | 数字输入框 | 只能输入数字，可设置min/max/step限制范围 |
| `date` | 日期选择框 | 浏览器自带日期选择器，选择年月日 |
| `range` | 滑块控件 | 用于选择范围内的数值 |
| `color` | 颜色选择器 | 浏览器自带取色器，选择十六进制颜色值 |
| `search` | 搜索框 | 带清除按钮的搜索输入框 |
| `url` | 网址输入框 | 浏览器自动验证URL格式 |

##### input核心通用属性
| 属性 | 作用 |
|------|------|
| `value` | 控件的值，提交给后端的参数值 |
| `placeholder` | 输入框的提示文本，输入内容后自动消失 |
| `required` | 布尔属性，必填项，提交时浏览器自动验证 |
| `disabled` | 布尔属性，禁用控件，不可编辑、不可提交 |
| `readonly` | 布尔属性，只读控件，不可编辑但可提交 |
| `checked` | 布尔属性，单选/复选框默认选中 |
| `maxlength` | 输入框最大输入字符数 |
| `autofocus` | 布尔属性，页面加载后自动聚焦 |
| `autocomplete` | 自动补全，on=开启，off=关闭 |

#### （3）其他表单控件
| 标签 | 核心作用 | 核心语法/示例 |
|------|----------|---------------|
| `<label>` | 表单控件的标签，点击label即可聚焦/选中对应控件，提升用户体验 | 1. 绑定id方式：<br>`<label for="username">用户名：</label><input type="text" id="username" name="username">`<br>2. 包裹方式：<br>`<label>密码：<input type="password" name="password"></label>` |
| `<textarea>` | 多行文本输入框，用于输入大段文本 | `<textarea name="desc" rows="5" cols="30" placeholder="请输入描述"></textarea>` |
| `<select>` | 下拉选择框，配合`<option>`使用 | `<select name="city">`<br>`<option value="beijing">北京</option>`<br>`<option value="shanghai" selected>上海</option>`<br>`</select>` |
| `<optgroup>` | 下拉选项分组，提升可读性 | `<select name="province">`<br>`<optgroup label="江苏省">`<br>`<option value="nanjing">南京</option>`<br>`</optgroup>`<br>`</select>` |
| `<button>` | 按钮标签，比input按钮更灵活，可包裹文本、图片 | `<button type="submit">提交</button>`<br>type：submit（提交）、reset（重置）、button（普通按钮） |
| `<fieldset>` | 表单控件分组，配合`<legend>`使用 | `<fieldset>`<br>`<legend>用户信息</legend>`<br>表单控件...<br>`</fieldset>` |

#### 完整表单示例
```html
<form action="./api/submit" method="post">
    <fieldset>
        <legend>用户注册</legend>

        <div>
            <label for="username">用户名：</label>
            <input type="text" id="username" name="username" required placeholder="请输入用户名">
        </div>

        <div>
            <label for="password">密码：</label>
            <input type="password" id="password" name="password" required minlength="6" placeholder="请输入密码">
        </div>

        <div>
            <label>性别：</label>
            <label><input type="radio" name="gender" value="male" checked> 男</label>
            <label><input type="radio" name="gender" value="female"> 女</label>
        </div>

        <div>
            <button type="submit">注册</button>
            <button type="reset">重置</button>
        </div>
    </fieldset>
</form>
```

### 9. 其他常用标签
| 标签 | 核心作用 | 核心属性 |
|------|----------|----------|
| `<iframe>` | 内联框架，在当前页面嵌入另一个网页 | `src`：嵌入页面地址<br>`sandbox`：安全沙箱，限制嵌入页面权限 |
| `<canvas>` | 画布标签，通过JS绘制图形、动画、游戏 | `width`/`height`：画布宽高 |
| `<svg>` | 矢量图形标签，绘制可缩放的矢量图形，不会失真 | - |
| `<template>` | 模板标签，内容不会被浏览器渲染，用于JS动态渲染 | - |

## 四、HTML5核心新增特性
1.  **语义化标签**：新增header、nav、main、article等语义化标签，替代div堆砌，提升SEO和无障碍。
2.  **原生多媒体支持**：新增audio、video标签，原生支持音视频播放，无需依赖flash。
3.  **画布与矢量图形**：新增canvas、svg标签，支持2D图形绘制、动画、游戏开发。
4.  **表单增强**：新增email、tel、date等input类型，新增required、placeholder等属性，原生支持表单验证。
5.  **本地存储**：新增localStorage、sessionStorage，实现浏览器端大容量本地数据存储。
6.  **新增API**：地理定位、拖放、Web Worker、Web Socket等API，丰富前端交互能力。
7.  **响应式支持**：meta viewport标签，完美支持移动端适配。

## 五、语法规范与最佳实践
### 1. 核心语法规范
1.  标签名、属性名全部使用小写字母，符合行业通用规范。
2.  所有标签必须正确闭合，双标签成对出现，单标签规范书写。
3.  标签必须正确嵌套，不能交叉嵌套，形成规范的DOM树形结构。
4.  属性值用双引号包裹，布尔属性可省略属性值。
5.  必须声明DOCTYPE，避免浏览器进入怪异模式。
6.  必须设置meta charset="UTF-8"，避免页面乱码。
7.  注释不能嵌套，合理添加注释提升代码可读性。

### 2. 最佳实践
1.  **语义化优先**：优先使用语义化标签，避免全用div+span，提升SEO、无障碍和代码可维护性。
2.  **结构与样式分离**：HTML只负责页面结构，样式全部用CSS实现，不使用内联style、废弃样式标签。
3.  **SEO优化**：合理使用h1-h6标题层级，一个页面一个h1；img标签必须加alt属性；设置title、meta description。
4.  **无障碍访问**：语义化标签、img的alt属性、表单的label标签，适配屏幕阅读器。
5.  **性能优化**：图片加loading="lazy"实现懒加载；script标签放在body末尾，避免阻塞渲染；精简DOM结构。
6.  **安全规范**：a标签target="_blank"必须加rel="noopener noreferrer"；iframe加sandbox属性；过滤用户输入，避免XSS攻击。

## 六、常见错误与避坑指南
1.  忘记写DOCTYPE声明，导致浏览器进入怪异模式，样式渲染错乱。
2.  标签交叉嵌套、不闭合，导致DOM结构错乱，样式、JS失效。
3.  img标签不写alt属性，影响SEO和无障碍，图片加载失败时无替代内容。
4.  滥用div标签，无语义化堆砌，不利于SEO和代码维护。
5.  a标签嵌套a标签，导致浏览器解析异常，链接跳转错误。
6.  p标签嵌套块级元素，浏览器自动截断p标签，结构错乱。
7.  表单input没有name属性，提交后后端无法获取对应数据。
8.  target="_blank"不加rel属性，存在跨页面安全风险。
9.  一个页面多个h1标签，稀释SEO权重，影响搜索引擎排名。
10. 用表格做页面布局，导致页面加载慢、适配性差、维护困难。


