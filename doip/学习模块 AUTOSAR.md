# 1. AUTOSAR学习模块

## 概述

AUTOSAR（AUTomotive Open System ARchitecture，汽车开放系统架构）成立于2003年秋天，是由汽车行业主要整车厂和供应商组成的标准化联盟。该联盟旨在为ECU软件制定参考架构，以解决现代车辆中软件日益复杂的问题。

## 动因

车辆功能的创新导致E/E车辆架构日益复杂。与此同时，开发要求通常自相矛盾：例如要求驾驶域辅助系统支持关键性驾驶操控，提高燃油经济性同时符合环境标准。信息娱乐和通信系统与实时车辆环境和在线服务的不断深入整合带来了更多挑战。

为持续满足如上需求，需要一种新的ECU软件架构，否则无法满足不断增加的客户要求和日益严苛的法律法规。

2003年，汽车整车厂和供应商创建AUTOSAR联盟。联盟的目标是阻止不断重复开发相似或相同的应用软件组件，为实现基本功能的协作奠定基础，同时为创新性新功能的竞争开发留出空间。联盟定义的AUTOSAR标准构成了未来车辆应用程序的基础，有助于弥合各领域之间的界限。在ECU网络中灵活地分布软件可以为系统范围的优化创造更多可能。

## AUTOSAR联盟

AUTOSAR联盟是由汽车整车厂、供应商以及软件、半导体和电子行业的公司组成的全球发展合作组织。

在“标准中合作，实现上竞争”的口号下，各成员以工作组为单位，享有并承担着不同的权利和义务。核心合作伙伴决定哪些团体可加入AUTOSAR联盟，高级成员则负责制定标准。

AUTOSAR的目标如下：

- 标准化应用程序软件功能之间的接口和基本功能的接口
- 定义ECU软件参考架构
- 将分布式开发过程的数据交换格式标准化

实现这些目标后，参与的公司希望获得以下好处：

- 通过功能的灵活集成、重新分配和交换来优化ECU网络
- 掌控日益提升的产品和流程的复杂程度
- 维护整个产品生命周期

AUTOSAR 3.0版本是第一个用于ECU量产的版本。

当然，AUTOSAR标准有其局限性。例如，AUTOSAR无法描述应用软件组件的功能行为。

