---
title: 荔枝派_buildroot文件系统编译
date: 2022-05-16 19:24:38
tags:
---

# 一、 获取buildroot文件系统源码

```bash
wget https://buildroot.org/downloads/buildroot-2019.08.tar.gz  # 下载源码
tar xvf buildroot-2019.08.tar.gz  # 解压
cd buildroot-2019.08
```

# 二、 配置buildroot

```bash
make menuconfig  # 配置buildroot文件系统
```

![menuconfig](/Dom/imgs/2022_05_16/menuconfig.png)

<!-- more -->

配置 `Target options -->` 设置目标芯片 配置如下

![menuconfigTarget](/Dom/imgs/2022_05_16/menuconfigTarget.png)

配置 `Toolchain -->` 设置编译工具链

首先需要查看编译工具链所在目录和编译工具链的版本

```bash
which arm-linux-gnueabihf-gcc  # 查看编译工具所在目录

vim /opt/gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf/arm-linux-gnueabihf/libc/usr/include/linux/version.h  # 查看编译工具版本号
# 263680的二进制为0x40600，则对应的内核版本号为4.6.0。
```

配置目标如下

![menuconfigToolchain](/Dom/imgs/2022_05_16/menuconfigToolchain.png)

# 三、 编译buildroot
```bash
time make # 编译buildroot 此过程需要半个小时左右
```
编译完成后，生成的文件系统在 `output/images/rootfs.tar`
