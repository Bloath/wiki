# pymysql

* [数据库连接-connect](#1)
* [执行SQL语句-execute/commit](#2)
* [获取查询数据-fetch/scroll](#3)



#### 1、<a name="1"></a>数据库连接-connect

示例：

```python
conn = pymysql.Connect(host='localhost',port=3306 user='username', password='password', database='db_module6', charset='utf8', cursorclass=pymysql.cursors.DictCursor)

cursor = conn.cursor() # 获取数据库指针
```



#### 2. <a name="2"></a>执行SQL语句-excute/commit

通过`cursor.execute(sql，args)`方法执行sql语句，增删改等语句，在执行完后通过`con.commit`方法进行提交

```python
def execute(self, query, args=None):
    """Execute a query

    :param str query: Query to execute.

    :param args: parameters used with query. (optional)
    :type args: tuple, list or dict

    :return: Number of affected rows
    :rtype: int

    If args is a list or tuple, %s can be used as a placeholder in the query.
    If args is a dict, %(name)s can be used as a placeholder in the query.
    """
```

例如插入多条，可以通过`cursor.executemany(sql, args)`，例如

`cursor.executemany(insert into table(c1,c2) values(%s,%s), [(a,b),(c,d)])`

#### 3. 获取查询数据-fetch/scroll

>在通过execute执行后，通过fetch获取到查询数据，数据的格式是元组或者是字典，默认为元组通过`cursorclass=ymysql.cursors.DictCursor`修改为字典格式。

* fetchone()
* fetchall()
* fetchmany(size)

示例

```python
num = cursor.execute("select * from user")
result1 = cursor.fetchone()
result2 = cursor.fetchall() # 执行过一次fetch之后，无法再次取出第一条
```
>通过fetch取出数据时，其内部维护有递增指针，执行一次fetchone，则递增1，可以通过scroll方法，改变指针位置
>

```python
def scroll(self, value, mode='relative'): 
	 # 内部关键语句如下
    if mode == 'relative':
        r = self.rownumber + value
    elif mode == 'absolute':
        r = value
   	

num = cursor.execute("select * from user")
result1 = cursor.fetchone()		
cursor.scroll(0,mode='absolute') # 绝对定位，将指针指向第一条数据
result2 = cursor.fetchone()



```

