![](https://i.imgur.com/Q8Jcei1.png)

# Notion 模块集成指导说明
---


## 1. 目的
本文档主要针对诺行模块设备基于 linux 系统的集成开发工作进行指导说明。

## 2. 范围 
本文档主要说明了在linux系统上支持诺行4G 模块设备的相关开发工作及其注意事项。


## 3. 简介 
要在 linux 系统上支持诺行 4G 模块设备，需要系统本身支持 cdc-acm 和 rndis 驱动，linux kernel 在 2.6.22 版本后已默认支持此驱动。本系统默认 Vendor ID 为 0x1286, Product ID 为 0x4e3d。


## 4. 集成步骤

### 4.1 模块功能确认 
在 kernel 版本高于 2.6.22 的 linux 操作系统上进行拨号操作，将装有模块的设备通过 usb 接入主侧设备，设备上电后，可以通过主侧 linux shell 查询 acm 设备是否加载成功，命令如下，  
![](https://i.imgur.com/PQJhDOb.png)

系统正确识别到 cdc-acm 驱动后会看到如下提示：

![](https://i.imgur.com/1urFxZg.png)

这里用于 at command 交互的口为 ttyACM1。  
接着，可以通过 linux 系统常用串口工具(例如 minicom 等)进行 at command 手动拨号验 证，如下：   
第一步，手动敲入“at”，正确提示如下：  
![](https://i.imgur.com/zd7wu14.png)

这里表示系统串口工作正常。  
第二步，手动敲入“at+cereg?”,正确提示如下：   
![](https://i.imgur.com/5s9UPPG.png)  

这里表示通信系统注册成功。
第三步，手动敲入“at+cgdcont?”,正确提示如下：  

![](https://i.imgur.com/GrCke3w.png)

此处我使用的是移动 4G sim 卡（供参考），这里表示系统附着 4G 网络成功。
同过系统命令可查看到有如下的网络设备信息：  
![](https://i.imgur.com/ubQbVEt.png) 
 
![](https://i.imgur.com/f90R19x.png)

至此，手动拨号动作完成，系统可直接通过移动 4G 网络连接 internet，诺行模块工作正常。

### 4.2	系统自动拨号
需要在系统初始化之后或者启动相关应用之前，在确认系统映射出 cdc-acm 驱动后，通过 ttyACM1 完成拨号动作，这里客户可依据各自系统实际情况进行。

### 4.3	低版本 linux 驱动移植
#### 4.3.1	USB cdc-acm 内核驱动模块集成
USB cdc-acm 驱动模块，是linux内核内置的标准驱动模块，其源文件在linux内核代码中位置为 drivers/usb/class/cdc-acm.c（可从高版本 kernel 中拷贝），在 Makefile 中添加驱动信息如下：  
![](https://i.imgur.com/vN6hGbd.png)

在Kconfig中添加驱动信息如下：  
![](https://i.imgur.com/AGkzftK.png)
 

 
编译后，生成名称为 cdc-acm.ko 的模块文件，此文件一般存在于linux 系统/lib/modules/usb/class 目录下，具体集成步骤如下：  

1.	Linux 内核编译配置时，需要选中以下配置选项(红色标注)：  
Device Drivers -> USB Support ->  
<M> Support for Host-side USB  
[*]	USB device filesystem  
<M>	EHCI HCD(USB 2.0) support  
<M>	OHCI HCD support  
<M>	UHCI HCD(most Intel and VIA) support  
<font color=#FF0000 > <M>	USB Modem(CDC ACM) support </font> 

2.	编译完成后，可以看到相关目录下的 cdc-acm.ko。

#### 4.3.2	USB rndis 内核驱动模块集成
USB rndis 驱动模块，也是 Linux 内核内置的标准驱动模块，其源文件在 linux 内核代码中位置为 drivers/net/usb/rndis_host.c（可从高版本 kernel 中拷贝），在 Makefile 中添加驱动信息如下：  
![](https://i.imgur.com/VFiwi1s.png)

在 Kconfig 中添加驱动信息如下：  
![](https://i.imgur.com/0KdBnlV.png)
 

编译后，生成名称为 rndis_host.ko 的模块文件，此文件一般存在于linux 系统/lib/modules/net/usb 目录下，具体集成步骤如下：
  
1.	Linux 内核编译配置时，需要选中以下配置选项(红色标注)  
Device Drivers -> Network device support -> USB Network Adapters ->  
  <*> Multi-purpose USB Networking Framework  
  <font color=#FF0000 > <M> Host for RNDIS and ActiveSync devices</font>

2.	编译完成后，可以看到相关目录下的 rndis_host.ko。


### 5. 附注
Notion模块系统提供四个USB接口，如下： 
![](https://i.imgur.com/6i6oO7c.png)