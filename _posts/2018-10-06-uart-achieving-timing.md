---
id: 38
title: "UART :: Achieving Timing closure"
date: 2018-10-06
excerpt: |
  Although my UART project worked fine when implemented on FPGA, there are some critical warnings related to timing which shows up during the compilation.
  
layout: post
permalink: /2018/10/06/uart-achieving-timing/
---
Although my [UART project](https://siddharth.pro/serialfpga.html) worked fine when implemented on FPGA, there are some critical warnings related to timing which shows up during the compilation.

![Step 1](/assets/images/UART_timing/Step1.png)

The contents in red indicates timing violations that have occured. There are unconstrained paths associated with some input and output ports.

![Step 2](/assets/images/UART_timing/Step2.png)

Also it can be seen that the setup slack is -3.975. Negative slack is undesirable as it means the timing requirements are not met. Here setup slack is negative, therefore setup timing requirements are not met.

We need to account for this timing violations by performing timing analysis. For Altera/Intel FPGAs this can be done by using TimeQuest Timing Analyzer.

![Step 3](/assets/images/UART_timing/Step3.png)

I started by creating new Timing Netlist by selecting Post-fit, Slow-corner delay model with Speed grade of 8. The Cyclone IV FPGA on my FPGA board is of speed grade 8.

After creating the netlist, under Task pane, Report All Summaries and Report Unconstrained Paths is clicked. This will show all the timing violations and unconstrained paths.

![Step 4](/assets/images/UART_timing/Step4.png)

Creating Slack Histogram shows the negative slack in red indicating timing violation. This is more evident from the Timing Report below.

![Step 5](/assets/images/UART_timing/Step5.png)

This clearly shows that setup slack is negative (violated) and also shows the Data Arrival Path and Data Required Path. The setup slack can be improved by creating a clock constraint.

![Step 6](/assets/images/UART_timing/Step6.png)

I created a Clock constraint with same period as the FPGA clock i.e 20ns. After the Clock constraint is created the timing netlist is updated.

![Step 7](/assets/images/UART_timing/Step7.png)

The Slack Histogram now shows all the timing in blue, meaning there is no setup violation which can be further confirmed from the timing report.

![Step 8](/assets/images/UART_timing/Step8.png)

But the Unconstrained Paths is still red and setup and hold values are not zero. This can be solved by setting delays for input and output ports.

![Step 9](/assets/images/UART_timing/Step1.png)

For the input ports I set the maximum delay of 3ns and minimum delay of 2ns.

![Step 10](/assets/images/UART_timing/Step10.png)

Similary for the output ports the maximum delay is set to 2ns and minimum delay as 1ns.

![Step 11](/assets/images/UART_timing/Step11.png)

After setting up the input and output delays, the timing netlist is updated again. Now looking at the Unconstrained paths there are zero Setup and Hold values.

![Step 12](/assets/images/UART_timing/Step12.png)

Also note the changes in the Slack Histogram.

![Step 13](/assets/images/UART_timing/Step13.png)

The Setup slack is now 8.406 which reduced from 15.025 after adding input and output delays. 

![Step 14](/assets/images/UART_timing/Step14.png)

After doing all the necessary analysis and obtaining timing closure, the SDC file must be saved.

![Step 15](/assets/images/UART_timing/Step15.png)

The design is recompiled again and it can be seen on the Table of contents that there are no timing violations occured.

![Step 16](/assets/images/UART_timing/Step16.png)

Setup Slack is now positive

![Step 17](/assets/images/UART_timing/Step17.png)

and also there are no Unconstrained paths.
