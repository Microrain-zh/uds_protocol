# AUTOSAR中的vLinkGen可以干嘛

**vLinkGen是做什么的？**

vLinkGen（Vector链接器脚本生成器）模块提供一种创建链接器脚本的抽象方法，以便将二进制代码，常量和变量（链接器符号）放置在目标内存（例如ROM，RAM等）中的预定义位置。

vLinkGen的基本思想是为链接程序脚本提供一个独立于编译器的描述（除了分配给特定于硬件的内存区域外），以便能够以类似的方式将其应用于多个项目。

链接程序脚本的配置可以完全完成，而无需了解编译器特定的链接程序脚本语言。然后，vLinkGen会以适合所选编译器的格式生成链接器脚本。

这个vLinkGen支持以下编译器

| **Compiler Name** | **Version Restrictions**   |
| ----------------- | -------------------------- |
| ARM               | only 5.x.x, 6.x.x or later |
| Wind River Diab   | -                          |
| GCC               | -                          |
| Green Hills       | -                          |
| HighTec           | -                          |
| IAR               | only 7.x.x or later        |
| Linaro            | -                          |
| Renesas           | -                          |
| Tasking           | -                          |
| Texas Instruments | -                          |

Vector的vLinkGen有好几个版本，而且差异挺大的，以下基于Version 2.1.0来讲解。



**条目定义**

**Hardware Memory Areas** (vBaseEnvMemLayoutHwRegion)

由vBaseEnv模块提供。它们代表硬件上的实际内存布局，无法更改。

**Memory Regions** (vLinkGenMemoryRegion)

是一个逻辑抽象层，用于将配置与实际内存布局分开，并使它独立于硬件。

**Memory Region Blocks**(vLinkGenMemoryRegionBlock)

可以将其链接到区域的下边界或上边界。它们用于生成的链接描述文件中的实际内存定义。

**Logical Groups** (vLinkGenLogical*Group)

是一个逻辑抽象层，用于将多个部分组或链接器符号分组并将它们映射到内存区域块。

**Section Groups** (vLinkGen*SectionGroup)

用于将一个类型的多个链接器节组合在一起。在编译器手册中，它们通常称为“输出部分”。

这些组的大小取决于链接器节的具体内容。同样，它们的起始地址取决于映射到同一存储区块的先前组的配置位置和大小。

**Linker Sections**

最小的实体，包含源代码中分配给它们的不同链接器符号（函数，常量，变量等）的具体内容。在编译器手册中，它们通常称为“输入节”。



简单举例说明配置工具里面的条目的分类



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx82Rb1UGoA6JMwAsphvhDNr2oAF4wnOdZHXRruZKK2GQiaULHhsIhboAwB8pPiblpt44AkSIgSup04g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**例说条目字段配置**

以下具体以常用的bss字段在配置工具里面的配置关系为例讲解这个vLinkGen的配置使用，还有里面的部分参数说明。

首先从vLinkGenLogicalVarGroups找到Data_Default，找到其下下级的bss，里面有些配置属性（详见后文描述）。

Compiler Specific Flags内容链接到vLinkGenPublishedInformation/vLinkGenVarGroupFlags/vLinkGenVarGroupFlag，这里这个Clear表示（初始化）要清除这个段内容的意思。

而Sections，链接到vLinkGenMemLayout/vLinkGenLogicalVarGroup/vLinkGenVarSectionGroup/vLinkGenVarSectionGroupRef，这个bss就是我们常说的存放未初始化的变量的字段（位置）。

更多内容和关系详见下图：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx82Rb1UGoA6JMwAsphvhDNrFRWkib1DTwZ9udV73LiadAo57PKkv182ibJibJg3z6QqRmbvxeVonenVaA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**Short Name**：指这个section group的名字

**Alignment**：这个section group起始地址的对齐

**End Alignment**：这个section group的End或者长度的对齐

**Gap Size**：此值指定section group之后要保留的可选间隙的大小

**Group Size**：此值指定section group的可选大小

**Init Policy**：指定是否在ECU启动时初始化section group。

​	NONE：无需初始化。

​	INIT：用预定义值（ROM到RAM的副本）初始化section group中的所有变量数据。

​	ZERO_INIT：用零完全初始化section group中的所有变量数据。

**Init Stage**：指定ECU启动时用于段组初始化的阶段。仅当Init Policy设置为NONE以外的值时，才有用。

​	NONE：未配置初始化。

​	EARLY：复位后总是直接初始化。仅在确实必要时使用。由于PLL那时可能未初始化，因此大区域的初始化可能需要很长时间。

​	ZERO：始终在默认vBRS启动例程期间首先进行初始化。

​	ONE：始终在默认vBRS启动例程期间进行第二次初始化。默认情况下，如果需要初始化，则应使用此值。

​	HARD_RESET_ONLY：在第一阶段之后立即初始化。但是，仅在发生硬件重置时才进行初始化。

​	TWO、THREE：可选的初始化阶段，将在HARD_RESET_ONLY阶段之后相应地初始化。仅在确实必要时使用。

**Position**：这个是指section group在上一级Logical Group的位置

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx82Rb1UGoA6JMwAsphvhDNrs8xXOKibtlCyzyE31URKoHIEq92qukJHsneU0mwPTc84odDe2DeNiaCA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6AXJMmPWkx82Rb1UGoA6JMwAsphvhDNrsXicpz1ur6HciaKX4NwBicrexEmoVToPAw2ossYDlQribJWxAHp7zBeYnQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这里有个File Generation，看看是什么东西。

