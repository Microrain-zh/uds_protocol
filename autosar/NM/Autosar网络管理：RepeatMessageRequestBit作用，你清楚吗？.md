# Autosar网络管理：RepeatMessageRequestBit作用，你清楚吗？

如标题：RPB(Repeat Message Request Bit）干啥用的？此问题源于群内小伙伴的讨论，本文将该问题带来的思考分享给大家。

**提示**：基于Can总线讨论

1

RPB的作用

首先，确定一下RPB的位置，RPB在CBV字节的Bit0，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vwmD9AoLAY2fve0fdd7hlyViaLI4icbBVGS0icyjQJruMydAA1xN6d8nwQicGB4aQ0fmwGUriaDNqiahy5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

RPB的作用是什么呢？看一下Autosar的官方解释，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vwmD9AoLAY2fve0fdd7hlyVMQ51VacqxFdgdAmHzxrEGfXWF8dp6PwQplVLrB3Tibk47r9yF3gXEyw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

意思就是：RPB = 1，有RMS(Repeat Message State)请求，否则没有RMS请求。这里我们需要从收/发两个层面理解：

- **接收**：如果接收到的网络管理报文中，RPB = 1，请求当前的节点进入RMS状态。
- **发送**：如果本节点的**上层逻辑主动**请求进入RMS，则会主动调用接口CanNm_RepeatMessageRequest()，之后本节点外发的网络管理报文中RPB = 1。提示：RPB置位与否的操作需要静态配置**CANNM_NODE_DETECTION_ENABLED**参数。

CanNm_RepeatMessageRequest()接口声明如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vwmD9AoLAY2fve0fdd7hlyV9iaYOsHOYyIE6B0WHqDvtQrC9EDRib36ibGmypPVg6SefmcST7LCvZZibg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2

RPB的使用场景

这里我们假设一种工况：某个网段存在3个ECU：ECU1、ECU2、ECU3，且ECU3具有PN功能，ECU1对应的网络管理报文0x501，ECU2对应的网络管理报文0x502，ECU3对应的网络管理报文0x503。三个ECU在总线上的拓扑关系如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vwmD9AoLAY2fve0fdd7hlyVFwJBjqVR9lywYqnItkwZGO08dETdibLdicqbGsgVdwQlWic7gxBfCVyBA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

具体解释3个节点的网络状态切换时序：

**t0时刻**：ECU1和ECU2正常通信，两者均处于NOS(Normal Operation State)状态，发送的网络管理报文中，RPB未置位(RPB = 0)。ECU3处于BSM(Bus-Sleep Mode)状态（ECU3具有PN功能，因为收到的网络管理报文中，对应的PNC未置位，所以此时ECU3处于休眠状态）。

**t1时刻**：**ECU1主动调用**接口CanNm_RepeatMessageRequest()请求进入RMS(Repeat Message State)状态，此时：

1. ECU1进入RMS状态，ECU1发送的网络管理报文中，**PNI(Partial Network Information Bit)置位（PNI = 1）**，且**关联ECU3的****PNC_ECU3 = 1**，ECU3网络被唤醒；
2. 且RPB = 1，随即ECU2和ECU3进入RMS状态；
3. ECU2和ECU3发送的网络管理报文中，RPB = 1，且稍微晚于ECU1。

**t2时刻：**ECU1、ECU2、ECU3依次进入NOS状态，且三者的RPB = 0。

如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vwmD9AoLAY2fve0fdd7hlyVg6HB3Gcxc0JFC3WHn0ibVypib1NLFYXxibiaCtuviar5wGicMjRPtwqv7Xjg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**注意**：同一网段内的所有节点，对应的CANNM_MSG_CYCLE_TIME、CANNM_REPEAT_MESSAGE_TIME、CANNM_WAIT_BUS_SLEEP_TIME、NM-TIME_OUT时间参数需要保持一致，以便于网段内所有节点在**近似相等的时间**内进入相同的网络状态。

综上述：RPB具有协调不同ECU节点状态切换的作用，以便于网段内所有节点在近似相等的时间内进入相同的网络状态。

RPB是否还有其他使用场景？期待你不同的看法。

![图片](https://mmbiz.qpic.cn/mmbiz_png/quuOyCqONwdD16hI11wAQWxGp4ajJ4DMnbsHGb4ViandFryeibcQb1Idxb3MHmrh20988OSES3OU1wPTicQbAr94g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



参考资料

SIMPLE TITLE

AUTOSAR_SWS_CANNetworkManagement.pdf