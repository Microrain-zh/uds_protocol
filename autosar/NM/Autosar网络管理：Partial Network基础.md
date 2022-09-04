# Autosar网络管理：Partial Network基础

学习Autosar网络管理，有很多的细枝末节需要理解，而这些细枝末节的东西需要时间琢磨。对于新进的学习者，不知道是否有过这样的感受：“这个概念查不到，这个东西在哪些场景下使用......”，这是我曾经的感受。说实话，理解不了一些概念很难受，概念不清就不能很好地理解对应模块功能，解决实际项目问题自然也就摸不着头脑。本文分享一点Autosar网络功能PN（Partial Network）部分的概念，希望能给大家一些帮助！我是一个Autosar/嵌入式学徒，不明白的很多，欢迎大家指正和交流。

**提示**：基于CAN总线讨论

1

Partial Network基础概念理解

## 1、NM PDU结构及PNC信息位置

一个经典CAN NM PDU格式如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vz3NkK5cUvMvtY3ultKibldumKa0QKib18b1thHnBHvWQfbv3JRQSgvufeb0WyxcJFNcciciaFt1Z9pgQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**NID**：Source Node Identifier，即对应节点CAN报文的CanID，比如节点NM Msg CanID为0x509，则NID = 0x09（配置工具配置），5用于表示网络管理报文类型。

**CBV**：Control Bit Vector，控制位向量。具体每个Bit信息如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vz3NkK5cUvMvtY3ultKiblduicXB5Gk95swZbB27XIFCeq4hnrfLqHfu2u6Rjph96UapfvdVkaNpu0w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

