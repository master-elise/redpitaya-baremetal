# Red Pitaya Zynq7010 (bare metal)

This repository is aimed to simplify the interaction with the Red Pitaya
Zynq 7010 by using standard Linux open source utilities (such as
gcc, gdb, openocd) instead of proprietary Xilinx SDK. It is something
like [libopencm3][3] but for Xilinx Zynq 7010.

## Quick Start on the Red Pitaya hardware

  1. Check if you have these software installed:

     - `arm-none-eabi-gcc` (package name `gcc-arm-none-eabi` with Debian GNU/Linux)
     - `picolibc-arm-none-eabi` (Debian `arm-none-eabi-gcc` lost compatibility with 
`newlib` according to this 
[https://www.mail-archive.com/debian-bugs-dist@lists.debian.org/msg2084326.html][post])

  2. A `buildroot` image including `uboot` as found at https://github.com/trabucayre/redpitaya

  3. Open terminal and check for `/dev/ttyUSB0` when connecting the middle-USB microB port of
the Red Pitaya

  4. When prompted by uboot, hit a key to stop the countdown leading the booting Linux

  5. type the uboot commands
```
dcache off                                                                
fatload mmc 0 0x0 blink.bin                                          
dcache flush                                                              
go 0x0    
```
and the red LED should blink.

## Using QEMU

The executable can be run from QEMU with
```
qemu-system-arm  -M xilinx-zynq-a9 -m 1024  -serial mon:stdio  -nographic -kernel examples/uart/uart.elf
```

## Using U-Boot

In order to benefit from U-Boot Zynq initialization, we might want to launch the executables
from this bootloader.

From Buildroot ``output/build/uboot-custom``, we generated ``u-boot.elf`` which is executed
and displays an output with 
```
$ qemu-system-arm  -M xilinx-zynq-a9 -m 1024  -serial mon:stdio  -nographic -kernel u-boot.elf 

U-Boot 2025.01 (Apr 03 2026 - 17:00:15 +0200)

Model: Red Pitaya Board
DRAM:  ECC disabled 512 MiB
Core:  24 devices, 17 uclasses, devicetree: board
Flash: 0 Bytes
NAND:  0 MiB
MMC:   mmc@e0100000: 0
Loading Environment from SPIFlash... zynq_qspi spi@e000d000: Invalid chip select 0:0 (err=-19)
*** Warning - spi_flash_probe_bus_cs() failed, using default environment

In:    serial@e0000000
Out:   serial@e0000000
Err:   serial@e0000000
Net:   Could not get PHY for eth0: addr 1
No ethernet found.

Preboot
Hit any key to stop autoboot:  0 
Zynq> 
```

In order to load a binary stored on a virtual non-volatile mass storage medium, create a 
FAT formatted filesystem:
```
dd if=/dev/zero of=flash.mmc bs=1 count=4M  # must be a power of 2 M
sudo mkfs -t vfat flash.mmc 
mkdir tmp
sudo mount -t vfat -o loop  flash.mmc tmp
sudo cp *.bin tmp
sudo umount tmp
qemu-system-arm  -M xilinx-zynq-a9 -m 1024  -serial mon:stdio  -nographic -kernel u-boot.elf -sd flash.mmc
```
and once U-Boot is launched, again 
```
dcache off
fatload mmc 0 0x0 uart.bin
dcache flush
go 0x0
```
to execute the application printing on UART0.

## Credits

Forked from https://github.com/3ap/zybo-z7-baremetal

  - `bsp/boot.S` & `bsp/Zynq.ld` from [bigbrett/zybo-baremetal][1]
  - `bsp/ps7_init_gpl.{c,h}` & `bsp/ps7_spl_init.c` from [Das U-boot][2]

[1]: https://github.com/bigbrett/zybo-baremetal
[2]: http://git.denx.de/?p=u-boot.git;a=summary
[3]: https://github.com/libopencm3/libopencm3
