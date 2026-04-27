# **一、STP协议出现背景（****Spanning Tree Protocol，生成树协议****）**

二层环路带来的问题：广播风暴；

MAC地址表的震荡；

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCEf7ec6e9a91bbf0d57f7a7d0e1396e3db/13646)



分析1：环路和广播风暴的产生

假设PC1发送了一个广播帧，SW1收到后后广播给PC2，然后通过g0/0/1口广播给SW2的g0/0/1，SW2又会广播给PC3和PC4，从g0/0/2口广播回给SW1。SW1又继续广播给它的其他接口，通过从sw1的g0/0/1广播出去给SW2。这时就形成了环路。相当于数据在两个设备间的g0/0/1和g0/0/2之间循环传递，走不出去了，这同时也会形成广播风暴。



分析2：MAC地址表的震荡

假设PC1发送了一个广播帧，通过g0/0/1口广播给SW2的g0/0/1。同时SW2交换机会进行MAC地址的学习。从g0/0/1口收到了PC1的广播帧，在MAC地址中存在这个记录：PC1的MAC地址和g0/0/1口相连；

SW2收到此广播帧后会广播出去。此时SW2的g0/0/2口也会收到该帧，在MAC地址中存在这个记录：PC1的MAC地址和g0/0/2口相连；

但是一个MAC地址只能对应一个接口，所以对于上述情况，交换机会修改MAC地址表。比如当SW2的g0/0/2口收到此广播帧的时候,将MAC地址表的记录修改下：PC1的MAC地址和g0/0/2口相连。当SW2的g0/0/1口收到此广播帧的时候,将MAC地址表的记录修改下：PC1的MAC地址和g0/0/1口相连。

这种反复修改的操作，叫做MAC地址表的震荡



# **二、STP定义**

stp是二层网络中用于消除环路的协议，通过阻断冗余链路来消除，网络中可能存在的环路；

当前活动路径发生故障时，激活冗余备份链路，恢复网络连通性；

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCEc28d2753bd1774e5c9ee38940e8808f9/13789)

分析：STP会临时阻塞冗余链路的端口，被阻塞的端口不能收发数据，这时，就不会形成环路。

当主链路DOWN，被阻塞的链路会被启用，保证数据正常转发。

思考：阻塞端口怎么选？如下图，我们肯定希望阻塞10Mbps的。但阻塞谁，肯定有一个阻塞机制。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCEa72516619c1a56929ad3ecf2f0425683/13711)

# **三、STP相关概念**

配置BDPU：

1、定义：桥协议数据单元，用于传递STP协议信息相关报文

2、分类：

（1）配置BPDU：传递STP配置信息

1. 网桥通过交互配置BPDU，获取STP计算所需的参数；
1. 配置BPDU基于二层组播方式发送，目的地址：01-80-C2-00-00-00；
1. 配置BPDU只由根桥周期性发出，发送周期为Hello Time（2秒）；

配置BPDU格式：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCEb9dc76801a5559137169f7ba663c2f79/13870)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCEc0c18c7cf3974e018aa3ab32abd135ea/41034)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCEbc9b40309b8512541aa1face3692a74b/13909)

1. 配置BPDU老化时间为Max Age即10个hello time；
1. 总结：配置BPDU是传递生成树选举所需信息的报文



（2）TCN BPDU：通告拓扑变更信息

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCEb6067963edd09ad8ff8b282b7d500956/13873)

**注意区分：TC BPDU（根桥发出的拓扑变化信息，其他交换机收到后会将MAC地址表老化时间由300秒缩短到15秒）、**

**TCA BPDU（拓扑变化确认），TCN BPDU（非根交换机发的拓扑变化通知，用于报错使用） 两者是属于配置BPDU中不同的flag置位**



# **四、STP选举机制**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCEf5879c41181d991785cd87be99fada69/24114)

BID中的优先级范围：优先级的数字越小，优先级越高，MAC地址越小，MAC地址的优先级越高

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCE592a7a21fbb0754e471d1e3265ff5279/21931)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCE2726503091bf02311999b08103d2f01f/13868)

**网络拓扑：**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCE51894d2ed9a61e66ce90ae82ff12b7e6/30987)



**根网桥和非根网桥选举情况：**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCEafb4d7b5820033a1e8a0499159d9a98c/30988)



**指定端口的选举情况**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCEd5f7f1dbb9b928f999d54dfefb5a3060/13771)



**阻塞端口：**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCE6bada7c13474bc767bb6426715253bcb/13774)

STP选举完后，修剪的网络拓扑，一颗没有环路的树

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCE2728daa24887019b94cf46a7a0c4bdb1/13776)

注：选举根桥和端口角色同步进行，不分先后；

stp计算时间是30秒；

总结：STP桥角色：根桥、非根桥

