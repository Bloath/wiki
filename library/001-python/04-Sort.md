# 排序 

## 一、sort sorted

* `list.sort(self, key, reverse)` return `None`
* `sorted(iterable, key, reverse)` return `list`

区别

1. 使用场合的不同
	* sort为列表的内置方法
	* sorted可以将任意的可迭代对象带入进行排序，可用于字典key的排列
2. 返回值的不同
	* sort返回None，会将原列表直接进行排序，无需重新赋值。
	* sorted会将排序结果导入到新的list中
3. 在py3中，sorted去掉了cmp参数，其实在py2中，cmp也很少用

## 二、key
>排序参数中key的使用可以说是是否能灵活运用的关键，其原理是，通过一个参数为排序对象的方法，根据计算结果进行排序

```
lister = [1,4,23,4,4,4,3,4,32,32,32,3,2,43,24,23,42,42]
lister.sort(key=lambda x: x%5)

print(lister)
>>[1, 32, 32, 32, 2, 42, 42, 23, 3, 3, 43, 23, 4, 4, 4, 4, 4, 24]
```

也可以通过对象的属性，元组列表中的不同索引的值进行排序

```python
class Student:
     def __init__(self, name, grade, age):
         self.name = name
         self.grade = grade
         self.age = age
     def __repr__(self):
         return repr((self.name, self.grade, self.age))

student_objects = [
     Student('john', 'A', 15),
     Student('jane', 'B', 12),
     Student('dave', 'B', 10),
 ]

# 利用排序方法针对age属性进行排序
student_objects.sort(key=lambda stu:stu.age)
print(student_objects)

>>[('dave', 'B', 10), ('jane', 'B', 12), ('john', 'A', 15)]
```

### 多级排序
>在对象排序或者多元素元组列表排序时，难免会有多级排序的需求，py中提供了`itemgetter()`，`attrgetter()`和`methodcaller()`方法，该方法既能满足上面的`key=lambda stu:stu.age`此类简单的需求，同时也能带入多个参数进行多级排序

* `itemgetter` 用于索引寻址的对象排序，例如 itemgetter(1,2,3)
* `attrgetter` 用于对象，通过带入属性名进行排序, 例如
attrgetter("age","grade","name")
* `methodcaller` 与上两个相同，可以带入多个方法，进行多级排序

```python
from operator import methodcaller,attrgetter,itemgetter

class Student:
    def __init__(self, name, grade, age):
        self.name = name
        self.grade = grade
        self.age = age

    def __repr__(self):
        return repr((self.name, self.grade, self.age))


student_objects = [
    Student('john', 'A', 15), 
    Student('jane', 'B', 12),
    Student('jane', 'C', 12),
    Student('alex', 'A', 12),
    Student('dave', 'B', 10),
    Student('egon', 'B', 12),
 ]


student_objects.sort(key=attrgetter("age","grade","name"))
print(student_objects)

>>[('dave', 'B', 10), ('alex', 'A', 12), ('egon', 'B', 12), ('jane', 'B', 12), ('jane', 'C', 12), ('john', 'A', 15)]
```