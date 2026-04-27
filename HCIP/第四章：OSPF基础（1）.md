# **一、OSPF基础**

## **1、技术背景（RIP中存在的问题）**

1. RIP中存在最大跳数为15的限制，不能适应大规模组网
1. 周期性发送全部路由信息，占用大量的带宽资源
1. 路由收敛速度慢
1. 以跳数作为度量值
1. 存在路由环路可能性
1. 每隔30秒更新

## **2、OSPF协议特点**

1. 没有跳数限制，适合大规模组网
1. 使用组播更新变化的路由和网络信息
1. 路由收敛快，可以触发更新
1. 以COST作为度量值
1. 采用SPF算法（最短路径优先算法）有效避免环路
1. 每隔30分钟周期性更新
1. 在互联网上大量使用，是运用最广泛的路由协议

注意：OSPF传递的是拓扑信息和路由信息，RIP传递的是路由表

## **3、OSPF三张表**

邻居表：记录邻居状态和关系

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE13b4735e6fe493b8df4d925820d572dc/42815)

拓扑表：链路状态数据库（LSDB）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCEd243d624a3222bb64e49bab8dd751ef9/8269)

路由表：记录由SPF算法计算的路由，存放在OSPF路由表中

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCEd575829e60756264d9e2f71d2b9afbd6/42813)

## **4、OSPF数据包(可抓包)**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE56c6454bef818c77ec3d46eefbf24ba0/8089)

OSPF报文直接封装在IP报文中，协议号89

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCEdbed0aade062e1f4bff47410a7904022/15044)

### **头部数据包内容：**

1. 版本(Version):对于OSPFv2，该字段值恒为2。
1. 类型(Type):该OSPF报文的类型。该字段的值与报文类型的对应关系是:1-Hello;2-DD;3-LSR;4-LSU;5-LSAck。
1. 报文长度（Packet Length):整个OSPF 报文的长度（字节数)。
1. 路由器ID (Router Identification):路由器的OSPF Router-ID。
1. 区域ID (Area Identification):该报文所属的区域ID，这是一个32bit 的数值。
1. 校验和(Checksum):用于校验报文有效性的字段。保证数据传输的完整性
1. 认证类型(Authentication Type):指示该报文使用的认证类型。
1. 认证数据（Authentication Data):用于报文认证的内容。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCEa2ad02817479ce136825146f1deb0efa/8203)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCEec6d5d39ea07162fae1e4729e79a7ea2/8209)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE33ae42f9a74d9038020aa85c5efcc23e/8204)

### **(1)hello：**

作用：用来周期保活的，发现，建立邻居关系。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE67c0d92cf4863b89ff565d80f3d41db1/15060)



1. 网络掩码(Network Mask):一旦路由器的某个接口激活了OSPF，该接口即开始发送Hello报文，该字段填充的是该接口的网络掩码。两台OSPF 路由器如果通过以太网接口直连，那么双方的直连接口必须配置相同的网络掩码，否则影响邻居关系建立。
1. Hello间隔(Hello Interval):接口周期性发送Hello报文的时间间隔(单位为s)。两台直连路由器要建立OSPF邻居关系，需确保接口的Hello Interval相同，否则邻居关系无法正常建立。
1. 路由器失效时间（Router Dead Interval):在邻居路由器被视为无效前，需等待收到对方Hello报文的时间（单位为s)。两台直连路由器要建立OSPF 邻居关系，需确保双方直连接口的Router Dead Interval相同，否则邻居关系无法正常建立。缺省情况下，OSPF路由器接口的Router Dead Interval为该接口的Hello Interval的4倍。

```ada
[R1-GigabitEthernet0/0/0]ospf timer hello 5   //修改hello时间
[R1-GigabitEthernet0/0/0]ospf timer dead 10   //修改dead时间
注：自行修改的dead时间应该大于hello时间,否则时间无法修改
```

1. 可选项（Options):该字段一共8bit，每个比特位都用于指示该路由器的某个特定的OSPF 特性。Options字段中的某些比特位会被检查，这有可能会直接影响到OSPF邻接关系的建立。（特殊区域的标记）

E：是否支持外部路由

MC：是否支持转发组播数据包

N/P：是否为NSSA区域

