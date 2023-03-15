# BuildRoot External Project
## Introdution
This repository contains the specific configuration files and packages' rules for Phytec boards.

To use this project repository you must at least download [BuildRoot](https://buildroot.org) on your
host computer. A good place may be */opt/buildroot*.

```
host$ cd /opt/
host$ sudo git clone http://git.buildroot.net/buildroot
host$ sudo mkdir -p /opt/buildroot/dl
host$ sudo chmod a+w /opt/buildroot/dl
```

After you need to clone this repository anywhere in your compute. All commands must be run from this repository.

```
host$ cd br_phytec
host$ make -O $PWD/output BR2_EXTERNAL=$PWD -C /opt/buildroot <defconfig>
host$ make menuconfig
...
host$ make
```

## Boards

### phyCore imx6ull and Segin board

The defconfig is *phytec_imx6ull_segin_defconfig*

```
host$ make -O $PWD/output BR2_EXTERNAL=$PWD -C /opt/buildroot phytec_imx6ull_segin_defconfig
host$ make
```

#### flashing NAND with SDcard

The first step is to generate the SDcard for file storage. This is a FAT file system usable with barebox installed on the board by default.

```
host$ sudo mkfs.vfat -n flashing /dev/mmcblk0
...
host$ cp images/zImage images/imx6ull-phytec-segin*.dtb images/rootfs.ubifs /media/developper/flashing
```

Insert the SDcard on the Segin board and start it, to stop the bootloader. The rest of the instruction are available on the phytec [documentation](https://www.phytec.de/cdocuments/?doc=K4DEHw#L844e-A6i-MX6ULULLBSPReferenceManual-BuildingtheBSP#L844e-A6i-MX6ULULLBSPReferenceManual-UpdatingNANDFlashfromSDCard).

```
bootload$ detect mmc0
bootload$ ubiformat /dev/nand0.root
bootloader$ ubiattach /dev/nand0.root
bootloader$ ubimkvol -t static /dev/nand0.root.ubi kernel 16M
bootloader$ ubimkvol -t static /dev/nand0.root.ubi oftree 1M
bootloader$ ubimkvol -t dynamic /dev/nand0.root.ubi root 0
bootloader$ ubiupdatevol /dev/nand0.root.ubi.kernel /mnt/mmc/zImage
bootloader$ ubiupdatevol /dev/nand0.root.ubi.oftree /mnt/mmc/imx6ull-phytec-segin-ff-rdk-nand.dtb
bootloader$ cp -v /mnt/mmc/rootfs.ubifs /dev/nand0.root.ubi.root
```

#### testing

 * kernel on NAND flash
 * rootfs on NAND flash


#### U-boot
The support is incomplete.
 - need a configuration fragment to disable CONFIG\_LDO\_BYPASS\_CHECK=n otherwise the ldo\_check symbol is missing.
 - the DCD table is empty.

**Note**: The *mx6ull_14x14_evk_defconfig* configuration file is able to generate a binary starting on Phycore imx6ull.
But only the emmc is readable, the mmc is not detected.

##### start U-Boot from emmc
The SOM must be configurated to start on board memory.

One connection is required before starting on the console UART1. With the FTDI controler over the UART, the
*/dev/ttyUSB0* must be opened with a terminal as minicom before powering the board:
```bash
$ minicom -D /dev/ttyUSB0
```

The u-Boot's prompt should be:
```bash
U-Boot 2020.04 (Jan 31 2023 - 17:20:02 +0100)

CPU:   i.MX6ULL rev1.1 792 MHz (running at 396 MHz)
CPU:   Industrial temperature grade (-40C to 105C) at 48C
Reset cause: POR
Model: i.MX6 ULL 14x14 EVK Board
Board: MX6ULL 14x14 EVK
DRAM:  512 MiB
MMC:   FSL_SDHC: 0, FSL_SDHC: 1
Loading Environment from MMC... *** Warning - bad CRC, using default environment

[*]-Video Link 0 (480 x 272)
        [0] lcdif@21c8000, video
In:    serial
Out:   serial
Err:   serial
switch to partitions #0, OK
mmc1(part 0) is current device
flash target is MMC:1
Net:   
Warning: ethernet@20b4000 using MAC address from ROM
eth1: ethernet@20b4000 [PRIME]Get shared mii bus on ethernet@2188000

Warning: ethernet@2188000 using MAC address from ROM
, eth0: ethernet@2188000
Fastboot: Normal
Normal Boot
Hit any key to stop autoboot:  0 
=> printenv
baudrate=115200
board_name=EVK
board_rev=14X14
boot_fdt=try
bootcmd=run findfdt;run findtee;mmc dev ${mmcdev};mmc dev ${mmcdev}; if mmc rescan; then if run loadbootscript; then run bootscript; else if run loadimage; then run mmcboot; else run netboot; fi; fi; elsei
bootcmd_mfg=run mfgtool_args;if iminfo ${initrd_addr}; then if test ${tee} = yes; then bootm ${tee_addr} ${initrd_addr} ${fdt_addr}; else bootz ${loadaddr} ${initrd_addr} ${fdt_addr}; fi; else echo "Run f;
bootdelay=3
bootscript=echo Running bootscript from mmc ...; source
console=ttymxc0
emmc_ack=1
emmc_dev=1
eth1addr=50:2d:f4:32:d7:08
ethact=ethernet@20b4000
ethaddr=50:2d:f4:32:d7:07
ethprime=eth1
fastboot_dev=mmc1
fdt_addr=0x83000000
fdt_file=undefined
fdt_high=0xffffffff
fdtcontroladdr=9de6b810
findfdt=if test $fdt_file = undefined; then if test $board_name = ULZ-EVK && test $board_rev = 14X14; then setenv fdt_file imx6ulz-14x14-evk.dtb; fi; if test $board_name = EVK && test $board_rev = 9X9; th;
findtee=if test $tee_file = undefined; then if test $board_name = ULZ-EVK && test $board_rev = 14X14; then setenv tee_file uTee-6ulzevk; fi; if test $board_name = EVK && test $board_rev = 9X9; then setenv;
image=zImage
initrd_addr=0x86800000
initrd_high=0xffffffff
ip_dyn=yes
kboot=bootz 
loadaddr=0x80800000
loadbootscript=fatload mmc ${mmcdev}:${mmcpart} ${loadaddr} ${script};
loadfdt=fatload mmc ${mmcdev}:${mmcpart} ${fdt_addr} ${fdt_file}
loadimage=fatload mmc ${mmcdev}:${mmcpart} ${loadaddr} ${image}
loadtee=fatload mmc ${mmcdev}:${mmcpart} ${tee_addr} ${tee_file}
mfgtool_args=setenv bootargs console=${console},${baudrate} rdinit=/linuxrc clk_ignore_unused 
mmcargs=setenv bootargs console=${console},${baudrate} root=${mmcroot}
mmcautodetect=yes
mmcboot=echo Booting from mmc ...; run mmcargs; if test ${tee} = yes; then run loadfdt; run loadtee; bootm ${tee_addr} - ${fdt_addr}; else if test ${boot_fdt} = yes || test ${boot_fdt} = try; then if run ;
mmcdev=1
mmcpart=1
mmcroot=/dev/mmcblk1p2 rootwait rw
netargs=setenv bootargs console=${console},${baudrate} root=/dev/nfs ip=dhcp nfsroot=${serverip}:${nfsroot},v3,tcp
netboot=echo Booting from net ...; ${usb_net_cmd}; run netargs; if test ${ip_dyn} = yes; then setenv get_cmd dhcp; else setenv get_cmd tftp; fi; ${get_cmd} ${image}; if test ${tee} = yes; then ${get_cmd} ;
script=boot.scr
sd_dev=1
serial#=234271d75d43c0e7
splashimage=0x8c000000
tee=no
tee_addr=0x84000000
tee_file=undefined

Environment size: 3414/8188 bytes
=> setenv fdt_file imx6ull-phytec-segin-ff-rdk-emmc.dtb
=> saveenv
=> boot
```

While the version of u-boot is not able to read the emmc, the root filesystem may set on mmc and the Linux bootargs must be changed to start on mmc:

```bash
=> setenv mmcroot /dev/mmcblk1p2 rootwait rw
=> saveenv
=> boot
```
###### Linux installing
Currently is impossible to install the System from u-boot.

#### Barebox
The configuration is imx\_v7 from the phytec barebox git repository:
 - BR2\_TARGET\_BAREBOX\_CUSTOM\_GIT=y
 - BR2\_TARGET\_BAREBOX\_VERSION="7fe12e65d770f7e657e683849681f339a996418b"
 - BR2\_TARGET\_BAREBOX\_CUSTOM\_GIT\_REPO\_URL="git://git.phytec.de/barebox"
 - BR2\_TARGET\_BAREBOX\_CUSTOM\_GIT\_VERSION="7fe12e65d770f7e657e683849681f339a996418b"
 - BR2\_TARGET\_BAREBOX\_USE\_DEFCONFIG=y
 - BR2\_TARGET\_BAREBOX\_IMAGE\_FILE="images/barebox-phytec-phycore-imx6ull-emmc-512mb.img"

##### start Barebox from USB OTG
Two USB(serial) connection is required before starting:
 - first to load Barebox on USB OTG.
 - second to have the prompt on Barebox at the starting.
From the host PC two terminals are required.

###### The Barebox loading
The */dev/ttyUSB0* must be opened with a terminal as minicom after powering the board:
```bash
$ minicom -D /dev/ttyUSB0
```

From the Buildroot output directory:
```bash
$ sudo ./host/bin/imx\_usb -v images/barebox-phytec-phycore-imx6ull-emmc-512mb.img
config file <./host/bin//../etc/imx-loader.d/imx_usb.conf>
vid=0x066f pid=0x3780 file_name=mx23_usb_work.conf
vid=0x15a2 pid=0x004f file_name=mx28_usb_work.conf
vid=0x15a2 pid=0x0052 file_name=mx50_usb_work.conf
vid=0x15a2 pid=0x0054 file_name=mx6_usb_work.conf
vid=0x15a2 pid=0x0061 file_name=mx6_usb_work.conf
vid=0x15a2 pid=0x0063 file_name=mx6_usb_work.conf
vid=0x15a2 pid=0x0071 file_name=mx6_usb_work.conf
vid=0x15a2 pid=0x007d file_name=mx6_usb_work.conf
vid=0x15a2 pid=0x0080 file_name=mx6ull_usb_work.conf
vid=0x1fc9 pid=0x0128 file_name=mx6_usb_work.conf
vid=0x15a2 pid=0x0076 file_name=mx7_usb_work.conf
vid=0x1fc9 pid=0x0126 file_name=mx7ulp_usb_work.conf
vid=0x15a2 pid=0x0041 file_name=mx51_usb_work.conf
vid=0x15a2 pid=0x004e file_name=mx53_usb_work.conf
vid=0x15a2 pid=0x006a file_name=vybrid_usb_work.conf
vid=0x066f pid=0x37ff file_name=linux_gadget.conf
vid=0x1b67 pid=0x4fff file_name=mx6_usb_sdp_spl.conf
vid=0x0525 pid=0xb4a4 file_name=mx6_usb_sdp_spl.conf
vid=0x1fc9 pid=0x012b file_name=mx8mq_usb_work.conf
vid=0x1fc9 pid=0x0134 file_name=mx8mm_usb_work.conf
vid=0x1fc9 pid=0x013e file_name=mx8mn_usb_work.conf
vid=0x3016 pid=0x1001 file_name=mx8m_usb_sdp_spl.conf
config file <./host/bin//../etc/imx-loader.d/mx6ull_usb_work.conf>
parse ./host/bin//../etc/imx-loader.d/mx6ull_usb_work.conf
Trying to open device vid=0x15a2 pid=0x0080
Interface 0 claimed
HAB security state: development mode (0x56787856)
== work item
filename images/barebox-phytec-phycore-imx6ull-emmc-512mb.img
load_size 0 bytes
load_addr 0x00000000
dcd 1
clear_dcd 0
plug 1
jump_mode 3
jump_addr 0x00000000
== end work item
loading DCD table @0x910000

<<<504, 504 bytes>>>
succeeded (security 0x56787856, status 0x128a8a12)
clear dcd_ptr=0x8000042c

loading binary file(images/barebox-phytec-phycore-imx6ull-emmc-512mb.img) to 80000000, skip=0, fsize=a8000 type=aa

<<<688128, 688128 bytes>>>
succeeded (security 0x56787856, status 0x88888888)
Verify success
jumping to 0x80000400
```

On the second terminal, the Barebox's prompt should appear:
```bash
barebox 2022.02.0 #1 Tue Mar 14 16:39:14 CET 2023


Board: PHYTEC phyCORE-i.MX6 ULL SOM with eMMC
Deep probe supported due to phytec,imx6ul-pcl063-emmc
detected i.MX6 ULL revision 1.1
i.MX6 ULL unique ID: 234271d75d43c0e7
netconsole: registered as netconsole-1
phySOM-i.MX6: Using environment in NAND flash
imx-usb 2184200.usb@2184200.of: USB EHCI 1.00
mdio_bus: miibus0: probed
imx-esdhc 2190000.mmc@2190000.of: registered as mmc0
imx-esdhc 2194000.mmc@2194000.of: registered as mmc1
eth0: got preset MAC address: 50:2d:f4:32:d7:07
state: New state registered 'state'
state: init error: Invalid argument: No meta data header found
state: init error: Invalid argument: No meta data header found
state: init error: Invalid argument: No meta data header found
state: init error: No such file or directory: no valid state copy in any bucket
state: init error: No such file or directory: format raw read failed
state imx6ul_phytec_boot_state.of: Failed to load persistent state, continuing with defaults, -2
malloc space: 0x8fe7cbc0 -> 0x9fcf977f (size 254.5 MiB)
barebox-environment chosen:environment-nand.of: probe deferred
environment load /dev/env0: No such file or directory
Maybe you have to create the partition.

Hit m for menu or any to stop autoboot:    2
barebox@PHYTEC phyCORE-i.MX6 ULL SOM with eMMC:/
```

###### System installing from SD card to emmc.
First copy the *./images/sdcard.img* file from the Buildroot's output directory to a SD card.

Insert the card on the board and open the Barebox's prompt.
```bash
barebox@PHYTEC phyCORE-i.MX6 ULL SOM with eMMC:/ detect mmc1
mmc1: detected MMC card version 5.0
mmc1: registered mmc1.boot0
mmc1: registered mmc1.boot1
mmc1: registered mmc1
barebox@PHYTEC phyCORE-i.MX6 ULL SOM with eMMC:/ detect mmc0  
mmc0: detected SD card version 2.0
mmc0: registered mmc0
barebox@PHYTEC phyCORE-i.MX6 ULL SOM with eMMC:/ ls /mnt/
eeprom0.update-eeprom       fat
mmc                         mmc0
mmc0.0                      mmc0.barebox
mmc0.barebox-environment    mmc1
mmc1.barebox                mmc1.barebox-environment
mmc1.boot0                  mmc1.boot1
ratp                        tftp
barebox@PHYTEC phyCORE-i.MX6 ULL SOM with eMMC:/ ls /mnt/mmc0.0
mounted /dev/mmc0.0 on /mnt/mmc0.0
sdcard.img      
barebox@PHYTEC phyCORE-i.MX6 ULL SOM with eMMC:/ cp -v /mnt/mmc0.0/sdcard.img /dev/mmc1
        [################################################################]
```

At this moment the emmc contains the new filesystem. Barebox needs to reload the emmc, a good way is to restart the board
and reload Barebox as the first time.

From the new Barebox's prompt it is possible to browse the new filesystem.
```bash
barebox 2022.02.0 #1 Tue Mar 14 16:39:14 CET 2023


Board: PHYTEC phyCORE-i.MX6 ULL SOM with eMMC
Deep probe supported due to phytec,imx6ul-pcl063-emmc
detected i.MX6 ULL revision 1.1
i.MX6 ULL unique ID: 234271d75d43c0e7
netconsole: registered as netconsole-1
phySOM-i.MX6: Using environment in NAND flash
imx-usb 2184200.usb@2184200.of: USB EHCI 1.00
mdio_bus: miibus0: probed
imx-esdhc 2190000.mmc@2190000.of: registered as mmc0
imx-esdhc 2194000.mmc@2194000.of: registered as mmc1
eth0: got preset MAC address: 50:2d:f4:32:d7:07
state: New state registered 'state'
state: Using bucket 0@0x00000000
malloc space: 0x8fe7cbc0 -> 0x9fcf977f (size 254.5 MiB)
barebox-environment chosen:environment-nand.of: probe deferred
environment load /dev/env0: No such file or directory
Maybe you have to create the partition.

Hit m for menu or any to stop autoboot:    2
barebox@PHYTEC phyCORE-i.MX6 ULL SOM with eMMC:/ detect mmc1
mmc1: detected MMC card version 5.0
mmc1: registered mmc1.boot0
mmc1: registered mmc1.boot1
mmc1: registered mmc1
barebox@PHYTEC phyCORE-i.MX6 ULL SOM with eMMC:/ ls /mnt/mmc1.0
ext4 ext40: EXT2 rev 1, inode_size 256, descriptor size 32
mounted /dev/mmc1.0 on /mnt/mmc1.0
bin           boot          dev           etc           home
lib           lib32         linuxrc       lost+found    media
mnt           opt           proc          root          run
sbin          sys           tmp           usr           var
barebox@PHYTEC phyCORE-i.MX6 ULL SOM with eMMC:/ ls /mnt/mmc1.0/boot
imx6ull-phytec-alc3p.dtb    zImage
```

###### Linux booting from emmc
From the Barebox's prompt :
 - **becareful no dot between global and linux**.
 - to start from mmc change mmc1.0 to mmc0.0

```bash
barebox@PHYTEC phyCORE-i.MX6 ULL SOM with eMMC:/ global.bootm.image=/mnt/mmc1.0/zImage
barebox@PHYTEC phyCORE-i.MX6 ULL SOM with eMMC:/ global.bootm.oftree=/mnt/mmc1.0/imx6ull-phytec-alc3p.dtb
barebox@PHYTEC phyCORE-i.MX6 ULL SOM with eMMC:/ global linux.bootargs.dyn.root="root=/dev/mmcblk1p2 rootflags='data=journal'"
barebox@PHYTEC phyCORE-i.MX6 ULL SOM with eMMC:/ bootm

Loading ARM Linux zImage '/mnt/mmc1.0/boot/zImage'
Loading devicetree from '/mnt/mmc1.0/boot/imx6ull-phytec-alc3p.dtb'
commandline: console=ttymxc0,115200n8
Starting kernel in secure mode
[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 5.15.63 (developper@Corto) (arm-buildroot-linux-gnueabihf-gcc.br_real (Buildroot 2022.02.2) 10.3.0, GNU ld (GNU Binutils) 2.36.1) #1 SMP PREEMPT Tue Jan 31 17:46:14 CET 2023
[    0.000000] CPU: ARMv7 Processor [410fc075] revision 5 (ARMv7), cr=10c5387d
[    0.000000] CPU: div instructions available: patching division code
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
[    0.000000] OF: fdt: Machine model: SCANTECH alc3p i.MX6 UltraLite
[    0.000000] Memory policy: Data cache writealloc
[    0.000000] cma: Failed to reserve 256 MiB
[    0.000000] Zone ranges:
[    0.000000]   Normal   [mem 0x0000000080000000-0x000000009fffffff]
[    0.000000]   HighMem  empty
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000080000000-0x000000009fffffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000080000000-0x000000009fffffff]
[    0.000000] percpu: Embedded 11 pages/cpu s15916 r8192 d20948 u45056
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 130048

```
