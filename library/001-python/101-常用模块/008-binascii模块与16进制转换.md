


##一、python中的16进制

#### 1.1 byte对象
>做惯了嵌入式，脑子中的想法就是int类型可以通过强制转换`(byte)int`来进行转换，但是在python中，并没有byte类型，而bytes中的每个值又是一个int。

```
a = b'\x00\xff'

type(a)
>><class 'bytes'>

[i for i in a]
>>[0, 255]

[type[i] for i in a]
>>[<class 'int'>, <class 'int'>]
``` 
bytes的初始化，bytes([int,int])，byte(int)则表示申请N个值

```
bytes(5) 		# 5代表字节个数
>>b'\x00\x00\x00\x00\x00'

bytes([255, 128]) 
>>b'\xff\x80'

bytes([256, 128])	# 队列/元组内的数据要在0-255的
>>ValueError: bytes must be in range(0, 256)
```

## 二、binascii模块

需求：如果用普通的方法，解析BCD，'10A1B2C3D4E5F6'

1. 先将BCD拆分成2字符的队列

	```
	re.findall('\w\w', '10A1B2C3D4E5F6')
	>>['10', 'A1', 'B2', 'C3', 'D4', 'E5', 'F6']
	```
2. 通过`int(x, base=10)`将字符串转换为int型，`base`为进制
	
	```
	[int(i,base=16) for i in re.findall('\w\w', '10A1B2C3D4E5F6')]
	>>[16, 161, 178, 195, 212, 229, 246]
	```
3. 再通过bytes转换
	
	```
	bytes([16, 161, 178, 195, 212, 229, 246])
	>>b'\x10\xa1\xb2\xc3\xd4\xe5\xf6'
	```

>binascii中的方法，可以直接将BCD字符串与bytes直接转换<br>b代表BinHex，a代表ascii，BCD格式指 b'\xb5' => 'B5'

---

### `b2a_hex()` 与 `a2b_hex()`
bytes(n字节) => ascii(2n字节)，在py3中需要解码（utf8 or ascii）

```
b2a_hex(b'\x10\xa1\xb2\xc3\xd4\xe5\xf6')
>>b'10a1b2c3d4e5f6'

b2a_hex(b'\x10\xa1\xb2\xc3\xd4\xe5\xf6').decode('utf8')
>>'10a1b2c3d4e5f6'
```

ascii(2n字节) => bytes(n字节)

```
a2b_hex(b'10A1B2C3D4E5F6')
>>b'\x10\xa1\xb2\xc3\xd4\xe5\xf6'
```

## 三、常用转换

|原格式|转换格式|方法|示例|返回值|
|:---:|:---:|:---:|:---:|:---:|
|字符串|int|int(x, base=10) => int|int('6800', 16)<br>int('6800')|26624<br>6800|
|int|字符串（16进制）|hex(i) => str|hex(256)|'0x100'|
