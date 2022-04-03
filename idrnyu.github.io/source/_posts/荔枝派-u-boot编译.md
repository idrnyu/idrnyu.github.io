---
title: 荔枝派_u-boot编译
date: 2022-03-12 12:58:52
tags:
comments: false
---

# 一、基础编译 

`linux（ubuntu）`、`git`、`python2`、`全志v3s`、`5寸电阻触摸屏（RTB屏）`、`USB TO UART`

## 1、安装python2

````bash
sudo apt-get update # 更新源
sudo apt update

sudo apt-get install python2.7 # 安装python2.7
sudo apt install python-pip # 安装python2.7对应的pip

python -V # 查看python版本验证是否安装成功
pyp -V #

# 如果python -V 无法读取到版本信息 或者不是2.7的版本  需要创建一个快捷方式（软连接）
ls /usr/bin/python*  # 查看python安装状态

irtualBox:~/Lichee_Pi/u-boot$ ls /usr/bin/python*
/usr/bin/python2.7  /usr/bin/python3  /usr/bin/python3.8

# 创建软连接
sudo ln -s /usr/bin/python2.7 /usr/bin/python

ls /usr/bin/pip  # 查看安装目录
ls /usr/bin/python*

# 如需卸载 python2.7 与 python-pip 执行下面两条命令
sudo apt-get purge python2.7
sudo apt-get purge --auto-remove python2.7  # 卸载python 及其依赖

# python2 与 python3 不兼容
# 如果系统有安装python3 则需要重新安装python2 因为编译u-boot时的工具链里面需要python2的环境 否则报错

````
![python](/Dom/imgs/2022_03_12/python.png)

## 2、安装交叉编译器与工具链

```bash
gcc-arm-linux-gnueabihf
# ARMHF 架构交叉编译器 安装此版本 gcc-linaro-6.3.1-2017.02-x86_64_arm-linux-gnueabihf (需手动下载)

device-tree-compiler
# ARM Linux 设备树  DTC (device tree compiler) 将.dts编译为.dtb的工具  dtc编译器

libncurses5-dev
# curses库 游标移动及屏幕的显示库
```

### gcc-arm-linux-gnueabihf  安装

链接: https://pan.baidu.com/s/1VjjnR4zSwde1HUAbgcg64A?pwd=4jbn 提取码: 4jbn 复制这段内容后打开百度网盘手机App，操作更方便哦

```bash
tar xvf gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf.tar.xz # 解压
sudo mv gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf /opt/ # 移动解压后的文件到linux /opt/目录下
sudo vim /etc/bash.bashrc  # 编辑添加环境变量（永久生效）
# 在 `/etc/bash.basher` 文件中添加下面一行环境变量
PATH="$PATH:/opt/gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf/bin"

source /etc/bash.bashrc # 重新激活 可以在所有用户生效
arm-linux-gnueabihf-gcc -v # 查看交叉编译器版本号 确认是否配置成功
```

### device-tree-compiler  安装

```bash
sudo apt-get install device-tree-compiler # 使用 apt-get 安装
```

### libncurses5-dev 安装

```bash
sudo apt-get install libncurses5-dev
```



## 3、下载u-boot源码并编译 

### 下载 u-boot

```bash
# 使用 git 从 github clone 下来荔枝派官方u-boot v3s主线分支
git clone https://github.com/Lichee-Pi/u-boot.git -b v3s-current

# 或者克隆包含 spi-flash 驱动的体验版本uboot（适用于spi flash烧录），该驱动目前尚未合并到主线
git clone https://github.com/Lichee-Pi/u-boot.git -b v3s-spi-experimental

# clone 网络错误请使用代理
git clone https://github.com.cnpmjs.org/***   # 后面带git地址
# 或者使用镜像代理
https://gitclone.com/
```

### u-boot 目录结构

