[TOC]


## 1 Hardware Details

| Base | I-Pi SMARC Plus |
|:----------------|:-----------|
| **Module** | **LEC-iMX8MP** |

## 2 Software Details
|   Debian   |   Ver 12   |
|:-----------:|:-----------:|
| **Codename**  | **Bookworm** |
| **Kernel**  | **6.6.3** |
| **U-Boot**  | **2023.04** |
| **Host OS** | **Ubuntu 22.04 or Debian 12** |

## 3 Flashing the Image and Booting

### 3.1 SD Boot

- On a linux host machine, insert the micro SD card (through an USB adapter).
- Check the device node of the micro SD card using lsblk and dmesg command.
- Copy the release image to the designated folder
- Unzip and flash the image to SD card as below:

```
$ unzip adlink-lec-imx8mp-debian-bookworm_V2_R1_250226.zip
$ cd adlink-lec-imx8mp-debian-bookworm_V2_R1_250226
$ sudo dd if=adlink-lec-imx8mp-debian-bookworm_V2_R1_250226.wic of=/dev/sd[X] bs=32M
$ sync
```

* /dev/sd[X] need to be changed to actual device node of the micro SD card.
* SD card of minimum size 16GB is needed for flashing the image

### 3.2 eMMC Boot

#### Download uuu utility

- Download uuu utility and copy to /usr/bin or utitlise the uuu bin file provided in the release.

- https://github.com/nxp-imx/mfgtools/releases/download/uuu_1.4.182/uuu

   ```
   $ sudo cp ~/Downloads/uuu /usr/bin
   $ sudo chmod +x /usr/bin/uuu
   ```

#### Boot into Recovery Mode

 * Set the boot switch into recovery mode.
 * Connect USB OTG cable to host.
 * Power on the board.

#### Flash image to eMMC

* Unzip ```adlink-lec-imx8mp-debian-bookworm_V2_R1_250226.zip``` and ```adlink-lec-imx8mp-debian-bookworm_V2_R1_250226_UBoot.zip``` to obtain wic and u-boot binaries.Copy them into a single folder.
  Then, Flash the wic image to eMMC as below:
 ```
  $ sudo uuu -b emmc_all adlink-lec-imx8mp-debian-bookworm_V2_R1_250226_UBoot.bin adlink-lec-imx8mp-debian-bookworm_V2_R1_250226.wic
 ```


## 4 Peripheral testing

### 4.1 USB Type A

* All USB type A ports are validated.

### 4.2 Micro USB

#### 4.2.1 Micro USB (Device mode)

 * Connect micro USB cable to access device in client mode.

 * Execute following commands in target to access device as serial gadget.
   ```
   # echo device > /sys/kernel/debug/usb/38100000.usb/mode
   # modprobe g_serial
   ```
   
 * Run ```lsusb``` from Host and check if board is detected as serial gadget.

#### 4.2.2 Micro USB (Host mode)

 * Connect micro USB to usb converter to the port and connect a storage medium to it.

 * Execute following commands in target to access storage medium,

   ```
   # echo host > /sys/kernel/debug/usb/38100000.usb/mode
   ```

 * Run ```lsusb``` from Host and check if sorage medium is detected.Do not connect the device through otg cable to a PC when in host mode.

### 4.3 HDMI

- HDMI function is enabled by default.

### 4.4 Serial Ports

#### 4.4.1 SER 1(RS232)- Console

* Connect UART RS232 to usb converter cable to CN1609 expansion connector to get boot logs.

Pin connection:

| Pin  | Function |
|:----:|:--------:|
| 1 | UART RX |
| 3 | UART TX |
| 5 | GND     |

#### 4.4.2 SER 2 - RS232

-  Connect the RX and TX cables.


 Pin connection:

| Pin  | Function |
| :--: | :------: |
|  7   | UART RX  |
|  9   | UART TX  |
|  6   |   GND    |

##### UART Tx Test

* Open minicom, 115200 baudrate with no hardware flow control setting.

* Run the below commands from a terminal on the target.

  ```
  $ cat /dev/ttymxc3 | head -n 1
  ```

  

##### UART Rx Test

* Run below command in another terminal on the target.

  ```
  $ stty -F /dev/ttymxc3 115200 cs8 -cstopb -parenb
  $ echo 'ADLINK' > /dev/ttymxc3
  ```

  'ADLINK' string will be displayed on the first terminal.

