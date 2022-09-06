# AUTOSAR 通信服务-ComM概念详解

**AUTOSAR ComM模块的分享分为ComM模块概念详解和ComM模块配置及代码分析，具体的项目实战请关注本号的后续文章，本篇为ComM模块概念详解篇。**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcyb4CUB4eIA1ASJb1BSk3ockcVoR6gQgM3yl4Os8Seevp21UKAM5A7broU81U0tmXGstzEzkC0KxQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**1 Introduction and functional overview**

ComM模块是BSW一个组件成员。ComM作为一个资源管理器封装了底层的通信服务。ComM模块控制与通信有关的基本软件模块，而不是软件组件或可运行实体。ComM模块收集通信请求者的总线通信访问请求，并协调总线通信访问请求。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcyb4CUB4eIA1ASJb1BSk3ocNU2ibdhiavGibrp7ZXB4micO7wxFsEulfgXFa9CialSLnb1rHT4EACa6P7g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

ComM模块存在的目标是：

为用户简化总线通信栈的使用。这包括一个简化的网络管理处理。

 

在一个ECU上协调多个独立软件组件的总线通信栈(允许信号的发送和接收)的可用性。

 

备注:用户不应该对硬件有任何了解(例如，在哪个通道上通信)。用户只需请求“通信模式”，ComM模块就会切换相应通道的通信能力。

 

提供一个API来禁用信号的发送，以防止ECU(主动)唤醒通信总线。

 

**Note****:**在每个消息都可以唤醒总线时，在FlexRay上只能用所谓的唤醒模式唤醒总线。

 

通过实现每个通道的通道状态机来控制一个ECU的多个通信总线通道。

 

**Note****:**ComM模块向相应的总线状态管理器模块请求通信模式。实际的总线状态由相应的总线状态管理器模块控制。

 

提供强制让ECU保持总线在“无通信”模式下保持清醒的可能性。

 

通过分配所请求的通信模式所需的所有资源来简化资源管理。

 

**Note****:**例如，当用户请求“完全通信”模式时，检查是否允许通信，并防止ECU在通信期间关闭。

 

# 2. **Dependencies to other modules**

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcyb4CUB4eIA1ASJb1BSk3ocVquK2BlxF39bdAy6J8NPLKo3iaTfbLEbGEChJoiahxiaWVHYTMx49eDkA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

**AUTOSAR Runtime Environment (RTE)****:** 每个用户都可以请求一个通信模式。RTE将用户请求传播给ComM模块，并将ComM指示的通信模式传递给用户。

 

**ECU State Manager (EcuM)****:** **EcuM-Fixed**

EcuM-Fixed负责初始化ComM，负责验证唤醒事件，并在唤醒验证时向ComM发送指示。EcuM-Fixed将指示ComM是否允许启动通信。然后EcuM-Fixed必须与ComM检查ECU是否可以关闭，即通信是否在进行中。

 

**ECU State Manager (EcuM)****:** **EcuM-Fixe**

以上功能(允许通信和ECU的关闭)由EcuM-Flex和BswM一起处理。如果在BswM的动作列表中配置为能够通过BswM请求ComM模式，BswM将用户请求传播到ComM模块。

 

**Basic Software Mode Manager (BswM)****:**

BswM实现了模式仲裁和模式控制两个功能，允许应用程序模式管理和车辆模式管理。如果在动作列表中配置了对Com_IpduGroupControl的调用，BswM将控制COM中的PDU组。ComM将所有通道主状态变化通知到BswM。

 

如果使用EcuM-Flex, BswM将是否允许通信通知到ComM。

 

**NVRAM Manager****:**

ComM使用NVM的功能却存储和读取非易失性数据。

 

**Diagnostic Communication Manager (DCM)****:**

DCM执行诊断pdu的调度。如果要进行诊断，DCM作为用户通过“DCM_ActiveDiagnostic”指示请求通信模式COMM_FULL_COMMUNICATION。DCM不提供启动/停止发送和接收的API，但保证通信能力符合ComM模块的通信模式。

 

**LIN State Manager****:**

LIN State Manager控制与ComM模块的通信模式对应LIN总线的实际状态。ComM模块向LIN State Manager请求“Communication Mode”，LIN State Manager将“Communication Mode”映射为总线状态。

 

**CAN State Manager:**