```bash
├── api                存放uboot提供的API接口函数
├── arch               平台
│   ├──arm
│   │   └──cpu
│   │   │   └──armv7
│   │   └──dts
│   │   │   └──*.dts   存放设备的dts,也就是设备配置相关的引脚信息
├── board              对于不同的平台的开发板对应的代码
├── cmd                大部分的命令的实现都在这个文件夹下面。
├── common             公共的代码
├── configs            各个板子的对应的配置文件都在里面，我们的Lichee配置也在里面
├── disk               对磁盘的一些操作都在这个文件夹里面，例如分区等。
├── doc                参考文档，平台等相关的使用文档。
├── drivers            驱动文件
├── dts                device tree
├── examples           官方给出的一些样例程序
├── fs                 文件系统
├── include            头文件
├── lib                常用库
├── Licenses           license声明
├── net                网络，小型网络协议栈
├── post               上电自检程序
├── scripts            编译脚本和Makefile文件
├── spl                second program loader，即相当于二级uboot启动。
├── test               单元测试程序。
└── tools              uboot常用工具。

```

### 编译 u-boot

```bash
cd u-boot # 打开 u-boot 目录
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LicheePi_Zero_800x480LCD_defconfig # 配置uboot
# 设置 make 宏
# ARCH：编译arch/arm下的代码；
# CROSS_COMPILE：编译器选arm-linux-gnueabihf-；
# LicheePi_Zero_800x480LCD_defconfig 使用 config 下 800*480LCD 的屏幕配置
# 或者使用 LicheePi_Zero480x272LCD_defconfig 或 LicheePi_Zero_defconfig 屏幕配置
# 也可以直接执行 `make LicheePi_Zero_800x480LCD_defconfig` 对荔枝派 Zero 显示设备进行配置

# 如果配置命令失败（错误信息如下）
#--------------------------------------------------------------------------------
gongyu@gongyu-VirtualBox:~/Lichee_Pi/u-boot$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LicheePi_Zero_800x480LCD_defconfig
  HOSTCC  scripts/basic/fixdep
/bin/sh: 1: cc: not found
make[1]: *** [scripts/Makefile.host:99：scripts/basic/fixdep] 错误 127
make: *** [Makefile:398：scripts_basic] 错误 2
#--------------------------------------------------------------------------------
# 如果配置命令失败 需要安装 bison、flex 这两个包
sudo apt install bison flex
```
![makeError1](/Dom/imgs/2022_03_12/makeError1.png)


```bash
make ARCH=arm menuconfig  # 打开配置菜单对u-boot配置

# 如果运行展开配置菜单报错（错误信息如下）
#--------------------------------------------------------------------------------
gongyu@DESKTOP-RQ1MEDL:/mnt/f/work/Lichee_Pi/u-boot$ make ARCH=arm menuconfig
  HOSTCC  scripts/kconfig/mconf.o
<command-line>: fatal error: curses.h: No such file or directory
compilation terminated.
make[1]: *** [scripts/Makefile.host:116: scripts/kconfig/mconf.o] Error 1
make: *** [Makefile:478: menuconfig] Error 2
#--------------------------------------------------------------------------------
# 如果运行展开配置菜单报错 需要安装 libncurses5-dev
sudo apt-get install libncurses5-dev

# 配置菜单配置完成后编辑并输入bin文件
time make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- 2>&1 | tee build.log # 编译u-boot 并输入日志
```

![image-20220312201449693](/Dom/imgs/2022_03_12/image-20220312201449693.png)

<strong style="color: red;"> 配置完u-boot后再编译 </strong>

编译完成后，在当前目录下生成了 u-boot-sunxi-with-spl.bin ，可以烧录到8K偏移处启动。（tf卡）

# 二、u-boot 配置

命令执行：`make ARCH=arm menuconfig` 使用菜单形势配置u-boot

![202203122156](/Dom/imgs/2022_03_12/202203122156.png)

## 1、简单配置

`Architecture select` 架构选择  选择arm架构

`ARM architecture` arm架构配置  lcd配置  ddr配置  芯片选型 等

![202203122214](/Dom/imgs/2022_03_12/202203122214.png)

`Boot images` cpu时钟配置 等

`delay in seconds before automatically booting` uboot开机的时候等待时间 默认为2s

`SPL / TPL` 外设配置

`Device Drivers` 设配驱动配置

## 2、配置 SPI Flash 

`由于目前Uboot环境变量固定存放在1MB位置之内，所有留给uboot的空间固定到flash前1MB的位置不变。每个分区的大小必须是擦除块大小的整数倍，XT25F128B的擦除块大小是64KB。`

