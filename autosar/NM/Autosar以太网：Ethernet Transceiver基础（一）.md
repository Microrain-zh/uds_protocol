# Autosar以太网：Ethernet Transceiver基础（一）

Transceiver是数字信号与模拟信号转化的物理硬件，如果对使用的Transceiver没有一定的认识，那么在Bug的排查中，往往会有种似懂非懂的感觉，来，认识以太网，从认识Ethernet Transceiver开始。

**重要提示**，Vector的E-Learning，对总线的入门学习很有帮助，链接：

https://elearning.vector.com/

**名词解释**

| **Name**    | **Description**                                  |
| ----------- | ------------------------------------------------ |
| EC          | Ethernet controller                              |
| ET          | Ethernet transceiver                             |
| **Eth**     | Ethernet Controller Driver (AUTOSAR BSW module)  |
| **EthTrcv** | Ethernet Transceiver Driver (AUTOSAR BSW module) |

**以太网基础概念**

在连接网口的时候，MDI和MDI-X两种模式要清楚。

**MDI**(medium dependent interface - II mode):平行模式

**MDI-X**(medium dependent interface - x mode):交错模式（crossover mode），即发送端的**发送Pin脚**与接收端的**接收Pin脚**连接，一般同种设备（比如：设备都是网卡）使用该方式。

**回波消除法**：以太网中，两个节点采用**双绞线**传输**对称差分电压**。作为发送节点时，将自己的差分电压施加到双绞线上；作为接收方时，将总线上的差分电压减去自己施加的电压，得到对方节点发送的电压，这就是回波消除法。

**PHY**：实际嵌入式开发中常说的PHY（Port Physical Layer，端口物理层），即下图中深灰色小块，可以理解为：**连接网线水晶头的接口+Transceiver**。

**Link**：连接两个节点的PHY。

以太网的拓扑结构是点对点结构（Point-To-Point），如果要构成局域网（LAN），则需要用到交换机（Switch），交换机可以有多个PHY，可以连接多个Node，即二层交换。

以太网是全双工通信，所以两个PHY之间可以同时发送，不会产生冲突。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzBNHb6thTTmvftAe71Iq3WBKasoeDTz2ibBe0MZ411bsl9OLRuS8u6x9kLMm5GobUbOwOC6ZmVWUg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**Ethernet BSW栈**

在Autosar中，Ethernet Transceiver所在的层级如下所示。这里我们要区分硬件层和软件层的概念，比如：ET是指硬件Transceiver，Eth是软件驱动，属于Autosar BSW模块。

类似CanIf，EthIf是所有以太网**上层模块获取数据/发送数据**的"必经之路"，很多难解的Bug，可以从这个层级动动脑筋。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzBNHb6thTTmvftAe71Iq3WxO6bKL1qf6aQKmCGCyHwMLIONe4ibyyvGw3g4ZuGRckFRDawt4oguvw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在Autosar中，不同类型的Ethernet Transceiver索引均从0开始索引，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzBNHb6thTTmvftAe71Iq3WdNRI0vAEUJ5gicTMzcIAhdcPvfNliaBSOFc3eeXqWSkysEp503t4KgWw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在CAN/CANFD、FlexRay等总线中，我们常说Transceiver，而在Ethernet开发中，我们常说PHY。PHY在物理介质和控制器之间，如下所示。

**MII**:Medium Independent Interface

**MDI**:Medium Dependent Interface

**uC**:Host，包含MAC，可以理解为ECU

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vx93YCxWu3SDD9ibLptEKicuU5bqwK6ZZIjgDWL7JFiaJ2FYlLHLscvcL90RW9DrUicwtnJI66DX4tdVQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

PHY可以集成在uC中，也可以独立在uC之外，实际常用的方式是独立于uC之外，即上图所示集成方式。

当通信速率不同时，MDI外接的双绞线数量也将不同，比如1000Mbps，会使用4对双绞线，100Mbps时使用两对双绞线，每对双绞线都是双向传输数据。

**RTL8211FD(I)**

Autosar的协议比较简单，我们来了解一下汽车嵌入式使用的一款Ethernet Transceiver:RTL8211FD(I)，这款千兆以太网Transceiver由瑞昱[yù]开发。RTL8211FD(I)对应Datsheet下载链接：

https://files.pine64.org/doc/datasheet/rock64/RTL8211F-CG-Realtek.pdf

## 1、Pin描述/MDI

阅读Transceiver Datasheet，先搞清名词：

| **Name** | **Description**                              |
| -------- | -------------------------------------------- |
| I        | Input                                        |
| O        | Output                                       |
| P        | Power                                        |
| G        | Ground                                       |
| PU       | Internal Pull Up During Power **On Reset**   |
| PD       | Internal Pull Down During Power **On Reset** |
| LI       | Latched Input During Power up or Reset       |
| IO       | **Bi-Directional** Input and Output          |
| OD       | Open Drain                                   |



## 2、RTL8211FD(I)结构图



解读下图信息：

- MAC的时钟由Transceiver提供，一般MAC集成在主芯片中；
- Transceiver和MAC由外部Vreg供电；
- 外部时钟给Transceiver提供"心跳"。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzBNHb6thTTmvftAe71Iq3WP5mIsKzv5CWPPcsRz7WdhgHlXoiciclq9DVg5VY90HgR2zqo0XvcTRcA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

RTL8211FD(I)内部原理图如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzBNHb6thTTmvftAe71Iq3WcpneqUV9LkunJ8zb3KcgVLbh4cnMylArUXkRFIgMDZWTt2Olhpw8hw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

好了，不能再多说了，吃多嚼不烂，下期更精彩...

