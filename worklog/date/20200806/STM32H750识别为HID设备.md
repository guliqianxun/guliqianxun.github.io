# STM32H750识别为HID设备

想让设备被识别成HID，就需要了解其中的交互方式和通信协议。HID是一种USB通信协议，无需安装驱动就能进行交互。

# 了解HID

## 导言-USB设备描述符

当插入USB设备后，主机会向设备请求各种描述符来识别设备。那什么是设备描述符呢？

Descriptor即描述符，是一个完整的数据结构，可以通过C语言等编程实现，并存储在USB设备中，用于描述一个USB设备的所有属性，USB主机是通过一系列命令来要求设备发送这些信息的。

描述符的作用就是通过命令操作作来给主机传递信息，从而让主机知道设备具有什么功能、属于哪一类设备、要占用多少带宽、使用哪类传输方式及数据量的大小，只有主机确定了这些信息之后，设备才能真正开始工作。

USB有5种标准描述符：

- 设备描述符
- 配置描述符
- 字符描述符
- 接口描述符
- 端点描述符

描述符之间有一定的关系，一个设备只有一个设备描述符，而一个设备描述符可以包含多个配置描述符，而一个配置描述符可以包含多个接口描述符，一个接口使用了几个端点，就有几个端点描述符。

由此我们可以看出，USB的描述符之间的关系是一层一层的，最上一层是设备描述符，下面是配置描述符，再下面是接口描述符，再下面是端点描述符。

在获取描述符时，先获取设备描述符，然后再获取配置描述符，根据配置描述符中的配置集合长度，一次将配置描述符、接口描述符、端点描述符一起一次读回。其中可能还会有获取设备序列号，厂商字符串，产品字符串等。

```c
/*设备描述符*/
struct _DEVICE_DEscriptOR_STRUCT
{
  BYTE   bLength;    //设备描述符的字节数大小
  BYTE   bDescriptorType;   //描述符类型编号，为0x01
  WORD  bcdUSB;   //USB版本号
  BYTE  bDeviceClass;   //USB分配的设备类代码，0x01~0xfe为标准设备类，0xff为厂商自定义类型，0x00不是在设备描述符中定义的，如HID
  BYTE   bDeviceSubClass;  //usb分配的子类代码，同上，值由USB规定和分配的，HID设备此值为0
  BYTE  bDeviceProtocl;  //USB分配的设备协议代码，同上HID设备此值为0
  BYTE   bMaxPacketSize0;  //端点0的最大包的大小
  WORD   idVendor;  //厂商编号
  WORD   idProduct;  //产品编号
  WORD  bcdDevice;  //设备出厂编号
  BYTE   iManufacturer;   //描述厂商字符串的索引
  BYTE   iProduct;   //描述产品字符串的索引
  BYTE   iSerialNumber;  //描述设备序列号字符串的索引
  BYTE   bNumConfiguration;   //可能的配置数量
}
```

```c
/*配置描述符*/
struct _CONFIGURATION_DEscriptOR_STRUCT
{
  BYTE  bLength;   //配置描述符的字节数大小
  BYTE   bDescriptorType;   //描述符类型编号，为0x02
  WORD   wTotalLength;   //配置所返回的所有数量的大小
  BYTE   bNumInterface;  //此配置所支持的接口数量
  BYTE   bConfigurationVale;   //Set_Configuration命令需要的参数值
  BYTE   iConfiguration;  //描述该配置的字符串的索引值
  BYTE  bmAttribute;  //供电模式的选择
  BYTE   MaxPower;   //设备从总线提取的最大电流

}
```

```c
/*字符描述符*/
struct _STRING_DEscriptOR_STRUCT

{
    BYTE bLength; //字符串描述符的字节数大小
    BYTE bDescriptorType; //描述符类型编号，为0x03
    BYTE SomeDescriptor[36]; //UNICODE编码的字符串
}
```

