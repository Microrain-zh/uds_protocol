# AUTOSAR 通信服务-Dcm配置分析

**正文**

# **6.****配置**

## **6.1 DCM**

DCM模块包括DcmConfigSet和DcmGeneral两个配置容器。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgHIoZUEMib9GaTtf3HIY9BKEDY7w9vpy7RQNjnPpiadx5sPFxsLKu7DTQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**DcmGeneral**：Dcm模块的通用配置项。

**DcmConfigSet**：这个容器包含配置参数和支持多个配置集的DCM模块的子容器。

 

### **6.1.1** **DcmGeneral**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgaWRpmoRxibxJoEqYuic26eh294Ksx7Iuk10l2UUicKDqUqlH594RdEMzw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



 

**DcmDDDIDStorage**：这个配置开关定义了DDDID定义是否存储为非易失性。

 

**DcmDevErrorDetect**：打开或关闭开发错误检测和通知。

 

**DcmRespondAllRequest**：如果设置为FALSE, Dcm将不会响应响应ID范围为0x40到0x7F或0xC0到0xFF(响应ID)的诊断请求。

 

**DcmTaskTime**：允许配置周期周期任务的时间。

 

**DcmVersionInfoAp**：配置是否是使用版本检测API。

 

### **6.1.2** **DcmConfigSet**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgKjtibriakyPH005UdQFIXCpLQuUC1ic9pSNgTHLmEADbJTslicNiaWGEtCQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**DcmDsd****：**DSD诊断调度子模块配置容器。

 

**DcmDsl****：**DLS诊断会话层子模块配置容器。

 

**DcmDsp****：**DSP诊断服务处理子模块配置容器。

 

**DcmPageBufferCfg****：**也缓冲配置容器。

 

**DcmProcessingConditions****：**模式仲裁配置容器。

#### **6.1.2.1** **DcmDsd**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgibkRuRHf9PganID2vn1oRjJACmXaDgb2icica0yKMjJTwzLgEXOMwGBdg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**DcmDsdRequestManufacturerNotificationEnabled****：**对制造商允许启用或禁用请求通知机制。

 

**DcmDsdRequestSupplierNotificationEnabled**：对供应商允许启用或禁用请求通知机制。

 

以上两个参数一般都配置位fasle。也就没有**DcmDsdServiceRequestManufacturerNotification**,

**DcmDsdServiceRequestSupplierNotification**这两个配置容器的配置。

 

**DcmDsdServiceTable**：这个容器包含每个具体诊断服务（0x10, 0x11等）的配置(DSD参数)。

#### **6.1.2.2** **DcmDsdServiceTable**

**DcmDsdSidTabId**：可能有多个DcmDsdServiceTable，DcmDsdSidTabId用来标识一个DcmDsdServiceTable。

 

**DcmDsdService**：一个具体诊断服务的配置容器。

 

#### **6.1.2.3 DcmDsdService**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgIiauayr5kHL40BUG4htepOFClj5u9iaRvmKcEr8yJza75BmTSdTLc38w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**DcmDsdServiceUsed**：配置是否使用该服务。

 

**DcmDsdSidTabFnc**：ECU Supplier特定组件针对特定服务的回调函数。该函数的原型如<Module>_< diagnostics service >所述。如果未配置此参数，则服务在dcm内部处理。

 

**DcmDsdSidTabServiceId**：诊断服务ID。

 

**DcmDsdSidTabSubfuncAvail**：该服务是否有子服务。

 

**DcmDsdSidTabModeRuleRe****f**：对控制服务执行的DcmDspModeRule的引用。如果没有配置引用，则不进行模式规则检查。

 

**DcmDsdSidTabSecurityLevelRef**：引用允许在其中执行服务的安全级别。一个服务允许多个引用。

 

**DcmDsdSidTabSessionLevelRef**：对允许执行服务的会话级别的引用。一个服务允许多个引用。

 

**DcmDsdSubService**：这个容器包含服务的子服务的配置(DSD参数)。只有那些服务可以有子服务，这些子服务将DcmDsdSidTabSubfuncAvail配置为TRUE。

 

