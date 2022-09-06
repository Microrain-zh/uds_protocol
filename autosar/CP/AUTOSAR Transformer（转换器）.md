# AUTOSAR Transformer（转换器）

## 简介和功能概述

转换器**（Transformer）**使得AUTOSAR系统能够使用数据转换机制（**data transformation mechanism**）对数据进行线性化**（linearize）**和转换**（transform）**。



转换器**（Transformer）**可以连接到通过RTE来执行的链路的数据转换**（transformer chains）**上，实现**ECU**内部和**ECU**之间的数据在通信过程中实现转换。



转换器为每钟通信关系（基于Port和基于Signal）提供定义良好的函数签名**（function signatures）。**这些签名被标记为用于转换功能。这些函数签名仅依赖于传输的数据元素（C/S操作签名或S/R接口签名）。但最终转换器的输出结果将始终是一个线性字节数组**（a linear byte array）**。



AUTOSAR系统可以支持链接多个转换器。其中第一个转换器的输入从RTE获取数据。后面的每一个转换器都使用上一个的转换器的输出作为输入。在第一个转换器之后的所有转换器都具有通用签名**（generic signature）**，仅使用一个字节数组作为IN和OUT参数。这样的体系结构可以用于设计系统，您可以灵活地向序列化流**（a serialized stream）**添加功能安全**（safety）**或信息保护**（security protection）**等功能。



约束和假设



数据转换和通信本身都是非常广阔的领域。因为很多用例和场景在理论上是可能的，并且可使得它们变得更加复杂。因为这些对转换器的功能有很大的影响（特别是在RTE中），这种多样性使得有必要对转换器施加一些限制和假设。



如果转换的目标主要是大型复杂数据元素的串行化，那么使用具有较大PDU的总线（例如：以太网）的通路进行通信，效率会最高。如果使用较小的PDU的总线（例如：CAN），序列化器产生的字节数组将必须跨越多个PDU，这是可实现的，但效率会比较低。

转换的主体主要为：

- **SenderReceiverInterface**端口类型的数据元素**（data element）**

- **ClientServerInterface**端口类型的操作**（ClientServerOperations）**

- **TriggerInterface**端口类型中**non-queued**的外部触发事件**（external trigger events）**

  

最重要的限制是无法转换整个PDU，主要原因是在RTE内部并不存在PDU。PDU其实是在**Com**模块内部构建的。



尽管如此，仍然可以将发送方/接收方通信的多个转换后的数据元素聚合到Com内部的一个大PDU中，每个转换后的数据元素**（data element）**在Com内部作为一个ISignal信号可见。但在这种情况下，包含在这个PDU中的所有数据元素/信号都相互独立地进行转换，每个都包含自己的头（如果转换添加了头）。因此，如果数据结构子元素是由不同的PortPrototypes/SWC的不同数据元素生成的，则不可能转换数据结构。



转换器链的长度不受这个概念中选择的解决方案的限制。但是为了启用高效的内存配置和实现，人为地将最大长度限制为**255。**同时当前的使用场景中，能看到的最大链长度其实只为**3**。

## 对其他模块的依赖关系

暂不依赖于AUTOSAR SWS模块。

## 功能规范

转换器从RTE中获取数据，对其进行处理，并将输出返回给RTE。它既可以：

- 序列化**（ serialize）**和线性化**（linearize）：**将结构化数据转换为线性形式
- 转换：修改或扩展线性数据 。（例如添加校验和）。



转换器是通信服务集群**（Communication Service Cluster）**中的BSW模块。这些集群主要为RTE提供通信服务。当RTE需要转换器提供的服务时，转换器由RTE执行。



转换器不是程序库**（library）。**因为转换器可以保存内部状态，但也可以无状态工作。



把一组转换器连接在一起形成一个转换器链是可行的。RTE协调转换器链的执行，并按照指定的顺序准确地调用该链中的转换器。使用这种机制，**ECU**内部和**ECU**之间的通信可以根据ARXML配置来实现转换。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhoqpWmzMEarCMcS8Tic6C6haicvYXe7RKSMJhawYCVcdbuGicsibEW6bE2YVibz3tmc7Gfqia0icicVHic3LfQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



在本例中，发送的应用程序的SWC需要发送复杂数据，这些数据使用带有两个转换器的转换器链进行转换。Transformer 1序列化数据，Transformer 2简单地对数据进行转换。在接收端，相同的转换器链与相应的转换器以相反的顺序执行。从SWC的角度来看，对于它们来说，使用哪些转换器或是否使用转换器是完全透明的。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhoqpWmzMEarCMcS8Tic6C6hazMxLTu0reQMcyPJqKRGg8aibNsGoOt9s6jz6WdFHC6e4qcvM0YPCT5w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



上图列举了如果通过ECU内部转换器，实现**NvBlockSwComponentType**和**DCM**之间不同格式数据的转换。



通常，转换器必须指定其输出格式，以使远程ECU**（ remote ECUs）**或硬件相关**（ hardware-dependent）**的BSW模块能够正确地处理转换后的数据。为此序列化的格式必须固定。



**注意:**

必须注意到，AUTOSAR目前没有规定任何只序列化有效负载并在前面不添加标题的转换器。SOME/IP Transformer可以序列化所有类型的数据，但它总是在数据前面添加部分SOME/IP头。



转换器应考虑目标ECU可能与发送端ECU有不同的架构（例如:8/16/32bit，little/big endian等等）。转换器应清楚地定义多字节字的端序。转换器应明确定义数据语义。(例如数据值的表示，例如：有符号整数的二的补码，文本数据的字符编码，等等)。



