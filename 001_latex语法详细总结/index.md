# LaTeX 语法详细总结


# LaTeX 语法详细总结
LaTeX 是一款基于 TeX 的专业排版系统，核心优势是**内容与格式分离**，尤其擅长数学公式、学术论文、书籍、幻灯片的排版。以下是覆盖入门到进阶的完整语法总结，兼顾通用性与实用性，适配中文排版需求。

---

## 一、基础文档结构
### 1. 最小可编译文档
```latex
% 文档类声明，必选，放在第一行
\documentclass{article}
% 导言区：加载宏包、全局设置，正文前执行
\usepackage{ctex} % 中文支持必备，适配XeLaTeX/LuaLaTeX编译
% 正文区，所有可见内容必须放在这里
\begin{document}
Hello World! 你好，LaTeX！
\end{document}
```

### 2. 常用文档类
| 文档类 | 适用场景 | 核心特点 |
|--------|----------|----------|
| article | 短文、论文、作业、技术报告 | 轻量化结构，无`\chapter`命令 |
| report | 中长篇报告、毕业论文 | 支持`\chapter`章节，适配正式报告结构 |
| book | 书籍、专著 | 完整章节体系，支持扉页、前言、附录、奇偶页差异化排版 |
| ctexart/ctexrep/ctexbook | 中文文档 | 内置ctex宏包，完美适配中文，无需额外配置中文环境 |
| beamer | 幻灯片/演示文稿 | 内置主题模板，支持动画、分帧、演示交互 |

### 3. 文档层级结构
按优先级从高到低排列，不同文档类支持的层级不同：
```latex
\part{部分名}       % 最高层级，全文档类通用
\chapter{章名}      % 仅book/report/ctexbook/ctexrep支持
\section{节名}      % 核心层级，全文档类通用
\subsection{小节名}
\subsubsection{子小节名}
\paragraph{段落名}
\subparagraph{子段落名}
```
- 目录生成：正文开头添加`\tableofcontents`，编译2次即可自动生成目录；
- 取消编号：命令后加`*`，如`\section*{无编号节}`，内容不会进入目录。

---

## 二、文本格式与基础排版
### 1. 换行、分段与分页
| 命令/写法 | 作用 | 注意事项 |
|-----------|------|----------|
| 空行 | 分段 | 生成新段落，默认触发首行缩进，段间距生效 |
| `\par` | 分段 | 等价于空行，多用于宏包/环境内的分段控制 |
| `\\` | 强制换行 | 仅换行不生成新段落，无首行缩进，禁止用于段落间换行 |
| `\\*` | 强制换行且不分页 | 禁止在此处触发分页 |
| `\newpage` | 强制分页 | 立即换页，剩余内容渲染到下一页 |
| `\clearpage` | 强制分页 | 同`\newpage`，同时输出所有未渲染的浮动体（图片/表格） |

### 2. 特殊字符转义
LaTeX中以下字符为系统保留字符，必须转义才能正常输出：
| 原字符 | 转义写法 | 输出效果 | 备注 |
|--------|----------|----------|------|
| # | `\#` | # | 宏包参数标记 |
| $ | `\$` | $ | 数学公式起止标记 |
| % | `\%` | % | 注释标记，行内%后内容全部被忽略 |
| & | `\&` | & | 表格/对齐环境列分隔符 |
| _ | `\_` | _ | 数学下标标记 |
| ^ | `\^{}` | ^ | 数学上标标记 |
| { } | `\{ \}` | { } | 分组/作用域标记 |
| ~ | `\textasciitilde` | ~ | 不可换行空格 |
| \ | `\textbackslash` | \ | 命令起始标记 |

