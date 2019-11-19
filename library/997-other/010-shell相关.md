# shell相关

[TOC]

## 常见问题

### ash Link unknown operand

这种是因为某个判断条件写得不正确

### 错误输出获取不到

在linux中，打开任意的应用程序都会产生3个流，输入（0），输出（1），错误（2）

在shell中

```shell
$VAR_NAME=`command`
```

这种方法其实是将command的输出“赋值”给变量，即使我们看到错误信息出现了，也无法在变量中找到，其原因就在于输出与错误是不同的文件描述符（fd）。

这时我们就需要将错误输出也输出到标准输出

```shell
$VAR_NAME=`command 2>$1`	# 将错误信息输出到标准输出
$VAR_NAME=`command >/dev/null 2>$1`		# 标准输出只输出错误信息
$VAR_NAME=`command 1>/dev/null 2>$1`	# 等价于上面那条

```

