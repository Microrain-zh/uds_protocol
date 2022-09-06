# AUTOSAR BswM（BSW模式管理器）

# 1. 简介和功能概述

本文档介绍了**AUTOSAR**基础软件模块**BSW**模式管理器（**BswM**）的功能、API 和配置。

**BSW**模式管理器是实现驻留在**BSW**中的部分车辆模式管理和应用程序模式管理概念的模块。它的职责是根据简单的规则对来自应用层**SW-C**或其他BSW模块的模式请求进行仲裁，并根据仲裁结果执行动作。

# 2. 缩略语

**BSW**

> 基础软件（**Basic Software**）

**BswM**

> 基础软件模式管理器（**BSW Mode Manager**）

**BSWMD**

> 基础软件模块说明（**Basic Software Module Description**）

**CDD**

> 复杂驱动程序（**Complex Driver**）

**Dem**

> 诊断事件管理器（**Diagnostic Event Manager**）

**Det**

> 默认错误跟踪器（**Default Error Tracer**）

**ECU**

> 电子控制单元（**Electronic Control Unit**）

**SWC/SW-C**

> 软件组件（**Software Component**）

**SWCD**

> 软件组件说明（**Software Component Description**）

# 3. 相关文档

## 3.1. 输入文件及相关标准和规范

[1] General Specification of Basic Software Modules

> AUTOSAR_SWS_BSWGeneral

[2] Guide to Mode Management

> AUTOSAR_EXP_ModeManagementGuide

[3] General Requirements on Basic Software Modules

> AUTOSAR_SRS_BSWGeneral

[4] Requirements on Mode Management

> AUTOSAR_SRS_ModeManagement

[5] Specification of Basic Software Mode Manager

> AUTOSAR_SWS_BSWModeManager

[6] Specification of RTE Software

> AUTOSAR_SWS_RTE

## 3.2. 相关规范

因为AUTOSAR提供了基础软件模块的通用规范（参见相关文档[1]），它也适用于**BSW**模式管理器。所以文档[1]应被视为**BSW**模式管理器的附加和必需规范。

有关**BSW**模式管理器的配置和使用的信息可在辅助文档中找到：模式管理指南（**Guide to Mode Management**）参见相关文档[2]。

# 4. 约束和假设

## 4.1. 限制

在一个分区（**Partition**）内最多可以使用一个**BSW**模式管理器实例。

## 4.2. 适用于汽车领域

**BSW**模式管理器适用于所有汽车域。

# 5. 对其他模块的依赖

**BSW**模式管理器具有与**AUTOSAR**体系结构中的许多**BSW**模块的接口。然而，这些接口中的大多数都是可选的，并且根据每个**ECU**的需要使用。

本章中列出的依赖项旨在概述**BswM**和其他模块之间的一些可能的交互。此处列出的交互和模块不应被视为所有可能性的详尽列表。

## 5.1. RTE

**BswM**通过**RTE**接收来自**SW-C**的模式请求。模式切换通知也通过**RTE**传播到**SW-C**。

## 5.2. EcuM Flex

**EcuM Flex**可以向**BswM**指示其唤醒源的状态。当使用**ECU**模式处理时，**BswM**可以设置**EcuM Flex**的状态，并根据**RUN**请求协议接收某些模式的状态。

## 5.3. WdgM

（此节内容已过时）。

**WdgM**可以通过**BswM_WdgM_RequestPartitionReset API**向**BswM**请求分区重置相关操作。**WdgM**分区重置请求的配置是通过**BswMWdgMRequestPartitionReset**模式请求源完成的。

## 5.4. ComM

来自**ComM**的模式切换指示（**Mode Switch Indications**）可以通过**BswM**进一步传播到**SW-C**。

**BswM**可以通过**ComMUsers**向**ComM**请求通信模式。

## 5.5. COM

**COM**中I-PDU组（**I-PDU Group**）的处理由**BswM**执行。作为I-PDU组启动/停止的一部分，**BswM**可以将包含的信号值重置为其相应的初始化值。

同时**BswM**处理**COM**中信号截止期限监视（**deadline monitoring**）的启用和禁用。**BswM**也可以触发**I-PDU**的传输。

## 5.6. PduR

**BswM**可以启用和禁用**PDU Router**的**I-PDU**路由组的路由功能。

## 5.7. CanSM

来自**CanSM**的模式切换指示可以通过**BswM**进一步传播到**SW-C**。

## 5.8. LinSM

**BswM**协调**LinSM**中LIN调度表（**LIN Schedule Tables**）的切换，和**COM**中相应I-PDU组的启动和停止。

来自**LinSM**的模式切换指示可以通过**BswM**进一步传播到**SW-C**。

## 5.9. LinTP

LIN传输协议（**LIN Transport Protocol**）作为**LinIf**一部分的，会向**BswM**请求模式，以确保在**LinTp**操作期间，正确的LIN调度表（**LIN Schedule Table**）处于活动状态。

## 5.10. FrSM

来自**FrSM**的模式切换指示可以通过**BswM**进一步传播到**SW-C**。

**FlexRay**单槽模式（**Single Slot Mode**）的使用，由**BswM**的请求，**FrSM**控制。FlexRay软件栈的发送能力可以由**BswM**通过**FrSM**调用**FrSM_SetEcuPassive API**来控制。

## 5.11. EthSM

来自**EthSM**的模式切换指示可以通过**BswM**进一步传播到**SW-C**。

## 5.12. DCM

**DCM**模块会根据所接收到的诊断请求，向**BswM**执行相应地模式请求。

示例：**DCM**可以发送禁止正常通信（**Disable Normal Communication**）请求。在此模式下，**BswM**将关闭相应的**I-PDU**组和**NM PDU**。

## 5.13. J1939Dcm

**J1939Dcm**会向**BswM**报告通信状态变化，以便再进一步把状态传播到**SW-C**。**BswM**可以通过**J1939Dcm_SetState**改变**J1939Dcm**的状态。

## 5.14. J1939Nm

**J1939Nm**可以通过**BswM_J1939Nm_StateChangeNotification**提供状态指示（**state indication**）。

## 5.15. J1939Rm

**BswM**通过**J1939Rm_SetState**可以改变**J1939Rm**的状态。

## 5.16. NM Interface

**BswM**将使用**Nm_EnableCommunication**和**Nm_DisableCommunication**来控制基于当前模式的**NM**通信。

示例：在禁用正常通信（**Disable Normal Communication**）模式下，**BswM**需要禁止相应网络管理通道上的网络管理通信。

**Nm**模块可以使用**BswM_Nm_CarWakeUpIndication**来指示整车被唤醒。

