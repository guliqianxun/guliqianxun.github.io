# STM32H750环境下，搭建带独立DMA的SPI通信

## 一、目标分析

首先分析SPI和DMA有哪些特点，在什么情况下工作。

任务要求：实现5个独立SPI收发，每个SPI有独立的DMA。

#### SPI：

SPI是串行外设接口（Serial Peripheral Interface）的缩写。关键词：**全双工，同步通信**

引脚接口如下：（MOSI：master in slave out, SS:slave select[也称CS])

![引脚接口](Recourse\SPI物理连接.png)

note：CS用作片选，因此，当只有一个从机时，可以不用这个引脚

## 二、系统实现

*操作环境：STM32CubeMX，keil5，芯片：STM32H750VB*

```c
/***SPI配置主机***/
  hspi1.Instance = SPI1;
  hspi1.Init.Mode = SPI_MODE_MASTER;
  hspi1.Init.Direction = SPI_DIRECTION_2LINES;
  hspi1.Init.DataSize = SPI_DATASIZE_8BIT;
  hspi1.Init.CLKPolarity = SPI_POLARITY_LOW;
  hspi1.Init.CLKPhase = SPI_PHASE_1EDGE;
  hspi1.Init.NSS = SPI_NSS_SOFT;
  hspi1.Init.BaudRatePrescaler = SPI_BAUDRATEPRESCALER_2;
  hspi1.Init.FirstBit = SPI_FIRSTBIT_MSB;
  hspi1.Init.TIMode = SPI_TIMODE_DISABLE;
  hspi1.Init.CRCCalculation = SPI_CRCCALCULATION_DISABLE;
  hspi1.Init.CRCPolynomial = 0x0;
  hspi1.Init.NSSPMode = SPI_NSS_PULSE_ENABLE;
  hspi1.Init.NSSPolarity = SPI_NSS_POLARITY_LOW;
  hspi1.Init.FifoThreshold = SPI_FIFO_THRESHOLD_08DATA;
  hspi1.Init.TxCRCInitializationPattern = SPI_CRC_INITIALIZATION_ALL_ZERO_PATTERN;
  hspi1.Init.RxCRCInitializationPattern = SPI_CRC_INITIALIZATION_ALL_ZERO_PATTERN;
  hspi1.Init.MasterSSIdleness = SPI_MASTER_SS_IDLENESS_00CYCLE;
  hspi1.Init.MasterInterDataIdleness = SPI_MASTER_INTERDATA_IDLENESS_00CYCLE;
  hspi1.Init.MasterReceiverAutoSusp = SPI_MASTER_RX_AUTOSUSP_DISABLE;
  hspi1.Init.MasterKeepIOState = SPI_MASTER_KEEP_IO_STATE_ENABLE;
  hspi1.Init.IOSwap = SPI_IO_SWAP_DISABLE;
  if (HAL_SPI_Init(&hspi1) != HAL_OK)
  {
    Error_Handler();
  }
/***SPI配置从机***/
  hspi2.Instance = SPI2;
  hspi2.Init.Mode = SPI_MODE_SLAVE;
  hspi2.Init.Direction = SPI_DIRECTION_2LINES;
  hspi2.Init.DataSize = SPI_DATASIZE_8BIT;
  hspi2.Init.CLKPolarity = SPI_POLARITY_LOW;
  hspi2.Init.CLKPhase = SPI_PHASE_1EDGE;
  hspi2.Init.NSS = SPI_NSS_SOFT;
  hspi2.Init.FirstBit = SPI_FIRSTBIT_MSB;
  hspi2.Init.TIMode = SPI_TIMODE_DISABLE;
  hspi2.Init.CRCCalculation = SPI_CRCCALCULATION_DISABLE;
  hspi2.Init.CRCPolynomial = 0x0;
  hspi2.Init.NSSPMode = SPI_NSS_PULSE_DISABLE;
  hspi2.Init.NSSPolarity = SPI_NSS_POLARITY_LOW;
  hspi2.Init.FifoThreshold = SPI_FIFO_THRESHOLD_08DATA;
  hspi2.Init.TxCRCInitializationPattern = SPI_CRC_INITIALIZATION_ALL_ZERO_PATTERN;
  hspi2.Init.RxCRCInitializationPattern = SPI_CRC_INITIALIZATION_ALL_ZERO_PATTERN;
  hspi2.Init.MasterSSIdleness = SPI_MASTER_SS_IDLENESS_00CYCLE;
  hspi2.Init.MasterInterDataIdleness = SPI_MASTER_INTERDATA_IDLENESS_00CYCLE;
  hspi2.Init.MasterReceiverAutoSusp = SPI_MASTER_RX_AUTOSUSP_DISABLE;
  hspi2.Init.MasterKeepIOState = SPI_MASTER_KEEP_IO_STATE_ENABLE;
  hspi2.Init.IOSwap = SPI_IO_SWAP_DISABLE;
  if (HAL_SPI_Init(&hspi2) != HAL_OK)
  {
    Error_Handler();
  }
/***SPI主机DMA配置，和引脚复用***/
    __HAL_RCC_SPI1_CLK_ENABLE();

    __HAL_RCC_GPIOB_CLK_ENABLE();
    /**SPI1 GPIO Configuration
    PB3 (JTDO/TRACESWO)     ------> SPI1_SCK
    PB4 (NJTRST)     ------> SPI1_MISO
    PB5     ------> SPI1_MOSI
    */
    GPIO_InitStruct.Pin = GPIO_PIN_4|GPIO_PIN_5;
    GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_VERY_HIGH;
    GPIO_InitStruct.Alternate = GPIO_AF5_SPI1;
    HAL_GPIO_Init(GPIOB, &GPIO_InitStruct);
		
		GPIO_InitStruct.Pin = GPIO_PIN_3;
    GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;
    GPIO_InitStruct.Alternate = GPIO_AF5_SPI1;
    HAL_GPIO_Init(GPIOB, &GPIO_InitStruct);

    /* SPI1 DMA Init */
    /* SPI1_RX Init */
    hdma_spi1_rx.Instance = DMA1_Stream4;
    hdma_spi1_rx.Init.Request = DMA_REQUEST_SPI1_RX;
    hdma_spi1_rx.Init.Direction = DMA_PERIPH_TO_MEMORY;
    hdma_spi1_rx.Init.PeriphInc = DMA_PINC_DISABLE;
    hdma_spi1_rx.Init.MemInc = DMA_MINC_ENABLE;
    hdma_spi1_rx.Init.PeriphDataAlignment = DMA_PDATAALIGN_BYTE;
    hdma_spi1_rx.Init.MemDataAlignment = DMA_MDATAALIGN_BYTE;
    hdma_spi1_rx.Init.Mode = DMA_NORMAL;
    hdma_spi1_rx.Init.Priority = DMA_PRIORITY_LOW;
    hdma_spi1_rx.Init.FIFOMode = DMA_FIFOMODE_DISABLE;
    if (HAL_DMA_Init(&hdma_spi1_rx) != HAL_OK)
    {
      Error_Handler();
    }

    __HAL_LINKDMA(spiHandle,hdmarx,hdma_spi1_rx);

    /* SPI1_TX Init */
    hdma_spi1_tx.Instance = DMA1_Stream5;
    hdma_spi1_tx.Init.Request = DMA_REQUEST_SPI1_TX;
    hdma_spi1_tx.Init.Direction = DMA_MEMORY_TO_PERIPH;
    hdma_spi1_tx.Init.PeriphInc = DMA_PINC_DISABLE;
    hdma_spi1_tx.Init.MemInc = DMA_MINC_ENABLE;
    hdma_spi1_tx.Init.PeriphDataAlignment = DMA_PDATAALIGN_BYTE;
    hdma_spi1_tx.Init.MemDataAlignment = DMA_PDATAALIGN_BYTE;
    hdma_spi1_tx.Init.Mode = DMA_NORMAL;
    hdma_spi1_tx.Init.Priority = DMA_PRIORITY_LOW;
    hdma_spi1_tx.Init.FIFOMode = DMA_FIFOMODE_DISABLE;
    if (HAL_DMA_Init(&hdma_spi1_tx) != HAL_OK)
    {
      Error_Handler();
    }

    __HAL_LINKDMA(spiHandle,hdmatx,hdma_spi1_tx);

    /* SPI1 interrupt Init */
    HAL_NVIC_SetPriority(SPI1_IRQn, 0, 0);
    HAL_NVIC_EnableIRQ(SPI1_IRQn);
```



