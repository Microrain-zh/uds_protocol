# AUTOSAR折磨，从新建工程开始

1**使用案例工程**

方法1，直接使用案例工程，一般SIP包会有一个创建好的案例工程，在这样的路径YOUR_SIP_DIR/Applications/SipAddon/StartApplication下面

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9WfcFW5PnMA7ctHndtBS7TlGKsMgYRowj4ScuZWQ3NMZpcRr2MJH1Nnicg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

直接打开这个*.dpa文件即可看到已经预先做好的工程：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9Wfz4JxTzicgsaialuZV9arWKkTDx7LSeZOGBvVnfhuHfgQzty6iaM6ZgQJw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

但是，这个也不是全的，也不一定完全正确，至少MCAL是没有配置好的（MCAL是IC厂商提供的，并不归属SIP包的一部分）。这样就需要你自己去配置你想要的模块，修改里面的错误。



2**创建空工程**

方法2，直接打开SIP包里面的DaVinciConfigurator软件，YOUR_SIP_DIR/DaVinciConfigurator/Core/DaVinciCFG.exe，如下：

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9WfDXJatdWGucUpT27pG3TerZfBeeYrjJhzI5Q00Ch0rHiaj8jc2x301qg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

根据下面的步骤可以创建一个空工程：

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9Wf45IaicsTrZfsXlaU9YBfcj9ShDib7hQlcEg5Cqgo46hlF8eORBHliaIMg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

呵呵？工程是要依赖SIP包的，选择你的SIP包，并给工程起一个名字。

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9WfyCEABiaTRbgsgjXC6YBecDTJGxQyEXp6ZfklLb8ibteZvxIbrJKUEBpQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

以下目录结构就是你创建工程后生成的结构，从下面的名字你可以大概猜测到各个目录的用途。其中这个GenData就是存放配置信息和生成的代码的目录。

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9WfIfzC6y5fkIGH2e4tO6jib7HUhUTYiaQZrumKLR2EFLRt0KrqHkZIEZsw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

选择你用的MCU和编译器，我这里以RH850_1587和GreenHills为例。

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9Wf10rPlYoCrLZicNSPoY8Q2DTuXaI6sUib4iaGNwGviany6oaPkepuibfOCHw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

好了，不骗你，创建的空工程，真的是空的。

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9Wf4XdRviaMdn98vge9WB6CPicMd51YNEX8EOs8mtRmmicqSaZtWFpLmWP4Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

那么，怎么添加模块呢？打开Project，选Project Settings

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9WfOW2F8x6Q7ayr7VtQzxicpJMQ98LJyTvmDibn9WuQMg5WejVP5vJKPOgQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这样，你可以看到个Modules，然后点击右边的“+”号，Add你所需的模块。

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9WfpNHrzv9YrhHDl55mbcBXwh8snbbayy4a85IDqcycT6TM9B4mkv8RibA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

到这一步，它会问你，所要添加的模块从哪里来？当然SIP啊！

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9Wf6niciabrJw52cYEbLG1IvcX665icgfkhnDCBMDIrduJSA5gKcQQ8eOiaLQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然后，勾选你SIP包里面所包含的模块吧，如果没有你想要的，有可能是你的SIP包里面没有（没购买），或者是非AUTOSAR标准模块。

  ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9Wf5r0P7YcP4coPJQLibp2jDaPHVicZzibf5Bmjb8OaBpe6vUib0YEt5Stwgw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9WfF1lN0ic6c7TlprjIejZfcykXm8mpjWNUfia3CMuk6cv3IQztaQLC4GKQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9Wfj5MtziaWQ4LxUu0jLWJljrGWHsNQftonNJ9pGCBfEkNDNnJuUKKhrgQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

添加好后，就长这样子了。

其中，左边的是按类组合分的，右边就是原始添加的一个个模块的模样（界面叫Basic Editor）

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9WfmX6M5SF77yIBX8Zcf9swJ7QQRg356FQWK7ax6SRkY5e1HTY69zM5wg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

问题来了，添加后的模块在Configurator自动检查后会提示你有很多错误。

然后，下面这个界面对于大部分错误都有提示或修改建议，有些可以双击一下会自动修复。文章篇幅有限，这里没办法写下所有的错误解决方法，后续有机会再针对具体的问题写分享吧。

如果解决不了的，只能靠经验或者请教有经验的人了。

当你解决完上面的错误，你可以点击检查和生成代码。

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9Wf38cjTSqU8zTicfeY9OmibTJefIOUCQibRGwDZFicZytADdF0IG9Aia6PQrA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

选择你要检查或生成的模块

 ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkxibNWYGS3HSgjs5vqAXkx9Wf9Z0ibOWQBR5JQ7YVOl6ic8tHsaTIBc68zicJCFoUviasHnQvSX6YUD1zBA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