```c
/*接口描述符*/
struct _INTERFACE_DEscriptOR_STRUCT

{

    BYTE bLength; //接口描述符的字节数大小
    
    BYTE bDescriptorType; //描述符类型编号，为0x04
    
    BYTE bInterfaceNunber; //接口的编号
    
    BYTE bAlternateSetting;//备用的接口描述符编号
    
    BYTE bNumEndpoints; //该接口使用端点数，不包括端点0
    
    BYTE bInterfaceClass; //接口类型 HID设备此值为0x03
    
    BYTE bInterfaceSubClass;//接口子类型 HID设备此值为0或者1
    
    BYTE bInterfaceProtocol;//接口所遵循的协议
    
    BYTE iInterface; //描述该接口的字符串索引值

}
```

```c
/*端点描述符*/
struct _ENDPOIN_DEscriptOR_STRUCT

{

    BYTE bLength; //端点描述符的字节数大小
    
    BYTE bDescriptorType; //描述符类型编号，为0x05
    
    BYTE bEndpointAddress; //端点地址及输入输出属性
    
    BYTE bmAttribute; //端点的传输类型属性
    
    WORD wMaxPacketSize; //端点收、发的最大包的大小
    
    BYTE bInterval; //主机查询端点的时间间隔

}
```

### HID设备描述符

当插入USB设备后，主机会向设备请求各种描述符来识别设备。
为了把一个设备识别为HID类别，设备在定义描述符的时候必须遵守HID规范

![](Resource\接口描述符.jpg)

从框图中，可以看出除了USB标准定义的一些描述符外，HID设备还必须定义HID描述符。另外设备和主机的通信是通过报告的形式来实现的，所以还必须定义报告描述符；而物理描述符不是必需的。还有就是HID描述符是关联于接口（而不是端点）的，所以设备不需要为每个端点都提供一个HID描述符。

- 设备描述符中：**bDeviceClass, bDeviceSubClass, bDeviceProtocol三个值必须为0**

- **接口描述符中bInterfaceClass的值必须为0x03，bInterfaceSubClass的值为0或1**，为1表示HID设备符是一个启动设备（Boot Device，一般对PC机而言才有意义，意思是BIOS启动时能识别并使用您的HID设备，且只有标准鼠标或键盘类设备才能成为Boot Device。如果为0则只有在操作系统启动后才能识别并使用您的HID设备）。

- **USB HID类描述符的结构**

  | 偏移量 | 域                | 大小 | 值                                                         |                             描述                             |
  | :----- | :---------------- | :--- | :--------------------------------------------------------- | :----------------------------------------------------------: |
  | 0      | bLength           | 1    | 数字                                                       |                此描述符的长度（以字节为单位）                |
  | 1      | bDescriptorType   | 1    | 常量                                                       |            描述符种类（此处为0x21即HID类描述符）             |
  | 2      | bcdHID            | 2    | 数字                                                       | HID规范版本号（BCD码），采用4个16进制的BCD格式编码，如版本1.0的BCD码为0x0100,版本为1.1的BCD码为0x0110 |
  | 4      | bCountryCode      | 1    | 数字                                                       |            硬件目的国家的识别码（BCD码）（见表3）            |
  | 5      | bNumDescritors    | 1    | 数字                                                       |                     支持的附属描述符数目                     |
  | 6      | bDescriptorType   | 1    | 常量                                                       | HID相关描述符的类型 0x21：HID描述符 0x22：报告描述符 0x23：物理描述符 |
  | 7      | wDescriptorLength | 2    | 数字                                                       |                       报告描述符总长度                       |
  | 9      | bDescriptorType   | 1    | 常量用于识别描述符类型的常量，使用在有一个以上描述符的设备 |                                                              |
  | 10     | wDescriptorLength | 2    | 数字                                                       |          描述符总长度，使用在有一个以上描述符的设备          |

### **报告描述符**

报告描述符比较复杂，它是以item形式排列组合而成，无固定长途，用户可以自定义长度以及每一bit的含义。item类型分三种：main，global和local，其中main类型又可分为5种tag：

- input item tag：指的是从设备的一个或多个类似控制管道得到的数据
- output item tag：指的是发送给一个或多个类似控制管道的数据
- feature item tag：表示设备的输入输出不面向最终用户
- collection item tag：一个有意义的input，output和feature的组合项目
- end collection item tag：指定一个collectionitem的终止