## 三、错误排查

### 20200805：

#### 现象：

SPI带DMA在48MHz时钟下，主机接受到的数据异常，从机接受到的数据正常。

在24M时钟下，主从机通信没有问题

SPI48M异常：

![SPI48M异常](Recourse\SPI48M异常.png)

主机接受的数据因为1~100（10进制，图片显示的为16进制），现在显示为00开始间隔增加一个，开头的00数据有疑惑，间隔原因也要解决。

SPI24M正常：

![SPI24M正常](Recourse\SPI24M正常.png)

#### 排查过程：

为了查看是发送的SPI问题还是DMA的存取问题，将DMA暂时不用。

```c
/*原来发送的带DMA的SPI程序，SPI1为主机，SPI2为从机*/
HAL_SPI_TransmitReceive_DMA(&hspi2, (uint8_t*)aTxBuffer, (uint8_t *)aRxBuffer, (uint16_t)TEST_BUFFER_SIZE);
HAL_Delay(100);	
HAL_SPI_TransmitReceive_DMA(&hspi1, (uint8_t*)aTxBuffer2, (uint8_t *)aRxBuffer2, (uint16_t)TEST_BUFFER_SIZE);
HAL_Delay(100);	
```

因为是一个芯片，在主函数while循环中只操控一个SPI，这里把从机在主循环外带DMA打开，主机在while循环中发送数据

