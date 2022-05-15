---
title: 荔枝派-linux内核编译
date: 2022-05-15 19:18:21
tags:
---

# 一、 获取Linux内核源码

```bash
git clone https://github.com/Lichee-Pi/linux.git
```

`源码切换到最新分支 zero-4.13.y`

！！！文件大小大概有 2.54G

# 二、 内核选项配置

执行 `make ARCH=arm menuconfig` 打开内核配置菜单

![menuconfigHome](/Dom/imgs/2022_04_07/menuconfigHome.png)

进入 `Device Drivers > Memory Technology Device (MTD) support` 选项

![menuconfigDeviceMTD](/Dom/imgs/2022_04_07/menuconfigDeviceMTD.png)

确保选择上 `TMD` 的 `<*> Command line partition table parsing`  支持，该项目用来解析uboot传递过来的flash分区信息；以及SPI-NOR 设备的支持。

![menuconfigDeviceMTD-sub](/Dom/imgs/2022_04_07/menuconfigDeviceMTD-sub.png)

添加对jffs2文件系统的支持，路径在 `File systems > Miscellaneous filesystems ‣ Journalling Flash File System v2 (JFFS2) support`

![menuconfigJffs2File](/Dom/imgs/2022_04_07/menuconfigJffs2File.png)

# 三、修改设备树文件，使其支持 SPI Flash

修改设备树文件

```bash
vim arch/arm/boot/dts/sun8i-v3s-licheepi-zero.dts
```

添加 SPI 节点配置

```dtd
&spi0 {
        status ="okay";

        xt25f128b:xt25f128b@0 {
                compatible = "jedec,spi-nor";
                reg = <0x0>;
                spi-max-frequency = <50000000>;
                #address-cells = <1>;
                #size-cells = <1>;
        };
};
```

`xt25f128b` 此型号必须是在 `drivers/mtd/spi-nor/spi-nor.c` 中支持的spi设备

修改 `drivers/mtd/spi-nor/spi-nor.c` 文件

```bash
vim drivers/mtd/spi-nor/spi-nor.c
```
添加以下代码
```c
{ "xt25f128b", INFO(0x0b4018, 0, 64 * 1024, 256, SPI_NOR_DUAL_READ | SPI_NOR_QUAD_READ) },
```
![spiFlash](/Dom/imgs/2022_04_07/spiFlash.png)


# 四、 内核编译

！！！ 此命令将会还原以上的 `menuconfig` 的一系列配置  如果还原需要重新 `make ARCH=arm menuconfig` 来配置 `make ARCH=arm licheepi_zero_defconfig `

```bash
time make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-
```

或者使用 -jn 来加速编译 n = 参与编译的内核个数

```bash
time make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4
```

编译需要话较长的时间！！！

镜像编译完成后  在 arch/arm/boot/ 文件夹下生成 zImage 文件

```bash
gongyu@gongyu-VirtualBox:~/Lichee_Pi/linux$ ls arch/arm/boot/
bootp  compressed  dts  Image  install.sh  Makefile  zImage
```

## 编译错误

### 错误信息1

```bash
error New address family defined, please update secclass_map.
```

![zimageCompileError1](/Dom/imgs/2022_04_07/zimageCompileError1.png)

（1）、修改 `vim scripts/selinux/mdp/mdp.c` 文件

注释掉 `#include <sys/socket.h>`

![zimageCompileError1pass](/Dom/imgs/2022_04_07/zimageCompileError1pass.png)

（2）、修改 `im security/selinux/include/classmap.h` 文件

添加 `#include <linux/socket.h>`

![zimageCompileError1pass2](/Dom/imgs/2022_04_07/zimageCompileError1pass2.png)

### 错误信息2

```bash
fatal error: openssl/opensslv.h: No such file or directory
```

![zimageCompileError2](/Dom/imgs/2022_04_07/zimageCompileError2.png)

需要安装 OpenSSL 开发包

```bash
sudo apt-get install libssl-dev
```

### 错误信息3

```bash
没有规则可制作目标“debian/canonical-revoked-certs.pem”，由“certs/x509_revocation_list” 需求。 停止。
```

![zimageCompileError3](/Dom/imgs/2022_04_07/zimageCompileError3.png)

编辑 `.config` 配置文件 `vim .config`

清空这两项配置

```bash
CONFIG_SYSTEM_TRUSTED_KEYS="debian/canonical-certs.pem"
CONFIG_SYSTEM_REVOCATION_KEYS="debian/canonical-revoked-certs.pem"

to

CONFIG_SYSTEM_TRUSTED_KEYS=""
CONFIG_SYSTEM_REVOCATION_KEYS=""
```

生成 dtbs

```bash
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- dtbs

# arch/arm/boot/dts/sun8i-v3s-licheepi-zero.dtb
# arch/arm/boot/dts/sun8i-v3s-licheepi-zero-dock.dtb
```

dtb文件在 `arch/arm/boot/dts/`

zImage在`arch/arm/boot/`

生成内核模块 驱动模块（@TODO）

```bash
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j16 INSTALL_MOD_PATH=out modules
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j16 INSTALL_MOD_PATH=out modules_install
```

驱动模块在 `out/` 目录下

