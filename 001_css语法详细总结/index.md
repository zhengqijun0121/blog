# CSS 语法详细总结


# CSS语法详细总结
CSS（Cascading Style Sheets，层叠样式表）是用于控制HTML/XML文档外观、布局和样式的标记语言，核心实现**结构与样式分离**，本文从基础语法到高级特性进行全维度、结构化的详细总结，兼顾语法规范与实战常用技巧。

---

## 一、CSS基础引入方式
CSS有4种核心引入方式，适用场景与优先级各有不同，语法规范如下：

### 1.1 行内样式（内联样式）
直接在HTML标签的`style`属性中编写样式，仅对当前标签生效。
- 语法：`<标签名 style="属性1:值1; 属性2:值2;">内容</标签名>`
- 示例：`<div style="width: 100px; height: 50px; color: red;">行内样式</div>`
- 特点：优先级最高，耦合度高，不利于复用，仅用于临时样式调整。

### 1.2 内嵌样式（内部样式表）
在HTML的`<head>`标签内通过`<style>`标签编写样式，仅对当前HTML文档生效。
- 语法：
```html
<head>
  <style>
    选择器 {
      属性1: 值1;
      属性2: 值2;
    }
  </style>
</head>
```
- 特点：结构与样式初步分离，适用于单页面专属样式，无法多页面复用。

### 1.3 外链样式（外部样式表）
将CSS写在独立的`.css`文件中，通过HTML的`<link>`标签引入，是企业级开发的标准方式。
- 语法：
```html
<!-- HTML文件中 -->
<head>
  <link rel="stylesheet" href="style.css" type="text/css">
</head>
```
```css
/* style.css文件中 */
选择器 {
  属性1: 值1;
  属性2: 值2;
}
```
- 核心属性：`rel="stylesheet"` 固定声明样式表关系，`href`为CSS文件路径
- 特点：完全实现结构与样式分离，可多页面复用，便于维护，浏览器可缓存。

### 1.4 导入样式
通过CSS的`@import`规则在样式表中导入其他CSS文件。
- 语法：
```css
/* 写在CSS文件最顶部，必须先于其他样式规则 */
@import url("common.css");
@import "reset.css";
```
- 特点：属于CSS层级的样式引入，加载顺序晚于`<link>`，会阻塞页面渲染，现代开发不推荐优先使用。

---

## 二、CSS核心基础语法
CSS的核心语法单元是**规则集（Rule Set）**，由「选择器」和「声明块」两部分组成，是所有CSS样式的基础结构。

### 2.1 基础语法结构
```css
/* 规则集完整结构 */
选择器 {
  属性名1: 属性值1; /* 声明1 */
  属性名2: 属性值2; /* 声明2 */
  /* 更多声明 */
}
```
- **选择器**：用于匹配HTML中需要添加样式的目标元素，是样式的作用目标
- **声明块**：包裹在`{}`中，由一条或多条声明组成
- **声明**：由`属性名: 属性值;`组成，`;`是声明的结束符，单条声明末尾可省略，多条声明必须严格书写
- **注释**：仅支持`/* 注释内容 */`格式，可单行/多行，不支持`//`单行注释，注释内容不会被浏览器解析

### 2.2 语法基础规范
1. CSS对大小写不敏感，但**选择器、属性名、属性值推荐全小写**（仅自定义属性、字体名称区分大小写）
2. 多个属性值之间用空格分隔，多个选择器之间用逗号分隔
3. 特殊字符、空格包裹的属性值需要加引号（如字体名称含空格、背景图路径）
4. 缩进与换行不影响解析，推荐按规范格式化，提升可读性

---

## 三、CSS选择器大全
选择器是CSS的核心，用于精准匹配HTML元素，按功能可分为6大类，覆盖全场景匹配需求。

### 3.1 基础选择器（核心）
| 选择器 | 语法 | 作用 | 优先级 | 示例 |
| :--- | :--- | :--- | :--- | :--- |
| 通配符选择器 | `*` | 匹配页面所有元素 | 最低 | `* { margin: 0; padding: 0; }` 清除默认内外边距 |
| 标签选择器 | 标签名 | 匹配对应类型的所有HTML标签 | 低 | `p { line-height: 1.5; }` 匹配所有p标签 |
| 类选择器 | `.类名` | 匹配所有class属性包含该类名的元素 | 中 | `.box { width: 200px; }` 匹配`<div class="box"></div>` |
| ID选择器 | `#ID名` | 匹配id属性等于该名称的唯一元素 | 高 | `#header { height: 80px; }` 匹配`<div id="header"></div>` |

> 注意：ID在HTML中必须唯一，类名可重复使用；ID选择器优先级远高于类选择器，非必要不滥用。

### 3.2 复合选择器（关系匹配）
基于元素的父子、兄弟关系进行匹配，是精准定位元素的核心方式。
1. **后代选择器**（空格分隔）：`选择器A 选择器B`，匹配A元素内所有的B元素（包括孙子、重孙等所有后代）
   - 示例：`div p { color: #333; }` 匹配div内所有层级的p标签
2. **子代选择器**（`>`分隔）：`选择器A > 选择器B`，仅匹配A元素的直接子元素B（不包含孙子级及以下）
   - 示例：`ul > li { list-style: none; }` 仅匹配ul的直接子li标签
3. **相邻兄弟选择器**（`+`分隔）：`选择器A + 选择器B`，匹配紧跟在A后面的第一个同级兄弟元素B
   - 示例：`h2 + p { margin-top: 10px; }` 匹配h2后面紧邻的第一个p标签
4. **通用兄弟选择器**（`~`分隔）：`选择器A ~ 选择器B`，匹配A后面所有同级的兄弟元素B
   - 示例：`.active ~ li { opacity: 0.5; }` 匹配.active后面所有同级li标签
5. **并集选择器**（`,`分隔）：`选择器A, 选择器B`，同时匹配A和B，实现样式复用
   - 示例：`h1, h2, h3 { font-weight: 600; }` 同时匹配所有h1/h2/h3标签
6. **交集选择器**（连写）：`选择器A选择器B`，匹配同时满足A和B的元素
   - 示例：`div.box { border: 1px solid #000; }` 仅匹配class包含box的div标签

