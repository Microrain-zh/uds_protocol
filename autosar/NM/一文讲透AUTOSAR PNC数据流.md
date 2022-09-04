# 一文讲透AUTOSAR PNC数据流

## 1.从PN到PNC

PN一般指Partial Networking，中文名是部分网络或局部网络。

根据AUTOSAR_EXP_LayeredSoftwareArchitecture这篇PPT的说法，PN的初衷是在AUTOSAR中，实施高效的能源管理，其目标是提供一种节能机制，尤其是在总线通信处于激活状态时（例如充电或KL15处于激活状态时）。

Partial Networking允许在不需要那么多ECU工作的时候，关闭一批ECU的网络通信。其他ECU可以继续在同一总线通道（比如动力CAN）上通信。对于从节点来说，就是需要你的时候，你必须在；不需要你的时候，你必须闭嘴。通常CAN和FlexRay是支持Partial Networking的。

Partial Networking的兄弟被称为Pretended Networking，姑且翻译为装模作样联网。这种方式允许在总线通信时关闭现有网络中的ECU，节点可以自行决定是否切换到休眠模式。比如一个从节点，把KL15拔了，ECU就不工作了，发什么CAN报文唤醒都不起作用。



![图片](https://mmbiz.qpic.cn/mmbiz_png/fOVQt3pZrMbtibM4SYp3FR25Sdia9rnibdtuLMmia929H8H2iccxKB3NnEkFyMySQvlGuzY56r2S3y91QibxrHawnGXQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)AUTOSAR_EXP_LayeredSoftwareArchitecture（V4.2.2）p155



如上图，黑线是真实的CAN总线，ECU A、B、C、D都被真实的双绞线连在了一起。但是！从功能上来讲，ECU A和B可以划分为一组，ECU B、C、D可以划分为一组。这样我们就把真实的物理CAN总线，圈成了两个相对独立的网络小组，组1和组2。我们管这样的小组叫做Partial Network Cluster，中文名是部分网络集群，姑且理解为虚拟CAN小组。这些小组成员的特点是，要醒一起醒，要睡一起睡。

PNC一般指Partial Network Cluster，是一组用于支持车辆功能的系统信号，这些功能分布在车辆网络中的多个ECU上。

PNC若是蝶，它化茧成蝶之前是VFC。VFC指Virtual Function Cluster， 是初期设计阶段的一种通信概念，用于实现一个或多个车辆功能所需的软件组件之间的端口级通信。这里要解释下AUTOSAR的开发思想，为了实现功能我们需要若干个SWC（Software component，软件组件）。这些SWC根据功能组成了若干个CSWC（Composition SWC），把CSWC之间的端口（Port）连在一起，就组成了VFC网络。



![图片](https://mmbiz.qpic.cn/mmbiz_png/fOVQt3pZrMbtibM4SYp3FR25Sdia9rnibdtaPjTXjp6ha51wxRxevsA8icBP5alAAsqkXUA7Z5W7ra7B1iblpiclKQvw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)AUTOSAR_EXP_LayeredSoftwareArchitecture（V4.2.2）p158



后来，图纸变成了现实，VFC变成了PNC（基于CAN的）和ECU内部的Interface，CSWC则变成了真实的ECU。



![图片](https://mmbiz.qpic.cn/mmbiz_png/fOVQt3pZrMbtibM4SYp3FR25Sdia9rnibdt9FK6TVIhibdUbfCk39HBia4n15SScmIemOjgLyTbE8XIVUX090SBS0dQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)AUTOSAR_EXP_LayeredSoftwareArchitecture（V4.2.2）p158



**总结：PNC是住在CAN Bus上的小团体，既求同年同月同日醒，又求同年同月同日睡。**

## 2.PNC醒和睡的暗号是什么

CAN上的网络管理帧有8个字节，通常我们会占用Byte2（含Byte2）之后的字节，作为PNC的区域。举个例子，Byte2里头有效的PNC位就是PNC16-PNC23，Byte7里头有效的PNC位就是PNC56-PNC63。以PNC16举例，如果这个位的值是1，就是PNC生效，反之为0则PNC失效。



![图片](https://mmbiz.qpic.cn/mmbiz_png/fOVQt3pZrMbtibM4SYp3FR25Sdia9rnibdtaGqXXJYXvibDib3Sib4wCibLDHAc7Z0ib3FsNG0WeKQlFI2xFNYZVPsjuJw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)AUTOSAR_SWS_CANNetworkManagement（V4.2.2）p32

![图片](https://mmbiz.qpic.cn/mmbiz_png/fOVQt3pZrMbtibM4SYp3FR25Sdia9rnibdtVibKCewZOeHakwC0ZMIOxk4xhAl2Y7S04EjW5AwrY7goyLaLZOql3Mw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)AUTOSAR_SWS_CANNetworkManagement（V4.2.2）p33



这里也要注意，对于一帧含有PNC信息的网络管理报文来说，位于Byte1（CBV，控制位向量）的PNI Bit是需要置起的，这是后续判断PNC生效与否的先决条件。即PNI Bit若为1，则需要继续检查PNC各个位是否置起；PNI Bit若为0，PNC信息整体丢失，注意不是失效，是上层收不到PNC信息。

**总结：PNC有效与失效的信息藏在网络管理报文的User data中，以位为最小单位，1有效，0无效。但PNI是前提条件，PNI为1，PNC信息才能向上层传递；PNI为0，算作没收到PNC信息。**

## 3.从站获取PNC信息的数据流



![图片](https://mmbiz.qpic.cn/mmbiz_png/fOVQt3pZrMbtibM4SYp3FR25Sdia9rnibdt3hiabAInRmx7zzJNYEqicz2blxEEKVF6AdO6HxX2eOXCuKiboQunlSJVQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)AUTOSAR_EXP_LayeredSoftwareArchitecture（V4.2.2）p159

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/fOVQt3pZrMbtibM4SYp3FR25Sdia9rnibdtGlaplMiaQ0WOo9KLLDFOZmZqUQb57j58qKvykeINicgHScQNiaHlWiaMYw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)



我们看下数据流的流向。为了获取到EIRA（External Internal Request Array）这个信息，我们在Ecu Config中设置了三个Global PDU，即PDU_CanIf_CanNm(8bytes)，PDU_EIRA_CanNm_PduR(6bytes)，PDU_EIRA_PduR_Com(6bytes)。

首先是CanIf，我们在这里可以先对网络管理报文根据CAN ID进行滤波，之后将数据放到PDU_CanIf_CanNm里面。

再向上是CanNm，8个字节去掉了Node ID和CBV，变成了6个字节。检查CBV中PNI bit的值，若为1则向上层传递User Data。PNI如果为0的话，就算没收到任何PNC，一定时间后会报超时。

到了PduR，我们配置了一条Path，把PDU送往Com（注意这里是Trigger发送），ComSignal我们假定主机厂要求只取前3个字节，后面3个字节被舍弃。这样我们只剩下了原来网络管理帧的Byte2-Byte4。

最后ComSignal传给了ComM，我们会进一步通过Pnc Id去找到Pnc的位置，并检查它的值是到底1还是0。