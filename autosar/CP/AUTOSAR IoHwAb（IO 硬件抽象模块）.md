# AUTOSAR IoHwAb（I/O 硬件抽象模块）

# 1. 简介和功能概述

本文介绍了**AUTOSAR**基础软件**I/O**硬件抽象（**I/O Hardware Abstraction**）的功能和配置。**I/O**硬件抽象是**ECU**抽象层（**ECU Abstraction Layer**）的一部分。

**I/O**硬件抽象不应被视为单个模块，实现上此模块可以被实现为多个模块。 **AUTOSAR**的**I/O**硬件抽象标准并未为了实现模块或模块组的标准化。相反标准的主要是为实现和其他模块的功能之间接口的指南。

**I/O**硬件抽象的目的是通过将**I/O**硬件抽象端口映射到**ECU**信号来提供对**MCAL**驱动程序的访问。提供给软件组件的数据可以完全从物理层中抽象出来。因此，软件组件设计人员不再需要详细了解**MCAL**驱动程序的**API**和物理层数据值的单位。

**I/O**硬件抽象始终针对**ECU**的特定实现，软件组件（**software component**）对基础软件（**basic software**）的需求必须适用具体**MCAL**实现的特征。

**I/O**硬件抽象需提供初始化整个**I/O**硬件抽象的服务。

在**AUTOSAR**的**I/O**硬件抽象指南中包含：

- 在定义**I/O**硬件抽象时，应使用软件组件模板（**Software Component template**）的哪一部分。
- 解释如何定义用于**ECU**信号映射的通用端口的方式。

在**AUTOSAR**的**I/O**硬件抽象指南中不包含：

- 提供**C**语言的**API**的定义
- 为每个**ECU**信号提供特定的形式化。此信息可在（车身域、动力总成、底盘域）的标准化的通过功能数据中找到。

# 2. 缩略语及术语

## 2.1. 缩略语

**BSW**

> 基础软件（**Basic SoftWare**）

**BSWMD**

> 基本软件模块说明（**Basic SoftWare Module Description**）

**DET**

> 默认错误跟踪器（**Default Error Tracer**）

**IoHwAb**

> 输入/输出硬件抽象（**Input/Output Hardware Abstraction**）

**ISR**

> 中断服务程序（**Interrupt Service Routine**）

**MCAL**

> 微控制器抽象层（MicroController Abstraction Layer）

**SWC**

> 软件组件（**SoftWare Component**）

## 2.2. 术语

**Callback**

> 术语回调函数（**Callback**）主要用于**API**服务，实现通知其他**BSW**模块。

**Callout**

> **Callout**是一种函数**Stub**。主要在配置时进行填写，目的是向提供**Callout**函数的模块添加功能。

**Class**

> 一个类（**Class**）代表一组具有相似电气特性的信号。如：模拟类（**Analogue class**）

**Client-Server communication**

> 客户端-服务器通信（**Client-Server communication**）涉及两个实体。客户端是要求服务的一方，服务器是提供服务的一方。客户端发起通信，请求服务器执行服务，必要时传输参数集合。服务器以**RTE**的形式，等待来自客户端的传入通信请求，执行请求的服务并发送对客户端请求的响应。因此服务启动的方向是用来对**AUTOSAR**软件组件是客户端还是服务器分类的主要标识。

**Sender-receiver communication**

> 发送方-接收方通信涉及原子数据元素组成的信号的发送和接收，这些原子数据元素由一个组件发送给另一个或多个组件。发送方-接收方接口可以包含多个数据元素。发送方-接收方通信是单向的 接收方发送的任何回复都可以视为一个单独的发送方-接收方通信。接收方组件的端口可以读取接口描述定义的数据元素，而提供方组件的端口可以按照接口描述定义写入数据元素。

**Electrical Signal**

> 电信号（**Electrical Signal**）代表**ECU**引脚上的物理信号。

**ECU pin**

> ECU引脚（**ECU pin**）是**ECU**与电子系统其余部分的电气硬件连接。

**ECU Signal**

> ECU信号（**ECU Signal**）是电信号的软件表示。一个**ECU**信号具有一些属性（**Attribute**）和一个符号名称（**Symbolic name**）。

**ECU Signal Group**

> ECU信号组（**ECU Signal Group**）是一组电信号的软件表示。

**Attributes**

> 属性（**Attributes**）是**ECU**中存在的每个**ECU**信号所包含的软件或者硬件特性。一些属性是固定的，由端口定义决定。还有一些属性是可以在**I/O**硬件抽象中进行配置的。

**Symbolic name**

> ECU信号的符号名称（**Symbolic name**）在**I/O**硬件抽象模块中是用来建立链接的。（包括：功能、引脚等）

## 2.3. ECU信号属性

**Range**

