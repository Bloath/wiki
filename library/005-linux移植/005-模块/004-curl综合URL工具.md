# 综合传输工具 CURL

> curl可以说是网络通讯中比较全面的工具了，在以HTTP建立的通讯当中，使用curl配合shell即可完成简单的信息获取与文件处理



## 一、编译与安装

[下载地址](https://curl.haxx.se/download.html)

```shell
# 解压步骤忽略

# 1. 编译配置，编译生成的多个文件会放置在$CURL_PATH中，并且需要设置CC与CXX 
./configure --prefix=$CURL_PATH --host=arm-linux CC=arm-linux-gcc CXX=arm-linux-g++

# 2. 编译并导出文件，完成后会在$CURL_PATH中生成4个文件夹bin lib include share
make && make install

# 3. 移植文件，将 $CURL_PATH/bin/curl $CURL_PATH/bin/curl-config 移动到目标板根文件系统的/bin
cp $CURL_PATH/bin/curl $ROOTFS/bin
cp $CURL_PATH/bin/curl-config $ROOTFS/bin

# 4. 移植链接库，将 $CURL_PATH/lib/libcurl.so.4 移动到目标板根文件系统的/lib
cp $CURL_PATH/lib/libcurl.so.4 $ROOTFS/lib
```



