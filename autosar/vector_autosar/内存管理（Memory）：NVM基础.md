# 内存管理（Memory）：NVM基础

项目开发中，存储关键参数到非易失存储区，几乎必不可少。非易失存储区介质有EEPROM(electrically erasable programmable read only memory)、ROM(Read-Only Memory)等。ROM又可以分为PFLASH(Program Flash)和DFLASH(Data Flash)。

本文，基于英飞凌tc397，聊一聊NVM(non-volatile memory)。

1

FLASH和EEPROM对比

## 相同点：

FLASH和EEPROM均是非易失存储介质，ECU掉电后，存储的数据不丢失。

## 不同点

1. FLASH存储次数低，一般≥10万次的擦写寿命。而稍微好一点的EEPROM，擦/写寿命≥100万次（也有≥10万次的EEPROM）；
2. FLASH数据存储时间短，一般≥20年。EEPROM数据存储时间一般≥100年；
3. FLASH价格低，容量大。EEPROM价格高，容量小；
4. FLASH读/写速度快，EEPROM读/写速度慢。

EERPOM读写速度慢的主要原因是受限于IIC和SPI的通信速度，以AT25040（SPI Serial EEPROMs）为例，最高时钟频率2.1MHz。而FLASH在uC内部，CPU访问FLASH的速度更快（CPU的主频一般都是**百万兆赫兹**）。

通过对比，可以看出EEPROM，在价格、存储容量、读/写速度上不占据优势，满足功能的前提下，每个企业都会努力地降低生产成本，以实现利益最大化。所以，相对于额外购买EEPROM芯片，使用uC内部的DFLASH模拟EEPROM会降低成本，所以，项目中常用此方式存储关键参数，也就是我们常说的“数据存NVM”。

项目中我们常说的“内部EEPROM”，一般指DFlash模拟的EEPROM，当然，uC内部也可以集成EEPROM；“外部EEPROM”是指通过SPI或者IIC与外部连接的EEPROM芯片或者外部DFLASH模拟的EEPROM。

Autosar的框架中，与外部EEPROM通信，仅支持SPI，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vz4icvq3ibv2UeSicmDx5TXp8LskMt5xeXa7l3ia32UIknz2yerK5KTibaeXsNNK2Qfialv4xAlnav1Idpg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**FEE**：Flash EEPROM Emulation，也就是DFlash模拟的EEPROM。

2

基础概念

tc397 NVM的子系统框架如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vz4icvq3ibv2UeSicmDx5TXp8LxzsP9Srg1ezJbOPKMRgzp3rXHITEicPkMdwXJHx09gF53fJqLHFO7ew/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

由上可以看出，NVM不仅仅是指我们存储数据所用的DF0，NVM还包括：PFlash、UCB（User Configuration Blocks）、CFS（Configuration Sector）、DFlash1。

**Flash Module**：简单说，就是包含Non Volatile Memory (NVM)的模块，我们平时常说的数据存"NVM"（本意要表达：将某些关键参数存NVM），多指DFlash模拟的EEPROM。

**Bank**：一个Flash Module包含不同的Bank，比如：PFlash包含多个PF Bank，DFlash包含多个DF Bank。

**NVM**：Non Volatile Memory，用于存储信息的物理内存，包含多个扇区（Sector），上层模块访问非易失存储区数据的统一接口。

## 1、Data Flash EEPROM地址空间

tc397中，可以模拟EEPROM的模块有DF0（Data Flash 0 EEPROM）和DF1（Data Flash 1 EEPROM），当使用HSM时，DF1分配给HSM使用。所以，常用DF0实现EEPROM功能，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vx7Mia1nJrIpFnFziaLYgQsviaEsovbKHLQkiaDtFBSlqRYcAZnj5N8JSUXusMNSYHEAT5wjUrY1mcROA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 2、扇区（Sector）

扇区分为物理扇区（Physical Sector）和逻辑扇区（Logical Sector），一个Physical Sector包含多个Logical Sector，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vz4icvq3ibv2UeSicmDx5TXp8LNPrTboJMlZ26HPzklLib6mooOeib8LDdqQhtpWWAoNgviccl8nv0lof1w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**1**、Logical Sector是FLASH擦除的最小单位（DFlash最小可擦除4 Kbyte或者2 Kbyte）。

**2**、DFLASH0_EEPROM分为两种模式：

- **single ended mode**：每个Logical Sector空间大小为4 Kbyte，最多分配256个Logical Sector，可以使用空间256 * 4 = 1024 Kbye = 1 Mbyte。
- **complement sensing mode**：每个Logical Sector空间大小为2 Kbyte，最多分配256个Logical Sector，可以使用空间256 * 2 = 512 Kbye，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vz4icvq3ibv2UeSicmDx5TXp8LpmzjZYudD6bw9MF7KsyJSWNQeyvTicxs5QLDSPpLuk86nOkHicqWWIWA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 3、页（Page）

