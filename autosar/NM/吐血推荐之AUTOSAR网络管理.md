# 吐血推荐之AUTOSAR网络管理

言：最近正好在学习CAN总线的AUTOSAR网络管理，前期踩了很多的坑，总结了一下最近所学和大家一起学习。学的很浅，有不正确的地方请各位前辈同仁不吝赐教~

## 1、什么是AUTOSAR？

**官方一点**：AUTOSAR 就是AUTomotive Open System ARchitecture的简称，中文翻译就是汽车开放系统架构。

**直白一点**：将汽车电子控制单元（ECU）的软件底层做了一个标准的封装。使得大家都能共用一套底层软件，只需要修改其中的一些参数，就可以匹配不同硬件，也可以匹配不同的应用层软件。如此之后，用户只需要专心负责应用层功能开发即可，底层都交给AutoSAR工程师就行了。

**再直白一点**：“就是一套写的比较好的底层软件”。其实现了硬件驱动的封装（类似于STM32的库），实现了操作系统的功能。用户只需要开发操作系统上层的软件应用即可（类似于基于安卓开发App）。

**再再再直白一点：**各个厂家在五花八门的硬件上随意开发，想怎么写就怎么写，怎么爽怎么来，导致开发一时爽，维护火葬场，如果底层硬件换掉了，上面的代码基本就要全部推倒重来，而且不同厂家之间的代码移植性也几乎没有，各个厂家和工程师都很头大，于是AUTOSAR应运而生。AUTOSAR将各个硬件的底层接口做了封装，以后如果换硬件，只需要配置一下AUTOSAR，告诉它我换硬件了，赶紧给我适配就可以了，上层代码完全不需要改动就可以使用。从开发的角度来讲，提高了代码的复用性，降低了代码的复杂度，提高了代码的可维护性。

## 2、什么是网络管理？

网络管理的目的是使网络中的ECU节点有序的睡眠和唤醒。在没有通信需求的时候睡眠，在需要通信的时候唤醒，可以节约汽车电池的电量。

## 3、什么是CAN总线？

这个CSDN和知乎都有很多的介绍，这里就不赘述了。

## 4、CAN总线的AUTOSAR网络管理报文（以下简称NM报文）长啥样？

首先要明确一点，NM报文就是CAN报文。NM报文符合CAN报文的格式，由帧起始、仲裁场、控制场、数据场、CRC场、应答场、帧结尾组成。

一般厂家在设计的时候会规定好NM报文的ID范围。

举个例子：规定标识符在0x500到0x5FF范围为NM报文。当在CANoe中抓取到此ID范围内的报文，那就是NM报文。



