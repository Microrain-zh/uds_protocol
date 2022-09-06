# AUTOSAR CAN Driver（CAN驱动程序）

# 1. 简介和功能概述

本文介绍了AUTOSAR基础软件模块CAN驱动程序的功能、API和配置。

Can模块是底层的一部分，主要执行硬件访问，并向上层提供与硬件无关的API。只有上层模块CanIf模块能够唯一访问Can模块的。

Can模块为发起传输，并调用CanIf模块的回调函数来通知事件提供服务，它独立于硬件。此外，它还提供控制属于同一CAN硬件单元（CAN Hardware Unit）的CAN控制器（CAN controller）的行为和状态的服务。多个CAN控制器可以由一个CAN模块控制，只要它们属于同一个CAN硬件单元。

# 2. 术语

| 术语                     | 中文                                  | 描述                                                         |
| ------------------------ | ------------------------------------- | ------------------------------------------------------------ |
| CAN controller           | CAN控制器                             | 一个CAN控制器仅服务于一个物理通道。                          |
| CAN Hardware Unit        | CAN硬件单元                           | CAN硬件单元可以由一个或多个相同类型的CAN控制器和一个或多个CAN RAM区域组成。CAN硬件单元可以是片上设备，也可以是外部设备。CAN硬件单元由一个CAN驱动程序表示。 |
| CAN L-PDU                | CAN L-PDU                             | 数据链路层协议数据单元。由标识符（Identifier）、数据长度（Data Length）和数据（Data SDU）组成。（见[19]） |
| CAN L-SDU                | CAN L-SDU                             | 数据链路层服务数据单元。在L-PDU内部传输的数据。(见[19])      |
| DLC                      | Data Length Code                      | CAN报文中有部分描述SDU的长度                                 |
| Hardware Object          | 硬件对象                              | CAN硬件对象定义为CAN硬件单元或者CAN控制器的CAN RAM中的一个PDU缓冲区。硬件对象定义为CAN硬件单元CAN RAM中的L-PDU缓冲器。 |
| HRH                      | Hardware Receive Handle               | 硬件接收句柄（HRH）由CAN驱动程序定义和提供。每个HRH通常只表示一个硬件对象。HRH可用于软件过滤的优化。 |
| HTH                      | Hardware Transmit Handle              | 硬件传输句柄（HTH）由CAN驱动程序定义和提供的。每个HTH通常只表示一个或多个硬件对象，这些硬件对象被配置为硬件传输缓冲池。 |
| Inner Priority Inversion | 内部优先级反转                        | 高优先级的L-PDU的传输由于在同一个发送硬件对象中存在一个未决的低优先级的L-PDU而被阻止。 |
| L-PDU Handle             | L-PDU句柄                             | L-PDU句柄被定义并放置在CanIf模块层内。通常每个句柄代表一个L-PDU，它是一个包含Tx/Rx处理信息的常量结构。 |
| Outer Priority Inversion | 外优先级反转                          | 由于两个连续发送的L-PDU之间存在时间间隔。在这种情况下，来自另一个节点的低优先级的L-PDU可以阻止发送自己的高优先级的L-PDU。此时，高优先级的L-PDU不能参与网络访问仲裁，因为低优先级的L-PDU已经赢得了仲裁。 |
| Physical Channel         | 物理信道                              | 一个物理通道代表一个从CAN控制器到CAN网络的接口。CAN硬件单元的不同物理通道可能访问不同的网络。 |
| Priority                 | 优先级                                | CAN L-PDU的优先级由CAN标识符（CAN Identifier）表示。标识符的数值越低，优先级越高。 |
| SFR                      | Special Function Register             | 控制控制器行为的硬件寄存器                                   |
| SPAL                     | Standard Peripheral Abstraction Layer | 标准外围抽象层                                               |

# 3. 优先级反转

## 3.1. 内部优先级反转（Inner Priority Inversion）



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhra58ibUqdbL6ADthdjm7KSGmCMcWLyicQgchC4tCE8pBGNt4iaDLSut4xJhbsia0aqTzcqxfXbI6VPLg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



如果只使用一个发送缓冲区，内部优先级反转可能会发生。由于低优先级，存储在缓冲区中的消息要等到“总线上的流量平静下来。在等待的时间内，该消息可以阻止由同一微控制器产生的高优先级消息在总线上传输。

## 3.2. 外部优先级反转（Outer Priority Inversion）



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhra58ibUqdbL6ADthdjm7KSGTNyv7xVJk1NXG7M4bS3sopH5t55suAEV3GrWzkqUx7SH2q8uicH916A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在某些CAN实现中可能会出现外部优先级反转的问题。

假设CAN节点希望传输一个连续的高优先级消息包，这些消息包被存储在不同的消息缓冲区中。如果在CAN网络中，这些消息之间的帧间距大于CAN标准定义的最小帧间距，则第二个节点可以开始传输优先级较低的消息。最小帧间空间由3个隐性位组成的Intermission字段决定。一个消息因为在另一个消息的传输过程中被挂起，在总线空闲期间开始发送，最早在Intermission字段之后的位。例外是，有等待传输消息的节点将把中断的第三位的主导位解释为帧开始位，并从第一个标识位开始传输，而不首先发送SOF位。CAN模块的内部处理时间必须足够短，以发送连续的消息，帧间空间最小，以避免在上述所有场景下的外部优先级反转。

# 4. CAN 硬件单元