端口角色：指定端口、根端口、阻塞端口

port id：格式：优先级+端口号，数字越小，优先级越高

优先级0-240，默认128，必须是16的倍数



# **五、stp初始化流程---端口状态**

disable：禁用状态，被手动shutdown的端口。不发送配置BPDU（也不能接收到来自根桥的配置BPDU），不进行MAC地址学习，不收发数据。

blocking：阻塞状态：不发送配置BPDU（但是能接收到来自根桥的配置BPDU），不进行MAC地址学习，不收发数据。

listening：监听状态，发送配置BPDU，不进行MAC地址学习，不收发数据，持续15秒。

learning：学习状态，发送配置BPDU，进行MAC地址学习，不收发数据，持续15秒。

forwarding：转发状态，发送配置BPDU，进行MAC地址学习，收发数据



发送延迟的过程：

从中间状态listening经过一个延迟进入另一个中间状态learning；

从中间状态learning经过一个延迟进入另一个中间状态forWording



思考：当交换机A和B之间的链路down，故障切换（网络收敛）需要多长时间？

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCEe6da3adc3c64a8d4fc65773a8018fac5/13797)



答案：需要30秒，因为上图中虽然有很多的阻塞口，可到底启用哪个口，就需要重新进行STP的选举过程。



# **六、STP计时器**

hello time：2秒，配置BPDU的发送周期

max age：20秒，判断链路故障的时间，10个hello time周期

forwarding delay：15秒，状态切换延迟



STP收敛时间总结：

（1）初次收敛时间：30秒（15侦听+15s学习）:存在直连检测=本地仅存在一个阻塞端口，可以收到来自根桥的BPDU

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCEce865c22dbdf9724c95560af6458b66c/24123)



分析：SW0上2口是阻塞口，当其上1口链路故障，2口只需经过30S就可以故障切换，因为它可以收到来自根桥的BPDU，所以不需要经过额外的20秒的故障检测机制。

（2）拓扑故障再收敛时间：若某个端口断开，将发送次优BPDU（以本地为根）给其他邻居交换机，其他交换机无视该数据，进行20s  max age计时，同时阻塞接口进入15s侦听，15s学习，故总50s

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCE4006725006d004eceeb47b01e303af27/24129)



# **七、STP拓扑变更机制**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCE3c3dc93954755a6c6cf49438b937149b/13814)



2---B

分析：主机B发送一个数据，SW1会从2口收到，在SW1的MAC地址表中会写进去这么一条记录：2口对应主机B的MAC地址

思考：SW2和SW4连接的链路down后，SW4的阻塞口2会被启用。但是这个故障切换对于SW1来说，并不知道，

假设现在主机A访问主机B，还能正常访问吗？

答案：不能，SW1通过查MAC地址表，发现要从自己的2口转发数据，数据到达SW2后，发现SW2和SW4连接的链路down了，所以数据传不到主机B

如果没有STP协议，则需要300秒的老化时间，才能更改SW1中的MAC地址表的记录为：3口对应主机B的MAC地址



**所以：拓扑变更后，要及时通知给上行交换机**



工作原理：

1.交换机检测到拓扑变更，交换机向根网桥发起TCN BPDU；

max age超时/有接口变更为转发状态，判断为拓扑发生变更

交换机上有端口从forwarding或learning状态转变为blocking；

2.沿途的非根桥收到TCN BPDU后，会继续向根桥转发，并在转发根桥的下一轮配置BPDU中，把TCA位置位，用于向下游交换机确认已经收到了TCN BPDU；

3.根桥收到TCN BPDU后，在下一轮的配置BPDU中，把TC位置位；

**4.所有交换机收到TC置位的BPDU后，MAC地址表老化时间由300秒缩短到15秒；**



**TC BPDU（根桥发出的拓扑变化信息，其他交换机收到后会将MAC地址表老化时间由300秒缩短到15秒）、**

**TCA BPDU（拓扑变化确认），**

**TCN BPDU（非根交换机发的拓扑变化通知，用于报错使用） 两者是属于配置BPDU中不同的flag置位**



# **八、缺点**

拓扑变更不灵活，主机频繁上下线，网络会产生大量TCN BPDU，导致网卡；

收敛时间长：拓扑层次越多，收敛时间越长，一个端口从blocking到forwarding至少需要30秒；

故障切换时间太长；



# **九、链路开销标准**

链路速率：100Mbps\1000Mbps\10Gbps

标椎：802.1D-1998

802.1t（华为标准）

legacy :私有标准（华三交换机私有）



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCE181070e4f3683cb77c228b15527198ce/15025)



# **十、STP实验**



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB26b0f1e41918aff2fdf5bb8712946c92/WEBRESOURCEa2f643635ac7b5824ad58a95882d3800/13856)