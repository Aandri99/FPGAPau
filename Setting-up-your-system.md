##  Prepare your PC
###  System Requirements
The system requirements are detailed within the Vivado User Guide UG973. To summarize, you are typically fine with:
* OS: Windows or Linux
* ~ 8GB RAM
* ~ 150GB free storage 

###  Install Vivado WebPACK
I recommend to follow the step-by-step instructions within the [Digilent Reference Documentation](https://reference.digilentinc.com/vivado/installing-vivado/start). 

###  Clone this repository
```bash
git clone https://github.com/dspsandbox/FPGA-Notes-for-Scientists
```

###  Install the Redpitaya-125-14 board files
To be able to generate your custom FPGA designs, it is recommended to use a board files that pre-configures your Vivado design environment. To date, no official board files for the Redpitaya-125-14 exist. For this reason, I created a custom board file that includes the most important settings of the Redpitaya-125-14. To install them, you have to:
* Navigate to [FPGA-Notes-for-Scientists/board_files](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/board_files) and copy the *redpitaya-125-14* folder.
* Navigate to *\<Vivado-Install-Path\>/data/boards/board_files* and paste the *redpitaya-125-14* folder. 

## Prepare your Redpitaya-125-14
###  SD card image
* Download [Pynq-Redpitaya-125-14-3.0.1.img](https://drive.google.com/file/d/1yZ7dBUzU9IY8ECg2v9Jc2SXsewsgbF-1/view?usp=sharing). For more information on how the PYNQ image was built see [here](https://github.com/dspsandbox/Pynq-Redpitaya-125).
* Write the PYNQ image onto your SD card. You can find detailed instructions [here](https://pynq.readthedocs.io/en/v3.0.0/appendix/sdcard.html).

### Power up the Redpitaya-125-14
* Insert the SD card into your Redpitaya-125-14.
* Connect the Redpitaya-125-14 to your ethernet switch/router.
* The Redpitaya-125-14 has two micro usb connectors label as **CON** and **PWR**:
   * Connect the **CON** port to your PC and access it via a serial terminal (see details [here](https://pynq.readthedocs.io/en/v2.0/getting_started.html#opening-a-usb-serial-terminal)).
   * Connect the **PWR** port to your PC or external power supply. The Redpitaya-125-14 will immediately boot and you can follow the boot sequence over the serial terminal.

### Assign a static IP address
* Use the **CON** terminal to enter the network configurations (password: *xilinx*):
```bash
sudo vi /etc/network/interfaces.d/eth0
```

* Press *a* and edit the file (*\<static-ip-address\>* and *\<net-mask\>* are placeholders, replace them with the corresponding static IP address of your choice and the netmask of your local network):
```bash
auto eth0
iface eth0 inet dhcp

auto eth0:1
iface eth0:1 inet static
address <static-ip-address>
netmask <net-mask>
```
* Press *Esc* followed by *:wq* and *Enter*, which will save the modified interface configurations.

* Reboot your Redpitaya-125-14:
```bash
sudo reboot
```

* Open a separate CMD / Terminal and verify that you can SSH into your Redpitaya-125-14 (password: *xilinx*):
```bash
ssh xilinx@<static-ip-address>
```

###  Verify PYNQ Server
* Open your web browser and navigate to \<static-ip-address\>. You should be prompted to the PYNQ Jupyter Notebook welcome page (password: *xilinx*).
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/Setting-up-your-system/welcome.png" width="1200"/>

* (OPTIONAL) The PYNQ image also includes a Samba file sharing service, which allows to access the file system as if it would be a remote drive. This is very handy to transfer files between your PC and the Redpitaya-125-14. The configuration process is described [here](https://pynq.readthedocs.io/en/v2.0/getting_started.html#accessing-files-on-the-board). Do not worry if your user permissions our your network settings do not allow you to access the Samba service. File transfers can also be done via SCP.

##  Speed up the design flow
### Define a RSA key pair
We will generate an RSA key pair that will prevent you from using the *xilinx* password in your SSH and SCP commands:
* Open a command line terminal (for Windows see :warning:). 
* Generate the key pair:
```bash
ssh-keygen -t rsa
```
* You can proceed with the default configurations and hit *Enter*.
* Upload the the generated key pair to the Redpitaya-125-14 (password: *xilinx*).
```bash
ssh-copy-id xilinx@<static-ip-address>
```
* Verify that you can use SSH without having to provide a password.
```bash
ssh xilinx@<static-ip-address>
```

***

:warning: The *ssh-copy-id* command is not supported within the regular Windows CMD, use instead the **Git Bash Terminal** or the **Linux Subsystem Terminal** for all the steps of this section. Alternatively, you can use the Windows Powershell to set up the RSA key sharing (see [here](https://www.chrisjhart.com/Windows-10-ssh-copy-id/)).

***

###  Create a Vivado shortcut to upload overlays
Overlays are file bundles created around a custom FPGA image, which include the generated bitstream and *hardware_handoff* files to provide information on the instantiated IPs, memory interfaces, etc. These files are typically manually collected out of the Vivado project, renamed and uploaded to the FPGA. The process is simple but require a few minutes of your time. I have created a simple TCL script that fully automates this process and which can be launched via a shortcut in the Vivado IDE. To set this up, you need to:
* Open Vivado.
* Open the upper toolbar and go to *Tools --> Custom Commands --> Customize Commands*.
* Add a new command that executes *\<FPGA-Notes-for-Scientists\>/tcl/upload_overlay.tcl*.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/Setting-up-your-system/customTclCommand.PNG" width="600"/>

* After pressing *OK*, a new <img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/Setting-up-your-system/tclButton.png" width="15"/> button appears on the Vivado toolbar. We will use it after bitstream creation to automatically construct the corresponding overlay and upload it to the Redpitaya-125-14.

* Close Vivado.

* Open [FPGA-Notes-for-Scientists/tcl/upload_overlay.tcl](https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/tcl/upload_overlay.tcl) and insert the static IP address of your Redpitaya-125-14:

```tcl
#############################
# PARAMETERS
#############################
set ip 192.168.1.99 # Replace with <static-ip-address>
set user xilinx
#############################
cd [get_property DIRECTORY [current_project]]
set projName [get_property NAME [current_project]]
# ...
```

* Save and close the file.

## Known Issues
* Vivado 2020.X does not re-open existing project. See solution [here](https://support.xilinx.com/s/question/0D52E00006hprvBSAQ/vivado-20211-stuck-initializing-language-server?language=en_US)