CAN硬件单元由一个或多个CAN控制器组成。这些控制器可以位于芯片上，也可以作为同一类型的外部独立设备，具有公共或单独的硬件对象（Hardware Objects）。

如下图所示，CAN硬件单元由两个CAN控制器连接两个物理通道组成:

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhra58ibUqdbL6ADthdjm7KSGNC5iaKqibYb2xDotdyoTLEuwEZZy3RbNQpXmZdbBkeV8CAjlCjtH3cvQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

# 5. 相关的文档

## 5.1. 输入文档

[1] Layered Software Architecture

> AUTOSAR_EXP_LayeredSoftwareArchitecture.pdf

[2] General Requirements on Basic Software Modules

> AUTOSAR_SRS_BSWGeneral.pdf

[3] General Requirements on SPAL

> AUTOSAR_SRS_SPALGeneral.pdf

[4] Requirements on CAN

> AUTOSAR_SRS_CAN.pdf

[5] Specification of CAN Interface

> AUTOSAR_SWS_CANInterface.pdf

[6] Specification of Default Error Tracer

> AUTOSAR_SWS_DefaultErrorTracer.pdf

[7] Specification of ECU State Manager

> AUTOSAR_SWS_ECUStateManager.pdf

[8] Specification of MCU Driver

> AUTOSAR_SWS_MCUDriver.pdf

[9] Specification of Operating System

> AUTOSAR_SWS_OS.pdf

[10] Specification of ECU Configuration

> AUTOSAR_TPS_ECUConfiguration.pdf

[11] Specification of SPI Handler/Driver

> AUTOSAR_SWS_SPIHandlerDriver.doc.pdf

[12] Specification of Memory Mapping

> AUTOSAR_SWS_MemoryMapping.pdf

[13] Specification of BSW Scheduler

> AUTOSAR_SWS_BSW_Scheduler.pdf

[14] Basic Software Module Description Template

> AUTOSAR_TPS_BSWModuleDescriptionTemplate.pdf

[15] List of Basis Software Modules

> AUTOSAR_TR_BSWModuleList.pdf

[16] General Specification of Basic Software Modules

> AUTOSAR_SWS_BSWGeneral.pdf

[17] Specification of Time Synchronization over CAN

> AUTOSAR_SWS_TimeSyncOverCAN.pdf

## 5.2. 相关标准及规范

[18] ISO11898 – Road vehicles - Controller area network (CAN)

[19] ISO/IEC 7498-1 – OSI Basic Reference Model

[20] CiA601-2 Node and system design Part 2: CAN controller interface specification

[21] CiA603 – CAN Frame time-stamping

# 6. 约束和假设

## 6.1. 限制

CAN控制器总是对应一个物理通道。允许在总线端连接物理通道。无论如何，CanIf模块将单独处理相关的CAN控制器。

有些CAN硬件单元可能支持通过使用CAN RAM组合几个CAN控制器，以扩展一个CAN控制器的消息对象的数量。这些组合的CAN控制器被CAN模块作为一个控制器处理。

Can模块不支持CAN的远程帧（CAN remote frames）。Can模块不传输由远程传输请求触发的消息。同时Can模块也应该初始化Can HW以忽略任何远程传输请求。

## 6.2. 对汽车领域的适用性

Can模块可用于任何使用Can协议的应用程序。

# 7. 对其他模块的依赖关系

## 7.1. 驱动程序服务的依赖

如果CAN控制器是片载的（on-chip），CAN模块不能使用其他驱动的任何服务。

Can_Init函数应该初始化CAN控制器使用的所有片上硬件资源。唯一的例外是数字I/O引脚配置（即使为CAN使用的引脚），这是是通过Port驱动程序完成的。

Mcu模块应配置与其他模块共享的寄存器设置。同时在初始化Can模块之前，需要先初始化Mcu模块。

如果使用芯片外CAN控制器，CAN模块可以使用其他MCAL驱动程序（如：SPI）的服务。同时如果Can模块使用了其他MCAL驱动的服务，必须确保这些驱动在初始化Can模块之前已经启动并运行。不同驱动的初始化顺序部分在[7]中指定。

Can模块应该使用底层MCAL驱动的同步API，并且不应该提供MCAL驱动可以调用的回调函数。因此μC和CAN硬件单元之间的连接类型只会对实现有影响，而对API没有影响。

## 7.2. 系统服务的依赖

在某些特殊的硬件情况下，Can模块可能需要轮询硬件事件。

Can模块需要使用系统服务提供的OsCounter进行超时检测，以防硬件在预期时间内没有反应（如：硬件的故障），从而避免出现循环。Can模块函数等待硬件反应的阻塞时间应小于Can主函数（即：Can_MainFunction_Read）触发时间（trigger period），因为Can主函数无法用于此目的。

## 7.3. Can模块用户

Can模块需与其他模块进行直接的交互。如：默认错误跟踪（DET），Ecu状态管理器（EcuM）和CanIf模块。驱动程序只将CanIf模块视为起点和目的地。

# 8. 功能规范

在L-PDU传输时，Can模块将L-PDU写入Can控制器硬件内部的适当缓冲区。在L-PDU接收时，Can模块以ID、数据长度和L-SDU指针为参数调用接收指示回调函数（RX indication callback function）。