每一个main item tag（input，output，feature）都表明了来自一个特定管道的数据的大小，数据相对还是独立，以及其他相关信息。在此之前，global和local item定义了数据的最大值和最小值，等等。local item仅仅描述下一个main item定义的数据域，而global item是这一个报告描述符中所有后续数据段的默认属性。

一个报告描述符可能包含多个main item，为了准确描述来自一个控制管道的数据，一个报告描述符必须包括以下内容：

- input（output，feature）
- usage
- usage page
- Logical Minimum
- Logical Maximum
- Report Size
- Report Count

```c
Usage Page (Generic Desktop);    //global item

Usage (Mouse);    //global item 
Collection (Application);    //Start Mouse collection
Usage (Pointer);    //
Collection (Physical);    //Start Pointer collection
Usage Page (Buttons)
Usage Minimum (1),
Usage Maximum (3),
Logical Minimum (0),
Logical Maximum (1) ;   //Fields return values from 0 to 1
Report Count (3),
Report Size (1);   //Create three 1 bit fields (button 1, 2, & 3)
Input (Data, Variable, Absolute);   //Add fields to the input report.
Report Count (1),
Report Size (5);   //Create 5 bit constant field
Input (Constant), ;Add field to the input report
Usage Page (Generic Desktop),
Usage (X),
Usage (Y),
Logical Minimum (-127),
Logical Maximum (127);    //Fields return values from -127 to 127
Report Size (8),
Report Count (2);    //Create two 8 bit fields (X & Y position)
Input (Data, Variable, Relative);   //Add fields to the input report
End Collection;   //Close Pointer collection
End Collection;   //Close Mouse collection
```

item的数据格式有两种，分别是短item和长item。

短item格式

![](Resource\Item数据格式.gif)

| bSize | 0：0个字节 1：1个字节 2：2个字节 3：4个字节                  |
| :---- | :----------------------------------------------------------- |
| bType | 0：main 1：global 2：local 3：保留                           |
| bTag  | item类型 8：input 9：output A：collection B：feature C：end collection |

长item，其bType位值为3，bTag值为F

![](Resource\长item.gif)

| bDataSize    | 0：0个字节 1：1个字节 2：2个字节 3：4个字节     |
| :----------- | :---------------------------------------------- |
| bLongItemTag | bLongItemTag 0：main 1：global 2：local 3：保留 |
| data         | 数据                                            |

### **物理描述符**

物理描述符被用来描述设备的行为特性，物理描述符是可选的，HID设备可以根据其本体的设备特性选择是否包含物理描述符。下表是HID的物理描述符结构。

**HID物理描述符的结构**

物理描述符用来描述行为特性，是可选的。

| 偏移量 | 域          | 大小 | 描述                                      |
| :----- | :---------- | :--- | :---------------------------------------- |
| 0      | bDesignator | 1    | 用来指定本体的哪一部分影响项目            |
| 1      | bFlags      | 1    | 位指定标志 位0~4：Effort 位5~7：Qualifier |

**bDesignator取值含义表**

| 取值 | 含义   | 取值      | 含义   |
| :--- | :----- | :-------- | :----- |
| 0x00 | 无     | 0x15      | 小指   |
| 0x01 | 手     | 0x16      | 头     |
| 0x02 | 眼球   | 0x17      | 肩     |
| 0x03 | 眉     | 0x18      | 腰骨   |
| 0x04 | 眼皮   | 0x19      | 腰     |
| 0x05 | 耳     | 0x1A      | 大腿   |
| 0x06 | 鼻     | 0x1B      | 膝盖   |
| 0x07 | 嘴     | 0x1C      | 小腿   |
| 0x08 | 上唇   | 0x1D      | 足     |
| 0x09 | 下唇   | 0x1E      | 脚     |
| 0x0A | 颚     | 0x1F      | 脚跟   |
| 0x0B | 颈     | 0x20      | 拇指   |
| 0x0C | 上臂   | 0x21      | 大拇指 |
| 0x0D | 手肘   | 0x22      | 第二指 |
| 0x0E | 前臂   | 0x23      | 第三指 |
| 0x0F | 手腕   | 0x24      | 第四指 |
| 0x10 | 手掌   | 0x25      | 小拇指 |
| 0x11 | 拇指   | 0x26      | 眉     |
| 0x12 | 食指   | 0x27      | 脸     |
| 0x13 | 中指   | 0x28~0xFF | 保留   |
| 0x14 | 无名指 | -         | -      |

