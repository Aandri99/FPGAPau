## About this tutorial
:stopwatch: 60 min

**Key learnings:**

* Long custom waveform generation/acquisition (up to ~30 000 000 16-bit samples).

* PL (Programable Logic): use AXI DMA IP for reading/writing data from/to RAM memory.

* PS (Processing System): use PYNQ for reading/writing data from/to RAM memory.

**Description:** 

FPGAs have typically very low on-chip memory resources (~250kB for the ZYNQ-7010 of your Redpitaya-125-14). This limitation can be circumvented through one or multiple external DDR memories which are connected to the FPGA through high bandwidth ports and controlled through a Direct Memory Access (DMA) engine. The Redpitaya-125-14 includes a 512MB DDR memory, which serves as RAM for the Zynq PS but can also be used to exchange data with the Zynq PL. 

In this tutorial we are going to build a dual channel data generation/acquisition design, where the samples are directly read/written to RAM memory via DMA. A PYNQ Jupyter Notebook will be used to preload waveforms to RAM memory, simultaneously trigger the waveform generation/acquisition and plot the acquired samples.

<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/DMA-transfer/acq.png" width="1000"/>


## Building the Vivado Design
### Create a new Vivado Project
Follow the steps in [LED blink](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/wiki/LED-blink#create-a-new-vivado-project) and use *DMA_transfer* as the new project name. 

### Import the Redpitaya-125-14 IP library
Follow the steps in [Analog echo](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/wiki/Analog-echo#import-the-redpitaya-125-14-ip-library).

### Block Design
* Create a new block design.
* Create an instance of the following IPs:
    * ZYNQ7 Processing System
    * Processor System Reset
    * AXI GPIO
    * AXI Interconnect (x2)
    * AXI Direct Memory Access (x2)
    * Redpitaya-125-14-clk
    * Redpitaya-125-14-adc
    * Redpitaya-125-14-dac

* Use the *Block Automation* (green field above your design) to route the DDR and FIXED_IO ports. 
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/DMA-transfer/block_automation.PNG" width="600"/>

* Double click on the ZYNQ7 instance and go to *PS-PL Configuration --> HP Slave AXI Interface* and enable the high performance interfaces *HP0* and *HP1* (data width 64 bit).
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/DMA-transfer/zynq.PNG" width="600"/>

* Double click on the GPIO instance and configure both ports as outputs of 1 and 32 bits. 
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/DMA-transfer/gpio.PNG" width="600"/>


* (x2) Double click on the AXI Interconnect instance and configure it to have 2 Slave Interfaces and 1 Master Interface.

<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/DMA-transfer/axi_interconnect.PNG" width="600"/>

* (x2) Double click on the AXI Direct Memory Access instance and configure it as shown below:
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/DMA-transfer/axi_dma.PNG" width="600"/>

* Connect the clocks and resets. 
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/DMA-transfer/clk.PNG" width="1000"/>

### AXI stream control module

We need a simple HDL module that controls the controls the data streams going into / out of the DMA engines. The main functionality is:
* Hold the incoming/outgoing data streams until a trigger signal is received.
* Rise the AXI stream *tlast* line when the last data sample gets acquired.

To include a stream control module please follow the steps below:
* Within the left panel go to *Project Manager --> Add sources* and select  *Add or create design sources*.
* Import [stream_ctrl.vhd](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/stream_ctrl.vhd) or [stream_ctrl.v](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/stream_ctrl.v).
* (x4) Add the stream control module to your design (right-click on the design and select *Add Module*) and draw the required connections.
   * *stream_i* / *stream_o* -->  as highlighted in diagram below.
   * *clk* / *resetn* -->  similar to the already instantiated logic, they are connected to *clk_125* and *peripheral_aresetn*.
   * *trig* --> AXI GPIO Channel 1.
   * *samples* --> AXI GPIO Channel 2.

<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/DMA-transfer/stream_ctrl.PNG" width="1000"/>



### Connect the MM2S and S2MM interfaces
* We have to connect the DMA memory-mapped-to-stream (MM2S) and stream-to-memory-mapped (S2MM) to the ZYNQ HP ports via AXI Interconnect instances. To this end, draw the following connections:
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/DMA-transfer/MM2S_S2MM.PNG" width="1000"/>

### Constraints and port creation
* Edit the constraints in *redpitaya-125-14.xdc* and uncomment the sections:
  * ADC (lines 8-56)
  * DAC (lines 59-90)
  * Clock constraints (lines 180-183)

* Go back to your design and create the required input/output ports for the *Redpitaya-125-14-clk*, *Redpitaya-125-14-adc* and *Redpitaya-125-14-dac* IPs. The fastest way is to right-click on a every port and choose *Create Port*  (*CTRL + K*) and proceed with the default settings. 

* Including all the required ports, the design becomes:
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/DMA-transfer/all_ports.PNG" width="1000"/>

### Connect memory mapped interfaces

* Run *Connection Automation* (green field above your design) to interconnect the AXI GPIO, DMA and ZYNQ instances. 
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/DMA-transfer/connection_automation.PNG" width="600"/>

* The final design becomes:
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/DMA-transfer/final_design.PNG" width="1000"/>

### Address Editor
* Open the Address Editor and automatically assign an address space to all interfaces (right-click on on *Network 0* and select *Assign All*).
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/DMA-transfer/assign_address.PNG" width="1000"/>


### Bitstream generation
Please follow the steps detailed in [LED blink](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/wiki/LED-blink#bitstream-generation).


## Running the design
***
:information_source: Please make sure to complete first the steps in [Prepare your Redpitaya-125-14](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/wiki/Setting-up-your-system#prepare-your-redpitaya-125-14) and [Speed up the design flow](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/wiki/Setting-up-your-system#speed-up-the-design-flow).
***

* Verify that you Redpitaya-125-14 is connected to your local network, e.g. using ping:
```bash
ping <static-ip-address>
```

* Create and upload the *overlay* for your custom design by pressing the <img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/Setting-up-your-system/tclButton.png" width="15"/> button. The following message should appear within your Vivado Tcl console:
```bash
Overlay "DMA_transfer" successfully uploaded to: 
xilinx@<static-ip-address>:/home/xilinx/pynq/overlays/DMA_transfer
```

* Open your preferred web browser and navigate to *\<static-ip-address\>* and enter the PYNQ Jupyter Notebook environment (password: *Xilinx*).

* Create a new Python 3 Jupyter Notebook.

* Open the Jupyter Notebook and edit it as shown in [FPGA-Notes-for-Scientists/jupyter_notebooks/DMA_transfer.ipynb](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/jupyter_notebooks/DMA_transfer.ipynb). 

* Use a pair of SMB cables to connect OUT1 --> IN1 and OUT2 --> IN2.

* Run the Jupyter notebook. 

## Next steps
See assignment on [DMA & Decimation](Assignments.md#4-dma--decimation-analog-io).
