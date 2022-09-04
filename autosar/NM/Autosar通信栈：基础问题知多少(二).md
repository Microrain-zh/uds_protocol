# Autosar通信栈：基础问题知多少(二)

"走得越远见识越多，认识的人越多，你就越能体会到，人这一辈子你真的在意的，同时又在意你的，就那么几个。这几个就是你全部的世界。三两知己，爱人在侧，父母康健，听起来平平无奇，但已经是中等偏上的人生答卷了。"

--《人世间》“李潈”评论

## QA1、Rx Signal Filter有何用？

**答**：Rx Signal Filter有什么用呢?通信开发中，大家应该碰到过这样的问题：信号阈值判断。举例：车速信号（Vehicel_Signal）用一个uint16数据类型表示，Vehicel_Signal有效取值范围：0~300Km/h，如果Vehicel_Signal ＞ 300Km/h，则丢弃该信号，保持上一次的有效值。所以，使用Rx Signal Filter，即可处理类似的问题。

一个I-PDU会包含1~N个Signal（N为大于1的正整数），这些Signal可以不包含在任何SignalGroup中，也可以包含在多个SignalGroup中。

为更好地理解Signal Filter，这里讨论一下信号的接收处理流程：

**Step1、**COM模块收到一个I-PDU，该I-PUD包含5个信号：Signal00、Signal01、Signal02、Signal03、Signal05。其中Signal01、Signal03、Signal05属于SignalGroup01;

**Step2、**在COM层，将接收到的I-PDU拆分成Signal，如果信号有Filter Condition，则进行Filter Condition判断；

**Step3、**如果Filter Condition = FALSE，则丢弃该信号，**对应的Rx Signal Buffer不更新**。如果Filter Condition = TRUE，则更新信号对应的Rx Signal Buffer；

Autosar规范对Filter Condition的解释如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyTgId0OibCnOsDBibr7aw9UuSjj4DaOC2w2bBHeMuUlMc8nVuVoz6agXB1pKIqv95nz3bdqQTibUEKg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Autosar规范中解释Signal Filter Condition = False，如果这个Signal不在SignalGroup中，该**信号**对应的接收缓存区数值无需更新；如果这个Signal**在SignalGroup中**，该**信号组**对应的接收缓存区数值无需更新，即：保存上次值，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyTgId0OibCnOsDBibr7aw9Uubz5UNCnkI5D6bm3qg37fbt0SrxWy8S5ZTqQ7NsHUM5VH74ShYyibCOQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**Step4、**如果信号不属于任何的SignalGroup，上层可以直接调用Com_ReceiveSignal()接口，从Rx Signal Buffer获取信号数据。如果信号属于某个SignalGroup，则需要通过Com_ReceiveSignalGroup()接口，将信号数据从Rx Signal Buffer缓存区Copy到SignalGroup Rx Buffer(shadow buffer)，之后上层再通过Com_ReceiveSignal()接口读取信号值。

Rx Signal接收流程如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyTgId0OibCnOsDBibr7aw9Uu7U4PoBX396nfcUic6G1iaIurxhd6LQkMprTtE9y5atiboXjhS8TqQOsuA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## QA2、Tx Signal Filter与Tx Mode有何关系？

**答**：从QA1中，可以知道Rx Signal Filter可以检查信号阈值。那么Tx Signal Filter能否也检查信号阈值呢？Autosar规范给出了解释，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyTgId0OibCnOsDBibr7aw9Uuc8ubaur0tBoibn3r9u77Yc436rI85fVggDiapic8h0JNOZASzF3kHHvHg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

总结：Tx Signal Filter不能检查信号发送的阈值，只是I-PDU发送模式的判断依据。Tx I-PDU有哪些发送模式呢？ComTxModeTrue或者ComTxModeFalse。如下通过一个示例分析ComTxMode与Tx Signal Filter关系：

