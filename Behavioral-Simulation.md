## About this tutorial 
:stopwatch: 30 min

**Key learnings:**

* Simulate a custom logic using Vivado Xsim

* Use RTL testbench 
* Use TCL script for stimulus for stimulus generation

**Description**:

In this tutorial we are going to simulate the operation of custom VHDL counter. Behavioural simulation is a key procedure to test and debug custom FPGA logic, which is strongly recommended before synthesis, implementation and hardware deployment. Note that this is an additional step not required in software development, where (in opposite to FPGA logic) compilation is typically completed after a few seconds and it is simple to perform runtime debugging. 

## Open Vivado Design
In this case we are not going to create a new design, instead we are going to simulate the VHDL counter that is part of the [LED blink](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/wiki/LED-blink) Vivado project. 

## Vivado Simulator
The Vivado toolchain allows you to simulate your logic at different points of the build process:
 * **Behavioral simulation**: *ideal* simulation, only based on the provided RTL (Verilog, VHDL...) code.
 * **Post-synthesis simulation**: simulation of the RTL code when converted to actual FPGA logic building blocks (Flip-Flops, LUTs...). 
 * **Post-implementation simulation**: simulation of the RTL code when converted to actual FPGA logic building blocks and placed within the specified FPGA chip. 

For most application it is sufficient to verify your logic via **Behavioural Simulations**. Vivado has built-in mechanisms that raise an error if synthesis and implementation fail (e.g. when the requested logic cannot meet timing constraints, when the requested resources are not available in the specified FPGA chip...).


The following discussion covers two different methods to simulate custom FPGA logic. 
1. Creating and executing an RTL testbench 
2. TCL stimulus. 

|1. RTL testbench |2. TCL stimulus |
|---|---|
| :heavy_check_mark: Standard procedure| :x: Vivado Xsim only procedure |
| :heavy_check_mark: Fast simulation and suited for exhaustive testing| :x:  Slow simulation and suited for simple tests.|
| :heavy_check_mark: Single programming language (Verilog, VHDL...)| :x: Requires to learn/use TCL scripts |
| :x: Requires development of testbench that instantiate the RTL to test| :heavy_check_mark: No testbench required  |
| :x: Slow to write| :heavy_check_mark: Fast and easy to write|
| :x: Testbench is fixed. Changes require to recompile simulation.| :heavy_check_mark: Simulation is entirely based on runtime TCL commands. Recompilation is not needed.|

## 1. RTL testbench
* We first have to create or input an RTL testbench that instantiates the [counter.vhd](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/counter.vhd) module. To this end, click on *PROJECT MANAGER --> Add Sources* (left panel of your Vivado window) and select *Add or create simulation sources*.

<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/behavioural-simulation/create_tb_0.PNG" width="450"/>

* Import [tb_counter.vhd](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/sim/tb_counter.vhd) or [tb_counter.v](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/sim/tb_counter.v) testbench and click *Finish*.

<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/behavioural-simulation/create_tb_1.PNG" width="450"/>

* Select *tb_counter.vhd* or *tb_counter.v* as the *Top* simulation file

<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/behavioural-simulation/set_top_tb.png" width="1000"/>


* Start the integrated Vivado behavioural simulator by clicking on *SIMULATION --> Run Simulation --> Run Behavioral Simulation*. 

<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/behavioural-simulation/start_sim_tb.png" width="1000"/>

* Vivado simulator will generate a simulation up to 1us (default setting), where the first 100ns are:

<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/behavioural-simulation/run_sim_tb.png" width="1000"/>


## 2. TCL stimulus

* Since we only want to simulate the [counter.vhd](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/counter.vhd) or [counter.v](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/counter.vhd) module, we have to make sure that it is set as the *Top* simulation file. 

<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/behavioural-simulation/set_top.png" width="1000"/>

* Start the integrated Vivado behavioural simulator by clicking on *SIMULATION --> Run Simulation --> Run Behavioral Simulation*. 

<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/behavioural-simulation/start_sim_tcl.png" width="1000"/>

* Execute the [sim_counter.tcl](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/sim/sim_counter.tcl) that is driving your simulation by writing into the TCL console (bottom panel of your Vivado window):

```
source <absolute-path-to-FPGA-Notes-for-Scientists>/sim/sim_counter.tcl
```

<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/behavioural-simulation/run_sim.png" width="1000"/>

## Next steps
See assignment on [PWM](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/wiki/Assignments#2-pwm-digital-io--simulation).