1. 路由器优先级(Router Priority):路由器优先级，范围：0-255，默认是1，也叫DR优先级，数字越大，优先级越高，该字段用于DR、BDR 的选举

```ada
[R1-GigabitEthernet0/0/0]ospf dr-priority ?     //修改路由器优先级
  INTEGER<0-255>  Router priority value
```

1. 指定路由器(Designated Router):网络中DR的接口IP地址。如果该字段值为0.0.0.0，则表示没有DR，或者DR尚未选举出来。
1. 备份指定路由器（Backup Designated Router):网络中 BDR的接口IP地址。如果该字段值为0.0.0.0，则表示网络中没有BDR，或者BDR尚未选举出来。
1. 邻居(Neighbor)。在直连链路上发现的有效邻居，此处填充的是邻居的Router-iD，如果发现了多个邻居，则包含多个邻居字段。



### **(2)DD--(DBD)（数据库描述报文）：**

作用：仅包含LSA摘要

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCEc4cbad1182c0c11591cb6fb4830bbb9f/15058)

1. 接口最大传输单元(Interface Maximum Transmission Unit):接口的MTU。DD报文默认的MTU=0，华为设备默认不开启接口的MTU值检测功能，但思科会检测。

一端开启MTU值检测功能，一端不开启，OSPF的邻居也可以正常建立。

邻居建立依据：对端发过来的DD报文中携带的MTU值小于本端的MTU值，就接收报文。

```ada
[R1-GigabitEthernet0/0/0]ospf mtu-enable      //开启接口的MTU检测功能，会使接口发送DD报文携带接口的MTU=1500
```

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE9a2561967a3a002ab944d6230842bef3/42847)

两端都开启MTU值的检测功能时，MTU值不一样会导致邻居关系停留在exstart状态。

```ada
[R1-GigabitEthernet0/0/0]mtu 1400
[R2-GigabitEthernet0/0/0]mtu 1500
<R1>reset ospf process     //有时候需要重启进程，才能观察到exstart状态
```

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCEb64f84c801166be9d4770db04457ec69/42852)

1. 可选项(Options):路由器支持的OSPF可选项。
1. DD报文置位符：

I：init位，I=1，这是第一次报文的第一个DD报文，两方交互的第一个DD报文是空的，没有包含LSA的任何信息，目的：告诉对方做好接收LSA摘要的准备工作；选出Master路由器（协商LSDB的主从）

M:more位，M=1表示后续还有DD报文

MS:master位，MS=1,表示本端为主

1. DD序列号(DD Sequence Number): DD报文的序列号，在DD报文交互的过程中，DD序列号被逐次加1，用于确保DD报文传输的有序和可靠性。

值得注意的是，DD序列号必须是由Master路由器来决定的，Router ID大的一方会成为Master，而Slave路由器只能使用Master路由器发送的DD序列号来发送自己的DD报文。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE9dce30b2649eae3c37a90626e5b37905/15051)

1. LSA头部(LSA Header):当路由器使用DD报文来描述自己的LSDB时，LSA的头部信息被包含在此处。一个DD报文可能包含一条或多条LSA的头部。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE6171bf1ae5c47662c15f0d534542ecdc/35352)



### **(3)LSR：**

作用：请求自己没有的或则比自己更新的链路状态详细信息

链路状态类型，链路状态ID，通告路由器 ---- “LSA三元组” --- 通过着LS三个参数可以唯一的标识出一条LSA

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE6cb8c655a00c231e400e8cb9d7d27cdf/8094)



### **(4)LSU：**

作用：链路状态更新信息

一个LSU报文可以包含多个完整的LSA

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE5fe6dddd487e88fca72eac1ec28dbe7d/8096)



当路由器感知到网络发生变化时，也可以触发LSU报文的泛洪，以便将该变化通知给网络中的其他OSPF 路由器。

**在MA网络中，非 DR、BDR路由器向224.0.0.6这个组播地址发送LSU报文，而DR会侦听这个组播地址，DR在接收LSU报文后向224.0.0.5发送LSU报文，从而将更新信息泛洪到整个OSPF域，所有的OSPF路由器都会侦听224.0.0.5这个组播地址。**

总结：

224.0.0.5所有运行OSPF的接口会监听

224.0.0.6所有DR的接口会监听



### **(5)LSAck:**

作用：对LSU的确认

