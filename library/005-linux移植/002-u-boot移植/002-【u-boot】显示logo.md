# u-boot移植笔记之 显示LOGO

* 开发板：正点原子imx6ul ALPHA开发板
* u-boot：NXP官方，2016.03版本

> 应该有很多应用场景，要求在启动u-boot时在LCD上显示LOGO，

[TOC]

## 一、修改文件以显示自定义logo

> u-boot中显示logo的流程如下

* 编译阶段：将logo图片转换为数组，并生成数据文件（两个.h）文件
* 运行阶段
  * 引用这两个数据文件
  * 并根据LCD的参数（数据位宽等）处理数据
  * 写入LCD缓冲并显示



### 1.1 logo图片查找

* 图片存放位置 `tools/logos/xxx.bmp`
* 图片路径查找，发现在`tools/Makefile`中，存在如下语句

```makefile
# Generic logo
ifeq ($(LOGO_BMP),)
LOGO_BMP= $(srctree)/$(src)/logos/denx.bmp

# Use board logo and fallback to vendor
ifneq ($(wildcard $(srctree)/$(src)/logos/$(BOARD).bmp),)
LOGO_BMP= $(srctree)/$(src)/logos/$(BOARD).bmp
else
ifneq ($(wildcard $(srctree)/$(src)/logos/$(VENDOR).bmp),)
LOGO_BMP= $(srctree)/$(src)/logos/$(VENDOR).bmp
endif
endif

endif
```

如果定义了`BOARD`那就用`./tools/logos/$(BOARD).bmp`作为logo，否则用`./tools/logos/$(VENDOR).bmp`

### 1.2 logo解码

> logo是一个bmp文件，需要将文件解码，将数据区提取出来，做成数据，由u-boot其他的代码调用并显示。u-boot提供了这种工具，工具的名称就叫bmp_logo，有由`tools/bmp_logo.c`编译生成

在`tools/Makefile`中找到

```makefile
$(LOGO_H):	$(obj)/bmp_logo $(LOGO_BMP)
	$(obj)/bmp_logo --gen-info $(LOGO_BMP) > $@

$(LOGO_DATA_H):	$(obj)/bmp_logo $(LOGO_BMP)
	$(obj)/bmp_logo --gen-data $(LOGO_BMP) > $@
	
我把他打印出来了
tools/bmp_logo --gen-info ./tools/logos/freescale.bmp > include/bmp_logo.h
tools/bmp_logo --gen-data ./tools/logos/freescale.bmp > include/bmp_logo_data.h

```

可以看到，最终的目的就是生成`include/bmp_logo.h`和`include/bmp_logo_data.h`文件‘，查看其中的`bmp_logo_data.h`就可以发现，其中有一个已经将图片文件转换成RGB阵列的数据



### 1.3 logo数据调用

通过搜索include/bmp_logo_data.h中的图片数据数组，发现，是如下函数使用了数组

```
-->stdio_add_device
	-->dri_video_init
		-->video_init
			-->video_logo		// 将logo以及console文字转换好，按照位置写入缓冲中
				-->logo_plot	
					-->plot_logo_or_black
```



### 1.4 总结

* 如果想要修改logo，只需要在`tools/logos/`文件夹下添加`$BOARD.bmp`或者`$VENDOR.bmp`即可，bmp必须为8位（后面会讲解）

* 在NXP提供的官方例程中，logo是伴随着console出现的，即logo和控制台信息等会在屏幕中显示，如果不需要控制台，只需要屏蔽

  ```c
  ./driver/video/cfb_console.c 的 static void *video_logo(void) 函数中
  
  	sprintf(info, " %s", version_string);
  
  	space = (VIDEO_COLS - VIDEO_INFO_X) / VIDEO_FONT_WIDTH;
  	len = strlen(info);
  
  	if (len > space) {
  		int xx = VIDEO_INFO_X, yy = VIDEO_INFO_Y;
  		uchar *p = (uchar *) info;
  		while (len) {
  			if (len > space) {
  				video_drawchars(xx, yy, p, space);
  				len -= space;
  
  				p = (uchar *) p + space;
  
  				if (!y_off) {
  					xx += VIDEO_FONT_WIDTH;
  					space--;
  				}
  				yy += VIDEO_FONT_HEIGHT;
  
  				y_off++;
  			} else {
  				video_drawchars(xx, yy, p, len);
  				len = 0;
  			}
  		}
  	} else
  		video_drawstring(VIDEO_INFO_X, VIDEO_INFO_Y, (uchar *) info);
  ```

  

​	

## 二、显示无调色板bmp文件（8位以上数据宽度）

Q：当我天真的以为只需要替换文件即可显示。但是我将一个24bit位宽的bmp文件进行替换时，就无法正常显示

R：u-boot中对于bmp文件的支持不够全面，只支持带有调色板的bmp文件（具体的请查阅相关资料）

A：修改代码（无调色板）

	* 读取logo文件，将图片数据信息直接读取（没有调色板，那bitmap就是颜色数据）
	* 在写入阶段，将数据直接写入，跳过调色板匹配阶段

### 2.1 图片格式 转 数据数组 部分

> /tools/bmp.logo.c 的 main函数

**旧流程**如下，具体的可以看代码

* 读取目标文件
* 查看头两个字节是否为"BM"
* 读取头文件，查看分辨率（宽 * 高），数据偏移量（头文件 + 调色板 长度），颜色索引数（无调色板为0）
* 调整颜色索引数（在这里，当索引为0时，颜色索引数自动切换为默认值），**需要修改**
* 读取调色板信息（16位），读取位图信息（8位），存储到相应的数组中，**需要修改**



可以看出来，这里u-boot就没考虑无调色板的处理（因为logo的数据要编译进u-boot.bin中，8bit带调色板文件比较小）

**新流程**添加如下

* 在`struct bitmap_s`中添加`hasPalette`
* 在读取出颜色索引数 和 数据偏移量之后，如果为无调色板数据
  * `b->hasPalette = false`
  * 在bmp_logo.h中生成`#define BMP_LOGO_NO_PALETTE`，用于后面LCD显示执行时
  * 在bmp_logo.h中生成`extern unsigned long bmp_logo_data[]`，注意使用`unsigned long`代表32位
* 删除对于0颜色索引数的默认修改
* 添加对无调色板数据的存储。



**数据顺序问题**

	* 图片扫描顺序，左->右，下->上
	* LCD显示顺序，左->右，上->下
	* 在生成数据时，就要以LCD显示顺序为准



### 2.2 数据读取，写入缓冲

> /driver/video 的 void plot_logo_or_black(void *screen, int x, int y, int black)

**旧流程**

1. 计算需要显示的长度 + 宽度 + skip （（屏幕宽度 - logo宽度）* 像素数据宽度）
2. 申请缓存，将调色板信息读取出来
3. 遍历8位位图表，带入调色板索引，将数据拓宽（8bit --> 32bit）放入显示缓存

**新流程**

	* 只需要跳过2，3步，将数据直接写入缓存即可

**一些宏定义的含义，方便查看源码**

```
VIDEO_LOGO_WIDTH
VIDEO_LOGO_HEIGHT
VIDEO_PIXEL_SIZE	一个像素所代表的宽度，在这里是4
VIDEO_DATA_FORMAT	bmp的格式，包括位宽以及格式，这里是一个枚举 GDF_32BIT_X888RGB = 3

```

## 三、下载地址

需要修改的文件如下

* `tools/bmp_logo.c`
* `drivers/video/cfb_console.c`

[下载链接](http://cs.bloath.com/file/uboot-logo.zip)