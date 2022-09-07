## 达芬奇配置器专业版中的 EcuC 文件参考

### 问题：

如何在**达芬奇配置器专业**版中使用**EcuC文件参考**？

### 答：

假设**我们有用例来**重新利用其他 **DPA** 项目的特定 **MCAL 配置**。

![img](https://support.vector.com/sys_attachment.do?sys_id=cf3a0fcbdbe5a41c4896115e6896198f)

**截流**器文件引用是一个 ARXML 文件，其中包含 **BSW 模块**的**截断器配置**。此机制的目的是允许在**达芬奇配置器专业**版中的**DPA**项目之间共享数据。**EcuC** 文件引用以只读方式加载到现有的 **DPA** 项目中。

文件引用只能由其源修改，并且修改由加载 ARXML 文件的 **DPA** 项目自动同步。无法通过**输入文件助手**导入这些文件。这只能通过菜单**添加ECUC文件参考...**或删除**ECUC文件参考...来完成**。

 

![img](https://support.vector.com/sys_attachment.do?sys_id=df3acbcbdbe5a41c4896115e68961922)

 

### **计算机与计算机辅助控制文件参考**

**PSC** 和外部 ECU 文件之间的区别在于，**PSC** 允许 **OEM 在** **EcuC 级别提供特定于项目的 EcuC** 元素和**诊断数据**预配置，而文件引用允许在 DPA 项目中重用 EcuC 元素。

有关文件参考的更多信息，请参阅《用户手册》第 5.2 章 **DaVinci 团队和平台支持**，该章节位于 **DaVinci 开发人员**安装路径或 SIP 文件夹中。`/Docs/``\external\Doc\UserManuals`