| 分区序号 |   分区大小   |  分区描述  |        地址空间及分区名        |
| :------: | :----------: | :--------: | :----------------------------: |
|   mtd0   |     1MB      | spl+uboot  | 0x0000000-0x0100000 : “uboot”  |
|   mtd1   |     64KB     |  dtb文件   |   0x0100000-0x0110000: “dtb”   |
|   mtd2   |     4MB      | linux内核  | 0x0110000-0x0510000 : “kernel” |
|   mtd3   | 10MiB 960KiB | 根文件系统 | 0x0510000-0x1000000 : “rootfs” |

### 2.1、配置 u-boot 支持 Nor Flash 

`荔枝派 zero 上面焊接了一个 芯天下 的 Nor Flash 型号为：XT25F128B 16MByte。`

需要将 `u-boot` 切换到 `v3s-spi-experimental `分支下

`make ARCH=arm menuconfig` 打开u-boot菜单配置

在 u-boot 配置下选择自己对应的 SPI Flash 对应的芯片厂商

XT25F128B 与 Winbond 公司的 w25qxxx 系列的 flash 兼容性很高  所以选中 `Winbond SPI flash support`

```
Device Drivers  ---> 
	SPI Flash Support  ---> 
     ┌───────────────────────────────────────────────────────────────────────────────────────────┐ 
    │ │          [*] Enable Driver Model for SPI flash                                          │ │
    │ │          [*] Legacy SPI Flash Interface support                                         │ │
    │ │          [ ]   SPI flash Bank/Extended address register support                         │ │
    │ │          [ ]   Atmel SPI flash support                                                  │ │
    │ │          [ ]   EON SPI flash support                                                    │ │
    │ │          [ ]   GigaDevice SPI flash support                                             │ │
    │ │          [ ]   Macronix SPI flash support                                               │ │
    │ │          [ ]   Spansion SPI flash support                                               │ │
    │ │          [ ]   STMicro SPI flash support                                                │ │
    │ │          [ ]   SST SPI flash support                                                    │ │
    │ │          [*]   Winbond SPI flash support                                                │ │
    │ │          [*]   Use small 4096 B erase sectors                                           │ │
    │ │          [ ]   AT45xxx DataFlash support                                                │ │
    │ │          [ ]   SPI Flash MTD support                                                    │ │
    │ │          [*] Support for SPI Flash on Allwinner SoCs in SPL                             │ │
```

### 2.2、配置 uboot 默认环境变量对Nor Flash的支持

`在 include/configs/sun8i.h 中添加 CONFIG_BOOTCOMMAND 和 CONFIG_BOOTARGS 环境变量设置`

命令：`vim include/configs/sun8i.h` 在 `#include <configs/sunxi-common.h>` 前面添加以下代码

```c
#define CONFIG_BOOTCOMMAND   "sf probe 0; "                           \
                             "sf read 0x41800000 0x100000 0x10000; "  \
                             "sf read 0x41000000 0x110000 0x400000; " \
                             "bootz 0x41000000 - 0x41800000"

#define CONFIG_BOOTARGS      "console=ttyS0,115200 earlyprintk panic=5 rootwait " \
                             "mtdparts=spi32766.0:1M(uboot)ro,64k(dtb)ro,4M(kernel)ro,-(rootfs) root=31:03 rw rootfstype=jffs2"
```

![config_sun8i_h](/Dom/imgs/2022_03_12/config_sun8i_h.png)

#### （1）、环境命令解析：

- `sf probe 0;`   初始化 Flash 设备 （CS 拉低）
- `sf read 0x41800000 0x100000 0x10000;`   从 Flash 0x100000 （1MB） 位置读取 dtb 放到内存  0x41800000 偏移地址。如果是 bsp 的 bin， 则是 0x41d00000 偏移地址。
- `sf read 0x41000000 0x110000 0x400000;`    从 Flash 0x110000 （1MB + 64KB） 位置读取 dtb 放到内存 0x41000000 偏移地址。
- `bootz 0x41000000 - 0x41800000`    启动内核  0x41000000 内核地址；0x41800000 dtb 地址。