### 3.3 属性选择器
基于元素的属性及属性值进行匹配，CSS3新增了模糊匹配能力，适配表单、链接等场景。
| 语法 | 匹配规则 | 示例 |
| :--- | :--- | :--- |
| `[attr]` | 匹配包含attr属性的所有元素 | `a[target]` 匹配带target属性的a标签 |
| `[attr="val"]` | 匹配attr属性值**完全等于val**的元素 | `input[type="text"]` 匹配文本输入框 |
| `[attr~="val"]` | 匹配attr属性值包含**独立单词val**的元素 | `div[class~="box"]` 匹配class含独立box的div |
| `[attr|="val"]` | 匹配attr属性值以val开头、且以`-`分隔的元素 | `div[lang|="zh"]` 匹配lang=zh/zh-CN的div |
| `[attr^="val"]` | 匹配attr属性值**以val开头**的元素（CSS3） | `a[href^="https"]` 匹配https开头的安全链接 |
| `[attr$="val"]` | 匹配attr属性值**以val结尾**的元素（CSS3） | `img[src$=".png"]` 匹配所有png格式图片 |
| `[attr*="val"]` | 匹配attr属性值**包含val字符**的元素（CSS3） | `input[name*="user"]` 匹配name含user的输入框 |

### 3.4 伪类选择器
伪类用于匹配元素的**特定状态、结构位置**，以`:`开头，需依附于基础选择器，当元素达到指定状态时样式生效。

#### 3.4.1 链接/交互状态伪类（最常用）
| 伪类 | 作用 | 注意事项 |
| :--- | :--- | :--- |
| `:link` | 匹配未被访问的链接 | 仅适用于a标签 |
| `:visited` | 匹配已被访问的链接 | 仅适用于a标签，可修改样式有限（安全限制） |
| `:hover` | 匹配鼠标悬浮的元素 | 所有元素都支持，是交互样式核心 |
| `:active` | 匹配鼠标点击激活状态的元素 | 点击按下未松开时生效 |
| `:focus` | 匹配获得焦点的元素 | 多用于input、button等表单元素 |

> 规范顺序：`link → visited → hover → active`，顺序错误会导致样式失效，简称LVHA规则。

#### 3.4.2 结构伪类（CSS3核心，基于DOM结构匹配）
| 伪类 | 匹配规则 | 示例 |
| :--- | :--- | :--- |
| `:first-child` | 匹配父元素的第一个子元素 | `li:first-child` 匹配父元素中第一个li |
| `:last-child` | 匹配父元素的最后一个子元素 | `li:last-child` 匹配父元素中最后一个li |
| `:nth-child(n)` | 匹配父元素中第n个子元素，n从1开始 | `li:nth-child(2)` 匹配第2个li；`li:nth-child(2n)` 匹配偶数行；`li:nth-child(2n+1)` 匹配奇数行 |
| `:nth-last-child(n)` | 从父元素末尾倒序匹配第n个子元素 | `li:nth-last-child(2)` 匹配倒数第2个li |
| `:only-child` | 匹配父元素中唯一的子元素 | `p:only-child` 匹配父元素中仅有的一个p标签 |
| `:first-of-type` | 匹配父元素中同类型的第一个元素 | `p:first-of-type` 匹配父元素中第一个p标签 |
| `:last-of-type` | 匹配父元素中同类型的最后一个元素 | `p:last-of-type` 匹配父元素中最后一个p标签 |
| `:nth-of-type(n)` | 匹配父元素中同类型的第n个元素 | `div:nth-of-type(3)` 匹配父元素中第3个div |

> 核心区别：`nth-child` 匹配所有子元素，不区分类型；`nth-of-type` 先筛选同类型元素，再匹配序号，是开发中更常用的方式。

#### 3.4.3 其他常用伪类
| 伪类 | 作用 |
| :--- | :--- |
| `:enabled` / `:disabled` | 匹配启用/禁用状态的表单元素 |
| `:checked` | 匹配被选中的单选框、复选框 |
| `:required` / `:optional` | 匹配必填/选填的表单元素 |
| `:not(选择器)` | 匹配不满足括号内选择器的元素（排除匹配） |
| `:target` | 匹配当前URL锚点指向的目标元素 |
| `:root` | 匹配文档根元素（HTML中的html标签），用于定义全局CSS变量 |

### 3.5 伪元素选择器
伪元素用于创建**DOM中不存在的虚拟元素**，或匹配元素的特定部分，CSS3规范中以`::`开头（单冒号`:`兼容旧浏览器），必须配合`content`属性使用。

| 伪元素 | 作用 | 核心示例 |
| :--- | :--- | :--- |
| `::before` | 在元素内容的**最前面**创建一个虚拟子元素 | `.btn::before { content: "★"; margin-right: 5px; }` 按钮前加图标 |
| `::after` | 在元素内容的**最后面**创建一个虚拟子元素 | `.clearfix::after { content: ""; display: block; clear: both; }` 清除浮动 |
| `::first-letter` | 匹配块级元素文本的第一个字符 | `p::first-letter { font-size: 2em; color: red; }` 首字下沉 |
| `::first-line` | 匹配块级元素文本的第一行 | `p::first-line { font-weight: 600; }` 首行加粗 |
| `::selection` | 匹配用户鼠标选中的文本内容 | `::selection { background: #165DFF; color: #fff; }` 自定义选中文本样式 |

> 核心区别：伪类是匹配元素的**状态/位置**，不改变DOM结构；伪元素是创建**新的虚拟元素**，可修改内容结构。

---

## 四、CSS核心单位与颜色表示
### 4.1 CSS长度单位
分为**绝对单位**和**相对单位**两大类，相对单位是响应式开发的核心。

#### 4.1.1 绝对单位（固定值，不随外部环境变化）
| 单位 | 含义 | 适用场景 |
| :--- | :--- | :--- |
| `px`（像素） | 屏幕最小显示单位，最常用 | 绝大多数固定尺寸、边框、内边距 |
| `pt`（磅） | 印刷单位，1pt=1/72英寸 | 打印样式、字体印刷设置 |
| `cm`/`mm`/`in` | 厘米/毫米/英寸 | 打印样式、物理尺寸相关设计 |

