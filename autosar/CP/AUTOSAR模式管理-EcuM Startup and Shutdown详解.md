# AUTOSAR模式管理-EcuM Startup and Shutdown详解

**前言**

**AUTOSAR EcuM模块的分享分为EcuM模块概念详解和EcuM模块配置及代码分析，具体的项目实战请关注本号的后续文章，本篇为\**EcuM模\**\**块--Sleep和Shutdown阶段详解\**。**

**[AUTOSAR 模式管理-EcuM模块功能概述](http://mp.weixin.qq.com/s?__biz=Mzg2NTYxOTcxMw==&mid=2247484853&idx=1&sn=63d030d452ee7dab01a3d29f738ed6ac&chksm=ce561fdbf92196cdf2c82d67dbdc6c7cd053425de0d717261ae8d83393ffe78d69767715f61b&scene=21#wechat_redirect)**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxxuM89Vr4IRSd5LHUTh4ib4MWqLFN3nMHFy9FCDVqSZXvmct7oEibF8LIYJ4VLrdKkyTJ7VkY3ASAQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**正文**

# **6.** **STARTUP****阶段详解**

图5展示了 ECU 的启动行为。当通过 EcuM_Init 调用时，EcuM模块控制 ECU 启动程序。通过调用 StartOS，EcuM模块暂时放弃对ECU的控制。要重新获得控制权，集成器（Integrator）必须执行一个自动启动的操作系统任务，并将调用 EcuM_StartupTwo 作为其第一个操作。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxxuM89Vr4IRSd5LHUTh4ib4Vp5tNVKoN95WvOvPypAXyoEUV7sdvaZxeZm79rBe6bALyXlibGywgTw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



图5：STARTUP序列

 

## **6.1****EcuM_Init之前的动作**

EcuM模块假定在调用EcuM_Init之前已经进行了 MCU 的最小初始化，以便设置堆栈并可以执行代码，并且已经执行了变量的 C 初始化。

 

EcuM_Init一般就是进入Main函数之后的第一行代码，也就是说EcuM_Init之前主要时MCU的启动代码阶段，一般完成堆栈的初始化，RAM的初始化，然后跳转到Main函数开始执行EcuM_Init。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxxuM89Vr4IRSd5LHUTh4ib4zd2nPQCPCW8WoxQlYHicQEwiaW1icgZs4vnlArjcTia1Tw1e87QMwqichgQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxxuM89Vr4IRSd5LHUTh4ib4Du6JC9PPRJibWicslpexiaUMJs1J6iblXPfcQjOpalNFQjIYHmyqES8hSw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## **6.2** **StartPreOS Sequence执行的动作**

 

| StartPreOS Sequence                      |                                                              |              |
| ---------------------------------------- | ------------------------------------------------------------ | ------------ |
| 初始化活动                               | 说明                                                         | 是否是可选项 |
| CalloutEcuM_AL_SetProgrammableInterrupts | 在具有可编程中断优先级的ecu上，必须在操作系统启动之前设置这些优先级。 | 是           |
| CalloutEcuM_AL_DriverInitZero            | 初始化不使用配置参数的BSW模块。callout不仅可以包含驱动程序初始化，还可以包含任何类型的os之前的低级初始化代码 | 是           |
| CalloutEcuM_DeterminePbConfiguration     | 调用将返回一个指向完全初始化的EcuM_ConfigType结构体的指针，该结构体包含ECU管理器模块和所有其他BSW模块的构建后配置数据 | 否           |
| 检查配置数据的一致性                     | 数据一致性检查失败后调用 EcuM_ErrorHook                      | 否           |
| CalloutEcuM_AL_DriverInitOne             | 不仅可以包含驱动程序初始化，还可以包含任何类型的os之前的低级初始化代码。 | 是           |
| Get reset reason                         | 调用MCU模块提供的AUTOSAR标准接口Mcu_GetResetReason获取复位原因，同时调用EcuM_SetWakeupEvent设置唤醒事件 | 否           |
| 选择默认的shutdown目标                   |                                                              | 否           |
| CalloutEcuM_LoopDetection                | 如果启用了循环检测，则每次启动时都会调用该调用。如果检测到复位回路，则返回true。否则返回false。 | 是           |
| Start OS                                 | 启动AUTOSAR OS                                               | 否           |

 

StartPreOS序列的目的是准备ECU初始化操作系统，应该尽可能短。如果可能的话，驱动程序应该在UP阶段初始化，并且调用也应该保持简短。在这个序列中不应该使用中断。如果必须使用中断，在StartPreOS序列中只允许I类中断。

 

驱动程序和硬件抽象模块的初始化不是由ECU管理器严格定义的。两个callout EcuM_AL_DriverInitZero和EcuM_AL_DriverInitOne被提供来定义init块0和1。这些块包含了与StartPreOS序列相关的初始化活动。

 

MCU_Init不提供完整的MCU初始化。此外，与硬件相关的步骤必须在系统设计时执行并定义。这些步骤应该在EcuM_AL_DriverInitZero或EcuM_AL_DriverInitOne callout中采取。详细信息可以在MCU驱动程序规范中找到。

 

ECU管理器模块应该调用 EcuM_GetValidatedWakeupEvents与配置的默认关机目标EcuMDefaultShutdownTarget.

 

StartPreOS序列将初始化启动操作系统所需的所有基本软件模块。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxxuM89Vr4IRSd5LHUTh4ib4X9gP0DiaEdq9ROZVd5ib4yQKR99D9Ku1KmzeX7klcgKe0DNTIp3TngwQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



图6-StartPreOS 序列

 

## **6.3** **StartPostOS Sequence执行的动作**

| StartPostOS Sequence                       |                                   |              |
| ------------------------------------------ | --------------------------------- | ------------ |
| 初始化活动                                 | 说明                              | 是否是可选项 |
| 初始化 BSW调度器（Init BSW Scheduler）     | 为BSW模块使用的临界区初始化信号量 | 否           |
| 初始化BSW管理模块（Init BSW Mode Manager） |                                   | 否           |

 

当通过EcuM_StartupTwo函数激活时，ECU Manager模块将执行startposto序列中的动作。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxxuM89Vr4IRSd5LHUTh4ib4Y2nIUgGMEb9h1IufDhmiaQKicIDVjG1y05o6ZCBpI8ARprWOCQdWqztA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



图7-StartPostOS序列

 

## **6.4 驱动初始化**

一个驱动程序在初始化过程中的位置很大程度上取决于它的实现和目标硬件设计。



驱动可以在STARTUP阶段的Init Block 0或Init Block 1中由ECU Manager模块初始化，也可以在WakeupRestart序列的EcuM_AL_DriverRestart callout中重新初始化。驱动程序也可以在UP阶段由BswM初始化或重新初始化。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxxuM89Vr4IRSd5LHUTh4ib4XCYtPcj0iaaIQ5iaKVcb4aicnRwzicLWg1VS1JiaGSFCC4iaTyXia22QIYfnw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## **6.5** **DET初始化**

默认错误跟踪模块是一个BSW模块，它包含用于调试的软件。在操作之前，DET必须被初始化(通过调用Det_Init)和启动(通过调用Det_Start)。

如果至少有一个模块被配置为跟踪开发错误，ECU Manager模块应该在StartPreOS序列中，在所有其他驱动程序之前初始化DET。

 

# **7.SHUTDOWN阶段详解**

BswMMode状态机切换到SHUTDOWN状态后调用EcuMSelectShutdownTarget_OFF_MCU和EcuMGoDown使得EcuM接管程序走SHUTDOWN Sequence。

 

当关机阶段发生唤醒事件时，ECU Manager模块应立即完成关机并重启。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxxuM89Vr4IRSd5LHUTh4ib43GFEfl6GV4Qz1I6mRAS38ndvvsOpFgick4ThWnKhYmQmAGPsyFy4ENg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图8-Shutdown阶段

 

## **7.1OffPreOS Sequence执行的动作**

| OffPreOS Sequence                                            |                                                            |              |
| ------------------------------------------------------------ | ---------------------------------------------------------- | ------------ |
| Shudown活动                                                  | 说明                                                       | 是否是可选项 |
| 反初始化BswM模块                                             |                                                            | 否           |
| 反初始化BswM调度器模块                                       |                                                            | 否           |
| 检查挂起的唤醒事件                                           | 目的是检测关机期间发生的唤醒事件                           |              |
| 如果唤醒事件挂起，将RESET设置为关机目标(将使用EcuMDefaultResetModeRef的默认重置模式) | 只有检测到挂起的唤醒事件时，才会执行此操作，以允许立即启动 |              |
| ShutdownOs                                                   | 该操作系统任务的最后一次操作                               |              |

 

作为它的最后一个活动，ECU管理器模块应该调用shutdownnos函数。操作系统在关机结束时调用关机钩子。

 

关机钩子函数应该调用EcuM_Shutdown来终止关机进程。EcuM_Shutdown不应返回，最后要么关闭ECU或复位MCU。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxxuM89Vr4IRSd5LHUTh4ib4zZpccXsar74icGSgiaKKMHh8KZIiaKMNBsYGPbs3xlsPlWUKqd8aTV7rw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图9-OffPreOS序列

 

## **7.2OffPostOS Sequence执行的动作**

OffPostOS序列执行最后的步骤，在操作系统关闭后到达关机目标。EcuM_Shutdown启动该序列。

 

关机目标可以是ECUM_STATE_RESET或ECUM_STATE_OFF，其中特定的复位模式由复位模式决定。

| OffPostOS Sequence                                |                                      |              |
| ------------------------------------------------- | ------------------------------------ | ------------ |
| Shudown活动                                       | 说明                                 | 是否是可选项 |
| Callout EcuM_OnGoOffTwo                           |                                      | 否           |
| Callout EcuM_AL_Reset or CalloutEcuM_AL_SwitchOff | 取决于Shutdown的目标（Off或者Reset） | 否           |

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxxuM89Vr4IRSd5LHUTh4ib4xibkXJUb3W9Oic5HmYdz08iaMjeBeFnOazOUiagTEmcvfQ2GH7VJn1ASzA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图11-OffPostOS序列



当关机目标为RESET时，ECU Manager模块将调用EcuM_AL_Reset callout。当关机目标为OFF时，ECU Manager模块应调用EcuM_AL_SwitchOff callout。