#### （2）、启动参数解析：

- `console=ttyS0,115200 earlyprintk panic=5 rootwait`    在串口0输出信息  115200 波特率
- `mtdparts=spi32766.0:1M(uboot)ro,64k(dtb)ro,4M(kernel)ro,-(rootfs) root=31:03 rw rootfstype=jffs2`   spi32766.0 是设备名，后面是分区大小，名字，读写属性。
- `root=31:03`    表示根文件系统时 mtd3；jffs2 格式。

### 2.3、配置 dtb 支持 Nor Flash （设备树驱动程序 ）@TODO

暂时不配置spi设备树，将uboot烧录到v3s中也可以运行，只是串口会输出错误信息，屏幕正常显示 log 和uboot版本信息

```bash
Hit any key to stop autoboot:  0 
SF: unrecognized JEDEC id bytes: 0b, 40, 18
Failed to initialize SPI flash at 0:0 (error -2)
No SPI flash selected. Please run `sf probe'
No SPI flash selected. Please run `sf probe'
=> 
```

#### （1）、修改uboot  spi_flash.c 文件

先查找错误位置 `vim drivers/mtd/spi/spi_flash.c`

![ubootError2](/Dom/imgs/2022_03_12/ubootError2.png)

添加XT25F128B SPI Flash id 到 `spi_flash_ids.c` `drivers/mtd/spi/spi_flash_ids.c`

由于需要移植的 flash 芯片特性和 w25qxxx 系列的 flash 相似，所以可以直接复制过来，修改后如下

![editspi_flash](/Dom/imgs/2022_03_12/editspi_flash.png)

0x0b4018 是uboot报错的id `SF: unrecognized JEDEC id bytes: 0b, 40, 18`

```bash
# 配置数据格式如下
/* Used when the "_ext_id" is two bytes at most */
#define INFO(_jedec_id, _ext_id, _sector_size, _n_sectors, _flags)    \
        .id = {                         \
            ...
        .id_len = (!(_jedec_id) ? 0 : (3 + ((_ext_id) ? 2 : 0))),   \
        .sector_size = (_sector_size),              \
        .n_sectors = (_n_sectors),              \
        .page_size = 256,                   \
        .flags = (_flags),
```

#### （2）、修改dts配置添加spi flash节点

修改 `vim arch/arm/dts/sun8i-v3s-licheepi-zero.dts` 文件

修改设备树中的 flash 相关的声明，添加上新增加的 flash 型号

```bash
&spi0 {
	status = "okay";
	
	xt25f128b:xt25f128b@0 {
		compatible = "winbond, xt25f128b", "jedec,spi-nor";
		reg = <0x0>;
		spi-max-frequency = <50000000>;
		#address-cells = <1>;
		#size-cells = <1>;
	};
};
```
![editdtsSpiFlash](/Dom/imgs/2022_03_12/editdtsSpiFlash.png)

#### （3）、重新编译uboot 并烧录

# 三、u-boot  SPI Flash 烧录

## 1、全志 `sunxi-tools` 烧录工具安装

### 1.1、安装

```bash
# 安装依赖包
sudo apt-get install pkg-config pkgconf zlib1g-dev libusb-1.0-0-dev
```
<span style="color: red;">安装时可能会报错</span>
![installError1](/Dom/imgs/2022_03_12/installError1.png)

好像 pkg-config 要后面装
```bash
sudo apt-get install pkgconf
sudo apt-get install zlib1g-dev
sudo apt-get install libusb-1.0-0-dev
sudo apt-get install pkg-config
```

```bash
# 获取源码
# v3s 分支
git clone -b v3s https://github.com/Icenowy/sunxi-tools.git

# v3s spiflash 分支 （荔枝派 Zero spi 版本）
git clone -b v3s-spi https://github.com/Icenowy/sunxi-tools.git

# f1c100s 分支
git clone -b f1c100s https://github.com/Icenowy/sunxi-tools.git

# f1c100s-spiflash 分支
git clone -b f1c100s-spiflash https://github.com/Icenowy/sunxi-tools.git

# 进入源码文件夹
cd sunxi-tools
# 编译和安装
make && sudo make install

```

