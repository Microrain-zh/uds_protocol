# AUTOSAR Mirror （总线镜像）

# 1. 简介和功能概述

本文档说明了AUTOSAR基本软件模块总线镜像（**Bus Mirroring**）的功能、API和配置。
总线镜像（**Bus Mirroring**）需要AUTOSAR 4.4.0以上版本支持。

总线镜像模块的目的是将内部总线的流量和状态复制到外部总线，这样连接到外部总线的测试设备（**tester**）就可以监控内部总线以进行调试。

通信流量的监控可以通过测试设备发送诊断命令对中间的ECU (**Intermediate ECUs**) (如：网关、子总线控制器) 进行配置。使用诊断协议可以确保在未通过安全检查的情况下，是无法启用镜像功能。

在文档中，术语总线 (**Bus**) 和网络 (**Network**) 可视为同义词。在大多数AUTOSAR规范中，更倾向于使用术语网络 (**Network**) 。因此在引用API参数、配置或协议布局 (**protocol layout**) 时使用它。但该模块被称为总线镜像 (**Bus mirroring**) ，因此在考虑到了镜像方向，比如源总线 (**source bus**) 或目的总线 (**destination bus**) 。



# 2. 相关的规范

AUTOSAR提供了基本软件模块的通用规范，对总线镜像模块来说同样适用。



# 3. 约束和假设



## 3.1. 限制

总线镜像模块不能用于影响配置为源总线 (**source bus**) 上的流量。为了确保这一点，必须避免消息环回 (**loop-back of messages**) 导致总线过载，生成的工具应确保没有总线同时作为源总线和目的总线连接到总线镜像模块。

总线镜像模块通过专用（服务）API由诊断控制应用程序 (**a diagnostic control application**) 控制。诊断仪可以过特殊的诊断服务来控制功能，这些诊断服务由DCM处理，并通过诊断控制应用程序实现。**DCM**提供了必要的安全性，以排除总线镜像的意外激活。总线镜像模块不提供另一个控制接口 (**control interface**) ，也不接收目标总线上的控制消息。

一般来说，总线镜像模块不支持源总线的帧大小超过目标总线，或者源总线包含更多附加信息。

例如以下的内容是不被支持的:

- CANFD转到CAN
- CAN转到LIN
- FlexRay到CAN
- 以太网到CAN
- 以太网到FlexRay。

总线镜像模块并不会切分帧的内容。

总线镜像模块只镜像总线接口模块实际接收或传输的流量数据。对于CAN来说，这意味着除了传输帧之外，只有那些通过硬件过滤器的数据帧会被镜像，而远程帧和错误帧不会被镜像。对于LIN，从节点（**Slave**）到从节点（**Slave**）的通信不会被LIN主节点 (**Master**) 镜像。对于FlexRay，只有发送的帧和接收到的帧被分配了接收缓冲区（可能是FIFO），才会被镜像。

总线镜像模块不应该重新序列化接收到的序列化帧，因为这将需要太多的资源。相反，序列化的PDU应该直接路由到目的总线。

总线镜像模块也不支持从以太网转发到以太网。AUTOSAR以太网交换机驱动程序的端口镜像特性已经涵盖了这个用例。



## 3.2. 汽车领域的适用性

总线镜像模块可用于所有带有外部CAN连接器和以太网连接器（Ethernet connectors）的车辆，例如诊断连接器（Diagnostic connector）。



# 4. 对其他模块的依赖关系

总线镜像（**Bus Mirroring**）模块的接口关联到CAN接口（**CanIf**）、LIN接口（**LinIf**）、FlexRay接口（**FrIf**）、PDU路由器（**PduR**）、默认错误跟踪器（**DET**）以及诊断应用程序（**diagnostic application**），它通过AUTOSAR的运行时环境（**RTE**）或者总线镜像模块的复杂驱动程序（**CDD**）API来访问这些服务端口（**service port**）的程序接口API。

总线镜像模块可以包含CanIf、LinIf、FrIf、PduR、DET、StbM和RTE的头文件。



# 5. 功能规格



## 5.1. 概述

总线镜像模块的任务是收集来自几个源总线（**source buses**）的帧，然后将这些帧转发到目标总线（**destination bus**）。转发是严格单向的，从而可以避免消息循环（**message loops**）和阻止入侵场景（**intrusion scenarios**）

下图显示了如何将总线镜像集成到AUTOSAR BSW通信堆栈中：

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhq5IEuAdcnb0v5sKXBmltOIwdpkvzZSrwOMTIhEwgiaWNGcSb924p2D3cDWdDtbQHEtyrRefzuPCvA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**总线镜像模块支持以下镜像场景:**

- CAN和LIN总线 => CAN总线
- CAN、CAN-FD和LIN总线 => CAN-FD总线
- CAN、CAN-FD、LIN和FlexRay总线 => FlexRay总线
- CAN、CAN-FD、 LIN和FlexRay总线 => IP总线
- CAN、CAN-FD、LIN和FlexRay总线 => 专有的总线（CDD）

为了避免目标总线过载，可以对每个源总线上接收的消息进行筛选。过滤器可以为每个总线进行单独配置，主要通过配置选择（**MirrorSourceCanFilter**、**MirrorSourceLinFilter**和**MirrorSourceFlexRayFilter**）进行配置，也可以在运行时调用程序接口动态配置。

LIN和CAN或者CANFD帧被镜像到CAN或者CANFD总线上，可以直接发送相同的数据。在CAN或者CANFD的场景下，可以保留CAN ID，但可以重新映射CAN ID以避免目的总线上的ID冲突。另一方面，LIN PID总是需要映射到相应的CAN ID。为了避免ID冲突，镜像帧可以使用扩展CAN ID（**extended CAN ID**）范围。

当帧被镜像到FlexRay总线、IP总线（以太网）或作为CDD连接的专有总线时，源帧被将使用指定的协议，封装到一个更大的帧中。当路由到FlexRay总线时，只有那些足够小的FlexRay帧才可以被路由到目标FlexRay帧中，从而减少协议开销。



## 5.2. 模块处理

总线镜像模块通过**Mirror_Init**函数进行模块初始化，通过**Mirror_DeInit**进行反初始化。除了**Mirror_GetVersionInfo**和**Mirror_Init**之外，总线镜像模块的API函数只能在模块正确初始化后被调用。

为了能够测量时间，总线镜像模块通过Mirror_MainFunction循环触发。



### 5.2.1. 源总线的选择（Selection of Active Source Buses）

在初始化时，总线镜像模块将处于非活动状态。没有启用源总线（**source bus**）。要启动总线镜像模块，必须激活一个已配置的源总线（参见MirrorSourceNetwork），这样才能开始收集来自源总线的帧和状态信息。

当使用**Mirror_StartSourceNetwork**使能源总线（**source bus**）时，从该总线获取的帧和状态将被启动，源总线（**source bus**）的状态将被重置，以便在第一次更新后直接报告。

当一个源总线（**source bus**）使用**Mirror_StopSourceNetwork**被禁用时，从该总线获取的帧和状态将被停止。已经收集的帧仍应传输到目的总线（**destination bus**）。要停止镜像，应用程序可以在任何时候调用Mirror_Offline。

