# Autosar开发：CanSM模块如何处理Busoff？

CAN总线中，相对其他通信类问题，Busoff问题比较难搞。本文从CanSM模块出发，就Busoff产生、Busoff信息交互、Busoff快/慢恢复等问题展开聊一聊。

这个话题前面有聊过，可以参考前文[Autosar网络管理：说说Busoff那点事](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247485790&idx=1&sn=edb25567e52d9f4c3f31788033d90b1e&chksm=fa2a572acd5dde3ca104329bda62db64b0b946b3332fe59e137598caf0829a7520479f075a8f&scene=21#wechat_redirect)。

1

Busoff产生

这里再说一次Busoff产生的条件：**TEC > 255**。也就是说ECU自身发出的报文错误，导致TEC(Transmit Error Counter)不断累加，直到TEC超过255产生Busoff，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vxYWsIOI8qsGYGFJhIM0SRnrOunxs1vsVHtLOZkWIhzT1ZBicnFG4SWxbCQJt5Bw3od4PYO707vrzw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**举例**：ECU1::CAN1发送的错误帧只能使ECU1::CAN1进入Busoff状态，而不能使ECU2::CAN1进入Busoff，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vxYWsIOI8qsGYGFJhIM0SRnp3N1icZJlrME9dUiatBdSOP5Eib3AvlvKcJ9lJaMFicSJBW9YnSricLSd8g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

因为错误由ECU1::CAN1自己产生，ECU1::CAN1有问题，自己脱离CAN总线即可，不要影响ECU2::CAN1继续使用CAN总线。

2

Busoff信息交互及Busoff恢复机制

节点产生Busoff以后，ControllerMode状态自动切换到CANIF_CS_STOPED模式，停止发送错误帧，避免影响总线其他节点的通信。既然Busoff已经发生，对应的信息就需要传递给上层，让上层决策后续的通信行为。怎样通知上层呢？

Can Controller通知到上层有两种方式：**Interrupt**或者**Polling**。

## Step1、Busoff事件信息如何通知到CanSM

**Interrupt方式：**Busoff中断发生->**CanInterruptStatus()**->CanHL_ErrorHandling()->

CanIf_ControllerBusOff()->CanSM_ControllerBusOff()->CanSM_BusOffIndicated()，CanSM_BusOffFlag = TRUE ...CanSM_MainFunction()周期性检查CanSM_BusOffFlag置位情况。

**Polling方式：****Can_MainFunction_BusOff()**->CanHL_ErrorHandling()->

CanIf_ControllerBusOff()->CanSM_ControllerBusOff()->CanSM_BusOffIndicated()，CanSM_BusOffFlag = TRUE ...CanSM_MainFunction()周期性检查CanSM_BusOffFlag置位情况。

**提示：**上述函数关联关系，除Autosar标准接口以外，其他接口，不同软件供应商，实现上可能存在不同。

## Step2、CanSM请求重启Can Controller，通知ComM、BswM模式切换

Busoff发生以后，CanSM调用CanIf_SetControllerMode()接口，请求将ControllerMode切到CANIF_CS_STARTED模式，以便于后续尝试恢复通信。同时关闭Tx PDU的发送，只能接收Rx PDU。所以这也是为什么在恢复期内可以收到报文的原因。CanSM调用BswM_CanSM_CurrentState()接口通知BswM进入CANSM_BSWM_BUS_OFF状态，调用ComM_BusSM_ModeIndication()接口通知ComM进入COMM_SILENT_COMMUNICATION状态。

Busoff发生以后，CanSM先告知ComM，ComM在请求CanSM对应Channel由FULL COMMUNICATION进入SILENT COMMUNICATION。进入CANSM_BSM_S_SILENTCOM_BOR状态，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyxvZkHlWF3Z4AysMQuAPiafpiasgcp0tXwmMqaerTUPGkpqricEbAkx80icE7O45OJjFsTXibKTK9icH0A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



Busoff发生以后，CanSM会启动一个Busoff Timer，Busoff Timer分为两种：

- **快恢复**时间参数：**CanSMBorTimeL1；**
- **慢恢复**时间参数：**CanSMBorTimeL2**。

具体Busoff Timer应该等于CanSMBorTimeL1还是CanSMBorTimeL2，取决于配置参数**CanSMBorCounterL1ToL2**。

- 如果Busoff连续发生次数 ＜ CanSMBorCounterL1ToL2，Busoff Timer = CanSMBorTimeL1；
- 如果Busoff连续发生次数≥ CanSMBorCounterL1ToL2，Busoff Timer = CanSMBorTimeL2；

**注意**：CanSMBorTimeL1、CanSMBorTimeL2、CanSMBorCounterL1ToL2三个参数均在CanSM模块配置，具体数值根据OEM需求配置。

测试中，busoff的快/慢恢复行为如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyxvZkHlWF3Z4AysMQuAPiafWJBuPaoQ58EVELczesibnhXoOlPuqsKtwAh1BxOFKeiaKppfcqmJfCibw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在快/慢恢复时间内，可以接收报文。

## Step3、CanSMBorTimeL1或者CanSMBorTimeL2耗尽

CanSMBorTimeL1或者CanSMBorTimeL2耗尽(elapse)，重新发送Tx PDU，让故障节点再次尝试向CAN总线发送报文。同时，CanSM通知BswM进入CANSM_BSWM_FULL_COMMUNICATION状态，通知ComM进入COMM_FULL_COMMUNICATION状态。可以启动CanSMBorTimeTxEnsured，确认Busoff是否恢复，也可以使用Confirm方式确认Busoff恢复。

## Step4、CanSMBorTimeTxEnsured耗尽

在CanSMBorTimeTxEnsured时间内，Busoff再次发生，则进行下一次的Busoff恢复机制，如果CanSMBorTimeTxEnsured耗尽，则说明成功从Busoff状态恢复。如果在CanSMBorTimeTxEnsured时间内，再次发生Busoff，则Busoff次数累加。

3

Busoff发生时的网络状态

这里主要讨论Busoff进入慢恢复期，节点在NOS(Normal Operation State)和RSS(Ready Sleep State)下是否会进行网络状态切换。

**NOS**：Busoff进入慢恢复期，如果**上层不主动请求释放网络**，网络状态无法进入RSS，所以，节点会一直在NOS状态下，一直处于慢恢复状态，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vw1H2htERegqYaeZuLibKd8g72J2PjfH9kVlYJdG9QLQjgaAaP9e8vsn21Rdv1Db8PzZCfqOlicrWeQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**RSS**：Busoff进入慢恢复期，如果在恢复期收不到有效的网络管理报文，NM-Timeout时间超时以后，进入PBSM(Pre Bus Sleep Mode)；如果可以收到有效的网络管理报文，则网络处于RSS状态，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vw1H2htERegqYaeZuLibKd8g71zGosKDy0lgkGdfRSzcIpTZUn5ib8feWw9Bnwn8ibv8R0lU0mNykZUw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如果节点在NOS状态下，一直处于慢恢复，会带来什么问题呢？节点一直在慢恢复期，意味着该节点不会外报文(应用报文和网络管理报文均不会外发)，其他节点会上报对应的节点丢失故障。

![图片](https://mmbiz.qpic.cn/mmbiz_png/quuOyCqONwdD16hI11wAQWxGp4ajJ4DMnbsHGb4ViandFryeibcQb1Idxb3MHmrh20988OSES3OU1wPTicQbAr94g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



参考资料

SIMPLE TITLE

AUTOSAR_SWS_CANStateManager.pdf



