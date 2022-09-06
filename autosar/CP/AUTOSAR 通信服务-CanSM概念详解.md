# AUTOSAR 通信服务-CanSM概念详解

AUTOSAR CanSM模块的分享分为CanSM模块概念详解和CanSM模块配置及代码分析，具体的项目实战请关注本号的后续文章，本篇为CanSM模块的概念详解篇。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxHibIHsfRkcqicSOeViaYRY1cyhbCrs3Adl2ByIyIRQ85FGgOF3SZQSI7nwjiaq3FlQUFT2549R4lYcA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



# **1 Introduction and functional overview**

AUTOSAR BSW栈为每个通信总线指定一个总线特定的状态管理器。CANSM实现CAN总线的数据流控制。CanSM隶属于通信服务层。CanSM和通信硬件抽象层以及系统服务层交互。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/CJj2CPh6RcxHibIHsfRkcqicSOeViaYRY1chVYogByqw8DsyDVkia7A9os1lyPjjdOnzq3SrZIS5muaCPsbeB5rn7Q/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

CanSM只用用于控制CAN通信。CanSM的任务就是操作CanIf模块去控制一个或者多个CAN控制器或者收发器驱动。

 

# 2. **Dependencies to other modules**

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxHibIHsfRkcqicSOeViaYRY1clBx3E2D74j9TZWI6gQuEZgplicWRlG31zsib0omFXpMUj5AfnHLa6RZg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**ECU State Manager (EcuM)**

EcuM模块初始化CanSM模块，并与CanSM模块进行交互，以进行CAN唤醒验证。

 

**BSW Scheduler (SchM)**

SchM模块周期调用CanSM模块的MainFunction函数。

 

**Communication Manager (ComM)** 

ComM模块使用CanSM模块的API来请求CAN网络的通信模式，并使用唯一的网络句柄进行标识。CanSM模块通知当前CAN网络的通信模式给ComM模块。

 

**CAN Interface (CanIf)**

CanSM模块使用CanIf模块的API来控制分配给CAN网络的CAN控制器和CAN收发器的操作模式。CanIf模块将外设事件通知给CanSM模块。

 

**Diagnostic Event Manager (DEM)**

CanSM模块将特定的总线错误上报给DEM模块。

 

**Basic Software Mode Manager (BswM)**

CanSM模块将总线模式的改变通知到BswM模块。

 

**CAN Network Management (CanNm)**

CanSM模块需要将能用的CAN网络通知给CanNM模块，CanSM模块也需要处理CanNM模块通知的超时异常。

 

**Default Error Tracer (DET)**

CanSM模块将运行时的错误上报给DET模块。

 

# 3. **Functional specification**

一个ECU可以有不同的通信网络。每个网络都必须用一个唯一的网络句柄来标识。通信模块向网络请求通信模式。它通过它的配置知道哪个句柄被分配给了什么类型的网络。对于CAN，它使用CanSM模块。CanSM模块负责CAN网络的控制流抽象:它根据来自ComM模块的模式请求改变配置的CAN网络的通信模式。

 

因此CanSM模块使用CanIf模块的API。CanIf模块负责所配置的CAN控制器和CAN收发器的控制流抽象(CanIf模块的数据流抽象与CanSM模块无关)。CAN控制器模式和CAN收发器的任何改变

 

模式将由CanIf模块通知到CanSM模块。根据CAN网络状态机的通知和状态，CanSM模块将通知ComM和BswM, CanSM模块将为每个配置的CAN网络实现这些通知和状态

 

## 3.1 **General requirements**

CanSM模块应该将每一个配置了的CAN通道的当前模式存储下来。

 

一个CAN网络没有配置CanSMTransceiverId参数，那么CanSM模块将绕过所有指定的CAN网络的CanIf_SetTrcvMode调用，并继续不同的状态转换，就像它已经获得了假定的 CanSM_TransceiverModeIndication。也就是说，一个CAN网络没有配置CanSMTransceiverId参数，当上层模块调用CanIf_SetTrcvMode函数，CanSM模块认为CAN模式转换成功，切换CanSM内部存储的CAN模式，同时调用CanSM_TransceiverModeIndication模式切换函数通知到上层模块。

 

