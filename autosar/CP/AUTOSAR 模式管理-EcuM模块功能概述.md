# AUTOSAR 模式管理-EcuM模块功能概述

**前言**

AUTOSAR EcuM模块的分享分为EcuM模块概念详解和EcuM模块配置及代码分析，具体的项目实战请关注本号的后续文章，本篇为**EcuM模****块****概念介绍篇--功能概述**。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyejvRYLlzicd6Kn5zSjKmC94ibicQMswrqxgg6aLDW55VrXaGvIqwHVTnHxkqiazmlfpbnicpL1Ioo7Sg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**正文**

# **1.功能概述**

ECU管理模块是一个基本的软件模块，用于管理ECU状态的公共方面。具体如下：

  . 初始化和反初始化OS，SchM，BswM以及一些基础软件的驱动模块

  . 在请求时将ECU配置为休眠（Sleep）和关机（Shutdown）

  . 管理ECU所有的唤醒事件

 

ECU管理器模块提供了唤醒验证协议来区分'真实的''唤醒事件'和'不稳定的'唤醒事件。

 

ECU management有两种：flexible and fixed

 

相比于Fixed management，Flexible management 使得ECU状态的固定模式和它们之间的转换被消除，以允许以下附加场景：

. 部分或快速启动，其中 ECU 以有限的能力启动，然后由应用程序决定，继续逐步启。

. 交错启动，其中 ECU 启动最少，然后启动 RTE 以尽快执行 SW-C 中的功能。然后它继续启动更多的 BSW 和 SW-C，从而将 BSW 和应用程序功能交织在一起。

. ECU 具有多个运行状态的多种操作状态。

. 多核Ecu：STARTUP、SHUTDOWN、SLEEP 和 WAKEUP 是在ECU的所有核上协调工作。

 

Flexible management采用以下模块提供的通用模式管理工具：

. RTE 和 BSW SchM模块合并为一个模块：该模块支持可自由配置的 BSW 和应用程序模式及其模式切换设施。

. BswM模块：该模块实现可配置的规则和操作列表，以评估切换 ECU 模式的条件并实施必要的操作。

 

因此，通过Flexible management，大多数 ECU 状态不再在 EcuM模块本身中实现。 通常，当通用模式管理工具在以下情况下不可用时，ECU 管理器模块会接管控制：

. STARTUP的第一阶段

. SHUTDOWN的最后阶段

. 设施被调度程序锁定的SLEEP阶段

 

在 ECU 管理器模块的 UP 阶段，BSW 模式管理器负责进一步的操作。而 ECU 管理器模块仲裁来自 SW-C 的 RUN 和 POST_RUN 请求，并通知 BswM 有关模式的状态。

 

RUN 请求协议是 ECU State Manager Fixed 中已建立的方法，用于确定 ECU 应保持活动状态还是准备关闭。

 

固定 ECU 管理以先前 AUTOSAR 版本的形式继续 ECU 管理。它有一组固定的 ECU 状态和它们之间的转换，对于没有特殊要求的传统 ECU 来说已经足够了，例如部分或快速启动、交错启动和多个操作状态（多个 RUN 状态）。固定 ECU 管理不支持多核 ECU，等等。

 

小结：Flexible EcuM和Fixed EcuM的区别与联系

| EcuM | Fixed                                                        | Flexible                                                     |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 区别 | . Fixed EcuM状态机固定，状态跳转唯一. 仅适用于单核启动，不支持多核启动.不支持局部、快速、交叉启动 | . 根据实际续期设计EcuM状态机，EcuM状态不固定。. 支持多核启动. 支持局部、快速、交叉启动 |
| 联系 | Fixed EcuM可以理解为Flexible EcuM的一个具体实例（**EcuM Flexible to Fixed Mapping**） | Flexible EcuM是EcuM模块的高度抽线，可根据实际需求实例化设计。 |

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyejvRYLlzicd6Kn5zSjKmC91sA6dfAloeoTAETFexY9kpq7eA4ntMGMYicq4eIch3icmeaGDc18DkNg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图1：固定 EcuM 到Flexible EcuM 的映射

 

# **2.关键概念理解**

**Callout**: Callout函数是系统设计人员可以实现特定功能的打桩函数，通常在配置时，以向 EcuM模块添加功能。Callout分为两类， 一类提供强制性 EcuM模块功能并用作硬件抽象层。另一个类提供可选功能。

 

**Mode**：模式是车辆中运行的、与特定实体、应用程序或整个车辆相关的各种状态机(不仅仅是ECU管理器)的一组特定状态。

 

**Passive Wakeup**：由连接的总线而不是内部事件（如计时器或传感器活动）引起的唤醒。

 

