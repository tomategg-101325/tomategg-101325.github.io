---
layout: post-cn
title:  "OpenLane 数字后端流程探索"
date:   2026-05-25 08:00:00 +0800
categories: zh-cn
tags: asic digital backend openlane
---

最近在自学数字 IC 设计的后端流程，使用的是开源的 OpenLane 套件搭配 SkyWater 130nm 工艺库。本帖用于记录学习过程和遇到的问题。

以下面的 `counter_4bit.v` RTL 代码作为示例。该代码描述了一个 4 位计数器，具有低电平有效的同步计数使能和同步清零的控制信号。

{% highlight verilog %}
module counter_4bit (
    output reg [3:0] value,
    input ce,               // active-low sync count enable
    input clr,              // active-low sync clear (higher priority)
    input clk,              // positive edge triggered
    input rst               // active-low async reset
);
    always @(posedge clk or negedge rst) begin  // sequential
        if (~rst) value <= 4'h0;
        else begin
            case ({ce, clr})
                2'b00:   value <= 4'h0;         // clr has priority
                2'b01:   value <= value + 1;
                2'b10:   value <= 4'h0;
                default: value <= value;
            endcase
        end
    end
endmodule
{% endhighlight %}

## OpenLane 工作流概览

(TODO)

## 逻辑综合 (Synthesis) 

### 逻辑综合

逻辑综合将高阶的行为级描述转换为低阶的门级电路。其输入是 RTL 代码，输出是门级网表 (gate-level netlist)。在此过程中，综合器根据 RTL 代码中的行为描述，选择指定工艺库中合适的标准单元 (standard cell) 生成功能一致的门级电路，并输出该电路的网表。

OpenLane 套件中，负责逻辑综合的工具是 Yosys。它接受我们写好的 RTL 代码文件，处理后输出 Verilog 格式的门级网表，其中采用指定的 SkyWater 130nm 工艺库的标准单元。其流程大致包括：

- 几轮优化，其中包括针对有限状态机 (FSM) 和内存 (MEM) 的特殊优化；
- 两轮 technology mapping，第一次生成对应的一般性的逻辑门，第二次映射到工艺库中的标准单元；
- 各类检查和统计数据输出。

<br>
两次 technology mapping 明确地体现生成的日志文件中。

1. 第一次 (47)：所有的逻辑门都是一般性的，用 `$_DFFE_PN0P_` 等表示。
2. 第二次 (61)：已经对应到了 SkyWater 130nm 工艺库中的标准单元，如 `sky130_fd_sc_hd__dfrtp_2`。上一次的输出结果中的同一种逻辑门在这一次可能会对应到不同的标准单元，且还会添加缓冲器 (buf) 以满足时序和驱动要求。

```
=== counter_4bit === (第一次)

   Number of wires:                 15
   Number of wire bits:             21
   Number of public wires:           5
   Number of public wire bits:       8
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:                 17
     $_ANDNOT_                       5
     $_DFFE_PN0P_                    4
     $_NAND_                         2
     $_NOR_                          1
     $_ORNOT_                        2
     $_XNOR_                         1
     $_XOR_                          2
```
<br>
```
=== counter_4bit === （第二次）

   Number of wires:                 19
   Number of wire bits:             22
   Number of public wires:           5
   Number of public wire bits:       8
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:                 18
     sky130_fd_sc_hd__a21boi_2       1
     sky130_fd_sc_hd__a21o_2         1
     sky130_fd_sc_hd__a21oi_2        1
     sky130_fd_sc_hd__and3b_2        3
     sky130_fd_sc_hd__and4b_2        1
     sky130_fd_sc_hd__buf_1          2
     sky130_fd_sc_hd__dfrtp_2        4
     sky130_fd_sc_hd__inv_2          2
     sky130_fd_sc_hd__o21a_2         1
     sky130_fd_sc_hd__o21ai_2        1
     sky130_fd_sc_hd__or2_2          1
```

<br>

在第 60 步的检查中，工具给出了警告，称 `Warning: Wire counter_4bit.\value [3] is used but has no driver.` 意思似乎是输出端口没有驱动。但是我检查了输入的 RTL 代码以及输出的门级网表，发现输出端口均有连接。基于已知信息，猜测原因可能是综合过程中对网络的名称进行了修改，在这一步还没完全解析出来。

### 静态时序分析 (STA)



