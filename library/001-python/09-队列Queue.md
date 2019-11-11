# 队列Queue

>在实际工作当中，在多线程操作时，经常需要多个线程对同一个数据进行处理，就用到队列这个概念，包含先进先出(FIFO)，后进先出(LIFO)两种模型

>python中自带了queue模块，只需要import导入即可

## 一. 初始化
python中一切都是对象，队列也不例外，初始化类似建立对象的过程

```
class Queue:
    '''Create a queue object with a given maximum size.

    If maxsize is <= 0, the queue size is infinite.
    '''

    def __init__(self, maxsize=0):
        self.maxsize = maxsize
        self._init(maxsize)
        
q = queue.Queue(5)				# 默认：先进先出
q = queue.PriorityQueue(5)		# 优先级
q = queue.LifoQueue(5)			# 后进先出

q = Queue(0) 		# 看文档说明，当参数<=0时，代表队列无限大
```

```python
# 优先级队列示例
q = queue.PriorityQueue(5)

# 在向队列添加数据时，需要通过元组的形式。(优先级数字，对象)，数字小优先
q.put((1,"dddd"))
q.put((0,"aaaa"))
q.put((4,"bbb"))
q.put((2, "ccc"))

print(q.get())  # >>(0, 'aaaa')
q.task_done()
```

## 二. 队列操作

#### 增加元素：put 方法
```
def put(self, item, block=True, timeout=None):
    '''Put an item into the queue.’‘’
    
q = queue.Queue(5)

for i in range(6):
    q.put(i,block=True, timeout=None)
```

* block=True timeout=None，默认添加时，当队列已满，则当前线程会阻塞。
* block=True timeout=n(秒)，当队列已满时，在timeout期间并未有空闲位置产生，则弹出Full错误
* block=False，当队列已满，再往里插入时，直接报错

#### 取出元素：get 方法，task_done

```python
def get(self, block=True, timeout=None):
	'''Remove and return an item from the queue.‘’‘

for i in range(6):
    _ = q.get()
    q.task_done()  # 需要在每个get后面加入task_done表示数据取出完成
```

* block=True timeout=None，默认添加时，当队列为空，则当前线程会阻塞，直到队列中有新的数据出现。
* block=True timeout=n(秒)，当队列为空时，在timeout期间有数据进来，则弹出Empty错误
* block=False，当队列已空，再取数据则弹出Empty错误

#### 队列属性

* 队列长度：`qsize`
* 队列是否为空：`empty`，实质是通过qsize判断
* 队列是否已满：`full`，实质是通过maxsize和qsize判断 

## 三、阻塞 join()

>在实际应用中，有这样一种实际场景，现在队列中填充一组数据，这组数据处理完之前不允许添加新的数据，那么就可以用join方法。

join方法在以下情况堵塞

* 队列不为空
* 在使用get方法时，没有使用task_done结束的情况




