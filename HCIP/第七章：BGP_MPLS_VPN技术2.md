## **一、MP-BGP技术---多协议BGP**

### **1、选用BGP来传递MPLS VPN私网路由：**

1. BGP基于TCP协议传输，可跨越设备建立邻居，传递路由；
1. 可扩展多种属性，以便于携带更多的表明路由特征的信息；
1. 便于控制路由的传播范围。



### **2、MP-BGP**

（1）产生背景：在原有的BGP协议的基础上对其进行一些扩展，形成MP-BGP，以便在MPLS  VPN上跑MP-BGP协议

（2）MP-BGP的协议扩展

1. 普通的BGP协议通过UPDATE报文来跟新和删除路由，update的结构如下：



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCE854518a59203b045f1b247ffc05a263c/27867)

Unfeasible routes length：标明Withdrawn Routes部分的长度。其值为零时，表示没有撤销的路由。

Withdrawn Routes：包含要撤销的路由列表

Total Path Attribute Length：标明Path Attributes的长度。其值为零时，表示没有路由及其路由属性要通告。

Path Attributes：路径属性：包含要更新的路由属性列表

Network Layer Reachability Information（NLRI）：包含要更新的地址前缀列表

1. MP-BGP中新增的两个属性：

使用来**MP_REACH_NLRI**代替原UPDATE消息中的NLRI（携带的路由信息，希望对方学习到新路由）字段

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCE5dd616dc5d44076a6df60b1e14da559e/27868)



使用**MP_UNREACH_NLRI**来代替原UPDATE消息中的Withdrawn routes（删除路由）字段

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCEc3fb1b1996ddf56e1135819f8e5c7b9a/27869)

增加地址族信息；

Withdrawn routes：RD+IPV4前缀/前缀长度。



### **3、RD（route distinguisher）**

（1）简介：

RD：路由区分，用于标识路由更新报文来自于哪一个VPN实例。**完美解决私网地址冲突问题**。如下图：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCEe73d614c938751d2f43565326d40015e/27870)

（2）RD作用：用来表示路由更新报文来自哪一个VPN实例，完美解决私网地址冲突问题

（3）RD的格式（3种）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCEae7e72dbcb27d71393234b8b110f8f6a/27871)

16位自治系统号：32位用户自定义数，例如：100:1

32位IP地址：16位用户自定义数，例如：172.1.1.1:1

32位自治系统号：16位用户自定义数字，例如：75536:1



（3）RD的本质

RD的本质是区分私网路由的撤销，因为BGP撤销路由时无法携带RT，导致PE在删除路由时无法判断是要撤销哪个VPN的路由

（4）注意：

在大多数情况下，两端PE设备的RD无需一致。因为：

1. **VPN路由的互通依赖于RT（Route Target）**：RT控制VPN路由的导入/导出规则，决定哪些路由可以共享给对端PE。
1. RD仅在本地PE有效，不会传递给对端PE（对端PE收到路由时，会剥离RD）

何时需要保持RD一致？

1. **跨域VPN（Inter-AS Option B）**：某些跨域部署中，如果路由需要重新标记RD（如ASBR重写RD），需确保域间协调。

### **4、RT（route target）**

（1）本质：是每个VPN实例表达自己的路由取舍及喜好的方式，用于控制传播范围

场景：实现总部与分部之间互访、分部与分部之间不互访，RD做不到

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCEb4b948659d7dd4aaf1629b11d216777a/30630)



（2）简介：RT由export target和import target两部分组成；

在PE设备上，发送某一个VPN用户的私网路由给其BGP邻居时，需要在MP-BGP的扩展团体属性区域中增加该VPN的Export Target属性；

在PE设备上，需要将收到的MP-BGP路由的扩展团体属性中所携带的RT属性值，与本地每一个VPN的Import Target属性值相比较，当这两个值存在交集时，就需要将这条路由添加到该VPN的路由表中去。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCE8f45fff45a8207bcb2e040e2d93487e2/27874)



（3）RT格式（3种）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCEeb143507ddaf2fa6c557a063d7c78fea/27875)

16位自治系统号：32位用户自定义数，例如：100:1

32位IP地址：16位用户自定义数，例如：172.1.1.1:1

32位自治系统号：16位用户自定义数字，例如：65536:1



（4）注意：

当我们在创建一个VPN实例时，每个路由器上都要配置RD、出方向RT、入方向RT；

同一个PE设备上VPN实例中的RD一定不要配置相同，不同的PE设备上同一个VPN实例中，RD可以相同，也可以不同。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCE11f483305f400153454c962521c719ec/27876)

（5）使用RT可以实现更加灵活的VPN访问控制。因为RT可以配置多个属性，接受时表现得是“或”属性

场景2：site 1和site 3都能访问site 2，site 1和site 3之间不能互访

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCE1368bc4205684b2c95935e78b7d82dbc/33028)



```ada
实现效果如下：
如图，Site 2作为能被VPN1和VPN2访问的共享站点，需要保证：
PE2能够接收PE1和PE3发布的VPNv4路由；
PE2发布的VPNv4路由能够被PE1和PE3接收；
PE2不把从PE1接收的VPNv4路由发布给PE3，也不把从PE3接收的VPNv4路由发布给PE1。
```

\

### 

### **5、私网标签**

（1）特点：

