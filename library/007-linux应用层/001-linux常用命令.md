

<br>
***
### 防火墙与网络服务
>在centos7中，原有的ifconfig改用nmcli，iptables基本也不用，改用firewall-cmd，打开关闭防火墙用 systemctl xxx firewalld

|命令|参数|用法|备注|
|:---:|:---:|:---:|:---:|
|nmcli|con show|显示所有的网卡信息||
||con show 网卡名|显示当前网卡所有属性||
||con modify 网卡名 属性名 属性值|网卡所有的属性都可以通过该命令修改|ipv4.method <br>auto con.autoconnect yes我比较常用|
|firewall-cmd|--list-all |查看所有可用端口||
||--reload|重载||
||--permanent|后面的操作为永久||
||--add-port=80/tcp|将80端口的tcp打开|删除为remove|
||--query-service [ftp,ssh,http]|一些服务的开关||
||--list-ports|列出所有的端口||
||--list-services|列出所有的服务|

<br>
***
### 用户与权限
>linux的权限、分组以及密码都是通过文件存储在本地，分别为
>
* `/etc/password`: 存储用户以及密码 `root:x:0:0:root:/root:/bin/bash`
	* 用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录Shell
	* 用户标识：1-99为系统，100以后为用户
	* 主目录：在该用户登录后的~文件夹
	* 登陆shell：如果设置为/etc/nologin，并非是shell接口，则登陆成功会显示/etc/nologin内容。如果是/usr/bin/bash，则登陆成功并显示shell
* `/etc/group`：存储用户分组信息 `users:x:1000`
* 
* `/etc/shadow`: 存储用户密码信息


|效果|命令|备注|
|:---:|:---:|:---:|
|添加用户|adduser 用户名<br>adduser 用户名 组名||
|删除用户|deluser -r 用户名<br>deluser 用户名 组名||
|添加密码|password 用户名||
|修改用户|usermod 用户名|-G 添加到某个群组<br>-s 修改用户shell<br>-L/-U 锁定/解除锁定|
<br>
***


### 压缩与解压缩 tar

|参数|效果|
|:---:|:---:|
|-z|使用gzip|
|-j|使用bzip2|
|-J|使用xz、lzip、lzma|
|-v|显示具体信息|
|-x|解压|
|-c|压缩|


### 多语句执行
1. `; ` 同时执行，
2. `&&` 前面的执行成功，才会执行后面的语句
3. `||` 前面的语句失败，则执行下一句，直到执行成功为止6