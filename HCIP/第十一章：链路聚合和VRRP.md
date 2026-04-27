# **一、聚合链路---（链路备份技术）**

## **产生背景：**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCEc6749939933f92c90295ac9b84c182ff/13638)



两个设备之间连接多条线路，可以实现负载分担（备份），但由于交换机自动开启STP，所以STP为了避免出现环路，会阻塞一些端口，不管两个设备之间连接多少条线路，最终只有一条线路可使用，所以极大的浪费了线路资源。为解决这一问题，这时链路聚合技术就出现了。

## **1、简介**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCE8e5f0e3ac7f0e56e41710ca7c1cf6a79/13114)

1. 链路聚合把多条物理链路聚合在一起，形成一条逻辑链路；
1. 提高链路带宽和冗余性，实现负载分担；



## **2、工作原理**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCEf47971fb4bc6f901b8233e9a6be3caf1/13118)



注意：一个聚合组里的所有物理接口，都必须配置相同

聚合接口：是一个手工配置的逻辑接口；

链路聚合组：随着聚合接口的创建产生



（1）操作KEY：

1. 用于选择链路聚合成员端口的配置信息
1. 由参考端口的端口属性和第二类配置生成
1. 成员端口的端口属性和第二类配置与操作KEY一致，端口才能被选中

（2）参考端口：用来选举聚合成员端口的标准端口（依据链路聚合模式）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCEaafba14e05b273f573ed982dab36b6b0/13398)



思考：三个端口加入了两个vlan中，哪些聚合接口能UP，哪些会down?    是1口UP，2和3口down？ 还是1口down，2和3口UP？



有些端口的配置必须和参考端口一样，有的配置不一定一样。例如：以下三种情况

（3）端口属性配置：包括速率、双工模式、链路状态（UP/DOWN）这三项配置，参考端口选举，会影响成员端口是否被选中

（4）端口的第一类配置：不参与操作KEY计算的配置信息；例如：mstp等

（5）端口的第二类配置：参与操作KEY计算的配置信息；例如：VLAN配置、端口类型配置、BPDU Tunnel（淘汰）、端口隔离、端口属性、QinQ、MAC地址学习配置；



## **3、聚合模式**

**手工负载分担模式****（静态聚合）：----华为设备默认的链路聚合模式**

特点：该模式下所有活动链路都参与数据的转发，平均分担流量，因此称为负载分担模式。

（1）端口不与对端设备交互信息，自己选自己参考端口，互不干扰；

（2）参考端口选举规则：

1. 高速全双工>高速半双工>低速全双工>低速半双工；例如：1000Mbps全>1000Mbps半>100Mbps全>100Mbps半
1. 端口ID小的优先；端口ID=端口LACP优先级+端口编号；端口优先级默认32768

LACP协议：链路聚合控制协议

**注：建议跨厂商设备使用静态聚合，因为双方可能协商不一致**

（3）静态聚合流程

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCE71d15b74f9069b285afd9175825e10f9/13125)



**动态聚合：**

（1）端口使用LACP协议主动对端设备交互信息

（2）参考端口选举规则：

1. 设备ID小的优先：设备ID=LACP优先级+MAC地址；LACP优先级默认32768

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCE3801aa8c7b33db30beb184d4b1030490/45364)

1. 聚合端口ID小的优先：端口ID=端口LACP优先级+端口编号；端口优先级默认32768

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCE076ce0bfc20f3b5495c1fe440c976ad0/45366)

**建议同厂商设备使用动态聚合，因为双方协商一致**

（3）动态聚合流程

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCEebb6cc290a7dfd1050f1efd0388a0666/13128)

## **4、相关命令**

**（1）手工聚合（静态聚合）：**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCEe98ce44906a0e3ab30aa2847d7d64aa6/18071)



（华为设备的聚合链路默认采用的是基于流的负载分担。 --- 华为设备默认通过源IP和目标IP来区分不同的数据流）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCE2c885954df8ba2170d99b26c7e5ce5da/13435)



**（2）LACP聚合：**聚合链路形成以后，LACP负责维护链路状态，在聚合条件发生变化时，自动调整或解散链路聚合。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCEe17797f3a2963f38822f00730d41bdab/18077)



**注意：需要保持两端链路聚合模式一致。**



## **5、链路聚合实验**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCE3c919860e549bebed591e2ae58e8f928/14177)