> 范围（**Range**）在这里是一个功能范围（**functional range**），而不是电气范围（**electrical range**）。所有范围都用于功能的需要或者诊断的检测。对于模拟**ECU**信号，可以定义上下限值 [lowerLimit…upperLimit]（电压、电流）。对于电阻信号和时序信号（周期）的特殊情况，下限值（**lowerLimit**）不能为负。

**Resolution**

> 分辨率（**Resolution**）属性适用于依赖于范围和数据类型的许多类别。如：(upperLimit - lowerLimit) / (2datatypelength -1)。对于其他类别，它可能是已知和被定义的。如：[-12 Volts…+12Volts]，数据类型: 16 bits，**Resolution** 为 24 / 65535。

**Accuracy**

> 精度（**Accuracy**）取决于用于采样或者生成的硬件外围设备。如：**ADC**转换器可以是 8/10/12/16 位转换器

**Inversion**

> 物理值与逻辑值的反转（**Inversion**）。此属性不可见，但可以由**I/O**硬件抽象来实现，并向用户传递预期值。

**Sampling rate**

> 采样率（**Sampling rate**）是获得信号值所需的时间段。

# 3. 相关文档

## 3.1. 输入文件

[1] List of Basic Software Modules

> AUTOSAR_TR_BSWModuleList.pdf

[2] Layered Software Architecture

> AUTOSAR_EXP_LayeredSoftwareArchitecture.pdf

[3] General Requirements on Basic Software Modules

> AUTOSAR_SRS_BSWGeneral.pdf

[4] Specification of ECU Configuration

> AUTOSAR_TPS_ECUConfiguration.pdf

[5] Glossary

> AUTOSAR_TR_Glossary.pdf

[6] General Requirements on SPA

> AUTOSAR_SRS_SPALGeneral.pdf

[7] Requirements on I/O Hardware Abstraction

> AUTOSAR_SRS_IOHWAbstraction.pdf

[8] Software Component Template

> AUTOSAR_TPS_SoftwareComponentTemplate.pdf

[9] Specification of RTE Software

> AUTOSAR_SWS_RTE.pdf

[10] Specification of ECU State Manager

> AUTOSAR_SWS_ECUStateManager.pdf

[11] Specification of ECU Resource Template

> AUTOSAR_TPS_ECUResourceTemplate.pdf

[12] Specification of ADC Driver

> AUTOSAR_SWS_ADCDriver.pdf

[13] Specification of DIO Driver

> AUTOSAR_SWS_DIODriver.pdf

[14] Specification of ICU Driver

> AUTOSAR_SWS_ICUDriver.pdf

[15] Specification of PWM Driver

> AUTOSAR_SWS_PWMDriver.pdf

[16] Specification of PORT Driver

> AUTOSAR_SWS_PORTDriver.pdf

[17] Specification of GPT Driver

> AUTOSAR_SWS_GPTDriver.pdf

[18] Specification of SPI Handler/Driver

> AUTOSAR_SWS_SPIHandlerDriver.pdf

[19] Basic Software Module Description Template

> AUTOSAR_TPS_BSWModuleDescriptionTemplate.pdf

[20] Specification of Standard Types

> AUTOSAR_SWS_StandardTypes.pdf

[21] General Specification of Basic Software Modules

> AUTOSAR_SWS_BSWGeneral.pdf

[22] Specification of OCU Driver

> AUTOSAR_SWS_OCUDriver.doc

## 3.2. 相关规范

**AUTOSAR**标准提供了基础软件模块的通用规范 [21]，此文档也适用于**IO** 硬件抽象模块。因此规范[21]也应被视为**IO**硬件抽象的附加和必需的规范。

# 4. 对其他模块的依赖

## 4.1. 与MCAL驱动程序的接口

### 4.1.1. 概述

下图显示了**I/O**硬件抽象模块。它位于**MCAL**驱动程序之上。这意味着**I/O**硬件抽象模块将调用驱动程序的**API**来管理片载设备。**MCAL**驱动程序的配置取决于**SWC**所需的**ECU**信号的质量。例如：当引脚电平发生相关变化（上升沿、下降沿）时，可能需要发出通知。系统设计人员必须配置**MCAL**驱动程序以允许通知给定信号。通知由**MCAL**驱动程序生成并在**I/O**硬件抽象模块中处理。

注意，**I/O**硬件抽象模块并非为了抽象**GPT**功能，而是使用**GPT**功能来执行自己的功能。下图显示了**I/O**硬件抽象模块与**GPT**驱动程序的接口，**GPT**驱动程序是**MCAL**的一部分。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97tx2O9icauNy2CqkPryNZvQ9wA7uJKosN8O8sf1RPY1tsQeGlqPHjcLJOQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**4.1.2. 与MCAL驱动程序的接口摘要**

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97txwO79F5hoIjUJh8s3Jobyib42mic2lL0JicwTG7w9Cy1DpsYrfJiaakHc9w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

- **I/O**硬件抽象调用**ADC**驱动程序的**API**
- **I/O**硬件抽象接收来自**ADC**驱动程序的通知。
- **I/O**硬件抽象不接收来自**DIO**驱动程序的通知。

