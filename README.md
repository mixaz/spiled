Simple flash LEDs connected to 74HC595 via SPI on Orange Pi Zero
and Beaglebone Black
================================================================

## Files:

 `spiled.c` - main applicaton module

 `stimer.h/stimer.c` - simple Linux timer wrapper

 `sgpio.h/sgpio.c` - simple GPIO wrappers to input/output over `/sys/class/gpio`

 `spi.h/spi.c` - simple Linux wrapper for access to `/dev/spidev`

 `Makefie` - make file to build example application (`spiled`)

 `Makefile.skel` - my "universal" Makefile for many projects

## Orange Pi Zero 26 pin connector:

 | GPIO | Signal |Pin |Pin | Signal  | GPIO |
 |:----:| ------:|:--:|:--:|:------- |:----:|
 |      |   3.3V |  1 | 2  | 5V      |      |
 |  12  |  SDA.0 |  3 | 4  | 5V      |      |
 |  11  |  SCL.0 |  5 | 6  | 0V      |      |
 |   6  | GPIO.7 |  7 | 8  | TxD1    | 198  |
 |      |     0V |  9 | 10 | RxD1    | 199  | 
 |   1  |   RxD2 | 11 | 12 | GPIO.1  | 7    |
 |   0  |   TxD2 | 13 | 14 | 0V      |      |
 |   3  |   CTS2 | 15 | 16 | GPIO.4  | 19   |
 |      |   3.3v | 17 | 18 | GPIO.5  | 18   |
 |  15  |   MOSI | 19 | 20 | 0V      |      |
 |  16  |   MISO | 21 | 22 | RTS2    | 2    |
 |  14  |   SCLK | 23 | 24 | CE0     | 13   |
 |      |     0v | 25 | 26 | GPIO.11 | 10   |

## 74HC595 DIP16 schematic:

 | Signal | Pin | Pin | Signal |          Description           |
 |:------:|:---:|:---:|:------:| ------------------------------ |
 |   QB   |  1  | 16  |  VCC   | VCC/GND - power +3.3...+5.0V   |
 |   QC   |  2  | 15  |  QA    | QA...QH - output               |
 |   QD   |  3  | 14  |  SI    | SI      - serial data input    |
 |   QE   |  4  | 13  |  nG    | nG      - Z output if '1'      |
 |   QF   |  5  | 12  |  RCK   | RCK     - storage reg. clock   |
 |   QG   |  6  | 11  |  SCK   | SCK     - shift reg. clock     |
 |   QH   |  7  | 10  |  nSCLR | nSCLR   - neg.reset shift reg. |
 |   GND  |  8  |  9  |  QH'   | QH'     - serial data output   |

## How to connect 74HC595 registor(s) (one or two) to Orange Pi Zero

 | 74HC595 signal | 74HC595 pin | Pi Zero pin              | Pi zero signal |
 |:--------------:|:-----------:| ------------------------ |:--------------:|
 |      GND       |      8      | 25,20,14,9 or 6          | 0V             |
 |      VCC       |     16      | 1 or 17                  | 3.3V           |
 |      SI        |     14      | 19                       | MOSI           |
 |      SCK       |     11      | 23                       | SCLK           |
 |      RCK       |     12      | 18                       | GPIO-18        |
 |      nG        |     13      | any 0V/GND               | 0V             |
 |      nSCLR     |     10      | any 3.3V via pullup R    | 3.3V           |
 |      QH'       |      9      | to SI of second 74HC595  | -              |

## How to connect 74HC595 registor(s) to Beaglebone Black (BBB)

 | 74HC595 signal | 74HC595 pin | BBB pin                  | BBB signal     |
 |:--------------:|:-----------:| ------------------------ |:--------------:|
 |      GND       |      8      | any 0V/GND               | DGND           |
 |      VCC       |     16      | P9-3 or P9-4             | VDD_3V3        |
 |      SI        |     14      | P9-18                    | SPI0_D1        |
 |      SCK       |     11      | P9-20                    | SPI0_SCLK      |
 |      RCK       |     12      | P9-23                    | GPIO-49        |
 |      nG        |     13      | any 0V/GND               | DGND           |
 |      nSCLR     |     10      | any 3.3V                 | VDD_3V3        |
 |      QH'       |      9      | to SI of second 74HC595  | -              |
  
74HC595 can be sourced from 5V pins of BBB as well (SYS_5V)

BBB SPI pins need to be pinmuxed to SPI mode:
```
config-pin P9_18 spi  
config-pin P9_22 spi_sclk
config-pin P9_23 gpio
```

or permanently enable SPI with DTS overlay in /boot/uEnv.txt:
```
uboot_overlay_addr4=/lib/firmware/BB-SPIDEV0-00A0.dtbo
```

Command for BBB (sudo is needed for GPIO access):
```
sudo ./spiled -d /dev/spidev0.1 -g 49
```

## How to connect LEDs to 74HC595 (one or two)

 8-16 LEDs connected to 15 and 1-7 pins 74HC595 via limit current
 resistors (~100 Ohm)

## How to build application

 Install `make`, `gcc` and `git` by apt-get or apitude on you Orange Pi Zero

 Clone this git repository from github:

```
  $ git clone https://github.com/azorg/spiled.git
  $ cd spiled 
```

 Run `make` in work directory:

> $ make

  Or use `_make.sh`:

> ./_make.sh

## How to install and run, programm command line options

 Run `make install` to install to `/usr/local/bin`
 or run `./spiled` from current directory.

 Run from root or use sudo.

```
Run:  spiled [-options] [interval-ms]
Options:
    -h|--help          - show this help
    -v|--verbose       - verbose output
   -vv|--more-verbose  - more verbose output (or use -v twice)
  -vvv|--much-verbose  - much more verbose output (or use -v thrice)
    -S|--stat          - output delay statistic to stdout (no verbose)
    -m|--reg-num       - number of 74HC595 registors (1 or 2)
    -d|--spi-dev       - SPI device name like '/dev/spidev0.0'
    -s|--spi-speed     - SPI max speed [Hz]
    -g|--rck-gpio      - GPIO channel connected to RCK 74HC595 (-1 to don't use)
    -a|--alt-num       - alternate mode number (>=0)
    -n|--negative      - negative output
    -r|--real-time     - real time mode (root required)
interval-ms            - timer interval in ms (100 by default)
```

