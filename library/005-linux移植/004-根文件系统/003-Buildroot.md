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





## 二、细节设置

### 2.1 控制行显示主机以及当前路径

```
sudo vim /etc/profile

删除
if [ "$PS1" ]; then
.....
fi

添加如下
PS1='[\u@\h] \w $ '
export PS1
```





### 2.2 openssh添加以及root空密码登陆

**在menuconfig中添加**

```
Target packages  --->
	Networking applications  --->
		[*] openssh
```

**修改**`/etc/ssh/sshd_config`

```
原 #PermitRootLogin prohibit-password 
PermitRootLogin yes

原 #PermitEmptyPasswords no
PermitEmptyPasswords yes
```

启动sshd

```
/usr/sbin/sshd
```