## 5.17. NvM

**NvM**模块通过NvM回调函数的集成代码（**integration code**）向**BswM**报告NvM数据块的状态。**BswM**具有NvM在启动和关闭期间，读取和写入所有数据块的操作。

## 5.18. OS

**BswM**所需的操作系统功能是需特定于实现的。

## 5.19. Sd

**BswM**通过以下得API从**Sd**模块接收状态指示。

- **BswM_Sd_ClientServiceCurrentState**
- **BswM_Sd_ConsumedEventGroupCurrentState**
- **BswM_Sd_EventHandlerCurrentState**

这些来自**Sd**的状态指示，可以配置为**BswMModeRequestSources**。

## 5.20. File structure

**BswM**可以使用未在本规范中明确定义的**AUTOSAR BSW**模块中的接口。

# 6. 功能规格

**BSW**模式管理器得基本功能的操作，可以描述为两个不同的任务：

- 模式仲裁（**Mode Arbitration**）
- 模式控制（**Mode Control**）

模式仲裁部分对**SW-C**或其他**BSW**模块接收到的模式请求和模式指示进行基于规则的仲裁，从而启动模式的切换。

模式控制部分则通过执行包含其他**BSW**模块的模式切换操作的动作列表，来执行模式切换。

**BswM**需被视为一个模式管理框架模块，其中行为完全通过相关配置定义。

可能可以通过不同的方式来实现**BswM**，例如：基于配置生成完整的**BswM**，或者作为在运行时解析编码配置的规则解释器。但是AUTOSAR的标准并不打算指定**BSW**模式管理器的任何实现细节。所以**AUTOSAR**标准文档中所有描述设计细节的任何示例都应视为解释性文本而不是需求。

## 6.1. 模式仲裁

**BswM**执行的模式仲裁简单且基于规则（**rule-based**）。用于模式仲裁的规则，可以通过**BSW**模式管理器模块的配置来指定。

因为这些规则由简单的布尔表达式组成，所以模式仲裁对运行时的影响很小。

为了知晓需执行哪些动作列表，**BswM**模块需要从先前的规则评估的模式仲裁结果检测出变化。此功能的如何完成，以及存储结果所需的内存结构如何，都是特定于实现的，在**AUTOSAR**标准文档中并不会进行描述定义。

### 6.1.1. 仲裁规则

规则是由一组模式请求条件（**mode request conditions**）组成的逻辑表达式（**logical expression**）。

规则评估会在以下两个场景进行：

1. 当输入模式请求（**mode requests**）和模式指示（**mode indications**）改变时
2. 在**BswM**模块主函数的执行时。

评估的结果（真或假）可用于决定相应模式控制动作列表（**mode control action list**）的执行。

### 6.1.2. 模式条件和逻辑表达式

构成模式仲裁规则的逻辑表达式可以使用不同的运算符，包括：**AND**、**OR**、**XOR**、**NOT**和**NAND**。表达式中的每一项对应一个模式请求条件（**mode request condition**）。

如果模式条件（**mode condition**）关联了一个**BswMModeRequestPort**，则该条件会验证请求的模式（**requested mode**）或者指示的模式（**indicated mode**）是否**EQUAL**或者**NOT_EQUAL**于某个模式。

如果条件关联了一个**BswMEventRequestPort**，则条件将验证请求端口是**SET**还是**CLEAR**了。

**BswMEventRequestPort**事件请求与模式请求的不同，在于请求者不向**BswM**发送请求的模式/值，也就没有模式条件供**BswM**评估。甚至**BswM**只能评估事件的接收情况。

当请求者发送/调用事件时，**BswMEventRequestPort**将处于**SET**状态。**BswM**可以稍后通过执行**BswMClearEventRequest**动作将**BswMEventRequestPort**置于**CLEAR**状态。下图展示了具有两个条件的示例规则。规则和可用逻辑操作集被定义为ECU配置的一部分。



当**BswMConditionType**配置为**BSWM_EVENT_IS_SET**，且**BswMModeCondition**引用了**BswMEventRequestPort**时：

- 如果**BswMEventRequestPort**处于**SET**状态，则**BswMModeCondition**应评估为**TRUE**。
- 如果**BswMEventRequestPort**处于**CLEAR**状态，则**BswMModeCondition**应评估为**FALSE**。

当BswMConditionType配置为**BSWM_EVENT_IS_CLEARED**，且**BswMModeCondition**引用了**BswMEventRequestPort**时：

- 如果**BswMEventRequestPort**处于**SET**状态，则**BswMModeCondition**应评估为**FALSE**。
- 如果**BswMEventRequestPort**处于**CLEAR**状态，则**BswMModeCondition**应评估为**TRUE**。

当**BswM**在配置的**BswMEventRequestPort**上接收到事件时，（例如：**BswM_ComM_InitiateReset**被**ComM**调用），**BswMEventRequestPort**需被置于**SET**状态。

当在**BswMEventRequestPort**上执行**BswMClearEventRequest**动作，则**BswMEventRequestPort**需被置于**CLEAR**状态。

### 6.1.3. 模式仲裁的需求

如上所述，**BswM**接受模式请求和模式指示作为模式仲裁的输入。模式请求通常来自于应用程序**SW-C**，但也可能来自于其他**BSW**模块，例如：**DCM**。而模式指示总是由其他**BSW**模块发出，例如：不同的总线特定状态管理器、**EcuM**和**WdgM**。在本文档中，通用术语模式仲裁请求（**mode arbitration request**）即可对应于模式指示（**mode indication**），也可对应于模式请求（**mode request**）。

**BswM**需根据以下的输入，执行模式仲裁。

1. 传入的模式请求（**mode requests**）
2. 输入模式指示（**mode indications**）
3. 传入的模式切换通知（**mode switch notification**）
4. 事件请求（**event requests**）
5. 事件请求的清除（**clearing of event requests**）

**注意：**

所有模式仲裁请求（请求和指示）都是由**BswM**模块以相同的方式处理。它们是通过在**BswMModeRequestSource**配置容器中，选择相应的模式条件类型（**mode condition type**）进行配置。

**BswM**也需支持使用所配置的规则执行模式仲裁。模式仲裁规则（**mode arbitration rules**）需支持通过模块的配置参数进行配置。

**BswM**禁止使用先前仲裁规则评估的结果作为逻辑表达式的输入。

**注意：**

禁止将规则评估的结果用作其他规则评估的输入，主要为zui大程度的满足了现有的**BswM**配置容器的结构。

作为评估**BswM**仲裁规则的结果而被调用的动作，只能在动作列表的上下文中被调用。

#### 6.1.3.1. 立即和延期操作