上层模块对每一个CAN网络的最近一次调用CanSM_RequestComMode函数且返回成功E_OK的请求应该被CanSM模块存储下来，这些请求时CAN网络状态机切换的条件。

 

## 3.2 **State machine for each CAN network**

CanSM模块为每一路CAN网络维持一个内部的状态机，包括：**CANSM_BSM_S_FULLCOM，CANSM_BSM_S_PRE_FULLCOM，CANSM_BSM_S_CHANGE_BAUDRATE，CANSM_BSM_WUVALIDATION，CANSM_BSM_S_SILENTCOM_BOR，CANSM_BSM_S_SILENTCOM，CANSM_BSM_S_PRE_NOCOM，CANSM_BSM_S_NOCOM，CANSM_BSM_S_NOT_INITIALIZED** 9种状态。

 

以下对于CanSM内部状态机切换的描述都是站在一路CAN状态机的角度来描述的。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxHibIHsfRkcqicSOeViaYRY1cD2MNUQic8ZGBcm8bmZAdG8VtCmRWCDDgKarWKNXwZicb6UygpMudP6dw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)





### 3.2.1 **Trigger: PowerOn**

ECU上电后，每一路CanSM状态机默认进入CANSM_BSM_S_NOT_INITIALIZED状态。

 

### 3.2.2 **Trigger: CanSM_Init**

EcuM模块调用CanSM_Init函数，CanSM状态从CANSM_BSM_S_NOT_INITIALIZED切换到CANSM_BSM_S_PRE_NOCOM状态。

 

### **3****.2.3 Trigger: CanSM_DeInit**

只有在CANSM_NO_COMMUNICATION状态下调用CanSM_DeInit函数才会发生状态改变，CanSM状态从CANSM_NO_COMMUNICATION状态且切换到**CANSM_BSM_S_NOT_INITIALIZED**状态**。**

 

**CanSM_DeInit****函数被调用且成功后会触发CanSM模块管理的所有CAN网络发生状态改变。**

 

### **3.2.4** **Trigger: T_START_WAKEUP_SOURCE**

CanSM状态机处于CANSM_BSM_S_PRE_NOCOM或者CANSM_BSM_S_NOCOM状态下， 上层模块调用CanSM_StartWakeUpSource开始唤醒源检测，并且函数返回E_OK，则CanSM状态机切换到CANSM_BSM_WUVALIDATION状态。

 

### **3****.2.5 Trigger: T_STOP_WAKEUP_SOURCE**

CanSM状态机处于CANSM_BSM_WUVALIDATION状态时，上层模块调用 CanSM_StopWakeUpSource函数停止唤醒源检测，并且函数返回E_OK，则CanSM状态机切换到CANSM_BSM_S_PRE_NOCOM状态。

 

### **3****.2.6 Trigger: T_FULL_COM_MODE_REQUEST**

CanSM状态机处于CANSM_BSM_WUVALIDATION或者CANSM_BSM_S_NOCOM状态时，上层模块调用CanSM_RequestComMode且函数参数ComM_Mode等于COMM_FULL_COMMUNICATION时，则函数参数 network标识的网络CanSM状态机切换到CANSM_BSM_S_PRE_FULLCOM状态。

 

### **3****.2.7 Trigger: T_NO_COM_MODE_REQUEST**

CanSM状态机处于CANSM_BSM_S_FULLCOM或者CANSM_BSM_S_PRE_FULLCOM或者CANSM_BSM_S_SILENTCOM状态时，上层模块调用CanSM_RequestComMode且函数参数ComM_Mode等于COMM_NO_COMMUNICATION时，则函数参数 network标识的网络CanSM状态机切换到CANSM_BSM_S_PRE_NOCOM状态。

 

### **3****.2.8 Trigger: T_BUS_OFF**

CanSM状态机处于CANSM_BSM_S_SILENTCOM状态时，CAN网络发生BusOff实践，CanSM模块的回调函数 CanSM_ControllerBusOff被调用，CanSM状态机切换到CANSM_BSM_S_SILENTCOM_BOR状态。

 

### **3****.2.9 Trigger: T_REPEAT_MAX**

