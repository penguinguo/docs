# Xilinx Zynq
This page is a collection of notes and how-to guides for the [Xilinx Zynq <i class="fa fa-external-link"></i>](https://www.xilinx.com/products/silicon-devices/soc.html){target="\_blank"} SoC devices.




## MPSoC
- Wiki: https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/444006775/Zynq+UltraScale+MPSoC
- Linux: https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841996/Linux
- Design resources: https://www.xilinx.com/support/documentation-navigation/design-hubs/dh0070-zynq-mpsoc-design-overview-hub.html
- Examples:
    - Ethernet: https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/478937213/MPSoC+PS+and+PL+Ethernet+Example+Projects
    - Ubuntu desktop: https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/148668419/Zynq+UltraScale+MPSoC+Ubuntu+VCU+Gstreamer+-+Building+and+Running+Ubuntu+Desktop+from+Sources
    - USB 3.0: https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/968785932/USB+Device+for+PL+Data+Acquisition+on+Zynq+UltraScale+MPSoC
    - Docker: https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/84508673/Docker+on+Zynq+Ultrascale+Xilinx+Yocto+Flow
    - Python in Yocto: https://hub.mender.io/t/how-to-work-with-python-applications-and-modules-in-yocto-project/1135
    - Run script at startup (PetaLinux): https://www.xilinx.com/support/answers/55998.html



### MPSoC Displayport
- Example: https://www.hackster.io/news/microzed-chronicles-displayport-controller-part-one-25734db13fad, https://www.hackster.io/news/microzed-chronicles-displayport-controller-part-two-1fa042f7a242
- Video can be provided either from PS (non-live) or PL (live)

### MPSoC heterogeneous architecture
- Example: https://www.hackster.io/pablotrujillojuan/using-apu-rpu-pl-on-genesys-zu-7b6ed2
- APU: Dual/Quad Arm Cortex-A53
    - Linux capable
- RPU: Dual Arm Cortex-R5F
    - Access to local program memories (TCM)

### MPSoC multiprocessing
See chapter 6 of [Xilinx UG1137](https://www.xilinx.com/support/documentation/user_guides/ug1137-zynq-ultrascale-mpsoc-swdev.pdf).

- Symmetric Multiprocessing (SMP): The use of multiple processors by a single operating system.
- Asymmetric Multiprocessing (AMP): Multiple processor with specific task for each
    - OpenAMP: https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_2/ug1186-zynq-openamp-gsg.pdf
    - Example: https://github.com/topalovicdraganl/nativeAMP





## Petalinux on Zybo board
Based on instructions in https://github.com/Digilent/Petalinux-Zybo/blob/v2017.4-1/README.md

### Ubuntu 16.04.3 LTS virtual machine
The instructions suggest to use version 16.04.3 of Ubuntu which can be found on https://old-releases.ubuntu.com/releases/16.04.3/ ([direct ISO download link](https://old-releases.ubuntu.com/releases/16.04.3/ubuntu-16.04.3-desktop-amd64.iso)).
It was installed as a virtual machine using Parallels Desktop 16.1.3 running on macOS 11.2.2.


### Install dependencies
```shell
sudo apt-get install tofrodos gawk xvfb git libncurses5-dev tftpd \
  zlib1g-dev zlib1g-dev:i386 libssl-dev flex bison chrpath socat \
  autoconf libtool texinfo gcc-multilib libsdl1.2-dev libglib2.0-dev \
  screen pax
sudo reboot
```

TFTP server:
```shell
sudo apt-get install apt-get install tftpd-hpa
sudo chmod a+w /var/lib/tftpboot/
sudo reboot
```


### Petalinux install
Prepare install location:
```shell
sudo mkdir -p /opt/pkg/petalinux
sudo chown $USER /opt/pkg/
sudo chgrp $USER /opt/pkg/
sudo chgrp $USER /opt/pkg/petalinux/
sudo chown $USER /opt/pkg/petalinux/
```

Download the installer from Xilinx (requires login): https://www.xilinx.com/member/forms/download/xef.html?filename=petalinux-v2017.4-final-installer.run

```shell
# Go to the directory where the install file was downloaded to
cd ~/Downloads
chmod +x ./petalinux-v2017.4-final-installer.run
./petalinux-v2017.4-final-installer.run /opt/pkg/petalinux
# Accept the license agreements
```

**NOTE:** Before running any petalinux commands the following file should be sourced
```shell
source /opt/pkg/petalinux/settings.sh
```

### Petalinux BSP file download for Zybo board
NOTE: The .bsp file must be on the virtual machines filesystem.
I tried to have it on the Mac and access it through a shared folder on thee virtual machine which did not work with the next step.
```shell
cd ~/Downloads
wget https://github.com/Digilent/Petalinux-Zybo/releases/download/v2017.4-1/Petalinux-Zybo-2017.4-1.bsp
```

### Create a project from the BSP file
```shell
cd ~/Downloads
petalinux-create -t project -s Petalinux-Zybo-2017.4-1.bsp
```

### Build the petalinux project
```shell
cd Zybo
petalinux-build
```

### Emulate in QEMU
Based on information from https://www.beyond-circuits.com/wordpress/tutorial/tutorial23/
```shell
petalinux-boot --qemu --kernel
```
Type `control-A x` to exit the emulator.


??? abstract "Example output"
    ```shell
    ubuntu@ubuntu:~/Downloads/Zybo$ petalinux-boot --qemu --kernel
    INFO: The image provided is a zImage
    INFO: TCP PORT is free 
    INFO: Starting arm QEMU
    INFO:  qemu-system-aarch64 -M arm-generic-fdt-7series -machine linux=on   -serial /dev/null -serial mon:stdio -display none -kernel /home/ubuntu/Downloads/Zybo/build/qemu_image.elf -gdb tcp::9000 -dtb /home/ubuntu/Downloads/Zybo/images/linux/system.dtb  -net nic,vlan=1 -net user,vlan=1,tftp=/var/lib/tftpboot -net nic -device loader,addr=0xf8000008,data=0xDF0D,data-len=4 -device loader,addr=0xf8000140,data=0x00500801,data-len=4 -device loader,addr=0xf800012c,data=0x1ed044d,data-len=4 -device loader,addr=0xf8000108,data=0x0001e008,data-len=4 -device loader,addr=0xF8000910,data=0xF,data-len=0x4    
    Warning: vlan 0 is not connected to host network
    rom: requested regions overlap (rom bootloader. free=0x0000000002d6b930, addr=0x0000000000000000)
    Uncompressing Linux... done, booting the kernel.
    Booting Linux on physical CPU 0x0
    Linux version 4.9.0-xilinx-v2017.4 (ubuntu@ubuntu) (gcc version 6.2.1 20161016 (Linaro GCC 6.2-2016.11) ) #3 SMP PREEMPT Sat Apr 24 17:59:37 CEST 2021
    CPU: ARMv7 Processor [410fc090] revision 0 (ARMv7), cr=10c5387d
    CPU: PIPT / VIPT nonaliasing data cache, VIPT nonaliasing instruction cache
    OF: fdt:Machine model: Zynq Zybo Development Board
    bootconsole [earlycon0] enabled
    cma: Reserved 64 MiB at 0x1c000000
    Memory policy: Data cache writealloc
    percpu: Embedded 14 pages/cpu @dbbad000 s25932 r8192 d23220 u57344
    Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 130048
    Kernel command line: console=ttyPS0,115200 earlyprintk uio_pdrv_genirq.of_id=generic-uio
    PID hash table entries: 2048 (order: 1, 8192 bytes)
    Dentry cache hash table entries: 65536 (order: 6, 262144 bytes)
    Inode-cache hash table entries: 32768 (order: 5, 131072 bytes)
    Memory: 400692K/524288K available (6144K kernel code, 208K rwdata, 1492K rodata, 44032K init, 231K bss, 58060K reserved, 65536K cma-reserved, 0K highmem)
    Virtual kernel memory layout:
        vector  : 0xffff0000 - 0xffff1000   (   4 kB)
        fixmap  : 0xffc00000 - 0xfff00000   (3072 kB)
        vmalloc : 0xe0800000 - 0xff800000   ( 496 MB)
        lowmem  : 0xc0000000 - 0xe0000000   ( 512 MB)
        pkmap   : 0xbfe00000 - 0xc0000000   (   2 MB)
        modules : 0xbf000000 - 0xbfe00000   (  14 MB)
          .text : 0xc0008000 - 0xc0700000   (7136 kB)
          .init : 0xc0900000 - 0xc3400000   (44032 kB)
          .data : 0xc3400000 - 0xc3434380   ( 209 kB)
           .bss : 0xc3434380 - 0xc346e118   ( 232 kB)
    Preemptible hierarchical RCU implementation.
      Build-time adjustment of leaf fanout to 32.
      RCU restricting CPUs from NR_CPUS=4 to nr_cpu_ids=2.
    RCU: Adjusting geometry for rcu_fanout_leaf=32, nr_cpu_ids=2
    NR_IRQS:16 nr_irqs:16 16
    efuse mapped to e0800000
    slcr mapped to e0802000
    L2C: platform modifies aux control register: 0x00000000 -> 0x30400000
    L2C: DT/platform modifies aux control register: 0x00000000 -> 0x30400000
    L2C-310 errata 588369 769419 enabled
    L2C-310 full line of zeros enabled for Cortex-A9
    L2C-310 cache controller enabled, 8 ways, 64 kB
    L2C-310: CACHE_ID 0x00000000, AUX_CTRL 0x00000000
    zynq_clock_init: clkc starts at e0802100
    Zynq clock init
    global-timer: non support for this cpu version.
    Failed to initialize '/amba/timer@f8f00200': -38clocksource: ttc_clocksource: mask: 0xffff max_cycles: 0xffff, max_idle_ns: 551318127 ns
    sched_clock: 16 bits at 52kHz, resolution 18904ns, wraps every 619458570ns
    timer #0 at e080a000, irq=16
    Console: colour dummy device 80x30
    Calibrating delay loop... 2459.23 BogoMIPS (lpj=12296192)
    pid_max: default: 32768 minimum: 301
    Mount-cache hash table entries: 1024 (order: 0, 4096 bytes)
    Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes)
    CPU: Testing write buffer coherency: ok
    CPU0: thread -1, cpu 0, socket 0, mpidr 80000000
    Setting up static identity map for 0x100000 - 0x100058
    CPU1: thread -1, cpu 1, socket 0, mpidr 80000001
    Brought up 2 CPUs
    SMP: Total of 2 processors activated (4911.92 BogoMIPS).
    CPU: WARNING: CPU(s) started in wrong/inconsistent modes (primary CPU mode 0x13)
    CPU: This may indicate a broken bootloader or firmware.
    devtmpfs: initialized
    VFP support v0.3: implementor 41 architecture 3 part 30 variant 9 rev 0
    clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
    pinctrl core: initialized pinctrl subsystem
    NET: Registered protocol family 16
    DMA: preallocated 256 KiB pool for atomic coherent allocations
    cpuidle: using governor menu
    hw-breakpoint: debug architecture 0x4 unsupported.
    zynq-ocm f800c000.ocmc: ZYNQ OCM pool: 256 KiB @ 0xe0840000
    zynq-pinctrl 700.pinctrl: zynq pinctrl initialized
    vgaarb: loaded
    SCSI subsystem initialized
    usbcore: registered new interface driver usbfs
    usbcore: registered new interface driver hub
    usbcore: registered new device driver usb
    media: Linux media interface: v0.10
    Linux video capture interface: v2.00
    pps_core: LinuxPPS API ver. 1 registered
    pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
    PTP clock support registered
    EDAC MC: Ver: 3.0.0
    FPGA manager framework
    fpga-region fpga-full: FPGA Region probed
    Advanced Linux Sound Architecture Driver Initialized.
    clocksource: Switched to clocksource ttc_clocksource
    NET: Registered protocol family 2
    TCP established hash table entries: 4096 (order: 2, 16384 bytes)
    TCP bind hash table entries: 4096 (order: 3, 32768 bytes)
    TCP: Hash tables configured (established 4096 bind 4096)
    UDP hash table entries: 256 (order: 1, 8192 bytes)
    UDP-Lite hash table entries: 256 (order: 1, 8192 bytes)
    NET: Registered protocol family 1
    RPC: Registered named UNIX socket transport module.
    RPC: Registered udp transport module.
    RPC: Registered tcp transport module.
    RPC: Registered tcp NFSv4.1 backchannel transport module.
    hw perfevents: enabled with armv7_cortex_a9 PMU driver, 1 counters available
    futex hash table entries: 512 (order: 3, 32768 bytes)
    workingset: timestamp_bits=30 max_order=17 bucket_order=0
    jffs2: version 2.2. (NAND) (SUMMARY)  © 2001-2006 Red Hat, Inc.
    io scheduler noop registered
    io scheduler deadline registered
    io scheduler cfq registered (default)
    dma-pl330 f8003000.dmac: Loaded driver for PL330 DMAC-241330
    dma-pl330 f8003000.dmac:  DBUFF-256x8bytes Num_Chans-8 Num_Peri-4 Num_Events-16
    xilinx-vdma 43000000.dma: Xilinx AXI VDMA Engine Driver Probed!!
    e0001000.serial: ttyPS0 at MMIO 0xe0001000 (irq = 26, base_baud = 1488095) is a xuartps
    console [ttyPS0] enabled
    console [ttyPS0] enabled
    bootconsole [earlycon0] disabled
    bootconsole [earlycon0] disabled
    xdevcfg f8007000.devcfg: ioremap 0xf8007000 to e0828000
    [drm] Initialized
    [drm] load() is defered & will be called again
    brd: module loaded
    loop: module loaded
    m25p80 spi0.0: m25p80 (1024 Kbytes)
    4 ofpart partitions found on MTD device spi0.0
    Creating 4 MTD partitions on "spi0.0":
    0x000000000000-0x000000500000 : "boot"
    mtd: partition "boot" extends beyond the end of device "spi0.0" -- size truncated to 0x100000
    0x000000500000-0x000000520000 : "bootenv"
    mtd: partition "bootenv" is out of reach -- disabled
    0x000000520000-0x000000fa0000 : "kernel"
    mtd: partition "kernel" is out of reach -- disabled
    0x000000fa0000-0x000000100000 : "spare"
    mtd: partition "spare" is out of reach -- disabled
    libphy: Fixed MDIO Bus: probed
    CAN device driver interface
    libphy: MACB_mii_bus: probed
    macb e000b000.ethernet eth0: Cadence GEM rev 0x00020118 at 0xe000b000 irq 28 (52:54:00:12:34:56)
    Marvell 88E1149R e000b000.etherne:01: attached PHY driver [Marvell 88E1149R] (mii_bus:phy_addr=e000b000.etherne:01, irq=-1)
    e1000e: Intel(R) PRO/1000 Network Driver - 3.2.6-k
    e1000e: Copyright(c) 1999 - 2015 Intel Corporation.
    ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
    ehci-pci: EHCI PCI platform driver
    usbcore: registered new interface driver usb-storage
    e0002000.usb supply vbus not found, using dummy regulator
    ULPI transceiver vendor/product ID 0x0000/0x0000
    ULPI integrity check: failed!ci_hdrc ci_hdrc.0: unable to init phy: -19
    mousedev: PS/2 mouse device common for all mice
    i2c /dev entries driver
    cdns-i2c e0004000.i2c: 400 kHz mmio e0004000 irq 22
    cdns-i2c e0004000.i2c: timeout waiting on completion
    cdns-i2c e0005000.i2c: 400 kHz mmio e0005000 irq 23
    uvcvideo: Unable to create debugfs directory
    usbcore: registered new interface driver uvcvideo
    USB Video Class driver (1.1.1)
    cdns-wdt f8005000.watchdog: Xilinx Watchdog Timer at e08ec000 with timeout 10s
    EDAC MC: ECC not enabled
    Xilinx Zynq CpuIdle Driver started
    sdhci: Secure Digital Host Controller Interface driver
    sdhci: Copyright(c) Pierre Ossman
    sdhci-pltfm: SDHCI platform and OF driver helper
    mmc0: SDHCI controller on e0100000.sdhci [e0100000.sdhci] using ADMA 64-bit
    ledtrig-cpu: registered to indicate activity on CPUs
    usbcore: registered new interface driver usbhid
    usbhid: USB HID core driver
    NET: Registered protocol family 10
    sit: IPv6, IPv4 and MPLS over IPv4 tunneling driver
    NET: Registered protocol family 17
    can: controller area network core (rev 20120528 abi 9)
    NET: Registered protocol family 29
    can: raw protocol (rev 20120528)
    can: broadcast manager protocol (rev 20161123 t)
    can: netlink gateway (rev 20130117) max_hops=1
    Registering SWP/SWPB emulation handler
    [drm] No max horizontal width in DT, using default 1920
    [drm] No max vertical height in DT, using default 1080
    OF: graph: no port node found in /amba_pl/xilinx_drm
    [drm] Supports vblank timestamp caching Rev 2 (21.10.2013).
    [drm] No driver support for vblank timestamp query.
    xilinx-drm amba_pl:xilinx_drm: No connectors reported connected with modes
    [drm] Cannot find any crtc or sizes - going 1024x768
    Console: switching to colour frame buffer device 128x48
    xilinx-drm amba_pl:xilinx_drm: fb0:  frame buffer device
    [drm] Initialized xilinx_drm 1.0.0 20130509 on minor 0
    cdns-i2c e0004000.i2c: timeout waiting on completion
    ssm2602 0-001a: Failed to issue reset: -110
    ssm2602 0-001a: ASoC: failed to probe component -110
    asoc-simple-card amba_pl:sound: ASoC: failed to instantiate card -110
    asoc-simple-card: probe of amba_pl:sound failed with error -110
    hctosys: unable to open rtc device (rtc0)
    of_cfs_init
    of_cfs_init: OK
    ALSA device list:
      No soundcards found.
    Warning: unable to open an initial console.
    Freeing unused kernel memory: 44032K (c0900000 - c3400000)    
    udevd[776]: starting version 3.2
    random: udevd: uninitialized urandom read (16 bytes read)
    random: udevd: uninitialized urandom read (16 bytes read)
    random: udevd: uninitialized urandom read (16 bytes read)
    udevd[777]: starting eudev-3.2
    random: udevd: uninitialized urandom read (16 bytes read)
    random: dd: uninitialized urandom read (512 bytes read)
    IPv6: ADDRCONF(NETDEV_UP): eth0: link is not ready
    macb e000b000.ethernet eth0: link up (100/Full)
    IPv6: ADDRCONF(NETDEV_CHANGE): eth0: link becomes ready
    random: dropbearkey: uninitialized urandom read (32 bytes read)
    random: dropbearkey: uninitialized urandom read (32 bytes read)
    random: dropbearkey: uninitialized urandom read (32 bytes read)
    random: dropbear: uninitialized urandom read (32 bytes read)
    random: tcf-agent: uninitialized urandom read (16 bytes read)
    /bin/autologin: line 1: -e: command not found
    root@Zybo:~#
    ```


### Package the project
```shell
petalinux-package --boot --force --fsbl images/linux/zynq_fsbl.elf \
  --fpga images/linux/system_wrapper.bit --u-boot
```


### Prepare SD card
Instructions from: https://reference.digilentinc.com/learn/programmable-logic/tutorials/zybo-zybot-guide/start

Identify the SD card in Ubuntu
```shell
lsblk
```

??? abstract "Output"
    ```shell
    ubuntu@ubuntu:~/Downloads/Zybo$ lsblk
    NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sr0     11:0    1 1024M  0 rom  
    sdd      8:32   1 29.7G  0 disk 
    ├─sdd2   8:34   1 29.5G  0 part /media/ubuntu/UNTITLED
    └─sdd1   8:33   1  200M  0 part 
    sda      8:0    0   64G  0 disk 
    ├─sda2   8:2    0    1K  0 part 
    ├─sda5   8:5    0    2G  0 part [SWAP]
    └─sda1   8:1    0   62G  0 part /
    sr1     11:1    1 1024M  0 rom  
    ```
SD card is at '/dev/sdd' in this example.

The first thing we need to do is to unmount the SD card:
```shell
umount /media/ubuntu/UNTITLED
```

Open fdisk with the SD card selected
```shell
sudo fdisk /dev/sdd
```

Delete current partitions, run twice and accept defaults:
```shell
d
<ENTER>
d
```

??? abstract "Output"
    ```shell
    Command (m for help): d
    Partition number (1,2, default 2): 

    Partition 2 has been deleted.

    Command (m for help): d
    Selected partition 1
    Partition 1 has been deleted.
    ```


Write partition 1 (BOOT):
```shell
n
<ENTER> # (partition number 1)
<ENTER> # (first sector default)
+1G # (1 GB for this partition)
```

??? abstract "Output"
    ```shell
    Command (m for help): n
    Partition number (1-128, default 1): 
    First sector (34-62333918, default 2048): 
    Last sector, +sectors or +size{K,M,G,T,P} (2048-62333918, default 62333918): +1G

    Created a new partition 1 of type 'Linux filesystem' and of size 1 GiB.
    ```

Write partition 2 (rootfs):
```shell
n
<ENTER> # (partition number 2)
<ENTER> # (first sector default)
<ENTER> # (use the remainder of the card for this partition)
```

??? abstract "Output"
    ```shell
    Command (m for help): n
    Partition number (2-128, default 2): 
    First sector (2099200-62333918, default 2099200): 
    Last sector, +sectors or +size{K,M,G,T,P} (2099200-62333918, default 62333918): 

    Created a new partition 2 of type 'Linux filesystem' and of size 28.7 GiB.
    ```

Write to the SD card:
```shell
w
```

??? abstract "Output"
    ```shell
    Command (m for help): w
    The partition table has been altered.
    Calling ioctl() to re-read partition table.
    Syncing disks.
    ```

Verify the new partitioning:
```shell
lsblk
```

??? abstract "Output"
    ```shell
    ubuntu@ubuntu:~/Downloads/Zybo$ lsblk
    NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sr0     11:0    1 1024M  0 rom  
    sdd      8:32   1 29.7G  0 disk 
    ├─sdd2   8:34   1 28.7G  0 part 
    └─sdd1   8:33   1    1G  0 part 
    sda      8:0    0   64G  0 disk 
    ├─sda2   8:2    0    1K  0 part 
    ├─sda5   8:5    0    2G  0 part [SWAP]
    └─sda1   8:1    0   62G  0 part /
    sr1     11:1    1 1024M  0 rom 
    ```


Create File Allocation Table on partition 1:
```shell
sudo mkfs -t vfat -n BOOT /dev/sdd1
```

??? abstract "Output"
    ```shell
    ubuntu@ubuntu:~/Downloads/Zybo$ sudo mkfs -t vfat -n BOOT /dev/sdd1
    mkfs.fat 3.0.28 (2015-05-16)
    ```


Create ext4 file system on partition 2:
```shell
sudo mkfs -t ext4 -L rootfs /dev/sdd2
```

??? abstract "Output"
    ```shell
    ubuntu@ubuntu:~/Downloads/Zybo$ sudo mkfs -t ext4 -L rootfs /dev/sdd2
    mke2fs 1.42.13 (17-May-2015)
    Creating filesystem with 7529339 4k blocks and 1884160 inodes
    Filesystem UUID: 49004b23-2a3d-4ae4-a9ec-c51f0a2309c2
    Superblock backups stored on blocks: 
      32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
      4096000

    Allocating group tables: done                            
    Writing inode tables: done                            
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done  
    ```


### Mount the SD card
```shell
sudo mkdir -p /media/ubuntu/BOOT
sudo mkdir -p /media/ubuntu/rootfs
sudo mount /dev/sdd1 /media/ubuntu/BOOT
sudo mount /dev/sdd2 /media/ubuntu/rootfs
lsblk
```

??? abstract "Output"
    ```shell
    ubuntu@ubuntu:~/Downloads/Zybo$ sudo mkdir -p /media/ubuntu/BOOT
    ubuntu@ubuntu:~/Downloads/Zybo$ sudo mkdir -p /media/ubuntu/rootfs
    ubuntu@ubuntu:~/Downloads/Zybo$ sudo mount /dev/sdd1 /media/ubuntu/BOOT
    ubuntu@ubuntu:~/Downloads/Zybo$ sudo mount /dev/sdd2 /media/ubuntu/rootfs
    ubuntu@ubuntu:~/Downloads/Zybo$ lsblk
    NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sr0     11:0    1 1024M  0 rom  
    sdd      8:32   1 29.7G  0 disk 
    ├─sdd2   8:34   1 28.7G  0 part /media/ubuntu/rootfs
    └─sdd1   8:33   1    1G  0 part /media/ubuntu/BOOT
    sda      8:0    0   64G  0 disk 
    ├─sda2   8:2    0    1K  0 part 
    ├─sda5   8:5    0    2G  0 part [SWAP]
    └─sda1   8:1    0   62G  0 part /
    sr1     11:1    1 1024M  0 rom 
    ```



### Configure SD rootfs
From: https://github.com/Digilent/Petalinux-Zybo/blob/v2017.4-1/README.md#configure-sd-rootfs
```shell
petalinux-config
-> Image Packaging Configuration -> Root filesystem type -> SD card
Save
Exit
```

??? abstract "Output"
    ```shell
    ubuntu@ubuntu:~/Downloads/Zybo$ petalinux-config
    [INFO] generating Kconfig for project
                                                                                                                             
    [INFO] menuconfig project
    /home/ubuntu/Downloads/Zybo/build/misc/config/Kconfig.syshw:30:warning: defaults for choice values not supported
    /home/ubuntu/Downloads/Zybo/build/misc/config/Kconfig:568:warning: config symbol defined without type


    *** End of the configuration.
    *** Execute 'make' to start the build or try 'make help'.

    [INFO] sourcing bitbake
    [INFO] generating plnxtool conf
    [INFO] generating meta-plnx-generated layer
    ~/Downloads/Zybo/build/misc/plnx-generated ~/Downloads/Zybo
    ~/Downloads/Zybo
    [INFO] generating machine configuration
    [INFO] generating bbappends for project . This may take time ! 
    ~/Downloads/Zybo/build/misc/plnx-generated ~/Downloads/Zybo
    ~/Downloads/Zybo
    [INFO] generating u-boot configuration files
                                                                                                                             
    [INFO] generating kernel configuration files
    [INFO] generating kconfig for Rootfs
    Generate rootfs kconfig
    [INFO] oldconfig rootfs
    [INFO] generating petalinux-user-image.bb
    [INFO] successfully configured project
    webtalk failed:PetaLinux statistics:extra lines detected:notsent_nofile!
    webtalk failed:Failed to get PetaLinux usage statistics!
    ```

Edit device tree:
```shell
project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi
# Find line with "bootargs" that says:
bootargs = "console=ttyPS0,115200 earlyprintk uio_pdrv_genirq.of_id=generic-uio";
# Replace with the following:
bootargs = "console=ttyPS0,115200 earlyprintk uio_pdrv_genirq.of_id=generic-uio root=/dev/mmcblk0p2 rw rootwait";
# Save and close
```

Build petalinux again
```shell
petalinux-build
```

??? abstract "Output"
    ```shell
    ubuntu@ubuntu:~/Downloads/Zybo$ petalinux-build
    [INFO] building project
    [INFO] sourcing bitbake
    INFO: bitbake petalinux-user-image
    Parsing recipes: 100% |###################################################################################| Time: 0:01:54
    Parsing of 2473 .bb files complete (0 cached, 2473 parsed). 3266 targets, 226 skipped, 0 masked, 0 errors.
    NOTE: Resolving any missing task queue dependencies
    Initialising tasks: 100% |################################################################################| Time: 0:00:09
    Checking sstate mirror object availability: 100% |########################################################| Time: 0:00:17
    NOTE: Executing SetScene Tasks
    NOTE: Executing RunQueue Tasks
    fsbl-2017.4+gitAUTOINC+77448ae629-r0 do_compile: NOTE: fsbl: compiling from external source tree /opt/pkg/petalinux/tools/hsm/data/embeddedsw
    NOTE: Tasks Summary: Attempted 2674 tasks of which 2595 didn't need to be rerun and all succeeded.
    INFO: Copying Images from deploy to images
    NOTE: Successfully copied built images to tftp dir:  /var/lib/tftpboot
    [INFO] successfully built project
    webtalk failed:PetaLinux statistics:extra lines detected:notsent_nofile!
    webtalk failed:Failed to get PetaLinux usage statistics!
    ```


### Copy files to SD card
```shell
sudo cp images/linux/BOOT.BIN /media/ubuntu/BOOT
sudo cp images/linux/image.ub /media/ubuntu/BOOT

sudo cp images/linux/rootfs.cpio /media/ubuntu/rootfs
```

### Unpack filesystem on SD card
From: https://reference.digilentinc.com/reference/software/petalinux#boot_options
```shell
cd /media/ubuntu/rootfs
sudo pax -rvf rootfs.cpio
cd ~/Downloads/Zybo
```

### Unmount SD card
```shell
sudo umount /media/ubuntu/BOOT /media/ubuntu/rootfs
```


### Boot on Zybo board
- Power off the board
- Insert SD card
- Move JP5 to SD position
- Insert micro USB cable to PROG/UART from host computer
- Power on the board
- Open a serial port on host computer with 115200/8/N/1 settings
- Reset by pressing  PS-SRST

??? success "Output"
    ```shell
    U-Boot 2017.01 (Apr 24 2021 - 17:52:44 +0200)

    Model: Zynq Zybo Development Board
    Board: Xilinx Zynq
    I2C:   ready
    DRAM:  ECC disabled 512 MiB
    MMC:   sdhci@e0100000: 0 (SD)
    Using default environment

    In:    serial
    Out:   serial
    Err:   serial
    Net:   ZYNQ GEM: e000b000, phyaddr 1, interface rgmii-id

    Warning: ethernet@e000b000 using MAC address from ROM
    eth0: ethernet@e000b000
    U-BOOT for Zybo
    ethernet@e000b000 Waiting for PHY auto negotiation to complete......... TIMEOUT !
    Hit any key to stop autoboot:  4 ... 3 ... 2 ... 1 ... 0 
    Device: sdhci@e0100000
    Manufacturer ID: 74
    OEM: 4a60
    Name: USDU1 
    Tran Speed: 50000000
    Rd Block Len: 512
    SD version 3.0
    High Capacity: Yes
    Capacity: 29.4 GiB
    Bus Width: 4-bit
    Erase Group Size: 512 Bytes
    reading image.ub
    3838864 bytes read in 332 ms (11 MiB/s)
    ## Loading kernel from FIT Image at 10000000 ...
       Using 'conf@2' configuration
       Verifying Hash Integrity ... OK
       Trying 'kernel@0' kernel subimage
         Description:  Linux Kernel
         Type:         Kernel Image
         Compression:  uncompressed
         Data Start:   0x100000d4
         Data Size:    3812944 Bytes = 3.6 MiB
         Architecture: ARM
         OS:           Linux
         Load Address: 0x00008000
         Entry Point:  0x00008000
         Hash algo:    sha1
         Hash value:   863c170235c173204916a246127eef3e56e6d13d
       Verifying Hash Integrity ... sha1+ OK
    ## Loading fdt from FIT Image at 10000000 ...
       Using 'conf@2' configuration
       Trying 'fdt@0' fdt subimage
         Description:  Flattened Device Tree blob
         Type:         Flat Device Tree
         Compression:  uncompressed
         Data Start:   0x103a3018
         Data Size:    24625 Bytes = 24 KiB
         Architecture: ARM
         Hash algo:    sha1
         Hash value:   806f3e9f43f0ed59b183c25833618dfa50cffca0
       Verifying Hash Integrity ... sha1+ OK
       Booting using the fdt blob at 0x103a3018
       Loading Kernel Image ... OK
       Loading Device Tree to 07ff6000, end 07fff030 ... OK

    Starting kernel ...
    Uncompressing Linux... done, booting the kernel.
    Booting Linux on physical CPU 0x0
    Linux version 4.9.0-xilinx-v2017.4 (ubuntu@ubuntu) (gcc version 6.2.1 20161016 (Linaro GCC 6.2-2016.11) ) #4 SMP PREEMPT Sun Apr 25 13:19:14 CEST 2021
    CPU: ARMv7 Processor [413fc090] revision 0 (ARMv7), cr=18c5387d
    CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
    OF: fdt:Machine model: Zynq Zybo Development Board
    bootconsole [earlycon0] enabled
    cma: Reserved 64 MiB at 0x1c000000
    Memory policy: Data cache writealloc
    percpu: Embedded 14 pages/cpu @dbbad000 s25932 r8192 d23220 u57344
    Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 130048
    Kernel command line: console=ttyPS0,115200 earlyprintk uio_pdrv_genirq.of_id=generic-uio root=/dev/mmcblk0p2 rw rootwait
    PID hash table entries: 2048 (order: 1, 8192 bytes)
    Dentry cache hash table entries: 65536 (order: 6, 262144 bytes)
    Inode-cache hash table entries: 32768 (order: 5, 131072 bytes)
    Memory: 443744K/524288K available (6144K kernel code, 208K rwdata, 1492K rodata, 1024K init, 231K bss, 15008K reserved, 65536K cma-reserved, 0K highmem)
    Virtual kernel memory layout:
        vector  : 0xffff0000 - 0xffff1000   (   4 kB)
        fixmap  : 0xffc00000 - 0xfff00000   (3072 kB)
        vmalloc : 0xe0800000 - 0xff800000   ( 496 MB)
        lowmem  : 0xc0000000 - 0xe0000000   ( 512 MB)
        pkmap   : 0xbfe00000 - 0xc0000000   (   2 MB)
        modules : 0xbf000000 - 0xbfe00000   (  14 MB)
          .text : 0xc0008000 - 0xc0700000   (7136 kB)
          .init : 0xc0900000 - 0xc0a00000   (1024 kB)
          .data : 0xc0a00000 - 0xc0a34380   ( 209 kB)
           .bss : 0xc0a34380 - 0xc0a6e118   ( 232 kB)
    Preemptible hierarchical RCU implementation.
    .Build-time adjustment of leaf fanout to 32.
    .RCU restricting CPUs from NR_CPUS=4 to nr_cpu_ids=2.
    RCU: Adjusting geometry for rcu_fanout_leaf=32, nr_cpu_ids=2
    NR_IRQS:16 nr_irqs:16 16
    efuse mapped to e0800000
    slcr mapped to e0802000
    L2C: platform modifies aux control register: 0x72360000 -> 0x72760000
    L2C: DT/platform modifies aux control register: 0x72360000 -> 0x72760000
    L2C-310 erratum 769419 enabled
    L2C-310 enabling early BRESP for Cortex-A9
    L2C-310 full line of zeros enabled for Cortex-A9
    L2C-310 ID prefetch enabled, offset 1 lines
    L2C-310 dynamic clock gating enabled, standby mode enabled
    L2C-310 cache controller enabled, 8 ways, 512 kB
    L2C-310: CACHE_ID 0x410000c8, AUX_CTRL 0x76760001
    zynq_clock_init: clkc starts at e0802100
    Zynq clock init
    sched_clock: 64 bits at 325MHz, resolution 3ns, wraps every 4398046511103ns
    clocksource: arm_global_timer: mask: 0xffffffffffffffff max_cycles: 0x4af477f6aa, max_idle_ns: 440795207830 ns
    Switching to timer-based delay loop, resolution 3ns
    clocksource: ttc_clocksource: mask: 0xffff max_cycles: 0xffff, max_idle_ns: 551318127 ns
    timer #0 at e080a000, irq=17
    Console: colour dummy device 80x30
    Calibrating delay loop (skipped), value calculated using timer frequency.. 650.00 BogoMIPS (lpj=3250000)
    pid_max: default: 32768 minimum: 301
    Mount-cache hash table entries: 1024 (order: 0, 4096 bytes)
    Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes)
    CPU: Testing write buffer coherency: ok
    CPU0: thread -1, cpu 0, socket 0, mpidr 80000000
    Setting up static identity map for 0x100000 - 0x100058
    CPU1: thread -1, cpu 1, socket 0, mpidr 80000001
    Brought up 2 CPUs
    SMP: Total of 2 processors activated (1300.00 BogoMIPS).
    CPU: All CPU(s) started in SVC mode.
    devtmpfs: initialized
    VFP support v0.3: implementor 41 architecture 3 part 30 variant 9 rev 4
    clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
    pinctrl core: initialized pinctrl subsystem
    NET: Registered protocol family 16
    DMA: preallocated 256 KiB pool for atomic coherent allocations
    cpuidle: using governor menu
    hw-breakpoint: found 5 (+1 reserved) breakpoint and 1 watchpoint registers.
    hw-breakpoint: maximum watchpoint size is 4 bytes.
    zynq-ocm f800c000.ocmc: ZYNQ OCM pool: 256 KiB @ 0xe0840000
    zynq-pinctrl 700.pinctrl: zynq pinctrl initialized
    vgaarb: loaded
    SCSI subsystem initialized
    usbcore: registered new interface driver usbfs
    usbcore: registered new interface driver hub
    usbcore: registered new device driver usb
    media: Linux media interface: v0.10
    Linux video capture interface: v2.00
    pps_core: LinuxPPS API ver. 1 registered
    pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
    PTP clock support registered
    EDAC MC: Ver: 3.0.0
    FPGA manager framework
    fpga-region fpga-full: FPGA Region probed
    Advanced Linux Sound Architecture Driver Initialized.
    clocksource: Switched to clocksource arm_global_timer
    NET: Registered protocol family 2
    TCP established hash table entries: 4096 (order: 2, 16384 bytes)
    TCP bind hash table entries: 4096 (order: 3, 32768 bytes)
    TCP: Hash tables configured (established 4096 bind 4096)
    UDP hash table entries: 256 (order: 1, 8192 bytes)
    UDP-Lite hash table entries: 256 (order: 1, 8192 bytes)
    NET: Registered protocol family 1
    RPC: Registered named UNIX socket transport module.
    RPC: Registered udp transport module.
    RPC: Registered tcp transport module.
    RPC: Registered tcp NFSv4.1 backchannel transport module.
    hw perfevents: enabled with armv7_cortex_a9 PMU driver, 7 counters available
    futex hash table entries: 512 (order: 3, 32768 bytes)
    workingset: timestamp_bits=30 max_order=17 bucket_order=0
    jffs2: version 2.2. (NAND) (SUMMARY)  ¬© 2001-2006 Red Hat, Inc.
    io scheduler noop registered
    io scheduler deadline registered
    io scheduler cfq registered (default)
    dma-pl330 f8003000.dmac: Loaded driver for PL330 DMAC-241330
    dma-pl330 f8003000.dmac: .DBUFF-128x8bytes Num_Chans-8 Num_Peri-4 Num_Events-16
    xilinx-vdma 43000000.dma: Xilinx AXI VDMA Engine Driver Probed!!
    e0001000.serial: ttyPS0 at MMIO 0xe0001000 (irq = 27, base_baud = 6250000) is a xuartps
    ¿console [ttyPS0] enabled
    console [ttyPS0] enabled
    bootconsole [earlycon0] disabled
    bootconsole [earlycon0] disabled
    xdevcfg f8007000.devcfg: ioremap 0xf8007000 to e0828000
    [drm] Initialized
    [drm] load() is defered & will be called again
    brd: module loaded
    loop: module loaded
    m25p80 spi0.0: found s25fl128s, expected m25p80
    m25p80 spi0.0: s25fl128s (16384 Kbytes)
    4 ofpart partitions found on MTD device spi0.0
    Creating 4 MTD partitions on "spi0.0":
    0x000000000000-0x000000500000 : "boot"
    0x000000500000-0x000000520000 : "bootenv"
    0x000000520000-0x000000fa0000 : "kernel"
    0x000000fa0000-0x000001000000 : "spare"
    libphy: Fixed MDIO Bus: probed
    CAN device driver interface
    libphy: MACB_mii_bus: probed
    macb e000b000.ethernet eth0: Cadence GEM rev 0x00020118 at 0xe000b000 irq 29 (d8:80:39:5c:17:ce)
    RTL8211E Gigabit Ethernet e000b000.etherne:01: attached PHY driver [RTL8211E Gigabit Ethernet] (mii_bus:phy_addr=e000b000.etherne:01, irq=-1)
    e1000e: Intel(R) PRO/1000 Network Driver - 3.2.6-k
    e1000e: Copyright(c) 1999 - 2015 Intel Corporation.
    ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
    ehci-pci: EHCI PCI platform driver
    usbcore: registered new interface driver usb-storage
    e0002000.usb supply vbus not found, using dummy regulator
    ULPI transceiver vendor/product ID 0x0424/0x0007
    Found SMSC USB3320 ULPI transceiver.
    ULPI integrity check: passed.
    ci_hdrc ci_hdrc.0: EHCI Host Controller
    ci_hdrc ci_hdrc.0: new USB bus registered, assigned bus number 1
    ci_hdrc ci_hdrc.0: USB 2.0 started, EHCI 1.00
    hub 1-0:1.0: USB hub found
    hub 1-0:1.0: 1 port detected
    mousedev: PS/2 mouse device common for all mice
    i2c /dev entries driver
    cdns-i2c e0004000.i2c: 400 kHz mmio e0004000 irq 23
    at24 0-0050: 256 byte 24c02 EEPROM, writable, 1 bytes/write
    cdns-i2c e0005000.i2c: 400 kHz mmio e0005000 irq 24
    uvcvideo: Unable to create debugfs directory
    usbcore: registered new interface driver uvcvideo
    USB Video Class driver (1.1.1)
    cdns-wdt f8005000.watchdog: Xilinx Watchdog Timer at e08ec000 with timeout 10s
    EDAC MC: ECC not enabled
    Xilinx Zynq CpuIdle Driver started
    sdhci: Secure Digital Host Controller Interface driver
    sdhci: Copyright(c) Pierre Ossman
    sdhci-pltfm: SDHCI platform and OF driver helper
    mmc0: SDHCI controller on e0100000.sdhci [e0100000.sdhci] using ADMA
    ledtrig-cpu: registered to indicate activity on CPUs
    usbcore: registered new interface driver usbhid
    usbhid: USB HID core driver
    NET: Registered protocol family 10
    sit: IPv6, IPv4 and MPLS over IPv4 tunneling driver
    NET: Registered protocol family 17
    can: controller area network core (rev 20120528 abi 9)
    NET: Registered protocol family 29
    can: raw protocol (rev 20120528)
    can: broadcast manager protocol (rev 20161123 t)
    can: netlink gateway (rev 20130117) max_hops=1
    Registering SWP/SWPB emulation handler
    [drm] No max horizontal width in DT, using default 1920
    [drm] No max vertical height in DT, using default 1080
    OF: graph: no port node found in /amba_pl/xilinx_drm
    [drm] Supports vblank timestamp caching Rev 2 (21.10.2013).
    [drm] No driver support for vblank timestamp query.
    mmc0: new high speed SDHC card at address 59b4
    mmcblk0: mmc0:59b4 USDU1 29.4 GiB 
     mmcblk0: p1 p2
    Console: switching to colour frame buffer device 240x67
    xilinx-drm amba_pl:xilinx_drm: fb0:  frame buffer device
    [drm] Initialized xilinx_drm 1.0.0 20130509 on minor 0
    ssm2602 0-001a: simple-card: set_sysclk error
    asoc-simple-card amba_pl:sound: ASoC: failed to init 43c20000.axi_i2s_adi-ssm2602-hifi: -22
    asoc-simple-card amba_pl:sound: ASoC: failed to instantiate card -22
    asoc-simple-card: probe of amba_pl:sound failed with error -22
    hctosys: unable to open rtc device (rtc0)
    of_cfs_init
    of_cfs_init: OK
    ALSA device list:
      No soundcards found.
    EXT4-fs (mmcblk0p2): couldn't mount as ext3 due to feature incompatibilities
    random: fast init done
    EXT4-fs (mmcblk0p2): recovery complete
    EXT4-fs (mmcblk0p2): mounted filesystem with ordered data mode. Opts: (null)
    VFS: Mounted root (ext4 filesystem) on device 179:2.
    devtmpfs: mounted
    Freeing unused kernel memory: 1024K (c0900000 - c0a00000)
    INIT: usb 1-1: new full-speed USB device number 2 using ci_hdrc
    version 2.88 booting
    input:   mini keyboard as /devices/soc0/amba/e0002000.usb/ci_hdrc.0/usb1/1-1/1-1:1.0/0003:1997:2433.0001/input/input0
    hid-generic 0003:1997:2433.0001: input: USB HID v1.01 Keyboard [  mini keyboard] on usb-ci_hdrc.0-1/input0
    input:   mini keyboard as /devices/soc0/amba/e0002000.usb/ci_hdrc.0/usb1/1-1/1-1:1.1/0003:1997:2433.0002/input/input1
    hid-generic 0003:1997:2433.0002: input: USB HID v1.01 Mouse [  mini keyboard] on usb-ci_hdrc.0-1/input1
    Starting udev
    input:   mini keyboard as /devices/soc0/amba/e0002000.usb/ci_hdrc.0/usb1/1-1/1-1:1.2/0003:1997:2433.0003/input/input2
    hid-generic 0003:1997:2433.0003: input: USB HID v1.01 Device [  mini keyboard] on usb-ci_hdrc.0-1/input2
    udevd[797]: starting version 3.2
    udevd[798]: starting eudev-3.2
    EXT4-fs (mmcblk0p2): re-mounted. Opts: data=ordered
    FAT-fs (mmcblk0p1): Volume was not properly unmounted. Some data may be corrupt. Please run fsck.
    hwclock: can't open '/dev/misc/rtc': No such file or directory
    Sun Apr 25 11:20:35 UTC 2021
    hwclock: can't open '/dev/misc/rtc': No such file or directory
    Starting internet superserver: inetd.
    INIT: Entering runlevel: 5
    Configuring network interfaces... IPv6: ADDRCONF(NETDEV_UP): eth0: link is not ready
    udhcpc (v1.24.1) started
    Sending discover...
    Sending discover...
    Sending discover...
    No lease, forking to background
    done.
    Starting Dropbear SSH server: dropbear.
    hwclock: can't open '/dev/misc/rtc': No such file or directory
    Starting syslogd/klogd: done
    Starting tcf-agent: OK
    /bin/autologin: line 1: -e: command not found
    .7.[r.[999;999H.[6nroot@Zybo:~#
    ```

Check of disk space:
```shell
root@Zybo:~# df -h
Filesystem                Size      Used Available Use% Mounted on
/dev/root                27.9G    298.9M     26.1G   1% /
devtmpfs                216.7M      4.0K    216.7M   0% /dev
tmpfs                   249.2M    124.0K    249.1M   0% /run
tmpfs                   249.2M     40.0K    249.1M   0% /var/volatile
/dev/mmcblk0p1         1022.0M      6.2M   1015.8M   1% /run/media/mmcblk0p1
```

Check of memory:
```shell
root@Zybo:~# free -m
             total       used       free     shared    buffers     cached
Mem:           498         36        461          0          4          7
-/+ buffers/cache:         23        474
Swap:            0          0          0
```

This shows that the second partition on the SD card is mounted properly and contains the root file system.
Otherwise the file system would be in RAM and we would have not have as much free RAM as we do now (461 MB out of 498 MB).


### Linaro Ubuntu Desktop
Info from: https://reference.digilentinc.com/learn/programmable-logic/tutorials/zybo-zybot-guide/start
```shell
sudo rm -r /media/ubuntu/rootfs/*
mkdir -p linaro
cd linaro
wget https://releases.linaro.org/archive/12.09/ubuntu/precise-images/ubuntu-desktop/linaro-precise-ubuntu-desktop-20120923-436.tar.gz .
sudo tar zxf *.tar.gz
sudo rsync -a ./ /media/ubuntu/rootfs
sudo umount /media/ubuntu/BOOT /media/ubuntu/rootfs
```

Color issue fix: https://forums.xilinx.com/t5/Video-and-Audio/Zybo-HDMI-output-Wrong-colors-rendering-on-Linux/td-p/950612




### Other resources
- https://www.instructables.com/Getting-Started-With-PetaLinux/
- https://www.instructables.com/Booting-Linux-on-the-ZYBO/
- https://www.youtube.com/watch?v=iDY5i1td4Wg