执行模式仲裁的处理有两种不同的方式:

1. 在模式请求/指示的上下文环境中立即执行，
2. 推迟到BswM的周期主函数中执行。

在调用者的上下文环境中执行立即（**immediate**）请求。系统集成者有责任确保动作列表（**action list**）的执行不会危及系统性能（**performance**）或一致性（**consistency**）。特别是，调用者可能在中断的环境中运行，则有关在中断环境中使用系统函数的限制必须被考虑。

下图展示了立即操作和延迟操作之间的区别。

**立即操作：**

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqFJvp67GNA9EKCoZHEBdZSZPeEmKVzhpR0OInCNHyeUOq31ddxic0eyaajWyhIpDaOGajo8bFlKGA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**延迟操作：**

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqFJvp67GNA9EKCoZHEBdZSSgYUpzCEiaianPe5Jbfku4bfm1ibSYic8ICN37TTSDdwmgfIo6dQ2IpruQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

模式仲裁规则可以包含立即和延迟模式仲裁请求的任意组合。

对于模式仲裁请求（**a mode arbitration request**），**BswM**模块可通过**BswMModeRequestPort**参数容器中的**BswMRequestProcessing**配置参数进行设定:

1. 当**BswM**模块需支持立即执行模式仲裁时，可将**BswMRequestProcessing**配置参数设置为**BSWM_IMMEDIATE**。
2. 当**BswM**模块需支持将模式仲裁推迟到BswM主函数执行时，可将**BswMRequestProcessing**配置参数设置为**BSWM_DEFERRED**。

只有使用特定的立即模式条件（**immediate mode condition**）的模式仲裁规则，才需要由**BswM**在该特定模式请求或者指示的上下文环境中进行评估。

对于事件的设置（**an event is set**)，**BswM**模块可通过**BswMEventRequestPort**参数容器中的**BswMEventRequestProcessing**配置参数进行设定:

1. 当**BswM**模块需支持立即执行模式仲裁时，可将**BswMEventRequestProcessing**配置参数设置为**BSWM_IMMEDIATE**。
2. 当**BswM**模块需支持将模式仲裁推迟到BswM主函数执行时，可将**BswMEventRequestProcessing**配置参数设置为**BSWM_DEFERRED**。

使用至少一个延迟模式条件的所有规则，需在每次执行**BswM**主函数（**main function**）期间进行评估。

**BswM**需推迟在其主函数处理期间所收到的模式仲裁请求，直到主函数完成。任何此类推迟的即时请求（**IMMEDIATE request**）都应在**BswM**主函数退出之前直接被处理。任何此类推迟的延迟请求（**DEFERRED request**）请求都应在下一个后续的**BswM**主函数中被处理。

**BswM**需推迟在处理**IMMEDIATE**请求期间收到的模式仲裁请求，直到此请求处理完成。任何此类推迟的即时请求（**IMMEDIATE request**）应在处理原始即时请求（**original IMMEDIATE request**）之后直接被处理。任何此类推迟的延迟请求（**DEFERRED request**）都应在下一个后续的**BswM**主函数中被处理。

**BswM**实现可以选择使用保护机制（**protection mechanisms**），例如：独占区（**Exclusive Area**），以保证动作的执行或者**BswM**主函数中的执行不会被任何其他更高优先级的任务中断。

**端口更新的术语说明：**

任何模式请求端口都有关联的值和状态。更新一个端口意味着改变它的值和状态。

**BswM**需在仲裁实际发生之前直接更新**IMMEDIATE**模式请求端口的值，而不是在模式请求端口被触发的时候。而当模式请求端口被触发时，**BswM**需更新**DEFERRED**模式请求端口的值。

### 6.1.4. 初始化后的仲裁行为

**BswM**初始化后的模式仲裁行为，由配置容器**BswMModeInitValue**控制。该参数可以为配置中的每个**BswMModeRequestPort**进行单次的配置。

如果容器**BswMModeInitValue**不存在或者**ModeRequest**尚无初始值，则**BswM**应将相应的模式条件视为未定义，直到相应的模式仲裁请求第一次更新后，才将其用于模式仲裁。

**BswM**需仅对在其逻辑表达式中不包含任何未定义模式条件的规则进行仲裁。

每个**BswMModeRequestPort**初始化后的初始值可以由配置容器**BswMModeInitValue**控制。

在定义**BswMModeInitValue**的情况下，**BswM**应在**BswM**被初始化时，使用**BswMBswModeInitValue**或者**BswMCompuScaleModeValue**的值来初始化相应的**BswMModeRequestSource**。对于单个的**BswMModeInitValue**，**BswM**需拒绝同时包含**BswMBswModeInitValue**和**BswMCompuScaleModeValue**的配置。该初始化值将用于仲裁规则，直到相应的模式仲裁请求已被更新，例如：每次调用**BswM_RequestMode**都需更新**GenericRequest**模式。

**注意：**

**Rte**和**SchM**的模式总是有一个初始值。

在**BswM**初始化时，所有**BswMEventRequestPorts**都需被初始化为**CLEAR**的状态。

## 6.2. 模式控制

**BswM**的模式控制部分，会根据模式仲裁的结果执行所有必需的操作。这是通过使用动作列表（**Action Lists**）来完成的。动作列表是**BswM**在由模式仲裁触发时，所执行操作的有序列表。

动作列表中的动作（**action**）可以是以下的三种类型：

1. 调用其他**BSW**模块或者**RTE**模块。预定义的动作可参考章节6.2.4-可用操作中所罗列的。
2. 链接要包含在执行中的其他动作列表。
3. 模式仲裁规则（**Mode arbitration rules**）。这些规则将在执行相应的动作列表时被评估。通过这种方式就获得了规则的层次结构（**a hierarchy of rules**）。

因为**BswM**不需要存储或响应任何**BSW**模块执行动作的特定返回值，所以**BSW**中的不同状态管理器（**different state managers**）将它们的当前状态作为模式仲裁的输入提示给**BswM**模块。

但是，如果错误（**E_NOT_OK**）被返回，**BswM**可以发出**DET**运行时错误，并可以取消当前执行的动作列表。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqFJvp67GNA9EKCoZHEBdZSN9HYNcQLeiat5TQdThFML3DuRdp7MGgrercdIB4lonSHfrEQstacIeA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)







如上图7-2所示，**BswM**可能包含多个动作列表（**Action List**），一个动作列表（**Action List**）可以容纳多个动作（**Action**）。为了减少动作列表的总数，动作列表应该可以被级联。

动作列表的元素可以是:

1. 一个具体动作（**a concrete action**）
2. 一个对另一个动作列表的引用（**a reference to another action list**）
3. 一个模式仲裁需要执行的规则（**a rule to be executed by the mode arbitration**）。

所以需要有一个标志连接到每个动作列表条目，说明它的类型：动作（**action**）/ 参考（**reference**）/ 规则（**rule**）。包含具体行动的列表与包含引用的列表，或者甚至混合列表的列表，它们的激活方式之间是没有区别。

### 6.2.1. 模式处理循环

下图7-3显示了模式请求的最小处理循环：

1. 模式请求者**SW-C**通过其发送端口（**Sender Port**）请求模式A。**RTE**分发请求，**BswM**通过接收端口接收（**Receiver Port**）到此请求。
2. **BswM**根据接收到的模式仲裁请求或在执行BswM主函数期间周期地评估这些规则。
3. 根据所选择的执行方式，执行相应的动作列表。具体细节可参见章节6.2.3触发和条件动作列表。
4. 在执行动作列表时，**BswM**可以发出一个或多个对**RTE Switch API**[6]的调用，来通知受影响的**SW-C**仲裁结果。任何**SW-C**，尤其是模式请求者都可以注册以接收模式切换指示。

**注意：**

模式请求者只能从本地**BswM**接收模式切换指示。对于来自本地代理**SW-C**发出的其他**ECU**的请求也同样适用。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqFJvp67GNA9EKCoZHEBdZSIewUyXXIq011yg8Ab21Ykls2FbFuPGItc0Yb4RFeMZZlF957Psur4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)







### 6.2.2. 模式控制需求

**BswM**应通过作为模式仲裁中规则评估结果而执行的动作列表来执行模式控制。

对于模式仲裁的每条规则，**BswM**应能够根据规则评估为**True**或者**False**执行不同的动作列表。

一个动作列表中可以包含一组**BswM**需顺序执行的动作，也可以包含了指向**BswM**应执行的其他动作列表的链接。

同时一个动作列表也可以包含指向**BswM**需在当前动作列表执行范围内所需评估的模式仲裁规则的链接。如果指定的动作列表中包含了模式仲裁规则，则由该评估产生的任何动作列表的执行都应由**BswM**在继续执行原动作列表之前执行。

如果使用了级联动作列表（**cascaded action lists**），即包含对其他规则或动作列表的引用，则动作列表结构可以包含多达七个层次级别。

**注意：**

此限制的目的是使**BswM**实现和生成工具的测试成为可能。此限制可以由生成工具进行检查。

与在模式仲裁请求上下文环境中评估的规则相关联的动作列表，需在模式仲裁触发时由**BswM**立即执行，而不是推迟到**BswM**主函数中执行。这样可以达到必要时对模式请求进行非常短的延迟（**short latencies**）。

一个顶级规则（**top-level rule**）是指并不嵌套在动作列表中的规则。一个顶级动作列表（**top-level action list**）是一种由顶级规则直接执行的动作列表，它不嵌套在另一个动作列表中。

在模式仲裁期间，如果一个顶级动作列表被多个规则触发，这会导致在模式控制期间只有单个触发执行了该动作列表的。它仅适用于顶级动作列表，并不适用于嵌套规则和嵌套动作列表，原因在于它们在父动作列表中的顺序是由用户自定义的。

在模式控制期间，如果要执行多个顶级动作列表，则执行顺序应**BswMActionListPriority**的设置从高到底。在**BswMActionListPriority**相同的情况下，执行顺序是任意的。对于不是顶级动作列表的动作列表，需忽略**BswMActionListPriority**。

如果没有**BswMActionListPriority**的动作列表应被解释为**BswMActionListPriority**等于**0**。

**BswM**需拒绝**BswMActionList**包含具有相同值**BswMActionListItemIndexes**的**BswMActionListItems**的配置。

在执行**BswMActionList**时，**BswM**应从具有最低**BswMActionListItemIndex**值的**BswMActionListItem**开始。随后的**BswMActionListItems**应按其 **BswMActionListItemIndex**的递增顺序执行。

在动作列表中，配置的**BswMActionListItemIndexes**不一定需要是连续的或从零开始的。**BswM**将开始执行具有最低索引的动作列表项，并继续执行具有最高索引的动作列表项。如果索引有间隙，即索引不连续的情况，这些间隙将被简单地忽略。因为动作列表是有序列表，所以不允许在动作列表的上下文中配置相同数值的**BswMActionListItemIndexes**。

### 6.2.3. 触发和条件动作列表

基于规则评估，动作列表有两种方式被执行：

1. **BSWM_CONDITION**：条件执行，即每次评估规则时被执行。
2. **BSWM_TRIGGER**：触发执行，即每次评估结果发生变化时才执行。

可以使用**BswMActionList**配置容器中的**BswMActionListExecution**配置参数，对动作列表的执行方式的进行设定。

但是对于不被规则直接引用的嵌套动作列表，**BswMActionListExecution**参数（如：**BSWM_CONDITION**或者**BSWM_TRIGGER**）是没有任何意义，并且不会影响嵌套动作列表的执行方式。每当其父动作列表被执行时，不被规则直接引用的嵌套动作列表也会被相应地执行。

如果为触发执行配置了一个**True**的动作列表，则**BswM**仅在相应规则的评估从**False**变为**True**时才会执行此动作列表。如果为触发执行配置了一个**False**的动作列表，则**BswM**仅在相应规则的评估从**True**变为**False**时才会执行此动作列表。

如果为条件执行配置了一个**True**的动作列表，则**BswM**应在每次将相应规则评估为**True**时执行此动作列表。如果为条件执行配置了**False**的动作列表，则**BswM**应在每次将相应规则评估为**False**时执行此动作列表。

如果某个动作返回**E_NOT_OK**，并且相应的**BswMAbortOnFail**配置参数设置为**True**，则**BswM**需中止此动作列表的继续执行。

### 6.2.4. 可用动作

**BswM**包含了部分预定义的动作集合，这些动作可以在动作列表中被使用。这样做的原因是为了简化**ECU**配置和**BswM**配置代码的生成。

**BswM**需要能够执行由配置容器**BswMAvailableActions**定义的预定义动作。同时**BswM**应能够调用**AUTOSAR BSW**中的任何函数，即使它不存在**BswMAvailableActions**配置定义的标准化动作中。