#### 4.4.3 SER 0 - CN1001

Connect the RX and TX cables.

Pin connection:

| Pin  | Function |
| ---- | -------- |
| 10   | UART RX  |
| 8    | UART TX  |
| 6    | GND      |

##### UART Tx Test

- Open minicom, 115200 baudrate with no hardware flow control setting.
- Run the below commands from a terminal on the target.

```
$ cat /dev/ttymxc1 | head -n 1
```

##### UART Rx Test

- Run below command in another terminal on the target.

```
$ stty -F /dev/ttymxc1 115200 cs8 -cstopb -parenb $ echo 'ADLINK' > /dev/ttymxc1
```

'ADLINK' string will be displayed on the first terminal.



### 4.5. Audio Codec

------

#### 4.5.1. WM8960 Audio Codec

Connect a headphone to the jack, then select the audio device from the **Debian Start Menu** in the top-right corner. The device name should be **"Built-in Headphone"**. Now, any audio on the system can be heard through the headphone.

Alternatively, you can use the following commands to test:

```
$ aplay -l 
$ aplay -D plughw:<card number>,<device number> <sample.wav>
```

To play the wav file on HDMI, find the card number and device number from aplay list command and excecute the command.

#### 4.5.2. TLV320 Audio Codec

* TLV320 Audio Codec feature can be enabled by enabling corresponding DTB file during boot.

* boot into uboot and run the below commands,

  ```
  # setenv fdt_file lec-imx8mp-tlv320aic3x.dtb
  # boot
  ```

* Connect headphone on the jack, Select audio device from the debian start menu on the top right corner.The name should be "built in headphone".Now any audio on the system can be heard from the headphone.Alternatively you can use the below commands to test,

  ```
  $ aplay -l 
  $ aplay -D plughw:<card number>,<device number> <sample.wav>
  ```


#### 4.5.3. HDMI Audio

- HDMI audio is enabled by default.you can use the below commands to test HDMI audio,

  ```
  $ aplay -l 
  $ aplay -D plughw:<card number>,<device number> <sample.wav>
  ```

### 4.6 GPIO on Expansion Connector

 GPIO on expansion connector (CN1001) can be accessed using libgpiod or sysfs entry,

#### 4.6.1. Libgpiod

Use the below commands perform loopback test on each gpio pin,

```
//gpiod interface
//loopback pin35 and pin36:
$ gpioset gpiochip<CHIP_NUM> <LINE_NUM>=<VALUE>
$ gpioget gpiochip<CHIP_NUM> <LINE_NUM>
ex:
root@imx8mp:/home/debian# gpioset gpiochip7 4=0
root@imx8mp:/home/debian# gpioget gpiochip7 5
0
root@imx8mp:/home/debian# gpioset gpiochip7 4=1
root@imx8mp:/home/debian# gpioget gpiochip7 5
1
```

The pin number to Line number and chip number mapping is mentioned below if you want to use the gpiod interface:

| Pin on expansion | Chip Number | Line Number |
| :--------------: | :---------: | :---------: |
|        7         |      5      |      2      |
|        12        |      5      |      3      |
|        11        |      5      |      4      |
|        13        |      5      |      5      |
|        15        |      5      |      6      |
|        16        |      5      |      7      |
|        18        |      5      |      8      |
|        22        |      5      |      9      |
|        29        |      7      |      0      |
|        31        |      7      |      1      |
|        32        |      7      |      2      |
|        33        |      7      |      3      |
|        35        |      7      |      4      |
|        36        |      7      |      5      |
|        37        |      7      |      6      |
|        38        |      7      |      7      |
|        40        |      7      |      8      |

To find information on GPIO lines, use the following command:

```
gpioinfo
```

#### 4.6.2. SYSFS Entry

```
//sysfs interface 
$ cd /sys/class/gpio/
$ echo GPIO_NUM > export
$ cd gpio<GPIO_NUM>
$ echo out > direction ("out" is to enable pin as ouput, "in" for input)
$ echo 1 > value       ("1" to drive high, "0" to drive low)
```

 The pin number to kernel gpio number mapping is mentioned below ,

