# 基于RH850芯片的MCAL详解--MCAL简介及PWM模块详解

**前言**

MCAL处于AUTOSAR架构的最底层，和具体的芯片强绑定，且不同的芯片使用不同的MCAL配置工具，例如英飞凌系列使用EB配置MCAL，瑞萨系列使用Davince配置MCAL。所以，除了AUTOSAR标准定义好的配置项及标准接口外，不同的厂商的MCAL还会独立于MCAL标准之外的配置，所以MCAL的学习最好是结合具体的工具和芯片来学习。本系列MCAL分享，将基于瑞莎RH850芯片使用Davince配置工具来讲解，**本文为MCAL简介及PWM模块详解篇**。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJocscaFWZ69SEtL0DQFeqOQrxVaAc8SDcNGqIIUGcRHaQf3jhTDQ1bQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**正文**

# **1.****MCAL简介**

微控制器抽象层（Microcontroller Abstraction Layer，MCAL）位于 AUTOSAR软件架构的最底层，与微控制器的内部单元及其外设相关， 接收上层指令，完成对硬件的直接操作；并获取硬件相关状态，反馈给上层，对上层屏蔽了硬件相关特征，只提供对应的操作接口。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJjuEhIGAzibGib29QauibU4su9HqAMxxQxC4spyh5OpHOF1d7zn2Nrtc0Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**简单理解：MCAL的作用就是隔离硬件，提供操作硬件的AUTOSAR标准接口。**

 

 

在AUTOSAR方法论中，AUTOSAR微控制器抽象层配置与实现属 于ECU级开发范畴，由于MCAL模块较多，并且需要使用MCAL配置工具完成配置，这里专门介绍AUTOSAR微控制器抽象层配置与实现方法。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJWP7NvibN6L2gwGTAGcNtucEsfhvN3hVPiaDCDyDJR8oztRowCgaoJ0Mw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJMXOsYsECWGVBSU7Z7w0zZocAGa9CPRia4Lt58CVW1UjvgqsP4J2zXcA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



MCAL驱动主要分为IO类驱动（DIO等）、通讯类驱动（CAN等）、内存类驱动（EEP等）、微控制器驱动（GPT等），根据使用个人使用经验，MCU模块（需要结合具体芯片的硬件时钟配置软件模块使用的时钟）和SPI模块（需要结合外围芯片的SPI通信振格式及时序要求来配置）的配置是难点，后面将详细讲解。

 

ADC驱动：Analog-to- Digital Converter Driver

CAN驱动：Controller Area Network Driver

DIO驱动：Digital Input/Output Driver

ETH驱动：Ethernet Driver

GPT驱动：General Purpose Timer Driver

ICU驱动：Input Capture Unit Driver

LIN驱动：Local Interconnect Network Driver

**MCU**驱动：Microcontroller Unit Driver

PORT驱动：Port Driver

PWM驱动：Pulse Width Modulation Driver

**SPI**驱动：Serial Peripheral Interface

 

# **2. PWM模块详解**

参考文档：

1）瑞莎芯片手册：RH850/U2A-EVA Group User’s Manual: Hardware

2）AUTOSAR_SWS_PWMDriver -- Specification of PWM Driver

 

## **2.1 PWM模块概念介绍**

PWM驱动提供初始化和控制MCU内部PWM状态（脉冲宽度调制）的功能。PWM驱动模块可以产生可变的脉冲宽度。它允许选择占空比和信号周期时间。

 

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJAJSjLqJbp3uibFD6gQOYxoOV3JgJlYdKL4YOEJpnsJh3Txh2UV582Uw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

图1：PWM信号描述

 

Polarity：极性，也就是PWM脉冲开始的电平状态，如果我们的空闲状态配置（Idle State）为低电平，则极性Polarity就为高电平。

Idle State：空闲状态，也就是没有PWM脉冲输出时的状态。

Duty Cycle：占空比周期，也就是一个PWM处于Polarity状态的时间。

Period：周期，Duty Cycle 加一个空闲状态的时间。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJ4ylE0RDJarRGpttcrZWTDnLfePKjTD9RjcfPx8778uXz1ia7VrhiaouA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

表1：PWM关键概念

 

## **2.2 PWM依赖的模块**

PWM依赖于于系统时钟。因此，系统时钟的改变（PLL on --> PLL off）会影响PWM硬件的设置。

 

PWM驱动依赖以下的模块：

-- PORT Driver: 设置Port pin的功能（port pin一般都是复用的）

-- MCU Driver: 设置预分频器，系统时钟和锁相环

-- DET: 默认的Error追踪

 

***Note: 配置PWM模块前要保证MCU模式时钟及PORT模块的Pin脚属性配置正确，详见Mcu和Port配置介绍。\***

 

## **2.3 PWM频率**

PWM模块的所有API服务都是使用Ticks数作为时间单元。Ticks使用的基准时钟由Mcu模块的配置（如PCLK），PWM模块可以对基准时钟进行再分频配置得到想到的Tick时钟。

 

Ticks数和时间的转换必须在应用层完成。

 

例如：如果项目使用的PWM通道的Tick需求都是1us，我们要得到1000Hz的频率，那么需要设置Period参数为1000（1/（1000 * 10^-6））。

（void PWM_SetPeriodAndDuty(

 PWM_ChannelType ChannelNumber,

 PWM_PeriodType Period,

 uint16 DutyCycle

)）

 

