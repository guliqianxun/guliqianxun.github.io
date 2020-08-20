# MCU输出时钟24M

讲实话，本来以为这是一个很简单的问题，但是发现经过MX设置之后，并没有直接输出时钟。特地开了一个模块来解决这个问题。

首先简单介绍一下RCC的时钟模块：

![](Resource\RCCBlockDiagram.png)

这里可以看到系统提供了两个时钟的输出引脚MCO1和MCO2

![](Resource\PinofMCO2Description.png)

看一下MCO的时钟走向和来源：

![](Resource\Top_levelClockTree.png)

![](Resource\MCO1andMCO2.png)

MCO时钟的来源有很多选项，但这里有些限定条件：

| MCO1      | MHz      | MCO2      | MHz      |
| --------- | -------- | --------- | -------- |
| hsi_ck    | 64       | sys_ck    | 480      |
| lse_ck    | no use   | pll2_p_ck | flexible |
| hse_ck    | 25       | hse_ck    | 25       |
| pll1_q_ck | flexible | pll1_p_ck | 480      |
| hsi48_ck  | 48       | csi_ck    | 4        |
|           |          | lsi_ck    | no use   |

这里的LSE和LSI是低速时钟，在Khz级别，所以不适用。

结合条件一看的话，因为选用了MCO2作为输出端口，那么只有选择调整pll2_p_ck这一个方式去给出24M时钟了。

![](Resource\MXclock.png)

这里的PLL2P输出到MCO2时，MCO自己还可以选择 1~15 分频，这里直接设置为24M就不需要分频啦。

## 问题：MCO2没有直接输出时钟

按理来说这样设置好之后，MCO2就能看到时钟的出现了，是并没有啊！！

**测试**:

1. 直接输入HSE，有波形
2. 启用MCO1，使用PLL1Q，有波形
3. 启用SPI1，master，使用时钟为PLL2输出，MCO2有波形

那么基本确定，问题出在PLL2没有使能上

## 朔源：

```c
HAL_RCC_MCOConfig(RCC_MCO2, RCC_MCO2SOURCE_PLL2PCLK, RCC_MCODIV_1);

HAL_RCCEx_PeriphCLKConfig(&PeriphClkInitStruct)
```

问题的关键出现在这两个函数上。

PLL的使能是在启动外设时使能的，如果没有外设，那么只有PLL1作为系统时钟的源头被使能，而PLL2没有被使能，所以尽管MCOConfig函数设置了正确的参数，但是并没有时钟出现。

```c
#include "stm32h7xx_hal_rcc_ex.h"

__HAL_RCC_PLL2_CONFIG(PeriphClkInitStruct.PLL2.PLL2M,
                          PeriphClkInitStruct.PLL2.PLL2N,
                          PeriphClkInitStruct.PLL2.PLL2P,
                          PeriphClkInitStruct.PLL2.PLL2Q,
                          PeriphClkInitStruct.PLL2.PLL2R);
	    /* Select PLL2 input reference frequency range: VCI */
    __HAL_RCC_PLL2_VCIRANGE(PeriphClkInitStruct.PLL2.PLL2RGE) ;

    /* Select PLL2 output frequency range : VCO */
    __HAL_RCC_PLL2_VCORANGE(PeriphClkInitStruct.PLL2.PLL2VCOSEL) ;

    /* Disable PLL2FRACN . */
    __HAL_RCC_PLL2FRACN_DISABLE();

    /* Configures PLL2 clock Fractional Part Of The Multiplication Factor */
    __HAL_RCC_PLL2FRACN_CONFIG(PeriphClkInitStruct.PLL2.PLL2FRACN);

    /* Enable PLL2FRACN . */
    __HAL_RCC_PLL2FRACN_ENABLE();
	    /* Enable the PLL2 clock output */
     __HAL_RCC_PLL2CLKOUT_ENABLE(RCC_PLL2_DIVP);
    /* Enable  PLL2. */
    __HAL_RCC_PLL2_ENABLE();
		HAL_Delay(100);
```

在main.c中添加头文件，并在SystemClock_Config()函数中，添加如上代码，人工使能PLL2P，就可以看到时钟的输出了。