Can模块需要提供了一个接口作为周期性处理函数，同时此函数由Basic Software Scheduler模块周期性地调用。此外，Can模块提供服务来控制CAN控制器（CAN controller）的状态。通过回调函数通知Bus-Off和唤醒（Wakeup）事件。Can 模块是访问硬件资源的基础软件模块。因此它需要满足AUTOSAR_SRS_SPAL中指定的基础软件模块的要求。

Can模块需要为所有所需的CAN硬件单元（CAN Hardware Unit）中断，实现中断服务程序（interrupt service routine）。同时Can模块也需要负责禁用CAN控制器（CAN controller）中所有未使用的中断。Can模块应在中断服务程序结束时重置中断标志（如果重置动作不是由硬件自动完成）。Can模块也不得设置向量表条目的配置（如：优先级）。

## 8.1. 驱动程序

一个Can模块提供对一个CAN硬件单元（CAN Hardware Unit）的访问，该硬件单元可能由多个CAN控制器（CAN controller）组成。对于不同类型的CAN硬件单元，需要实现不同的CAN模块。

如果在一个ECU中实现了多个CAN硬件单元（相同或不同供应商），则Can模块需要确保函数名称和的全局变量的名称不能重复。

命名约定如下：

<**Can module name**>*<**vendorID**>*<**Vendor specific API name**><**driver abbreviation**>()

如果必须在一个ECU上支持多种不同的CAN控制器类型，则必须使用以上的命名约定。如果只使用一种控制器类型，则没有任何<**driver abbreviation**>扩展的原始命名约定已经足够了。

## 8.2. Can驱动模块的状态机

Can模块只有一个非常简单的状态机，包括两个状态：CAN_UNINIT和CAN_READY。图7.1显示了状态机。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhra58ibUqdbL6ADthdjm7KSG5MvrYibmiaVzxgCMRSsibCEoDceKsvib0ajH28b2LW5vjFLK3bsI9TLQPg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

1. 上电/复位后，Can模块应处于CAN_UNINIT状态。
2. Can_Init函数需在初始化硬件单元内的所有控制器之后，将Can模块更改为CAN_READY状态。
3. Can_DeInit函数需在取消初始化硬件单元内的所有控制器之前，将Can模块更改为 CAN_UNINIT状态。

Can_Init函数应根据其配置初始化所有CAN控制器。然后必须通过调用函数 **Can_SetControllerMode(CAN_CS_STARTED)**单独启动每个CAN控制器。

影响硬件单元内所有CAN控制器的硬件寄存器设置只能在函数Can_Init中设置。ECU状态管理器（EcuM）模块在运行时只能调用一次Can_Init函数。

## 8.3. CAN控制器状态机

每个CAN控制器都有以硬件实现的复杂状态机。为简化起见，本文中将状态的数量减少为以下四种基本状态：未初始化（UNINIT）、停止（STOPPED）、启动（STARTED）和睡眠（SLEEP）。

任何CAN硬件访问都被Can模块的功能封装，但Can模块并不记忆状态的变化。Can模块提供Can_Init、Can_SetBaudrate和Can_SetControllerMode等服务。这些服务执行必要的寄存器设置，最终会导致硬件CAN控制器状态发生所需的更改。

同时外部事件（external event）也能触发状态改变，这里有两种可能的外部事件：

- Bus-off事件。
- 硬件唤醒事件。

以上的事件可以通过中断或者在Can_MainFunction_BusOff或 Can_MainFunction_Wakeup函数中轮询的状态位实习。

Can模块需要进行必要的寄存器设置，以实现所需的行为（如：在Bus-Off的情况下无硬件恢复）。然后Can模块通过调用相应的回调函数来通知CanIf模块，相关的软件状态也会在此回调函数中被更改。

如果开发错误被启用并且上层请求不允许转换，Can模块将产生开发错误CAN_E_TRANSITION。但Can模块在执行Can_Write或调用回调函数之前，则不会检查实际状态。

### 8.3.1. CAN控制器状态描述

CAN控制器未初始化状态（UNINIT）

在这种状态下，CAN控制器还未初始化。属于CAN模块的所有寄存器都处于复位状态，CAN中断被禁用，CAN控制器未参与CAN总线活动。

CAN控制器停止状态（STOPPED）

在这种状态下，CAN控制器被初始化但不参与总线活动。此外也不会发送错误帧和确认帧。对于许多进入初始化模式（Entering an initialization-mode）的Can控制器，会导致CAN控制器进入停止状态。

CAN控制器启动状态（STARTED）

在这种状态下，CAN控制器已经处于具有完整功能的正常运行模式，也就是意味着它参与了网络。对于许多离开初始化模式（Leaving the initialization-mode）的控制器，会导致CAN控制器进入启动状态。

CAN控制器睡眠状态（SLEEP）

对于支持睡眠模式的CAN硬件，只有硬件设置与的STOPPED状态不同，即能通过CAN硬件直接支持的CAN总线唤醒。

当CAN硬件支持睡眠模式，并被触发转换为SLEEP状态时，Can模块需要将控制器设置为SLEEP状态，以便于可以通过硬件来唤醒CAN总线。

当CAN硬件不支持睡眠模式，并被触发转换为SLEEP状态时，Can模块应模拟逻辑的SLEEP状态，并且仅当CAN控制器被软件触发转换为STOPPED状态后才从函数返回。CAN硬件应保持在STOPPED状态，而逻辑的SLEEP状态处于活动状态。

### 8.3.2. CAN控制器的状态转换