## **2.4 PWM占空比**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJzzGm5ric85G6A9m4cMsBLAg8iaZMphpLh70L4q8owzJibrVEYhNaPMgSw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJyiaKDFsFuoJJ1ETwSf23ZwxxDiaGNOVrnrzCgSUQqGKgQNDEWIGQwjbA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

0x00--0x8000表示占空比0%--100%，所以我们要配置占空比50%，则DutyCycle参数应配置为0x4000。

 

## **2.4 PWM重要数据类型**

### **2.4.1 Pwm_EdgeNotificationType**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJ8ExicB1DpYK8Kr6VMKZf06KIe3icibhN7zgnicKxQ9nHJrRRRFI47ia9X2w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJ1sgibLth5bVA84ezaSAS9ibsBsiciaJ8CNn23YFIo1GwQINfDwPtkz9Ilg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

PWM_EdgeNotificationType配置是上升沿、下降沿还是双边沿调用通知函数（Notification，需要配置）。

### **2.4.2 Pwm_ChannelClassType**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJTXXicTh2zyjNMews2SCxOjv7rFoWibbICMN7MAE88ulPgIZ4Yto8SblQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



PWM_ChannelClassType配置PWM输出的模式：

PWM_VARIABLE_PERIOD：周期（频率）和占空比都可改变

PWM_FIXED_PERIOD：固定周期（频率），占空比可改变

PWM_FIXED_PERIOD_SHIFTED：固定周期和占空比。

 

必要和可选的配置项：

PWM对应的硬件通道、默认周期、默认Duty cycle、极性、空闲状态、通道类型是必须的配置项。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJ0SORKK30Sicvhr7H7yqSuLQMqe04EGQIUoKFXnfC0py7BbEUOuVKiaNA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## **2.5 PWM重要API**

### **2.5.1 Pwm_Init**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJibouTFkep4yNntvJspy9fI9vxblC1OhibegLj9MVtEAy6zBjVliadZTCg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



Pwm_Init函数初始化PWM模块，将所有PWM通道设置为默认状态。

 

### **2.5.2 Pwm_SetPeriodAndDuty**

Pwm_SetPeriodAndDuty函数设置PWM通道的周期和占空比。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJAyPnNydbsVo6Mb9K9QdBEOBicDGIR6rxekAEbBveubIJAiaCIYGTicEzQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJL0BEYkgLCPBKhNxZHoLOlOp33O0atzTtaQt5nwsbEfKfQKgsKPGxBw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 

周期的参数也是Tick数，看配置给这一路PWM输出的时钟Revolution，如果revolution == 1us，则Period == Tick number * 1us。

如果将Period设置为0，则占空比的设置无效。在这种情况下输出的占空比为0。

 

### **2.5.3 Pwm_EnableNotification**

Pwm_EnableNotification使能PWM的通知功能。如果我们配置PWM通道的Notification函数，同时在使用PWM功能的模块调用了Pwm_EnableNotification使能PWM的通知功能，则PWM的配置边沿（上升沿、下降沿、双边沿）产生中断调用我们配置的Notification函数，一般在需要知道边沿变化的场景下使用。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJEgdqr67HKiaDOtQibbccaAiaLeicXg7yH4vNG4nlGyjbzZAtYohWW3iaHNw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## **2.6 PWM配置实例**

需求：需要配置基于RH850-U2A芯片的ECU的M_PO_PwmTest脚位PWM输出口，且频率和占空比可调，频率输出大概位1000Hz。

### **2.6.1 配置Port口**

芯片的Port口一般是复用口，需要配置Port口的工具为PWM输出脚。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJ02vy2horAibEd5Z0oGB6vwFsKNTBSQhM9FqI1a4La5licu9s12PBcgiaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



### **2.6.3 PWM通用项配置**

配置PWM模块的通用（General）项，包括芯片类型，使用二类中断等。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJcyDY2mPasribcJHH9t6kU1CAiciatzibwBbiaaECl5hfr4JmyeIgLzorfsw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



### **2.6.4 配置可用服务**

配置PWM模块需要使用的那些API服务，我们主要使用PWM的Set Period and Duty的功能。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJdKToQeAc4wLnDCJNgsqaOgSKFCnCgNWOfO7JPibTvSTuoSvFx4oGxQg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



### **2.****6.5 配置PWM输出需要的时钟**

PWM模块使用的时钟源是80MHz，我们想得到1us的Tick基准，使用PCLK分频后的Ck3作为PWM通道的时钟。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJFWH61h9Rrp5S75OvibyLJSevicqc5gtrzGe3k8uBlAQtdGBwmOib5dT4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



### **2.6.3 配置PWM通道**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJcOTyqujBL7Eibo3eNat2MAm0zY7U0WGic7442Sj19ibQiaYwicvk82BblwQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## **2.7 总结**

配置好PWM模块后，PWM功能的使用模块就可以分装PWM提供的标准接口来实现特定的需求。一般我们会在IoHwAb模块分装PWM模块的使用细节，让ASW直接使用IoHwAb模块分装后的接口。具体可以留言本公众号后续对于IoHwAb模块的分享。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcygLh48UFmu7KKJetEPeZDJt2ibodm4wls8AjIOvcRtSkc7jZ82tQPCQkuao9DDMDfONvI0b30rXUA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)