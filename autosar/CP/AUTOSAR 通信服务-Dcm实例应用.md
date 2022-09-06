# AUTOSAR 通信服务-Dcm实例应用

**正文**

## **7.1** **DCM诊断处理流程示例**

为了获得安全访问，DSD子模块必须调用DSP子模块来从应用程序获得种子值。如果没有检测到错误，则以积极响应的方式发送种子值。

 

在第二步中，DSP子模块得到测试器计算的密钥，并请求应用程序将该密钥与内部计算的密钥进行比较。如果没有发生错误，则在DSL子模块中设置新的访问类型，并发送一个正响应。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVlbtQDbPiaibpG794YZp8n4Q6ibE3BR4LzYqg45HrsRgE2W6DNyHtDRaCjA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

从上诉过程可以看出，我们在实际开发过程中需要做的是根据实际需求配置DCM模块（DSL, DSD, DSP），同时还需要设计一个Dcm_User模块来真正处理（processor）诊断请求。

 

下面我们就以最常用的诊断服务--**读取软件版本号**为例来说过整个开发流程。

 

## **7.2 开发案例分析**

### **7.2.1 需求**

服务：0x22

DID：0xF189读取ECU软件版本号

数据长度：17个字节

DID服务只有读的功能

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVledAwFTFiaxGb4fa5KHusVGBSGYxmCz6ynpVXM13wricNWDH6gib8YWiarA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

也就是上位机机发送：22 F1 89（省略诊断ID和长度信息）

ECU需要回：52 F1 89 +17字节的软件版本号（14字节的零件号 + 3字节的软件版本号）

 

### **7.2.2** **模块接口设计**

**Dcm模块**：AUTOSAR静态模块，需要配置支持0x22服务下的0xF189服务

**Dcm_User模块**：所有Dcm服务的一个中转模块，Dcm模块通过Dcm_User模块提供的服务来完成诊断响应，Dcm_User模块和其他各个应用模块连接，使用各个APP模块提供的服务。

**Vers模块**：提供读取软件版本号的服务，Vers里面需要实现真正的软件版本记录。

**Interfaces和R/P-Port设计**：如下图所示。



![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVladOnTHibOVHNK1nz1b4WoYmGepXV1icAiaFFCgwwPCbceTS3CSItsHZnw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

### **7.2.3配置DCM模块**

配置0x22服务

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVlVC04bWbf2P8JAia57vmSRkC3qW9HYoqPVhibpThFAkTzqYgvcIOiautWw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

配置0xF189服务的数据信息

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVlKeO2x0wQfgTU9Uiaqw2ILkktEJicia6aQ0ibfP9U2m4HrG3QsiboXwzPOIQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

配置DidInfo，配置完之后，有这种读写属性的DID都可以引用这个DidInfo

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVlVTaykYSpv2D00XeJ7ZobkxBw5iaW8pq98MLVGLE2WIQhnQibCFhsfV1g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

配F189这个DID的信息

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVlWZl0vK8UrdJ2ibYLtxbkibefz4VV1swzGYPASzyKficUUhmrzDfAp8SmQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### **7.2.4配置Interface**

配置Dcm和Dcm_User之间数据访问的Interface

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVlIl4pgBiaUlrTZ0VQHvX0wjYqweDkibhnas7cqrBN87wQu3BDkLicpKLxA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

配置Dcm_User和Ver模块之间的Interface

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVl58eHVjtb7pZRKdJKrtmsEWeLwQSehJmJm1WP8B0qcLicXjh7JMuqkUQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

### **7.2.5配置Dcm模块的Port信息**

Dcm模块相对Dcm_User模块是Client端，需要使用Dcm_User提供的服务（Server端），所以需要给Dcm配置数据访问的R-Port。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVlricCuofibhribeMaGicQFEVye6diaicMB3lxYMTZlVyjPlk9icEUEdq2IHWLQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### **7.2.6配置Dcm_User模块的Port信息**

Dcm_User模块相对Dcm模块是Server端，需要提供服务（Server端），所以需要给Dcm_User配置数据访问的P-Port。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVlwGAXcribCH9WL7QD1P9dpbhxOFhibkT457ic7nTd7G0rmBU2bPwZ7EH8w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Dcm_User模块相对Vers模块是Client端，需要使用Ver模块提供服务（Server端），所以需要给Dcm_User配置数据访问的R-Port。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVlSCoNLUP2oxxNviaCDP6omZ1iaYbuicFTLJ3JqwBIBQAKMfNrleZj8Uz5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) 

### **7.2.7配置Vers模块的Port信息**

Vers模块相对Dcm_User模块是Server端，需要提供服务（Server端），所以需要给Dcm_User配置数据访问的P-Port。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVlbPgIfEnwWgJWZziaXDVluhJFfWQvONQRz0GHh5Rk03ssbQ44QBkZJnQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



### **7.2.8生成的代码分析**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVlXYAjXDUmq3GesYptnrn85qMDwArDTcxx4TrKoBJOV4uQfib7Ba6MTVQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVl2dDzXtyIxwxaBgr4LE5dGziaibXuJfn82j9B5VAGVwpicNSQMxXT9MW9g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVlUC3Ew0rz4H442edn6eEmARx1Qbxoyvv0cXWhxcibiazc8MYrJM6GUWIQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxOZM64icq4bytGDE7Y8olVlBzPUqlRkyaVDOHthbeDajUp2BB24suiasNszQXmZkV6fMtPVkeCUXnA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### **7.2.9 小结**

实际应用中，对DCM模块内部的机制不会太关注，一般是根据实际需求配置DCM模块，然后设计DCM_User等实现模块。而DCM_User等实现模块的设计包括端口Port设计，端口接口Interface的设计，已经具体功能的算法逻辑实现。