#### **6.1.2.4 DcmDsdSubService**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFg2UAlRo83RBvk6WH9sOegvpFXK3Kic4LUGD9n4C23HjAFxQdHvxtgO4g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

**DcmDsdSubServiceFnc**：ECU Supplier特定组件针对特定服务的回调函数。ISOLAR工具没有提供这个配置项。

 

**DcmDsdSubServiceId**：子服务ID。

 

**DcmDsdSubServiceUsed**：是否使用该子服务。ISOLAR工具没有提供这个配置项。

 

**DcmDsdSidTabModeRuleRe****f**：对控制服务执行的DcmDspModeRule的引用。如果没有配置引用，则不进行模式规则检查。

 

**DcmDsdSidTabSecurityLevelRef**：引用允许在其中执行服务的安全级别。一个服务允许多个引用。

 

**DcmDsdSidTabSessionLevelRef**：对允许执行服务的会话级别的引用。一个服务允许多个引用。

 

#### **6.1.2.5 DcmDsl**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgcgsPzzx5r48FicxuZvkC2FBhOFavKMoxtt4KMlMPfjA5HJ25dicL5m5w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

**DcmDslBuffer**：此容器包含诊断缓冲区的配置。

 

**DcmDslCallbackDCMRequestService**：每个DcmDslCallbackDCMRequestService容器使用CallbackDCMRequestServices接口定义一个R-Port, 应用使用使用接口向Dcm模块请求协议更改的权限。

 

**DcmDslDiagResp**：此容器包含Dcm中自动 requestCorrectlyReceivedResponsePending响应管理的配置。

 

**DcmDslProtocol**：这个容器包含Dcm中使用的诊断协议的配置。

 

#### **6.1.2.6 DcmDslBuffer**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgODd0MXUZEEiaFDl5nWEtC3gQIDAIl2HiboRaP1bERILAIFXibbsOicLiatA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**DcmDslBufferSize**：诊断缓冲区大小(以字节为单位)。对于线性缓冲区，大小应该和最长的诊断消息(请求或响应)一样大。对于分页缓冲区，大小会影响应用程序性能。

 

#### **6.1.2.7 DcmDslCallbackDCMRequestService**

一般实际应用中不使用这个配置。

 

#### **6.1.2.8 DcmDslDiagResp**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgibBAzBVHfpIic3bbCDFqGMBboYB1wlGCrRl5doRA8TZmTPS5gzSahGkQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

**DcmDslDiagRespMaxNumRespPend**：一个请求允许的带有响应码0x78 (requestCorrectlyReceivedResponsePending)的最大否定响应数。如果Dcm达到此限制，将自动发送0x10 (generalReject)最终响应，并取消服务处理。

 

#### **6.1.2.9 DcmDslProtocol**

**DcmDslProtocolRow**：该容器包含Dcm中使用的一种特定诊断协议的配置。

 

#### **6.1.2.10 DcmDslProtocolRow**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgtKkcRJ3tvLKPR5X0WGr4JVkGPzdH4icYfLS7SnYBtfnJ4GzbIcEl4rw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**DcmDslProtocolID**：正在配置的DCM DSL协议的诊断协议类型。一般都是基于UDS的CAN诊断。

 

**DcmDslProtocolMaximumResponseSize**：定义响应消息的最大长度。

 

**DcmDslProtocolPriority**：协议抢占时使用的协议优先级。高优先级的协议可能会抢占低优先级的协议。数值越低表示协议优先级越高。

 

**DcmDslProtocolTransType**：仅当协议类型为DCM_ROE_ON_xxx时，此参数有效。

 

**DcmSendRespPendOnTransToBoot**：参数指定ECU在转换到引导加载程序之前是否应该发送NRC 0x78(响应等待)(参数设置为TRUE)，或者是否应该启动转换而不发送NRC 0x78(参数设置为FALSE)。

 

**DcmTimStrP2ServerAdjust**：该参数用于确保在总线上的诊断响应在到达P2之前可用，通过调整当前的DcmDspSessionP2ServerMax。该参数主要表示由DCM发起传输到消息实际传输到总线的时间之间依赖于软件架构的通信延迟。

 

