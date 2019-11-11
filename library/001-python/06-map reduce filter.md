# map reduce filter

* [简介](#1)
* [map，filter，reduce用法](#2)

>个人认为这三个方法/类配合匿名函数lambda大大增加了代码可读性，从C过来的人表示非常简洁，以前通过几行for循环完成的事情，可以通过上述三个方法一行完成。

## 一、<a name="1"></a>简介

* map：`class map : __init__(self, func, *iterables)`
* filter：`class filter : __init__(self, function_or_None, iterable)`
* reduce：`def reduce(function, sequence, initial=None)`

|名称|返回|作用|
|:---:|:---:|:---:|
|map|map对象|将可迭代对象内的所有方法执行func方法，并返回一个map对象（迭代器）|
|filter|filter对象|将可迭代对象进行遍历，将满足function的item保留，并返回filter对象（迭代器）|
|reduce|最终的计算值|将可迭代对象中的item通过function计算，与初始值initial相叠加，返回一个值|

<br>

## 二、<a name="2"></a>map，filter，reduce用法

#### 2.1 map 

* 最终返回map对象，为迭代器，可以通过list()进行强制转换为列表
* map中可以添加多个可迭代对象，func的参数要与可迭代对象个数相同。多个可迭代对象同时遍历，当较短的可迭代对象STOP时，整个流程结束，返回map对象，看下面的lister2

```python

lister1 = [1,2,3,4,5,6,7]
lister2 = [1,1,1,1,1]

new_list1 = list(map(lambda x: x*2, lister1))  # >>[2, 4, 6, 8, 10, 12, 14]
new_list2 = list(map(lambda x, y: x*y, lister1,lister2)) # >>[1, 2, 3, 4, 5]

```

#### 2.2 filter

>filter与字面意思相同，就是过滤，func是返回bool的方法，通过遍历可迭代对象，保留执行func后返回True的item，返回filter对象（迭代器）

```python
lister = [2,31,4,1,53,24,2,4123,3,12,3123,32]

new_list = list(map(lambda x : (x%3) == 0, lister))

>>[24, 3, 12, 3123]

```

#### 2.3 reduce

> reduce与其他两个不同，function必须为两个参数

```python
from functools import reduce

lister = [1,2,3,4,5,6,7]

value = reduce(lambda x,y: x+y, lister)
>>28
value = reduce(lambda x,y: x+y, lister, 2) # 2为初始值
>>30
```

<br>