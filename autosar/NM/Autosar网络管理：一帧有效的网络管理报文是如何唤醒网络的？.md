# Autosar网络管理：一帧有效的网络管理报文是如何唤醒网络的？

前面我们聊过唤醒源的Check和Set，可以回顾前文[Autosar网络管理：唤醒源的Check和Set](http://mp.weixin.qq.com/s?__biz=MzUyNDU4NTc1NQ==&mid=2247486550&idx=1&sn=7005420549c282d303587799ad9f5a8a&chksm=fa2a5222cd5ddb34affecc231964686d81efbc6a7420f2437c56dbd5e42b7fd62a8eb3d6e8cf&scene=21#wechat_redirect)。本文我们接着探究一帧有效的网络管理报文是如何把网络唤醒的。

**提示**：基于Can总线讨论

1

Set唤醒源

唤醒事件即可以使用Trcv(Can Transceiver)检查，也可以使用Controller检查,检查方式有中断、轮询两种方式。

假设:采用Trcv(Can Transceiver)中断方式检查唤醒事件，当Trcv接收到一帧报文时，与Controller相连的Rx Pin拉低，触发ICU中断(设置下降沿或者双边沿触发)，之后进入ICU中断处理程序。

**提示**：如果Trcv没有设置过滤功能，任何一帧报文均可以使能Trcv控制的SBC，进而唤醒ECU，甚至符合Wakeup-Pattern的总线扰动也能唤醒ECU。如果Trcv(硬件)设置了过滤功能，则可以实现特定的网络管理报文唤醒网络。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyBErk8Xq2emA8vVdRd2Kk3DtITiaziaaIuxrfruoIBjicl30KT6p5KR1owj6TFTniaUyumARJjhpD7ibA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

通过上图，我们可以看出：Trcv通过调用EcuM_SetWakeupEvent(EcuM_WakeupSourceType)接口告知EcuM，是谁唤醒ECU，唤醒源在开发阶段事先配置，为每一个唤醒源分配一个句柄（identifier）。Set的主要目的是把唤醒事件挂起，以便EcuM(**EcuM_MainFunction()**)后续检查唤醒事件的有效性，唤醒事件的检查中，可以不Check，但是不能不验证唤醒事件的有效性(Validation)。唤醒事件的挂起操作如下：

- 
- 

```
/* #34 Add this source to the global pending wakeups variable. */EcuM_PendingWakeups |= WakeupSource;
```

将唤醒事件在一个全局变量中置位(一个唤醒源对应一个Bit)。

2

EcuM如何Validate唤醒事件

为便于细节的展开，我们将Validate分为3个Part:Part1、Part2、Part3，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyBErk8Xq2emA8vVdRd2Kk31nQMvfhyAkjrhRAHUKqicc0Ix69DeLX85icUicuMYYVicu7cAPTzbW2rIg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## Part1

在Set部分，已经通过一个全局变量EcuM_PendingWakeups将待验证的唤醒事件缓存，在EcuM_MainFunction()中即可对待验证的唤醒事件进行验证。在验证唤醒事件之前，需要将Trcv和Controller切换到工作模式，而两者状态的切换，通过EcuM_StartWakeupSources()即可完成，EcuM_StartWakeupSources()进一步调用CanSM_StartWakeupSource()接口。

验证过程不能遥遥无期，因此会设置一个验证超时时间参数EcuMValidationTimeout，比如：3s等。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyBErk8Xq2emA8vVdRd2Kk3at7vYcOriaf9EyumQ6DTereueBeSw5rhiaKkQNGU9cKuxUFJibfZpdEvw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## Part2

至此，真正对唤醒事件的验证才开始，和Check一样，谁发现谁验证。还是老规矩EcuM_CheckValidation()->CanIf_CheckValidation()->EcuM_ValidateWakeupEvent()。这里将WakeupSource = CanIf，不同软件供应商，此处实现方式可能有所不同。

**注意**：一般CanIf_CheckValidation()通过**回调函数**调用EcuM_ValidateWakeupEvent()。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyBErk8Xq2emA8vVdRd2Kk3BcRBbRGwWbxBvzKZtFynIdzn5rLiasdGwxReu6aicxBhsC8niaxUjJUYw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如果唤醒事件有效，EcuM有两个工作要做：

- 通过ComM_EcuM_WakeUpIndication()告知ComM唤醒事件有效
- 通过BswM_EcuM_CurrentWakeup()告知BswM唤醒事件有效

EcuM具体如何告知ComM和BswM呢？

如果BswM还没有初始化，EcuM将此信息通过一个全局变量缓存，eg:EcuM_BswM_BufferedWakeups。如果EcuM运行状态＞ECUM_STATE_STARTUP_TWO，则调用BswM_EcuM_CurrentWakeUp()接口。

而对于ComM，如果ComM初始化完成，则调用ComM_EcuM_WakeUpIndication()接口；如果ComM未初始化完成，也会通过一个全局变化缓存此信息，eg:EcuM_ComM_BufferedWakeups。

## Part3

此部分和Part2是一个if..else..关系，此部分表示唤醒事件无效后的处理动作，如果唤醒事件无效，EcuM会将此信息告知BswM，让BswM执行对应的ActionList。

同时，EcuM调用接口EcuM_StopWakeupSources()去切换Trcv和Controller的状态，即关闭两者，之后对应的报文无法接收。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyBErk8Xq2emA8vVdRd2Kk3IicNNQHCJlCt0U8J1FcZydJU1ibmMiaHWrOh81uX7BhYOxGwQNvHJxyFg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

3

ComM状态切换

在Part2中，唤醒事件有效以后，会调用ComM_EcuM_WakeUpIndication()接口，之后ComM状态如何切换呢？

ComM对应的通道会切换到COMM_NO_COM_REQUEST_PENDING子状态，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyQoajZVskFiawRYIInwwFWL7U8dDFTwqsPPB88pnmuoAP2EECMn8vsWiaPfmO5DRjqaGuhuauwb8Bg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如上可以看出：ComM对应的通道会切换到COMM_NO_COM_REQUEST_PENDING子状态，存在多个or情况，收到网络管理报文也属于其中一种。

当没有User主动请求通信时，唤醒过程被认为是Passive Wakeup过程，比如：**收到网络管理报文就属于Passive Wakeup过程，**如果CanNM模块在Bus-Sleep Mode收到网络管理报文，会调用Nm_NetworkStartIndication()通知NM模块，之后NM模块通过ComM_Nm_NetworkStartIndication()接口告知ComM。被动唤醒时，EcuM、ComM和NM的交互时序如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyBErk8Xq2emA8vVdRd2Kk3fF9iaw1tlGmXyHqTI8TOyxMPZQyIAQ4IZPIL1RCicic5w8c3XUfyicicHGw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

注意：

ComM从COMM_NO_COM_REQUEST_PENDING子状态想要进入COMM_FULL_COM_NETWORK_REQUESTED，需要满足两个条件：

1. 通信允许，即CommunicationAllowed = True
2. CanSM请求COMM_FULL_COMMUNICATION

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyBErk8Xq2emA8vVdRd2Kk3X9x2uSEpoTbDukOFa9NqThpzEwPnB3sC8XHBO7yjia9JPzibyXOuSWyg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 谁控制CommunicationAllowed = True

既然需要满足CommunicationAllowed = True，那谁来控制这个条件呢？对于Flexible EcuM，每个Channel的CommunicationAllowed条件由BswM调用ComM_CommunicationAllowed()接口控制。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyBErk8Xq2emA8vVdRd2Kk3Weibovd9gwl56bDG7aS0AabRyRR0satsmhK01rJQBYrb3Lvfic3oUBZg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们知道，BswM干活有规矩：先模式仲裁(Arbitration)，后执行对应的动作(ActionList)，两者配合如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyQoajZVskFiawRYIInwwFWLPsYoZl00Sy9zfyfQObgn0NsPR0cVG2bIZxWvPdP4DVPJm7rpjF8ejQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

模式仲裁请求包括：Requests和Indications，而BswM_EcuM_CurrentWakeup()接口的调用，属于Indication，意味着EcuM告知BswM，**有效的唤醒事件已确认**。同时，**检查是否收到网络管理报文**（如果Trcv具有Filter功能，即识别网络管理报文，则唤醒事件有效 等价于收到网络管理报文，软件即可不用处理），如果这两个Rule满足，则允许通信，之后BswM执行对应的ActionList，其中就包括ComM_CommunicationAllowed()接口的调用，即设置CommunicationAllowed = True。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyQoajZVskFiawRYIInwwFWL2NAIySEHYzonfaYD44eJIg1QLe1jkRibqJdyaOWOQAosAL7tydaNyEA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 谁请求COMM_FULL_COMMUNICATION 

当ComM对应的Channel允许通信以后，且EcuM在RUN Phase时，进入

COMM_FULL_COMMUNICATION状态，谁请求COMM_FULL_COMMUNICATION呢？如果回答了这个问题，就意味着接下来**通信COM、网络NM**对应的报文可以外发，即：对应节点的网络被一帧网络管理报文唤醒了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyQoajZVskFiawRYIInwwFWL6Zu2uw0aicYhu3AOeJWFjHuKvlcHojicYNdyUdbfhuticW8Pokq7f1Yiag/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在EcuM调用ComM_EcuM_WakeUpIndication()接口时，EcuM即请求了

COMM_FULL_COMMUNICATION::COMM_FULL_COM_READY_SLEEP状态，因此ComM可以在其Task(ComM_MainFunction)中进行状态的切换，此时ComM会干两件事：

- **打开通信栈**，即请求对应的XxSM模块进入FULL_COMMUNICATION状态，**Eg**:CanSM_StartWakeupSource()。代码示例：

```
ComM_RequestBusSMMode( Channel, COMM_FULL_COMMUNICATION );
```

- **切换网络状态**，即调用接口Nm_PassiveStartUp(Channel)，网络进入NetworkMode::RepeatMessageState。

Nm_PassiveStartUp()进一步的调用**CanNm_PassiveStartUp()**，之后的网络流程如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyQoajZVskFiawRYIInwwFWLYOABoqPDoF8lzXyc6o7BTicopBl3Iuhp8WnrCjqIxVn6yA9Xdq2JtsA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

至此，本文将收到一帧网络管理报文，到网络唤醒的流程“串烧”了一遍，当然，这里只是讨论了一种实现方式，实际的工程中，可能存在些许偏差。

![图片](https://mmbiz.qpic.cn/mmbiz_png/quuOyCqONwdD16hI11wAQWxGp4ajJ4DMnbsHGb4ViandFryeibcQb1Idxb3MHmrh20988OSES3OU1wPTicQbAr94g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



参考资料



AUTOSAR_SWS_COMManager.pdf

AUTOSAR_SWS_ECUStateManager.pdf

AUTOSAR_SWS_BSWModeManager.pdf

AUTOSAR_SWS_CANNetworkManagement.pdf