CAN状态管理器控制CAN总线的实际状态，对应于ComM模块的通信模式。ComM模块从CAN状态管理器请求一个通信模式，然后CAN状态管理器将通信模式映射到总线状态。

 

**FlexRay State Manager****,Ethernet State Manager**类似于Lin/Can State Manager。

 

**Network Management (NM)****:**

ComM模块使用NM同步控制整个网络的通信能力(同步启动和关闭)。

 

**Communication (COM):**

AUTOSAR通信模块(COM)应使用COM信号分发pnc的状态信息。

# 3. **Functional specification**

通信管理(ComM)模块简化了用户的资源管理，其中用户可以是可运行实体、SW-C、BswM(例如通过BswM的SW-C请求)或DCM(诊断所需的通信)。

 

ComM模块提供了三种通信模式：最高等级的COMM_FULL_COMMUNICATION模式，最低等级的COMM_NO_COMMUNICATION模式，只有AUTOSAR NM模块才会使用的COMM_SILENT_COMMUNICATION模式。普通用户（user）只能通过ComM_RequestComMode()函数请求COMM_FULL_COMMUNICATION模式和COMM_NO_COMMUNICATION模式。

 

ComM模块允许查询特定用户请求的“通信模式”。通过ComM模块可以查询通道的实际“通信模式”。

 

在COMM_FULL_COMMUNICATION模式下，ComM模块应该允许在该模式的物理通道上进行传输和接收。

 

在COMM_NO_COMMUNICATION模式下，ComM模块不允许在该模式的物理通道上进行传输和接收。

 

如果有多个用户对同一通道请求不同的通信模式，ComM模块以高优先级的通信模式作为最终的通信模式。

 

ComM模块不应该对用户请求进行排队。同一用户的最新用户请求应覆盖旧用户请求，即使该请求尚未完成。

 

每一个通信通道都有一个目标通信模式，这个目标通信模式可能临时状态下和实际的通信模式（由对应的总线状态管理模块管理）是不一致的。因为通过相应的总线状态管理器模块进行模式切换需要时间，并且可以激活模式抑制。

 

ComM模块应将ComM_GetCurrentComMode()的调用传播到总线状态管理器模块，用于用户配置到的通道。状态请求必须传播到相应的总线状态管理器模块，因为ComM模块并不控制实际的总线状态。“普通的SW-C”不使用这个特性，因为他们不了解通道。这个特性对于有特权的sw - c是必要的，它们(必须)了解系统拓扑，例如系统诊断功能。

 

对于支持CommunicationAllowed标志的物理通道，ComM模块应该存储这些通道的CommunicationAllowed标志状态（TRUE or FALSE）。ComM初始化后的默认值为communication is not allowed，即CommunicationAllowed=FALSE。允许或不允许的通信状态变化应在ComM_CommunicationAllowed(<channel>， TRUE|FALSE)指示中提供给ComM。

 

## **3.1 Partial Network Cluster Management**

ComM对每一个局部网络集群（Partial network cluster, PNC）实现一个状态机。可以通过配置参数ComMPncSupport来使能是否使用这个功能，一般不使用这个功能。

 

## **3.2 ComM channel state machine**

ComM模块为每一个通道（Channel, CAN/Lin/Eth等）都实现一个独立的状态机。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcyb4CUB4eIA1ASJb1BSk3ocHFcZ1wyWuicvGSFB8NxUEj6iaooK1ibybNp7F0wwial3QL4zK06qZxKLRQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

ComM为每个通道维持的状态机共有三个状态:

**COMM_NO_COMMUNICATION****:**

COMM_NO_COMMUNICATION下有两个子状态，COMM_NO_COM_REQUEST_PENDING，COMM_NO_COM_NO_PENDING_REQUEST

 

**COMM_SILENT_COMMUNICATION****:** 状态机的初始状态。



**COMM_FULL_COMMUNICATION****:**

COMM_FULL_COMMUNICATION下有两个子状态， COMM_FULL_COM_NETWORK_REQUESTED, COMM_FULL_COM_READY_SLEEP

 

COMM_FULL_COM_READY_SLEEP和COMM_SILENT_COMMUNICATION是同步总线上的通信关闭所必需的。如果只有一个ECU关闭通信，其他ECU将存储错误，因为该ECU停止发送应用程序信号。

 

