# **汽车电子构架演进（二）AUTOSAR的组成和演进**

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXoVe9QXfibQA4pg5tPicWlofiaydFQN0iaQ3XOWCicpibQXu9PVp8M2XFKsotg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

前文了解了汽车ECU和域控制器，从汽车**功能**上进行了说明。这些功能的具体实现还是需要**电路板**+**软件**来实现的。AS这个平台（代码路径：https://github.com/thatway1989/as）通过qemu虚拟机可以模拟了一个**域控制器的硬件**，并且上面跑的代码就是**AUTOSAR CP**的代码。硬件这里不去研究，上面运行的AUTOSAR软件结构是什么样的，怎么进行编程实现？本文将进行说明。



**1. ECU到域控制器的变化**

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXooCXD7H1WxW8hkV05SCh4lWmjqAxyibmibrguDqFthxMMiacNBAZiaQ5hNA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图中列举了两个功能：**车门**ECU和**车顶灯**ECU ECU的软件框架。可以看到，除了最上面的一层控制逻辑的SWC，下面的部分是可以共用的，也就是软件是可以**复用**的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXo6uhmOZ4RKGWUGlbfEFIzDgDylGFjON02L4Pic1E2NpT4TuuiazP6txNA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

融合的思路很简单，第一步把**相同**的部分找出来，第二步部分规定一个**接口规范**，如果有不同的部分就按照接口去跟相同的部分对接。这样就拼接成一个整体了。如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXonSh17iblDI3FibbnNUfbLlRiaYRq3GB1227WsqARkz78W42Ng5BLQiaWNA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

复用就涉及到统一接口的问题，大家都按照统一的一个规范搞，就可以兼容互换，以此诞生了**AUTOSAR**（AutomotiveOpen System Architecture）组织，中文是“**汽车开放系统架构**”，如下图中**应用软件**层是根据不同的需求会变化的，同样对于**底层硬件**根据供应商价格和性能的不同也是会**不断更换**。可以把中间的这一层不变的东西抽取出来，任你上下面怎么变，我都不用动，一次做好就不需要投入人力去开发这一部分，这样就节省了软件开发成本。

相同的部分，**抽取**出来如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXoOCWpR8fFaorNmhoKZqkwT8jiakBF0iaZY1QpSRibdstlcpZAUsYC9Wic4w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#  

# **2. AUTOSAR的分层结构**

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXo2IAhe5huGlzCuJbggeBsLdhBvqrmnvLwFGCPJpAeHrGpia0YdkibmR2w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**分层架构**是AUTOSAR的一个核心，是实现**软硬件分离**的关键。

这个过程比如做菜的话，SWC就是**炒菜**，RTE就是**锅**，那BSW就是**切好的菜**包括洗菜、切菜等，硬件就是**未加工的菜**。如果要做不同菜，未加工的菜（Hardware）和炒菜的方法（SWC）可以根据客人的需求变化，但是锅（RTE）和洗菜切菜（BSW）就不用变化了。

 

**SWC（SoftwareComponent）**

as中代码路径：as/com/as.application/common/test
   对于**应用**来说每个车的功能都不同，**千变万化又有创新性**，比如车灯控制，每个车的车灯个数和位置都不一样，车顶控制的软件肯定不一样。

应用软件层（Application Software Layer，ASW）包含若干个软件组件（**Software Component**，**SWC**），软件组件间通过端口（Port）进行交互。

 

**RTE（RuntimeEnvironment）**

as中代码路径：as/com/as.application/common/rte

当SWC变化的时候，下面的软件都不变，这时候就需要一个软总线，就好像一条**高速公路**，不管你从什么类型的路上这条高速都按照高速公路的**规则**运行。这个高速公路就是**RTE**。

