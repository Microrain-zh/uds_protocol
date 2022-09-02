# AUTOSAR架构的故事

1**AUTOSAR架构概览**

欧洲大陆的车企们在2002年成立了一个联盟，搞了个叫AUTOSAR的标准，以期一统天下。次年，他们就开搞了，开始制作这个AUTOSAR的草图。

话说，这是要定义一套标准，一个统一的架构，那这架构有什么内容呢？

一位工程师，将其想法用草图表达了出来

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibUZofzhuLE633lPKWtWontRVyoCzTHVtmuIOMEuEDXChIOtUOTBia1dib7YibVPiamCBibcIJ4CxYG2yA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

并解释说，这个架构大概分三层，然后看看在座的各位。会议上的其他人面面相觑，都想说，这么简陋，能统一江湖？这位工程师也不理会，不慌不忙，继续画下去：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9L6bqBibggdRyp6LMMKfajs9Xu53H19zvc7EAyeN3yjn5q7ZllgZQbMib8ZB33A53AxBBCPqOQqwfA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

工程师解释说，关键在这个BSW，还可以分个三四层：

- **Service Layer**：这个是BSW的最高层，是给Application访问由Abstraction Layer覆盖的IO信号等。
- **ECU Abstraction Layer**：这个是给底层驱动提供抽象接口的。
- **Microcontroler Abstraction Layer**：这个是基础软件的最底层，它包含有μC的驱动以及外围的设备驱动等。

还有，Application就是应用，跟其他架构写的应用类似，就不用多说了。而这个Runtime Environment(RTE)呢？

RTE是给应用提供通信服务的，Applcation通过它可以访问BSW的功能，做统一标准的接口，从而实现非常方便的可移植性了，各模块也独立了。

工程师言简意赅，没有做更多的解释，然后就想细化这个架构图：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibUZofzhuLE633lPKWtWontKEY1c9ZrrymgrHXlibdff2vewlLuViazicukSl86ylqIrBFsR7wOJuibqw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后，他微微笑了下：这应该清晰了吧，横着看竖着看都很好理解。比如说这个Memory Service，就可以让Application通过RTE调用，做了很多方法application访问的接口，它是通过调用Memory Hardware Abstraction来访问Memory Drivers的。

通过这三层关系就实现市面上大部分功能需求了。但是，总会有这几层实现不了的功能吧，怎么办？Complex Drivers就可以提供给用户自己定义啦，可以做一些复杂的驱动。

总的来说，基础软件（BSW）可以按以下类型分：

- **Input/Output (I/O)**

  标准化访问sensors, actuators以及板上的外围器件。

- **Memory**

  标准化访问内部或外部的Memory（NVM）。

- **Crypto**

  标准化访问密码原语，包括内部/外部硬件加速器

- **Communication**

  标准化访问车辆网络系统，ECU车载通信系统和ECU内部软件

- **Off-board Communication**

  标准化访问车辆到X的通信，车辆无线网络系统中，ECU车外通信系统

- **System**

  提供标准化的（操作系统，计时器，错误存储器）和特定于ECU的（ECU状态管理，看门狗管理器）服务和库功能

等等，还有个东西差点漏了——**Library**

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxic0kIGIvqSV9VnXBlFjKwvKkhcIhUm8R3Sr5ZyTRYWT90o9fZ3bRNGiaBsYHNuPyjNib4EeSiaxO4b6Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这个Library实际上是很多functions的集合，它可以：

- 由BSW模块（包括RTE），SW-C，库或集成代码调用
- 在同一保护环境中在调用方上下文中运行
- 只能调用库
- 可重入的
- 没有内部状态
- 不需要任何初始化
- 是同步的，即它们没有等待点



2**AUTOSAR的基础软件**



基础软件，即BSW。工程师打算详细讲解下这个BSW，因为它非常重要。

话说，座上的与会大咖听这位工程师讲的津津有味，越来越觉得AUTOSAR一统江湖指日可待，并期望能讨论更多细节。

工程师也准备好了设计原稿，对这些细节娓娓道来。

**MCAL**

从底下往上讲，第一个Microcontroler Abstraction Layer（即MCAL），其有以下模块：

