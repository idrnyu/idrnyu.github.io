---
title: 荔枝派_SPI-Flash烧录
date: 2022-05-16 20:59:10
tags:
---

烧录 `uboot` `Linux主线Kernel` `buildroot文件系统` 到 全志v3s；参考往期内容；

[荔枝派-u-boot编译](//2022/03/12/荔枝派-u-boot编译/)

[荔枝派-linux内核编译](//2022/05/15/荔枝派-linux内核编译/)

[荔枝派-buildroot文件系统编译](//2022/05/16/荔枝派-buildroot文件系统编译/)


# 一、 准备烧录文件
1. uboot编译后的产物：`u-boot-sunxi-with-spl.bin`;
2. linux内核编译后的产物：`arch/arm/boot/zImage` 和 `arch/arm/boot/dts/sun8i-v3s-licheepi-zero.dtb`;
3. buildroot根文件系统编译后的产物：`output/images/rootfs.tar`;

将4个文件放在一个文件夹下 如 v0.1

# 二、 将根文件系统生成jffs2.img

使用使用各种文件系统生成工具生成的文件系统文件一般都类似是： `rootfs.tar` 的压缩包文件，这个是无法没有办法适应nor flash的需要使用转换的工具软件转成类似`jffs2.img`的文件。

安装 jffs2 文件系统制作工具

```bash
sudo apt-get install mtd-utils
```

<!-- more -->

解压rootfs文件系统

```bash
mkdir rootfs
tar vxf rootfs.tar -C rootfs
```

使用命令生成：
这里使用的是16M的flash所以`-p`后面需要填的值即文件系统的空间为：16M-1M-64K-4M=0xAF 0000

```bash
mkfs.jffs2 -s 0x100 -e 0x10000 -p 0xAF0000 -d rootfs/ -o jffs2.img
```

>- `-s` 页大小0x100 即：256Byte (可选选项)
>- `-e` 块大小0x10000 即：64KByte (可选选项)
>- `-p` 十六进制表示输出的文件大小即文件系统的大小
>- `-d` 要做成img的源文件夹
>- jffs2分区总空间0xAF0000 即：10MB 又 960KB
>- jffs2.img是生成的文件系统镜像。

生成后

```bash
-rw-r--r--  1 gongyu gongyu 5.0M 5月  15 21:37 jffs2.img
```

# 三、 打包二进制包：flashimg.bin

>- 第一步： 生成一个空文件名字为flashimg.bin，大小是32MB或者是16MB
>- 第二步： 将uboot添加到文件开头
>- 第三步： 将dtb放到1M偏移处
>- 第四步： 将kernel放到1M+64K偏移处
>- 第五步： 将rootfs放到1M+64K+4M偏移处

偏移大小是seek，单位是KB。

16M Flash 打包脚本

```sh
#!/bin/sh
dd if=/dev/zero of=flashimg.bin bs=1M count=16
dd if=u-boot-sunxi-with-spl.bin of=flashimg.bin bs=1K conv=notrunc
dd if=sun8i-v3s-licheepi-zero.dtb of=flashimg.bin bs=1K seek=1024  conv=notrunc
dd if=zImage of=flashimg.bin bs=1K seek=1088  conv=notrunc
dd if=jffs2.img of=flashimg.bin  bs=1K seek=5184  conv=notrunc
```

32M Flash 打包脚本

```shell
#!/bin/sh
dd if=/dev/zero of=flashimg.bin bs=1M count=$1
dd if=u-boot-sunxi-with-spl-$2.bin of=flashimg.bin bs=1K conv=notrunc
dd if=sun8i-v3s-licheepi-zero-$2.dtb of=flashimg.bin bs=1K seek=1024  conv=notrunc
dd if=zImage of=flashimg.bin bs=1K seek=1088  conv=notrunc
dd if=jffs2.img of=flashimg.bin  bs=1K seek=5184  conv=notrunc
```

将对应的打包脚本命名并指定运行权限

```bash
vim bin.sh

# 输入 16M Flash 打包脚本 保存并退出
:wq

chmod 777 bin.sh  # 给予权限

./bin.sh # 运行脚本
```

![buildBin](/Dom/imgs/2022_05_16/buildBin.png)

烧录bin到 v3s

```bash
gongyu@gongyu-VirtualBox:~/Lichee_Pi/v1.1$ sudo sunxi-fel -p spiflash-write 0 flashimg.bin
100% [================================================] 16777 kB,   82.3 kB/s
```

烧录成功后重新重新上电板子 打开串口即可

```bash
sudo minicom -s # 配置串口
```
![minicom](/Dom/imgs/2022_05_16/minicom.png)
![setminicom](/Dom/imgs/2022_05_16/setminicom.png)

运行结果如下

```bash
Welcome to minicom 2.7

OPTIONS: I18n
Compiled on Apr 22 2017, 09:14:19.
Port /dev/ttyUSB0

Press CTRL-A Z for help on special keys


U-Boot SPL 2017.01-rc2-00073-gdd6e8740dc-dirty (May 15 2022 - 18:46:47)
DRAM: 64 MiB
Trying to boot from sunxi SPI

U-Boot 2017.01-rc2-00073-gdd6e8740dc-dirty (May 15 2022 - 18:46:47 +0800) Allwinner Technology

CPU:   Allwinner V3s (SUN8I 1681)
Model: Lichee Pi Zero
DRAM:  64 MiB
MMC:   SUNXI SD/MMC: 0
SF: Detected xt25f128b with page size 256 Bytes, erase size 4 KiB, total 16 MiB
*** Warning - bad CRC, using default environment

Setting up a 800x480 lcd console (overscan 0x0)
dotclock: 33000kHz = 33000kHz: (1 * 3MHz * 66) / 6
In:    serial@01c28000
Out:   serial@01c28000
Err:   serial@01c28000


U-Boot 2017.01-rc2-00073-gdd6e8740dc-dirty (May 15 2022 - 18:46:47 +0800) Allwinner Technology

CPU:   Allwinner V3s (SUN8I 1681)
Model: Lichee Pi Zero
DRAM:  64 MiB
MMC:   SUNXI SD/MMC: 0
SF: Detected xt25f128b with page size 256 Bytes, erase size 4 KiB, total 16 MiB
*** Warning - bad CRC, using default environment

Setting up a 800x480 lcd console (overscan 0x0)
dotclock: 33000kHz = 33000kHz: (1 * 3MHz * 66) / 6
In:    serial@01c28000
Out:   serial@01c28000
Err:   serial@01c28000
Net:   No ethernet found.
starting USB...
No controllers found
Hit any key to stop autoboot:  0
SF: Detected xt25f128b with page size 256 Bytes, erase size 4 KiB, total 16 MiB
device 0 offset 0x100000, size 0x10000
SF: 65536 bytes @ 0x100000 Read: OK
device 0 offset 0x110000, size 0x400000
SF: 4194304 bytes @ 0x110000 Read: OK
## Flattened Device Tree blob at 41800000
   Booting using the fdt blob at 0x41800000
   Loading Device Tree to 42dfa000, end 42dffc30 ... OK

Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 4.13.16-licheepi-zero+ (gongyu@gongyu-VirtualBox) (gcc version 6.3.1 20170404 (Linaro GCC 6.3-2017.05)) #3 SMP Sun M2
[    0.000000] CPU: ARMv7 Processor [410fc075] revision 5 (ARMv7), cr=10c5387d
[    0.000000] CPU: div instructions available: patching division code
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
[    0.000000] OF: fdt: Machine model: Lichee Pi Zero
[    0.000000] Memory policy: Data cache writealloc
[    0.000000] percpu: Embedded 16 pages/cpu @c3de6000 s33868 r8192 d23476 u65536
[    0.000000] Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 15883
[    0.000000] Kernel command line: console=ttyS0,115200 earlyprintk panic=5 rootwait mtdparts=spi32766.0:1M(uboot)ro,64k(dtb)ro,4M(kernel)ro,-(r2
[    0.000000] PID hash table entries: 256 (order: -2, 1024 bytes)
[    0.000000] Dentry cache hash table entries: 8192 (order: 3, 32768 bytes)
[    0.000000] Inode-cache hash table entries: 4096 (order: 2, 16384 bytes)
[    0.000000] Memory: 53584K/64036K available (6144K kernel code, 217K rwdata, 1456K rodata, 1024K init, 264K bss, 10452K reserved, 0K cma-reser)
[    0.000000] Virtual kernel memory layout:
[    0.000000]     vector  : 0xffff0000 - 0xffff1000   (   4 kB)
[    0.000000]     fixmap  : 0xffc00000 - 0xfff00000   (3072 kB)
[    0.000000]     vmalloc : 0xc4000000 - 0xff800000   ( 952 MB)
[    0.000000]     lowmem  : 0xc0000000 - 0xc3e89000   (  62 MB)
[    0.000000]     pkmap   : 0xbfe00000 - 0xc0000000   (   2 MB)
[    0.000000]     modules : 0xbf000000 - 0xbfe00000   (  14 MB)
[    0.000000]       .text : 0xc0008000 - 0xc0700000   (7136 kB)
[    0.000000]       .init : 0xc0900000 - 0xc0a00000   (1024 kB)
[    0.000000]       .data : 0xc0a00000 - 0xc0a367c0   ( 218 kB)
[    0.000000]        .bss : 0xc0a3daf0 - 0xc0a7fcac   ( 265 kB)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000] Hierarchical RCU implementation.
[    0.000000]  RCU event tracing is enabled.
[    0.000000]  RCU restricting CPUs from NR_CPUS=8 to nr_cpu_ids=1.
[    0.000000] RCU: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=1
[    0.000000] NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
[    0.000000] arch_timer: cp15 timer(s) running at 24.00MHz (virt).
[    0.000000] clocksource: arch_sys_counter: mask: 0xffffffffffffff max_cycles: 0x588fe9dc0, max_idle_ns: 440795202592 ns
[    0.000009] sched_clock: 56 bits at 24MHz, resolution 41ns, wraps every 4398046511097ns
[    0.000021] Switching to timer-based delay loop, resolution 41ns
[    0.000185] clocksource: timer: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 79635851949 ns
[    0.000409] Console: colour dummy device 80x30
[    0.000447] Calibrating delay loop (skipped), value calculated using timer frequency.. 48.00 BogoMIPS (lpj=240000)
[    0.000462] pid_max: default: 32768 minimum: 301
[    0.000591] Mount-cache hash table entries: 1024 (order: 0, 4096 bytes)
[    0.000607] Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes)
[    0.001201] CPU: Testing write buffer coherency: ok
[    0.001571] /cpus/cpu@0 missing clock-frequency property
[    0.001594] CPU0: thread -1, cpu 0, socket 0, mpidr 80000000
[    0.002022] Setting up static identity map for 0x40100000 - 0x40100060
[    0.002208] Hierarchical SRCU implementation.
[    0.002727] smp: Bringing up secondary CPUs ...
[    0.002740] smp: Brought up 1 node, 1 CPU
[    0.002749] SMP: Total of 1 processors activated (48.00 BogoMIPS).
[    0.002756] CPU: All CPU(s) started in SVC mode.
[    0.003517] devtmpfs: initialized
[    0.006545] VFP support v0.3: implementor 41 architecture 2 part 30 variant 7 rev 5
[    0.006807] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
[    0.006839] futex hash table entries: 256 (order: 2, 16384 bytes)
[    0.007006] pinctrl core: initialized pinctrl subsystem
[    0.007883] random: get_random_u32 called from bucket_table_alloc+0xf4/0x244 with crng_init=0
[    0.008014] NET: Registered protocol family 16
[    0.008504] DMA: preallocated 256 KiB pool for atomic coherent allocations
[    0.009612] hw-breakpoint: found 5 (+1 reserved) breakpoint and 4 watchpoint registers.
[    0.009629] hw-breakpoint: maximum watchpoint size is 8 bytes.
[    0.022177] SCSI subsystem initialized
[    0.022469] usbcore: registered new interface driver usbfs
[    0.022535] usbcore: registered new interface driver hub
[    0.022631] usbcore: registered new device driver usb
[    0.022870] pps_core: LinuxPPS API ver. 1 registered
[    0.022882] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    0.022905] PTP clock support registered
[    0.023130] Advanced Linux Sound Architecture Driver Initialized.
[    0.024937] clocksource: Switched to clocksource arch_sys_counter
[    0.035847] NET: Registered protocol family 2
[    0.036437] TCP established hash table entries: 1024 (order: 0, 4096 bytes)
[    0.036470] TCP bind hash table entries: 1024 (order: 1, 8192 bytes)
[    0.036494] TCP: Hash tables configured (established 1024 bind 1024)
[    0.036622] UDP hash table entries: 256 (order: 1, 8192 bytes)
[    0.036672] UDP-Lite hash table entries: 256 (order: 1, 8192 bytes)
[    0.036913] NET: Registered protocol family 1
[    0.037502] RPC: Registered named UNIX socket transport module.
[    0.037520] RPC: Registered udp transport module.
[    0.037525] RPC: Registered tcp transport module.
[    0.037531] RPC: Registered tcp NFSv4.1 backchannel transport module.
[    0.039598] workingset: timestamp_bits=30 max_order=14 bucket_order=0
[    0.048527] NFS: Registering the id_resolver key type
[    0.048579] Key type id_resolver registered
[    0.048586] Key type id_legacy registered
[    0.048630] jffs2: version 2.2. (NAND) Â© 2001-2006 Red Hat, Inc.
[    0.050156] random: fast init done
[    0.053011] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 249)
[    0.053031] io scheduler noop registered
[    0.053039] io scheduler deadline registered
[    0.053310] io scheduler cfq registered (default)
[    0.053322] io scheduler mq-deadline registered
[    0.053329] io scheduler kyber registered
[    0.057685] sun8i-v3s-pinctrl 1c20800.pinctrl: initialized sunXi PIO driver
[    0.126907] Serial: 8250/16550 driver, 8 ports, IRQ sharing disabled
[    0.130262] console [ttyS0] disabled
[    0.150544] 1c28000.serial: ttyS0 at MMIO 0x1c28000 (irq = 33, base_baud = 1500000) is a U6_16550A
[    0.739727] console [ttyS0] enabled
[    0.747572] m25p80 spi32766.0: xt25f128b (16384 Kbytes)
[    0.752833] spi32766.0: parser cmdlinepart: 4
[    0.757272] 4 cmdlinepart partitions found on MTD device spi32766.0
[    0.763533] Creating 4 MTD partitions on "spi32766.0":
[    0.768701] 0x000000000000-0x000000100000 : "uboot"
[    0.774206] 0x000000100000-0x000000110000 : "dtb"
[    0.779408] 0x000000110000-0x000000510000 : "kernel"
[    0.784706] 0x000000510000-0x000001000000 : "rootfs"
[    0.790529] libphy: Fixed MDIO Bus: probed
[    0.795117] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    0.801643] ehci-platform: EHCI generic platform driver
[    0.807174] ehci-platform 1c1a000.usb: EHCI Host Controller
[    0.812803] ehci-platform 1c1a000.usb: new USB bus registered, assigned bus number 1
[    0.820762] ehci-platform 1c1a000.usb: irq 25, io mem 0x01c1a000
[    0.854971] ehci-platform 1c1a000.usb: USB 2.0 started, EHCI 1.00
[    0.862206] hub 1-0:1.0: USB hub found
[    0.866158] hub 1-0:1.0: 1 port detected
[    0.870659] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[    0.876963] ohci-platform: OHCI generic platform driver
[    0.882528] ohci-platform 1c1a400.usb: Generic Platform OHCI controller
[    0.889256] ohci-platform 1c1a400.usb: new USB bus registered, assigned bus number 2
[    0.897218] ohci-platform 1c1a400.usb: irq 26, io mem 0x01c1a400
[    0.970040] hub 2-0:1.0: USB hub found
[    0.973864] hub 2-0:1.0: 1 port detected
[    0.981458] udc-core: couldn't find an available UDC - added [g_cdc] to list of pending drivers
[    0.991180] sun6i-rtc 1c20400.rtc: rtc core: registered rtc-sun6i as rtc0
[    0.998080] sun6i-rtc 1c20400.rtc: RTC enabled
[    1.002619] i2c /dev entries driver
[    1.007604] input: ns2009_ts as /devices/platform/soc/1c2ac00.i2c/i2c-0/0-0048/input/input0
[    1.017160] sunxi-wdt 1c20ca0.watchdog: Watchdog enabled (timeout=16 sec, nowayout=0)
[    1.085196] sunxi-mmc 1c0f000.mmc: base:0xc4077000 irq:23
[    1.092128] usbcore: registered new interface driver usbhid
[    1.097807] usbhid: USB HID core driver
[    1.103419] NET: Registered protocol family 17
[    1.108119] Key type dns_resolver registered
[    1.112543] Registering SWP/SWPB emulation handler
[    1.123073] simple-framebuffer 43e89000.framebuffer: framebuffer at 0x43e89000, 0x177000 bytes, mapped to 0xc4400000
[    1.133733] simple-framebuffer 43e89000.framebuffer: format=x8r8g8b8, mode=800x480x32, linelength=3200
[    1.152018] Console: switching to colour frame buffer device 100x30
[    1.164514] simple-framebuffer 43e89000.framebuffer: fb0: simplefb registered!
[    1.173084] usb_phy_generic usb_phy_generic.0.auto: usb_phy_generic.0.auto supply vcc not found, using dummy regulator
[    1.184473] musb-hdrc musb-hdrc.1.auto: MUSB HDRC host driver
[    1.190317] musb-hdrc musb-hdrc.1.auto: new USB bus registered, assigned bus number 3
[    1.199404] hub 3-0:1.0: USB hub found
[    1.203266] hub 3-0:1.0: 1 port detected
[    1.208558] using random self ethernet address
[    1.213049] using random host ethernet address
[    1.218666] usb0: HOST MAC 72:d4:27:dc:c4:8a
[    1.222985] usb0: MAC 5a:ef:c7:4f:6c:26
[    1.226971] g_cdc gadget: CDC Composite Gadget, version: King Kamehameha Day 2008
[    1.234448] g_cdc gadget: g_cdc ready
[    1.238489] sun6i-rtc 1c20400.rtc: setting system clock to 1970-01-01 00:00:12 UTC (12)
[    1.246780] vcc3v0: disabling
[    1.249752] vcc5v0: disabling
[    1.252716] ALSA device list:
[    1.255732]   No soundcards found.
[    1.299597] random: crng init done
[    2.213859] VFS: Mounted root (jffs2 filesystem) on device 31:3.
[    2.221316] devtmpfs: mounted
[    2.225633] Freeing unused kernel memory: 1024K
[    2.784077] g_cdc gadget: high-speed config #1: CDC Composite (ECM + ACM)
Starting syslogd: OK
Starting klogd: OK
Running sysctl: OK
Initializing random number generator... done.
Starting network: OK

Welcome to Buildroot
buildroot login: root
# cd /
# ls
bin      lib      media    proc     sbin     usr
dev      lib32    mnt      root     sys      var
etc      linuxrc  opt      run      tmp
#
CTRL-A Z for help | 115200 8N1 | NOR | Minicom 2.7 | VT102 | Offline | ttyUSB0

```
