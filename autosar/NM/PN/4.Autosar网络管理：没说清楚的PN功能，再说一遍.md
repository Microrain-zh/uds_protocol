# Autosar网络管理：没说清楚的PN功能，再说一遍

前面就Autosar的PN（Partial Networking）功能聊过，可以回顾[Autosar网络管理：从CanNM模块看Partial Networking](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247485683&idx=1&sn=47c6527a7b6dabb3d60a3ca5f17d2c3b&chksm=fa2a5687cd5ddf91c55c850897f28e09c1697acdcd912009c1ef97787993f8bb2058a48a7b8f&scene=21#wechat_redirect)和[Autosar网络管理：从ComM模块看Partial Networking](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247485690&idx=1&sn=41fbe614aa3b2b5d3961e291b86db03c&chksm=fa2a568ecd5ddf988f4039202a519a311c52ae19492db4121d405219661a6fde6ffedb57e3bf&scene=21#wechat_redirect)，由于篇幅限制，还是有些东西没有聊透，本文接着聊，把自己对PN的理解，再给大家分享一下。

1

PN功能中，CanNm、COM、ComM的角色

想弄清楚PN功能，要先清楚CanNm、COM、ComM三者之间的关系，Autosar CanNm模块有一段描述将三者的关系表达得很清晰，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vwjrSOBYkNibQFrz7KLGpamDBC8v3yvTmCibnYPF5tHIIW7dWYaAxYgksNoZBaA5mcS4VaEica287lcQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

CanNm负责提供PN请求信息，也就是将User Data部分的PNC信息缓存；COM模块负责将CanNm中缓存的PNC信息传递给ComM；ComM通过COM模块的标准接口将PNC信息进行拆解，并执行PNC状态机的管理，PNC状态机的管控最终会作用到COM的IPDU-Groups，而COM IPDU-Groups的使能/禁用，由BswM的仲裁结果决定，仲裁信息来自ComM对PNC Mode的切换。

## PNC与物理通道的关系

进一步的讨论PN功能之前，我们先做一个需求假设，如：VCU包含两路CAN（CAN1、CAN2），有8个PNC（PNC1~PNC8），每个PNC与CAN1、CAN2的关联关系如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vwjrSOBYkNibQFrz7KLGpamDDibfNmc28m9UmtXEG7pqWlva0qQPU2tVIdNwujspsahqOiaKJFqT7GEQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

举例说明：

- PNC1与CAN1、CAN2均关联，CAN1收到PNC1 Bit = 1时，CAN1网络需要唤醒，且将此信息路由给CAN2，CAN2的网络需要唤醒；
- PNC3与CAN1、CAN2均不相关，收到的PNC3 Bit = 0/1，CAN1网络不会唤醒，也不会路由给CAN2；
- PNC5与CAN1相关，与CAN2不相关，收到的PNC5 bit = 1，激活CAN1网络，CAN2网络不激活。

**注意**：假设CAN1和CAN2都是默认的ACTIVE网关类型。

2

如何理解PDU与PNC关系

实际的开发中，我们会有这样的表述“PNC1关联Pdu01，或者PNC1不关联Pdu02”，这什么意思呢？我们以实际的例子分析，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vwjrSOBYkNibQFrz7KLGpamDEeTVXtDastCROnG8VGzvysUS73QFDgCkoUt5UdWOVZGuf6B9GJq6bA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

假设当前VCU的CAN1发送23帧应用报文。发送的应用报文分为两类，一类与PNC关联，一类与PNC不关联，上图中，PNC = NONE表示此应用报文与PNC不关联。与PNC关联的应用报文会根据PNC ID划分不同的Pdu Group。这样做的目的是什么呢？主要是为了便于管理应用报文，这里分两种情况讨论：

## 1、非PNC关联的应用报文

节点收到网络管理报文，当某些应用报文与PNC不相关时，如：Tx_PduGroup09对应的应用报文。这些应用报文的发送状态与CanNm模块所处的状态机有关。即：收到的网络管理报文，所有PNC bit = 0，网络处于NM(Network Mode)，该节点对应的外发应用报文（与PNC不相关），需正常外发。由于接收到的PNC1 bit = 0，Tx_PduGroup01对应的应用报文不会外发，同理，Tx_PduGroup02~Tx_PduGroup08对应的应用报文不会外发。

## 2、PNC关联的应用报文

节点收到网络管理报文，当某些应用报文与PNC相关时，如：Tx_PduGroup01对应的应用报文均与Tx_PNC1关联。这些应用报文的发送状态与ComM模块控制的PN状态机有关。即：收到的网络管理报文，Tx_PNC1 bit = 1，Tx_PduGroup01/Tx_PduGroup09对应的应用报文需要外发，如果Tx_PNC1 bit = 0 AND Tx_PNC1_Timeout，则停发Tx_PduGroup01/Tx_PduGroup09对应的应用报文。

这里可以这样理解：每个PNC对应一组PDU(PduGroup，至少一个PDU)，每一个PNC对应一个PN状态机，每个PNC状态机只管控关联的PduGroup行为。

3

ERA/ERIA

EIRA(External Internal Request Array)和ERA(External Request Array)从开发的角度看是两个数组，从概念的角度理解是两个状态集。怎么理解这两个概念呢？我们从PNC信息交互过程说起。

## 1、PNC信息交互过程

还是以开始的VCU(包含CAN1和CAN2)为例。当CAN1收到总线上的网络管理报文，CanIf会通知到CanNm，CanNm获取Data以后，会将User Data部分的信息缓存到CAN1对应的ERA1和EIRA1缓存区。我们知道，User Data部分存储着PNC信息，且一个PNC对应一个Bit。同理，CAN2收到总线上的网络管理报文以后，也会将User Data部分的信息缓存到CAN2对应的ERA2和EIRA2缓存区。而在Autosar的规范中，EIRA是可以共用的，因此在实现的角度，EIRA1和EIRA2可以共用一个缓存区，即：EIRA缓存区。但是ERA1和ERA2不能共用，每个物理通道均独立开辟ERA缓存区。

CanNm既然存储了ERA和EIRA信息，那如何传递给COM呢？通过PduR，CanNm通过PduR_CanNmRxIndication()接口通知PduR，之后PduR通过Com_RxIndication()将信息转交给COM，此时CanNm相当于一个Low模块，COM相当于Up模块，调用的接口如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzmWPQweq3T5dB1AGBuCz54errFmDbMKqaBAlHf7wn9JSsadkeXFsiaMn8Tz8g6dp5viaAx1ByGicGicQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vxB97M4gWvxVCIj8k3KveYWOY5yDwmmRrVKvhKqIHlfsa34Tj5QavVDpqxL6fSxEHxlC029TZAMbg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

CanNm向上传递数据的时候，Node ID和CBV两个字节对上层（COM）来说没有意义，只传输User Data部分数据即可。

**注意**：此接口属于一个配置接口。

COM将所有接收的信号放在一个大的缓存区中，通过ByteIdx和BitIdx查找对应Signal位置，同样，也会通过对应的ByteIdx和BitIdx缓存接收到的ERA Signal和EIRA Signal信息。

这里需要补充一些信息：每个物理通道，比如CAN1，在COM层会对应两个接收信号，即：CAN1_Rx_ERA_Signal和CAN1_Rx_EIRA_Signal，一个发送信号，即：CAN1_Tx_EIRA_Signal。对于多路物理通道，如本例：VCU包含CAN1和CAN2，CAN1和CAN2会共用一个EIRA Signal，即CAN2_Rx_EIRA_Signal和CAN1_Rx_EIRA_Signal复用一个信号，即CAN_Rx_EIRA_Signal，这与CanNm缓存方式对应。CAN1_Rx_ERA_Signal和CAN2_Rx_ERA_Signal是不同的接收信号，不能复用。对于发送信号，CAN1、CAN2均有独立的发送信号CAN1_Tx_EIRA_Signal、CAN2_Tx_EIRA_Signal。

所以，对于多物理通道，即：CAN1和CAN2，在COM层需要配置3个接收信号：CAN1_Rx_ERA_Signal、CAN2_Rx_ERA_Signal和CAN_Rx_EIRA_Signal；在COM层需要配置2个发送信号：CAN1_Tx_EIRA_Signal、CAN2_Tx_EIRA_Signal。这些信号主要为ComM所用，实现路由（Gateway）功能。

当ERA或者EIRA信息在COM更新以后，COM会通过回调函数ComM_COMCbk_<sn>()通知到ComM，ComM在下一个轮询中处理PN状态，ComM_COMCbk_<sn>()原型如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzmWPQweq3T5dB1AGBuCz54Ot5WMdI1VyUw3Tw3dq2QiatFGdsEzQjm7gojPFSEeEQFEdpGHuGOlOA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

ComM处理每个PNC之前，需要获取每个PNC的信息，怎么获取的呢？ComM通过调用Com_ReceiveSignal()接口来读取ERA Signal或者EIRA Signal，即可识别到每个PNC的信息，进而进行每个PNC状态机的管理。这就是前面说的，每个物理通道需要有2个接收信号的原因。

刚才提到，ComM的Main函数会周期性的处理每个PNC的状态，这里需要注意一下，每个PNC最终切换到什么状态受到User、ERA和ERIA的约束，且有先后顺序，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzmWPQweq3T5dB1AGBuCz54LZqVrDRibeaCHaNkW41zas6NFQprMF5dwjvNhbhLjN8j4MUqoCPll9w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

什么意思呢？在ComM，对每个物理通道或者PNC的通信请求有最高优先级覆盖原则，比如：user请求了FULL Communication，在同一个轮询里又请求了No Communication，则本轮询的结果是FULL Communication，即FULL Communication请求优先级最高。

当ComM的配置参数ComMPncGatewayEnabled = TRUE，即Gateway功能打开时，收到的PNC请求信息不仅要映射回本物理通道，还需要映射到其他与此PNC关联的物理通道。

举例：

CAN1的网关类型为COMM_GATEWAY_TYPE_ACTIVE，PNC1关联CAN1和CAN2。CAN1收到网络管理报文的PNC1 bit = 1，ComM调用Com_SendSignal()接口，不仅将PNC bit = 1信息发送给CAN1，还会发送给CAN2，即两次调用Com_SendSignal()接口，对应的句柄分别是CAN1_Tx_EIRA_Signal、CAN2_Tx_EIRA_Signal。这就是前面一直说的"路由"。

至此，我们将PNC数据的流转过程说完了，回过头再看PNC的状态机原理图是不是有点眉目了？PNC状态机关系如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzmWPQweq3T5dB1AGBuCz54flISFcnrO4WuawnC9t2VTDEC6rJ3pqCf07KuUXl2nribZhOlG7b31hg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**提示**：

- 信号级别的路由在COM，PDU级别的路由在PduR。ERA和EIRA在CanNm、PduR、COM都可以看作是PDU。同时，在COM，ERA和EIRA还可以看作Signal，即接收到ERA PDU和EIRA PDU不进行拆分，当作ERA Signal和EIRA Signal处理；
- PNC状态控制的目的是控制关联PDU的收/发行为；
- 每次PNC状态切换时，均会通过BswM_ComM_CurrentPncMode()接口通知BswM，以便BswM进行模式仲裁，根据仲裁结果决定是否使能/禁用对应PNC的PDU行为。

## 2、Timer与物理通道的关系

在阅读Autosar CanNm模块的时候，大家是否对Timer与物理通道、PNC关系产生过困惑？老实说，我开始没有太理解。

对于EIRA，一个PNC关联多个物理通道时，一个Timer即可，说明如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzmWPQweq3T5dB1AGBuCz54Y6uQ6uHwDiabhicFKP4eyHJDHKEGqBlPic44jOibCGJNvuJPmkKe6sDEnA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

对于ERA，一个PNC关联多个物理通道时，每个通道对应的PNC均需要一个Timer，说明如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzmWPQweq3T5dB1AGBuCz54oImvhczM9XWy3oXDeh8HqGDHvF9Z5WfzwIeY0McU956iatuQB4hGibXg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

也就是说，对于多通道共用的EIRA来说，共用Timer可以，有多少个PNC就有多少个EIRA_Timer，与物理通道数量没有关系。对于ERA来说，外部请求的信息会来自不同的物理通道，CAN1/CAN2接收到网络管理报文的时机、频率均可能不同，此时对于同一个PNC（比如本文的PNC1，均关联CAN1和CAN2），使用一个Timer显然不合适，因此不同的物理通道，对于关联的PNC需要使用不同的Timer。

这里提到的Timer，就是开发中需要配置的CanNmPnResetTime，如果网络管理报文的超时间是（CanNmTimeoutTime）3s，一般会要求CanNmPnResetTime<CanNmTimeoutTime，比如CanNmPnResetTime = 2.95s，同时CanNmPnResetTime > CanNmMsgCycleTime(如：1s)。

当CanNmPnResetTime  = 0时，对应的PNC bit = 0。

4

拓展

（1）项目开发中需要注意CanNm模块的主函数CanNm_MainFunction()所在Task周期不要超过节点的快发周期。为什么呢？使用PN功能时，每个PNC会对应一个CanNmPnResetTime，而PNC关联的外发PDU行为与CanNmPnResetTime有关，CanNmPnResetTime = 0时，PNC关联的外发PDU停发。而CanNmPnResetTime的复位（重新赋值，如：CanNmPnResetTime = 2.95s）与收/发网网络管理报文的时机有关；

**Case1**：假设ECU A的CanNm_MainFunction()周期20ms，ECU B的网络管理报文快发周期10ms，ECU A和ECU B在同一个网段（均只有一路CAN），且PNC_AB关联ECU A和ECU B，当ECU A收到网络管理报文PNC_AB = 1时，ECU A对应CanNmPnResetTime_A复位，即CanNmPnResetTime_A= 2.95，需要ECU A的PN激活。假设：ECU B在网络的快发模式，以10ms周期发送网络管理报文，且PNC_AB = 0，则ECU A的PNC_AB被覆盖，即ECU A收到的PNC_AB = 0，CanNmPnResetTime_A= 0，导致ECU A的PN网络未激活（实际需要激活），与PNC_AB关联的外发PDU不能外发。因此，开发中需要注意此细节。

如何避免此问题的发生？确保CanNm_MainFunction()周期小于网络管理报文快发周期，比如：CanNm_MainFunction() = 5ms，网络管理报文的快发周期 = 20ms。

（2）与PNC关联的PDU可以是周期型应用报文，也可以是事件型应用报文。

（3）当某个PDU关联到某个PDU group，并且此PDU group的控制条件为某个PNC，则包含此PDU的发送报文被认为关联到此PNC，即当接收或者发送此PNC时，该网段应该发送此报文。

（4）当某个PDU没有关联到任何PDU group，则包含此PDU的发送报文应该在任意PNC下都能发送。

**参考资料**

AUTOSAR_SWS_CANNetworkManagement.pdf

AUTOSAR_SWS_PDURouter.pdf

AUTOSAR_SWS_COMManager.pdf