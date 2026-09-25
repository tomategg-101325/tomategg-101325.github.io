---
layout: post-en
title:  "A Multisim Debugging Journey"
date:   2026-09-25 10:00:00 +0800
categories: en-us
tags: debugging EDA TA 
math: False
---

This week, as a TA, I finished the first lab of the "Introduction to Logic Design" course. During the demo, two students ran into the same failure: **the Multisim simulation was correct, but after the generated bitstream was loaded onto the FPGA development board, it misbehaved.** I traced backward from the bitstream step by step, first ruling out hardware failure, and eventually located the problem by inspecting an intermediate VHDL file. It was fixed by restarting the students' computers.

## Background and Problem Description

This "Introduction to Logic Design" course has a lab component. The first three labs are done with both Multisim and Xilinx Vivado. Students design and simulate logic circuits in Multisim, then use its "PLD export with a tool" feature to invoke Vivado on the computer for logic synthesis and related steps, finally generating a bitstream and loading it onto the FPGA development board. The Vivado invocation steps are all handled automatically by Multisim, with no manual intervention required.

When I was TAing the Tuesday lab, a student told me her simulation was fine, but after loading onto the board, the functionality was wrong. I took a look and found that two switches that should have had an effect did nothing when toggled, while only one switch produced a response. I checked her Multisim simulation; the result was correct. Then another student reported the same problem, with identical symptoms.

## Troubleshooting

### First Reaction: An Intermittent Failure

Even though there were two identical problems, my first reaction was still that it was an intermittent failure. These FPGA development kit boards have been used for several cohorts; aging and loose connectors can cause the bitstream to be corrupted during programming due to poor contact, and that was not a factor I could ignore. So I had them go through the programming flow again and watched to see whether the problem reproduced.

The whole programming flow takes about 5 minutes. But after loading, the functionality was still wrong. This showed that the problem was **not intermittent but reproducible**.

### Twists and Turns: Ruling Out Hardware Failure

I decided to start with the hardware and see whether the development board itself was the cause. The bitstream file generated during the flow is saved by Multisim, so I had one of the students send me the bitstream, and then I manually imported it into my own board of the same model using Vivado.

After loading, I flipped the switches: wrong functionality, same symptoms. **Hardware was ruled out.**

### A Breakthrough: Some `open`s That Should Not Appear in the VHDL File

Now it was clear that the problem lay in the bitstream and its upstream. But Multisim performs the whole series of operations automatically, and there were no errors in the log. Was there any way to see the intermediate artifacts?

Oh, there is, my friend. There is. In fact, when Multisim does "Export to PLD", there are three options. The easiest is "Program the Connected PLD," which directly completes all the operations and loads the bitstream onto the development board connected to the computer. But there are two more options below, as shown in the figure.

{% include image.html
  src="/assets/images/multisim-debug/pld-export-selection.png"
  width="400"
  alt="Illustration of Multisim's 'Export to PLD' options"
  caption="Multisim's three 'Export to PLD' options."
 %}
 
The second option invokes Vivado to generate a bitstream; the third option **directly generates a VHDL file** without invoking Vivado. VHDL is a hardware description language similar to Verilog HDL, and here the **VHDL file is the intermediate artifact of the entire Multisim flow**: Multisim generates a VHDL file describing the same functionality based on the design, then invokes Vivado to convert the VHDL file into a bitstream adapted to the corresponding development board.

I have a foundation in Verilog HDL but have not looked at much VHDL. When I opened the generated VHDL file, I had no clue at first. Fortunately, just ten minutes earlier another student had asked me for help programming, and I had **a local copy of a VHDL file that ultimately worked, for comparison**. My computer runs Linux, so I immediately thought of using the `colordiff` command-line tool to compare the differences between the two files visually.

Below is part of the key output from `colordiff`. Red represents the reference version, i.e., the successful VHDL file; green represents the version being compared, i.e., the faulty VHDL file.

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

As can be seen, in the faulty version, the VHDL file contains some `open`s that should not appear, i.e., open circuits. In other words, the hardware described by this VHDL file itself has many nets that are not connected. The problem was found: **the VHDL file generated by Multisim was erroneous.**

### Final Pinpointing: Multisim's Own Bug

Although the problem was found, its root cause was still unclear. Although Multisim's simulation passed, I still could not be entirely sure that the student's design was correct. So I had one of the students send me all the design files, and I used my own Multisim to generate a VHDL file. The result: in the VHDL file I generated, `open` disappeared. **The student's design was not the problem.**

Then, there is only one truth: **Multisim itself has a bug!**

As the saying goes, restarting solves 90% of problems. So I had both students restart their computers and try again. This time, **the functionality was correct, and they passed the demo**.

## Some Reflections

I suspect this bug may be caused by some part of Multisim losing synchronization internally: compared with the circuit diagram seen by the simulator and the UI front end, the circuit diagram seen by "Export to PLD" is missing several key connections. Restarting the computer refreshes the synchronization state. However, I have no access to Multisim's internal code, so I have no way of knowing what exactly caused this software bug.

I also increasingly feel that **the world is just a giant ramshackle operation**. Multisim can have this kind of bug; a few days ago, when I was running a simulation with Cadence Virtuoso, it even triggered a segfault that crashed the entire program. **Engineering thinking** allowed me to locate the cause of this failure fairly precisely, but **the engineering philosophy of "if it runs, don't touch it"** is also quite possibly what caused this failure. :P
