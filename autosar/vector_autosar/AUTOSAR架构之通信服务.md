# AUTOSAR架构之通信服务

**1Communication Services – General**



**Communication Service**通信服务是一组用于车辆网络通信（CAN，LIN，FlexRay和以太网）的模块。它们通过通信硬件抽象与通信驱动程序接口。

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicSjYv20Qe3NauEibLxlk0o8syXw2mW810f3zWR7tWX7QXkB3xlUt4gTZtbGXz0Xb2RfeAbXzDz9Fw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicSjYv20Qe3NauEibLxlk0o85aYTYpFPspicllFWibWpzOmZicV8GQfT7Ciakia7IB4G6jSlPK3rHicKY7jQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**任务：**

为车辆网络提供统一的接口以进行通信。

提供统一的网络管理服务提供统一的车辆网络接口以进行诊断通信

在应用程序中隐藏协议和消息属性。

**特性：**

- 实现了µC和ECU硬件独立，部分取决于总线类型
- 上层接口与µC，ECU硬件和总线类型无关。
- 通信服务将在以下页面中详细介绍每个相关的车辆网络系统。



**2Communication Stack – CAN**



**CAN通信服务**是一组模块，用于与通信系统CAN进行车辆网络通信。

它提供与CAN网络的统一接口。在应用程序中隐藏协议和消息属性。

CAN通信栈支持，经典CAN通讯（CAN 2.0）和CAN FD通信（如果硬件支持）

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicSjYv20Qe3NauEibLxlk0o8jVuvNFL0vcn54c0jT7tdpFCYicTIRibhsLoagM3ne3HEsGA58sI6eUUg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicSjYv20Qe3NauEibLxlk0o8jBF6vUH1o8CcPEygbjK3yZf14fWk41athRpibiaJ7RLYBaImuR9TLQWQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**特性：**

- 实现了µC和ECU硬件独立，部分取决于CAN。
- AUTOSAR COM，通用NM（网络管理）接口和诊断通信管理器对于所有车辆网络系统都是相同的，每个ECU作为一个实例存在。
- 通用NM接口仅包含一个调度程序， 不包括其他功能。对于网关ECU，它还可以包括NM协调器功能，该功能允许同步多个（相同或不同类型的）不同网络以同步唤醒或关闭它们。
- CAN NM专用于CAN网络，并将在每个CAN车辆网络系统中实例化。
- 特定于通信系统的Can State Manager可以处理依赖于通信系统的启动和关闭功能。此外，它还控制COM的不同选项，以发送PDU并监视信号超时。

**Communication Stack Extension – TTCAN**

**特性：**

- TTCAN是CAN的绝对超集，即支持TTCAN的CAN栈可以同时服务于CAN和TTCAN总线。
- CanIf和CanDrv是仅有的需要扩展才能为TTCAN通信提供服务的模块。
- 对于具有TTCAN功能的CAN，Communication Stack CAN的属性也适用。



**3Communication Stack Extension – J1939**



**J1939通信服务**扩展了普通的CAN通信栈，用于重型车辆中的车辆网络通信。

它提供J1939所需的协议服务。在不需要的地方从应用程序隐藏协议和消息属性。

**注意，CAN栈中有两个传输协议模块（CanTp和J1939Tp），可以在不同的通道上交替使用或并行使用**

**它们的用法如下：**

CanTp：ISO诊断（DCM），标准CAN总线上的大型PDU传输

J1939Tp：J1939诊断，J1939驱动的CAN总线上的大型PDU传输

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicSjYv20Qe3NauEibLxlk0o8KtRQGCnvNoL7IuiabMBTXibeZLIqN8TUAhzvKVqJ1SUkmB9WPrZOS6OQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicSjYv20Qe3NauEibLxlk0o8qdtEo7zsVglkPo6G1oLB80PibBFiboJsIYPJun2LEmcvS1BVDjlpG5Jg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**特性：**

- 实现了基于CAN，独立于µC和ECU硬件。
- AUTOSAR COM，通用NM（网络管理）接口和诊断通信管理器对于所有车辆网络系统都是相同的，每个ECU作为一个实例存在。
- 支持在配置时未知的动态帧标识符。
- J1939网络管理可为每个ECU分配唯一的地址，但不支持睡眠/唤醒处理以及诸如部分联网之类的相关概念。
- 提供J1939诊断和请求处理。



**4Communication Stack – LIN**



**LIN通信服务**是用于与通信系统LIN进行车辆网络通信的一组模块。

它提供到LIN网络的统一接口。在应用程序中隐藏协议和消息属性。

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicSjYv20Qe3NauEibLxlk0o8QoYkx7QHa7yvMaVEC1IBkx7nD8jArNWiagw4etWrNMLJ5zKb4vibuReA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicSjYv20Qe3NauEibLxlk0o8oiclrY2XkZNcVwgX8SWd1hicfY7C3aHKRIeNCHyM9DL19JibGdNd5UgUg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**LIN通信服务包含：**