- BswMClearEventRequest
- BswMComMAllowCom
- BswMComMModeLimitation
- BswMComMModeSwitch
- BswMCoreHaltMode
- BswMDeadlineMonitoringControl
- BswMEcuMDriverInitListBswM
- BswMEcuMGoDownHaltPoll
- BswMEcuMSelectShutdownTarget
- BswMEcuMStateSwitch
- BswMEthIfStartAllPorts
- BswMEthIfSwitchPortGroupRequestMode
- BswMFrSMAllSlots
- BswMIdsMBlockState
- BswMJ1939DcmState
- BswMJ1939RmStateSwitch
- BswMLinScheduleSwitch
- BswMNMControl
- BswMPduGroupSwitch
- BswMPduRouterControl
- BswMRteModeRequest
- BswMRteStart
- BswMRteStop
- BswMRteSwitch
- BswMSchMSwitch
- BswMSdClientServiceModeRequest
- BswMSdConsumedEventGroupModeRequest
- BswMSdServerServiceModeRequest
- BswMSdServiceGroupSwitch
- BswMSwitchIPduMode
- BswMTimerControl
- BswMTriggerIPduSend
- BswMUserCallout

BswM也能够调用用户定义的函数（**user defined functions**），同时用户定义函数的参数及其数值需在**ECU**配置时定义，通过**BswMUserCallout**配置容器来定义。

### 6.2.5. 初始化后模式控制的行为

**BswM**初始化后，模式控制的行为通过**BswMRule**配置容器内的**BswMRuleInitState**配置参数进行配置。它定义了在初始化后第一次评估规则后，决定执行哪个动作列表时，需要使用的先前评估结果（**previous evaluation result**）。在**BswMActionList**配置容器内的**BswMActionListExecution**配置参数也会影响初始化后的动作列表执行。

在初始化后第一次评估规则时，**BswM**需根据表7-1中的规定进行操作。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqFJvp67GNA9EKCoZHEBdZSerBpuTSZia94CrR5icpaRPY1ct60xWODwuhu7v7XLM2kKcMSSIoheuJQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**注意：**

表格中的**true**和**false**的动作列表对于每条规则都是可选的。

## 6.3. 等待功能

有时需要延迟某个特定的操作（**specific actions**）或等待进一步的模式控制（**mode controls**）。出于这个原因，**BswM**添加了定时器处理（**Timer handling**）。

一个定时器总是包含两部分构成：

1. 一个作为**BswMModeRequestSource**的**BswMTimer**。
2. 一个控制此**BswMTimer**的相应动作（可参见**BswMTimerControl**配置参数）

**BswMTimer**的数值包括：**BSWM_TIMER_STOPPED**、**BSWM_TIMER_STARTED**、**BSWM_TIMER_EXPIRED**。

定时器的操作只能在**BswMTimerControl**动作的上下文环境中控制。同时**BswMTimer**的值可以被**BswM**中所配置的其他规则进行评估，并以此触发动作列表。外部接口（**external interface**）是无法来控制或者操作此定时器的。

每个**BswMTimer**在初始化期间，都是停止状态，也就是说**BswMTimer**的初始值需为**BSWM_TIMER_STOPPED**。

**BSWM_TIMER_START**的**BswMTimerAction**动作需使用相应配置在**BswMTimerValue**参数中的定时器值，重新加载引用的**BswMTimer**（通过：**BswMTimerRef**），并将定时器的模式更改为**BSWM_TIMER_STARTED**。

注意：定时器只能通过 BswMTimerAction 动作重新加载（不可能自动重新加载）。

每个**BswMTimer**处于**BSWM_TIMER_STARTED**模式时，需在**BswM_MainFunction**中递减定时器的计数（需按**BswM_MainFunction**的周期时间）。

**注意：**

**BswMTimer**分辨率（**resolution**）需是**BswM_MainFunction**周期时间的倍数。此外**BswMTimer**的准确性取决于**BswM_MainFunction**的准确性。

如果处于**BSWM_TIMER_STARTED**模式的**BswMTimer**时间到期时，则**BswMTimer**模式需更改为**BSWM_TIMER_EXPIRED**，接着**BswMTimer**模式需在同一个**BswM_MainFunction**的周期中进行仲裁。

**BSWM_TIMER_STOP**的**BswMTimerAction**动作需立即停止引用的**BswMTimer**（通过：**BswMTimerRef**）并将其模式更改为**BSWM_TIMER_STOPPED**。

与**BswMTimer**相关的**BswMRequestProcessing**（例如：**IMMEDIATE**、**DEFERRED**）配置会被**BswM**忽略。**BswM**会始终将**BswMTimer**的处理视为**DEFERRED**，并在**BswM**主函数期间进行仲裁。

注意：在**BswM_TIMER_EXPIRED**模式下的**BswMTimer**不会被**BswM**自动设置为**BSWM_TIMER_STOPPED**。为了将**BswMTimer**从**BSWM_TIMER_EXPIRED**转换到另一种模式，用户需要配置一个动作（**Action**）。如果没有配置任何动作使**BswMTimer**退出**BSWM_TIMER_EXPIRED**模式，那么**BswMTimer**将在接下来的**BswM**主函数周期中继续被仲裁为**BSWM_TIMER_EXPIRED**。

## 6.4. 多分区支持

对于多个**BswM**实例的情况，每个**BswM**实例将根据自己的配置集生成自己单独的服务组件描述。集成商需要将这些单独的服务组件分配给相应的分区。

**BswM**存在于每个分区中，并具有特定于此分区的配置，即每个分区的**BswMConfig**都是单独实例的。包含的动作列表只在本地分区执行。

## 6.5. 错误分类

### 6.5.1. 运行时错误

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqFJvp67GNA9EKCoZHEBdZS0n3NbWPPeI9f8cPiaibyJdZPib0licSrl3NW5rXGevDXm3kqdKfjGCRR5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

当**BswMActionListItem**配置了**BswMReportFailRuntimeErrorId**，如果操作返回**E_NOT_OK**，则**BswM**需向**Det**报告**BSWM_E_ACTION_FAILED**运行时错误。**BSWM_E_ACTION_FAILED**运行时错误中报告的**ErrorId**由**BswMReportFailRuntimeErrorId**中配置的值给出。

## 6.6. BswM接口和端口

本章节描述了BswM提供的**AUTOSAR**接口（**Interface**）和端口（**Port**）。

请注意，**RTE**两侧的端口都是需要地：

1. **BswM**服务的**SW-C**描述将定义**RTE**下方的端口。
2. 每个使用该服务的AUTOSAR的**SW-C**必须在其自己的**SW-C**描述中包含它的服务端口。

这些端口需使用相同的接口，并且必须连接到BswM的端口上，以便**RTE**可以生成适当的**ID**和所需的符号。

