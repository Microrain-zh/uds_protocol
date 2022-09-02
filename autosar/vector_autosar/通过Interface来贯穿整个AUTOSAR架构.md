# 通过Interface来贯穿整个AUTOSAR架构

AUTOSAR里面的架构知识是非常丰富的，除了之前讲的模块，还有接口呢。如果比喻说模块是人体的各大功能组织，那么这个接口就可以理解为人体的血管神经了，其重要性可想而知了。

AUTOSAR的架构为什么要强调这个接口呢，又有什么特别的地方呢？下面且听我细细道来。



1**AUTOSAR Interface分类**



开门见山，AUTOSAR是有三种接口的，为了更好对照AUTOSAR里面的标准概念，我保留英文名称：

**1.** **AUTOSAR Interface**

**AUTOSAR Interface**定义了软件组件和/或BSW模块之间交换的信息。该描述独立于特定的编程语言，ECU或网络技术。 **AUTOSAR Interface**用于定义软件组件和/或BSW模块的端口。通过这些端口，软件组件和/或BSW模块可以彼此通信（发送或接收信息或调用服务）。AUTOSAR使得可以在本地或通过网络在SoftwareComponents和/或BSW模块之间实现这种通信。

**2. Standardized AUTOSAR Interface**

**Standardized AUTOSAR Interface**是其语法和语义在AUTOSAR中标准化的**AUTOSAR Interface**。**Standardized AUTOSAR Interface**通常用于定义AUTOSAR服务，这是AUTOSAR基本软件向应用程序软件组件提供的标准化服务。

**3. Standardized Interface**

**Standardized Interface**是一种在AUTOSAR中标准化的API，无需使用**AUTOSAR Interface**技术。这些**Standardized Interface**通常是为特定的编程语言（如C语言）定义的。因此，通常在始终位于同一ECU上的软件模块之间使用**Standardized Interface**。当软件模块通过**Standardized Interface**进行通信时，将无法再通过网络路由软件模块之间的通信。

这些文字描述有些抽象，拿个图来看看也许会更直观点：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxiciaRTcRj2IuNx9CbYB8wsorwsJpxZiaNqDvQOXBEH9mbFnlqpoCqw9FJNwJGLtehgs0v0iblYo5A2Iw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

从上图可以看出，简单地理解，**AUTOSAR Interface**多用于Application、Abstraction于Complex Driver上；**Standardized AUTOSAR Interface**多用于BSW中的Service上；而**Standardized Interface**呢，是AUTOSAR定义的BSW中的模块直接交互用的接口。



2**AUTOSAR Interface通用规则**



AUTOSAR定义了这么多个模块，他们直接的调用也定义三种不同的接口，是不是他们就可以相互调用了？有没有什么规则或限制呢？

首先，来看个图就很容易明白了：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxiciaRTcRj2IuNx9CbYB8wsorqKick82IEjpq5N8Prhn7hEA8U5RulJhRKKcMqVA3khBDl8XdlKRgxKA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

来看看这图上的箭头和颜色，还有这个绿色的圆点代表什么意思。

| ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxiciaRTcRj2IuNx9CbYB8wsor5FX8picic6NibibpNGzwMDBfIp80aU1eK3YOalWcGbw5bqHvqSj99upFpA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) | 1. 对于服务层，允许水平接口调用。例如，Error Manager使用NVRAM Manager保存故障数据2. 对于ECU抽象层，允许水平接口调用。3. 复杂的驱动程序可能会使用其他选定的BSW模 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxiciaRTcRj2IuNx9CbYB8wsorcib7mupxEv4KLbaIficVaRgH2ctO0iaoeb5ysCNfLuWJx6KWcTYPcpIwQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) | 对于µC抽象层，不允许使用水平接口调用。但是由于性能原因，允许可配置的Notification。 |
| ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxiciaRTcRj2IuNx9CbYB8wsor3FlezOpcjfrHDsrz5j1licEszhrEHQuicerS2I9a9S0WIvkia1ibpYI4jQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) | 一层可以访问下面的SW层的所有接口。                           |
| ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxiciaRTcRj2IuNx9CbYB8wsorSnRsovovhk3QLn9ib2iay0Uticwiant6gHBVvyjAM0m09v24iaCib7b2pXNg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) | 应避免绕过一个软件层。                                       |
| ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxiciaRTcRj2IuNx9CbYB8wsorHiaUlKfdytnrY9E6EjaMcicSFhHLiazYfnYgW54Zy4K0SgOzu3jbnwSvA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) | 不允许绕过两个或多个软件层； 不允许绕过µC抽象层。            |
| ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxiciaRTcRj2IuNx9CbYB8wsorKHU9fupGkv1nyVuKuic8xoPiabQZAJAFjZTJUsboE6kLicC3sMzPapydw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) | 一个模块可以访问另一个层组的较低层模块（例如，用于外部硬件的SPI）。 |
| ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxiciaRTcRj2IuNx9CbYB8wsorlzKNx8Gpv715sdU2DzbRdDvSRKfQ4dGhTmG02VXHfFrko9yHw29Fxg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) | 所有层都可以与系统服务交互。                                 |

以上内容都很好理解，但对于更详细的模块组件之间的相互之间调用关系和约束，可以通过下面这个矩阵表来说明。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxiciaRTcRj2IuNx9CbYB8wsorXYKOKUMMVEFkbYGcBlB6qPYQJ7aSV0cndbKZu5Y2bd6AFYC04eRXAw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

注：

√ 表示允许使用× 表示不允许使用△ 表示使用受限（但callback是允许的）这个矩阵表是按行读取的，例如，允许I/O驱动程序使用系统服务和硬件，但不能使用其他层。（灰色背景表示“非基本软件”层）



3**与Complex Driver层接口交互**



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxiciaRTcRj2IuNx9CbYB8wsorhiayD0URuvgpwCEtPicOEZTDNvATQBB78MOxxkwcMgBEOxKXTMEnx9zA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

复杂驱动程序可能需要与分层软件体系结构中的其他模块接口交互，或者分层软件体系结构中的模块可能需要与复杂驱动程序接口交互。在这种情况下，适用以下规则：

**1. 从分层软件体系结构的模块到复杂驱动程序的接口
**

仅当复杂驱动程序提供可以通过访问AUTOSAR模块进行一般配置的接口时，才允许这样做。

一个典型的例子是PDU路由器：复合驱动程序可以实现新总线系统的接口模块。

在PDU路由器的配置中已经解决了这一问题。

**2.从复杂驱动程序到分层软件体系结构的模块的接口**

同样，仅当分层软件体系结构的各个模块提供接口并准备由复杂驱动程序访问时，才允许这样做。通常这意味着：

- 各个接口定义为可重入。
- 如果使用Callback程序，则名称是可配置的
- 没有管理模块状态的上层模块（并行访问会更改状态，而不会被上层模块通知）

**通常，可以访问以下模块：**

- SPI驱动
- GPT驱动程序
- I/O驱动程序通常只对单独的组/通道/等存在重入限制。并行访问相同的组/通道/等，通常是不允许的。在配置过程中必须注意这一点。
- NVRAM Manager作为内存堆栈的独占访问点
- Watchdog Manager作为看门狗堆栈的独占访问点
- PDU Router作为通信堆栈的专用总线和协议无关的访问点
- 特定于总线的接口模块，作为通信堆栈的专有总线专用访问点
- NM接口模块作为网络管理堆栈的专用访问点
- COM Manager（仅来自上层）和基本软件Mode Manager作为状态管理的专用访问点
- Det，Dem和Dlt
- 只要分层的软件体系结构的模块不使用所使用的OS对象，就可以使用该OS
- 对于每个模块，仍然需要检查相应功能是否标记为可重入。例如，“ init”功能通常是不可重入的，只能由ECU State Manager调用。