状态转换由调用软件的Can_SetControllerMode函数触发，并将所需转换状态作为参数。当触发状态成功转换后，回调函数CanIf_ControllerModeIndication会通知上层CanIf模块。监控请求的状态是否完成，应该由上层CanIf模块负责，而不属于Can模块。

一些转换是由总线上的硬件事件触发的，这些转换通过回调函数进行通知。AUTOSAR标准未对无效转换的行为进行定义。图7-2 显示了所有有效的状态转换。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhra58ibUqdbL6ADthdjm7KSGFwZRM1WOILiakoZXpEcvXZ2gmWPbsHcAJ49M6dy9eg3sMsNT8KgwCGQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### 8.3.3. Can_Init函数引起的状态转换

1. 状态的变化：UNINIT -> STOPPED（适用于硬件单元中的所有控制器）。
2. 由软件调用Can_Init函数触发。
3. 对硬件单元内的所有CAN控制器进行配置，所有CAN控制器寄存器根据静态配置进行设置。

Can_Init函数应将所有CAN控制器设置为STOPPED状态。当进入Can_Init函数时，Can模块不在CAN_UNINIT状态或CAN控制器不在UNINIT状态时，将触发CAN_E_TRANSITION的错误。

### 8.3.4. Can_SetBaudrate函数引起的状态转换

1. 状态的变化：STOPPED -> STOPPED; SLEEP -> SLEEP; STARTED -> STARTED
2. 由软件调用Can_SetBaudrate函数触发
3. 更改CAN控制器配置，所有CAN控制器寄存器根据静态配置进行设置。

如果调用Can_SetBaudrate函数后，将导致CAN控制器重新初始化，并且CAN控制器未处于STOPPED状态，则函数应返回E_NOT_OK。

如果需要重新初始化，Can_SetBaudrate函数需要将CAN控制器保持在STOPPED状态；如果需要重新初始化， Can_SetBaudrate函数也需要确保不会设置任何会导致CAN控制器参与网络的设置。

### 8.3.5. Can_SetControllerMode函数引起的状态转换

软件可以使用Can_SetControllerMode函数来触发CAN控制器状态转换。依赖于CAN硬件，寄存器设置更改后，转换到新的CAN 控制器状态，可能会存在时间的延迟。Can模块在状态转换成功后，通知上层模块（CanIf_ControllerModeIndication）新状态。监控请求的状态是否完成，应该由上层CanIf模块负责，而不属于Can模块。

Can_Mainfunction_Mode函数将轮询CAN状态寄存器的标志，直到该标志发出更改生效的信号。然后通过调用 CanIf_ControllerModeIndication函数通知上层模块，由CanIf模块的ControllerId引用的相应CAN控制器状态转换成功。

Can_SetControllerMode函数需要使用系统服务GetCounterValue，来进行超时监控以避免函数阻塞。如果表明状态更改标志一直没有生效，并且CanTimeoutDuration参数定义的最大时间已过，则应退出Can_SetControllerMode函数，但Can_Mainfunction_Mode应继续轮询状态更改标志。

Can_Mainfunction_Mode函数需要调用CanIf_ControllerModeIndication函数通知上层模块，由CanIf模块的ControllerId引用的相应CAN控制器状态转换成功，以防状态转换由Can_SetControllerMode函数再次触发。

#### 8.3.5.1. Can_SetControllerMode(CAN_CS_STARTED)

1. 状态的变化：STOPPED -> STARTED
2. 软件触发

Can_SetControllerMode(CAN_CS_STARTED) 需要通过设置硬件寄存器，以使CAN控制器能参与网络。

Can_SetControllerMode(CAN_CS_STARTED) 需要等待一段有限的时间，已确保CAN控制器已经完全工作。在CAN控制器运行之前发起的传输请求会丢失。CAN控制器可操作性的唯一指标是接收到TX确认或RX指示。发送实体可能会收到确认超时，并且需要能够应对这种情况。

当Can_SetControllerMode(CAN_CS_STARTED) 被调用时，如果CAN控制器未处于STOPPED状态时，这需要被检测为一个无效的状态转换。

#### 8.3.5.2. Can_SetControllerMode(CAN_CS_STOPPED)

1. 状态的变化：STARTED -> STOPPED; SLEEP -> STOPPED
2. 由软件触发

Can_SetControllerMode(CAN_CS_STOPPED) 需要设置CAN硬件内部的配置，以使CAN控制器停止参与网络。Can_SetControllerMode(CAN_CS_STOPPED) 需要等待一段有限的时间，确保CAN控制器真正关闭，并且CAN控制器已经处于STOPPED状态。

如果CAN硬件不支持睡眠模式，则从SLEEP到STOPPED的转换，应从逻辑睡眠模式退出，但对CAN控制器状态其实没有影响，因为控制器在逻辑睡眠模式时，已经处于停止状态。

Can_SetControllerMode(CAN_CS_STOPPED) 应取消所有的pending的消息。

#### 8.3.5.3. Can_SetControllerMode(CAN_CS_SLEEP)

1. 状态的变化：STOPPED -> SLEEP
2. 软件触发

Can_SetControllerMode(CAN_CS_SLEEP) 应将控制器设置为睡眠模式。

如果CAN硬件支持睡眠模式，Can_SetControllerMode(CAN_CS_SLEEP) 将等待有限的时间，直到CAN控制器处于SLEEP状态，并确保CAN硬件能够被唤醒。如果CAN硬件不支持睡眠模式，Can_SetControllerMode(CAN_CS_SLEEP) 应将CAN控制器设置为逻辑睡眠模式。仅当调用 Can_SetControllerMode(CAN_CS_STOPPED) 时，CAN控制器才需要离开此逻辑睡眠模式。

