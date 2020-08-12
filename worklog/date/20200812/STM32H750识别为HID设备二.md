# STM32H750识别为HID设备二



[TOC]

# 软件框架

使用 STM32CubeMX 和 keil5 作为软件开发环境，STM32CubeProgrammer 烧写程序， 调试工具： Bus hound

## STM32CubeMX ：

建立工程选择对应芯片，选择外部陶瓷晶振，默认25M

![](Resource\MX选择外部晶振.png)

然后打开USB功能，选择引脚为HS引脚，但使用FS模式

![](Resource\MX打开USBHS.png)

在Middleware中选择设置USB设备

![](Resource\MX选择HS为HID类.png)

设置USB设备描述符

![](Resource\MX设置设备描述符.png)

生产的芯片引脚赋用如下

![](Resource\芯片引脚选用.png)

芯片时钟，主频为480M，USB使用最高时钟48M

![](Resource\USB时钟.png)

之后设置工程属性，导出代码，就可以在keil5中编辑了



## Keil5

打开工程后，产生的代码构架如下：

![](Resource\cubeMX生成的软件框图.png)

使能一个HID设备的思路如下

![](Resource\usbHID实现步骤.png)

![](Resource\文件框架.png)

# 参考目录：

[STM32 USB初级培训.USB IP介绍](https://www.stmcu.com.cn/Designresource/design_resource_detail/file/561104/lang/ZH/token/5a1440f4a26de5d194fc3adb721387fd)