1. 需要在数据报文中增加一个标识，以帮助PE判断该报文是去往本地的那个VPN实例。
1. 每个VRF会自动产生私网标签，同路由更新一起传递至对端PE
1. 由于MPLS支持多层标签的嵌套，这个标识可以定义成MPLS标签的格式，即私网Label。数据进入MPLS网络时，先封装私网标签，再封装MPLS公网标签。

（2）工作过程

1. 一旦VPN实例创建好，双方在相互传递路由的时候，每个VPN实例会自动随机给自己产生一个私网标签；

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCE338c172e8a0f6c5aadbf96968b6b5586/30647)

CE1 ping  CE3时，数据包的传递过程：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCE6322e8437b30aebc59195a9555884377/30668)



## **二、BGP MPLS VPN基本原理**

### **1、不带RR**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCE2cf2193a082055ff88241f022c8da50d/27987)



工作流程：

（1）公网隧道的建立

首先要到达公网全网通（动态、静态路由协议随便选），主要是PE设备上的环回口要通

公网路由器之间需要配置MPLS，建立LDP邻居。此目的主要是为了分别建立一条到1.1.1.1/24和2.2.2.2/24的LSP

（2）本地VPN的建立

在PE设备上划分VPN实例，并将相应端口绑定到VPN实例中;

手动给每个VPN实例配置RT和RD;

（3）私网路由的传递-----MP-BGP实现----本端PE到对端PE

（4）PE与CE之间运行路由协议多实例，将PE路由传递至CE

（5）私网数据的传递过程：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCE8cf5a42bb4869da0569fef3bfbfd90fe/27991)



### **2、带RR**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBf58d6a94dfea559bfbde34db623b11b8/WEBRESOURCE3d628777013ee24957de2f92ea7797e8/37496)

```abap
[R2]bgp 100
[R2-bgp]peer 1.1.1.1 as 100
[R2-bgp]peer 3.3.3.3 as 100 
[R2-bgp]peer 1.1.1.1 connect-interface l0
[R2-bgp]peer 3.3.3.3 connect-interface l0
[R2-bgp]ipv4-family vpnv4
[R2-bgp-af-vpnv4]peer 1.1.1.1 enable 
[R2-bgp-af-vpnv4]peer 3.3.3.3 enable 
[R2-bgp-af-vpnv4]peer 1.1.1.1 reflect-client 	
[R2-bgp-af-vpnv4]peer 3.3.3.3 reflect-client 
```

## **三、domain id**

domain id一样，则原来1、2、3类LSA，当作3类引入，5类当作5类引入。

domain id不一样，则为5类引入。

## **四、BGP MPLS VPN配置命令**

1、配置公网公共隧道

[PE1]mpls lsr-id 1.1.1.1 相当于OSPF的router-id作用

[PE1]mpls

[PE1]mpls ldp 系统模式下，使能mpls ldp

[PE1-GigabitEthernet0/0/1]mpls  //在接口下开启mpls功能，每个参与标签传输的接口都要开mpls

[PE1-GigabitEthernet0/0/1]mpls ldp  //在接口下开启mpls LDP功能



2、本地VPN的建立

（1）创建VPN实例，进入VPN视图

[PE1]ip vpn-instance h3c

（2）配置RD和RT

[PE1-vpn-instance-h3c]r:oute-distinguisher 1001

[PE1-vpn-instance-h3c-af-ipv4]vpn-target 100:1 import-extcommunity

[PE1-vpn-instance-h3c-af-ipv4]vpn-target 100:1 export-extcommunity

（3）配置接口与VPN实例绑定（此接口是CE与PE相连的PE端的接口），同时给接口配置IP地址

注意：在本地VPN配置之前，PE端的接口可以配置任何命令，只有等接口与VPN实例绑定后，才能进行一致配置，比如：接口IP地址的配置

[PE1]int g0/0/0

[PE1-GigabitEthernet0/0/0]ip binding vpn-instance h3c

[PE1-GigabitEthernet0/0/0]ip add 10.1.1.2 24



（4）配置PE与CE之间的路由实例与VPN绑定，并宣告PE与CE之间的网络，把私网路由从CE传递至PE

以OSPF为例：

[PE1]ospf 100 vpn-instance h3c

[PE1-ospf-100]area 0

[PE1-ospf-100-area-0.0.0.0]network 10.1.1.0 0.0.0.25



3、配置MP-BGP，穿越公共网络建立BGP邻居

（1）配置PE之间的MP-BGP邻居

进入BGP-VPNV4地址族视图，在PE1上创建BGP进程，与PE2使用Loopback0口建立IBGP邻居，并在vpnv4地址族视图下使能邻居

[PE1]bgp 100

[PE1-bgp]peer 3.3.3.3 as-number 100

[PE1-bgp]peer 3.3.3.3 connect-interface LoopBack 0

[PE1-bgp]ipv4-family vpnv4

[PE1-bgp-af-vpnv4]peer 3.3.3.3 enable

（2）配置本地VPN路由与MP-BGP之间的路由引入引出

在PE上配置BGP和各OSPF实例的路由互相引入，来把私网路由传递到对端站点

配置命令如下：

**将OSPF进程引入到BGP**

[PE1]bgp 100

[PE1-bgp]ipv4 vpn-instance h3c

[PE1-bgp-h3c]import-route ospf 100

**将BGP进程引入到OSPF**

[PE1]ospf 100   vpn-instance h3c

[PE1-ospf-100]import-route bgp