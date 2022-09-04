# Autosar网络管理：网络问题QA

最近和一些读者讨论了一些Autosar网络管理相关问题，有几个问题做了一下梳理，再此和大家分享一下。

## Q1：CanNmImmediateRestartEnabled使能，NM PDU的外发行为？

**A1**：先说CanNmImmediateRestartEnabled的作用，在PBSM(Prepare-Bus-Sleep mode)模式下，收到总线的通信请求(比如：KL15硬线唤醒)，使能NM-PDU的发送，即：进入RMS状态。NM-PDU的发送行为如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzkqL2nJHwQmInOymXvLRy6F1PAEmh66DP9BQHly6C2vyLcialLjHboI4gbSxjr2lThXays8KjFfow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图可以看出，进入RMS模式以后，以正常的发送周期发送NM-PDU。

对于此问题，Autosar要求：

- “CanNmImmediateRestartEnabled = true then CanNmImmediateNmTransmissions = 0”，意思是说，使能CanNmImmediateRestartEnabled，就没有快发模式；
- “CanNmPnHandleMultipleNetworkRequests == True" then "CanNmImmediateNmTransmissions > 0”，意思是说，使能PN功能以后，需要快发模式。

可以看出，上述两点在实现时，只能使能其中一个。

## Q2：NM-Timeout何时重置？

**A2**：不讨论PN功能时。当节点收到/发送NM-PDU时，NM-Timeout重置，假设，某CAN网段内存在三个节点：ECU1、ECU2、ECU3，各个节点的NM-Timeout重置时机和网络状态如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzkqL2nJHwQmInOymXvLRy6FMugqnO7DnP7OZSYlqaLic1CQRBT8whqQBU1Iag3ukLhQMiaKZcjRZUA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

提示：NO（Normal Operation）、RS（Ready Sleep）、PBSM（Prepare-Bus-Sleep mode）。黑色实心表示发送NM-PDU，黑色空心表示接收NM-PDU。

上图可以看出：每个节点释放网络的时机不确定，每个节点的网络释放时机，取决于节点的上层实现。注意，不是进入RS状态重置NM-Timeout。节点释放网络的时候，NM-Timeout继续递减，不会重置。当网段内没有节点发送网络管理报文，且NM-Timeout走完，所有节点一起进入PBSM模式。

## Q3：如何理解CAN通信的串行通信？****

**A3**：串行通信，就是用一个Pin发送/接收Bit位流信息。如下图：对于发送节点(uC1)，通过Controller的Tx Pin发送Bit位流信息，即：uC1发送报文。同时，uC1通过Controller的Rx Pin，回读自身发送的信息是否正确。对于同网段内的其他节点(uC2)，通过Rx Pin接收uC1的发送信息，即：接收报文。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyczfibHN9PKEcPMz0JOY9fX2XASJk5BDiaNlPb8ZpAguPT88ib2YxFoC6GxYfWY38l5ealZEebLnkqA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)