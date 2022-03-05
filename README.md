
# Overview

The **i2cblock** is a Linux utility to test the large IO chunks' capabilities
and benchmark performance of the I2C controllers and Linux kernel I2C bus
drivers.

The i2cblock was originally written to save me the hassle of utilizing the
Linux i2c tools like i2ctransfer, i2cdump, i2cset, and i2cget to debug and test
the hid-ft260.ko Linux kernel driver for the FTDI FT260 chip, which implements
the USB HID to I2C bus bridge. And specifically, to issue the data transfers
exceeding a single USB HID report size or maximum size of the EEPROM Page Write
operation. The maximum transfer size in the Page Write mode differs between
EEPROM chips. For example, it's 8 bytes for the 24C02 and 128 bytes for the
24C512. The size of the internal address, also known as offset, is different
for small and large EEPROMs. It can be either one or two bytes depending on
the EEPROM size.

The eeprog utility of the i2c-toots package provides a very limited EEPROM
read-write functionality only, which is not sufficient to exercise the I2C bus
driver. Unlike the I2C standard Linux tools like i2cdetect, i2cget, i2cset,
i2cdump, and i2ctransfer tools, it is not a part of the Linux distributions
and requires downloading, compilation, and i2c-dev driver loaded. Then, it can
perform a single byte i2c read and write IO transactions only that adds a
significant overhead per reading IO since, for every single I2C read byte
transfer, it performs a write address of one or two bytes transfer. It neither
generates test patterns nor measures performance and is helpless for the I2C
bus driver benchmarking.

This bash script leverages the standard Linux i2c tools simplifying their
configuration to achieve reads and writes of big data blocks in the differently
sized EEPROM chips. It allows performance benchmarking of the I2C controller
and Linux kernel I2C bus driver by aggregating I2C IO transfer time of multiple
separate chunks of data.

```
Usage: i2cblock [-h] [-v] [-f TYPE] [-d TYPE] [-o SIZE] [-s SIZE] [-r FIRST-LAST] BUS ADDRESS
        BUS                    I2C bus master device number
        ADDRESS                EEPROM device address (0x50 - 0x57)
optional arguments:
        -h, --help             Show this help message and exit
        -v, --verbose          More verbose output
        -t, --stats            Statistics
        -f, --fillmode         Write block:
                               1 - Fill block with zeros via i2ctransfer
                               2 - Fill block with increment via i2ctransfer by chunks
                               3 - Fill block with increment via i2cset byte by byte
                               4 - Fill block with pseudo random via i2ctransfer by chunks
        -d, --dumpmode         Read block:
                               1 - Read block via i2cdump byte by byte
                               2 - Read block via i2ctransfer by chunks
        -r, --range FIRST-LAST Address range to operate
        -s, --xfersize SIZE    Bytes in write chunk
        -o, --offsize          Offset size:
                               1 - One byte offset like in 24C01 - 24C16 EEPROMs
                               2 - Two bytes offest, for larger than 256B EEPROMs

```

Tested on Ubuntu 20.04.3 LTS with:
- 24c02, 24c32, 24c512 with UMFT260EV1A EVB [1] and hid-ft260 kernel driver [2]
- 24c32, 24c512  with CH341A black board [3] and 2c-ch341-usb kernel driver [4]

# Examples

## Write 1KB of data by chunks of 16 bytes size
### 24c512 EEPROM, UMFT260EV1A EVB [1], hid-ft260 kernel driver [2]
```
$ ./i2cblock -f 2 -o 2 -s 16 -r 0-0x3ff 1 0x51 -S
Fill mode       Fill block with increment via i2ctransfer by chunks
Data chunks     64
Chunk bytes     16
Payload bytes   1024
IOs total sec   1.223862
Chunk avg sec   .019122
```
### 24c512 EEPROM, CH341A black board [3], 2c-ch341-usb kernel driver [4]
```
$ ./i2cblock -f 2 -o 2 -s 16 -r 0-0x3ff 1 0x50 -S
Fill mode       Fill block with increment via i2ctransfer by chunks
Data chunks     64
Chunk bytes     16
Payload bytes   1024
IOs total sec   1.417975
Chunk avg sec   .022155
```

## Read 1KB of data by chunks of 16 bytes size
### 24c512 EEPROM, UMFT260EV1A EVB [1], hid-ft260 kernel driver [2]
```
$ ./i2cblock -d 2 -o 2 -s 16 -r 0-0x3ff 1 0x51 -S
Dump mode       Read block via i2ctransfer by chunks
Data chunks     64
Chunk bytes     16
Payload bytes   1024
IOs total sec   2.303542
Chunk avg sec   .035992
```
### 24c512 EEPROM, CH341A black board [3], 2c-ch341-usb kernel driver [4]
```
$ ./i2cblock -d 2 -o 2 -s 16 -r 0-0x3ff 1 0x50 -S
Dump mode       Read block via i2ctransfer by chunks
Data chunks     64
Chunk bytes     16
Payload bytes   1024
IOs total sec   2.454836
Chunk avg sec   .038356

```
## Write 256B of data byte by byte
### 24c512 EEPROM, UMFT260EV1A EVB [1], hid-ft260 kernel driver [2]
```
$ ./i2cblock -f 3 -o 2 -r 0-0xff 1 0x51 -S
Fill mode       Fill block with increment via i2cset byte by byte
Data chunks     16
Chunk bytes     16
Payload bytes   256
IOs total sec   4.527803
Chunk avg sec   .282987
```
## Read 256B of data byte by byte
### 24c512 EEPROM, UMFT260EV1A EVB [1], hid-ft260 kernel driver [2]
```
$ ./i2cblock -d 1 -o 2 -r 0-0xff 1 0x51 -S
Dump mode       Read block via i2cdump byte by byte
Data chunks     16
Chunk bytes     16
Payload bytes   256
IOs total sec   2.487354
Chunk avg sec   .155459
```

# Source
https://github.com/MichaelZaidman/i2cblock.git


# References
[1] https://www.ftdichip.com/Support/Documents/DataSheets/Modules/DS_UMFT260EV1A.pdf

[2] https://github.com/MichaelZaidman/hid-ft260

[3] https://www.onetransistor.eu/2017/08/ch341a-mini-programmer-schematic.html

[4] https://github.com/allanbian1017/i2c-ch341-usb