```c
/*将主机函数去掉DMA，转移到主循环中，查看数据现象*/
HAL_SPI_TransmitReceive(&hspi1, (uint8_t*)aTxBuffer2, (uint8_t *)aRxBuffer2, (uint16_t)TEST_BUFFER_SIZE,100);
HAL_Delay(100);	
```

![SPI1去除DMA48M](Recourse\SPI1去除DMA48M.png)

可以看到，数据依然有问题，且现象一致。想法：问题可能主要出在SPI从机的发送和主机的接收上。



将收发函数，主机改为接收，从机改为发送

```c
/*从机*/
HAL_SPI_Transmit_DMA(&hspi2, (uint8_t*)aTxBuffer, (uint16_t)TEST_BUFFER_SIZE);
HAL_Delay(100);	
/*主机，在while循环中*/
HAL_SPI_Receive(&hspi1, (uint8_t *)aRxBuffer2, (uint16_t)TEST_BUFFER_SIZE,100);
```

![主收从发](Recourse\主收从发.png)

##### 问题锁定

想法：是SPI的全双工模式出了问题？硬件电路不稳定导致的接收问题？或者是数据位设置错误，导致的间隔取数（或者过采样？）？系统时钟设置问题，导致取数不正确？

###### 方案一：改变SPI的工作模式

将主机改为仅收，从机改为仅发

```c
hspi1.Init.Direction = SPI_DIRECTION_2LINES_RXONLY;
hspi2.Init.Direction = SPI_DIRECTION_2LINES_TXONLY;
```

现象与上述保持一致，问题排除。

###### 方案二：系统时钟架构，DMA传输慢于SPI

![](Recourse\DMA系统架构.png)

![](Recourse\SPI145构架.png)

![](Recourse\SPI23构架.png)

![](Recourse\SPI6构架.png)

在时间架构上，这里的100M和200M的最高频率是针对xB系列的，对于目前使用的STM32H750VB，最高可以达到240和120M。

![](Recourse\SPI动态特性.png)

**在datesheet中查看引脚电气特性，发现作为全双工的从机时，发送最高是31M，而发送可以到达150M，这是从机接受数组正确，而主机接受数组错误的原因**

#### 总结：

这个问题困扰我接近一周之久，先是怀疑自己的程序问题，后来又查看是不是电路原因，之后又查看是否是输出问题。做芯片开发，一定要了解芯片手册，包括系统构架、电气特性，这些都是成功的程序的前提条件啊！！