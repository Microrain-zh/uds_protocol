# AUTOSAR 通信服务 - NM概念详解

**1.NM简介**

NM是NetWork Management的简称，是对具体总线网络管理的抽象管理，统管所有总线网络管理。在AUTOSAR BSW 层中，其上层是通信管理模块（ComM）,下层是具体总线网络管理模块（如Can网络管理CanNm，J1939Nm，FrNm，LinNm，UdpNm等）

 

**2.为什么需要网络管理呢？**

网络管理的目的是使ECU中的节点有序的睡眠和唤醒，也就是说休眠唤醒在本质上就是为了节约汽车的电量，特别是在车主下车后断掉高压电后，小电瓶失去高压动力电池充电后，车内所有ECU都需要依赖小电瓶供电，如果所有的ECU同时都在唤醒状态的情况下，那么小电瓶很快就会没电，所以需要将没有通信需求的ECU睡眠，在需要通信的时候才唤醒，可以节约汽车电池的电量。

 

**3.NM模块与其他模块联系**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczdQ18BeT4hLicgCn6584cCXyEAtDWAiakALnasVkLqA2H1bD2VD9eDZxEe3XFqsZwgvqlCzZfG6v8Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



BswM:NM可以通过BswM_Nm_CarWakeUpIndication()函数给BswM模块传递模式通知，在BswM中可以自定义模式规则以及模式控制。

EcuM:在EcuM模块初始化过程中，可以将Nm_Init初始化。

ComM, CanNm, J1939Nm, FrNm, LinNm, UdpNm :通用网络管理接口模块 (Nm) 为通信管理器 (ComM) 并使用总线特定网络管理模块的服务：

包括：

• CAN Network Management ([3, CanNm])

• FlexRay Network Management ([4, FrNm])

• LIN Network Management ([5, LinNm])

• Ethernet Network Management ([6, UdpNm]).

• J1939 Network Management ([7, J1939Nm])

 

因为NM和COMM模块关系最为紧密，因此特别说明下NM与COMM的关系。

| NM  variant | 是否能唤醒整车网络       | 下电方式                                                 |
| ----------- | ------------------------ | -------------------------------------------------------- |
| NONE        |                          | 不能通过ComM模块请求休眠，只能通过下电通信总线才能休眠。 |
| LIGHT       |                          | ComM经过请求后，经过一段时间后休眠                       |
| PASSIVE     | 控制器不允许唤醒总线网络 | 通过请求NM休眠                                           |
| FULL        | 控制器允许唤醒总线网络   | 通过请求NM休眠                                           |

 

 NM variant是在ComM模块中配置的，表示ComM模块对NM模块的控制方式。

1）当NM Variant为Passive或者FULL时，ComM的通道状态改变时，会去同步调用NM的函数请求对应通道的NM状态。

2）当NM Variant为FUll时，只有下电时总线通信才会休眠。

3）当NM Variant为Light时，ComM模块在接收到NO COMMUNICATION时，经过一个timeout时间后，进入休眠。

 

**4.NM在网络管理中的作用：**

1）网络管理模块作为具体总线管理网络管理模块的公共接口，统管CAN与以太网的网络管理，与ComM模块进行交互。

2）协同功能：网络管理模块具有集群协同功能，可以协同同一集群的多条通道进行休眠。



**4.1如何理解NM具有总线管理网络管理公共接口的作用呢？**

NM模块统管网络管理状态，因此应用层需要控制具体总线的网络管理的状态时，不需要直接调用具体总线网络管理模块的接口（CanNm，J1939Nm，FrNm，LinNm，UdpNm）,只需要调用NM模块提供的公共接口便可控制。在AUTOSAR架构中，NM模块对上将网络管理控制接口暴露给COMM模块进行调用从而控制通信网络管理状态；对下提供接口控制CanNm，J1939Nm，FrNm，LinNm，UdpNm。并且对于回调，Nm 向总线特定的网络管理模块提供通知回调，也就是说，NM模块将提供相应的回调函数给CAN,LIN等网络栈来获取响应网络状态，并调用 ComM 提供的通知回调。

 

**4.2.如何理解NM模块提供的协同功能呢？**

通过配置多条通道（比如can通道）属于一个集群中，网络管理NM模块提供协同一个集群通道所有通道的休眠。

在开启协同休眠场景下，只要集群中至少有一条通道仍然属于网络请求状态，协同功能就会让集群中所有的通道都处于工作状态，不会单独休眠。只有集群中所有网络都没有网络请求需求，且没有被请求的需求时，协同算法会协同集群中所有的通道释放网络，开始休眠。

 

**协同休眠的基本流程为：**

1.应用层在通道没有网络请求时，主动释放网络。这一步NM不会真正释放网络，而是进行记录。