主要状态表示每个信道的通信能力的抽象状态，这些状态是用户关注的焦点。子状态表示中间状态，它们执行活动来支持与外部合作伙伴和管理协议(例如NM)的同步转换。

 

每个ComM通道状态机只能在子状态COMM_NO_COM_REQUEST_PENDING中评估其相应的通信状态标志CommunicationAllowed。

 

如果不再COMM_NO_COM_REQUEST_PENDING状态下，ComM_CommunicationAllowed(<channel>,FALSE)函数不会立即产生关闭通信的效果。也就是说，如果Channel通道状态不再COMM_NO_COM_REQUEST_PENDING状态（如，COMM_FULL_COMMUNICATION）下，ComM的Channel状态不会立即切换到 COMM_NO_COMMUNICATION状态。

 

每个ComM通道状态机负责用连接的总线状态管理器处理一个通道/网络，ComM模块包含一个或多个ComM通道状态机。ComM通道状态机直接与它连接的总线状态管理器通信，其他接口由ComM模块处理。每个状态下通信能力如下表：

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcyb4CUB4eIA1ASJb1BSk3oc0TpDMLI2Nrbgr1xtGwYemQt1V7ppxAYsywGkiaRDdvly6ibodqg6Otbw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



### **3.2.1 Behaviour in state COMM_NO_COMMUNICATION**

进入 COMM_NO_COMMUNICATION状态后首先进入COMM_NO_COM_NO_PENDING_REQUEST子状态。初始化后默认进入COMM_NO_COMMUNICATION状态时，ComM模块不通过RTE或BswM向用户指示模式变化。因为这个时候，RTE还没有完成初始化。

 

在进入COMM_NO_COMMUNICATION状态时，ComM通道状态机应关闭发送和接收功能。这应该由ComM通道状态机从总线状态管理器模块（CanSM，LinSM等）请求相应的通信模式来执行。

 

在进入COMM_NO_COMMUNICATION状态且配置参数ComMNmVariant=FULL时，ComM模块应通过Nm_NetworkRelease接口向network Management模块请求释放网络。

 

在状态COMM_NO_COMMUNICATION中，ComM通道状态机不能从总线状态管理器模块请求配置通道的总线通信。

 

其中一个信道的通信方式是本地的，ECU仍然可以通过其他信道进行通信。

 

#### **3.2.1.1 COMM_NO_COM_NO_PENDING_REQUEST sub-state**

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcyb4CUB4eIA1ASJb1BSk3ocdtsWff0yAk3tFlpS7Y5T25c1xTib2LVUYHqdibWM1RMWyrZRIJ4j21sQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

COMM_NO_COM_NO_PENDING_REQUEST子状态通过以下几种方式切换到COMM_NO_COM_REQUEST_PENDING子状态：

 

**方式1：**用户通过ComM_RequestComMode(<user>,COMM_FULL_COMMUNICATION)请求通信且通信限制是Disable的（Communication limitation disabled）。则**当前请求通道**切换到COMM_NO_COM_REQUEST_PENDING子状态。

 

**方式2：**DCM模块调用ComM_DCM_ActiveDiagnostic(ChX)且配置参数 ComMNmVariant=FULL|LIGHT|NONE，则**当前请求通道**切换到COMM_NO_COM_REQUEST_PENDING子状态。

 

**方式3：**ComM_EcuM_WakeUpIndication(ChX)被调用且配置参数ComMSynchronousWakeUp=FALSE，则**当前请求通道**切换到COMM_NO_COM_REQUEST_PENDING子状态。

 

**方式4：**NM模块重启，NM模块调用 ComM_Nm_RestartIndication()，则**当前请求通道**切换到COMM_NO_COM_REQUEST_PENDING子状态。

 

**方式5：**ComM_EcuM_WakeUpIndication函数被调用且配置参数ComMSynchronousWakeUp=TRUE，则ComM管理的**所有通道**都切换到COMM_NO_COM_REQUEST_PENDING子状态。

 

**方式6：** ComM_EcuM_PNCWakeUpIndication(<PNC>)函数被调用且配置参数 ComMSynchronousWakeUp=FALSE，则则**当前请求通道**切换到COMM_NO_COM_REQUEST_PENDING子状态。

 

**方式7：**ComM_EcuM_PNCWakeUpIndication(<PNC>)函数被调用且配置参数 ComMSynchronousWakeUp=TRUE，则ComM管理的**所有通道**都切换到COMM_NO_COM_REQUEST_PENDING子状态。

 

