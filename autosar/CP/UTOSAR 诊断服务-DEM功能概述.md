# UTOSAR 诊断服务-DEM功能概述

**前言**

AUTOSAR DEM模块的分享分为DEM模块概念详解和Dcm模块配置及代码分析，具体的项目实战请关注本号的后续文章，本篇为**DEM模块概念介绍篇--功能概述**。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwicfwE2vricYVdicq6FU9ichxj6VpSrIkfbS07Ldpz77RN7DcJT2nRUmPDVGFN9xcY0wC8dyuGW8VyEA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**正文**

# **1****.功能概述**

服务组件诊断事件管理器(Dem)负责处理和存储诊断事件(错误)和相关数据。此外，Dem向Dcm提供故障信息(例如，从事件内存中读取所有存储的dtc)。Dem提供到应用层和其他BSW模块的接口。如DCM可以通过

*Dem_ReturnGetStatusOfDTCType Dem_DcmGetStatusOfDTC(*

*uint32 DTC,*

*Dem_DTCOriginType DTCOrigin,*

*uint8\* DTCStatus)*

这个标准接口同步或异步的读取到指定的DTC的故障状态。

 

# **2.****关键概念理解**

**Aging**：在定义的操作周期数后从事件存储器中取消学习/删除不再失败的事件/DTC--自愈功能。

 

**Aging Counter**：“Aging Counter--化计数器”或“Aging Cycle Counter--老化周期计数器”或“DTC Aging Counter --DTC老化计数器”指定用于执行老化的计数器。它计算操作周期数，直到从事件存储器中删除事件/DTC。

 

**Debounce counter**：基于计数器的去抖动算法的内部计数器。

 

**DTC group**：唯一标识一组 DTC。DTC 组被映射到有效 DTC 的范围内。通过提供一组 DTC，表示对该组的所有 DTC 请求某个操作。DTC 组定义由 ISO 14229-1 提供。

 

**Event combination**：事件组合是一种将多个事件合并到一个特定组合 DTC 的方法。用于使不同的监测结果适应一个重大故障，这在服务站中是可以明确评估的。

 

**Event debouncing**：去抖动是一种评估诊断事件是否合格的特定机制（例如基于计数器）。 这适用于潜在的信号去抖动，可以在 SW-C 或 Dem 内完成。

 

**Event confirmation**：如果通过故障确认计数器评估的循环或时间重复检测合格事件，则确认诊断事件。 因此，UDS 状态位 3（ConfirmedDTC）也被设置。

 

**Event memory**：一个事件存储器（例如主存储器）由多个事件存储器条目组成。

 

**Event memory entry**：事件存储器条目是事件及其事件相关数据的单个存储容器。 事件存储器条目动态地分配给特定事件。

 

**Event memory overflow indication**: 事件存储器溢出指示，如果该特定事件存储器已满并且下一个事件发生将存储在该事件存储器中。

 

**Event qualification**: SWC或者BSW模块检测到诊断事件passed或者failed。

 

**Event related data**： 事件相关数据是附加数据，例如 传感器值或时间戳/里程，与事件一起存储在事件存储器中。ISO 定义了两种类型的事件相关数据：冻结帧（快照记录）和扩展数据。

 

**Event status byte**：ISO 14229-1 [1] 中定义的状态字节，基于事件级别。

 

**Extended data record**：扩展数据记录是用于存储分配给故障的特定信息的记录。

 

**Failure counter**：失败计数器对发生故障的操作周期（驾驶周期）进行计数。 如果计数器达到阈值（例如，2 个驱动周期），则确认位从 0 变为 1。

 

**Freeze frame**：冻结帧被定义为数据记录（DID/PID）。冻结帧与 ISO-14229-1[2] 中的 SnapShotRecords 相同。

 

**Healing**：诊断事件确认后，如果一定周期或者事件后事件没有发生，则关闭警告指示器，包括在一段时间/几个操作周期内处理报告的通过结果。

 