当Can_SetControllerMode(CAN_CS_SLEEP) 被调用，但CAN控制器既不在STOPPED状态也不处于SLEEP状态时，它应被检测为无效的状态转换。

### 8.3.6. Can_DeInit函数引起的状态转换

1. 状态的变化：STOPPED -> UNINIT; SLEEP -> UNINIT（适用于硬件单元中的所有控制器）。
2. 由软件调用Can_DeInit函数触发。
3. 准备重新配置硬件单元内的所有CAN控制器。

Can_DeInit函数需要将所有CAN控制器设置为UNINIT状态。当Can_DeInit函数被调用，如果Can模块不在CAN_READY状态或者任何CAN控制器处于STARTED状态时，需要引发CAN_E_TRANSITION的错误。

### 8.3.7. 硬件事件引起的状态转换

#### 8.3.7.1. 硬件唤醒（由CAN总线唤醒事件触发）

1. 状态的变化：SLEEP -> STOPPED。
2. 由接收到的L-PDU触发。
3. 通过EcuM_CheckWakeup函数通知EcuM模块。

这种状态转换只会发生在硬件支持睡眠模式的情况下。

在硬件唤醒（由CAN总线的唤醒事件触发）时，CAN控制器应转换为STOPPED状态，Can模块应在中断函数或者Can_MainFunction_Wakeup 中调用函数EcuM_CheckWakeup。Can模块不应进一步处理引起唤醒的L-PDU。

在睡眠转换期间，CAN总线唤醒的情况下，函数 Can_SetControllerMode(CAN_CS_STOPPED) 应返回 E_NOT_OK。

#### 8.3.7.2. Bus-Off（由CAN控制器的状态变化触发）

1. 状态的变化：STARTED -> STOPPED。
2. 当CAN控制器达到Bus-Off状态，由硬件触发。
3. 当转换为STOPPED状态后，通过CanIf_ControllerBusOff函数通知CanIf模块，通过CanIf模块的ControllerId引用的相应CAN控制器通知状态已发生变化。

检测到Bus-Off后，CAN控制器应转换为STOPPED状态，Can模块需要确保CAN控制器不再参与网络，同时取消仍待处理（pending）的消息。

Can模块禁用或抑制自动的Bus-Off恢复。

## 8.4. Can模块/控制器初始化

ECU状态管理器模块（EcuM）应在启动阶段通过调用Can_Init函数来初始化Can模块，之后Can模块的其他接口函数才被允许被调用。

Can_Init函数应初始化：

- 静态变量，包括所有标志。
- 完整CAN硬件单元的通用设置。
- 每个CAN控制器的特定设置。

但是Can_Init函数不允许更改任何CAN控制器未使用硬件资源的寄存器。

Can模块需要遵守以下有关控制器寄存器初始化的规则：

- 如果硬件只允许寄存器的一种用途，则实现该功能的Can模块负责初始化寄存器。
- 如果寄存器可以影响多个硬件模块，并且如果它是一个I/O寄存器，它应该由PORT驱动程序初始化。
- 如果寄存器可以影响多个硬件模块，并且如果它不是一个I/O寄存器，则应由MCU驱动程序初始化。
- 一次性可写寄存器需要在复位后直接初始化，应由启动代码初始化。
- 所有其他寄存器都应由启动代码初始化。

如果Can_SetBaudrate确定目标配置需要重新初始化，并且CAN控制器也处于STOPPED状态，则Can_SetBaudrate函数需要重新初始化CAN控制器并进行控制器的特定设置。如果重新初始化是必须，则CAN控制器必须切换到STOPPED状态，然后才能执行 Can_SetBaudrate来应用新的波特率配置。

Can_SetBaudrate函数只影响包含单个CAN控制器的特定配置的寄存器区域。

Can模块配置定义全局CAN硬件单元设置和对默认CAN控制器配置集的引用。

## 8.5. L-PDU的传输

在L-PDU传输时，Can模块将L-PDU内容，ID和数据长度转换为硬件特定格式（如果需要），然后触发传输。

CAN到存储器的数据映射定义为先发出的CAN数据字节为数组第0个元素，最后发出的CAN数据字节为数组第7个元素或者为CAN FD的数据第63个元素。

如果CAN硬件缓冲区内的表示与AUTOSAR定义不同，则Can模块必须为上层提供经过调整的 SDU-Buffer。

可以配置多个具有唯一HTH的TX硬件对象。CanIf模块提供HTH作为TX请求的参数。有关可能的配置，请参见图 7-3。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhra58ibUqdbL6ADthdjm7KSG0iaj12etGk5HDypskab3jxP1c73Q4lInCGTNd297o1d8cWcFZ4D1DJg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Can_Write函数需要一直保存参数PduInfo中给出的swPduHandle，直到Can模块为此发送请求，调用了CanIf_TxConfirmation，其中swPduHandle作为参数给出。此特性用于CanIf模块实现中减少搜索的时间。

Can模块应调用CanIf_TxConfirmation以指示成功传输。在轮询模式的情况下，它也可以在相应硬件资源的TX中断服务程序中（TX-interrupt service routine）调用或在Can_MainFunction_Write函数内部调用。