报文中包含着路由器所确认的LSA的头部（每个LSA头部的长度为20byte)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE933c9aff4578a9e8e2a2b4612d85fc21/8098)



## **5、OSPF工作过程**

邻居：双方通过hello报文，相互认识

邻接：邻居关系建立好后，进行一系列报文交互，当两台路由器LSDB同步完成，开始独立计算路由时，这两台路由器形成了邻接关系

### **（1）确认可达性，建立邻居**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCEf4cdb8ee4a764442ed889232cd2ba165/15063)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE0e97b82d2b2085b7287c33b3075b4760/8073)

#### **router ID :**

作用：标明的是路由器身份

手工配置：IPV4地址格式

自动选举：

**环回口:IP地址大的优先**

物理口：IP地址大的优先



#### **2-way前，确认DR/BDR**

选举原因：广播网络中使路由信息交换更加高速有序，可以降低需要维护的邻居关系数量

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCEc47f9169b4c06110df85c32890fb7d4f/15088)

邻居关系过多缺点：

（1）大量产生hello包，消耗CPU性能

（2）产生重复路由通告，消耗CPU性能

（3）任何一台路由器的路由变化都会导致多次传递,浪费了带宽资源



选举范围：每个网段都需要选出一个DR和BDR（0-255）

选举规则：1.优先级大的优先，默认优先级是1；2.router-id大的优先

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCEdaf4a791400383fb7049ba84882ad797/15089)



注：DR/BDR的选举没有抢占性



关系状态：DRother与DR建立邻接关系；

DRother与BDR建立邻接关系；

DR /BDR建立邻接关系；

DRother之间保持邻居关系



### **（2）****邻接路由器之间交换链路状态信息，实现区域内链路状态数据库同步**

1. 向邻接路由器发送DBD报文，通告本 地LSDB中所有LSA的摘要信息
1. 收到DBD报文后，与本地LSDB对比，向对方发送LSR报文，请求发送本地所需要的LSA的完整信息
1. 收到LSR后,把对方所需的LSA的完整信息打包为一条LSU报文，发送至对方
1. 收到LSU后，向对方回复LSAck报文，进行确认

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE371b7e0353b490674a39f23bbc9b7bd2/15081)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE7456afa3ce4eeffe76b749bfb4809346/15083)



### **（3）完整信息同步，****完全邻接关系建立**

完全邻接关系建立，LSDB表与OSPF路由表形成

## **6、OSPF的状态机**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCEb7d2662cf562754bea19d34d601f9858/8035)

**(1)down：**关闭状态（稳定状态），这种情况处于手动指定邻居的情况下，发送hello包之后进入下一个状态

(2)INIT：初始化状态，收到对方的hello报文，但没有收到对方的hello确认报文

(3)Attempt：一般不会出现，只出现在NBMA网络中，发出hello，但收不到对方的hello包

**(4)2-way（稳定状态）：**双方互相发现，邻居状态稳定，并确认了DR/BDR的角色；

当选举完毕后，就算出现一台优先级更高的路由器，也不会替换成新的DR\BDR；

需要原DR\BDR失效，或重置OSPF进程才会成为新的DR\BDR；

**2-way的前提：**

1. Router-id无冲突，修改router-id需要重置ospf进程使生效；
1. 掩码长度一致（MA网络中）
1. 区域ID一致；
1. 验证密码一致；
1. hello-time一致；
1. dead-time一致；
1. 特殊区域类型一致；



(5)Exstart：交换开始状态；发送第一个DD报文，但不发送LSA摘要，仅用于确定LSDB协商的主从，ROUTER-ID大的成为master

(6)Exchange：交换状态；发送后续DD报文，用于通告LSDB中的LSA摘要

(7)Loading：读取状态，进行LSA的请求、加入和确认

**(8)Full：**邻接状态（稳定状态），两端同步LSDB；

FULL的前提：两端MTU一致，否则可能卡在EXSTART\Exchange状态

能够计算路由的前提：两端网络类型一致，否则邻居状态full,但无法学习路由



## **7、OSPF邻居建立条件总结**

#### **（1）所有网络类型的共同条件**

Router-id无冲突，修改router-id需要重置ospf进程使生效；

掩码长度一致（MA网络中），P2P网络不需要掩码一致也能建立邻居