- **Microcontroller Drivers**

  具有直接µC访问权限的内部外围设备（例如看门狗，通用定时器）驱动程序（例如核心测试）

- **Communication Drivers**

  车载ECU（例如SPI）和车辆通信（例如CAN）的驱动程序。OSI层：数据链路层的一部分。

- **Memory Drivers**

  片上存储设备（例如内部闪存，内部EEPROM）和存储器映射的外部存储设备（例如外部闪存）的驱动程序。

- **I/O Drivers**

  用于模拟和数字I / O的驱动器（例如ADC，PWM，DIO）。

- **Crypto Drivers**

  用于SHE或HSM等片上加密设备的驱动程序。

- **Wireless Communication Drivers**

  用于无线网络系统的驱动程序（车载或车外通信）。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxic0kIGIvqSV9VnXBlFjKwvKkdCl3xOn8undgHFHjXKooEYuolceXobgrZJHySnguaG8nafVgDEibiaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

举例说明，**SPIHandlerDriver**是允许多业务并发访问，为了抽象出SPI的所有功能，这些已经用为SPI功能的IO是不能做他用的。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxic0kIGIvqSV9VnXBlFjKwvKPvReywIVUC3DKVyAEREgbJjOMs3KCJEEOWqHV95HQiaqcqNbslahbqQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxic0kIGIvqSV9VnXBlFjKwvKKle8h22kicnEB9AicPueXVibibr2xzmpy0ak4MIRkWtHH3QRkCoL3ibMK0w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**CDD**

CDD即Complex Driver，上文也提到了，它是用来实现BSW里面非标准化功能的。也就是说，标准化定义以外的功能可以通过CDD来实现，如UART，MCAL是没有定义UART的标准化的。


**I/O Hardware Abstraction**

I/O Hardware Abstraction是一组模块，从外围I/O设备（片上或板上）的位置和ECU硬件布局（例如µC引脚连接和信号电平反转）中抽象出来。I/O硬件抽象不会从传感器/执行器中抽象出来！可以通过I /O信号接口访问不同的I/O设备。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxic0kIGIvqSV9VnXBlFjKwvKR9f7Yy1VG9ia7Eia16vGUPRpgZk4H5ssF6icSej9d8ia3rpTw0M499EFBw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**Communication Hardware Abstraction**

Communication Hardware Abstraction是一组模块，从通信控制器的位置和ECU硬件布局中抽象出来。对于所有通信系统，都需要特定的通信硬件抽象（例如，对于LIN，CAN，FlexRay）。

例如，ECU具有带2个内部CAN通道的微控制器和带4个CAN控制器的附加板载ASIC。CAN-ASIC通过SPI连接到微控制器。

可通过总线特定的接口（例如CAN接口）访问通信驱动程序。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxic0kIGIvqSV9VnXBlFjKwvKiagvXlrnGKlfQ8WB5T3pGAWCSSxibtM2CmqstjDq4ziax0e2iaNUuxDZpg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**Memory Hardware Abstraction** 

Memory Hardware Abstraction 是一组模块，从外围存储设备（片上或板载）的位置和ECU硬件布局中抽象出来。

例如，可以通过相同的机制访问片上EEPROM和外部EEPROM器件。可以通过特定于存储器的抽象/仿真模块（例如EEPROM抽象）访问存储器驱动程序。通过在闪存硬件单元顶部模拟EEPROM抽象，可以通过存储器抽象接口对两种类型的硬件进行通用访问。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxic0kIGIvqSV9VnXBlFjKwvKxMBCorwKrLjhh3N4NW3qofh7NfAeBS0nNZ2p5syNRpAfpOqMJwb5qA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**Onboard Device Abstraction**

Onboard Device Abstraction 包含用于ECU板载设备的驱动程序，不能将其视为传感器或执行器，例如内部或外部看门狗。这些驱动程序通过µC抽象层访问ECU车载设备。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxic0kIGIvqSV9VnXBlFjKwvKnWnRNqocla7JibML7pJHxGicDaAiaSco2QwYZvqInFWABHyFxHWKlQDbQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**Crypto Hardware Abstraction**

