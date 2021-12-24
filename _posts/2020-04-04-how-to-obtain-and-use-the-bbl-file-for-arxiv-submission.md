---
layout: post
permalink: /how-to-obtain-and-use-the-bbl-file-for-arxiv-submission/
title: 如何上传论文到 arXiv
date: 2020-04-04T22:41:00
lang: zh-CN
---

上传你的论文到 [arXiv](https://arxiv.org/)，看似麻烦，实则简单。

### 步骤

新建文件夹，将  `mwe.tex` 和 `mwe.bib` 放进去。

文件 `mwe.bib`：

```tex
@Book{Goossens,
  author    = {Goossens, Michel and Mittelbach, Frank and 
               Samarin, Alexander},
  title     = {The LaTeX Companion},
  edition   = {1},
  publisher = {Addison-Wesley},
  location  = {Reading, Mass.},
  year      = {1994},
}
@Book{adams,
  title     = {The Restaurant at the End of the Universe},
  author    = {Douglas Adams},
  series    = {The Hitchhiker's Guide to the Galaxy},
  publisher = {Pan Macmillan},
  year      = {1980},
}
```

文件 `mwe.tex`：

```tex
\documentclass[10pt,a4paper]{article}

\usepackage{hyperref} % for better urls


\begin{document}

This is text with \cite{Goossens} and \cite{adams}.

\nocite{*} % to test all bib entrys
\bibliographystyle{unsrt}
\bibliography{mwe} % file mwe.bib

\end{document}
```

首先运行 `pdflatex mwe.tex`，会生成一系列新文件，例如  `mwe.auc`。

然后运行`bibtex mwe`。 BiBTeX 会编译生成新文件： `mwe.bbl` 和 `mwe.blg`. `mwe.blg` 是 BiBTeX 运行产生的日志文件， `mwe.bbl` 是提交需要的文件。

文件 `mwe.bbl` 内容：

```tex
\begin{thebibliography}{1}

\bibitem{Goossens}
Michel Goossens, Frank Mittelbach, and Alexander Samarin.
\newblock {\em The LaTeX Companion}.
\newblock Addison-Wesley, 1 edition, 1994.

\bibitem{adams}
Douglas Adams.
\newblock {\em The Restaurant at the End of the Universe}.
\newblock The Hitchhiker's Guide to the Galaxy. Pan Macmillan, 1980.

\end{thebibliography}
```

**第二次**运行 `pdflatex mwe.tex` 得到正确的页码。

复制 `mwe.tex` 到 `mwe-arxiv.tex` 并删除对 `bibtex` 的引用，复制 `mwe.bbl` 文件内容或引用 `mwe.bbl` 文件到 `mwe-arxiv.tex`。

直接复制`mwe.bbl` 到 `mwe-arxiv.tex` 文件（适合引用较少，此时上传文件只需要包含`mwe-arxiv.tex`即可）：

```tex
\documentclass[10pt,a4paper]{article}

\usepackage{hyperref} % for better urls


\begin{document}

This is text with \cite{Goossens} and \cite{adams}.

\nocite{*} % to test all bib entrys
%\bibliographystyle{unsrt} % <======================== not longer needed!
%\bibliography{\jobname} % <========================== not longer needed!
\begin{thebibliography}{1} % <================================== mwe.bbl

\bibitem{Goossens}
Michel Goossens, Frank Mittelbach, and Alexander Samarin.
\newblock {\em The LaTeX Companion}.
\newblock Addison-Wesley, 1 edition, 1994.

\bibitem{adams}
Douglas Adams.
\newblock {\em The Restaurant at the End of the Universe}.
\newblock The Hitchhiker's Guide to the Galaxy. Pan Macmillan, 1980.

\end{thebibliography} % <======================================= mwe.bbl

\end{document}
```

使用 `\input{mwe.bbl}` （上传文件包含`mwe-arxiv.tex` 和 `mwe.bbl`）：

```tex
\documentclass[10pt,a4paper]{article}

\usepackage{hyperref} % for better urls


\begin{document}

This is text with \cite{Goossens} and \cite{adams}.

\nocite{*} % to test all bib entrys
%\bibliographystyle{unsrt} % <======================== not longer needed!
%\bibliography{\jobname} % <========================== not longer needed!
\input{mwe.bbl} % <============================================= mwe.bbl

\end{document}
```


### 总结

1. 首先运行 `pdflatex mwe.tex` 和 `bibtex mwe` 生成必要文件。
2. 然后运行 `pdflatex mwe.tex` 得到正确的页码。
3. 将文件`mwe.bbl` 引入到 `mwe.tex`。
4. 上传`mwe.tex` 和 `mwe.bbl` 文件（其他 Figures 文件正常引入即可）。



### Tips

支持的 Figure 格式：

- PostScript (PS, EPS) — requires [LaTeX processing](https://arxiv.org/help/submit_tex#latex)
- JPEG, GIF, PNG or PDF figures — requires [PDFLaTeX processing](https://arxiv.org/help/submit_tex#pdflatex)

文件名只允许包含 `a-z A-Z 0-9 _ + - . , =` 字符，`Figure1.PDF` and `figure1.pdf` 不是同一个文件。

Schedule(all times Eastern US):

| Submissions received between     | Will be announced | Mailed to subscribers              |
| -------------------------------- | ----------------- | ---------------------------------- |
| Monday 14:00 – Tuesday 14:00     | Tuesday 20:00     | Tuesday night / Wednesday morning  |
| Tuesday 14:00 – Wednesday 14:00  | Wednesday 20:00   | Wednesday night / Thursday morning |
| Wednesday 14:00 – Thursday 14:00 | Thursday 20:00    | Thursday night / Friday morning    |
| Thursday 14:00 – Friday 14:00    | Sunday 20:00      | Sunday night / Monday morning      |
| Friday 14:00 – Monday 14:00      | Monday 20:00      | Monday night / Tuesday morning     |


### 参考

1. https://tex.stackexchange.com/questions/329198/how-to-obtain-and-use-the-bbl-file-in-my-tex-document-for-arxiv-submission

2. https://arxiv.org/help/submit
