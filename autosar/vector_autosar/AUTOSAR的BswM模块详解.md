# AUTOSAR的BswM模块详解

**0. 关于BswM模块**

**BswM**即BSW Mode Manager，它是实现位于 BSW 中的车辆模式管理（Vehicle Mode Management ）和应用程序模式管理概念（Application Mode Management ）的一部分的模块。

它的职责是根据简单的规则对来自应用层 SW-C 或其他 BSW 模块的模式请求进行仲裁，并根据仲裁结果执行操作。

BswM在AUTOSAR上跟很多模块有关联的，例如EcuM、ComM、OS等等，我们从下图就可以看出来：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicfyr5fE8qFVcIQJmxic1kZOZjZAS4HNibxURWaRmDfIyibYrU7wkSLZj690F7KAAKK6Tia4eIiahVqyZA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

BSWM的操作功能可以描述为两部分：**Mode Arbitration** 和**Mode Control**。

Mode Arbitration部分启动模式切换，SW-C 或其他 BSW 模块接收的模式请求和模式指示基于规则的仲裁会触发模式切换。

Mode Control部分通过执行包含其他 BSW 模块的模式切换操作的Action List来执行模式切换。

BswM 应该被视为一个模式管理框架模块，其中的行为完全由其配置定义。

对于Mode Arbitration 和Mode Control，我们可以简单粗暴地理解为前者是条件判断的，后者是动作执行的。

以下内容，我们会接触到以下概念：**Rules（规则）、Mode Request（模式请求）、Expression（表达式）、Action List（执行列表）**等等，从下图可以看出，这几个东西都是AUTOSAR Configurator的BswM里面配置项。

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicfyr5fE8qFVcIQJmxic1kZOj4DY358xhxEAcwIcfdy5bwMb6EFtVYJ4KTicXTgrs2zHlj6tfjQOOTw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**1. Mode Arbitration**

BswM 执行的Mode Arbitration是基于规则（rule-based）的而且很简单。用于Mode Arbitration的rules在 BSW Mode Manager模块的配置中指定。

这些规则由简单的布尔表达式组成，因此Mode Arbitration对运行时的影响很小。

为了知道要执行哪些Action List，BswM 需要检测来自先前规则评估的Mode Arbitration结果的变化。

**1.1 Arbitration Rules**

Rules是由一组模式请求条件组成的逻辑表达式。

当输入模式请求和模式指示改变时，或在 BswM主函数执行期间，将评估Rule。评估结果（True or False）用于决定执行相应的模式控制Action List。

**1.2 Mode Conditions and Logical Expressions**

构成模式仲裁规则的Logical Expressions可以使用不同的运算符，例如 AND、OR、XOR 和 NAND。表达式中的每一项都对应一个模式请求条件。这些条件验证所请求或指示的模式对于特定模式是 EQUAL还是NOT_EQUAL。下图显示了具有两个条件的示例Rule。Rule和可用逻辑操作集被定义为ECU 配置的一部分。

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicfyr5fE8qFVcIQJmxic1kZOWwvvDgrXXWGP9Q1BGmia7SGZvWBdxXLrDZhyxZNx2Em0gwFlLVQp78A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**1.3 Requirements of Mode Arbitration**

如上所述，BswM接受模式请求和模式指示作为模式仲裁的输入。模式请求通常源自应用程序SW-C，但也可能源自其他 BSW模块，例如 DCM。模式指示总是由其他BSW模块发出，例如不同的总线特定状态管理器、EcuM 和WdgM。

**Immediate and Deferred Operation**

有各种不同方式来调度处理Mode Arbitration：

- 在模式请求或模式指示上下文immediately处理；
- 推迟到BswM MainFunction（周期轮询函数）处理。

“**Immediate**”请求在调用者的上下文中执行。系统集成商（如Vector)有责任确保操作列表的执行不会危及系统性能或一致性。

特别是，如果调用者在中断上下文中运行（或可能运行），则有关在中断上下文中使用系统函数的限制仍然适用。

