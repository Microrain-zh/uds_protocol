# Autosar网络管理：主动唤醒源/被动唤醒源与网络主动唤醒/被动唤醒的关系

网络管理中，主动唤醒源/被动唤醒源与网络主动唤醒/被动唤醒的关系有时让人傻傻分不清，本文侃侃这几个名词。

**提示**：基于CAN节点讨论。

1

主动唤醒源/被动唤醒源

**主动唤醒源**：承担着主动唤醒网络责任的唤醒源，称为主动唤醒源。比如：KL15硬线，User请求，ERA信号等。

1. **KL15硬线**：通过KL15硬线方式唤醒网络，说明当前网络没有节点参与通信，为了快速将网络唤醒，建立通信功能，被KL15硬线唤醒的节点，需要主动地去唤醒网络，进而将网络上其他节点唤醒。所以，可以将KL15硬线看作主动唤醒源。同理，类似于KL15硬线唤醒网络的其他硬线唤醒方式，也可以看作主动唤醒源；
2. **User请求**：User请求，是指通过ComM_RequestComMode()接口请求通信的方式，发起点为SWC，由于功能需要，节点需要在某些工况下主动拉起其他节点通信；
3. **ERA信号**：ERA信号怎么看作是主动唤醒源呢？首先，ERA信号的使用，说明当前节点有多个物理Channel(ComM的Channel与之一一对应)，PNC信息需要在不同的Channel之间路由，以实现不同网络唤醒的目的。

比如：CAN 1在CAN BUS 1上收到一帧网络管理报文，包含PNC #n = 1，且PNC #n与CAN1和CAN2均关联，PNC #n需要由CAN1路由到CAN2，CAN BUS2网段内可能节点均没有唤醒，需要有节点承担唤醒CAN BUS2 网络的责任，即：主动唤醒CAN BUS2网段内的节点。此时，路由到CAN 2节点的ERA信号就可以充当主动唤醒CAN BUS2上节点的责任，所以ERA信号可以看作主动唤醒源。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzmX0JBRKq368kkqh9XBYta8SHV4l012YqjWicJ3VApRfzBCSA0Zd3xN7ZRib9DUQxobVs7uoWrYibYA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

除了上述的的主动唤醒源，还有一些定时器、传感器也可以作为主动唤醒源。传感器一般与硬线连接，类似于KL15硬线。定时器的使用场景不清楚大家有没有遇到，这里给一个场景：智能补电。如果车辆长时间处于休眠状态，蓄电池可能亏电，亏电会导致车辆无法正常使用。为了防止蓄电池亏电，有些车上会配置智能补电功能，通过定时器设置定时时间，如果此时间内车辆未有启动，则定时器主动触发对应节点的唤醒，对蓄电池进行补电。

**被动唤醒源**：不需要承担唤醒网络责任的唤醒源，称为被动唤醒源。比如：收到NM Msg。对于收到NM Msg需要分情况讨论：

1. **网络管理没有PN功能**：节点收到的网络管理报文没有PNC信息，此时网络管理报文看作被动唤醒源。
2. **网络管理具有PN功能**：如果对应的ECU充当Gateway角色，且有多个物理Channel，PNC #n关联多个Channel，网络管理报文可看作主动唤醒源（前面提到的ERA信号）；如果PNC #n仅关联本Channel，不需要路由，网络管理报文看作被动唤醒源。

2

网络主动唤醒/被动唤醒

**网络主动唤醒**：由主动唤醒源触发，调用CanNm_NetworkRequest()接口唤醒网络的方式称为网络主动唤醒。

**网络被动唤醒**：由被动唤醒源触发，调用CanNm_PassiveStartUp()接口唤醒网络的方式称为网络被动唤醒。

## 问题拓展思考

对于PNC模式的切换，群内小伙伴提出了这样一个问题："ERA = 1时，PNC由PNC_NO_COMMUNICATION切换到PNC_REQUESTED。而EIRA = 1时，PNC由PNC_NO_COMMUNICATION切换到PNC_READY_SLEEP"，两者为什么不同呢？

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vwcPcNScv5Cyaq5xv7Ja0w9dTt81k9w8V51zajog2g4ImIEVMickxOhyGdtbWuh2iactWiawVufwc7Gw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

关于ERA、EIRA前文已经聊过，可以参考[Autosar网络管理：Partial Network基础 之 ERA/EIRA、PNC Gateway](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247488858&idx=1&sn=163dd2e1d9000507d5a726543a8d5c21&chksm=fa2a4b2ecd5dc238db82f4e6ba7be2accd7e50b5e30f0f506b860b0351c097df8b3f5510c1bd&scene=21#wechat_redirect)和[Autosar网络管理：CanNmPnResetTime对关联Tx PDU的发送影响](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247488904&idx=1&sn=3c2edb2298cdbeee772471a70241f56c&chksm=fa2a4bfccd5dc2ea2beb373cfea36e63d37e0882d664794e1b86c4de49f7a21b89c653362b1d&scene=21#wechat_redirect)。这里说一下个人理解：ERA的使用需要配合Gateway的使能，当某个PNC = 1时，说明有节点（假设节点A）需要通信，假设节点A需要和不同网段的其他节点（假设节点C）通信，需要经过节点B、节点D的路由，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyMkgspzzsiajg4oWzCdQQDicFJ2YwF9ttSpialibyB57f5ewsyQjmsps6fQWh7rrvUwKHlayOpFzSwrQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如果想唤醒Can2 Bus的节点C网络，需要节点D（与节点C同一个网段）发送网络管理报文唤醒节点C。主动发起通信的节点A在Can1 Bus，需要和Can2 Bus上的节点C通信，需要外部信号（PNC #n = 1）发送给节点B，由节点B路由给节点D，将PNC信息发送给节点C。

- ERA = 1，与此PNC相关的节点（B、D）进入PNC_REQUESTED状态，节点B、D的Channel请求进入 COMM_FULL_COMMUNICATION 状态，调用Nm_NetworkRequest()接口将Can 2 Bus上的节点唤醒；如果进入的是PNC_READY_SLEEP模式，ComM将会释放COMM_FULL_COMMUNICATION状态，且PNC信息不能路由，Can 2 Bus上的节点无法唤醒，节点A、C无法通信。
- EIRA = 1，只是想把通信留在本网段，当前节点参与通信即可，不需要和外部网段通信，因此进入PNC_READY_SLEEP状态，网络被动唤醒。