当**Mirror_Offline**被调用时，所有的源总线（**source bus**）应该停用，目的总线（**destination bus**）应该重置为**MirrorInitialDestNetworkRef**，所有静态配置的过滤器应该被禁用。同时所有其他的过滤器应该被移除。任何仍在等待传输的镜像帧将被丢弃。

当目标网络改变时，源总线（**source bus**）也被禁用。



### 5.2.2. 目的总线的切换（Switching the Destination Bus）

初始化时，**MirrorInitialDestNetworkRef**配置的目标总线（**MirrorDestNetwork**）会被选中。镜像启动前，需转发的帧和状态信息并不会被发送。

当使用**Mirror_SwitchDestNetwork**更改目的总线时，所有源总线（**source bus**）应被禁用，所有静态配置的过滤器应被禁用，同时所有其他过滤器应被移除。仍在等待传输的镜像帧将被丢弃。这确保了发送到目标总线的信息的选择必须针对该总线类型（**bus type**）进行选择。否则切换到不同的目的地总线（**destination bus**）很容易造成该总线超载（**overload**），特别是当它是另一个内部总线时。

镜像停止时，目标总线（**destination bus**）将被重置。



### 5.2.3. 控制帧的过滤（Controlling Frame Filters）

帧过滤器可以静态配置（**MirrorSourceCanFilter**、**MirrorSourceLinFilter**和**MirrorSourceFlexRayFilter**），也可以在运行时为每个源总线（**source bus**）进行动态配置。

在初始化时，总线镜像模块的所有静态配置的过滤器都被禁用，并且没有可用的动态过滤器。

静态配置的过滤器可以使用**Mirror_SetStaticFilterState**显式地激活和去激活。动态过滤器可以在运行时添加，使用总线特定的**Mirror_Add…Filter**服务（例如**Mirror_AddCanMaskFilter**），并可以通过使用**Mirror_Add...Filter**返回的过滤ID调用**Mirror_RemoveFilter**删除服务。当镜像停止或者目标网络被更改时，过滤器也将被禁用或者删除。

当一个过滤器是激活的（通过调用**Mirror_SetStaticFilterState**激活的静态配置，或者通过使用总线特定的**Mirror_Add…Filter**动态添加的服务），对应源总线上所有匹配过滤的帧都将被镜像。这意味着，只要没有过滤器处于活动状态，源总线就不会镜像任何帧。

当一个静态配置的过滤器被**Mirror_SetStaticFilterState**被停用，或者一个动态添加的过滤器被**Mirror_RemoveFilter**被移除，在停用/移除之前已经被接受的帧仍然会被镜像到目的总线（**destination bus**）。



## 5.3. 访问源总线

总线镜像模块支持CAN、LIN和FlexRay作为源总线（**source bus**）。为了获取这些总线的帧和状态信息，总线镜像模块与相应的总线接口模块进行交互。报告的帧在被镜像到目的总线之前会被过滤。总线镜像模块只能从同一个分区（**same partition**）调用CAN、LIN和FlexRay接口模块的接口，同时还必须是**MirrorSourceNetwork**配置所引用的那个**ComMChannel**所分配的那个通道。



### 5.3.1. 访问CAN的源总线

总线镜像模块通过CAN接口模块（**CanIf**）访问CAN总线。在总线镜像模块启动CAN总线的镜像后，CanIf模块将接收和发送的CAN帧上报给总线镜像模块。同时在**Mirror_MainFunction**函数中会周期的轮询CAN总线状态（**CAN bus state**）。



#### 5.3.1.1. CAN源总线激活

初始化后，CAN接口模块不会向总线镜像模块报告任何帧。

当**Mirror_StartSourceNetwork**被调用来启动CAN源总线时，总线镜像模块将调用**CanIf_EnableBusMirroring**，并将**MirroringActive**设置为TRUE，来开始报告从相应的CAN控制器接收和发送的CAN帧。

**Mirror_StartSourceNetwork**接收一个**ComMChannelId**参数作为网络（**network**），而**CanIf_EnableBusMirroring**接收一个**CanIfCtrlId**参数作为**ControllerId**。这两个参数的的转换可以在代码生成时，通过ECU配置**ComMChannelId**到**CanIfCtrlId**的引用来确定。

当**Mirror_StopSourceNetwork**被调用来停止CAN源总线时，总线镜像模块将调用**CanIf_EnableBusMirroring**，并将**MirroringActive**设置为**FALSE**，来停止从相应的CAN控制器接收和发送的CAN帧的报告。



#### 5.3.1.2. CAN报文采集

CAN接口模块（**CanIf**）通过调用**Mirror_ReportCanFrame**来报告接收和发送的CAN帧。从接收中断或任务中报告接收帧，而从传输确认中断或任务中报告发送帧。

总线镜像模块需要运用适当的机制来确保，**Mirror_ReportCanFrame**接口能在**MirrorComMNetworkHandleRef**引用的**ComMChannel**所分配到的那个分区中被调用。例如在该分区中提供一个卫星服务（**satellite**）。

对于上报的每一个CAN帧，CAN接口模块（**CanIf**）提供接收的CAN控制器、CAN ID、CAN ID类型（扩展帧或标准帧）、CAN帧类型（CAN-FD或CAN 2.0）、帧长度和实际负载信息。

当**Mirror_ReportCanFrame**被调用来报告接收或发送的CAN帧时，总线镜像模块需要将包含实际CAN ID、ID类型和帧类型的canId与相应源总线中所有活动的静态配置和动态添加的过滤器进行匹配。如果CAN帧至少匹配了一个过滤器，它会被总线镜像模块接受。

当镜像到FlexRay、IP或专有的目的总线时，源总线由**network ID**标识。但**Mirror_ReportCanFrame**报告确实**cotrollerID**。**network ID**到**controllerID**的转换，可以在代码生成时确定，通过ECU配置中的**MirrorComMNetworkHandleRef**里的**CanIfCtrlId**到**MirrorNetworkId**的引用来确定。



#### 5.3.1.3. CAN报文的过滤

CAN掩码过滤器（**CAN mask filter**）可以静态配置为**MirrorSourceCanFilterMask**的匹配报告的**canId**，如果该**canId**被**MirrorSourceCanFilterCanIdMask**屏蔽掩码计算后等于**MirrorSourceCanFilterCanIdCode**。

CAN掩码过滤器（**CAN mask filter**）也可以通过调用**Mirror_AddCanMaskFilter**动态添加用来匹配报告的**canId**，如果这个**canId**被**mask**参数屏蔽掩码计算后等于**id**参数。

CAN 范围过滤器（**CAN range filter**）可以静态配置为**MirrorSourceCanFilterRange**来匹配报告的**canId**，如果该**canId**的值大于等于**MirrorSourceCanFilterLower**，并且小于等于**MirrorSourceCanFilterUpper**时，匹配上报的**canId**。

CAN 范围过滤器（**CAN range filter**）也可以通过调用**Mirror_AddCanRangeFilter**动态添加用来匹配上报的**canId**，如果该**canId**的值大于等于**lowerId**参数，并且小于等于**upperId**参数。



#### 5.3.1.4. CAN状态采集