Crypto Hardware Abstraction是从加密基元（内部或外部硬件或基于软件）的位置抽象的一组模块。例如，AES原语在SHE中实现或作为软件库提供。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxic0kIGIvqSV9VnXBlFjKwvKQn8oGhSkLCZLn6V0SXqkQvRnW8hoSibugd2QEjPY7saevOvxNEVtCWA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**Crypto Services**

Crypto Services包含两个模块：

1. 加密服务管理器负责加密作业的管理
2. 密钥管理器与密钥预配置主机（在NVM或加密驱动程序中）进行交互，并管理证书链的存储和验证



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxic0kIGIvqSV9VnXBlFjKwvKkMWj9nibicPrToGn8bJkhULKPOSfzMsKmcs33Dv06GnlUAJ8nbF3lDiaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**Communication Services**

Communication Services是一组用于车辆网络通信（CAN，LIN，FlexRay和以太网）的模块。它们通过通信硬件抽象与通信驱动程序接口。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxic0kIGIvqSV9VnXBlFjKwvKDKFeRebg84LU6ia6jejlhGdiae2Py1NV5C6BP1cIGHxQvcDASSxf9RWA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxic0kIGIvqSV9VnXBlFjKwvKXpraXdxbB55Uyxywd4zu5ZBoMaGiave9bnAfYZ9FH12pAl2420lAxFg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

讲到这里，会上的其他人听着听着有些懵逼，越讲越复杂了，工程师停了下，说，通信服务这部分，涉及到CAN、LIN甚至TCP/IP等，内容很多也很重要，后续我们另外召开会议讨论吧。

**Memory Services**

Memory Services由一个模块NVRAM管理器组成。它负责非易失性数据的管理（从不同的内存驱动器读取/写入）。

它以统一的方式向应用程序提供非易失性数据。内存位置和属性的摘要。提供非易失性数据管理机制，例如保存，加载，校验和保护和验证，可靠的存储等。
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFYVpAmHuXauNc6mbtExZafA8S7mFeD9cSN9JzDKgtLiaGuEhJsyHrsO2A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
**System Services**
System Services是一组模块和功能，可由所有层的模块使用。示例包括实时操作系统（包括计时器服务）和错误管理器。其中一些服务是：取决于µC（例如OS），并且可能支持特殊的µC功能（例如Time Service），部分依赖ECU硬件和应用程序（例如ECUM Management）或硬件和µC独立。它提供基本的申请服务以及基本软件模块。
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFYeoZpUice2tgBamgdrbFHPE4gKkFicRbRibcdQiaKgHekD4PXwy07qog5zA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFYM13h3wPhCPm9aWZqg5boyvCrxTMAg4cxSZG9icmGmKaS81cHnk55mVA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
有专门的模块可用于AUTOSAR中错误处理的不同方面， 例如：

* 诊断事件管理器负责处理和存储诊断事件（错误）和相关的FreezeFrame数据。
* 诊断日志和跟踪模块支持对应用程序进行日志记录和跟踪。它收集用户定义的日志消息并将其转换为标准化格式
* 基本软件中检测到的所有开发错误都报告给默认错误跟踪程序。
* Diagnostic Communication Manager提供了用于诊断服务的通用API
* 其他的，等等

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFYbHCljLBmpWw3l5Q30gVZ1UORdH5wOquVAqyAE6djL7pQ0sPjfwq5mA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFY4ehm0wibdbB6ux8TbG3oPmX5icsX7tvTYgMUJSwWOgN9N1z4cDFWhclA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

3**AUTOSAR的多核处理**

此刻，有人提问，市面上μC更新换代非常快，性能也不断提升，甚至有多核处理器，这个架构如何应对多核处理？

工程师想了想，show出了下图

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFYsB2hO87NGBdhjbb05YVRaPpTl4la7JtdssmiaelZJszFEwicia9pOib7SQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
并结合下图给出解释：

* BSW模块可以分布在多个分区和核心中。所有分区共享相同的代码。
* 每个分区上的模块可以完全相同，如图中I/O堆栈中的DIO驱动程序所示。
* 作为替代，他们可以使用依赖于核心的分支来实现不同的行为。Com服务和PWM驱动程序使用卫星主机通信来处理从相应卫星到主机的呼叫。
* 主站与卫星之间的通信不规范。例如，它可以基于BSW调度程序提供的功能，也可以基于共享内存。
* 箭头指示服务调用的处理涉及哪些组件，这取决于分发方法和调用的来源。

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFY6ZemySKU4w5QLCNOF0fIzPzemXp3kL2axS7wsRzlHwrMsIdl0j61xA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

