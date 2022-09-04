# [Classic AUTOSAR学习] SecOC通信安全模块（入门篇）

## 简介

SecOC模块的目的是在PDU的级别，针对关键数据作资源高效且可行的验证机制，保证数据安全，这种安全机制可以无缝集成到AUTOSAR项目当中。

SecOC既可以支持对称加密方式，也能支持非对称加密方式，AUTOSAR规范主要基于对称加密方式进行说明。在对称加密过程当中，消息认证码（MAC）是关键，它能以更小长度的密钥以及实现的便利程度，达到非对称加密同样的安全等级。

对于既有项目来说，这样的方式也充分考虑了过去的系统当中所能提供的有限资源，尽量将资源消耗程度降到最低。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ITL4l9Iicic4AbTTSpy7PWn317Pelcp1NwepTXA1h7OhTG1s0JJK0b934b0QxNHR0tzeEeibCTGVXAd3xSpbpAAcw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

SecOC既作为PduR模块的上层模块，也是PduR的下层模块，在数据流的收发双向中做数据的加密与解密。以Can总线上的接收为例，CanIf将接收到的加密信息传给PduR，PduR交给SecOC，SecOC利用CSM的服务和RTE之上的Key management和Counter management进行解密，然后SecOC再将解密后的数据给到PduR，PduR继续路由给对应的模块。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ITL4l9Iicic4AbTTSpy7PWn317Pelcp1NwrgvMk0xecRPvoq7B0eV288fpQ46KnSA1qx1mEb18wfsqUygPhEfAkA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

所以，很显然地，SecOC和底层通信协议无关。

## AUTOSAR中的安全解决方案

确保接收到的数据从正确的ECU发送而来，并且是正确的数据，对于车辆功能系统完成正确且安全的操作，是非常重要的。在这里，涉及到**认证（Authentication）**及**完整性（Integrity）**两个概念。

SecOC模块提供了必要的功能，来验证车内ECU通信的PDU的真实性（authenticity）和新鲜度（freshness）。

在发送方，SecOC为IPdu加上验证信息，组成了Secured IPdu。验证信息包括验证器Authenticator（例如消息验证码，下称MAC）以及可选的新鲜值字段。当使用了新鲜值，而不是时间戳的情况下，新鲜值计数器应当在生成验证信息前，由新鲜值管理器不停地自加并提供这个值。

基于同样验证方法以及新鲜值管理器（下称FvM）的接收方，SecOC模块负责检查Authentic IPdu的新鲜值以及验证码。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ITL4l9Iicic4AbTTSpy7PWn317Pelcp1Nwq0w8dgc6TfsRZjtFaQKn7VSPohiaI9H98dw7uzz1Cjb2v3RVZuNxIMQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

一般地，在使用对称加密时，我们使用术语MAC，在使用非对称加密时，我们使用术语签名（signature）或者电子签名（digital signature）。

## Authentic I-Pdu和Secured I-Pdu

Secured IPdu的payload包括Authentic IPdu以及MAC值，也可以包含新鲜值（可选），并且这个新鲜值会被用来作为计算MAC值的一部分。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ITL4l9Iicic4AbTTSpy7PWn317Pelcp1NwIIWeKvvd53icickUpibqgCvL0jv3SxDiczuMKNR2ejZAhAK6L6MY8gCeTQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

对于不同的Authentic IPdu，新鲜值和MAC值可以设置为不同的长度。

MAC值由密钥，Secured IPdu的数据ID，payload（Authentic IPdu）以及新鲜值作为输入并计算而得，具有唯一性，足以在收发双方进行安全的通信。

由于可能会出现数据长度有限，无法将MAC值每一位都放在消息中进行发送的情况，SecOC模块也可以设置截断值，将MAC值截取一部分放在payload当中。不同的Secured IPdu都可以设置不同的截断值。

对于MAC值，应当截取高位数据，而对于新鲜值，截取的是低位数据。

截取位长度该怎么选，这是个问题。从规范上的说明来看，理想状态下，最好是有至少128bit长度的MAC值，不过，一般情形下，64bit以上也能覆盖大多数场景，不会被攻破。当然，具体怎么截取，需要由项目的信息安全负责人来确定。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ITL4l9Iicic4AbTTSpy7PWn317Pelcp1NwHCQiajffoFWEqZnpPlIKrHPXrj3aAAthR2icDc1iaMia6WjTBLedrYbF3w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

