# **一、BGP产生背景**

BGP（Border Gateway Protocol，边界网关协议）是一种用于自治系统间的动态路由协议，是一种外部网关协议。

自治系统AS：一组被进行统一管理，运行同一个IGP协议的路由器组成的网络范围。通常使用相同的路由策略。比如：通过写路由策略让某些流量走特定的线路。控制对邻居AS的路由的接收和发布。

自治系统编号：

2字节AS编号：取值范围0-65535，其中0和65535保留

公有AS:1-64511

私有AS:64512-65534

4字节AS编号：2的32次方的编号数量

公有：65536–4199999999

私有：4200000000–4294967294



www.cidr.report



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE71dbeb480dfae1eb03da99a73e33e4b1/18661)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEeaf6199702ac72f3606a24c1de302dba/18662)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEe1b11e79fd8e689443424a7a638cfb34/18664)



# **二、路由协议的分类**

IGP：RIP、OSPF、IS-IS

EGP：BGP



# **三、BGP协议特性**

1、BGP 只负责把路由从一个AS传到另一个AS；从其他AS传递过来的路由在本AS内部的扩散依靠IGP；

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEaf3634676dc0a70f151d2ffe1e2822ed/18628)

2、BGP是路径矢量协议，一跳是一个自治系统；

当一条路由传入某个AS，该路由的下一跳会变为上个AS的出接口的IP地址；当一条路由在某个AS内部的路由器之间传递时，下一跳不变。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE4eb5f3dd731cd5a352a7b62a78768b38/18631)



3、AS防环机制：路径矢量路由协议，从设计上避免了环路的发生（AS_PATH防环机制）

当一条路由每从一个AS传出，会把该AS的编号按照从右至左的顺序依次记录在AS_PATH属性中；

一个路由器从其他AS收到一条路由通告，会检查本路由器的AS标号是否出现在该路由条目的AS_PATH属性中，如果出现了则该条目不学习；

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE750f3d8a539945aeb5b114f3b0b12b76/18634)



例如：一条路由从AS 200中的R3传出，则其AS_PATH=(300,400,500,100,200),当这条路由回传给AS200时，R4会检查AS_PATH，发现重复路径则不接收。



4、BGP基于TCP协议传输，端口号179；必须手动配置邻居；

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEe8f01f4aaff69aca3dfa11fe118faca6/18637)

5、BGP第一次发送完整的路由表，后续只发送增量更新；

6、BGP有多种属性可以控制路由选择。（难点）

7、支持路由聚合；

8、路由过滤和路由策略；



# **四、BGP基本术语**

## **1、基本术语**

BGP Speaker：运行BGP协议的路由器称为BGP发言者

BGP Peer：相互之间存在TCP连接、相互交换路由信息的BGP发言者之间互称为BGP对等体

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE2ca2384642eb9dd8a038d062fe76380b/18640)



## **2、BGP对等体：**

BGP邻居可以直连，也可以非直连

### **（1）EBGP对等体：**

跨AS之间的邻居，一般情况下EBGP对等体是物理上直连的。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE0d67d76d4a4a6a90c355da85548f4c45/18643)

BGP发言者从EBGP对等体获得的路由会向它所有BGP对等体通告（包括EBGP和IBGP）

### **（2）IBGP对等体：**

同一个AS内部的邻居

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE8531cb9203f96a5645ab6f65dba01b00/18646)

### **（3）BGP邻居非直连的情况**

思考：哪些路由器需要跑BGP？（R2\R3\R5\R6）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEdce86e38baad1c6dea9e12f5b56d2396/18650)



思考：（R2\R3\R5\R6）邻居关系怎么建立？

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEf5481c3510e5bcb073deeaa97fae6473/18653)



思考：R3-R4之间，R6-R7之间，路由怎么传？

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE9127699485cc041f63e4ff0149dbcb12/18655)



# **五、BGP规划问题（规划不当容易产生路由黑洞）**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE7d19f73a8a0ae278cc77046b2be70a58/18658)

R2：peer 5.5.5.5 next-hop-local

R5：peer2.2.2.2 next-hop-local



目标IP：6.6.6.6/32 源IP：1.1.1.1/32+数据

**路由黑洞产生原因：**由于IBGP邻居之间有，没有运行BGP协议的路由器，无法获得BGP的路由；从而导致的数据包进入路由器被丢弃。

**路由黑洞解决方法：**

（1）BGP引入IGP（上图可以在R2和R5上把bgp引入ospf），这个方法有可能会造成引入BGP后，外部网络的路由太多了，内网的路由器承载不住。

（2）在黑洞路由器上配置目的网段的静态路由（不现实，比如：需要配置的目的网段太多的情况，累死网络管理员了）

（3）IBGP全连接   -----IBGP防环机制：IBGP水平分割：从IBGP邻居学习到的路由不会传递给其他IBGP邻居

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEbb4b818c0b1ff2d4d225f4e19bb73fc3/18211)

