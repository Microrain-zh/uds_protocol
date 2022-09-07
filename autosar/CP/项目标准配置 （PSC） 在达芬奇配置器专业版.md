## 项目标准配置 （PSC） 在达芬奇配置器专业版

### 问题：

什么是**项目标准配置** （**PSC**）？

### 答：

**项目标准配置**（**PSC**）是一个ARXML文件，主要由OEM提供，其中包含**基本软件模块**的**ECU配置**（**EcuC**）工件，可以作为输入文件导入**到达芬奇配置器专业版****DPA**项目的**输入文件助手**中。该格式允许 **OEM 在** **EcuC 级别提供特定于项目的 EcuC** 元素和诊断数据预配置。

 

![img](https://support.vector.com/sys_attachment.do?sys_id=ebe28f87db65a41c4896115e68961994)

 

导入的参数将以只读方式处理。请注意，在导入 **PSC** 文件之前，**PSC** 文件会覆盖用户配置的基本**软件模块**的已存在数据或从输入文件派生的数据。

 

**合并 PSC** 文件：

**PSC** 文件是“可拆分的”，这意味着如果格式符合 AUTOSAR 拆分规则，则可以与其他 **PSC** 文件合并。

### 注意：

不允许在多个 ARXML 文件中导入一个 **BSW 模块**的 **PSC**。**DaVinci配置器**将在**输入文件助手**中显示以下错误：

 

 ![img](https://support.vector.com/sys_attachment.do?sys_id=3be2cf87db65a41c4896115e68961970)

 

**PSC** 文件中的**电子控制**模块 CAN（“可以”）示例：

![img](https://support.vector.com/sys_attachment.do?sys_id=3ad3074fdb65a41c4896115e68961936)

 

### 限制：

SIP 必须提供与 **PSC** 文件模型匹配的 **BSW** 定义 [数据结构]。

在**达芬奇配置器专业版**中，可以导出一个或多个**BSW模块的****EcuC**。导出的文件可以作为**PSC**文件导入**到达芬奇配置器专业版**中。

在现有的 **DPA** 项目中

- 转到**文件|出口**，
  ![img](https://support.vector.com/sys_attachment.do?sys_id=a3e203c7db65a41c4896115e689619d6)
- 选择要导出的**模块配置**。
  ![img](https://support.vector.com/sys_attachment.do?sys_id=ebe203c7db65a41c4896115e6896198b)

**PSC** 可以作为输入文件导入到另一个 **DPA** 项目中：

![img](https://support.vector.com/sys_attachment.do?sys_id=7fe2cf87db65a41c4896115e68961997)