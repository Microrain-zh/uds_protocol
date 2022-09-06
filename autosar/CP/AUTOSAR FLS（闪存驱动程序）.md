# AUTOSAR FLS（闪存驱动程序）

# 1. 介绍

本文介绍了**AUTOSAR**基础软件模块闪存驱动程序（**Flash Driver**）的功能、**API**和配置，AUTOSAR的闪存驱动程序同时适用于内部和外部闪存设备。

通过底层硬件支持，闪存驱动程序提供了读取、写入和擦除闪存的服务，以及用于设置或者重置关于写入及擦除保护的配置接口。

在**ECU**的应用程序模式下，**Flash**驱动程序仅供**Flash EEPROM emulation**模块用于写入数据使用。**Flash**驱动程序并未打算提供在应用程序模式下把程序代码写入闪存的服务，此部分应在**AUTOSAR**范围之外的引导模式下完成，如**Flash Bootloader**中实现。

内部闪存驱动程序可以直接访问微控制器（**microcontroller**）硬件，它位于**MCAL**微控制器抽象层（**Microcontroller Abstraction Layer**）。然而外部闪存通常连接到微控制器的数据/地址总线，通过内存映射访问，通常闪存驱动程序使用这些总线的处理程序/驱动程序来访问外部闪存设备，所以外部闪存设备的驱动程序位于ECU抽象层（**ECU Abstraction Layer**）。

对于内部和外部驱动程序的功能要求和功能范围是相同的，所以在语义上API的定义应该也是相同的。

# 2. 缩略语

**DET**

> 默认错误跟踪器（**Default Error Tracer**），实现报告开发错误的模块。

**DEM**

> 诊断事件管理器（**Diagnostic Event Manager**）。

**Fls, FLS**

> AUTOSAR闪存驱动程序的官方缩写

**AC**

> 闪存访问代码（**access code**）的缩写。

**闪存扇区**

> 闪存扇区（**Flash sector**）是一次可以擦除的最小闪存量。闪存扇区的大小取决于闪存技术，因此取决于硬件。

**闪存页**

> 闪存页面（**Flash page**）是一次可以编程的最小闪存量。闪存页面的大小取决于闪存技术，因此取决于硬件。

**闪存访问代码**

> 由主函数（作业处理函数）调用的内部闪存驱动程序例程，用于擦除或写入闪存硬件。

# 3. 相关文档

## 3.1. 输入文件

[1] List of Basic Software Modules

> AUTOSAR_TR_BSWModuleList.pdf

[2] Layered Software Architecture,

> AUTOSAR_EXP_LayeredSoftwareArchitecture.pdf

[3] General Requirements on Basic Software Modules,

> AUTOSAR_SRS_BSWGeneral.pdf

[4] General Requirements on SPAL

> AUTOSAR_SRS_SPALGeneral.pdf

[5] Requirements on Flash Driver

> AUTOSAR_SRS_FlashDriver.pdf

[6] Requirements on Memory Hardware Abstraction Layer

> AUTOSAR_SRS_MemoryHWAbstractionLayer.pdf

[7] Specification of ECU Configuration

> AUTOSAR_TPS_ECUConfiguration.pdf

[8] Basic Software Module Description Template

> AUTOSAR_TPS_BSWModuleDescriptionTemplate.pdf

[9] General Specification of Basic Software Modules

> AUTOSAR_SWS_BSWGeneral.pdf

## 3.2. 相关规范

AUTOSAR提供了基础软件模块的通用规范[9]，它同样也适用于**Flash**驱动程序。

因此基础软件模块的通用规范[9]也应被视为**Flash**驱动程序的附加和必需的规范。

# 4. 约束和假设

## 4.1. 限制

闪存驱动程序仅擦除或编程（**programm**）完整的闪存扇区和闪存页面。也就是说因为闪存驱动程序不使用任何内部缓冲区，所以它也不提供任何类型的重写策略。

