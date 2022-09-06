# AUTOSAR 通信服务-Dcm子模块DSP详解

**正文**

## **3.4 诊断服务处理层(DSP)**

### **3.4.1 DSP功能概述**

DSP为诊断服务的处理，当接收到DSD请求处理诊断服务，服务处理过程：

. 分析收到的请求消息

. 检查格式和是否支持寻址子功能

. 在DEM、SWC或其他BSW模块上获取数据或执行所需的函数调用

. 组装响应

 

#### **3.4.1.1 检查格式及是否支持其子功能**

DSP对请求消息的分析导致格式错误或长度错误时，DSP子模块将触发NRC 0x13 (Incorrect message length or invalid format)的负响应。

 

#### **3.4.1.2 组装消息**

DSP子模块通过对包含响应服务标识符的响应消息进行组装，确定响应消息长度。

 

#### **3.4.1.3 否定响应码的处理**

除非另一个特定的NRC被指定，否则当API调用执行服务没有返回OK时，DSP子模块应该触发NRC 0x10 (generalReject)的负响应。

 

#### **3.4.1.4 诊断模式声明组**

Dcm作为诊断模式的模式管理员，管理以下7类模式切换:

1). DcmDiagnosticSessionControl (service 0x10)

2). DcmEcuReset (partly service 0x11)

3). DcmSecurityAccess (service 0x27)

4). DcmModeRapidPowerShutDown (partly service 0x11)

5). DcmCommunicationControl_<symbolic name of ComMChannelId>. (service

0x28)

6). DcmControlDTCSetting (service 0x85)

7). DcmResponseOnEvent_<RoeEventID> (service 0x86)

 

以DcmSecurityAccess为例，DCM声明安全访问相关的模式组:

ModeDeclarationGroup DcmSecurityAccess

{

{

 DCM_SEC_LEV_LOCKED

DCM_SEC_LEV_1

 ...

DCM_SEC_LEV_63

}

 initialMode = DCM_SEC_LEV_LOCKED

};

 

ModeSwitchInterface  SchM_Switch_<bsnp>_DcmSecurityAccess

{

 isService = true;

 SecLevel currentMode;

};

 

#### **3.4.1.5 模式相关的请求执行**

可以根据模式条件限制请求的执行。这使得Dcm能够格式化的进行环境检查。

 

#### **3.4.1.6 Sender/Receiver通信**

如果配置容器DcmDspData的参数DcmDspDataUsePort配置为USE_DATA_SENDER_RECEIVER或者USE_DATA_SENDER_RECEIVER_AS_SERVICE则DcmDspDataUsePort为一个R-Port，需要为这个R-Port顶一个Interface（一个参数，参数的实现数据类型为 DcmDspDataType）。

 

如果配置容器DcmDspData的参数DcmDspDataUsePort配置为USE_DATA_SENDER_RECEIVER或者USE_DATA_SENDER_RECEIVER_AS_SERVICE则DcmDspDataUsePort为一个P-Port，需要为这个P-Port顶一个Interface（一个参数，参数的实现数据类型为 DcmDspDataType）。

 

#### **3.4.1.7 将SwDataDefProps属性从DEXT文件传递到Dcm Service SW-C**

用例:将SwDataDefProps细节如CompuMethod、DataContraints和Units传递给Dcm Service SW-C，并使它们在每个DID DataElement / RoutineControl信号中可用。有两个可选的工作流程。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rczh3oBUh7URLj3ld89iaP162XdyKh1Te7EeXatx1Xbf5J9R6tmSTosrGLCIbdg98P0OpU8OBF0eJdQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rczh3oBUh7URLj3ld89iaP162kF0U7xVia2dK3eD5iabumOLXibMhrxjcolezWVRAzct9EsQrHRIicjnEQw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



#### **3.4.1.8 异步调用行为**

如果一个Dem函数返回DEM_PENDING, Dcm将在稍后的时间点再次调用该函数。

 

如果一个诊断任务请求的负面响应数量达到配置参数eter DcmDslDiagRespMaxNumRespPend中定义的值，Dcm模块将停止处理活动的诊断请求，通知应用程序或BSW(如果这个诊断任务意味着调用一个SW-C接口或BSW接口)，通过设置OpStatus参数，active端口接口，DCM_CANCEL，并发送一个NRC 0x10的否定响应(一般拒绝)。

 

如果之前的返回状态是E_PENDING，那么Dcm_SetProgConditions API将在下一个Dcm主函数循环中被再次调用。

 