### 3. 字体格式设置
#### （1）核心字体样式
| 命令 | 效果 | 适用场景 |
|------|------|----------|
| `\textbf{内容}` | 加粗 | 重点内容强调 |
| `\textit{内容}` | 斜体 | 术语、书名、变量标注 |
| `\underline{内容}` | 下划线 | 核心内容标注 |
| `\sout{内容}` | 删除线 | 需加载`ulem`宏包 |
| `\texttt{内容}` | 等宽打字机字体 | 代码、命令、终端输出 |
| `\textsf{内容}` | 无衬线字体 | 标题、标注、海报 |
| `\textnormal{内容}` | 恢复默认正文字体 | 嵌套格式中重置样式 |

#### （2）字号大小（相对文档默认字号，默认10pt）
按从小到大排序，用花括号限制作用范围：
```latex
\tiny  \scriptsize  \footnotesize  \small  \normalsize  \large  \Large  \LARGE  \huge  \Huge
```
用法示例：`{\large 大号文本}` 或 `\large 作用域内全部文本`。

#### （3）字体颜色
需加载`xcolor`宏包，基础用法：
```latex
\usepackage{xcolor} % 导言区加载
\textcolor{red}{红色文本} % 单行文本上色
\color{blue} 整个段落/环境内文本变为蓝色
```
支持预定义颜色：black、white、red、green、blue、yellow、gray、cyan、magenta等，也支持自定义RGB颜色。

### 4. 段落对齐与缩进
#### （1）对齐环境
```latex
% 左对齐
\begin{flushleft}
左对齐文本，默认段落为左对齐，多用于局部重置
\end{flushleft}

% 居中对齐
\begin{center}
居中对齐文本\\
第二行居中
\end{center}

% 右对齐
\begin{flushright}
右对齐文本
\end{flushright}
```
- 简化命令：`\centering`（居中）、`\raggedright`（左对齐）、`\raggedleft`（右对齐），作用于当前分组/环境，无额外垂直间距，多用于浮动体内部。

#### （2）段落格式全局设置
```latex
\parindent=2em % 首行缩进2个字符，设为0pt可取消首行缩进
\parskip=1ex % 段间距，设置段落之间的垂直间距
\linespread{1.5} % 全局行间距1.5倍，导言区设置生效
```

---

## 三、列表环境
LaTeX提供3种核心列表环境，支持无限嵌套，默认4层以内样式自动区分。

### 1. 无序列表（itemize）
```latex
\begin{itemize}
    \item 一级列表项1
    \item 一级列表项2
    \begin{itemize}
        \item 二级列表项1
        \item 二级列表项2
    \end{itemize}
    \item 一级列表项3
\end{itemize}
```
- 自定义符号：`\item[★] 自定义符号的列表项`

### 2. 有序列表（enumerate）
```latex
\begin{enumerate}
    \item 一级有序项1
    \item 一级有序项2
    \begin{enumerate}
        \item 二级有序项1
        \item 二级有序项2
    \end{enumerate}
    \item 一级有序项3
\end{enumerate}
```
- 自定义编号：`\item[(1)] 自定义编号项`，推荐用`enumitem`宏包实现全局样式修改。

### 3. 描述列表（description）
用于术语解释、定义说明，自带加粗标签：
```latex
\begin{description}
    \item[LaTeX] 专业排版系统，核心优势为学术文档与数学公式排版
    \item[TeX] LaTeX的底层排版引擎，由高德纳开发
\end{description}
```

### 4. 高级自定义（enumitem宏包）
导言区加载`\usepackage{enumitem}`，可全局/局部设置列表的缩进、标签、间距、编号样式：
```latex
% 有序列表全局设置为(1)(2)样式，缩进2em
\setlist[enumerate]{label=(\arabic*), leftmargin=2em}
% 无序列表全局设置为●符号
\setlist[itemize]{label=$\bullet$}
```

---

## 四、数学公式（核心功能）
LaTeX的核心优势是数学公式排版，**必须加载`amsmath`宏包**（标准数学宏包），进阶符号需加载`amssymb`宏包。

### 1. 公式环境分类
#### （1）行内公式（嵌入文本中）
最常用写法：`$ 公式内容 $`，等价于`\( 公式内容 \)`，示例：
```latex
勾股定理可表示为 $a^2 + b^2 = c^2$，其中$a,b$为直角边，$c$为斜边。
```