| Pin on expansion | Kernel Gpio number |
|:----------------:|:------------------:|
|      7        |      514    |
|      12       |      515    |
|      11       |      516    |
|      13       |      517    |
|      15       |      518    |
|      16       |      519    |
|      18       |      520    |
|      22       |      521    |
|      29       |      546    |
|      31       |      547    |
|      32       |      548    |
|      33       |      549    |
|      35       |      550    |
|      36       |      551    |
|      37       |      552    |
|      38       |      553    |
|      40       |      554    |

To find the line to corresponding gpio number mapping use(the line numbers are the offset from the first pin in the current gpiochip),

```
cat /sys/kernel/debug/gpio 
```

### 4.7 SPI on expansion connector
 Two instances of SPI are available for the user.

 Follow below procedure to perform loop-back test:

#### SPI1 Loopback test
* Connect Pin 19 and 21 in CN1001 connector
* Run below command to send and receive data over SPI1
    ```
    $ spidevtest -D /dev/spidev1.0 -p "12345" -N -v
    ```

#### SPI2 Loopback test

- Connect Pin 27 and 28 in CN1602 connector
- Run below command to send and receive data over SPI2

```
$ spidevtest -D /dev/spidev2.0 -p "12345" -N -v
```

### 4.8 CAN interface (CN1602)

Setup CAN0 & CAN1 Loopback: Connect Pins (13 - 14) and (15 - 16) in CN1602 connector.

Sender should execute below commands:

1. Configure the CAN0 ports as
   ```
   $ ip link set can0 type can bitrate 500000
   $ ip link set can0 up
   ```

2. Configure the CAN1 ports as
   ```
   $ ip link set can1 type can bitrate 500000
   $ ip link set can1 up
   ```

3. Dump CAN data on can0:
   ```
   $ candump can0 &
   ```

4. Send data over can1:
   ```
   $ cansend can1 01a#11223344AABBCCDD
   ```

    Now, data sent from CAN1 will be dumped back on CAN0.


### 4.9 RTC

* Debian will update date/time from internet when connected to network. Date/time settings is available under:

  Settings -> Date & time

* Set the region here correctly and the time automatically updated now disconect the internet connection and follow the below steps on a terminal,

  ```
  # hwclock -w -f /dev/rtc1
  
  Turn off the system for 5 mins and run below command,
  # hwclock -s -f /dev/rtc1
  # date
  ```

* For each reboot,if not connected to the internet, you may sync the time using the above command 

### 4.10. Wifi/BT

- WiFi and Bluetooth support and functionalities can be managed using the **Debian Settings** application.

### 4.11. MIPI DSI display

* DSI display feature can be enabled by enabling corresponding DTB file during boot.

* boot into uboot and run the below commands,

  ```
  # setenv fdt_file lec-imx8mp-auoB101UAN01-mipi-panel.dtb
  # boot
  ```

### 4.12. LVDS display

* LVDS display feature can be enabled by enabling corresponding DTB file during boot.
* boot into uboot and run the below commands,

  ```
  # setenv fdt_file lec-imx8mp-hydis-hv150ux2.dtb
  # boot
  ```

### 4.13. PCIe
* Run ```lspci``` command from adb shell to list connected PCI devices.
* Vendor ID details can be extracted by running lspci command with '-n' argument.

### 4.14. Ethernet

#### 4.14.1 Ethernet in u-boot
 * Press any key to break into U-Boot command prompt.
 * Execute the below commands to configure u-boot network (The following are provided as an example, please change appropriately)
   ```
   u-boot=> setenv ipaddr 192.168.1.126
   u-boot=> setenv serverip 192.168.1.5
   u-boot=> setenv netmask 255.255.255.0
   ```
##### ETH0 - FEC
* Execute the below commands to ping from ETH0 port.

      u-boot=> setenv ethact eth0
      u-boot=> ping 192.168.1.1

##### ETH1 - DWMAC
* Execute the below commands to ping from ETH1 port.

      u-boot=> setenv ethact eth1
      u-boot=> ping 192.168.1.1

#### 4.14.2 Ethernet in OS

 * Execute the below commands to check presence of network,

   ```
   $ ifconfig
   $ ping 8.8.8.8 
   ```

### 4.15 I2C

I2C devices connected to 0,1,2,4,5,6 can be checked using the following,

```
i2cdetect -y -r <device number>
```
