# 1. 引言

早期的电脑简单许多。由于系统中的众多元件，如 CPU、memory、大容量储存装置（mass storage）、以及网卡（network interface），是一同被发展出来的，也因此在效能上相当平衡。举例来说，在提供资料时，memory与网卡并不比 CPU 来得快（非常多）。

这个情况随著电脑基础结构的稳定、以及硬件开发者专注于最佳化独立的子系统而改变了。某些电脑元件的效能突然大幅地落后并成为了瓶颈。尤其是大容量储存装置与memory –– 由于成本的缘故 –– 相比于其他系统，进步得十分缓慢。

大容量储存装置的效能问题主要由软件技术解决：操作系统（operating system）在主memory（main memory）–– 其在存取上比硬盘（hard disk）快了几个数量级 –– 中保存了最常使用（并且最有可能被使用）的资料。cache亦被加到储存装置中，其不需对操作系统作任何改变就能提升效能。[^1]由于偏离本文的主旨，我们就不继续深入针对大型储存装置存取的软件最佳化细节了。

不若储存子系统一般，解决主memory的瓶颈已被证实更加困难，而且几乎所有的解决方法都必须改变硬件。现今这些改变主要有以下这些方式：

* RAM 的硬件设计（速度以及平行度〔parallelism〕）。
* memory控制器（controller）的设计。
* CPU cache。
* 装置的直接memory存取（Direct Memory Access，DMA）。

本文主要涉及 CPU cache以及memory控制器设计的一些影响。在探索这些主题的过程中，我们将对 DMA 有更深入的理解。不过，我们要先概观地从现今商用硬件的设计开始。这是理解高效使用memory子系统的问题与限制的必要条件。我们也会 –– 稍加详细地 –– 学到不同类型的 RAM，并阐述为何这些差异依旧存在。

这篇文件绝非最终且完整的。本文仅止于商用硬件，而且仅限这类硬件的一个子集。同时，许多主题的讨论仅点到为止，以能达到本文目的为主。对于这些主题，建议读者去寻找更详尽的文件。

当提及特定操作系统的细节与解法时，本文仅特指 Linux。无论何时都不会涵盖其他操作系统的任何资讯。作者无意著墨于其它操作系统。假如读者认为他必须使用不同的操作系统，那他该去寻求供应商撰写与本文相似的文件。

开始前的最后一点说明。本文包含许多「通常」以及其他类似的修饰语。在这里讨论的技术在现实世界中存在著非常非常多不同的变种，而本文仅针对最常见、最主流的版本。这些技术很少能下定论，是故以此言之。

## 文件结构

本文主要写给软件开发者，不会深入对硬件导向的读者有用的硬件技术细节。但在我们能够讨论对开发者实用的资讯之前，有许多基础得打。

为了达到这个目标，第二节会以技术细节面描述随机存取memory（Random-Access Memory，RAM）。这一节的内容值得一读，但对于理解后面几节并非必要。在需要此节内容之处都会加上合适的引用（back reference），所以心急的读者可以先略过本节的大部分内容。

第三节描述了许多 CPU cache行为的细节。本节会用上一些图表以避免文字变得枯燥乏味。本节内容是理解本文后续章节所不可或缺的。第四节简述了虚拟memory（virtual memory）如何实作。这也是其余部份的必要基础。

第五节描述了非均匀memory存取（Non-Uniform Memory Access，NUMA）系统的诸多细节。

第六节是本文的核心章节。其将先前几节的资讯总结在一起，并给予程序开发者如何能在不同情境中撰写良好运作的程序建议。非常不耐烦的读者可以从此节开始，并且 –– 必要的话 –– 回到先前的章节回顾基础技术的知识。

第七节介绍了一些能够帮助程序开发者做得更好的工具。即使完全理解了这些技术，距离明确复杂软件专案的问题所在仍十分遥远。某些工具是必要的。

最后在第八节，我们将展望可以在未来几年期待、或是希望拥有的技术。

## 回报问题

作者想要更新这份文件一段时间。这包含了不仅得随著技术推进而更新，也要修正错误。乐于回报问题的读者可以来信给作者。但请在回报中包含精确的版本资讯。版本资讯可以在文件的最后一页找到。

## 致谢

我要感谢 Johnray Fuller 以及 LWN 的夥伴们（尤其是 Jonathan Corbet 承担著将作者式的英文改成更加传统形式的艰巨任务）。Markus Armbruster 针对本文的问题与疏忽提供了诸多有价值的建议。

## 关于本文

本文标题致敬于 David Goldberg 的经典论文《[What Every Computer Scientist Should Know About Floating-Point Arithmetic](https://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html)》。这篇论文仍鲜少人知，虽然这应该是任何勇于严谨地撰写程序而敲下键盘者的先决条件。

[^1]: 然而，为了保证使用储存装置cache时的资料完整性（data integrity），改变是必要的。
