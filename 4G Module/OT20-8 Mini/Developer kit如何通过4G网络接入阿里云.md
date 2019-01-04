![](https://i.imgur.com/Q8Jcei1.png)

# Developer kit如何通过4G网络接入阿里云

----------
## 目录

- [概要](#概要)
- [准备条件](#准备条件)
- [云端配置](#云端配置)
	- [登录物联网平台](#登录物联网平台)
	- [创建产品](#创建产品)
	- [添加设备](#添加设备)
	- [得到四元组信息](#得到四元组信息)
- [设备端配置](#设备端配置)
	- [Developerkit配置](#Developerkit配置)
		- [代码修改](#代码修改)
		- [编译目标](#编译目标)
		- [烧录固件](#烧录固件)
- [连接阿里云](#连接阿里云)

----------
## 概要

　　本文将以 **Developerkit + OT20-8** 为例，教大家如何基于MQTT，在Developerkit上使用4G网络接入阿里云物联网平台。（亦可使用Linkkit组件接入智能生活开放平台和LinkDevelop平台，这里暂只以物联网平台为例）

　　**OT20-8**属于OT20系列，是由[上海诺行信息技术有限公司](www.notioni.com)设计开发的一款全网通4G模块，广泛应用于需要高速传输的产品中，和诸多IOT设备应用场景。

　　**物联网平台**是阿里云针对物联网领域开发人员推出的一款设备管理平台。首次使用请先注册阿里云账号，并开通物联网平台服务。

 - 物联网平台地址：https://iot.console.aliyun.com
 - 物联网平台介绍：https://help.aliyun.com/product/30520.html?spm=5176.11485173.0.0.25ef59afTH0thn

## 准备条件
1. Developerkit V13 x1（甄别方式：开发板右下角丝印的有A20V1.3）
2. OT20-8 x1
3. SIM卡 x1 （这里使用的是移动SIM卡）
4. 天线 x2
5. 螺丝及螺丝刀（用于将模组固定在Developerkit上）
![](https://i.imgur.com/udRluBl.jpg)

## 云端配置

### 登录物联网平台
　　可直接访问　https://iot.console.aliyun.com ，使用已有账号登录

　　或 访问 https://www.aliyun.com ，登录后，进入控制台，搜索“物联网平台”

![](https://i.imgur.com/MMtnfco.png)

![](https://i.imgur.com/Wk1QLFT.png)

### 创建产品
　　展开“设备管理”，选择“产品”，在右侧页面点击“创建产品”

![](https://i.imgur.com/vqSN8aJ.png)

　　可选择“基础版”和“高级版”，这里我们以“基础版”为例。

![](https://i.imgur.com/sNAxuDK.png)

　　下一步，填写产品信息。必填产品名称和节点类型。

![](https://i.imgur.com/7WqOl1z.png)

### 添加设备
　　产品创建成功后，点击“查看”进入产品详情页。

![](https://i.imgur.com/2fpGtuw.png)

　　点击“前往管理”，可添加测试设备

![](https://i.imgur.com/5EYVjLG.png)

![](https://i.imgur.com/GC1Lnz6.png)

### 得到四元组信息
　　设备创建成功后，我们将得到接入云端所必需的设备证书，**product_key、device_name、device_secret、product_secret**。

![](https://i.imgur.com/NsyniLj.png)

注：**product_secret**的信息可返回 产品管理-产品详情页 查看

![](https://i.imgur.com/8n9pfiN.png)

　　记录下这组信息，称其为“四元组”，后面将会用到。

![](https://i.imgur.com/juhU0Lk.png)

　　此时可看到新建设备的状态为“未激活”。

![](https://i.imgur.com/pRRTq5y.png)

　　到这里云端配置已完成，接下来开始设备端的配置。

## 设备端配置
　　首先，请将准备好的Developerkit和OT20-8组装起来：

1. 将SIM卡插入Developerkit正面的SIM卡槽中
2. 再将OT20-8通过Developerkit背面的PCIE接口接入，使用螺丝固定，安装好天线
3. 务必确保Developerkit上J408处有两个跳线帽

注：模组内已预先烧录好了所需的软件，与Developerkit组装后，Developerkit烧录对应的固件即可使用。 

![](https://i.imgur.com/pRDzqDt.jpg)

![](https://i.imgur.com/1wvGJqA.jpg)

### Developerkit配置
#### 代码修改

　　从https://github.com/alibaba/AliOS-Things 获取AliOS-Things master分支代码。

　　获取方式：`git clone https://github.com/alibaba/AliOS-Things.git`

　　打开AliOS-Things代码目录，进入\board\developerkit\developerkit.mk，修改module定义为**lte.m02h**

　　`module ?= lte.m02h`

![](https://i.imgur.com/KnjTeld.png)

　　再打开\app\example\mqttapp\mqtt_example.c，将云端配置中获取到的四元组信息替换到代码中

```sh
#define PRODUCT_KEY             "a1If2unfH7o"
#define PRODUCT_SECRET          "Vfe4dqmuaBRvOntL"
#define DEVICE_NAME             "test1"
#define DEVICE_SECRET           "zMlKC13IEB8QQSgQx3EGGDEnHnJqwEoJ"
```

![](https://i.imgur.com/NzjKdCw.png)

#### 编译目标 mqttapp@developerkit

　　`aos make mqttapp@developerkit`

#### 烧录固件

 　　生成的可执行文件为\out\mqttapp@developerkit\binary\mqttapp@developerkit.bin ，将其烧录到Developerkit中。

## 连接阿里云
　　通过USB ST-LINK口给Developerkit上电，此时建议打开串口调试工具，设置波特率115200，一同观察。等待10s左右，模块注册上网络，可看到串口出现`mqtt connect success!`。

![](https://i.imgur.com/2jajoiZ.png)

　　回到云端，可以看到设备状态变为“在线”，表示设备已经成功连接上阿里云。
  
![](https://i.imgur.com/foSbMwY.png)

|  Version |          Comment        |    Author      |   Date     |
|:--------:|:-----------------------:|:--------------:|:----------:|
|    1.0   |  Document initialization| Notion-LiSijia | 2018-12-29 |



