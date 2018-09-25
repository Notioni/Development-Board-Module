![](https://i.imgur.com/Q8Jcei1.png)

# Module  AT Sequency in Dialer User Manual
---


## 1 About This Document
### 1.1 Purpose
This document describes the AT commands used in dialing and sample procedure of different case. 


## 2 AT Sequency in Dialer 
 
### 2.1 Check network registration status 
 
Before setuping data link, network registration status should be checked. Only in home network or roaming state can MIFI start dialing and setup data link.   
Cause MIFI focus on PS mainly, only AT+CGREG? (for PS in 2/3G) or AT+CEREG? (for PS in 4G) is needed. 
 
>AT+CGREG? 
+CGREG: \<n>,\<stat>[,[\<lac>],[\<ci>],[\<AcT>]  
\<stat>: integer type; indicates the GPRS registration status  
0 not registered, MT is not currently searching an operator to register to   
1 registered, home network   
2 not registered, but MT is currently trying to attach or searching an operator to register to   
3 registration denied  
4 unknown (e.g. out of GERAN/UTRAN coverage)  
5 registered, roaming
  


>AT+CEREG? +CEREG: \<n>,\<stat>[,[\<tac>],[\<ci>],[\<AcT>]   
\<stat>: integer  type; indicates the EPS registration status  
0 not registered, MT is not currently searching an operator to register to   
1 registered, home network   
2 not registered, but MT is currently trying to attach or searching an operator to register to   
3 registration denied   
4 unknown (e.g. out of E-UTRAN coverage)   
5 registered, roaming   
 
###2.2 Setup data link 

#### 2.2.1 Define PDP context. 
This is for primary PDP 
 
AT+CGDCONT=[<\cid>[,\<PDP_type>[,\<APN>[,\<PDP_addr>[,\<d_comp>[,\<h_comp>[,\<IPv4AddrAll oc>[,\<emergency indication>[,\<P-CSCF_discovery>[,\<IM_CN_Signalling_Flag_Ind>]]]]]]]]]] 
 
Sample:  AT+CGDCONT=5,”IP”,”cmnet” 
 
#### 2.2.2 Define secondary PDP context
This is for secondary PDP 
 
AT+CGDSCONT=[\<cid>,\<p_cid>[,\<d_comp>[,\<h_comp>[,\<IM_CN_Signalling_Flag_Ind>]]]] 
 
Sample:  AT+CGDSCONT=6,5 
 
#### 2.2.3 Define EPS quality of service
This is for 4G PS. 
 
AT+CGEQOS=[\<cid>[,\<QCI>[,\<DL_GBR>,\<UL_GBR>[,\<DL_MBR>,\<UL_MBR]]]] 
 
Sample: AT+CGEQOS= 5, 1, 64,64,64,64 
 
#### 2.2.4 Traffic flow template
This is for secondary PDP 
 
AT+CGTFT=[\<cid>,[\<packet filter identifier>,\<evaluation precedence index>[,\<remote address and subnet mask>[,\<protocol number (ipv4) / next header (ipv6)>[,\<local port range>[,\<remote port range>[,\<ipsec security parameter index (spi)>[,\<type of service (tos) (ipv4) and mask / traffic class (ipv6) and mask>[,\<flow label (ipv6)>[,\<direction>]]]]]]]]]] 
 
#### 2.2.5 Enter data state 
 
AT+CGDATA=\<L2P>[,\<cid>[,\<cid>[,...]]] 
 
Sample: AT+CGDATA=””,5   
Return:  CONNECT 


### 2.3 Check current PDP context 
 
AT+CGDCONT?   
+CGDCONT: \<cid>,\<PDP_type>,\<APN>,\<PDP_addr>,\<d_comp>,\<h_comp>[,\<IPv4AddrAlloc>[,\< emergency indication>[,\<P-CSCF_discovery> [, \<IM_CN_Signalling_Flag_Ind>]]]] 
 
 
MIFI will allocate IP to TCP/IP protocol stack after this AT and then user can access internet. 
 
### 2.4 Sample procudure 
 
#### 2.4.1 Primary PDP for 2/3G 
AT+CGREG?  
AT+CGDCONT=5,”IP”,”cmnet”   
AT+CGDATA=””,5  
AT+CGDCONT? 
 
#### 2.4.2 Default primary PDP in 4G 
AT+CEREG?  
AT+CGDCONT? 
 
#### 2.4.3 Dedicated Primary PDP in 4G 
AT+CEREG?  
AT+CGDCONT=5,”IP”,”cmnet”  
AT+CGEQOS= 5,0,64,64,64,64  
AT+CGDATA=””,5  
AT+CGDCONT?   
 
#### 2.4.4 Default secondary or dedicated secondary PDP in 4G 
 
AT+CEREG?  
AT+CGDSCONT=6,5  
AT+CGEQOS= 5,0,64,64,64,64  
AT+CGTFT=  
AT+CGDATA=””,6  
AT+CGDCONT?   
