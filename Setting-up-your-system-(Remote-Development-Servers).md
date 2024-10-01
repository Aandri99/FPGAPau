
***

:warning: The instructions below only apply to registered participants of the FPGA training course with remote AWS development servers. Please disregard them if you are setting up a local development environment.

***

## Access FPGA Server Control 
* Navigate to [http://18.100.25.145:8000/](http://18.100.25.145:8000/)
* Introduce your server tag and user password (see credentials cheat sheet below)
* Select *Get Server State* and hit *Submit*
* Inspect the response and keep the IP address (you will need it in the following sections)
* Use the *Start Server* and *Stop Server* options to start or stop your remote server

## Configure Redpitaya
* Download the preconfigured SD card image from [Pynq-Redpitaya-125-14-3.0.1-preconfig.img](https://drive.google.com/file/d/12Qe6CwB-lKOjUyjOQ-NQwc00YPkJRAwn/view?usp=sharing)
* Flash the SD card (min 16GB). Instructions for writing SD card images can be found in [pynq-doc](https://pynq.readthedocs.io/en/v3.0.0/appendix/sdcard.html) and [redpitaya-doc](https://redpitaya.readthedocs.io/en/latest/quickStart/SDcard/SDcard.html#download-and-install-the-sd-card-image).
* Once SD card has been flashed, insert it into your PC and open the PYNQ partition. Edit the *boot.py* file and put your server's IP address into line 4:
   ```
   server_ip = "X.X.X.X"   #CHANGE TO SERVER IP
   ```
* Save and eject SD card
* Insert SD card into your Redpitaya (should be powered down)
* Power up your Redpitaya

## Verify Connectivity
* Open FPGA Server Control
* Start your Server (please remember to **stop you server** when unused)
* Connect via remote desktop. Protocol: RDP. Address: server IP. Credentials: see cheat sheet below
* On your remote machine, open the Firefox web browser and navigate to *10.0.0.2:9090*
* You should be prompted to the PYNQ Jupyter Notebook welcome page (password: see credentials cheat sheet below).
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/Setting-up-your-system/welcome.png" width="1200"/>

### Create and share RSA key pair
We will generate an RSA key pair that will prevent you from using passwords in your SSH and SCP commands:
* Open a command line terminal (Ubuntu Applications Menu)
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


## Create a Vivado IDE shortcut to upload overlays
Overlays are file bundles created around a custom FPGA image, which include the generated bitstream and *hardware_handoff* files to provide information on the instantiated IPs, memory interfaces, etc. These files are typically manually collected out of the Vivado project, renamed and uploaded to the FPGA. The process is simple but require a few minutes of your time. I have created a simple TCL script that fully automates this process and which can be launched via a shortcut in the Vivado IDE. To set this up, you need to:
* Open Vivado (Ubuntu Applications Menu).
* Open the upper toolbar and go to *Tools --> Custom Commands --> Customize Commands*.
* Add a new command that executes *~/Documents/FPGA-Notes-for-Scientists/tcl/upload_overlay.tcl*.
<img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/Setting-up-your-system/customTclCommand.PNG" width="600"/>

* After pressing *OK*, a new <img src="https://github.com/dspsandbox/FPGA-Notes-for-Scientists/blob/main/doc/Setting-up-your-system/tclButton.png" width="15"/> button appears on the Vivado toolbar. We will use it after bitstream creation to automatically construct the corresponding overlay and upload it to your Redpitaya-125-14.
* Close Vivado.



## Credentials Cheat Sheet

|Resource | User | Password |
|-|-|-|
| FPGA (scp, ssh, jupyter notebook)| *xilinx* | *xilinx*|
| Remote Server (RDP) | *ubuntu* | *top_secret*|
| FPGA Server Ctrl | tag sent via e-mail | pwd sent via e-mail|





 