同时，闪存驱动程序也不提供提供数据完整性的机制。例如：校验和（**checksums**）、冗余存储（**redundant storage**）等。

# 5. 对其他模块的依赖

## 5.1. 系统时钟（System clock）

如果内部闪存的硬件依赖于系统时钟，系统时钟的改变（例如：**PLL On**切换到**PLL Off**）也可能影响到闪存硬件的时钟设置。

## 5.2. 通信或I/O驱动程序

如果闪存位于外部设备，则需要通过相应的I/O驱动程序实现对该设备的通信访问。

# 6. 功能规格

## 6.1. 一般设计规则

**FLS**模块应为闪存操作（包括：读/写/擦除）提供异步（**asynchronous**）服务。

**FLS**模块无需对数据进行额外的缓存，它只需通过API传递的指针参数引用应用程序的数据缓冲区即可。同时**FLS**模块也无需确保应用程序缓冲区数据的一致性。但**FLS**模块需负责确保闪存读取或写入操作期间闪存数据的一致性。

**FLS**模块需负责静态检查所有的静态配置参数是否正确（最迟在编译期间）。

**FLS**模块需将所有可用的闪存区域组成一个线性地址空间（通过参数**FlsBaseAddress**和**FlsTotalSize**表示）。

**FLS**模块需根据闪存区域的物理结构，将读取、写入、擦除和比较函数所使用的地址和长度参数进行虚拟地址（**virtual addresses**）到物理地址（**physical addresses**）的映射转换。只要有关这些地址对齐的限制能被满足，**FLS**模块就可以允许跨越物理闪存区域的边界，进行读取、写入或擦除的作业。**FLS**模块需在内部进行数据缓冲区对齐处理，从而替代对**RAM**缓冲区进行对齐处理。因为传递的指针定义为**uint8***，**FLS**模块而需将指针指向的地址内容看作字节对齐（**byte-aligned**）来处理。

如果一个**ECU**中使用了多个闪存驱动程序实例，则必须为各个实例分配一个唯一的实例**ID**。该实例**ID**应配置为参数**FlsDriverIndex**。如果在ECU中仅使用闪存驱动程序的一个实例，则该实例应将参数**FlsDriverIndex**配置为**0**。

## 6.2. 初始化

初始化会由EcuM调用Fls_Init实现。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhoJPZPMEqZyUrlKtGGxLqPuzpWy3YyTKRPMWOQ5YYazLPIsvicL6BLX2qGtMFibt7trjMqN5oNXqJ9A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 6.3. 同步机制

在FLS模块中，以下服务操作使用同步机制调用：

- **Fls_GetJobResult**
- **Fls_GetStatus**
- **Fls_SetMode**

下面的时序图显示了函数**Fls_GetJobResult**作为该模块的同步操作的示例。同样的顺序也适用于函数**Fls_GetStatus**和**Fls_SetMode**。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhoJPZPMEqZyUrlKtGGxLqPuXCtib6v9icmcrCep9fAjUNr6Prmiatvt2aic5WDgeJhHLp7IFh4ibAg0uCg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 6.4. 异步机制

在FLS模块中，以下服务操作使用同步机制调用：

- **Fls_Erase**
- **Fls_Write**
- **Fls_Read**
- **Fls_Compare**

以下时序图显示了闪存写入功能作为该模块异步操作的示例（设置了配置选项**FlsAcLoadOnJobStart**）。相同的顺序适用于擦除、读取和比较作业，唯一的区别是对于读取和比较作业，不需要将闪存访问代码加载到**RAM**或从**RAM**中卸载。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhoJPZPMEqZyUrlKtGGxLqPuylrUWYv5zLbTo57Wyiciania1q575qotkOYicYDlzfuyhBTxjmQTeyKLSA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 6.5. 取消正在运行的作业

