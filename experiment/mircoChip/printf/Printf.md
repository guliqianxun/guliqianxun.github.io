写c嘛，肯定都喜欢用printf啦。

.c

```c
void usart_send(const char * buffer, uint32_t size)
{
	uint8_t sendtemp[size];
	for(int i=0;i<size;i++)
	{
	    // send function 
			sendtemp[i] = buffer[i];				
	}
		while(RESET == HAL_UART_GetState(&huart1)){
	}
		HAL_UART_Transmit(&huart1, sendtemp, size, 100);

}

void uart_put(uint8_t dat)
{
    while(RESET == HAL_UART_GetState(&huart1)){
    }
    /* write one byte in the USART0 data register */
    HAL_UART_Transmit(&huart1, &dat, 1, 100);
}
// put char
int fputc(int ch, FILE *f)
{
    uart_put((uint8_t)ch);
    return ch;
}
// ????
void printf_byte(uint8_t data)
{
    printf("0x%X%X ",data >> 4,data & 0x0f);
}
// ????
void printf_array(uint8_t *buffer, int size)
{
    if(buffer==NULL)
    {
        printf("printf_array==NULL\n");
        return ;
    }
    for(int i=0;i<size;i++)
    {
        printf_byte(buffer[i]);
    }
    printf("\n");
}
```

.h

```c
#define LOGLEVEL_CRITI          0
#define LOGLEVEL_ERROR          1
#define LOGLEVEL_WARN           2
#define LOGLEVEL_INFO           3
#define LOGLEVEL_DEBUG          4
	
#define TP_LOGLEVEL   LOGLEVEL_ERROR

void usart_send(const char * buffer, uint32_t size);
void printf_byte(uint8_t data);
void printf_array(uint8_t *buffer, int size);

#if TP_LOGLEVEL >=  LOGLEVEL_DEBUG
    #define LOG_DEBUG(...) printf("%8d:DEBUG:%s ",SysTick->VAL,__func__),printf(__VA_ARGS__)
#else
    #define LOG_DEBUG(...) 
#endif
 
#if TP_LOGLEVEL >= LOGLEVEL_INFO
    #define LOG_INFO(...) printf("%8d:INFO:%s ",SysTick->VAL,__func__),printf(__VA_ARGS__)
#else
    #define LOG_INFO(...) 
#endif
 
#if TP_LOGLEVEL >=  LOGLEVEL_WARN
#define LOG_WARN(...) printf("%8d:WARN:%s ",SysTick->VAL,__func__),printf(__VA_ARGS__)
#else
    #define LOG_WARN(...) 
#endif
 
#if TP_LOGLEVEL >=  LOGLEVEL_ERROR
    #define LOG_ERROR(...) printf("%8d:ERROR:%s ",SysTick->VAL,__func__),printf(__VA_ARGS__)
#else
    #define LOG_ERROR(...) 
#endif
 
#if TP_LOGLEVEL >=  LOGLEVEL_CRITI
    #define LOG_CRITI(...) printf("%8d:CRITI:%s ",SysTick->VAL,__func__),printf(__VA_ARGS__)
#else
    #define LOG_CRITI(...) 
#endif
```