### 8.5.1. 优先级反转

多路复用传输（Multiplexed transmission）可用于避免外部/内部优先级反转。同时Can模块允许多路复用传输功能在预编译时可静态配置使能或者禁止（ON | OFF）。

多个传输硬件对象（由CanHwObjectCount定义）可以由一个HTH分配，并且向上层模块表现为一个传输实体。

Can模块需要支持设备的多路传输机制：

- 多个发送硬件对象被分组到一个发送实体，并且可以填充在同一个寄存器集，同时微控制器将L-PDU自动存储到一个空闲缓冲区中。
- 硬件提供寄存器或函数来识别传输实体内的自由传输硬件对象。

Can模块需要支持按L-PDU优先级顺序发送L-PDU的设备的多路传输。

注意：

按优先级排序L-PDU避免了分配给配置为多路传输的Basic-CAN的L-PDU的内部优先级反转问题。避免内部优先级反转的另一种可能性是将所有HTH配置为Full-CAN，如果CAN 硬件能够使用CAN ID或相关优先级字段在传输时进行优先级传输。

应避免优先处理的软件仿真，因为额外的开销会使多路传输的优势无效。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhra58ibUqdbL6ADthdjm7KSGWxqJbgC34X6VdoDtgKQCuxleWn1pxfiaichZZhjqwXMF6CPBVU22x6Kw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### 8.5.2. 传输数据一致性

Can模块需要直接从上层模块缓冲区复制数据。上层模块负责保持缓冲区一致，直到Can_Write函数被调用后返回。

## 8.6. L-PDU的接收

当接收到L-PDU时，Can模块应调用RX指示回调函数CanIf_RxIndication，其中包含了 ID、HOH，Mailbox参数中包含抽象的CanIf的ControllerId，以及参数PduInfoPtr中的数据长度和指向L-SDU缓冲区的指针。

在扩展CAN帧的情况下，Can模块应将ID转换为标准化格式，因为上层CAN IF模块不知道接收到的CAN帧是标准CAN帧（Standard CAN frame）还是扩展CAN帧（Extended CAN frame）。如果是扩展CAN帧，需要将接收到的CAN帧ID的MSB设为1，以将接收到的 CAN帧标记为扩展。

在相应硬件资源的RX中断服务程序（RX-interrupt service routine）或者轮询模式下的Can_MainFunction_Read函数需要调用回调函数CanIf_RxIndication。

CAN到内存的数据映射定义和数据传输一样，首先接收的CAN数据字节为数组第0个元素，最后接收的CAN数据字节为数组元素第7个或者CAN FD的第63个元素。如果CAN硬件缓冲区内的表示与AUTOSAR定义不同，则Can模块必须为上层模块提供经过调整的SDU-Buffer。

Can驱动需要指示接收到的消息是传统CAN帧（Conventional CAN frame）还是CAN FD帧（CAN FD frame），如Can_IdType中定义，通过使用两个最高有效位来指定帧类型：

- -00 带有标准CAN ID的传统CAN帧
- -01 带有标准CAN ID的CAN FD帧
- -10 带有扩展CAN ID的传统CAN帧
- -11 带有扩展CAN ID的CAN FD帧

### 8.6.1. 接收数据一致性

为了防止接收到消息的丢失，一些控制器支持为一组硬件对象（hardware object）构建一个先进先出队列（FIFO）。然而在其他一控制器上，可以配置另一个具有相同属性的硬件对象，在主对象出现忙碌的介入用作影子缓冲区（shadow buffer）。

CAN驱动程序需要支持实现了硬件FIFO的控制器。FIFO的大小通过CanHwObjectCount配置。

不支持硬件FIFO的控制器通常提供了实现影子缓冲区机制的能力。当主要硬件对象忙时，其他硬件对象会接管。硬件对象的数量也是通过CanHwObjectCount来进行配置。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhra58ibUqdbL6ADthdjm7KSGCFQW1gibR629t80uDOfniadRWcjLibjD6DlVUXWrY0XT5xgHiawibRptBUw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如果CAN硬件无法保护（锁定）RX缓冲区，以防止数据被新接收到的消息所覆盖。Can模块需要在接收后，将L-SDU复制到影子缓冲区中。

如果CAN硬件不可全局访问，Can模块也需要将L-SDU复制到影子缓冲区中。完整的RX处理（包括复制到目标层，例如：COM）需要在RX中断程序的上下文或者 Can_MainFunction_Read函数的上下文中完成。

Can模块应保证ISR和Can_MainFunction_Read函数都不能被自己中断。这样CAN硬件（或影子）缓冲区始终是一致的，因为它是在一个永远不会被自身中断的函数中按顺序写入和读取的。

如果CAN硬件无法配置为在接收后锁定RX硬件对象（硬件功能），则可能会发生硬件缓冲区被新到达的消息覆盖的情况。在这种情况下，如果硬件支持，CAN控制器会检测到覆盖（overwrite）事件。

如果CAN硬件可以配置为在接收后锁定RX硬件对象，则可能会发生新到达的消息无法存储到硬件缓冲区的情况。在这种情况下，如果硬件支持，CAN控制器会检测到溢出（overrun）事件。

如果检测到“覆盖”或“溢出”事件，Can模块需要引发运行时CAN_E_DATALOST错误。系统设计者应确保消息接收（中断驱动或轮询）的运行时间与系统中可能的快速接收相匹配。

## 8.7. 唤醒概念

