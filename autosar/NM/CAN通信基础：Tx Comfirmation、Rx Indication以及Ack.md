# CAN通信基础：Tx Comfirmation、Rx Indication以及Ack

嵌入式开发中，知识细碎，工程实际中，如果某些细碎的问题理解不到位，就可能成为Bug的拦路虎。本文，聊一聊CAN通信中的Tx Comfirmation、Rx Indication以及Acknowledgement（Ack）。

## 1、Tx Comfirmation 与 **Acknowledgement(Ack)**

了解Tx Comfirmation之前，我们需要先清楚“发送请求（Transmit Request）”，只有先发送请求，才有对请求结果的确认（Comfirmation）。可以参考前文[Autosar通信栈：发送返回OK和发送确认是一回事吗](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247487913&idx=1&sn=a5e02e3ae6887dd7f14ac617ad6b4623&chksm=fa2a4fddcd5dc6cb2e7f894df52815da7aabb2daf503448f1096f8ed65ccb034b95a826cc56e&scene=21#wechat_redirect)。

**先说发送请求**，当用户请求发送CAN报文时，最终会调用Can_Write()接口，完成报文发送的请求动作，该接口有一个返回值，表示发送请求成功与否。如果发送请求成功，返回E_OK；如果发送请求不成功，返回E_NOT_OK。E_OK表示什么意思呢？意思是说：CAN驱动有可用的缓存空间（RAM），请求发送的报文信息已经被成功地放入CAN驱动缓存区（注意，还没有发送到CAN BUS)，这只是完成了发送请求，如下绿色框图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vxGRIaTrn4xPltek94SlVYWuRRVmFyjk2ImaypFUEUc6XP9RMxeW5u0DibjrG7icRAiaDiaRrLbEmEZkA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

当发送的报文信息被成功添加到驱动缓存区以后，这些报文就时刻准备发送到总线。我们知道，一个网段内可以有多个节点，每个节点都等着发送各自缓存区中的报文，谁先发送呢？答：通过仲裁决定哪个节点可以使用CAN BUS，仲裁赢的节点，获取总线使用权，进而发送报文。谁发送的报文优先级高（CAN ID越小，优先级越高）谁就可以抢占总线使用权。可以参考前文[CAN总线仲裁原理](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247489165&idx=1&sn=9f9c3edaeab7611c228e4d76ab8e8d2e&chksm=fa2a48f9cd5dc1efc810269798bfaa0a3c31fc71c99ec59d67ab3f33ad91818e8516f4cad777&scene=21#wechat_redirect)。

当网段内的某个节点获取总线使用权以后，就开始发送缓存区中最高优先级的报文，发送节点为了获悉自己所发报文信息是否正确，在使用Tx Pin发送的同时，使用Rx Pin接收（CAN总线是串行总线)，当然，总线上的其他节点（处于监听状态的节点）也会通过CAN BUS接收总线上的报文。

发送节点边发送边接收过程如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_gif/eEEQvxEw8vxGRIaTrn4xPltek94SlVYW3jvIR22L3xvXyibSsgiaTXBCh96qlYqUvMRjJqofH6IkgWHnpNO7Fhpg/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

发送节点按照CAN报文格式发送各个位域，发送节点发送ACK Field(ACL Slot、ACK Delimiter)时，2 Bit均发送隐性位（Recessive，1），当接收节点接收到ACK Field的第一个Bit（ACK Slot，ACK应答槽）时，接收节点将其置为显性位（Dominant，0），由于"线与"原则（显性位覆盖隐性位），所以，CAN BUS上，ACK Slot = 0，这个动作称为应答（Acknowledgement），即：接收节点应答发送节点，告知发送节点，其发送的报文已被成功接收，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyg5ap9aAQfVnGq8pQJwQkoCqw9rE7vHHcAsBItP4vyUcNFjDqFG4nZ6xRcpkqEvKYbCAKoG5lBjA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接收节点收到隐性（"1"）的Ack Slot时，将其置为显性（"0"），这个动作由**接收节点**的Tx Pin触发。

**提示**：

- Ack动作由Controller完成，不是Transceiver；
- 总线上可能有多个接收节点，只要其中一个应答（Ack）,发送节点就认为数据发送成功；
- 当总线上没有节点应答发送节点时（即：总线上只有一个节点），发送节点检测到Ack Slot为隐性（"1"），会发出错误帧（Acknowledgment Error），同时，发送节点的TEC累加，之后发送节点尝试重传。所以，TEC很快会达到128，进入Error Passive模式。注意，如果一直是Acknowledgment Error，TEC不会再累加，不会进入Bus Off。

当发送节点的发送报文被其他节点成功接收以后，发送节点就会将这个信息反馈给对应的User，告诉请求发送的User，这个过程就是Tx Comfirmation。这个反馈途径有两种：

1. 中断方式，成功将一帧报文发送到总线，且被其他节点有效接收以后，发送节点进入发送完成中断例程，调用User注册的回调接口，比如：CanIf_TxConfirmation()，CanIf在进一步调用目标上层Xx_TxComfirmation 回调接口告知User发送结果。
2. Polling方式，CAN驱动的Main函数周期性（eg:5ms）的检测是否有报文被成功发送，如果有，则调用目标User的Xx_TxConfirmation()，告知其请求发送报文被接收的结果，即：已被其他节点成功接收，且被应答。

## 2、Rx Indication

什么是Rx Indication呢？CAN BUS上有节点发送报文，就有目标节点接收报文，当接收节点接收到所需的报文以后，需要将接收的信息给到目标User，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_gif/eEEQvxEw8vxGRIaTrn4xPltek94SlVYWwfqicdhEGGEPxksrEsCiaczramia9mOP2iaJ36t2L8zkCbu9GV7sV0NQ9A/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

## 如何通知目标节点呢？两种方式： 中断方式，当接收节点成功接收到一帧CAN报文以后，程序进入接收完成中断例程，在接收完成中断例程中，会通过上层注册好的Call Back（eg:CanIf_RxIndication()）将接收的报文信息送给目标上层。Polling方式，CAN驱动的Main主函数不断地查询是否有报文接收，如果有，则调用Xx_RxIndication()将接收信息送给目标User处理。 如上，将接收报文信息通知到目标User的过程就是Rx Indication。