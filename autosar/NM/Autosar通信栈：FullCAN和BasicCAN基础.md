# Autosar通信栈：FullCAN和BasicCAN基础

在搞清楚FullCAN和BasicCAN是什么之前，我们先搞清楚一些基础的东西。

1

基础概念

**提示：**以英飞凌tc397为例。

## 1、CAN Module与CAN Node、Controller关系

平时开发中，我们说“ECU有3路CAN”，所说的“3路CAN”和3个Node是一个概念吗？不是。**我们平时所讨论的“3路CAN”是指3个网络，也就是我们口语中的“节点”**。而芯片手册中（Data Sheet），一个CAN Module会包含多个Node（即，Controller），比如：tc397芯片手册中，MCMCAN Module包含3个CAN Module，每个Module包含4个Controller，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyP3p9jEnk3vRzHtwD1qv7QTOupsu9gtN6CrfG1cALymPHVXpqmf6Dvvh3SicRgiaTq8IAzGshhrByw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**2、Controller与Transceiver关系**

在实际的使用中，**一个Controller必须配一个Transceiver**，Controller+Transceiver = Network，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzIkpfAqvqURIgo9x6ZSibKFAUdoUMJoAC6ORic0PXZHNj7HYNkT8w70dMyiaPZ3q54tzTryxgzHmicUA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

所以，平时我们口语话的“3路CAN”是指3个Controller+Transceiver组合，即：3个Network，我们也常称“3个节点”。

## 3、Controller与RAM资源关系

刚提到，tc397中，一个CAN Module包含4个Controller，那每个Controller可以发送多少个CAN报文，接收多少个CAN 报文呢？这里我们要区分硬件缓存CAN报文的数量和项目中要求发送/接收报文的数量。

- **硬件缓存CAN报文数量**：是指上层请求发送报文或者接收报文时，CAN驱动最多能缓存的数量；
- **项目中要求发送/接收报文的数量**：是指当前节点要外发或者接收的报文数量。

以发送CAN报文数量为例：需求要求当前网络节点发送100帧CanID不同的CAN报文，实际该节点CAN Controller可用的硬件发送缓存区最多有32，意味着底层硬件最多缓存32帧发送报文，如果超过32帧发送请求，则会因没有硬件空间缓存而发送请求失败。

tc397 CAN Module资源情况如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzIkpfAqvqURIgo9x6ZSibKF2bcwyJRCrEEf4KvXGFQicmD7cyoL2lGFcVO18d1JHGa9YKqJCd94jEw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**提示**：上图中的Controller用“Node”表示。

由上可以看出，3个CAN Module，共12个Controller。**每****个CAN Module（4个Controller）****共用32个发送Tx Buffer，共用64个Rx Buffer**...

对于发送缓冲区，每个CAN Module共用32个发送缓冲区，如果配置了32个Tx Dedicated Buffer，则没有空间配置Tx FIFO/Queue；同理，每个CAN Module虽然有两个Rx FIFO，如果配置了64个Rx Dedicated Buffer，则没有空间配置Rx FIFO。一般，Tx/Rx Buffer配置时，会混合使用，比如：

- 20 Tx Dedicated Buffer+ 12 Tx Queue
- 40 Rx Dedicated Buffer+ 24 Rx FIFO

MCMCAN Module RAM区地址划分顺序如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzIkpfAqvqURIgo9x6ZSibKFA37htL34icV7o1AmFSejvNesdAic7pMB8eEOd7c4qKfuia9TlkUo5e4zw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 4、Mailbox、HRH、HWObject

**Mailbox**：邮箱，就是CAN驱动所具有的接收缓存区和发送缓存区，接收缓存区和发送缓存区均在RAM区。

**HWObject**：硬件对象，包含CAN ID、DLC、Data等信息的RAM区。

**HRH**：Hardware Receive Handle，接收句柄，一个HRH表示一个接收HWObject。

**HTH**：Hardware Transmit Handle，发送句柄，一个HTH表示一个发送HWObject。
Mailbox、HWObject、HRH、HTH、Controller、Transceiver之间的关系如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzIkpfAqvqURIgo9x6ZSibKF3z1Arrc59KyI1Upliashib6zibn0zG3Cy6hrCROfMFzKKkw7QK16yBkmA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2

FullCAN和BasicCAN是什么？

首先，FullCAN和BasicCAN是CanIf模块配置的参数。