## 4.2. 与通信驱动程序的接口

如果对板载设备进行管理，**I/O**硬件抽象实现需为软件组件提供对通信驱动程序的访问（例如：通过**SPI**通讯）。

下图显示了**I/O**硬件抽象，其中：

- 接受一些来自**SPI**处理程序的信号。
- 使用**SPI**处理程序进行设置。

根据分层软件架构（**Layered Software Architecture**）[2] (**ID03-16**)，I/O 硬件抽象包含专用驱动程序来管理外部设备，例如：

- 通过**SPI**连接外部**ADC**驱动程序。
- 通过**SPI**连接在ASIC设备上的**I/O**驱动程序。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97txDicOOvDY7eV0vXfZWzOvbFxW7pc9uRkGP4TD3ww66MJE8uqgxBic75WA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## 4.3. 与系统服务的接口

**I/O**硬件抽象模块需与以下系统服务进行交互：

- **ECU**状态管理器（初始化功能）
- **DET**：默认错误跟踪器
- **BSW**调度程序



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97txLk6XAFSO2oDPDribgIy6L3MdtKGDOdO2qkTiaypI39vUIAQ62XALhLBQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## 4.4. 与DCM的接口

**I/O**硬件抽象模块需提供与**DCM**的接口，为软件组件提供功能诊断。**DCM**将使用功能诊断来读取和控制已实现的**ECU**信号。提供给**DCM**的接口原型，应在每个**ServiceComponent**的头文件IoHwAb_Dcm.h中定义。

# 5. 功能规格

## 5.1. 集成代码

**I/O**硬件抽象作为**ECU**抽象的一部分，被定义为集成代码。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97txT9M7Ux5Ewys7x03W1pFOuibUvJSE3K5NMmFGX1lpqDRQ5RwXPbfKswg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### 5.1.1. 背景

根据**AUTOSAR**词汇表[5]，集成代码是**ECU**原理图相关的软件，位于**AUTOSAR RTE**下方。

### 5.1.2. 需求说明

集成代码通常意味着该软件是为了适应特定的**ECU**硬件Layout。所有保护硬件的策略都应包含在此软件包中。

硬件保护意味着，当在某个输出上检测到故障时，**I/O**硬件抽象模块需切断输出信号。故障包括：接地短路、电源短路、过热、过载等。

**I/O**硬件抽象模块不需包含故障恢复策略。故障恢复操作应由负责的**SWC**决定。

**I/O**硬件抽象的内部行为是项目特定的，所以无法标准化。**I/O**硬件抽象模块不包含可扩展性。**SWC**定义需要什么信号已经信号质量，而**I/O**硬件抽象模块提供相关服务。

## 5.2. ECU信号概念

### 5.2.1. 背景介绍

**I/O**硬件抽象模块并不会为**AUTOSAR SW-C**提供标准化的**AUTOSAR**接口，因为它与上层SW-C的接口强烈依赖于信号的采集链。但**I/O**硬件抽象模块会提供**AUTOSAR**接口。这些**AUTOSAR**接口用来表示来自**ECU**输入的电信号或者定位到**ECU**输出的电信号的抽象。

同时，这些电信号也可以来自其他**ECU**或被定位到其他的**ECU**（例如：通过**CAN**网络）。

端口（**Port**）是**AUTOSAR**组件的入口点。端口由**AUTOSAR**接口（AUTOSAR interface）来表示。这些接口对应于每个**ECU**信号。

ECU信号的概念来自于保证硬件平台互换性（**interchangeability of hardware platforms**）的必要性。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97txxgQWsKV1HmNCbl4rah9zgsojyAUlm5ibQpY4BTomaewI70z3WkkMA1Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### 5.2.2. 需求说明

**I/O**硬件抽象模块处理直接连接到**ECU**的所有输入和输出，但不包含那些具有专用驱动程序的输入和输出。（如:**CAN**驱动）。它所包括的所有输入和输出，会直接映射到微控制器端口或一个板载外围设备上。所有微控制器和外围设备之间的通信（但不包括传感器、执行器以及由复杂驱动程序管理的外围设备）都需被**I/O**硬件抽象模块隐藏，并提供相应的访问接口（**interfaces**）。

**ECU**通过网络以及输入和输出引脚连接到系统的其余部分。但网络并不包含在**I/O**硬件抽象模块的范围内。

一个**ECU**信号代表一个电信号，这意味着至少代表了一个输入或输出的ECU引脚。

在该层中实现的软件需对**ECU**引脚进行抽象化。通过使用示波器观测，输入和输出只不过是一个电信号，所以**I/O**硬件抽象模块定义的所有内容都与电信号的概念有关。对这个概念的一个扩展是诊断（如：电气故障状态）。通过**ECU**连接器（**connectors**）诊断并不可见，但**I/O**硬件抽象模块会提供相关服务。

