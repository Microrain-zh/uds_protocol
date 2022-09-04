# Autosar通信：网络管理报文和应用报文，谁先发？

汽车嵌入式开发中，具有**周期性**外发的报文目前只有网络管理报文和应用报文，当通信开启的时候，这两种类型的报文谁先外发呢？本文从需求角度，聊聊这两种报文到底该谁应该先发送，谁应该后发送。

1

IpduGroup概念

为了便于对应用报文的管理，COM层将发送和接收的I-PDU进行分组处理（分组情况根据需求配置）。COM通过Com_IpduGroupControl()接口具体的控制I-PDUs的收/发行为。

Com_IpduGroupControl()函数原型如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzW0KUgCslNz5D9iaXPxvnwsPnTeY9ibsJib2hWDSxnXzzmLVANgtQWpn8651xkMPHEERqHhjEbHodHA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

以控制Tx PDU为例，ipduGroupVector、IpduGroup、I-PDU关系如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzW0KUgCslNz5D9iaXPxvnwsCXKh5MwdUwibwG3h1ngndghjNRaf5Ds4sLWibk59Bfr5tGEBx3yObSqA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

再看看Autosar怎么描述IpduGroup、I-PDU关系的，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzW0KUgCslNz5D9iaXPxvnwsWlCIYGTFTX8g4kjQMo5DwxYKbS5ZjvDybr06yuLmbNwKSMHF5YD7TA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

有点晕？没关系，咱们简单说说：

一个I-PDU可以放在一个IpduGroup里，也可以放在多个IpduGroup里；

一个I-PDU既可以放在Rx IpduGroup里，也可以放在Tx IpduGroup;

一个I-PDU所在的**任意一个**IpduGroup激活，此I-PDU的收/发行为被激活。

2

网络管理报文和应用报文谁先发？

首先，需要明确当前Node的网络类型，如果当前Node的网络类型是Active的，Node可以外发网络管理报文，那么就需要明确一下两者谁先发送的问题了。为了快速的唤醒网络，需求中多数会要求“**外发的第一帧报文是网络管理报文**”，且会有时间约束，比如：150ms＋10%。这个话题前面我们有提到过，可以回顾前文[Autosar网络管理：确保第一帧是网络管理报文方法](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247485362&idx=1&sn=36ba60564be3450103d2575922f3e1d4&chksm=fa2a59c6cd5dd0d004ff096c6127049f345227e20b7843f0d2f8638f9ade44fe205d2f872b9a&scene=21#wechat_redirect)。

如果当前Node的网络类型是Passive的，意味着本Node不会外发网络管理报文，只能接收网络管理报文。所以，此种情况，就**只有应用报文先外发**，即使是应用报文，第一帧报文的外发时间也是有约束的，比如：100ms＋10%。刚才我们提到Com_IpduGroupControl()控制应用报文的外发行为，当然COM层也管控着应用报文的外发时机，即：配置参数ComTxModeTimeOffset。不要忽视此参数的配置，如果ComTxModeTimeOffset ＞0，意味着应用报文的外发时机要Offset，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzW0KUgCslNz5D9iaXPxvnwsGjF3pZ14jawro7jRsQkhFBYN1xWIcvcBxB6N2tr8vdc3u9nKoVgMYw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

ComTxModeTimeOffset具体作用如下所示，可以看出，ComTxModeTimeOffset作用于PERIODIC or MIXED两种类型的应用报文。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzW0KUgCslNz5D9iaXPxvnwsBRgmEwDJ3YJTZa7MD4NuLhKZ8GcMeH41FW4CHBRtP9ItS6C5nUlz4g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

当ComTxModeTimeOffset ＞ 0时，会延迟Com_MainFunctionTx()的调用，进而使得第一帧应用报文的外发时间延长。所以，在实际项目的开发中，嫌少使用该参数，或者将其配置为0。

**如果遇到外发第一帧应用报文时间超时，可以看一下该参数的配置情况**。

这里还有一个问题我们没有解释：Com_MainFunctionTx()对应的ComTxModeTimeOffset谁来计时？既然Com_MainFunctionTx()需要周期性调度（比如：5ms调度一次），就会被OS管理，如果Com_MainFunctionTx()所在Task想延迟一段时间激活。可以，OsAlarm粉墨登场，ComTxModeTimeOffset延迟时间由OsAlarm精准把控，比如：延迟10ms，激活Com_MainFunctionTx()所在Task，那么周期性应用报文的第一次外发将会被延迟10ms。

TaskOsAlarm作用如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzW0KUgCslNz5D9iaXPxvnwshp4qN6pZiaibIav9Izuvp6KicM1tKk9JJoN2uUDpXVXkCHz7ATxVVbX3w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 拓展思考

前面我们讨论过：“如何确保外发第一帧报文是网络管理报文”。同样，我们可以利用ComTxModeTimeOffset参数，延迟Com_MainFunctionTx()所在任务的激活时机来实现该功能。注意：需要先确定使用的软件包是否具有该功能，因为并不是每家Autosar供应商都实现了该功能。

**参考资料**

AUTOSAR_SWS_COM.pdf

AUTOSAR_SWS_OS.pdf