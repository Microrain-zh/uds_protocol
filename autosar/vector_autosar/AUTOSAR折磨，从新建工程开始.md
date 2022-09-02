# AUTOSAR折磨，从新建工程开始

1**使用案例工程**

方法1，直接使用案例工程，一般SIP包会有一个创建好的案例工程，在这样的路径YOUR_SIP_DIR/Applications/SipAddon/StartApplication下面

 ![1](https://user-images.githubusercontent.com/80186561/188132803-cab1b33b-aa92-4c45-aead-66da12e77724.png)


直接打开这个*.dpa文件即可看到已经预先做好的工程：

![2](https://user-images.githubusercontent.com/80186561/188132856-5fde9d85-03f3-4492-87b1-5ef667d097c7.png)


但是，这个也不是全的，也不一定完全正确，至少MCAL是没有配置好的（MCAL是IC厂商提供的，并不归属SIP包的一部分）。这样就需要你自己去配置你想要的模块，修改里面的错误。



2**创建空工程**

方法2，直接打开SIP包里面的DaVinciConfigurator软件，YOUR_SIP_DIR/DaVinciConfigurator/Core/DaVinciCFG.exe，如下：

 ![3](https://user-images.githubusercontent.com/80186561/188132891-42b18e24-30ff-4341-86bf-98955453108b.png)


根据下面的步骤可以创建一个空工程：

 ![4](https://user-images.githubusercontent.com/80186561/188132923-c7f72dcd-257c-422a-ac5c-d611326512a1.png)


呵呵？工程是要依赖SIP包的，选择你的SIP包，并给工程起一个名字。

 ![5](https://user-images.githubusercontent.com/80186561/188132965-8228eed8-c62a-4404-bfdd-8d11ccc3ff87.png)


以下目录结构就是你创建工程后生成的结构，从下面的名字你可以大概猜测到各个目录的用途。其中这个GenData就是存放配置信息和生成的代码的目录。

 ![6](https://user-images.githubusercontent.com/80186561/188133002-ad8fc6ee-e5a2-4731-a4f7-fbabea49edc7.png)


选择你用的MCU和编译器，我这里以RH850_1587和GreenHills为例。

 ![7](https://user-images.githubusercontent.com/80186561/188133088-b9947b9a-0f0e-4b68-9227-cc7df213ded3.png)


好了，不骗你，创建的空工程，真的是空的。

 ![8](https://user-images.githubusercontent.com/80186561/188133203-7f3b4df2-799a-4ba7-b95d-077e2d6e2979.png)
 ![9](https://user-images.githubusercontent.com/80186561/188133340-8385fbe8-9ec3-4a51-9b73-ea50f489dc85.png)

那么，怎么添加模块呢？打开Project，选Project Settings

 ![10](https://user-images.githubusercontent.com/80186561/188133754-a127e7b3-fe17-43c4-a7ec-bbfa9cf47fec.png)


这样，你可以看到个Modules，然后点击右边的“+”号，Add你所需的模块。

 ![11](https://user-images.githubusercontent.com/80186561/188134769-5da27e15-d102-4aad-a280-4db67ba7a791.png)
 

到这一步，它会问你，所要添加的模块从哪里来？当然SIP啊！

 ![12](https://user-images.githubusercontent.com/80186561/188135096-bd8c992c-6124-4ce7-823c-94e05bcc9e35.png) 

然后，勾选你SIP包里面所包含的模块吧，如果没有你想要的，有可能是你的SIP包里面没有（没购买），或者是非AUTOSAR标准模块。

 ![13](https://user-images.githubusercontent.com/80186561/188135309-524696c4-93c4-4f8b-ba3e-76a9ed67bcdb.png)

 ![14](https://user-images.githubusercontent.com/80186561/188135578-2df57afb-dcfd-4f68-819c-c396d818be00.png)

 ![15](https://user-images.githubusercontent.com/80186561/188135795-901bc1aa-8d14-417f-8e1e-b9c8fb308523.png)

添加好后，就长这样子了。

其中，左边的是按类组合分的，右边就是原始添加的一个个模块的模样（界面叫Basic Editor）

 ![16](https://user-images.githubusercontent.com/80186561/188137801-271668c4-d844-4874-bb45-0797507eb3a0.png)


问题来了，添加后的模块在Configurator自动检查后会提示你有很多错误。

然后，下面这个界面对于大部分错误都有提示或修改建议，有些可以双击一下会自动修复。文章篇幅有限，这里没办法写下所有的错误解决方法，后续有机会再针对具体的问题写分享吧。

如果解决不了的，只能靠经验或者请教有经验的人了。

 ![17](https://user-images.githubusercontent.com/80186561/188138120-ef558761-9e76-43ad-95ee-2119d7bd8197.png)

当你解决完上面的错误，你可以点击检查和生成代码。

 ![18](https://user-images.githubusercontent.com/80186561/188138225-9271aa5c-0e16-49be-a757-6cf4d72c993d.png)


选择你要检查或生成的模块

 ![19](https://user-images.githubusercontent.com/80186561/188138277-74e043a6-323b-45db-9107-e05014b04c7c.png)