具有类似行为的电信号可以形成一个类别。因此通过软件表示的电信号的**ECU**信号，可能与某个特定于实现的类相关联。

## 5.3. 属性

尽管每个**ECU**信号的大部分特性都是由**SW-C**定义的，但必须为每个信号添加一些属性以实现**SW-C**期望的信号质量。

为了详细说明信号采集链，需定义了一个属性列表来识别**ECU**信号的可配置特征。

### 5.3.1. 过滤/去抖动（Filtering/Debouncing）属性

所有**ECU**信号都应具有过滤（**Filtering**）/去抖动（**Debounce**）的属性。以便于所捕获的原始值（**raw value**）在传递到上层之前可以被过滤或去抖动。此属性仅对输入信号有效。它会影响信号值采集和访问的具体实现。

### 5.3.2. Age属性

由**I/O**硬件抽象模块处理的所有**ECU**信号都取决于**ECU**硬件设计。这意味着设置**ECU**输出信号的时间和获取**ECU**输入信号的时间，可能每个信号都是不同的。因此为了保证所有类型的**ECU**信号（输入/输出）的模板行为，定义了一个通用的年龄属性，并且需为每个**ECU**信号配置它。

所有**ECU**信号都应具有年龄属性。根据**ECU**信号的方向（如：输入或者输出），年龄属性有两个特定的名称。但无论如何它总是包含一个最大时间值。

下面的描述解释了该属性对每种**ECU**信号的含义:

- **ECU**输入信号：该属性的具体功能是限制信号的寿命。该值定义了此信号数据的最大允许年龄。如果生命周期为**0**，则必须立即从物理寄存器中检索信号。如果生命周期大于**0**，则信号在指定时间内有效。
- **ECU**输出信号：此属性的具体功能是将信号输出限制为最大延迟。该值定义了实际设置此信号之前的最大允许时间。如果延迟为**0**，则必须立即将信号设置到物理寄存器中。如果延迟大于**0**，则可以等待配置的时间到达后，才将信号设置到物理寄存器中。

## 5.4. I/O 硬件抽象和软件组件模板

### 5.4.1. 背景

本章描述的内容和参考文档[8]相关，当参考文献档的内容有更改时，可能会影响本章描述的内容。

本章总结了如何将**I/O**硬件抽象模块定义为一个软件组件（**SW-C**），并简要概述其内部行为。此内部行为描述主要涵盖**BSW**调度（**BSW scheduling**）机制。

### 5.4.2. 概述

**I/O**硬件抽象模块需基于参考文档[8]中指定的软件组件模板（**Software Component Template**）。

与任何其他软件组件一样，**I/O**硬件抽象模块可能是子结构的（**sub-structured**），具体需取决于**ECU**的复杂性。

事实上，**I/O**硬件抽象模块属于一个经典的组件原型（**Component Prototype**），它可以是原子型的（**atomic**），也可以时组合型的（**composed**）。它定义了提供和所需要的接口（**interface**）。此外，**I/O**硬件抽象模块只能通过其**PortPrototypes**与**RTE**之上的其他软件组件进行交互。任何使用**PortPrototypes**表达的隐藏依赖项是不被允许的。

**I/O**硬件抽象模块，通过标准化接口与**MCAL**驱动程序相连接，而通过**RTE**与其他软件组件相连接。所以**I/O**硬件抽象模块需使用虚拟端口的概念（**Virtual Ports Concept**）。

**I/O**硬件抽象应实现为**EcuAbstractionComponentType**的一个或多个实例。具体有关**EcuAbstractionComponentType**的更多信息，可参阅参考文档[8]。**EcuAbstractionComponentType**的实例化提供了一组端口。在**RTE**生成期间，仅需考虑与软件组件连接的那些端口。

### 5.4.3. 端口概念

参考AUTOSAR标准，建议**I/O**硬件抽象端口需使用软件组件模板来进行定义。因此具体的使用细节可参考软件组件模板文档[8]，以了解更多的使用的术语和概念。同时，**I/O**硬件抽象的属性（**Attribute**）定义需通过使用**IoHwAbstractionServerAnnotation**来进行定义。

### 5.4.4. 软件组件和可运行概念

软件组件具有实现其策略和内部行为的功能。这些部分使用可运行实体进行描述。前者包含在**runnables**中，而后者取决于**runnables**设计。可运行实体由原子软件组件提供，并且（至少间接地）是底层操作系统调度的主题。

原子软件组件的实现必须在其内部行为（**InternalBehavior**）中为每个**Runnable**提供代码入口点。有关详细信息，可参阅规范[8]。

可运行实体是最小的代码片段，可以独立激活。它们由原子软件组件提供并由**RTE**激活。例如：**Runnables**.被设置为响应服务器上的数据交换或操作调用。可运行实体具有三种可能的状态：**Suspended**、**Enabled** 和 **Running**。在运行时，原子软件组件的每个可运行对象（通过成为**OS**任务的成员）处于这些状态中的某种状态。