**Qualifier取值含义**

| 取值 | 含义 | 取值 | 含义     |
| :--- | :--- | :--- | :------- |
| 0x00 | 无   | 0x04 | 其中之一 |
| 0x01 | 右   | 0x05 | 中间     |
| 0x02 | 左   | 0x06 | 保留     |
| 0x03 | 同时 | 0x07 | 保留     |



## USB HID类可采用的通信管道

所有的HID设备通过USB的控制管道（默认管道，即端点0）和中断管道与主机通信。

控制管道主要用于以下3个方面：

- 接收/响应USB主机的控制请示及相关的类数据
- 在USB主机查询时传输数据（如响应Get_Report请求等）
- 接收USB主机的数据

中断管道主要用于以下两个方面：

- USB主机接收USB设备的异步传输数据
- USB主机发送有实时性要求的数据给USB设备
- 从USB主机到USB设备的中断输出数据传输是可选的，当不支持中断输出数据传输时，USB主机通过控制管道将数据传输给USB设备。

表1、USB HID规范定义的HID设备可用端点

| 管道        | 要求 | 说明                                            |
| :---------- | :--- | :---------------------------------------------- |
| 控制(端点0) | 必须 | 传输USB描述符、类请求代码以及供查询的消息数据等 |
| 中断输入    | 必须 | 传输从设备到主机的输入数据                      |
| 中断输出    | 可选 | 传输从主机到设备的输出数据                      |



## HID设备6种特定请求

HID类请求（命令）包格式

| 偏移量 | 域            | 大小 | 说明                                                         |
| :----- | :------------ | :--- | :----------------------------------------------------------- |
| 0      | bmRequestType | 1    | HID设备类请求特性如下： 位7： 0＝从USB HOST到USB设备 1＝从USB设备到USB HOST 位6~5： 01＝请求类型为设备类请求 位4~0： 0001＝请求对象为接口（interface） 因而，针对HID的设备类请求，仅仅10100001和00100001有效 |
| 1      | bRequest      | 1    | HID类请求（参考下表）                                        |
| 2      | wValue        | 2    | 高字节说明描述符的类型 0x21：HID描述符 0x22：报告描述符 0x23：物理描述符 低字节为非0值时被用来选定实体描述符。 |
| 4      | wIndex        | 2    | 2字节数值，根据不同的bRequest有不同的意义                    |

HID类请求

| 数值 | HID类请求描述符 | 注释                                                         |
| :--- | :-------------- | :----------------------------------------------------------- |
| 0x01 | GET_REPORT      | 主机用控制传输从设备接收数据，所有HID类设备都要支持这个请求； |
| 0x02 | GET_IDLE        | 主机读取设备当前的空闲速率，设备可以不支持此请求；           |
| 0x03 | GET_PROTOCOL    | 仅仅适应于支持启动功能的HID设备（Boot Device）               |
| 0x09 | SET_REPORT      | 设备用控制传输接收主机的数据，设备可以不支持此请求；         |
| 0x0A | SET_IDLE        | 设置闲置状态，设备可不支持此请求；                           |
| 0x0B | SET_PROTOCOL    | 仅仅适应于支持启动功能的HID设备（Boot Device）               |

GET_REPORT：主机通过控制端点获取一个Report

| 域            | 值                                                           | 描述 |
| :------------ | :----------------------------------------------------------- | :--- |
| bmRequestType | 0xA1                                                         |      |
| bRequest      | 0x01                                                         |      |
| wValue        | 高字节表示报告类型 0x01:input 0x02:output 0x03:feature other:reserved 低字节表示ReportID，如不使用设为0 |      |
| wIndex        | HID的interface索引值                                         |      |
| wLength       | Report长度                                                   |      |
| Data          | Report内容                                                   |      |

SET_REPORT：主机发送一个Report给设备，用以设置input，output或者feature