总线镜像模块通过循环调用**Mirror_MainFunction**中的**CanIf_GetControllerMode**和**CanIf_GetTrcvMode**来轮询每个被激活CAN源总线的状态。如果返回的**ControllerModePtr**为**CAN_CS_STARTED**，而**TransceiverModePtr**为**CANTRCV_TRCVMODE_NORMAL**，则上报的CAN源总线状态为在线（**online**），否则为离线（**offline**）。

如果总线处于在线（online）状态，总线镜像模块调用**CanIf_GetControllerErrorState**，如果返回的**ErrorStatePtr**为**CAN_ERRORSTATE_PASSIVE**或**CAN_ERRORSTATE_BUSOFF**，则上报的CAN源总线状态分别设置为**Passive**错误或**Bus-off**错误。同时如果总线是在线（online）的，总线镜像模块也应该调用**CanIf_GetControllerTxErrorCounter**，并将返回的**TxErrorCounterPtr**添加到报告的CAN源总线状态中。



### 5.3.2. 访问LIN的源总线

总线镜像模块通过LIN接口模块（**LinIf**）访问LIN总线。总线镜像模启动LIN总线的镜像后，LIN接口模块将接收和发送的LIN帧报告给总线镜像模块。部分的LIN总线的状态是与LIN帧内容一起被报告，部分状态时通过**Mirror_MainFunction**循环轮询。



#### 5.3.2.1. LIN源总线激活

初始化后，LIN接口模块（**LinIf**）不会向总线镜像模块报告任何帧。

**当Mirror_StartSourceNetwork**被调用来启动LIN源总线时，总线镜像模块应该调用**LinIf_EnableBusMirroring**并将**MirroringActive**设置为**TRUE**，来开始报告从该总线接收和发送的LIN帧。

当**Mirror_StopSourceNetwork**被调用来停止LIN源总线时，总线镜像模块应该调用**LinIf_EnableBusMirroring**并将**MirroringActive**设置为**FALSE**，来停止从该总线接收和发送LIN帧的报告。



#### 5.3.2.2. LIN帧采集

LIN接口模块通过调用**Mirror_ReportLinFrame**来报告接收和发送的LIN帧。在执行了相应的状态检查之后，接收和发送的帧被LIN调度处理报告。

总线镜像模块需要运用适当的机制，允许**MirrorComMNetworkHandleRef**引用的**ComMChannel**被分配到的分区中调用**Mirror_ReportCanFrame**。（例如在该分区中提供一个卫星服务（**satellite**）。

对于每个上报的LIN帧，LIN接口模块（**LinIf**）提供接收总线、受保护ID (**PID**)、帧长度、实际负载以及接收或传输状态等信息。

当**Mirror_ReportLinFrame**被调用来报告接收或发送的LIN帧时，总线镜像模块将从上报的PID中提取帧ID（Frame ID），并将其与相应源总线中所有静态配置和动态添加的活动过滤器进行匹配。如果LIN帧匹配至少一个过滤器，它被总线镜像模块接受。LIN帧的帧ID（Frame ID）是从PID中去掉两个最重要的位来计算的。



#### 5.3.2.3. LIN帧的过滤

LIN掩码过滤器（**LIN mask filter**）可以静态配置为**MirrorSourceLinFilterMask**用来匹配报告的**frame ID**，如果这个**frame ID**被**MirrorSourceLinFilterLinIdMask**掩码屏蔽计算后等于**MirrorSourceLinFilterLinIdCode**。

LIN掩码过滤器（**LIN mask filter**）也可以通过调用**Mirror_AddLinMaskFilter**动态添加用来匹配报告的**frame ID**，如果这个**frame ID**被**mask**参数掩码屏蔽计算后等于**id**参数。

LIN范围过滤器（**LIN range filter**）可以静态配置为**MirrorSourceLinFilterRange**用来匹配报告的**frame ID**，如果这个**frame ID**的值大于等于**MirrorSourceLinFilterLower，并且小于等于MirrorSourceLinFilterUpper**。

LIN范围过滤器（**LIN range filter**）也可以通过调用**Mirror_AddLinRangeFilter**动态添加用来匹配报告的**frame ID**，如果这个**frame ID**的值大于或等于**lowerId**参数，并且小于或等于**upperId**参数。



#### 5.3.2.4. LIN状态采集

总线镜像模块应评估**Mirror_ReportLinFrame报告的状态。如果是LIN_TX_HEADER_ERROR**、**LIN_TX_ERROR**、**LIN_RX_ERROR或LIN_RX_NO_RESPONSE**，则上报的LIN源总线状态应设置为报头传输错误（**header transmission error**）、传输错误（**transmission error**）、接收错误（**reception error**）或无响应（**no response**）。

总线镜像模块通过从**Mirror_MainFunction**循环调用**LinIf_GetTrcvMode**来轮询每个激活LIN源总线的状态。如果返回的**TransceiverModePtr**为**LINTRCV_TRCV_MODE_NORMAL**，则上报的LIN源总线状态应设置为在线（**online**），否则设置为离线（**offline**）。



### 5.3.3. 访问FlexRay的源总线

总线镜像模块通过FlexRay接口模块（**FrIf**）访问FlexRay总线。当总线镜像模块启动FlexRay总线的镜像后，FlexRay接口模块将接收到的FlexRay帧上报给总线镜像模块。FlexRay总线状态是通过**Mirror_MainFunction**循环轮询的。一个FlexRay源总线对应一个FlexRay集群，它可以连接到多个控制器。



#### 5.3.3.1. FlexRay源总线激活

初始化后，FlexRay接口模块不会向总线镜像模块报告任何帧。

当**Mirror_StartSourceNetwork**被调用来启动FlexRay源总线时，总线镜像模块应该调用**FrIf_EnableBusMirroring**，并将**FrIf_MirroringActive**设置为**TRUE**，开始报告从相应的FlexRay集群接收和发送的FlexRay帧。

**Mirror_StartSourceNetwork**收到一个**ComMChannelId**作为网络，而**FrIf_EnableBusMirroring**期望**FrIfClstIdx**作为**FrIf_ClstIdx**。**ComMChannelId**到**FrIf_ClstIdx**个的转换可以在代码生成时确定，通过通过ECU配置项的**ComMChannelId**引用到相关的**FrIfClstIdx**。

当**Mirror_StopSourceNetwork**被调用来停止FlexRay源总线时，总线镜像模块应该调用**FrIf_EnableBusMirroring**, 并将**FrIf_MirroringActive**设置为**FALSE**，停止从相应的FlexRay集群接收和发送的FlexRay帧的报告。



#### 5.3.3.2. FlexRay帧采集

FlexRay接口（**FrIf**）模块通过调用**Mirror_ReportFlexRayFrame**来报告接收和发送的FlexRay帧。接收和传输的帧由FlexRay接口（**FrIf**）的作业列表执行函数或传输函数报告。

总线镜像模块应应用适当的机制，允许**MirrorComMNetworkHandleRef**引用的**ComMChannel**被分配到的分区中调用**Mirror_ReportFlexRayFrame**接口。（例如在该分区中提供一个卫星服务（**satellite**）。