#### （2）行间公式（单独成行）
- 无编号公式（推荐）：`\[ 公式内容 \]`，**禁止用`$$ 公式 $$`**（存在间距兼容问题）
```latex
勾股定理的标准形式为：
\[ a^2 + b^2 = c^2 \]
```
- 有编号公式（可交叉引用）：`equation`环境，自动编号
```latex
\begin{equation}\label{eq:pythagorean}
a^2 + b^2 = c^2
\end{equation}
```
引用方式：`式\ref{eq:pythagorean}为勾股定理`，编译2次可正常显示编号。
- 无编号equation环境：`equation*`，需amsmath宏包支持。

### 2. 常用数学符号与结构
#### （1）上下标
- 上标：`^`，下标：`_`，多个字符需用`{}`包裹
- 示例：`$x_{i}^2$` → $x_i^2$；`$a_{n-1}^{k+2}$` → $a_{n-1}^{k+2}$

#### （2）分式与根式
- 分式：`\frac{分子}{分母}`，行内分式可用`\dfrac{分子}{分母}`强制显示为行间大小
- 根式：`\sqrt[开方次数]{被开方数}`，默认平方根，无需写次数
- 示例：`$\sqrt{2} + \sqrt[3]{x+y} = \frac{1}{2}$` → $\sqrt{2} + \sqrt[3]{x+y} = \frac{1}{2}$

#### （3）大型运算符
求和、积分、极限等，行间公式自动显示上下限，行内公式默认压缩到上下标位置，可用`\limits`强制显示上下限。
| 运算符 | 命令 | 示例 | 输出效果 |
|--------|------|------|----------|
| 求和 | `\sum` | `$\sum_{i=1}^n i$` | $\sum_{i=1}^n i$ |
| 积分 | `\int` | `$\int_{a}^{b} f(x)dx$` | $\int_a^b f(x)dx$ |
| 乘积 | `\prod` | `$\prod_{k=1}^n k$` | $\prod_{k=1}^n k$ |
| 极限 | `\lim` | `$\lim_{x \to 0} \frac{\sin x}{x}=1$` | $\lim_{x \to 0} \frac{\sin x}{x}=1$ |
| 最大值 | `\max` | `$\max_{x \in [0,1]} f(x)$` | $\max_{x \in [0,1]} f(x)$ |

#### （4）希腊字母
小写字母命令为小写开头，大写字母命令为大写开头（仅部分大写希腊字母有单独命令），常用示例：
| 小写 | 命令 | 大写 | 命令 |
|------|------|------|------|
| α | `\alpha` | Γ | `\Gamma` |
| β | `\beta` | Δ | `\Delta` |
| γ | `\gamma` | Θ | `\Theta` |
| δ | `\delta` | Λ | `\Lambda` |
| ε | `\epsilon` | Σ | `\Sigma` |
| θ | `\theta` | Ω | `\Omega` |
| λ | `\lambda` | Π | `\Pi` |
| μ | `\mu` | Φ | `\Phi` |
| π | `\pi` | Ψ | `\Psi` |
| σ | `\sigma` |  |  |
| φ | `\phi` |  |  |
| ω | `\omega` |  |  |

#### （5）括号与定界符
- 基础括号：`()`、`[]`、`\{ \}`、`| |`、`\| \|`
- 自动适配大小：`\left` + 左括号 + 内容 + `\right` + 右括号，自动匹配内容高度，必须成对出现，无对应括号用`.`代替（如`\right.`）
- 示例：
```latex
\[ \left( \frac{1}{1+x^2} \right)^2 \]
\[ \left| \int_{0}^{1} f(x) dx \right| \]
```
- 手动指定大小：`\big`、`\Big`、`\bigg`、`\Bigg`，从小到大，示例：`\bigg( \Big( \big( () \big) \Big) \bigg)`

