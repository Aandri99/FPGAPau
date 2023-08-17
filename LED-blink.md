## About this tutorial 
:stopwatch: 45 min

**Key learnings:**

* Create a new Vivado project & block design

* Use the AXI GPIO to write data from PS to PL

* Write custom HDL modules

* Define board constraints

* Create a simple Jupyter Notebook to control the FPGA design

**Description**:

In this tutorial we are going to demonstrate how to blink the LEDs of your Redpitaya-125-14. The LEDs will be driven by the most-significant-bit (MSB) of a counter that continuously overflows. The counter is implemented within the Programable Logic (PL) and controlled through a simple PYQN Jupyter Notebook running on the Processing System (PS). 


<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/LED_blink.gif" width="400"/>

## Building the Vivado Design
### Create a new Vivado Project
* Open Vivado and click on *Create Project* and hit _Next_.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/startVivado.PNG" width="450"/>

* Set the name of the new project (e.g. _LED_blink_) and hit _Next_.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/newProject.PNG" width="450"/>

* Use the default project type (_RTL Project_) and hit _Next_.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/projectType.PNG" width="450"/>

* Leave the _Add Sources_ screen empty and hit _Next_.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/addSources.PNG" width="450"/>

* Use *Add File* to include the constraints file within [FPGA-Notes-for-Scientists/sdc/redpitaya-125-14.xdc](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/sdc/redpitaya-125-14.xdc) and hit _Next_.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/addConstraints.PNG" width="450"/>

* Under the *Boards* tab select Redpitaya-125-14 (see [here](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/wiki/Setting-up-your-system#Install-the-Redpitaya-125-14-board-files) for instructions to install the Redpitaya-125-14 board files) and hit *Next*.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/defaultPart.PNG" width="450"/>

* After finishing this initial configurations, a new Vivado Project is created. You will see the *Project Manager* page:
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/projectSummary.PNG" width="1000"/>

### Create a new Block Design

* Create a new block design by clicking on *Create Block Design* (left panel). The design name will default to *design_1*.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/createBlockDesign.PNG" width="1000"/>

* Right-click on the blank design and select *Add IP* to instantiate:
    * ZYNQ 7 Processing System
    * AXI GPIO (we will use this IP to write from PS to PL)
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/zynqGpio_1.PNG" width="1000"/>

* Click on *Run Block Automation* (green field above your design) to route the DDR and FIXED_IO ports of the ZYNQ instance. To this end, leave the configuration window in its default state.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/blockAutomation.PNG" width="450"/>
  
* Click on *Run Connection Automation* (green field above your design) to create the required clocking, reset and AXI interconnect infrastructure. To this end, within the configuration window only select *S_AXI*. We do not select *GPIO*, because we will manually configure & connect the AXI GPIO.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/connectionAutomation.PNG" width="450"/>

* After running block & connection automation, the design becomes:
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/zynqGpio_2.PNG" width="1000"/>

* Double-click on the AXI GPIO instance and enable Channel 1 and Channel 2 as outputs of 1 and 32 bits, respectively. Later in the design process, we will use Channel 1 to reset the counter logic and Channel 2 to define the counter increment value. 
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/axiGpio.PNG" width="450"/>

* Double-click on the ZYNQ instance and verify (under *Clock Configuration --> PL Fabric Clocks*) that *FCLK_CLK0* is enabled and set to 50 MHz. 
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/zynqPLClk.PNG" width="450"/>

### Create an HDL counter
***

:information_source: Two main Hardware Description Languages (HDLs) exist: VHDL  and Verilog. There is not a good answer to which language is *better* and it typically ends up being a matter of taste and your *environment* (colleagues, language of previously developed codes ...), that drives you to develop in one ore another. Vivado supports both languages and for this reason also all example codes of this tutorial come in pairs of *.vhd* and *.v* files. 

* The next step is to create a counter logic with variable counter increment. Click on *Add Sources* (left panel) and select *Add or create design sources*.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/addSourceVivado.PNG" width="450"/>

* At this point you can either select and already existing HDL file or create a new one. We will follow the latter.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/newSource.PNG" width="450"/>

* Define the module ports (they can also be changed later, within the HDL code).
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/defineModule.PNG" width="350"/>

* The new source file (*counter.vhd* or *counter.v*) will be added to the project source tree. 
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/counterVhd.PNG" width="1000"/>

* Edit *counter.vhd* or *counter.v* to include the counter logic (see [VHDL](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/counter.vhd) and [Verilog](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/counter.v) working examples).

* Add the HDL counter module to your design. To this end, right-click on the design and select *Add Module*.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/addModule.PNG" width="220"/>



### Constraints and signal routing
* Open the constraints file (*redpitaya-125-14.xdc*) and uncomment lines 166-177, which define and configure the FPGA ports connected to the LEDs.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/ledSdc.PNG" width="1000"/>

* Return to your design, right-click and select *Create Port*. Name the port *led_o* and configure it as shown below:
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/createPort.PNG" width="300"/>

* Now we need to connect the MSB of the HDL counter to the output port. For this purpose, instantiate:
   * Slice IP
   * Concat IP

* Configure the Slice IP to accept a 32 bit wide input and return the MSB.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/slice.PNG" width="450"/>

* Configure the Concat IP to bundle 8 inputs (1 bit wide).
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/concat.PNG" width="450"/>

* Wire up the AXI GPIO, HDL counter, Slice IP, Concat IP and *led_o* port as shown below:
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/fullDesign.PNG" width="1000"/>

### Bitstream generation

* Before bitstream generation, a HDL design wrapper needs to be created, which is an automatically generated HDL code that represents your graphical design.

    Within your project source tree, right-click on top of your design  and select *Create HDL Wrapper* (use default settings).
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/hdlWrapper.PNG" width="1000"/>


* Click on *Generate Bitstream* (left panel) and proceed with the default settings. This will automatically take you through:
     * **Synthesis**: translates the HDL wrapper (i.e. your custom FPGA design) into logical elements such as flip-flops, LUTs...
     * **Implementation**: places and routes the logical elements of the synthesized design into the particular chip architecture.
     * **Bitstream generation**: creates a binary image that contains the implemented design.

* After a few minutes, the process completes. Press *Cancel* to close the pop-up window.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/LED-blink/bitstreamCompleted.PNG" width="250"/>

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
Overlay "LED_blink" successfully uploaded to: 
xilinx@<static-ip-address>:/home/xilinx/pynq/overlays/LED_blink
```

* Open your preferred web browser and navigate to *\<static-ip-address\>* and enter the PYNQ Jupyter Notebook environment (password: *Xilinx*).

* Create a new Python 3 Jupyter Notebook.

* Open the Jupyter Notebook and edit it as shown in [FPGA-Notes-for-Scientists/jupyter_notebooks/LED_blink.ipynb](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/jupyter_notebooks/LED_blink.ipynb). 

* Run the Jupyter notebook. 

## Next steps
See assignments on [FPGA Pendulum Waves](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/wiki/Assignments#1-fpga-pendulum-waves-digital-io) and [PWM](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/wiki/Assignments#2-pwm-digital-io--simulation).

 