以下时序图显示了如何取消正在运行的作业的示例。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhoJPZPMEqZyUrlKtGGxLqPuM9hJeYKGK0wiaEnMhDeGORHgSbib3oQuDPT4EUicT4Tx9sHDg91b1GpDQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**注意：** **FLS**模块在运行**Fls_MainFunction**调用期间，函数**Fls_Cancel**是不允许调用的。

这是通过以下的任意一种调度配置赖实现的：

1. **NVRAM**管理器和闪存驱动的工作功能是同步的（例如：在一个**Task**中顺序调用）
2. 调用**Fls_MainFunction**函数的任务，不能被其他**Task**抢占。

## 6.6. 外置闪存驱动程序（External flash driver）

在外部闪存驱动程序初始化期间，**FLS**模块需根据相应的发布参数，检查外部闪存设备的硬件**ID**。如果发生硬件**ID**不匹配的情况，**FLS**模块需返回错误代码**FLS_E_UNEXPECTED_FLASH_ID**报告给默认错误跟踪器（**DET**），并将**FLS**模块状态设置为**FLS_E_UNINIT**，并且无需再自行初始化了。

所需参数的完整列表已在SPI处理程序/驱动程序软件规范中指定。

## 6.7. 加载、执行和卸载闪存访问代码

**技术背景信息：**

基于闪存内存空间分段等技术，可能需要在RAM空间里执行访问闪存硬件的例程（如：内部擦除，写入例程等）。主要原因是在闪存的编程过程中是不允许读取闪存，即：用于载入执行代码所需的读取指令。

**FLS**模块的会将此类加载到RAM中的闪存访问例程的代码，单独放在C模块的**Fls_ac.c**文章中，便于区分。

**FLS**模块的闪存访问例程应仅在必要时，才能禁用中断（**disable interrupt**）并等待擦除/写入命令完成。也就是说，在擦除或者写入时，必须确保没有其他代码被执行。同时，**FLS**模块也需要尽可能地缩短闪存访问代码的执行时间。

### 6.7.1. 加载闪存访问代码

如果**FLS**模块配置为在作业开始时需将闪存访问代码加载到**RAM**中，则**FLS**模块的擦除例程应将用于擦除闪存的访问代码，加载到闪存驱动程序配置集中包含的擦除功能指针所指向的**RAM**中的位置。

如果**FLS**模块配置为在作业启动时需将闪存访问代码加载到**RAM**中，则**FLS**模块的写入例程应将用于写入闪存的访问代码，加载到闪存驱动程序配置集中包含地写入功能指针所指向的**RAM**中的位置。

**注意：**

**FLS**模块只有在闪存ROM无法执行访问码时，才需将访问代码加载到**RAM**中。

### 6.7.2. 执行闪存访问代码

**FLS**模块的主处理程序需执行闪存访问代码程序。

**FLS**模块的主处理例程，应通过**FLS**模块配置集中包含的相应函数指针，访问闪存访问代码例程，无论闪存访问代码例程是否已加载到**RAM**中，或者闪存访问代码可以从闪存的**ROM**中直接执行。

### 6.7.3. 卸载闪存访问代码

擦除或写入作业完成或取消后，如果闪存访问代码（包括：擦除和写入例程）已被闪存驱动程序加载到**RAM**，则**FLS**模块的主处理例程需要把闪存访问代码从**RAM**中卸载。

# 7. 错误处理

## 7.1. 运行时错误

**FLS_E_VERIFY_ERASE_FAILED**

> 当擦除验证功能开启（通过编译开关：**FlsEraseVerificationEnabled**）且擦除验证功能（空白检查）失败时，需报告运行时错误代码**FLS_E_VERIFY_ERASE_FAILED**。

**FLS_E_VERIFY_WRITE_FAILED**

> 当写入验证功能开启（通过编译开关：**FlsWriteVerificationEnabled**）且写验证功能（比较）失败时，需报告运行时错误代码**FLS_E_VERIFY_WRITE_FAILED**。

**FLS_E_TIMEOUT**

