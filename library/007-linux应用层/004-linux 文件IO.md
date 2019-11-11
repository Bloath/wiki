# linux 文件I/O

* [标准IO](#1)
* [linux文件IO](#2)
	* [打开关闭文件](#file-description)
	* [读写文件](#file-rw)
	* [文件定位](#file-location)
	* [文件权限](#file-permission)
	* [文件夹](#file-dir)
* [IN-OUT-ERROR](#in-out-error)
* 附录
	* [标准IO - 打开文件模式](#fopen-mode)
	* [linux文件IO - 打开文件模式](#oflag)
	* [linux文件IO - 权限属性宏定义](#mode_t)

>linux的文件主要分为6种

* 普通文件
* 目录文件
* 符号链接文件
* 管道文件
* 套接字文件
* 设备文件

>对于linux来说，操作I/O有两种方式，ANSIC的标准I/O以及POSIX的文件I/O，可以说，标准I/O接口是基于POSIX的文件I/O编写的。
>

||标准I/O|文件I/O|
|:---:|:---:|:---:|
|标准|ANSIC|POSIX|
|特点|具有缓冲机制<br>属于上层API<br>通过流控制|直接读写文件<br>介于用户层与文件系统之间偏底层<br>通过文件描述符控制|
|读写特点|先写入流，在流满或换行时，进行系统调用|每次读写都会引起系统调用|

>每一个进程在启动时，系统都给三个标准流，输入、输出以及错误

* 对于标准I/O，stdin、stdout、stderr作为流，带入标准I/O中进行操作
* 对于linux文件IO。标准流则以0，1，2三个文件描述符的形式



## 一、 **标准I/O**

* **打开/关闭流**
	* `FILE *fopen( const char *filename, const char *mode );`，[流模式列表](#fopenMode)
	* `int fclose( FILE *stream );`
* **读写流**
	* 按单字符读写	
		* `int fgetc( FILE *stream );`
		* `int getc( FILE *stream );` 
		* `int fputc( int ch, FILE *stream );`
		* `int putc( int ch, FILE *stream );`
		* `getchar()` = `fgetc(stdin)` `putchar()` = `fputc(stdout)`
	* 按行读写
		* `char *fgets( char *str, int count, FILE *stream );`，`str`为读出字符串后写入的部分，`count`为最大接收长度（一般为str的长度），产生错误返回值为NULL，否则为str
		* `int fputs( const char *str, FILE *stream );`
		* `gets(str)` = `fgets(str, 1000, stdin)` `puts(str)` = `fputs(str, stdout)`
	* <font color="red">按对象读写</font>：`size`为对象类型的宽度(`sizeof(int)`)，`count`为要写入的长度
		* `size_t fwrite( const void *buffer, size_t size, size_t count, FILE *stream );`
		* `size_t fread( void *buffer, size_t size, size_t count, FILE *stream );`
	* 按格式读写，与`printf`差不多，区别仅在于写入的流不同，`printf`写入的是`stdout`
		* `int fprintf( FILE *stream, const char *format, ... );`
		* `int sprintf( char *buffer, const char *format, ... );`
* **位置处理**
	* `long ftell( FILE *stream );`
	* `void rewind( FILE *stream );` 等同于 `fseek(stream, 0, SEEK_SET);`
	* `int fseek( FILE *stream, long offset, int origin );`, `origin`参数分别有`SEEK_SET/ SEEK_CUR/SEEK_END`，代表头/当前/尾
* **条件判断**
	* `int feof( FILE *stream );`
* **其他**
	* `int fflush( FILE *stream );`，直接将流中的数据写入文件
	* `void perror( const char *s );`，打印错误信息（错误信息已经在`errorno`或者`strerr`中），`s`为打印信息的前导字


## 二、文件I/O

### 2.1 文件描述符<a name="file-description"></a>

>通过文件I/O打开文件时，返回的是 **文件描述符**，一个非负整数。

* 打开/关闭文件
	* `int open(const char *path, int oflag, int perms);`需要`#include <fcntl.h>`
	* `int close(int fd);`，需要`#include <unistd.h>`

关于参数`oflag` ==> [linux文件IO - 打开文件模式](#oflag)
       
<br>
***
### 2.2 文件的读写操作<a name="file-rw"></a>

>文件的读写则比较简单，只有写字节操作，返回值都为操作正确

* `ssize_t write(int fildes, const void *buf, size_t nbyte);`，需要`#include <unistd.h>`
* `ssize_t read(int fildes, void *buf, size_t nbyte);`，需要添加以下三个头文件
	* `#include <sys/types.h>`
	* `#include <sys/uio.h>`
	* `#include <unistd.h>`
 

<br>
***

### 2.3 文件定位<a name="file-location"></a>

`off_t lseek(int fildes, off_t offset, int whence);`，需要`#include <unistd.h>`

* 参数
	* `whence`与标准库的相同，`SEEK_SET/ SEEK_CUR/SEEK_END`，代表头/当前/尾
* 返回值，返回定位后的具体位置。

<br>
***
### 2.4 文件权限与属性<a name="file-permission"></a>

*  文件属性 `#include <sys/stat.h>`
	* `int stat(const char *path, struct stat *buf);`，链接文件所指文件属性
	* `int lstat(const char *path, struct stat *buf);`，链接文件本身属性
	* `int fstat(int fd, struct stat *buf);` 
	
		```
		struct stat  
		{   
		    dev_t       st_dev;     /* ID of device containing file -文件所在设备的ID*/  
		    ino_t       st_ino;     /* inode number -inode节点号*/    
		    mode_t      st_mode;    /* protection 权限*/    
		    nlink_t     st_nlink;   /* number of hard links -链向此文件的连接数(硬连接)*/    
		    uid_t       st_uid;     /* user ID of owner -user id*/    
		    gid_t       st_gid;     /* group ID of owner - group id*/    
		    dev_t       st_rdev;    /* device ID (if special file) -设备号，针对设备文件*/    
		    off_t       st_size;    /* total size, in bytes -文件大小，字节为单位*/    
		    blksize_t   st_blksize; /* blocksize for filesystem I/O -系统块的大小*/    
		    blkcnt_t    st_blocks;  /* number of blocks allocated -文件所占块数*/    
		    time_t      st_atime;   /* time of last access -最近存取时间*/    
		    time_t      st_mtime;   /* time of last modification -最近修改时间*/    
		    time_t      st_ctime;   /* time of last status change - */    
		};  
	
		```
	
* 文件权限 
	* `#include <sys/types.h>` 
	* `#include <sys/stat.h>`
	* `int chmod(const char *path, mode_t mode);`
	* `int fchmod(int fd, mode_t mode);` [mode_t宏定义列表](#mode_t)


<br>
***
### 2.4 文件夹相关操作<a name="file-dir"></a>

头文件`#include <dirent.h>`

* 打开目录文件
	* `DIR *opendir(const char *filename);`
	* `DIR *fdopendir(int fd);`
	* `int closedir(DIR *dirp);`
	* `DIR`类型与`FILE`类型了解即可，无需我们去深究
* 读目录下文件列表
	* `struct dirent *readdir(DIR *dirp);`，顺序读取文件夹下元素，直到返回NULL
	* `struct dirent`
	
		```
		struct dirent   
		{   
		　　long d_ino; /* inode number 索引节点号 */  
		    off_t d_off; /* offset to this dirent 在目录文件中的偏移 */  
		    unsigned short d_reclen; /* length of this d_name 文件名长 */  
		    unsigned char d_type; /* the type of d_name 文件类型 */  
		    char d_name [NAME_MAX+1]; /* file name (null-terminated) 文件名，最长255字符 */  
		}  
		```


<br>

## 三、IN-OUT-ERROR<a name="in-out-error"></a>

>在开启一个进程时，系统都会分配三个流，输入流、输出流、错误流，

* 对于标准I/O，分别为`stdin`、`stdout`、`stderr`，以`FILE`流的形式存在，可以通过流函数进行操作
* 对于linux文件I/O，分别为`STDIN_FILENO`、`STDOUT_FILENO`、`STDERR_FILENO`，以文件描述符的形式存在

>对于输入输出来说，用法比较常见，这里重点讲一下错误

我们常用的打印错误的函数为`perror(char* prefix)`，以文件不存在错误为例

1. 进程中维护一个全局错误代码数据`errno`，在`errno.h`中
* 产生错误后，将错误代码写入`errno`中，有新的错误`errno`原有值被替换
* 通过`strerror(int errnum)`函数将`int`型的`errno`转换为可看懂的字符串
* `perror(char* prefix)`在标准流中等价于`fprintf(stderr, "%s: %s", prefix, strerror(errno))`
	* 将错误代码转换为字符串
	* 再将该字符串，添加字符串前缀，输出到err标准流中（对于linux I/O则是写入`STDERR_FILENO`描述符所指文件中）



<br>

## 附录

#### `fopen`函数模式<a name="fopen-mode"></a>

<div align="center"><img src="http://cs.bloath.com/img/fopenMode.png" alt=""></div>

#### `oflag`模式<a name="oflag"></a>
```
// 前三个为互斥关系
O_RDONLY        open for reading only
O_WRONLY        open for writing only
O_RDWR          open for reading and writing
   
O_NONBLOCK      do not block on open or for data to become available
O_APPEND        append on each write
O_CREAT         create file if it does not exist
O_TRUNC         truncate size to 0
O_EXCL          error if O_CREAT and the file exists
O_SHLOCK        atomically obtain a shared lock
O_EXLOCK        atomically obtain an exclusive lock
O_NOFOLLOW      do not follow symlinks
O_SYMLINK       allow open of symlinks
O_EVTONLY       descriptor requested for event notifications only
O_CLOEXEC       mark as close-on-exec
```
#### `mode_t`宏定义<a name="mode_t"></a>

```
#define S_IFMT 		0170000    	/* type of file */
#define S_IFIFO 	0010000  	/* named pipe (fifo) */
#define S_IFCHR 	0020000  	/* character special */
#define S_IFDIR 	0040000  	/* directory */
#define S_IFBLK 	0060000  	/* block special */
#define S_IFREG 	0100000  	/* regular */
#define S_IFLNK 	0120000  	/* symbolic link */
#define S_IFSOCK	0140000  	/* socket */
#define S_IFWHT 	0160000  	/* whiteout */

#define S_ISUID 	0004000  	/* set user id on execution */
#define S_ISGID 	0002000  	/* set group id on execution */
#define S_ISVTX 	0001000  	/* save swapped text even after use */

#define S_IRWXU 0000700    /* RWX mask for owner */
#define S_IRUSR 0000400    /* R for owner */
#define S_IWUSR 0000200    /* W for owner */
#define S_IXUSR 0000100    /* X for owner */

#define S_IRWXG 0000070    /* RWX mask for group */
#define S_IRGRP 0000040    /* R for group */
#define S_IWGRP 0000020    /* W for group */
#define S_IXGRP 0000010    /* X for group */

#define S_IRWXO 0000007    /* RWX mask for other */
#define S_IROTH 0000004    /* R for other */
#define S_IWOTH 0000002    /* W for other */
#define S_IXOTH 0000001    /* X for other */

#define S_ISUID 0004000    /* set user id on execution */
#define S_ISGID 0002000    /* set group id on execution */
#define S_ISVTX 0001000    /* save swapped text even after use */
```

