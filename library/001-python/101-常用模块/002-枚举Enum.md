# 枚举Enum
* [初始化](#1) 
	* [普通初始化](#1.1)
	* [自动值初始化 auto](#1.2)
	* [唯一值初始化 @unique](#1.3)
* [枚举的使用 name以及value](#2)
* [其他特性](#3)
	* [可迭代对象](#3.1)
	* [可哈希对象](#3.2)

>在日常撸代码的时候，通常有多种状态需要切换或者展示，比如指示一个事情，有未完成，完成，过期等多种形式，如果用字符串存储，难免会出现拼写错误。最简单的方法就是讲字符串与数字列表建立**对应关系**，并且对应关系是可以互相切换的，最简单的就是通过枚举来实现

### 一、<a name="1"></a>初始化

#### 1.1 <a name="1.1"></a>普通初始化
>python提供了Enum类，在建立自己的枚举类时，需要继承。枚举成员建立类似类对象。

```python
from enum import Enum

class Priority(Enum):
    default = 0
    danger = 1
    warning = 2
    active = 3
```
#### 1.2 <a name="1.2"></a>自动值初始化 auto
>枚举成员的value值由auto方法生成，适用于value值不关心场合

```python
from enum import Enum,auto

class Priority(Enum):
    default = auto()
    danger = auto()
    warning = auto()
    active = auto()

print(list(Priority))

>>[<Priority.default: 1>, <Priority.danger: 2>, <Priority.warning: 3>, <Priority.active: 4>]
```

#### 1.3 <a name="1.3"></a>唯一值初始化 @unique
>在建立枚举之前用@unique，会检查枚举成员的value是否重复，与数据库的unique相同，当有重复项是会报错

```python
from enum import Enum,unique

@unique
class Priority(Enum):
    default = 0
    danger = 1
    warning = 2
    active = 2

>>ValueError: duplicate values found in <enum 'Priority'>: active -> warning
```
### 二、<a name="2"></a>枚举的使用 name以及value

使用枚举有三种方法，以下三种方法获取的是枚举对象 `<enum 'Priority'>`

* 对象属性：`Priority.default`
* 字典：`Priority["danger"]` 适用于已知枚举成员字符串的情况
* 对象: `Priority(1)` 适用于已知枚举成员数字值的情况

枚举对象有两个属性

* name：获取对象的字符串，`Priority(0).name` ==> `"default"`
* value：获取对象的数字值，`Priority["danger"].value` ==> `1`




### 三、<a name="3"></a>其他特性

#### 3.1 <a name="3.1"></a>可迭代对象
>枚举类时一个可迭代对象，可以使用for循环，用于展示字符串字符串

```python
for i in Priority:
    print(i)

>>Priority.default
>>Priority.danger
>>Priority.warning
>>Priority.active

```

#### 3.2 <a name="3.2"></a>可哈希对象

>枚举对象存在__hash__()的方法，意味着可以作为dict的key，适应更多应用

```python
priority={}
priority[Priority.danger] = "危险"
priority[Priority.warning] = "警告"
priority[Priority.warning] = "默认"

print(priority)
print(priority[Priority(1)])xiaohe
```

