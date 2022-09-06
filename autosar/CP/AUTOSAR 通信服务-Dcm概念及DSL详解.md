# AUTOSAR 通信服务-Dcm概念及DSL详解

**前言**

AUTOSAR Dcm模块的分享分为Dcm模块概念详解和Dcm模块配置及代码分析，具体的项目实战请关注本号的后续文章，本篇为Dcm模块简介及DSL子模块详解篇。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczuYCicL573OokUicFZZshN08gXicC819h1wiaFbcVNIYyFm7AfSJbf2DL8aQODeSlaA18dMBrzSmP8Rw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**正文**

# **1. Introduction and functional overview**

诊断通信管理(Diagnostic Communication Manager, DCM)模块作为AutoSar诊断模块的重要组成部分，主要负责诊断数据流和管理诊断状态，包括诊断会话、安全状态及诊断服务分配等。

主要功能贯穿汽车的开发生产及售后等过程，如开发过程中EMC、DV等实验均可使用诊断服务实现，生产过程中的软件下载更新、ECU产线EOL、汽车产线EOL等、售后过程中读取DTC、控制输出调试功能等。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczuYCicL573OokUicFZZshN08R2sC6zPEGTyoiadaq3z9zTlkn8Ar4PNQsqzE5a3WzIiczf2ibOaYX0icrg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



DCM模块相关的标准主要包括三部分：ISO 14229(UDS，DCM遵循的主要标准)、ISO 15031(ISO 15031 (1-7))及SAEJ1939(OBD，与OBD相关的$01 -$0A服务)。

 

从网络分层角度看，DCM模块属于上层模块，主要为应用层提供服务。主要包括5-7层，包括会话层服务及应用层等，会话层包括服务定时及服务分配等，应用层为具体的服务功能实现。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczuYCicL573OokUicFZZshN08pZNJ0jqddm1esPUln7AOopm78bZoqRPZ7ptOtDb3ibl5hjxbTe4KLDA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



在AUTOSAR架构中，Dcm（Diagnostic Communication Manager）模块位于通信服务层。Dcm模块是独立于具体的网络的（不依赖于具体的CAN,Lin,Eth,Flexray等网络来实现）。PduR模块为Dcm模块提供独立于具体网络的接口。Dcm模块从PduR模块接收诊断信息，Dcm模块在内部处理和检查诊断消息。作为处理所请求的诊断服务的一部分，Dcm将与其他BSW模块或SW-Components(通过RTE)交互，以获取所请求的数据或执行所请求的命令。诊断服务处理与特定的服务请求强绑定（不同的诊断请求依赖于不同的一个或几个模块来实现）。通常，Dcm将汇集收集到的信息，并通过PduR模块发送回消息。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczuYCicL573OokUicFZZshN08xlv0INQXjnmgiaDtyap1kHXWBiacvcvibekibEiaw5ric4enibAjXdVVgmKgg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



# **2. Dependencies to other modules**

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczuYCicL573OokUicFZZshN08c3aX8Z3Yrg9NuaQWpJastC8L8qH2WKLNRZoQMMt5TGKMsichd3mMhwQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**Dcm和Dem的交互：**DEM模块提供了检索与故障内存相关的所有信息的功能，以便Dcm模块能够通过从故障内存中读取数据重新响应测试人员的请求，通俗的讲就是Dcm能够读取Dem记录的DTC信息。

 

**Dcm和PduR的交互：**PduR模块接收和发送诊断数据。PduR为Dcm模块提供一个与具体通信协议无关的接口。

 

**Dcm和ComM模块的交互：**Dcm模块可以指示状态“活动”和“非活动”用于诊断通信。Dcm模块提供了处理通信需求“完全/静默/无通信”的功能。此外，Dcm模块提供了在ComM模块要求时启用和禁用诊断通信的功能。

 

**SWC通过和RTE接口和Dcm交互：**Dcm模块在完成诊断功能的时候需要通过RTE接口来读写/函数调用其他SWC的数据/服务。

 

**BswM和Dcm模块的交互：**如果Dcm的初始化是从引导加载程序跳转的结果，则Dcm通知BswM应用程序已更新。Dcm也向BswM指示通信模式的改变。

# **3. Fun****ctional specification**

## **3.1 Dcm子模块功能概述**