运行时环境（Runtime Environment，RTE）作为应用软件层与基础软件层交互的桥梁，为**软硬件分离**提供了可能。RTE可以实现软件组件间、基础软件间以及软件组件与基础软件之间的通信。RTE封装了基础软件层的通信和服务，为应用层软件组件提供了标准化的基础软件和通信接口，使得应用层可以通过RTE接口函数调用基础软件的服务。此外，RTE抽象了ECU之间的通信，即RTE通过使用标准化的接口将其统一为软件组件之间的通信。由于RTE的实现与具体ECU相关，所以必须为每个ECU分别实现。

 

**BSW（Basic SoftwareLa****yer）**

as中代码路径：as/com/as.infrastructure

基础软件层（**Basic Software Layer，BSW**）又可分为四层，即服务层（Services Layer）、ECU抽象层（ECU Abstraction Layer）、微控制器抽象层（Microcontroller Abstraction Layer，MCAL）和复杂驱动（Complex Drivers），如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXoj8sWicBnYCYlUdICiawGEyYxuySe8J9v6o78UNnLPoM8RtHwEaKAcuAQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

AUTOSAR基础软件层上述各层又由一系列基础软件组件构成，包括系统服务（System Services）、存储器服务（Memory Services）、通信服务（Communication Services）等，如下图所示。它们主要用于提供基础软件服务，包括标准化的系统功能和功能接口。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXoKmlVWINWeZd7F7SjRt0emP8ZBl2aP9qREXLAt2qqrPzMZz5TBXPLibw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**AUTOSAR基础软件层结构**

（1）**服务层服务层**（Services Layer）提供了汽车嵌入式系统软件常用的一些服务，其可分为系统服务（System Services）、存储器服务（Memory Services）以及通信服务（Communication Services）三大部分。提供包括网络通信管理、存储管理、ECU模式管理和实时操作系统（Real Time Operating System，RTOS）等服务。除了操作系统外，服务层的软件模块都是与ECU平台无关的。

（2）**ECU抽象层ECU抽象层**（ECU Abstraction Layer）包括板载设备抽象（Onboard Devices Abstraction）、存储器硬件抽象（MemoryHardware Abstraction）、通信硬件抽象（Communication HardwareAbstraction）和I/O硬件抽象（Input/OutputHardware Abstraction）。该层将ECU结构进行了抽象，负责提供统一的访问接口，实现对通信、存储器或者I/O的访问，从而不需要考虑这些资源是由微控制器片内提供的，还是由微控制器片外设备提供的。该层与ECU平台相关，但与微控制器无关，这种无关性正是由微控制器抽象层来实现的。

（3）**微控制器抽象层微控制器抽象层**（MicrocontrollerAbstraction Layer，MCAL）是实现不同硬件接口统一化的特殊层。通过微控制器抽象层可将硬件封装起来，避免上层软件直接对微控制器的寄存器进行操作。微控制器抽象层包括微控制器驱动（Microcontroller Drivers）、存储器驱动（MemoryDrivers）、通信驱动（Communication Drivers）以及I/O驱动（I/O Drivers），如下图所示。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXoXgLsr8Pxs4PqcS5A9BgEQRBvhpYw0WYsdOuIribJE6F4N6c0L3ylAEA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

（4）**复杂驱动层**由于对复杂传感器和执行器进行操作的模块涉及严格的时序问题，难以抽象，所以在AUTOSAR规范中这部分没有被标准化，统称为复杂驱动（Complex Drivers）。

 

**SWC拓展知识：**

