# VSCODE远程编辑

> 在linux中使用vim修改小型文件还是比较得心应手，如果修改稍微长一点的文件则有些蹩脚，所以对于一个工程的修改还是用宇宙无敌的ide-vscode比较舒服一些，vscode中有一个插件remote-ssh就解决了这个问题

[TOC]

## 一、remote-ssh的安装与设置

### 1.1 安装

在vscode的插件中找到remote-ssh插件并安装即可，注意，现阶段只有vscode insider中才有

### 1.2 设置

* 按下F1，在搜索框中输入` remote`，得到remote-ssh插件的一系列操作

* 选择`connect to host`，会出现`add new ssh host`与`configure ssh hosts`两个选项

* 选择`add new ssh host`

* 键入ssh命令，例如`ssh root@192.168.0.125`

* 选择配置文件位置，我选择用户下面的配置文件

  > 到这里remote-ssh就配置好了

### 1.3 打开

* 按下F1，在搜索矿中输入`remote`，选择`remote-ssh : connect to host`

* 此时就会出现你之前键入的ssh命令的主机名称

* 选择主机并且回车即可连接

  **注意**：在此之前要进行免密设置`ssh-copy-id USERNAME@HOSTNAME`才能够使用remote-ssh



## 二、ubuntu的相关设置

在remote-ssh通过非root用户登陆了ubuntu系统之后，去修改文件时总是发现权限不足

即使通过

* gpasswd -a USERNAME root
* 在`/etc/sudoers`中添加`USERNAME 	ALL=(ALL:ALL) ALL`

都 **无法** 在非root下修改文件

所以只能直接使用root进行登陆

### 1.1 修改root密码

在安装完成之后，root密码是随机生成的，需要修改root密码

```
sudo passwd #修改root密码
Enter new UNIX password: 
Retype new UNIX password:
su - #切换root用户
```

### 1.2 在sshd中允许root登陆

```
vim /etc/ssh/sshd_config

将
PermitRootLogin prohibit-password
修改为
PermitRootLogin yes

重启sshd
service ssh restart
```