![image](https://user-images.githubusercontent.com/80186561/180648004-c58d2941-17e1-4cff-b310-ae29ef50e014.png)
![image](https://user-images.githubusercontent.com/80186561/180648012-a594c29c-e9ca-4ffa-9a7b-a3feea870496.png)


## AUTOSAR概念

SWC（Software Components，应用软件组件）与BSW（Basic Software，基础软件）之间的明确接口是AUTOSAR架构的组成之一。BSW模块提供基本的标准服务，例如总线通信、存储管理、IO访问、系统和诊断服务。

RTE（Runtime Environment，实时运行环境）同样是AUTOSAR架构的组成之一，负责控制SWC之间的连接以及从SWC到BSW的连接。

VFB（Virtual Functional Bus，虚拟功能总线）是AUTOSAR中的概念基础，用于SWC之间的通信以及BSW服务的使用。由于SWC通信全部基于VFB，因此SWC独立于ECU硬件。从而可实现在不同项目和平台之间重用SWC。通过为每个ECU配置RTE以及相应的BSW，从而在实际车辆中实现VFB。

## AUTOSAR分层模型

AUTOSAR对ECU软件进行抽象，并划分为基础软件、实时运行环境和应用软件层。

基础软件包含许多定义好的模块，并划分为不同的层。例如，MCAL（Microcontroller Abstraction Layer，微控制器抽象层）提供用于访问微控制器的存储器、通信和输入/输出（IO）的驱动程序。

ECUAL（ECU Abstraction Layer，ECU抽象层）为访问ECU的所有功能提供统一接口，例如通信、存储或IO，不管这些功能是属于微控制器还是由外围组件实现。

服务层为应用软件层提供不同类型的后台服务，例如网络服务、存储管理和总线通信服务。该层也包含操作系统（operating system）。

RTE将应用软件层从基础软件中抽象出来，控制应用软件层的运行时行为并实现数据交换。在应用软件层中，ECU的应用程序功能以单个应用软件组件的形式实现。

分层模型简化了从软件到各种硬件的移植（porting）。以前，如果软件架构设计不佳，移植需要在各个方面（直至应用软件层）进行大范围修改。基于AUTOSAR，只需替换MCAL中所有微控制器相关的驱动程序即可，重新配置ECU抽象层中的模块，其他所有层均不受移植影响。这大大减少了开发和测试工作以及相关的风险。

![image](https://user-images.githubusercontent.com/80186561/180648420-1b86751e-d6f5-4073-aacb-f73f3963aff1.png)
![image](https://user-images.githubusercontent.com/80186561/180648429-7f89b2e0-9f1f-43ea-852a-9ed6d65ea258.png)
![image](https://user-images.githubusercontent.com/80186561/180648436-55f4faa3-f5cc-4f07-af7c-945d55d189a7.png)
![image](https://user-images.githubusercontent.com/80186561/180648472-d0961586-6baa-4428-a928-30d1b3cbd4e9.png)


## AUTOSAR中的接口定义

AUTOSAR定义三种类型的接口：

- AUTOSAR接口
- 标准AUTOSAR接口
- 标准接口

AUTOSAR接口是通用接口，源自任意SWC的端口。AUTOSAR接口由RTE提供，用于SWC之间或SWC与ECU固件（IoHwAb、复杂设备驱动）之间的接口。例如，SWC可以通过这些接口读取输入值并写入输出值。

标准AUTOSAR接口是由AUTOSAR标准预定义的特殊AUTOSAR接口。SWC使用这些类型的接口访问由服务层的BSW模块（例如ECU状态管理器或诊断事件管理器）提供的AUTOSAR服务。

标准接口是AUTOSAR标准用C语言预定义的API接口，用于连接BSW模块、RTE与操作系统，或RTE与Com模块。

![image](https://user-images.githubusercontent.com/80186561/180648510-88bab198-d12d-4563-b5be-b6190795da61.png)
![image](https://user-images.githubusercontent.com/80186561/180648515-e87c783a-04cf-4ed7-8b9a-8d4b65da1dcd.png)
![image](https://user-images.githubusercontent.com/80186561/180648519-ba2f207a-9e3d-4fd1-a9d3-5ec0fc437979.png)


# 2. AUTOSAR方法论

## 开发阶段

AUTOSAR组织在AUTOSAR方法论中定义了ECU软件的开发方法，将开发过程划分为各种操作，并通过一组XML文件对开发伙伴之间的数据交换进行标准化。

定义SWC并将其分配到ECU，从而在系统设计阶段建立应用程序架构，同时还定义网络通信。由此生成系统描述文件（AUTOSAR XML文件），并由系统描述文件为每个ECU生成系统描述的特定提取。

在ECU开发期间，开发人员实现SWC，并配置BSW和RTE。在配置期间，开发人员定义特定项目所需的基础软件内容，从而优化整个ECU软件。配置后，开发人员会获得ECU配置描述文件（AUTOSAR XML文件），该描述文件与系统描述文件的ECU提取保持一致。

代码生成器根据ECU配置描述文件生成或修改ECU软件的基础应用软件组件，同时还会生成特定的RTE代码。

应用程序的开发与此过程无关。SWC的描述文件（AUTOSAR XML文件）描述SWC的接口。基于这些描述文件，可以单独实现和测试SWC，从而简化整车厂和一级供应商（Tier1）的应用软件集成工作。

![image](https://user-images.githubusercontent.com/80186561/180648835-7cbbb140-2224-4ed6-9fc0-824d347eb321.png)
![image](https://user-images.githubusercontent.com/80186561/180648851-acb57ca3-5dfe-4118-94a8-304b83f245cf.png)
![image](https://user-images.githubusercontent.com/80186561/180648856-d69da572-2ae1-4d43-91c8-3eef012ace08.png)
![image](https://user-images.githubusercontent.com/80186561/180648862-2f795472-cb66-4143-b0a7-b22ef55b31ef.png)
![image](https://user-images.githubusercontent.com/80186561/180648872-0bf095da-c42e-4362-9067-5f4444ab18ac.png)
![image](https://user-images.githubusercontent.com/80186561/180648883-55bea42f-fbab-4a61-8ac3-27da2cce3fc9.png)

## AUTOSAR交换格式

AUTOSAR定义以下交换格式：

- SWC描述文件：
  供应商或整车厂定义SWC，并为每个SWC创建XML文件（SWC描述文件），描述SWC的接口和资源要求。然后，供应商或整车厂创建用于实现的相关C文件。
- 系统描述文件：
  整车厂首先根据SWC定义独立于ECU的整个车辆的功能内容，然后整车厂设计通信网络并将SWC分配到现有的ECU，结果保存在系统描述文件中。

------

从AUTOSAR 4.0开始：

- 系统提取文件：
  对于每个ECU，整车厂会将系统描述文件简化为系统描述文件的系统提取，以便分享给相关ECU的供应商。此文件替换之前用于配置BSW模块的.dbc（CAN总线数据库格式）、FIBEX（现场总线交换格式）或.ldf（LIN总线描述文件格式）文件。
- • ECU提取文件（ECUEX）：
  “扁平化”进程用于根据系统描述文件的系统提取生成系统描述文件的ECU提取。虽然与系统提取一致，但从平面角度来看ECU提取仅包含原子SWC。一级供应商可以使用内部生成的SWC扩充ECU提取。

AUTOSAR 3:

- ECU提取文件（ECUEX）：
  对于每个ECU，整车厂会将系统描述文件简化为系统描述文件的ECU提取，以便传递给相关ECU的供应商。此文件替换之前用于配置BSW模块的.dbc（CAN总线数据库格式）、FIBEX（现场总线交换格式）或.ldf（LIN总线描述文件格式）文件。
- 完整的系统描述文件的ECU摘要：
  以系统描述文件的ECU提取为起点，供应商开始集成自己的SWC，并生成完整且最新的系统描述文件的ECU提取，包含ECU的所有SWC的描述，不管SWC来自整车厂还是供应商。

------

- BSW模块描述文件：
  ECU配置的另一个前提条件是BSW模块描述文件，其中包含数据结构定义以及BSW模块的所有可配置参数。这些文件与具体实现相关，并且与生成器一样，属于AUTOSAR协议栈供应商提供的BSW模块的静态代码内容。
- ECU配置描述文件（ECUC）：
  供应商根据系统描述文件的最新ECU提取文件和BSW模块描述文件创建了初始ECU配置描述文件，并根据该文件配置ECU。这需要设置和检查BSW模块和RTE参数的工具。相关生成器以ECU配置描述文件为基础，生成特定ECU的RTE和BSW模块

灵活的AUTOSAR方法论可适合不同项目和不同整车厂的实际要求。例如，在系统描述文件中可自由选择是否使用SWC。

![image](https://user-images.githubusercontent.com/80186561/180648988-831e428c-d330-4f8c-a4e1-3f5b69b4bd5b.png)


## 功能软件的开发

车辆的功能软件最初被描述为一个整体系统，随后细分为多个子功能，即SWC。这些SWC通过接口（端口）将信息（数据元素）传输给其他SWC。

从概念上讲，SWC之间的通信由VFB处理。由于开发早期尚未确定将哪个应用软件组件分配给哪个ECU，因此这只是一个逻辑上的观点。VFB表示ECU内部以及ECU之间的通信。应用程序并不了解底层技术，这样可以确保应用程序软件的开发和使用不受硬件影响。

设置并定义所有SWC和接口后，将其分配到相关ECU。

然后，特定ECU的实时运行环境作为实现虚拟功能总线的实现，组织应用软件组件之间的通信，并在操作系统的帮助下处理应用软件组件的执行。

可运行实体（Runnable Entity）是执行单元，最终以C函数的形式实现。对可运行实体的函数调用由开发人员配置，然后由RTE实现。例如，可以周期性或自动响应接收数据元素。

AUTOSAR提供端口作为通信接口，并定义了两种通信方式：

- 在SR（Sender-Receiver，发送方/接收方模式）通信中，数据元素从一个应用软件组件传输到另一个应用软件组件。应用软件组件之间经常使用这种通信。 语法示例：Rte_Write_<Port_Name>_<Data_Element_Name(…)
- 在CS（Client-Server，客户/服务器端）通信中，客户端异步或同步调用服务器的操作。这相当于函数调用，常在基础软件的应用程序和服务（诊断、存储管理等）之间发生。 语法示例：Rte_Call_<Port_Name>_<Operations_Name(…)

单独创建的SWC描述文件中记录了SWC及其接口和可运行实体。但AUTOSAR无法描述功能行为。

![image](https://user-images.githubusercontent.com/80186561/180649100-a596ba30-af06-487d-a5d0-395d714093b9.png)
![image](https://user-images.githubusercontent.com/80186561/180649108-40fd366b-d579-4e66-83bd-2e3749ece254.png)
![image](https://user-images.githubusercontent.com/80186561/180649117-3551cfbf-9033-4f96-bfce-48000715e8b0.png)


# 3. 实践中的功能软件

## AUTOSAR工作产品：从整车厂到一级供应商

配置AUTOSAR ECU的基础是ECU提取文件（ECUEX）。这是一个由AUTOSAR定义的XML文件（*.arxml），包含ECU配置的规范，通常由整车厂生成。

AUTOSAR在内容方面提供很大的自由度，具体内容取决于整车厂。以下是可能的常规变体：

- 网络描述（用例1）
- 网络描述和应用软件组件作为规范（用例2）
- 网络描述和实现的应用软件组件（用例3）

一级供应商必须基于ECUEX生成ECU配置描述文件（ECUC），用于保存基础软件的详细配置。

在向AUTOSAR过渡期间，可能并非所有参与者都仅使用AUTOSAR方法。因此，供应商可能会从整车厂接收到.dbc、.ldf或FIBEX数据库，并基于此数据库来生成ECUC。

![image](https://user-images.githubusercontent.com/80186561/180649230-b8743f26-3216-4da5-9afd-69e9c1baec8a.png)
![image](https://user-images.githubusercontent.com/80186561/180649274-eaaec1cf-7e22-48a1-9a7b-2e78ac1d7c5e.png)
![image](https://user-images.githubusercontent.com/80186561/180649340-f126f5bc-5e7c-430f-b62d-2890de670fc2.png)
![image](https://user-images.githubusercontent.com/80186561/180649342-4b0ba84b-ee8c-4843-ada8-a91bb62e960f.png)
![image](https://user-images.githubusercontent.com/80186561/180649347-459b2f78-8847-4a90-8698-dae74f6d1a2e.png)


## 功能软件

在AUTOSAR中，ECU的功能软件通过应用软件组件实现，其核心原理是创建SWC的形式描述（SWC描述文件），然后从中获得SWC的C语言接口。SWC描述文件存储在AUTOSAR定义的XML文件中。

使用以下选项之一创建与SWC描述文件匹配的SWC实现：

- 手写代码开发
  SWC可以通过手写代码实现。
- 基于模型的开发
  模型工具用于创建SWC的行为模型， C代码基于模型自动生成。

SWC的AUTOSAR概念的特点在于SWC的实现具有独立于微控制器的接口，从而为在不同硬件平台上运行SWC提供了所需的技术条件，进而可以更好地在不同ECU中重复使用SWC。当然，由于存在其他限制，因此可能无法在任何的ECU上运行任意SWC。例如，即使提供的接口允许，在车门ECU上运行发动机控制器功能也并不合理。

为在实际开发过程中实现复用和接口兼容， SWC功能的正确至关重要。由于明确定义了SWC的接口，因此可以对SWC执行测试，例如单元测试。这样可以开发一个独立于其他SWC的SWC，然后将其作为经过全面测试的单元存放在库中，甚至可以将SWC作为COTS组件提供。

![image](https://user-images.githubusercontent.com/80186561/180649374-61e0d117-0c21-4656-9af3-26db2504b351.png)
![image](https://user-images.githubusercontent.com/80186561/180649384-4aef3b2f-228e-4756-9678-3cc4b58c2c07.png)
![image](https://user-images.githubusercontent.com/80186561/180649394-2cc5c0b1-1653-4a0f-8217-ce5c3bce8fcc.png)


# 4. 基础软件和RTE

## 基础软件的任务

AUTOSAR系统是通过VFB（或特定于ECU的RTE）互连的应用软件组件的形式描述。此抽象概念可能表明AUTOSAR基础软件仅涵盖通信。情况并非如此。

AUTOSAR基础软件还提供内部ECU服务，例如状态管理（ECU状态和通信通道控制）、诊断服务、看门狗服务、操作系统以及非易失性存储管理，当然也包括IO。

半导体制造商通过MCAL，在IO的标准化中发挥了特殊作用。由于标准化，以前由ECU供应商解决的某些问题现在由基础软件供应商解决。

如下的软件架构图是Vector对AUTOSAR标准的实现，即MICROSAR。

![image](https://user-images.githubusercontent.com/80186561/180649463-d569e634-215d-4a34-83bb-62e6b2def8dd.png)
![image](https://user-images.githubusercontent.com/80186561/180649472-846ba552-8e29-449c-a58d-5d2a9dd04316.png)
![image](https://user-images.githubusercontent.com/80186561/180649486-b98ad70c-3375-466d-aea0-906db79aaa2c.png)
![image](https://user-images.githubusercontent.com/80186561/180649494-831e0efb-56ea-4864-b1cb-156f2281b35e.png)
![image](https://user-images.githubusercontent.com/80186561/180649501-7795ecf4-e1e3-434b-9d61-de69c674503d.png)
![image](https://user-images.githubusercontent.com/80186561/180649511-6b14570b-2055-4fd5-bd1b-eb68b0ac67fb.png)
![image](https://user-images.githubusercontent.com/80186561/180649554-8721e1f8-872e-4d0f-92d1-8acd4c54fca2.png)
![image](https://user-images.githubusercontent.com/80186561/180649559-2be51650-1fa2-4e0a-849c-bc16a4143cb0.png)
![image](https://user-images.githubusercontent.com/80186561/180649572-e5fa7a32-31ea-43b0-9200-15bcb435c2af.png)
![image](https://user-images.githubusercontent.com/80186561/180649578-f507298f-a870-4248-b665-125d7518d810.png)
![image](https://user-images.githubusercontent.com/80186561/180649585-ef865524-b53c-4145-82d2-28fa642cf80f.png)
![image](https://user-images.githubusercontent.com/80186561/180649594-26a2760e-513f-4f93-bda9-1f3d87dd0f19.png)
![image](https://user-images.githubusercontent.com/80186561/180649601-6066bd80-e943-47c0-aaba-d6a9085a10c2.png)
![image](https://user-images.githubusercontent.com/80186561/180649604-40b9ae7b-dc03-4967-8abe-0b137cab531a.png)
![image](https://user-images.githubusercontent.com/80186561/180649627-2f8cec04-0dc2-4a95-8eea-d0e0492dc67a.png)
![image](https://user-images.githubusercontent.com/80186561/180649639-9f37a80f-a353-4d8a-ac4c-b0cc21e1ea2f.png)
![image](https://user-images.githubusercontent.com/80186561/180649643-a09d4019-1fdd-4f5f-8a03-e0a523ee8aa2.png)
![image](https://user-images.githubusercontent.com/80186561/180649647-c0310fcb-1fb6-47c0-baf9-84116dca8fd9.png)
***
![image](https://user-images.githubusercontent.com/80186561/180649746-abc50253-fedd-4a38-b5a9-c64ca937d531.png)
![image](https://user-images.githubusercontent.com/80186561/180649753-07ea0c68-c4a3-4fa7-9f8f-e1227101d3f1.png)
![image](https://user-images.githubusercontent.com/80186561/180649758-064a3eb7-c722-4493-9052-b7137f2d2812.png)
![image](https://user-images.githubusercontent.com/80186561/180649780-4d0c7d46-44ff-46ca-a43f-42284d538db3.png)
![image](https://user-images.githubusercontent.com/80186561/180649791-f473c66e-440b-4eb8-ae3b-1b565c01fc47.png)
![image](https://user-images.githubusercontent.com/80186561/180649794-a7752c6e-c69d-4952-8d7c-26983749fcf5.png)
![image](https://user-images.githubusercontent.com/80186561/180649799-aa2b1200-26ec-44c7-9395-57d4dff97e9e.png)
![image](https://user-images.githubusercontent.com/80186561/180649806-e03aba7e-1ea8-43eb-9d50-2da210084a37.png)
![image](https://user-images.githubusercontent.com/80186561/180649811-f8e47a69-9d92-4df5-ae2d-eec5d96cc701.png)
![image](https://user-images.githubusercontent.com/80186561/180649815-850ff122-3a9f-4fed-965c-974f486ef9b2.png)
![image](https://user-images.githubusercontent.com/80186561/180649820-4d9daadd-1d50-410f-aca1-424d3b398c7f.png)
![image](https://user-images.githubusercontent.com/80186561/180649827-5bdf5354-545c-4e1a-bb63-2323c7754c0f.png)
![image](https://user-images.githubusercontent.com/80186561/180649831-0782d18d-fab3-4f1c-8845-fd99e2efdaea.png)
![image](https://user-images.githubusercontent.com/80186561/180649838-85258d5b-1884-4049-9508-1d0effc5f358.png)
![image](https://user-images.githubusercontent.com/80186561/180649845-50bf80b6-2ce0-4a75-bb2f-1d25969e21a8.png)
![image](https://user-images.githubusercontent.com/80186561/180649859-1682e037-1c22-4d94-b26f-10c2974bf6cf.png)
![image](https://user-images.githubusercontent.com/80186561/180649870-1f06cd22-2cd4-4564-91c5-bc15ccd47063.png)
![image](https://user-images.githubusercontent.com/80186561/180649876-e46dee21-9efa-4cf3-a41a-0788a0c759d1.png)
![image](https://user-images.githubusercontent.com/80186561/180649879-fa66e20c-4c9e-45f3-996d-f7f72ac18da1.png)
![image](https://user-images.githubusercontent.com/80186561/180649884-7a2486cf-d765-4c51-9c80-56b0e352115f.png)
***
![image](https://user-images.githubusercontent.com/80186561/180649895-c5db174e-016e-4449-95e0-45482cb52db7.png)


## 基础软件的属性

只有在BSW提供所需机制时，才能以应用软件组件的形式进行抽象。

因此，BSW会通过这些机制对RTE进行补充。在与RTE及其操作紧密交织的通信协议栈和操作系统方面，这一点表现得尤其明显。

例如，BSW必须生成事件并为RTE提供计时器。BSW还通过通信总线将数据传输到其它ECU。在执行应用软件组件的可运行实体时，程序流控制和系统状态管理都是BSW的任务。同时，BSW还提供同步机制，序列化并行进程的访问。



## RTE及其最佳配置

实现应用软件组件的描述文件中所包含的通信和调用机制需要高效的实时运行环境（RTE）。

SWC的形式描述允许软件设计的自动分析以及实时运行环境的派生、生成和优化。

软件设计的形式化描述中所包含的信息包括调用可运行实体时所在的上下文，以及可运行实体与同一SWC或另一个SWC的其他部分进行交互的方式。

通过考虑基础软件的配置等其他限制，可以决定如何以最佳方式实现函数调用。

基本上，必须做出以下方面的决定：

- 如果需要，选择阻止机制，以阻止并行运行的其他可运行实体进行访问。
- 调用方法。
  可以（以宏或C函数的形式）直接进行调用，也可以通过与操作系统一起触发的RTE事件进行调用。
- 读写访问类型。
  可能是对变量的访问，也可能是对基础软件的API的调用。此外，必须考虑不同访问类型的语义及其实现机制。

根据具体配置，这些决定可能会对性能产生重大影响。

一般来说，实时运行环境的生成器应尽量少用OS事件和警报。适当配置系统可显著节省资源和减少执行时间。为此，需要了解在应用软件组件设计阶段每个决定带来的影响。

## 基础软件中的OEM依赖项

AUTOSAR基础软件的特点之一就是高度模块化，表现在水平方向的不同区域（集群）中，以及垂直方向的不同抽象层中。AUTOSAR允许使用基础软件的不同粒度（一致性分类，ICC），因此可将BSW模块组合（群集化）为单体式基础软件，该软件仅由一个包含全部基础软件功能的模块构成。

AUTOSAR基础软件不一定具有整车厂自定义特性，但在某些方面，各个整车厂的BSW协议栈通常有所不同。

例如，在非AUTOSAR标准的特定BSW模块的数量和任务方面，协议栈可能会有所不同，也可以通过添加应用软件组件的形式对AUTOSAR基础软件进行功能扩展。

在基础软件的结构中，此类差异发生在以下区域：

- DIAG：诊断事件管理器、诊断通信管理器
- SYS：通信通道处理
- COM：网络管理
- COM：网关功能
- 加密模块、专有传输协议等特定服务。

在基础软件的配置方面，不同整车厂的工作流程也有所不同：

- 整车厂需求定义的方式（.dbc文件、系统描述文件的ECU提取或单独的SWC描述文件）
- 诊断布局和参数设置（ODX、CANdela文件等）
- 对供应商的其他要求，例如通信协议栈发布后的配置能力（Post-build）、以库的形式交付

如下的软件架构图是Vector对AUTOSAR标准的实现，即MICROSAR。

![image](https://user-images.githubusercontent.com/80186561/180650186-cdb86880-f378-4ab9-bb63-3e155b4b30bd.png)
![image](https://user-images.githubusercontent.com/80186561/180650222-cf6f2c4a-af77-42f2-b51c-06da1ab7a83e.png)
![image](https://user-images.githubusercontent.com/80186561/180650232-7a59d9d2-fe0e-480f-af4b-cd1704d409a8.png)
![image](https://user-images.githubusercontent.com/80186561/180650233-b8560c0b-f578-49a3-8db8-1ec101ffa561.png)
![image](https://user-images.githubusercontent.com/80186561/180650252-d50b77ef-27f1-435e-8232-f60b8195e94f.png)
![image](https://user-images.githubusercontent.com/80186561/180650263-aef63926-2b8a-408c-81ff-6ad81491a49d.png)
![image](https://user-images.githubusercontent.com/80186561/180650268-881ec605-50a6-4780-8580-190eb1bb5974.png)
![image](https://user-images.githubusercontent.com/80186561/180650297-63afa427-0151-4f88-96a8-5bf8dcf05448.png)



## 基础软件从何而来

供应商处理基础软件的方式因整车厂而异。

例如，在某些情况中，整车厂已从特定软件提供商购买了硬件无关的基础软件模块。在这种情况下，供应商只需要采购硬件相关模块。在其他情况下，整车厂会为供应商免费提供基于特定目标平台的，已经预集成的基础软件包。

整车厂也可能仅指定基础软件模块的功能，ECU的技术实现和集成则完全由供应商负责。供应商可以向各个软件提供商采购模块。然后，供应商可以自行集成所购买的模块，也可以让软件供应商完成此项工作。

一些大型供应商已经开发了自己的AUTOSAR基础软件，因此也可以充当内部的软件提供商。根据与整车厂的合作关系，可能需要就此软件的使用进行具体协商。

![image](https://user-images.githubusercontent.com/80186561/180650322-f166140f-1ecc-4499-90b2-e17706663c84.png)


# 5. 工具、移植和测试

## 工具

创建AUTOSAR ECU软件需要使用不同的工具，可大致分为以下类型：

- 系统设计工具
  用于定义网络架构和通信，以及SWC的设计和分发。
- 基础软件和RTE配置工具
  用于创建ECU的BSW模块的配置描述文件（ECU配置描述文件）。
- BSW模块的代码生成器
  根据ECU配置描述文件，为ECU生成专门配置的BSW模块。

这些任务通常由不同的工具处理。因此，标准化的XML格式是AUTOSAR的一个重要组成部分，并且是不同工具之间交换设计和配置数据的基础。

这种标准化至关重要，因为同一开发项目中通常会使用不同制造商的工具。例如，独立于微控制器的BSW可能来自一家软件公司，而MCAL及其相关的代码生成器则由半导体厂商提供。

每个BSW模块可能会使用不同的工具。但从实践角度来看，建议使用统一的工具配置BSW。

![image](https://user-images.githubusercontent.com/80186561/180650373-9d3eb369-7e27-4f5f-adce-d2cc64e9932c.png)
![image](https://user-images.githubusercontent.com/80186561/180650413-1fed8ce7-e07f-4de2-ab6f-852de5f7f2ee.png)


## 移植解决方案

AUTOSAR标准支持将非AUTOSAR方法开发的ECU软件移植到AUTOSAR体系中。为此，AUTOSAR定义了特定的复杂驱动（Complex Driver）。

复杂驱动可以理解为特殊类型的SWC，不需要基于SWC模板的形式化描述。

复杂设备驱动无需使用RTE，可以直接访问AUTOSAR基础软件。这意味着，从应用程序的角度来看，“仅”基础软件发生了更改，而应用程序可以在很大程度上保持不变。在移植的框架中，也可以将应用程序视为复杂设备驱动。这是迈向AUTOSAR软件架构的第一步。从开发工作的角度来看，第一步最具成本效益。

使用这种方法，虽然可以从部分AUTOSAR功能中获益，例如，通过RTE定期调用应用程序的内容，或经RTE进行通信和诊断，但应用程序核心的实现仍不符合AUTOSAR标准。

从长远来看，应用程序中未按AUTOSAR标准建模的部分将被消除，特别是所有任务主体和对操作系统的调用，以及所有具有中断阻塞的点和其他基础软件的访问。可使用符合AUTOSAR的标准的元素将其替代。如果使用完善的设计方法，可以比以前更有效地实现这些应用程序。

![image](https://user-images.githubusercontent.com/80186561/180650466-d4655649-0561-4a5d-8108-e549c70c5070.png)
![image](https://user-images.githubusercontent.com/80186561/180650547-1a38c3b6-6220-4732-94a0-90be53747fb5.png)


## 测试AUTOSAR ECU

测试期间，同样的测试方法可以用于AUTOSAR ECU和非AUTOSAR ECU。将ECU视为黑盒时，只需考虑以下事项：

- 网络管理：
  AUTOSAR定义专属的NM（Network Management，网络管理）协议，不同于以前使用的OSEK NM等协议。测试环境必须考虑此NM协议，并正确准备和处理网络通道的相关报文。
- 用于描述网络通信的文件格式：
  AUTOSAR网络通信描述文件属于系统描述文件。根据整车厂的要求，.dbc、FIBEX或.ldf等以前的格式将替换为新格式。测试环境必须能够处理这种格式。

AUTOSAR的标准化内部软件架构确保每个AUTOSAR ECU中都存在某些状态变量，并可在测试环境中使用这些变量从而为测试和调试ECU提供附加价值。例如，EcuM模块中提供的ECU状态，以及ComM模块中存储的各个网络通道的通信状态。通过适当配置BSW模块，可以通过XCP与ECU的连接（例如通过某一网络或者JTAG或Nexus等调试接口）来访问这些状态变量。BSW生成器可以提供这些状态变量的匹配描述文件（A2L）。作为替代方案，还可以使用AUTOSAR为此专门定义的监控和调试协议。

AUTOSAR在访问应用程序级别方面也提供了好处。例如，可以生成RTE，从而能够访问SWC之间交换的数据。同样，RTE生成器也可以生成合适的A2L文件。

![image](https://user-images.githubusercontent.com/80186561/180650577-e5f552c1-6637-461d-b8f0-a5fa57a648a8.png)
![image](https://user-images.githubusercontent.com/80186561/180650588-cbb401f5-a8e1-4119-a421-705084254c25.png)
