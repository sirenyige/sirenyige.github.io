---
title: CLion+STM32CubeMX+WSL进行STM32开发
date: 2020-03-10 22:06:01
categories:
- STM32
tags:
- STM32
---

使用CLion和STM32CubeMX进行配置编辑

WSL进行编译与调试，无需安装MinGW

<!-- more -->

# Windows软件安装

## 1、[CLion](https://www.jetbrains.com/clion/download/#section=windows)

## 2、[STM32CubeMX](https://www.st.com/zh/development-tools/stm32cubemx.html)

## 3、[openOCD](http://openocd.org/)

## 4、WSL-[Ubuntu](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10)

# 编译工具安装

## 1、[arm-none-eabi-gcc](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads)安装

下载x86_64-linux版本压缩包，解压在WSL自定义目录下（如/usr/lib/arm-gcc/）将解压目录重命名一下，方便后续调用

![arm安装路径](/CLion-STM32CubeMX-WSL进行STM32开发/arm安装路径.png)

## 2、配置WSL以方便CLion进行ssh连接

```shell
wget https://raw.githubusercontent.com/JetBrains/clion-wsl/master/ubuntu_setup_env.sh && bash ubuntu_setup_env.sh
```

执行成功后会显示开启ssh的端口号

## 3、配置CLion编译链

![编译链设置](/CLion-STM32CubeMX-WSL进行STM32开发/编译链设置.png)

连接设置

![ssh配置](/CLion-STM32CubeMX-WSL进行STM32开发/ssh配置.png)

配置成功的话会自动发现WSL中已有编译链，其中Debugger需要设置成 /usr/lib/arm-gcc/gcc-arm-none-eabi/bin中的arm-none-eabi-gdb-py（C和C++编译器可以不管由后续CmakeList中指定）

> 如果多次执行ubuntu_setup_env.sh脚本，ssh工具和CLion都无法进行ssh连接，可以尝试删除C:\Users\xxx\\.ssh中的known_hosts文件后重新连接

## 4、开发工具配置

![开发工具](/CLion-STM32CubeMX-WSL进行STM32开发/开发工具.png)

# STM32CubeMX工程生成

STM32CubeMX正常配置后，生成工程文件时Toolchain/IDE选择SW4STM32，即可生成工程文件夹

![工程生成](/CLion-STM32CubeMX-WSL进行STM32开发/工程生成.png)

# CLion打开工程

##  1、大小不敏感配置

![大小写参数](/CLion-STM32CubeMX-WSL进行STM32开发/大小写参数.png)

添加 idea.case.sensitive.fs=true 配置项

## 2、CLion打开工程文件夹

![工程打开](/CLion-STM32CubeMX-WSL进行STM32开发/工程打开.png)

## 3、选择兼容开发板

进入工程后会出现对话框，选择芯片相同的兼容开发板，开发板配置可以在配置的openOCD目录下（C:\OpenOCD\share\openocd\scripts\board）

如果没有显示或者选错，在下文可以重新设置

![开发板选择](/CLion-STM32CubeMX-WSL进行STM32开发/开发板选择.png)

具体配置可以查询openOCD的配置

> 我是使用STM32F407VGT6芯片、DAP的SWD方式进行下载调试
>
> 选择了相同芯片的stm32f4discovery.cfg
>
> 修改接口类型、传输模式等参数
>
> ```
> # This is an STM32F4 discovery board with a single STM32F407VGT6 chip.
> # http://www.st.com/internet/evalboard/product/252419.jsp
> # 设置接口类型为cmsis-dap
> source [find interface/cmsis-dap.cfg]
> # 设置模式为SWD
> transport select swd
> # 设置芯片存储大小
> set WORKAREASIZE 0x10000000
> # 设置芯片类型
> source [find target/stm32f4x.cfg]
> # 注释掉复位方式-DAP-SWD需要
> #reset_config srst_only
> ```

## 4、修改CMakelist

此时工程会告警，找不到指定的编译器arm-none-eabi-gcc，需要修改工程的CMakeLists.txt

![cmakelist告警](/CLion-STM32CubeMX-WSL进行STM32开发/cmakelist告警.png)

将CMakeLists.txt头部指定交叉编译器的变量，加上绝对路径，改完后Tools--Cmake--Reload Cmake Project，即可解决告警

![编译变量修改](/CLion-STM32CubeMX-WSL进行STM32开发/编译变量修改.png)

## 5、修改编译下载任务

进去左上角任务设置，在Target、Executable下拉菜单中选中工程同名的elf文件

此处Board config file 点击 Assist 就可以重新选择兼容开发板信息

![编译下载1](/CLion-STM32CubeMX-WSL进行STM32开发/编译下载1.png)

![编译下载2](/CLion-STM32CubeMX-WSL进行STM32开发/编译下载2.png)

修改成功后，回到主页稍等几秒，任务设置图标上的红叉会消失，并且run和debug的按钮可点击

![任务配置成功](/CLion-STM32CubeMX-WSL进行STM32开发/任务配置成功.png)

此时连接开发板，点击run或者debug即可进行烧写和debug

>  Run信息始终为红色，不一定是烧写失败

