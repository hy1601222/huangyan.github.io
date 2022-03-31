# huangyan.github.io
以前从事网站运营，会一些java开发，后期开始从事一些网络方面的工作，成为一名网络工程师，闲时写一点关于网络方面的文章，供初学者参考也为自己时常翻阅使用。
# 一、IPSec应用场景

#  1、IPSec隧道嵌套技术

IPSec(Internet Protocol Security)：是一组基于网络层的，应用密码学的安全通信协议族，是一个开放的协议族。IPSec主要是解决数据传输过程中的机密性、完整性、真实性、抗重播这些问题，利用IPSec的安全机制在局域网安全互联、与移动用户远程安全接入、与多分支机构的远程安全接入通过隧道嵌套技术，实现安全传输。

隧道嵌套技术是指使用L2TP over IPSec、GRE over IPSec、DSVPN over IPSec这些方式实现点到点，点到多点的安全传输，通过嵌套的方式，结合不同的虚拟专网特点，实现不同的安全等级，不同的保密级别。企业数据与公网数据的安全隔离，内网之间不同部门的安全隔离。

比如：总部通过公网传输数据到分支机构，在internet网上加密不被探视、窃取、修改、领造。但在总部与分部之间不同部门，如财务部部门、研发部门、销售部门、行政管理部门之间传输的数据也需要保密，这就是隧道嵌套技术所据有的安全功能。

