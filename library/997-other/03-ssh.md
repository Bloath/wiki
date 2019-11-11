# SSH

[TOC]

## 一、原理
SSH之所以能够保证安全，原因在于它采用了公钥加密。
整个过程是这样的：

1. 远程主机收到用户的登录请求，把自己的公钥发给用户。
2. 用户使用这个公钥，将登录密码加密后，发送回来。
3. 远程主机用自己的私钥，解密登录密码，如果密码正确，就同意用户登录。

>关于公钥和私钥
>
>可以看做AB，A加密的，用B可以解密，B加密的，A可以解密
>
>主机要把公钥发给用户，用户用公钥将登陆密码加密后，主机通过私钥解密密码，然后再验证

## 二、登陆

### 2.1 首次登陆
在首次请求登陆主机时`ssh user@host`，主机会下发自己的公钥，并提示

```
The authenticity of host 'host (12.18.429.21)' can't be established.
RSA key fingerprint is 98:2e:d7:e0:de:9f:ac:67:28:c2:42:2d:37:16:58:4d.
Are you sure you want to continue connecting (yes/no)?
```
通过确认后，将远程主机的公钥保存在**home/.ssh/known_hosts**中，下次请求登录时通过比对发现已存该主机的公钥，就不会弹出警告直接进入密码登陆环节
>除了每个用户自身保存的已知主机列表外，系统本身也自带know_hosts，保存在**/etc/ssh**文件夹下

### 2.2 公钥登陆
每次登陆都需要输入密码比较繁琐，ssh提供了公钥登陆方式，原理如下

1. 用户将公钥发送到主机端保存
2. 用户登陆时，主机生成**随机数字**，发送给用户
3. 用户用私钥进行加密发送给主机
4. 主机通过之前保存的公钥解密，进行验证


首先，用户需要生成自己的公钥和私钥

```bash
ssh-keygen # 生成时会提示添加口令，如果不需要可以为空
```
在home/.ssh/中生成了**id_rsa**和**id_rsa.pub**，通过名字就可以看出id_ras.pub为公钥，下一步就是将公钥发送至主机

```
ssh-copy-id user@host
```
>os x没有该命令，通过该命令进行安装

`curl -L https://raw.githubusercontent.com/beautifulcode/ssh-copy-id-for-OSX/master/install.sh | sh `

>主机接受到用户公钥后，将其保存在$HOME/.ssh/authorized_keys中

这样就完成了用户公钥的发送，可以通过`ssh user@host`的方式直接登陆

## 三、文件的上传与下载
|操作|命令|示例|
|:---:|:---:|:---:|
|上传文件|`scp [本地文件路径] [user@host]:[远程目录]`|`scp /etc/nginx/nginx.conf root@11.11.11.11:/etc/nginx` |
|下载文件|`scp [user@host]:[远程文件路径] 本地目录`|`scp root@11.11.11.11:/etc/nginx/nginx.conf ~/Document`|
>文件夹上传时，加入 **-r** 参数即可，操作时会**直接替换**相同文件