无论是新鲜值，MAC还是Data ID等，SecOC模块都是基于大端模式来处理数据。

### 用来计算验证码的数据

上文已经提及到，这些数据会被用来计算验证码：Data ID，（被保护的）Authentic IPdu数据，完整的新鲜值。

### 新鲜值

每一个Secured IPdu都可以配置至少一个新鲜值，新鲜值用来保证Secured IPdu的新鲜程度。新鲜值需要有新鲜值管理器作为一个SWC来提供，既可以是消息计数器，也可以是时间戳。

如果想要配置截取值，可以配置SecOCFreshnessValueTruncLength，如果此项配置为0，那么新鲜值将不会被包含在Secured IPdu内。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ITL4l9Iicic4AbTTSpy7PWn317Pelcp1NwkdSnl0JeRK9EoyxhhGXibx3PlGrCKKBC9TR07KNWjcboqH5P1IhlM4A/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

在降低总线负载，进行新鲜值截取的同时，又不希望新鲜值重复导致丢弃本应该接收的消息，我们还可以来利用SecOCUseAuthDataFreshness选项，让Authentic IPdu的部分内容作为新鲜值的一部分。我们可以设置对应起始位(bit)以及长度(bit)----比如Authentic IPdu中有4bit的E2E Counter---作为新鲜值的一部分，这样就可以不用对原有的新鲜值做扩展。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ITL4l9Iicic4AbTTSpy7PWn317Pelcp1NwVgyBW9wKjiceu3kvochv27Eoha4ERgofbb6DgzQ0eaq48BbluO1asMQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

如果设置SecOCAuthDataFreshnessStartPosition为11，SecOCAuthDataFreshnessLen为4，假设收到的PDU数据为0xABCD，那么它的**Authentic Data新鲜值**为0xD。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ITL4l9Iicic4AbTTSpy7PWn317Pelcp1NwjTYY2BDtV4F73jqoNzYlYlPoj8Q4Ad0C4G2EbYY5bLfOAje4xibN4KA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

获取新鲜值有两种方案，一种是经由RTE，通过FvM SWC的FreshnessManagement_GetTxFreshness()来获取，另一种是直接调用C API接口SecOC_GetTxFreshness()：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ITL4l9Iicic4AbTTSpy7PWn317Pelcp1NwfFSvLWhcddLKEAIBy9m8yxribmsXxCSvAnXY13s45rSwmxErVaHN6hg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

## Secured IPdu的创建

创建一个Secured IPdu分为以下六步：

- 准备Secured IPdu，分配所需buffer
- 获取待构建数据，也即Data ID，Authentic IPdu还有新鲜值
- 生成验证码
- 构建Secured IPdu
- 增加新鲜值
- 发送Secured IPdu

## IPdu的验证

Secured IPdu的验证也分为六步：

- 解析Authentic IPdu，新鲜值和验证码
- 从新鲜值管理器获取新鲜值
- 获取待验证数据
- 检查验证信息
- 给新鲜值管理器发送确认
- 将Authentic IPdu传给上层

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ITL4l9Iicic4AbTTSpy7PWn317Pelcp1Nwbw0ERZTZAzoIROibmUJVFslgMJLAalSYV0AGV9iajKw8A4zuqRuRp1WA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)MAC验证

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ITL4l9Iicic4AbTTSpy7PWn317Pelcp1NwLNDo38vXCIXyibpXgXnUfwbQIY114iaeUScbwQ3M5NydTCr7WZcrsUjw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)新鲜值验证

根据配置的不同，验证结果可以通过Verification Status Callout API接口或者服务接口SecOC_VerificationStatusIndication通知出去。

## 如果是非对称加密呢？

在使用非对称加密的场景当中，密钥分为私钥和公钥。发送方使用私钥生成签名，接收方使用公钥来验证接收到的数据。接收方无法拥有私钥，也无法通过公钥获取私钥。

在验证阶段，接收方需要完整的签名信息，因此之前提及的MAC截取，在这种场景下是不能使用的。

另外，对称加密通信时，接收方的SecOC会请求重新计算MAC值并与接收到的MAC值作比较；而对于非对称加密时，所有数据作为输入给到验证器，通过一个布尔结果值知晓验证结果。

