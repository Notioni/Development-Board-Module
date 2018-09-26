![](https://i.imgur.com/Q8Jcei1.png)

# Hardware Design Manual of OT20-8 Mini LTE Module
---


## 1. 简介 

OT20-8 系列是一款标准的 PCIe 接口的 LTE 模块，能够支持的网络模式有 TDD-LTE/FDD-LTE/UMTS/EDGE/GPRS。LTE 速率可以达到下行 150Mbps,上行 50Mbps; 可以提供高速稳定的移动宽带数据连接。丰富的射频频段以及接口可以广泛的使用在 需要高速数据传输的 CPE，无线路由器，DTU，网关，PAD 上网卡，视频监控，车载上网设 备等场景。同时亦可使用在广告机，自动售卖机，POS 机，充电桩，跑步机，按摩椅等物 联网设备上，不受运营商 2G 网络退网的影响，还能为用户提供信号更好，速度更快，基本无延迟的完美体验。

## 2. 主要参数 

![](https://i.imgur.com/JM6bfu8.png)


## 3. 接口应用介绍 

OT20-8T 硬件框图，通过如下电源、地、SIM、USB、RST 接口实现 LTE 转 USB 功能，除开始开发了框图中提到的接口，其他引脚的定义请参考表五

![](https://i.imgur.com/VGXjNxK.png)


引脚定义以及描述

![](https://i.imgur.com/ptCWbDW.png)

注意：
1. VDD_3V3的供电需要外加电容，不能大于100uF，推荐用22uF。VDD_3V3上的容值太大会给上电时序带来很大影响。  
2. 模块上电自动启动，只要连接 VDD3V3、GND、USB-DM 和 USB-DP就可以工作,其他管脚不用的话必须悬空，防止CPE上的GPIO影响模块的正常上电启动  
3. Reset（1.8V IO）管脚可以预留兼容的 1k 姆电阻连接到大板的 GPIO，默认可以不贴，注意该管脚在模块正常工作时为 1.8V 的高电平，该管脚收到一个50ms的低电平就会使模块复位并重启；  
4. 死机或异常后，硬件复位并重启模块的方案:  
   方案a. 给模块断电，然后重新上电，时间间隔需要大于 1s；  
   方案b. 给 RST 管脚一个 50-100ms 的低电平；


## 4. 射频参数 
射频参数是衡量 OT20-8T 模块的重要指标，OT20-8T模块满足 3GPP 协议标准，每片主板均经过标准测试。发射功率、接收灵敏度颗参考下表的典型值。   
### 4.1 频段支持  
![](https://i.imgur.com/I4B1zKH.png)

### 4.2 发射功率
![](https://i.imgur.com/Lq9wcXn.png)

### 4.3 灵敏度
![](https://i.imgur.com/xYcV7Vs.png)

### 4.4 数据传输率
![](https://i.imgur.com/3ZOy7mx.png)

## 5. 封装信息  
OT20-8T 采用标准的 Mini PCIe 封装，详细尺寸信息如下： 
![](https://i.imgur.com/zpefGoq.png)

## 6. 实物图
![](https://i.imgur.com/z3Nb5FC.png)

