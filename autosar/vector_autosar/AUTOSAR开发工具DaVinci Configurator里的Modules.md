# AUTOSAR开发工具DaVinci Configurator里的Modules

DaVinci Configurator 里面有个Module这个概念。

如你所想，基本上跟AUTOSAR架构里面的Module相对应

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9ILot2kT5BiasmlZQ0ASxMyJm3kA2rWm6EBhREqmAh4J5eoCTgXggicLGCGZmlT7PJKGtkWaUFmKWw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

从软件的Project菜单中的Basic Editor项可以打开

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9ILot2kT5BiasmlZQ0ASxMyNn5e1KtZlBb6LH8ia6u9ftFfpTJcvY3gPYdIdhCkBDhXumMEAe90r0w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

打开这个菜单后，会看到很多Modules项以及其相关配置项

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9ILot2kT5BiasmlZQ0ASxMy5via5txXgznJUW7UQ33MItDN9QvJPPwojdgXqmhmeicicib2tgHIAFJk6g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这个Basic Editor显示出整个ECU配置中的所有Module配置项

即使是Configuration Editor里面的配置项都能在Basic Editor找到对应的，例如下图的IoHwAb

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9ILot2kT5BiasmlZQ0ASxMy2e8zicFQ9ibjQLPtlNJQ6DyxYgfFdwafxXJjnENXfqeXsvZ9Vun7k3zQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

对于Basic Editor里面的Module内容，也许你会有几个疑问：

1. 为什么Module有几种不同的颜色图标，各代表什么意思？
2. Module下面的选项也有不同图标，各又是什么意思？

以下一一讲解。

不同颜色Module图标代表的意思：

| ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9ILot2kT5BiasmlZQ0ASxMy8f1MibkkSGDTBibw7HkkexPxrENpcQUdBmP447EtMSXPRdwDof3pRZPA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) | AUTOSAR module.                                  |
| ------------------------------------------------------------ | ------------------------------------------------ |
| ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9ILot2kT5BiasmlZQ0ASxMydD5CaVqG5DriazHl2SPx8TPBfLBV0gpFO8h7DOoANrwAWvYmiakdaAbg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) | AUTOSAR driver module.                           |
| ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9ILot2kT5BiasmlZQ0ASxMySW18wL48rn1wlBch1EKlOYAcXmaD423qFSrIxlgIh1KgGw4zJalBnA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) | Non AUTOSAR module.                              |
| ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9ILot2kT5BiasmlZQ0ASxMyBYvbiaqSpO0f9lu4ChdrYd1uB53iaCgnsrsqS5GHO4cf9pb6vWljvG8w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) | Non AUTOSAR driver module.                       |
| ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9ILot2kT5BiasmlZQ0ASxMySK8QmdNP3oqXVwNCmMDyNPD7ktOEonUm4xibaNSLxh6xCj3FKm9PgMg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) | Module without associated BSWMD file in the SIP. |

不同颜色Module图标代表的意思：

![image-20220902172839264](C:\Users\wangfeitao\AppData\Roaming\Typora\typora-user-images\image-20220902172839264.png)

对了，还有个问题，Module是怎么添加进来的？

从Project菜单中的Project Settings界面

  ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9ILot2kT5BiasmlZQ0ASxMyaB7OLhvso2hfWppIHljqB16HwoHkJBI5mepfnJoVz8axOfMGv5q63w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后点击Modules，在右侧点击**+**或**x**图标来增删Module。

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx9ILot2kT5BiasmlZQ0ASxMy8TIvNjIpr5SDcqS98r4gNrJvCqScdRjG5kgnC0iad6PIsUym4qiaOEQA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

至于你能增加哪些Module，就取决于你的SIP包了。