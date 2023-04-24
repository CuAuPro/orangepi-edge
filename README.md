# orangepi-edge
OrangePI IoT Edge device platform. In this case, [Orange Pi Zero2](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-Zero-2.html) is used.

# Table of contents
- [OS Installation & Preparation <a name="introduction"></a>](#os-installation-preparation)
  * [Prepare OS image](#prepare-os-image)
  * [Install OS on Orange Pi](#install-os-on-orange-pi)
  * [Setup SSH remote access](#setup-ssh-remote-access)
    + [Setup static IP address (optional)](#setup-static-ip-address)
    + [Ubuntu](#ubuntu)
    + [Windows](#windows)
    + [Add SSH Key to VS Code and Connect to a Host (optional)](#add-ssh-key-to-vs-code-and-connect-to-a-host)
  * [Additional preparation](#additional-preparation)
- [Adding GPIO support](#adding-gpio-support)
  * [Manual build](#manual-build)
    + [Prerequisites](#gpio-prerequisites)
    + [Build and install](#build-and-install)
    + [Usage](#gpio-usage)
      - [General IO](#general-io)
- [Adding I2C support](#adding-i2c-support)
  * [Preparation](#i2c-preparation)
- [Adding SPI support](#adding-spi-support)
  * [Preparation](#spi-preparation)


# OS Installation & Preparation  <a id='os-installation-preparation'></a>

## Prepare OS image <a id='prepare-os-image'></a>

1. Download OS from official [webpage](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-Zero-2.html). In this case, we use [Ubuntu Jammy Server](https://drive.google.com/drive/folders/1ohxfoxWJ0sv8yEHbrXL1Bu2RkBhuCMup).

2. Use [BalenaEtcher](https://www.balena.io/etcher) and flash image onto SD card.

## Install OS on Orange Pi <a id='install-os-on-orange-pi'></a>


Detailed instructions can be downloaded from official webpage documentation section. For Orange Pi Zero2, user guide can be downloaded from [link](https://drive.google.com/drive/folders/1ToDjWZQptABxfiRwaeYW1WzQILM5iwpb). 

Summarized workflow is:


1. Insert the SD card.

2. Connect USB Keyboard, HDMI and Ethernet cable.

3. Connect USB-C Power Supply.

4. Turn on Power Supply. Wait for boot-up.


## Setup SSH remote access <a id='setup-ssh-remote-access-pi'></a>

### Setup static IP address (optional) <a id='setup-static-ip-address'></a>

Connect to your Orange Pi via SSH or a direct connection to the console. Edit the `/etc/network/interfaces` file using a text editor such as nano or vi: `sudo nano /etc/network/interfaces`.

Add following (change values to suit your network configuration):

```
auto eth0     
iface eth0 inet static
address <IP address>
netmask <netmask>
gateway <Gateway IP>
```

Restart networking service: `sudo service networking restart`.

*If you have problems with restarting networking service, try to disable (`sudo ifconfig eth0 down`) and then enable (`sudo ifconfig eth0 down`) the interface.

### Ubuntu <a id='ubuntu'></a>

1. Get the IP address of the development board.

2. Log in to the linux system remotely through the ssh command: `ssh <user>@<IP>`. Note: `<user>` needs to be replaced with `root` or `orangepi`, `<IP>` needs to be replaced with the IP address of the development board. The default password is `orangepi`.


### Windows <a id='windows'></a>

MobaXterm can be used to remotely log in to the development board under Windows. First create a new ssh session:
1. Open Session.
2. Select SSH in Session Setting.
3. Enter the IP address of the development board in Remote host.
4. Enter the username `root`.
5. Finally click OK.
6. You will be prompted to enter a password. The default passwords for `root` and `orangepi` users are `orangepi`.

**After a successful SSH login system, you can get access to the system remotely.**
It is now recommended to reboot system.

### Add SSH Key to VS Code and Connect to a Host (optional) <a id='add-ssh-key-to-vs-code-and-connect-to-a-host'></a>


That is extremely useful feature while developing. Extended tutorial is available at [link](https://adamtheautomator.com/add-ssh-key-to-vs-code/).

Summarized workflow is:

1. Create the SSH Key.
2. Create the `.ssh` folder in your home directory on host: `sudo mkdir -p /root/.ssh`
3. Upload the id_rsa.pub file to the host into `.ssh` directory.
4. Rename the file to `authorized_keys` (lower case).
5. Open VS Code and press `Open a remote window`.
6. Select `Configure SSH Hosts`.
7. Add following (adapt your needs):
```
Host <name>
  HostName <IP>
  User <user>
```
8. Press again `Open a remote window`.
9. Select Platform of the remote host: `Linux`
10. Enter password.


## Additional preparation <a id='additional-preparation'></a>

Update `sudo apt-get update`.

Upgrade `sudo apt-get upgrade`.

Install git: `sudo apt install git`.


# Adding GPIO support <a id='adding-gpio-support'></a>

We will use [wiringOP](https://github.com/orangepi-xunlong/wiringOP) and its [Python wrapper](https://github.com/orangepi-xunlong/wiringOP-Python).

## Manual build <a id='manual-build'></a>
Clone github repository:
```
cd /opt
git clone --recursive https://github.com/orangepi-xunlong/wiringOP-Python.git
cd wiringOP-Python/
```
Repository is cloned in `/opt` because it is reserved for the installation of add-on application software packages. It can be potentially cloned in `/home` also.



### Prerequisites <a id='gpio-prerequisites'></a>
To rebuild the bindings you must first have installed `swig`, `python3-dev`, and `python3-setuptools`. wiringOP should also be installed system-wide for access to the gpio tool:

```
sudo apt-get install swig python3-dev python3-setuptools
```

### Build and install <a id='build-and-install'></a>

Generate Binding:
```
python3 generate-bindings.py > bindings.i
```

Install:
```
sudo python3 setup.py install
```

Build then wiringOP seperately (optional):

```
cd wiringOP/
sudo ./build clean
sudo ./build
```
We can look GPIO status with command `gpio readall`:
| ![](/docs/img/gpio_readall.png) |
|:--:| 
|GPIO status read.|

It is possible to get help with all commands described with `gpio -h`.

## Test GPIO <a id='test-gpio'></a>

Test output (we want to turn on and off physical pin 7, which is wPi 2 – refer to `gpio_readall` or Orange Pi Zero2 user manual):
```
gpio mode 2 out
gpio write 2 0
gpio readall
gpio write 2 1
gpio readall
gpio write 2 0
gpio readall
```


### Usage <a id='gpio-usage'></a>

```
import wiringpi

# One of the following MUST be called before using IO functions:
wiringpi.wiringPiSetup()      # For sequential pin numbering
```

#### General IO <a id='general-io'></a>

```
LED_PIN = 2 # pin
wiringpi.pinMode(LED_PIN, 1)       # Set pin LED_PIN to 1 ( OUTPUT )
wiringpi.digitalWrite(LED_PIN, 0)  # Write 1 ( HIGH ) to pin LED_PIN
out = wiringpi.digitalRead(LED_PIN)      # Read pin LED_PIN
print(out)
wiringpi.digitalWrite(LED_PIN, 1)  # Write 0 ( LOW ) to pin LED_PIN
out = wiringpi.digitalRead(LED_PIN)      # Read pin LED_PIN
print(out)
```

# Adding I2C support <a id='adding-i2c-support'></a>

## Preparation <a id='i2c-preparation'></a>

According to the schematic diagram of 26 pin, the I2C available for Orange Pi Zero 2 is `i2c3`, that is disabled by default and
needs to be manually opened before it can be used.
```
sudo nano /boot/orangepiEnv.txt
```

Add following:
```
overlays=i2c3
```

After starting the linux system, first confirm that there is an `i2c3` device node under `/dev`: 
```
ls /dev/i2c-*
```

To use I2C from the command line, install the "i2c-tools" package using the following command:
```
sudo apt-get install i2c-tools
```

Detect I2C devices connected to the bus
```
sudo i2cdetect -y 3 # for I2C3
```

# Adding SPI support <a id='adding-spi-support'></a>



## Preparation <a id='spi-preparation'></a>

`spi1` is turned off by default and needs to be manually turned on to use it.
```
sudo nano /boot/orangepiEnv.txt
```

Add following:
```
overlays=spi-spidev
param_spidev_spi_bus=1
param_spidev_spi_cs=1
```
*If some parameter already exists, just add value with space, eg.:
```
overlays=i2c3 spi-spidev
```

After starting the linux system, first confirm that there is an `spidev` device node under `/dev`: 
```
ls /dev/spidev1*
```
If it exists, it means that `spi` has been set and can be used directly.

Compile the `spidev_test` test program in the examples of wiringOP:

```
make spidev_test
```

Do not short the `MOSI` and `MISO` pins of `spi1` first. The output result of running `spidev_test` is as follows:
```
sudo ./spidev_test -v -D /dev/spidev1.1
```
You can see that the data of TX and RX are inconsistent:
```
spi mode: 0x0
bits per word: 8
max speed: 500000 Hz (500 KHz)
TX | FF FF FF FF FF FF 40 00 00 00 00 95 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF F0 0D  |......@.........................|
RX | FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF  |................................|
```


Then short the two pins of `spi1` `MOSI` (pin 19 in the 26pin interface) and `MISO` (pin 21 in the 26pin interface) and then run the output of `spidev_test` as follows. 
```
sudo ./spidev_test -v -D /dev/spidev1.1
```
You can see the sent and received the same data:
```
spi mode: 0x0
bits per word: 8
max speed: 500000 Hz (500 KHz)
TX | FF FF FF FF FF FF 40 00 00 00 00 95 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF F0 0D  |......@.........................|
RX | FF FF FF FF FF FF 40 00 00 00 00 95 FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF F0 0D  |......@.........................|
```