vLinkGen提供了生成具有不同内容的不同变体形式的链接描述文件的可能性。可以通过配置中的文件生成（vLinkGen / vLinkGenGeneral / vLinkGenFileGeneration）参数选择它们。

有关不同文件类型及其内容的详细信息，可见下表：

| **STANDARD**             | 生成完整的编译器特定的链接器脚本。分配给一个或多个变量的逻辑组用C预处理程序宏封装。这意味着，如果配置了变体，则可能需要对该文件进行预处理，然后才能将其传递给链接器。 |
| ------------------------ | ------------------------------------------------------------ |
| **ONE_FILE_PER_VARIANT** | 每个变体生成完整的编译器特定的链接器脚本。这些文件可以按原样使用，并通过特定选项传递给链接器。不需要对这些文件进行预处理。 |
| **MODULE_SNIPPETS**      | 生成包含所有逻辑组（包括所有节组）和模块引用的几个链接描述文件片段。 |
| **VARIANT_SNIPPETS**     | 生成几个链接脚本片段，其中包含一个变体的所有逻辑组（包括所有节组）。 |

**数据段初始化**

这个配置能否配置出可以在特定阶段初始化的数据段？答案是有的。

根据配置，vLinkGen会生成一些结构（ 初始化表），可用于：

- 用0初始化存储区块（ECC initialization）。
- 初始化变量具体值（INIT）或零（ZERO_INIT）。
- 将代码从ROM复制到RAM，以便在那里执行。

初始化本身通常由vBRS的启动代码完成。该代码接管对存储区的零写入或将值从ROM复制到RAM。

以下表格列举了生成结构和内容描述：

| **名称**                                            | **内容**                                                     |
| --------------------------------------------------- | ------------------------------------------------------------ |
| vLinkGen_MemAreaSet**vLinkGen_ZeroInit__BlocksSet** | 在某个启动阶段将用零初始化的内存区域块。                     |
| vLinkGen_MemAreaSet**vLinkGen_ZeroInit__GroupsSet** | 在某个启动阶段将用零初始化的VAR节组。                        |
| vLinkGen_RamMemAreaSet**vLinkGen_Init__GroupsSet**  | 在特定的启动阶段要从ROM复制到RAM的节组。这可以是带有初始化数据的VAR节组，也可以是带有要从RAM执行的代码的CONST节组。 |

以上的<Stage>为以下内容（上面的例子也提到过）：

| **Stage**       | **解释**                                                     |
| --------------- | ------------------------------------------------------------ |
| NONE            | 未配置初始化。                                               |
| EARLY           | 复位后总是直接初始化。仅在确实必要时使用。由于PLL那时可能未初始化，因此大区域的初始化可能需要很长时间。 |
| ZERO            | 始终在默认vBRS启动例程期间首先进行初始化。                   |
| ONE             | 始终在默认vBRS启动例程期间进行第二次初始化。默认情况下，如果需要初始化，则应使用此值。 |
| HARD_RESET_ONLY | 在第一阶段之后立即初始化。但是，仅在发生硬件重置时才进行初始化。 |
| TWO、THREE      | 可选的初始化阶段，将在HARD_RESET_ONLY阶段之后相应地初始化。仅在确实必要时使用。 |

来看看具体生成的代码是怎样的：

例如vLinkGen_ZeroInit_Early_Blocks：

```c

/* Memory region blocks to be initialized with zeros in stage Early */
const vLinkGen_MemArea vLinkGen_ZeroInit_Early_Blocks[VLINKGEN_CFG_NUM_ZERO_INIT_EARLY_BLOCKS] =
{
  {
    0uL, 
    0uL, 
    0uL, 
    0uL
  }
};
```

vLinkGen_ZeroInit_One_Blocks

```
/* Memory region blocks to be initialized with zeros in stage One */
const vLinkGen_MemArea vLinkGen_ZeroInit_One_Blocks[VLINKGEN_CFG_NUM_ZERO_INIT_ONE_BLOCKS] =
{
  { /*  LOCAL_RAM_0  */ 
     /*  .Start  */ (uint32)0xFEBD0000uL, 
     /*  .End  */ (uint32)0xFEBF0000uL, 
     /*  .Core  */ 0uL, 
     /*  .Alignment  */ 0uL
  }, 
  {
    0uL, 
    0uL, 
    0uL, 
    0uL
  }
};
```

**集成或生成文件**

这个vLinkGen会包含哪些代码文件？

| 文件名          | 描述                                       |
| --------------- | ------------------------------------------ |
| vLinkGen_Lcfg.c | 包含初始化列表的生成文件                   |
| vLinkGen_Lcfg.h | 包括类型定义和初始化列表外部声明的生成文件 |
| vLinkGen_Cfg.h  | 包含预编译时的宏定义的生成文件             |

还要一个叫vLinkGen_Template.*<ext>*的链接文件，其中*<ext>*是后缀如.ld。

更多的内容，可以参考Vector的TechnicalReference文档，或者对照Davinci Configurator以及生成的配置文件，还要链接文件以前分析，找到对应的符号就很容易理解。此处不累述了。