# AUTOSAR开发工具DaVinci Configurator里的Modules

DaVinci Configurator 里面有个Module这个概念。

如你所想，基本上跟AUTOSAR架构里面的Module相对应

 ![1](https://user-images.githubusercontent.com/80186561/188121890-f000fe08-f732-49a3-a987-b12a9f1769e4.png)


从软件的Project菜单中的Basic Editor项可以打开

![2](https://user-images.githubusercontent.com/80186561/188121943-cebc8316-681e-4edb-8877-1327b135e3ca.png)


打开这个菜单后，会看到很多Modules项以及其相关配置项

![3](https://user-images.githubusercontent.com/80186561/188121988-0f93de68-b5f1-4699-b1d7-2bfcaf43e623.png)


这个Basic Editor显示出整个ECU配置中的所有Module配置项

即使是Configuration Editor里面的配置项都能在Basic Editor找到对应的，例如下图的IoHwAb

 ![4](https://user-images.githubusercontent.com/80186561/188122055-66d4513a-ec98-46f9-874b-52b8182cb7d0.png)


对于Basic Editor里面的Module内容，也许你会有几个疑问：

1. 为什么Module有几种不同的颜色图标，各代表什么意思？
2. Module下面的选项也有不同图标，各又是什么意思？

以下一一讲解。

不同颜色Module图标代表的意思：

| ![1](https://user-images.githubusercontent.com/80186561/188122259-09ac0727-52d1-4332-9c80-94d2e4f18a3e.png) | AUTOSAR module.                                      |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| ![2](https://user-images.githubusercontent.com/80186561/188122397-a4510324-7d39-4a5d-9cb7-8538769ac447.png) | **AUTOSAR driver module.**                           |
| ![3](https://user-images.githubusercontent.com/80186561/188122447-f24f4316-d153-4a19-9201-55b08002d523.png) | **Non AUTOSAR module.**                              |
| ![4](https://user-images.githubusercontent.com/80186561/188122488-171a5d71-5e60-48f4-9cab-0250f94a63e5.png) | **Non AUTOSAR driver module.**                       |
| ![5](https://user-images.githubusercontent.com/80186561/188122527-b9a8a5f8-4f83-4dc4-9da2-5b1d3d24cb8f.png) | **Module without associated BSWMD file in the SIP.** |

不同颜色Module图标代表的意思：

![image-20220902172839264](https://user-images.githubusercontent.com/80186561/188117935-896c305b-9f88-40dc-bebc-0c64fbfb5fc3.png)


对了，还有个问题，Module是怎么添加进来的？

从Project菜单中的Project Settings界面

  ![5](https://user-images.githubusercontent.com/80186561/188122123-88ce39f0-1626-4c16-8d56-9e0cead2debe.png)


然后点击Modules，在右侧点击**+**或**x**图标来增删Module。

 ![6](https://user-images.githubusercontent.com/80186561/188122177-2531f7fc-cb57-409d-bcdd-8f1f97a7b315.png)


至于你能增加哪些Module，就取决于你的SIP包了。
