## 1. FPGA Pendulum Waves (Digital IO)
We are going to implement a digital version of the beautiful [Pendulum Waves Experiment](https://www.youtube.com/watch?v=yVkdfJ9PkRQ&t=0s&ab_channel=HarvardNaturalSciencesLectureDemonstrations).

1. Extend the [LED_blink](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/wiki/LED-blink) example to use 8 independent counters for every LED of your Redpitaya-125-14 with independently controllable counter **incr** but common **resetn**. 
 
---
ℹ️ The AXI GPIO IP features only 2 register outputs. For more register outputs please use multiple instances of this IP or replace it by an [AXI4 Register Bank](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/tree/main/ip/AXI4-register-bank) IP instance, which has up to 16 independent register inputs/outputs. See [here](analog-echo#import-the-redpitaya-125-14-ip-library) for instruction to import the corresponding IP library.

---

2. Extend the [LED_blink.ipynb](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/jupyter_notebooks/LED_blink.ipynb) Jupyter Notebook to configure LED frequencies of the type:

    **f[i] = f0 + i * df**

    for **i=0,1...,7** and where **f0** is the base frequency and **df** the frequency increment.


## 2. PWM (Digital IO & Simulation)
In this assignment we are going to develop a [Pulse-Width Modulation (PWM)](https://en.wikipedia.org/wiki/Pulse-width_modulation) module. 

1. Based on the [counter.vhd](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/counter.vhd), create a new VHDL module that has the following ports

|Name|In/Out | Width (bits) |Description |
|---|---|---|---|
|clk|in|1|System clock|
|resetn|in|1|System reset (active low)|
|incr_i|in|32|Internal counter increment per clock cycle| 
|duty_cycle_i|in|32|Internal counter threshold below (above) which *pwm_o = 1* (*pwm_o = 0*) |
|pwm_o|out|1|Pulse-width modulation output| 
  
2. Simulate and validate the new VHDL module (follow the instructions in [Behavioral Simulation](Behavioral-simulation)).

3. Connect the PWM module to the Redpitaya LEDs and verify that you can use it to dim the LED power.

4. Connect the PWM module to the *pwm_o* ports (requires to uncomment *PWM DAC* section in [redpitaya-125-14.xdc](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/sdc/redpitaya-125-14.xdc)) and verify on an oscilloscope that the you can use PWM in combination with the integrated low-pass filter to generate an analogue output.

**HINT** : If you have problems creating your own PWM module, see [pwm.vhd](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/assignments/pwm.vhd).

## 3. Axi Stream Waveform Generator (Analog IO)
The AXI Stream interface is the golden standard for **continuous data flows**. A comprehensive introduction to this interface type can be found [here](https://lauri.xn--vsandi-pxa.com/hdl/zynq/axi-stream.html).

The most commonly used signals for an AXI stream interface are:

|Signal name| Direction | Optional| Description|
|---|---|---|---|
|<interface_name>_tvalid| Master --> Slave | No | Indicates if the current data are valid. In many application this signal can be tied to 1.|
|<interface_name>_tdata| Master --> Slave | Yes| Contains the actual data to transmit. |
|<interface_name>_tlast| Master --> Slave | Yes (defaults to 1 if not present)| Indicates if the current sample is the last one to transmit. Typically only need within DMA transfers|
|<interface_name>_tready| Slave --> Master| Yes (defaults to 1 if not present)| Indicates if the slave interface is ready to accept new data.|

---
ℹ️ A data transaction is only performed when both *_tvalid* and *_tready* are asserted on the same clock cycle. As a result, this two signals can be used to throttle the data transaction, i.e. pausing the slave (via *_tvalid = 0*) until data are ready or pausing the master (via *_tready = 0*) until the slave is available for processing new data.

---


In this assignment we are going to convert the previously used counter and PWM modules and make their outputs AXI4-Stream compliant. You will be able to directly connect them to the DAC of your Redpitaya and create your own saw-tooth and square waveforms.

1. Open the [Analog-Echo](analog-echo) design example

1. Create a new VHDL module based on [counter.vhd](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/counter.vhd) where the 16 most-significant-bits (MSB) are connected to a *<interface_name>_tdata* output. 

2. Create a *<interface_name>_tvalid* output that is always 1.

3. Connect the generated module to the DAC of your Redpitaya and observe on an oscilloscope the generated analog waveform. You should observe a saw-tooth waveform.

4. Repeat steps (1)-(3) for the PWM module. Note that you will have to slightly change the output logic, since *pwm_o = 1* and *pwm_o = 0* should be mapped to *<interface_name>_tdata = "01111111111111"* (binary representation of max 16-bit integer) and *<interface_name>_tdata = "1000000000000000"* (binary representation of min 16-bit integer). Once connected to the DAC of your Redpitaya, you should see square waveforms on your oscilloscope.

**HINT** : If you have problems creating your own axis stream saw-tooth module, see [test_waveform.vhd](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/assignments/test_wave.vhd).


## 4. DDS (Analog IO)
In this assignment we are going to use the [Vivado DDS compiler](https://docs.xilinx.com/v/u/en-US/pg141-dds-compiler) to generate configurable sin/cos waveforms.

1. Open the [Analog-Echo](analog-echo) design example.

2. Click on *Add Sources* (left control panel) and import:
   * [FPGA-Notes-for-Scientists/hdl/assignments/dds/dds_compiler_0.xci](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/assignments/dds/dds_compiler_0.xci)
   * [FPGA-Notes-for-Scientists/hdl/assignments/dds/dds_wrapper.vhd](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/assignments/dds/dds_wrapper.vhd)

3. Remove the connection from ADC to DAC and insert the *dds_wrapper.vhd* with the *cos* and *sin* outputs connected to *dac_data_1* and *dac_data_2*, respectively.

4. Connect the *phase_incr*, *phase_off* and *phase_clr* to the outputs of a configurable  [AXI4 Register Bank](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/tree/main/ip/AXI4-register-bank) (part of the FPGA-Notes-for-Scientists IP library).

5. Create HDL wrapper and generate bitstream.

6. Create a Jupyter notebook to control the DDS:
   * **phase_inc = (frequency / 125 MHz) * 2^32**
   * **phase_off = (phase_offset / 360 degree) * 2^32** 

7. Verify the output on your oscilloscope

8. Extend the FPGA design to include [amplitude_ctrl.vhd](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/amplitude_ctrl.vhd) module before between the DDS and the DAC. The amplitude can be set via the AXI4 Register Bank.

9. Re-create HDL wrapper, generate bitstream and verify that you can also control the amplitude of the outputs.

10. DDS Multitone generation

10.1. Create 4 separate hierarchies containing:
   * AXI4 Register Banks for *phase_inc*, *phase_off* and *amplitude*. 
   * DDS wrapper
   * Amplitude control

10.2. Use a separate AXI4 Register Bank to provide a common *phase_clr* to all hierarchies.

10.3. Add the 4 *sin* outputs via a cascade of [stream_adder.vhd](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/hdl/stream_adder.vhd). Leave the 4 *cos* outputs disconnected. 

10.4 Connect the output of the cascade to *dac_data_1* and disable *dac_data_2* (double click on the DAC IP and uncheck *Ch2 Enabled*).

10.5 Re-create HDL wrapper, generate bitstream and verify that you are able to generate a linear combination of 4 frequency tones.

10.6. Configure the frequency, phase and amplitudes to generate Square, Triangular... waveforms. To this end, have a look at the [Table of common Fourier series](https://en.wikipedia.org/wiki/Fourier_series#Table_of_common_Fourier_series)