如果是序列化器，转换器应该清楚地定义：

- 复杂数据中包含的数据元素**（data elements）**的顺序。
- 字节数组表示的数据的源(=target)数据类型。（这是由连接的PortPrototype/SystemSignal决定的）

### 缓冲处理

转换器通常处理数据，生成一些协议信息，并将这些信息存储在header或者footer中，所以它需要一个地方来写入结果。



转换器可以工作在两种缓冲处理模式：就地缓冲**（In-place buffer）**和非就地缓冲**（out-of-place buffer）**。使用哪一个是由ARXML中的配置决定的，并最终影响转换器使用的接口。



使用就地缓冲**（In-place buffer）**的转换器，应将输入缓冲器也用作输出缓冲器。在这种情况下，转换函数只接受一个缓冲区指针参数。



使用非就地缓冲**（out-of-place buffer）**的转换器，不得改变输入缓冲器的数据。RTE分配转换器使用的缓冲区，它会计算输出在最坏情况下所需的缓冲区大小。



根据转换器链中转换器的特定位置，并不是所有的转换器都能够使用就地缓冲**（In-place buffer）**，因为转换器不允许修改SWC原始数据。此外，接收端上的最后一个转换器也不能使用就地缓冲**（In-place buffer）**，因为它必须将其结果直接写入SWC的缓冲区。也就是说：发送端链上的第一个转换器必须使用错位缓冲；接收端链上的最后一个转换器也必须使用错位缓冲。

### 转换器类别

不同种类的转换器具有完全不同的功能。因此可以把转换器进行分类。转换器类别包含所有具有类似功能的转换器。每个转换器链最多允许有一个转换器类的转换器。



目前，定义了以下转换器类：

- 序列化器**（Serializer）**
- 功能安全**（Safety）**
- 信息安全**（Security）**
- 自定义**（Custom）**



将来的AUTOSAR版本中可能会定义更多的转换器类别。



**序列化器（Serializer）**

**
**

序列化器**（Serializer）**转换器接受从RTE的复杂的数据。包括：**Sender/Receiver**的数据元素**（data element）**或**Client/Server**的**Operation**参数，或没有数据**（no data）**的触发通信**（Trigger communication）**，然后序列化器**（Serializer）**转换器提供生成的字节数组**（byte array）**作为**ISignal**或**IPdu**的，最后通过通讯栈**（COM stack）**传送到接收者。



所谓的“旧世界”**（old-world）**可变大小数组数据类型不被序列化转换器支持，只有“新世界”**（new-world）**可变大小数组数据类型可以被转换。

**SOME/IP Transformer**是属于AUTOSAR标准化的序列化转换器。



**功能安全（Safety）**

**
**

功能安全转换器保护通信不受无意**（unintentional）**的修改，以确保数据的正确的传输。功能安全转换器应为与功能安全相关的SWC的ECU间通信提供保护。功能安全转换器应当保证：

- 数据传输的正确顺序。

- 数据传输内容正确。

  

可以通过添加符合功能安全需求**（safety requirements）**的序列计数器（**sequence counters）**和校验值**（checksums）**和来实现。



**信息安全（Security）**

**
**

信息安全转换器保护通信不受故意**（intentional）**修改的影响，以确保总线通信的信息安全。信息安全转换器应为于信息安全相关的SWC的ECU间通信提供保护。信息安全转换器应当保证：

- 数据传输的真实性**（authenticity ）。**
- 数据传输的完整性**（ integrity）**。
- 数据传输的新鲜度**（freshness ）**。



可以通过添加符合信息安全要求**（security requirements）**的序列计数器（**sequence counters）**和校验值**（checksums）**和来实现。



**自定义（Custom）**

**
**

自定义转换器不是由AUTOSAR标准定义的，但可以由开发工作流程中的任何一方定义，并实现此非标准化的转换器。自定义转换器可以用CDD来实现。

### 错误处理

转换器需将错误返回给RTE,。并由RTE来协调进一步的执行，并将错误通知发送给SWC。



RTE需要决定返回代码是继续执行转换器链还是中止。转换器存在两种不同类型的错误：软错误**（Soft Errors）**和硬错误**（Hard Errors）**。如果转换器返回一个软错误，Rte将继续执行转换器链。如果一个转换器返回一个硬错误，Rte将中止转换器链的执行，因为该错误非常严重，以至于转换器链中的下一个转换器没有任何有意义的输入数据。



转换器错误的划分为:

| 错误码      | 含义   |
| ----------- | ------ |
| 0x00        | 成功   |
| 0x01 - 0x7F | 软错误 |
| 0x80 - 0xFF | 硬错误 |

如果转换器不能产生有效的输出时，它将返回一个硬错误。同时应该保持输出缓冲区不变。



**注意:**

转换器链早期阶段的软错误（按执行顺序）可能会被该链后面的转换器中的连续硬错误所掩盖，成为错误处理的最终结果。



**例如:**

如果端到端转换器**（E2E transformer）**检测到一个已损坏（错误的CRC）或伪装（错误的ID/CRC）的消息，它会抛出一个软错误。接着如果消息不能被反序列化，SomeIpXf很可能会用一个硬错误覆盖这个软错误。因此，端到端状态机的状态转换可能会被反序列化转换器的硬错误所掩盖。然而，只要消息无效，状态机**（state machine）**状态就会保持无效**（INVALID）**，因此一旦反序列化器能够反序列化消息，应用程序就会看到**INVALID**状态。



在这种情况下，仅希望依赖于端到端转换器状态机状态的应用程序，需要在应用程序中正确地评估反序列化器的硬错误。