**对于多核体系结构，还有其他规则：**

- BSW可以分布在多个核心上。负责执行对BSW服务的调用的核心由其BswOperationInvokedEvent的任务映射确定。

- 仅允许使用主/从实现模块内部通信跨越分区和核心边界。

- 因此，如果CDD需要访问BSW的标准化接口，则它必须位于同一核心上。

- 如果CDD驻留在不同的内核上，则可以使用常规端口机制访问AUTOSAR接口和标准化的AUTOSAR接口。这将调用RTE，后者使用操作系统的IOC机制将请求传输到另一个内核。

- 但是，如果CDD需要访问BSW的标准化接口并且不在同一核心上，

- 1. 提供标准接口的任何从端都可以在CDD所在的核心上运行，并将调用转发到另一个核心；
  2. 2. 或CDD的存根部分需要在另一个内核上实现，并且通信需要使用类似于RTE的操作系统的IOC机制在本地CDD上进行组织。

- 另外，在后一种情况下，CDD的初始化部分也需要驻留在不同内核中。

以上内容是不是看上去一头雾水（我是从AUTOSAR官方文档翻译过来的）。CDD即复杂驱动，名字取得一点都没错，哈哈哈。这个模块本身就不是一个标准的模块，而是为了那些非标准实现不了的设计而定的，也没有固定的规则，所以这里的接口限制内容就会有些繁杂。

看不懂也没关系，要用到相关内容的时候，再来对照下或许会更明了。

为了更好理解这些接口的功能和用法，以下通过实例来解说。



4**AUTOSAR 层之间交互（Memory）**



在讲解Memory之前，我们先思考下几个问题：

- 软件层到底是怎么交互的？
- 软件Interface是长啥样的？
- ECU Abstraction Layer内部是什么？
- 如何高效地实现Abstraction Layer？

现在不知道答案也没关系，我们带着这些疑问，一起探讨下Memory里面的接口交互。

以下案例是讲述NVRAM Manage和Watchdog Manager与Driver交互的情况。这个案例是基于以下假设进行的：

- ECU硬件包含有一个外部的EEPROM和一个外部的Watchdog，并通过SPI连接到MCU。
- 在SPIHandlerDriver控制下，可以同时访问SPI外设，同时设定Watchdog的优先级比EEPROM的高。
- MCU还包含一个内部的Flash，和EEPROM并列存在。EEPROM的Abstraction和Flash EEPROM Emulation有着相同语义的API。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxic8BTIeeRSiadYCiaxVm0dTJTJ7lDzFwCEozxCAL1Kw4a8ssbP3dhxu3161OByRVYySwBAxcxqkWmyw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Memory Abstraction Interface可以通过以下方式实现：

- 在运行时根据Device Index（int / ext）进行处理
- 在运行时根据Block Index进行处理（例如0x01FF 为外部EEPROM）
- 在配置期间通过具有NVRAM Manager内部功能指针的ROM表进行路由（在这种情况下，Memory Abstraction Interface仅“虚拟”存在）

在架构上，NVRAM Manager 通过Memory Abstraction Interface访问Driver. 它用设备Index来寻址不同的Memory设备.

Memory Abstraction Interface有以下接口:

```c
Std_ReturnType MemIf_Write
(
uint8 DeviceIndex,
uint16 BlockNumber,
uint8 *DataBufferPtr
)
```

EA（即EEPROM Abstraction）以及FEE（Flash EEPROM Emulation）有以下接口:

```c

Std_ReturnType Ea_Write
(
uint16 BlockNumber,
uint8 *DataBufferPtr
)
```

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx8rNyibTtFnVU9E4TibxYOxcscGre4ylCedqJJqRGOSkricnAzRYEicsCkKpUAg217IKia6Xu05zE6vciag/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

从上图看，有两种情形或者方案：

**1.** **只有一个NV device类型在使用**