进入COMM_FULL_COMMUNICATION状态后，ComM通道状态机立即切换到子状态COMM_FULL_COM_NETWORK_REQUESTED。如果没有用户请求COMM_FULL_COMMUNICATION模式，则AUTOSAR NM响应COMM_FULL_COMMUNICATION模式。ComM的ComMTMinFullComModeDuration模块定时器防止在COMM_NO_COMMUNICATION和COMM_FULL_COMMUNICATION之间切换，以克服系统的init启动时间，在可能的用户请求发生之前。

#### **3.2.1.2** **COMM_NO_COM_REQUEST_PENDING sub-state**

COMM_NO_COM_REQUEST_PENDING状态切换到COMM_FULL_COMMUNICATION状态：CommunicationAllowed=TRUE

 

COMM_NO_COM_REQUEST_PENDING状态切换到COMM_NO_COM_NO_PENDING_REQUEST状态：没有任何的 COMM_FULL_COMMUNICATION挂起（Pending）请求。

 

### **3.2.2 Behaviour in state COMM_SILENT_COMMUNICATION**

COMM_SILENT_COMMUNICATION状态下，ComM将会使用对应通道的状态管理模块（CanSM, LinSM, EthSM等）的状态请求接口关闭通道的信号传输功能，保留通道的信号的接收功能。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcyb4CUB4eIA1ASJb1BSk3ocPicOQV7szTuP6ms8grmqibauicDD5CJV0D7ezicKGIZ3ZTgLL3iay9lFx1w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

COMM_SILENT_COMMUNICATION通过以下两种方式切换到 COMM_FULL_COMMUNICATION状态：

**方式1：**用户调用ComM_RequestComMode(<user>,COMM_FULL_COMMUNICATION)函数请求通信且通信限制Disabled。

 

**方式2：**ComM_DCM_ActiveDiagnostic(ChX)函数被调用且配置参数 ComMNmVariant=FULL|LIGHT|NONE。

 

**方式3：**接收到网络管理器模块调用ComM_Nm_NetworkMode()的通知。通道状态机切换到COMM_FULL_COMMUNICATION状态的COMM_FULL_COM_READY_SLEEP子状态。

 

COMM_SILENT_COMMUNICATION切换到COMM_NO_COMMUNICATION状态：接收到网络管理器模块调用ComM_Nm_BusSleepMode()的通知。

 

### **3.****2.3 Behaviour in state COMM_FULL_COMMUNICATION**

从其他状态切换到 COMM_FULL_COMMUNICATION状态时，如果没有特别的指定COMM_FULL_COMMUNICATION状态下的子状态，则默认进入COMM_FULL_COM_NETWORK_REQUESTED子状态。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcyb4CUB4eIA1ASJb1BSk3ocZyNdYyuM0OrnWtDhibtpJOwq6qQHSZf8MZZP5GO0ib19pHCpoIuUS4bA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在COMM_FULL_COMMUNICATION状态下，ComM通过调用对应状态管理模块（CanSM, LinSM, EthSM等）的接口开启对应通道的接收和发送信号的功能。也就是说，在COMM_FULL_COMMUNICATION状态下，对应通道的数据收发功能都是开启的。

 

COMM_FULL_COMMUNICATION状态切换到COMM_NO_COMMUNICATION状态：接收到网络管理器模块调用ComM_Nm_BusSleepMode()的通知。

 

COMM_FULL_COMMUNICATION状态切换到 COMM_SILENT_COMMUNICATION：接收到网络管理器模块调用ComM_Nm_PrepareBusSleepMode()的通知且配置参数 ComMNmVariant=FULL|PASSIVE。

#### **3.2.3.1** **COMM_FULL_COM_NETWORK_REQUESTED sub-state**

进入到COMM_FULL_COM_NETWORK_REQUESTED子状态后，如果配置参数ComMNmVariant=LIGHT|NONE，用于ComMTMinFullComModeDuration的定时器将会被开启。

 

从COMM_NO_COM_REQUEST_PENDING子状态切换到COMM_FULL_COM_NETWORK_REQUESTED子状态且 EcuM模块调用y ComM_EcuM_WakeUpIndication(<channel>)通知ComM模块有唤醒事件产生， ComM模块将调用Nm_PassiveStartup(<channel>)请求NM模块开启网络管理。

 

