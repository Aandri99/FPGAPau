
***

:warning: The instructions below only apply to registered participants of the FPGA training course with remote AWS development servers. Please disregard them if you are setting up a local development environment.

***

## Access FPGA Server Control 
* Navigate to [http://18.100.25.145:8000/]
* Introduce your user credentials (tag & password)
* Select *Get Server State* and hit *Submit*
* Inspect the response and keep the IP address (you will need it in the following section)
* Use the *Start Server* and *Stop Server* options to start or stop your remote server

## Configure Redpitaya
* Download the preconfigured SD card image from [Pynq-Redpitaya-125-14-3.0.1-preconfig.img](https://drive.google.com/file/d/12Qe6CwB-lKOjUyjOQ-NQwc00YPkJRAwn/view?usp=sharing)
* Flash the SD card (min 16GB). Instructions for writing SD card images can be found in [pynq-doc](https://pynq.readthedocs.io/en/v3.0.0/appendix/sdcard.html) and [redpitaya-doc](https://redpitaya.readthedocs.io/en/latest/quickStart/SDcard/SDcard.html).
* Once SD card has been flashed, insert it into your PC and open the PYNQ partition. Edit the *boot.py* file and put your server's IP address into line 4:
   ```
   server_ip = "X.X.X.X"   #CHANGE TO SERVER IP
   ```

 