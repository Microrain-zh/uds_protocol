## 达芬奇配置器专业版中的 BSW 模块导入

### 问题：

**BSW 模块**导入有什么好处？

### 答：

**BSW 模块**导入机制允许从 **DPA 项目或达芬奇配置器专业**版 **DPA** 项目中的其他第三方工具导入基本软件模块的 **ECU 配置** **（****EcuC**） 文件或**基本软件模块**的 ECU 配置。**BSW 模块**作为读写导入一次，并成为配置的一部分。

这些文件的导入通过选项**“文件|进口**

### 例：

导入 **DPA** 项目的 **BSW 模块配置**：

- **单击导入**或**按住 Ctrl+I**，
  ![img](https://support.vector.com/sys_attachment.do?sys_id=4a094f07dbe5a41c4896115e68961957)

- 选择 **DPA** 项目，
  ![img](https://support.vector.com/sys_attachment.do?sys_id=5a09cb07dbe5a41c4896115e6896193c)

- 选择一个现有的 **DPA** 文件，
  ![img](https://support.vector.com/sys_attachment.do?sys_id=1a094f07dbe5a41c4896115e689619fc)

- 在**“文件选择**”中，选择要导入的 **EcuC 模块配置**。
  ![img](https://support.vector.com/sys_attachment.do?sys_id=d6094f07dbe5a41c4896115e689619e7)

 

如果**BSW**模块已经存在，则可以使用**替换**<>**合并**机制（通过**比较和合并助手）**)

![img](https://support.vector.com/sys_attachment.do?sys_id=d2098f07dbe5a41c4896115e6896192f)

 

有关**差异&合并助手**的更多信息，请参阅位于SIP的“`\外部\文档\用户手册`”文件夹中的**DaVinci团队和平台支持**。

### 注意：

通过此机制导入 **EcuC** 模块，ARXML 文件的内容将成为配置的一部分。因此，如果需要，可以修改导入的内容。如果通过输入文件助手将**EcuC**模块作为**输入文件**导入，**DaVinci配置器**将作为**项目标准配置**（**PSC**）而不是模块导入进行处理。

有关 **PSC**的更多信息，请参阅知识库条目[项目标准配置](https://support.vector.com/kb/?id=kb_article_view&sysparm_article=KB0012875)。

请注意，模块导入无法通过更新工作流替换从输入文件派生的实际内容。特别是对于通信或诊断数据，请先通过更新工作流导入文件，然后再通过**BSW**模块导入传输手动配置。

有关 **DaVinci** 项目配置的更新工作流程的更多信息，请参阅 SIP 文件夹中的技术参考**工作流程概述**。`\external\Doc\TechincalReferences`



***



## BSW Module Import in DaVinci Configurator Pro

### Question:

What is the **BSW Module** import good for?

### Answer:

The **BSW Module** import mechanism allows to import an **Ecu Configuration** (**EcuC**) file of a **Basic Software Module** or the **Ecu Configuration** of the **Basic Software Modules** from a **DPA** project or another third party tool in a **DPA** project of **DaVinci Configurator Pro**. The **BSW Modules** are imported once as read-write and become part of the configuration.

The import of these files takes place via the option **File | Import**

### Example:

Import **BSW Module Configurations** of a **DPA** project:

- Click **Import** or **Ctrl+I**,
  ![img](https://support.vector.com/sys_attachment.do?sys_id=4a094f07dbe5a41c4896115e68961957)

- select the **DPA** project,
  ![img](https://support.vector.com/sys_attachment.do?sys_id=5a09cb07dbe5a41c4896115e6896193c)

- select an existent **DPA** file,
  ![img](https://support.vector.com/sys_attachment.do?sys_id=1a094f07dbe5a41c4896115e689619fc)

- in **File Selection** select the **EcuC Module Configuration** to import.
  ![img](https://support.vector.com/sys_attachment.do?sys_id=d6094f07dbe5a41c4896115e689619e7)

 

In case that the **BSW** module already exists it is possible to use a **Replace** <-> **Merge** mechanism (via the **Diff & Merge Assistant**)

![img](https://support.vector.com/sys_attachment.do?sys_id=d2098f07dbe5a41c4896115e6896192f)

 

For further information regarding the **Diff& Merge Assistant**, please refer to the User Manual **DaVinci Team and Platform Support** located in the folder “`\external\Doc\UserManuals`” of your SIP.

### Note:

By importing an **EcuC** module via this mechanism the content of the ARXML file becomes a part of the configuration. Therefore, the imported content can be modified if required. If an **EcuC** module is imported as input file via the **Input File Assistant**, **DaVinci Configurator** handles it as a **Project Standard Configuration** (**PSC**) and not as a module import.

For further information regarding **PSC**s please refer to the KnowledgeBase entry [Project Standard Configuration](https://support.vector.com/kb/?id=kb_article_view&sysparm_article=KB0012875).

Please be aware that a module import cannot replace the actual derived from input files via the update workflow. Especially for communication or diagnostic data use the file import via update workflow first, before transferring the manual configuration via **BSW** module import.

For further information regarding the update workflow of a **DaVinci** project configuration, please refer to the technical reference **Workflow Overview** located in the folder `\external\Doc\TechincalReferences` of your SIP.