# **二、VRRP协议（虚拟路由器冗余协议）**

## **1、VRRP背景：**

如果网关出现故障，则内网设备无法访问外网。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCEa26739ff86e8112c0f4085526390e600/13454)

## **2、VRRP主备份**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCE45347ed99e594a53a559b4c166ddbd86/13458)



VRRP可将多个路由器加入到备份组中，形成一台虚拟路由器，承担网关功能；

只要备份组中仍有一台路由正常工作，虚拟路由器就仍然正常工作；



## **3、VRRP概述**

（1）VRRP版本

VRRPV2：是一种容错协议，在提高可靠性的同时，简化了主机的配置 ，只支持IPV4

VRRPV3：同时支持IPV4\IPV6

（2）组播地址：224.0.0.18，用于发送组播报文，传递路由信息

（3）虚拟路由器由LAN上唯一的VIRTUAL ROUTER ID标识。并具有虚MAC地址：00-00-5E-00-01-{VRID}



## **4、工作原理**

一个VRRP备份组会选举出一个主网关和若干备份网关

（1）VRRP选举规则：

1. 优先级大的优先：

默认优先级是100，可配置的范围0-254；

1. IP地址大的优先：

注：若主备角色没定，IP地址大的优先成为MASTER；若主备角色已确定，IP地址不影响角色选举；

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCEe78bbc43ad91ecd8aef01f7430edd1a4/13479)



**特殊优先级值**：

1. **优先级=0**：表示当前路由器主动放弃Master角色（通常用于强制主备切换）。
1. **优先级=255**：如果路由器拥有虚拟路由器的IP地址和某个设备真实的网关地址一致，则真实的设备被称为（“IP地址所有者”），则自动以255优先级运行，直接成为Master（无需选举）



（2）VRRP 角色切换条件：

当主网关设备故障时，会导致备份网关无法接受到心跳（组播）报文；

默认工作在抢占模式，会发生角色抢占；（例如：当备份组中出现优先级更高的设备）



## **5、VRRP接口监视**

技术背景：若网关设备的上行链路故障，而网关本身正常，不会发生角色切换，但发往本网关的数据无法连通外面网络

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCE7a3d82277cb0c5320db3f8ef2c99d932/13501)



解决方法：设置VRRP上行接口监视功能，当上行接口DOWN后，主动降低本网关优先级，降30，以触发角色抢占。



## **6、VRRP负载分担：**

VRRP将多台路由器同时承担业务，形成多台虚拟路由器，分担内外网之间的流量。（网段间的主备相互备份）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCE646b5489a0304214aa6e03724d81d1f8/45373)

## **7、****VRRP和MSTP结合应用---黄金组合**

在堆叠技术未出现前，所有的大中型组网。从接入层到汇聚层之间用的黄金组合

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCEdb559a211eac86159199a9f8d649c3ae/13510)

特点：

1. MSTP是将一个或多个VLAN映射到一个生成树的实例，若干个VLAN共用一个生成树，MSTP可以实现负载均衡。
1. VRRP配置网关可以灵活根据网络拓扑变化而自动切换，提高网络可靠性。
1. VRRP+MSTP可以在实现负载分担的同时保证网络设备的冗余备份。



## **8、相关命令**

```abap
创建VRRP备份组并给备份组配置虚拟IP地址
[interface-GigabitEthernet0/0/0] vrrp vrid 1 virtual-ip virtual-address
配置路由器在备份组中的优先级
[interface-GigabitEthernet0/0/0] vrrp vrid 1  priority priority-value
注意：通常情况下，Master设备的优先级应高于Backup设备。
配置备份组中设备的抢占延迟时间----主网关
[interface-GigabitEthernet0/0/0] vrrp vrid virtual-router-id preempt-mode timer delay delay-value
配置VRRP备份组监视接口---主网关
[interface-GigabitEthernet0/0/0] vrrp vrid virtual-router-id track interface interface-type interface-number [ increased value-increased | reduced value-decreased ]
显示VRRP备份组的状态信息
[switch]display vrrp brief
```



## **9、VRRP实验**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCE7a2da0e5ff237eb87da1f9dca95c84ee/13621)



# **三、交换部分综合实验**



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB5cf09d2bc3fa0e898fe62cd39d8f2f4f/WEBRESOURCE716ad198117583bc57a8ca952052c39b/13624)