对于每一个上报的FlexRay帧，FlexRay接口模块（**FrIf**）会提供接收到的FlexRay控制器（**FlexRay controller**）、**Slot ID**和周期（**cycle**）、帧的长度和实际负载，以及传输冲突（**transmission conflicts**）的信息。

当**Mirror_ReportFlexRayFrame**被调用来报告一个接收或发送的FlexRay帧（**txConflict**被报告为FALSE）时，总线镜像模块应该匹配对应源总线中所有静态配置和动态添加的活动过滤器的**slotId**和**cycle**参数。如果FlexRay帧匹配至少一个过滤器，它被总线镜像模块接受。

在目标总线上，源总线由网络ID（**Network ID**）标识，但**Mirror_ReportFlexRayFrame**报告控制器ID（**Controller ID**）。一个到另一个的转换可以在生成时通过遵循从**FrIfCtrlIdx**到**MirrorNetworkId**的引用，通过通过**MirrorComMNetworkHandleRef**的**ECU**配置来确定

在目标总线（**destination bus**）上，源总线由网络ID（**network ID**）标识，但**Mirror_ReportFlexRayFrame**却报告是**controllerId**。**controllerId**到**network ID**的转换可以在代码生成时，通过ECU配置的**MirrorComMNetworkHandleRef**里的**FrIfCtrlIdx**到**MirrorNetworkId**的引用来确定。



#### 5.3.3.3. FlexRay帧过滤器

FlexRay过滤器（**FlexRay filter**）可以静态配置**MirrorSourceFlexRayFilter**来匹配被报告的**slotId**和**cycle**参数。匹配算法为**slotId**参数大于或等于**MirrorSourceFlexRayFilterLowerSlot**参数，并且小于或等于**MirrorSourceFlexRayFilterUpperSlot**。同时**cycle**参数以**MirrorSourceFlexRayFilterCycleRepetition**取模，大于或等于 **MirrorSourceFlexRayFilterLowerBaseCycle**，并且小于等于**MirrorSourceFlexRayFilterUpperBaseCycle**。

FlexRay滤波器（**FlexRay filter**）动态添加，可以调用**Mirror_AddFlexRayFilter**来匹配被报告的**slotId**和**cycle**参数。匹配算法为**slotId**参数大于或等于**lowerSlotId**，并且小于或等于**upperSlotId**；同时**cycle**参数以**cycleRepetition**取模，大于或等于**lowerBaseCycle**和小于或等于**upperBaseCycle**。



#### 5.3.3.4. FlexRay状态采集

当**Mirror_ReportFlexRayFrame**被调用来报告传输冲突（txConflict被报告为TRUE）时，总线镜像模块应该匹配所被激活的静态配置和动态添加的过滤器的**slotId**和**cycle**。如果它至少匹配一个过滤器，该帧所报告的FlexRay源总线状态应设置为传输冲突。

当**Mirror_ReportFlexRayChannelStatus**被调用来报告FlexRay通道状态时，总线镜像模块将报告的状态与之前报告的状态进行比较。如果第1位（vSS!SyntaxError）、第2位（vSS!ContentError）和第4位（vSS!Bviolation）的状态不同，总线镜像模块应该相应地更新所报告的FlexRay源总线状态。

总线镜像模块通过从**Mirror_MainFunction**循环调用**FrIf_GetState**来轮询每个活跃的FlexRay源总线的状态。如果返回的**FrIf_StatePtr**为**FRIF_STATE_ONLINE**，则报告的FlexRay源总线状态设置为在线（online），否则设置为离线（offline）。如果总线是在线的，总线镜像模块也需要为每个连接到FlexRay集群的控制器调用**FrIf_GetPOCStatus**。如果所有控制器返回的**Fr_POCStateType**为**FR_POCSTATE_NORMAL_ACTIVE**，则报告的源总线状态应为同步且正常激活。如果**Fr_POCStateType**对于至少一个控制器是**FR_POCSTATE_NORMAL_PASSIVE**，则报告的源总线状态应该是同步的，但不是正常活动的。如果**Fr_POCStateType**处于至少一个控制器的任何其他状态，则报告的源总线状态应该既不是同步的也不是正常活动的。



## 5.4. 镜像协议 (Mirroring Protocol)

总线镜像模块中，镜像协议（Mirroring Protocol）应用于IP、FlexRay和CDD连接的专有网络作为目的总线中。如图所示，在本例中，该协议用于以太网目的总线。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhq5IEuAdcnb0v5sKXBmltOIfzibL3S1uEh6ByTpY9WqGFF4icwPvBiaFtDqrI2V1GuvAic0AjG8Lqyxjw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

协议由一个协议头（Header）和几个数据项组成。

字节数的增加顺序与字节在目的总线上传输的顺序相同，并从0开始。对于字节中位的定义是，一个字节的最高有效位是第7位，最低有效位是0位。



### 5.4.1. 镜像协议头布局（Header）

每个目的帧都有一个协议头，如图所示。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhq5IEuAdcnb0v5sKXBmltOIQ7wBxJbVZwxmNlulK8ibQic0472FNCp5KUZuZZjqHyDU9eAExTLh7mlA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

总线镜像目标帧的协议头应按此顺序包含以下字段:

1. ProtocolVersion
2. SequenceNumber
3. HeaderTimestamp
4. DataLength



#### 5.4.1.1. 协议版本（ProtocolVersion）

协议版本用来表示协议头和数据项的布局。当前定义的布局固定使用1来标识**ProtocolVersion**。范围[2::127]保留给AUTOSAR标准，用来扩展未来协议定义。客户特定的协议可使用范围[128::255]来定义。

协议版本允许诊断仪（**Tester**）正确地解释协议，并启用协议的不同布局。

ProtocolVersion字段的宽度为8个Bit。



#### 5.4.1.2. 序列号（SequenceNumber）

SequenceNumber应该随着每一个目的帧的传输而增加。在初始化后或在Mirror_SwitchDestNetwork切换到新的目标总线后，SequenceNumber应该从0重新开始计数。

SequenceNumber允许诊断仪（**Tester**）识别丢失的目标帧。

SequenceNumber字段的宽度为8个Bit。这意味着SequenceNumber将在达到255后绕圈到0。诊断仪（**Tester**）必须能够处理这种行为，并且仍然能够正确地对帧进行排序。



#### 5.4.1.3. 协议头时间戳（HeaderTimestamp）

HeaderTimestamp应该反映数据项收集到目的帧开始的时间。这个时间是从1970年1月1日以来的绝对秒数和纳秒数。

HeaderTimestamp字段的宽度应为10字节，布局如图所示。HeaderTimestamp字段的元素应按网络字节顺序（MSB优先）进行编码。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhq5IEuAdcnb0v5sKXBmltOIibuUK3z6iad7mK7jfgOLwicpc9LQwFFa5kyNckAr0TDvPX0KndNkZNZ5w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



#### 5.4.1.4. 数据长度（DataLength）

DataLength应该给出报头后面的字节数。它是目标帧中所有数据项的长度之和。

DataLength字段的宽度应为16位。它应该以网络字节顺序（MSB优先）进行编码。



### 5.4.2. 镜像协议数据项的布局（Data Item Layout）