诊断通信管理(DCM)主要包括三个子模块：诊断服务层(Diagnostic Service Layer，DSL)、诊断服务调度(Diagnostic Service Dispatcher, DSD)、诊断服务处理(Diagnostic Service Processing, DSP)

 

Diagnostic Service Layer：确定诊断数据请求和响应的数据流；监控和确保诊断请求和响应的时序，管理诊断状态（特别是诊断会话和安全状态）

 

Diagnostic Service Dispatcher：接收到的诊断请求转发给数据处理器；当数据处理器触发时，通过PDUR传输诊断响应。

 

Diagnostic Service Processing：处理实际的诊断请求。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczuYCicL573OokUicFZZshN08QibjibxN1xyByEoYnGhwzcMty5DibAk7AgEEYweczahIC84a3sIezYUew/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## **3.2 诊断会话层(DSL)**

DSL模块主要用于诊断请求的处理及诊断时序的控制等，具体存在几个功能如下：

### **3.2.1 DSL功能概述**

**1）处理诊断请求：**

收到请求时，PDUR会将数据诊断请求数据从下层的Buffer中Copy到诊断的接收Buffer中，DSD会从此Buffer中取出数据进行处理。

 

维持诊断功能在线（保活逻辑）。

 

**2）处理诊断响应：**

收到发送数据时，PDUR会将数据诊断请求数据从DSL的发送Buffer中Copy到PDUR的发送Buffer中，具体后续处理由CAN的协议栈进行处理。

 

保证诊断时序。

 

支持周期性传输数据。

 

支持事件型响应（ResponseOnEvent）。

 

支持分段响应（segmented response）。

 

支持由应用触发的挂起响应（ResponsePending）。

 

**3）管理安全等级：**

用于获取或者设置安全等级

 

**4）会话状态处理：**

管理会话状态（session state）。

 

非默认会话的激活状态跟踪。

 

允许修改时序（应用可以通过诊断命令动态修改时序，也就是修改时序参数）。

 

**5）诊断协议处理：**

处理不同的诊断协议。

 

管理资源。

 

**6）通信模式处理：**

处理通信请求（Full-/Silent-/No Communication）。

 

标识诊断功能是否激活（active/inactive）。

 

打开/关闭（Enabling/disabling）所有类型的诊断传输。

 

### **3.2.2 将请求从PduR模块转发到DSD子模块**

带有诊断属性的PDU到达PduR模块后，PduR调用Dcm_StartOfReception,通知Dcm模块接收的数据大小,并提供第一帧或单帧的数据,如果数据规模溢出缓冲区大小,或者请求的服务不可用允许Dcm拒绝接收诊断数据。PduR模块之后调用Dcm_CopyRxData函数请求Dcm模块拷贝数据到Dcm buffer。

如果DCM模块接收诊断请求（Dcm_StartOfReception,函数返回成功），PduR模块就会调用Dcm_TpRxIndication通知到Dcm模块诊断请求接收完成。

 

### **3.2.3 维持诊断功能在线（保活逻辑）**

DSL子模块需要实现诊断会话在线功能，测试者通过物理寻址方式发送0x3E 80诊断请求，DSL接收到0x3E 80的请求后需要维持当前的诊断会话不退出（通过重置超时计数器S3Server）。

 

### **3.2.4** **将DSD子模块的响应转发到PduR模块**

DSD模块准完成诊断响应后会向DSL模块请求发送诊断响应报文。PduR模块接收到请求后调用调用PduR_DcmTransmit()函数发送数据，数据发送后，PduR模块调用Dcm_TxConfirmation()函数通知Dcm模块数据发送结果。

 

### **3.2.5 通用连接处理**

以CAN网络上的通用诊断连接处理为例。CAN网络上的每个ECU节点内部都会记忆两个诊断请求ID，物理寻址请求ID和功能寻址请求ID。

 

示例：Tester为连接到CAN网络上发送诊断请求的上位机，可发送任意诊断报文。ECU_A本地配置的物理寻址ID为0x728，功能寻址ID为0x7DF。ECU_B本地配置的物理寻址ID为0x72E，功能寻址ID为0x7DF。

 

**物理寻址：1:1的单播诊断请求**