当进入COMM_FULL_COM_NETWORK_REQUESTED子状态时，Nm模块调用ComM_Nm_RestartIndication(<channel>)指示重启， ComM模块应向Network Management请求Nm_PassiveStartup(<channel>)。

 

当进入COMM_FULL_COM_NETWORK_REQUESTED子状态时，Nm模块调用 ComM_Nm_NetworkStartIndication(<channel>)指示NM开启， ComM模块应向Network Management请求Nm_PassiveStartup(<channel>)。

 

从其他状态切换到COMM_FULL_COM_NETWORK_REQUESTED状态且配置参数r ComMNmVariant=FULL且用户调用ComM_RequestComMode(<user>,COMM_FULL_COMMUNICATION)请求通信，则ComM模块将调用网络管理模块的Nm_NetworkRequest(<channel>)接口函数开启该通道的网络管理。

 

 

从其他状态切换到COMM_FULL_COM_NETWORK_REQUESTED状态且配置参数r ComMNmVariant=FULL且DCM模块调用 ComM_DCM_ActiveDiagnostic(<channel>)通知到ComM模块，则ComM模块将调用网络管理模块的Nm_NetworkRequest(<channel>)接口函数开启该通道的网络管理。

 

在 COMM_FULL_COM_NETWORK_REQUESTED状态下且配置参数ComMNmVariant=LIGHT|NONE且ComMTMinFullComModeDuration对应的定时器超时且没有用户调用ComM_RequestComMode(<user>,COMM_FULL_COMMUNICATION)请求通信且DCM模块没有调用ComM_DCM_ActiveDiagnostic(<channel>)函数，则ComM模块将该通道状态切换到COMM_FULL_COM_READY_SLEEP状态。

 

在 COMM_FULL_COM_NETWORK_REQUESTED状态下且配置参数ComMNmVariant=FULL且没有用户调用ComM_RequestComMode(<user>,COMM_FULL_COMMUNICATION)请求通信且DCM模块没有调用ComM_DCM_ActiveDiagnostic(<channel>)函数，则ComM模块将该通道状态切换到COMM_FULL_COM_READY_SLEEP状态。

 

在 COMM_FULL_COM_NETWORK_REQUESTED状态下且配置参数ComMNmVariant=PASSIVE，则ComM模块将该通道状态切换到COMM_FULL_COM_READY_SLEEP状态。

 

 

在 COMM_FULL_COM_NETWORK_REQUESTED状态下且DCM模块没有调用ComM_DCM_ActiveDiagnostic(<channel>)函数且请求了通信限制，则ComM模块将该通道状态切换到COMM_FULL_COM_READY_SLEEP状态同事删除ComMTMinFullComModeDuration定时器。

 

#### **3.****2.3.2 COMM_FULL_COM_READY_SLEEP sub-state**

进入 COMM_FULL_COM_READY_SLEEP状态后且配置参数rComMNmVariant=FULL，ComM模块调用对应通道的NM网络管理模块的Nm_NetworkRelease()函数请求释放网络。

 

进入 COMM_FULL_COM_READY_SLEEP状态后且配置参数ComMNmVariant=LIGHT，ComMNmLightTimeout对应的定时器被开启。

 

在COMM_FULL_COM_READY_SLEEP状态通过以下两种方式切换到COMM_NO_COMMUNICATION状态：

**方式1：**进入 COMM_FULL_COM_READY_SLEEP状态后且配置参数ComMNmVariant=LIGHT且ComMNmLightTimeout对应的定时器超时，ComM模块将该通道对应的状态机切换到COMM_NO_COMMUNICATION状态。

 

**方式2：**进入 COMM_FULL_COM_READY_SLEEP状态后且配置参数 ComMBusType=COMM_BUS_TYPE_INTERNAL，ComM模块将该通道状态机切换到COMM_NO_COMMUNICATION状态。

 

 

在COMM_FULL_COM_READY_SLEEP状态通过以下两种方式切换到COMM_FULL_COM_NETWORK_REQUESTED状态：

**方式1：**在COMM_FULL_COM_READY_SLEEP状态下且用户调用ComM_RequestComMode(<user>,

COMM_FULL_COMMUNICATION)请求通信且通信限制Disabled，则ComM将通道状态机切换到COMM_FULL_COM_NETWORK_REQUESTED状态。

 