Can模块处理可由Can控制器本身检测到的唤醒，而不是通过Can收发器（Can transceiver）检测到的唤醒。有两种可能的情况：中断唤醒（wakeup by interrupt）和轮询唤醒（wakeup by polling）。对于中断唤醒，当硬件检测到唤醒时调用Can模块的ISR。

如果唤醒事件的ISR被调用，它应该依次调用EcuM_CheckWakeup。传递给EcuM_CheckWakeup的参数应该是CanWakeupSourceRef配置参数引用的唤醒源的ID。然后，ECU状态管理器将设置MCU，并通过Can接口（CanIf）模块再调用Can模块，从而调用 Can_CheckWakeup函数。

当通过轮询检测到唤醒事件时，ECU状态管理器会像以前一样通过Can接口（CanIf）模块周期性地调用Can_CheckWakeup函数。

在这两种情况下，Can_CheckWakeup都会检查Can控制器是否检测到唤醒并返回结果。然后，CAN驱动程序将通过EcuM_SetWakeupEvent通知ECU状态管理器（EcuM）收到唤醒事件（wakeup event）。

唤醒验证主要用于防止错误唤醒事件，它主要由ECU状态管理器（EcuM）和Can接口（CanIf）模块完成，无需Can模块的任何帮助。

有关唤醒机制和唤醒序列图的一般描述，请参阅ECU状态管理器规范[7]。

## 8.8. 通知概念（Notification concept）

Can模块仅向上层CanIf模块提供事件触发通知接口。每个通知都由一个回调函数表示。

硬件事件可以通过中断或轮询硬件对象的状态标志来检测。有关轮询的配置可能性取决于硬件（即：哪些事件可以轮询，哪些事件需要轮询），并且不受AUTOSAR规范的限制。驱动程序同时也需要可以支持配置为完全不使用中断，即完全轮询。

轮不轮询的配置在Can驱动程序内部，在模块外部是不可见。轮询在CAN主函数（Can_MainFunction_xxx）内完成。同时轮询事件也是通过适当的回调函数通知上层模块。只不过回调函数地调用不是在ISR中，而是在CAN模块的主函数中。所有回调函数的实现都和在ISR中的实现相同。更多详细信息，可以参见CAN模块的主函数Can_MainFunction_Read、Can_MainFunction_Write、Can_MainFunction_BusOff和Can_MainFunction_Wakeup的说明。

## 8.9. 重入问题（Reentrancy issues）

例程必须满足以下条件才能可重入：

- 它以原子方式使用所有共享变量，除非每个变量都分配给函数的特定实例。
- 它不调用不可重入函数。
- 它不会以非原子方式使用硬件。

传输请求仅会在CanIf模块的CanIf_Transmit函数内进行转发。由于函数CanIf_Transmit是可重入的，所以Can_Write函数需要实现线程安全（例如：通过使用互斥锁）。

当写入无法重入时，进一步的（抢占式）调用将返回CAN_BUSY。（例如：允许写入不同的硬件TX句柄，不允许写入相同的TX句柄）。在CAN_BUSY的情况下，CanIf模块需要对写入请求进行队列管理。（行为与所有硬件对象都忙的情况相同）。

Can_EnableCanInterrupts和Can_DisableCanInterrupts可能会在可重入函数中被调用，所以这两个函数也是需要是符合可重入设计的。

所有其他服务不需要实现为可重入函数。CAN主函数（如：Can_MainFunction_Read）不允许被自己中断，所以这些CAN主函数都是不可重入的。

## 8.10. 硬件时间戳

如果CAN控制器支持基于硬件的时间戳，Can模块可以使用硬件时间戳。这样可以提高CAN时间同步的精度。

如果支持基于硬件的时间戳，CAN驱动程序需要提供以下API：

- Can_GetCurrentTime
- Can_EnableEgressTimeStamp
- Can_GetEgressTimeStamp
- Can_GetIngressTimeStamp

这些API需要通过配置参数CanGlobalTimeSupport使能。

基于硬件的时间戳功能的CAN控制器，需要提供一个自由运行（free-running）的计数器，用于获取CAN消息接收和发送的时间戳。自由运行计数器需要在达到其指定最大值后，向上计数并溢出到零。CiA603标准中规定自由运行计数器对时钟周期进行计数，分辨率支持至少需要为1μs，最多可以1ns。强烈建议提供32位时间戳寄存器和32位计数器。

当CAN帧被认为有效时，发送和接收的CAN消息的时间戳将被捕获。CiA603标准中给出了详细信息。

## 8.11. CAN FD的支持

出于性能原因，一些CAN控制器允许使用称为CAN FD的灵活数据速率功能。在仲裁阶段表明可以在有效负载和CRC期间切换到更高的波特率。必须通过使用CanControllerFdBaudrateConfig参数对原CanControllerBaudrateConfig进行扩展，实现配置第二个波特率。如果具有CAN FD配置的波特率处于活动状态（请参阅：CanControllerFdBaudrateConfig），则为该控制器启用CAN FD功能。

可以使用第二个波特率来支持比特率切换（bit rate switch）BRS功能，实现CAN FD帧的接收。但第二个波特率是否用于传输，取决于配置参数CanControllerTxBitRateSwitch。