Tester发送物理寻址ID为0x728的诊断请求时只有ECU_A会响应。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczuYCicL573OokUicFZZshN08icysFjPkSB50cVw35nVwRyWiamtxg1xXw9KQ4d4UIQmvSCSpk1GeVodg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



Tester发送物理寻址ID为0x72E的诊断请求时只有ECU_B会响应。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczuYCicL573OokUicFZZshN08c1BM4Go0OaLibiavicgVc0D8bcbeqTs8NHswE79eIFa2P90npouIiarRPQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

**功能寻址：1:n的广播诊断请求**

Tester发送功能寻址ID为0x7DF的诊断请求时ECU_A和ECU_B都会响应。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczuYCicL573OokUicFZZshN08PInICbE1yZkwTSBHUpENfIHxunAtmOyzwicJ5PjB5ia8umW6ukWFibd2g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

### **3.2.6 发送Busy响应来保证诊断时序**

上位机发送诊断请求后会要求ECU在一个最大响应时间（一般是50ms）内发出响应报文，但是对于一些请求的处理，ECU内部会耗时比较长（比如0x2E写NVM的请求），来不及在规定时间发出响应。这个时候ECU可以发送0x78（Pending）的否定响应吗来通知上位机ECU内部繁忙（Busy），需要上位机等待（Pending，一版Pending时间可以设置为1秒）。上位机收到0x78（Pending）的否定响应码后就会等待，直到ECU发出积极响应或者Pending时间超时。这个发送Bussy响应来保证诊断时序的功能由DSL模块实现。

 

### **3.2.7 支持周期传输**

UDS提供的“ReadDataByPeriodicIdentifier (0x2A)”服务，用于向由一个或多个periodicDataIdentifiers标识的ECU请求定期传输数据记录值。实际项目中没有用到过。

 

### **3.2.8 支持ROE传输。**

通过UDS Service ResponseOnEvent (0x86)，测试人员可以请求ECU启动或停止传输由指定事件发起的响应。当注册一个事件进行传输时，测试者也指定相应的服务来响应(例如:UDS service ReadDataByIdentifier 0x22)。实际项目中没有用到过。

 

### **3.2.9支持分段响应**

如果启用(DcmPagedBufferEnabled=TRUE)， Dcm module将提供一种机制来发送比配置的和allocated诊断缓冲区更大的响应。也就是分段响应机制。实际项目中没有用到过。

 

### **3.2.10 支持由应用程序触发的响应**

在某些情况下，例如在例程执行的情况下，应用程序需要立即请求一个NRC 0x78(响应等待)，该请求应立即发送，而不是仅仅在到达响应时间之前(P2ServerMax分别为P2*ServerMax)。

 

当Dcm模块调用一个操作并得到一个错误状态DCM_E_FORCE_RCRRP时，DSL子模块将触发NRC 0x78(响应等待)的负响应传输。这个响应需要从一个单独的缓冲区发送，以避免覆盖正在进行的请求处理。

 

### **3.2.11 管理安全等级**

**安全等级理解：**DCM模块的每个诊断服务可以配置安全访问等级，一般有无需通过安全访问、需要通过安全等级1访问、需要通过安全等级2访问三种等级。比如对于0x2E下的服务都设定为需要过安全访问等级1后，我们在请求0x2E的服务时需要通过特定的安全校验后才能通过安全访问等级1，这样才能得到0x2E服务的积极响应。

 

DLS子模块应该保存当前激活的安全等级状态。DLS模块提供两个接口用来设置和获取当前安全等级：

获取当前激活的安全等级: Dcm_GetSecurityLevel

设置新的安全等级: DslInternal_SetSecurityLevel()

 

DCM模块初始化完后，安全等级设置为0x00（DCM_SEC_LEV_LOCKED，使能了安全访问功能）。

如果我们通过安全校验算法后后使得安全等级不为0，当发生以下任意事件后，安全等级状态将回到初始状态0x00（DCM_SEC_LEV_LOCKED）：

1）诊断会话（session）从非缺省会话（0x0, default session）切换到非缺省会话，包括从当前非缺省会话切换到本身自己。

2）诊断会话（session）从非缺省会话（0x0, default session）切换到缺省会话。

 

### **3.2.12管理会话状态**

