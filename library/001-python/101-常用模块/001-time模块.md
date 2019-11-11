#time模块

* [time模块](#1)
	* [time的三种形式](#1.1)
	* [三种形式的互相转换](#1.2)
	* [time模块的其他功能](#1.3)


## time模块 <a name="#1"></a>

###1.1、time的三种展示形式<a name="#1.1"></a>

1. **timestamp**，时间戳，数字(float)，在上位机下位机交互时，经常使用时间戳作为事件的交互格式
	* `time.time()`--> 1509875395.843578
2. **format time**，自定义格式化的时间，字符串，通过用户指定的显示方式生成的表示时间的字符串，例如“2017-01-01 12:01:29”，
	* `time.strftime('%Y-%m-%d %X')`--> 2017-11-05 17:51:10

	* 常用：
		* 年月日 `%Y-%m-%d` 或者 `%x`(11/05/17==%m/%d/%y)
		* 时分秒 `%H:%M:%S` 或者 `%X`(18:03:11)

3. **struct time**，结构化时间，time.struct_time对象，将时间所有的信息通过属性和属性值，用户可调用对象的属性访问时间。(对象内部没有__dict__属性你敢信)
	* `time.localtime([seconds])` 按照本地时区转换
	* `time.gmtime([seconds])`	 按照0时区转换
	* 包含如下属性，
		* 常用 `tm_zone,tm_year,tm_mon,tm_mday,tm_hour,tm_min,tm_sec`
		* 非常用 `tm_wday(weekday[0-6])`,`tm_yday(year day [1-366])`, `tm_gmtoff`以及`tm_isdst`
		
---

### 1.2、三种time形式的互相转换<a name="#1.2"></a>

##### 1.2.1 源格式：timestamp时间戳
	
* **struct time**，`time.localtime([seconds])`以及`time.gmtime([seconds])`
* **string**，`time.ctime([seconds])`输出格式固定如下 `Sun Nov  5 20:46:58 2017`，
	
##### 1.2.2 源格式：format string 格式化时间字符串

* **struct time**，`time.strptime(string, format)`，string为要转换的时间字符串，format为该时间字符串对应的格式，例如`time.strptime("2017-1-2", "%Y-%m-%d")`

##### 1.2.3 源格式：struct time 结构化时间对象

* **format time**，`time.strftime(format, p_turple=none)`，format是指时间格式化规则字符串，p_turple则指的是需要转换的struct time，示例`time.strftime("%Y-%m-%d", time.localtime())`
* **string**，`time.asctime(p_tuple=None)`输出格式固定如下 `Sun Nov  5 20:46:58 2017`
* **timestamp**，`time.mktime(p_tuple=None)`，转换为时间戳

---

### 1.3、time模块其他功能<a name="1.3"></a>

* **time.sleep(seconds)** 线程休眠
* **time.process_time()** 从进程开启到执行该语句为止所用的时间，对系统记录很有帮助，3.3加入，替代原有的time.clock()（在unix和windows中，clock()返回不同的值）
* **time.perf_counter()** 系统运行时间

<br>



