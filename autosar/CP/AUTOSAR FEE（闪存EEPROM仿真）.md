# AUTOSAR FEE（闪存EEPROM仿真）



# 1. 介绍

本文档描述了闪存EEPROM仿真（**Flash EEPROM Emulation**）模块的功能、API和配置。

**FEE**模块应从设备特定的寻址方案和分段中抽象出来，并为上层提供虚拟寻址方案和分段（**virtual addressing scheme and segment**）以及虚拟的无限数量的擦除周期。

# 2. 缩略语

**EA**

> EEPROM抽象（**EEPROM Abstraction**）

**EEPROM**

> 电可擦除可编程ROM（**Electrically Erasable and Programmable ROM**）。

**FEE**

> 闪存EEPROM仿真（**Flash EEPROM Emulation**）。

**LSB**

> 最低有效位（**Least significant bit**）。

**MemIf**

> 内存抽象接口（**Memory Abstraction Interface**）。

**MSB**

> 最高有效位（**Most significant bit**）。

**NvM**

> NVRAM管理器（**NVRAM Manager**）。

**NVRAM**

> 非易失性RAM（**Non-volatile RAM**）。

**NVRAM block**

> NVRAM管理器所见的管理单元。

**逻辑块（Logical block）/ 块（Block）**

> 逻辑块是模块用户看到的最小可写/可擦除单元。由一个或多个虚拟页面（**Virtual page**）组成。

**Virtual page**

> 虚拟页面可能由一个或多个物理页面组成，以便于处理逻辑块和地址计算.

**内部残留物（Internal residue）**

> 内部残留物是指，如果配置的块大小不是虚拟页面大小的整数倍，则最后一个虚拟页面末尾的未使用空间（参见图 3）.

**虚拟地址（Virtual address）**

> 虚拟地址是由16位逻辑块号和逻辑块内的16位偏移量组成的地址.

**物理地址（Physical address）**

> 物理地址是指用于访问逻辑块的设备特定格式的地址信息，取决于底层EEPROM驱动程序和设备。

**数据集（Dataset）**

> NVRAM管理器的一个概念：用户可寻址的相同大小的块数组。

**冗余副本（Redundant copy）**

> NVRAM管理器的一个概念：将相同的信息存储两次以提高数据存储的可靠性.

# 3. 相关文档

## 3.1. 输入文件

[1] List of Basic Software Modules

> AUTOSAR_TR_BSWModuleList.pdf

[2] Layered Software Architecture

> AUTOSAR_EXP_LayeredSoftwareArchitecture..pdf

[3] General Requirements on Basic Software Modules

> AUTOSAR_SRS_BSWGeneral.pdf

[4] General Requirements on SPAL

> AUTOSAR_SRS_SPALGeneral.pdf

[5] Requirements on Memory Hardware Abstraction Layer

> AUTOSAR_SRS_MemoryHWAbstractionLayer.doc

[6] Specification of Default Error Tracer

> AUTOSAR_SWS_DefaultErrorTracer.pdf

[7] Specification of ECU Configuration

> AUTOSAR_TPS_ECUConfiguration.pdf

[8] Basic Software Module Description Template

> AUTOSAR_TPS_BSWModuleDescriptionTemplate.pdf

[9] General Specification of Basic Software Modules

> AUTOSAR_SWS_BSWGeneral.pdf

## 3.2. 相关标准和规范

[10] AUTOSAR Specification of NVRAM Manager

> AUTOSAR_SWS_NVRAMManager.doc

[11] Specification of Memory Abstraction Interface

> AUTOSAR_SWS_MemoryAbstractionInterface.pdf

[12] Specification of EEPROM Abstraction

> AUTOSAR_SWS_EEPROMAbstraction.pdf

[13] Specification of Memory Access

> AUTOSAR_SWS_MemoryAccess.pdf

[14] Specification of Memory Driver

> AUTOSAR_SWS_MemoryDriver.pdf

## 3.3. 相关规范

**AUTOSAR**提供了基础软件模块的通用规范[9] (AUTOSAR_SWS_BSWGeneral.pdf)，它也适用于Flash EEPROM 仿真。因此基础软件模块的通用规范需被视为Flash EEPROM仿真的附加和必需规范。

# 4. 功能规格

## 4.1. 一般行为

### 4.1.1. 寻址方案和分段

FEE模块为上层提供**32**位虚拟线性地址空间和统一分段方案。该虚拟**32**位地址应包括：

- **16**位块序号（**block number**）：理论上允许数量为**65536**个逻辑块。
- **16**位块偏移（**block offset**）：理论上每块数据块为**64K**字节的块大小。