**Page**：可编程的最小单位，即：一次最少写一个Page。对于tc397 DFlash，Page = 8 bytes，所以，写一次NVM，最少要写8 bytes数据。假设：程序只更新了一个uint16类型参数Argu16(2 byte)，也必须写8 byte数据，实际需要更新此参数所在的Block，需要重新写一遍此Block。

假设：100 bytes个有效数据需要存储，为了能存储这100 bytes有效数据，在实现上，实际需要分配空间：8（起始行）+8（结束行）+（100 + 2（CRC））/ 7*8 = 136 byte。136 byte需要分配17 个Page，会有3个bytes用不到（可以填充处理），为了保证对齐，3 byte的浪费不可避免。

tc3xx，FEE Block信息示意如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vxlsOmjJ5XYPjuTjUT0W1vGenpy7Wqj5tz9RpYLR7mZuC9E3lp2j32n2C6pvRF22o8vxW5WF9MmuA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

- **A3**：表示Block的起始行，占用一个byte
- **65**：表示Block的结束行，占用一个byte
- **9C**：表示数据存储行，占用一个byte

这些信息类似诊断的PCI（Protocol Control Information），不同厂家的实现方式会有所不同，同一厂家的不同uC版本也可能不同。

## 4、motorola格式

在识别数据时，有时会犯迷糊，这里补充一下数据格式知识点。假设NVM存储使用motorola格式，在地址0xAF0004F0位置，存储首行信息，如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vxlsOmjJ5XYPjuTjUT0W1vGlXKYO1L2xPaG6pIaXAsM6FL9P1n8snrtX93xibDiblhv7FA3zMClXSTQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

分析Number of Data Block Pages（15 bits），Byte6 = 0x80，byte7 = 0x12(motorola格式：低地址存储高字节数据)，从Byte7的bit0到Byte6的bit6就是Number of Data Block Pages，即0x21。Intel格式（低地址代表低字节，高地址代表高字节）本文不讨论。

3

DF0模拟EEPROM

**假设**：需要分配3个Blcok，Block1存储100 bytes参数，Block2存储38 bytes参数，Block3存储40 bytes参数。

根据项目的实际情况，在DF0的末尾分配两个64 Kbyte空间模拟EEPROM，即Page0和Page1（注意与物理层page区分）。当前Page 0处于有效可写的模式，NVM的使用情况如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vykH8h38yn8sHxZXln4XmAO02bWAK4PpOIId4Wtk7LzR4wa95vcQY39hpFjLVwib2b3BicOIMHaRujw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**注意**：工程中，我们习惯将Page 0和Page 1称为"Bank"。

- Block1存储100 bytes参数，实际需要分配104 bytes空间(4 bytes未使用)，因为一次写操作，最少写一个page（8 bytes）,如果需要写Block 1，则每次需要写13个Page;
- Block2存储38 bytes参数，实际需要分配40 bytes空间(2 bytes未使用)，因为一次写操作，最少写一个page（8 bytes）。如果需要写Block 2，则每次需要写5个Page;
- Block3存储40 bytes参数，实际需要分配40 bytes空间，空间充分利用。如果需要写Block 3，则每次需要写5个Page。

如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vykH8h38yn8sHxZXln4XmAO2LlxicC7Xl0HZeZMbzQs4gnVBsAfiayPXbZhUicWJwp5KYdHrVibOSozkA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 工程思考 

当某个Block的参数修改以后，NVM需要存储最新的参数，而一次存储是将整个Block的参数一次性存储，如果Block空间分配过大，消耗的NVM空间就过大，这样会导致一个Page（假设Page 0）很快写完，进行换页操作，过快的换页操作也意味着DFlash擦除次数的提高，使得DFlash变得“不耐用”。而且切页操作耗时，可能会影响到某一时刻程序的运行状态。所以，当某些参数改写频繁时，可以将这些参数单独配置一个Block，不要与数据变动率低的参数放到一个Blcok中，这样可以延长NVM的使用寿命。

**假设**：Page大小为2048 byte，Block 1占用500 byte空间，Block 2占用50 byte空间。每次Argu1（4 byte）更新（存储NVM），需要将Argu1所在的Block整体重新写一遍，而写一遍Block 1需要使用500 byte空间。如果把Argu1放在Block 2中，只需要消耗50 byte空间。如果使用Block 1存储Argu1，Argu1更新到第5次时，就需要切页；如果使用Block 2存储Argu1，Argu1更新到第41次时，才需要切页，极大的降低了NVM的擦/写频率，延长其使用寿命。Block1、Block2示意如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/eEEQvxEw8vykH8h38yn8sHxZXln4XmAOYvL7qTSPj11vFNQZV77uDFLia5n5FzsL4MtdRyHT8tDEGemAQKakJCg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



**致 谢**

对于NVM，我的理解还不够深刻，学习NVM过程中，请教了几位不错的工程师，特此感谢！在此，将我所学、所理解的分享给大家。如有理解偏差，请大家指正。