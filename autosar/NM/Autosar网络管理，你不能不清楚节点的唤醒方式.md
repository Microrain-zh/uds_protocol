# Autosar网络管理，你不能不清楚节点的唤醒方式

我想玩Autosar嵌入式开发的朋友都知道，Autosar中有NM模块，也知道为什么要有网络管理，但是大家未必都能说清楚节点（Node）的唤醒方式，本文通过分析节点供电方式、网络唤醒源来聊聊节点的唤醒方式。



 唤醒源



开发的过程中，提到网络管理，我们首先就得明确唤醒源有哪些，如果搞不清，查bug可能有的折腾。一般来说，唤醒源会分为本地唤醒源和远程唤醒源。前面已经说过，唤醒源能唤醒ECU，而未必能唤醒网络，**唤醒ECU是唤醒网络的必要不充分条件，**可以参考前文：**[（节点唤醒==网络唤醒）？FALSE:TRUE；](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247483765&idx=1&sn=eddd18684737ed67aff7da4b7b6bd501&chksm=fa2a5f01cd5dd61735d991cf0e4a34a14d8fbb6f81274e5f8c96a6cf93ad5c80b363c6d06c25&scene=21#wechat_redirect)**。如下说一下个人对两种类型唤醒源的理解。

- **本地唤醒源：**和硬线相关的唤醒方式一般称为本地唤醒源。如：KL15硬线，硬件传感器信号（如：脚踢门，后备箱打开）。
- **远程唤醒源：**简单说就是和总线信号相关的唤醒方式。比如收到网络管理报文或者指定诊断报文，或者包含KL15信号的应用报文（有些节点没有KL15硬线，而是网关转发包含KL15信号的应用报文唤醒）。

遇到过一种不常见的唤醒方式（确切说我遇到的少），这里和大家聊一聊。比如节点A所在uC是ECU1，有一个外围芯片ECU2控制Vreg，进而控制ECU1的供电。这里使用ECU2主要是想在**ECU1休眠时，定时唤醒ECU1，比如ECU1休眠5h后，自动唤醒ECU1**。这里可以将此种方式理解为本地唤醒源。这里会利用ECU2的定时功能，该功能一般会在ECU1进入休眠前通过SPI设置唤醒时间，定时时间到，触发Vreg给ECU1供电，进而ECU1上电，主程序启动（Application Program）。

该唤醒方式的示意如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vw6CYLS0fo1ib3icoa8yHibMdnKW1XJbwbJuh3Fth4D4H10S144libliaeTIuOndrTrdRXiazqvzo7HErsg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在实际的Autosar NM开发中，大家一定要和系统沟通清楚本地唤醒源和远程唤醒源的定义，以及这些唤醒源有哪些，因为这些在问题排查时是大有裨益的。



节点类型



一个ECU中包含多个节点（Node），这个大家要清楚，不要将ECU和节点混为一谈。举例来说：一个ECU有3路CAN，2路Flexray，1路Ethernet，应该说该ECU有6个节点。也可以具体说，该ECU有3个CAN节点，2个Flexray节点，1个Ethernet节点。总之，在表述的时候要尽可能准确，他人在表达不到位的情况下，我们也要知道ta表达的真实意图。

一般来说，可以根据节点的**启动方式**、**静态电流的消耗**以及**启动时间**分为4种唤醒方式，这里分为A、B、C、D四种类型。**提示：图片可以放大看**。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzAbPgJQM0LoibPicVwEUCrKcjicc9NJ0pzAX2pHbHuWh95jhRJW01ScIj4eTlQJ5un8TaGR8ZN8298g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

也有按照唤醒源组合方式分类的说法，即**本地唤醒源和远程唤醒源组合成四种唤醒方式**。如下说说这四种类型节点网络唤醒的方式。

**Node TypeA：**

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzAbPgJQM0LoibPicVwEUCrKcY5zGfOBDGCAfhn2F0IoErAPFjKpltunTthWnZA0dDpicG1WYiaAIibcwA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这种类型的节点常连KL30（俗称小电瓶，即长电），即使车辆停车和点火钥匙关闭时，该种类型的节点也有唤醒网络的能力。一般这样的节点在不通信的时候会进入低功耗模式，不是说完全的掉电，而是将电流限制在一定阈值下，如100uA，同时支持通信网络唤醒和重新启动应用程序的能力。主要是关闭掉不必要的外设使得电流降低。但是应用程序在保持周期性运行（这个轮询的周期不必太大），以便有网络请求时能快速唤醒ECU和网络。如果收到远程唤醒源的唤醒请求，Transceiver会使能Vreg给uC供电，进而使得ECU唤醒，**ECU唤醒以后再进一步判断收到的远程唤醒源是否是有效的网络唤醒源**（比如：网络管理报文），如果不是则ECU走下电流程，网络没有唤醒；如果是有效网络唤醒源，则唤醒网络。

需要注意的是，有些节点如果有PNC功能，即使收到NM报文，也未必唤醒该节点网络，必须是该节点指定的NM报文。

**Node TypeB：**

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzAbPgJQM0LoibPicVwEUCrKcRDOibqc71QUKJwMv4364YH4qR8Efib4Q900qK8iaerx0M3Xic1JPRW2Z1Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这种类型的节点常连KL30，可以通过硬线和网络总线唤醒，即这两种方式均可以使得uC供电，即ECU唤醒。如果是硬线唤醒，如KL15硬线，网络直接唤醒；如果是网络总线唤醒，则需要进一步判断唤醒源的有效性，即网络不一定唤醒。一般来说，该节点休眠时会将transceiver的工作模式设置为Sleep Mode，当然，这需要看transceiver是否有Sleep Mode。以CAN Transceiver为例，NXP TJA1145有Sleep Mode，可以通过SPI设置其进入该模式。而NXP TJA1143则没有，一般会进入Standby Mode，即伪Sleep Mode。

**Node TypeC：**

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzAbPgJQM0LoibPicVwEUCrKck8IJo80GxhTw6kOfUawBJccBgmDhGia9bMr9auQYDD6nKJicFeqfbyog/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这种类型的节点常连KL30，可以通过硬线唤醒。相对于Node TypeB，Node TypeC在网络休眠时更节能，因为可以完全将Transceiver供电切断。

**Node TypeD：**

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzAbPgJQM0LoibPicVwEUCrKcgiayxxgTROWtgYSQOibhHN29ZuGv7GlF61FTThJYEqlqN7v1H6AWedpg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这种类型的节点不常连KL30，受控于KL15硬线，如果KL15断开则整个主芯片以及Transceiver均断电，而无法工作。相对其它唤醒方式，这种节点的网络启动时间要更长，因为uC和Transceiver均是冷启动。



写在最后



嵌入式Autosar开发中，如果是作为供应商的角色，在拿到OEM或者甲方需求时，要清楚自家产品的唤醒方式，而这需要看一下自家产品的原理图和使用的uC、Transceiver类型才能更好的开发符合需求的Code。