#### （6）数学字体
需加载`amssymb`宏包支持部分字体，常用：
| 字体命令 | 效果 | 适用场景 |
|----------|------|----------|
| `\mathbf{}` | 数学黑体 | 向量、矩阵 |
| `\mathbb{}` | 黑板粗体 | 数集：$\mathbb{R}$(实数)、$\mathbb{Z}$(整数)、$\mathbb{N}$(自然数) |
| `\mathcal{}` | 花体 | 集合、算子 |
| `\mathit{}` | 数学斜体 | 多字符变量名 |
| `\mathrm{}` | 数学正体 | 算子、单位：$\mathrm{sin}x$、$\mathrm{m}$ |

### 3. 多行公式环境（amsmath）
#### （1）对齐公式（align/align*）
最常用的多行公式环境，用`&`指定对齐点，`\\`换行，align自动编号，align*无编号：
```latex
\begin{align*}
f(x) &= (x+1)(x-1) \\
     &= x^2 - 1
\end{align*}
```
多列对齐：用`&`分隔列，示例：
```latex
\begin{align}
x + y &= 3  &  x - y &= 1 \\
2x + y &= 5 &  2x - y &= 3
\end{align}
```

#### （2）无对齐多行公式（gather/gather*）
每行居中，无需对齐点，适合无关联的多行公式：
```latex
\begin{gather*}
a^2 + b^2 = c^2 \\
\sin^2 x + \cos^2 x = 1
\end{gather*}
```

#### （3）分段函数（cases）
用于分段函数、条件表达式，自动生成大括号：
```latex
\[
f(x) =
\begin{cases}
x, & x \geq 0 \\
-x, & x < 0
\end{cases}
\]
```

#### （4）长公式换行（multline/multline*）
首行左对齐，末行右对齐，中间行居中，适合超长公式：
```latex
\begin{multline*}
(a+b+c+d+e+f+g+h+i+j+k+l+m) \\
\times (n+o+p+q+r+s+t+u+v+w+x+y+z) \\
= \text{超长公式的换行示例}
\end{multline*}
```

### 4. 矩阵环境（amsmath）
所有矩阵环境都必须放在数学公式内，用`&`分隔列，`\\`换行，核心环境：
| 环境名 | 括号类型 | 适用场景 |
|--------|----------|----------|
| matrix | 无括号 | 基础矩阵 |
| pmatrix | 圆括号() | 最常用，矩阵标准写法 |
| bmatrix | 方括号[] | 通用矩阵写法 |
| Bmatrix | 大括号{} | 集合类矩阵 |
| vmatrix | 单竖线| | 行列式 |
| Vmatrix | 双竖线|| | 范数 |

示例：
```latex
\[
A = \begin{pmatrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{pmatrix}, \quad
|B| = \begin{vmatrix}
a & b \\
c & d
\end{vmatrix} = ad - bc
\]
```

---

## 五、表格排版
### 1. 基础表格（tabular环境）
核心语法：`\begin{tabular}{列格式} 内容 \end{tabular}`
- 列格式说明符：`l`(左对齐)、`c`(居中)、`r`(右对齐)、`|`(竖线)、`p{宽度}`(固定宽度列，自动换行)
- 列分隔符：`&`，行结束符：`\\`
- 横线：`\hline`(全宽横线)，`\cline{起始列-结束列}`(局部横线)

基础示例：
```latex
\begin{center}
\begin{tabular}{|c|c|c|}
\hline
姓名 & 年龄 & 性别 \\
\hline
张三 & 20 & 男 \\
\hline
李四 & 21 & 女 \\
\hline
\end{tabular}
\end{center}
```

### 2. 学术规范三线表（booktabs宏包）
学术论文通用标准，无竖线，横线粗细区分，需加载`booktabs`宏包，核心命令：
- `\toprule`：顶部粗线
- `\midrule`：中间细线
- `\bottomrule`：底部粗线