- 符合ISO 17987的通信栈
- 计划表管理器，用于处理切换到其他计划表的请求（对于LIN主节点）
- 不同LIN框架类型的通讯处理
- 传输协议，用于诊断
- 唤醒和睡眠界面
- 基本的LIN驱动程序：
- 实施LIN协议并访问特定的硬件
- 同时支持简单的UART和基于复杂帧的LIN硬件

注意将LIN集成到AUTOSAR中的情况：

- LIN接口控制WakeUp / Sleep API，并允许Slave端使总线保持唤醒状态（分散式方法）。
- 特定于通信系统的LIN状态管理器处理与通信相关的启动和关闭功能。此外，它控制来自Communication Manager的通信模式请求。LIN状态管理器还通过连接COM来控制I-PDU组。
- 发送LIN帧时，LIN接口在需要数据的时间点（即在发送LIN帧之前）向PDU路由器请求帧（I-PDU）的数据。



**5Communication Stack – FlexRay**



**FlexRay通信服务**是一组模块，用于与通信系统FlexRay进行车辆网络通信。

它提供与FlexRay网络的统一接口。在应用程序中隐藏协议和消息属性。

**注意：**

- FlexRay栈中有两个传输协议模块，可以交替使用
- FrTp：FlexRay ISO传输层
- FrArTp：FlexRay AUTOSAR传输层，提供与AUTOSAR R3.x的总线兼容性

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicSjYv20Qe3NauEibLxlk0o8QoYkx7QHa7yvMaVEC1IBkx7nD8jArNWiagw4etWrNMLJ5zKb4vibuReA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicSjYv20Qe3NauEibLxlk0o8D6DhZpndOHQJiauibldjTx0sHV9jt5xld021qjuUGA2wrrsl7OAATK5w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**特性：**

- 实施µC和ECU硬件独立，部分取决于FlexRay。
- AUTOSAR COM，通用NM接口和诊断通信管理器对于所有车辆网络系统都是相同的，每个ECU作为一个实例存在。
- 通用NM接口仅包含一个调度程序。不包括其他功能。对于网关ECU，将其替换为NM协调器，该协调器还提供了同步多个不同网络（相同或不同类型）以同步唤醒或关闭它们的功能。
- FlexRay NM专门用于FlexRay网络，并在FlexRay车载网络系统中实例化。
- 特定于通信系统的FlexRay状态管理器处理与通信系统有关的启动和关闭功能。此外，它还控制COM的不同选项，以发送PDU并监视信号超时。



**6Communication Stack – TCP/IP**



**TCP/IP通信服务**是一组模块，用于与通信系统TCP/IP进行车辆网络通信。

它提供一个到TCP/IP网络的统一接口。在应用程序中隐藏协议和消息属性。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicSjYv20Qe3NauEibLxlk0o8oxvicnUHpkylChpZ8v3b0veuLfXAbkZstkLAhwkE8p14MDeR5TOibu4w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicSjYv20Qe3NauEibLxlk0o86Ya7DAJcVGibepTrSZM2g7U79ibb5uSGbZR4eHNicbJ6ymuica2FeSnsWQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**特性：**

- TcpIp模块实现TCP / IP协议家族的主要协议（TCP，UDP，IPv4，IPv6，ARP，ICMP，DHCP）并通过以太网提供基于套接字的动态通信。
- 套接字适配器模块（SoAd）是TcpIp模块的唯一上层模块。



**7Communication Stack – General**



**General Communication Stack**属性：

- 信号网关是AUTOSAR COM的一部分，用于路由信号。
- 基于PDU的网关是PDU路由器的一部分。
- IPDU复用提供了添加信息的可能性，以实现I-PDU的复用（内容不同，但总线上的ID相同）。
- 多I-PDU到容器的映射提供了将多个I-PDU组合成一个较大的（容器）I-PDU的可能性，以便在一个（特定于总线的）帧中进行传输。
- 上层接口：µC，ECU硬件和网络类型无关。
- 有关GW体系结构的细化，请参阅“示例通信”



**8Off-board Communication Stack – Vehicle-2-X**



**Off-board Communication Service**是用于通过自组织无线网络进行**Vehicle-to-X**通信的一组模块。

- 实现用于接收和传输标准化**V2X**消息的功能，为特定于车辆的SW-C建立接口
- 基本传输协议=第4层
- 地理网络=第3层（根据地理区域寻址，相应的以太网帧具有自己的以太类型）
- **V2X**管理：管理跨层功能（例如动态拥塞控制，安全性，位置和时间）

它提供与无线以太网网络的统一接口。在应用程序中隐藏协议和消息属性。

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicSjYv20Qe3NauEibLxlk0o8ic5xvVM8tZicDSUfhpkoQ0SeqQVW2a6FYmrXaQB5MbjsQz5An0gx6GTw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicSjYv20Qe3NauEibLxlk0o8pLSxYv6E0znmnWsUKJiaygj2Czs9HEzqYZujvrv6Sv4UduPDHoqzSqA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



*本文参考AUTOSAR官方架构文档，图片也来源AUTOSAR官方。*