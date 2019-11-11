# TCP/UDP与socketserver模块

* [TCP/UDP](#1)
* [socketserver模块](#2)


## 一、<a name="1"></a>TCP与UDP
>TCP与UDP是基础中的基础，其区别以及连接、通讯流程需要熟知

### 1.1 TCP===>SOCK_STREAM
##### TCP服务器
```python
import socket


client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.bind(("", 6000))
client.listen()		# 填入参数则设定最大连接说亮

while True:
    con, address = client.accept()
    while True:
        data = con.recv(1024)		
        con.send(data)
```

##### TCP客户端
```python
import socket

client = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
client.connect(("", 6000))

client.send(b"aaaaaa")
```

>在使用TCP通讯时要注意以下几个要点

1. 在使用完socket时，一定要将socket关闭
2. 关闭后的socket无法再重新使用，必须重新生成
3. 在任意一方连接断开后，接收方的响应不同，需要针对不同系统进行处理
	* windows系统：接收会报错
	* linux系统：会接收`b''`，空的字节


### 1.2 UDP===>SOCK_DGRAM
```python
import socket

s = socket.socket(socket.AF_INET,socket.SOCK_DGRAM)
s.bind(("",9000))  
# address是一个由ip地址和端口组成的元组，例如('192.168.4.1',9000)，绑定本地时不需要填写

# 发送与接收(py3)
s.sendto('123'.encode(utf8), ("192.168.4.1", 9000)) # UDP的发送是可以发送空的
data,address = s.recvfrom(1024) # 取多取少就这样了

```

## 二、<a name="2"></a>阻塞的TCP/UDP通讯
### 
>在TCP/UDP连接中，接收连接或者接收数据方法调用时，会让当前线程阻塞，其本质是等待内核区的数据进入，当进入后，取出数据，退出阻塞。在多连接的情况下，阻塞就是一个比较棘手的问题。

### 2.1 设置为非阻塞
>比较笨的方法就是将阻塞方法改为非阻塞，通过不断轮询处理数据。实际操作起来比较繁琐

```python
client.setblocking(False)
>>BlockingIOError: [Errno 35] Resource temporarily unavailable

client.settimeout(2)	# 设置超时时间，2s内获取不到则弹错
>>socket.timeout: timed out
```
通过上述方法将通讯流程设置为非阻塞，当执行到接收连接或者接收数据等原阻塞方法时，如果内核区并没有连接或者数据，则会弹出相应错误。

### 2.2 IO多路复用 select



## 三、<a name="3"></a>socketserver模块
>socketserver模块就是通过select与线程，完成的多路通讯

3.1 Demo

```python
import socketserver

# 1. 继承socketserver.BaseRequestHandler类，重写handle，finishi方法
class server(socketserver.BaseRequestHandler):
    def handle(self):
        con = self.request
        while True:
            data = con.recv(1024)
            con.send(data)

    def finish(self):
        self.request.close()

socketserver.TCPServer.allow_reuse_address = True   # 端口复用开启

# 2.sockectserver对象初始化， 根据不同的应用使用不用的类，把之前写的类带入
obj = socketserver.ThreadingTCPServer(("", 6001), server)   

# 3.开启服务
obj.serve_forever()    

```

3.2 源码解析