CanSM状态机处于CANSM_BSM_S_SILENTCOM_BOR状态时，CanSM模块调用CanIf模块的接口尝试从新开启CAN控制器，如果没有返回E_OK且没有收到预定的开始CAN控制器的成功通知的次数（失败的次数）大于配置参数T_REPEAT_MAX后，状态机机切换到CANSM_BSM_S_PRE_NOCOM状态。

 

### **3.2.10 Guarding condition: G_FULL_COM_MODE_REQUESTED**

CanSM状态机处于CANSM_BSM_S_CHANGE_BAUDRATE状态时，上层模块（ComM）调用CanSM_RequestComMode函数请求Can通信模式的最后的请求是全通信模式（ComM_Mode等于COMM_FULL_COMMUNICATION）,状态机切换到CANSM_BSM_S_FULLCOM状态。

 

### **3****.2.11 Guarding condition: G_SILENT_COM_MODE_REQUESTED**

CanSM状态机处于CANSM_BSM_S_CHANGE_BAUDRATE状态时，上层模块（ComM）调用CanSM_RequestComMode函数请求Can通信模式的最后的请求是静默模式（ComM_Mode等于COMM_SILENT_COMMUNICATION）,状态机切换到CANSM_BSM_S_SILENTCOM状态。

 

### **3.2.12 Effect: E_PRE_NOCOM**

CanSM状态机处于CANSM_BSM_S_FULLCOM或者CANSM_BSM_S_SILENTCOM或者CANSM_BSM_S_SILENTCOM_BOR状态，CanSM状态机由于满足T_NO_COM_MODE_REQUEST,T_REPEAT_MAX或者T_NO_COM_MODE_REQUEST条件切换到CANSM_BSM_S_PRE_NOCOM状态，则BswM_CanSM_CurrentState函数将被调用（ Network := CanSMComMNetworkHandleRef and CurrentState := CANSM_BSWM_NO_COMMUNICATION）。

 

### **3****.2.13 Effect: E_NOCOM**

CanSM状态机处于CANSM_BSM_S_PRE_NOCOM状态切换到CANSM_BSM_S_NOCOM状态时，如果已经存在网络的通信模式请求，且存储的通信模式请求为COMM_NO_COMMUNICATION，那么CanSM_BSM状态机的效果E_NOCOM将使用参数Channel:= CanSMComMNetworkHandleRef和ComMode:= COMM_NO_COMMUNICATION调用API ComM_BusSM_ModeIndication。

 

### **3****.2.14 Effect: E_FULL_COM**

CanSM状态机从CANSM_BSM_S_PRE_FULLCOM状态切换到CANSM_BSM_S_FULLCOM状态时，将会产生以下的动作（Effect）。

（1）如果 ECU被动模式（ECU passive mode）等于true，则 CanIf_SetPduMode被调用（ ControllerId := CanSMControllerId， PduModeRequest :=CANIF_ONLINE.）。

 

（2）如果 ECU被动模式（ECU passive mode）等于false，则 CanIf_SetPduMode被调用（ ControllerId := CanSMControllerId， PduModeRequest :=CANIF_TX_OFFLINE_ACTIVE.）。

 

（3）CanSM模块将会调用 ComM_BusSM_ModeIndication函数（Channel :=

CanSMComMNetworkHandleRef， ComMode := COMM_FULL_COMMUNICATION）。

 

（4）CanSM模块将会调用 BswM_CanSM_CurrentState函数（ Network :=

CanSMComMNetworkHandleRef，CurrentState :=CANSM_BSWM_FULL_COMMUNICATION）。

 

### **3****.2.15 Effect: E_FULL_TO_SILENT_COM**

CanSM状态机从CANSM_BSM_S_FULLCOM状态切换到CANSM_BSM_S_SILENTCOM状态时，将会产生以下的动作（Effect）。

（1）CanSM模块将会调用 BswM_CanSM_CurrentState函数（ Network :=

CanSMComMNetworkHandleRef，CurrentState :=CANSM_BSWM_SILENT_COMMUNICATION）

 

（2）CanIf_SetPduMode被调用（ ControllerId := CanSMControllerId， PduModeRequest :=CANIF_TX_OFFLINE.）。

 

