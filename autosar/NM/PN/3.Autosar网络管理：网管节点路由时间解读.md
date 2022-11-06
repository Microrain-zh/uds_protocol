# Autosar网络管理：网管节点路由时间解读

Autosar网络管理中，如果节点是网关节点，对开发和测试来说都是不小的挑战，如果对需求解读不到位，开发架构设计错误，后期的测试也就bug bug bug...

本文针对网关节点（包含PNC功能）解读路由需求以及开发注意事项。本文讨论的内容涉及PN（Partial Network）功能，本文源于工程实际，还是能给大家点启发的。

**提示**：基于can总线讨论

1

需求明确

需求：某个ECU包含两个节点：Node1和Node2，两者为网关节点，均包含PNC功能。要求网络管理报文的路由时间＜15ms。

**提示**：

- Node1和Node2是主动激活节点，即两个Node均具有快发模式;
- PNC1和PNC2**均关联Can1和Can2**。



2

需求说明

这里我们从测试角度分析需求应该如何测试。

**举例分析**：上位机（Tester）模拟发送一帧网络管理报文0x5xx（网络管理报文有效范围：0x500~0x53F）到Can1 Bus，Can1 Node收到这帧网络管理报文以后，内部转发给Can2 Node（实际由ComM判断PNC，进而决定哪些Node网络状态切换）。在Normal Mode模式下，Node1会发送网络管理报文0x502到Can1 Bus，Node2会发送网络管理报文0x503到Can2 Bus。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzOzTCMfibKg3PvAiaFWGZVianEg7gibLVLmHSa6k9I5zfkxJoQOvDunRHTt34maiamYk1PxQkMtaWXEOw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

测试关键步骤：

1. Tester发送**仅包含**PNC1的网络管理报文0x5xx；
2. 5s后，Node1和Node2进入NOS(Normal Operation State)状态，且两者均以1s周期外发各自的网络管理报文；
3. 此时上位机模拟发送一帧网络管理报文(**包含PNC1、PNC2**)给Node1，Node1、Node2均进入快发模式，Can1 bus总线上**第一次**出现PNC2置位的**模拟网络管理报文**时间记为T1；
4. Node2也进入快发模式，当Node2发送出**第一帧包含PNC2的网络管理报文0x503**的时间记为T2（Node2此时处于快发模式），如果T2-T1 < 15ms+(15*0.01)ms = 16.5ms，则测试通过。

测试分析图如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzOzTCMfibKg3PvAiaFWGZVianIpYAr4ibc1K4xIuSGia7CZzmvbaKfBeZD2O6SCkqSaQ7laJ55JnVrmOg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

3

开发注意



当理解了需求以后，开发者实现过程中有几点需要注意：

1. Node1接收的网络管理报文是一个范围，而非某帧网络管理报文，比如：本例网络管理报文的范围是0x500~0x53F，该范围内的任一帧网络管理报文，如果PNC关联Node2，均应使得Node2进入快发模式，反之亦然；
2. Node1和Node2的唤醒与PNC相关，与应用报文的路由不要混为一谈。PNC关联哪些Node，ComM会请求哪些Node的网络状态切换，而应用报文的路由可以通过PDUR进行PDR级别路由或者Com层的信号（Signal）路由；
3. **配置参数CanNmPnHandleMulti勾选；**
4. **网络管理有PN功能时，ComM负责调用CanNm_NetworkRequest()接口。**

**坑点**：

Node1和Node2均有Pn功能，配置参数CanNmPnHandleMultipleNetworkRequests需要勾选，当状态由NOS->RMS(Repeat Message State)切换的时候，Node进入快发模式。