# AUTOSAR入门-汽车电子架构演进（一）ECU和域控制器

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7AS5ldglcxvUGpgAuI0WrgnR0D4zwic7ktbzYWqy6MuiatNkicBfKwFVaeVAytv5t8ZAk6DKrGvw1KPZw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

本次文章不**show me the code**，对一些汽车基础问题进行梳理。首先我们讲的这套AS平台的代码https://github.com/thatway1989/as，实现了**AUTOSAR cp**的功能，但是这套**代码跟汽车**有什么关系？整天AUTOSAR、诊断、CAN什么的，但是不知道在车上怎么用，就像**只听见过猪叫，没见过猪跑**一样。

首先AS这个平台**模拟了一个域控制器**，并且上面跑的代码就是AUTOSAR cp的代码。那么什么是域控制器，在汽车软件演进的过程中又扮演了什么角色，AUTOSAR的软件框架又是什么，跟汽车业务又有什么关系？这些将在文中一步一步分析。

另外，汽车电子框架演进为三步：**分布式（ECU）-》集中式（域控制器）-》中央式（硬件虚拟化+SOA）**，也将在文中说明。

汽车电子构架演进涉及的内容比较多，这里分**三篇**进行介绍。本次文章先介绍**ECU**和**域控制器**。

#  

# **1. 什么是ECU？**

**电子电路**相对于**机械**控制的优点在于**精细化**和**自动化**，是**智能化**的基础。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7AS5ldglcxvUGpgAuI0WrgnRlQvAJTJiaZYeicTHicicKdyKKedoQFx7IvibhvHyicaK9KS3fiayYsgNVQJsg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

首先，要精细化的控制，就需要在传统机械设备上**引入电子电路**。1968 年电子设备首次出现在汽车中，当时大众汽车在大众1600轿车的发动机中安装了电子控制单元 (ECU)，以帮助**控制燃油喷射**。

 

另外，在当今**智能化**汽车对**控制**要求更加严格，内燃机提供的动力比较奔放，不利于进行细微的控制，而电机，特别是**步进电机**，转动一圈360度可以按角度进行控制，极其精密，这样给**智能驾驶**提供了可能，智能驾驶需要非常细微的控制，复杂难控制的**燃油发动机**是**不具备精细控制**的条件的，不然把人撞死就完了。除却**新能源**，这就是电动车将要取代燃油车一个重要原因。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7AS5ldglcxvUGpgAuI0WrgnRf0toaXnEcgcu1SxnkurzCuibj9pK9kqibDG3JX2XxTgrzZic3U7Al8p4Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 上面提到**ECU**（Electronic Control Unit）电子控制器单元，它们的用途就是控制汽车的行驶状态以及实现其各种功能，是一块**独立的电路板**。主要是利用各种传感器、总线的数据采集与交换，来判断车辆状态以及司机的意图并通过执行器来操控汽车。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7AS5ldglcxvUGpgAuI0WrgnRkUZNib0U8wNHM0UGDziaFvjTzjl0RdaIpCTJ5HUxWnv9aCtntjVZlA4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 随着汽车智能化的发展，汽车里面ECU逐渐增多，达到了**100+**。这么多ECU，汽车软件这时候的构架是**分布式**的，汽车里的各个ECU都是通过**CAN和LIN**总线连接在一起，如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7AS5ldglcxvUGpgAuI0WrgnRd2jvlOoJ4cpIhO0v2OdUM0LGT6ZpaWDNAIFfI29jSYErC6sggB5iapQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

下面罗列一些主要的ECU：

EMS（Engine Mangement System）**发动机管理系统**，应用在包括汽油机PFI（如上图）、GDI，柴油机，混合动力系统等，主要控制发动机的喷油、点火、扭矩分配等功能。

TCU（Transmission Control Unit）**自动变速箱控制单元**，常用于AMT、AT、DCT、CVT等自动变速器中，根据车辆的驾驶状态采用不同的档位策略。

BCM（Body Control Module）**车身控制模块**，主要控制车身电器，比如整车灯具、雨刮、洗涤、门锁、电动窗、天窗、电动后视镜、遥控等。

