# AUTOSAR的基本概念

1**AUTOSAR的解决方案**

面对日益复杂的汽车E/E架构，在欧洲大地上诞生的AUTOSAR组织，提出了解决方案。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibcFsDak8GUj8lRpj9QiaiaRb5KApYl0v6xQAJTadGHLn973dgSVV35eFfnY0juUtDuYgTfb981WWjQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9L6bqBibggdRyp6LMMKfajs9Xu53H19zvc7EAyeN3yjn5q7ZllgZQbMib8ZB33A53AxBBCPqOQqwfA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

而且做了标准化：

- 软件接口
- 交换格式
- 方法论

首先，其目标要：

软件功能模块在不同车型之间被重用

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9L6bqBibggdRyp6LMMKfajs6EVMmotEedYmrlG7nlPjlUWDUha5YEpxsibRqoVRQxluicLH5kImBYNw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9L6bqBibggdRyp6LMMKfajsH8WGJfQiaoiaPvjkVzCWGIRsfZbJKIUpxJuP3Hv3MtDZRNNFo5bSnbYA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

还有，标准化AUTOSAR的代码配置/建模工具

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9L6bqBibggdRyp6LMMKfajs6Ea3wgB108JZ7Xj8Su4a1gMPfic9ok4lno8CuPXyaGBLtRuL9yubunw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

标准化接口（也可见上图）：

| **AUTOSAR Interface **              | “AUTOSAR Interface”定义了软件组件和/或BSW模块之间交换的信息。该描述独立于特定的编程语言，ECU或网络技术。 |
| ----------------------------------- | ------------------------------------------------------------ |
| **Standardized AUTOSAR Interface ** | “Standardized AUTOSAR Interface”是其语法和语义在AUTOSAR中标准化的“ AUTOSAR Interface”。“Standardized AUTOSAR Interface”通常用于定义AUTOSAR服务，这是AUTOSAR基本软件向应用程序软件组件提供的标准化服务。 |
| **Standardized Interface **         | “Standardized Interface”是一种在AUTOSAR中标准化的API，无需使用“ AUTOSAR Interface”技术。这些“Standardized Interface”通常是为特定的编程语言（如“ C”）定义的。 |

交换格式标准化（arxml）

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9L6bqBibggdRyp6LMMKfajs0icicC3LqV1QnyeKaap16lc2NZch3NQM6spMHmtNLgibZsibSGZ9HeGouA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

arxml到底长什么样？以下截取一段来熟悉下：

```xml
<AUTOSAR>
  <AR-PACKAGES>
    <AR-PACKAGE>
      <SHORT-NAME>DataTypes</SHORT-NAME>
      <ELEMENTS>
        <IMPLEMENTATION-DATA-TYPE>
          <SHORT-NAME>uint8</SHORT-NAME>
          <CATEGORY>VALUE</CATEGORY>
          <SW-DATA-DEF-PROPS>
            <SW-DATA-DEF-PROPS-VARIANTS>
              <SW-DATA-DEF-PROPS-CONDITIONAL>
                <BASE-TYPE-REF DEST="SW-BASE-TYPE">/DataTypes/BaseTypes/uint8</BASE-TYPE-REF>
                <SW-CALIBRATION-ACCESS>NOT-ACCESSIBLE</SW-CALIBRATION-ACCESS>
                <DATA-CONSTR-REF DEST="DATA-CONSTR">/DataTypes/DataConstrs/uint8_DataConstr</DATA-CONSTR-REF>
              </SW-DATA-DEF-PROPS-CONDITIONAL>
            </SW-DATA-DEF-PROPS-VARIANTS>
          </SW-DATA-DEF-PROPS>
          <TYPE-EMITTER>Platform_Type</TYPE-EMITTER>
        </IMPLEMENTATION-DATA-TYPE>

        // 部分内容省略

          </ELEMENTS>
        </AR-PACKAGE>
      </AR-PACKAGES>
    </AR-PACKAGE>
  </AR-PACKAGES>
</AUTOSAR
```

再来看看其他几个基本概念：

**SWC**

SWC，即Software Component，是封装了部分或者全部汽车电子功能的模块，其包括了其具体的功能实现以及与对应的描述。

例如，我们可以把Dimmer、Switch、Door Control设计成SWC

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxics93L4uRWvjIW6fyAicykl950O9EURf3DBJlYsfB8vTkqmsjnvw7rtcB4SoMvsohVUQ5OVtjMhibcg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

SWC分类：

- Atomic component (最小的逻辑单元，无法再分)

- - Application（普通应用类）
  - Sensor/actuator（给Application提供I/O控制等）

