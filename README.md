# Interactivity and Sensor/Effector integration
A set of scripts to integrate streaming data with LXStudio over OSC for controlling the Tree of Tenere


This project aims to provide a reference system and suite of sensor integrations for TENERE,  written mostly in Python.  The initial platform involves the use of a Raspberry Pi (3 model B - tested) to receive streaming raw data from various sensors, including:

Works on PC/Mac or Raspberry Pi
* Muse headband (https://choosemuse.com, Bluetooth LE)
* USB microphone


Raspberry Pi specific parts list:
* Heartbeat sensor (http://pulsesensor.com, analog input)
* Grove expansion shield (https://www.dexterindustries.com/grovepi/, I2C)
* Pimoroni Blinkt (https://shop.pimoroni.com/products/blinkt, SPI) - equivalent to 1 of Tenere's leaves
* Grove accelerometer (http://wiki.seeed.cc/Grove-3-Axis_Digital_Accelerometer-16g/, I2C)
* Grove oled display (http://wiki.seeed.cc/Grove-OLED_Display_0.96inch/, I2C)
* Grove I2C Hub (http://wiki.seeed.cc/Grove-I2C_Hub/)
* Grove Button (https://www.seeedstudio.com/Grove-Button-p-766.html)
* USB Microphone (https://kinobo.co.uk/)


For those that would just like to Get 'er Done, Here is an Amazon wish list (Please make sure to change the Filter on the list to show both purchased and unpurchased items): http://a.co/fIXIPa8


![TenerePi](/Images/tenere-raspberrypi-reference.png)


# Sensor Integration
The following outlines the installation process for a Raspberry Pi 3 using the latest version of the Raspian image (July 2017):

## Setting up the Raspberry Pi

* Start by following the installation instructions for downloading and writing the raspian image to a SD Card: https://www.raspberrypi.org/documentation/installation/installing-images/
* The Raspian Jessie with Deskop image has been tested: https://www.raspberrypi.org/downloads/raspbian/
* For Tenere, we suggest first configuring various options.  Be sure to turn on I2C, SPI, set your locale, keyboard layout, setup network, etc.  Please follow the relevant guides here: https://www.raspberrypi.org/documentation/configuration/
* Don't forget to change the default password to something more secure (use the `raspi-conf` tool)!!!
* Now, update all of the base packages and restart:

```
sudo apt-get update
sudo apt-get -y upgrade
sudo apt-get -y dist-upgrade
sudo apt-get -y install fail2ban
sudo shutdown -r now
```

From your home directory (`/home/pi`), let's create a directory to hold all of our software:
```
cd
mkdir SOFTWARE
cd SOFTWARE
```

At the end of the tutorial, you should have a directory structure that looks something like this:
```
pi@raspberrypi:~/SOFTWARE $ ls
GrovePI
grovepi-zero
liblsl
Interactivity
Pimoroni
Tenere
pi@raspberrypi:~/SOFTWARE $
```

* Next, let's install the libraries we are going to use and clone any additional repositories (you may not need all of these for your specific setup, this tutorial includes everything for our reference system):

```
sudo apt-get -y install vim nano git git-core cmake python-pip python-dev
```

## Raspberry Pi accessories

* Install relevant libraries to enable the Blinkt LED strip
```
sudo apt-get -y install python-rpi.gpio python3-rpi.gpio
sudo apt-get -y install python-psutil python3-psutil python-tweepy
sudo apt-get -y install pimoroni python-blinkt python3-blinkt
cd ~/SOFTWARE
mkdir Pimoroni
cd Pimoroni
git clone https://github.com/pimoroni/blinkt.git
cd library
sudo python setup.py install
```

* Install relevant libraries for the Grove Pi expansion board (https://www.dexterindustries.com/GrovePi/get-started-with-the-grovepi/setting-software/)
```
sudo apt-get -y install libi2c-dev python-serial i2c-tools python-smbus python3-smbus arduino minicom
cd ~/SOFTWARE
git clone https://github.com/DexterInd/GrovePi.git
cd GrovePi/Script
sudo chmod +x install.sh
sudo ./install.sh
pip install grovepi
cd ~/SOFTWARE
git clone https://github.com/initialstate/grovepi-zero.git
```

## Voice control with Jasper
```
cd ~/SOFTWARE/Interactivity/voicecontrol
```
Please follow the instructions at https://github.com/treeoftenere/Interactivity/tree/master/voicecontrol


## Pulse Sensor and other peripherals with the TenerePi
```
cd ~/SOFTWARE/Interactivity/sensors
```
Please follow the instructions at https://github.com/treeoftenere/Interactivity/tree/master/sensors





## Muse headband

* For Muse Integration, several libraries are required.  Please note, this setup is only valid for the Muse 2016 (or later) versions:
```
sudo apt-get -y install python-liblo python-matplotlib python-numpy python-scipy python3-scipi python-seaborn liblo-tools
pip install pygatt
pip install bitstring
pip install pexpect
```


Now turn on your Muse and let's figure out its network address
```
sudo hcitool lescan
```

You should see something like (please write down the hex address as we will use it later to connect):
```
00:55:DA:BO:0B:61 Muse-OB61
```

Now let's get the Muse talking to LXStudio.  This assumes you have the latest version of LXStudio running somewhere on your local network.  That is, we can test the Muse with LXStudio running on the same computer as the Muse is connecting.  However, our preference is to have the Muse stream data to the Raspberry Pi and then have the Pi send this data over a network to a show control computer (typically a Mac or PC) dedicated to running LXStudio.

To do this, first grab the latest version of Processing and install for your desired platform (https://processing.org/download/)

Then clone the lastest version of LXStudio (see more at: https://github.com/treeoftenere/Tenere)
```
git clone https://github.com/treeoftenere/Tenere.git
```

To get data from the Muse, we first use the Lab Streaming Layer library (previously installed, https://github.com/sccn/labstreaminglayer) to connect to the Muse over Bluetooth LE.  We then have a script that reads the streaming messages from LSL and then converts them to a format appropriate for OSC (http://opensoundcontrol.org/).  The `liblo` python package then takes care of streaming this newly processing sensor stream in OSC format to our show computer running LXStudio.

To test, let's clone this repository and launch our sensor processing pipeline (a big shout-out to @brainwaves for creating this):
```
cd ~/SOFTWARE
git clone https://github.com/treeoftenere/Interactivity
cd Interactivity
cd muse-sock
```

Now using the address we discovered previously, start the script that connects to the Muse (replace `00:55:DA:BO:0B:61` with the address of your Muse Headband):
```
python muse-sock.py --address 00:55:DA:BO:0B:61
```

Then in a second terminal, start our script for OSC streaming to LXStudio (replace `192.168.0.50` with the IP address of the machine where you are running Tenere's LXStudio:
```
cd ~/SOFTWARE/Interactivity/muse-sock
python muse-listener.py --oscip 192.168.0.50
```

Congratulations, you are now controlling Tenere with your brainwaves!!!!


![Tenere_Muse_LXStudio](/Images/Tenere_Muse-EEG_LXStudio.png)

### Setup raspberry pi to connect to a muse on boot
```
apt-get install tmux
```
Now edit `/etc/rc.local` and add the following line before (`exit 0`):

```
sudo -u pi bash /home/pi/SOFTWARE/Interactivity/tmux_start.sh
```

`tmux_start.sh` (In: https://github.com/treeoftenere/Interactivity) looks like this:
```
$ cat tmux_start.sh 
#!/bin/bash
tmux new-session -d -s "musesock" "/home/pi/SOFTWARE/Interactivity/muse-sock/muse-reconnect 00:55:DA:B0:32:B1 10.0.0.2 9999"
```
Replace `00:55:DA:B0:32:B1` with the MAC address of the muse, replace `10.0.0.2` with the IP address of the machine where you are running Tenere's LXStudio, and replace `9999` with the port that `muse-listener.py` is listening to (it is `9999` by default). 

You should now be able to reboot the PI and have it connect to your muse on boot,  and auto-reconnect

If you want to check that the process is running, you can ssh into the Pi from another computer and attach to the tmux session

```
Starfox:~ chris$ ssh pi@10.0.0.12

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Aug  6 08:21:22 2017 from 10.0.0.2
pi@raspberrypi:~ $ tmux attach
```
If you have a muse connected,  you tmux session will have debug output like this:
```
('waited: 0.031495', 'dataloss: 0.0', 'avgloss: 0.000000')
('waited: 0.001427', 'dataloss: 0.0', 'avgloss: 0.000000')
[musesock] 0:bash*                        "raspberrypi" 15:43 07-Aug-17
```

Detach from the tmux session by pressing `^b` then `d`

### Setting up multiple Muses
Connect Multiple muses to Tenere by setting up one Raspberry Pi per muse. 
#### On each Raspberry Pi
Edit `tmux_start.sh` for each of the Pi's to connect to a different muse,  and send to different Ports on the Tenere LXstudio machine.   For example, `Pi[0]` would send to port `9910` on LX machine, `Pi[1]` would send to `9911`, and so on.

#### On the Tenere LX studio host computer
Run in separate terminal sessions:
For `Pi[0]`:
```
python muse-listener.py --port 9910 --oscip 127.0.0.1 --oscport 7810
```
For `Pi[1]`:
```
python muse-listener.py --port 9911 --oscip 127.0.0.1 --oscport 7811
```
And so on...

### Tenere LXstudio
Tenere LXstudio will listen for Muse inputs with a different port (sequentially numbered) for each muse.  One port per muse. 
(`commit b0cd179031c5277de0cb7bf161bf4b4e2f530473`)

Configure Tenere LXstudio to Listen for Multiple Muse inputs
Looks at Lines `36:41` in `Tenere.pde`
```
//Muliple Muses
//each muse sends to a differnt port numbered sequentially starting with musePortOffset
Muse[] muse;
UIMuse[] uiMuse;
int num_muses = 3;
int musePortOffset=7810;
```

Now when you run Tenere.pde,  3 muses will appear as a collapsible section undernead `Sensors`.   The Muses are Named with the Port associated with them.  

There are three sets of sliders for each Muse:  
* The first four are the correspond to the 4 EEG sensors of the Muse (ordered as:  Back Left, Front Left, Front Right, Back Right).  This is Raw EEG data updated at 256Hz, scaled to be (0-1) .  Its very hard to use Raw EEG,  Signal procesing for these will be added soon.   
* The next three are Accelerometer,  one for each axis.  Updated at 50Hz, scaled to (0,1)
* The last three are Gyroscope,  one for eah axis.  Updated at 50Hz, scaled (0,1)

(`default.lxp` in `commit b0cd179031c5277de0cb7bf161bf4b4e2f530473` will show the muse gyro output from 3 muses on the Tree)

### Visual feedback of connected muse using Pimironi Blinkt on the Pi

Install liblo and pylibo to send and receive OSC messages on the Pi

```
sudo apt-get install liblo-dev cython
sudo pip install pyliblo
```

Mount the Blinkt to the PI, and run:
```
python museStatus.py
```
Now in another terminal session on the Pi, run the muse connection script:
```
./muse-reconnect-status 00:55:DA:B0:32:B1 192.168.1.118 9999
```


### MuseLSL
LSL (Lab Streaming Layer) is a standard for EEG research.  
Follow these instrucations to get muselsl working on the Pi
so that you can connect to a muse and send the data out using LSL  
```
sudo pip install pylsl
```

Unfortunately, there is an error in the latest pylsl package that distributes a library that is compiled for the wrong architecture.  We can fix this with the following:

```
cd ~/SOFTWARE
mkdir liblsl
cd liblsl
wget http://sccn.ucsd.edu/pub/software/LSL/SDK/liblsl-C-C++-1.11.zip
unzip liblsl-C-C++-1.11.zip
sudo cp liblsl-bcm2708.so /usr/local/lib/python2.7/dist-packages/pylsl-1.10.5-py2.7.egg/pylsl/liblsl32.so
```

Get muselsl from https://github.com/alexandrebarachant/muse-lsl and follow alexandre's instructions.


## Muse headband signal processing
Signal processing scripts that receive muse data and create useful signals to use in Tenere LXstudio

Check out: 
[/muse-sigproc/README.md](/muse-sigproc/README.md)

