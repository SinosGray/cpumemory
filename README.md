# 每位程序开发者都该有的memory知识

本文翻译自 [Ulrich Drepper](https://de.wikipedia.org/wiki/Ulrich_Drepper) 于 2007 年撰写的论文《[What Every Programmer Should Know About Memory](https://www.akkadia.org/drepper/cpumemory.pdf)》(版次: 1.0)，原文共 114 页。

在 CPU 核 (core) 在速度和数量增长的同时，memory存取限制著当今大多数程序的效率，甚至未来一段时间也会如此。
尽管硬件设计者已提出日趋复杂的memory处理与加速机制 —— 例如 CPU cache —— 但若程序开发者无法善用，仍无法有效发挥硬件作用。
不幸的是，论及电脑的memory子系统或 CPU cache时，无论是其内部的结构，抑或存取成本，对大多程序开发者仍相当陌生。
本文解释用于现代电脑硬件的memory子系统的结构、阐述 CPU cache发展的考量、它们如何运作，以及程序该如何针对memory操作调整，从而达到最佳的效能。

## 翻译资讯
译者: [Chi-En Wu](https://github.com/jason2506), [Jim Huang](https://github.com/jserv)
> **[info]**
> 关于繁体中文翻译内容的修正、改进建议，和贡献，请造访 [sysprog21/cpumemory-zhtw](https://github.com/sysprog21/cpumemory-zhtw)

转载注 :我将原仓库的 gitbook 进行了繁译简, 对于部分语言使用差异进行了修改或者直接使用英文(如 memory, bit, byte 等), 注意有些容易混淆的地方, 例如繁中的行列与简中的行列对应关系是反过来的, 请留意