**Immediate**和**Deferred**操作之间的区别在下图。

（1）Immediate Operation

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicfyr5fE8qFVcIQJmxic1kZOibmFPZU8ogKP4DPQQqmYn9zgFWib1S3TobmyZZG20oaOg4CamLXYXiawg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

（2）Deferred Operation

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicfyr5fE8qFVcIQJmxic1kZOt4UpChiavGWop3gHL8GVWia5eO8uHmO7icGtib2atAsmeDb0tSPQhWkpTQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**1.4 Arbitration Behavior after Initialization**

BswM初始化后的Mode Arbitration行为由Configurator Container BswMModeInitValue控制。该参数可以为配置中的每个BswMModeRequestPort配置一次。例如

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicfyr5fE8qFVcIQJmxic1kZOFMBoz1q4ticmEyFokMst0JGXT86j2mwgFIr6IRHicQOWibQ2FFML8ZvJw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**2. Mode Control**

BswM的Mode Control部分根据Mode Arbitration的结果执行所有必需的操作。这是使用Action List完成的。Action List是由Mode Arbitration触发时BswM执行的动作的有序列表。

Action List中的Action可以分为三种类型：

1. 调用其他BSW模块或 RTE。
2. 链接到要包含在执行中的其他Action List。
3. Mode Arbitration规则。当执行相应的Action List时，将评估这些规则。这样可获得规则的层次结构。

BswM不需要在其执行的操作上存储或响应任何 BSW 模块特定的返回值。因此，BSW 中的不同状态管理器将它们的当前状态给BswM，以用作Mode Arbitration的输入。但是，如果返回错误 (E_NOT_OK)，BswM 可以发出 DEM事件和/或取消当前正在执行的Action List。

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicfyr5fE8qFVcIQJmxic1kZO014T1MDbtsHL3QNM1Wgne0ScPl1TtYicTyObFIziaKKicjn3B5yfVHeAA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如图所示，BswM可能包含多个Action List，一个Action List可以容纳多个Action。为了减少Action List的总数，应该可以将它们级联。Action List的元素可以是具体动作或对另一个Action List的引用，或者如上所述，Mode Arbitration要执行的规则。应该有一个标志连接到每个动作列表条目，说明其类型（动作/引用/规则）。带有具体动作的列表和带有引用或甚至是混合列表的列表的激活方式应该没有区别。

**2.1 Mode Processing Cycle**

通过下图来讲解Mode Request的处理周期

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicfyr5fE8qFVcIQJmxic1kZOoUYdoNZic8hCw59hgias8KzZtImibiaeIxudQWw7Hp0vovajtXVMibSGNdQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

1. 模式请求者 SW-C 通过其发送者端口请求模式 A。 RTE 分发请求，BswM 通过其接收器端口接收它。
2. BswM作为接收到的模式仲裁请求的结果或在 BswM 主函数执行期间循环评估其规则。
3. 根据选择的执行方法执行相应的动作列表。
4. 在执行动作列表时，BswM 可能会向 RTE Switch API发出一次或多次调用，作为通知受影响的 SW-C 仲裁结果的动作。任何 SW-C，尤其是模式请求者都可以注册接收模式切换指示。

注意模式请求者只能从本地BswM接收模式切换指示；这对于源自本地代理 SW-C 发出的不同 ECU 的请求也是如此。

**2.2 Behavior of Mode Control after Initialization**

BswM初始化后模式控制的行为由BswMRuleInitState 参数配置（在BswMRule Container内）。它定义了在初始化后第一次评估规则后决定执行哪个Action List时使用的“前一个评估结果”。配置参数BswMActionListExecution（在BswMActionList Container内）也会影响初始化后的动作列表执行。

BswMRuleInitState是什么样的，可见下图的“Rule Init State”

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicfyr5fE8qFVcIQJmxic1kZObUq31OasH4tZDqx66EQP60hlrGySsMwY18iafrVEHTvulXkszibLentw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