有关定义原子软件组件的每个可运行文件的可用选项和属性，可参阅规范[8]。

## 5.5. 调度概念

### 5.5.1. 概述

**I/O**硬件抽象模块可能会包含几个**BSW**模块，（例如：板载设备驱动程序）。这些**BSW**模块中的每一个都可以提供**BSW**可运行实体，在**RTE**规范中也称为**BswModuleEntity**，具体内容可参见参考文档[9]。

为了实现并行运行，**BswModuleEntity**等效于**SWC**的可运行实体（**runnable entities**），在**AUTOSAR**词汇表 [5] 给出了以下定义：可运行实体是可以执行的原子软件组件的一部分，它可以独立于该原子软件组件的其他可运行实体，被执行及调度。

这意味着**I/O**硬件抽象可以同时使用**Runnable Scheduling**和**BSW Scheduling**。**Runnable Scheduling**处理**Runnable Entities**并且是强制性的。与**Runnable Scheduling**不同，**BSW Scheduling**是可选的，与**BSW**调度器的接口必须手动完成。

对于**SWC**的可运行实体，这些实体会在**AUTOSAR OS**任务主体中被调用。**Runnables**在**SW-C**描述中给出。**SW-C**的**Runnables**激活很大程度上取决于**RTE**的事件。

与**SWC**经常由**RTEEvents**激活的方式相同，可调度的**BswModuleEntities**可以由**BswEvents**激活。还有一种**BswModuleEntity**可以在中断的上下文（**interrupt context**）中被激活。这引出了两个子类别：**BswSchedulableEntity**和**BswInterruptEntity**。

### 5.5.2. Ports提供的接口的操作

**I/O**硬件抽象模块是通过接口（**interface**）来进行描述，在SW-C的实现中可对应与**PortInterfaces**。在可运行实体（**Runnable Entities**）中实现了**SW-C**所需的提供端口，包括了服务器端口（**Server port**）及发送方/接收方端口（**Sender/Receiver port**）。

**I/O**硬件抽象的提供端口服务的实现是特定于**ECU**的，与相应的**PortInterface**的映射需文档化在软件组件描述中。

### 5.5.3. Get操作

对于与配置为输入信号的**PortInterface**关联的**ECU**信号，**I/O**硬件抽象应提供**GET**操作，并且操作的名称（**operation short name**）可以自由选择。

### 5.5.4. Set操作

对于与配置为输出信号的**PortInterface**关联的**ECU**信号，**I/O**硬件抽象应提供**SET**操作，并且操作的名称（**operation short name**）可以自由选择。

### 5.5.5. 通知和回调

**I/O**硬件抽象需定义**BswInterruptEntities**（与 **BswSchedulableEntity** 相对的 **BswModuleEntity** 的子类），以实现通知（**notification**）或回调（**callback**）机制，在中断上下文中实现与RTE下层的其他模块进行数据交换。

**I/O**硬件抽象可能包含一个或多个回调函数。可用的回调函数需要连接到**MCAL**驱动程序的通知接口。所以这些函数必须尊重**MCAL**驱动程序的原型定义，即：没有传递参数，也没有返回参数。

实现必须考虑到回调函数将在中断的上下文中执行的情况。回调函数还需支持可以提供触发**I/O**硬件抽象之外的软件组件的能力。这些通知需要通过**RTE**的发送方端口（**sender port**）进行处理。

可用回调函数的数量和执行顺序取决于实现，并且需要文档化在**I/O**硬件抽象的**BSWMD**描述中。

通过**RTE**路由的**I/O**硬件抽象的回调函数的函数原型需按照以下规则实现：

```
StdReturnType Rte_Call_<p>_<o>(<parameters>)
```

回调函数必须与**RTE**的**Rte_Call_<p>_<o>**的**API**兼容，并启用**AUTOSAR**服务和**IO**硬件抽象的类型安全配置及实现。

### 5.5.6. 主函数/作业处理函数

**I/O**硬件抽象模块可能包含一个或多个作业处理函数，它们是**BswSchedulableEntities**（与**BswInterruptEntity** 相对的**BswModuleEntity**的子类）。例如，每个设备驱动程序对应一个作业处理函数。它们需根据其用途被激活。

这些作业处理函数将由**BSW**调度程序进行时间触发（**time-triggered**）。它们可以与其他可运行实体同步执行。**BswSchedulableEntities**的数量及其执行顺序将取决于实现，并且必须文档化在**I/O**硬件抽象**BSWMD**描述中。

### 5.5.7. 初始化、去初始化和Callout

**I/O**硬件抽象需定义**BswModuleEntries**，实现在非中断上下文中与**RTE**下层的其他软件交换数据，例如：在**BSW**初始化和去初始化的这些情况。这些**BswModuleEntries**链接到一个专用的**BswModuleEntity**，它将被调用以执行服务或者交换数据。