**DcmTimStrP2StarServerAdjust**：该参数用于保证在到达P2Star之前，总线上的诊断响应是可用的，通过调整当前的DcmDspSessionP2StarServerMax。该参数主要表示由DCM发起传输到消息实际传输到总线的时间之间依赖于软件架构的通信延迟。

 

**DcmDemClientRef**：在Dem配置中引用DemClient。由Dem用于区分不同的客户端调用。

 

**DcmDslProtocolRxBufferRef**：引用已配置的诊断缓冲区，该缓冲区用于接收协议的诊断请求。

 

**DcmDslProtocolSIDTable**：对用于此协议的诊断请求处理的服务表的引用。

 

**DcmDslProtocolTxBufferRef**：引用已配置的诊断缓冲区，用于传输协议的诊断响应。

 

**DcmDslConnection**：这个容器包含一个特定协议的通信通道配置。注意，它允许与多个Tester通信，因此可以为一个协议配置多个连接。

 

#### **6.1.2.11 DcmDslConnection**

**DcmDslMainConnection****：**这个容器包含诊断协议的主连接的配置。

 

**DcmDslPeriodicTransmission****：**这个容器包含定期传输连接的配置。实际应用中一般使用不到。

 

**DcmDslResponseOnEvent**：该容器包含ResponseOnEvent连接的配置。

 

#### **6.1.2.12 DcmDslMainConnection**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgiawEofnLnPWUQd2L9jrp6wEuBWjZRs4icpG9G9iaLSt3gRHLhlgfKstrg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

**DcmDslProtocolRxTesterSourceAddr**：测试器源地址唯一地描述了一个客户端，并将在跳转到Bootloader接口中使用。对于通用连接(MetaDataLength >= 1的DcmPdus)，不需要此参数。

 

**DcmDslPeriodicTransmissionConRef**：对用于处理周期性传输事件的周期性传输连接的引用。实际应用中一般很少使用。

 

**DcmDslProtocolComMChannelRef**：引用接收DcmDslProtocolRxPdu和发送DcmDslProtocolTxPdu的ComMChannel。

 

**DcmDslROEConnectionRef**：引用一个用于处理ResponseOnEvent事件的ResponseOnEvent连接。实际应用中一般很少使用。

 

**DcmDslProtocolRx**：此容器包含诊断连接中接收通道的配置参数。

 

**DcmDslProtocolTx**：此容器包含诊断连接中传输通道的配置参数。

 

#### **6.1.2.13 DcmDslProtocolRx**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgrhIINAq3yYAR3sfFx07Iic83oNGYJslyusreWcjV5AK8Lk5Omm0KFcw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

**DcmDslProtocolRxAddrType** ：选择接收信道的寻址类型。1:1通信采用物理寻址，1:1通信采用功能寻址。

 

**DcmDslProtocolRxPduRef**：参考EcuC中用于此接收通道的Pdu。

#### **6.1.2.14 DcmDslProtocolTx**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgJHIE1KNx7ahqtSkKFD63LsfkeoHxNqLY9t3pwlJGeR6x8yy3I1pGyQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

**DcmDslProtocolTxPduRef**：参考EcuC中用于此传输通道的Pdu。

 

#### **6.1.2.15 DcmDsp**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgnKsb5qqpPDTQbTbJcMQwRwOJ2Hyte1hX7ePNib588SGfy8PiaOHeusLQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

**DcmDspDDDIDcheckPerSourceDID**: 使用ReadDataByIdentifier (0x22)定义对每个源DIDs的会话、安全性和模式依赖项的检查。

true: Dcm模块应该用一个ReadDataByIdentifier (0x22)检查每个源DIDs的会话、安全性和模式依赖关系，DID的范围是0xF200到0xF3FF。

 

false: Dcm模块不检查每个源DIDs的会话、安全性和模式依赖关系，每个源DIDs的ReadDataByIdentifier (0x22)的DID范围是0xF200到0xF3FF。

 