IBGP全连接（full-fresh）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE5e4e326510f6567d7e0464890befed5e/18213)

为啥有这种防环机制，看下图

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/ed5ae408f77a939ff78d7de74d567d1b/35755)

注：要是邻居太多，手工配置邻居，会导致工作量比较大



（4）BGP路由反射器（无视IBGP的防环机制，可以减少邻居关系的数量）

以R2作为反射器：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE6ec4b105f2d01ff756c2ab35c65033bb/18214)

（5）BGP联盟（完美的避开了IBGP全连接的坑------IBGP水平分割限制）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE3014157d8c3d472268f8d5d6783a092d/18215)



# **六、BGP环路问题****（水平分割）**

1、EBGP水平分割（AS_PATH）

通过AS_PATH属性防环，在学习到的路由中，若有本地AS号，则拒绝学习，防止环路

2、IBGP水平分割

当路由器从一个IBGP对等体学习到某条BGP路由时，它将不能再把这条路由通告给任何IBGP对等体。

造成问题：一些路由器无法学到去往其他AS的路由

解决方法：路由反射器和联盟



# **七、BGP消息种类（数据包可抓包看）**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE1d5b9754691b6bab6bdd6337d20c8971/18216)

## **（1）BGP头部信息：**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEdd7ae17ad41b200353170353071ac9da/18217)

1. 标记（Marker）：该字段被保留下来用于解决协议兼容性问题，没有其他含义
1. 长度（length）：指示BGP报文的长度（字节数）
1. 类型（Type）：该字段指示了BGP报文的类型，就是后面提到的BGP5种数据包类



## **（2）数据包种类**

#### **open：**

用于建立BGP对等体之间的连接关系，正常收发一次即可；携带route-id；

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEaaef2bd82b53f1e415442850106c8c9c/18218)

Hold time：保持时间，该字段表示路由器在收到Keepalive消息或者Update消息之前等待的最长时间，默认180s，如果邻居双方的保持时间不一致，将以较短的时间作为双方可接收的保持时间。

可选参数长度：指示了BGP报文中可选参数的长度

可选参数：Open包中包含多个可选参数，主要用于宣告及协商BGP对等体的某些能力特征。



#### **Keepalive：**

周期性的向BGP对等体发出Keepalive（周期保活）消息，用于保持连接的有效性，在默认情况下每60秒发送一条Keepalive消息，或者以已协商一致的保持时间的1/3为周期发送Keepalive消息。默认60s，超时180s。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE65d1fddde09572bfe278ffd047cfded3/18219)



#### **Update：**

携带的是路由更新（删减、增加）信息

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE1a1ca1973efd4649ba026fd5cf396108/35736)

更新一条路由信息：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE83e40bbd2056a63155407331b7eb79e4/44789)

1. Unfeasible（Withdrawn ） routes length：标明Withdrawn Routes部分的长度。其值为零时，表示没有撤销的路由。
1. Withdrawn Routes：包含要撤销的路由列表
1. Total Path Attribute Length：标明Path Attributes的长度。其值为零时，表示没有路由及其路由属性要通告。
1. Path Attributes：路径属性：包含要更新的路由属性列表
1. Network Layer Reachability Information（NLRI）：网路层可达信息，包含要更新的地址前缀列表（前缀和前缀长度）

删除一条路由信息

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE7057c467364a348dd312824149469da4/44793)



#### **notfication:通告**

当BGP检测到错误状态时，就向对等体发出notfication消息，之后BGP连接会立即被关闭(邻居关系结束了)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE8c0a2a0a8651a721857b07bdf6c12f56/44797)

#### **router-refresh：路由刷新**

用于在改变路由策略后，要求对等体重新发送指定地址族的完整路由表信息；只有支持路由刷新能力的路由器才会响应router-refresh报文。

地址族：典例：BGP版本不同，进入的地址族不同

BGP-----普通BGP，跑IPV4

BGP4+----适用于跑IPV6

MP-BGP---用于MPLS VPN的一个扩展版的BGP

组播版本的BGP

注：不管啥版本，都在同一个BGP进程中做配置，只是建立的邻居关系的方式不同而已。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEb414b6bef2a6871644e4fe8a67c3ac80/18625)



用于改变路由策略后：要求对等体重新发送指定地址族的完整路由表信息

## **ORF（Outbound Route Filters，出口路由过滤器）---了解**

设备希望只接收自己需要的路由

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE9f1cc2ec9ee94b4f85f7c0393bc64a98/44831)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEa874ca31bdbca138e31407bb6cd82d5e/44833)

IP所学的过滤方法，但R2仍然会传递4条路由给R1

```ada
[R1]acl 2000
[R1-acl-basic-2000]rule permit source 1.1.1.0 0.0.0.255
[R1]route-policy aa permit node 10
[R1-route-policy]if-match acl 2000
[R1]bgp 1
[R1-bgp]peer 10.1.12.2 route-policy aa import
```