（3）CanSM模块将会调用 ComM_BusSM_ModeIndication函数（Channel :=

CanSMComMNetworkHandleRef， ComMode := COMM_SILENT_COMMUNICATION）。

 

### **3****.2.16 Effect: E_BR_END_FULL_COM**

CanSM状态机从CANSM_BSM_S_CHANGE_BAUDRATE状态切换到CANSM_BSM_S_FULLCOM状态。产生的动作（Effect）和3.2.14一样。

 

### **3****.2.17 Effect: E_BR_END_SILENT_COM**

CanSM状态机从CANSM_BSM_S_CHANGE_BAUDRATE状态切换到CANSM_BSM_S_SILENTCOM状态。产生的动作（Effect）和3.2.15一样。

 

### **3****.2.18 Effect: E_SILENT_TO_FULL_COM**

CanSM状态机从CANSM_BSM_S_SILENTCOM状态切换到CANSM_BSM_S_FULLCOM状态。产生的动作（Effect）和3.2.14一样。

 

### **3****.2.19 Sub state machine CANSM_BSM_WUVALIDATION**

CANSM_BSM_WUVALIDATION状态包含：S_TRCV_NORMAL，S_TRCV_NORMAL_WAIT，S_CC_STOPPED，S_CC_STOPPED_WAIT，S_CC_STARTED，S_CC_STARTED_WAIT，WAIT_WUVALIDATION_LEAVE六个子状态。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxHibIHsfRkcqicSOeViaYRY1cfu7IZVv1B0QGCtpBNC5TxmAQTnswBXIibBKHicyJ7XHVbKbL6BHwowjw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)





默认进入S_TRCV_NORMAL状态，CanSM模块会调用 CanIf_SetTrcvMode( TransceiverMode ==  CANTRCV_TRCVMODE_NORMAL)函数设置CanTrcv收发器到 CANTRCV_TRCVMODE_NORMAL状态，如果接收到CanIf的Indication（CanTrcv转换到CANTRCV_TRCVMODE_NORMAL模式），则进入到S_CC_STOPPED状态。如果没有接收到CanIf的Indication，则进入到S_TRCV_NORMAL_WAIT状态（S_TRCV_NORMAL_WAIT状态下也可以接收到CanIf的Indication切换到S_CC_STOPPED状态）。

 

同样的逻辑：

S_CC_STOPPED 切换到 S_CC_STARTED状态的条件：CanSM模块调用CanIf_SetTrcvMode( TransceiverMode ==  CAN_CS_STOPPED)且接收到CanTcv切换到CAN_CS_STOPPED模式的Indication。同样存在TimeOut的情况。

 

S_CC_STARTED切换到WAIT_WUVALIDATION_LEAVE状态的条件：调用CanIf_SetTrcvMode( TransceiverMode ==  CAN_CS_STARTED)且接收到CanTcv切换到CAN_CS_STARTED模式的Indication。同样存在TimeOut的情况。

 

最后，如果整个子状态机从WAIT_WUVALIDATION_LEAVE退出，则进入到CANSM_BSM_S_PRE_FULLCOM状态，否则（超过重复请求最大次数，T_REPEAT_MAX）进入到CANSM_BSM_S_PRE_NOCOM状态。

 

### **3.2.20 Sub state machine: CANSM_BSM_S_PRE_NOCOM**

CANSM_BSM_S_PRE_NOCOM状态包含：CANSM_BSM_DeinitPnNotSupported，CANSM_BSM_DeinitPnSupported

CANSM_BSM_DeinitPnSupported两个子状态。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxHibIHsfRkcqicSOeViaYRY1ceVWgBZsrPdlhiaAZRPHluJTxRHOqZg9bNWgOaljtzfk71VWIZvFhNibw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)





进入CANSM_BSM_S_PRE_NOCOM状态后，根据配置参数CanTrcvPnEnabled进入不同的子状态：

CanTrcvPnEnabled == FALSE，进入CANSM_BSM_DeinitPnNotSupported子状态。

 

CanTrcvPnEnabled == TRUE，进入CANSM_BSM_DeinitPnSupported子状态。

 