每个源帧被放置在一个数据项中，如图7所示。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhq5IEuAdcnb0v5sKXBmltOItxNudXShmnISuwbg7wDB7l75icNMs4bcF3I4ozH24uuzuBzM1s8L37w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

总线镜像目的帧的数据项应按此顺序包含以下字段：

1. Timestamp
2. NetworkStateAvailable
3. FrameIDAvailable
4. PayloadAvailable
5. NetworkType
6. NetworkID
7. NetworkState (optional)
8. FrameID (optional)
9. PayloadLength (optional)
10. Payload (optional)



#### 5.4.2.1. 时间戳（Timestamp）

时间戳（**Timestamp**）应该反映源帧接收到HeaderTimestamp的时间偏移量，即从开始收集数据项到目的帧的总时间。它的单位将是**10us**的倍数。

**Timestamp**字段的宽度应为16位。它应该以网络字节顺序(MSB优先)进行编码。



#### 5.4.2.2. NetworkStateAvailable

**NetworkStateAvailable**表示网络状态（**NetworkState**）字段是否存在于数据项中。如果**NetworkStateAvailable**为**1**，则存在该字段。如果为**0**，则该字段信息需要被省略。

**NetworkStateAvailable**字段的宽度应为1位。



#### 5.4.2.3. FrameIDAvailable

**FrameIDAvailable**表示**FrameID**字段是否存在于数据项中。如果**FrameIDAvailable**为**1**，则存在该字段。如果为**0**，则该字段信息需要被省略。

**FrameIDAvailable**字段的宽度应为1位。



#### 5.4.2.4. PayloadAvailable

**PayloadAvailable**表示**Payload**和**PayloadLength**字段是否存在于数据项中。如果**FrameIDAvailable**为**1**，则存在这些字段。如果为**0**，则这些字段信息需要被省略。

**PayloadAvailable**字段的宽度应为1位。



#### 5.4.2.5. NetworkType

**NetworkType**表示源总线的类型。NetworkType字段的宽度应为5位，可能的值如表所示。范围[5::15]预留给AUTOSAR自有协议的未来扩展，范围[16::31]可用定义客户特定的总线类型。

| Network Type | Numerical |
| ------------ | --------- |
| Invalid      | 0         |
| CAN          | 1         |
| LIN          | 2         |
| FlexRay      | 3         |
| Ethernet     | 4         |



#### 5.4.2.6. NetworkID

**NetworkID**应唯一地标识特定**NetworkType**的总线，即相同的**NetworkID**可以出现在不同的**NetworkType上**，但不能出现在相同的**NetworkType**上。

**NetworkID**字段的宽度应为8位。



#### 5.4.2.7. NetworkState

**NetworkState**应提供有关源总线状态的信息。只有当源总线的状态自上次被报告以来发生变化时，它才会出现，该状态应由NetworkStateAvailable表示。

**NetworkState**字段的宽度应为8位，布局是总线特定的.每个总线分别定义为**NetworkStateCAN**、**NetworkStateLIN**和**NetworkStateFlexRay**。

**NetworkState**的第7位（最高位）应该始终包含帧丢失（**Frames Lost**）状态。这是一个不定时发生的错误（**sporadic error**），与同一数据项中报告的源帧无关，但不应在单独的数据项中报告。当一个或多个通过过滤器的源帧，因为目的总线的队列满或传输失败而丢失后，帧丢失状态应该被设置为1。接着再次设置为0。

**NetworkState**的第6位总是包含总线在线状态（**Bus Online State**）。这是一种连续状态（**continuous state**），与在同一数据项中报告的源帧无关。即使在**FrameIDAvailable**和**PayloadAvailable**字段设置为0是，它仍然会报告在数据项中。当源总线在线时，即控制器和收发器都能通信时，总线在线状态设置为1。否则设置为0。



##### 5.4.2.7.1. NetworkStateCAN

CAN总线的网络状态（**NetworkStateCAN**）布局如表所示。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhq5IEuAdcnb0v5sKXBmltOINud4vXRQZHZy3YRClKiaqicFAlJThKWBdOe61w466GgmicKDwiblbLZxlQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

NetworkStateCAN的第5位应该包含错误被动状态（**Error-Passive state**）。这是一种连续状态，与在同一数据项中报告的源帧无关。也可能在**FrameIDAvailable**和**PayloadAvailable**字段设置为0的数据项中报告。
当CAN控制器处于错误被动状态（**Error-Passive state**）时，应将无错误状态设置为1。当它处于错误主动（**Error-Active**）或**Bus-Off**状态时，值为0。

**NetworkStateCAN**的第4位应该包含**Bus-Off**状态。这是一种连续状态，与在同一数据项中报告的源帧无关，也可能在**FrameIDAvailable**和**PayloadAvailable**字段设置为0的数据项中报告。当CAN控制器处于**Bus-Off**状态时，Bus-Off状态应设置为1，当CAN控制器处于主动错误（**Error-Active**）或被动错误（**Error-Passive**）状态时，Bus-Off状态应设置为0。

NetworkStateCAN的第0位到底3位，应该包含可以控制器的发送（**Tx**）错误计数器，数值位Tx错误计算除以8。这是一种连续状态，与在同一数据项中报告的源帧无关，也可能在**FrameIDAvailable**和**PayloadAvailable**字段设置为0的数据项中报告。



##### 5.4.2.7.2. NetworkStateLIN

LIN总线的网络状态（**NetworkStateLIN**）布局如表所示。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhq5IEuAdcnb0v5sKXBmltOIY7UmIicXicuPPGic4kiaHolwI4mQylcRI40UZIibiaLsw5mpic4cufktE9iaqg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**NetworkStateLIN**的第5位和第4位目前被保留。它们总是被设为0。

**NetworkStateLIN**的第3位应该包含报头发送错误状态（**Header Tx Error state**）。这是一个与同一数据项中报告的源帧相关的错误。当LIN控制器在LIN报头传输过程中检测到错误时，发送错误状态（**Header Tx Error state**）应设置为1。否则设置为0。

**NetworkStateLIN**的第2位应包含发送错误状态（**Tx Error state**）。这是一个与同一数据项中报告的源帧相关的错误。当LIN控制器在传输LIN帧期间检测到传输错误时，发送错误状态（**Tx Error state**）应设置为1。否则设置为0。

**NetworkStateLIN**的第1位应包含接收错误状态（**Rx Error state**）。这是一个与同一数据项中报告的源帧相关的错误。当LIN控制器在接收LIN帧期间检测到接收错误时，接受错误状态（**Rx Error state**）应设置为1。否则设置为0。

**NetworkStateLIN**的第0位应包含报头接收无响应状态（**Header Rx No Response state**）。这是一个与同一数据项中报告的源帧相关的错误。当LIN控制器在传输LIN报头后没有收到预期的LIN帧时，报头接收无响应状态（**Header Rx No Response state**应设置为1。否则设置为0。



##### 5.4.2.7.3. NetworkStateFlexRay