- 服务软件组件（Service SWC）。
- 应用软件组件（Application SWC）主要用于实现应用层控制算法。
- 传感器/执行器软件组件（Sensor/ActuatorSWC）用于处理具体传感器/执行器的信号，可以直接与ECU抽象层交互。
- 标定参数软件组件（Parameter SWC）主要提供标定参数值。
- ECU抽象软件组件（ECUAbstraction SWC）提供访问ECU具体I/O的能力。该软件组件一般提供引用C/S接口的供型端口，即Server端，由其他软件组件（如传感器/执行器软件组件）的需型端口（Client端）调用。此外，ECU抽象软件组件也可以直接和一些基础软件进行交互。
- 复杂设备驱动软件组件（Complex Device Driver SWC）推广了ECU抽象软件组件，它可以定义端口与其他软件组件通信，还可以与ECU硬件直接交互。所以，该类软件组件灵活性最强，但由于其和应用对象强相关，从而导致其可移植性较差。
- 服务软件组件（Service SWC）主要用于基础软件层，可通过标准接口或标准AUTOSAR接口与其他类型的软件组件进行交互。



# **3.   AUTOSAR方法论**

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXoTTn7K5oB3iaCOG4EDb1VktAGdGzKZqplFjbr22TDbJ9PBRf0ibYxE9vg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里的方法论见我之前的评价：[AUTOSAR入门-江湖](http://mp.weixin.qq.com/s?__biz=MzUzMDMwNTg2Nw==&mid=2247483744&idx=1&sn=3e35c538147bca92822d97894d2c6212&chksm=fa528744cd250e5262ca1821149fa5a79d50701d4dc7b2c15d3c8369a2c2d8cc756e68995060&scene=21#wechat_redirect)

软件提供商（外国的）感觉有点**居心叵测**，把软件做成，能多**傻瓜化**就多傻瓜，做成了**工具链**，车厂的人在界面傻瓜**点点**配置下就自动生成软件了，一行代码也看不到，给的代码全是宏就不是让人看的。让你啥都不会，然后涨价爱用不用，不用没了，反正你自己也不会也搞不出来。--这就是AUTOSAR的**方法论**，这里好像解释成**阴谋论**了。下面就来说说怎么在界面上点一点这个过程。



**1)ECU提取文件生成**

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXo4hJQrRX3x3jg72645uOjiaEpz0YfjuGduEHCd6Lrm60XELCbsAfgkJA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上面图中就是OEM提供**ECU提取文件**的过程，这里面用到SWC 描述文件、系统约束描述文件、ECU 资源描述文件。

过程如下：三种文件导入系统配置的编辑工具中，生成系统配置描述文件，就是整车的描述文件。整车的描述文件导入系统配置提取工具中，得到每个 ECU 的提取文件，包含了每个 ECU 需要用到的信息。

ECUEX：ECU Extractof System Description，即前面的 ECU 提取文件，由 OEM 交给 TIER1，TIER1 根据这个文件设计和开发 ECU。ECUEX 是 arxml 文件，但如果只做通信矩阵，DBC 也可以。

 

**2）arxml文件生成**

首先根据需求，对硬件进行配置，也就是使用EB工具配置MACL，生成MACL信息文件的arxml文件。

 

**3）代码生成**

根据arxml文件，这里生成的代码分为两类，给BSW用的和给SWC用的。

a）首先给BSW系统服务用的，这部分生成的是配置代码，可以理解为C语言的全局变量，可以作为函数执行的参数，另一方面是生成宏，控制程序的执行流程，然后跟BSW中不动的代码一块参与编译。as中代码路径：as/com/as.tool/config.infrastructure.system/argen

b）给SWC用的代码一般是智能化方面的代码，比如智能驾驶深度学习的参数，需要一些工具比如matlab生成。



# **4. AUTOSAR从CP到AP过渡**

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXoYRwmC79XHUIEj20qnKYnJziazFebkY41HeC7XFxs0bQfxlJnRdNicjpA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上面讲的都是AUTOSAR CP的功能。CP里面首先做到了**硬件的复用**即用一块电路板实现多块的功能，然后功能上进行了复用，比如里面对CAN的服务就一个模块。但是在**软件资源**方面，比如**内存**和**CPU**并没有进行复用。这里先说一个静态系统和动态系统的概念：

**静态系统**：**系统的资源在运行前就已经分配好了**，就像10块钱，5个人分，每人2块，多少就这样了。