**Operating cycle**：“操作周期”是事件确认的基础，也是 Dem 调度（例如点火钥匙断开周期、驾驶周期等）的基础。

 

**OBD**：车载诊断或 OBD 是一个通用术语，指的是车辆的自我诊断和报告能力。OBD 系统使车主或维修技术人员可以访问各种车辆子系统的健康状态信息。

 

**UDS status bit 0**：testFailed bit of the UDS status byte。表示最近执行的测试的结果。

 

**UDS status bit 1**：testFailedThisOperationCycle bit of the UDS status byte。指示诊断测试是否在当前操作周期内的任何时间报告了 testFailed 结果。

 

**UDS status bit 2**：pendingDTC bit of the UDS status byte。指示诊断测试是否在当前或上一次最后完成的操作周期中的任何时间报告了testFailed 结果。

 

**UDS status bit 3**：confirmedDTC bit of the UDS status byte。指示是否检测到故障的次数足够保证将 DTC 存储在长期存储器中。

 

**UDS status bit 4**：testNotCompletedSinceLastClear bit of the UDS status byte。指示自上次调用 ClearDiagnosticInformation 以来，DTC 测试是否曾经运行并完成。

 

**UDS status bit 5**：testFailedSinceLastClear bit of the UDS status byte。指示自上次调用 ClearDiagnosticInformation 以来 DTC 测试是否已完成但结果失败。

 

**UDS status bit 6**：testNotCompletedThisOperationCycle bit of the UDS status byte。指示 DTC 测试在当前操作周期内是否曾经运行并完成。

 

**UDS status bit 7**：warningIndicatorRequested bit of the UDS status byte。报告与特定 DTC 相关的任何警告指示器的状态。

 

**UDS status byte**：ISO 14229-1 [1] 中定义的状态字节，基于 DTC 级别。

 

# **3. DEM与其他模块交互**

从架构的角度看DEM，DEM处于BSW的服务层，见图一，与之交互的模块有SWC、BSW、ECUM、FIM、DCM、NVM，见图二。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwicfwE2vricYVdicq6FU9ichxjfsk60grfHRULUCmgia7QMlPlFG7H5A6ib0iaUXu7ct4mtMUdNW0z9PdIg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图一：mapping of basic software modules to AUTOSAR layers

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwicfwE2vricYVdicq6FU9ichxjg849VibqXjCS281eQRO9jOPp6ibRR9XyzPoMJRlyU3fZP0MTRSjniaiaMw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图二：dependencies of the Diagnostic Event Manager to other software modules

 

**DEM和****SWC**：应用层软件组件SWC会周期调用故障监控函数，并周期调用AUTOSAR标准接口*Std_ReturnType Dem_SetEventStatus(*

*Dem_EventIdType EventId,*

*Dem_EventStatusType EventStatus)*

把故障及状态通知到DEM，DEM会根据静态配置的debounce方式调用相对应的函数。然后根据debounce的结果决定是否把当前故障存储到fault memory中和触发FIM。

 

**DEM和****BSW**：基础软件BSW也会给DEM报故障，根据AUTOSAR规定，BSW通过调用AUTOSAR标准接口

*void Dem_ReportErrorStatus(*

*Dem_EventIdType EventId,*

*Dem_EventStatusType EventStatus)*

标准接口给DEM报故障，故障内容如NVM写入失败或者排队任务数溢出或者校验错误。该类故障在DEM中的debounce方式是no debounce，不需要debounce，所以故障状态只有DEM_EVENT_STATUS_FAILED或者DEM_EVENT_STATUS_PASSED。

 

**DEM和****ECUM**：ECUM主要负责在不同时序调用DEM的初始化工作，DEM初始化应包括对每个故障的debounce status做处理；初始化fault memory相关数据；初始化DEM中存储的BSW的故障数据。



**DEM和****FIM**：FIM全称function inhibition manager，主要负责给SWC提供一个控制机制，可以Enable或者Disable SWC的功能，如SWC中驱动电路短路检测功能，此时由于其该功能被抑制，SWC此时使用Dem_SetEventStatus报给DEM的故障状态是no condition。