区域ID一致；

验证密码一致；

hello-time一致；

dead-time一致；

特殊区域类型一致；



### **（2）P2P网络中的网段和掩码不一致，邻居也可以建立**

在P2P(PPP,HDLC)的网络类型邻居关系建立条件:(P2P网络类型不对hello报文中携带的接口地址和掩码长度是否一致做检测)

1.1.1.1 ppp 2.2.2.2

1. 地址在一个网段，但掩码不一致能建立邻接关系比如:10.1.12.1/30------- 10.1.12.2/24路由也能计算也能通
1. 不在一个网段能建立邻居关系，路由也能计算1.1.1.1/24 ------- 2.2.2.2/30
1. 一个接口包含另外一个接口的IP也能建立邻居关系，路由也能计算 10.1.12.1/30 - 10.1.12.10/24

华为以太网P2P链路，邻居和路由均能正常建立和计算，但无法实现单播通信，原因是ARP解析出现问题，下一跳路由器不回复arp请求报文。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE1f58cda26e4309e13375cd857aef1a26/43006)

### **（3）MA网络中，同网段，掩码必须一致，才能建邻**

广播型，NBMA网络类型(同思科)(广播型和NBAM 对hello报文的掩码长度做一致性检测)

1. 如果不在一个网段，邻居关系无法建立
1. 如果不在一个网段，但一个接口包含另外一个接口的IP，邻居关系无法建立
1. 如果链路地址都在一个网段，但彼此掩码不一致，邻居关系无法建立
1. 如果链路地址都在一个网络,和掩码相同，邻居关系可以正常建立



## **8、LSDB的更新**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCEaa6efc80dc8d9a4c13cb84e3ef3cd0c0/15095)

注：广播网络中的更新：只由DR发起



## **9、OSPF开销计算**

### **（1）参考带宽**

1. 默认是100M
1. 计算开销的基准带宽值
1. 参考带宽仅本地有效
1. 建议把网络中最高的链路带宽设置为参考带宽（不然 千兆链路和百兆链路的开销值都是1，无法判断哪条线路最优）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE908069ae8bd72c3601932768f830d5a0/35314)

[R1-GigabitEthernet0/0/0]ospf cost 200    //修改接口的COST

### **（2）计算方法**

链路带宽大于等于参考带宽   cost=1

反之，cost=参考带宽/链路带宽（Mb）

注：在路由器上。千兆以太网、百兆以太网、十兆以太网的缺省OSPF链路开销值分别是：1、1、10



### **（3）COST	**

OSPF以“累计cost”为开销值，也就是流量从源网络到目的网络所经过所有路由器的出接口的cost总和。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE224c2b27760d786bbe333d890e0539c2/42807)

修改cost值的例子：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE6e4164461a7495c6d5f63fdbcaaa09e6/42810)

# **二、OSPF的区域划分**

## **1、区域产生背景**

1. OSPF路由器在同一个区域中泛洪LSA。为了确保每台路由器都拥有对网络拓扑的一致认知，LSDB需要在区域内进行同步。
1. 如果OSPF域仅有一个区域，随着网络规模越来越大，OSPF路由器的数量越来越多，这将导致诸多问题。

## **2、分区好处**

减少LSA泛洪范围；

提高网络扩展性，有利于组建大规模的网络；

## **3、区域类型**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE32afac5b26fd8a74afd4187397a2f96a/8120)

骨干区域：

非骨干区域：

特殊区域：优化路由表、优化LSDB表



多区互连原则：

非骨干区域与非骨干区域不能直接相连；

所有非骨干区域必须与骨干区域相连；

此设计是为防止区域间环路；



## **4、OSPF的路由器类型**



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCEa9fb11cf8dc4957ca6e6181c52829abd/8116)



区域内路由器（IR）：所有接口在同一区域

骨干路由器（BR）：所有接口都在骨干区域

区域边界路由器（ABR）：连接骨干区域和非骨干区域，连接多个区域的设备

自治系统边界路由器（ASBR）：运行多种路由协议并引入外部路由



# **三、OSFP基础实验**

OSPF基础实验并尝试抓取OSPF的数据包

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBffa7ad50d542250953daf1c6834d8da1/WEBRESOURCE9e7fe97e4dcc856af62cbf056c7bdc82/43253)