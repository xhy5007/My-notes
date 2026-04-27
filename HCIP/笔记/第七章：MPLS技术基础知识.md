## **一、MPLS技术产生背景**

1、IP采用最长掩码匹配原则，需要多次查表，效率不高

2、当时路由器多采用CPU 处理数据转发，性能有限

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCE9944f97d8d3d557f22e9971f733b1f92/27858)

3、ATM技术（异步传输网络）采用唯一精确匹配，一次查表，效率高但协议复杂、成本高

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCE3dd10811ebb8d4b545e0fa099018ca64/27859)



## **二、MPLS网络组成（基本概念）**

### **1、MPLS技术简介：Multiprotocol  Lable  Switching，多协议标签交换技术**

1. 它可承载在各种链路层协议上，如：PPP\ATM\帧中继\以太网等
1. 原理上各种报文也可以承载在MPLS之上，如IPV4\IPV6、也包括各种链路层报文如ATM等
1. 采用一个短而定长的标签来封装网络层，交换机或路由器根据标签转发数据

MPLS的转发数据的思想：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCE88b0da18a8c82b72916200176a39d7b0/27885)



### **2、MPLS网络组成**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCEdc08f75e28f163238214d01fe98b8267/8963)

（1）基本概念：

1. 非MPLS网络：没有运行MPLS的网络，如;普通网络
1. MPLS网络：使用支持MPLS协议传输数据的网络设备

注：MPLS网络和非MPLS网络可兼容，报文可在MPLS网络和非MPLS网络之间平滑过渡。

1. LSR：Lable Switching Router，标签交换路由器，指运行了MPLS协议的网络设备，LSR具有标签分发和标签交换能力，是MPLS网络中的基本元素。
1. Transit LSR：中间节点LSR，根据标签沿着由一系列LSR构成的LSP将报文传送给出口LSR。
1. ingress LSR：入节点LSR，指数据报文的入口LSR，负责为进入MPLS网络的报文添加标签
1. Egress LSR：出节点LSR，负责报文中的标签，并转发至非MPLS网络

注意点：出入节点根据数据流的方向划分，边缘LSR可能既是入节点，也是出节点

1. FEC：Forwarding Equivalence Class，转发等价类，MPLS将具有相同特征的报文归为一类，称FEC

例如：  一组按照同一LSP路径转发的数据流。

拥有相同的QOS优先级。

1. LSP：Lable Switching Path，标签交换路径，同一个报文在MPLS网络中经过的路径。



## ** 三****、MPLS的优势**

1. 转发数据唯一精确匹配，查询速度快
1. 支持多协议，无需改换专用设备



## **四、MPLS标签**

### **1、定义：**

1. 一个短而定长的标识，通常具有局部意义，所谓的局部意义，就是某个标签值并一定所有路由器都认识，所以可撕标签重复使用。
1. 标签通常位于二层和三层的头部之间
1. LSR根据MPLS标签决定如何转发数据

### **2、标签结构：**



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCE0504366b84fd35e14553f8fbdc9f0379/9044)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCEceebb722bdc1b3a4e557aef5d4259689/9017)

标签只有4个字节，32个bits

分为4个区域：

（1）label：标签值，长度20bits，是标签转发的关键索引。

0-15为保留标签：0表示该标签必须弹出，交给IPV4处理；2表示该标签必须弹出，交给IPV6处理；3表示倒数第二跳弹出

16-1023为静态标签

1024-65536为动态标签

（2）TC位：Traffic Class field，流量类别字段，用于QOS标识优先级，长度3bits，数字越大，优先级越高

EXP ，Experimental Use，实验性使用字段，预期用途是作为**“服务等级”(Class of Service，CoS) **字段

注意：TC和EXP所表示的意思是一样的，有的文档里用的EXP，有的文档里用的是TC，现在EXP”字段被重命名为“TC”字段

（3）S：栈底标识，长度1bits；

S为1表示为最后一个标签；

S为0表示后续还有标签。这就意味着我们可以多次封装标签，嵌套标签。这在MPLS VPN和BGP MPLS VPN中会被使用，如下图：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCE9798de2b222167eac12faa14c08896a7/9054)

（4）TTL：存活时间，长度8bits，用于当网络出现环路时，防止标签报文被无限制转发。

它有两种处理模式：

Uniform：IP报文进入mpls网络时，拷贝IP头部的TTL至标签交换，每经过一次标签交换，标签TTL-1,经过出节点时，把标签TTL再次-1后替  换到原IP头部的TTL

pipe：IP头部进入MPLS时，IP头部TTL-1，MPLS标签中的TTL为固定值，每经过一次标签交换，标签TTL-1，直到经过出节点时，将IP头部TTL-1

这两种模式最大的区别在于Uniform可以使接收设备感到TTL值的变化，可以知道自己经过了几个路由器，而pipe做不到。

### **3、标签识别：**

以太网帧中，通过Type字段对MPLS进行识别

Type=8847,代表承载的是MPLS报文

Type=0800,代表承载的是IP报文

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCE75124332ee7aa2b711a5bba170b70f5c/9078)



## **五、标签分配协议---LDP（Lable Distribution Protocol）**

### **1、定义：**

用于在LSR之间分配标签，建立LSP，简单可靠，是MPLS网络中应用最广泛的标签分配协议之一。可手工配置LDP产生标签，也可动态生成LDP标签。

### **2、标签分配协议的种类：**

（1）LDP

（2）RSVP-TE

（3）MP-BGP----专门在BGP网络中，支持标签分配协议的，适合用IPV4