FlexRay总线的网络状态（**NetworkStateFlexRay**）布局如表所示。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhq5IEuAdcnb0v5sKXBmltOI6vBibYQs6qBcthua3DnibVLAJ5ibNiaV0Al0tkP5Ch4SSBsibfcrIOTMQpA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**NetworkStateFlexRay**的第5位应该包含总线同步状态（**Bus Synchronous state**）。这是一种连续状态，与在同一数据项中报告的源帧无关，也可能在**FrameIDAvailable**和**PayloadAvailable**字段设置为0的数据项中报告。当所有连接到总线上的FlexRay控制器都与网络时间同步时，总线同步状态应设置为1。否则设置为0。

**NetworkStateFlexRay**的第4位应包含正常激活状态（**Normal Active state**）。这是一种连续状态，与在同一数据项中报告的源帧无关，也可能在**FrameIDAvailable**和**PayloadAvailable**字段设置为0的数据项中报告。当所有连接到总线的FlexRay控制器都是同步的并且处于正常激活状态时，正常激活状态（**Normal Active state**）应该设置为1。否则设置为0。

**NetworkStateFlexRay**的第3位应该包含语法错误状态（**Syntax Error state**）。这是一个FlexRay通道的聚合的错误标志，与通道分配的**FrameID**有关，但不是源帧（**Source Frame**）和它的FrameID报告在同一数据项。它也可以在数据项中报告，其中**PayloadAvailable**字段设置为0，**FrameIDAvailable**设置为1，而FrameID的Slot有效标志（Slot Valid Flag）设置为0。当FlexRay控制器检测到语法错误时，（**Syntax Error state**）应该设置为1。否则设置为0。

**NetworkStateFlexRay**的第2位应包含内容错误状态（**Content Error state**）。这也是一个FlexRay通道的聚合的错误标志，与通道分配的**FrameID**有关，但不是源帧（**Source Frame**）和它的FrameID报告在同一数据项。也可以在数据项中报告，其中**PayloadAvailable**字段设置为0，**FrameIDAvailable**设置为1，而FrameID的Slot有效标志（Slot Valid Flag）设置为0。当FlexRay控制器检测到内容错误时，内容错误状态（**Content Error state**）应该设置为1。否则设置为0。

**NetworkStateFlexRay**的第1位应包含边界违反状态（**Boundary Violation state**）。这是一个FlexRay通道的聚合的错误标志，与通道分配的**FrameID**有关，但不是源帧**Source Frame**）和它的**FrameID**报告在同一数据项。它也可以在数据项中报告，其中**PayloadAvailable**字段设置为0，**FrameIDAvailable**设置为1，而FrameID的Slot有效标志（Slot Valid Flag）设置为0。当FlexRay控制器检测到边界违规时，边界违规状态（**Boundary Violation state**应该设置为1。否则设置为0。

**NetworkStateFlexRay**的第0位应包含Tx冲突状态（**Tx Conflict state**）。这是一个与使用相同FrameID报告的前一个源帧相关的错误，并且总是在一个数据项中报告，其中**FrameIDAvailable**字段设置为1，**PayloadAvailable**字段设置为0。当FlexRay控制器检测到传输冲突时，Tx冲突状态（**Tx Conflict state**）应该设置为1。否则设置为0。



#### 5.4.2.8. FrameID

**FrameID**应提供源帧（**Source Frame**）的标识。此标识对于一个由**NetworkType**和**NetworkID**标识的源总线（**Source Bus**）应该是唯一的。当报告源总线状态改变时，FrameID可以被省略，**FrameID**存在与否应由**FrameIDAvailable**来表示。

**FrameID**字段的宽度和布局是总线特定的，针对不同的总线类型，分别定义了**FrameIDCAN**、**FrameIDLIN**和**FrameIDFlexRay**。



##### 5.4.2.8.1. FrameIDCAN

CAN总线的FrameID布局如表7.6.

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhq5IEuAdcnb0v5sKXBmltOIzKGS8Fk0iaSZiclt48wqb7ia5JkVIf9gY1gByBJj8zyGDuURH7vZbKzqA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**FrameIDCAN**的布局对应于**Mirror_ReportCanFrame**提供的Can_IdType。**FrameIDCAN**字段的宽度应为4字节。

对于扩展CAN ID（**Extended CAN ID**），第0字节的第7位应该设置为1，对于标准CAN ID （**Standard CAN ID**），应该设置为0。**FrameIDCAN**的第0字节的第6位表示是否是CANFD帧，对应CANFD帧格式设置为1，对于CAB 2.0帧应该设置为0。**FrameIDCAN**的第0字节的第5位目前被保留。它总是被设为0。FrameIDCAN的字节0的第0位到第4位以及字节1到字节3，包含了以网络字节顺序（MSB优先）报告的CAN帧的CAN ID。



##### 5.4.2.8.2. FrameIDLIN

LIN总线的FrameID布局如表所示

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhq5IEuAdcnb0v5sKXBmltOIypMmTpBjO4ibJvCnxCicoOsEk8TUn8d4eSUicp1umX1LJJ04P0Do1meQA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**FrameIDLIN**字段的宽度应为1字节。**FrameIDLIN**的字节0应该包含报告LIN帧的LIN PID。



##### 5.4.2.8.3. FrameIDFlexRay

**FlexRay**总线的**FrameID**布局如表所示。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhq5IEuAdcnb0v5sKXBmltOIKlEN6kOBT2VjUGzS6tFq7d9zwU0RiacP5ffxa3KSOcXzVcmNZjNk6Bg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**FrameIDFlexRay**字段的宽度应为3字节。

**FrameIDFlexRay**的第0字节的第6位到第7位应该包含报告的FlexRay帧的通道分配。如果报告的FlexRay帧在FlexRay控制器的B通道上可用，则第7位设置为1，否则设置为0。如果报告的FlexRay帧在FlexRay控制器的A通道上可用，则第6位设置为1，否则设置为0。一个报告的FlexRay帧要么被分配给通道A或B，要么被分配给两个通道。这个通道分配的布局对应于**Mirror_ReportFlexRayFrame**报告的**Fr_ChannelType**。

**FrameIDFlexRay**的第0字节的第4位到第5位目前被保留。它们总是被设为0。

**FrameIDFlexRay**字节0第3位应当包含一个标志，指示是否报道Solt ID和周期是否有效的（标志为1），或者未使用（标志为0）。当一个聚合的FlexRay通道错误报告的独立源或传输帧冲突，它必须设置为0,。否则它总是设置为1。

**FrameIDFlexRay**字节0和字节1的第0位到第2位，包含所报告的FlexRay帧的Slot ID，按网络字节顺序(MSB优先)。

**FrameIDFlexRay**的字节2包含了发送或接收所报告的FlexRay帧的周期。对于接收到的帧和在静态段发送的帧，周期总是可靠的。对于在动态段中发送的帧，由于可能不会在计划的周期内发送，因此无法提前知道实际的周期。



#### 5.4.2.9. PayloadLength

**PayloadLength**应提供源帧的有效载荷长度。当报告源总线状态变化时，可以省略。它的存在与否通过**PayloadAvailable**来表示。

**PayloadLength**字段的宽度应为8位。



#### 5.4.2.10. Payload （有效负载）

有效负载（Payload）应提供源帧的实际有效载荷。当报告源总线状态变化时，可以省略。它的存在与否通过**PayloadAvailable**来表示。

