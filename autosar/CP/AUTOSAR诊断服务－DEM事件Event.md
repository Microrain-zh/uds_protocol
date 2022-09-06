# AUTOSAR诊断服务－DEM事件Event

**前言**

**AUTOSAR DEM模块的分享分为DEM模块概念详解和Dcm模块配置及代码分析，具体的项目实战请关注本号的后续文章，本篇为\**DEM模块Event详细介绍篇\**。**

**[AUTOSAR 诊断服务-DEM功能概述](http://mp.weixin.qq.com/s?__biz=Mzg2NTYxOTcxMw==&mid=2247484841&idx=1&sn=eb2474651f14db5896c191a20a7780a5&chksm=ce561fc7f92196d1aa7b0920232141ffca462a07dd8a87a032362adc05e4fa9d3f086ce3e1b4&scene=21#wechat_redirect)
**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwqZIVg0vfKkrQNnJDiab1hbibAMIeLnkQltIdZIIqyEO3xQD3YGztticl2QlNLQ0Ru21ibkLXAx8EWvA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**正文**

# **4.****Dem模块详细介绍**

Dem模块可以处理和存储BSW和SWC检测到的诊断事件（Dem_SetEventStatus）。同时，BSW模块和SWC可以通过AUTOSAR提供的标准接口获得Dem存储的事件信息（Dem_GetEventStatus）。



## **4.1** **Dem模块启动行为**

Dem模块应区分预初始化模式和全初始化模式（操作模式）。Dem_PreInit函数应初始化Dem模块的内部状态，并使用Dem_SetEventStatus和Dem_ResetEventDebounceStatus处理事件并重置SW-C或BSW模块报告的Debounce计数器。



在NVM初始化之前，在ECU的启动阶段，由EcuM调用函数Dem_PreInit。NVM初始化之后，也就是NVM调用NVM_ReadAll完成所有NVM数据拷贝到RAM之后，在ECU启动阶段需要调用Dem_Init完成Dem模块的完全初始化。随后，具有诊断事件监控的Monitors(SWC-c or BSW)完成初始化，可以开始诊断事件监控和设置。



Dem_Shutdown之后如果需要调用Dem_Init重新初始化Dem模块。



Note:

1）Dem_PreInit一般在EcuM的Init One阶段调用

2）Dem_Init一般BSWM模式管理的Action中调用（Startup --> Run or PreShutDonw --> Run）

3）如果调用Dem_Shutdown后ECU又需要恢复Dem功能，但是忘记了重新初始化Dem模块（Dem_Init），就会发生很奇怪的现象，比如0x10,0x19服务就会一直Pending后回0x10的否定响应码，这个坑要记得避免（本质就是在Dem_Shutdown后忘记调用Dem_Init导致Dem不能正常使用）。



## **4.2** **监视器（Monitors）重新初始化**

应用程序中的监视器的主要初始化是通过Rte_Start来完成的。监视器的事件特定部分的初始化可以由Dem触发。



Dem应提供InitMonitorForEvent接口，以触发诊断监视器的初始化。



功能参数InitMonitorReason指示初始化的原因/触发器。



配置容器 DemCallbackInitMForE用来为每个事件（Event）来定义相关的Port和Callback函数。



Note: 这个功能在实际项目中基本没有用到，配置参数也不会配置，这里不详细介绍了，需要用到的时候可以再研究。



## 4.3 **诊断事件Event定义**

“诊断事件”定义了可由Dem模块处理的原子单元。“诊断事件”的状态表示监视器的结果。Dem通过RTE或其他BSW模块从SW-C接收监控器的结果。



Dem模块使用EventId来管理系统的“诊断事件”的状态，并对单个测试结果执行所需的操作，例如存储冻结帧。



Dem模块应通过事件标识和相关事件名称表示每个诊断事件，所有监视器（SWC）和BSW模块都使用事件Id作为符号事件名称。Dem配置工具将用数字替换符号名称，事件标识和相关事件名称应是唯一的。



Dem模块的设计不能够处理多个监视器共享一个EventId的情况。Dem模块使用内部监视器状态来存储所报告事件的状态。向Dcm报告UDS状态。Dem模块支持几个特定于事件的配置参数，如下图所示。有关详细的描述，请参阅后面的配置规范。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwqZIVg0vfKkrQNnJDiab1hbdxGjicGpsSH5uVCO1VWhRyg2RqYqibLcCmegDWpcxziccc4phFP7ibtzHw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图三：: Event parameter configuration



对于事件的简单理解：

1）比如BSW底层的一个IoHwAb_Bat模块就是一个Monitor（监视器），它会周期性（10ms调度周期）的去检测电源电压Bat_Voltage是否过高（大于16V）这个事件（Event），如果在一定时间内Bat_Voltage一直过高，则监视器（Monitor）也就是IoHwAb_Bat这个模块就会调用Rte封装的C-S接口Rte_Call_xxx_SetEventStatus（也就是分装了Dem_SetEventStatus这个接口）向Dem模块报告这个Event实践。



2）而Dem模块对于每个Event定义或者描述不光只包括一个EventId，还需要优先级，属于BSW还是SWC等相关信息，这就需要为每个Event提供一个配置容器（里面包含一个Event需要的所有的配置信息），也就是DemEventParameter这个配置容器。每一个Event都对应一个具体的DemEventParameter配置实例（Instance）。



### **4.3.1** **事件优先级**

事件优先级被定义为基于重要性级别对事件进行排序。它用于确定当存储的事件数量超过最大内存条目数（事件内存已满）时，可以从事件内存中删除哪些故障条目。



每个支持的事件应具有优先级（参见DemDTC属性中的参数DemDTCPrifisty）。



优先级值1为最高优先级。优先级值越大，重要性就越低。



问题：为什么我们需要定义事件的优先级？

答：当我们的诊断事件树木大于我们设置的诊断事件状态码的存储数据的时候我们需要考虑定义事件的优先级，这样就能保证在诊断事件状态码存储区域已经存满的情况下，也能保证优先级高的诊断事件的状态码还能存储到NVM当中去（删除一些优先级低的）。



### **4.3.2** **事件发生**

几个相关概念理解：

1）Event memory

An event memory (e.g. Primary memory) consists of several event memory entries.

事件内存（例如主内存）由多个事件内存条目组成。



2）Event memory entry

The event memory entry is a single storage container for an event and its event related data. The event memory entry is dynamically assigned to specific events.

事件内存条目是事件及其事件相关数据的单个存储容器。事件内存条目被动态地分配给特定的事件。



3）Event memory overflow indication

The event memory overflow indication indicates, if this specific event memory is full and the next event occurs to be stored in this event memory.

事件内存溢出指示表明，如果此特定事件内存已满，并且发生的下一个事件将存储在此事件内存中。



Dem模块应为每个事件存储器条目提供一个事件计数器，如果在各自的事件内存中输入了相关事件，则Dem模块应使用值1初始化发生计数器。



如果配置参数DemOccurrenceCounterProcessing为DEM_PROCESS_OCCCTR_TF，则如果相关事件已经存储在事件内存中，则由UDS状态位0（测试失败）由0-->1将触发发生计数器加1。



如果配置参数DemOccurrenceCounterProcessing为DEM_PROCESS_OCCCTR_CDTC，如果相关事件已经存储在事件存储器中，并且UDS状态位3等于1，则由UDS状态位0（测试失败）由0-->1将触发发生计数器加1。



Dem模块应不再增加特定事件的发生计数器，如果它已达到其最大值(255）。



### **4.3.3** **事件类型**

有两种不同类型的事件：

1）与BSW相关的事件(通过C-API-Dem_SetEventStatus报告)

2）SW-C相关事件（通过RTE操作报告-设置事件状态）



每个事件都需要通过DemEventParameter配置容器下的DemEventKind参数配置事件类型。



这是必要的，因为BSW事件可能在完全Dem初始化之前报告，需要缓冲。



### **4.3.4** **事件存储**

相关概念理解：

DemMemoryDestinationRef: 内存目标为一个或两个内存目标分配dtc。如果将多个内存目标分配给一个特定的DTC，则该DTC可以出现在相应的事件内存中。在这种情况下，其中一个引用必须是删除镜像内存。该参数可以配置为:

DemMirrorMemory,DemPrimaryMemory,DemUserDefinedMemory



DemPrimaryMemory: 此容器包含Dem模块的主事件内存特定参数。

DemPrimaryMemory: 此容器包含Dem模块的镜像事件内存特定参数。

DemUserDefinedMemory：此容器包含Dem模块的用户定义的事件内存特定参数。





配置参数DemMemoryDestinationRef（参考DemDTC属性）定义了事件及其相关数据的专用存储位置。



不同内存类型的定义和使用是特定于OEM的。



对于Dcm-Dem接口，使用参数DTCOrigin来区分不同的内存区域。其目的是允许对不同的内存区域（主内存、用户定义内存、永久内存和镜像内存）进行特定的操作。



DemMemoryDestinationRef配置参数的限制：

1）如果一个事件的配置参数DemMemoryDestinationRef配置为DemMirrorMemory，则这个事件的DemMemoryDestinationRef配置参数还需要在配置一个DemPrimaryMemory或者 DemUserDefinedMemory。



2）一个DTC通过DemMemoryDestinationRef参数引用 event memories，而这些event memories是在不同的DemEventMemorySet中的，这样的场景Dem模块是不支持的。Dem模块只支持一个DTC通过DemMemoryDestinationRef参数引用同一个DemEventMemorySet中的不同的event memories。



### **4.3.5** **事件监控定义**

诊断监视器是确定组件的正确功能的常规实体。此监控功能识别特定故障类型（如对地短路、开路负载等）对于一个监视路径。监视路径表示正在被监视的物理系统或电路（例如，传感器输入）。每个监控路径都只与一个诊断事件相关联。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwqZIVg0vfKkrQNnJDiab1hbssbVvKv2KlVAiclWAFgsPYbwUuJe44Kd89n9MqS3zEnXddlvyg6cPnw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**图四：: Example for a monitor embedded within a SW-C**



如果监视器（SWC）自己进行Debounces，则只有在限定的结果（已通过或失败）可用后，才会调用报告API。如果结果发生了变化，则需要进行报告。然而，通常监视器总是调用Dem在计算上效率更高，因此应该是首选。



如果监视器使用Dem内部debouncing机制，则在执行具有功能检查的代码时调用报告API。



实例理解：

事件定义：电源电压Bat_Voltage大于16V事件大于3秒后记录一个DTC

实现方式1：检测器（Monitor）检测到电源电压Bat_Voltage大于16V后直接调用Dem_SetEventStatus(EventID_Bat, TRUE)接口向Dem报告过压事件，Dem模块来进行3秒的计时（Debounce过程）。



实现方式2：Dem模块不做Debounce过程，收到Monitor的事件报告后就记录DTC，检测器（Monitor）检测到电源电压Bat_Voltage大于16V后进行计时（Debounce），达到3秒后调用Dem_SetEventStatus(EventID_Bat, TRUE)接口向Dem报告过压事件。



### **4.3.6** **事件依赖关系**

相关概念理解：

DemComponent: 此容器将配置受监视的组件和系统依赖关系。



分配的DemComponent的优先级以及DemComponent之间的依赖关系用于过滤到故障内存的错误报告存储。



### **4.3.7** **组件可用性**

Dem应该支持DemComponent的可用性。不可用的组件应被视为不包括在系统中。



Note: 这个功能在实际项目中基本没有用到，配置参数也不会配置，这里不详细介绍了，需要用到的时候可以再研究。



## **4.4** **小结**

BSW或SWC会监控所有的事件Event（过压，欠压，过流等），每个事件发生后SWC或者Dem进行Debounce（计时或者计数），达到确定阈值后Dem模块就会记录一个DTC。而通过DemEventParameter配置容器描述/定义每一个Event。