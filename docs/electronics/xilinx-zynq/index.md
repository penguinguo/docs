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


??? example "Example output"
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

### Boot on Zybo board
==TODO==


### Other resources
- https://www.instructables.com/Getting-Started-With-PetaLinux/
- https://www.youtube.com/watch?v=iDY5i1td4Wg