2.通道在超时时间内没有收到网络请求，具体总线类型网络管理模块（CANNM）会上报NM.

3.NM根据协同算法规则，检查到协同休眠唤醒条件满足时，会主动释放网络。

4.具体总线类型网络管理模块上报具体总线网络状态。

5.NM根据协同算法，判断协同是否完成。同时会将网络状态上报上层管理模块。



**5.协同算法中的唤醒网络与关闭网络**

对于某个ECU节点，节点网络的唤醒与关闭不需要NM模块决定，而是依赖于COMM模块决定。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczdQ18BeT4hLicgCn6584cCXo4rqt8CA5004UXX3U4NSjXibN4TibvicvUrnlXcktMhxrnOXuvaicQ4diaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**5.1被动唤醒**

**被动唤醒**：来自总线上其他模块对该模块的网络请求。被动唤醒的节点，发送网络管理报文和应用报文的先后顺序无特别要求。

ECU上电或唤醒后，如果检测到为远程唤醒或其他添加需要ECU进行passive唤醒时，调用 ComM_EcuM_WakeUpIndication()（如果ECUM中的wakeup source绑定了ComM通道，则在调用EcuM_CheckWakeup()时自动调用），如果通道的NMVariant为FULL或PASSIVE，ComM调用 Nm_PassiveStartUp()请求NM进行passive唤醒，并调用 CanSM_RequestComMode()请求CanSM将相应的Can通道状态切换为FULLCOM。

 

**5.2 主动唤醒**

**主动唤醒**：来自模块内部对网络的请求，比如KL15唤醒。主动唤醒节点的网络管理报文必须先于应用报文发送。

 

ECU上电或唤醒后，如果检测到为本地唤醒或其他条件需要ECU进行主动唤醒时，用户调用ComM接口ComM_RequestComMode（）请求ComM COMM_FULL_COMMUNICATION以使能通信，ComM在接收到请求后，调用 CanSM_RequestComMode()请求CanSM将相应的Can通道状态切换为FULLCOM，CanSM再通过CanIf切换控制器和收发器状态，调用如果该通道的NMVariant为FULL，调用NM接口 Nm_NetworkRequest()，NM再调用CanNM接口 CanNm_NetworkRequest()请求进入主动唤醒。ComM进入COMM_FULL_COMMUNICATION后，可通过BSWM或手动方式，启动相应通道的Com IPdu Groups，通信开始

 

**5.3 网络休眠**

当某个网络通道需要休眠时，调用ComM接口ComM_RequestComMode（）请求COMM_NO_COMMUNICATION以释放通信请求，COMM在接收到请求后，调用 CanSM_RequestComMode()请求CanSM将相应的Can通道状态切换为NOCOM，如果该通道的NMVariant为FULL，调用NM接口Nm_NetworkRelease()请求NM进入sleep，NM在等待总线同步休眠后（其他节点都停发了网络管理报文准备休眠），进入Bus-Sleep状态，反馈给ComM，ComM进入NOCOM状态，如果BswM中配置了ComM模块状态为NO COMMUNICATION就执行ECUM下电动作时，此时ECUM就可以启动下电流程。



**6.API参考**

| Nm_Init                   | 初始化Nm模块                                                 |
| ------------------------- | ------------------------------------------------------------ |
| Nm_PassiveStartUp         | 被动模式唤醒网络                                             |
| Nm_NetworkRequest         | 主动模式唤醒网络                                             |
| Nm_NetworkRelease         | 释放网络                                                     |
| Nm_DisableCommunication   | 关闭Nm PDU传输功能                                           |
| Nm_EnableCommunication    | 打开Nm PDU传输功能                                           |
| Nm_SetUserData            | 设置下一个要发送的Nm  PDU中User Data数据                     |
| Nm_GetUserData            | 获取最近一次接收Nm  PDU中的User Data数据                     |
| Nm_RepeatMessageRequest   | 请求网络状态状态重新进入NM_STATE_REPEAT_MESSAGE              |
| Nm_GetState               | 获取当前网络状态信息                                         |
| Nm_NetworkStartIndication | 在睡眠模式下，如果具体总线类型网络管理模块收到Nm  PDU，会回调该接口通知Nm模块 |
| Nm_NetworkMode            | 具体总线类型网络模块进入Network  Mode ，会回调该接口通知Nm模块 |
| Nm_PrepareBusSleepMode    | 具体总线类型网络管理模块进入Prepare  Bus Sleep Mode，会回调该接口通知Nm模块 |
| Nm_BusSleepMode           | 具体总线类型网络模块进入Bus  Sleep Mode，会回调该接口通知Nm模块 |
| Nm_PduRxIndication        | 具体总线收到Nm PDU的报文，回调该接口通知Nm模块               |



参考文档：Specification of CAN Network Management 4.3.1 

https://www.autosar.org/nc/document-search/