**DcmDspMaxDidToRead**: 指示在单个“ReadDataByIdentifier”请求中允许的最大DIDs。

 

**DcmDspMaxPeriodicDidToRead**: 允许在单个“ReadDataByPeriodicIdentifier”请求中读取的最大periodicDIDs。

 

**DcmDspPowerDownTime**: 此参数向客户端指示服务器将保持下电顺序的备用顺序的最小时间。

 

**DcmDspClearDTC**: 该容器包含Clear DTC服务的配置。

 

**DcmDspComControl**: 提供CommunicationControl机制的配置。

 

**DcmDspCommonAuthorization**: 该容器包含多个服务/子服务相同的公共授权的配置(参数)。

 

**DcmDspControlDTCSetting**: 提供ControlDTCSetting机制的配置。

 

**DcmDspData**: 该容器包含属于DID的Data的配置(参数)。

 

**DcmDspDataInfo**: 这个容器包含一个Data的配置(参数)。

 

**DcmDspDid**: 这个容器包含DID的配置(参数)。

 

**DcmDspDidInfo**: 这个容器包含DID的Info的配置(参数)。

 

**DcmDspDidRange**: 这个容器定义DID范围。

 

**DcmDspEcuReset**: 该容器包含DcmDspEcuReset服务的配置。

 

**DcmDspMemory**: 这个容器包含内存访问的配置。

 

**DcmDspPeriodicTransmission**: 此容器包含定期传输调度程序的配置(参数)。

 

**DcmDspPid**: 该容器定义了PID对DCM的可用性。

 

**DcmDspRequestControl**: 此容器包含“请求控制机载系统、测试或组件”服务(服务$08)的配置(参数)。

 

**DcmDspRequestFileTransfer**: 该容器包含RequestFileTransfer的配置。该容器仅在配置了RequestFileTransfer时才存在。

 

**DcmDspRoe**: 提供ResponseOnEvent机制的配置。

 

**DcmDspRoutine**: 这个容器包含例程的配置(参数)>

 

**DcmDspSecurity**: 该容器包含安全级别配置(DSP参数)(每个安全级别)描述该容器包含DcmDspSecurityRow的行。

 

**DcmDspSession**: 父容器保存单行来配置特定的会话。

 

#### **6.1.2.16 DcmDspComControlAllChannel**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgB8EeYf0FPzyqpibZibzklGCiaIF1XsP4gaA0p7IFkFMVpBXOHmhQR5ibibQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

DcmDspAllComMChannelRef: 引用ComM通道。

 

#### **6.1.2.17 DcmDspCommonAuthorization**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgJ1rS5fPyicrz55tlKSCSWttk1UpibcicmgOtMu5iam4n819veNMSIt2ptg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

**DcmDspCommonAuthorizationModeRuleRef**: 引用一个DcmModeRule。控制此服务/子服务的模式规则。如果没有参考，则不进行模态规则检查。

 

**DcmDspCommonAuthorizationSecurityLevelRef**: 引用DcmDspSecurityRow允许控制此服务/子服务的安全级别。如果没有引用，则不进行安全级别检查。

 

**DcmDspCommonAuthorizationSessionRef**: 引用DcmDspSessionRow允许控制此服务/子服务的会话。如果没有引用，则不应检查会话级别。

#### **6.1.2.18 DcmDspControlDTCSetting**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgvI0f4kiczVCIW0UkUhd0t8l00giam3BNR7v3m7jJXOiaAU3Z9SsicNw9jw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**DcmSupportDTCSettingControlOptionRecord**: 这个配置开关定义请求消息中是否通常支持DTCSettingControlOptionRecord。

 

**DcmDspControlDTCSettingReEnableModeRuleRef**：引用DcmModeRule。控制DCM重新启用controlDTCsetting的模式规则。DCM模块应执行一个ControlDTCSetting。关闭(调用Dem_EnableDTCSetting())，以便不再满足所引用的模式规则。

 

#### **6.1.2.19 DcmDspData**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgunW02pIarnicPL9ZgNm9L5YWwSv7SvEibUg1Ts0AfQ6v7fqxcDiceunzg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

