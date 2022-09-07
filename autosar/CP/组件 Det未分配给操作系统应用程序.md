## 组件 <Det>未分配给操作系统应用程序

### 题：

RTE 生成失败，因为 RTE 生成器引发错误
*RTE13009 - 组件未分配给操作系统应用程序*。

### 问题：

RTE 生成失败，因为 RTE 生成器引发错误
*RTE13009 - 组件未分配给操作系统应用程序*。

![img](https://support.vector.com/sys_attachment.do?sys_id=e3ee76f01bf710908e9a535c2e4bcbe9)

### 溶液：

有两种方法可以解决此问题：

1. 为 **BSW 定义一个电子分区**
   1. 打开基本编辑器并创建 **Ecuc 分区收集**子容器（如果不存在）
      ![img](https://support.vector.com/sys_attachment.do?sys_id=d00f36f01bf710908e9a535c2e4bcbba)
   2. 至少创建一个**“生态分区”
      ![img](https://support.vector.com/sys_attachment.do?sys_id=651ffeb01bf710908e9a535c2e4bcbf4)
      **
   3. 配置所需的 **ASIL** 并选择此分区以执行 BSW 模块
      ![img](https://support.vector.com/sys_attachment.do?sys_id=a32f32f01bf710908e9a535c2e4bcb16)
   4. 将 SWC 指定给此分区
      ![img](https://support.vector.com/sys_attachment.do?sys_id=e84ffe701bf710908e9a535c2e4bcba6)
   5. 将**“弹性分区”**分配给 BSW 操作系统应用程序
      ![img](https://support.vector.com/sys_attachment.do?sys_id=aa5f32f01bf710908e9a535c2e4bcb1b)
2. 将可运行的虚拟应用程序映射到操作系统应用程序
   1. 创建一个**“设置”容器（**如果该容器不存在）
      ![img](https://support.vector.com/sys_attachment.do?sys_id=687f76f01bf710908e9a535c2e4bcbfa)
   2. 创建虚拟 **DetModule** 容器
      ![img](https://support.vector.com/sys_attachment.do?sys_id=4e8f7a341bf710908e9a535c2e4bcbe1)
   3. 同步系统或 SWC 描述
      ![img](https://support.vector.com/sys_attachment.do?sys_id=e79f76f01bf710908e9a535c2e4bcbf5)
   4. 将可运行的虚拟**报告错误**映射到您的 BSW 任务
      ![img](https://support.vector.com/sys_attachment.do?sys_id=44bf7a341bf710908e9a535c2e4bcbe7)

 
 

### 背景：

将 SWC 分配给操作系统应用程序通常基于相关可运行项的映射来确定。默认情况下，Det SWC 不会定义任何可运行的。要克服这种情况，您需要定义一个可映射到操作系统应用程序的虚拟可运行项，或者您需要使用 **Ecuc 分区**将 Det SWC 显式分配给操作系统应用程序。



***



## Component <Det> Is Not Assigned to an OS Application

### Issue:

The RTE generation fails because the RTE generator raises error
*RTE13009 - Component is not assigned to an OS Application*.

![img](https://support.vector.com/sys_attachment.do?sys_id=e3ee76f01bf710908e9a535c2e4bcbe9)

### Solution:

There are two ways to solve this issue:

1. Define an

    

   EcucPartition 

   for the BSW

   1. Open the Basic Editor and create the **EcucPartitionCollection** Sub Container if it does not exist
      ![img](https://support.vector.com/sys_attachment.do?sys_id=d00f36f01bf710908e9a535c2e4bcbba)
   2. Create at least one **EcucPartition
      ![img](https://support.vector.com/sys_attachment.do?sys_id=651ffeb01bf710908e9a535c2e4bcbf4)
      **
   3. Configure the required **ASIL** and choose this partition for the BSW module execution
      ![img](https://support.vector.com/sys_attachment.do?sys_id=a32f32f01bf710908e9a535c2e4bcb16)
   4. Assign the Det SWC to this partition
      ![img](https://support.vector.com/sys_attachment.do?sys_id=e84ffe701bf710908e9a535c2e4bcba6)
   5. Assign the **EcucPartition** to the BSW Os Application
      ![img](https://support.vector.com/sys_attachment.do?sys_id=aa5f32f01bf710908e9a535c2e4bcb1b)

2. Map a dummy runnable to an Os Application

   1. Create a **DetConfigSet** container if it does not exist
      ![img](https://support.vector.com/sys_attachment.do?sys_id=687f76f01bf710908e9a535c2e4bcbfa)
   2. Create a dummy **DetModule** container
      ![img](https://support.vector.com/sys_attachment.do?sys_id=4e8f7a341bf710908e9a535c2e4bcbe1)
   3. Synchronize the System or SWC description
      ![img](https://support.vector.com/sys_attachment.do?sys_id=e79f76f01bf710908e9a535c2e4bcbf5)
   4. Map the dummy **ReportError** runnable to your BSW Task
      ![img](https://support.vector.com/sys_attachment.do?sys_id=44bf7a341bf710908e9a535c2e4bcbe7)

 
 

### Background:

The assignment of a SWC to an OS Application is usually determined based on the mapping of the related runnables. The Det SWC does not define any runnable by default. To overcome this situation, you either need to define a dummy runnable that can be mapped to an Os Application or you need to assign the Det SWC explicitly to an Os Application using an **EcucPartition**.