| 域            | 值                                                           | 描述 |
| :------------ | :----------------------------------------------------------- | :--- |
| bmRequestType | 0x21                                                         |      |
| bRequest      | 0x09                                                         |      |
| wValue        | 高字节表示报告类型 0x01:input 0x02:output 0x03:feature other:reserved 低字节表示ReportID，如不使用设为0 |      |
| wIndex        | HID的interface索引值                                         |      |
| wLength       | Report长度                                                   |      |
| Data          | Report内容                                                   |      |

GET_IDLE

| 域            | 值                                        | 描述 |
| :------------ | :---------------------------------------- | :--- |
| bmRequestType | 0xA1                                      |      |
| bRequest      | 0x02                                      |      |
| wValue        | 高字节0 低字节表示ReportID，如不使用设为0 |      |
| wIndex        | HID的interface索引值                      |      |
| wLength       | 1                                         |      |
| Data          | 空闲速率                                  |      |

SET_IDLE

| 域            | 值                                         | 描述 |
| :------------ | :----------------------------------------- | :--- |
| bmRequestType | 0x21                                       |      |
| bRequest      | 0x0A                                       |      |
| wValue        | 新的速率 低字节表示ReportID，如不使用设为0 |      |
| wIndex        | HID的interface索引值                       |      |
| wLength       | 0                                          |      |
| Data          | 无                                         |      |

GET_PROTOCOL

| 域            | 值                                    | 描述 |
| :------------ | :------------------------------------ | :--- |
| bmRequestType | 0xA1                                  |      |
| bRequest      | 0x03                                  |      |
| wValue        | 0                                     |      |
| wIndex        | HID的interface索引值                  |      |
| wLength       | 1                                     |      |
| Data          | 0 = Boot Protocol 1 = Report Protocol |      |

SET_PROTOCOL

| 域            | 值                                    | 描述 |
| :------------ | :------------------------------------ | :--- |
| bmRequestType | 0x21                                  |      |
| bRequest      | 0x0B                                  |      |
| wValue        | 0 = Boot Protocol 1 = Report Protocol |      |
| wIndex        | HID的interface索引值                  |      |
| wLength       | 0                                     |      |
| Data          | 无                                    |      |



## HID的特点

- 交换的数据存储在报告的结构内，设备必须支持HID报告格式。
- 每笔事务可以携带小量或中量的数据。低速设备每笔事务最大为8字节，全速设备每笔最大为64字节，高速设备最大为1 024字节；
- 有最大传输速度的限制。低速设备最快10 ms一笔事务，最高速度为800 B/s；全速设备最快1 ms一笔事务，最高速度为64 KB/s；高速设备最快125 μs一笔事务，最高速度为24.576 MB/s。
- 没有传输速度的保证。

 USB设备有4种传输方式与主机进行通信： 控制方式、中断方式、批量方式和同步方式。每种方式都有它的应用领域。HID只支持控制和中断传输方式。如图2所示，HID设备必须要有默认的控制管道和一个中断输入端点；中断输出端点是可选的。

<img src="Resource\HID传输.jpg" style="zoom:150%;" />

中断输出传输是USB1.1规范才有的内容，且必须获得Windows系统的支持。从Windows98 SE版本开始才支持中断输出传输方式，所以如果需要中断输出传输方式的设备应该选择相应的操作系统。下表列出了传输类型和相关情况。

![](Resource\传输类型.jpg)

 USB协议定义了11种请求命令，通过这些请求来获得设备的信息及对设备进行设置。HID类设备除了要支持这11种标准的请求外，还要实现以下6种特定请求：

- Get_Report——主机用控制传输从设备接收数据，所有HID类设备都要支持这个请求；
- Set_Report——设备用控制传输接收主机的数据，设备可以不支持此请求；
- Get_Idle——主机读取设备当前的空闲速率，设备可以不支持此请求；
- Set_Idle——设置闲置状态，设备可不支持此请求；
- Get_Protocol——主机获得设备的当前活动是引导协议还是报告协议；
- Set_Protocol——在引导协议和报告协议间切换，设备如果支持系统引导（如键盘和鼠标），就必须支持Get_Protocol和Set_Protocol请求。