- **bit0**：节点进入RMS状态标志位，如果节点由RSS状态或者NOS状态主动进入RMS状态，需要调用CanNm_RepeatMessageRequest()接口，节点由RSS或者NOS状态进入RMS状态时，发送的NM Msg中，bit0 = 1，离开RMS状态时，bit0 = 0。这个前文有聊过，详情可以参考前文[勘误篇(一)：Autosar网络管理：RepeatMessageRequestBit作用，你清楚吗？](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247487447&idx=1&sn=0fa6ba9f73244ef1a8ab4ea4a17b6923&chksm=fa2a51a3cd5dd8b54a0ab073a13b9eb12d7dcc9d80587f59b8ce482fca278d66f8e80625ecfc&scene=21#wechat_redirect)；
- **bit4：**如果节点主动唤醒网络，在RMS以及NOS状态下，bit4 = 1。需要配置参数CanNmActiveWakeupBitEnabled = TRUE；
- **bit6：**网络管理使用PN功能时，发送网络管理报文时，bit6 = 1；接收到网络管理报文时，需要识别PNI，如果PNI = 0，则PN功能不处理。

**PNC**：Partial Network Cluster ，部分网络簇。User Data存储PNC信息，假设某网段内有48个PNC，User Data中的每一个Bit代表一个PNC，可以用PNC 16 ~ PNC 63标识这48个PNC。注意：节点具体需要处理多少个PNC，根据需求配置。比如：网段内有20个PNC（PNC16~PNC35），使用User Data的3个bytes即可（3 * 8 ＞ 20）标识，PNC与User Data对应关系如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vx7Mia1nJrIpFnFziaLYgQsviaBicWXOMHfk5N8y3usSJ7uvdRZ9a55zl7GYWXicNa458gsbhnb7wcEJTg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 2、如何理解VFC？

**VFC**：Virtual Function Cluster ，虚拟功能簇。SWC之间通过Port交互信息，实现一个或者多个车辆功能。需要使能某个功能时，通过特定的SWC（VFC-Controller）置位VFC #n（ = 1）请求对应的通信功能，使得不同SWC之间可以交互信息，对应功能开启；如果不需要某个功能时，复位对应的VFC #n( = 0 )请求关闭通信，功能禁止。

VFC #n状态信息可以通过RTE Port传递给BSW。

VFC与PNC的映射关系示例，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vx7Mia1nJrIpFnFziaLYgQsviam1VficMwjiaVLULcT0W8hK9WNSb3SePibmbmpI5Mhnq8dw6pKZlUWQ3xw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

- 一个VFC可以映射一个PNC；
- 多个VFC可以映射一个PNC；
- 一个VFC可以映射多个PNC；
- 一个VFC也可以不映射PNC。

## 3、如何理解节点关联PNC？

一个网段内有多个节点，多个节点之间可以形成多个PNC，也就意味着：具体到某个节点，并不是所有的PNC #n都对该节点有效。比如：ECU1有两个节点Node1和Node2，Node1收到的网络管理报文中，只关注PNC20、PNC22、PNC30是否置位（ = 1），因为Node1只参与PNC 20、PNC22、PNC30三个网络簇，并不关心PNC16或者其他的PNC #n状态，示例如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vx7Mia1nJrIpFnFziaLYgQsviaqqtc3C6Enyfx0IibQz9Hmia6of3dNfBkJcAzZicqYqcrnSvWDmNEtAz9w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 4、CanNmPnResetTime时间参数

如何理解CanNmMsgCycleTime＜**CanNmPnResetTime** ＜CanNmTimeoutTime ？

假设：某节点关联PNC1，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vz3NkK5cUvMvtY3ultKiblduWh2jAFib44kW6diaHQoRd99UOZxAB8xxNjTsGX8KzPibib1h57YNsDsM9g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**一、为什么CanNmMsgCycleTi****me＜CanNmPnResetTime ？**

如果节点的NM Msg发送周期CanNmMsgCycleTime = 1000ms，CanNmPnResetTime  = 800ms。

**t0时刻**，PNC1置位（PNC1 = 1），即：期望本节点，下一帧NM PDU中的PNC1 = 1，关联的Tx I-PDU继续发送；

**t1时刻**，由于CanNmPnResetTime = 0，导致PNC1复位（PNC1 = 0），关联的Tx I-PDU停发;

**t2时刻**，发送NM PDU中的PNC1 = 0，而不是预期的PNC1 = 1，所以CanNmMsgCycleTime需要小于CanNmPnResetTime。如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vz3NkK5cUvMvtY3ultKibldu2EC6Yb3vibyxamibSw1BnEhuMU511ut3QvCRb2RpvLnnuy5NEdu8FFbw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**二、为什么CanNmPnResetTime ＜CanNmTimeoutTime ？**

如果节点的NM Msg发送周期CanNmTimeoutTime= 3000ms，CanNmPnResetTime  = 3200ms。

**t0时刻**，由NOS进入RSS之前，最后一帧NM PDU中的PNC1置位（PNC1 = 1），在RSS中停发NM PDU；

**t1时刻**，由于CanNmPnResetTime != 0，PNC1 = 1，CanNmTimeoutTime = 0，节点由RSS进入PBSM状态；

**t2时刻**，CanNmPnResetTime = 0，PNC1 = 0。但是，在t1~t2时间内，由于PNC1 = 1，关联PNC1的Tx I-PDU继续外发，进入PBSM以后，除了底层硬件缓存的应用报文可以外发，上层请求的应用报文(Tx I-PDU)不应发送。所以CanNmPnResetTime 需要小于CanNmTimeoutTime 。如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vz3NkK5cUvMvtY3ultKiblduIgmOD7QNfUVeoEJCxQX5EKHWjQSKPtvX18PjL3xxmQ1Aam3tle1QaQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 5、ERA和EIRA是什么？

当网络管理使用PN功能时，ERA(External Request Array)和EIRA(External Internal Request Array)是绕不开的两个配置项。

**ERA：**外部网络请求集，什么意思呢？

1. 节点收到其他节点的网络管理报文就是“External”；
2. 其他节点发送的网络管理报文，如果PNC #n = 1，且与接收节点关联，就表示“Request”，请求接收节点保持PNC的网络状态；
3. 一个节点可以关联多个PNC，所以会形成“Array”。

ERA的使用，需要配置参数CanNmPnEraCalcEnabled = 1。

**EIRA：**内部/外部网络请求集，外部网络请求和ERA一致，不再赘述。需要配置参数CanNmPnEiraCalcEnabled = TRUE。

节点的上层（ASWC），使用VFC主动请求本节点保持网络的行为就是“Internal”，一个节点有多个VFC，也可以关联多个PNC，VFC置位（ = 1）时，对应的PNC置位（ = 1），主动请求激活/保持该PNC的网络状态。

比如：SWC 01识别到某个VFC = 1，通过RTE与BswM交互，之后BswM调用ComM的接口请求通信模式切换；SWC 02也可以识别VFC状态，直接通过RTE，调用ComM接口，请求通信模式，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyBD5FAibpXIF6mMca67EWHnVicu9x1fprgZ1uH9IF0ANSn0wUuQwY3qkibCjxMRrZ3zf6pbb9pFdwpw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

ERA和路由（Gateway）相关，本文不展开讨论，后续会讨论，敬请关注。

## 6、如何理解网络管理报文的"快/慢"发送

对于主动(Active)网络节点，网络管理的需求中可能会有这样的需求："支持快发模式（Immediate），进入RMS（Repeat Message State）状态，需要连续发送NM Msg 10帧，NM Msg间隔20ms，快发模式之后，进入正常发送模式，NM Msg发送间隔1000ms。”

何时快发，何时慢发呢？如果**节点网络****被动唤醒**，比如：节点接收到其他节点的有效网络管理报文，调用CanNm_PassiveStartUp()接口唤醒节点网络，节点的NM Msg正常发送（假设周期1000ms），这就是慢发送；如果**节点网络主动唤醒**，节点主动请求网络，比如：调用CanNm_NetworkRequest()接口唤醒网络，进入RMS以后，进入快发模式（注意：CanNmImmediateNmTransmissions＞0），假设快发周期20ms，快发10帧，之后进入慢发（假设周期1000ms）。

具体的NM Msg发送行为可以参考前文[Autosar网络管理：网络管理报文的收/发与网络管理时间配置参数解析](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247488000&idx=1&sn=fae533f0404e3c5cf4d570e9d625ba6d&chksm=fa2a4c74cd5dc562ba7be2c1e59dab384aaaf00efd597ed2d724c0ed56fc5c3a06447299ae1d&scene=21#wechat_redirect)。