#### 4.1.2 相对单位（基于参考对象动态计算，响应式核心）
| 单位 | 参考基准 | 核心用法与场景 |
| :--- | :--- | :--- |
| `%` | 相对于父元素的对应属性 | 宽度、高度、内外边距的百分比适配，流式布局 |
| `em` | 相对于当前元素的`font-size`，若自身未设置则继承父元素 | 段落缩进、行高、内边距的字体联动适配 |
| `rem` | 相对于根元素`<html>`的`font-size` | 移动端响应式布局，全局统一适配，无层级嵌套影响 |
| `vw` | 视口宽度的1%，1vw=视口宽度×1% | 移动端全屏适配、响应式字体、宽度自适应 |
| `vh` | 视口高度的1%，1vh=视口高度×1% | 全屏高度容器、自适应高度布局 |
| `vmin` | vw和vh中的较小值 | 横竖屏切换时的最小尺寸适配，保证元素完整显示 |
| `vmax` | vw和vh中的较大值 | 横竖屏切换时的最大尺寸适配，保证元素占比 |

### 4.2 CSS函数型单位（动态计算）
| 函数 | 语法 | 作用 |
| :--- | :--- | :--- |
| `calc()` | `calc(表达式)` | 动态计算长度值，支持加减乘除运算，运算符前后必须加空格 |
| `min()` | `min(val1, val2, ...)` | 取多个值中的最小值，适配上限 |
| `max()` | `max(val1, val2, ...)` | 取多个值中的最大值，适配下限 |
| `clamp()` | `clamp(最小值, 首选值, 最大值)` | 动态锁定取值范围，超出范围时取边界值，响应式字体首选 |

> 示例：`font-size: clamp(12px, 2vw, 16px);` 字体最小12px，最大16px，正常随视口宽度2vw变化。

### 4.3 CSS颜色表示方式
CSS提供5种主流颜色表示方法，支持透明度设置，适配不同设计场景。
1. **颜色关键字**：直接使用预定义的颜色名称，如`red`、`blue`、`transparent`（透明）、`currentColor`（继承当前color值）
2. **十六进制颜色**：语法`#RRGGBB`，RR/GG/BB为00-FF的十六进制数，可简写`#RGB`（两两重复时），支持透明度`#RRGGBBAA`/`#RGBA`
   - 示例：`#ff0000`（红色）、`#f00`（简写红色）、`#ff000080`（半透明红色）
3. **RGB/RGBA模式**：
   - 语法：`rgb(红, 绿, 蓝)`，取值0-255或0%-100%
   - 语法：`rgba(红, 绿, 蓝, 透明度)`，透明度0（完全透明）-1（完全不透明）
   - 示例：`rgb(255, 0, 0)`（红色）、`rgba(255, 0, 0, 0.5)`（50%透明红色）
4. **HSL/HSLA模式**：基于色相、饱和度、亮度的直观配色模式，设计首选
   - 语法：`hsl(色相, 饱和度%, 亮度%)`，色相0-360（0红、120绿、240蓝）
   - 语法：`hsla(色相, 饱和度%, 亮度%, 透明度)`
   - 示例：`hsl(0, 100%, 50%)`（红色）、`hsla(0, 100%, 50%, 0.5)`（半透明红色）
5. **渐变颜色**：CSS3提供的渐变函数，用于背景、边框等场景，详见下文高级特性。

---

## 五、CSS核心属性分类详解
CSS属性按功能可分为布局、盒模型、文本字体、背景、装饰五大类，覆盖90%以上的开发场景。

### 5.1 文本属性
用于控制文本的显示效果，核心属性如下：
| 属性 | 核心取值 | 作用 |
| :--- | :--- | :--- |
| `color` | 颜色值 | 设置文本颜色 |
| `text-align` | left/center/right/justify | 设置文本水平对齐方式 |
| `text-decoration` | none/underline/overline/line-through | 设置文本装饰线（下划线、删除线等） |
| `text-indent` | 长度单位/百分比 | 设置首行文本缩进 |
| `line-height` | 数字/长度单位/百分比 | 设置行高，多行文本垂直居中、行间距控制核心 |
| `letter-spacing` | 长度单位 | 设置字符间距 |
| `word-spacing` | 长度单位 | 设置单词间距 |
| `text-transform` | none/capitalize/uppercase/lowercase | 设置文本大小写转换 |
| `white-space` | normal/nowrap/pre/pre-wrap/pre-line | 设置空白符处理与换行规则 |
| `text-overflow` | clip/ellipsis | 设置文本溢出容器的显示方式，配合overflow:hidden使用 |

> 常用实战技巧：
> 1. 单行文本溢出省略号：
> ```css
> .text-ellipsis {
>   white-space: nowrap;
>   overflow: hidden;
>   text-overflow: ellipsis;
> }
> ```
> 2. 多行文本溢出省略号（webkit内核）：
> ```css
> .text-ellipsis-multi {
>   overflow: hidden;
>   text-overflow: ellipsis;
>   display: -webkit-box;
>   -webkit-line-clamp: 2; /* 控制显示行数 */
>   -webkit-box-orient: vertical;
> }
> ```

### 5.2 字体属性
用于控制字体的显示样式，核心属性如下：
| 属性 | 核心取值 | 作用 |
| :--- | :--- | :--- |
| `font-family` | 字体名称列表 | 设置字体族，多个字体用逗号分隔，优先使用前面的字体 |
| `font-size` | 长度单位/百分比/关键字 | 设置字体大小 |
| `font-weight` | normal/bold/100-900 | 设置字体粗细，400=normal，700=bold |
| `font-style` | normal/italic/oblique | 设置字体样式（正常/斜体） |
| `font-variant` | normal/small-caps | 设置小型大写字母 |

> 字体简写属性：必须按固定顺序书写，否则样式失效
> 语法：`font: font-style font-weight font-size/line-height font-family;`
> 示例：`font: italic 600 16px/1.5 "Microsoft YaHei", sans-serif;`
> 注意：`font-size`和`font-family`是必填项，其他属性可省略。