当**SW-C**向**BswM**模块请求模式，**SW-C**需提供了一个发送端口（**Sender Port**），此端口需定义为一个的特殊**SenderReceiverInterface**模式请求接口（**Mode Request Interface**），接口定义必须包含了一个数据元素（**data element**）。 **BswM**模块中对应的接收器端口（**Receiver Port**）在章节6.6.1中描述。数据元素的类型需与对应模式的模式声明组（**Mode Declaration Group**）中的模式声明（**Mode Declarations**）类型相同，具有相同的数值。因为数据元素的**ImplementationDataType**需映射到**Mode Declaration Group**。

因为**SW-C**也可能需要知道**BswM**的仲裁结果，所以请求模式的**SW-C**也可以是一个模式用户（**a mode user**）。此类**SW-C**有一个模式切换端口（**Mode Switch Port**），模式切换端口被定义为一个模式切换接口（**Mode Switch Interface**）的**R-Port**，模式切换接口（**Mode Switch Interface**）也包含一个数据元素（**data element**）。这个数据元素的类型就是模式声明组（**Mode Declaration Group**）本身。

此外其他即使不请求模式但依赖于模式的**SW-C**也可以包含这样的模式切换端口。有关模式用户接口的详细说明，请参见章节6.6.3 模式切换的通知 。

**注意：**

如果**BSW**模式管理器需要知道除了自身决策中请求的模式之外的当前模式，**BSW**模式管理器也需要一个模式切换的**R-Port**。

当**BSW**模式管理器切换相应模式时，**RTE**会发送模式通知。为此**BSW**模式管理器有一个模式切换端口的**P-Port**，**SW-C**可以连接到该端口。有关模式切换端口的详细说明，请参见章节6.6.2 模式切换端口

在请求**SW-C**的上下文环境中，定义了模式请求端口（**Mode Request Port**），包括发送方（**Sender**）和接收方（**Receiver**）。**Bsw**模式管理器的配置会参考了此端口的定义。

举例：一个**SW-C**定义了一个应用模式**AppModeType**，一个对应的**AppModeRequestType**和一个将这两种类型相互映射的**AppModeTypeMap**的例子

```
ModeDeclarationGroup AppModeType {
    { APP_MODE_A, APP_MODE_B, APP_MODE_C }
    initialMode = APP_MODE_A;
};
ImplementationDataType AppModeRequestType {
    lowerLimit = 0;
    upperLimit = 2;
};
ModeRequestTypeMap AppModeTypeMap {
    modeGroup = AppModeType;
    implementationDataType = AppModeRequestType;
};
```

**SW-C**定义了下面的两个接口：

1. **AppModeRequestInterface**：**SW-C**作为发送者，接口为**SenderReceiverInterface**。
2. **AppModeInterface**：SW-C根据使用场景，可以是**P-Ports**和**R-Ports**，接口为**Mode Switch Interface**：

下图**7-4**显示了应用程序**SW-C**的端口如何连接到**BSW**模式管理器的服务端口。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqFJvp67GNA9EKCoZHEBdZSco8OXYxAEa5IIqtst0Tr7926YbGfUMMAj2qseEyiaQCich7hicR4XmbgA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



命名为应用程序模式管理器（**Application Mode Manager**）的**SW-C**包含了两个端口：

1. 一个模式请求端口（**Mode Request Port**）的**P-Port**。
2. 一个模式切换（**Mode Switch**）的**R-Port**。（命名为**modeSwitchPort**以区别于模式切换**P-Port**）。

第一个模式请求端口是请求更改应用程序模式，而第二个模式切换端口是在**BswM**执行模式更改时用来接收通知。应用程序模式管理器的模式请求端口（**modeRequestPort**）连接到相应的**BSW**模式管理器的模式请求端口。由于这是正常的发送方/接收方通信（**Sender/Receiver communication**），所以应用程序模式管理器可以连接到多个**BSW**模式管理器，即使在远程**ECU**上也是如此。

为了切换应用模式，**Bsw**模式管理器有一个模式切换端口（**modeSwitchPort_{Name}**），通过本地**RTE**实现。

当**RTE**执行模式切换时，它会通知所有通过模式切换**R-Ports**连接到提供端口的连接实体，包括：BSW模块（**BSW Modules**）或**SW-C**。以下示例展示了应用程序模式管理器（**Application Mode Manager**）、其他与模式相关的应用程序部分（**Application Part**）和**BSW**模式管理器自身（注意它的名称为**modeSwitchPort_{Name}**，但端口类型为模式切换端口）。所有这些连接也是本地的。

图7-5显示基于**SW-C**的应用程序模式管理器（在 AUTOSAR R3.1 和更早版本中使用）直接切换应用程序模式，而不向**BSW**模式管理器请求它。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqFJvp67GNA9EKCoZHEBdZShErwtx5x0y3U9YRNtl5G6cqA4cR1Azc88gYqEYtZwHRJC65iccibMkcQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)





它们直接将模式切换端口连接到本地**RTE**。这意味着应用程序模式需要在ECU本地，并且**BSW**模式管理器中的仲裁是不可能的。尽管如此，**BSW**模式管理器可以使用当前应用模式作为其规则的输入，因为它可以有一个模式切换**R-Port**（图中名为 **modeNotificationPort0**）用于该应用模式。

**注意：**

为了配置**BswM**，需要了解基于特定**ECU**需求的模式请求端口和需要及可用的**ECU**资源。所以**BswM**的SW-C描述（**SW-C description**）只能在**ECU**配置期间才能完成。

从现在开始，以下所有接口定义都被解释为包含在以下的**ARPackage**中:

```
ARPackage AUTOSAR_BswM/BswModuleDescription
```

**注意：**

本章节介绍的伪代码并不准确，但提供了如何定义相应模型元素的提示。

### 6.6.1. 模式请求端口

**BSW**模式管理器必须声明一个接收器端口（**Receiver Port**），此端口接口定义在**SW-C**的上下文中定义：

```
RequirePort AppModeRequestInterface modeRequestPort_{ArbName}_{ReqName};
```

要读取当前请求的模式，**BSW**模式管理器实现必须调用：

```
Rte_Read_modeRequestPort_{ArbName}_{ReqName}_requestedMode(& <variable> );
```

### 6.6.2. 模式切换端口

与模式请求一样，**BSW**模式管理器只引用模式切换对应的**SW-C**描述中定义的**Provide Ports**的接口。

对于上面的示例，模式切换的声明如下：

```
ProvidePort AppModeInterface modeSwitchPort_{ModConName}_{SwitchName};
```

配置参数**BswMModeSwitchInterfaceRef**引用此模式切换接口。

要切换当前活动模式，**BSW**模式管理器实现必须插入以下调用中的某一个调用到其动作列表中：

