# **一、OSPF的网络类型**

## **1、定义**

对于不同的二层链路类型的网段，OSPF会生成不同的网络类型

不同的网络类型，DR\BDR选举，LSA细节，协议报文发送形式等会有所不同

## **2、OSPF网络类型**

### **（1）NBMA（非广播多点可达网络）**

1. 非广播多点可达网，帧中继默认的网络类型
1. 单播发送协议报文，需手动指定邻居

命令：[r2-ospf-1]peer 192.168.1.1 (邻居IP地址)

1. 需要选举DR\BDR，为了减少LSA的泛洪，减少网络负担
1. hello-time 是30秒，dead-time 是120秒

### **（2）P2MP（点到多点网络）**

1. 点到多点网络，由其他网络类型手动更改：例如在ospf接口下：ospf network-type 网络类型
1. 模拟组播发送协议报文

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB3f7dd9ba81d14a181e0f34e9ba5ba5d9/WEBRESOURCEee0471ae296f141bf87e4ead03db991d/8390)

1. 不选举DR\BDR
1. hello-time 是30秒，dead-time 是120秒

### **（3）broadcost**

1. 广播网络，以太网默认的网络类型
1. 组播发送协议报文
1. 需要选举DR\BDR
1. hello-time 是10秒，dead-time 是40秒

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB3f7dd9ba81d14a181e0f34e9ba5ba5d9/WEBRESOURCE33752dcb76ec739cf0398b5db61dcd3f/35437)

transmit：LSA的传输延时，默认1秒

retransmit：重传时间，一台路由器在向邻居发送一个LSA（链路状态通告）或一个LSU（链路状态更新）包后，等待对方确认（Ack）的最大时间。如果在这个时间内没有收到确认，则认为传输失败，将重新发送该数据包。

Poll ： 轮询时间：就是专门在NBMA网络中用来替代 Hello Interval，专门用于 Down 状态邻居的、一个更慢的“保活”探测机制。

1. 默认值：通常 120秒（2分钟）。这比30秒的HelloInterval长得多。
1. 工作流程：

a.你手动配置了一个NBMA邻居（例如，neighbor 10.1.1.2）。

b.该邻居因为某种原因失效，状态变为 Down。

c.此时，你的路由器将停止以 HelloInterval（30秒）的频率向该邻居发送Hello包。

d.转而开始使用 Poll Interval（120秒）的频率，每隔2分钟向 10.1.1.2 发送一个Hello包。

e.一旦这个 Down 的邻居重新上线并做出了响应，邻居关系开始重新建立，状态从 Down 变为其他状态（如 Init, 2-Way）。

f.这时，你的路由器会立即停止使用Poll Interval，并恢复使用正常的 HelloInterval（30秒）来与该邻居进行通信。

### **（4）P2P（点到点网络）**

1. 点到点网络，ppp默认网络
1. 组播协议发送报文
1. 不选举DR\BDR
1. hello-time 是10秒，dead-time 是40秒
1. 环回接口：华为设备定义为P2P类型，但实际上无数据收发。环回接口默认学习32位的主机路由

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB3f7dd9ba81d14a181e0f34e9ba5ba5d9/WEBRESOURCE554a3beef6f205c79e4963d9b556dfcf/43520)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB3f7dd9ba81d14a181e0f34e9ba5ba5d9/WEBRESOURCE70b211fc771fe8b5709040731eb55daf/43525)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB3f7dd9ba81d14a181e0f34e9ba5ba5d9/WEBRESOURCE5434730b58740a67b03d3a0d456b352b/35439)

隧道接口、串线接口、环回接口，均是P2P网络类型

# **二、基于OSPF的MGRE实验**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB3f7dd9ba81d14a181e0f34e9ba5ba5d9/WEBRESOURCE14789b8719ec4fa6e75146f1fbb0be45/15834)



基于ospf的MGRE出现问题：ospf的路由表学习不全

问题1：Tunnel接口类型为P2P类型，一条线只能建立一个邻居，



解决方法：更改网络中tunnel接口类型为广播或者P2MP

[R2]interface Tunnel 0/0/0

[R2-Tunnel0/0/0]ospf network-type broadcast



问题2：DR和BDR选举混乱，路由表没法正常学习

更改网络类型后，广播网络中中心站点和分支站点处于同一个广播域，此时需要进行DR和BDR的选举，但是在分支站点的世界里只和中心站点认识，分支站点和分支站点不认识，这就会发生多个分支站点和一个中心站点互相竞选DR和BDR，这样会造成选举结果混乱，可在中心站点看到混乱的场景

解决方法：将分支站点的dr选举优先级变0，这样就能保证中心站点是整个广播网络中唯一的DR



1. 隧道接口带宽 = 64 Kbps = **0.064 Mbps**
1. Cost = 100 Mbps / 0.064 Mbps ≈ **1562.5**

由于Cost必须是整数，OSPF会**向下取整**，最终结果就是 **1562**。