### 5.3 背景属性
用于控制元素的背景样式，CSS3大幅增强了背景能力，核心属性如下：
| 属性 | 核心取值 | 作用 |
| :--- | :--- | :--- |
| `background-color` | 颜色值 | 设置背景颜色，默认transparent |
| `background-image` | url(图片路径)/渐变函数 | 设置背景图片，可设置多张 |
| `background-repeat` | repeat/no-repeat/repeat-x/repeat-y | 设置背景图片平铺方式 |
| `background-position` | 方位词(left/center/right/top/bottom)/长度单位/百分比 | 设置背景图片位置 |
| `background-size` | 长度单位/百分比/contain/cover | 设置背景图片尺寸，CSS3新增 |
| `background-attachment` | scroll/fixed/local | 设置背景图片是否随页面滚动 |
| `background-origin` | padding-box/border-box/content-box | 设置背景图片的定位原点，CSS3新增 |
| `background-clip` | border-box/padding-box/content-box/text | 设置背景的裁剪区域，CSS3新增 |

> 背景简写属性：无严格顺序要求，推荐按以下顺序书写
> 语法：`background: color image repeat position/size attachment origin clip;`
> 示例：`background: #f5f5f5 url("bg.png") no-repeat center/cover fixed;`
> 注意：`position`和`size`必须用`/`分隔，且必须按`position/size`顺序书写。

### 5.4 盒模型核心属性
CSS盒模型是布局的核心，所有HTML元素都可看作一个盒子，由**内容content、内边距padding、边框border、外边距margin**四部分组成。

#### 5.4.1 盒模型类型（box-sizing）
`box-sizing`属性控制盒子宽高的计算方式，是解决布局错乱的核心属性。
| 属性值 | 计算规则 | 特点 |
| :--- | :--- | :--- |
| `content-box`（默认） | 标准盒模型，W3C规范<br>元素总宽度 = width + padding + border + margin<br>元素总高度 = height + padding + border + margin | width/height仅设置内容区大小，padding/border会撑大盒子 |
| `border-box` | 怪异盒模型（IE盒模型）<br>元素总宽度 = width + margin<br>元素总高度 = height + margin | width/height包含内容区+padding+border，padding/border不会撑大盒子，开发首选 |

> 示例：
> ```css
> /* 全局设置怪异盒模型，企业级开发标配 */
> * {
>   box-sizing: border-box;
> }
> .box {
>   width: 200px;
>   padding: 20px;
>   border: 10px solid #000;
>   /* border-box下，盒子实际内容宽度=200-20*2-10*2=140px，总宽度仍为200px */
> }
> ```

#### 5.4.2 内边距padding
用于设置元素内容与边框之间的间距，不可为负值。
- 语法：
  ```css
  /* 四值写法：上 右 下 左（顺时针） */
  padding: 10px 20px 30px 40px;
  /* 三值写法：上 左右 下 */
  padding: 10px 20px 30px;
  /* 两值写法：上下 左右 */
  padding: 10px 20px;
  /* 单值写法：四个方向统一 */
  padding: 10px;
  /* 单方向设置 */
  padding-top: 10px;
  padding-right: 20px;
  padding-bottom: 30px;
  padding-left: 40px;
  ```

#### 5.4.3 边框border
用于设置元素的边框，由边框宽度、边框样式、边框颜色三部分组成。
- 基础语法：
  ```css
  /* 简写：border: 宽度 样式 颜色; */
  border: 1px solid #333;
  /* 单方向设置 */
  border-top: 2px dashed #f00;
  border-right: 3px dotted #0f0;
  border-bottom: 4px double #00f;
  border-left: 1px solid #333;
  ```
- 核心边框样式：`solid`（实线）、`dashed`（虚线）、`dotted`（点线）、`double`（双线）、`none`（无边框）
- CSS3边框扩展：
  1. `border-radius`：圆角属性，语法同padding，支持百分比，`border-radius: 50%`可实现圆形
  2. `box-shadow`：盒子阴影，语法：`box-shadow: 水平偏移 垂直偏移 模糊半径 扩散半径 颜色 inset(内阴影);`
     - 示例：`box-shadow: 0 2px 8px rgba(0,0,0,0.1);` 常用卡片阴影
  3. `border-image`：边框图片，CSS3新增，用图片替代纯色边框

#### 5.4.4 外边距margin
用于设置元素与元素之间的间距，可为负值。
- 语法：与padding完全一致，支持四值、三值、两值、单值写法，支持单方向设置`margin-top/right/bottom/left`
- 核心特性：
  1. **水平居中**：块级元素设置`margin: 0 auto;`可实现水平居中（前提：元素必须设置固定width）
  2. **外边距合并（塌陷）**：垂直方向的相邻外边距会发生合并，取最大值，分为两种场景：
     - 兄弟元素合并：上下两个兄弟元素，上元素的margin-bottom和下元素的margin-top合并，取最大值
     - 父子元素合并：父元素无border/padding/overflow:hidden，子元素的margin-top会传递给父元素，导致父元素整体下移
  3. 合并解决方案：
     - 兄弟元素：仅给其中一个元素设置margin，避免垂直方向相邻margin
     - 父子元素：给父元素添加`overflow: hidden`、`border-top`、`padding-top`，或触发BFC

### 5.5 列表与表格属性
#### 列表属性
| 属性 | 核心取值 | 作用 |
| :--- | :--- | :--- |
| `list-style-type` | none/disc/circle/square/decimal等 | 设置列表项标记类型 |
| `list-style-position` | inside/outside | 设置列表标记的位置 |
| `list-style-image` | url(图片路径) | 用图片替换列表标记 |
| `list-style` | 简写属性 | 示例：`list-style: none;` 清除列表默认样式，开发最常用 |

#### 表格属性
| 属性 | 核心取值 | 作用 |
| :--- | :--- | :--- |
| `border-collapse` | collapse/separate | 设置表格边框合并，`collapse`合并相邻边框，开发首选 |
| `border-spacing` | 长度单位 | 设置边框间距，仅separate模式生效 |
| `caption-side` | top/bottom | 设置表格标题位置 |
| `empty-cells` | show/hide | 设置空单元格是否显示边框 |
| `table-layout` | auto/fixed | 设置表格布局算法，fixed固定列宽，渲染更快 |

---

## 六、CSS布局核心语法
布局是CSS的核心应用场景，主流布局方式包括：常规流布局、浮动布局、定位布局、Flex弹性布局、Grid网格布局五大类。

