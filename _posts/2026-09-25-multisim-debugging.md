---
layout: post-cn
title:  "记一次 Multisim 的 debug 历程"
date:   2026-09-25 10:00:00 +0800
categories: zh-cn
tags: debugging EDA TA 
math: False
---

这周作为助教，带完了“逻辑设计导论”这门课的第一个 lab。两位同学在验收时出现了同样的故障：**Multisim 仿真无误，生成的 bitstream 导入 FPGA 开发板之后却出了错**。我通过 bitstream 向上反推一步步排查，先排除硬件故障，最终通过观察中途生成的 VHDL 文件内容定位到了问题所在，并通过重启学生电脑的方式顺利解决。

## 背景与问题描述

这门“逻辑设计导论”课程有 lab 环节。前三个 lab 通过 Multisim 和 Xilinx Vivado 共同完成，学生需要在 Multisim 里完成逻辑电路的设计与仿真，并通过其“PLD export with a tool”功能，调用电脑上的 Vivado 完成逻辑综合等操作，最终生成 bitstream 并导入 FPGA 开发板。其中，调用 Vivado 的步骤均由 Multisim 自动完成，无需手动干预。

在我带周二的 lab 的时候，一位同学向我反映，自己的仿真都没问题，但是导入开发板之后功能出了错。我接过来一看，发现本应起效果的两个开关在拨动之后完全没有反应，只有一个开关拨了之后产生反应。看了她的 Multisim 仿真，结果正确无误。紧接着，另一位同学也向我反映了相同的问题，症状一模一样。

## 排查历程

### 第一反应：偶发故障

虽然有两份完全一致的问题，但我的第一反应仍然是偶发的故障。这些 FPGA 开发板套件已经用了好几届了，数据线接口老化松动引起接触不良导致 bitstream 损坏是一个不可忽略的原因。于是我让他们重新走一遍烧录的流程，观察问题是否复现。

整个烧录的流程大约要花 5 分钟。但是导入完毕之后，功能依旧错误。这就说明，此问题**不是偶发故障，可复现**。

### 山重水复：排除硬件故障

我决定先从硬件入手，看看是不是开发板硬件的原因。流程中生成的 bitstream 文件会被 Multisim 保存下来，我就让其中一位同学把 bitstream 发给我，然后我手动用 Vivado 导入自己的同款开发板。

导入完毕之后，我一拨开关，功能错误，症状相同。**硬件的锅被排除了。**

### 柳暗花明：VHDL 文件中一些不该出现的 `open`

现在我明确了，问题出在 bitstream 及其上游。但是 Multisim 调用一系列操作的过程是自动完成的，log 里也没有任何报错。有没有什么办法看中间产物呢？

有的，兄弟，有的。事实上 Multisim 在“Export to PLD”时有三个选项，最简便的就是“Program the Connected PLD”，即直接完成所有操作并把 bitstream 导入到连接在电脑上的开发板上。但其实底下还有两个选项，如图所示。

{% include image.html
  src="/assets/images/multisim-debug/pld-export-selection.png"
  width="400"
  alt="Multisim “Export to PLD”选项图示"
  caption="Multisim 的 “Export to PLD”三个选项。"
 %}
 
其中，第二个选项是调用 Vivado 生成 bitstream，第三个选项是**直接生成 VHDL 文件**，不调用 Vivado。VHDL 也是一种类似于 Verilog HDL 的硬件描述语言，而这里的 **VHDL 文件就是 Multisim 整个流程的中间产物**：Multisim 根据设计生成描述相同功能的 VHDL 文件，再调用 Vivado 将 VHDL 文件转化成适配对应开发板的 bitstream。

我有 Verilog HDL 的基础但没怎么看过 VHDL，打开生成的 VHDL 文件一时也没有头绪。但幸运的是，十几分钟前刚刚有一位同学找我帮忙烧录，我本地**留有一份最终能成功的 VHDL 文件可供比对**。我的电脑跑着 Linux 操作系统，我就立刻想到了用 `colordiff` 命令行工具直观比对两个文件的差异。

以下是 `colordiff` 输出的部分关键结果。红色代表参考版本，即成功的那份 VHDL 文件；绿色代表需比对的版本，即功能出错的那份 VHDL 文件。

```diff
95,99c92,96
< 		port map( A => \PLD1/A\, B => \PLD1/B\, Y => \3\ );
< 	U2 : XOR2_NI
< 		port map( A => \PLD1/A\, B => \PLD1/B\, Y => \1\ );
< 	U3 : OR2_NI
< 		port map( A => \2\, B => \3\, Y => \PLD1/Carry\ );
---
> 		port map( A => PLD1_l_A, B => PLD1_l_B, Y => open );
> 	U2 : AND2_NI
> 		port map( A => open, B => PLD1_l_C, Y => open );
> 	U3 : XOR2_NI
> 		port map( A => PLD1_l_A, B => PLD1_l_B, Y => open );
101,103c98,100
< 		port map( A => \1\, B => \PLD1/C\, Y => \PLD1/Sum\ );
< 	U5 : AND2_NI
< 		port map( A => \1\, B => \PLD1/C\, Y => \2\ );
---
> 		port map( A => open, B => PLD1_l_C, Y => PLD1_l_Sum );
> 	U5 : OR2_NI
> 		port map( A => open, B => open, Y => PLD1_l_Carry );
```

<br>

可以看到，在出错的版本里，VHDL 文件中出现了一些不该出现的 `open`，即开路。也就是说，这个 VHDL 文件描述的硬件本身，就有很多线路没连上。问题发现了：**Multisim 生成的 VHDL 文件，有错误。**

### 最终锚定：Multisim 自己的 bug

虽然问题发现了，但其根源尚不明确；虽然 Multisim 的仿真能过，但也不能完全排除同学的设计是正确的。于是我让其中一位同学把所有的设计文件发送给我，我用自己的 Multisim 生成 VHDL 文件。结果：我生成的 VHDL 文件中，`open` 消失了。**同学的设计没有问题**。

那么，~~真実は、いつもひとつ~~ 真相只有一个：**Multisim，它自己出 bug 了！**

俗话说得好，重启解决 90% 的问题。于是我就让两位同学重启了一下自己的电脑再试一遍。这一次，**功能对了，验收通过**。

## 一些思考

我猜测，这个 bug 可能是 Multisim 软件内部有些地方失去了同步，“Export to PLD”看到的电路图相较于仿真器和 UI 前端看到的电路图少了几根关键的连线。重启电脑相当于刷新了一遍同步状态。然而我无权查看 Multisim 内部的代码，因而究竟是什么导致了这个软件 bug 我也无从了解。

我也越来越感受到**这个世界就是个巨大的草台班子**。Multisim 能出这种 bug，前几天用 Cadence Virtuoso 跑仿真的时候还触发了个段错误导致程序整体崩溃。**工程思维**能让我比较精准地定位到这次故障的原因所在，但**“能跑就别动”的工程哲学**也很有可能是导致这次故障的原因。:P