BswMActionListExecution可见下图的“Action List Execution”

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicfyr5fE8qFVcIQJmxic1kZOm2YuaBPQwNVODCwrpavjluFyr8eetjHn5rG76qwcibkKKNV0tD8UKHw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**3. BswM Interfaces and Ports**

BSWM Service的 SW-C将定义RTE下的Port。每个使用Service的 AUTOSAR SW-C都必须在其自己的 SW-C中包含Service Port。这些Port使用相同的Interface接入，并且必须连接到BSWM的Port，以便 RTE可以生成适当的 ID和所需的符号。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicfyr5fE8qFVcIQJmxic1kZO7PXnX8ENh07rO12k9phy37oXuTrdTcCeyibDzq1cJ1eSGaial5g3VytA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

以下讲讲Mode Request Port、Mode Switch Port、Mode Notification Port，即下图这几个东西：

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxicfyr5fE8qFVcIQJmxic1kZOCwtz2bcbkyCw8bVa0rjsyCGmRZnyVNDddHTlPzictzJs1aBXRPkks5A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**3.1 Mode Request Ports**

BSW Mode Manager必须声明一个接收器端口，其接口定义在 SW-C的上下文中：

```
RequirePort AppModeRequestInterface modeRequestPort_{ArbName}_{ReqName};
```

要读取当前请求的模式，BSW Mode Manager实现必须调用：

```
Rte_Read_modeRequestPort_{ArbName}_{ReqName}_requestedMode( &<variable> );
```

**3.2 Mode Switch Ports**

与Mode Request一样，BSW Mode Manager仅引用在其为模式切换提供端口的相应SW-C 描述上下文中定义的模式切换Port。对于上面的例子，Mode Switch的声明是：

```
ProvidePort AppModeInterface modeSwitchPort_{ModConName}_{SwitchName};
```

配置参数BswMModeSwitchInterfaceRef引用此模式切换接口。



要切换当前激活模式，BSW Mode Manager实现必须将以下调用之一插入其操作列表：

```c
Rte_Switch_modeSwitchPort_{ModConName}_{SwitchName}_currentMode(<new_mode>);
SchM_Switch_modeSwitchPort_{ModConName}_{SwitchName}_currentMode(<new_mode>);
```

**3.3 Notifications of Mode Switches**

除了Mode Request之外，当前激活的模式也可以用作Mode Arbitration的输入。对于应用程序和车辆模式，BSW Mode Manager需要注册为模式用户。然后它通过Mode Switch Port接收有关更改模式的通知。对于上面的例子，模式通知的声明是：

注意：为了更容易区分ModeSwitchPort类型的RequirePort和ProvidePort，下面示例中RequirePorts被命名为Mode Notification Port。

```
RequirePort AppModeInterface modeNotificationPort_{ArbName}_{ModeName};
```

要读取当前激活模式，BSW Mode Manager实现必须调用以下函数之一：

```
Rte_Mode_modeNotificationPort_{ArbName}_{ModeName}_currentMode( &<variable> );SchM_Mode_modeNotificationPort_{ArbName}_{ModeName}_currentMode( &<variable> );
```



如果配置了增强型 Rte_Mode 或 SchM_Mode，BSW Mode Manager实现必须调用以下函数之一：

```
Rte_Mode_modeNotificationPort_{ArbName}_{ModeName}_currentMode( &<variable>, &<previousmode>, &<nextmode> );SchM_Mode_modeNotificationPort_{ArbName}_{ModeName}_currentMode( &<variable>, &<previosmode>, &<nextmode> );
```

至于，BswM在应用中是怎么配置的，请看《[AUTOSAR BswM Shutdown流程配置详解](http://mp.weixin.qq.com/s?__biz=MzI0MTg5MDY3NA==&mid=2247484695&idx=2&sn=59fc307b1e902002474e11f9aa81b11c&chksm=e905e10ade72681c8bbe5c3e8845c94c367d48d39ba457a3c5ec3466e0fbac08ce7f4025899a&scene=21#wechat_redirect)》，通过这个文件，你可以理解Rules和Action List是怎么用的，Notification又是干嘛的等等。