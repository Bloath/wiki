# 【Linux C】文件IO

> linux的特点就是一切皆文件，那么对于任何操作来讲，在应用层面，本质就是对于文件的读写操作

[linux man page -- linux 接口查询](http://man.he.net/)

[TOC]

## 一、API列表

### 标准I/O

> 标准I/O操作的对象称之为流（stream），在打开文件时，系统会创建FILE结构体将实际文件进行关联。
>
> 一般用于访问 **普通文件**
>
> 在应用程序启动时，系统会自动打开三个流，stdin、stdout、stderr，stderr是通过`void perror(const char *prefix)`进行输出

```c
#include <stdio.h>

// 打开与关闭
FILE* fopen(const char *path, const char *mode);
int fclose(FILE *stream);

// 读写：字节
int fgetc(FILE *stream);
int getc(FILE *stream);
int fputc(FILE *stream);
int putc(FILE *stream);
int getchar(void); 	// 等价于 getc(stdin)
int putchar(int c);	// 等价于 putc(stdout)

// 读写：行
int fgets(char *s, int size, FILE *stream);
int fputs(char *s, FILE *stream);

// 读写：对象，按照指定大小进行读写，size为对象宽度，nmemb为对象数量
size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);
size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);

// 读写 按照指定格式
int fscanf(FILE *stream, const char *format, ...);
int sscanf(const char *str, const char *format, ...);
int scanf(const char *format, ...);			// 等价于 fscanf(stdin)
int fprintf(FILE *stream, const char *format, ...);
int sprintf(char *str, const char *format, ...);
int printf(const char *format, ...);		// 等价于 fprintf(stdout)

// 定位
int fseek(FILE *stream, long offset, int whence);
long ftell(FILE *stream);					// 返回当前读写位置
int feof(FILE *stream);

// 缓冲相关
int fflush(File *stream);
int setvbuf(FILE *stream, char *buf, int mode, size_t size);		// 设置缓冲类型
void setbuffer(FILE *stream, char *buf, size_t size);	// 等价于 setvbuf(stream, buf, buf ? _IOFBF : _IONBF, BUFSIZ);
void setlinebuf(FILE *stream);	// 等价于 setvbuf(stream, NULL, _IOLBF, 0);
```

### 文件I/O

>标准I/O操作的对象称之为文件描述符，为一个int型整数
>
>通常用于读写 **设备文件**，**管道文件** 等
>
>在应用程序启动时，系统会自动打开三个文件描述符，分别为0，1，2，STDIN_FILENO，STDOUT_FILENO，STDERR_FILENO

```c
#include <sys/stat.h>
#include <sys/types.h>
#include <fcntl.h>
#include <unistd.h>

// 打开与关闭
int open(const char *pathname, int flags, int perms);
int close(int fd);

// 读写
ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, void *buf, size_t count);

// 定位
off_t lseek(int fd, off_t offset, int whence);

```





## FAQ. 常见问题

### 减少文件写入频次

通过`setvbuf`系列函数，将缓冲扩大，默认的缓冲为1KB，可以根据自己的要求进行修改