示例：
```latex
\begin{table}[htbp]
  \centering
  \caption{三线表示例}
  \label{tab:demo}
  \begin{tabular}{lccr}
    \toprule
    姓名 & 语文 & 数学 & 总分 \\
    \midrule
    张三 & 90 & 95 & 185 \\
    李四 & 88 & 98 & 186 \\
    王五 & 92 & 93 & 185 \\
    \bottomrule
  \end{tabular}
\end{table}
```

### 3. 单元格合并
- 列合并：`\multicolumn{合并列数}{列格式}{内容}`，全表格通用
- 行合并：`\multirow{合并行数}{宽度}{内容}`，需加载`multirow`宏包

合并示例：
```latex
\begin{tabular}{|c|c|c|}
\hline
\multicolumn{2}{|c|}{合并两列} & 单列 \\
\hline
\multirow{2}{*}{合并两行} & 单元格1 & 单元格2 \\
\cline{2-3}
& 单元格3 & 单元格4 \\
\hline
\end{tabular}
```

### 4. 表格浮动体与跨页表格
- 浮动体：`table`环境，用于控制表格位置，添加标题、标签，位置参数`[htbp]`：
  - `h`(here)：当前位置，`t`(top)：页面顶部，`b`(bottom)：页面底部，`p`(page)：单独浮动页
- 跨页长表格：`longtable`环境，需加载`longtable`宏包，无需放在table环境内，支持自动分页，可设置表头、表尾重复。

---

## 六、图片插入
需加载`graphicx`宏包（标准插图宏包，必加），支持png、jpg、pdf、eps等格式，推荐用XeLaTeX/pdfLaTeX编译。

### 1. 基础插图命令
```latex
\includegraphics[可选参数]{图片文件路径/文件名}
```
常用可选参数：
| 参数 | 作用 | 示例 |
|------|------|------|
| width= | 固定宽度 | `width=8cm`、`width=\linewidth`(占满当前行宽) |
| height= | 固定高度 | `height=5cm` |
| scale= | 按比例缩放 | `scale=0.8`(缩小为原尺寸80%) |
| angle= | 旋转角度 | `angle=90`(顺时针旋转90度) |
| keepaspectratio | 保持宽高比 | 配合width/height使用，防止图片变形 |

### 2. 图片浮动体
`figure`环境，控制图片位置，添加标题、标签，和table环境用法一致：
```latex
\begin{figure}[htbp]
  \centering % 图片居中
  \includegraphics[width=0.8\textwidth]{demo.jpg}
  \caption{图片标题}
  \label{fig:demo}
\end{figure}
```
- 引用方式：`图\ref{fig:demo}为示例图片`

### 3. 子图与多图并排
推荐用`subcaption`宏包实现子图排版，导言区加载`\usepackage{subcaption}`。

#### （1）子图排版（带独立子标题）
```latex
\begin{figure}[htbp]
  \centering
  \begin{subfigure}{0.45\textwidth}
    \centering
    \includegraphics[width=\linewidth]{fig1.jpg}
    \caption{子图1标题}
    \label{fig:sub1}
  \end{subfigure}
  \hfill % 两个子图之间填充空白
  \begin{subfigure}{0.45\textwidth}
    \centering
    \includegraphics[width=\linewidth]{fig2.jpg}
    \caption{子图2标题}
    \label{fig:sub2}
  \end{subfigure}
  \caption{总标题}
  \label{fig:total}
\end{figure}
```

#### （2）简单并排图片（无独立子标题）
用`minipage`环境实现：
```latex
\begin{figure}[htbp]
  \centering
  \begin{minipage}{0.45\textwidth}
    \centering
    \includegraphics[width=\linewidth]{fig1.jpg}
  \end{minipage}
  \hfill
  \begin{minipage}{0.45\textwidth}
    \centering
    \includegraphics[width=\linewidth]{fig2.jpg}
  \end{minipage}
  \caption{两张图片并排}
\end{figure}
```

---

