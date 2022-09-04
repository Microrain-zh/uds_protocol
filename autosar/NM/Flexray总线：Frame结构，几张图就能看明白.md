# Flexray总线：Frame结构，几张图就能看明白

Flexray报文包含：帧头、有效负载、帧尾三部分组成。如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vydFM3zFKibVhZ3rltJxbAhFd8RjOiaaR4w6XIBJteUcCI2qj51LpQDOC9Qfw70iahXOyVOyUL9AUwNg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Flexray的Frame结构如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzOXpHnGAb03FHcnyGiawUXIJ8SIQyN0X5IDlVHplhUGksnht7TRDFsLmVIZXx5RYXgWoQuFibPXoyQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**提示**：参考《FlexRayCommunicationSystem V2.1》

1

帧头



帧头共由40个Bit构成（5 Byte），具体包括：Indicators、ID、Payload Length、CRC、Cycle Count。

**1、Indicators**

Indicators由5bit构成，精确指示报文信息。

**Bit1**:保留位，发送节点将发送逻辑0，接收节点忽略该bit位值。

**Bit2**:有效负载指示位。

- 当该bit = 1时，如果是**静态报文**，则有效负载中正在传输**网络管理向量**；如果是动态报文，则有效负载中正在传输**报文标识符**。
- 当该bit = 0时，说明静态报文没有传输网络管理向量或者动态报文没有传输标识符。

**Bit3**:空帧指示位。

- 该bit  =1，表示负载段包含数据；
- 该bit =0，表示负载段数据无效，发送节点可以将负载段的数据全设为0 。

**Bit4**:同步帧指示位，指示**静态段**中传输的报文是否为同步帧。

- 该bit = 1，所有的接收节点要进行同步处理；
- 该bit = 0，接收节点不做同步处理。

**Bit5**:启动帧指示位。指示**静态段**中传输的报文是否为启动帧。

- 该bit = 1，此静态Frame是启动Frame，规范中要求只有冷启节点同步帧置位时，启动指示位才能置位，即Bit4 = Bit5 = 1；
- 该bit = 0，此静态Frame不是启动帧。

**2、ID**

ID由11 Bit表示。ID标识报文，并与时隙相对应，0x00表示无效报文（接收节点进行错误处理），所以有效ID范围是1~2047（2^11）。

**3、Payload Length**

Payload Length由7 bit构成，表示**负载段**数据的大小（**以word为单位**）。一条报文最多可以传输254 byte，即最大Payload Length(cPayloadLengthMax) = 127（2^7）。

在静态段中，同一Cycle中所有的Frame负载长度是固定的，即长度一致，比如都是32 byte，一般可配置gPayloadLengthStatic参数。

**4、CRC**

CRC序列由11Bit构成（这里的CRC是Header的CRC），该序列的计算基于ID、Payload Length、同步帧指示符(Bit4)和启动帧指示符(Bit5)。

![图片](https://mmbiz.qpic.cn/mmbiz_gif/eEEQvxEw8vydFM3zFKibVhZ3rltJxbAhFKxb16j7Uf1mWvv3AIHm1iafU1Eiaxfvr2JXoCwuF4BjmyAr5X6Jic8AGg/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

**5、Cycle Count**

帧头的最后是周期计数器，由6个bit构成，表示报文发送的周期数。周期计数器的范围是0到63。

2

有效负载

一帧Flexray报文最多可以传输254个字节的数据。

为静态Flexray报文设置了有效负载指示位，则静态段中前0~12 Bytes用来传输网络管理向量，用于FlexRay的网络管理；如果为动态Flexray报文设置了有效负载指示位，则负载段的前2 Bytes表示Message ID。

3

帧尾

帧尾是24 Bit的CRC计算域，计算的范围包括帧头和有效负载，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_gif/eEEQvxEw8vydFM3zFKibVhZ3rltJxbAhFqvCicRBrOyvLXPBExayXy3SvaXmcV1z0OEudibfA0KFf9vExkFaZIt6g/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)

4

编码

Flexray帧的传输并不是从报文的Header开始传输，是从TSS（Transmission Start Sequence）开始传输，TSS包含3~15个低电平，之后是FSS（Frame Start Sequence）和BSS（Byte Start Sequence）。至此，报文头才真正的开始传输，但是报文每个字节被传输之前都需要传输一个BSS，这样接收方可以**通过BSS的跳变进行同步**，当报文的最后一个字节传输完成后，以FES（Frame End Sequence）标识。**在FES之后是11 Bit的隐性位，即通道空闲界定符**。

静态报文的传输时序如下所示（图片来自Vector）：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vydFM3zFKibVhZ3rltJxbAhFgVEr2hBMAXFuWPc4RPXYficWeib5alIUMqqvl5z9sIknouZe7baZZia8Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

动态报文的传输时序如下所示（图片来自Vector），相对于静态报文，动态报文在FES之后还有一个DTS（Dynamic Trailing Sequence）,这样可以让接收方准确的知道动态报文结束时机。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vydFM3zFKibVhZ3rltJxbAhFvcbogFQfyoOaEH3V6S0lSjQVsCXB7iaFDHMp4UorgxM29ZWmWQf9WcA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

