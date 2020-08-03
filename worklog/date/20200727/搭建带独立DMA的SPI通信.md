# STM32H750环境下，搭建带独立DMA的SPI通信

## 一、目标分析

首先分析SPI和DMA有哪些特点，在什么情况下工作。

任务要求：实现5个独立SPI收发，每个SPI有独立的DMA。

#### SPI：

SPI是串行外设接口（Serial Peripheral Interface）的缩写。关键词：**全双工，同步通信**

引脚接口如下：（MOSI：master in slave out, SS:slave select[也称CS])

![引脚接口](Recourse\SPI物理连接.png)

note：CS用作片选，因此，当只有一个从机时，可以不用这个引脚