16位块序号代表一种可配置的虚拟分页（**virtual paging**）机制。而地址对齐（**address alignment**）的值可以从底层闪存驱动程序和设备导出。虚拟分页（**virtual paging**）可通过参数**FeeVirtualPageSize**进行配置。

**FEE**模块的配置应使虚拟页面大小（在**FeeVirtualPageSize**中定义）需是物理页面大小的整数倍，即不允许配置比实际物理页面大小更小的虚拟页面。

**注意：**

AUTOSAR规范要求允许逻辑块的物理起始地址能被计算，而不是通过制作查找表（**Lookup table**）来进行地址映射。

**示例：**

因为虚拟页的大小被配置为8个字节，所以地址为8字节对齐。如果块序号为**1**的逻辑块会放置在物理地址**x**的地址，则块序号为**2**的逻辑块将被放置在**x+8**的地址，块序号**3**将被放置在**x+16**的地址。

每个配置的逻辑块需占用所配置的虚拟页面大小的整数倍。

**示例：**

通过将设置参数**FeeVirtualPageSize**设定为**8**，则地址对齐/虚拟分页配置为**8**个字节。逻辑块编号 1 配置为具有 32 个字节的大小（参见图 3）。这个逻辑块将使用 4 个虚拟页面。因此，下一个逻辑块将获得块号 5，因为块号 2、3 和 4 被第一个逻辑块“阻塞”。第二个块配置为 100 字节大小，占用 13 个虚拟页面，最后一页的 4 个字节未使用。因此，下一个可用的逻辑块号将是 17。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqJW4XDCaC7VOrSh0JEc3D5AKBE1gwHmntuhr0AJTzdBRBDh7V6rHwA30icLM69eYPCuvIvKUJTrZg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

逻辑块不得相互重叠，也不得包含在彼此之内。块序号**0x0000**和**0xFFFF**不能配置为逻辑块。

## 4.2. 地址计算

依赖于**FEE**模块的实现和确切地址格式的使用，FEE模块的功能需能通过合并**16**位块序号（**block number**）和**16**位地址偏移量（**address offset**）来计算得出底层闪存驱动程序所需的物理闪存地址。

**注意：**

底层闪存驱动程序所需的确切地址格式，以及如何从给定的**16**位块序号（**block number**）和**16**位地址偏移量（**address offset**）导出物理闪存地址的机制，取决于闪存设备和该模块的实现，所以不需要被标准化。

只有那些不表示特定数据集（**dataset**）或冗余副本（**redundant copy**）的16位块序号（**block number**）的位需用于地址的计算。

**注意：**

由于**NVRAM**管理器需要此类信息，所以可以使用参数**NVM_DATASET_SELECTION_BITS**为**NVRAM**管理器配置编码的位数。

**示例：**

数据集（**Dataset**）信息配置为使用16位块序号（**block number**）中的四个**LSB**中进行编码，即：每个**NVRAM**块最多允许16个数据集，总共**4094**个**NVRAM**块）。实现者决定直接相邻存储**NVRAM**块的所有数据集，并使用块的长度和指针来访问每个数据集。为了计算块的起始地址（第一个数据集的地址），实现者只使用**12**个MSB。为了访问特定的数据集，实现者需使用块的大小乘以数据集的索引（即：四个**LSB**） 来作为这个数据集的起始地址（如图：3）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/kKdBiaJlWLhqJW4XDCaC7VOrSh0JEc3D5Aiapaias9S9skAYSAZ3TvgZVk6bIkmDkqErMrlYkZBvrickJK51OAbqLA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 4.3. 擦除周期的限制

**FEE**模块的配置需在配置参数**FeeNumberOfWriteCycles**中定义每个逻辑块的预期擦除/写入周期数。

如果底层闪存设备或设备驱动程序没有提供至少配置的每个物理存储单元的擦除/写入周期数，则**FEE**模块需提供分散写入访问的机制，以使物理设备不会承受过大压力。这也适用于**FEE**模块内部使用的所有管理数据。

**示例：**

逻辑块序号1配置为预期的**500,000**个写入周期，底层闪存设备和设备驱动程序仅指定为**100,000**个擦除周期。在这种情况下，**FEE**模块必须提供（至少）五个单独的内存区域，并在内部交替访问这些区域，以便每个物理内存位置最多只能擦除指定的**100,000**个周期。

## 4.4. 处理“即时”数据

包含即时数据（**immediate data**）的块必须立即写入，即**FEE**模块必须确保它可以写入此类块而无需擦除相应的内存区域（例如，通过使用预擦除内存）并且写入请求不会被当前正在运行的模块内部管理操作而延迟。

**注意：**

在即时数据（**immediate data**）写入之前，**NVRAM**管理器需取消正在进行的较低优先级读取/擦除/写入或比较作业。**FEE**模块只需确保该写请求可以即时执行。

**注意：**

