# AUTOSAR模式管理-EcuM多核处理及其他概念

**前言**

***\**\*AUTOSAR EcuM模块的分享分为EcuM模块概念详解和EcuM模块配置及代码分析，具体的项目实战请关注本号的后续文章，本篇为\*\*EcuM模\*\*\*\*块--多核及其他概念详解\*\*。\*\**\***

**[AUTOSAR模式管理-EcuM模块功能概述](http://mp.weixin.qq.com/s?__biz=Mzg2NTYxOTcxMw==&mid=2247484853&idx=1&sn=63d030d452ee7dab01a3d29f738ed6ac&chksm=ce561fdbf92196cdf2c82d67dbdc6c7cd053425de0d717261ae8d83393ffe78d69767715f61b&scene=21#wechat_redirect)**

**[AUTOSAR模式管理-EcuM Startup and Shutdown详解](http://mp.weixin.qq.com/s?__biz=Mzg2NTYxOTcxMw==&mid=2247484920&idx=1&sn=a220b4f0449f0c823ccc1e3a6c70fd31&chksm=ce561f96f92196800f8b503d4db5dbc47d78d3abf2b5d1ca76c7d3e6351a200c38cd6790eb7c&scene=21#wechat_redirect)
**

[**AUTOSAR模式管理-EcuM Sleep and UP详解**](http://mp.weixin.qq.com/s?__biz=Mzg2NTYxOTcxMw==&mid=2247484936&idx=1&sn=e0fc51b2e59772458d5d98fd32958bf8&chksm=ce561c66f92195706b131e06aa8fa2378ba4f221c9ec62e51bb3d568e83324406264dcaafee2&scene=21#wechat_redirect)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUm2LLaiawELRJHPNn1ENl83gbHXDEawHLyThwKjNHncbFVFYO1Pz7lmw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**正文**

# **10.关机目标Shutdown Target**

“关机目标”是一个描述性术语，用于描述没有执行代码的所有ECU状态。它们被称为关闭目标，因为它们是状态机在离开UP阶段时将驱动到的目标状态。以下状态为关机目标:

· Off

· Sleep

· Reset

确定关闭目标的时间并不一定是关闭的开始时间。由于BswM现在控制了大部分ECU资源，它将决定何时应该设置关机目标，并将其直接或间接地进行设置。因此，BswM必须确保，例如，在调用EcuM_GoHalt或EcuM_GoPoll之前，必须将关机目标从其默认值更改为ECUM_STATE_SLEEP。

 

在ECU Manager模块的早期版本中，睡眠目标被特殊处理，因为在ECU中实现的睡眠模式取决于ECU的功能。这些睡眠模式取决于硬件，通常在时钟设置或硬件提供的其他低功耗特性上有所不同。这些不同的特性可以通过MCU驱动程序作为所谓的MCU模式进行访问。

 

还有不同的复位方式，由不同的模块控制或触发:

· Mcu_PerformReset

· WdgM_PerformReset

· Toggle I/O Pin via DIO / SP

 

ECU管理器模块提供了一个工具，通过跟踪时间和前一次复位的原因来管理这些复位模式。这些复位模式，使用与睡眠相同的模式设施。

 

## **10.1 Sleep**

在SLEEP阶段不能错过任何唤醒事件。如果GoSleep序列中发生了唤醒事件，则不能输入Halt或Poll序列。

 

ECU管理器模块可以定义一组可配置的睡眠模式，其中每个模式本身是关机目标。

 

ECU管理器模块应允许将MCU的睡眠模式映射到ECU的睡眠模式，从而允许它们作为关机目标。

 

ShutdownTarget睡眠将使所有核进入睡眠状态。

 

## **10.2 Reset**

ECU管理器模块应定义一组可配置的复位模式，其中每个模式本身是关机目标。该集合最少包括以下复位方式：

· Mcu_PerformReset

· WdgM_PerformReset

· Toggle I/O Pin via DIO / SPI

 

ECU管理器模块应定义一组可配置的复位原因。该集合应最少包括以下复位原因：

 EcuM状态机进入shutdown状态

 WdgM检测到错误

 Dcm模块请求shutdwon

 

ECU管理模块应为BSW模块和SW-C提供以下服务：

·记录关机原因

·获取一组最近关机原因

 

# **11.定时唤醒-Alarm Clock**

ECU管理器模块提供了一个可选的持久时钟服务，即使在睡眠期间也保持“活跃”。因此，它保证了ECU将在未来的某个时间被唤醒(假设硬件没有故障)，并为长期活动提供时钟服务(例如，以小时、天、甚至年计算)。

 

通常，该服务将通过ECU中的计时器来实现，该计时器可以诱导唤醒。然而，在某些情况下，外部设备也可以使用常规中断线来周期性唤醒ECU。无论使用何种机制，服务都会私下使用一个唤醒源。

 

EcuM模块维护一个主报警时钟（master alarm clock），主报警时钟的设置决定了ECU被唤醒的时间。此外，EcuM管理一个内部时钟，EcuM时钟，它是用来和主报警时钟进行比较校准。

 

定时唤醒机制只与SLEEP阶段相关。SWC和BSW模块可以在UP阶段设置和检索报警值(仅在UP阶段)，设置的值在SLEEP阶段被体现。

 

与其他可以使用通用ECU管理器模块实现的定时/唤醒机制相比，Alarm Clock服务在计时器过期之前不会启动WakeupRestart序列。当ECU模块检测到它的计时器引起了一个唤醒事件，它增加它的计时器并立即返回睡眠，除非时钟时间超过了闹钟时间。

 

# **12.EcuM模式处理**

ECU模式处理为sw - c引入了一个公共接口，即固定状态机(EcuMFixed)的ECU状态管理器。带有灵活状态机(EcuMFlex)的ECU状态管理器为sw - c提供了请求和释放模式RUN和POST_RUN(可选)的接口.

EcuMFixed使用特定的接口来决定ECU是否必须保持活动或准备关闭。与EcuMFixed相比，EcuMFlex只仲裁sw - c发出的请求和释放，并将结果传播到BswM。EcuM和BswM之间的合作是必要的，因为只有BswM可以决定何时可以过渡到不同的模式。由于EcuM没有自己的状态机，EcuM依赖于BswM进行的状态转换。因此，EcuM不请求状态。此外，它将所有请求的当前仲裁通知BswM。当RTE执行了属于某个模式的所有Runnables时，BswM会得到通知。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUqoJEWEwxXUIMp6a2hVkX4D7XPhXhYoeSaDNticNlO9dvtHLy61fztnQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图19-ECU模式处理架构图

 

当BswM通过EcuM_SetState()设置EcuM的状态时，EcuM应将对应的模式更新到RTE。

 

当最后一个RUN请求被释放时，ECU State Manager模块应该使用API BswM_EcuM_RequestedState(POST_RUN, ECUM_RUNSTATUS_RELEASED .)向BswM请求状态POST_RUN。

 

如果SW-C在POST_RUN期间需要post运行活动(例如，shutdown准备)，那么它必须在释放run请求之前请求POST_RUN。否则，不能保证这个SW-C将有机会运行它的POST_RUN代码。

 

当ECU状态管理器不处于SWC所请求的状态时，它应该使用BswM_EcuM_RequestedState() API通知BswM所请求的状态。

 

POST_RUN状态为SW-C提供了一个运行后阶段，并允许它们保存重要数据或关闭外设。

 

当最后一个POST_RUN请求被释放时，ECU State Manager模块应该使用API BswM_EcuM_RequestedState(SHUTDOWN，ECUM_RUNSTATUS_RELEASED)向BswM请求状态SHUTDOWN。

 

提示:为了防止ECU mode的模式机实例滞后，以及EcuM和RTE的状态脱离相位，EcuM可以使用确认反馈进行模式切换通知.

 

请注意，EcuM只向RUN和POST_RUN请求模式，SLEEP Mode必须由BswM设置，因为EcuM没有关于何时可以进入该模式的信息.

| 状态机（State） | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| STARTUP         | 初始值。当Rte_Start()被调用时，由Rte设置。                   |
| RUN             | 当所有必要的BSW模块初始化后，BswM将切换到此模式。            |
| POST_RUN        | 当没有可用的RUN请求时，EcuM请求POST_RUN。                    |
| SLEEP           | 当没有RUN和POST_RUN请求时，EcuM请求SLEEP模式,Shutdown Target设置为SLEEP。 |
| SHUTDOWN        | 当没有RUN和POST_RUN请求时，EcuM请求SHUTDOWN模式，SHUTDOWN Target设置为SHUTDOWN。 |

 

# **13.****多核-Multicore**

## **13.1 多核功能简介**

本节介绍BSW模块在不同分区上的分布。

 

一个分区可以被看作是映射到一个核上的一个独立的部分。因此，每个核(包括单核和多核架构)至少包含一个分区，但也可以包含任意数量的分区。但是没有一个分区可以跨越多个核。

 

BSW模块可以分布在不同的分区上，因此也可以分布在不同的内核上。一些BSW模块（如BswM）必须包含到每个分区中。其他模块(如OS或EcuM)被包含到每个核心的一个分区中。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUj3gpGOuVFTMbYt4KWPWFTH5jeiaSLFzB9PrXib1FPPwicmSiagia9fNlo3Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图20-ECU内部的分区Partitions

 

在多核架构中，EcuM必须以每个核存在一个实例的方式分布。

 

有一个指定的主内核，其中引导加载程序通过EcuM_init启动主EcuM。主EcuM启动一些驱动程序，决定Post-Build配置，并启动所有剩余的核心与所有卫星EcuM。然后，每个EcuM启动本地Core操作系统和所有本地BswM(在每个分区中恰好驻留一个BswM)。

 

如果相同的EcuM映像在ECU的每个核心上执行，ECU Manager的行为必须在不同的核心上有所不同。这可以由ECU管理器完成，首先测试它是在主核还是在从核上，并采取适当的行动。

 

ECU管理器模块支持与传统ECU相同的阶段(即STARTUP, UP, SHUTDOWN和SLEEP)。

 

如果使用了安全机制，ECU状态管理器必须以完全信任级别运行。

 

## **13.2 主核-Master core**

多核系统有一个主核。主核由引导加载程序决定。主核的EcuM模块作为第一个BSW模块启动，并执行初始化操作。然后开始初始化其他核的EcuM模块。EcuM完成初始化后，对应核的OS和BswM也完成了初始化。

 

## **13.3 从核-Slave core**

在每个从核上，必须运行一个EcuM模块。一个核上可以包含多个分区，每个核必须包括一个EcuM。

 

## **13.4 主从和通信-Master Core and Slave Core Signalling**

操作系统为同步主核和从核上操作系统的启动提供了基本机制。Scheduler Manager为跨分区边界的BSW模块通信提供了基本机制。每个核心有一个BSW模式管理器负责启动和停止RTE。

 

## **13.5 示例-关机同步 Shutdown Synchronization**

在调用ShutdownAllCores之前，“主”ECU管理模块必须启动所有“从”ECU管理模块的关闭，并且必须等待所有模块对它们负责的BSW模块进行反初始化并成功关闭。

 

因此，主ECU管理模块设置了一个关机标志，所有从模块都可以读取该标志。EcuM为每个配置的从核激活后续任务。从模块读取主例程中的标志，并在被请求时关闭。任务名称为“EcuM_SlaveCore<X>_Task”，其中X为数字。该任务需要由系统集成-er配置。需要激活的任务数可以通过计算EcuMPartitionRef的实例减1来计算，因为主服务器使用一个EcuMFlexPartionRef。

 

示例:配置三个EcuMPartitionRef实例。然后在调用EcuM_GoDown()时，“EcuM_SlaveCore1_Task”和“EcuM_SlaveCore2_Task”会被启动。从模块读取主例程中的标志，并在被请求时关闭。

 

操作系统扩展了OSEK SetEvent函数。一个核上的任务可以等待另一个核上设置的事件。下图说明了如何在调用ShutdownAllCores之前同步内核的问题(省略了反初始化的细节)。Set/WaitEvent函数接受一个位掩码，该位掩码可以用来指示各个从核上的shutdown就绪状态。每一个来自“从”ECU管理模块的SetEvent调用将停止“主”ECU管理模块的等待。因此，“主”ECU管理器模块必须跟踪各个从核的状态，并设置等待，直到所有核都已就绪。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUVGZdfUleIUqTyVjgpxcjibcbLFXPWwxmYOUYV6AwtAhuo3ZpkCib9xgw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图21-关机同步

 

## **13.6 UP Phase**

从硬件的角度来看，唤醒中断可能发生在所有的核上。然后整个ECU被唤醒，运行在上面的EcuM处理唤醒事件。

 

与单核情况一样，BswM（由集成者配置）负责控制ECU资源，确定本地核心可下电或深度休眠，以及在将其核心的控制移交给EcuM之前，去初始化适当的应用程序和BSW。

 

## **13.7 STARTUP Phase**

ECU管理器模块在所有核心上的功能几乎相同。也就是说，对于单核的情况，ECU Manager模块执行Startup指定的步骤;最重要的是启动操作系统、初始化SchM和启动核心本地bswm。

 

主EcuM在调用InitBlock 1并进行reset/wakeup验证后激活所有从核。在被激活之后，从核执行它们的启动例程，在它们的核心上调用EcuM_Init。

 

在每个EcuM调用其核上的StartOs之后，操作系统在每个核启动钩子函数之前同步这些核，并在每个核上执行第一个任务之前再次同步这些核心。

 

每个分区上的一个BswM必须为该核心启动RTE。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUOurBl8eXXZgFyNvMZqdVkcTGzQ2KOicEoJdU3Xqp8eHGIOjibW7CFQeg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图22-Master Core StartPreOS Sequence

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUHJVdH9PAjoqsUfPU4icjaP2pp9YlpTMicQYiciaoJlibl9du6Y8kiaCAODkQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图23-Master Core StartPostOS Sequence

 

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUl5wdyR9h9S8PgJAcJILylGupu4RtxcKmHzOYibYN2WCMdZCu4a6Wwxg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图24-- Slave Core StartPreOS Sequence

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbU7ha01NlmuNh0ibsqml96hR7btAYDdI1bVwrhSO4ia0B9nZQPJmLUjdiaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图25-Slave Core StartPostOS Sequence

 

## **13.8 SHUTDOWN Phase**

不支持个别核心关闭(即当ECU的其余部分继续运行时)。所有核心同时关闭。

 

当ECU应该关闭时，主ECU管理模块调用ShutdownAllCores而不是通过某种方式调用各个核上的ShutdownOs。ShutdownAllCores停止所有核上的操作系统，同时也停止所有核。

 

因为主核可以在所有从核完成处理之前发出ShutdownAllCores，所以在进入SHUTDOWN之前必须同步内核.

 

BswM(分布在所有分区上)确定ECU应该关闭并与ECU中的每个BwsM同步。所有BswM反初始化所有分区的Bsw、Swc和Cdd，并向其他BswM发送适当的信号，表明它们准备关闭。

 

对于ECU的关闭，BswM(位于主EcuM的同一个分区中)最终调用主核上的GoOff，该GoOff将请求分发给所有的从核。“主”EcuM对BswM和SchM进行反初始化。从内核上的EcuMs取消了它们的SchM和BswM的初始化，然后发送一个信号，表明内核已经为ShutdownOs做好了准备。

 

主EcuM等待来自每个从核EcuM的信号，然后像往常一样在主核上启动关机(主EcuM调用ShutdownAllCores，并且ECU与全局关机挂钩放在bed上)。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUcNLofyro760s7FuNaMQO88Ht7C0P9qnsJlcOOtEt8CKwG4vJOSHwpQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图26- Master Core OffPreOS Sequence

 

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUrde3f9hgffo1JQb4qjoiauWwGxChu5FPBibYB9rFONew30FFaIzxdibzA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图27-Master Core OffPostOS Sequence

 

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUs8ZHU4usmeoQ0c3ibyTyyGvnZHiaoUaJ9l9vpwRpKhUHvdurTXPhx5Eg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图28-- Slave Core OffPreOS Sequence

 

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUYczUBvzOwfiaBpwIicGnibibFOSDict2U8sUFqrZLXHCgRn8BgKwtJGG6uw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图29-Slave Core OffPostOS Sequence

 

## **13.9 SLEEP Phase**

当Shutdown目标Sleep]被请求时，所有内核将同时休眠。MCU必须为每个核心发出一个Halt指令。由于任务的定时和优先级是能在本地核的OS中完成，因此调度程序和RTE都不需要在MCU进入Halt状态后进行同步。因为主核可以在所有从核完成处理之前让MCU停止，所以内核必须在进入GoHalt之前进行同步。

 

BswM确定应该启动睡眠，并为每个核心分配适当的ECU模式。从核上的bsw、swc和cdd必须被它们的分区本地BswM通知，适当地去初始化并向BswM发送适当的模式请求，以表明它们的准备就绪。

 

如果ECU被置于睡眠状态，“halt”必须被同步，以便在主核计算校验和之前所有的从核都被停止（Halted）。主核上的ECU管理器模块使用了与GoOff上同步核相同的“信号”机制。

 

类似地，主核上的ECU管理器模块必须在从核从“停止”状态释放之前验证校验和.

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUNZEOMTnUlxoAicKP15xjjmZczTNjMlPrgibjmY3Y5o0KDNQKgtMRbh0g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图30- Master Core GoSleep Sequence

 

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUBhtrwN3fKf22l0SzhalQ8NH7Qte5M0YqDZrYl2nrSlbzVObbb130gQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图31-Master Core Halt Sequence

 

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUibpb4eFG20RLYJ7e7am6FVe0PgIpVqlzUWXP9p7XPco8Rrrh1MoGhgQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图32- Master Core Poll Sequence

 

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbU3bt9FPmvbVaYjIRd3a6ZJTt9MeolEL0qfkAtLE93RBFCUw0eGFSbrQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图33- Master Core WakeupRestart Sequence

 

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUnalYtETwGGsoP5rFTgp699NfCoEuO6Rcn6Q5THK7QE4JzbPAIW6COg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图34-Slave Core GoSleep Sequence

 

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUibd1D8YG6Uib29zTZBekGWI93GLIh1DgDefMtus99iabDpstbFb4lpCkQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图35-Slave Core Halt Sequence

 

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUGIzvicYzuDZdPSZQbpRRpibymmFUb3GLoEA2R49Nr1Jibaiazkga3aukNQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图36-Slave Core Poll Sequence

 

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcwuofibWknGdibL2mEMjf9dbUM72kQbKcZibx0s99BhK9dpxspoHOaEj8pHJklibVapVeTY8MMS5D9IoA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图37-Slave Core WakeupRestart Sequence

 

# **14.****EcuM在使用经验**

多数情况下MCU在使用的时候一般都会使用相关的SBC芯片，因此在EcuM的Shutdown Target中直接选择对应的OFF模式就可以。在使用了外部的SBC控制相关的MCU的下电的时候，在OFF阶段最后调用SBC进入到Sleep的指令。

 

在MCU走最后的下电，让外部的SBC休眠流程的时候一定要确保SBC唤醒相关的引脚没有高电平，否则操作SBC进入休眠会导致SBC相关的错误。

 

在APP层可以添加相关的模块单独做ECU的唤醒检测，并不需要使用EcuM内部的唤醒功能做，也可以基于两者设计唤醒相关的检测机制。

 

APP中也可以定义模式管理模块来协调BSW的模式管理，同时管理APP层的各个模块状态切换和运行。

 

在多核下电的时候，一定要让从核进行GetCoreID的判断，保证主核进入到最终的下电操作。

 

EcuM_OnGoOffTwo和EcuM_AL_SwitchOff可以直接使用一个实现，并不一定完全使用CP AUTOSAR定义的这两个下电流程。

 

对于RESET的Shutdown Target可以不要做，直接做成OFF，然后根据相关的功能（DCM请求、程序流监控出错、内存相关的权限监控出错等）使用直接调用Mcu_PerformReset相关的函数执行RESET操作。



***Note:未完待续，下一篇--终极篇，ISOLAR with DavinceCFG工具下EcuM & BswM配置及使用实例\***