# 拙诚系列产品wifi配网方案

[一、应用场景]()<br>
[二、配网方案](#2)<br>
[三、APP操作](#3)


## 一、应用场景
>带有wifi的系列产品，可以连接用户的无线路由器，既而与服务器连接，进行数据上传与下载。

在用户首次拿到设备时，需要通过APP经过一系列操作将设备连接到家中的路由器。其实质就是手机与设备通讯，将需要连接的wifi信息，包含ssid和密码，发送给设备，设备根据wifi信息进行连接的过程

## 二、<a name="2"></a>配网方案
### 2.1 原理
1. 设备产生wifi热点(后面简称<a name="wifi-device"></a><font color="red">wifi-Device</font>)，开启DHCP，设备IP为192.168.4.1
2. 设备开启UDP，端口为9000
2. 手机连接wifi-Device，开启UDP，端口随意
3. 手机将家中无线路由器的wif(后面简称<a name="wifi-home"></a><font color="red">wifi-Home</font>)的ssid和密码通过UDP发送给设备
4. 设备尝试连接，连接成功并得到服务器响应/连接失败，将响应结果发送给手机
5. 手机得到成功的回复后，切换回之前连接的wifi
6. 结束配网，进入主操作界面


### 2.2 流程图
![流程](http://on-img.com/chart_image/59e6a69ce4b0b640aeebb95d.png)

## 三、<a name="3"></a>APP操作
### 3.1 连接[wifi-Device](#wifi-device)
1. 通过扫描二维码，获取该设备的wifi以及密码信息，并进行连接
2. 开启UDP，尝试向192.168.4.1,9000发送数据`fct`（first connection test）
3. 收到设备发来的`{"fct":true}`表示连接成功，否则判断为超时

### 3.2 发送[wifi-home](#wifi-home)信息至设备
1. APP中模拟wifi连接界面，选择要连接的wifi
2. 将选中的wifi的ssid与password，通过udp发送，格式为`wifi:ssid,password`，例如`wifi:wifi-home,12345678`
3. APP接收到设备的回复
    * `{"wifi":true}`表示成功
    * `{"wifi":null}`表示连接失败
    *  `{"wifi":false}`表示连接成功，但是连接服务器失败。

### 3.3 切换wifi
断开wifi-Device的连接，连接wifi-Home，进入主界面进行操作