CANSM_BSM_DeinitPnSupported子状态下又包含极为复杂的子状态，一般我们CanTrcvPnEnabled 配置为FALSE，这里不再深入讲解。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxHibIHsfRkcqicSOeViaYRY1cJtLaGiaR6KYGvySmRrnKVRtz3YKGtrV10s8Izje769ibOgjy3USYFc1w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)





CANSM_BSM_DeinitPnNotSupported子状态下又包含：S_CC_STOPPED，S_CC_STOPPED_WAIT，S_CC_SLEEP，S_CC_SLEEP_WAIT，S_TRCV_NORMAL，S_TRCV_NORMAL_WAIT，S_TRCV_STANDBY，S_TRCV_STANDBY_WAIT八个子状态。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxHibIHsfRkcqicSOeViaYRY1cHKiaibqhlFXhEjcKy9zPickkcbEpkenoqvGCjX1icgpK38d9aK5o7jf38A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



默认进入S_CC_STOPPED子状态，随后的逻辑和3.2.19中子状态迁移逻辑类似。

 

S_CC_STOPPED状态切换到S_CC_SLEEP状态：CanSM调用CanIf_SetControllerMode(ControllerMode ==  CAN_CS_STOPPED)且接收到Can控制器模块（Can Controller）的模式切换成功的Indication。

 

S_CC_SLEEP状态切换到S_TRCV_NORMAL状态：CanSM调用CanIf_SetControllerMode(ControllerMode ==  CAN_CS_SLEEP,)且接收到Can控制器模块（Can Controller）的模式切换成功的Indication。

 

 

S_TRCV_NORMAL状态切换到S_TRCV_STANDBY状态：CanSM调用CanIf_SetTrcvMode(TransceiverMode == CANTRCV_TRCVMODE_NORMAL)且接收到Can收发器（CanTrcv）模块的模式切换成功的Indication。

 

S_TRCV_STANDBY状态退出整个CANSM_BSM_DeinitPnNotSupported状态：CanSM调用CanIf_SetTrcvMode(TransceiverMode == CANTRCV_TRCVMODE_STANDBY)且接收到Can收发器（CanTrcv）模块的模式切换成功的Indication。

 

整个子状态切换过程中，任何一个状态切换如果没有收到对应的Indication，则会进入到超时TimeOut状态，然后重试Repeat，一旦重试次数大于T_REPEAT_MAX，则状态机退出整个CANSM_BSM_DeinitPnNotSupported状态。

 

### **3****.2.21 Sub state machine: CANSM_BSM_S_SILENTCOM_BOR**

CANSM_BSM_S_SILENTCOM_BOR状态下包括：S_RESTART_CC，CANSM_BSM_S_RESTART_CC_WAIT两个子状态。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxHibIHsfRkcqicSOeViaYRY1cIiaDc8ylcStiaC6DhBXO4Mc7E5pRX6331jznqeo2Bu6QS3w8Zjwic8Cfw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)





默认进入S_RESTART_CC子状态，在S_RESTART_CC子状态下CanSM模块调用CanIf_SetControllerMode( ControllerMode ==  CAN_CS_STARTED)函数，如果接收到Can控制器模式切换成功的Indication，则退出CANSM_BSM_S_SILENTCOM_BOR状态，否则CanIf_SetControllerMode函数调用后进入CANSM_BSM_S_RESTART_CC_WAIT子状态。

 

### **3****.2.22 Sub state machine: CANSM_BSM_S_PRE_FULLCOM**

CANSM_BSM_S_PRE_FULLCOM状态下包括：S_TRCV_NORMAL，S_TRCV_NORMAL_WAIT，S_CC_STOPPED，S_CC_STOPPED_WAIT，S_CC_STARTED，S_CC_STARTED_WAIT六个子状态。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxHibIHsfRkcqicSOeViaYRY1c7S6jsAVIib83ndPGUVcXo6UT4F91hxFDwP4dLB4xmheanDtZzhPkaYw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)





默认进入S_TRCV_NORMAL状态。

 

