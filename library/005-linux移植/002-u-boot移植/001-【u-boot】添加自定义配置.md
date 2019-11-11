# u-boot添加自定义配置

* 开发板：正点原子imx6ul ALPHA开发板

* u-boot：NXP官方，2016.03版本



相关的文件有

```
./configs/mx6ull_14x14_evk_defconfig	<-- 默认配置
./arch/arm/cpu/armv7/mx6/Kconfig	<--芯片所包含的板子资源列表
./include/configs/mx6ullevk.h		<--板子头文件
./board/freescale/mx6ullevk/		<--板子其他相关文件
	imxumage.cfg
	Kconfig
	MAINTAINERS
	Makefile
	mx6ullevk.c
	plugin.S
```

### 1. 添加默认的配置

`./configs/mx6ull_14x14_evk_defconfig`找到目标文件并进行复制

通常该默认配置中会包含**目标芯片与目标板**的CONFIG

```
CONFIG_SYS_EXTRA_OPTIONS="IMX_CONFIG=board/freescale/mx6ullevk/imximage.cfg"
CONFIG_ARM=y
CONFIG_ARCH_MX6=y	<---目标芯片
CONFIG_TARGET_MX6ULL_14X14_EVK=y		<---目标板
CONFIG_CMD_GPIO=y
```

那是如何找到对应板子的资源呢，包含头文件以及其他驱动文件呢，

### 2. 在芯片的kconfig列表中添加目标板索引

搜索`TARGET_MX6ULL_14x14_EVK`可以找到在`arch/arm/cpu/armv7/mx6/Kconfig`

这个文件中就包含了mx6所包含的所有的板级资源

```
if ARCH_MX6			<---在defconfig中配置了

config MX6UL
	select SYS_L2CACHE_OFF
	select ROM_UNIFIED_SECTIONS
	bool

config MX6ULL
	bool
	select MX6UL
	
config TARGET_MX6ULL_14X14_EVK			<---板级资源目标
	bool "Support mx6ull_14x14_evk"
	select MX6ULL
	select DM
	select DM_THERMAL
    
source "board/freescale/mx6ullevk/Kconfig"	<---板级资源目标，这个文件里有SYS_BOARD、SYS_VENDOR、SYS_CONFIG_NAME重要定义

endif
```

### 3. 复制目标板头文件

`include/configs/mx6ullevk.h`

在这个文件当中

### 4. 复制目标板文件夹