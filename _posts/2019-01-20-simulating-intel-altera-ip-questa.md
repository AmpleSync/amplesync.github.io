---
layout: post
title: "Simulating Intel/Altera FIFO IP using UVM in Questasim"
date: "2019-01-20"
---

FPGA designs often contain one or more built-in IPs provided by the FPGA vendor tools. 
But verifying those designs is not a straight forward task. In case of Intel/Altera FPGA 
tools they provide their own version of Modelsim called Modelsim-Altera to simulate the 
designs created in Quartus Prime or Quartus II. But it is limited in what it can perform and if you want to use advanced 
verification techniques you will mostly want to go with Questasim. 

There's a method by which Questasim can be made to work with Altera IPs. It involves compiling the
standard altera components to create libraries manually in Questasim and making changes to
the root configuration file. So it might be better to save a backup of the modelsim.ini
file if anything goes wrong.

I used the below tcl script to compile the component files. Make sure to include
the correct file name for the FPGA you have. In my case it is Cyclone IV E.
Rest of the files should remain the same. Also make sure these files are actually present in
the Quartus II installation folder. Check under Quartus II installation directory/quartus/eda/sim_lib.

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

Save this script as altera_lib.tcl in the Questasim root directory path.
Now open up Questasim and change the directory to the path where you placed
the tcl script i.e. in the root directory.

Now run the file from the transcript window using do command "``` do altera_lib.tcl ```".

This will take a while. After that's done, check the Questasim installation folder for the new folders created by the command.

Now we must make changes in the configuration file modelsim.ini as below.

```
lpm_ver = $MODEL_TECH/../lpm_ver
altera_lnsim_ver = $MODEL_TECH/../altera_lnsim_ver
altera_mf_ver = $MODEL_TECH/../altera_mf_ver
altera_ver = $MODEL_TECH/../altera_ver
sgate_ver = $MODEL_TECH/../sgate_ver
cycloneive_ver = $MODEL_TECH/../cycloneive_ver
```

These must be included above [vcom]. After that's done, save the file and open Questasim. You should be able to see the new altera libraries created under Library window. The Questasim can now be used to verify the designs with Altera IPs.
