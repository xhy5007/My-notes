# **一、RSTP（****Rapid Spanning Tree Protocol，****快速生成树）**

## **1、背景：**

RSTP从STP发展而来，具备STP的所有功能，可以兼容stp运行

## **2、RSTP与STP不同点**

### **（1）减少端口状态**

**STP:**disabled\blocking\listening\learning\forwarding

**RSTP：**

discarding：不发送配置BPDU，不进行MAC地址学习，不收发数据

learning：发送配置BPDU，进行MAC地址学习，不收发数据

forwarding：发送配置BPDU，进行MAC地址学习，收发数据

### **（2）增加端口角色**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCEeda1a6bdce91e96d1929568d48b02514/31194)

**STP：**指定端口、根端口、阻塞端口

**RSTP:**

根端口：

指定端口：

Alternate端口：替代端口，根端口的备份

Backup端口：备份端口，指定端口的备份

总结：有替代端口后，就算网络拓扑发生变化，也不会发生重新选举STP的情况，白白浪费30秒



### **（3）BPDU格式不同**

RSTP中：

协议版本ID变为2；

BPDU type变为2；

使用了Flag字段的全部8位，而STP只使用了0号和7号两位；

增加version  1 length字段；（为了兼容STP）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE5143bac69d79bb7455d1f297e974b08a/31189)

### **（4）BPDU处理方式不同**

1. 每台交换机都能从指定端口发出RST BPDU，发送周期为hello time。不需要等待来自根桥的RST BPDU；
1. RST BPDU老化时间（max age）为3个连续的hello time时长；

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE680597f9bb54f55e5f7812da81fb8336/13906)

1. 阻塞状态收到低优先级RST BPDU的处理：会立即做出回应；

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE99469a2885045bc7b29acf919e1c5953/13911)



分析：当SWB和SWA的链路down时，SWB就暂时收不到来自根桥的RST BPDU，此时它认为自己是根桥。会向SWC发送RST BPDU告知自己是根桥，SWC收到后就很惊讶，明明根桥好着呢，然后会向SWB发送RST BPDU告知它，SWA是根桥。这就是阻塞状态收到低优先级RST BPDU的处理：会立即做出回应；



### **（5）当网络拓扑发生变化时，RSTP可以更快地恢复网络连通性**



## **3、RSTP的快速收敛机制**

### **（1）边缘端口机制：**

定义：直接和终端相连的端口

边缘端口可以直接进入转发状态，不需要延时，并不会触发拓扑改变，不会成环；

边缘端口收到BPDU后，会转变为非边缘端口，重新参与生成树计算；

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE56c3bdf4e18f9d669ddc11046c3b921c/13938)



### **（2）根端口的快速切换**

根端口故障后，如果新的根端口对端的指定端口处于转发状态，则新的根端口立即进入转发状态，不需要向stp一样等30秒切换时间

过程：

网络收敛好后：（被阻塞的口是AP，根端口的替代端口）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE804a9ab03b120a8a14346e635b1bd6b5/13953)

当SWA和SWC之间的链路故障时：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE33027f8d792df5fd07bcf1a31e15641a/13956)



### **（3）指定端口的快速切换--P/A机制**

握手请求报文：proposal

握手回应报文：agreement

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCEedbf7916cde00290361218300224f572/13964)

1. sw1新增链路或故障链路恢复，指定端口进入转发状态前，向对端sw2发送**proposal报文（把P位置位为1的BPDU报文）；**
1. sw2收到proposal报文后，立刻进行同步操作，即把除了边缘端口和收到p置位的BPDU报文端口外，其他所有端口阻塞掉，同时防止临时环路；
1. 同步完成后，向对端sw1发送Agreement报文；
1. 收到Agreement报文后，sw1的指定端口进入转发状态；
1. sw2上被同步阻塞的端口继续进行P/A流程，直至整个网络收敛

临时环路的产生：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE64a3b0fb6580b2211e8eecc19d0824b8/31193)

‘

分析：sw2收到proposal报文后，立刻进行同步操作，即把除了边缘端口和收到p位置的BPDU报文端口外，其他所有端口阻塞掉。如果不阻塞掉，假设SW3和SW1相连，会在物理上形成一个环路



