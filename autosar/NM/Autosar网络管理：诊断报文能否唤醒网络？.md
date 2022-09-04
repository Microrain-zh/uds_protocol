# Autosar网络管理：诊断报文能否唤醒网络？

本文焦点：诊断报文能否像网络管理报文那样唤醒网络？对于这个问题，我先给出个人答案：可以。为什么可以？我将从Autosar规范协议和工程问题角度谈一下个人理解。

1

Autosar规范“看”诊断报文唤醒网络

下图CanNM网络状态机，大家应该都比较熟悉，ECU冷启后，由BSM（Bus-Sleep Mode）切换到NM（Network Mode）模式需要调用CanNm_PassiveStartup()或者CanNm_NetworkRequest()接口。一般来说，收到其他节点的网络管理报文，我们归为被动唤醒流程，即：调用CanNm_PassiveStartup()接口，使得节点网络进入NM模式。如果收到的报文是诊断报文，是否可以调用CanNm_PassiveStartup()接口，让节点进入NM模式？个人理解：不能。既然不能，就只能调用CanNm_NetworkRequest()接口切换网络模式，如何理解这个操作呢？

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vylC8EmJ5823gF3V92icUKiaE5mouMTFCw0B8Czt8PWaqjLjIkKHPoywCP2dNlrmITWAIhYE4aBjN8g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们知道，CanNm_NetworkRequest()接口的调用意味着有User主动请求网络通信，所以，诊断报文是否可以看作一个User，且请求网络呢？答：可以。我们看一下Autosar如何定义User，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vylC8EmJ5823gF3V92icUKiaEVXpFayNib2libAxCoPkpHPjJMF8sD9bMDFx77ygbyYer6qbZdt190KdA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**解释**：runnable entities、SW-Cs、 BswM、DCM均可以作为User。既然，DCM可以作为User，可以主动请求和释放网络，那条件是什么呢？继续理解Autosar规范，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vylC8EmJ5823gF3V92icUKiaEemmJhbBcIhAExabk6PtiaoT2DuNLS3auEeds6whDEllDIk0fVZricgTQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

解释，如果要DCM请求(request)网络，需要满足如下条件：

- 诊断会话如果处于默认会话下，诊断需要处于激活状态，ActiveDiagnostic == DCM_COMM_ACTIVE；
- 接收到诊断报文，通过ComM_DCM_ActiveDiagnostic(NetworkId)接口请求对应的Channel处于Full Communication模式。进入Full Communication模式，意味着总线（Bus）通信被requested，**注意**：COMM_NM_VARIANT = FULL，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vylC8EmJ5823gF3V92icUKiaE2rHNWFxjibH6dRlVyYGJcsnhHzkP4ib4mgt1dictiaeWNRibJRQZFuz6VNA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

DCM如何知道已经进入Full Communication模式呢？ComM会通过

Dcm_ComM_FullComModeEntered()接口告知DCM，其请求的Full Communication模式已经进入，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vylC8EmJ5823gF3V92icUKiaEdsVuPTyjicOMDJlqmQLA0VSibia0XOqHdulgoYeh5RnR3VNQIhdxSbHWQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**注意**：ComM进入Full Communication模式不能超过P2ServerMax时间，否则，诊断响应会超时。

ActiveDiagnostic何时进入DCM_COMM_ACTIVE模式呢？答：当DCM模块完成初始化的时候，ActiveDiagnostic进入DCM_COMM_ACTIVE模式，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vylC8EmJ5823gF3V92icUKiaEd1aPV3CQHgve8O7Uk1FygTqOSHnRQJYbxDYuXTiciaL52O2vvPGicX9Ng/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

DCM是如何释放（release）网络的呢？如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vylC8EmJ5823gF3V92icUKiaE41p0Wy3EuB29mAmC9tmUDSrvHlglF6GsibzQ7IkMQG7T40tdflJicnAA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**解释：**

诊断会话如果处于默认会话下，通过接口Dcm_TpTxConfirmation()告知DCM，诊断请求已被成功应答，DCM调用ComM_DCM_InactiveDiagnostic(NetworkId)接口告知ComM，其不再需要Full Communication通信模式。

诊断会话处于默认会话下，DCM请求网络/释放网络的其它节点解释：

- 如果DCM回复NRC 0x78，无需释放网络，即：网络仍然被请求，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vylC8EmJ5823gF3V92icUKiaEjQ733WT32ricogA6a1IceQjZotCzFFj7H6knetkF5KedtendOfL57ow/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