```
Rte_Switch_modeSwitchPort_{ModConName}_{SwitchName}_currentMode ( <new_mode> );
SchM_Switch_modeSwitchPort_{ModConName}_{SwitchName}_currentMode( <new_mode> );
```

### 6.6.3. 模式切换的通知

除了模式请求之外，当前激活的模式也可以用作模式仲裁的输入。对于应用程序和车辆模式，**BSW**模式管理器需要注册为模式用户（**a mode user**）。然后它通过模式切换端口接收有关模式更改的通知。

对于上面的示例，模式通知的声明如下：

**注意：**

为了更容易区分**ModeSwitchPort**类型的**RequirePort**和**ProvidePort**，以下示例将**RequirePort**命名为模式通知端口。

```
RequirePort AppModeInterface modeNotificationPort_{ArbName}_{ModeName};
```

要读取当前活动模式，**BSW**模式管理器实现必须调用以下函数中的某一个函数：

```
Rte_Mode_modeNotificationPort_{ArbName}_{ModeName}_currentMode( &<variable> );
SchM_Mode_modeNotificationPort_{ArbName}_{ModeName}_currentMode( &<variable> );
```

如果配置了增强型**Rte_Mode**或**SchM_Mode**，则**BSW**模式管理器实现必须调用以下函数中的某一个函数：

```
Rte_Mode_modeNotificationPort_{ArbName}_{ModeName}_currentMode( &<variable>, &<previousmode>, &<nextmode> );
SchM_Mode_modeNotificationPort_{ArbName}_{ModeName}_currentMode( &<variable>, &<previosmode>, &<nextmode> );
```

### 6.6.4. 组件类型和内部行为

**BSW**模式管理器是一个服务组件（**Service Component**），它为**ECU**本地的模式请求提供服务。**BSW**模式管理器的**ServiceComponentType**声明了所有上述所有端口和一些内部行为（**Internal Behavior**）。

```
ServiceComponentType BswM {
    ...
    InternalBehavior {
        ...
    };
};
```

内部行为（**Internal Behavior**）依赖于相关的模式请求端口的参数**BswMRequestProcessing**。对于**BSWM_DEFERRED**，因为**BSWM**模式管理器在其**BswM_MainFunction**中循环读取请求，所以**RTE**不会执行任何特殊操作。相比之下，对于**BSWM_IMMEDIATE**，因为**RTE**必须立即触发模式仲裁，所以**BSW**模式管理器需要注册一个触发模式仲裁的触发函数。对于上面的示例，模式请求的立即处理将需要在**BSW**模式管理器的内部行为中进行以下声明：

```
RunnableEntity ModeArbitrationRunnable {
    symbol = <mode_arbitration_function>;
    canBeInvokedConcurrently = TRUE;
};
DataReceiveEvent AppModeRequestEvent {
    port = modeRequestPort0;
    dataElement = requestedMode;
    startOnEvent = ModeArbitrationRunnable;
};
```

**注意：**

如要处理来自其他**ECU**的模式请求，需要另一种服务组件（**service component**）。在**VFB**级别上，它看起来像一个全局服务组件，但实际上它被实例化为驻留在每个**ECU**的**RTE**之上的一个服务组件。为了支持这一点，**SW-C**模板提供了**ServiceProxyComponentType**而不是普通的**ServiceComponentType**。

因为模式管理服务代理组件的规范是用户特定的，所以没有在本文档中进行描述。

## 6.7. 以太网交换机端口组交换

当前版本的**SWS BswM**支持以太网交换机端口组交换。基于当前请求的**PNC**，**BswM**将**PNC**请求映射到所配置的**EthIfSwitchPortGroup**并调用**EthIf_SwitchPortGroupRequestMode**。如果积累的链路状态已更改，则由**EthIf**模块指示**BswM**模块。积累的链接状态也可用于通知应用程序。这可用于覆盖 **EthIfSwitch**端口组的请求和当前积累的链路状态相互矛盾并且需要启动错误处理的错误场景。

**AUTOSAR_EXP_ModemanagementGuide**文档包含了有关以太网交换机端口组切换的**BswM**配置指南。

# 7. API规范

## 7.1. 函数定义

### 7.1.1. BswM_BswMPartitionRestarted

**说明**: 如果包含**BswM**的分区已经被重新启动，函数则被**Restart Task**调用。

```
void BswM_BswMPartitionRestarted (
    void
)
```

### 7.1.2. BswM_CanSM_CurrentState

**说明**: 函数被**CanSM**模块调用，用于指示**CanSM**当前状态。

```
void BswM_CanSM_CurrentState (
    NetworkHandleType Network,
    CanSM_BswMCurrentStateType CurrentState
)
```

### 7.1.3. BswM_ComM_CurrentMode

**说明**: 函数被**ComM**模块调用，用于指示**ComM**通道的当前通信模式。

```
void BswM_ComM_CurrentMode (
    NetworkHandleType Network,
    ComM_ModeType RequestedMode
)
```

### 7.1.4. BswM_ComM_CurrentPNCMode

**说明**: 函数被**ComM**模块调用，用于指示**PNC**的当前模式。

```
void BswM_ComM_CurrentPNCMode (
    PNCHandleType PNC,
    ComM_PncModeType CurrentPncMode
)
```

### 7.1.5. BswM_ComM_InitiateReset

**说明**: 函数被**ComM**模块调用，用于发出关闭信号。

```
void BswM_ComM_InitiateReset (
    void
)
```

### 7.1.6. BswM_Dcm_ApplicationUpdated

**说明**: 函数被**DCM**模块调用，用于报告更新的应用程序。

```
void BswM_Dcm_ApplicationUpdated (
    void
)
```

### 7.1.7. BswM_Dcm_CommunicationMode_CurrentState

**说明**: 函数被**DCM**模块调用，用于通知**BswM**通信模式的当前状态。

```
void BswM_Dcm_CommunicationMode_CurrentState (
    NetworkHandleType Network,
    Dcm_CommunicationModeType RequestedMode
)
```

### 7.1.8. BswM_Deinit

**说明**: 去初始化**BSW**模式管理器。

```
void BswM_Deinit (
    void
)
```

### 7.1.9. BswM_EcuM_CurrentState

**说明**: 函数被**EcuM**模块调用，用于指示当前的**ECU**操作模式。

```
void BswM_EcuM_CurrentState (
    EcuM_StateType CurrentState
)
```

### 7.1.10. BswM_EcuM_CurrentWakeup

**说明**: 函数被**EcuM**模块调用，用于指示唤醒源的当前状态。

```
void BswM_EcuM_CurrentWakeup (
    EcuM_WakeupSourceType source,
    EcuM_WakeupStatusType state
)
```

