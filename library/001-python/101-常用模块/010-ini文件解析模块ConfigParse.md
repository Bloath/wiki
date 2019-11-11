# ini文件解析模块ConfigParse

>在py中的json，yaml，pickle都是需要先读取文件在，再进行dump，load操作，而ini文件则不同，有着单独的文件以及多种方法完成配置文件的读写，ini文件内所有值皆为字符串，数字类取出后需要转换

配置文件conf.ini内容如下

```ini
[server_info]
host=114.114.114.114
port=8000

[root]
password=12345678

[app]
app_id=90-dakljdlkasjd-dal
app_secret=dalkhjdhjgbda-00

```

### 一、创建对象，读取配置文件

```python
import configparse

config = configparse.ConfigParse()  # 创建configparse对象
config.read("conf.ini")		# 读取配置文件
```

### 二、读取节点

|方法名|用法|作用|
|:---:|:---:|:---:|
|sections|`config.sections()`|返回配置文件中所有节点`["server_info", "root", "app"]`|
|options|`config.options("server_info")`|返回指定节点中所有的选项`["host", "port"]`|
|items|`config.items("server_info")`|返回指定节点的选项和值，元组列表的形式<br>`[("host","114.114.114.114"), ("port", ”8000“)]`|
|get|`config.options("root", "password")`|获取指定节点指定选项的值`12345678`|