现实中多数是这种情况. 在这种情况下，在源代码可用的情况下，可以将Memory Abstraction实现为忽略DeviceIndex参数的简单宏。

MemIf.h:

```
#include “Ea.h“ /* for providing access to the EEPROM Abstraction */
// ...
#define MemIf_Write(DeviceIndex, BlockNumber, DataBufferPtr) \
Ea_Write(BlockNumber, DataBufferPtr)
```

MemIf.c(不需要这个文件)

不需要其他代码，NVRAM Manager实际上可以直接访问EA或FEE。

 **2.** **有两个或更多不同类型的NV devices在使用**

在这种情况下，必须使用DeviceIndex选择正确的NVDevice。通过使用指向功能的指针数组来实现是非常有效的。以下示例仅显示写功能：

File MemIf.h:

```c

extern const WriteFctPtrType WriteFctPtr[2];
#define MemIf_Write(DeviceIndex, BlockNumber, DataBufferPtr) \
WriteFctPtr[DeviceIndex](BlockNumber, DataBufferPtr)
```

File MemIf.c:

```c
#include “Ea.h“ /* for getting the API function addresses */
#include “Fee.h“ /* for getting the API function addresses */
#include “MemIf.h“ /* for getting the WriteFctPtrType */
const WriteFctPtrType WriteFctPtr[2] = {Ea_Write, Fee_Write};
```

使用同样的代码和运行时，就好像Function Pointer Table是位于NVRAM Manager中一样。内存抽象接口不会产生任何开销。

这样，1. 抽象层可以非常有效地实现；2. 抽象层可以缩放；3. Memory Abstraction Interface简化了NVRAM管理器对一个或多个EEPROM和Flash Device的访问。

以上是讲了Memory的接口方面的定义和使用，后续再写个文章，专门讲解EA和EEPROM方面的内容。



5**AUTOSAR 层之间交互（Communication）**



接下来讲下Communication的接口，下图是通信模块在AUTOSAR架构的位置。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx8rNyibTtFnVU9E4TibxYOxcsMNG6Jibib0bhOxxDMicNhySFk9cQSHkOdawsm4aUYTJ7fhkatVKSd787w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

以下以PDUR为例，PDUR是啥玩意？好了，先来学习几个概念：

| SDU  | SDU是“ Service Data Unit”的缩写。 它是上层传递的数据，带有传输数据的请求。 也是在下层接收之后提取并传递到上层的数据。SDU是PDU的一部分。 |
| ---- | ------------------------------------------------------------ |
| PCI  | PCI是“ Protocol Control Information”的缩写。 需要此信息才能将SDU从特定协议层的一个实例传递到另一实例。 例如。 它包含源和目标信息。PCI由发送侧的协议层添加，并在接收侧再次被删除。 |
| PDU  | PDU是“ Protocol Data Unit”的缩写。PDU包含SDU和PCI。 在传输侧，PDU从上层传递到下层，后者将此PDU解释为其SDU。 |

这个数据流是如下图这样：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx8rNyibTtFnVU9E4TibxYOxcs9Cj5BO1Bz3fSGUpGVgyuS51Thq7eD6AqD9Vb4ibk8iccBUo56OvHbfmw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

另外，PDU和SDU的命名规则如下：

对于PDU: <bus prefix> <layer prefix> - PDU

对于SDU: <bus prefix> <layer prefix> - SDU

这个bus prefix 和layer prefix可以看以下表格描述:

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx8rNyibTtFnVU9E4TibxYOxcskQ3qQRNQv0c5eBeR5W8C3FftxJm5y2kaptjRIOI10jfPJyzcXy86Og/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

其中，**SF**: Single Frame，**FF**: First Frame，**CF**: Consecutive Frame，**FC**: Flow Control

例如：

- I-PDU or I-SDU
- CAN FF N-PDU 或者FR CF N-SDU
- LIN L-PDU 或者FR L-SDU

（有关帧类型的详细信息，请参阅CAN，TTCAN，LIN和FlexRay的AUTOSAR传输协议规范。）

