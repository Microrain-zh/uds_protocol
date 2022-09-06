# AUTOSAR模式管理-EcuM Sleep and UP详解

**前言**

***\*AUTOSAR EcuM模块的分享分为EcuM模块概念详解和EcuM模块配置及代码分析，具体的项目实战请关注本号的后续文章，本篇为\*\*EcuM模\*\*\*\*块--Sleep和UP阶段详解\*\*。\****

***[AUTOSAR 模式管理-EcuM模块功能概述](http://mp.weixin.qq.com/s?__biz=Mzg2NTYxOTcxMw==&mid=2247484853&idx=1&sn=63d030d452ee7dab01a3d29f738ed6ac&chksm=ce561fdbf92196cdf2c82d67dbdc6c7cd053425de0d717261ae8d83393ffe78d69767715f61b&scene=21#wechat_redirect)\**
\***

***[AUTOSAR模式管理-EcuM Startup and Shutdown详解](http://mp.weixin.qq.com/s?__biz=Mzg2NTYxOTcxMw==&mid=2247484920&idx=1&sn=a220b4f0449f0c823ccc1e3a6c70fd31&chksm=ce561f96f92196800f8b503d4db5dbc47d78d3abf2b5d1ca76c7d3e6351a200c38cd6790eb7c&scene=21#wechat_redirect)\***

***\*
\****

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxxuM89Vr4IRSd5LHUTh4ib4MWqLFN3nMHFy9FCDVqSZXvmct7oEibF8LIYJ4VLrdKkyTJ7VkY3ASAQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**正文**

# **8.****SLEEP阶段详解**

BswMMode状态机切换到SLEEP状态后调用EcuMSelectShutdownTarget_SLEEP_MCU和EcuMGoHalt或者EcuMGoPoll使得EcuM接管程序走SLEEP Sequence。

 

EcuM_GoHalt和EcuM_GoPoll启动两个不同的控制流，它们在实现睡眠的机制上不同。然而，它们在准备睡觉和从睡眠中恢复时的顺序是相同的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyQTaaMwiaQMV4qAaGkIYtic5uleMURE5j5OjUf6cJzhGxvmIyjMibJWIbasq2ttayWImZKuWuNrC8bw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



图12-Sleep阶段

 

另一个模块，可能是BswM，也可以是一个SW-C，必须确保在调用EcuM_GoHalt或EcuM_GoPoll之前选择了适当的ECUM_STATE_SLEEP关机目标。

 

# **8.1** **GoSleep Sequence执行的动作**

在GoSleep序列中，ECU管理器模块为即将到来的睡眠阶段配置硬件，并为下一次唤醒事件设置ECU。

 

为了在接下来的睡眠模式中设置唤醒源，ECU Manager模块应该对每个在目标睡眠模式的EcuMWakeupSourceMask中配置的唤醒源执行EcuM_EnableWakeupSources调用。

 

***Note: Callout函数EcuM_EnableWakeupSources中的实现需要根据具体的芯片来实现。如果芯片厂商的提供的MCAL代码中提供了标准接口（例如RH850_F1KM提供了Mcu_WakeUpFactor_Preparation接口）则直接调用，如果没有提供，则需要去Callout函数中设置唤醒中断。\***

 

相对于SHUTDOWN阶段，ECU Manager模块在进入SLEEP阶段时不会关闭操作系统。睡眠模式，即EcuM睡眠阶段和Mcu模式的结合，应该对操作系统透明。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyQTaaMwiaQMV4qAaGkIYtic5DsDKcg96zX3Jb40KhDzO9lmeXSJFiaRItd0r3Vboq7NLRXxWVqxm2og/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图13-GoSleep序列

 

当在多核ECU上运行时，ECUM应该为每个核预留一个专用的资源(RES_AUTOSAR_ECUM)，在GoSleep期间分配。

 

## **8.2 Halt Sequence执行的动作**

ECU管理器模块应在休眠模式下执行Halt序列，使微控制器停止运行。在这些睡眠模式下，ECU管理器模块不执行任何代码。

 

ECU管理器模块应该在MCU进入深度休眠前调用EcuM_GenerateRamHash callout，处理器从halt返回后将会调用EcuM_CheckRamHash callout。

 

如果应用多核和存在“slave”EcuM(s)，这个检查应该只在“master”EcuM上执行。“主”EcuM从其可触及的所有数据中生成散列。“slave”EcuMs的私有数据不在检查范围内。

 

当ECU长时间处于休眠状态时，Ram内存可能会损坏。因此，应该检查RAM内存的完整性，以防止不可预见的行为。系统设计者可以选择一个适当的校验和算法来执行校验。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyQTaaMwiaQMV4qAaGkIYtic5LubfKeNu8TexCpFKDzryXpsP137dKJeEtqtiaG2Ivmw8Z7eOlTmTvdg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图14-Halt Sequence

 

ECU管理器模块应该调用EcuM_GenerateRamHash，系统设计者可以放置一个RAM完整性检查。

 

## **8.3 Poll Sequence执行的动作**

ECU Manager模块应在睡眠模式下执行Poll序列，以降低微控制器的功耗，但仍执行代码。在Poll序列中，EcuM将在阻塞循环中调用调用EcuM_SleepActivity()和EcuM_CheckWakeup()，直到报告了一个挂起的唤醒事件。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyQTaaMwiaQMV4qAaGkIYtic5xU9TGruvsG7iba7gxnocgTgU4COQ3ZeW14qSjBph81WKEfpEl8kfeMw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图15-Poll Sequence

 

## **8.4 Leave Halt or Poll后执行的动作**

如果ECU处于Halt或Poll状态时发生了唤醒事件(例如切换唤醒线电平状态、CAN总线上的通信等)，那么ECU状态管理器模块的ECU管理器规范将通过执行WakeupRestart序列重新获得控制并退出SLEEP阶段。

 

可以调用ISR来处理唤醒事件，但这取决于硬件和驱动程序实现。

 

当ECU处于Halt或Poll状态时，如果出现不规则事件(硬件复位或电源循环)，ECU Manager模块将在STARTUP阶段重新启动ECU。

 

## **8.5 WakeupRestart Sequence执行的动作**

| WakeupRestart Sequence             |                                                              |              |
| ---------------------------------- | ------------------------------------------------------------ | ------------ |
| Wakeup活动                         | 说明                                                         | 是否是可选项 |
| 控制MCU从Sleep模式恢复到Normal模式 | 所选MCU模式在配置参数EcuMNormalMcuModeRef中进行配置          |              |
| 获取挂起的唤醒源事件               | 禁用当前等待的唤醒源，但让其他唤醒源武装起来，以便以后可以唤醒。 |              |
| Callout EcuM_DisableWakeupSources  | 目的是检测关机期间发生的唤醒事件                             |              |
| Callout EcuM_AL_DriverRestart      | 初始化需要重启的驱动                                         |              |
| Unlock Scheduler                   | 从此时起，所有其他任务都可以再次运行。                       |              |

 

ECU管理器模块调用EcuM_AL_DriverRestart callout，用于重新初始化驱动程序。其中，具有唤醒源的驱动程序通常需要重新初始化。

 

在重新初始化期间，驱动程序必须检查它所分配的唤醒源中是否有一个是上次唤醒的原因。如果这个测试为true，驱动程序必须调用它的' wakeup detected '回调，它必须调用EcuM_SetWakeupEvent。

 

驱动程序实现应该只调用wakeup回调一次。此后，在通过显式函数调用重新武装之前，它不应该再次调用wakeup回调。因此，驱动程序必须重新武装以再次触发回调。

 

当WakeupRestart序列结束时，如果ECU Manager模块有一个唤醒源候选列表，ECU Manager模块应该在EcuM_MainFunction中验证这些唤醒源候选列表。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyQTaaMwiaQMV4qAaGkIYtic5dX3AhpiaCIL9yhvGQoUIzakfpvblMdaBQz3t79IOLKBgBTbDETeRCQw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图16-WakeupRestart Sequence

# **9.UP阶段详解**

在UP阶段，EcuM_MainFunction定期执行，它有三个主要功能:

\- 要检查唤醒源是否已被唤醒，并启动唤醒验证，如果需要的话

\- 更新Alarm计时器

\- 仲裁RUN和POST_RUN请求和释放

 

## **9.1 Alarm时钟处理**

如果EcuMAlarmClockPresent参数配置使用了Alarm时钟服务，则 EcuM_MainFunction中需要更新Alarm时钟。

 

## **9.2 唤醒源状态处理**

唤醒源不仅在唤醒期间被处理，而且持续地与所有其他EcuM活动并行处理。这个功能运行在EcuM_MainFunction中，完全与ECU通过模式请求进行管理的其余部分解耦。

 

唤醒源状态表：

| 状态      | 描述                                 |
| --------- | ------------------------------------ |
| ENABLED   | 唤醒源被使能                         |
| NONE      | 没有检测到唤醒事件或者唤醒事件被清除 |
| PENDING   | 检测到唤醒事件，但尚未验证           |
| VALIDATED | 检测到唤醒事件并成功验证             |
| EXPIRED   | 检测到唤醒事件，但验证失败           |

 

下图说明了唤醒源状态与引起状态更改的条件函数之间的关系。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyQTaaMwiaQMV4qAaGkIYtic5wgfFibFiatBcXsib7C0sx8giavC8KvNpoSQLCgO4DS0zK8oouHPaUsfU7w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图17-唤醒源状态

 

当ECU Manager操作导致唤醒源状态改变时，ECU Manager模块应向BswM发出模式请求，将唤醒源的模式改变为新的唤醒源状态。

 

当ECU Manager模块处于UP阶段时，唤醒事件通常不会触发状态变化。然而，它们触发了Halt和Poll阶段的结束。ECU管理器模块然后自动执行WakeupRestart序列，然后返回到UP阶段。

 

由集成商在BswM中配置规则，以便ECU正确地对唤醒事件作出反应，因为该反应完全依赖于当前ECU(而不是ECU Management)状态。

 

如果唤醒源有效，BswM将返回ECU的RUN状态。如果所有唤醒事件都返回NONE或EXPIRED, BswM将再次为SLEEP或OFF准备BSW，并根据上一个关机目标调用EcuM_GoPoll或EcuM_GoHalt或EcuM_GoDown。

 

总结:每个未决事件都被独立地验证(如果配置了)，EcuM将结果作为模式请求发布给BswM，而BswM反过来可以触发EcuM中的状态更改。

 

## **9.3 唤醒状态的内部表现**

EcuM管理器模块提供以下接口来确定这些唤醒源的状态：

· EcuM_GetPendingWakeupEvents

· EcuM_GetValidatedWakeupEvents

· EcuM_GetExpiredWakeupEvents

并通过以下接口操作唤醒源的状态:

· EcuM_ClearWakeupEvent

· EcuM_SetWakeupEvent

· EcuM_ValidateWakeupEvent

· EcuM_CheckWakeup

· EcuM_DisableWakeupSources

· EcuM_EnableWakeupSources

· EcuM_StartWakeupSources

· EcuM_StopWakeupSources

 

ECU Manager模块最多可管理32个唤醒源。唤醒源的状态通常在上面命名的EcuM接口中通过EcuM_WakeupSourceType位掩码表示，其中单个唤醒源对应于一个固定的位位置。有5个预定义的位位置，其余的可以通过配置分配。

 

ECU Manager模块一方面管理各个唤醒源的模式。另一方面，ECU Manager模块假设有“内部变量”(即EcuM_WakeupSourceType实例)来跟踪哪些唤醒源处于特定的状态(特别是NONE(即清除)、PENDING、VALIDATED和EXPIRED)。ECU Manager模块在各自的接口定义中使用这些“内部变量”来定义接口的语义。

 

因此，这些“内部变量”是否确实被执行是次要的。它们只是用来解释接口的语义。

## **9.4 WakeupValidation Sequence执行的动作**

由于唤醒事件可能在无意中产生(例如can线上的EVM峰值)，因此有必要在ECU完全恢复运行之前验证唤醒。

 

所有唤醒源的验证机制都是相同的。当一个唤醒事件发生时，ECU从它的SLEEP状态被唤醒，并在MCU驱动的MCU_SetMode服务中继续执行。当WakeupRestart序列完成时，ECU管理器模块将有一个等待验证的唤醒事件列表(参见SWS_EcuM_02539)。ECU管理模块然后释放BSW调度器和所有BSW MainFunctions;在这种情况下，最明显的是，EcuM MainFunction可以恢复处理。

 

实现提示:由于SchM将在StartPostOS和WakeupRestart序列的末尾运行，所以有可能EcuM_MainFunction会对一个堆栈还没有初始化的源启动验证。集成开发者应该配置适当的模式，表明堆栈不可用，并相应地禁用EcuM_MainFunction。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyQTaaMwiaQMV4qAaGkIYtic5AVwtQzpUyZKIjJibItHb6HicposqTws31wZp1JicmqQvVcOjo1uruw0Mw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图18-WakeupValidation Sequence

 

ECU管理器模块只能在配置要求的唤醒源上调用唤醒验证。如果没有配置验证协议，那么调用EcuM_SetWakeupEvent也意味着调用EcuM_ValidateWakeupEvent。

 

ECU管理器模块应该为每个有待验证的唤醒事件启动验证超时。超时时间应该是每个唤醒事件具体设置的。

 

*实现提示:实现只提供一个计时器就足够了，当报告新的唤醒事件时，该计时器将被延长到最大超时*。

 

当一个挂起的唤醒事件的验证超时时，EcuM_MainFunction(OR-operation)在内部过期唤醒事件变量中设置超时位。

 

当一个挂起的唤醒事件的验证超时过期时，EcuM_MainFunction应该调用BswM_EcuM_CurrentWakeup，使用EcuM_WakeupSourceType位掩码参数，该位对应于唤醒事件设置，状态值参数设置为ECUM_WKSTATUS_EXPIRED

 

在验证唤醒源或计时器过期时，BswM将被配置为通过来自EcuM的模式切换请求来监视唤醒验证。如果最后一次验证超时过期没有进行验证，则BswM将认为唤醒验证失败。如果至少有一个挂起的事件被验证，那么整个验证应该已经通过。

 

挂起的事件通过调用EcuM_ValidateWakeupEvent来验证。这个调用必须放在驱动程序中，或者中断处理函数中。放置它的最佳位置取决于硬件和软件设计。

 

### **9.4.1 通信通道唤醒Wakeup of Communication Channels**

如果在通信通道上发生了唤醒，相应的总线收发驱动必须通过调用EcuM_SetWakeupEvent函数通知ECU管理器模块。

 

### **9.4.2 唤醒源和ECU管理器的交互**

ECU管理器模块应以相同的方式处理所有唤醒源。程序如下:

当发生唤醒事件时，对应的驱动器应通知ECU Manager模块唤醒。这种通知最可能的方式是:

-- 退出Halt或Poll序列后。在这个场景中，ECU管理器模块调用EcuM_AL_DriverRestart来重新初始化相关的驱动程序，这些驱动程序反过来有机会扫描他们的硬件，例如等待唤醒中断。

--如果唤醒源实际上处于睡眠模式，驱动程序必须自动扫描唤醒事件;通过轮询或等待中断。

 

如果一个唤醒事件需要验证，那么ECU管理器模块应该调用验证协议。

 

如果唤醒事件不需要验证，ECU管理器模块应该发出模式切换请求，将事件的模式设置为ECUM_WKSTATUS_VALIDATED。

 

如果唤醒事件被验证了(要么是立即验证，要么是通过唤醒验证协议验证)，ECU Manager模块应该通过EcuM_GetValidatedWakeupEvents来显示它是当前ECU唤醒的一个源。

 

### **9.4.3 唤醒验证超时**

ECU管理模块应提供单个唤醒验证超时计时器，或每个唤醒源提供一个计时器。

 

需要满足以下的需求：

ECU Manager模块在EcuM_SetWakeupEvent时启动唤醒验证超时定时器。

EcuM_ValidateWakeupEvent将停止唤醒验证超时定时器。

 

如果后续同一个唤醒源调用了EcuM_SetWakeupEvent, ECU Manager模块将不会重启唤醒验证超时。

 

如果只使用一个定时器，建议采用以下方法:

EcuM_SetWakeupEvent被调用时，如果一个唤醒源在同一个唤醒周期中还没有被触发，那么ECU Manager模块应该延长该唤醒源的验证超时时间。

 

### **9.4.4 唤醒源对驱动程序的需求**

当检测到唤醒事件时，驱动程序必须调用EcuM_SetWakeupEvent，并提供一个EcuM_WakeupSourceType参数来标识唤醒的来源。

 

从Halt或者Poll或者Off状态退出时，EcuM模块将检测驱动程序初始化之前发生的唤醒。驱动程序必须提供一个API为Sleep状态提供唤醒功能，提供启用或禁用唤醒源的功能，提供设置相关外设进入休眠状态的功能。这些要求仅适用于硬件提供这些功能的情况。

 

驱动程序应该在其初始化函数中启用回调调用。

 

## **9.4 唤醒验证的需求Requirements for Wakeup Validation**

如果唤醒源需要验证，这可以由基本软件的任何一个(但只有一个)适当的模块来完成。这可能是一个驱动程序、一个接口、一个处理程序或一个管理器。

 

验证通过调用EcuM_ValidateWakeupEvent函数来完成。

 

如果EcuM不能确定Mcu驱动返回的复位原因，那么EcuM会为默认的唤醒源ECUM_WKSOURCE_RESET设置一个唤醒事件。

 

## **9.5 唤醒源和服务原因**

ECU Manager模块API只提供了一种类型，可以描述ECU启动或唤醒的所有原因。

ECU管理器模块不能对以下唤醒源进行验证:

· ECUM_WKSOURCE_POWER

· ECUM_WKSOURCE_RESET

· ECUM_WKSOURCE_INTERNAL_RESET

· ECUM_WKSOURCE_INTERNAL_WDG

· ECUM_WKSOURCE_EXTERNAL_WDG

 

## **9.6 集成电源控制唤醒源 Wakeup Sources with Integrated Power Control**

通过系统芯片（SBC）控制单片机的电源来实现睡眠。典型的例子是带有集成电源的CAN收发器，它可以在应用请求时关闭电源，在CAN活动时打开电源。其结果是，对于这种类型的硬件上的ECU管理器模块来说，SLEEP看起来像是OFF。

 

实际的影响是，CAN上的被动唤醒看起来就像ECU复位时的电源。因此，在一个唤醒事件之后，ECU将继续执行STARTUP序列。尽管如此，唤醒验证仍然是必需的，系统设计师必须考虑以下主题:

-- CAN收发器在一个驱动初始化块中被初始化(默认情况下在BswM控制下)。这是配置或生成的代码，即在系统设计人员控制下的代码。

-- CAN收发器驱动程序API提供了一些功能，用来发现是否是CAN收发器在被动唤醒时启动了ECU。系统设计师的职责是检查CAN收发器的唤醒原因，并通过使用EcuM_SetWakeupEvent和EcuM_ClearWakeupEvent函数将该信息传递给ECU管理器模块。

 

这些原理适用于所有具有综合功率控制的唤醒源。此处仅以CAN收发器为例。



*Note:未完待续，下一篇--EcuM 多核处理介绍*