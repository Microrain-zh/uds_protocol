# Autosar网络管理：ERA/EIRA问题QA

**Q1：****ERA****、EIRA谁针对****网关节点？**

**A1**：Autosar网络管理中，使能PN(Partial Network)功能以后，会有ERA和EIRA配置项。两者有什么区别呢？搞清楚两者的区别，需要先清楚开发的节点（ECU）是否是网关（Gateway）节点。

- **对于网关节点**，则会涉及到ERA的配置，为什么这样说呢？充当网关节点的ECU，意味着此ECU包含多个物理通道，eg:2路CAN、1路Flexray等。当网关节点的某一路（eg:CAN1）收到PNC #n和其他路关联时（eg:CAN2），网关节点需要承担主动唤醒CAN2的责任，因此需要PNC信息路由，此时需要ERA将CAN1收到的PNC #n信息给到CAN2。更多细节可以参考前文[Autosar网络管理：主动唤醒源/被动唤醒源与网络主动唤醒/被动唤醒的关系](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247489138&idx=1&sn=6975890311fad819099b92c96df00c77&chksm=fa2a4806cd5dc11017a8c3bd833a596005101c028d55467134baaff978653e7bb31c12493956&scene=21#wechat_redirect)。
- **对于非网关节点**，没有路由PNC信息的任务，使能EIRA功能即可。

## Q2：对于ERA，为什么6个通道8个PN，需要48 个计时器？

**A2**：对于ERA，Q1中已经提到，涉及不同物理通道之间的路由，或者说，不同网段之间PNC信息路由。8个PN需要每个网段分别处理，即：PNC #n需要在每个网段独立处理其PN状态，以此协调各网段内的PN状态，因此需要6 * 8个ERA Timer分别计时。

**注意**：EIRA信号，每类总线共用一个，比如：3路CAN，均参考一个EIRA接收信号的PNC信息即可，而ERA需要每路总线，各自处理自己的ERA接收信号，以便于路由给其他网段。

## Q3： 外部PN请求被镜像回请求总线，并提供给中央网关（必需的）物理通道。在子网关情况下，请求位不得镜像回请求的物理通道，以避免中央网关和子网关间的静态唤醒。如何理解这里的"镜像"？

**A3**：如上这段话的出处先了解一下，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vxGyJCka2Mj7QwXnZLWq8GC7aNJumrmL1iaZVrcgRGSfulicN52071zItQP5Szyy8WiccTHicsaDwI5vg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**解释：**

子网关收到PNC #n信息，发送网络管理报文时，不要将PNC #n发送到接收的物理通道。比如：ECU4::E节点收到ECU2::C节点的PNC #n，ECU4::E在发送网络管理报文的时候就不要置位PNC #n（=1）。而中央网关，如：ECU1::D需要将收到的PNC #n发送回CAN2 Bus。为什么子网关不能将PNC #n发送回对应的总线呢？

按照规范要求，一个网段内有一个Active PNC Gateway，其余的为Passive PNC Gateway，ECU1是中央网关（节点D为Active PNC Gateway）、ECU4是子网关（节点E设计为Passive PNC Gateway），5个ECU的关联关系如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vwcicEfbUDIqmzJBqzgcFtIRNkicLwBib9KeTxcpcAwk2iaYBhV2Q5vIt8Cn8GdPbTdSWGUwbujFevWSg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**假设:**

**不按照****规范要求****，一个网段内有两个Active PNC Gateway，其余的为Passive PNC Gateway，ECU1是中央网关（节点B、D为Active PNC Gateway，分别对应Can1 Bus和Can2 Bus）、ECU4是子网关（节点E、F也为Active PNC Gateway，分别对应Can2 Bus和Can3 Bus），5个ECU的关联关系如下所示：**

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vwcicEfbUDIqmzJBqzgcFtIRa0VdAO3ggdFM4wPib3ANQUrmMRicInWCHs5hVu7HUpw1vSu47icLAjMuQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这样会出现什么问题呢？规范要求：Active PNC Gateway节点是网段内最后一个释放PN网络的节点，如果在一个网段内存在两个Active PNC Gateway节点，会使得两个Active PNC Gateway一直不释放网络，导致网络锁死（谁都不释放，都要最后一个释放PNC）。Autosar规范解释如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vxGyJCka2Mj7QwXnZLWq8GCdyrMaFt6lAMZSPNMRL1gplgIqlEyoYAQ1eaYLLtia5mSaUPrfDc2Egw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

先消化一下Autosar的这个解释，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vxGyJCka2Mj7QwXnZLWq8GCCefTPL3yTCvW7vdicAhu2RKWVN232LyPr9y6rmpXeAiaicvz64CVHf3vg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**解释：**

一个ComM通道如果映射到了两种不同的PNC Gateways，只能有一个主动协调此通道的网络状态，其他的被动协调（或者说不协调）。说白了就是一个ComM Channel有一个Active PNC Gateway节点协调即可。所以，在设计网关节点的PNC Gateway类型时，需要小心。

因此，中央网关和子网关的节点均关联到同一个网段，需要将子网关的节点设置为Passive PNC Gateway，以此避免网络状态锁死。

“镜像”就是将从总线收到的PNC #n信息再发送到总线。

![图片](https://mmbiz.qpic.cn/mmbiz_png/quuOyCqONwdD16hI11wAQWxGp4ajJ4DMnbsHGb4ViandFryeibcQb1Idxb3MHmrh20988OSES3OU1wPTicQbAr94g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



参考资料

SIMPLE TITLE

AUTOSAR_SWS_COMManager.pdf

AUTOSAR_SWS_CANNetworkManagement.pdf