S_TRCV_NORMAL状态切换到S_CC_STOPPED状态：CanSM调用CanIf_SetTrcvMode(TransceiverMode == CANTRCV_TRCVMODE_NORMAL)函数，且接收到Can收发器（CanTrcv）模块的模式切换成功的Indication，否则进入到S_TRCV_NORMAL_WAIT状态。

 

S_CC_STOPPED状态切换到S_CC_STARTED状态：CanSM调用CanIf_SetControllerMode(ControllerMode==  CAN_CS_STOPPED)函数，且接收到Can控制器（Can Controller）模块的模式切换成功的Indication，否则进入到S_CC_STOPPED_WAIT状态。

 

S_CC_STARTED状态切换到退出整个CANSM_BSM_S_PRE_FULLCOM状态：CanSM调用CanIf_SetControllerMode(ControllerMode==  CAN_CS_STARTED,)函数，且接收到Can控制器（Can Controller）模块的模式切换成功的Indication，否则进入到S_CC_STARTED_WAIT状态。

 

整个子状态切换过程中，任何一个状态切换如果没有收到对应的Indication，则会进入到超时TimeOut状态，然后重试Repeat，一旦重试次数大于T_REPEAT_MAX，则状态机退出整个CANSM_BSM_S_PRE_FULLCOM状态。

 

### **3.2.23 Sub state machine CANSM_BSM_S_FULLCOM**

CANSM_BSM_S_FULLCOM状态下包括：S_BUS_OFF_CHECK，S_RESTART_CC，S_NO_BUS_OFF，CANSM_BSM_S_TX_TIMEOUT_EXCEPTION，CANSM_BSM_S_RESTART_CC_WAIT，S_TX_OFF六个子状态。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxHibIHsfRkcqicSOeViaYRY1cia0fTKyRgCEDenSfodmuOrHBibzsGZnoTZrRbOFVbV2fzCbD2GRXUCzg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



S_BUS_OFF_CHECK状态切换到S_NO_BUS_OFF状态：CANSM_BOR_TX_CONFIRMATION_POLLING功能被Enabled且 CanSM函数调用CanIf_GetTxConfirmationState函数返回 CANIF_TX_RX_NOTIFICATION。

 

S_NO_BUS_OFF状态退出CANSM_BSM_S_FULLCOM状态切换到CANSM_BSM_S_CHANGE_BAUDRATE状态：CanSM调用CanSM_SetBaudrate且没有被拒绝。如果切换成功，CanSM调用 BswM_CanSM_CurrentState(Network := CanSMComMNetworkHandleRef,CurrentState

:= CANSM_BSWM_CHANGE_BAUDRATE).

 

S_BUS_OFF_CHECK状态切换到S_RESTART_CC状态：CanSM模块的回调函数CanSM_ControllerBusOff被调用。如果状态切换成功，CanSM调用 BswM_CanSM_CurrentState( Network := CanSMComMNetworkHandleRef and CurrentState

:= CANSM_BSWM_BUS_OFF)，CanSM调用ComM_BusSM_ModeIndication( Channel := CanSMComMNetworkHandleRef,ComMode := COMM_SILENT_COMMUNICATION)。

 

S_RESTART_CC状态切换到S_TX_OFF状态：CanSM模块调用CanIf_SetControllerMode(ControllerMode == CAN_CS_STARTED)且接收到CAN控制器模式切换成功的Indication，否则进入CANSM_BSM_S_RESTART_CC_WAIT状态。

 

S_TX_OFF状态切换到S_BUS_OFF_CHECK状态：

条件：1 || 2 || 3

（1）CanSMEnableBusOffDelay == FALSE，且Bus-Off恢复从第1级(短恢复时间)切换到第2级(长恢复时间)之前的Bus-Off计数小于 CanSMBorCounterL1ToL2，且总线恢复计时满足快恢复时间 CanSMBorTimeL1。

 

（2）CanSMEnableBusOffDelay == FALSE，且Bus-Off恢复从第1级(短恢复时间)切换到第2级(长恢复时间)之前的Bus-Off计数大于 CanSMBorCounterL1ToL2，且总线恢复计时满足慢恢复时间 CanSMBorTimeL2。

 

（2）CanSMEnableBusOffDelay == TRUE,总线恢复后随意指定的时间后即可。

 

执行的动作：1 && 2 && 3 && 4

