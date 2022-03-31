---
title: STM32F1-DAPLink源码编译
date: 2019-06-27 12:30:46
tags:
---

# DAPLink也带有虚拟串口  web-usb   U盘拖拽下载功能

    DAPLink电路  GitHub  ARM官方
    https://github.com/ARMmbed/mbed-HDK-Eagle-Projects
    DAPLink源码  GitHub  ARM官方 开发文档
    https://github.com/ARMmbed/DAPLink/blob/master/docs/DEVELOPERS-GUIDE.md

    淘宝买的Daplink
    http://www.eemaker.com/daplink-data.html  资料在这

    别人把DAPLink移植到STLink上
    https://bh3nvn.github.io/2019/05/DAPLink2STlink/

步骤如下：
<!-- more -->

#### 要 `python`   `git`   `mdk` 这几个软件


``` bash
git clone https://github.com/mbedmicro/DAPLink     // git 克隆官方项目下来
cd DAPLink                                         // 进入这个项目
pip install virtualenv                             // 安装 virtualenv
virtualenv venv                                    

venv\Scripts\activate.bat                          // 运行
pip install -r requirements.txt                    // 安装需要的依赖包


编译输出全部目标版文件
progen generate -t uvision

编译输出局部目标板文件
progen generate -p stm32f103xb_bl -t uvision                   //  bootloader文件  需要编译后烧录到目标板
progen generate -p stm32f103xb_stm32f103rb_if -t uvision	//  _if_crc.bin文件  需要把nRST引脚接地插入目标板试别成U盘再考进去，再上啦nRST即可  nRst不是单片机的自身复位

```

> 通过MDK打开目标板的工程文件，记住，换工程的时候不能关闭这个命令打开的MDK，只能在这个MDK中openProject
tools\launch_uvision.bat   
这一步可能会报错， 因为每个人的MDK安装路径不一样，  打开这个文件  修改安装路径即可 DAPLink\tools\launch_uvision.bat 文件

<img src="/Dom/imgs/2019_06_27/01.png" width="40%"/>


> 编译的过程中。可能也会报错  python3
DAPLink\tools\flash_algo.py   文件中的 StringIO   改为  io   还有 StringIO.StringIO(data)  改为  io.StringIO(data)
DAPLink\tools\post_build_script.py  文件中  0xffffffffL  改为  0xffffffff

编译完  _bl 文件  直接通过MDK烧录到目标板中   拔掉目标板，  将nRST下拉  插入 会试别为U盘
编译 _if 文件  将 _if_crc.bin  结尾的文件考到U盘中 拔掉U盘  nRST上拉  完成

``` bash
venv\Scripts\deactivate.bat   
```

完成