会话：session，一般有三种，缺省会话（0x00, Default session）、编程会话（0x01, Programming session）、扩展会话（0x02Extended session）。每种会话下可以进行的诊断服务不同，根据实际需求由OEM自己定义。一般来说诊断（UDS）刷写功能需要在编程会话下进行，涉及到NVM关键存储数据的写功能需要在扩展会话下进行。根据实际需求可以自己定义会话，比如定义0x60（EOL session）专门用于EOL工厂下线处理（关于EOL下线，后面将会由文章具体接收，请关注本公众号后续文章）。

 

DLS子模块应该保存当前激活的会话状态。DLS子模块应该提供两个接口用于获取/设置会话状态：

1）获取当前会话状态：Dcm_GetSesCtrlType()

2）设置会话状态：DslInternal_SetSesCtrlType()

 

DCM模块初始化完成后，诊断会话进入缺省会话。

 

### **3.2.13 跟踪激活的非缺省会话**

从缺省会话进入非缺省会话后，S3Server定时器就会开始计时（只要收到诊断请求报文就会清零），如果定时器超时（S3Server），DLS模块就会将会话状态切换到缺省会话状态。

 

### **3.2.14 允许修改时序**

P2ServerMin, P2ServerMax, P2*ServerMin, P2*ServerMax, S3Server这些参数值将会影响DCM模块的诊断响应时序。

 

P2ServerMin=0, P2*ServerMin=0, S3Server = 5为固定值。协议参数影响诊断会话层的时序，不会影响到传输层时序。当协议激活时，可以通过以下方法修改其中一些定时参数:

• UDS Service diagnostics sessioncontrol (0x10)

• UDS服务AccessTimingParameter (0x83)

 

DSL子模块提供了以下功能来修改定时参数:

•提供活动定时参数，

•设置新的定时参数。只允许在发送响应之后激活新的计时值

 

### **3.2.15 处理不同的诊断协议**

DSL子模块需要处理不同的诊断协议。实际应用中一般都是使用DCM_UDS_ON_CAN协议。

 

### **3.2.16 管理资源**

由于资源有限，DSL应该考虑以下几点作为设计提示:

1）在Dcm模块中只允许使用和分配一个诊断缓冲区，这个缓冲区随后用于处理诊断请求和响应

2）NRC 0x78 (Response pending)响应的输出是通过一个单独的缓冲区完成的

3）支持分页缓冲处理

 

### **3.2.17 通信模式处理**

DCM模块和ComM模块的交互由DSL子模块完成。ComM模块把每个通信通道（channel）的当前通信状态（No-com, Full-com, Silent-com）通知给Dcm模块。Dcm模块的诊断功能将会调用ComM模块的接口功能可能会阻止Ecu进入shutdown模式（ActiveDiagnostic ==’DCM_COMM_ACTIVE’时ECU必须保持唤醒状态，也就是不允许 进入shutdown/sleep）。

 

应用程序可以调用Xxx_SetActiveDiagnostic()接口通知Dcm模块ActiveDiagnostic的当前状态。

 

**No Communication****：**

ComM模块调用 Dcm_ComM_NoComModeEntered接口通知Dcm模块通信关闭。Dcm模块收到通知后就会Disable所有的诊断接收/传输。

 

**Silent Communication**:

ComM模块调用 Dcm_ComM_SilentComModeEntered接口通知Dcm模块通信静默。Dcm模块收到通知后就会Disable所有的诊断传输。

 

**Full Communication**:

ComM模块调用  Dcm_ComM_FullComModeEntered接口通知Dcm模块通信静默。Dcm模块收到通知后就会Enable所有的诊断接收/传输。

 

**Diagnostic Activation State**:

Dcm将所有网络的内部诊断状态通知ComM模块。网络上的诊断状态有两种选择。在“活动”诊断状态下，Dcm正在处理来自该网络的一个或多个诊断请求，或者Dcm处于非默认会话中。在“非活动”诊断状态下，Dcm处于默认会话，并且没有在该网络上处理诊断请求。

 

当一个网络没有正在进行的通信时，Dcm会将诊断ac激活状态设置为' inactive '。当网络上有诊断通信时，Dcm将诊断状态设置为“活动”。在任何非默认会话中，诊断状态保持为“活动”状态。通信状态也可以通过API Xxx_SetActiveDiagnostic来控制。