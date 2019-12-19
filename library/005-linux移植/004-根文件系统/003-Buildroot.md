# Buildroot 

> buildroot = busybox + 配置文件（以前需要手动添加的）+ 各种其他模块（以前需要手动移植的）

* [下载地址](https://busybox.net/downloads/)
* 开发板：正点原子imx6ull
* 内核版本：4.1.5
* 编译器版本：linaro 4.9

[TOC]

## 一、基础设置

```
sudo make menuconfig
```

**目标平台的设置**

```
Target options  --->
	 Target Architecture (ARM (little endian))  --->
	 Target Binary Format (ELF)  --->
	 Target Architecture Variant (cortex-A7)  --->
	 Target ABI (EABIhf)  --->
	 Floating point strategy (NEON/VFPv4)  --->
	 ARM instruction set (ARM)  --->
```

**工具链设置**

```
Toolchain  --->
	Toolchain type (External toolchain)  --->
	Toolchain (Custom toolchain)  --->
	Toolchain origin (Pre-installed toolchain)  --->
	() Toolchain path	// 填写地址，只需要倒根就可以，不需要到bin那一级
	($(ARCH)-linux-gnueabihf) Toolchain prefix
	
	External toolchain gcc version (4.9.x)  --->
	External toolchain kernel headers series (4.1.x)  --->
	External toolchain C library (glibc/eglibc)  --->
	
	[*] Toolchain has SSP support?
	[*] Toolchain has RPC support?
	[*] Toolchain has C++ support?
	
	[*] Enable MMU support
```

**编译**

```
sudo make 
```

在编译完成之后会生成`output/images/rootfs.tar`，这就是最终的根文件系统





## 二、单独修改package

> 如果想要单独修改某一个package的话，通过直接修改package再编译的方法是不行的。

buildroot对选中的package一般进行下载、解压缩、配置、编译等步骤

那么每一步都在`out/build/$PACKAGENAME_$VERSION/`中生成如下文件

```
.stamp_built
.stamp_configured
.stamp_dotconfig
.stamp_downloaded
.stamp_extracted
.stamp_kconfig_fixup_done
.stamp_patched
.stamp_target_installed
```

想要重新配置并重新编译进入文件系统

* 不修改`.stamp_configured`
* 删除`.stamp_built`