有效负载字段的宽度应与报告的源帧相对应。LIN和CAN 2.0的最大值为8字节，CAN-FD的最大值为64字节，FlexRay的最大值为254字节。



## 5.5. 镜像到FlexRay、IP和CDD

当镜像到FlexRay目的总线、IP目的总线（如以太网）或作为CDD连接的专有网络时，总线镜像模块应用协议将几个较小的帧打包成目标总线的一个大帧。



### 5.5.1. 创建

当总线镜像模块初始化或当Mirror_SwitchDestNetwork叫调用来激活FlexRay（**MirrorDestNetworkFlexRay**）, IP（**MirrorDestNetworkIp**），或专有的（**MirrorDestNetworkCdd**）为目的地总线，总线镜像模块将激活一个新的目的地帧缓冲器和重置**SequenceNumber**为0。

当第一个数据项被添加到一个空的目的帧缓冲器，总线镜像模块应首先将报头写入缓冲器。**ProtocolVersion**字段设置为1，**SequenceNumber**设置为上一帧目的帧的**SequenceNumber**加1, **HeaderTimestamp**填充**StbM_GetCurrentTime**返回的信息，**DataLength**字段设置为0。

如果配置了可选配置参数**MirrorDestTransmissionDeadline**，总线镜像模块将启动传输超时定时器。

当源帧已经收到CAN帧, LIN帧，或者FlexRay帧，总线镜像模块创建一个新的数据项，并将其作为当前活动的最后目的地帧缓冲区定义的布局和应当添加到目的地帧缓冲器里，同时把新增的数据项的大小设定到报头字段**DataLength**字段中。

新数据的时间戳（Timestamp）字段应设置为报头中包含时间戳和当前通过StbM_GetCurrentTime获得时间的差，单位为10us的倍数，**FrameIDAvailable**和**PayloadAvailable**位设置为1，并且**NetworkType**、**NetworkID**、**FrameID**、**PayloadLength**和**Payload**应根据接收到的源帧进行设置。

如果报告的源总线状态在源帧最后一次传输后发生了变化，**NetworkStateAvailable**位应该设置为1,**NetworkState**字段应该设置为报告的源总线状态。否则，**NetworkStateAvailable**位应设为0，**NetworkState**字段应省略。

当一个新的**FlexRay**传输冲突（Tx Conflict）被报告，总线镜像模块应当创建一个新的数据项，并将其在当前活动的最后目的地帧缓冲添加新的数据段，同时添加新的数据项的大小到报头的**DataLength**字段。

新数据的时间戳（Timestamp）字段应设置为报头中包含时间戳和当前通过StbM_GetCurrentTime获得时间的差，单位为10us的倍数。**FrameIDAvailable**和**NetworkStateAvailable**位设置为1，和**NetworkType**、**NetworkID**和**FrameID**应根据上报的传输冲突进行设置。**NetworkState**字段应设置为所报告的源总线状态。

**PayloadAvailable**位设置为0，**PayloadLength**和**Payload**字段应省略。

每报告一个FlexRay传输冲突，前一个FlexRay帧就会失效。无效的FlexRay帧可能位于另一个目标帧，而不是相应的传输冲突。

当报道源总线状态（**source bus state**）改变了，在一个主功能周期里，相同的源总线（**source bus**）没有收到任何源帧（**source frame**），总线镜像模块应当创建一个新的数据项，并将其添加到当前活动的目的地帧缓冲最后，并将新数据项的大小，添加到报头**DataLength**字段。

新数据的时间戳（Timestamp）字段应设置为报头中包含时间戳和当前通过StbM_GetCurrentTime获得时间的差，单位为10us的倍数。**NetworkStateAvailable**位设置为1，**NetworkType**和**NetworkID**字段根据上报的源总线设置，**NetworkState**字段设置为上报的源总线状态。

根据当前报告的源总线状态，**FrameIDAvailable**应该设置为1或0。在第一种情况下，**FrameID**应根据报告的源总线设置；在后一种情况下，FrameID应省略。

**PayloadAvailable**位设置为0，**PayloadLength**和**Payload**字段应省略。



### 5.5.2. 队列

当一个数据项不适合当前活动的目标帧缓冲区的剩余空间时，总线镜像模块应该将这个缓冲区放入队列并激活一个新的目标帧缓冲区。然后数据项应放置在新的缓冲区中。

当数据项的相对时间戳超过655.35ms时，总线镜像模块应将当前活动的目标帧缓冲区放入队列中，并激活一个新的目标帧缓冲区。然后数据项应放置在新的缓冲区中。

如果可选的配置参数**MirrorDestTransmissionDeadline**被配置并且传输超时超时，总线镜像模块将把当前激活的目标帧缓冲区放到队列中，并激活一个新的目标帧缓冲区。

序列化的目的帧的队列大小由配置参数**MirrorDestQueueSize**决定，队列元素的大小由**MirrorDestPduRef**引用的**Pdu**的**PduLength**决定。

如果目标帧因为队列已经满而不能放入队列中，总线镜像模块将丢弃该目标帧，并报告镜像运行时错误**MIRROR_E_QUEUE_OVERRUN**，并且设置当前活动的目标帧缓冲区中创建的下一个数据项的**NetworkState**的Frame Lost位为1。



### 5.5.3. 传输

为了启动已经队列化并且序列化的目的帧的发送，总线镜像模块需要调用**PduR_MirrorTransmit**函数, 同时**PduInfoPtr->MetaDataPtr**参数设置为**NULL_PTR**, **PduInfoPtr->SduLength**设置为目的帧实际写入的部分。如果**MirrorDestPduUsesTriggerTransmit**配置被使能, 那么**PduInfoPtr->SduDataPtr**参数必须设置为**NULL_PTR**，否则设置为队列目的帧被使用的部分。

设置**PduInfoPtr->SduDataPtr**参数为**NULL_PTR**，能够确保目的总线接口模块（**FrIf**、**SoAd**或**CDD**）使用**Mirror_TriggerTransmit**来获取目的帧。

如果**PduR_MirrorTransmit**返回**E_NOT_OK**错误，总线镜像模块应该立即从队列中删除目的帧，报告运行时错误**MIRROR_E_TRANSMIT_FAILED**，并且在当前活动的目的帧缓冲区中创建的新的数据项作为下一个数据项，同时把数据项中的**NetworkState**的**Frame Lost**位设置为1。

总线镜像模块应该从**Mirror_MainFunction**和**Mirror_TxConfirmation**回调函数初始化队列化序列化的目标帧的传输。这确保了队列目标帧的传输速度尽可能快。为了在FlexRay目的总线上实现一个合适的吞吐量，**MirrorDestNetworkFlexRay**可能包含一组**MirrorDestPdu**。

如果一个**MirrorDestNetworkFlexRay**配置了一组**MirrorDestPdu**，总线镜像模块可以按照任意顺序使用该组的**PDU**。数据项的**SequenceNumber**和**Timestamp**字段将确保诊断仪（**Tester**）能够正确地对它们进行排序。

如果激活的目标通道是**MirrorDestNetworkIp**或**MirrorDestNetworkCdd**，总线镜像模块不会在通过调用**Mirror_TxConfirmation**未确认上一个目标帧已传输成功之前，传输下一个序列化的目标帧。