- 如果诊断请求的响应抑制位置位（bit7 = 1）或者功能寻址的否定响应抑制，对于功能寻址，哪些否定响应可以不回，可以参考前文[Uds诊断：这10个诊断问题，你能回答几个？](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247486116&idx=1&sn=22bca31380b532115b7f2a6874f1247c&chksm=fa2a54d0cd5dddc6b6529bf81c9876292811e75bd4345e4139ef2d697778dea1f5220644b386&scene=21#wechat_redirect)。DCM需要释放网络，如下所示。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vylC8EmJ5823gF3V92icUKiaEupgDLn6YkQoYBkfYJ48mhLnpcW9EsxtTeblYdwdytlEodlloTo7OgQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 诊断处于非默认会话，网络处于何种状态

如果诊断会话处于非默认会话下，ComM的Channel处于Full Communication模式，意味着网络一直被请求，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vylC8EmJ5823gF3V92icUKiaEy2zDxyshv0ex6jzicFgqyA501BaZu0IEIRBoPFFJ5ibuYvV9xkfElDgA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2

工程开发思考

## Q1：诊断报文可以唤醒ECU吗

**A1**：之前我多次提到：ECU唤醒是网络唤醒的前提。所以，在思考诊断报文是否可以唤醒网络之前，我们要确认诊断报文是否可以唤醒ECU（ECU被供电）。如果Trcv具有PN（Partial Network）功能，意味着：Trcv硬件即可过滤特定的报文，比如：常用TJA 1145过滤指定的网络管理报文（CanID:0x500 ~ 0x53F，PNC = 0x16 ~ 0x47），即：只有满足CanID和PNC要求的网络管理报文，才能将ECU唤醒，进而唤醒网络。如果不将诊断报文作为过滤源，显然，诊断报文将无法唤醒ECU，也就谈不上唤醒网络。所以，Trcv硬件过滤需要注意此条件。

如果Trcv硬件不支持PN过滤，比如：使用TJA 1044这样的Trcv。只要收到的报文满足Trcv的过滤时序，即可将ECU唤醒，之后由软件实现网络管理报文和诊断报文的过滤，以及网络的唤醒。

## Q2：RSS状态下，收到诊断报文，可以进入NOS状态吗？

**A2**：如果节点的网络处于RSS（Ready Sleep State），收到诊断报文，能否进入NOS（Normal Operation State）状态，需要分情况讨论。

首先，如果节点的网络类型是被动网络节点（CANNM_PASSIVE_MODE_ENABLED = TRUE），即：该节点没有发送网络管理报文的功能，也就没有进入NOS状态的可能。如下的讨论建立在节点为主动网络节点的前提讨论。

**（一）节点没有PN功能**

如果节点没有PN功能（CanNmPnHandleMultipleNetworkRequests == FALSE），当节点收到诊断报文时，DCM这个User，请求ComMFull Communication模式以后，即可调用CanNm_NetworkRequest()接口主动请求网络，进而进入NOS状态，进入NOS状态以后，节点可以发送网络管理报文。如果节点在默认诊断会话下，且诊断被Confirmation，意味着节点的网络状态被Release，网络进入RSS状态。如果节点在非默认会话下，则节点在NOS状态下一直周期性发送网络管理报文。如果S3超时，或者会话再次切换到默认会话，则DCM释放网络，网络进入RSS状态。

**（二）节点具体PN功能**

如果节点具有PN功能，且CanNmPnHandleMultipleNetworkRequests == TRUE，当节点收到诊断报文时，DCM请求网络，节点的网络状态由RSS切换到RMS（Repeat Message State），如果当前诊断会话处于默认会话，则诊断响应被Confirmation以后，DCM释放网络，如果Repeat Message Timer超时，且DCM仍然在请求网络，则进入NOS状态，如果DCM已经释放网络，则再次进入RSS状态。如果诊断处于非默认会话，且S3没有超时，则节点由RMS状态进入NOS状态，节点在NOS状态下发送网络管理报文。

## Q3、节点$11 01/81复位后，节点网络处于何种状态？

**A3**：一般的升级流程中，对于某个节点升级以后，会使用物理寻址的$11 01/81使该节点复位，以便于更新后的App程序生效。而$11 01/81服务使得节点硬复位，更新节点的软件程序会重新初始化，网路状态恢复到BSM模式，且诊断切换到默认会话。此时，节点如果能收到诊断报文，节点虽然会请求网络，请求Full Communication通信，但是诊断请求被应答后，DCM又会释放网络；如果使用$11 81（同理：$3E 80也不请求网络），则节点不会请求网络，这意味着，节点一直在BSM模式。那么，这个节点，此时如何实现网络唤醒呢？

**（一）其他节点唤醒**

节点升级以后，会使用功能寻址将网段内其他节点的通信打开（0x28服务），其他节点发出网络管理报文，即可正常地将升级节点的网络状态切换到预期模式。

**（二）上位机发送NM Msg**

当节点升级以后，也可以通过上位机发送网络管理报文，让升级节点进入预期的网络模式，避免其休眠。如果上位机发送NM Msg，可以简化软件实现的复杂度。如果这样，上位机充当了主动网络节点角色。

**（三）其他节点发送NM Msg**

也可以让进入Bootloader的节点，周期性的发送NM Msg，eg：2s。这样做，有两个用意：

- 识别节点是否处于Boot程序中；
- 节点完成升级后，可以被网络管理切换进入指定的网络状态。

## Q4：Bootloader是否具有网络管理功能？

**A4**：Bootloader存在的主要目的有两个：

1. 引导启动程序；
2. 更新程序。

而且，目前车用的Bootloader程序，多数手写，不会考虑网络状态。

**参考资料**

AUTOSAR_SWS_COMManager.pdf

AUTOSAR_SWS_CANNetworkManagement.pdf

AUTOSAR_SWS_DiagnosticCommunicationManager.pdf