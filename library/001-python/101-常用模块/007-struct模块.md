# struct模块
>在通讯协议中，大部分数字信息都通过定长的2字节4字节来表示，struct模块可以通过自定义`fmt`对固定长度的bytes数据进行解析，解析为python常用格式，对于处理通讯协议极为友好


### 一、常用方法

|方法名|作用|备注|
|:---|:---|:---|
|反格式化|||
|`struct.unpack(fmt, buffer)`|反序列化，返回元组||
|`struct.unpack_from(fmt, buffer, offset=0)`||offset到后面的长度要符合`fmt`的长度|
|`struct.iter_unpack(fmt, buffer)`|反序列话，返回迭代器||
|格式化化|||
|`struct.pack(fmt, *args)`|返回bytes||
|`struct.pack_into(fmt, buffer, offset, *args)`|不返回，向buffer offset往后添加||
|其他|||
|`struct.calcsize(fmt)`|返回`fmt`的长度||

### 二、Struct对象

如果该fmt需要大量使用时，则可以通过建立Struct对象，通过对象内置方法完成操作
```python
import struct

fmt = struct.Struct("BBH")
message = b"\x68\x10\x12\x00"
print(fmt.unpack(message))
```

### 三、format字符
>在介绍struct format字符之前，针对不同平台不同编译器产生的对齐方式差异(高字节在前/低字节在前)、类型长度的差异也有相应的对策，通过`fmt`字符串的第一个字符来确定采用什么样的格式

|字符|byte order|size|alignment|
|:---:|:---:|:---:|:---:|
|`@`|native|native|native|
|`=`|native|standard|none|
|`<`|little-endian|standard|none|
|`>`|big-endian|native|none|
|`!`|network(=big-endian)|standard|none|

* 默认情况下，为@
* `size`在`native`的情况下，取决于C编译器，而在`standard`情况下，则只取决于特殊字符所指长度
* 如果是用于网络通讯时，用`!`，例如`!BBH`

|字符|C Type| python Type| 长度| 备注|
|:---:|:---:|:---:|:---:|:---:|:---:|
|`x	`| pad byte |	|||
|`c	`| char|	bytes of length 1|1|| 
|`b	`| signed char|	integer|1||
|`B	`| unsigned char	|integer|	1||
|`?`| 	_Bool|	bool|1||
|`h	`| short|	integer|2||
|`H	`| unsigned short|integer|	2||
|`i	`| int|integer|4||	
|`I	`| unsigned int|	integer|4||
|`l	`| long|integer|4||
|`L	`| unsigned long|integer|4||
|`q	`| long long|	integer|8||
|`Q	`| unsigned long| long	integer|8||
|`n	`| ssize_t|integer|||
|`N	`| size_t|integer|||
|`e	`| (7)	|float	|2||
|`f	`| float|	float	|4||	
|`d	`| double	|float|8||
|`s	`| char[]|bytes|	|需要在前面加数量，`7s`|
|`p	`| char[]|bytes||| 
|`P	`| void *|integer|| |

	 	