### 6.1 常规流布局（文档流）
默认布局方式，元素按HTML书写顺序，从上到下、从左到右依次排列：
- 块级元素（div、p、h1-h6等）：独占一行，可设置width/height，默认宽度100%
- 行内元素（span、a、em等）：并排显示，不可设置width/height，宽高由内容撑开
- 行内块元素（img、input、button等）：并排显示，可设置width/height，不独占一行

> 控制元素显示类型的核心属性：`display`
> 核心取值：`block`（块级）、`inline`（行内）、`inline-block`（行内块）、`none`（隐藏元素，不占页面空间）

### 6.2 浮动布局（float）
通过`float`属性实现元素的横向排列，是早期多列布局的核心方案，目前多用于图文环绕场景。
- 核心取值：`left`（左浮动）、`right`（右浮动）、`none`（不浮动，默认）
- 核心特性：
  1. 浮动元素会**脱离文档流**，不占常规流空间，父元素高度会塌陷（子元素全浮动时，父元素高度变为0）
  2. 浮动元素之间横向并排，超出父元素宽度会自动换行
  3. 浮动元素不会覆盖文字，实现文字环绕效果
- 核心问题：父元素高度塌陷，解决方案（清除浮动）
  ```css
  /* 方案1：单伪元素清除法（推荐） */
  .clearfix::after {
    content: "";
    display: block;
    clear: both;
  }
  /* 方案2：overflow触发BFC */
  .parent {
    overflow: hidden;
  }
  ```
> `clear`属性：用于清除浮动对当前元素的影响，取值`left`/`right`/`both`，`both`清除所有浮动影响。

### 6.3 定位布局（position）
通过`position`属性设置元素的定位方式，精准控制元素的位置，适用于弹窗、悬浮、固定导航等场景。

| 定位类型 | 取值 | 定位基准 | 是否脱离文档流 | 核心特点 |
| :--- | :--- | :--- | :--- | :--- |
| 静态定位 | `static` | 无，默认文档流 | 否 | 正常文档流，top/left等偏移属性无效 |
| 相对定位 | `relative` | 元素自身原来的位置 | 否 | 不脱离文档流，原来的位置保留，仅视觉偏移 |
| 绝对定位 | `absolute` | 最近的**已定位（非static）**祖先元素，无则相对于html根元素 | 是 | 脱离文档流，不占空间，宽高默认由内容撑开 |
| 固定定位 | `fixed` | 浏览器视口（viewport） | 是 | 脱离文档流，页面滚动时位置固定不变，适用于固定导航、回到顶部按钮 |
| 粘性定位 | `sticky` | 视口+最近的滚动祖先元素 | 否 | 结合relative和fixed特性，滚动到指定位置时变为固定定位，适用于吸顶导航 |

- 核心偏移属性：`top`、`right`、`bottom`、`left`，设置元素相对于定位基准的偏移量，仅非static定位生效
- 层叠属性：`z-index`，设置元素的层叠顺序，数值越大越靠上，仅非static定位生效，默认值auto，可设负值
  > 注意：`z-index`仅在同一个层叠上下文中比较，父元素`z-index`会限制子元素的层叠层级。

### 6.4 Flex弹性布局（CSS3）
Flex是一维布局模型，是目前移动端、PC端主流的布局方案，完美解决元素对齐、均分、自适应等问题，告别浮动和定位的繁琐。

#### 6.4.1 核心概念
- 弹性容器：设置`display: flex;`的元素，称为flex容器
- 弹性项目：容器的直接子元素，自动成为flex项目
- 主轴：项目排列的方向，默认水平从左到右
- 交叉轴：与主轴垂直的方向，默认垂直从上到下

#### 6.4.2 容器属性（父元素设置）
| 属性 | 核心取值 | 作用 |
| :--- | :--- | :--- |
| `flex-direction` | row（默认）/row-reverse/column/column-reverse | 设置主轴方向（水平/垂直） |
| `flex-wrap` | nowrap（默认）/wrap/wrap-reverse | 设置项目是否换行，nowrap默认不换行，会压缩项目宽度 |
| `flex-flow` | 简写 | `flex-flow: direction wrap;` 合并主轴方向和换行设置 |
| `justify-content` | flex-start/flex-end/center/space-between/space-around/space-evenly | 设置项目在**主轴**上的对齐方式 |
| `align-items` | stretch（默认）/flex-start/flex-end/center/baseline | 设置项目在**交叉轴**上的对齐方式 |
| `align-content` | stretch/flex-start/flex-end/center/space-between/space-around | 设置多行项目在交叉轴上的对齐方式，仅wrap换行生效 |

#### 6.4.3 项目属性（子元素设置）
| 属性 | 核心取值 | 作用 |
| :--- | :--- | :--- |
| `order` | 数字，默认0 | 设置项目的排列顺序，数值越小越靠前，可设负值 |
| `flex-grow` | 数字，默认0 | 设置项目的放大比例，父容器有剩余空间时，按比例分配剩余空间 |
| `flex-shrink` | 数字，默认1 | 设置项目的缩小比例，父容器空间不足时，按比例缩小项目 |
| `flex-basis` | 长度单位，默认auto | 设置项目在主轴上的初始大小，优先级高于width/height |
| `flex` | 简写属性 | `flex: grow shrink basis;`，默认值`0 1 auto`，常用`flex: 1;` 自适应占满剩余空间 |
| `align-self` | auto/stretch/flex-start/flex-end/center/baseline | 单独设置当前项目在交叉轴上的对齐方式，覆盖容器的align-items |

> 常用示例：水平垂直居中
> ```css
> .parent {
>   display: flex;
>   justify-content: center; /* 主轴水平居中 */
>   align-items: center; /* 交叉轴垂直居中 */
> }
> ```

### 6.5 Grid网格布局（CSS3）
Grid是二维布局模型，同时控制行和列，是复杂多列布局的最优方案，适配大屏、复杂网格场景。

#### 6.5.1 核心概念
- 网格容器：设置`display: grid;`的元素
- 网格项目：容器的直接子元素
- 网格线：划分网格的线，水平为行网格线，垂直为列网格线
- 网格轨道：两条相邻网格线之间的空间，即行和列
- 网格单元格：行和列交叉的区域，最小布局单位

