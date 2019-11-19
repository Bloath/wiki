# ubuntu基本设置

[TOC]

##  环境变量

### 1.sudo 重置环境变量的问题

> 对于ubuntu来讲，登陆的用户并非root，所以说在获取权限时，需要在命令前加sudo，但是sudo会将环境变量进行重置，明明通过修改`~/.bashrc`文件中添加了环境变量，但是sudo时扔然找不到路径。

解决方法：加入`sudo env PATH=$PATH`，但是每次加入都很麻烦，所以说直接使用别名即可，在`.bashrc中加入alias sudo="sudo PATH=$PATH"`即可



### 2.修改root密码

- `sudo passwd` 修改root密码
- `su root`切换到root用户



### 3. 设置关闭盖子不休眠

**电源部分**

```shell
sudo vim /etc/UPower/UPower.conf

修改： IgnoreLid=true

service upower restart
```

**登陆部分**

```shell
sudo vim /etc/systemd/logind.conf

#HandleLidSwitch=suspend
改成下面，记得去掉“#”号：
HandleLidSwitch=ignore

service systemd-logind restart
```





## ftp文件传输

**相关命令**

```shell
sudo apt install vsftpd -y
sudo /etc/init.d/vsftpd restart
```

**修改配置文件**

```shell
sudo vim /etc/vsftpd.conf

解开如下注释
local_enable=YES
write_enable=YES
```

使用[FileZilla]()新建连接，并填入用户名即可



## NFS 挂载

**相关命令**

```shell
sudo apt install nfs-kernel-server -y
sudo /etc/init.d/nfs-kernel-server restart
sudo ufw disable 	# 关闭防火墙
```

**创建文件夹并修改配置文件**

```
mkdir $TARGET_DIR

sudo vim /etc/exports
添加
$TARGET_DIR *(rw,sync,no_root_squash,no_subtree_check)

ro 该主机对该共享目录有只读权限
rw 该主机对该共享目录有读写权限
root_squash 客户机用root用户访问该共享文件夹时，将root用户映射成匿名用户
no_root_squash 客户机用root访问该共享文件夹时，不映射root用户
sync 资料同步写入到内存与硬盘中
async 资料会先暂存于内存中，而非直接写入硬盘
insecure 允许从这台机器过来的非授权访问

all_squash 客户机上的任何用户访问该共享目录时都映射成匿名用户
anonuid 将客户机上的用户映射成指定的本地用户ID的用户
anongid 将客户机上的用户映射成属于指定的本地用户组ID
```

**测试**

```
exportfs -r 测试配置文件是否合法
showmount -e 查看可挂载的文件夹
```



## SSH 远程登陆

```
sudo apt install openssh-server -y
```

 **使用root登陆ssh**

- sudo vim /etc/ssh/sshd_config，将`PermitRootLogin`该为yes
- `sudo /etc/init.d/ssh restart`重启



## TFTP 文件传输

**安装**

```shell
sudo apt install tftp-hpa tftpd-hpa -y
```

**创建tftp目录并添加配置文件**

```shell
sudo mkdir $TFTP_DIR
sudo chmod 777 $TFTP_DIR
sudo mkdir /etc/xinetd.d/ && sudo vim /etc/xinetd.d/tftp

文件中添加如下内容
server tftp
{
	socket_type = dgram
	protocol = udp
	wait = yes
	user = root
	server = /usr/sbin/in.tftpd
	server_args = -s $TFTP_DIR
	disable = no 
	per_source = 11 
	cps = 100 2
	flags = IPv4
}

sudo service tftpd-hpa start	//启动tftp
```

**修改默认配置文件**

```
sudo vim /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="$TFTP_DIR"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="-l -c -s"
```

**重启服务**

```shell
sudo service tftpd-hpa restart
```



## 网络

### 固定ip

```
sudo vim /etc/network/interfaces

将iface $TARGET_NET inet dhcp
修改为
iface $TARGET_NET inet dhcp

并添加如下
address xxx.xxx.xxx.xxx
netmask xxx.xxx.xxx.xxx
gateway xxx.xxx.xxx.xxx

sudo /etc/init.d/networking restart
ifdown $TARGET_NET
或者直接重启
sudo reboot
```

### DNS

```
sudo vim /etc/resolv.conf

添加
nameserver xxx.xxx.xxx.xxx
nameserver xxx.xxx.xxx.xxx
```