**方式2：**在COMM_FULL_COM_READY_SLEEP状态下且配置参数 ComMNmVariant=FULL|LIGHT|NONE且DCM模块调用ComM_DCM_ActiveDiagnostic通知到ComM模块，则ComM将通道状态机切换到COMM_FULL_COM_NETWORK_REQUESTED状态。

 

在COMM_FULL_COM_READY_SLEEP状态下且配置参数ComMNmVariant=LIGHT，切换到 COMM_FULL_COM_NETWORK_REQUESTED状态后ComMNmLightTimeout对应的定时器将被删除。

 

## **3.3** **Bus communication management**

ComM模块应使用总线状态管理器模块的相应接口来控制通信的打开和关闭。如果配置参数commbusstype =COMM_BUS_TYPE_INTERNAL，则ComM模块将忽略控制通信能力的调用。

 

## **3.4** **Network management dependencies**

ComM模块应支持shutdown同步变量(配置ComMNmVariant)，如下表所示。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcyb4CUB4eIA1ASJb1BSk3ocBPaN65ZD8xeODRA0GibITdFYibDPTovGXicLD0ibB8fIZBujzYtiagmdStA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

只有变量FULL和PASSIVE保证了网络中所有节点之间的同步关机（shutdown）。注意，由于NmIf不能在所有网络准备进入睡眠状态之前启动同步关闭协调的网络，因此会考虑从ComM到NmIf的请求，以便在这样一个协调的总线上释放网络通信，但并不总是直接执行。NmIf仍然会回答E_OK，但网络不会被释放，直到所有协调的网络准备进入睡眠状态。

 

在LIGHT版本中，同步关机是不可能的，因此ECU可能会因为节点稍后关机的消息而不断重启(“切换”)。

 

当配置参数ComMNmVariant=LIGHT|NONE时，ComM模块应忽略对NM服务的调用。

 

如果配置参数ComMNmVariant=PASSIVE, ComM模块将忽略从NM调用Nm_NetworkRequest()

 

# 4.**关键API**

## **4.1** **Function definitions**

**ComM_Init****:**

**void ComM_Init(const ComM_ConfigType\* ConfigPtr)**

初始胡ComM模块，重启ComM模块管理的所有内部状态机。

 

**ComM_GetState:**

Std_ReturnType ComM_GetState(NetworkHandleType Channel, ComM_StateType* State)

通过State参数返回档期通道Channel的所处的状态。

 

**ComM_GetStatus:**

Std_ReturnType ComM_GetStatus( ComM_InitStatusType* Status)

通过Status参数获取ComM模块的当前状态，COMM_INIT/COMM_UNINIT。

 

**ComM_GetInhibitionStatus:**

Std_ReturnType ComM_GetInhibitionStatus(NetworkHandleType Channel,ComM_InhibitionStatusType* Status)

通过Status参数获取当前通道Channel的抑制状态。

 

**ComM_RequestComMode****:**

Std_ReturnType ComM_RequestComMode(ComM_UserHandleType User,ComM_ModeType ComMode)

用户User请求通信模式ComMode（ COMM_NO_COMMUNICATION，COMM_FULL_COMMUNICATION）。

 

**ComM_GetMaxComMode****:**

Std_ReturnType ComM_GetMaxComMode(ComM_UserHandleType User,ComM_ModeType* ComMode)

通过ComMode返回User请求的最大的通信能力（Full Communication, or No Communication, ro limitation/inhibition）。

 

**ComM_GetRequestedComMode****:**

Std_ReturnType ComM_GetRequestedComMode(ComM_UserHandleType User,ComM_ModeType* ComMode)

通过ComMode返回User当前请求的通信模式。

 

**ComM_GetCurrentComMode:**

Std_ReturnType ComM_GetCurrentComMode(ComM_UserHandleType User,ComM_ModeType* ComMode)

通过ComMode返回User当前处于的通信模式。内部实现直接使用对应通道的状态管理模块的接口（CanSM, LinSM等）获取当前通道的状态。

 

**ComM_PreventWakeUp****:**

Std_ReturnType ComM_PreventWakeUp(NetworkHandleType Channel,boolean Status)

改变通道Channel的一致状态。

FALSE: Wake up inhibition关闭 

TRUE: Wake up inhibition 打开

 

**ComM_LimitChannelToNoComMode****:**