## 七、交叉引用与超链接
### 1. 基础交叉引用
核心逻辑：`\label{标签名}` 标记对象 → `\ref{标签名}` 引用编号 → `\pageref{标签名}` 引用页码
- 标签命名规范（避免冲突）：
  - 章节：`sec:xxx`，公式：`eq:xxx`，图片：`fig:xxx`，表格：`tab:xxx`，定理：`thm:xxx`
- 注意：必须编译2次以上，引用编号才能正常显示，单次编译会显示`??`。

示例：
```latex
\section{引言}\label{sec:intro}
本文在\ref{sec:method}章节介绍方法，核心公式见式\ref{eq:pythagorean}，实验数据见表\ref{tab:demo}，图片见图\ref{fig:demo}。

\section{方法}\label{sec:method}
方法内容...
```

### 2. 智能引用（cleveref宏包）
导言区加载`\usepackage{cleveref}`，**必须放在hyperref之后**，可自动识别引用类型，添加前缀，无需手动写“图/表/式”：
```latex
\cref{fig:demo} % 自动输出“图1”
\cref{tab:demo,eq:pythagorean} % 自动输出“表1和式(1)”
```

### 3. 超链接（hyperref宏包）
导言区加载`\usepackage{hyperref}`，**必须放在所有宏包的最后**（极少数宏包除外，如cleveref），自动为目录、引用、参考文献生成超链接，支持自定义样式。

常用命令：
```latex
\url{https://www.latex-project.org} % 直接显示网址，自动生成超链接
\href{https://www.latex-project.org}{LaTeX官方网站} % 自定义显示文本的超链接
\hypertarget{锚点名}{锚点文本} % 文档内锚点
\hyperlink{锚点名}{跳转文本} % 跳转到文档内锚点
```
- 常用选项：`\usepackage[colorlinks,linkcolor=blue,citecolor=blue,urlcolor=red]{hyperref}`，设置彩色链接，取消默认边框。

---

## 八、参考文献排版
LaTeX提供3种主流参考文献排版方式，适配不同场景。

### 1. 手动排版（thebibliography环境）
适合10条以内的少量参考文献，无需额外编译，示例：
```latex
% 正文末尾，\end{document}之前
\begin{thebibliography}{99} % 99为最大编号位数，两位数写99
\bibitem{latex_book}
  刘海洋. LaTeX入门[M]. 电子工业出版社, 2013.
\bibitem{lamport1994}
  Lamport L. LaTeX: A Document Preparation System[M]. Addison-Wesley, 1994.
\end{thebibliography}
```
- 引用方式：正文内`\cite{latex_book}`，输出引用编号。

### 2. BibTeX方式（学术投稿通用）
适合中大量参考文献，分离文献数据与排版格式，期刊投稿主流方案，步骤：
1.  创建`.bib`文献库文件（如`ref.bib`），存放参考文献条目，示例：
```bibtex
% 期刊文章
@article{zhang2020,
  author  = {张三 and 李四},
  title   = {论文标题},
  journal = {期刊名},
  year    = {2020},
  volume  = {10},
  number  = {2},
  pages   = {123-145}
}
% 书籍
@book{liu2013,
  author    = {刘海洋},
  title     = {LaTeX入门},
  publisher = {电子工业出版社},
  year      = {2013},
  address   = {北京}
}
% 会议论文
@inproceedings{wang2022,
  author    = {王五 and 赵六},
  title     = {会议论文标题},
  booktitle = {会议名称},
  year      = {2022},
  pages     = {456-460},
  address   = {上海}
}
```
2.  导言区设置样式：`\bibliographystyle{样式名}`
    - 常用样式：`plain`(标准排序)、`ieeetr`(IEEE期刊格式)、`apalike`(APA格式)、`unsrt`(按引用顺序排序)
3.  正文末尾添加文献库引用：`\bibliography{ref}`（ref为.bib文件名，无需后缀）
4.  正文内引用：`\cite{zhang2020}`
5.  编译流程：pdfLaTeX → BibTeX → pdfLaTeX × 2

