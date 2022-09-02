

# AUTOSAR初学者最想搞懂的东西

**1. 什么是Tire1、Tire2、OEM、ECU**



这里提几个概念，什么是**Tire1、Tire2、OEM**？虽然跟AUTOSAR关系不是很大，但常常遇到，了解下比较好。

没在车载行业混过或者刚入职车载行业的小伙伴可能不知道。

**Tire1**，即Tire One，意为车厂一级供应商，给设备厂商供货，也就是车厂零部件的供应商。

那么**Tire2**呢，就是二级供应商，可以理解为Tire1的供应商，例如Tire1在搞仪表产品，就需要向Tire2购买零部件，如电机、指针等等。

而**OEM**，是Original Equipment Manufacturer的缩写，通常指设备厂商/主机厂/整车厂，例如宝马、丰田、大众、广汽、BYD等等。

**ECU**就是Electronic Control Unit，就是你开发的那个项目器件，例如空调控制器算是一个ECU、娱乐系统主机也是一个ECU。



**2. 什么是SIP**



SIP或者叫SIP包，即**S**oftware **I**ntegration **P**ackage，是Tier1在做AUTOSAR项目前，向Vector购买集成了AUTOSAR方案的软件包，Vector最终交付给Tire1时的软件包就是SIP包 。

那么Tire1开发者，就基于这个SIP包来做项目上的应用开发。