#### 6.5.2 容器属性（父元素设置）
| 属性 | 核心用法 | 作用 |
| :--- | :--- | :--- |
| `grid-template-columns` | `grid-template-columns: 100px 1fr 2fr;` | 设置列的数量和宽度，`fr`为剩余空间分配单位，支持`repeat(3, 1fr)`重复创建 |
| `grid-template-rows` | `grid-template-rows: 80px 1fr 60px;` | 设置行的数量和高度，用法同columns |
| `grid-template-areas` | 命名网格区域 | 给网格区域命名，配合项目的grid-area使用，实现语义化布局 |
| `gap` / `grid-gap` | `gap: 10px 20px;` | 设置行间距和列间距，简写`gap: 10px` 行列间距统一 |
| `justify-items` | start/end/center/stretch | 设置项目在单元格内的水平对齐方式 |
| `align-items` | start/end/center/stretch | 设置项目在单元格内的垂直对齐方式 |
| `place-items` | 简写 | `place-items: align justify;` 合并水平垂直对齐 |
| `justify-content` | start/end/center/space-between等 | 设置整个网格在容器内的水平对齐方式（容器宽度大于网格总宽度时生效） |
| `align-content` | start/end/center/space-between等 | 设置整个网格在容器内的垂直对齐方式 |
| `place-content` | 简写 | 合并justify-content和align-content |

#### 6.5.3 项目属性（子元素设置）
| 属性 | 核心用法 | 作用 |
| :--- | :--- | :--- |
| `grid-column` | `grid-column: 1 / 3;` | 设置项目占据的列范围，基于网格线，`起始线 / 结束线` |
| `grid-row` | `grid-row: 2 / 4;` | 设置项目占据的行范围，用法同column |
| `grid-area` | 区域名称/网格线范围 | 配合grid-template-areas使用，指定项目所在的命名区域 |
| `justify-self` | start/end/center/stretch | 单独设置当前项目在单元格内的水平对齐方式 |
| `align-self` | start/end/center/stretch | 单独设置当前项目在单元格内的垂直对齐方式 |
| `place-self` | 简写 | 合并justify-self和align-self |

> 常用示例：三栏布局
> ```css
> .container {
>   display: grid;
>   grid-template-columns: 200px 1fr 200px; /* 左200px，中间自适应，右200px */
>   grid-template-rows: 80px 1fr 60px; /* 头部80px，主体自适应，底部60px */
>   grid-template-areas:
>     "header header header"
>     "aside main sidebar"
>     "footer footer footer";
>   gap: 10px;
> }
> .header { grid-area: header; }
> .aside { grid-area: aside; }
> .main { grid-area: main; }
> .sidebar { grid-area: sidebar; }
> .footer { grid-area: footer; }
> ```

---

## 七、CSS3高级特性
### 7.1 过渡transition
过渡用于实现元素属性变化时的平滑动画，是交互效果的核心，无需JavaScript即可实现简单动画。
- 核心语法：`transition: 过渡属性 过渡时长 速度曲线 延迟时间;`
- 拆分属性：
  | 属性 | 作用 | 示例 |
  | :--- | :--- | :--- |
  | `transition-property` | 指定需要过渡的CSS属性，`all`表示所有可过渡属性 | `transition-property: width, background;` |
  | `transition-duration` | 过渡动画的时长，单位s/ms，必填项 | `transition-duration: 0.3s;` |
  | `transition-timing-function` | 过渡的速度曲线 | `ease`（默认，先快后慢）/`linear`（匀速）/`ease-in`（先慢后快）/`ease-out`（先快后慢）/`cubic-bezier()`自定义 |
  | `transition-delay` | 过渡动画的延迟执行时间，单位s/ms | `transition-delay: 0.1s;` |
- 示例：
  ```css
  .btn {
    width: 100px;
    height: 40px;
    background: #165DFF;
    /* 所有属性变化，0.3s平滑过渡，匀速 */
    transition: all 0.3s linear;
  }
  .btn:hover {
    background: #4080ff;
    transform: scale(1.05);
  }
  ```
> 注意：不是所有CSS属性都支持过渡，仅数值型属性（宽高、颜色、位置、transform等）支持，display:none不支持过渡。

### 7.2 动画animation
animation实现复杂的关键帧动画，可控制动画的播放次数、方向、填充模式等，比transition更强大。

#### 7.2.1 第一步：定义关键帧@keyframes
```css
/* 语法：@keyframes 动画名称 { 关键帧节点 { 样式 } } */
@keyframes move {
  /* 起始状态，等价于0% */
  from {
    transform: translateX(0);
  }
  /* 结束状态，等价于100% */
  to {
    transform: translateX(200px);
  }
}
/* 多节点关键帧 */
@keyframes fadeInOut {
  0% { opacity: 0; }
  50% { opacity: 1; }
  100% { opacity: 0; }
}
```

#### 7.2.2 第二步：调用动画属性
- 核心简写语法：`animation: 动画名称 时长 速度曲线 延迟 播放次数 播放方向 填充模式 播放状态;`
- 拆分属性：
  | 属性 | 作用 | 核心取值 |
  | :--- | :--- | :--- |
  | `animation-name` | 调用的@keyframes动画名称，必填 | 自定义动画名 |
  | `animation-duration` | 动画单次播放时长，单位s/ms，必填 | 如1s、500ms |
  | `animation-timing-function` | 动画速度曲线 | 同transition，支持steps()帧动画 |
  | `animation-delay` | 动画延迟播放时间 | 单位s/ms，可设负值 |
  | `animation-iteration-count` | 动画播放次数 | 数字，默认1，`infinite`无限循环 |
  | `animation-direction` | 动画播放方向 | `normal`（默认）/`reverse`反向/`alternate`正反交替/`alternate-reverse`反正交替 |
  | `animation-fill-mode` | 动画结束后的填充状态 | `none`（默认，回到初始状态）/`forwards`（停在结束帧）/`backwards`（延迟时应用起始帧）/`both`（同时应用forwards和backwards） |
  | `animation-play-state` | 动画播放状态 | `running`（默认，播放）/`paused`（暂停），可配合hover实现暂停 |

- 示例：无限循环呼吸动画
  ```css
  .breath {
    width: 100px;
    height: 100px;
    background: #165DFF;
    border-radius: 50%;
    animation: breath 2s ease-in-out infinite alternate;
  }
  @keyframes breath {
    from {
      transform: scale(0.8);
      opacity: 0.8;
    }
    to {
      transform: scale(1.2);
      opacity: 1;
    }
  }
  ```