> 当超时监督功能开启（通过编译开关：**FlsTimeoutSupervisionEnabled**）并且读、写、擦除或比较作业（在硬件中）的超时失败时，需报告运行时错误代码**FLS_E_TIMEOUT**。

## 7.2. 瞬态故障

**FLS_E_ERASE_FAILED**

> 当闪存擦除功能失败（硬件故障）时，应报告瞬时故障代码**FLS_E_ERASE_FAILED**。

**FLS_E_WRITE_FAILED**

> 当闪存写入功能失败（硬件故障）时，应报告瞬时故障代码**FLS_E_WRITE_FAILED**。

**FLS_E_READ_FAILED**

> 当闪存读取功能失败（硬件故障）时，应报告瞬时故障代码**FLS_E_READ_FAILED**。

**FLS_E_COMPARE_FAILED**

> 当闪存比较功能失败（硬件故障）时，应报告瞬时故障代码**FLS_E_COMPARE_FAILED**。

**FLS_E_UNEXPECTED_FLASH_ID**

> 当期望的闪存**ID**不匹配时，应报告瞬态故障代码**FLS_E_UNEXPECTED_FLASH_ID**。

# 8. API规范

## 8.1. 函数定义

### 8.1.1. Fls_Init

**说明**: 初始化闪存驱动程序。

```
void Fls_Init ( const Fls_ConfigType* ConfigPtr )
```

### 8.1.2. Fls_Erase

**说明**: 擦除闪存扇区。

```
Std_ReturnType Fls_Erase ( Fls_AddressType TargetAddress, Fls_LengthType Length )
```

### 8.1.3. Fls_Write

**说明**: 写入一个或多个完整的闪存页面。

```
Std_ReturnType Fls_Write ( Fls_AddressType TargetAddress, const uint8* SourceAddressPtr, Fls_LengthType Length )
```

### 8.1.4. Fls_Cancel

**说明**: 取消正在进行的作业。

```
void Fls_Cancel ( void )
```

### 8.1.5. Fls_GetStatus

**说明**: 返回驱动程序状态。

```
MemIf_StatusType Fls_GetStatus ( void )
```

### 8.1.6. Fls_GetJobResult

**说明**: 返回上一个作业的结果。

```
MemIf_JobResultType Fls_GetJobResult ( void )
```

### 8.1.7. Fls_Read

**说明**: 从闪存读取。

```
Std_ReturnType Fls_Read ( Fls_AddressType SourceAddress, uint8* TargetAddressPtr, Fls_LengthType Length )
```

### 8.1.8. Fls_Compare

**说明**: 将闪存区域的内容与应用程序数据缓冲区的内容进行比较。

```
Std_ReturnType Fls_Compare ( Fls_AddressType SourceAddress, const uint8* TargetAddressPtr, Fls_LengthType Length )
```

### 8.1.9. Fls_SetMode

**说明**: 设置闪存驱动程序的操作模式。**MEMIF_MODE_SLOW**：缓慢的读取访问/正常的SPI访问。**MEMIF_MODE_FAST**：快速读取访问/SPI突发访问（Burst Access）

```
void Fls_SetMode ( MemIf_ModeType Mode )
```

### 8.1.10. Fls_GetVersionInfo

**说明**: 返回此模块的版本信息。

```
void Fls_GetVersionInfo ( Std_VersionInfoType* VersioninfoPtr )
```

### 8.1.11. Fls_BlankCheck

**说明**: 函数**Fls_BlankCheck**将验证给定的存储区域是否已被擦除，并还未被编程。该函数应将每个主函数周期的最大检查闪存单元数分别限制为配置值**FlsMaxReadNormalMode**或**FlsMaxReadFastMode**。

```
Std_ReturnType Fls_BlankCheck ( Fls_AddressType TargetAddress, Fls_LengthType Length )
```

### 8.1.12. Fls_MainFunction

**说明**: 执行FLS作业的处理的主函数。

```
void Fls_MainFunction ( void )
```