但是，可能也存在需要以下的场景，在支持CAN-FD消息的网络中传输传统CAN 2.0消息。（例如：便于CAN的选择性唤醒。）在这些情况下，有必要支持CAN-FD消息和传统CAN消息交替传输。这问题可以通过在调用Can_Write函数的时候，利用CanId参数的两个最高有效位，以指定使用哪种类型的帧，从而在数据帧的级别上实现交替发送。

CAN FD还支持扩展的有效载荷（Payload），允许传输最多64个字节的数据。此功能也取决于CAN FD配置（请参阅：CanControllerFdBaudrateConfig）。如果CAN控制器处于CAN FD模式（有效的CanControllerFdBaudrateConfig）并且在传递给 Can_Write函数的CanId中设置了CAN FD标志，则Can驱动支持传输长度最大为64字节的PDU。如果有发送CAN FD帧的请求，且CAN控制器未处于CAN FD模式（无CanControllerFdBaudrateConfig），则只要PDU长度<= 8字节，该帧就会作为传统CAN帧发送。

# 9. API规范

## 9.1. 影响整个硬件单元的服务

### 9.1.1. Can_Init

说明: 初始化Can模块。

```
void Can_Init (const Can_ConfigType* Config )
```

### 9.1.2. Can_GetVersionInfo

说明: 返回该模块的版本信息。

```
void Can_GetVersionInfo ( Std_VersionInfoType* versioninfo )
```

### 9.1.3. Can_DeInit

说明: 取消初始化的Can模块。

```
void Can_DeInit ( void )
```

## 9.2. 影响单个 CAN 控制器的服务

### 9.2.1. Can_SetBaudrate

说明: 设置CAN控制器的波特率配置。根据必要的波特率修改，控制器可能必须重置。

```
Std_ReturnType Can_SetBaudrate ( uint8 Controller, uint16 BaudRateConfigID )
```

### 9.2.2. Can_SetControllerMode

说明: 执行CAN控制器状态机的软件触发状态转换。

```
Std_ReturnType Can_SetControllerMode ( 
    uint8 Controller, Can_ControllerStateType Transition )
```

### 9.2.3. Can_DisableControllerInterrupts

说明: 禁用此CAN控制器的所有中断。

```
void Can_DisableControllerInterrupts ( uint8 Controller )
```

### 9.2.4. Can_EnableControllerInterrupts

说明: 启用此CAN控制器所有允许的中断。

```
void Can_EnableControllerInterrupts ( uint8 Controller )
```

### 9.2.5. Can_CheckWakeup

说明: 检查指定的控制器是否发生了唤醒。

```
Std_ReturnType Can_CheckWakeup ( uint8 Controller )
```

### 9.2.6. Can_GetControllerErrorState

说明: 获取CAN控制器的错误状态。

```
Std_ReturnType Can_GetControllerErrorState ( 
    uint8 ControllerId, Can_ErrorStateType* ErrorStatePtr )
```

### 9.2.7. Can_GetControllerMode

说明: 报告所请求的CAN控制器的当前状态。

```
Std_ReturnType Can_GetControllerMode ( 
    uint8 Controller, Can_ControllerStateType* ControllerModePtr )
```

### 9.2.8. Can_GetControllerRxErrorCounter

说明: 返回CAN控制器的Rx错误计数器。此值可能不适用于所有CAN控制器，在这种情况下将返回E_NOT_OK。请注意，在API返回时，计数器的值可能不正确，因为Rx计数器是在硬件中异步处理的。对于当前总线状态的任何假设，应用程序都不应信任此值。

```
Std_ReturnType Can_GetControllerRxErrorCounter ( 
    uint8 ControllerId, uint8* RxErrorCounterPtr )
```

### 9.2.9. Can_GetControllerTxErrorCounter

说明: 返回CAN控制器的Tx错误计数器。此值可能不适用于所有CAN控制器，在这种情况下将返回E_NOT_OK。请注意，在API返回时计数器的值可能不正确，因为Tx计数器是在硬件中异步处理的。对于当前总线状态的任何假设，应用程序不应信任此值。

```
Std_ReturnType Can_GetControllerTxErrorCounter ( 
    uint8 ControllerId, uint8* TxErrorCounterPtr )
```

### 9.2.10. Can_GetCurrentTime

说明: 根据硬件的能力从硬件寄存器中返回一个时间值。重要提示：Can_GetCurrentTime可能需要在专用区域（exclusive area）内调用。

```
Std_ReturnType Can_GetCurrentTime ( 
    uint8 ControllerId, Can_TimeStampType* timeStampPtr )
```

### 9.2.11. Can_EnableEgressTimeStamp

说明: 激活专用HTH上的出口时间戳。一些硬件会存储一次出口时间戳标记，而一些硬件在传输之前总是需要这个值。由于消息类型总是由网络设计时间戳，因此不会有禁用功能。

```
void Can_EnableEgressTimeStamp ( Can_HwHandleType Hth )
```

### 9.2.12. Can_GetEgressTimeStamp

说明: 读回专用消息对象上的出口时间戳。它需要在TxConfirmation函数中调用。

```
Std_ReturnType Can_GetEgressTimeStamp ( 
    PduIdType TxPduId, Can_HwHandleType Hth, Can_TimeStampType* timeStampPtr )
```

### 9.2.13. Can_GetIngressTimeStamp

说明: 读回专用消息对象上的入口时间戳。它需要在RxIndication函数中调用。

```
Std_ReturnType Can_GetIngressTimeStamp ( 
    Can_HwHandleType Hrh, Can_TimeStampType* timeStampPtr )
```