# CAN总线指定帧唤醒的硬件实现方式

CAN的指定帧唤醒是一种网络管理的场景，对于我这个偏硬件的工程师来说，网络管理也就是通过CAN来唤醒不同的ECU，而指定帧唤醒就是特定的某些CAN ID的报文能够唤醒ECU。

讲到CAN总线就必须要涉及到跟CAN总线相关的稍微“高级”一点的用法，那就是指定帧、任意帧唤醒的一点知识。最近也是接触到了一些可能需要同一个网络上某些节点唤醒而另外一些节点不用唤醒的案例，但在分配网络管理帧的时候依然遇到一些问题，针对这种最经典的应用案例，在没有广泛了解CAN ID如何分配的前提下，我觉得可能有解决方案，正所谓不知者无畏。



**一 指定帧唤醒的硬件基础**



从目前应用最为广泛的带指定帧唤醒的CAN收发器TJA1145的管脚定义如下，其中跟唤醒相关性最强的就是INH脚，规格书上这个注释直译是“禁止输出以切换外部稳压器”，其实不用整这么麻烦，它的用途就是当CAN总线上有唤醒帧的时候，INH会置位变成高电平可以用来使能外部的电源芯片。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3vPe9ic1J8QDphpOt5QnlpBWZbKmwmzyNbAKRoqr6fffpOuTKGzkF5VCXanAhD52TxpNquW2NNz28AicHo7Dnic1g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

从TJA1145的芯片内部示意图里面可以看到大概的用途，当报文过滤器的的报文与唤醒帧寄存器相匹配的时候，COMPARE LOGIC就会认为检测到唤醒帧，然后就会闭合INH内部的开关，让INH脚输出12V。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3vPe9ic1J8QDphpOt5QnlpBWZbKmwmzyNlInNf31JaQe45C09FbQsuibj9npZPyichyvRr4JqKyffb4Z1Mibx9S1OQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

所以从上面看来，CAN唤醒需要硬件配合来实现才行，下图就是比较典型的一种网络管理唤醒的硬件拓扑，首先带唤醒的CAN收发器必须要12V常电供电，另外INH脚需要连接到电源芯片的使能脚，这时当CAN总线上有网络管理帧的时候，INH变成高电平去唤醒电源芯片，就完成了一次完整的网络管理唤醒。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3vPe9ic1J8QDphpOt5QnlpBWZbKmwmzyNkjKLaTCiaicdIW1kAjes0r2ickCedmIjZTtph6RfTiajFrzRv4DMjnHGHw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

下图这个是TJA1145芯片手册中推荐的应用电路，基本上跟我画的拓扑差不多，如果有兴趣的话可以直接去网上下载TJA1145的芯片手册去了解一下。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3vPe9ic1J8QDphpOt5QnlpBWZbKmwmzyNo0hicFFExrP1nRAO8cCr59x1Lz0ZzxPod1URaAgB5hZC019McqQ67pA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**二 指定帧唤醒的配置方式**



在芯片内部框图可以看到有一个Wakeupframe configuration memory，这个寄存器就是用来配置唤醒报文的。之前我也讲到过CAN报文的格式，其中CAN ID是11位，也就是从0x000~0x7FF这个范围。一般来说定义网络管理帧是各个主机厂自己定义的，常用的包括0x4xx，0x5xx，0x6xx，0x7xx都是有人用的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3vPe9ic1J8QDphpOt5QnlpBWZbKmwmzyNrHEPmvVl3GohUZpxFefZnUQBVyPBq3CsoqiagyDug3F5AdRFFWVtILA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

对于配置指定帧的寄存器，分为两个部分，一个是11位的CAN ID区域，一个是11位的ID mask区域。用通俗一点的语言就是CAN ID区域就是用来标注制定帧的具体唤醒ID，而mask区域与之相对应的位里面，如果是0，就表示对应的ID那一位是需要必须满足的，如果是1，就表示对应的ID那一位可以不用关注。因此在规格书上的这个例子就是表示唤醒帧是00110100xxx，后三位xxx可以是0也可以是1，所以网络管理唤醒帧的范围就是从0x1A0到0x1A7。

还是上面这个例子，如果ID mask中放开的位数只有1个，那就表示只有2个ID的报文才能唤醒CAN收发器。假设ID mask是0000000 0100，那对应的制定CAN ID就是0x1A4，0x1A0。我们如果把这2个ID分配成一个收，一个发并且给到同一个ECU，这样的话，我们就能够实现精准的网络管理唤醒，对于同一个网络的不同节点，虽然都支持指定帧唤醒，但是我依然可以用不同的网络管理帧来实现不同的唤醒需求。



**总结**



   当然这个只是从理论上来说一下网络管理唤醒的理想状态，在实际应用过程中，同一个CAN总线上不同的节点之间一般都是存在相互通讯的需求，只唤醒某些节点必然会导致其他节点校验出来报文丢失的故障，因此实际应用中，配置网络管理通常是直接配置一个网段，例如上面我说到的0x4xx，0x6xx这种。