（4）MP-BGP(BGP4+)----专门在BGP网络中，支持标签分配协议的，适合用IPV6

（5）CR-LDP（基于约束的标签分配协议）

### **3、LDP消息类型**

发现消息（discovery messages）：用于LDP邻居的发现和维持。

会话消息(session messages)：用于LDP邻居会话的建立、维持和中止。

通告消息(advertisement messages)：用于LDP实体向LDP邻居宣告Label、地址等信息。

通知消息(Notification messages)：用于向LDP邻居通知事件或错误。

### **4、****LDP会话建立和维护**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCEf1c7cfbdd148b4d26ddf58f288932d79/9102)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCEff012502a1544d79c1417017a6d48c11/37463)

### **5、标签转发表**

LDP会话建立完成后，路由器根据路由表进行标签分配，形成MPLS标签转发表

标签转发表包含入标签、出标签和出接口

入标签：接收到的报文携带的标签

出标签：转发数据把入标签替换为出标签

出接口：报文数据发出的接口

### **6、LSP建立流程（标签分配的过程）**

上游与下游：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCEa295b1370a589ece0e34f9e3226ada95/9128)

设备的上下游，与数据转发的方向相对，数据先到达的地方是上游，后到达的地方是下游。

流程：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCEcf96af0e1c1b3e9febda0308411708b3/27893)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCEd1cf515e7c1642e1c4c1520c3a5ace03/27897)



**总结：**

（1）出节点LSR收到上游标签分配请求后，建立LSP

1. 出标签为空
1. 入标签设置为3或者0或者2，视情况而定
1. 出接口为IP路由表中目的网段的出接口

（2）出节点LSR向上游LSR发布标签映射消息，通告本机LSR的入标签

（3）上游LSR根据标签映射消息建立LSP

1. 出标签为下游LSR通告的入标签
1. 入标签随机产生
1. 出接口为收到标签映射消息的接口

（4）LSR继续向上游发布标签映射消息，直到入节点

（5）入节点LSR建立LSP

1. 出标签为下游LSR通告的入标签
1. 入标签为空
1. 出接口为收到标签映射消息的接口



### **7、标签通告模式**

（1）DOD：downstream-on-demand，下游按需标记分配，华三默认模式

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCEab63fa8c0a27eb792cc304ab3f22cbfb/9174)

特征：上游LSR先向下游LSR发送标签请求信息；下游LSR收到标签请求消息后，为此FEC分配标签，并向上游逐层通告。

优点：没有访问需求的地址，不会建立LSP，减轻路由器的性能负担

缺点：有访问需求才会触发建立LSP，会导致触发报文的前几个无法连通

（2）DU：downstream unsolicited，下游自主标记分配，华为默认模式

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCEc410c528c5cefde6e82467e8e3aebf6a/9175)

特征：下游LSR在LDP会话建立后，主动向上游LSR通告标签映射消息，无需等待上游请求

优点：无需统一访问请求触发，不会存在一组FEC前几个包不通的情况

缺点：路由器会主动建立所有路由表中下一跳为非LDP邻居的网段的LSP；

导致大量的LSP信息，而且很多可能是暂时无用的。



### **8、标签控制模式**

有序：只有从最下游的LSR开始建立标签后，才能逐层通告

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCEbf2ada0ce9c4a657e50f78aa28421629/9176)

无序（独立）：不管有没有收到下游的标签映射消息，都立即向上游发送标签映射消息（即使标签重复也无所谓）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCEac141694497add0e973c14f502735636/9177)

### **9、标签保持方式**

（1）保守模式：只保留最优路径的，来自下一跳邻居的标签，丢弃所有非下一跳邻居发来的标签；

如果IP路由表中存在等价路由，LSP会建立等价路径，做负载均衡。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCE12775000b5aa1d42dc47933027f73270/9178)

特征：

增加LSP的收敛时间；（一旦主路故障了，需要启动备用路径，重新建立标签分配的过程）

节省内存空间和标签。

（2）自由模式：保留所有邻居标签

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCE899f9e7c0053ab37325b50721f41ecdb/9179)

特征：

减少LSP收敛时间；

需要更多的内存和标签空间。



### **10、带标签的MPLS报文转发流程**

1. 报文进入MPLS网络，入节点检查标签转发表，进行PUSH操作，如下图：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCE7e162048235e86ed6a2cd74947b579aa/9180)



1. 报文在Transilt LSR中传输时，路由器检查标签，并在标签转发表中匹配，进行标签SWAP操作

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCE43347e023620e2d64498726d21592ba3/9298)

1. 报文到达出节点，路由器弹出pop标签，并按照普通数据报文进行报文

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCE7caf3a979db581a81d847861b099e499/9301)



## **七、默认 LSP的创建策略**

默认：给环回口的主机路由创建LSP，LOOPBACK口掩码必须是32



提高IP转发效率，这种场景下为所有路由创建LSP是有意义的。--all模式

用于支持VPN业务，LSP做隧道，IP转发优势突出，这种场景下为环回口路由创建LSP是有意义的。---host模式

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB29c070b10d53fc32265cd9f6fa9af84a/WEBRESOURCEda6e7c3e422f32cd5b2d584540c89ad9/37489)

## **六、MPLS的实际应用**

随着硬件技术的进步，进行转发的高速路由和三层交换技术得到了广泛的应用，使得IP转发性能大为提高，可以满足网络数据转发性能的需求，MPLS技术在提高转发性能应用上未能发挥优势。

但MPLS支持多层标签嵌套和面向连接的特点，使得其在VPN、流量工程（TE）、QOS等方面得到广泛应用。