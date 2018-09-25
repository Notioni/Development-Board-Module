![](https://i.imgur.com/Q8Jcei1.png)

# CATStudio抓取LOG的方法
---


## 一. 双击打开 CATStudio.exe
路径：CATStudio_V2_5_1_48\CATStudio_V2_5_1_48\Exec\CATStudio.exe，打开之后会呈  现以下主界面。

![](https://i.imgur.com/aKNZlhC.png)



选择 PXA1802/1920/986/1088 LWG Online，点击 OK，进入主界面

![](https://i.imgur.com/GjyHbBX.png) 

 

## 二. 打开 Logger
1.在界面右边找到 Logger 面板  

![](https://i.imgur.com/whgiwCU.png) 



点击 Update![](https://i.imgur.com/Q1G38C7.png)，会出现以下界面
 
![](https://i.imgur.com/M0xCbsl.png)

2.导入 Database  
1) 导入 DIAG.mdb NVM.mdb   
![](https://i.imgur.com/qUst59B.png)



 
在与对应的软件包中选择 mdb 文件。如果版本不一致，则选择的数据是无效的。
![](https://i.imgur.com/wdsIP1r.png) 

 
 
2) 点击打开后，再点击右边的 Update  
![](https://i.imgur.com/Mtee50U.png)



通过上述步骤，如果 USB 连接模块后，右边 2 个指示灯中：CP 会变绿![](https://i.imgur.com/W7gAJe9.png) ，AP 灯是红 ![](https://i.imgur.com/sUXOv1W.png)；则表示导入成功。如果全部是红色，则不成功。如下图:  
![](https://i.imgur.com/2s8BJgT.png)

  
 
## 三. 保存 log
1）database 配置成功之后，主界面的 log 会一直在跑。

![](https://i.imgur.com/SYluNF3.png)

2）log 会自动保存在 CATStudio_V2_5_1_48\CATStudio_V2_5_1_48\Exec\Bin Logs 目录中。单个 log 大小默认是 50Mb，如果一次抓的 log 过多，会分成多个 log 文件保存。

![](https://i.imgur.com/WcVr0YD.png)