```ada
ORF方法，彻底过滤
[R1]ip ip-prefix aa permit 192.168.0.0 24
[R1]ip ip-prefix aa permit 192.168.1.0 24
[R1]bgp 1
[R1-bgp]peer 10.1.12.2 ip-prefix aa import
[R1-bgp]peer 10.1.12.2 capability-advertise orf cisco-compatible ip-prefix send   //发送需求
[R2]bgp 2
[R2-bgp]peer 10.1.12.1 capability-advertise orf cisco-compatible ip-prefix receive   //接收需求
```



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEd77b07910a8cb5760f8163946608d482/44835)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEafc65eeee7259b5c9e3f9690fa099f96/44838)



# **八、BGP状态机**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE20907b2d64941af5adc52a39fd9f2041/18222)



Idle:空闲状态，停留30秒，初始化开始准备TCP连接并监视远程对等体，此状态持续30秒；

connect：TCP连接中状态，本端为TCP被动连接方，若连接建立失败则进入Active状态，反复尝试连接；

Active:活跃状态，本端为TCP主动连接方，TCP连接没建立成功，反复尝试连接；

open-sent：开始发送状态，成功建立TCP连接，发出open报文，open报文中携带参数协商对等体的建立，正在等待接受对方open报文；

open-confirm：开始确认状态，收到open报文，发出Keepalive报文，等待第一个Keepalive报文；

established：已连接状态，收到Keepalive报文，最终成功建立邻居；



# **九、BGP邻居建立条件**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE61021ca9142feb5071ea2079c12ea706/18223)



## **IBGP：**

物理口建邻：

1. 建议使用直连接口地址（物理地址）来指定IBGP邻居；

环回口建邻：

1. 对方接口要有IP地址，TCP可达（需要具有到达对方IP地址的路由），建议使用环回口地址来指定IBGP邻居；
1. 更新源地址必须和指定的邻居地址一致，需要修改更新源为环回；[R1-bgp]peer 2.2.2.2 connect-interface LoopBack 0
1. IBGP邻居关系不需要直连；



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE3654c57e2da9a3e96532f4d26120ac63/18224)



## **EBGP：**

物理口建邻：

1. 建议使用直连接口地址（物理地址）来指定EBGP邻居；

环回口建邻：

1. 对方接口要有IP地址，TCP可达，
1. 可通过修改EBGP最大跳数来使EBGP非直连；
1. 更新源地址必须和指定的邻居地址一致；

注意： 默认IBGP邻居间数据包的TTL值为255，EBGP邻居间TTL为1；故一旦使用环回建立ebgp邻居关系，必须修改TTL值，否则无法建立TCP三次连接

没修改跳数前：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEed62317d765f047b9af74880cf4f5b71/18225)

修改跳数后：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE28f6647ed01bb0109c715f897fcc10e0/18226)

[r4-bgp]peer  5.5.5.5 ebgp-max-hop 2    //修改TTL值（最大跳数值）



# **十、BGP基本配置**

1、启动BGP并创建BGP连接

[huawei]bgp 'as-number'     启动BGP

[huawei-bgp]router-id 'router-id'   配置router-id；（可选配置）

[huawei-bgp]peer 'ip-address' as-number 'as-unmber'    指定BGP对等体及AS号；



[huawei-bgp] address-family ipv4  unicast   创建BGP地址族，并进入相应地址族视图。----华三

[huawei-bgp-ipv4] peer ip-address enable   使能本地路由器与指定对等体交换路由信息的能力



2、优化BGP连接：

[huawei-bgp]peer 'ip-address' connect-interface ‘interface’    指定建立TCP连接使用的源接口；

[huawei-bgp]peer 'ip-address ebgp-max-hop 'hop-count'       指定EBGP对等体最大跳数；默认跳数是1，配置允许同非直接相连网络上的邻居建立EBGP连接



3、配置BGP生成路由

将本地路由发布到BGP路由表中：

[Router-bgp] network ip-address [ mask | mask-length ] [route-policy route-policy-name ]



引入其它路由协议的路由：

[Router-bgp] import-route protocol [ { process-id | all-processes } [ allow-direct | med med-value | route-policy route-policy-name ]  ]



4、查看命令：

[router]display bgp peer

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE41a7d7faa8e273aab4bfa030dba0e087/18227)

[router]display bgp routing-table

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCEb5753afa695e7d645b675a0bd7118c17/18228)

*-- 代表可用，可用路由会参与路由信息的优选

>--  代表优选

i:代表该路由信息通过IBGP对等体学到



起源码：一条路由是如何进入BGP路由表，一条BGP路由条目的来源

I:代表这条路由信息起源于AS内部，使用network通告的；

e:代表来自于EGP协议；

？：代表除了以上两种方式，其他获取的路由信息都是？



# **十一、BGP基础实验**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2609c08a0f1e8917a920f2380fc077f4/WEBRESOURCE9ecfabe80cdad1e0405f0618c181fde4/18229)