（1）CanSM调用 CanIf_SetPduMode(ControllerId := CanSMControllerId, PduModeRequest :=

CANIF_ONLINE)

 

（2）CanSM调用CanIf_SetPduMode(ControllerId := CanSMControllerId，PduModeRequest :=

CANIF_TX_OFFLINE_ACTIVE)

 

（3）CanSM调用 BswM_CanSM_CurrentState( Network := CanSMComMNetworkHandleRef ，CurrentState := CANSM_BSWM_FULL_COMMUNICATION)

 

（4）CanSM调用ComM_BusSM_ModeIndication(CanSMComMNetworkHandleRef ,ComMode :=

COMM_FULL_COMMUNICATION)

 

S_NO_BUS_OFF状态切换到CANSM_BSM_S_RESTART_CC_WAIT状态：回调函数 CanSM_TxTimeoutException被调用。

CANSM_BSM_S_RESTART_CC_WAIT状态下包括：S_CC_STOPPED，S_CC_STOPPED_WAIT，S_CC_STARTED，S_CC_STARTED_WAIT四个子状态。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxHibIHsfRkcqicSOeViaYRY1cnYpN8yLCoOf3wTozDUFBEUaIs1ZcmPaiba1cc0sWp6ORPH2lnibR7BpA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



默认进入S_CC_STOPPED子状态，和之前分析一样，调用CanIf_SetControllerMode先进入 CAN_CS_STOPPED状态后尝试恢复CAN控制器（S_CC_STARTED）。不过值得注意的事，没有超时退出CANSM_BSM_S_TX_TIMEOUT_EXCEPTION状态的机制。

 

### **3****.2.24 Sub state machine: CANSM_BSM_S_CHANGE_BAUDRATE**

CANSM_BSM_S_CHANGE_BAUDRATE状态下包括CANSM_BSM_CHANGE_BR_SYNC，S_CC_STOPPED，S_CC_STOPPED_WAIT，S_CC_STARTED，S_CC_STARTED_WAIT五个子状态。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxHibIHsfRkcqicSOeViaYRY1c2UfJRTYdCju2aWNaqx9gpUhZBps1QGxPlxpNkPFNXEiadVZIqMqgnug/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



实际项目中一般不会在运行时动态改变波特率，这里不再详细讲解。

 

# 4.**关键API**

## 4.1 **Function definitions**

**CanSM_RequestComMode**

Std_ReturnType CanSM_RequestComMode(NetworkHandleType network,ComM_ModeType ComM_Mode)

改变network参数标识的网络的通信模式为Mode。

 

**CanSM_GetCurrentComMode**

Std_ReturnType CanSM_GetCurrentComMode(NetworkHandleType network,ComM_ModeType* ComM_ModePtr)

通过指针参数ModePtr获取network标识的网络的通信模式。

**CanSM_StartWakeupSource**

Std_ReturnType CanSM_StartWakeupSource(NetworkHandleType network)

EcuM模块调用该函数开始CAN唤醒源检测。

 

**CanSM_StopWakeupSource**

Std_ReturnType CanSM_StopWakeupSource(NetworkHandleType network)

EcuM模块调用该函数停止CAN唤醒源检测。

 

## 4.2 **Call-back notifications**

CanSM_ControllerBusOff

void CanSM_ControllerBusOff(uint8 ControllerId)

通知到CanSM模块ControllerId标识的网络发生Bus-Off事件。

 

**CanSM_ControllerModeIndication**

void CanSM_ControllerModeIndication(uint8 ControllerId,Can_ControllerStateType ControllerMode)

通知到CanSM模块，ControllerId标识的网络发生了模式切换到了ControllerMode事件。

 

**CanSM_TransceiverModeIndication**

void CanSM_TransceiverModeIndication(uint8 TransceiverId,CanTrcv_TrcvModeType TransceiverMode)

通知到CanSM模块，TransceiverId标识的收发器发生了模式切换到了TransceiverMode事件。

 

## **4.3 Scheduled functions**

CanSM_MainFunction

void CanSM_MainFunction(void)



参考文档：Specification of CAN State Manager 4.3.1

https://www.autosar.org/nc/document-search/