# Autosar网络管理：节点外发第一帧网络管理报文处理策略，再说一次

提示：基于Can总线讨论。

## 1、如何确保节点外发第一帧报文是网络管理报文？

**答**：在回答这个问题之前，我们需要清楚：**节点可以发送网络管理报文**，**说明此节点的网络类型不是Passive的**，这个前文有说明，可以回顾[Autosar网络管理：基础问题知多少](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247486784&idx=1&sn=bf632b1d953d812193a355a7b4e85b1e&chksm=fa2a5334cd5dda220361c955fa9f42850979a105977c0a97a8ae5fb5327bbbbd8e06c194f215&scene=21#wechat_redirect)。

首先，我们要清楚导致Node外发第一帧不是网络管理报文的原因，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzNCQmfJ3iaRnDatwwYysLlPOmvVz4fqh1LnOzOSXbKl8bUkXy4vQrUrfJnibOHegt8Tl0tWibffdD9Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如上：假设应用报文所在区间0x00~0x4FF（一个节点一般包含多个应用报文），网络管理报文CanID所在区间：0x500~0x5FF，当前节点的网络管理报文CanID = 0x500。

**具体解释**：当ECU被加电或者程序复位（t0时刻），完成初始化动作和准备工作，在t1时刻允许通信，此时如果应用报文和网络管理报文都请求发送，应用报文（这里主要指周期性应用报文）会被优先发送，因为应用报文的CanID优先级高于网络管理报文的CanID。

**解决策略**：既然应用报文的优先级高，那么在第一次周期性应用报文发送的时候，将其Offset一段时间，让**网络管理报文在t1时刻发送**，**应用报文在t2时刻发送**即可避免该问题的出现，具体需要配置哪些模块参数，可以参考前文：[Autosar网络管理PN：路由超时大Bug](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247484944&idx=1&sn=796eb70b20d845359f5e019656773d1a&chksm=fa2a5864cd5dd17285e66df13316d623d54c04647e9a19bd697311df110883dbf8cb7a91a0ea&scene=21#wechat_redirect)，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzNCQmfJ3iaRnDatwwYysLlPTOgiaia6ibl2GtneLmXyPaUIgJtD6SAHOVrD3ks9ms96KaHHBxykhbx9g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如上处理就万事大吉了吗？不一定，假设ECU3的网络管理报文CanID = 0x504，在其发送网络管理报文的时候，可能由于其网络管理报文CanID优先级低于其他节点网络管理报文的CanID，而导致其仲裁失败，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyPic7iapP5dwib7WklM6GntDcNOguppDYg6EBiaDlPCP7QRGLQZIAWFeEDR72wN5qqePlXicWnGIbR6Qw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

当然，也可能由于其他某些工况导致当前周期的网络管理报文发送失败，所以，为了确保网络管理报文的发送成功，需要确保其在当前周期发送失败以后，在下一个轮询周期(Task)内，再次尝试网络管理报文的发送，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzNCQmfJ3iaRnDatwwYysLlPricEe6ZD2SOicNtHSVIVsGIuruBK64X8waT7VpU5bFlYSH4AVRW3BKiaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**提示**：在CanNM模块，配置CanNmRetryFirstMessageRequest参数即可实现NM Msg的重发。注意，如果当前节点的网络类型是Passive的，CanNmRetryFirstMessageRequest参数配置无效。



![图片](https://mmbiz.qpic.cn/mmbiz_png/quuOyCqONwdD16hI11wAQWxGp4ajJ4DMnbsHGb4ViandFryeibcQb1Idxb3MHmrh20988OSES3OU1wPTicQbAr94g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



参考资料

SIMPLE TITLE

AUTOSAR_SWS_CANNetworkManagement.pdf