如果激活的目的通道是**MirrorDestNetworkFlexRay**，总线镜像模块不会在通过调用**Mirror_TxConfirmation**未确认上一个使用**MirrorDestPdu**目标帧已传输成功之前，使用相同的**MirrorDestPdu**发送下一个序列化的目的帧。

当一个序列化的目的帧被调用**Mirror_TriggerTransmit**时，总线镜像模块将队列中的目的帧被使用的部分拷贝到**PduInfoPtr->SduDataPtr**中，并相应地更新**PduInfoPtr->SduLength**。

如果**Mirror_TriggerTransmit**提供的**PduInfoPtr->SduLength**对于当前传输的序列化目的帧来说太小，总线镜像模块需要将目的帧从队列中移除，并报告运行时错误**MIRROR_E_TRANSMIT_FAILED**。在当前活动的目的帧缓冲区中创建的新的数据项作为下一个数据项，同时把数据项中的**NetworkState**的**Frame Lost**位设置为1。并且应该返回**E_NOT_OK**来停止这个传输。

当调用**Mirror_TxConfirmation**来报告一个序列化的目的帧的成功或失败传输时，总线镜像模块应该从队列中删除目的帧。

如果**Mirror_TxConfirmation**报告了一个序列化的目的帧传输失败（结果是**E_NOT_OK**），总线镜像模块应该报告运行时错误**MIRROR_E_TRANSMIT_FAILED**，在当前活动的目的帧缓冲区中创建的新的数据项作为下一个数据项，同时把数据项中的**NetworkState**的**Frame Lost**位设置为1。



## 5.6. 镜像到CAN

当镜像到CAN目的总线时，总线镜像模块将接收到的CAN和LIN帧直接发送到目的总线，尽管可能会更改CAN ID，以避免与目的总线上的常规消息冲突。

本章定义了总线镜像模块如何转换CAN id和队列源帧，以及如何创建和队列状态帧，然后在目的总线上传输它们。



### 5.6.1. 源帧的处理



#### 5.6.1.1. ID映射

通常情况下，CAN源帧可以不用任何改变就能在目的总线上进行传输，而LIN源帧的**PID**需要映射到一定范围的**CAN ID**。但有时很难找到连续的未使用的**CAN ID**序列来映射**LIN PID**。或者传输的帧也使用相同的**CAN ID**在目的CAN总线上已经被使用。在这种情况下，这些**CAN ID**和**LIN PID**必须映射到另一个特殊的**CAN ID**。



##### 5.6.1.1.1. CAN上的ID映射

如果CAN源帧的**canId**与**MirrorSourceCanSingleId**的**MappingSourceCanId**相匹配，则CAN目的帧（**CAN destination frame**）就需要使用转换成**MirrorSourceCanSingleIdMappingDestCanId**后再传输。

如果CAN源帧的**canId**与**MirrorSourceCanMaskBasedIdMapping**配置的**MirrorSourceCanMaskBasedIdMappingSourceCanIdMask**屏蔽计算（**Masked**）和**MirrorSourceCanMaskBasedIdMappingSourceCanIdCode**相匹配，则CAN目的帧（**CAN destination frame**）需要将经过屏蔽计算的**canId**加上**MirrorSourceCanMaskBasedIdMappingDestBaseId**后再进行传输。

如果CAN源帧的**canId**与**MirrorSourceCanSingleIdMapping**和**MirrorSourceCanMaskBasedIdMapping**都不匹配，则CAN目的帧（**CAN destination frame**）需要使用原始的canID来进行传输。相同的CAN ID：包括ID类型（扩展或标准）和帧类型（CAN-FD或CAN 2.0）。



##### 5.6.1.1.2. LIN上的ID映射

如果从LIN源帧的**PID**中提取的**Frame ID**与**MirrorSourceLinToCanIdMapping**的**MirrorSourceLinToCanIdMappingLinId**相匹配，则该CAN目的帧需是使用**MirrorSourceLinToCanIdMappingCanId**进行传输。

如果从LIN源帧的**PID**中提取的**Frame ID**与**MirrorSourceLinToCanIdMapping**不匹配，则在发送CAN目的帧时，需要将**LIN Frame ID**加上**MirrorSourceLinToCanBaseId**中后再进行传输。



### 5.6.2. 排队

总线镜像模块应将所有的CAN目的帧放入队列中。CAN目的帧的队列大小由配置参数**MirrorDestQueueSize**决定，队列元素的大小由**MirrorDestPduRef**引用的**Pdu**的**PduLength**决定。

如果目标帧因为队列已经满而不能放入队列，总线镜像模块将丢弃该目标帧，并报告运行时错误（**MIRROR_E_QUEUE_OVERRUN**），并且设置当前活动的目标帧缓冲区中创建的下一个数据项的**NetworkState**的**Frame Lost**位为1。



### 5.6.3. 传输

为了发起一个排队的CAN目的帧的传输，总线镜像模块需要调用**PduR_MirrorTransmit**, **PduInfoPtr->MetaDataPtr**设置为包含目的帧**CAN ID**的元数据，**PduInfoPtr->SduLength**设置为目的帧的长度。如果启用了**MirrorDestPduUsesTriggerTransmit**, **PduInfoPtr->SduDataPtr**必须设置为**NULL_PTR**，否则为源帧的负载。

**PduInfoPtr->SduDataPtr**的**NULL_PTR**确保目标总线接口模块（**CanIf**）使用**Mirror_TriggerTransmit**来获取目标帧。

如果**PduR_MirrorTransmit**返回**E_NOT_OK**，总线镜像模块应该立即从队列中删除目的帧，应该报告运行时错误（**MIRROR_E_TRANSMIT_FAILED**），并且设置当前活动的目标帧缓冲区中创建的下一个数据项的**NetworkState**的**Frame Lost**位为1。

总线镜像模块应该从**Mirror_MainFunction**和**Mirror_TxConfirmation**回调函数开始传输队列中的CAN目标帧。这确保了队列目标帧的传输速度尽可能快。

在上一个目的帧被**Mirror_TxConfirmation**调用确认之前，总线镜像模块不能传输下一个CAN目的帧。

当CAN目的帧调用**Mirror_TriggerTransmit**时，镜像模块将源帧的负载复制到PduInfoPtr->SduDataPtr上，并更新PduInfoPtr->SduLength。

CAN总线，它是不可能**Mirror_TriggerTransmit**提供**PduInfoPtr->SduLength**太小的问题，因为目标帧已经为CAN 2.0配置8字节，为CAN-FD配置64字节，同时**CanIf**总是提供相应的硬件缓冲区大小。

当**Mirror_TxConfirmation**被调用来报告CAN目的帧的成功或失败传输时，总线镜像模块应该将目的帧从队列中移除。

如果**Mirror_TxConfirmation**报告CAN目的帧传输失败（结果是**E_NOT_OK**），总线镜像模块应报告运行错误（**MIRROR_E_TRANSMIT_FAILED**），并且设置当前活动的目标帧缓冲区中创建的下一个数据项的**NetworkState**的**Frame Lost**位为1。