- Composition（可以包含数个SWC的逻辑集合）

**Port**

Port是SWC之间通信用，算是SWC的组成部分。

Port分两大类：S/R(Sender/Receiver)和C/S(Client/Server)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxics93L4uRWvjIW6fyAicykl9icUt8gmN4E5SdnPjEbnkeEtAy5gYjk1xShXwusvib1gDJSs76shcDVGw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**Runnables**

Runnables，即Runnable entities，也是SWC的组成部分，但它是运行在RTE里面，由RTE周期事件触发或者其他事件触发时调用。Runnable包含着实际运行的函数。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxics93L4uRWvjIW6fyAicykl98DiaosvDRozibgfRTn2SCykNeic0MPX6u2nsnaM558fgenEBFWTJFaVeA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2**AUTOSAR的方法论**

方法论，可以说是AUTOSAR的灵魂，就像一道菜的配料和方法，如果没有这个方法，那么食材仅仅是食材，而不是一道美味的菜肴。

既然，说方法论是AUTOSAR的灵魂，那么什么能承载这个灵魂，没有载体的灵魂就是孤魂野鬼啊。ARXML就能担此重任。其实，ARXML本质就是XML格式的文本，只是被AUTOSAR组织将其披上一件美丽的外衣。

方法论具体有哪些要求呢？见下图

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/6AXJMmPWkxics93L4uRWvjIW6fyAicykl9dqs2ia2skYoiaHhtqPeBdlYvOxkcwvGCUdGwicLWXcSU6S0ibRsZTj7VbA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

方法论描述了从系统底层配置到ECU可执行代码产生过程的设计步骤。所以，这个ARXML文件也是挺复杂的。我们先看个图感受下：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9L6bqBibggdRyp6LMMKfajsC4ribkVsBTAAyhlic6p7HAEtJLVDAeajcImEJW3riaxm2dyowibkLmLLnA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这个ARXML关联着整个开发的方方面面

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibCKicAUC3EOjA7bH9Hk8Y1365EXv9QKCnlc9U6rTcmBOxenHDbnMlabLM7ibvr5yK8aPzJHkjelaOA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这个开发过程简单抽象起来就像：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibCKicAUC3EOjA7bH9Hk8Y13uY1BH6pFQ6CHK1eCBsUQ0HvtodDrzdiaV5GYTLdbpIxcbyNQXaGYu4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

抽取其中BSW的配置和生成过程来看看

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/6AXJMmPWkxics93L4uRWvjIW6fyAicykl9dDWWck9EAAiaheTGQzUGgwTK5nHAzMZa5SS8OIayb4G3mJX6oiaKQFCA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)



3**AUTOSAR的实时环境**

RTE，Run Time Environment实时运行环境，是整个AUTOSAR架构运行的桥梁，各个模块SWC之间的通信不是直接交互的，而是经过该层作为运行的基础，RTE里包含着OS大量的运行策略和服务。RTE也是VFB（Virtual Functional Bus）的实现。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxics93L4uRWvjIW6fyAicykl9UDVYqjvdE0C32kic8EewiabKbhtibJ3lALf4Zeibb8CpEA7ibrlF79xCicjg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

- RTE需要配置(e.g. 把runnables对应到OS的tasks中去)
- 通过RTE的事件触发runnables的运行
- 生成调用runnables的task代码
- 配置OS的一部分 (tasks, events, alarms)
- 实现SWC之间的通信
- 每个ECU的RTE因SWC的需求而异
- RTE抽象了OS，防止SWC直接访问OS和BSW



4**AUTOSAR的基础软件**



基础软件，即BSW。从AUTOSAR架构看，中间一层，都是BSW。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxics93L4uRWvjIW6fyAicykl96tb8DsLysvibG2WfHeLU266S6rXia3aV3frAt8EbAh5HxQw04jPrF3Ew/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

细化后

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxics93L4uRWvjIW6fyAicykl9zlEH9sMxFBUOrJe8p1zlIaADWmUUg6ImowCVcL9fDwkDb7YjRU5azw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

再细化

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxics93L4uRWvjIW6fyAicykl9dchibib42zGts9ZKKhEFsntY4PJA4keKw5mG8F7G3ktj3s56WdibU4YJA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

可以看出，其内容非常丰富，严格遵循着AUTOSAR的各项标准。

BSW抽象程度比较高，包含着许多基础软件。

从图上可以看出，其分了很多类，对应不同的功能。例如Memory、Communication、System等等。

特别一提的是，Complex Driver，是应对比较复杂的驱动的，这个在AUTOSAR的标准上是没有很明确的定义的，可由用户去实现。