### 3. BibLaTeX+Biber方式（现代推荐）
功能更强大，格式自定义更简单，适配Unicode，是现代LaTeX推荐方案，步骤：
1.  导言区加载宏包并设置：
```latex
\usepackage[backend=biber,style=ieee,citestyle=ieee]{biblatex}
\addbibresource{ref.bib} % 加载.bib文件，必须写后缀.bib
```
2.  正文内引用：`\cite{zhang2020}`
3.  正文末尾输出参考文献列表：`\printbibliography[title=参考文献]`
4.  编译流程：XeLaTeX/pdfLaTeX → Biber → XeLaTeX/pdfLaTeX × 2

---

## 九、中文排版支持
中文排版**强烈推荐使用ctex宏包/文档类**，完美适配中文标点、换行、字体、章节格式，无需额外配置，兼容XeLaTeX和LuaLaTeX编译（优先推荐XeLaTeX）。

### 1. 最简中文文档
```latex
\documentclass{ctexart} % 中文article文档类，内置ctex宏包
\begin{document}
\section{中文标题}
你好，LaTeX！这是中文排版示例，支持自动换行、标点压缩、首行缩进。
\end{document}
```

### 2. 常用中文设置
```latex
% 导言区设置
\ctexset{
  section = {
    name = {第,节}, % 章节名前缀，如“第1节”
    number = \chinese{section}, % 章节编号用中文数字
  },
  chapter = {
    name = {第,章},
    number = \chinese{chapter}
  }
}
```

---

## 十、高级常用功能
### 1. 定理环境（amsthm宏包）
用于学术论文的定理、引理、定义、证明等环境，导言区加载`\usepackage{amsthm}`。

#### （1）定义定理环境
```latex
% 语法：\newtheorem{环境名}{显示名称}[编号层级]
\newtheorem{theorem}{定理}[section] % 按section编号，如“定理1.1”
\newtheorem{lemma}[theorem]{引理} % 和theorem共用编号
\newtheorem{definition}{定义}[section]
\newtheorem{corollary}{推论}[theorem]
```

#### （2）使用示例
```latex
\begin{theorem}[勾股定理]\label{thm:pyth}
直角三角形两直角边的平方和等于斜边的平方，即：
\[ a^2 + b^2 = c^2 \]
\end{theorem}

\begin{proof}
证明过程...
\end{proof}
```
- `proof`环境自动生成“证明”标题，末尾自动添加证毕符号□。

### 2. 页面布局设置（geometry宏包）
导言区加载`\usepackage{geometry}`，快速设置纸张大小、页边距等，替代原生复杂命令：
```latex
\usepackage{geometry}
\geometry{
  a4paper,
  left=2.5cm,
  right=2.5cm,
  top=2.5cm,
  bottom=2.5cm,
  headheight=15pt, % 页眉高度
  footskip=10pt % 页脚距离
}
```

### 3. 页眉页脚自定义（fancyhdr宏包）
导言区加载`\usepackage{fancyhdr}`，自定义页眉页脚内容：
```latex
\pagestyle{fancy} % 启用fancy样式
% 清除默认设置
\fancyhf{}
% 页眉设置
\lhead{左侧页眉} % 左页眉
\chead{居中页眉} % 中页眉
\rhead{\thepage} % 右页眉，显示页码
% 页脚设置
\lfoot{左侧页脚}
\cfoot{}
\rfoot{右侧页脚}
% 章节首页页眉页脚设置
\fancypagestyle{plain}{
  \fancyhf{}
  \rfoot{\thepage}
  \renewcommand{\headrulewidth}{0pt} % 取消页眉横线
}
```