### 7.1.11. BswM_EcuM_RequestedState

**说明**: 函数被**EcuM**模块调用，用于通知运行请求协议（**Run Request Protocol**）的当前状态。。

```
void BswM_EcuM_RequestedState (
    EcuM_StateType State,
    EcuM_RunStatusType CurrentState
)
```

### 7.1.12. BswM_EthIf_PortGroupLinkStateChg

**说明**: 函数被**EthIf**模块调用，用于指示某个以太网交换机端口组（**Ethernet switch port group**）的链路状态变化

```
void BswM_EthIf_PortGroupLinkStateChg (
    EthIf_SwitchPortGroupIdxType PortGroupIdx,
    EthTrcv_LinkStateType PortGroupState
)
```

### 7.1.13. BswM_EthSM_CurrentState

**说明**: 函数被**EthSM**模块调用，用于指示**EthSM**当前状态。

```
void BswM_EthSM_CurrentState (
    NetworkHandleType Network,
    EthSM_NetworkModeStateType CurrentState
)
```

### 7.1.14. BswM_FrSM_CurrentState

**说明**: 函数被**FrSM**模块调用，用于指示**FrSM**当前状态。

```
void BswM_FrSM_CurrentState (
    NetworkHandleType Network,
    FrSM_BswM_StateType CurrentState
)
```

### 7.1.15. BswM_GetVersionInfo

**说明**: 返回**BswM**模块的版本信息。

```
void BswM_GetVersionInfo (
    Std_VersionInfoType* VersionInfo
)
```

### 7.1.16. BswM_Init

**说明**: 初始化**BSW**模式管理器。

```
void BswM_Init (
    const BswM_ConfigType * ConfigPtr
)
```

### 7.1.17. BswM_J1939DcmBroadcastStatus

**说明**: 此**API**告诉**BswM**可用网络的所需通信状态。该状态通常通过**COM I-PDU Group Switch**激活。

```
void BswM_J1939DcmBroadcastStatus (
    uint16 NetworkMask
)
```

### 7.1.18. BswM_J1939Nm_StateChangeNotification

**说明**: J1939Nm状态更改后，通知当前的J1939Nm状态。

```
void BswM_J1939Nm_StateChangeNotification (
    NetworkHandleType Network,
    uint8 Node,
    Nm_StateType NmState
)
```

### 7.1.19. BswM_LinSM_CurrentSchedule

**说明**: 函数被**LinSM**模块调用，用于指示特定**LIN**通道的当前活动调度表。

```
void BswM_LinSM_CurrentSchedule (
    NetworkHandleType Network,
    LinIf_SchHandleType CurrentSchedule
)
```

### 7.1.20. BswM_LinSM_CurrentState

**说明**: 函数被**LinSM**模块调用，用于指示**LinSM**当前状态。

```
void BswM_LinSM_CurrentState (
    NetworkHandleType Network,
    LinSM_ModeType CurrentState
)
```

### 7.1.21. BswM_LinTp_RequestMode

**说明**: 函数被**LinTP**模块调用，用于请求相应**LIN**通道的模式。与**LinTp_Mode**相关的**LIN**调度表需被使用。

```
void BswM_LinTp_RequestMode (
    NetworkHandleType Network,
    LinTp_Mode LinTpRequestedMode
)
```

### 7.1.22. BswM_Nm_CarWakeUpIndication

**说明**: 函数被**Nm**模块调用，用于指示整车被唤醒。

```
void BswM_Nm_CarWakeUpIndication (
    NetworkHandleType Network
)
```

### 7.1.23. BswM_Nm_StateChangeNotification

**说明**: **Nm**状态更改后，当前**Nm**状态的通知。

```
void BswM_Nm_StateChangeNotification (
    NetworkHandleType Network,
    Nm_StateType currentState
)
```

### 7.1.24. BswM_NvM_CurrentBlockMode

**说明**: 函数被**NvM**模块调用，用于指示**NvM Block**的当前块模式（**block mode**）。

```
void BswM_NvM_CurrentBlockMode (
    NvM_BlockIdType Block,
    NvM_RequestResultType CurrentBlockMode
)
```

### 7.1.25. BswM_NvM_CurrentJobMode

**说明**: 函数被**NvM**模块调用，用于通知**BswM**多块作业（**multi block job**）的当前状态。

```
void BswM_NvM_CurrentJobMode (
    NvM_MultiBlockRequestType MultiBlockRequest,
    NvM_RequestResultType CurrentJobMode
)
```

### 7.1.26. BswM_RequestMode

**说明**: 对请求模式（**request mode**）的通用函数调用（**Generic function call**）。该功能只能由没有特定模式请求接口的其他的**BSW**模块使用。

```
void BswM_RequestMode (
    BswM_UserType requesting_user,
    BswM_ModeType requested_mode
)
```

### 7.1.27. BswM_Sd_ClientServiceCurrentState

**说明**: 函数被**Sd**模块调用，用于指示客户端服务的当前状态（可用/关闭）。

```
void BswM_Sd_ClientServiceCurrentState (
    uint16 SdClientServiceHandleId,
    Sd_ClientServiceCurrentStateType CurrentClientState
)
```

### 7.1.28. BswM_Sd_ConsumedEventGroupCurrentState

**说明**: 函数被**Sd**模块调用，用于指示**Consumed Eventgroup**的当前状态（可用/关闭）。

```
void BswM_Sd_ConsumedEventGroupCurrentState (
    uint16 SdConsumedEventGroupHandleId,
    Sd_ConsumedEventGroupCurrentStateType ConsumedEventGroupState
)
```

### 7.1.29. BswM_Sd_EventHandlerCurrentState

**说明**: 函数被**Sd**模块调用，用于指示**EventHandler**的当前状态（已请求/已释放）。

```
void BswM_Sd_EventHandlerCurrentState (
    uint16 SdEventHandlerHandleId,
    Sd_EventHandlerCurrentStateType EventHandlerStatus
)
```

### 7.1.30. BswM_SoAd_SoConModeChg

**说明**: 函数被**SoAd**模块调用，用于通知套接字连接的状态变化。

```
void BswM_SoAd_SoConModeChg (
    SoAd_SoConIdType SoConId,
    SoAd_SoConModeType State
)
```

### 7.1.31. BswM_WdgM_RequestPartitionReset（OBSOLETE）

**说明**: 函数被**WdgM**模块调用，用于请求分区重置的函数。

```
void BswM_WdgM_RequestPartitionReset (
    ApplicationType Application
)
```

## 周期函数

### 7.1.32. BswM_MainFunction

**说明**: **BswM**的主函数。

```
void BswM_MainFunction (
    void
)
```