**DcmDspDataByteSize**: 以字节为单位定义数组长度或变量的最大数组长度。

 

**DcmDspDataConditionCheckReadFnc**: 这是函数名字（函数指针）。如果读取这个DID的时候需要应用去检查一些系统状态，就可以定义这个函数去实现这个功能。

 

**DcmDspDataConditionCheckReadFncUsed**: 状态检查函数是否被使用。

 

**DcmDspDataEndianness**: 定义诊断请求或响应消息中属于DID的数据的字节顺序

 

**DcmDspDataFreezeCurrentStateFnc**: 函数名（函数指针），请求应用程序冻结IOControl的当前状态。(FreezeCurrentState-function)。

**DcmDspDataGetScalingInfoFnc**: 函数名（函数指针），向应用程序请求DID的伸缩信息。(GetScalingInformation-function)。该参数与接口Xxxx_GetScalingInformation有关。

 

**DcmDspDataReadDataLengthFnc**: 函数名（函数指针），向应用程序请求DID的数据长度。(ReadDataLength-function)。该参数与接口Xxx_ReadDataLength有关。

 

**DcmDspDataReadEcuSignal**: 由DCM读访问某个ECU信号的函数名称。(IoHwAb_Dcm_Read < EcuSignalName >function)。仅当DcmDspDataUsePort==USE_ECU_SIGNAL有效。

 

**DcmDspDataReadFnc**: 函数名向应用程序请求DID的数据值。(ReadData-function)。该参数与接口Xxx_ReadData有关。

 

**DcmDspDataUsePort**: 定义要访问数据的接口应使用。

 

#### **6.1.2.20 DcmDspDidInfo**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgjkSkcUO7zhjJAPG9NMuC4yfkQ2tDeBgLbsojJNZGxpUV5iaibUkibibSOA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

#### **6.1.2.21** **DcmDspDidRead**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgQqNChZqONW7jAqTxCbSmmAhb3rgRPVQupBK8u3IGyibyBTBribTHot6g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### **6.1.2.22 DcmDspDidWrite**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgFnibanQfO4XANfmKxfUDmzZFTjS7ZBYBo3DSVATkDcHT0FLSIaAkmnA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### **6.1.2.23 DcmDspDid**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgWLxv5RGBkSUcw3ZaY4tWpHNmYBXo2hqdDXLbk0d1edaCP5DskaE8hw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### **6.1.2.24 DcmDspDidSignal**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFg8dibV0aPibPibxJNzM2ZK7cPQKz2fiarTibSaHd1LkJEzAmznw3w0AI9eGg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### **6.1.2.25 DcmDspRouting**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgqLlvibrGNS1NibDLlxmwBNMJCzIerIZVOvVEyWJC6X1OicYJYARAXzs5w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### **6.1.2.26 DcmDspRequestRoutineResults**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFg4XyxHo0PulAjoVYmiarbicImujibc5oPn3XFzPGSEsYa0icAweuNNZ40Rg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### **6.1.2.27 DcmDspStartRoutine**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgRt5hibib9TtHRFDeziaWJMDGAzPBR7zJ1s6JQQaU7sHsc1fibndTLsQN1A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### **6.1.2.28 DcmDspStopRoutine**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgEkQ0QjngFeSo81IJFKn3bsqXVFC0bEBQe3HsotDBykIQe40Bl68zlg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### **6.1.2.29 DcmDspSecurity**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgB0RypjRdJLfkib9aSrbkI6CJRibplpfibbp4Qpk2pUd5Pyu2sCVDtpgqQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### **6.1.2.30 DcmDspSecurityRow**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgB0RypjRdJLfkib9aSrbkI6CJRibplpfibbp4Qpk2pUd5Pyu2sCVDtpgqQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### **6.1.2.31 DcmDspSessionRow**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcyZEIftTd7XJvqibfJZTwsFgFoS9r9N1KombYqWn37COgzTPlYCw5P8dKuoiaqfPUdzceKF77sMaFJQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)