以上提到的PDUR，即PDU Router，为了解释这个案例，我们先看看关于通信的组件描述：

**PDU Router：**

- 提供不同抽象通信控制器和上层之间的PDU Router
- Router的比例是ECU特定的（如果只有一个通讯控制器，则不能缩小）
- 实时提供TP路由。在缓冲全部TP数据之前，开始TP数据的传输

**COM：**

- 提供不同I-PDU之间的单个信号或一组信号的路由

**NM** **Coordinator：**

- 通过NM协调器处理的网络管理同步连接到ECU的不同通信通道的网络状态

**Communication State Managers：**

- 通过接口启动和关闭通信系统的硬件单元
- 控制PDU组

好了，看看这个架构的PDUR层之间的交互：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx8rNyibTtFnVU9E4TibxYOxcsJOOHNWfkpEmSnEyUCniaO1aPby5u0wicqb69iceDZia0yPBFBBtW8sqZhw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

下图显示了以太网协议栈内部的相互作用

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx8rNyibTtFnVU9E4TibxYOxcs9BYlcB0GgOEBiaajfAic7vwQR8mhbZ7x9JPibibQITP7nKc6JZO6ZnGNgg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

看完之后，也许还有点懵逼。带着以下问题，做进一步讲解

- 软件层如何交互？
- 软件Interface是怎样的？

**案例**

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx8rNyibTtFnVU9E4TibxYOxcs9QAc2YN6xHrl3JIEa5RQpkwIHLZJgliaz48FcT7IBBWU4EYM08aIckg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

此示例显示了将数据转换用于ECU间通信时的数据流。

SW-C将配置为要发送到远程ECU的数据进行数据转换。此数据转换不使用就地缓冲区处理。

有以下功能

- RTE将SOME/IP转换器称为链中的第一个转换器，并从SW-C传输数据。
- SOME/IP转换器执行转换并将输出（字节数组）写入RTE提供的缓冲区。
- 之后，RTE将执行在转换器链中排第二的安全转换器。安全转化器的输入是SOME/IP转化器的输出。
- 安全转换器保护数据，并将输出写入RTE提供的另一个缓冲区。由于不使用就地缓冲区处理，因此需要新的缓冲区。
- RTE将最终的输出数据作为字节数组传输到COM模块。

**进一步研究接口**

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx8rNyibTtFnVU9E4TibxYOxcsB2KdsvE6WHNgxF7SCYib4iaqcKLMQ8WqvnUUpUkfrcuAibrBMLLyupzfw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在架构上，RTE使用位于System Service层的转换器

在该案例中，转换器有以下接口：

```c

SomeIpXf_SOMEIP_Signal1
(
uint8 *buffer1,
uint16 *buffer1Length,
<type> data
)
SafetyXf_Safety_Signal1
(
uint8 *buffer2,
uint16 *buffer2Length,
uint8 *buffer1,
uint16 buffer1Length
)
```

基于COM的转换

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx8rNyibTtFnVU9E4TibxYOxcscWSicmEHdjsuUo7mIG8VfwXPsT8LibcLRlB813GpQAUs3XMBYCwJn2JA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

基于COM的转换器基于固定的通信矩阵为转换器链提供序列化功能。

固定通信矩阵允许将信号优化放置到PDU中（例如，BOOL数据可以配置为仅占用PDU中的一位）。这样可以在Can或Lin等低负载网络中使用转换器链。

具有以下功能

- 基于COM的转换器是第一个转换器（序列化器），并通过RTE从应用程序获取数据。
- 基于COM配置（通信矩阵），以完全与COM模块相同的方式序列化数据（字节序，符号扩展）。
- 其他转换器可能会增加具有CRC和序列计数器（SC）的有效负载。
- 变转换器有效负载通过Com_SendSignalGroupArray API作为一个字节数组传递到COM模块。
- COM模块可以配置为根据通信矩阵定义执行传输模式选择。