**Step1、**如果信号不包含在任何TxSignalGroup中，上层通过调用Com_SendSignal()接口，直接更新Tx Signal Buffer。如果信号包含在某个TxSignalGroup中，上层通过调用Com_SendSignal()接口，更新SignalGroup Tx Buffer(Shadow Buffer)中信号数据，之后通过Com_SendSignalGroup()接口更新Tx Signal Buffer；

**Step2、**通过Filter Condition检查Tx Signal过滤条件；

**Step3、**TMS(Transmission Mode Selector）判断Tx I-PDU中每个Signal的发送模式接口，如果其中一个Signal的检查结果是TRUE，则Tx I-PDU选择ComTxModeTrue发送行为；如果所有的Signal的检查结果均为FALSE，则Tx I-PDU选择ComTxModeFalse发送行为。

Tx Signal的发送行为如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyTgId0OibCnOsDBibr7aw9UuH9G14s1t53obibMmK8G4jKjjSck72puoVFY4KG2j9qubGPZibwa4lTwQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## QA3、ComTxModeTrue、ComTxModeFalse使用场景？

**答：**为什么一个Tx I-PDU会有ComTxModeTrue、ComTxModeFalse**发送模式**呢？一个Tx I-PDU的**发送类型**有：DIRECT、MIXED、PERIODIC。

DIRECT：事件型，比如：连续发送3次，发送最小间隔20ms。

PERIODIC：周期型，比如：每100ms发送一次。

MIXED：包含DIRECT、PERIODIC两种发送行为。

ComTxModeTrue、ComTxModeFalse可以存在这样的使用场景：Tx I-PDU Mode = TRUE时，使用PERIODIC发送行为；Tx I-PDU Mode = FALSE时，使用DIRECT发送行为。

## QA4、Tx Signal的发送行为注意事项

**答**：上层通过RTE调用Com_SendSignal()接口，更新需要发送Signal数值，注意：此时信号数值只是更新到了发送缓冲区。信号值的发送，依赖信号所在的Tx I-PDU周期。同时，COM层的发送主函数Com_MainFunctionTx()周期判断每一个Tx I-PDU周期到时与否，如果Tx I-PDU周期到时，调用PduR的发送接口发送Tx I-PDU.

**对于周期型报文**，假设：Com_MainFunctionTx()周期 = 10ms，周期型报文Tx I-PDU对应的发送周期为30ms，发送行为如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyTgId0OibCnOsDBibr7aw9UuBt0VC3Fjo2gNMQviccD9tcRicFGID0r1icYchH8pflUS7PYsicibricZicabg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

需要注意：上层信号的更新频率不应超过Tx I-PDU的发送周期，否则信号值被覆盖，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyTgId0OibCnOsDBibr7aw9UuhsHAH38xqzfIzzia4uhZeyeBGOhPUnzBINLtMeFZsPSKpB5wXlOXbZg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

大家可能在想：使用Queue机制不可以吗？信号数据队列发送，遗憾的是：目前，Autosar规范中的信号发送行为，不考虑Queue。

**对于事件型报文**，假设：Com_MainFunctionTx()周期 = 10ms，事件型报文Tx I-PDU对应的最小发送间隔为20ms，发送行为如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyTgId0OibCnOsDBibr7aw9UuUNLHXjNoJBTPmNUo1tmUPibPLN1RCsOC81gBrCavMhvoUUuAMuz78tQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

同样，上层信号的更新频率过快，会导致发送信号的数值被覆盖，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyTgId0OibCnOsDBibr7aw9Uu4DYygOXNXfA0GKw9AXNrHC3LypWI1j1APGutwa7Npdz3KlNQyWllzg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/mmbiz_png/quuOyCqONwdD16hI11wAQWxGp4ajJ4DMnbsHGb4ViandFryeibcQb1Idxb3MHmrh20988OSES3OU1wPTicQbAr94g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



参考资料

SIMPLE TITLE

AUTOSAR_SWS_COM.pdf