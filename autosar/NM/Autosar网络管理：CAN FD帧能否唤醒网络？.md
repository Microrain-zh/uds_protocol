# Autosar网络管理：CAN FD帧能否唤醒网络？

如标题问题，你能给出具体答案吗？这个问题源于Autosar群内小伙伴的讨论，先说一下我的答案：不能。这有点打脸我之前的说法，先给读者说声抱歉。本文就这个问题，深入讨论一下。

1

11898-2中的WUF(wake-up frame)定义

11898-2规定：Transceiver识别有效WUF(wake-up frame)时，**首先要识别该CAN Frame是否是Classical CAN Frame**，也就是说：Transceiver接收到CAN FD Frame，不能作为有效的WUF。有效WUF检查的condition如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyJLz1icZMHMXQ3ZudfEU5k4EicIsYy2OxKB4AttHFeVdics4UxHLwDKJUANNSCeDtLwiaQiaw2Ezyswsg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2

TJA1145手册中的WUF(wake-up frame)定义

TJA1145手册中，在描述CAN FD的部分也明确说明CAN FD帧不能作为有效的唤醒帧。意思就是说：即使收到CAN FD类型的网络管理报文也不能唤醒网络，会被忽略，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyJLz1icZMHMXQ3ZudfEU5k4X3NyUT1MXN6tpc0IeP0OuT1YpPGYMSuxgTeOSVfov8X37PibVwpYibeg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

一般来说，uC的供电受控于Transceiver。而Transceiver控制对应的电源模块时，首先要收到Wakeup Pattern，之后才能触发唤醒事件，进而使能供电模块，uC被供电。对于1145 Transceiver的Wakeup Pattern分两种情况讨论：

## 1、Standard Wake-up Pattern

Standard Wake-up Pattern唤醒时序如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyJLz1icZMHMXQ3ZudfEU5k4sXbgqS8IlWco611uwZ3k6gzlHFRESb7SEDJxd57hIgsC89h3fJzicmA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

怎么理解呢？只要满足上图的电压时序变化就能触发Transceiver的唤醒事件，进而供电模块工作，uC供电，这也是为什么**任意一帧报文都能唤醒ECU的原因**，注意：网络此时未必唤醒，软件需要进一步识别是否收到有效唤醒源。同理，对普通的Transceiver（比如：1045），也是任意一帧报文（CAN/CAN FD均可）均能唤醒ECU。

再进一步分析，如果能模拟出Standard Wake-up Pattern，不用发送报文也能触发唤醒，比如：极端天气情况，Transceiver的抗电磁干扰能力又不行，也可能触发唤醒。

## 2、WUF，Partial Networking使能

此种方式需要Transceiver支持partial networking功能，即：只能特定的报文触发唤醒，比如：目前使用率比较高的1145。

这需要ECU在断电前，设置好1145的PN唤醒功能，即：设定可以唤醒的网络管理报文。这也是目前使用1145常用的做法，不然买1145干啥呢？

能否将CAN FD类型的网络管理报文作为有效的WUF呢？我的理解：软/硬件都可以实现。但是为什么硬件不支持呢？这个可能需要从商业化的视角看，做工程，需要产出和效益，不可能顶着亏损为某种特殊功能而做产品。不管何种工业产品，都会遵循标准和规范。既然11898-2这种国际规范都没有说CAN FD帧可以作为有效的WUF，至少Transceiver的生产商不会将其作为一种附加功能生产。如果增加CAN FD帧作为有效的WUF，那么产品的成本会增加，产品成本增加，会转嫁到客户身上，而客户又不需要这样的功能（或者一小部分客户需要），为什么要买这样的产品呢？如果某个客户的需求量大，而且会有持续的需求，可以和Transceiver的生产商谈，为这样的客户单独供货。

所以，短期内，硬件实现CAN FD帧唤醒网络还不现实，或许随着技术的迭代，支持CAN FD的Transceiver成本和普通Transceiver成本相差无，支持CAN FD的Transceiver淘汰普通Transceiver时，CAN FD帧作为有效WUF会提到日程上，并且对应的规范也将其补充进来。这也许不会太远，因为CAN XL规范都已悄然到来，CAN XL的吞吐量(最高2048 bytes)和速率(10Mbps+)值得期待。

3

工程问题思考

网络管理部分，在实际的项目中，大家是否遇到过网络管理报文是CAN FD类型的？目前，我没有碰到过网络管理报文类型要求是CAN FD类型。即使网段内有的节点使用普通Transceiver（比如：1043），有的节点使用支持CAN FD的Transceiver（比如：1145），但是，**唤醒网络的报文类型还是会要求是Classical CAN帧**。因为支持CAN FD类型的Transceiver向后兼容，所以混用没有太大问题。

任何总线速率的提升都会带来抗干扰能力差的风险，所以，抗电磁干扰测试必不可少。尤其汽车，作为一个移动的交通工具，会在多种天气情况下使用。

**提醒**：CAN FD的项目中，会配置SSP(Secondary Sample Point)，而且会测试该采样点。开发时，不要配置错误，严格按照需求走。

## CAN FD速率切换位置

这里给大家补充一点细节：CAN FD报文中，速率切换位置和各个域（field）对应位置。假设：项目中使用CAN FD的速率是500Kbps~2Mbps。真正2M速率开始切换的位置是**从BRS Bit到CRC Delimiter Bit**。由下图可以看出，2Mbps的速率不仅仅在Data field，CRC filed和Control field也有2Mbps的bit。

下图中标识**Arbitration Phase**的位置使用500Kbps，标识**Data Phase**的位置使用2Mbps。所以，**不要混淆Data Phase和Data Field**。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyJLz1icZMHMXQ3ZudfEU5k4f1wz2PkJ46ttyqO7zHskibRKKYlBfq0Wqz0ibnKNtEySjz8O0CFJ0zDA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

采样点的测试中，对于CAN FD报文，BRS、CRC Delimiter 位是没有必要测试的，这两个位置速率在切换，测试准确性不高。



写在最后

写公众号的日子，遇到很多技术很棒的读者，他们给我指正、交流，受益颇多，感谢！有需要的朋友，可以进群聊，群里有很多专业的小伙伴可以答你所惑！



![图片](https://mmbiz.qpic.cn/mmbiz_png/quuOyCqONwdD16hI11wAQWxGp4ajJ4DMnbsHGb4ViandFryeibcQb1Idxb3MHmrh20988OSES3OU1wPTicQbAr94g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



参考资料

SIMPLE TITLE

ISO 11898-1-2015.pdf

TJA1145 High-speed CAN transceiver for partial networking.pdf