Std_ReturnType ComM_LimitChannelToNoComMode(NetworkHandleType Channel,boolean Status)

更改从COMM_NO_COMMUNICATION到更高的Communication Mode的信道的抑制状态。

FALSE:限制 channel 到 COMM_NO_COMMUNICATION

disabled

TRUE: 限制channel 到 COMM_NO_COMMUNICATION enabled

 

**ComM_LimitECUToNoComMode****:**

Std_ReturnType ComM_LimitECUToNoComMode(boolean Status)

更改ECU(=所有通道)的抑制状态，将comm_no_communication改为更高的通信模式。

FALSE: 限制ECU 到COMM_NO_COMMUNICATION disabled

TRUE: 限制ECU 到COMM_NO_COMMUNICATION enabled

## **4.2 Callback notifications**

这些函数由ComM模块定义，其他模块使用。

### **4.2.1** **AUTOSAR Network Management Interface**

**ComM_Nm_NetworkStartIndication****:**

void ComM_Nm_NetworkStartIndication(NetworkHandleType Channel)

表示在总线休眠模式中收到了一个NM消息，这表示网络中的一些节点已经进入了网络活动模式。

 

**ComM_Nm_NetworkMode****:**

void ComM_Nm_NetworkMode(NetworkHandleType Channel)

通知到ComM模块网络管理模块已经进入了网络模式。

 

**ComM_Nm_PrepareBusSleepMode****:**

void ComM_Nm_PrepareBusSleepMode(NetworkHandleType Channel)

通知到ComM模块网络管理模块已经进入了预睡眠模式。

 

**ComM_Nm_BusSleepMode:**

void ComM_Nm_BusSleepMode(NetworkHandleType Channel)

通知到ComM模块网络管理模块已经进入了睡眠模式。

 

**ComM_Nm_RestartIndication:**

void ComM_Nm_RestartIndication(NetworkHandleType Channel)

如果NmIf已经开始关闭协调的总线，并且不是所有协调的总线都指示总线休眠状态，并且至少在一个协调的总线上，NM重新启动，然后，NM接口将使用已经指示总线休眠状态的通道的nmNetworkHandle调用回调函数ComM_Nm_RestartIndication。

 

### **4.2.2** **AUTOSAR Diagnostic Communication Manager Interface**

**ComM_DCM_ActiveDiagnostic****:**

void ComM_DCM_ActiveDiagnostic(NetworkHandleType Channel)

指示DCM的诊断请求通信开启。

 

ComM_DCM_InactiveDiagnostic:

void ComM_DCM_InactiveDiagnostic(NetworkHandleType Channel)

指示DCM的诊断请求通信关闭。

 

### **4.****2.3** **AUTOSAR ECU State Manager Interface**

ComM_EcuM_WakeUpIndication:

void ComM_EcuM_WakeUpIndication(NetworkHandleType Channel)

通知到ComM模块当前通道被唤醒。

 

### **4.****2.4** **AUTOSAR ECU State Manager and Basic Software Mode Manager Interface**

**ComM_CommunicationAllowed****:**

void ComM_CommunicationAllowed(NetworkHandleType Channel,boolean Allowed)

EcuM 或者 BswM通知ComM模块，通信是被允许的。

If EcuM/Fixed is used: EcuM/Fixed.

If EcuM/Flex is used: BswM

 

### **4.****2.5** **Bus State Manager Interface**

**ComM_BusSM_ModeIndication****:**

void ComM_BusSM_ModeIndication(NetworkHandleType Channel,ComM_ModeType ComMode)

由相应的总线状态管理器指示实际的总线模式。ComM应通过RTE和BswM将指示状态传播给用户。

 

### **4.****2.****6 COM Interface**

void ComM_COMCbk_<sn>( void )

当在COM中更新EIRA或ERA时，这个回调函数被调用。通话中只通知ComM ERA和EIRA的变化。实际的处理在下一次调用ComM_MainFunction_<Channel_Id>时完成，并更改相应的PN State机。

 

## **4.3 Scheduled functions**

**void ComM_MainFunction_<Channel_Id>(void)**

该函数应执行AUTOSAR通信活动的处理，这些活动不是直接由调用(例如来自RTE)发起的。每个通信通道都应有一个专用的主功能。



参考文档：Specification of Communication Manager 4.3.1 

https://www.autosar.org/nc/document-search/