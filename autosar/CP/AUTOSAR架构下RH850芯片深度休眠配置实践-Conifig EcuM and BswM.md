# AUTOSAR架构下RH850芯片深度休眠配置实践-Conifig EcuM and BswM

**前言**

AUTOSAR EcuM模块的分享分为EcuM模块概念详解和EcuM模块配置及代码分析，具体的项目实战请关注本号的后续文章，***\**\*\*\*本篇为EcuM模块配置实践篇。\*\*\*\*\****

**[AUTOSAR 模式管理-EcuM模块功能概述](http://mp.weixin.qq.com/s?__biz=Mzg2NTYxOTcxMw==&mid=2247484853&idx=1&sn=63d030d452ee7dab01a3d29f738ed6ac&chksm=ce561fdbf92196cdf2c82d67dbdc6c7cd053425de0d717261ae8d83393ffe78d69767715f61b&scene=21#wechat_redirect)****
**

**[AUTOSAR模式管理-EcuM Startup and Shutdown详解](http://mp.weixin.qq.com/s?__biz=Mzg2NTYxOTcxMw==&mid=2247484920&idx=1&sn=a220b4f0449f0c823ccc1e3a6c70fd31&chksm=ce561f96f92196800f8b503d4db5dbc47d78d3abf2b5d1ca76c7d3e6351a200c38cd6790eb7c&scene=21#wechat_redirect)
**

**[AUTOSAR模式管理-EcuM Sleep and UP详解](http://mp.weixin.qq.com/s?__biz=Mzg2NTYxOTcxMw==&mid=2247484936&idx=1&sn=e0fc51b2e59772458d5d98fd32958bf8&chksm=ce561c66f92195706b131e06aa8fa2378ba4f221c9ec62e51bb3d568e83324406264dcaafee2&scene=21#wechat_redirect)
**

**[AUTOSAR BswM(1) 模块详解](http://mp.weixin.qq.com/s?__biz=Mzg2NTYxOTcxMw==&mid=2247484278&idx=1&sn=e7d9c72d2978c0ed6663378ef9648f2f&chksm=ce561918f921900e9286ddcc04fff9f4808eaec888acae9d076cbce6c2070900e8aafdee2a04&scene=21#wechat_redirect)
**

**[AUTOSAR BswM（2）配置实践](http://mp.weixin.qq.com/s?__biz=Mzg2NTYxOTcxMw==&mid=2247484328&idx=1&sn=cf07d2d0915f091e47b887f327d45bcf&chksm=ce5619c6f92190d0aaa08049f528243da209c84489e914a55b8dad4e8504b53725e0453fd9d8&scene=21#wechat_redirect)*****\**\*
\*\**\***



![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdfZjJgDtmbNtcCBGJPcOMLQLX3fbyV3RlBV6m2PbLhc3e3BspricosNg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**正文**

项目中的ECU硬件设计为休眠系统（无SBC，满足休眠条件后系统进入深度休眠状态），整个项目的模式管理及ECU休眠唤醒功能涉及到EcuM，BswM，SWC三方交互，设计及实现中遇到了不少问题，最终都被一一解决。本文将详细介绍模式管理实现过程中的关键步骤。

 

**MCU：Renesas RH850 F1KM**

**MCAL：DavinceCFG**

**BSW：ISOLAR-9.1**

 

在开始系统配置之前先回答以下问题：

**1）.为什么在硬件上要设计为深度休眠系统？**

--减少SBC（System base chip）降低成本，热启动提高启动速度。

 

**2）.基于RH850芯片的ECU设计为深度休眠系统后，怎么让系统进入深度休眠状态？**

--BswM中维持一个EcuM状态的状态机，当状态机跳转到SLEEP状态后执EcuM_GoHalt和EcuM_SelectShutdownTarget(EcuMSelectShutdownTarget_SLEEP_MCU,SLEEP_MCU)动作后EcuM接管程序，EcuM调用Mcu模块的Mcu_SetMode(Deep_Sleep)进入深度休眠状态。

 

**3）.基于RH850芯片的ECU设计为深度休眠系统后，AUTOSAR架构下在哪里设置系统进入深度休眠？**

--EcuM没有提供Callout函数让系统设计去设置Mcu进入深度休眠，EcuM在检测到系统满足休眠条件后在Sleep Sequence中直接调用Mcu_SetMode(Deep_Sleep)进入深度休眠。

 

**4）.基于RH850芯片的ECU设计为深度休眠系统后，AUTOSAR架构下在哪里及怎么开启唤醒中断？**

--EcuM提供EcuM_EnableWakeupSources Callout中调用RH850提供的Mcu_WakeUpFactor_Preparation函数设置唤醒中断。

 

**5）.基于RH850芯片的ECU进入深度休眠状态后被唤醒是直接复位还是接着往下跑？**

-- 芯片特性，直接复位。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdMNg671mWV3RibicSL02oIbmoY7xgvl98l3tv7HjoVWQwC9k5YpSicCT6g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdhnrYzf34Ym9IFEGWe7WibfrAaQ6mWJ3Ej7aMUyTf9cEuX7cC9ibbcheA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4Md8r92cWRFCI9dEicIKbwcofpw6fI3zc5knKW59GQFseib6UGQnmtahMpA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdoKPibgw65hXpeDf5t6tGm92vmeC8WRTsXKDm0om2kXzPibfFRZZNCibZQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

**6）.基于RH850芯片的ECU进入深度休眠状态后被唤醒如果是复位，AUTOSAR架构下在哪里调用Mcu_PerforReset？**

-- 不用调用Mcu_PerforReset，RH850-F1KM芯片自动从0地址开始运行（相当于复位）。

 

**7）.基于RH850芯片的ECU进入深度休眠状态后被唤醒如果是接着跑，AUTOSAR架构下在哪里关闭唤醒中断？**

-- RH850芯片进入深度休眠后被唤醒会自动复位，所以不用进行关闭唤醒中断的操作。但是，如果是NXP S32K系列的芯片在进入深度休眠后，被唤醒后是接着往下跑的，这个时候在AUTOSAR架构下改怎么去关闭唤醒中断了（目前作者也还没有答案），这个问题留个大家思考。

# **1.****状态机设计**

本系统配置为Flexible EcuM系统，EcuM存在STARTUPONE,STARTUPTWO,RUN,RESHUTDOWN,SLEEP五种状态。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdC56LG4cdYu8e7khP1MSIXibfzn0MKavxELxhBZdztsVP48eRrmeCuuA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图1-EcuM在BswM中的状态机-BswMMode

 

ASW请求接口SleepReq == TRUE（同时满足其他，NvMWriteAll成功 && IGNOff等）后BswMMode状态机切换到SLEEP状态后调用EcuMSelectShutdownTarget_SLEEP_MCU和EcuMGoHalt使得EcuM接管程序走SLEEP Sequence。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdiciaoiblpjialXL7IPCHNH07Oqkgu5paN6MptPfSJbe7TIgC38ulicKrKgw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图2-Sleep阶段

 

# **2.****Davince配置MCAL**

DavinceCFG根据硬件设计配置唤醒脚和唤醒中断，需要配置MCAL架构中的Port和MCU模块。Port模块中配置唤醒脚的Pin属性，MCU模块中配置深度休眠模式及其对应的唤醒中断源。

## 2.1 配置port模块

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdRmcibPKvINetYQh2IHRdBoibtIhmOs8iciaZ4MdaDBwFib2U2qwEWey5CYQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

查看软硬件接口表找到硬件唤醒脚。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdQ9JzgxgfMkMbbAknbUqOxLIx9qKeIHarXCcTzwib5eKgicZhv7tCYuhw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Davince中配置Port模块的唤醒Pin的属性。

 

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdenNOj7ICJiaWVG7Yd34W9bGSiclRh0tBicpUhFXAq2aiccoz2icXibXrMsoA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdTEJPyA8JhudvPQibo75abSrwIl4lB3SkVhnvsOe2TibicThfyOJulUJ5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

配置中断唤醒Pin的电平触发模式。

 

## **2.2 配置MCU模块**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdpvDTZWPFy6zLnXY5d78kgI1CEcYwX6kTlS0rTqR8AoEPlFtq0qqceg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

添加一个Mcu深度休眠模式。

 

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdykbveCvl6zEC5zyl7nDa4dvsOKtv9ux57YZpoEeGJNtcNyuHeynJTw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

添加MCU唤醒Pin。

# **3.****ISOLAR配置BSW**

ECU模式管理中主要配置的BswM模块和EcuM模块。EcuM主要配置系统休眠模式及唤醒源信息，BswM模块配置模式管理接口（MRP）、模式仲裁及模式控制。

## **3.1** **添加MCU模块**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4Mdnn3lVQ3WDKByS7SUnBEefibO3ajNmgfDDzicIrmYDNlyiaHSk8TSOnbCQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

EcuM模块配置Sleep Mode的时候需要引用到Mcu模块的的信息，所以需要在ISOLAR工程种配置一个Mcu的模块。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdfBcCZVCk16DN4yLyo0wbOmVzxFT7uFFnOmoIp4a1hHPtycz2sZzKaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

ISOLAR工程目录xxx\ISOLAR9_CfgPrj\Config\ecu_config\paramdefs（这个是作者的工程目录）

添加模块UI信息。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdyqYqXQ2L3y2I6WELQpc85czOSk2EqxzqRKsXC1gshdtITueBUWjicaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

新建一个Mcu模块。

 

## **3.2** **配置OS模块**

虽然F1KM是单核系统，但是在配置EcuM模块的时候还是需要在OS配置一个RES_AUTOSAR_ECUM的资源锁，不然在生成BSW的过程中ISOSALR会报错。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdXS7rS8frVxo80jNGuoHaybhexicTRIkbXI81vwOpTQGnlE8WPwysLPw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdTlibWrnC9ibqvlx6SIZXYp1UlAPtbR1c1rgaV8FVbPFuFJiaQawx2G55w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

## **3.3** **配置EcuM模块**

### **3.3.1 EcuM引用RES_AUTOSAR_ECUM**

 

### **3.3.2 配置EcuMSleepModes**

设计为休眠系统的EcuM模块必须配置EcuMSleepModes。EcuM_Cfg.h文件里面的ECUM_SLEEP_SUPPORT_ENABLE宏需要配置了EcuMSleepModes后才会配置为TURE。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdjYYv38PictxZpHAo6SS8VKXNQWxXbBdz91yaGkK1k5icg7S5hUQF2icdw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



### **3.3.3 配置唤醒源**

以下五个唤醒源是系统默认配置的：

EcuMConf_EcuMWakeupSource_ECUM_WKSOURCE_INTERNAL_RESET EcuMConf_EcuMWakeupSource_ECUM_WKSOURCE_EXTERNAL_WDG   EcuMConf_EcuMWakeupSource_ECUM_WKSOURCE_INTERNAL_WDG        EcuMConf_EcuMWakeupSource_ECUM_WKSOURCE_POWER EcuMConf_EcuMWakeupSource_ECUM_WKSOURCE_RESE

其他唤醒源需要根据实际系统设计配置。本项目有一个本地Local唤醒源（IGN）及一个CAN唤醒源。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdIePboiagEf2pSCxtgQiaBUj9H5qYjIejoglSOp1fuWh1WBxn2Eicbz0LA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

### **3.3.4 引用NormalMcuMode**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdCrLzo1wgVD0nGic9LzZohd1pf1ibHOzZBlO8fqvHlYvl9HqrBeg37qVw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## **3.4** **配置BswM模块**

BswM模块的配置最主要的就是配置：一个和SWC交互的MRP（SleepReq，应用层满足休眠条件后请求休眠的模式管理请求接口）；EcuMGoHalt的Action（BswMode进入SLEEP状态后调用）；EcuMSelectShutdownTarget_SLEEP_MCU的Action（BswMode进入SLEEP状态后调用）。

### **3.4.1** **添加Sleep相关的Action**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdhVP55BJomb7hlYKpLCWNLAF7V2QDicicEW3CIt3ic1ahj3R861xp5M01g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4Md2B9PgAHUQrFeVx2LG1ENglTdGmmSc5OGBjryvkuZUy0E07fviasmXOg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### **3.4.2** **添加ActionList**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdR3Ajk6vFpGzicH32fFtAXfCa6p3cn1dGic0JnCZsASUicsQjyThia8ttJQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



### **3.4.3** **PreShutdownToShutdown添加Sleep的ActionList**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4Md1mhhrkial8C5eYaFH8fYbtFVFcbe0jOnq11l0txZLPqyC6nP4HxTICw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



# **4.****Callout函数实现**

EcuM模块提供的Callout函数EcuM_EnableWakeupSource中调用MCAL提供的标准接口（不是AUTOSAR标准接口）Mcu_WakeUpFactor_Preparation函数设置唤醒中断。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdL3U0lsYZtxL0icn7Z6yIuicICFDaC1gz3SmFpGfcTibdmz4LEs7Cpju5g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

# **5.****ASW配置**

设计一个CDD_WKSM SWC模块来专门管理应用层的休眠唤醒功能。CDD_WKSM模块设计一个Pport设置当前应用层是否满足休眠条件（SleepReq == TRUE）。

## **5.1** **添加应用请求休眠接口**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdQRknZvoK26b1MEFICmicnfoKWZQZLWxwplibUJQWvU4FfM6f8JZHEO1w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## **5.2 SWC****中清除预留唤醒源**

系统每次被唤醒后如果没有检测到唤醒源，会默认设置一个Reset的唤醒源。因为系统休眠后立即被复位了，复位起来后最好将默认的唤醒源都清除掉。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4Md07dxJYKvlXQX1oALU99mx7lzKkicgv18jicF9h7PxtNlQHKdHjfQkThQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## **5.3** **设置唤醒事件**

在休眠管理模块中检测和设置唤醒实践。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczNfp5QDuYtXiaEJOTPGx4MdNxlCibjKeDBX6DYaug9z6QThP3OW7zHNUWUyFu8I1J2E5aHEiazAKWAQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

# **6.****实际效果**

满足IGN == IgOff & SleepReq == TRUE后系统进入了深度休眠状态。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RczvsGGnibkYCjibKdXtD9cT9p0YNUIQCT7TMYtJYQEBMcI3eMgVzAcNBBVXHFcCnZF2TNicBJfL7d3Xw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



# **7.总结**

AUTOSAR架构下的模式管理是SWC - BSW - ECUM三方交互的一个过程，SWC请求，BSW仲裁后调用ECUM的请求接口，ECUM完成最后的执行。