**I/O**硬件抽象可能包含一个或多个初始化和去初始化函数。例如：每个设备驱动程序定义一个初始化和去初始化函数。与**MCAL**驱动程序类似，初始化函数应包含一个参数，以便能够将不同的配置传递给设备驱动程序。该函数应将**I/O**硬件抽象驱动程序使用的所有局部和全局变量初始化为初始状态。

初始化和去初始化函数应由**ECU**状态管理器专门使用及处理。有关详细信息，可参阅参考文档[10]。

可用函数的数量和执行顺序取决于实现，并且需要文档化在**I/O**硬件抽象的**BSWMD**描述中。

### 5.5.8. 调度示例1 - ADC采样

以下示例展示了**ADC**转换的调度示例。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97tx8w32q1oicvpSNttdSdx1Zh62rFlEIXzF6ZCeGcgQbXZ2gJsWbuExFNw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**I/O**硬件抽象提供了两个**P-port**。范例中的软件组件接口是**af_pressure**。

**ECU**状态管理器能够触发**BswModuleEntry**来初始化**ADC**驱动程序，通过使用**Adc_ConfigType**结构调用**Adc_Init**函数。

软件组件获取**af_pressure**值，步骤如下：

1. **RTE**触发专用**P-Port** 的**OP_GET**操作。

2. **R1**是一个可运行实体，它允许调用相关的**ADC**驱动程序服务：

3. - ADC_EnableNotification
   - ADC_StartGroupConversion

4. 转换结束时，**ADC**在中断上下文中触发**R2**这个**BswModuleEntry**。因为该接口允许通知，在**ADC**驱动程序中指定**ADC_NotificationGroup**调用的**callout**函数。

5. 然后通过**RTEevent**将通知发送到软件组件。

以下是示例的时序图：



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97txVgCfPnGzxB1qgUI2PSBK1mRon4wHjQrfXOj0SM8YJq2HSLjiaX5OdEw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



### 5.5.9. 调度示例2 - 同步调度

以下示例展示了用于设置一个连接到**SMART**电源的灯的调度。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97txqvnL6CRhtSOdWBqyYyQ2oaNORKBEcP85plhF712zONiaBP1NHKThXcQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



因为**SMART**电源通过**SPI**总线连接到微控制器，所以专用代码需使用**SPI**处理程序。

**FrontLeftLamp**的设置值通过**RTE**，保存在**I/O**硬件抽象模块的缓冲区中。

而另一个**SMART**电源的输出线，在设置时会同步触发ADC驱动对相同电信号进行**ADC**转换。在转换结束后，转换结果变成可用，同时通知设置到模拟输入管理器（**Analog input manager**）将值存储在缓冲区内，以便用于硬件的诊断。

在这个例子中，周期性处理是通过一个**BswSchedulableEntity**来实现的。

## 5.6. I/O 硬件抽象层描述

**I/O**硬件抽象层与软件组件有一些类似，特别是通过端口定义来实现**RTE**的数据通信的。他们主要区别在于**I/O**硬件抽象位于RTE之下（在**ECU**抽象层中），而软件组件位于RTE之上。**I/O**硬件抽象是基础软件模块和应用软件之间的一种接口。

对于**I/O**硬件抽象模块，以及其他的服务，当前的方法论需要填写两个不同的模板。例如：为了在**AUTOSAR**的**ECU**上集成**NVRAM**管理器，需使用**BSWMD**模板记录其对**BSW**调度程序、操作系统资源等的需求。此外也需要使用**SWC**模板来描述和**RTE**相连接的端口信息。

**I/O**硬件抽象是**BSW**的一部分。它可以被视为是一组软件模块。虽然**IoHwAb**是集成代码，但**IoHwAb**的每个模块都可以适用**BSWDT**。

同时**ECU**信号需映射到**VFB**端口。可以使用软件组件模板（**Software Component Template**）来描述**I/O**硬件抽象的实现和应用应用软件组件实现之间的接口定义。

### 5.6.1. I/O硬件抽象端口定义

**I/O**硬件抽象规范仅定义了端口使用方法的建议，但端口的实例化应在配置过程中完成，并且会特定于某个**ECU**的电子设计。

**I/O**硬件抽象建议为每个可识别的**ECU**信号创建一个端口，但关联到**ECU**输出信号的**ECU**诊断信号除外。同时**ECU**信号和端口之间的关系也许也需被定义。

**例如：** 某个**ECU**有**10**个模拟输入引脚（**Analog input pins**）、**15**个PWM输出引脚（**PWM output pins**）、**15**个数字输出引脚（**Digital output pins**）。

则**I/O**硬件抽象为每个**ECU**信号定义了至少一个端口。在这个简单的例子中，端口被实例化了**40**次。

## 5.7. 电源模式

### 5.7.1. 异步电源状态转换

下图讲述了**I/O**硬件抽象模块设计的电源状态转换，实现将**ADC**和**PWM**设置为低功耗模式的状态。