### 7.3 变形transform
transform用于对元素进行平移、旋转、缩放、倾斜等变形操作，配合过渡/动画实现炫酷效果，不会影响其他元素的布局。
- 核心语法：`transform: 变形函数1 变形函数2 ...;` 多个变形函数用空格分隔，执行顺序从右到左
- 2D变形核心函数：
  | 函数 | 作用 | 示例 |
  | :--- | :--- | :--- |
  | `translate(x, y)` | 平移，沿X/Y轴移动元素 | `translate(100px, 50px)` 右移100px，下移50px；`translateX(100px)` 仅X轴；`translateY(50px)` 仅Y轴 |
  | `rotate(角度)` | 旋转，单位deg，正数顺时针，负数逆时针 | `rotate(45deg)` 顺时针旋转45度 |
  | `scale(x, y)` | 缩放，数值为缩放比例，1为默认 | `scale(1.2)` 整体放大1.2倍；`scaleX(0.8)` 仅X轴缩放 |
  | `skew(x角度, y角度)` | 倾斜，单位deg | `skew(10deg, 5deg)` X轴倾斜10度，Y轴倾斜5度 |
- 3D变形核心函数：
  | 函数 | 作用 |
  | :--- | :--- |
  | `translate3d(x, y, z)` | 3D平移，Z轴平移需配合透视属性 |
  | `rotate3d(x, y, z, 角度)` | 3D旋转，x/y/z为0-1的数值，代表旋转轴的向量 |
  | `scale3d(x, y, z)` | 3D缩放 |
- 辅助属性：
  1. `transform-origin`：设置变形的原点，默认元素中心`50% 50%`，支持方位词、长度单位、百分比
  2. `perspective`：透视属性，设置3D透视距离，数值越小透视效果越强，给父元素设置
  3. `transform-style: preserve-3d`：保留3D空间，给父元素设置，实现子元素的3D效果
  4. `backface-visibility: hidden`：隐藏元素背面，3D旋转时生效

### 7.4 渐变gradient
渐变是CSS3提供的图像类型，可替代背景图片，实现平滑的颜色过渡效果，无需加载图片资源。
1. **线性渐变**：沿直线方向的颜色过渡
   - 语法：`linear-gradient(方向, 颜色1 位置, 颜色2 位置, ...)`
   - 示例：
     ```css
     /* 从左到右，红到蓝渐变 */
     background: linear-gradient(to right, red, blue);
     /* 45度角，多色渐变 */
     background: linear-gradient(45deg, #ff0000 0%, #00ff00 50%, #0000ff 100%);
     ```
2. **径向渐变**：从中心点向外辐射的圆形/椭圆形渐变
   - 语法：`radial-gradient(形状 大小 at 位置, 颜色1 位置, 颜色2 位置, ...)`
   - 示例：
     ```css
     /* 圆形径向渐变，中心红到边缘蓝 */
     background: radial-gradient(circle, red, blue);
     /* 椭圆渐变，指定位置 */
     background: radial-gradient(ellipse at top left, #ff0000, #0000ff);
     ```
3. **锥形渐变**：围绕中心点旋转的锥形渐变，适用于饼图、色轮
   - 语法：`conic-gradient(起始角度 at 位置, 颜色1 角度, 颜色2 角度, ...)`
   - 示例：`background: conic-gradient(red 0deg, yellow 120deg, blue 240deg, red 360deg);`

### 7.5 CSS变量（自定义属性）
CSS变量允许开发者定义可复用的属性值，支持作用域、动态修改，是组件化、主题切换的核心方案。
- 定义语法：`--变量名: 属性值;` 变量名区分大小写，推荐小写+连字符
- 使用语法：`var(--变量名, 备用值)` 备用值为变量不存在时的兜底值
- 作用域：
  1. 全局变量：定义在`:root`伪类中，整个文档都可使用
  2. 局部变量：定义在选择器中，仅当前选择器及其后代元素可使用
- 示例：
  ```css
  /* 定义全局变量 */
  :root {
    --primary-color: #165DFF;
    --font-size-base: 14px;
    --spacing-base: 16px;
  }
  /* 使用变量 */
  .btn {
    background: var(--primary-color);
    font-size: var(--font-size-base);
    padding: calc(var(--spacing-base) / 2) var(--spacing-base);
  }
  /* 局部变量，仅.box内生效 */
  .box {
    --box-bg: #f5f5f5;
    background: var(--box-bg);
  }
  ```
> 优势：可通过JavaScript动态修改CSS变量，实现一键主题切换、动态样式调整，无需修改CSS文件。

---

## 八、CSS优先级与层叠规则
CSS全称层叠样式表，核心就是层叠规则，用于解决多个样式作用于同一个元素时的冲突问题，决定最终生效的样式。

### 8.1 优先级权重计算
样式优先级从高到低排序，权重越高越优先生效：
1. **!important**：最高优先级，强行覆盖所有样式，非必要绝对不滥用（会破坏整个优先级体系，难以维护）
2. **行内样式**：style属性内联样式，权重1000
3. **ID选择器**：权重0100
4. **类选择器、伪类选择器、属性选择器**：权重0010
5. **标签选择器、伪元素选择器**：权重0001
6. **通配符选择器*、继承样式**：权重0000
7. **浏览器默认样式**：最低优先级

> 权重计算规则：
> - 权重按位叠加，不进位，10个类选择器权重也低于1个ID选择器
> - 复合选择器的权重为所有组成选择器的权重之和
> - 并集选择器的权重各自独立计算，互不影响
> - 示例：`#header .nav li` 权重 = 0100 + 0010 + 0001 = 0111

### 8.2 层叠核心规则
1. **权重不同**：权重高的样式覆盖权重低的样式，无论书写顺序
2. **权重相同**：后面书写的样式覆盖前面的样式（就近原则）
3. **继承样式**：继承的样式优先级永远低于直接作用于元素的样式
4. **!important例外**：
   - 只有!important能覆盖!important
   - 行内!important优先级最高
   - 不要用!important覆盖行内样式，会导致样式完全失控

