# Arm-linux 使用USB 4G模块上网

[TOC]

> 开发环境

* 主机系统：ubuntu 16.04 LTS
* 嵌入式：NUC980 
* 内核版本：4.4
* 工具链gcc版本：4.8
* 4G模块：EC20



## 一、简介

> 使用USB 4G模块的目的是，能够在系统中生成网卡，可以使用系统的socket完成通讯

### 拨号方式不同

USB 4G模块支持的网卡分为两种

* ppp拨号：生成ppp0网卡，通过USB虚拟串口完成，
* GobiNet：
* QMI：
* ECM：



## 二、PPP拨号上网的配置操作

> 以下步骤皆参考官方文档

### 2.1 内核中添加VID和PID列表

```
KERNEL_PATH/driver/usb/serial/options.c

static const struct usb_device_id option_ids[] = {
#if 1 //Added by Quectel
{ USB_DEVICE(0x05C6, 0x9090) }, /* Quectel UC15 */
{ USB_DEVICE(0x05C6, 0x9003) }, /* Quectel UC20 */
{ USB_DEVICE(0x2C7C, 0x0125) }, /* Quectel EC25/EC20 R2.0 */
{ USB_DEVICE(0x2C7C, 0x0121) }, /* Quectel EC21 */
{ USB_DEVICE(0x05C6, 0x9215) }, /* Quectel EC20 */
{ USB_DEVICE(0x2C7C, 0x0191) }, /* Quectel EG91 */
{ USB_DEVICE(0x2C7C, 0x0195) }, /* Quectel EG95 */
{ USB_DEVICE(0x2C7C, 0x0306) }, /* Quectel EG06/EP06/EM06 */
{ USB_DEVICE(0x2C7C, 0x0296) }, /* Quectel BG96 */
#endif
```



### 2.2 添加内核USB Serial以及PPP协议支持

```
[*] Device Drivers --
	[*] USB Support --
		[*] USB Serial Converter support --
			[*] USB driver for GSM and CDMA modems
			
[*] Device Drivers --
	[*] Network device support --
		[*] PPP (point-to-point protocol) support
		<*> PPP 下面带PPP的都勾上
		
```

以上步骤完成之后，在插入4G USB模块后，通过`lsusb`命令，即可看到USB列表中包含我们刚才在内核添加的移远的模块ID

### 2.3 下载ppp并编译

> 2.4.5版本经过测试，2.4.6以上，缺少utmp支持，无法使用

#### 2.3.1 下载并交叉编译ppp

```
sudo wget https://download.samba.org/pub/ppp/ppp-2.4.5.tar.gz
sudo tar -xzvf ppp-2.4.5.tar.gz
sudo cd ppp-2.4.5
sudo ./configure
sudo make CC=arm-linux-gcc
```

在上述步骤完成后，分别生成`chat`，`pppd`，`pppdump`，`pppstats`四个可执行文件，在同名文件夹中，将其复制到根文件目录的`/bin`下。



#### 2.3.2 移动链接库

将工具链中`/lib`中的以下三个链接库移动到根文件系统的`/lib`中

* `libcrypt.so.0`
* `libdl.so.0`
* `libutil.so.0`

移动之后就可以正常启动`pppd`了



### 2.4 启动拨号

在启动拨号之前，需要将拨号脚本文件安放在以下位置

#### 2.4.1 放置拨号脚本

```
etc
├── ppp
    ├── ip-up
    ├── peers
        ├── quectel-chat-connect
        ├── quectel-chat-disconnect
        ├── quectel-ppp
        └── quectel-ppp-kill
        
将quectel的四个脚本以及ip-up脚本按照该结构放置好
```

#### 2.4.2 修改拨号参数

> 拨号上网需要user、password与APN，这三个参数是需要与运营商联系索要

* 修改user与password， `quecterl-ppp`中，修改user与password
* 修改APN，`quectel-chat-connect`中

```
# Insert the APN provided by your network operator, default apn is 3gnet
OK AT+CGDCONT=1,"IP","3gnet",,0,0

将3gnet修改为正确的APN
```

#### 2.4.3 启动拨号

```
mkdir /var/lock       启动之前先新建该文件夹，否则会报错
pppd call quectel-ppp &


在启动完成后通过ifconfig查看网卡，则完成拨号
ppp0      Link encap:Point-to-Point Protocol
          inet addr:10.104.219.164  P-t-P:10.64.64.64  Mask:255.255.255.255
          UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1500  Metric:1
          RX packets:4 errors:0 dropped:0 overruns:0 frame:0
          TX packets:4 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:3
          RX bytes:52 (52.0 B)  TX bytes:58 (58.0 B)

```

#### 2.4.4 挂断拨号

```
/etc/ppp/peers/quectel-ppp-kill	
```



## 2.5 出现的问题以及解决方案

### 2.5.1 ppp出现segmentation fault

这个问题是文件系统不匹配，之前是通过github上的buildroot直接编译生成的文件系统，使用厂家提供得文件系统即可

### 2.5.2 connection terminated

这个问题一般是APN设置的有问题，例如我的卡，有的时候是CMNET，有的时候是CMIOT

### 2.5.3 有些网址ping不通

这个文件是因为DNS服务器不足引起的，需要添加多条DNS服务器地址

```
vi /etc/resolv.conf

添加DNS服务器低脂
nameserver 8.8.8.8
nameserver 114.114.114.114
```

### 2.5.4 老是断线

一般是因为信号不好引起的，可以查询信号强度，15以下就是比较弱，好的话一般在25左右

```shell
# 在拨号之前进行查询，拨号之后就查不了了
echo -e "AT+CSQ\r\n" >> /dev/ttyUSB2		# 发送查询指令
cat /dev/ttyUSB2	# 查看结果
```





