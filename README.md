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
