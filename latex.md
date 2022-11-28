## What is Latex
不同于像 Word 这样的所见即所得的文本编辑器，latex 文档是一个包括诸多 latex 命令的纯文本，之后送进 *TeX engine* 输出 PDF 文档。

## 下载配置 latex
关于这部分我目前还有很多疑问，尤其是关于不同 tools 生成的文件类型和在 VS Code 中的配置内容，每次编译 latex 后会在目录下生成一堆杂七杂八的东西还不甚了解，打算之后慢慢研究，这里附上我采用的一篇在 Mac 上安装 Latex 以及配置 VS Code 的[文章](https://zhuanlan.zhihu.com/p/165411114)

## 从文档来学 latex
```latex
\documentclass{article}
\usepackage[UTF8, scheme = plain]{ctex}
\begin{document}
测试中文sdfasd
\end{document}
```
第一行 `\documentclass{article}` 声明了文档类型（class），不同的文档类型（简历、书、报告）可能会用到不同的 class（如 report/book 等）。  
文章的正文在 `\begin{document}` 和 `\end{document}` 之间，

## Preamble
在文章正文之前的内容都叫做 preamble，用来作为文档的 “setup”。  
在 preamble 中你可以：
* 定义文档的 class 以及编写文档时要使用的语言等细节；
* 加载您想要使用的包；
* 应用其他类型配置。

一个最简单的 preamble 如图所示
```latex
\documentclass[12pt, letterpaper]{article}
\usepackage{graphicx}
```
方括号中的是对于这个 article 实例的参数，在这个例子中
* 12pt 设置了字体大小（默认10pt）
* letterpaper 设置纸张大小（[其他选项](https://www.overleaf.com/learn/latex/Page_size_and_margins) 如 a4paper/legalpaper）

```latex
\usepackage{graphicx}
```
是加载一个外部包(这里是 graphicx )以扩展 LATEX 的功能，使其能够导入外部图形文件的示例。更多关于包的讨论见 [这里](https://www.overleaf.com/learn/latex/Learn_LaTeX_in_30_minutes#Finding_and_using_LaTeX_packages)。

## 标题、作者和日期信息
在 preamble 中多加三行
```latex
\title{My first LaTeX document}
\author{Hubert Farnsworth\thanks{Funded by the Overleaf team.}}
\date{August 2022}
```
之后为了排版标题内容，只需要在 **正文** 中加入
```latex
\maketitle
```
## 注释
Latex 作为一种负责排版的编程语言，也有自己的注释，在行前加入 `%` 便可以注释整行代码。

## 粗体 斜体 下划线
* 粗体： `\textbf{...}` 
* 泄题: `\textit{...}`
* 下划线：`\underline{...}`

## 加入图片
如下命令可在文档中加入图片。
```latex
\documentclass{article}
\usepackage{graphicx} % 导入图片的库
\graphicspath{{images/}} %configuring the graphicx package
 
\begin{document}
The universe is immense and it seems to be homogeneous, 
on a large scale, everywhere we look.

% The \includegraphcs command is 
% provided (implemented) by the 
% graphicx package
\includegraphics[width=0.75\textwidth]{universe}

 
There's a picture of a galaxy above.
\end{document}
```

## 图例、标签和参考
```latex
\begin{document}
\maketitle
测试中文sdfasdasdf

% 这里的图片和下面的列表都需要在一个 \begin 环境中
\begin{figure}[h]
    \centering
    \includegraphics[width=0.75\textwidth]{test}
    \caption{A nice plot.} % 这里是图名
    \label{fig:anyName} % 这里是标签内容，会自动分配图号以及引用的图号
\end{figure}

As you can see in figure \ref{fig:anyName}, the function grows near the origin. This example is on page \pageref{fig:anyName}.
\end{document}
```
## 列表
```latex
% 无序列表
\begin{itemize}
    \item The individual entries are indicated with a black dot, a so-called bullet.
    \item The text in the entries may be of any length.
\end{itemize}

% 有序列表
\begin{enumerate}
    \item This is the first entry in our list.
    \item The list numbers increase with each entry we add.
\end{enumerate}
```

## 加入公式
### 行内公式
`\( ... \)`、 `$ ... $` 或 `\begin{math} ... \end{math}` 都可以表示行内公式。
### 展示模式
`\[ ... \]`、 `\begin{displaymath} ... \end{displaymath}` 或 ``\begin{equation} ... \end{equation}`都可以使用展示模式展示公式。之前，`$$ ... display math here ...$$` 也可以展示公式，但已不再被推荐。
```latex
\documentclass[12pt, letterpaper]{article}
\begin{document}
The mass-energy equivalence is described by the famous equation
\[ E=mc^2 \] discovered in 1905 by Albert Einstein. 

In natural units ($c = 1$), the formula expresses the identity
\begin{equation}
E=m
\end{equation}
\end{document}
```



## 基本文档结构
### Abstracts
学术论文通常都要写摘要：
```latex
\documentclass{article}
\begin{document}
\begin{abstract} % 摘要
This is a simple paragraph at the beginning of the 
document. A brief introduction about the main subject.
\end{abstract}
\end{document}
```

### 段落和换行
* 两次 `回车` 结束此段落并开始下一段落。
* 使用 `\\` 或 `\newline` 手动加入断行但不开始下一个段落（定格开始下一行）。

```latex
\documentclass{article}
\begin{document}

\begin{abstract}
This is a simple paragraph at the beginning of the 
document. A brief introduction about the main subject.
\end{abstract}

After our abstract we can begin the first paragraph, then press ``enter'' twice to start the second one.

This line will start a second paragraph.

I will start the third paragraph and then add \\ a manual line break which causes this text to start on a new line but remains part of the same paragraph. Alternatively, I can use the \verb|\newline|\newline command to start a new line, which is also part of the same paragraph.
\end{document}
```

## Chapters and sections
LaTeX 还提供文档结构命令，但是可用的命令及其实现（它们做什么）可以依赖于所使用的文档类。举例来说，使用 book 类创建的文档可以分为部分、章节、节、子节等等，但是 letter 类不提供任何命令来实现这一点。
```latex
\documentclass{book}
\begin{document}

\chapter{First Chapter}

\section{Introduction}

This is the first section.

Lorem  ipsum  dolor  sit  amet,  consectetuer  adipiscing  
elit. Etiam  lobortisfacilisis sem.  Nullam nec mi et 
neque pharetra sollicitudin.  Praesent imperdietmi nec ante. 
Donec ullamcorper, felis non sodales...

\section{Second Section}

Lorem ipsum dolor sit amet, consectetuer adipiscing elit.  
Etiam lobortis facilisissem.  Nullam nec mi et neque pharetra 
sollicitudin.  Praesent imperdiet mi necante...

\subsection{First Subsection}
Praesent imperdietmi nec ante. Donec ullamcorper, felis non sodales...

\section*{Unnumbered Section}
Lorem ipsum dolor sit amet, consectetuer adipiscing elit.  
Etiam lobortis facilisissem...
\end{document}
```
常用命令：
* \part{part}
* \chapter{chapter}
* \section{section}
* \subsection{subsection}
* \subsubsection{subsubsection}
* \paragraph{paragraph}
* \subparagraph{subparagraph}

每个标签开头的数字标号都是自动的，可以使用 `*` 如 `\section*{}` 来禁用自动编号。  
> 注意，`\part` 和 `\chapter` 命令只在 report 和 book 类中可用。

## 表格
### 一个基本表格
```latex
\begin{center}
\begin{tabular}{c c c} % c 表示居中 l/r 表示居左或居右，三个 c 表示每行三列
 cell1 & cell2 & cell3 \\ % & 表示一个的结束， \\ 换行
 cell4 & cell5 & cell6 \\  
 cell7 & cell8 & cell9    
\end{tabular}
\end{center}
```

### 边框
在 tabular 参数中加入 `|` 表示竖直线，`\hline` 表示水平线，两个 `hline` 表示双线/
```latex
\begin{center}
\begin{tabular}{|c|c|c|} 
 \hline
 cell1 & cell2 & cell3 \\ 
 \hline\hline
 cell4 & cell5 & cell6 \\ 
 cell7 & cell8 & cell9 \\ 
 \hline
\end{tabular}
\end{center}
```

### 表名
和 `\begin{figure}` 类似，只不过换用 `\begin{table}`。
```latex
Table \ref{table:data} shows how to add a table caption and reference a table.
\begin{table}[h!]
\centering
\begin{tabular}{||c c c c||} 
 \hline
 Col1 & Col2 & Col2 & Col3 \\ [0.5ex] 
 \hline\hline
 1 & 6 & 87837 & 787 \\ 
 2 & 7 & 78 & 5415 \\
 3 & 545 & 778 & 7507 \\
 4 & 545 & 18744 & 7560 \\
 5 & 88 & 788 & 6344 \\ [1ex] 
 \hline
\end{tabular}
\caption{Table to test captions and labels.}
\label{table:data}
\end{table}
```