* 每个运行BSW模块的分区中的基本软件模式管理器（BswM）
* 所有这些分区都是受信任的
* 每个内核一个EcuM（每个在一个受信任的分区中）
* 通过引导加载程序启动的那个核心上的EcuM是主EcuM
* 主EcuM启动所有卫星EcuM![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFYCtjqnpsiaEdr4iaic5oKgxaLXDxHicRZ72Dy38xkvHwdQDlapibo022Lcicg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
* 如图所示，IOC提供了通讯服务，这些客户端可以访问需要跨同一ECU上OS-Application边界进行通讯的客户端。IOC是操作系统的一部分。
* BSW模块可以在多个内核上执行，例如图中的ComM。在运行时确定负责执行服务的核心。
* 每个核心都运行一种ECU状态管理
 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFYwvcAqaggLVOiafrwbedrjnvicibVgQVlvsTN8R8TgvvqCdoYhCc5uHdaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFYang6icDnEbDpIoLtwlszVbp2FuDgY2WiaVkbHlNRicEzT70ZZe9D8SMiaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

4**AUTOSAR的Safety功能**

那么，安全功能如何？工程师都有考虑和准备：

AUTOSAR提供了一种灵活的方法来支持与安全相关的ECU，可以使用两种方法：

* 所有BSW模块均根据所需的ASIL开发
* 所选模块是根据ASIL开发的，ASIL和非ASIL模块分为不同的分区（BSW分发）

注意：分区基于OSApplications，OS-Application的TRUSTED属性与ASIL/non-ASIL不相关。![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFYIkOEUmTYYVKcwFMAGypjBMIAhEAat6CNA4DsuS5gVo5nROiaGbTOF0Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
使用不同BSW分区的示例，看门狗堆栈放置在自己的分区中

* ASIL和非ASIL SW-C可以通过RTE访问WdgM
* BSW的其余部分放在自己的分区中
  ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFYibYCrFpJYvmOIFrLU6kXX4n8ja6LCMEobWCnxJjiaKCdtyZn8MiavtN7w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

经过以上的解答和分解，会议上的各位都点头表示同意，但是还是有人提了个问题，我们定义这么多功能模块，不一定所有产品都会用到，如何裁剪？

5**AUTOSAR的ICC**

好了，我们接下来讨论ICC（Implementation Conformance Class）：

下图显示了基本软件模块到AUTOSAR层的映射
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFY2OKlIuuib7M32FYqfnJhlgk3pHu8CmjFe5J4VjeEKA3NEGrdTBh1gDQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
我们称这个为ICC3，那ICC2是怎样的？
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFYP5nFDKyJHOAfstia8zN6yjicxujBricIANUX94oHNMj5NWTDibD9ZiaZDkQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
这个便是ICC2了。

到目前为止，本文档中显示的集群是该项目定义的集群。 

当前，AUTOSAR并未将ICC2级别上的群集限制为专用群集，因为许多不同的约束和优化标准可能会导致不同的ICC2群集。 可能存在不同的AUTOSAR ICC2集群，可以根据要定义的ICC2遵从性方法声明符合性。

ICC1呢？
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFYdF2r76HKUAxMMn8qcVLpCh2AdQDS2L17G1a06gCDViaa8Kp6icby3ebQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
在符合ICC1的基本软件中，不需要模块或集群。未指定此专有基本软件的内部结构。纳尼？又回到原来的三层结构？
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9icibYmqKI1xxVMzW0AxmOFYc7uwAryXbejlv0q8biaiaOHVvicmCIpprKj9rFEpL1VMkgQs0tdRhTBAw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
兼容AUTOSAR（ICC1-3）的基本软件（包括RTE）必须具有ICC3模块规范指定的外部性能。例如，针对以下行为：总线引导加载程序和应用另外，关于ICC3中的系统描述，ICC1 / 2配置应兼容。

也是说，这个架构能屈能伸，只需遵循相关标准要求即可，非常灵活。