![1](https://user-images.githubusercontent.com/80186561/188119770-c21c818b-0205-4a81-a41d-a2bee5fb5b98.png)

除了SIP这个名称，你可能还会遇到SLP、HLP等概念，即

Software License Package (SLP)
Hardware License Package (HLP)

而SIP又有分几种类型，如：Beta SIP 、Production SIP 、QM Approval SIP、Update SIP 、Prototype SIP 和Mini SIP 等。

是不是开始蒙圈了，好了，先不要管这些，记住SIP包这个概念即可，其他的你慢慢就会懂的了。



**3. SIP里有什么**



SIP里有什么？直接打开SIP包看不就知道了，这个问题是不是有点多余？也并不是，如果刚接触这个东西的小伙伴，可能搞不清里面有什么，因为里面的文件太多了，压缩包都有好几百MB。

直接截个图来看看，你知道这里面这些是啥么？


![2](https://user-images.githubusercontent.com/80186561/188119834-388d9103-e714-4da8-b95c-feb81bb34647.png)


实际上，对初学者来说，不知道也影响不大，如果你好奇，那就参考下我的理解：

| **内容 **           | **解释 **                                                    |
| ------------------- | ------------------------------------------------------------ |
| Applications        | 是Vector对这个软件包，做了一个应用工程，可以理解为一个Demo，你可以根据这个案例来建你的工程。 |
| BSW                 | 一些BSW层的源码，在通过Configurator添加模块生成代码得的时候，工具会将这些代码拷贝到你的工程。 |
| BSWMD               | 这个文件夹里面存放这生成BSW配置的一些策略和关联关系，都是些arxml文件来的，和Configurator息息相关。 |
| DaVinciConfigurator | 就是Vector的第二个工具了，另外一个是Developer，这个Configurator是一个运行软件，和SIP集成在一起，有可能是因为版本和License问题才这么绑定的。 |
| Doc                 | 就是这个SIP包的一些参考文档，很有用。                        |
| Generators          | 就是一些组件的配置生成器，相当于Configurator的插件，通常是写exe等文件。 |
| Misc                | 一些不好分类的杂项。                                         |
| ThirdParty          | 就是Vector以外的第三方的内容，一般是MCAL                     |



**4. DaVinci Developer
**

Developer是干什么的呢？简而言之，就是配置SWC（Software Component）即Application Layer上的东西用的？


![3](https://user-images.githubusercontent.com/80186561/188119910-ee67f8b6-b308-4d2b-b25d-381a4667913d.png)


是不是有点懵逼，Application要配置啥子？

再给你个图看看：


![4](https://user-images.githubusercontent.com/80186561/188119956-2ce9f8c7-c675-4330-b53a-289e4c3491c3.png)


上图的这些Applications之间的接口是需要配置的，因为接口有一套特殊的约定。


![5](https://user-images.githubusercontent.com/80186561/188120030-6a558641-0d40-4483-aeb7-c7e2e0bb84a8.png)


目前，先了解下这些概念，后续慢慢深入比较好，我之前也有类似的文章讲解这些东西的概念和具体实操演练，里面涉及到SWC、Port和Runnable等概念。放个传送门：

- [AUTOSAR SWC详解](http://mp.weixin.qq.com/s?__biz=MzI0MTg5MDY3NA==&mid=2247485793&idx=1&sn=26eb6993383121af6967241400dc01a9&chksm=e905ed7cde72646a53724f606ad798a21064ff85c6a9875a6f74be0f026470bea5d9043d38d1&scene=21#wechat_redirect)
- [AUTOSAR Port原理概念详解](http://mp.weixin.qq.com/s?__biz=MzI0MTg5MDY3NA==&mid=2247485904&idx=1&sn=77efca0f78deb1429ca8ff7ed51796fc&chksm=e905edcdde7264db6e92ffc6f7bdacf02b238d51f1eb1a033dcc3346743cbe12584760f4d93e&scene=21#wechat_redirect)
- [AUTOSAR Port配置教程](http://mp.weixin.qq.com/s?__biz=MzI0MTg5MDY3NA==&mid=2247485905&idx=1&sn=e891c71d46841c894bd2ba60bf28f818&chksm=e905edccde7264daaf735c77cf74975868a3ec7c24ac65332542d0323589032ec064a4cddf8b&scene=21#wechat_redirect)
- [AUTOSAR Runnable详解和配置步骤](http://mp.weixin.qq.com/s?__biz=MzI0MTg5MDY3NA==&mid=2247485940&idx=1&sn=70308b7f2a9cc201f34be58552d20706&chksm=e905ede9de7264ff6b2d956764b0eb67ce32c8cf6410d9a9753764bf059f2af3983c7de6145d&scene=21#wechat_redirect)

这里有个疑问，做AUTOSAR开发是否一定要用Developer，好像不一定，有人用MATLAB建模，也可以生成代码。本文对这个就不深入讨论了。



**5. DaVinci Configurator**



这个就是上文提到的DaVinciConfigurator，有时候看到Configurator Pro也是这玩意。

那么它是做什么用的呢？

可以如果你不想看文字，我这里有个视频可以了解下。

![6](https://user-images.githubusercontent.com/80186561/188120403-3323fb4b-a1a0-447e-a1ea-2e71ec1b6dd7.png)

![7](https://user-images.githubusercontent.com/80186561/188120480-0db1f3f8-e334-4d17-8b89-bd27d9655358.jpg)

如果不想看视频，那就看下面文字简单介绍下。

> DaVinci Configurator Pro 让您可以为您的 ECU 配置和生成 AUTOSAR 基础软件 (BSW) 和 RTE——无论它们是 Vector (MICROSAR) 的 BSW 模块还是第三方生产商（例如半导体制造商的 MCAL），甚至是您自己创建的 BSW 模块 . 多阶段和基于规则的验证过程确保所有配置参数的模块间一致性。

最简单直观的理解，它是用于做中间层的配置和生成代码的，但这样理解也不完整，因为它还可以生成SWC和MCAL的配置代码。

综合来说，Developer配置好SWC以及其Port和Runnable后，这个过程是体现在arxml的配置文件上的，也就是Developer做了一大堆的设计，是更改了相应的arxml文件。这时需要Configurator打开工程（相当于导入了这些arxml），然后verify或generate代码。

对于BSW和RTE层，例如OS、RTE、BSWM等，这些是直接在Configurator上面做配置的，然后verify或generate代码。

那么MCAL呢，对于Vector来说，MCAL是他们的ThirdParty内容，SIP里面提供了相关方法将MCAL集成到SIP中，即将MCAL里面的生成器、驱动源码、ARXML等按预定的方法集成到SIP中。这样Configurator可以引用MCAL的ARXML文件以及调研MCAL提供的生成器来生成MCAL的配置代码。

以上简单描述了Developer和Configurator的一些基本功能或作用，如果你深入学习研究，可能还会发现一些其他的作用。



**6. ARXML**



上面提到了ARXML这个东西，到底是什么？
可以理解为它就是XML格式，只是它有更严格的定义，用于AUTOSAR的。
从上面的讲解，可以指定ARXML文件承载着各种各样的配置信息，而且还穿插在SWC、RTE、BSW和MCAL之间。可想而知，他是有一套很规范的定义的。这些东西，实际上可以联系到，在看AUTOSAR规范时遇到的“方法论”这个概念，就是这个方法论贯穿于整个AUTOSAR和工具的使用。
但对初学者来说，知道这些概念就够了，暂时没必要搞懂这个方法论是什么、ARXML定义了什么内容。



**7. AUTOSAR的理论知识和架构**



搞懂了这些概念和工具的用途后，接下来你就会很想了解AUTOSAR是啥东西了。也许你在开始搞这个AUTOSAR的项目之前，你应该通过一些简单的培训或者阅读过介绍的文档，知道了AUTOSAR这个框架了。
我这里也有几个文章讲解这个的，有需要可以参考下，对初学者有一定帮助：

- [我淡定地撸了一遍AUTOSAR的基本概念](http://mp.weixin.qq.com/s?__biz=MzI0MTg5MDY3NA==&mid=2247484012&idx=1&sn=d019f6beb54e1643cd1de3a656a945ad&chksm=e905e671de726f672842c0379e256633098e3f6ae9333f15b2731073252b8270ff5dc5b6a359&scene=21#wechat_redirect)
- [如何研读AUTOSAR官方文档](http://mp.weixin.qq.com/s?__biz=MzI0MTg5MDY3NA==&mid=2247484494&idx=1&sn=217e39ef28c5a50b3738ed12b846e767&chksm=e905e053de726945ab8c62eee3185f08aa8ab465bd6b70cb8f899b70a0cb5d1c5d15b739ec8a&scene=21#wechat_redirect)
- [AUTOSAR架构的故事（干货）](http://mp.weixin.qq.com/s?__biz=MzI0MTg5MDY3NA==&mid=2247484191&idx=1&sn=2d3a6726662f5a8b2d35f17c2a781080&chksm=e905e702de726e14913399585e1f5bcef9658ce39f0ab437d57615ebd9ffbac07223f73ae3f9&scene=21#wechat_redirect)
- [AUTOSAR架构之通信服务（干货）](http://mp.weixin.qq.com/s?__biz=MzI0MTg5MDY3NA==&mid=2247484217&idx=1&sn=b46b7d6e66ea40544ba160df3170e637&chksm=e905e724de726e32919fccc88f8ef6e59e1a8ebef6dcda5ed43b48f0238badd0c0fa248dac62&scene=21#wechat_redirect)
- [这次我要通过Interface来贯穿整个AUTOSAR架构](http://mp.weixin.qq.com/s?__biz=MzI0MTg5MDY3NA==&mid=2247484352&idx=1&sn=19c2f338835102e36a99aec4063cbb68&chksm=e905e7ddde726ecb4f7a12be085168d861ed9898569bd3e63b636e8801538228e959d6a118b7&scene=21#wechat_redirect)

本文就不重复这些内容了。



**8. AUTOSAR的工具怎么用？**



上面只提到了DaVinci Developer和Configurator的用途，大家刚接触这套工具链的时候，还会很疑惑，怎么用它。这是正常的，除了迷糊，你还很渴望驾驭它。
我这方面的教程不是很多，目前有两个可以参考下：

- [AUTOSAR折磨，从新建工程开始](http://mp.weixin.qq.com/s?__biz=MzI0MTg5MDY3NA==&mid=2247484297&idx=1&sn=f0118d55d4a1edf8c097b381b4ba0a2f&chksm=e905e794de726e82b7bcec11dc13e5c22304f83d6b2a71362d8337995f164520146d4aed1c52&scene=21#wechat_redirect)
- [AUTOSAR开发工具DaVinci Configurator里的Modules](http://mp.weixin.qq.com/s?__biz=MzI0MTg5MDY3NA==&mid=2247485477&idx=1&sn=186c8b46f9532160b99628c91344c910&chksm=e905ec38de72652edb540a1b76a56a6e832836d8d03a9b595bca1cf5420d0920df6a1e9c4bb5&scene=21#wechat_redirect)

后续，我会针对DaVinci Developer和Configurator做个专门详细的讲解。