DCM_E_PENDING的返回应该做一个重新触发(例如在下一个MainFunction循环中)。

 

调用OpStatus为DCM_CANCEL的接口的返回值将被忽略。

### **3.4.2 UDS服务**

Dcm模块需要支持以下UDS服务

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rczh3oBUh7URLj3ld89iaP162VgbPvx1UDm8c2xoicUzMWxWWaCrWRQpyjdH6Jia6AbW1ciaXZFTXdJueA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

*Note:具体每个服务的介绍，请关注本公众号的后续文章*



# **2.关键****API**

## **2.1 F****unction definitions**

### **2.1.1** **Dcm_Init**

函数原型：void Dcm_Init(const Dcm_ConfigType* ConfigPtr)

参数：

ConfigPtr：指向Dcm配置集合的指针（指向const数组的指针）

返回值：无

功能描述：根据动态配置初始化Dcm模块

 

### **4.1.2 Dcm_DemTriggerOnDTCStatus**

函数原型：

Std_ReturnType Dcm_DemTriggerOnDTCStatus(uint32 DTC, Dem_UdsStatusByteType DTCStatusOld, Dem_UdsStatusByteType DTCStatusNew)

 

参数：

DTC：DTC标识符（代表某个具体的DTC）

DTCStatusOld：DTC改变前的状态

DTCStatusNew：DTC改变后的状态

返回值：总是返回E_OK

功能描述：UDS状态字改变后就会调用这个接口，改变DTC状态。

 

### **2.1.2** **Dcm_GetVin**

函数原型：

Std_ReturnType Dcm_GetVin(uint8* Data)

 

参数：

Data：存放VIN码地址的指针

 

返回值:

E_OK：成功获取VIN到Data指向的地址空间

E_NOT_OK：在DoIP中将使用默认VIN

 

功能描述：获取VIN码

 

### **2.1.3** **Dcm_GetSecurityLevel**

函数原型：

Std_ReturnType Dcm_GetSecurityLevel(Dcm_SecLevelType* SecLevel)

 

参数：

SecLevel：获取的安全等级值存放到SecLevel指针指向的内存。

 

返回值：总是返回E_OK

 

功能描述：获取安全等级值。

 

### **2.1.4** **Dcm_GetSesCtrlType**

函数原型：

Std_ReturnType Dcm_GetSesCtrlType(Dcm_SesCtrlType* SesCtrlType)

参数：

SesCtrlType：获取的会话状态值存放到SesCtrlType指针指向的内存。

 

返回值：总是返回E_OK

 

功能描述：获取会话状态值。

 

### **4.1.5Dcm_ResetToDefaultSession**

函数原型：

Std_ReturnType Dcm_ResetToDefaultSession(void)参数：

 

返回值：总是返回E_OK

 

功能描述：将当前会话状态切换到default默认状态。

 

### **4.1.6Dcm_ SetActiveDiagnostic**

函数原型：

Std_ReturnType Dcm_SetActiveDiagnostic(boolean active)

参数：

Active：是否激活诊断功能的标志。

 

返回值：总是返回E_OK

 

功能描述：如果active == true，DCM就会调用ComM_DCM_ActiveDiagnostic()，否则不调用。

 

## **4.2** **Call-back notifications**

介绍底层BSW模块提供的功能。回调函数的函数原型将在Dcm_Cbk.h文件中提供。也就是Dcm模块实现这些回调函数，其他模块（ComM, PduR等）使用这些回调函数。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcx5732ibq8dLFqw2tKicIOVdGxDvJb2HvNIr6gSsm0icBCxiaxnr1ibY7XPgCaH49DiaHH8Wia8uxZxvWDjQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



 

### **4.2.1Dcm_StartOfReception**

BufReq_ReturnType Dcm_StartOfReception(

PduIdType id,

const PduInfoType* info,

PduLengthType TpSduLength,

PduLengthType* bufferSizePtr

)

 

功能描述：PduR模块接收到诊断报文的第一帧（单针SF, 或者多帧的首帧FF）时调用，通知Dcm模块开始接收诊断报文。

 

### **4.2.2 Dcm_CopyRxData**

BufReq_ReturnType Dcm_CopyRxData(

PduIdType id,

const PduInfoType* info,

PduLengthType* bufferSizePtr

)

 

功能描述：PduR模块接收到多帧的连续帧（CF）时调用，将诊断数据传递到DCM模块。

 