### 4. 代码高亮排版
#### （1）listings宏包（无额外依赖，兼容性好）
导言区加载`\usepackage{listings,xcolor}`，示例：
```latex
\lstset{
  language=C++, % 代码语言
  numbers=left, % 行号在左侧
  numberstyle=\tiny\color{gray}, % 行号样式
  basicstyle=\ttfamily\footnotesize, % 代码基础样式
  keywordstyle=\color{blue}\bfseries, % 关键字样式
  commentstyle=\color{green!60!black}, % 注释样式
  stringstyle=\color{red}, % 字符串样式
  frame=single, % 边框
  breaklines=true, % 自动换行
  tabsize=4 % tab宽度
}

% 插入代码
\begin{lstlisting}
#include <iostream>
using namespace std;
int main() {
  cout << "Hello LaTeX!" << endl;
  return 0;
}
\end{lstlisting}
```

#### （2）minted宏包（高亮效果更优）
需安装Python和pygments库，编译时加`-shell-escape`参数，高亮效果接近IDE。

### 5. 分栏排版（multicol宏包）
导言区加载`\usepackage{multicol}`，实现灵活的多栏排版，支持跨栏标题、自动平衡栏高：
```latex
\begin{multicols}{2} % 双栏排版，数字为栏数
这里是双栏排版的内容，会自动分栏，支持自动换行、分页。
% 单栏内容
\onecolumn
单栏内容，占满整行
\twocolumn
恢复双栏排版
\end{multicols}
```

---

## 十一、编译方式与常见问题
### 1. 主流编译方式选择
| 编译方式 | 适用场景 | 核心优势 |
|----------|----------|----------|
| pdfLaTeX | 英文文档，无生僻字符 | 编译速度快，兼容性好，直接生成PDF |
| XeLaTeX | 中文文档，Unicode字符 | 完美支持中文、各类字体，无需额外配置 |
| LuaLaTeX | 高级排版，复杂宏包 | 支持Lua脚本扩展，Unicode兼容性强 |
| Biber | 配合biblatex使用 | 替代BibTeX，支持Unicode，格式自定义灵活 |

### 2. 常见问题解决
1.  **中文乱码/不显示**：使用ctex文档类，用XeLaTeX编译，不要用pdfLaTeX。
2.  **引用显示??**：至少编译2次，确保\label和\ref对应，无拼写错误。
3.  **图片/表格位置不对**：调整浮动体位置参数`[htbp]`，优先用`[!htbp]`强制优先当前位置。
4.  **宏包冲突**：精简宏包，hyperref宏包必须放在导言区最后加载。
5.  **公式报错**：确保数学公式放在$...$或\[...\]内，特殊字符转义，括号成对，amsmath宏包已加载。
6.  **参考文献不显示**：按正确编译流程执行，确保.bib文件路径正确，引用键无拼写错误。

---

## 十二、常用宏包速查表
| 分类 | 宏包名 | 核心功能 |
|------|--------|----------|
| 中文支持 | ctex | 中文排版全支持，适配字体、标点、章节格式 |
| 数学公式 | amsmath | 核心数学环境、公式排版，必加 |
| 数学符号 | amssymb | 扩展数学符号、黑板粗体等 |
| 定理环境 | amsthm | 定理、引理、证明环境定义 |
| 表格 | booktabs | 三线表排版，学术规范 |
| 表格 | multirow | 单元格行合并 |
| 表格 | longtable | 跨页长表格 |
| 图片 | graphicx | 图片插入核心宏包，必加 |
| 图片 | subcaption | 子图排版，子标题设置 |
| 文本格式 | xcolor | 字体、背景颜色设置 |
| 文本格式 | ulem | 下划线、删除线等文本装饰 |
| 页面布局 | geometry | 页边距、纸张大小快速设置 |
| 页眉页脚 | fancyhdr | 自定义页眉页脚 |
| 列表 | enumitem | 列表样式、缩进、间距自定义 |
| 引用 | hyperref | 超链接、书签生成，必加 |
| 引用 | cleveref | 智能交叉引用，自动识别类型 |
| 参考文献 | biblatex | 现代参考文献排版，配合biber使用 |
| 代码高亮 | listings | 代码插入与高亮，兼容性好 |
| 分栏 | multicol | 灵活多栏排版 |
| 幻灯片 | beamer | LaTeX幻灯片制作 |