**动态系统**：**系统的资源运行起来后了按需分配**，比如还是5个人，但是不会同时需要用钱，一共准备5块钱就够用了，谁需要就给谁，这样就节省了资源。

CP就是静态系统，所有的数据通道内存资源都是分好的，并且CPU是单片机时间片也是分好的，优点就像**计划经济**，**稳定**。但是这样一个系统的规模是有瓶颈的，当需要分配的单位越来越多的时候，硬件已经不堪重负了。不复用不变通，整个系统就会走向灭亡。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXoqGcqDGl91Dicbn8sKWBxF8Gc7TtaPNuJyibVTWG2eibJiaWZb2XEDKicJCQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图中可以看到AP中有了**操作系统**和**服务**，采用的面向对象的语言**C++**，当有服务请求的时候就从服务类里面new一个对象，**现用现分资源，用完立即资源回收**。从整个系统通信角度看，也从**面向信号**到**面向服务**转变。

汽车智能化，构件越来越复杂是推动CP到AP的根本因素，在这个过程中有两个方面尤为突出就是基于**以太网**的组网和**复杂CPU**的发展。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXoEPaTThxmYTFfkI0VnXRGl5waNnibI5XDJFQ2mhUnd2BIiaibib8QJXGTjQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

智能化要联网协作，节点越来越多，并且可能会通过无线接入Internet网络，实现远程控制。另外联网的设备比如摄像头的数据量激增，汽车上传统的基于信号的通信比如CAN和LIN已经满足不了，需要采用更复杂功能更强的**以太网通信**，车载以太网为汽车ECU带来了更高的**带宽**，使得数据的大量传输能够在短时间得以实现。以太网为更加有效地传输**长消息**和提供**点对点通信**提供了有效的解决方案。然而，AUTOSAR另一平台CP则是为了传统的车载通信技术CAN设计的，不能很好地兼容以太网，难以支持基于车载以太网的通信。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXotyPtqaRVoagZpOicoribhqkZ5VgRAibyEnBehRic9NiacFartnPibZsjgCeg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

近些年来汽车变得越来越智能，随之汽车对处理器的性能也提出了更高的要求，诸如自动泊车、环境感知、路径规划等高级功能对处理器的高算力需求远远高于对多核的需求。活多就得一个人都干了，像上图中的**一人乐队**一样。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7ASRYhSM9Yjw7RvDEvfgFrXoDh1KhBPERNF4RsQqiczWweUCFAzZcsOsn4djnUNNP7bUKtruBwYDw9w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图中可以看出来，由传统观念的**一个操作系统**用**一个CPU**，变成**多个CPU提供一个服务，一个服务支撑多个操作系统**。

就像**有地一块种，有饭一起吃一样**，这样才能集中力量办大事。

从处理器和半导体的技术角度来看，一个CPU提高性能的唯一方法是**多核**并行运行，然后如果还不行那就多个CPU再**并行**起来，以提供更强的算力。

另一方面，不同的操作系统，比如控制系统的**实时操作系统**，娱乐域的**非实时操作系统**，控制系统的**RTOS**等可以满足不同的需求需要在系统中**共存**。那么所有的软件都运行一个硬件集合上面而不是各自为战这样效率就又可以提升。真是计算机计算的发展就是未来**榨取更多的硬件资源**。就像很多各种各样的烧煤风力等发电站，现在一个**大型核电站**就搞定了，所有能源集中供应。

AP和CP的优势不同，在系统上需要**同时运行**起来。所以AP 不会取代 CP 或非 AUTOSAR 平台。相反，AP 和后端系统以及路边基础设施**共同协作**，发挥各自的优势，一起工作形成一个完整的系统。



**后记：**

Autosar AP还在规划中，速度比较慢，但是对于汽车电子的一些趋势已经很明显的显现，将在下次的文章中说明。