# Error

>我觉得嵌入式与软件开发比较明显的区别就在于错误处理，从常规的类型错误到后面的处理错误，在初学软件开发时，感觉就是像在堵窟窿，要预判到各种error的出现，防止软件的崩溃。

## 一、常规使用方法

```python
try:
    a = '123'
    b = 1
    c = a + b
except Exception as e:
    print(type(e))			# <class 'TypeError'>
    print(e.__repr__())		# TypeError('must be str, not int',)
    print(e.args)			# ('must be str, not int',)
    print(e)              	# must be str, not int
finally:
    c = str(a)+str(b)
```

## 二、Error对象

>可以看到上面的例子中，e其实是TypeError对象（python中一切都是对象）

#### 2.1 Error类的继承关系

所有的内置错误类都是继承[Exception类](https://docs.python.org/3.6/library/exceptions.html#Exception)（开发者在自定义错误类的时候继承Exception类），Exception继承[BaseException类](https://docs.python.org/3.6/library/exceptions.html#BaseException)。

#### 2.2 Error对象的属性

>Error对象中比较常用的属性就属`args`了，在大多数情况下（OSError略有不同），如果用户并未指定，则args是包含错误信息的元组。

除了在常规弹出错误，在`raise Error()`时，也可以将参数传入Error对象中，通过调用`args`属性来做下一步处理

```python
try:
    raise TypeError('title','value1','value2')
except Exception as e:
    print(type(e))	# <class 'TypeError'>
    print(e.args)	# ('title', 'value1', 'value2')
    print(e)  		# ('title', 'value1', 'value2')
    
try:
    raise TypeError('title')
except Exception as e:
    print(type(e))	# <class 'TypeError'>
    print(e.args)	# ('title',)
    print(e)   		# title
```

#### 2.3 自定义Error类

用户在自定义Error类时，同样可以添加属性，方便处理。

```python
class CustomError(Exception):

    class_attr = 'classAttr'

    def __init__(self, obj_attr, *args, **kwargs):
        self.obj_attr = obj_attr

error = CustomError('title','arg1','arg2')

try:
    raise error		# 此处raise CustomError('title','arg1','arg2')效果相同
except Exception as e:
    print(type(e))   			# <class '__main__.CustomError'>
    print(e.args)				# ('title', 'arg1', 'arg2')
    print(e.class_attr)			# classAttr
    print(e.obj_attr)			# title，与e.args第一个参数相同
```

```python
class CustomError(Exception):

    def __init__(self, *args, obj_attr=None, **kwargs):
        self.obj_attr = obj_attr

try:
    raise CustomError('title','arg1','arg2', obj_attr='111')
except Exception as e:
    print(e.args)                # ('title', 'arg1', 'arg2') 与位置变量相同
    print(e.obj_attr)            # 111，建议采用带入关键字变量的方式进行对象自定义属性初始化

```

在处理错误时，要注意该错误类型是否有自定义属性，防止再次报错

### 2.4 `Error.__repr__`

在实际测试中，想要打印出错误的类型以及错误信息，用`print(e)`是不够的，看例子

```python
try:
    a = 1
    b = ''
    c = a + b
except Exception as e:
    print(e)            # unsupported operand type(s) for +: 'int' and 'str'
    print(e.__str__())  # unsupported operand type(s) for +: 'int' and 'str'
    print(e.__repr__()) # TypeError("unsupported operand type(s) for +: 'int' and 'str'",)
```
在使用`__repr__()`内置方法，才能打印出完整的错误信息。
所以在自定义错误类时，建议对`__str__`方法（print优先调用的方法）或者`__repr__`进行修改，在抓错误时写日志时，可以快速定位错误

```python
class SqlError(Exception):
    '''
        自定义错误类型，用于在数据库操作时产生错误的集中处理
    '''
    def __init__(self, *args, error=None, **kwargs):
        self.error = error	 # 系统产生的错误
        self.message = args[0] # 系统产生的错误提示信息
        self.error_method = "method:%s" % args[1] if len(args) > 1 else ""

    def __str__(self):
    	 '''返回的错误信息，可直接打印'''
        return "[SqlError]%s %s==> [%s]" % (self.message, self.error_method, self.error.__repr__())
```