![图片](https://mmbiz.qpic.cn/mmbiz_jpg/fOVQt3pZrMb0EicMpEXxeGvjRAicqs0BUu7qZjMqGtfNFLy44BkUHFMCREw77HSOZ9wTZ2pWew6BnibC0WXwBMO0Q/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fOVQt3pZrMb0EicMpEXxeGvjRAicqs0BUuL0we4iboEcpiciclNfYRcSDcUb2Yolv6OXQSMfDN6cxI8pcaXWHP4BF6g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



NM报文的重点在于数据场8字节里的内容：



![图片](https://mmbiz.qpic.cn/mmbiz_png/fOVQt3pZrMb0EicMpEXxeGvjRAicqs0BUutHEbuvEkdxAcH9tKckP3xstcz78pZxPFHslFGd5gC9WZY6dibQvq68Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**Byte0**：这里填的是ECU的地址，或者叫ECU的ID；

此报文的ID=一个基础值+ECU的ID，例如厂家规定基础值为0x500，那么此报文的ID=0x500+0x8=0x508；

这里要注意区分报文的ID和ECU ID的概念，很容易混淆；

**Byte1**：



![图片](https://mmbiz.qpic.cn/mmbiz_png/fOVQt3pZrMb0EicMpEXxeGvjRAicqs0BUuFbh210xg0eWWsQeLGrIonz6SwxKkfNdtruBSYGFIkIz7zfMGzr2otQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



这里关注下bit0和bit4：

bit0：当此位置1时强制进入RMS（下面会讲到）；

bit4：告诉其他节点自身是怎么被唤醒的。

置0：被动唤醒、远程唤醒，比如被其他节点发送的NM报文唤醒；

置1：主动唤醒、本地唤醒，比如给ECU上电；

**byte2-byte7**里的user data数据由用户自行定义。

## 5、CAN NM状态介绍

AUTOSAR网络管理有三种状态：

睡眠模式（Bus-Sleep Mode）：当节点没有本地网络唤醒以及远程唤醒请求时，ECU通讯控制器切换至睡眠模式，ECU功耗降低至适当水平；此模式下，NM报文**只收不发**，APP报文**不收不发**，当出现有效唤醒源时**必须要被唤醒**；

预睡眠模式（Prepare Bus-Sleep Mode）：这个状态是为了等待总线上的所有节点能够在进入Bus-Sleep Mode之前有时间停止节点的active状态（如清空队列中为发送的报文）；此模式下，NM报文**只收不发**，APP报文**不收不发，**如果缓冲区有APP报文那可以继续发完；

网络模式（Network Mode）：

包含3个子状态：

重复报文状态（Repeat Message State）：NM报文可收可发，APP报文可收可发；

正常工作状态（Normal Operation State）：NM报文可收可发，APP报文可收可发；

准备睡眠状态（Ready Sleep State）：**NM报文只收不发**，APP报文可收可发；

总结见下图：



![图片](https://mmbiz.qpic.cn/mmbiz_png/fOVQt3pZrMb0EicMpEXxeGvjRAicqs0BUuR1tkhziauEf6tGNSuzicicF12PlRSEZ2N7VRwdBtjcet6YtY9ACiaCHAIQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## 6、定时器及参数介绍



![图片](https://mmbiz.qpic.cn/mmbiz_png/fOVQt3pZrMb0EicMpEXxeGvjRAicqs0BUuDhQmcAdzjTRWficNYJ7E7mWeCG8tjoz4GibYzSbBkCFjgD85r9Lliac2g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



第5小节和第6小节的内容看一遍可能理解不了，学完下面的状态迁移图，再回过来多看几遍就能理解了。

## 7、状态机



![图片](https://mmbiz.qpic.cn/mmbiz_jpg/fOVQt3pZrMb0EicMpEXxeGvjRAicqs0BUuE5LedyLrbT5zKUKtSTkSNZf0wIOUibJBf9kZWHPrThBica3S2BsF8DGQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)



现在终于来到AUTOSAR网络管理的最难理解也是最容易使人秃头的状态机了，这里我不打算把每一条状态转换的文字描述直接贴上来，跟着我的思路，我们来一个一个看吧。

在开始之前，先了解一下各种缩略语：

> BSM-睡眠模式  NM-网络模式  PBM-预睡眠模式  
> RMS-重复报文模式  NOS-正常操作状态  RSS-准备睡眠模式

**01**：给ECU上电，ECU自己就会初始化进入睡眠模式。如果没有唤醒源来唤醒此节点，那就会一直待在睡眠模式。

**02+03**：当出现本地唤醒（03）或者远程唤醒（02）时，进入RMS状态。这里再解释下，本地唤醒就是我自己想要**主动**和其他节点通信；远程唤醒是其他节点想要和我通信。

**04**：我们现在已经走到网络模式的重复报文子状态了。话说为什么叫重复报文子状态呢，因为在这个状态里的时候，ECU需要一直发送周期报文，来告诉别人：我在线，性感ECU在线陪聊，你再不来找我我就要开始想念你......

如果是走03（本地唤醒）进来的，那么需要先在NM Immediate Transmit State中以很快的周期发送N帧报文（例：以20ms的周期连续发送5帧报文），发完这N帧报文再进入到NM Normal Transmit State中以正常的周期发送报文（例：500ms为周期发送报文。这个在上面的表格里有定义）。如果是直接走02进来的，那么直接以正常周期发送NM报文就可以了。一直发到T_repeat_message定时器超时。

这一步的目的是如果是本地唤醒的话，可能此ECU下面还有很多从属节点，当此ECU唤醒之后，需要同时唤醒其他兄弟节点一起通信，所以最开始的N帧报文周期很短，目的是为了快速、低延迟地唤醒其他节点。为什么被远程唤醒就不需要这一步呢？欢迎大家在评论区里一起讨论~

**06+12**：且慢，我们先来计算一下从BSM到这一步花费了多少时间了。参考上面定时器的定义，在02或03中，最大唤醒时间为T_wake_up=200ms；在04中，T_repeat_message=1600ms。总计1800ms，差不多为2s的时间，此时ECU有可能已经不需要通信了（2019-11-29补充：ECU持续处于唤醒状态的条件是有持续的唤醒源，例如一直有NM报文远程唤醒、或一直有本地唤醒源例如上电）。如果还需要继续通信，走06，进入NOS，继续周期发送NM报文，可以收发APP报文，当不再需要通信了，就停止发送NM报文，等待T_NM_timeout超时之后走09；如果直接不需要通信了，直接走12。

**10**：收到本地唤醒，进入NOS。

**11**：收到NM报文的byte1字节的重复请求位如果置1，强制进入RMS。

**08+14+05**：T_NM_timerout定时器超时，不改变当前状态。定时器需要重置。

**13**：在RSS状态，NM报文不可以发送。等待T_NM_TIMEOUT定时器超时后进入PBM。

**15+16**：PBM状态只可以接收NM报文，其他报文不发不收。收到远程唤醒，走15；收到本地唤醒，走16。

**17**：如果PBM状态收不到任何唤醒源，在T_WAIT_BUS_SLEEP定时器超时后进入BSM。

以上就是CAN总线AUTOSAR网络管理的内容分享。之前写这篇文章的时候只是理论学习，最近在实际测试的时候，对AUTOSAR网络管理有了一些新的认识。

**DUT在RSS状态的时候，如果收到本地唤醒（如KL15电闭合），会走NM\*11进入RMS状态；\*那如果收到远程唤醒报文呢？**

根据主机厂的设计不同，可能会有下面两种action：①在RSS收到远程唤醒报文，不会发生状态跳转，但是会重置T_NM_TIMEOUT定时器，即：在RSS状态收到持续的远程报文唤醒，会一直保持在RSS状态，此时DUT不能发出NM报文，但是可以收发APP报文；②在RSS状态收到远程唤醒报文，会重新进入NOS。