- **BasicCAN**：一个HWObject(Hardware Object)可以处理一段范围的CanId
- **FullCAN**：一个HWObject(Hardware Object)只能处理单个CanId

Autosar对FullCAN和BasicCAN的解释如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzIkpfAqvqURIgo9x6ZSibKFUxlKELUXcL84DtzfE0ygLqMac2xeDBYuMfCLwcJMEr1on4EmVmibziag/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

将上述的解释进一步细化，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzIkpfAqvqURIgo9x6ZSibKFczZfiaEJfRbA47wpfZoaSKCDJcjEaiaCtPfGibGM1CvL5puh3cbPmJCtw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

使用工程中，MCAL会将缓存区分配成FIFO和Dedicated Buffer，FIFO和Dedicated Buffer的区别是什么呢？Dedicated Buffer区域，**Hareware Object与HRH/HTH一一对应**，而FIFO区域，**一个****HRH/HTH对应多个Hareware Object**，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vxSRuDpmo2G7icibBqU2lo7hQWY2QwxGHGRGqxOWZCUia8zPrwgiayqNNGH9mmOZDk8zswtBm9gtbrtyQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

3

为什么需要FullCAN和BasicCAN？

在CAN驱动层，可以通过过滤的方式，过滤一段范围内的CanID，也就是说：会有一段范围内的报文接收进来，但是接收进来的这一段范围的报文并不一定都是上层所需要的，怎么办呢？用软件方式，再过滤一遍，由CanIf过滤所需要的CAN报文。因此，提出了FullCAN和BasicCAN的概念。

比如：HRH对应BASIC CAN类型，接收CanID范围：0x10~0x18，CanIf根据过滤算法，在0x10~0x18范围内过滤出需要的0x10、0x13、0x14、0x16、0x17送给上层，而其余的丢弃，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vxSRuDpmo2G7icibBqU2lo7hQp0RzZmbADuWoyXhVMkLsAu7XjBYzoy3H9lJSZ3TkaZnqG0XC3h6d2w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

CanIf可以通过设置CANIF_HRHRANGE_LOWER_CANID、CANIF_HRHRANGE_UPPER_CANID方式过滤，也可以通过设置CANIF_HRHRANGE_BASEID、CANIF_HRHRANGE_MASK进行过滤。

## 不同报文类型如何选择FULL CAN/BASIC CAN

**应用报文**：一般选择配置成FULL CAN类型，对于应用报文，一般不需要缓存，使用最新接收的数据即可。对于发送的应用报文，都配置成FULL CAN类型需要一个前提：**上层需要****发送应用报文数量＜底层硬件缓存区数量**。比如：底层发送硬件缓存区数量为32，节点需要发送的应用报文数量为50，显然无法将50个发送的应用报文都配置成FULL CAN。遇到这种情况，一般会将重要的应用报文配置成FULL CAN，而其他要发送的应用报文配置成BASIC CAN。这样配置以后，硬件缓存区的分配就需要混用，即：Dedicated Tx Buffers+Tx Queue或者 Dedicated Tx Buffers+Tx FIFO，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vyP3p9jEnk3vRzHtwD1qv7QwR6L9ASvOFTbqlKlsZ1vfGzDOxJibHaVKqvprfrcfXTTTJlCc6YajdQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如上图，ID3、ID15、ID20是比较重要的应用报文，配置成FULL CAN，分别给一个独立的缓存区；其他的缓存区则配置成BASIC CAN，即：一个缓存区可以发送多个不同CanID的报文。

**诊断报文**：一般选择配置成BASIC CAN类型（结合FIFO Buffer使用），因为诊断报文的请求/响应不能错序，需按照顺序处理，且数据不能覆盖；

**网络管理报文**：接收一般选择配置成BASIC CAN类型，因为一个节点一般会要求接收一段范围的网络管理报文，eg:0x500~0x53F。发送网络管理报文配置成FULL/BASIC CAN类型均可，如果资源够用，推荐配置成FULL CAN类型，因为每个节点的发送网络管理报文唯一；

**标定报文**：一般选择配置成FULL CAN类型。



![图片](https://mmbiz.qpic.cn/mmbiz_png/quuOyCqONwdD16hI11wAQWxGp4ajJ4DMnbsHGb4ViandFryeibcQb1Idxb3MHmrh20988OSES3OU1wPTicQbAr94g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



参考资料

SIMPLE TITLE

Infineon-AURIX_TC3xx_Part2-UserManual-v01_00-EN.pdf

AUTOSAR_SWS_CANInterface.pdf