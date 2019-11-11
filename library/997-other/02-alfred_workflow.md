# alfred workflow 开发
>alfred集成了workflow，workflow在提供打开浏览器、文件搜索、APP打开等功能的同时，也提供了脚本的运行借口，支持bash、python、php等语言，下面就通过我写的第一个例子，通过s建将后面的内容作为搜索文本进行淘宝、咸鱼、京东等网站的搜索

## 一、输入与输出
<div align="center"><img src="http://img.bloath.com/search_result.png" alt="" width="400px"></div>

>workflow与alfred之间的交互是通过**输入**、**输出**、**触发**，输入是通过参数的形式`{query}`，输出则是通过xml形式，格式如下

```xml
<items>
    <item autocomplete = "autocompletex" uid = "123321" arg = "argsx" >
    <title >title</title>
    <subtitle >subtitle</subtitle>
    <icon >icon</icon>
    </item>
</items>
```


## 二、新建一个workflow

>在alfred的偏好设置中，有个workflow标，左边是你的workflow列表，右边就是对应workflow的连线图。下面的加号就是新建一个自定义的workflow。

<div align="center"><img src="http://img.bloath.com/workflow.png" alt="" width="400px"></div>

在新建workflow时输入一些必要信息，具体就不列举了，一看就知道是什么意思

<br>

#### 2.1 设定输入、动作
在右侧空白处右键，开始选择输入和动作，我们的目的是通过`s`键触发，通过输入搜索项，最后通过浏览器打开。则在`inputs`中选择`keyword`，在`action`中选择`Open_URL`

<div align="center">
	<img src="http://img.bloath.com/workflow-input.png" alt="" width="300px">
	<img src="http://img.bloath.com/workflow-action.png" alt="" width="300px">
</div>

<br>

#### 2.2 选定热键以及脚本
在选定`input->keyword`之后，进入下面的界面，设置参考如下
<div align="center"><img src="http://img.bloath.com/workflow-keyword.png" alt="" width="500px"></div>

#### 2.3 设置OpenURL
我们最终的目的是打开URL，我们在之前经过处理，输出的URL放到这一步进行打开，设置如下，`{query}`其实就是上一步的输出结果

<div align="center"><img src="http://img.bloath.com/workflow-openurl.png" alt="" width="500px"></div>

#### 2.4 连线

将2.2和2.3中的模块通过鼠标直接连接即可完成上下连接操作

## 三、python库
workflow的输入输出有第三方库支持
[Alfred workflow](https://github.com/deanishe/alfred-workflow/)
>简单示例，输出一个结果，自定义标题与副标题

```python
#!/usr/bin/python
# encoding: utf-8

import sys
from workflow import Workflow

def main(wf):
    args = wf.args  # 在workflow对象初始化时，self.args = sys.argv 已经获取了运行脚本时由{query}传进来的字符串数组参数
    wf.add_item(u'Item title', u'Item subtitle') # 添加item，添加主标题和副标题，如果多个结果则添加多次
    wf.send_feedback()  # 将结果打包成xml发送给alfred显示


if __name__ == '__main__':
    wf = Workflow()		# 创建workflow对象
    sys.exit(wf.run(main))  # 与alfred交互
```


#### 搜索结果的再执行
>在命令执行后，获得多个result，想要继续根据选择的result进行下一步动作，首先先做好连接图，将input和action直接连接即可（可以设置组合按键），其次，要在input脚本中的result xml中设置**valid**和**arg**参数

```python
def add_item(self, title, subtitle='', modifier_subtitles=None, arg=None,
                 autocomplete=None, valid=False, uid=None, icon=None,
                 icontype=None, type=None, largetext=None, copytext=None,
                 quicklookurl=None):

# valid参数，将此参数设置为True，则可以被响应
#:param valid: Whether or not item can be actioned
#:type valid: ``Boolean``

# arg参数 将作为参数用于下一次的传递
#:param arg: Argument passed by Alfred as ``{query}`` when item is actioned
#:type arg: ``unicode``
```
>如果需要对搜索结果进行响应，必须在添加result时设置以上两个参数

## 四、例程


```python
#-*- coding: utf-8 -*-
# usr/bin/python

import sys
from workflow import Workflow3
import urllib2
import yaml

def main(wf):

    args = wf.args

    # 读取配置文件，获取参数
    with open("url.yaml", "r") as f:
        urls = yaml.load(f)

    # 将参数合并，防止带空格识别为多参数
    content = u"".join(u"%s " % i for i in args)

    # 根据读取的参数个数，添加多个result
    for item in urls:
        wf.add_item(item["name"], subtitle=u'在%s中搜索%s' % (item["name"], content),
                    valid=True,icon=item["icon"],
                    arg=item["url"].format(content=urllib2.quote(content.encode("utf8"))))

    wf.send_feedback()


if __name__ == '__main__':
    wf = Workflow3()
    sys.exit(wf.run(main))

```



