# AUTOSAR诊断服务－DEM诊断故障码DTC

**前言**

**AUTOSAR DEM模块的分享分为DEM模块概念详解和Dcm模块配置及代码分析，具体的项目实战请关注本号的后续文章，本篇为\**DEM模块诊断故障码DTC详细介绍篇\**。**

[AUTOSAR 诊断服务-DEM功能概述](http://mp.weixin.qq.com/s?__biz=Mzg2NTYxOTcxMw==&mid=2247484841&idx=1&sn=eb2474651f14db5896c191a20a7780a5&chksm=ce561fc7f92196d1aa7b0920232141ffca462a07dd8a87a032362adc05e4fa9d3f086ce3e1b4&scene=21#wechat_redirect)

**[AUTOSAR诊断服务－DEM事件Event](http://mp.weixin.qq.com/s?__biz=Mzg2NTYxOTcxMw==&mid=2247485683&idx=1&sn=aa02b73edeb92fb47c2b181f926a072f&chksm=ce56129df9219b8b3e10976f09eaf55b668ced2918c7ebf166cf33dd213e74cf43f9c000fad7&scene=21#wechat_redirect)****
**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczprrlDtia4GpE9hib4jibMgVn9RyU8VA9giab7Z3xgGdFz3mDpQHWibFSzz05p640NA5Qof5r8o2mNgLg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**正文**



“诊断故障码-DTC”定义了一个映射到Dem模块的“诊断事件”的唯一标识符（显示给诊断仪）。Dem为Dcm模块提供“故障诊断码”的状态（例如通过：19 06 EF 02 88 FF读取单个DTC（通过EF 02 88唯一标识，代表总线BusOff）状态，）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczprrlDtia4GpE9hib4jibMgVnQSWgicaicjnpS8qcPaYXMZvpGn7bGTuzQ18F9mPjjSzk9LwnE96icVFDg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图四：:DTC configuration



DemDTC这个配置容器就是对DTC的具体描述，一个DTC对应一个DemDTC的配置。



## **5.1 DTC类型**

两种不同类型的DTC：

. 非OBD相关的DTC（统一诊断协议UDS DTC）

. OBD相关的DTC



如果配置了DemObdDTC，则 DemObdDTC配置容器下的DTC及其相关的Event就是OBD相关的。



DTC类型与诊断服务ReadDTCInformation和一些OBD服务(0x19/$03/$07/$0A，参考Dem_SetDTCFilter)所使用的DTC的选择相关。诊断服务ControlDTCSetting (0x85，参考Dem_DisableDTCSetting和Dem_EnableDTCSetting)也与DTC类型相关。





***\**问题\**\*****：*****\**OBD 诊断与 UDS 诊断有什么区别？汽车故障诊断相关。OBD和UDS是在车辆上同时存在还是选择一个？它们有什么不同？\**\***



***\**答\**\*****：*****\**OBD（全称：On BoardDiagnostics）\**\*****，即车载自动诊断系统，是汽车排放和驱动性相关故障的标准化诊断规范，有严格的排放针对性，*****\**其实质就是通过监测汽车的动力和排放控制系统来监控汽车的排放\**\*****。当汽车的动力或排放控制系统出现故障，有可能导致一氧化碳(CO)、碳氢化合物(HC)、氮氧化合物(NOx)或燃油蒸发污染量超过设定的标准，故障灯就会点亮报警。**



**首先，OBD是面向汽车排放问题而制定的规范，也就是说对所有车辆统一适用，在OBDⅡ计划实施之后，任一技师可以使用同一个诊断仪器诊断任何根据标准生产的汽车。而且OBD Ⅱ程序使得汽车故障诊断简单而统一，维修人员不需专门学习每一个厂家的新系统。其次，OBDII使用标准的16针诊断接口，并且统一各车种相同故障代码和意义，这样一方面，这是为了方便技师维修，当故障车辆来到4S店后，技师可用专用的诊断工具读取汽车存在的故障码，故障发生时的时间、里程、故障发生次数等重要参数，从而提高维修效率。而OBD系统更重要的另一方面，也是它设计的初衷，就是为了控制排放，能在发生了尾气排放超标的故障时及时提醒车主，尽快去修复故障。**

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczprrlDtia4GpE9hib4jibMgVnbLuEu9d91cpDBCZ0HlfmTSt7wZ9X72fdFk55uQfDKbJwibCamYtLXZw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图五：:OBD接口



***\**UDS\**\*****（全称：UnifiedDiagnostic Services），即统一诊断服务，是诊断服务的规范化标准，为诊断服务提供一个基本框架，这些诊断服务允许诊断仪在车载电子控制单元里面控制诊断功能，以便维修人员能够准确的解决故障。**

**UDS在使用过程中除了协议中已经定义好的通用的代码指令之外，还有一部分未定义留给整车厂自行定义，这样就会形成不同厂家ECU的DID不同，所以对ECU的诊断过程需要事先了****解内部定义。**



**对比之下：**

***\**.\**\*** ***\**OBD是关注车辆实时排放的理念形成的行业规范，而UDS是诊断服务的统一化规范。\**\***

***\**.\**\*** ***\**而且，UDS是面向整车所有ECU(电控单元)的，而OBD是面向排放系统ECU的。\**\***

***\**.\**\*** ***\**两者之间并不存在谁替代谁。\**\***



## **5.2 DTC格式**

Dem模块应支持的DTC格式：

. ISO-14229-1[2]

. SAE J2012 OBD DTC (aka 2-byte DTC) [14]

. SAE J1939-73[15]

. ISO 11992-4[16]

. SAE J2012 WWH-OBD DTC (aka 3-byte DTC) [14]



配置参数DemTypeOfDTCSupported用来指定ECU支持的一种DTC格式。一个DTC可以有三种格式(UDS、OBD和J1939)的任意组合，即同时使用一种、两种或三种格式。因此，Dem将在内部处理三个DTC值列表。报告的格式取决于Dem_DTCFormatType或由相关API的上下文定义。



DTC应将DTC值报告为uint32，字节0=低字节，1=中字节，字节2=高字节和字节3未使用。对于OBD DTC格式，只有两个字节（高字节，低字节）被使用。Dem服务应将这些dtc报告为uint32，字节1=低字节，字节2=高字节，字节3未使用，字节0=0x00。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczprrlDtia4GpE9hib4jibMgVnx68g6YLhqUiaTTCTeDAYVA7tMJd2W5XsKVzJgaKSguxmsE8ZqAfNHJg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



图六：DTC Byte Order



函数Dem_GetDTCOfEvent将获得由Dem配置映射到EventId的DTC。



## **5.3 DTC组**

除了单个DTC值外，还可以配置由ISO14229-1[2]-附件D.1所定义的DTC组(参见DemGroupOfDTC)。每个DTC组都有自己的DTC组值（该值必须对任何其他DTC和DTC组值唯一）。当请求DTC组的操作（如CleaerDTC和禁用/启用DTC）时，通过DTC值选择DTCGroup。



DemGroupOfDTC配置容器定义了DTC组的边界值。在被请求组的DTC代码和下一个更高的dtc组代码之间的范围内的所有DTC都应被视为属于该组（例如：0x3FFF00和0x7FFFF0是两个组边界值，则0x3FFF00 --> 0x7FFF0F之间的DTC属于0x3FFF00这个组）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczprrlDtia4GpE9hib4jibMgVnFibeMfBYesXY0YDbk2DQNPn1Wos76hGLMF2EkTp2W3KWT5ic03xcGUvw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



以下的DTC组是预定的：

.‘所有DTc的DTC组(强制，固定值=0x FFFFFF)

. WWH-OBD排放相关的DTC(0x FFFF 33)



请注意，DTC组“所有DTC”将不会在DemGroupOfDTC中进行配置，因为它总是要由Dem模块提供。



DTC组与诊断服务ClearDiagnosticInformation（0x14，参考Dem_ClearDTC）以及诊断服务ControlDTCSetting（0x85，参考Dem_DisableDTCSetting和Dem_EnableDTCSetting）相关。





**Note:实际项目中，在定义DTC码的时候会考虑分组（比如动力相关的DTC，车身相关的DTC，热管理相关的DTC会被分配在不同的DTC码端中），但是在Dem模块中不再配置DTC Group。**



## **5.4 DTC严重性**

根据ISO-14229-1[2]，附录D.3“DTC严重程度和类别定义”，DTC严重程度用于提供有关特定事件重要性的信息。DemDTCS严重性仅可用于UDS DTC。



如果调用了API Dem_GetSeverityOfDTC或Dem_GetNextFilteredDTCAndSeverity，并为选中的DTC配置了一个在DemDTCSeverity中的严重性值，则Dem应将在DemDTCSeverity中配置的值设置为DTCSeverity。



如果调用了API Dem_GetSeverityOfDTC或Dem_GetNextFilteredDTCAndSeverity，并且对于所选的DTC没有在DemDTCSeverity中配置严重性值，则Dem应将DEM_SEVERITY_NO_SEVERITY设置为DTCSeverity。



## **5.5 功能单元**

如果调用了API Dem_GetFunctionalUnitOfDTC，并为所选的DTC配置了一个功能单元值DemDTCFunctionalUnit，则Dem应在输出参数DTCFunctionalUnit中设置该值。



如果调用了API Dem_GetFunctionalUnitOfDTC，并且对于所选的DTC，没有在DemDTCFunctionalUnit中配置功能单元，则Dem应在输出参数DTCFunctionalUnit中设置0x00的值。



## **5.6 DTC显著性**

DTC有两种不同的显著性水平：

. 故障：对故障进行分类，该故障与组件/ECU本身有关（例如，需要进行修复操作）

. 发生：对问题进行分类，该问题指示有关系统行为不足的附加信息(例如与ECU控制之外的条件相关)



这个显著性级别是每个事件都可配置的(参考demdtattributes中DemDTCSignificance)，并且可以映射为一个数据元素(参考DEM_SIGNIFICANCE)。



## **5.7 抑制DTC输出**

在运行时通过API调用动态抑制事件/ DTC。被抑制的DTC的行为对测试人员来说是不可见的，但可以由诊断监视器连续处理。



如果DTC和事件之间存在一对一的关系，则如果相关事件不可用，则DTC将被抑制。如果DTC是一个组合DTC，则如果所有组合事件都不可用，则抑制DTC。



## **5.8 事件的可用性**

Dem模块提供一种将事件标记为不可用的方法，而无需将事件从实际配置中删除。不可用的事件将被视为它没有包含在Dem配置中。



一个示例性的用例可以是控制单元及其软件支持要控制的不同类型的硬件。硬件变体在某些部件可用或不可用时有所不同。仍然应采用相同的控制单元和软件。通过配置参数DemEventAvailable可以配置事件是否可用。Dem提供Dem_SetEventAvailable接口来配置事件的可用性。



如果事件设置为不可用，则将相应事件视为系统中未配置(例如Dem_SetEventStatus和Dem_GetEventUdsStatus应返回E_NOT_OK)。以下api会受到影响：

• Dem_SetEventStatus

• Dem_GetEventUdsStatus

• Dem_ResetEventDebounceStatus

• Dem_ResetEventStatus

• Dem_PrestoreFreezeFrame

• Dem_ClearPrestoredFreezeFrame

• Dem_GetDebouncingOfEvent

• Dem_GetDTCOfEvent

• Dem_GetFaultDetectionCounter

• Dem_GetEventFreezeFrameDataEx

• Dem_GetEventExtendedDataRecordEx

• Dem_ClearDTC

• Dem_DcmGetDTRData

• Dem_RepIUMPRFaultDetect

• Dem_RepIUMPRDenLock

• Dem_RepIUMPRDenRelease

• Dem_SetWIRStatus

• Dem_DltGetMostRecentFreezeFrameRecordData

• Dem_DltGetAllExtendedDataRecords



当API Dem_SetEventAvailable被调用，用状态=为false时，Dem应返回E_NOT_OK：

•对于该事件，有一个事件内存项已经存在

•已设置任何事件状态标志“测试失败”、“待定”、“已确认”或“请求的警告指示符”



## **5.9 小结**

本小结着重介绍了DTC的定义及相关属性配置，实际项目开发中对于DTC就可以简单理解为一个3个字节的码（例如：EF0288），一个DTC和一个事件绑定（EF0288和ECAN BusOff这个Event绑定），如果一个事件发生，则这个事件绑定的DTC的状态就会发生改变（状态码的对应标志置位）。本节描述的其他相关属性，实际项目中基本没有用到，如果用到的时候可以再着重研究。不过，在当今芯片短缺经常换硬件元器件的场景下，DemEventAvailable事件可用性这个特性可以考虑利用起来。做到一版软件配置所有硬件方案。



------

*参考文档：*

*1.Specification of Diagnostic Event Manager*

*1.https://www.zhihu.com/question/26374239/answer/369991927*