**注：根和指定端口的快速切换有一个共同点：根端口选举好后，会跳过listening和learning，直接进入forwarding**



**RSTP故障切换时间一般是1秒左右 **



## **4、RSTP拓扑改变机制**

（1）STP与RSTP拓扑改变机制区别：

1. STP需发TCN BDPU到根桥，由根桥发送TC置位的BPDU报文给其他交换机，而RSTP是直接发送TC置位的 BPDU报文给其他交换机；
1. 收到TC置位的BPDU，RSTP清除其他所有端口学习的MAC地址，而stp是缩短MAC地址老化时间从300至15；



（2）注意：**由于每台交换机都可以主动发起RST BPDU,所以取消了TCN机制**



## **5、RSTP和STP的兼容**

RSTP端口连续三次收到版本为STP的BPDU，则端口协议直接切换到stp协议；

切换成STP协议的RSTP端口将丧失快速收敛特性；

出现stp与rstp混用的情况，建议将stp设备放在网络边缘；

运行stp协议网桥移除后，由RSTP模式切换到stp模式的端口仍将运行在stp模式；



## **6、基本配置**

【h3c】stp global enable   全局模式下，开启STP功能

【H3c-ethernet0/1】undo stp enable  如果确定某个端口不会存在环路，就可关闭该接口的STP功能

[h3c]stp mode{stp|mstp|rstp|pvst}  配置生成树的模式

【H3c-ethernet0/1]stp edged-port  enable  配置边缘端口

【H3c-ethernet0/1]stp cost cost值   配置端口的COST值

【h3c】stp priority  优先级值

【h3c】stp pathcost-standard {dot1d-1998|dot1t|legacy}  改端口的开销标准

【h3c】stp timer hello 配置hello时间

hello time时间过长会造成，生成树计算消耗，对链路故障迟缓

hello time时间过短，会造成频繁发送配置消息，加重CPU和网络的负担

【h3c】stp timer max-age 时间

max age时间过长会造成，对链路故障迟缓,不能及时被发现；

max age时间过短会造成，在网络拥塞的时候交换机误认为链路故障，造成频繁的生成树重新计算；

【h3c】stp timer forward-delay 时间

时间过长会造成，生成树收敛太慢；

时间过短会造成，在拓扑改变的时候，引入暂时的路径环路；



# **二、MSTP（****Multiple Spanning Tree Protocol，多生成树协议****）**

## **1、MSTP产生的背景**

STP与RSTP的局限：

1. 两个都是单一生成树，所有VLAN共享一颗生成树，造成资源浪费；
1. 无法实现不同VLAN在多条trunk链路上的负载分担；

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCEbfe30d5a69aa9e1d21cf8e01f41d249a/13999)

## **2、定义：**

基于实例计算出多颗生成树，实例间实现负载分担

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE8a06773fb934d577ecf2cb10562f15a0/14002)



## **3、MSTP基本概念**

1. MSTP分区域的好处：每个区域独立计算自己的生成树，减少网络拓扑层次，生成树计算时间大幅度缩短，方便每个区独立管理；
1. MST域：拥有相同MSTP配置标识的交换机构成的集合，划区域可加快收敛速度和方便管理，相同域的必要条件；

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE6d3b883ab11351af416ebc14636ead98/14010)

1. IST：内部生成树，默认存在，每个MST域独立计算的内部生成树实例
1. CST：公共生成树，默认存在。用来互联MST域的单生成树（把每个MST域作为一台交换机，计算出生成树实例）
1. CIST：公共内部生成树，默认存在。CST+每个域内部IST。
1. 实例0：默认所有VLAN都映射在实例0
1. MST：手动创建的生成树实例，只在一个区域内有效
1. CIST域根：每个IST的根网桥
1. CIST总根：整个CIST的根网桥

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE71020cf9aa3c43e741a314076853c8a7/14012)

1. master端口：CST的根端口，单域MSTP中不可能存在Master端口，多域MSTP中，根域不可能存在Master端口，其他域只有一个；

MSTP 与RSTP的端口角色一样，新增了一个master端口

思考：只有一个域，还存在master端口吗？

答案：不存在



## **4、MSTP的BPDU格式=RST BPDU+MST专有字段（了解）**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCEd6e0fe7d6962597f17bb96239c98ef51/14026)