**Phase**:  ECU管理器的动作和事件（例如启动，关机，休眠...）在逻辑上或者时间上的集合。

 

**Shutdown Target**：ECU 在进入休眠状态、断电或复位前必须关闭。因此，SLEEP、OFF 和 RESET 是有效的关机目标。通过选择关闭目标，应用程序可以在下一次关闭后将其对 ECU 行为的期望状态传达给 ECU 管理器模块。

 

**State**: 状态在它们各自的BSW组件内部，因此对应用程序不可见。所以它们只被BSW的内部状态机使用。ECU管理器内部的状态组成阶段（Phase），并被用于模式处理。

 

**Wakeup Event**：导致唤醒的物理事件。 CAN 消息或切换 IO 线可以是唤醒事件。

 

**Wakeup Reason**：唤醒原因是唤醒事件，即上次唤醒的实际原因。

 

**Wakeup Source**：处理唤醒事件的外设或 ECU 组件称为唤醒源。

 

# **3.EcuM模块对其他模块的依赖**

## **3.1 Mcu模块**

MCU 驱动程序是由 ECU 管理器模块初始化的第一个基本软件模块。然而，当 MCU_Init 返回时，MCU 模块和 MCU 驱动模块不一定完全初始化。可能需要额外的 MCU 模块特定步骤来完成初始化。ECU 管理器模块提供了两个可放置此附加代码的Callout函数。

 

## **3.2 具有唤醒能力外设模块**

唤醒源必须由驱动程序处理和封装。这些驱动程序必须遵循EcuM详细设计文档中提供的协议和要求，以确保无缝集成到 AUTOSAR BSW。基本上，协议如下：

1）驱动程序必须调用 EcuM_SetWakeupEvent 来通知 ECU 管理器模块已检测到挂起的唤醒事件。驱动程序不仅必须在 ECU 在睡眠阶段等待唤醒事件时调用 EcuM_SetWakeupEvent，而且在驱动程序初始化阶段和 EcuM_MainFunction 运行时的正常操作期间也必须调用

 

2）驱动程序必须提供一个显式函数来使唤醒源进入睡眠状态。 该函数将使唤醒源进入低功耗模式并重新装备唤醒通知机制。

 

3）如果唤醒源能够生成虚假事件，则驱动程序或使用驱动程序的软件或其他适当的 BSW 模块必须为唤醒事件提供验证时间有效性的Callout函数或调用 ECU 管理器模块的验证功能。如果不需要验证，则此要求不适用于相应的唤醒源。

 

## **3.3 操作系统OS**

ECU 管理器模块启动 AUTOSAR 操作系统并关闭它。ECU 管理器模块定义了在操作系统启动之前如何处理控制以及在操作系统关闭之后如何处理控制的协议。

 

## **3.4** **BSW调度器**

ECU 管理器模块初始化BSW调度器，ECU 管理器模块还包含 EcuM_MainFunction，它被BSW Scheduler周期调度，评估唤醒请求并更新Alarm Clock。

 

## **3.5** **BswM模块**

ECU 状态一般实现为 AUTOSAR 模式，BSW 模式管理器负责监控 ECU 中的变化并影响 ECU 状态机的相应变化。

 

BSW 模式管理器只能在模式管理运行后管理 ECU 状态机 - 即在 SchM 初始化之后，直到 SchM 被取消初始化或停止。当 BSW 模式管理器无法运行时，ECU 管理器模块会控制 ECU。

 

因此，ECU 管理器模块在 ECU 启动后立即取得控制权，并在初始化 SchM 和 BswM 后将控制权交给 BSW 模式管理器。

 

BswM 将 ECU 的控制权传递回 ECU 管理器模块以锁定操作系统并处理唤醒事件。BswM 还会在 OS 在关机时停止之前立即将控制权传递回 ECU 管理器。当唤醒源被验证时，ECU 管理器模块通过模式切换请求向 BswM 指示唤醒源状态变化。

 

## **3.6 应用软件组件SWC**

ECU 管理器模块处理以下 ECU 范围的属性：Shutdown target。SWC可通过AUTOSAR端口可以设置Shutdown target。

 

## **3.7 小结**

上文中我们提到Fixed EcuM是Flexible EcuM的具体实现。所以我们在使用Flexible EcuM的时候可以设计一个软件组件SWC来监控ECU的状态，通过配置BswM进行模块管理，三个模块（SWC，BswM，EcuM）协同工作，最终也会Fixed一个EcuM。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyejvRYLlzicd6Kn5zSjKmC9Pmvm12xKbHh6rphqH8QLdOLLAZ5dNrx0naNbtXHCC7qFdFTfc3FZRQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图2：SWC、EcuM及BswM三方交互

 

# **4.EcuM模块的各个阶段介绍**