![隧道嵌套技术](https://img-blog.csdnimg.cn/f346c64f59c64af697ac4033f3b7175f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMOS4jjHkuYvml4U=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

#  2、GRE over IPSec隧道

GRE最初是一种封装方法的名称。IETF首先在RFC1701中描述GRE，一个在任意一种网络协议上传送任意其它网络协议的封装方法。GRE的特点：支持多种协议和多播、能够用来创建弹性的VPN、支持多点隧道、能够实施QOS。GRE的缺点是缺乏加密机制，没有标准的控制协议来保持GRE隧道。

IPSec是针对数据流页设计，其协议机制决定了其难于支持组播，对于组播的不支持也决定了IPSec对于路由协议的不支持，且对于IP协议中多种协议的支持性差。
![GRE over IPSec](https://img-blog.csdnimg.cn/a1d847b0def244cf9016fab299dd92cd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMOS4jjHkuYvml4U=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

结合两者的优缺点，既对组播、路由协议的支持，又能安全传输，产生了GRE over IPSec这种隧道方式。GRE over IPSec的中文意思是GR隧道E穿越IPSec隧道，两者结合有以下优势：

|    特性    | GRE是否支持 | IPSec是否支持 | GRE over IPSec是否支持 |
| :--------: | :---------: | :-----------: | :--------------------: |
|   多协议   |      Y      |       N       |           Y            |
|  虚拟接口  |      Y      |       N       |           Y            |
|    组播    |      Y      |       N       |           Y            |
|  路由协议  |      Y      |       N       |           Y            |
|  IP协议簇  |      Y      |  支持度不高   |           Y            |
|   机密性   |      N      |       Y       |           Y            |
|   完整性   |      N      |       Y       |           Y            |
| 数据源验证 |      N      |       Y       |           Y            |

利用GRE将组播、广播和非IP报文封装成普通的IP报文，通过IPSec将封装后的IP报文安全地传输。 因为隧道模式跟传输模式相比多增加了IPSec头，导致报文长度更长，更容易导致分片。所以推荐采用传输模式GRE over IPSec。

GRE over IPSec的报文封装，GRE隧道封装与IPSec隧道封装独立工作，原始IP数据到达Tunnel口收到此报文后，通过Tunnel口封装成载荷协议。 IPSec隧道将GRE封装包进行封装。
![GRE over IPSec报文封装](https://img-blog.csdnimg.cn/55263dda87bf45bea47e9375434525ef.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMOS4jjHkuYvml4U=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
配置虚拟隧道接口建立GRE over IPSec隧道案例：
![在这里插入图片描述](https://img-blog.csdnimg.cn/ed23a972231b49438e6b421a506f60bf.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMOS4jjHkuYvml4U=,size_16,color_FFFFFF,t_70,g_se,x_16#pic_center)
配置方案

采用如下思路配置虚拟隧道接口建立GRE over IPSec隧道：

(1) 配置物理接口的IP地址和到对端的静态路由，保证两端路由可达。
(2) 配置GRE Tunnel接口。
(3)配置IPSec安全提议，定义IPSec的保护方法。
(4)配置IKE对等体，定义对等体间IKE协商时的属性。
(5)配置安全框架，并引用安全提议和IKE对等体。
(6)在Tunnel接口上应用安全框架，使接口具有IPSec的保护功能。
(7)配置Tunnel接口的转发路由，将需要IPSec保护的数据流引到Tunnel接口。

(1) 、 配置物理接口的IP地址

```java
[Huawei]sysname RA
[RA]int g0/0/0
[RA-GigabitEthernet0/0/0]ip add 192.10.10.2 24
[RA-GigabitEthernet0/0/0]int g0/0/1
[RA-GigabitEthernet0/0/1]ip add 10.1.1.1 24
[RA-GigabitEthernet0/0/1]dis th
```

RouterA上配置到对端的静态路由，此处假设到对端的下一跳地址为192.10.10.3。

```java
[RA]ip route-static 202.18.18.10 255.255.255.0 192.10.10.3
```

路由器B配置此处略

(2) 、配置GRE Tunnel接口

```java
[RA]int Tunnel 0/0/0
[RA-Tunnel0/0/0]ip add 10.1.2.1 24
[RA-Tunnel0/0/0]tunnel-protocol gre
[RA-Tunnel0/0/0]source 192.10.10.2
[RA-Tunnel0/0/0]destination 202.18.18.10
```

路由器B配置此处略

(3) 、创建IPSec安全提议

```java
[RA]ipsec proposal safe1
[RA-ipsec-proposal-safe1]esp authentication-algorithm sha2-256
[RA-ipsec-proposal-safe1]esp encryption-algorithm aes-128
[RA-ipsec-proposal-safe1]di th
```

(4) 、配置IKE安全提议。

```java
[RA]ike proposal 5
[RA-ike-proposal-5]authentication-algorithm sha2-256
[RA-ike-proposal-5]encryption-algorithm aes-128
[RA-ike-proposal-5]dh group14
```

查看安全提议
[RA]display ike proposal

(5) 、配置IKE对等体。

```java
[RA]ike peer sub v1
[RA-ike-peer-sub]ike-proposal 5
[RA-ike-peer-sub]pre-shared-key cipher tunnel-123
```

(6) 、创建安全框架

```java
[RA]ipsec profile gre1
[RA-ipsec-profile-gre1]proposal safe1
[RA-ipsec-profile-gre1]ike-peer sub
[RA-ipsec-profile-gre1]di th
```

[RA-Tunnel0/0/0]display ipsec profile
![在这里插入图片描述](https://img-blog.csdnimg.cn/6acca7004a044662bad2097735b7be6b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMOS4jjHkuYvml4U=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
(7) 、接口上应用各自的安全框架

```java
[RA]int tun 0/0/0
[RA-Tunnel0/0/0]ipsec profile gre1
```

(8) 、配置Tunnel接口的转发路由，将需要IPSec保护的数据流引到Tunnel接口

```java
[RA]ip route-static 10.1.8.0 255.255.255.0 tunnel 0/0/0
```

查看ike sa
[RA]display ike sa

#  3、L2TP over IPSec隧道

L2TP通过拨号网络，基于PPP的协商，建立企业分支用户到企业总部的隧道，使远程用户可以接入企业总部。PPPoE（PPP over Ethernet）技术更是扩展了L2TP的应用范围，通过以太网络连接Internet，建立远程移动办公人员到企业总部的L2TP隧道。缺点是L2TP与GRE隧道封装都不能保障数据的机密性、完整性、真实性、抗重播这些问题。

LAC根据PPP报文中所携带的用户名或者域名信息，在LAC端和LNS建立L2TP隧道连接，将PPP协商延展到LNS。结合IPSec主要是解决数据传输过程中的机密性、完整性、真实性、抗重播这些特性，进行安全的数据传输。

![L2TP over IPSec](https://img-blog.csdnimg.cn/72a304807822413bb8ca697077cd6f93.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMOS4jjHkuYvml4U=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

L2TP拨号网络分为两种应用模式：独立LAC模式、分支用户LAC接入拨号模式。


![在这里插入图片描述](https://img-blog.csdnimg.cn/8e7ed7240c7d42fa8a995d5ddc755bdf.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMOS4jjHkuYvml4U=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
总部网关能指定符合条件的分支网关接入。可以在分支网关与总部网关之间建立IPSec隧道来实施安全保护。配置L2TP over IPSec的方式来加密保护企业分支和总部的业务。
![L2TP over IPSec案例](https://img-blog.csdnimg.cn/286c8f9cb54045a88163b7b0cc097c02.png#pic_center)

配置方案：
采用如下思路配置分支机构与总部之间通过L2TP Over IPSec方式实现安全互通：

(1) 、配置接口的IP地址和到对端的静态路由，保证两端路由可达。

(2) 、在LAC上配置L2TP功能，PPP用户通过L2TP隧道向总部发出接入请求，总部认证成功后建立隧道。

(3) 、在LAC上配置到达LNS的路由，使能LAC的自拨号功能。

(4) 、在LNS上配置L2TP功能及PPP用户，并配置访问公网的路由。

(5) 、配置ACL，以定义需要IPSec保护的数据流。

(6) 、配置IPSec安全提议，定义IPSec的保护方法。

(7) 、 配置IKE对等体，定义对等体间IKE协商时的属性。

(8) 、配置安全策略，并引用ACL、IPSec安全提议和IKE对等体，确定对何种数据流采取何种保护方法。

(9) 、在接口上应用安全策略组，使接口具有IPSec的保护功能。


(1) 、配置接口的IP地址和到对端的静态路由，保证两端路由可达。

```java
<Huawei>sys
[Huawei]sysname LAC
[LAC]int g0/0/0
[LAC-GigabitEthernet0/0/0]ip add 192.10.10.2 24
[LAC-GigabitEthernet0/0/0]int g0/0/1
[LAC-GigabitEthernet0/0/1]ip add 10.1.1.1 24
[Huawei-GigabitEthernet0/0/1]quit
[LAC]ip route-static 202.18.18.10 255.255.255.0 192.10.10.3
```

路由器RB

```java
<Huawei>sys
[Huawei]int g0/0/0
[Huawei-GigabitEthernet0/0/0]ip add 202.18.18.10 24
[Huawei-GigabitEthernet0/0/0]int g0/0/1
[Huawei-GigabitEthernet0/0/1]ip add 10.1.8.1 24
[Huawei-GigabitEthernet0/0/1]quit
[Huawei]sysname LNS
[LNS]ip route-static 192.10.10.2 255.255.255.0 202.18.18.11
```

(2) 、在LAC端上配置L2TP功能

```java
[LAC]l2tp enable                 //LAC上全局使能L2TP
[LAC]l2tp-group 1                //创建L2TP组
[LAC-l2tp1]
[LAC-l2tp1]tunnel name lac
//配置为用户名称为huawei的用户建立到达LNS的L2TP连接。
[LAC-l2tp1]start l2tp ip 202.18.18.10 fullusername huawei  
```

在LAC端启用通道验证并设置通道验证密码。

```java
[LAC-l2tp1]tunnel authentication        //通道验证
[LAC-l2tp1]tunnel password cipher huawei       //验证密码
```

配置虚拟PPP用户的用户名和密码，PPP认证方式以及IP地址。

```java
[LAC]interface virtual-template 1                  //创建虚拟模板1
[LAC-Virtual-Template1]ppp chap user huawei        //用户名为huawei的PPP用户
[LAC-Virtual-Template1]ppp chap password cipher 123456
[LAC-Virtual-Template1]ip address ppp-negotiate    //通过协商获取IP地址
```

(3) 、配置LAC 自拨号

```java
[LAC] interface virtual-template 1
[LAC-Virtual-Template1] l2tp-auto-client enable
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/b8d2d59b55be4b05b31dede6242c9a7d.png#pic_center)
在LAC上配置私网路由，私网静态路由与公司总部连接

```java
[LAC]ip route-static 10.1.8.0 255.255.255.0 virtual-template 1
```

(4) 、LNS的配置

```java
[LNS]aaa                //配置LNS的AAA认证
[LNS-aaa]local-user huawei password cipher 123456    //用户名huawei密码123456
[LNS-aaa]local-user huawei service-type ppp         //服务类型PPP
```

在LNS上配置LNS的IP地址池，,为LAC的拨号接口分配IP地址。作为远程用户的动态IP地址资源。

```java
[LNS]ip pool 1
[LNS-ip-pool-1]network 10.1.10.0 mask 24       //分配地址的网段
[LNS-ip-pool-1]gateway-list 10.1.10.1          //网关
```

在LNS上创建虚拟接口模板并配置PPP协商等参数。

```java
[LNS]interface virtual-template 1                 //创建虚拟模板1
[LNS-Virtual-Template1]ppp authentication-mode ?
  chap  Enable CHAP authentication
  pap   Enable PAP authentication
[LNS-Virtual-Template1]ppp authentication-mode chap   //配置PPP认证方式为chap         
[LNS-Virtual-Template1]remote address pool 1        //指定地址池1
[LNS-Virtual-Template1]ip add 10.1.10.1 255.255.255.0      //远程访问网关
```

 在LNS上使能L2TP服务及配置LNS本端隧道名称、启用隧道认证功能

```java
[LNS]l2tp enable                      //全局使能L2TP功能
[LNS]l2tp-group 1                     //配置L2TP组
[LNS-l2tp1]tunnel name lns            // 配置LNS本端隧道名称及指定LAC的隧道名称
[LNS-l2tp1]allow l2tp virtual-template 1 remote lac  //允许隧道虚拟接口指定lac的隧道
[LNS-l2tp1]tunnel authentication      //启用隧道认证
[LNS-l2tp1]tunnel password cipher huawei    //设置隧道认证字huawei 和LAC验证密码保持一致。
```

在LNS上配置私网路由，使得企业总部与企业分支用户私网互通。

```java
[LNS]ip route-static 10.1.1.0 255.255.255.0 virtual-template 1
```

------------------------------------------------------------------------------------------------------------------

(5) 、配置ACL，以定义需要IPSec保护的数据流。
在LAC上配置ACL

```java
[LAC] acl number 3001
[LAC-acl-adv-3001]rule 5 permit ip source 192.10.10.0 0.0.0.255 destination 202.18.18.0 0.0.0.255
```

在LNS上配置ACL。

```java
[LNS]acl number 3001
[LNS-acl-adv-3001]rule 5 permit ip source 202.18.18.0 0.0.0.255 destination 192.10.10.0. 0.0.0.255
```

(6) 、配置IPSec安全提议，定义IPSec的保护方法。这里演示用最简单的加密和认证
LAC上配置IPSec安全提议。

```java
[LAC]ipsec proposal safe1
[LAC-ipsec-proposal-safe1]esp authentication-algorithm md5
[LAC-ipsec-proposal-safe1]esp encryption-algorithm des
```

LNS上配置IPSec安全提议。

```java
[LNS]ipsec proposal safe1
[LNS-ipsec-proposal-safe1]esp authentication-algorithm md5
[LNS-ipsec-proposal-safe1]esp encryption-algorithm des
```

(7) 、 配置IKE对等体，定义对等体间IKE协商时的属性。
LAC上配置IKE安全提议。

```java
[LAC]ike proposal 10
[LAC-ike-proposal-10]encryption-algorithm des-cbc 
[LAC-ike-proposal-10]authentication-algorithm md5
[LAC-ike-proposal-10]dh group14
```

<LAC>display ike proposal
![在这里插入图片描述](https://img-blog.csdnimg.cn/f2896e760e4f4a79b79c8da0b844e186.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMOS4jjHkuYvml4U=,size_18,color_FFFFFF,t_70,g_se,x_16#pic_center)

在LAC上配置IKE对等体，并根据默认配置，配置预共享密钥和对端ID。

```java
[LAC]ike peer sub v1
[LAC-ike-peer-sub]ike-proposal 10
[LAC-ike-peer-sub]pre-shared-key cipher 123456
[LAC-ike-peer-sub]remote-address 202.18.18.10
```

LNS上配置IKE安全提议。

```java
[LNS]ike proposal 10
[LNS-ike-proposal-10]encryption-algorithm des-cbc 
[LNS-ike-proposal-10]authentication-algorithm md5
[LNS-ike-proposal-10]dh group14
```

在LNS上配置IKE对等体。

```java
[LNS]ike peer sub v1
[LNS-ike-peer-sub]ike-proposal 10
[LNS-ike-peer-sub]pre-shared-key cipher 123456
[LNS-ike-peer-sub]remote-address 192.10.10.2
```

(8) 、配置安全策略
在LAC上配置IKE动态协商方式安全策略。

```java
[LAC]ipsec policy plan1 5 isakmp
[LAC-ipsec-policy-isakmp-plan1-5]ike-p	
[LAC-ipsec-policy-isakmp-plan1-5]ike-peer sub
[LAC-ipsec-policy-isakmp-plan1-5]proposal safe1
[LAC-ipsec-policy-isakmp-plan1-5]security acl 3001
```

[LAC-ipsec-policy-isakmp-plan1-5]display ipsec policy
![在这里插入图片描述](https://img-blog.csdnimg.cn/e25f0c41dbef43b89d88bc82c92182a9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMOS4jjHkuYvml4U=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
在LNS上配置IKE动态协商方式安全策略。

```java
[LNS]ipsec policy plan1 5 isakmp
[LNS-ipsec-policy-isakmp-plan1-5]ike-peer sub
[LNS-ipsec-policy-isakmp-plan1-5]proposal safe1
[LNS-ipsec-policy-isakmp-plan1-5]security acl 3001
```

(9) 、在接口上应用安全策略组，使接口具有IPSec的保护功能。
在LAC的接口上引用安全策略组。

```java
[LAC]int g0/0/0
[LAC-GigabitEthernet0/0/0]ipsec poli	
[LAC-GigabitEthernet0/0/0]ipsec policy plan1
```

在LNS的接口上引用安全策略组。

```java
[LNS]int g0/0/0
[LNS-GigabitEthernet0/0/0]ipsec policy use1
```

查看当前由IKE建立的安全联盟。
[LAC]display ike sa
