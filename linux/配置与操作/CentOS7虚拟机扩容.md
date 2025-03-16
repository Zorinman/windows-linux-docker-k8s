在 CentOS 中，分区类型可以分为 LVM（Logical Volume Manager） 和 非 LVM（传统分区）  

非 LVM 分区是直接使用物理磁盘上的分区（如 /dev/sda1、/dev/sda2 等），每个分区独立管理  

LVM 是一种更高级的磁盘管理方式，它将物理磁盘（或分区）抽象为逻辑卷（Logical Volume），提供更灵活的存储管理  

-------
接下来分别介绍非LVM和LVM的扩展方式(下面有教你如何查看你的分区是什么类型）  

这里使用的是VMware，先关闭虚拟机，然后右键虚拟机设置调整磁盘最大大小为所需容量（注意是加上扩容后的总大小而不是扩容多少）  
![img_28](https://github.com/user-attachments/assets/5c83ed01-4b7a-46bf-b3df-796f8b4ef32a)


接下来打开虚拟机进行操作  

使用`fdisk -l` 查看磁盘分区信息，如果是非LVM看第一个，如果是linux LVM请看第二个  

# 一、fdisk -l查看磁盘分区信息，分区类型为Linux（非LVM）
可以看到/dev/sda3是Linux分区类型 因此使用以下方法扩展   如果是Linux LVM，则查看另外一种方法  
32.2G是新增分区后的总容量 ，此容量原系统卷轴容量加扩容容量  
![img_24](https://github.com/user-attachments/assets/c8569b3b-3552-4f55-bffc-48e12546e4eb)

### 1 终端输入`df -h` 查看文件系统磁盘    
**/dev/sda3挂载到根部目录/**，这就是我们需要扩容的物理卷  
![img_22](https://github.com/user-attachments/assets/73cbf883-34cc-452b-af65-8f8116bd72de)


### 2 使用growpart 工具扩展分区  
如果没有则先安装  
`sudo yum install cloud-utils-growpart`  
扩展分区 /dev/sda 3  
`sudo growpart /dev/sda 3`  

### 3 扩展文件系统  

检查/dev/sda3 的文件系统类型   
`blkid /dev/sda3`    
![img_26](https://github.com/user-attachments/assets/4adc6cd3-828c-4cc0-a427-49c3525cc8ba)
   
如果是 XFS 文件系统：xfs_growfs / （直接对挂载点/操作（需已挂载）自动将根分区扩展到新增的磁盘空间‌）  
如果是 ext3/ext4 文件系统：resize2fs /dev/sda3  （在线扩展无需卸载分区‌）  

**我们这里是XFS且/dev/sda3挂载到了根目录/所以执行 `sudo xfs_growfs / `**  
（貌似xfs_growfs /dev/sda3也可以）

`df -h` 检查磁盘空间 容量增加扩容成功  
**关键步骤总结**

| 步骤           | 命令/工具          | 作用                            |
|----------------|-------------------|---------------------------------|
| 扩展物理磁盘   | VMware/VirtualBox设置 | 增加虚拟机磁盘总容量          |
| 扩展分区       | growpart           | 将新增空间合并到现有分区      |
| 调整文件系统   | resize2fs 或 xfs_growfs | 扩展文件系统以占用新空间      |

-----
# 二、fdisk -l查看磁盘分区信息，扩展分区类型为Linux LVM （LVM类型）
⭐/dev/sda3是Linux分区类型  因为Linux的扩展方法我们已经介绍，**这里我们假设是Linux LVM**，  
⭐**这里我们实际上是/dev/sda3非LVM，但一般LVM都名称为/dev/<卷组名>/<逻辑卷名>  这里需要特别提醒**

32.2G是新增分区后的总容量 ，此容量原系统卷轴容量加扩容容量  
![img_24](https://github.com/user-attachments/assets/c8569b3b-3552-4f55-bffc-48e12546e4eb) 

### 1 终端输入`df -h` 查看文件系统磁盘    
/dev/sda3挂载到根部目录/，这就是我们需要扩容的物理卷  
![img_22](https://github.com/user-attachments/assets/73cbf883-34cc-452b-af65-8f8116bd72de)  

### 2 使用growpart 工具扩展分区  
如果没有则先安装  
`sudo yum install cloud-utils-growpart`  
扩展分区 /dev/sda 3  
`sudo growpart /dev/sda 3`  


### 3 扩展LVM
**对LVM的PV,VG,LV进行扩容**  
```
物理卷（Physical Volume, PV）：物理磁盘或分区，被 LVM 管理。

卷组（Volume Group, VG）：由一个或多个物理卷组成，是逻辑卷的存储池。

逻辑卷（Logical Volume, LV）：在卷组上创建的虚拟分区，可以被格式化和挂载。

物理扩展（Physical Extent, PE）：LVM 管理的最小存储单元，默认为 4MB
```
**更新 LVM 中物理卷PV的大小将新增的空间加入 LVM 管理（该命令会根据底层物理设备（ /dev/sda3）的实际大小，更新 LVM 中物理卷的大小信息）**  

`pvresize /dev/sda3`  

**查看卷和卷组以及Free  PE / Size(剩余未使用的磁盘空间)**   
`vgdisplay`  
可以看到有10G未被使用
![img_20](https://github.com/user-attachments/assets/58b828c0-0fe6-47d9-819a-6e3c0ad0fa2e)


**接下来进行逻辑卷LV扩展**   
把Free  PE中的所有未使用容量（100%）扩展到现有逻辑卷
`lvextend -l +100%FREE /dev/sda3`


###  4 扩展文件系统
检查/dev/sda3 的文件系统类型   
`blkid /dev/sda3`    
![img_26](https://github.com/user-attachments/assets/4c05e43f-ffe8-43c6-87d8-9d6e04f08843)
  
如果是 XFS 文件系统：xfs_growfs / （直接对挂载点/操作（需已挂载）自动将根分区扩展到新增的磁盘空间‌）  
如果是 ext3/ext4 文件系统：resize2fs /dev/sda3  （在线扩展无需卸载分区‌）  

**我们这里是XFS且/dev/sda3挂载到了根目录/所以执行 `sudo xfs_growfs / `**  
（貌似xfs_growfs /dev/sda3也可以）  
`df -h` 检查磁盘空间 容量增加扩容成功  

**关键步骤总结**

| 步骤    | 命令/工具          | 作用           |
|-------|-------------------|--------------|
| 扩展物理磁盘 | VMware/VirtualBox设置 | 增加虚拟机磁盘总容量   |
| 扩展分区  | growpart           | 将新增空间合并到现有分区 |
| 扩展LVM | lvextend          | 将新增空间扩展到LVM  |
| 调整文件系统 | resize2fs 或 xfs_growfs | 扩展文件系统以占用新空间 |