*Note: 未特殊说明，以下描述都只针对F**lexible* *EcuM。*

Flexible EcuM没有固定的状态或者模式，EcuM的设计者必须决定需要哪些状态并配置它们。

 

当使用 ECU 模式处理时，标准状态 RUN 和 POST_RUN 由 RUN 请求协议仲裁并传播到 BswM。系统设计者在通过 BswM 操作设置 EcuM 模式时必须确保满足各个状态的前提条件。

## **4.1 STARTUP阶段**

STARTUP 阶段的目的是将基础软件模块初始化到通用模式管理工具可操作的点。

 

## **4.2 UP Phase阶段**

本质上，当 BSW 调度程序启动并调用BswM_Init时，UP 阶段开始。那时，内存管理没有初始化，没有通信栈，没有 SW-C 支持 (RTE) 并且 SW-C 还没有启动。以特定模式（Startup后配置的下一个模式）开始，并具有相应的可运行对象，即 BSW MainFunctions。BswM模式仲裁开始工作，然后BswM的模式控制的Actions能够触发和禁用SWC。

 

然而，从 ECU 管理器模块的角度来看，ECU 处于“UP”状态。然后 BSW 模式管理器模块启动模式仲裁，所有进一步的 BSW 初始化、启动 RTE 和（隐式）启动 SW-C 成为在 BswM 的动作列表中执行的代码或由依赖于模式的调度驱动，这些都是可以在BswM的设计阶段又集成者（integrator设计控制的）。

 

因此，初始化 NvM 并调用 NvM_Readall 也成为集成代码。这意味着继承者（integrator）负责在 NvM_ReadAll 结束时触发 Com、DEM 和 FIM 的初始化。当 NvM_ReadAll 完成时，NvM 将通知 BswM。

 

注意 RTE 可以在 NvM 和 COM 初始化后启动。另请注意，在初始化 COM 之前不需要完全初始化通信堆栈。

 

这些更改会以任意顺序初始化 BSW 模块以及启动 SW-C，直到 ECU 达到满负荷为止，并且此后这些更改还将继续确定 ECU 的功能。

 

最终模式开关会停止 SW-C 并取消初始化 BSW，以便当 ECU 达到可以关闭电源的状态时，Up 阶段结束。

 

因此，就 ECU 管理器模块而言，BSW 和 SW-C 会一直运行，直到它们准备好让 ECU 关闭或进入睡眠状态。

 

## **4.3 SHUTDOWN阶段**

SHUTDOWN 阶段处理基础软件模块的受控关闭，最终导致选定的关闭目标 OFF 或 RESET。

 

## **4.4 SLEEP阶段**

ECU在SLEEP阶段进入低功耗阶段。通常情况下，不执行任何代码，但电源仍然提供，如果进行相应的配置，ECU在这种状态下是可唤醒的。ECU管理器模块提供了一组可配置的(硬件)睡眠模式，通常是在功耗和重启ECU的时间之间进行权衡。

 

ECU 管理器模块唤醒 ECU，以响应有意或无意的唤醒事件。由于意外唤醒事件应该被忽略，ECU 管理器模块提供了一个协议来验证唤醒事件。该协议指定了处理唤醒源的驱动程序和 ECU 管理器之间的协作过程。

 

## **4.5 OFF阶段**

ECU 在断电时进入OFF 状态。在这种状态下，ECU 可能是可唤醒的，但仅适用于具有SBC（System Base Chip）电源控制的唤醒源。 在任何情况下，ECU 都必须是可启动的（例如通过重置事件）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyejvRYLlzicd6Kn5zSjKmC9nqtQW3UYtbSc3Eb32a1TTZzRzyFnNqDicsAZibWdUnB3de96uYHp5x4w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图4：EcuM的各个阶段

 

# **5.** **ECU Manager的结构描述**

图4说明了ECU Manager模块与其他BSW模块接口的关系。在大多数情况下，ECU管理器模块只负责初始化。然而，有一些模块与ECU管理器模块有功能关系，这将在后续段落中解释。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyejvRYLlzicd6Kn5zSjKmC9KZPsOKVXZ6oADK0iavn5iciaaEcWpPu4X9Y5SXmNhYcDFfjm5hOytDFRQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图5：EcuM模块需要使用的其他模块的接口

 

一些基本的软件驱动模块在EcuM模块唤醒时被初始化、关闭和重新初始化。操作系统由EcuM初始化并关闭。在操作系统初始化之后，EcuM模块再将控制权传递给BswM之前执行额外的初始化步骤。在操作系统关闭之前，BswM立即将执行控制交还给EcuM模块。SW-C 使用 AUTOSAR 端口与 ECU 管理器模块交互。



*Note:未完待续，下一篇--EcuM Start Sequence介绍*