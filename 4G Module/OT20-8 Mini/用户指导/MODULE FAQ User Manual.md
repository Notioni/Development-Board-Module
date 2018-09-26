![](https://i.imgur.com/Q8Jcei1.png)

# MODULE FAQ User Manual
---


## 一．环境安装
### 1. 如何在 Linux 环境下开发我们的模块功能？
1)：先在 XP/WIN7 电脑上，安装“MBB_DRIVER_150529.rar”的驱动，用超级终端或者SecureCRT工具，输入"at+mifiusbmode=w,4"命令将模块切换成GENERIC USB MODE模式。  
![](https://i.imgur.com/hkPcenq.png)

2)：在 Ubuntu 或者其它 Linux 环境下，安装 minicom（2.7 版本)。  
3)：按照《诺行 4G 模块在 LINUX PC 下的模块基本功能验证和 LINUX 驱动移植简介.pdf》这个文档进行操作。

## 二．Band 相关
### 1.	查询Band指令" AT*BAND? "返回值在文档中并无对应值，具体见下图：
![](https://i.imgur.com/wfSKpTR.png)

![](https://i.imgur.com/DgSU7W9.png)
 无 10
 

![](https://i.imgur.com/JtxglDg.png)
无 131

![](https://i.imgur.com/aQP6GhX.png)
无 480

![](https://i.imgur.com/rvELoI2.png)
无 524485

请问该如何计算当前Band值？  
那几个值是通过求和的出来的。  
比如第4个为480，即 480=32+64+128+256，即 TDLTE band38，band39，band40，band41
其余没列出来的， 比如LTEbandL中，2即表示 FDDLTE_BAND_2 ，16即表示FDDLTE_BAND_5，依次类推。
 
### 2.	如何查询 MIFI/模块 当前连接的 Band 和 Channel

![](https://i.imgur.com/HlORG9F.png)


可以使用如下指令：  
AT+EEMOPT=1  
AT+EEMGINFO?  

执行后会返回：  
+EEMLTESVC：\<mcc>, \<length of mnc>, \<mnc>, \<tac>, \<PCI>, <font color=#FF0000 >\<dlEuarfcn>,\<ulEuarfcn >, \<band></font>, \<dlBandwidth>, \<ci>,\<rsrp>,\<rsrq>, \<sinr>, \<MainRsrp>, \<DiversityRsrp>,\<MainRsrq>, \<DiversityRsrq>, \<rssi>, \<cqi>,\<ErrorModeState>, \<emmState>, \<serviceState>,\<IsSingleEmmRejectCause>,\<EMMRejectCause>, \<MmeGroupId>, \<MmeCode>, \<mTmsi>  
其中，<font color=#FF0000 >\<band></font>就是当前的Band，<font color=#FF0000 >\<dlEuarfcn>, \< ulEuarfcn ></font>就是当前的Channel number.  
<font color=#FF0000 ><ci></font>对应 minilog 中的 physical cellid.
 
![](https://i.imgur.com/PvRwDhW.png) 

![](https://i.imgur.com/VXe8GmB.png)
 
比如 minilog 中 Channel 为 1650，则应该是Band3。

### 3.	关于TDS下锁band只锁定在34或39的问题
1）按照CMCC的TDS网络的规范要求，是不可以锁定 B34 或者 B39 的. 协议要求必须同时支持 B34 和 B39  
2）现在发布的所有版本，按照CMCC的要求，默认已经都是同时支持 B34 和 B39 了，不需要额外发送命令  
3）TDS模式下在同时支持band34 和band39 的情况下，两个Band 搜索的优先级是先搜Band34,再搜Band39

### 4.	客户反馈有2个模块，一个模块可以正常拨号上网，另外一个at+cgdcont?返回 error。
1.	首先怀疑网络问题：插上SIM卡没有？天线有没有插好？当地小区信号不好？插入的 sim 卡有问题？ SIM 卡欠费被停机等原因。将2个模块的SIM卡互换，看现象跟板子走还是跟SIM卡走，排除SIM卡的问题。
2.	再通过AT+CEREG?和AT+CGREG?查询网络注册状态
3.	再通过at*band?和 at*bandind?查询 band 支持情况。
后来客户发现异常板子 at*band?查询出来的优先级（1）跟正常板子（11）不一样，将其改成跟正常板子一样就 OK了。  
![](https://i.imgur.com/cvclZ4u.png)

## 三．频点信息以及 PIN 码和锁网信息
### 1. 客户碰到的问题：当前频点查询时不上报物理小区 id 等信息
软件上要确保 3 点：  
1）at_client_init(AT_CLIENT_USB_MODEM, TRUE);  这样 IND 才会起作用！！  
2）把那个定制宏 NOTION_ATCMD 给去掉。因为这个里面有几个主动发 REQ 的命令，可能会导致某些命令的 IND 返回数据！！  
3）processEMLteINFOSVCInfoInd()中那个 resp 要恢复成打开  
操作时要确保：   
1）AT+EEMOPT=1 和 AT+EEMGINFO?  
2）AT*bandind?要确保返回注册成LTE的

 
这样才能满足要求：开机没有主动上报的 EEMLTESVC；手动输入后才有。

### 2.	客户提到文档中+EEMLTESVC 主动上报的信息个数与实际不一致。文档中只有 26 个，但实际返回 27 个。  
这是由于文档有误。按照代码应该有27个。

+EEMLTESVC: \<mcc>, \<length of mnc>, \<mnc>, \<tac>, \<ci>, \<dlEuarfcn>, \< ulEuarfcn >,
\<band>, \<dlBandwidth>,\<cellId>,\<rsrp>,<rsrq>,\<sinr>,\<MainRsrp>,\<DiversityRsrp>,
\<MainRsrq>,\<DiversityRsrq>,\<rssi>,\<cqi>,\<ErrorModeState>,\<emmState>,\<serviceState>,\<IsSingleEmmRejectCause>,\<EMMRejectCause>,\<MmeGroupId>,\<MmeCode>,\<mTmsi>  

如下截图所示：
  
CEREG 的 tac=0x7025=28709,cellID=0x06c4a101=113549569  
刚好对应EEMLTESVC主动上报的\<tac>和\<cellId>   
\<dlEuarfcn>, \< ulEuarfcn >, \<band>对应 minilog 中的 channel 和 band  
<ci>对应 minilog 中的 physical cellid 
 
![](https://i.imgur.com/ErdUdEv.png)

  
<font color=#FF0000 >另外，对于 3G TD-SCDMA，主动上报的信息如下：</font>  

>AT+EEMOPT=1  
 OK AT+EEMGINFO?  
 +EEMGINFO : 3, 1 OK
 +EEMUMTSSVC:3, 1, 1, 0,-68, -32768, -99, -32768, 0, 1, 1120, 0, 57611, 113552949, 4459, 78,10080, 42, 0, 0, 1, 64, 64, 64,  0, 0,	<font color=#0000FF >///// P16 对应catstudio中cellParameterId，P17对应catstudio 中的ARFCN</font>  
 +EEMUMTSINTER: 0, -96, -32768, -99, 65535, 65535, 65534, 1, 10088, 110 <font color=#0000FF >// <p9>arfcn <p10> cellParameterId</font>  
 +EEMUMTSINTER: 1, -103, -32768, -99, 65535, 65535, 65534, 2, 10112, 122  
 +EEMUMTSINTER: 2, -90, -32768, -99, 65535, 65535, 65534, 3, 10096, 90

<font color=#FF0000 >对于 2G GSM，主动上报信息如下：</font>  
>AT+CGREG?  
 +CGREG: 3,1  
 OK AT^SYSINFO  
 ^SYSINFO: 2,3,0,3,1,3 OK  
 AT+EEMOPT=1 OK AT+EEMGINFO?  
 +EEMGINFO : 0, 0 OK  
 +EEMGINFOBASIC: 0  
 +EEMGINFOSVC: 1120,0,28709,54591,2,0,<font color=#0000FF >21</font>,32,142,0,7,42,38,38,7,0,79,0,8,4,1,1,<font color=#0000FF >546</font>,2,0,142, 120,30,0,1,1,32,0,0,0,0  <font color=#0000FF >////// P7<bsic>的21对应catstudio的Bsic P23<arfcn>的546对应CATSTUDIO 的 BCCH</font>   
 +EEMGINFOPS: 0, 2, 1, 0, 37, 0, 8, 255, 4, 0, 0, 0, 0, 0, 0, 5, 1, 0, 0, 0, 1, 0, 0  
 +EEMGINFONC: 0, 1120, 0, 28709, 0, 44592,40, 40, 30, 134, 79, 0, 134  
 +EEMGINFONC: 1, 1120, 0, 28709, 0, 44593,38, 48, 28, 134, 88, 0, 134  
 +EEMGINFONC: 2, 1120, 0, 28709, 0, 54593,36, 19, 26, 136, 536, 0, 136  
 +EEMGINFONC: 3, 1120, 0, 28688, 0, 44915,34, 20, 24, 134, 556, 0, 134  
 +EEMGINFONC: 4, 1120, 0, 28688, 0, 44916,32, 56, 22, 126, 518, 0, 126  
 +EEMGINFONC: 5, 1120, 0, 28709, 0, 0,23, 39, 13, 123, 520, 0, 123  
 +EEMGINBFTM: 1, 1120, 0, 28709, 54591, 21, 32, 142, 0, 7, 42, 38, 38,7, 0, 79, 0, 8, 4  

### 3.	如何使用 celllock 锁小区和锁频点
以 LTE 为例, 锁定频点的 at 命令顺序:  
1. AT\*CellLock=1,2,38350（38350 为公网频点号）  
2. AT+CFUN=0  
3. AT+CFUN=1  
锁定 cell 的 at 命令顺序：
1. AT\*CellLock=2,2,38350,286（286 为公网 38350 信号的 cell ID）  
2. AT+CFUN=0  
3. AT+CFUN=1  
针对代码总结如下：  
AT*CellLock=<mode>[,<network_mode> [,<arfcn>[,<scrambling_code>]]]  
mode 取值为 0（disable）,1(lock freq), 2(lock cell), 3(IRAT)  
network_mode 取值为 0（GSM), 1(UMTS), 2(LTE)  
arfcn 取值为 0---41589  
scrambling_code 取值为 0---503    
其中如果 mode 为 1 即 lock freq  
当 network_mode=0 即 GSM 时，会报参数无效；  
当 network_mode=1 即 UMTS 时，arfcn 范围是【10054，10121】和【9404,9596】；  
当 network_mode=2 即 LTE 时，arfcn 范围是【0,41589】；

如果 mode 为 2 即 lock cell    
当 network_mode=0 即 GSM 时，会报参数无效；  
当 network_mode=1 即 UMTS 时，arfcn 范围是【10054，10121】和【9404,9596】；    
scrambling_code 范围是【0,127】；    
当 network_mode=2 即 LTE 时，arfcn 范围是【0,41589】，scrambling_code 范围是【0,503】；  

以做的实验为例说明如下：  
>AT+EEMOPT=1  
OK  
AT+EEMGINFO?  
+EEMGINFO : 3, 2  
OK
+EEMLTESVC:1120,2,0,28688,162,37900,37900,38,5,112724484,47,20,12,46,36,28,7,59,0,1,10,0,0, 65535,804,146,-1064174437 <font color=#0000FF >// 查询出来目前physical cellid 为162，band为38，ARFCN为37900</font>   
+EEMLTEINTER: 0, 134, 38400, 43, 21  
+EEMLTEINTER: 1, 250, 39250, 18, 12  
+EEMLTEINTER: 2, 253, 39250, 23, 21	<font color=#0000FF >// 查询出来 cellid 为 253， ARFCN 为 39250</font>  
+EEMLTEINTRA: 0, 283, 37900, 28, 0	<font color=#0000FF >// 查询出来 cellid 为 283， ARFCN 为 37900</font>  
at*celllock=1,2,38400	<font color=#0000FF >// 锁频点 38400</font>  
OK  
AT+CFUN=0	<font color=#0000FF >// detache</font>  
+CGEV: EPS DED DEACT 1,5  
^SRVST: 2  
+CREG: 1  
+CEREG: 0  
+CGEV: ME DETACH  
^SIMST: 0  
+MMSG: 1, 0  
+CPIN: SIM REMOVED  
^CARDMODE: 255  
^SIMST: 255,0  
OK  
^SRVST: 0  
+CREG: 0  
AT+CFUN=1	<font color=#0000FF >// attach</font>  
^SRVST: 0  
+CREG: 2  
+CEREG: 2  
^SRVST: 1  
+CREG: 11  
+CEREG: 11,"7010","06b80a01",7  
OK  
ICCID: 898600a1179407112134  
^SIMST: 0    
+MSTK:11,D0260103022500020281820509804E2D56FD79FB52A80F104E80005500530049004D53614FE1606F  
+CPIN: READY  
^SIMST: 1  
^SRVST: 2  
+CREG: 1  
+CEREG: 1,"7010","06b80a01",7  
+CGEV: EPS PDN ACT 5  
+NITZ: 15/06/23,11:06:50+32,0  
+MSTK: 14  
+CPIN: READY  
^SIMST: 1  
+EEMLTESVC:1120, 2, 0, 28688, 134, 38400, 38400, 39, 5, 112724481, 47, 19, 13, 255, 255,
255, 255, 255, 65535, 2, 10, 0, 0, 65535, 804, 146, -1063846767	<font color=#0000FF >// 查询出来确实 physicalcellid 为 134， ARFCN 位 38400， band39</font>  
+EEMLTESVC:1120, 2, 0, 28688, 134, 38400, 38400, 39, 5, 112724481, 46, 19, 14, 46, 19, 27,
127, 60, 0, 2, 10, 0, 0, 65535, 804, 146, -1063846767  
+MMSG: 2, 0  
+MPBK: 1  
+MMSG: 3, 0  
+MMSG: 0, 0  

### 4.	如何查询 PIN 码信息？
查询状态可以用 at+cpin? 或者 at+clck="SC",2  
解锁 PIN 码可以用 AT+CPIN=1234  
1	PIN  开启 at+clck="SC",1,1234	返回+CLCK: 1  
2	PIN  关闭 at+clck="SC",0,1234	返回+CLCK: 0  

### 5.	锁网以及锁子网的 查询、锁、解锁
><font color=#0000FF >/////////////// 锁网 开始 ///////////////</font>  
at+clck="PN",1  
OK
+CPIN: PH-NET PIN  
^CARDMODE: 255  
^SIMST: 255,1  
at+cpin?  
+CPIN: PH-NET PIN  
OK  
at+clck="PN",0,123456	<font color=#0000FF >//解锁网</font>  
at+clck="PN",2	<font color=#0000FF >//  查询状态	返回 1 为 active	0 为 not active</font>    
*CLCK: 1    
OK  
<font color=#0000FF >/////////////// 锁网 结束 ///////////////  
/////////////// 锁子网 开始（必须要软件支持锁子网） /////////////// </font>  
at+clck="PU",1	<font color=#0000FF >// 锁子网</font>  
OK  
at+clck="PU",2	<font color=#0000FF >//  查询状态返回 1 为 active	0 为 not active</font>  
*CLCK: 0  
OK  
at+cpin?      
+CPIN: PH-NETSUB PIN  
OK  
at+clck="PU",0,123456	<font color=#0000FF >// 解锁子网</font>  
OK  
at+cpin?  
+CPIN: READY  
OK  
<font color=#0000FF >/////////////// 锁子网 结束 ///////////////</font>


## 四．IPV4，IPV6，IPV4V6
### 1.	How to changed the APN?
If you want to change the APN for 4G network, you should follow this sequnce.  
><font color=#0000FF >//change default bearer to IPV6
AT+CFUN=4   
AT*CGDFLT= 1,<PDP_type>,<APN>，1  
AT+CFUN=1   
   
>//query whether LTE is registed AT+CEREG? 
   
>//IF LTE registed, then attach.  
AT+CGDCONT? 
   
>AT*CGDFLT = <mode>,[<PDP_type>,[<APN>],<etiflag>]  
<mode>: integer type; indicates whether saved to NVM.   
         0 - not save to NVM   
         1 - save to NUM   
<PDP_type>: string type; specifies the type of packet data protocol   
         1 IP Internet Protocol (IETF STD 5)   
         2 IPV6 Internet Protocol, version 6 (IETF RFC 2460)  
         3 IPV4V6 Virtual <PDP_type> introduced to handle dual IP stack UE capability. (See 3GPP TS 24.301 [83])   
\<APN>: string type; used to select the GGSN or the external packet data network.</font>     
\<etiflag>: integer type;it must be set 1.
  
 
And it is really two questions:
1st in the real network, 2nd in the simulation environment just like in cmw500 or mt8820c.
1st in the real network: the APN name is very important,and the really network whether accept the APN name that you are modified is a question.
2nd in the simulation environment just like in cmw500 or mt8820c: you can reference the attachment. 

### 2.	模块支持 IPV6 吗？需要怎么测试？是否支持双栈？
模块是支持 IPV6 的，但是大陆实网不支持 IPV6 单栈。  
如果想验证的话，可以用 CMW500 这样的仪器，插白卡测试在仪器下，IPV4，IPV6，IPV4/IPV6 都可以。  
对于模块，前缀的获取可以通过 CGDCONT 来获取，但是 IPV6 的地址构成需要在 AP 端通过 RS&RA 流程来完成。  

### 3.	4G 以及 3G 下如何设置 IPV6？
1）LTE 修改默认的那个 APN 流程如下(中国实网下不行，仪器模拟可以)
><font color=#0000FF >//修改 default bearer为IPV6  
AT+CFUN=4  
AT*CGDFLT= 1,2,<APN>,1  
AT+CFUN=1  
//查询是否注册上 LTE  
AT+CEREG?  
//注册上 LTE 后，执行下面命令，将 IP 设置到网卡。  
AT+CGDCONT?  

>AT*CGDFLT = <mode>,[<PDP_type>,[<APN>],<etiflag>]  
<mode>: integer type; indicates whether saved to NVM.  
    0 - not save to NVM
    1 - save to NUM
<PDP_type>: string type; specifies the type of packet data protocol  
    1	IP	Internet Protocol (IETF STD 5)  
    2	IPV6	Internet Protocol, version 6 (IETF RFC 2460)  
    3	IPV4V6	Virtual <PDP_type> introduced to handle dual IP stack UE capability. (See 3GPP TS 24.301 [83])  
<APN>: string type; used to select the GGSN or the external packet data network.
<etiflag>: integer type;it must be set to 1.</font>
 
2）3G TDSCDMA 模拟网络下设置 APN 并使用 IPV6 拨号
><font color=#0000FF >AT+CGREG?  
+CGREG: 3,1,"0039","000100e7",6,2  
OK  
AT+CGREG?  
+CGREG: 3,1,"0039","000100e7",6,2  
OK  
AT+CGREG?  
+CGREG: 3,1,"0039","000100e7",6,2  
OK  
at+cfun=0  
AT+CGDCONT=5,"IPV6","9cmri.com"   
OK   
at+cfun=1  
AT+CGDATA="",5  
CONNECT  
+CGEV: ME ACT ""IPV6"", "32.1.14.128.194.16.0.1.0.2.0.0.151.27.15.174",5  
AT+CGDCONT?  
+CGDCONT:5,"IPV6","9cmri.com","32.1.14.128.194.16.0.1.0.2.0.0.151.27.15.174",,, 00031020014860486000000000000000008888  
OK</font>

3G 的流程建议流程改成下面试试：
><font color=#0000FF >AT+CGREG?  
at+cfun=0  
AT+CGDCONT=5,"IPV6","9cmri.com"	----》这个地方可以改 IPV4,IPV6 或者IPV4V6，还可以改APN的名字  
at+cfun=1  
AT+CGREG?	----->这个最好确保状态为 1 时再进行下面步骤    
AT+CGDATA="",5  
AT+CGDCONT?   
AT+CGCONTRDP=5   
AT+GETIP=5</font>

## 五．信号质量
### 1. 模块可以正常上网，但信号强度显示异常，一直是99，请问是需要其他指令吗？
![](https://i.imgur.com/1EOILyl.png)


>AT+CESQ
+CESQ:\<rxlev>,\<ber>,\<rscp>,\<ecno>,\<rsrq>,\<rsrp>

其中：
对于GSM：\<rxlev>,\<ber>	接收信号电平（Received Signal Level）和误码率（Bit Error Rate）  
对于 3G：\<rscp>,\<ecno>	接收信号码功率（Received Signal Code Power dBm）码片与总功率之比（ratio of the received energy per PN chip to the total received power spectral density）  
对于4G：\<rsrq>,\<rsrp>	参考信号接收质量（Reference Signal Receiving Quality）参考信号接收功率(Reference Signal Receiving Power dBm)   
另外，根据中移测试要求|：  
极好点： RSRP>-85dBm； SINR>25  
好点：	RSRP=-85～-95dBm；SINR:16-25  
中点：	RSRP=-95～-105dBm；SINR:11-15  
差点：	RSRP=-105～-115dBm；SIN:3-10  
极差点： RSRP<-115dB;SINR<3  

## 六．杂项
### 1.	模块是否支持 PPP 拨号呢？有拨号脚本提供参考吗？
平台目前是不支持的。

### 2.	LTE 模块在和 SD 卡配合工作，它们之间的传输数据频率情况？它们之间传输数据时速率有多高？
我们的模块支持 sd3.0 协议，理论速率可达 100M/s, 及时传输速率需要根据实际进行的数据业务进行评估。

### 3.	是否支持 DM 短信自注册？
模块的 DM 短信自注册应该是在 AP 端完成的。 
另外DM功能概述：  
目前我们所说的 DM 测试，只是测试终端在开启 DM 功能下，第一次开机会向移动特定的一个中心号码发送一条特定格式信息。该信息包括该终端的 IMEI 号、硬件版本号、软件版本号、厂商名（此厂商名需要在移动集团网注册）。移动网络端收到有效的信息后，会向终端回复一条信息，终端收到该信息不做任何处理直接丢掉。DM 流程结束。  
获取信息的 AT 命令如下：  
1）AT+CGMI 给出模块厂商的标识。    
2）AT+CGMM 获得模块标识。这个命令用来得到支持的频带（GSM 900，DCS 1800或 PCS 1900）。当模块有多频带时，回应可能是不同频带的结合。    
3）AT+CGMR 获得改订的软件版本。  
4）AT+CGSN 获得 GSM 模块的 IMEI（国际移动设备标识）序列号。  
5）AT+CSCS 选择 TE 特征设定。这个命令报告 TE 用的是哪个状态设定上的 ME。ME于是可以转换每一个输入的或显示的字母。这个是用来发送、读取或者撰写短信。  
6）AT+WPCS 设定电话簿状态。这个特殊的命令报告通过 TE 电话簿所用的状态的ME。ME 于是可以转换每一个输入的或者显示的字符串字母。这个用来读或者写电话簿的入口。                                                                           7）AT+CIMI 获得 IMSI。这命令用来读取或者识别 SIM 卡的 IMSI（国际移动签署者标识）。在读取 IMSI 之前应该先输入 PIN（如果需要 PIN 的话）。  
8）AT+CCID 获得 SIM 卡的标识。这个命令使模块读取 SIM 卡上的 EF-CCID 文件。  

### 4.	是否支持在发起业务时默认填写参数为 Subscribed,并接受网络配置的 Qos 参数？ 是否支持具备业务与底层间 Qos 参数传送机制，数据类终端底层可以接收上层业务指派的具体 Qos 需求，并上传给网络？
支持，用 AT+CGQREQ，并且用它操作结果。

>at+cgqreq?   
CGQREQ: 	1,0,0,0,  
0,0  
CGQREQ: 	2,0,0,0,  
0,0  	
CGQREQ: 	3,0,0,0,  
0,0  
CGQREQ: 	4,0,0,0,  
0,0   
CGQREQ: 	5,3,1,3,  	   ------》默认查出来是这个值  
9,31  
CGQREQ: 	6,0,0,0,  
0,0  
CGQREQ: 	7,0,0,0,  
0,0  
CGQREQ: 	8,0,0,0,  
0,0  
CGQREQ: 	9,0,0,0,  
0,0  
CGQREQ: 	10,0,0,0,  
0,0  
 
>CGQREQ: 11,0,0,0, 0,0   
CGQREQ: 12,0,0,0, 0,0   
CGQREQ: 13,0,0,0, 0,0   
CGQREQ: 14,0,0,0, 0,0   
CGQREQ: 15,0,0,0, 0,0   
OK   
at+cgqreq=5,3,3,3,3,3     ----》进行修改OK   
at+cgqreq?   
CGQREQ: 1,0,0,0, 0,0   
CGQREQ: 2,0,0,0, 0,0   
CGQREQ: 3,0,0,0, 0,0   
CGQREQ: 4,0,0,0, 0,0 
CGQREQ:  5,3,3,3,       ----》修改后查询确实变了3,3   
CGQREQ: 6,0,0,0, 0,0   
CGQREQ: 7,0,0,0, 0,0   
CGQREQ: 8,0,0,0, 0,0   
CGQREQ: 9,0,0,0, 0,0   
CGQREQ: 10,0,0,0, 0,0   
CGQREQ: 11,0,0,0, 0,0   
CGQREQ: 12,0,0,0, 0,0   
CGQREQ: 13,0,0,0, 0,0   
CGQREQ: 14,0,0,0, 0,0   
CGQREQ: 15,0,0,0,   