### 8.3 CSS继承性
部分CSS属性会自动从父元素继承给子元素，无需重复设置，减少代码冗余。
- **可继承属性**：字体相关（font-*）、文本相关（color、text-*、line-height、letter-spacing等）、列表相关（list-style-*）、表格相关属性
- **不可继承属性**：布局相关（width、height、margin、padding、border）、背景相关、定位相关、浮动、overflow、transform等
- 强制继承：可通过`inherit`关键字强制子元素继承父元素的属性，示例：`div { box-sizing: inherit; }`

---

## 九、CSS响应式核心：媒体查询@media
媒体查询是响应式布局的核心，可根据设备的屏幕尺寸、分辨率、横竖屏等特性，应用不同的CSS样式，实现一套代码适配多终端。

### 9.1 基础语法
```css
@media 媒体类型 and (媒体特性) and (媒体特性) {
  /* 满足条件时生效的CSS样式 */
  选择器 {
    属性: 值;
  }
}
```

### 9.2 核心组成
1. **媒体类型**：指定样式生效的设备类型
   - `all`：所有设备，默认值
   - `screen`：电脑、手机、平板等屏幕设备，最常用
   - `print`：打印预览/打印设备
2. **媒体特性**：指定样式生效的条件，用括号包裹，多个条件用`and`连接
   | 常用媒体特性 | 作用 |
   | :--- | :--- |
   | `width` / `height` | 视口宽度/高度 |
   | `max-width` | 视口最大宽度，小于等于该值时生效 |
   | `min-width` | 视口最小宽度，大于等于该值时生效 |
   | `max-height` / `min-height` | 视口最大/最小高度 |
   | `orientation` | 屏幕方向，`portrait`竖屏，`landscape`横屏 |
   | `device-pixel-ratio` | 设备像素比，适配高清屏 |

### 9.3 常用示例
1. 移动端适配（屏幕宽度≤768px时生效）
   ```css
   @media screen and (max-width: 768px) {
     body {
       font-size: 14px;
     }
     .container {
       width: 100%;
       padding: 0 10px;
     }
   }
   ```
2. 平板适配（768px≤屏幕宽度≤1200px时生效）
   ```css
   @media screen and (min-width: 768px) and (max-width: 1200px) {
     .container {
       width: 90%;
     }
   }
   ```
3. PC大屏适配（屏幕宽度≥1200px时生效）
   ```css
   @media screen and (min-width: 1200px) {
     .container {
       width: 1200px;
       margin: 0 auto;
     }
   }
   ```

### 9.4 移动端适配必备：viewport设置
媒体查询生效的前提是正确设置viewport元标签，在HTML的`<head>`中添加：
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
```
- 核心属性：`width=device-width` 让视口宽度等于设备屏幕宽度，`initial-scale=1.0` 初始缩放比例1:1

---

## 十、CSS常用@规则
@规则是CSS的特殊语句，用于定义特殊规则、导入资源、条件判断等，以`@`开头。

| @规则 | 语法 | 核心作用 |
| :--- | :--- | :--- |
| `@import` | `@import url("style.css") 媒体查询;` | 导入外部CSS文件，必须写在CSS文件最顶部 |
| `@font-face` | `@font-face { font-family: 自定义名称; src: url(字体路径); }` | 自定义网页字体，实现特殊字体效果 |
| `@keyframes` | 见上文动画部分 | 定义CSS动画关键帧 |
| `@media` | 见上文媒体查询部分 | 条件判断，实现响应式布局 |
| `@supports` | `@supports (属性: 值) { 样式 }` | 特性查询，判断浏览器是否支持某个CSS属性，实现渐进增强 |
| `@layer` | `@layer 层级名 { 样式 }` | 定义级联层，手动控制样式的优先级，解决第三方样式冲突 |
| `@namespace` | `@namespace url(XML命名空间);` | 定义XML命名空间，用于XML/XHTML文档 |

> `@supports`示例：判断浏览器是否支持flex布局，支持则应用flex样式
> ```css
> @supports (display: flex) {
>   .box {
>     display: flex;
>   }
> }
> ```

---

## 十一、CSS书写规范与兼容性
### 11.1 基础书写规范
1. **命名规范**：推荐使用小写字母+连字符`-`（kebab-case），如`header-nav`，避免驼峰、下划线、中文；复杂组件推荐BEM命名规范（块-元素-修饰符）
2. **属性书写顺序**：按功能优先级排序，提升可读性和渲染性能
   - 第一优先级：布局相关（display、position、float、flex、grid等）
   - 第二优先级：盒模型相关（width、height、margin、padding、border等）
   - 第三优先级：文本字体相关（font、color、text-align、line-height等）
   - 第四优先级：视觉装饰相关（background、box-shadow、border-radius、transform等）
   - 第五优先级：动画过渡相关（transition、animation等）
   - 第六优先级：其他（opacity、z-index、overflow等）
3. **代码格式化**：每个声明独占一行，冒号后加空格，分号结尾，选择器与`{`之间加空格，合理缩进

### 11.2 浏览器兼容性处理
1. **浏览器私有前缀**：针对部分CSS3新特性，添加浏览器厂商前缀，兼容旧版本浏览器
   - `-webkit-`：Chrome、Safari、Edge、移动端webkit内核浏览器
   - `-moz-`：Firefox火狐浏览器
   - `-ms-`：IE浏览器、旧版Edge
   - `-o-`：旧版Opera浏览器
   - 示例：
     ```css
     .box {
       -webkit-border-radius: 10px;
       -moz-border-radius: 10px;
       border-radius: 10px;
     }
     ```
2. **渐进增强与优雅降级**
   - 渐进增强：先保证基础功能在低版本浏览器正常使用，再针对高版本浏览器添加新特性，提升体验
   - 优雅降级：先基于高版本浏览器开发完整功能，再针对低版本浏览器做兼容处理，保证核心功能可用
3. **兼容性查询**：通过[caniuse.com](sslocal://flow/file_open?url=https%3A%2F%2Fcaniuse.com&flow_extra=eyJsaW5rX3R5cGUiOiJjb2RlX2ludGVycHJldGVyIn0=)查询CSS属性的浏览器支持情况，针对性做兼容处理
4. **自动化兼容**：企业级开发中，通过PostCSS的autoprefixer插件自动添加浏览器前缀，无需手动书写