另外在SWC调用Dem_SetEventStatus时，如果故障状态发生变化，DEM会通过调用AUTOSAR标准接口

*void FiM_DemTriggerOnEventStatus(*

*Dem_EventIdType EventId,*

*Dem_UdsStatusByteType EventStatusByteOld,*

*Dem_UdsStatusByteType EventStatusByteNew )*

来抑制SWC本身的功能和与之相关的事件的功能。

 

**DEM和****DCM****：**DCM和DEM之前有着密切关系，因为DCM中有ReadDTCInformation（0x19读取DTC信息服务）和ClearDiagnosticInformation（0x14清除DTC信息服务）这两个服务都是都是要从DEM中读取信息或者传递命令。比如根据DTC读冻结帧（快照信息）（19 04 xx xx xx yy），在19服务的自服务的处理函数里面，首先就要调用AUTOSAR标准接口

*Dem_ReturnGetStatusOfDTCType Dem_DcmGetStatusOfDTC(*

*uint32 DTC,*

*Dem_DTCOriginType DTCOrigin,*

*uint8\* DTCStatus)*

给DEM传递DTC和该DTC所属的memory类型，来获得DTCStatus；然后调用AUTOSAR标准接口

*Dem_ReturnGetSizeOfDataByDTCType Dem_DcmGetSizeOfFreezeFrameByDTC(*

*uint32 DTC,*

*Dem_DTCOriginType DTCOrigin,*

*uint8 RecordNumber,*

*uint16\* SizeOfFreezeFrame)*

获得冻结帧（快照）的数据大小，因为数据大小和数据内容是提前配置好的。最后调用AUTOSAR标准接口

*Dem_ReturnGetFreezeFrameDataByDTCType Dem_DcmGetFreezeFrameDataByDTC(*

*uint32 DTC,*

*Dem_DTCOriginType DTCOrigin,*

*uint8 RecordNumber,*

*uint8\* DestBuffer,*

*uint16\* BufSize)*

获取冻结帧。但是DCM和DEM是两个不同的任务，所以以上几个函数一般是异步执行，DCM只负责把请求命令和写入目标给到DEM，在DEM任务中轮询DCM的任务请求，并实现数据的填充，后通知DCM任务完成。DCM再通过肯定相应回复数据。

 

**DEM和****NVM**：NVM之于DEM主要在于诊断数据存储（快照信息级诊断状态字），NVM也就是NVRAM manager，司职于非易失性数据的存储和维护。而DEM中又有大量数据需要存储在非易失性存储模块（如Dflash、EEPROM）中，但两者的交互关系都发生在上电初始化（startup）和下电（shutdown）过程中，当然，如果没有NvM_WriteAll的过程，也可以在运行过程中写入。DEM需要存储在非易失性存储模块的数据包括19服务中会用的相关数据（如第一次出现的故障，第一个confirm的故障、最新的故障等等）、primary fault memory中的数据、mirror memory中的故障、OBD相关的数据、冻结帧、预存的数据、Debounce的状态（甚至是debounce counter）等，这些数据会在NVM_Readall的时候读取到DEM的RAM中，下电的时候会在NVM的writeall时从DEM的target RAM写入非易失性存储区（non-volatile memory）。

 

DEM内部分很多子模块，每个子模块分工不同，任务明确。INIT负责对DEM内部变量根据配置内容进行初始化。debounce负责对事件发生故障的有效性和事件表现正常的有效性的做滤波判断，如KL30电电压超过16伏100ms，该事件才会报KL30过压故障；primary fault memory负责存储当前发生的故障的DTC及相关数据，如冻结帧、扩展数据等。operational cycle management负责管理各种操作循环的判断，如对于OBD-related ECU中OBD drving cycle开启时才能故障状态才能进入confirm的状态，这也和项目配置有关；其它子模块及以上模块详细内容，敬请期待下一次的分享——DEM子模块详细分析。