由于硬件上正在进行的某个操作，例如：写一页或擦除一个扇区，一旦开始通常不能中止，所以即使对于即时数据（**immediate data**）也必须接受最长硬件操作的最大时间作为延迟。

**示例：**

三个各**10**字节的块已配置为即时数据（**immediate data**）。**FEE**模块/配置工具需保留**30**个字节。如果需要，需加上每个块/页的实现的特定开销，仅供这些即时数据（**immediate data**）使用。也就是说，该内存区域不得用于存储其他类型的数据块。

现在**NVRAM**管理器已经请求**FEE**模块写入**100**字节的数据块。在该**100**字节块被写入的同时，一个（或多个）即时数据块出现需要写入的情况。所以**NVRAM**管理器首先取消正在进行的写请求，并随后发出对包含第一个即时数据块的写请求。正在进行的写入请求的取消由**FEE**模块同步执行，并且可以立即启动底层闪存驱动程序（即：即时数据的写入请求），而无需任何延迟。然而在即时数据（**immediate data**）的第一个字节可以被写入之前，**FEE**模块或更确切地说是底层闪存驱动程序必须等待来自先前写入请求的正在进行的硬件访问结束，例如：写入页面，擦除扇区，通过 SPI 传输，…）。

## 4.5. 管理块正确性信息

**FEE**模块需为每个块管理信息，从**FEE**模块的角度来看，该块是否正确，即：有无未损坏（**corrupted**）。此信息应仅涉及块的内部处理，而不涉及块的内容。

当一个块写入操作开始时，**FEE**模块需将相应的块标记为损坏（**corrupted**）。在块写入操作成功结束后，该块需被再次标记为未损坏（**not corrupted**）。

**注意：**

此内部管理信息不应与通过使用**Fee_InvalidateBlock**服务操作的块的有效性信息相混淆，即：**FEE**应能够区分损坏的块（**corrupted block**）和被上层应用故意标识为无效的块。

## 4.6. 缓冲区对齐

**FEE**模块需将内部缓冲区与**FeeBufferAlignmentValueRef**的值对齐。

**FEE**模块需将读取请求与**FeeMinimumReadPageSize**的值对齐。

# 5. API规范

## 5.1. 函数定义

### 5.1.1. Fee_Init

**说明**: 初始化**FEE**模块的服务。

```
void Fee_Init ( const Fee_ConfigType* ConfigPtr )
```

### 5.1.2. Fee_SetMode

**说明**: 切换底层**Flash Driver**模式的函数。

```
void Fee_SetMode ( MemIf_ModeType Mode )
```

### 5.1.3. Fee_Read

**说明**: 启动读取作业的服务。

```
Std_ReturnType Fee_Read (
    uint16 BlockNumber, 
    uint16 BlockOffset, 
    uint8* DataBufferPtr, 
    uint16 Length )
```

### 5.1.4. Fee_Write

**说明**: 启动写入作业的服务。

```
Std_ReturnType Fee_Write ( 
    uint16 BlockNumber, 
    const uint8* DataBufferPtr )
```

### 5.1.5. Fee_Cancel

**说明**: 调用底层闪存驱动程序的取消函数的服务。

```
void Fee_Cancel ( void )
```

### 5.1.6. Fee_GetStatus

**说明**: 返回状态的服务。

```
MemIf_StatusType Fee_GetStatus ( void )
```

### 5.1.7. Fee_GetJobResult

**说明**: 查询上层软件下发的最后一个接受作业的结果的服务。

```
MemIf_JobResultType Fee_GetJobResult ( void )
```

### 5.1.8. Fee_InvalidateBlock

**说明**: 使逻辑块无效的服务。

```
Std_ReturnType Fee_InvalidateBlock ( uint16 BlockNumber )
```

### 5.1.9. Fee_GetVersionInfo

**说明**: 返回**FEE**模块版本信息的服务。

```
void Fee_GetVersionInfo ( Std_VersionInfoType* VersionInfoPtr )
```

### 5.1.10. Fee_EraseImmediateBlock

**说明**: 擦除逻辑块的服务。

```
Std_ReturnType Fee_EraseImmediateBlock ( uint16 BlockNumber )
```

## 5.2. Callback通知

### 5.2.1. Fee_JobEndNotification

**说明**: 向此模块报告异步操作成功结束的服务。

```
void Fee_JobEndNotification ( void )
```

### 5.2.1. Fee_JobErrorNotification（已过时）

**说明**: 向此模块报告异步操作失败的服务。

```
void Fee_JobErrorNotification ( void )
```

## 5.2. 周期调度函数

### 5.2.1. Fee_MainFunction

**说明**: 处理请求的读/写/擦除作业和内部管理操作的服务。

```
void Fee_MainFunction ( void )
```