## **5、MSTP计算过程（了解）**

先选举总根，再选举域根；

CIST和MST的计算是同步进行的；

在对比优先级向量时，先对比外部向量，再对比内部向量；



MSTP计算结果

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE8368b8aadae246a4922377c3de9f9aa4/31191)

## **6、MSTP中的P|A机制**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE54edcf1563cd8b8468ba77913ba3049d/14037)

上游桥发送proposal BPDU中，P、A标志位都置为1；

下游收到P标志位和A标志位的proposal BPDU，再将端口同步后会回应Agreement BPDU,此时A标志位为1，使得上游的定端口快速进入转发状态



特殊情况：SW1给SW2发送  P置位的BPDU，但SW2只认识P、A都置位的BPDU，所以这种情况下P/A机制失败了。失去了快速收敛的特性，只能通过30秒STP计算收敛网络。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE3c7a5bbcdad544b0993ed855e16a61b4/31192)

解决RSTP和MSTP的P/A机制不兼容问题：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCEe66801e460810fbd7ea4d930705a891c/14051)



## **7、MSTP兼容性**

1. 当上游交换机是RSTP（根桥），当RSTP做快速转发时，只能发送P置位为1的BPDU报文，而下游的MSTP收到后，不会有任何反应，因为MSTP需要p|A双置位，会导致上游交换机收不到回复，不能进行P|A机制，不能快速收敛，只能等30秒收敛；
1. 解决MSTP与RSTP兼容性用此命令：在下游设备设置【sw1-ethernet0/1】stp no-agreement-check；



1. 【sw1-ethernet0/1】stp compliance{auto|dot1s|legacy}   配置MSTP的兼容性

auto  自动识别

dot1s  标椎格式

Legacy  与非标准格式兼容的格式



1. 摘要侦听（了解）：

全局开启：[h3c]stp config-digest-snooping

端口开启：[h3c-ethernet1/0/1]stp config-digest-snooping

其他厂商设备可能用私有秘钥计算配置摘要

开启摘要侦听使设备不再对比配置摘要，但会导致，只要域名和修订级别一致，VLAN映射关系不一致，也能属于同一个域



# **三、STP保护机制**

## **1、BPDU保护：**

针对边缘端口的保护：

技术背景：若边缘端口收到配置消息，将会转变为非边缘端口新参与生成树计算；使用pc伪造的BPDU报文恶意攻击，将导致频繁参与生成树计算，导致网络不稳定。

措施：启动BPDU保护功能后，如果边缘端口收到了配置消息，MSTP就将这些端口自动关闭。

配置命令：[sw1]stp bpdu-protection

[sw1-ethernet1/0/1]stp edged-port

## **2、根桥保护：**

技术背景：合法根桥收到优先级更高的BPDU，将失去根桥的角色，并重新引起STP计算

解决方案：在指定端口角色下设置根桥保护端口，一旦收到优先级更高的BPDU，则立刻将端口设置为lestening状态，不再转发。

配置命令:[sw1-ethernet1/0/1]stp root-protection

## **3、环路保护：**

技术背景：环形链路拥塞，导致没及时收到BPDU，从而重新计算STP、开启阻塞端口，形成逻辑环路

解决方案：配置环路保护端口，当接受不到对端交换机的BPDU时，若端口参与了STP计算，则该端口进入discarding状态。

配置命令：[sw1-ethernet1/0/1]stp loop-protection

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE2957dc56a6d898f98712e897940b2005/36038)

## 

## **4、TC保护：**

技术背景：在有伪造的TC BPDU报文恶意攻击设备时，设备短时间会收到很多TC BPDU报文，频繁的删除操作会给设备带来很大负担，给网络的稳定带来很大隐患。

解决方案：

设置设备在收到TC BPDU报文后的10秒内，记性地址表项删除操作的最大次数；

监控在该时间内收到的TC BPDU报文树是否大于门限值；

配置命令:[sw1]stp tc-protection

[sw1]stp tc-protection threshold 数目



# **四、MSTP实验**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBdc40c9ab7b5727f5409af5e4c250e3d7/WEBRESOURCE487f57be87708d7da26e7a71e68497bb/31190)