ESP（Electronic Stability Program）**车身电子稳定控制系统**，车身电子稳定控制系统。ESP可以使车辆在各种状况下保持最佳的稳定性，在转向过度或转向不足的情形下效果更加明显。ESP是博世公司的专门叫法，譬如日产的车辆行驶动力学调整系统VDC（Vehicle Dynamic Control ），丰田的车辆稳定控制系统VSC（Vehicle Stability Control ），本田的车辆稳定性控制系统VSA（Vehicle Stability Assist Control），宝马的动态稳定控制系统DSC（Dynamic Stability Control ）等。现如今很多中高端合资车、国产车都会配备这个模块。

BMS（Battery Management System）**电池管理系统**，顾名思义这个控制器是专门针对配备有动力电池的电动车或者混合动力车准备的。主要功能就是为了能够提高电池的利用率，防止电池出现过度充电和过度放电，延长电池的使用寿命，监控电池的状态。

VCU（Vehicle Control Unit）**整车控制器**，用于混合动力/纯电动汽车动力系统的总成控制器，负责协调发动机、驱动电机、变速箱、动力电池等各部件的工作，提高新能源汽车的经济性、动力性、安全性并降低排放污染。

# **2. 什么是域控制器？**

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7AS5ldglcxvUGpgAuI0WrgnRH8nS9z4ic2tP5tRgYklvJGsoicZWiaMAib4jIx7hxeM3ecZvqQGkphO6tg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

当出现了这么多ECU，首先从成本上来说，这么多电路板，每个电路板上都有芯片，成本非常的高，并且这么多ECU要连线，**线束的总长度很大**，**成本很高**。这些电路板很多**功能都是一样**的，完全可以**一个电路板**把活干完，但是由于汽车不同零部件的供应商不一样，**很难协调**。另外这些ECU如果需要交互，在一起协调工作就变的很困难，**满足不了智能控制**的需求。

为了解决**分布式EEA**（Electrical ElectronicArchitecture电子电气架构）的这些问题，人们开始逐渐把很多功能相似、分离的ECU功能集成整合到一个比ECU性能更强的处理器硬件平台上，这就是汽车“**域控制器****（Domain Control Unit，DCU）**”。域控制器的出现是汽车EE架构从ECU**分布式**EE架构演进到域**集中式**EE架构（**如下图**）的一个重要标志。

![图片](https://mmbiz.qpic.cn/mmbiz_png/KibCRZcaN7AS5ldglcxvUGpgAuI0WrgnRphZ8J4SxD0qGxySVBSRjYL1Bg2RiaEZC3FdmhslicaIEpN4fvXK8Q7iaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

域控制器是汽车每一个功能域的核心，它主要由域**主控处理器**、**操作系统**和**应用软件****及算法**等三部分组成。**平台化、高集成度、高性能和良好的兼容性**是域控制器的主要核心设计思想。依托高性能的域主控处理器、丰富的硬件接口资源以及强大的软件功能特性，域控制器能将原本需要很多颗ECU实现的核心功能集成到进来，极大提高系统功能集成度，再加上数据交互的标准化接口，因此能极大降低这部分的开发和制造成本。

对于功能域的具体划分，各汽车主机厂家会根据自身的设计理念差异而划分成几个不同的域。比如BOSCH划分为5个域：

**动力域****（PowerTrain）、**

**底盘域****（Chassis）、**

**车身域****（Body/Comfort）、**

**座舱域****（Cockpit/Infotainment）、**

**自动驾驶域****（ADAS）**。

这也就是最经典的**五域集中式EEA**，如上图所示。也有的厂家则在五域集中式架构基础上进一步融合，把原本的动力域、底盘域和车身域融合为整车控制域，从而形成了**三域集中式EEA**，也即：

**车控域控制器****（VDC，Vehicle Domain Controller）、**

**智能驾驶域控制器****（ADC，ADAS\AD Domain Controller）、**

**智能座舱域控制器****（CDC，Cockpit Domain Controller）**。

大众的MEB平台以及华为的CC架构都属于这种三域集中式EEA。



后记：

  前面说到域控制器是由**域****主控处理器**、**操作系统**和**应用软件****及算法**等三部分组成，而这三部分正式AUTOSAR软件构架所规范的内容，可以这么说**AUTOSAR规范了域控制器的实现**方式。下节介绍AUTOSAR软件框架。