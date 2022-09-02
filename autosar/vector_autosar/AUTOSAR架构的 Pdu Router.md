# AUTOSAR架构的 Pdu Router

前言：



PDU Router（路由器）在本文将简称为PduR，考虑到个人对PduR模块认识深度有限，且接触的CAN通讯功能运用PduR模块功能也较简单，所以本文仅对PduR模块做简单介绍。

## 

## *1* **基本概念**

首先了解下**路由**的概念，**路由是指路由器从一个接口上收到数据包，根据数据包的目的地址进行定向并转发到另一个接口的过程**（引自百度百科）。

接着了解下PduR更多的作用，引自[1]：PduR模块提供路由I-PDU（Interaction Layer Protocol Data Units）服务，使用在通讯接口模块（比如CanIf，CanNM，FrIf）和传输协议模块（比如CanTp，COM和DCM），如下图1所示。常用的PDU路由使用模块有：与UDS服务相关的AUTOSAR 诊断通讯管理模块（Diagnostic Communication Manager,DCM）和传输协议模块，与CAN通讯相关的AUTOSAR COM，通讯协议模块等。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/6PYxCJZfNqfRkmhGovkInXP5Xr6mIiaJnribN4RRy1Vv9Ufic7IZZb6Yib9yuOsoeEAcjDcujEAZKjnV7HjKCF4dDQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

图1 AUTOSAR架构下的PduR模块，引自[1]

PduR主要由2部分组成：

- PduR路由表：静态路由表描述每个被路由的I-PDU的路由属性；I-PDU路由的执行是基于静态定义的I-PDU ID。
- PduR引擎：根据PduR路由表执行路由动作的实际代码，该引擎不得不路由I-PDU从源头到目的地，以及根据I-PDU ID的源头翻译其目的地。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/6PYxCJZfNqfRkmhGovkInXP5Xr6mIiaJn20IQvXm8v2HubibPEztBgnib76FZVDicRXZp2rLPVYRoPA5icPZ1xFJd1A/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

图2 PduR模块的组成，引自[1]

通过以上知识对应到CAN通讯，**就是PduR模块从CAN接口模块/COM模块接收到了PDU，然后根据PDU ID查找已定义好的静态路由表，获得其目标地址，定向并转发到COM模块/CAN接口模块，即路由PDU，故称为PDU Router。**

## *2* **发送与接收操作**

从CAN通讯的发送与接收来看，再来理解下PduR模块的作用，即：

- 发送时，PduR模块将来自COM模块的发送请求路由到Can接口模块，将来自Can接口模块的确认路由到COM模块，如下图3。
- 接收时，PduR模块将来自Can接口模块的通知路由到COM模块，如下图4。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/6PYxCJZfNqfRkmhGovkInXP5Xr6mIiaJnxOgSicc50icuQv76qiavAWKXmdx13DZ3ZjqoE0ibvxQeHo7lC3SU33j5MQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

图3

![图片](https://mmbiz.qpic.cn/mmbiz_png/6PYxCJZfNqfRkmhGovkInXP5Xr6mIiaJnR8Uu50qjdL3n1g8CynfEpNIfNwbEonJhmRORcZ3UicPtf04AqCe0CjA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图4

下面借助文档了解下上述函数的定义，发送请求函数如下图5所示：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/6PYxCJZfNqfRkmhGovkInXP5Xr6mIiaJnXOexwgkC4CVc0Nib0VZmN7yGB3M97hRbaGFbxbkwkdluMfibbJFu65XQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

图5 发送请求函数，引自[1}

注意User:Up与当前的功能有关，CAN通讯的话，User:Up为Com，即发送时，COM模块调用PduR_ComTransmit函数。当然作为PduR模块的函数，会根据不同功能路由到其他模块，自然需要采用这种定义方式。同样地发送确认函数和接收通知函数也一样。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/6PYxCJZfNqfRkmhGovkInXP5Xr6mIiaJn4Qty8GROZxlluEQcXsxvUMiapYXfZ5RcVQSJ2sLRHhSwt6h2bGTojIQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

引自[1]

发送确认函数的定义如下图6，其中User：Lo的定义如下表，发送时，Can接口模块调用PduR_CanIfTxConformation函数向上确认。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/6PYxCJZfNqfRkmhGovkInXP5Xr6mIiaJnVGsvQf8GvtA4vheFhCibZIJmtXF35tWnwrbky9bTJFaT0x6243y9fRQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

图6 发送确认函数，引自[1]

![图片](https://mmbiz.qpic.cn/mmbiz_png/6PYxCJZfNqfRkmhGovkInXP5Xr6mIiaJn2ts6aJKobrQRWQankgkmC4H34AuhhdcDicD0KQfg7xCCjuibaoe28u3w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接收通知函数的定义如下图7，其中User：Lo的定义如上表，即Can接口模块调用PduR_CanIfRxIndication函数。在此就发现图4用PduR_RxIndication函数就不够准确咯。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/6PYxCJZfNqfRkmhGovkInXP5Xr6mIiaJnVzNCbUVWdQ4EGjZBs7NMrqhsXgYGFS3sQabsWqmNaPebR2ibHRhUAibQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

图7 接收通知函数，引自[1]

## *3*

## **路由表**

目前觉得PduR模块最关键还是路由表的定义，一是路由路径的确定，二是由源头ID到目的地ID的确定。特此再引用两例说明，如下图8、9。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/6PYxCJZfNqfRkmhGovkInXP5Xr6mIiaJnfCiaUOFyOSrvgIoO2tbOSicVgGN8oWrLxe0dyUsDUo4zGibSQ343QianvA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

图8 接收通知路由，引自[2]

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/6PYxCJZfNqfRkmhGovkInXP5Xr6mIiaJn2oRIDUmTyo7iaRcOzxQHvTpXIicibstk7qenR7s7QSJSUdwjDjr3Scabw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

图9 发送请求路由，引自[2]

------

以上就是简单介绍了PduR模块在CAN通讯的发送与接收所起的作用，当向上进入COM层后，简单地说就是：接收时，将PDU解包成一个一个的信号，供ASW使用；发送时，将一个一个的信号打包成PDU，向下发送。那么具体怎么实现？请看下篇文章。