### **4.2.3 Dcm_TpRxIndication**

void Dcm_TpRxIndication(

PduIdType id,

Std_ReturnType result

)

 

功能描述：在通过TP API接收到I-PDU后调用。

 

### **4.2.4 Dcm_CopyTxData**

BufReq_ReturnType Dcm_CopyTxData(

PduIdType id,

const PduInfoType* info,

const RetryInfoType* retry,

PduLengthType* availableDataPtr

)

 

功能描述：调用该函数获取I-PDU段(N-PDU)的传输数据。

 

### **4.2.5 Dcm_TpTxConfirmation**

void Dcm_TpTxConfirmation(

PduIdType id,

Std_ReturnType result

)

 

功能描述：这个函数是在I-PDU在其网络上传输后调用的，结果表明传输是否成功。

 

### **4.2.6 Dcm_TxConfirmation**

void Dcm_TxConfirmation(

PduIdType TxPduId,

Std_ReturnType result

)

 

功能描述：下层通信接口模块确认PDU发送，或者PDU发送失败。

 

### **4.2.7  Dcm_ComM_NoComModeEntered**

void Dcm_ComM_NoComModeEntered(

uint8 NetworkId

)

功能描述：ComM模块通知Dcm模块NetworkId标识的网络进入了COMM_NO_COMMUNICATION状态。

 

**4.2.8 Dcm_ComM_SilentComModeEntered**

void Dcm_ComM_SilentComModeEntered(

uint8 NetworkId

)

 

功能描述：ComM模块通知Dcm模块NetworkId标识的网络进入了COMM_SILENT_COMMUNICATION状态。

 

**4.2.8 Dcm_ComM_FullComModeEntered**

void Dcm_ComM_FullComModeEntered(

uint8 NetworkId

)

 

功能描述：ComM模块通知Dcm模块NetworkId标识的网络进入了COMM_FULL_COMMUNICATION状态。

 

## **4.3 Callout Definitions**

Callouts是在ECU集成过程中必须添加到Dcm的代码片段。大多数Callouts的内容是手写的代码，对于一些Callouts，Dcm配置工具会生成一个默认实现，由集成器手动编辑。从概念上讲，这些Callouts属于ECU固件。

 

由于Callouts不是Dcm的服务，它们没有分配的服务ID。注意:Autosar架构不提供使用物理地址访问ECU内存的可能性。这是使用标识内存块的BlockId实现的。

 

根据这一点，Dcm不能完全支持ISO14229- 1服务的实现，这些服务要求物理内存访问。因此，Dcm定义了callout来实现这种内存访问。通过定义BlockId和物理内存地址之间的映射，可以简单地实现这个调出实现。

 

## **4.4Scheduled functions**

### **4.4.1** **Dcm_MainFunction**

void Dcm_MainFunction(void)

 

## **4.5Expected interfaces**

### **4****.****5****.1 Mandatory interfaces**

实现模块核心功能所需的所有接口

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcx5732ibq8dLFqw2tKicIOVdGfF9WRFxibBfycILWhibaY2OSMc7NhlnBTaNdfPgqhbV2wqH6Wc2zYNkQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



### **4.5.2 Optional interfaces**

实现模块的一个可选功能所必需的接口

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcx5732ibq8dLFqw2tKicIOVdGq17LQMGa2W6UFf6TfEHgI6q2M0HicNOT68uuweTibyavXv9w7awricEibw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcx5732ibq8dLFqw2tKicIOVdGKlyGXEwlBBw8skJvaO3C0jeibX1GYlCCEGAic9fenwicfia6KibcfgARTXg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcx5732ibq8dLFqw2tKicIOVdGRictibClMbsjV9rpKPEFnWQwqkueEoSictIsEaswYicbQDrJFTWMKQfic1A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)





### **4.5.3 Configurable interfaces**

Dcm配置中定义Dcm将使用的实际函数的接口。根据配置的不同，这些功能的实现可以由其他bsw模块(通常是DEM)或软件组件(通过RTE)提供。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcx5732ibq8dLFqw2tKicIOVdGibMm0uMpZvXCbLq03qgKMdgdOhnEYzRuzkETIaapMrbriaQeCmmF43rQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6Rcx5732ibq8dLFqw2tKicIOVdGbwia0ZMgqictvv8d9MMNIp3oKKicMvfXNoLYCdAHQH3lH1EkSgOBsPh1Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)