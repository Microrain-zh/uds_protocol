# Autosar概念：Callout与Callback，请不要混肴

本文讨论焦点是Callout与Callback，在Autosar里面，Callout与Callback分别是什么？两者区别是什么？搞Autosar这些年了，你分清楚了没？

1

Callback

来，先看一下，Autosar规范里是如何定义“Callback”的，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vy8uOnSVqfz28p2lUE2X9aG1KFfRQq0n6FN3Fpo0wWqBZicKQHxJuFYNEpxs38KV9a1ibDfw2W514Pg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里就不照着翻译了，来说一下个人理解。Autosar的软件架构将各个模块进行了分层，每个模块各司其职，但又需要信息交互。信息交互的过程中，底层需要为上层提供服务，上层也需要时时知道自己请求服务的状态（是否执行成功），比如：有应用层消息来了，路由模块（如：PDUR）将该消息路由给COM层。但是这个消息怎么给COM呢？对，就是上图说的，COM需要在PDUR提前注册一个函数，当有消息给COM的时候，PDUR直接调用对应接口通知到COM即可，而COM不必一直等待。

这里说的消息就是上图中发生的特定事件（Certain Events）或者异步处理完成事件（asynchronous processing completes）。

## Com_RxIndication 

以Com_RxIndication 为例，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vy8uOnSVqfz28p2lUE2X9aGnMZfuhO91icrSdg47Ok6ydjJTicVDouR2tKjcCv6IeeO3Nm7NHXiav3XQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

也就是说，当PDUR接收到一个I-PDU（xxIf模块传给PDUR），且该I-PDU需要送给COM时，PDUR需要调用COM提前在PDUR注册好的接口：Com_RxIndication。

Com_RxIndication接口有什么特点呢？**它是Autosar标准接口**。在Autosar规范中，其函数原型有详细说明，如上图。

在Autosar中，Callback会有独立的章节讨论，比如COM层，其Callback接口如下所示，大家在看Autosar规范时可以留意一下。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vy8uOnSVqfz28p2lUE2X9aGczmGPezGjG960WlMJnJTciakr1mMQCqVfNyibPzoonDTSfX9ZdOJNXjA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## Callback的百度词条解释 

百度词条中给出的callback解释如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vzPvthbrfEqtIfbqTcWUNIStClqSq0kUschuURmv2ZWyCvcqWkwlic2OqW4POmuVjQeZK7LGgC9sqQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Callback在项目的开发中，使用还是很频繁的。这里谈论一下自己开发中的使用经验。Callback的地址经常会当作一个函数形参，我常常用其作为一个功能模块的“后门”，为什么这样做呢？做项目开发中，有些框架或者模块成熟了以后，后期的开发一般会移植、复用。但是当新的项目有额外功能要求时，又不想推倒原有的框架重来，那么通过预留的Callback实现，就方便了很多，而且不用修改框架。

**模块中Callback的使用示例**：

```
typedef void* (*callBack_Pre)(void* argu1, void* argu2);
typedef void* (*callBack_Post)(void* argu1);

typedef struct{
  int  Module_A_arg;
  callBack_Pre Module_A_CallbackPre;
  callBack_Post Module_A_CallbackPost;
}Module_A_Type;

Module_A_Type Module_A_Obj;

void Module_A_Handler(Module_A_Type* Module_A_Obj_Info)
{
  /* Module A Pre Handle */
  if(Module_A_Obj_Info->Module_A_CallbackPre != nullptr)
  {
    Module_A_Obj_Info->Module_A_CallbackPre(nullptr, nullptr);
  }
  /* Module A Post Handle */
  if(Module_A_Obj_Info->Module_A_CallbackPost != nullptr)
  {
    Module_A_Obj_Info->Module_A_CallbackPost(nullptr, nullptr);
  }
}typedef void* (*callBack_Pre)(void* argu1, void* argu2);
typedef void* (*callBack_Post)(void* argu1);

typedef struct{
  int  Module_A_arg;
  callBack_Pre Module_A_CallbackPre;
  callBack_Post Module_A_CallbackPost;
}Module_A_Type;

Module_A_Type Module_A_Obj;

void Module_A_Handler(Module_A_Type* Module_A_Obj_Info)
{
  /* Module A Pre Handle */
  if(Module_A_Obj_Info->Module_A_CallbackPre != nullptr)
  {
    Module_A_Obj_Info->Module_A_CallbackPre(nullptr, nullptr);
  }
  /* Module A Post Handle */
  if(Module_A_Obj_Info->Module_A_CallbackPost != nullptr)
  {
    Module_A_Obj_Info->Module_A_CallbackPost(nullptr, nullptr);
  }
}
```

**说明：**

如上，需要开发一个Module A模块，实现某个功能需求，一般我会先定义这个模块所需的信息**对象**，即定义一个**结构体**Module_A_Type，该结构体里面会包含两个函数指针，一个是Module A处理信息前的Action，一个是Module A处理信息后的Action。

需要添加特定功能时，可以根据实际情况，决定在Module A处理信息之前添加还是处理信息之后添加。

2

Callout

Autosar规范对Callout的具体定义如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vy8uOnSVqfz28p2lUE2X9aG5TsTQYMGc3suibcBpMjRyqqXMqR7ibetxiar6dRrEpVxYyeUaBHdLXaPg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

简单说：**拓展Autosar功能的函数叫Callout**，但Autosar并没有明确Callout的形式。

## 为什么要用Callout

为什么要用Callout呢？我们应该知道，实际工程项目中，需求是多变的，在一个项目的工程周期内，OEM可能会增加N次的需求修改，有些需求按照Autosar是没法实现的，说白了，配置工具是配不出来定制需求的。怎么办？给系统说做不了？项目不做了？别急，万事有办法。制定Autosar的这帮家伙，都是人精一样的存在，他们怎么会没有想到呢。所以，在Autosar的框架里，很多模块都给你提供了Callout的配置项，如下，给出部分含有Callout选项的模块。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vy8uOnSVqfz28p2lUE2X9aG03bQm8NibhNHgpdfyQ3h3rbibibObiadTAHyPXAH04DLA1WntFH6qJyQuw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## Callout Class1/Class2

在Autosar中，Callout又分为Class1和Class2两个层级。

Class1：强制的，开发者没得选，必须结合项目实际完成此部分的功能实现；

Class2：可选则，开发者根据项目情况决定是否需要拓展此功能。

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vy8uOnSVqfz28p2lUE2X9aGKTUF6BqYc4ickf9B63Js5YyeXib7zt3ricwerV19R2xyb4eNE4VSBE2nw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如上图，“Opt”栏，“no”表示此Callout必须实现，比如：获取复位原因，ECU每次复位时，会从对应寄存器或者读取目标IO口读取复位原因通知上层。该功能必须要有，但是每个项目复位的触发方式和数量不同，因此Autosar留给开发者实现。“yes”表示此Callout可选，可以实现，也可以不实现。

Callout特点：**针对指定模块**。Callout不像Callback那样，可以跨模块。Callout是本模块功能的拓展，或者说某个函数功能的拓展，而Callback即可以是上下层模块信息交互的标准接口，也可以是本模块中，某个函数的形参。

**参考资料**

AUTOSAR_TR_Glossary.pdf

AUTOSAR_SWS_COM.pdf

AUTOSAR_SWS_ECUStateManager.pdf

https://baike.baidu.com/item/CALLBACK