其中外设配置为异步电源状态转换。在收到准备电源状态的请求后，外围设备的驱动程序向调用者**IoHwAbs**发出通知，通知它已准备好转换到新的电源状态。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97tx0FyQiaicKicsd9oAHMmoUtC9SOtOsS9Oqo3mjQnmAUNVOgrFXb8OyCF3Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**5.7.2. 同步电源状态转换**

下图展示了一个同步转换，即：外围设备能够立即完成状态的转换，无需实现一个异步等待的过程。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97txRW26ibib9Unoj8I9ZkhZibWA7rElP90vFd2z9KuMGfXwYOBghghoWrJZw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## 5.8. 具体示例

### 5.8.1. 示例1 - 板载硬件

此示例来自于一个供电的**ECU**的例子。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97tx9Nju5zlibMWsPbOcib6zRS44icW01SicicJhJQicOiciaswljNk2OcNWbNWOwA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



此**ECU**包含多个数字输入信号（DI）:

- 第一组是控制机械开关的慢速数字输入信号
- 第二组是用于诊断功率半导体（Power IC）的快速数字输入信号。这些信号输出电流是否过流（**over current**）。同时信号也没有从**ECU**中引出。

由于**MCU**没有足够的**PIN**脚，所以慢速的数字输入信号连接到了一个8位多路复用器，每个多路复用器会有**3**条地址线和**1**条数据线组成。

OEM客户需求如下：

1. 发生过流时到关闭**Power IC**的最长时间需为1ms
2. 开关的反应不得迟于**100**毫秒
3. 每个数字输入信号必须通过5票的3票（**3 of 5 voting**）来消除抖动。然而实际测试，因为机械开关和**Power IC**不会产生干扰信号，所以去抖的类型不是非常重要。

当前的解决方案是每**0.8**毫秒周期任务去读取一次所有数字输入。输入信号包括了慢速的和快速的。慢速数字输入的扫描速率可能可以更低一些，但额外任务的开销高于运行时节省。每次循环中，因为慢速数字输入的去抖是1次，所以最差的去抖值延迟是**3.2**毫秒。

如果检测到过流，该引脚将会被再次读取几次。但在同一个任务循环中，**Power IC**需被立即关闭。

应用程序会每**10**毫秒运行一次，并读取开关的去抖后的数字输入数值和诊断信息的反馈。

AUTOSAR架构上的分解： 

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97txs0BJLRDic0qewv64THlu9x3TDX5yPfq9UEFjhJ9Cibb8wzUtRDZIibo8Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



### 5.8.2. 示例2 - 故障监测

在本示例中，首先需定义一个信号用于输出的诊断，它是基于**I/O**硬件抽象级别的诊断属性来定义。所以一个输入信号会用于执行相关输出诊断的反馈。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97tx2YxTvuZE5QW4TIVF2MGKuxIzWa2kHNPtFm8QNh59mhqOnqY8OqGtdg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

当**I/O**硬件抽象要求定位一个输出时，并调用Dio_WriteChannel，通道反馈的读取是通过配置为ECU输入引脚完成的。

ICU驱动程序向**I/O**硬件抽象发送通知。保护策略位于集成代码中。

软件组件可以通过端口定义的诊断操作来获取诊断值。

### 5.8.3. 示例3 - 输出功率

ECU 硬件有一个功率级ASIC。所有ECU引脚都需在**I/O**硬件抽象层上作为信号（**signal**）使用，在**RTE**层下面。

- 一些输出通过**SPI**处理程序控制。
- 一些输入通过**DIO**驱动程序直接控制。
- 一些电压值和频率通过**PWM**驱动程序设置。

功率级驱动程序（**Power Stage Driver**）提供了所有输出的视图。它会调用**PWM**、**DIO**驱动程序以及SPI处理程序的服务。从软件组件的角度来看，信号抽象使所有这些输出都可见，信号需映射到端口上。同时功率级驱动程序是可配置的。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97txdPr4exXovZNxGcbxtJpcyBYlPic40ZpZJwnhIVaRvqwjH0icLuARsGFw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**诊断：** 每个故障都可以在功率级的级别上检测到。诊断数据流会通过**SPI**通信，传送到功率级驱动程序。接着通过**S/R**接口，将诊断结果提供给所有的软件组件。

### 5.8.4. 示例4 - 在低功耗状态下设置传感器和控制外围设备

**ECU**通过**ADC**和**DIO**接口来控制外部的传感器（**External Sensor**）。在特定情况下，ECU进入传感器关闭且ADC设置为低功耗状态的操作模式。

操作动作顺序如下：

