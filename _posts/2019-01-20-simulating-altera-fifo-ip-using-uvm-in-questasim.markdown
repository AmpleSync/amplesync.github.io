---
layout: post
title: "Simulating Intel/Altera FIFO IP using UVM in Questasim"
date: "2019-01-20"
---


In the previous post I explained a method for simulating Altera IPs in Questasim, as an example I tried to verify Altera FIFO IP using UVM in Questasim.
For that, first you need to create a FIFO in Quartus II using Tools -> MegaWizard Plug-In Manager option. Instantiate a FIFO with required data width and depth. I chose data width of 8 and depth of 256 and used same clock for both read and write, which makes it synchronous. 

Under Tools -> Options -> EDA Tools Options, provide the execuatable path of Questasim. 
Select Assignments -> Settings -> Simulation and choose QuestaSim as Tool name and provide the testbench location in NativeLink settings.  
Compile the project and click on Tools -> Run EDA Simulation Tool -> EDA RTL Simulation.

Now you should be able to see a .do file in the project location\Simulation\modelsim along with the netlist file .vo if you had chosen verilog
in the Simulation option. Open the .do file and remove the library mappings since we have already compiled the libraries and added them to the
Questasim. 

It should look like below

{% highlight code linenos %}

transcript on

if {[file exists rtl_work]} {
	vdel -lib rtl_work -all
}
vlib rtl_work
vmap work rtl_work


vlog -vlog01compat -work work +incdir+F:/Files/HDLfiles/2019/January/FIFO_verification/simulation/modelsim {F:/Files/HDLfiles/2019/January/FIFO_verification/simulation/modelsim/fifo.vo}

vlog -sv -work work +incdir+F:/Files/HDLfiles/2019/January/FIFO_verification/simulation/modelsim {F:/Files/HDLfiles/2019/January/FIFO_verification/simulation/modelsim/tb_fifo_top.sv}

vsim -t 1ps -L altera_ver -L lpm_ver -L sgate_ver -L altera_mf_ver -L altera_lnsim_ver -L cycloneive_ver -L rtl_work -L work -voptargs="+acc" tb_fifo_top

add wave sim:/tb_fifo_top/intf/*
view structure
view signals
run -all

{% endhighlight %}

Create a Questasim project in the location where you have compiled the above files and run the .do file from the transcript window.   

#### Output Waveform

![Waveform](/assets/images/alt_fifo.png) 

[View code on GitHub](https://github.com/gitSiddharth/altera_fifo_ip_verification)
