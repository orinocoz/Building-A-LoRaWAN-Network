# Building-A-LoRaWAN-Network
A tutorial covering all aspects of building a LoRaWAN network including a gateway, node and server.

## Build a LoRaWAN Gateway

The LoRaWAN gateway build in this tutorial very closely follows the tutorial presented by [The Things Network Zurich](https://github.com/ttn-zh "The Things Network Zurich"), specifically, their [iC880a-gateway](https://github.com/ttn-zh/ic880a-gateway "The Things Network Zurich ic880a-gateway") repository.

### Gateway Parts

A Do-It-Yourself (DIY) approach was taken to build the LoRaWAN gateway documented in this project. This solution involved using a LoRa capable board (iC880a) and connecting it to a host computer (Raspberry Pi). The following parts were sourced to build the LoRa gateway:

| Part            | Price           | Link                                         |
|-----------------|-----------------|----------------------------------------------|
| Raspberry Pi 2  | €32.00          | Farnell Online shop                          |
| ic880a          | €155.00         | IMST Online shop                             |
| Antenna pigtail | €6.50           | IMST Online shop                             |
| Antenna         | €6.50           | IMST Online shop                             |
| Jumper cables   | N/A             | Purchase at any electronics shop             |

Details on all parts:

iC880a concentrator board:

+ [ic880a Product Information](https://wireless-solutions.de/products/long-range-radio/ic880a "iC880A - LoRaWAN Concentrator 868MHz") 
+ SPI model (not USB)
+ 868 Mhz (European LoRa standard)
+ Multiple channel LoRaWAN gateway solution

Antenna pigtail: 

+ Pigtail (cable) for antenna port of iC880A-SPI
+ U.FL to SMA female cable, length: 100 mm

Antenna: 

+ SMA-antenna for iC880A-SPI
+ To connect the SMA-antenna to the iC880A-SPI board

Raspberry Pi 2:

+ Model B Version 1.1
+ Micro SD card (4GB +)
+ Power supply also required
+ WiPi (or similar if not connecting gateway via Ethernet)

### Install Raspbian Lite on the Raspberry Pi

We need an Operating System for the Raspberry Pi. We are going to use "Raspbian Jessie Lite" - a minimal operating system based on Debian Jessie (version 8) without an X Windows system (graphical interface). This provides a lightweight operating system to run the gateway. Specific details of the Raspbian version used in this project are as follows:

+ Version: April 2017
+ Release date: 2017-04-10
+ Kernel version: 4.4
+ SHA-1: c24a4c7dd1a5957f303193fee712d0d2c0c6372d
+ [Raspbery Pi download page](https://www.raspberrypi.org/downloads/raspbian/ "Download Raspbian for Raspberry Pi") 

Linux, specifically Ubuntu 16.04-2, was used to prepare the micro SD card for the Raspberry Pi. Although instructions are provided, it is reccommended to follow the [official documented procedure](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md, "Installing Operating System Images on Linux"). However, to be thorough, the following steps were performed in this project.

The first step was to unzip the Raspbian Jessie Lite download. This was achieved with the following command:

```bash
unzip 2017-04-10-raspbian-jessie-lite.zip
```

You should now have a .img file available. We need to determine where the micro SD card is located in order to write the .img file to. Use the df command to find the micro SD card:

```bash
df -h
```

If you are having trouble identifying your micro SD card, try the following steps:

+ Unplug the micro SD card
+ Run the df -h command
+ Plug in the micro SD card
+ Run the df -h command
+ Comapre the output from both commands, and determine the new device

In the preceeding discussion, my micro SD card was identified as mmcblk0. Specifically, one partition was discovered on /dev/mmcblk0p1. This partition needs to be unmounted to write the Raspbian image to the card:

```bash
umount /dev/mmcblk0p1
```

Now write the extracted Raspbian image (.img file) to the micro SD card.

```bash
sudo dd bs=4M if=2017-04-10-raspbian-jessie-lite.img of=/dev/mmcblk0
```

Finally, flush the cache to ensure all data is written:

```bash
sync
```

NOTE: We could easily enable Secure Shell (SSH) on the Raspberry Pi now by adding a folder called "SSH" to the boot folder in the micro SD card. However, I set up SSH later using the raspi-config tool.

All done installing Raspbian Lite. Insert the micro SD card into the Raspberry Pi.

### Connecting the ic880a to the Raspberry Pi

Now that the Raspberry Pi is installed we should connect the iC880a board to the Raspberry Pi.

Firstly, connect the antenna pigtail to the iC880a board. The U.FL connector should easily "clip" into the adapter on the iC880a board. Check the beofe and after picture below for more information on antenna location.

INSERT: Photo of antenna connector, before and after

Next, attach the antenna to the SMA connector on the pigtail. The antenna pigtail we purchased from IMST is a U.FL to SMA female. 

WARNING: Never power up the device without connecting the antenna!

Lastly, we need to connect the iC880a to the Raspberry Pi using our 7 female jumper cables. The following table documents the required connections between the devices.

| iC880a pin | Description | RaspPi pin |
|:----------:|:-----------:|:----------:|
|     21     |  Supply 5V  |      2     |
|     22     |     GND     |      6     |
|     13     |    Reset    |     22     |
|     14     |   SPI CLK   |     23     |
|     15     |     MISO    |     21     |
|     16     |     MOSI    |     19     |
|     17     |     NSS     |     24     |

The photo below displays the pin connector on the iC880a and Raspberry Pi.

INSERT: Photo of labelled pins

The photo below displays the iC880a and Raspberry Pi connected.

INSERT: Photo of connected boards

We are now ready to power on the Raspberry Pi and configure our gateway.

### Configuring the Raspberry Pi

Connect the Raspberry Pi to an Ethernet cable on the local network

```bash
ssh pi@raspberrypi.local
```

or, where 192.168.1.10 is the IP of the Raspberry Pi:

```bash
ssh pi@192.168.1.10
```

The default username and password for a fresh Raspbian Lite install is:

+ username: pi
+ password: raspberry

We should change these values now (better security practices). In the following examples, the username selected was "ttn-user", however, you can select any username.

pi@raspberrypi~#: sudo adduser ttn-user

You will be prompted to enter the details of the new user including a password and full name etc.

Now, add the new user to the sudoers group so that they can run commands as using the sudo prefix.

```bash
sudo adduser ttn-user sudo
```

We must enable SPI on the Raspbian install. Additionally, we should also expand the filesystem to the entire micro SD card. To achieve both of these, we can use raspi-config utility, as described below:

```bash
sudo raspi-config
```

Select > [5] Interfacing options
Select > P4 SPI

Also, expand the file system:

Select > [7] Advanced options -> A1 Expand filesystem

You will be asked to reboot after exiting the raspi-config utility, select yes to reboot. Or manually reboot using:

```bash
sudo reboot
```

When the Raspberry Pi reboots, login with the new credentials (the username and password that you just created). Now, elete the default "pi" user (good security practice) using the following command:

```bash
sudo userdel -rf pi
```

Perform an update on the device and upgrade any packages using the following commands:

```bash
sudo apt-get update
sudo apt-get upgrade
```

Now, install the git package which allows us to download the gateway software from the git repository:

```bash
sudo apt-get install git
```

Now comes the part where we download and install the software required to operated the iC880a board, capture LoRa packets and forward the packets to a backend server. Luckily for us, there is a dedicated GitHub repository available which provides a simple script to install each software components required.

Clone the repository from The Things Network Zurich. Make sure to clone the "spi" branch:

```bash
git clone -b spi https://github.com/ttn-zh/ic880a-gateway.git ~/ic880a-gateway
```

Change to the directory where we downloaded the software:

```bash
cd ~/ic880a-gateway
```

And, finally, run the software using the following command:

```bash
sudo ./install.sh spi
```

Ok, so that was pretty simple. But what does this install script actually do? We will discuss each component below.

+ Firstly, the script updates the git repository
+ The scripts then attempts to set the gateway EUI (gateway ID) using the devices (Raspberry Pi's) MAC address
+ Then the hostname of the Raspberry Pi is set (default of "ttn-gateway")
+ WiringPi is removed (if installed)
+ All remaining software is then installed in: /opt/ttn-gateway/
+ Clones the latest "lora_gateway" software from https://github.com/TheThingsNetwork/lora_gateway.git
+ Builds the packet forwarder software from https://github.com/TheThingsNetwork/packet_forwarder.git
+ Writes symbolic links for packet forwarder and it's configuration
+ Starts the packet forwarder as a service
+ Reboots

Use the test tool:

```bash
ttn-user@ttn-gateway:~ $ sudo /opt/ttn-gateway/lora_gateway/util_tx_test/util_tx_test -f 868 -r 1257
Sending -1 packets on 868000000 Hz (BW 125 kHz, SF 10, CR 1, 16 bytes payload, 8 symbols preamble) at 14 dBm, with 1000 ms between each
INFO: concentrator started, packet can be sent
Sending packet number 1 ...OK
Sending packet number 2 ...OK
Sending packet number 3 ...OK
Sending packet number 4 ...OK
Sending packet number 5 ...OK
Sending packet number 6 ...OK
Sending packet number 7 ...OK
Sending packet number 8 ...OK
Sending packet number 9 ...OK
Sending packet number 10 ...OK
```

## Building a LoRa Node

Now that we have a LoRa gateway all setup and ready, we can now build our node to connect via the gateway. 

### LoRa Node Parts

A Do-It-Yourself (DIY) approach was taken to build the LoRa node documented in this project. This solution involved using a LoRa capable board (Dragino LoRa Shield) and connecting it to an Arduino-like board (RedBoard). The following parts were sourced to build the LoRa node:

| Part            | Price           | Link                                         |
|-----------------|-----------------|----------------------------------------------|
| RedBoard        | $19.99          | Spark Fun Online shop                        |
| LoRa Shield     | $20.00          | Tindie Online shop                           |

Details on all parts:

RedBoard:

+ [RedBoard Product Information](https://learn.sparkfun.com/tutorials/redboard-hookup-guide
 "RedBoard Hookup Guide") 
+ Very similar to Arduino Uno

Dragino LoRa Shield: 

+ [LoRa Shield Product Information](http://www.dragino.com/products/module/item/102-lora-shield.html
 "Dragino - LoRa Shield") 
+ 868 Mhz (European LoRa standard)
+ Comes with an antenna

### Setting up the Arduino IDE Environment

First we need to install and configure the Arduino IDE environment. Download the latest versions of the IDE from the [Arduino Software](https://www.arduino.cc/en/main/sofware
 "Arduino Software") web page. Specific details of the Arduino IDE used in this project are as follows:

+ Version: 1.8.2
+ SHA-1: f6aab34f33e4fab07f460d006caddb5af20df45c

In this tutorial we are using the Arduino IDE on Microsoft Windows 10. The installation is simple. Double-click the arduino-1.8.2-windows.exe file, and install using default options.

Now we need to install the Arduino LMIC library.

Navigate to the following page

https://github.com/matthijskooijman/arduino-lmic

Select "Clone or download" button.

Then select:

Download zip

Now, you can directly install the zip file content in the Arduino IDE. 

"Sketch" -> "Include Library" -> "Add .ZIP Library..."

Now we are ready to build and configure the node.

### Building and Configuring the LoRa Node

Connect the Dragino LoRa Shield to the RedBoard (or Arduino Uno). This is a simple procedure, allign the pins and gently press together.

Open Arduino IDE

Connect ardino via usb

Download sketch (.ino file)

git clone https://github.com/dragino/Lora/tree/master/Lora%20Shield/Examples/lora_shield_ttn

lora_shield_ttn.ino is the sketch

double click sketch to open

Make changes to the sketch

static const PROGMEM u1_t NWKSKEY[16] = {  };

static const u1_t PROGMEM APPSKEY[16] = {  };

static const u4_t DEVADDR = 0x00000000 ; 

Compile and upload

The default payload is "Hello World"

## Resources: IMST IC880a SPI

+ https://github.com/ttn-zh/ic880a-gateway/wiki
+ https://www.youtube.com/watch?v=ZFVA6cQyheY

## Resources: Dragino Lora Shield

+ http://www.dragino.com/products/module/item/102-lora-shield.html
+ http://wiki.dragino.com/index.php?title=Lora_Shield#Example2_--_Use_the_RadioHead_Library_With_Arduino_Boards
+ https://github.com/dragino/Lora/tree/master/Lora%20Shield
+ https://www.youtube.com/watch?v=duwUwXt-hs8
+ https://github.com/matthijskooijman/arduino-lmic