1. 应用程序电源模式管理器（**Application Power Mode Manager**）向**BswM**发出模式请求，希望能切换到低功耗模式（**LowPowerMode**）。
2. **BswM**评估请求。如果所有先决条件都满足，则向**Power Mode Manager**和传感器SWC（**Sensor SWC**）发出模式切换。
3. 传感器SWC停止读取传感数据，即不再向**IoHwAb**请求任何**Get**操作。
4. **IoHwAbs**模块从ADC模块注销通知（**notification**），并最终停止周期性的硬件采集。
5. **IoHwAbs**发送命令让外部传感器硬件进入低功耗模式或将其关闭。
6. **IoHwAbs**调用低功耗模式准备的Callout函数，接着会调用低功耗模式设置Callout函数，并按配置中定义获得与请求的应用低功耗模式（**LowPowerMode**）相关的 ADC的电源状态。

通过引入更细粒度的模式请求并对确认和开关作出反应，可以逐步控制该过程。



![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqWQJZX48lUaFeGDTaA97txkgwJ0jmERKUlmgJ5ODZJoDJFDsnEkfvuRmIYL6FtSbnA4zyto1rDFg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



# 6. API规范

## 6.1. 函数定义

### 6.1.1. IoHwAb_Init<Init_Id>

**说明**: 初始化所有**IO**硬件抽象软件或者是**IO**硬件抽象的一部分。

```
void IoHwAb_Init<Init_Id> ( 
    const IoHwAb{Init_Id}_ConfigType* ConfigPtr )
```

### 6.1.2. IoHwAb_GetVersionInfo

**说明**: 返回此模块的版本信息。

```
void IoHwAb_GetVersionInfo ( 
    Std_VersionInfoType* versioninfo )
```

### 6.1.3. IoHwAb_AdcNotification<#groupID>

**说明**: 当组 <#groupID> 的组转换完成时将由**ADC**驱动程序调用。

```
void IoHwAb_AdcNotification<#groupID> ( void )
```

### 6.1.4. IoHwAb_Pwm_Notification<#channel>

**说明**: 当通道 <#channel> 上出现信号边沿时，将由**PWM**驱动程序调用。。

```
void IoHwAb_PwmNotification<#channel> ( void )
```

### 6.1.5. IoHwAb_IcuNotification<#channel>

**说明**: 当通道 <#channel> 上出现信号边沿时，将由**ICU**驱动程序调用。

```
void IoHwAb_IcuNotification<#channel> ( void )
```

### 6.1.6. IoHwAb_GptNotification<#channel>

**说明**: 当通道 <#channel> 上的计时器值到期时，将由**GPT**驱动程序调用。

```
void void IoHwAb_GptNotification<#channel> ( void )
```

### 6.1.7. IoHwAb_OcuNotification<#channel>

**说明**: 当阈值的当前值与通道<#channel>上的阈值匹配时，将由**OCU**驱动程序调用。

```
void IoHwAb_OcuNotification<#channel> ( void )
```

### 6.1.8. IoHwAb_Pwm_NotifyReadyForPowerState<#MODE>

**说明**: 当模式 <#Mode> 的请求电源状态准备完成时，**PWM**驱动程序将调用**API**。

```
void IoHwAb_Pwm_NotifyReadyForPowerState<#Mode> ( void )
```

### 6.1.9. IoHwAb_Adc_NotifyReadyForPowerState<#MODE>

**说明**: 当为模式 <#Mode> 请求的电源状态准备完成时，**ADC**驱动程序应调用**API**。

```
void IoHwAb_Adc_NotifyReadyForPowerState<#Mode> ( void )
```

### 6.1.10. IoHwAb_Dcm_<EcuSignalName>

**说明**: 此函数提供**DCM**模块对某个**ECU**信号的控制访问，其中<EcuSignalname>是**ECU**信号的符号名称。通过此函数可以进行**ECU**信号锁定（**locked**）和解锁（**unlocked**）。锁定代表将**ECU**信号冻结（**freeze**）为当前值、配置的默认值或者参数Signal给出的值。

```
void IoHwAb_Dcm_<EcuSignalName> ( 
    uint8 action, <EcuSignalDataType> signal )
```

### 6.1.11. IoHwAb_Dcm_Read<EcuSignalName>

**说明**: 该函数提供**DCM**模块对某个**ECU**信号的读取访问,（其中<EcuSignalname>是**ECU**信号的符号名称。

```
void IoHwAb_Dcm_Read<EcuSignalName> (
     <EcuSignalDataType>* signal )
```

### 6.1.12. IoHwAb_PreparePowerState<#MODE>

**说明**: 函数需由**IoHwAb**调用，为转换到给定的电源状态做准备工作。此函数的目的是封装所有操作，以便使硬件为预定义的电源模式做好准备，将应用程序电源定义与硬件电源状态分离。

```
void IoHwAb_PreparePowerState<#Mode> ( void )
```

### 6.1.13. IoHwAb_EnterPowerState<#MODE>

**说明**: 函数需由**IoHwAb**调用，以便有效地进入由函数IoHwAb_PreparePowerState<#Mode>准备的电源状态。此函数的目的是封装所有操作，以便将硬件设置为对应于预定义电源模式的电源状态，从而将应用程序电源定义与硬件电源状态分离。

```
void IoHwAb_EnterPowerState<#Mode> ( void )
```