### 1.2、基础用法

参考 [[编译和使用sunxi-tools]](http://nano.lichee.pro/step_by_step/two_sunxi-tools.html)

```bash
# 列出所有芯片的信息：
sudo sunxi-fel -l

# 查看芯片信息：
sudo sunxi-fel ver

# 加载并执行uboot的spl
sudo sunxi-fel spl 文件名

# 显示spiflash的信息 
sudo sunxi-fel spiflash-info

# 调用指定地址的函数 
sudo sunxi-fel exec 地址

# 把文件内容写入内存指定地址(-p是显示写入进度) 
sudo sunxi-fel -p write 地址 文件名

# 读取spiflash指定地址的数据并写入到文件 
sudo sunxi-fel spiflash-read 地址 长度 存放数据的文件路径

# 写入指定文件的指定长度的内容到spiflash的指定地址 
sudo sunxi-fel spiflash-write 地址 长度 存放数据的文件路径

```

## 2、进入 FEL 模式使用 `sunxi-fel` 工具烧录 （全志专用工具）

>- <strong>TF卡和 spi flash 都没有可启动的镜像</strong>
>
>TF卡空或者不插，spi flash 内容为空，上吊就会自动进入 fel 下载模式
>
>- <strong>TF卡中有进入FEL模式的特殊固件 `el-sdboot.sunxi` </strong>
>
>如果 spi flash 有启动镜像，那么需要在 TF 卡中烧录一个 sunxi 提供的启动工具，那么插入该 TF 卡上电会进入 fel 模式；
>
>命令： `dd if=fel-sdboot.sunxi of=/dev/mmcblk0 bs=1024 seek=8`
>
>- <strong>上电时将 SPI_MISO 引脚拉为低电平</strong>
>
>该引脚为 `boot` 引脚，上电时如果检测到该引脚为低电平就会进入 fel 下载模式。

## 3、准备烧录uboot（SPI Flash）

```bash
//检查是否可以探测到芯片信息：
sudo sunxi-fel -l
//确认是否成功进入fel模式：
sudo sunxi-fel ver
```

![20220325151309](/Dom/imgs/2022_03_12/20220325151309.png)

### 3.1、烧录到 RAM 中执行，以 uboot file-with-spl 形式运行（单次运行，测试使用）

```bash
sudo sunxi-fel uboot u-boot-sunxi-with-spl.bin

//或者执行烧录地址
sudo sunxi-fel -p write 0x40000000 u-boot-sunxi-with-spl.bin
sudo sunxi-fel exec 0x40000000
```

### 3.2、烧录到 SPI Flash （烧录完成后重新上电）

```bash
sudo sunxi-fel -p spiflash-write 0 u-boot-sunxi-with-spl.bin
```

![uboot烧录成功后Uart0输出的日志](/Dom/imgs/2022_03_12/20220325153053.png)

u-boot运行效果

![ubootimg1](/Dom/imgs/2022_03_12/ubootimg1.jpg)

#  附录

```bash
sudo ln -s /opt/工具 /usr/local/bin/工具  # 将opt目录下指定的文件 软连接到 /usr/local/bin/文件 中（创建快捷方式）
sudo mv /工具 /opt/ # 将的文件移动到 opt 下
tar -xJf 工具.tar.xz  # 解压指定压缩包（不显示进度，静默解压）
tar -xvf 工具.tar.xz  # 解压指定压缩包（显示详细解压进度）
sudo apt-get purge 工具 # 卸载指定工具
sudo apt-get purge --auto-remove 工具  # 卸载指定工具 及其依赖
rm -rf 文件或者文件夹  # 删除指定的文件或者文件夹
mkdir 文件夹 # 创建文件夹
sudo cp ./文件 /某某文件夹  # 将指定的文件拷贝到指定的文件目录

ls /usr/bin/python # 查看运行目录下是否有安装指定的应用
ls /usr/local/bin/python # 查看本地运行目录下是否有安装的指定应用
ls /usr/bin/pyth* # 模糊查询

dmesg | grep tty # 查看串口设备

sudo tzselect # 修改系统时区
sudo hwclock --systohc # 时区同步到硬件
```

