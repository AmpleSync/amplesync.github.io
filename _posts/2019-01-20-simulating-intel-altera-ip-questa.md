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

```

set path_to_quartus c:/altera/10.1/quartus
set type_of_sim compile_all


if {[string equal $type_of_sim "compile_all"]} {
	vlib lpm_ver
	vmap lpm_ver lpm_ver
	vlog -work lpm_ver $path_to_quartus/eda/sim_lib/220model.v

	vlib altera_mf_ver
	vmap altera_mf_ver altera_mf_ver
	vlog -work altera_mf_ver $path_to_quartus/eda/sim_lib/altera_mf.v
        
	vlib altera_lnsim_ver
	vmap altera_lnsim_ver altera_lnsim_ver
	vlog -work altera_lnsim_ver $path_to_quartus/eda/sim_lib/altera_lnsim.sv
	
	vlib altera_ver
	vmap altera_ver altera_ver
	vlog -work altera_ver $path_to_quartus/eda/sim_lib/altera_primitives_quasar.v
	vlog -work altera_ver $path_to_quartus/eda/sim_lib/altera_primitives.v
	
	vlib sgate_ver
	vmap sgate_ver sgate_ver
	vlog -work sgate_ver $path_to_quartus/eda/sim_lib/sgate.v

	vlib cycloneive_ver
	vmap cycloneive_ver cycloneive_ver
	vlog -work cycloneive_ver $path_to_quartus/eda/sim_lib/cycloneive_atoms.v

} elseif {[string equal $type_of_sim "functional"]} {
	vlib lpm_ver
	vmap lpm_ver lpm_ver
	vlog -work lpm_ver  $path_to_quartus/eda/sim_lib/220model.v
        
        vlib altera_lnsim_ver
	vmap altera_lnsim_ver altera_lnsim_ver
	vlog -work altera_lnsim_ver  $path_to_quartus/eda/sim_lib/altera_lnsim.sv

	vlib altera_mf_ver
	vmap altera_mf_ver altera_mf_ver
	vlog -work altera_mf_ver  $path_to_quartus/eda/sim_lib/altera_mf.v
} elseif {[string equal $type_of_sim "ip_functional"]} {

	vlib lpm_ver
	vmap lpm_ver lpm_ver
	vlog -work lpm_ver  $path_to_quartus/eda/sim_lib/220model.v

        vlib altera_lnsim_ver
	vmap altera_lnsim_ver altera_lnsim_ver
	vlog -work altera_lnsim_ver  $path_to_quartus/eda/sim_lib/altera_lnsim.sv

	vlib altera_mf_ver
	vmap altera_mf_ver altera_mf_ver
	vlog -work altera_mf_ver $path_to_quartus/eda/sim_lib/altera_mf.v
	vlib sgate_ver
	vmap sgate_ver sgate_ver
	vlog -work sgate_ver  $path_to_quartus/eda/sim_lib/sgate.v

} elseif {[string equal $type_of_sim "cycloneive"]} {
	vlib cycloneive_ver
	vmap cycloneive_ver cycloneive_ver
	vlog -work cycloneive_ver  $path_to_quartus/eda/sim_lib/cycloneive_atoms.v
} else {
	puts "invalid code"
}

```
Create a Questasim project in the location where you have compiled the above files and run the .do file from the transcript window.   

#### Output Waveform

![Waveform](/assets/images/alt_fifo.png) 

[View code on GitHub](https://github.com/gitSiddharth/altera_fifo_ip_verification)
