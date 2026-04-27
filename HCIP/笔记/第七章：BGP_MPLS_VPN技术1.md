## **一、技术背景**

解决大规模组网的问题，一般小规模组网，用不上BGP MPLS VPN；

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB6403087041f93f4d821e3b2966081d53/WEBRESOURCE669baaa1bc3effcce204510cc62f54a5/27865)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB6403087041f93f4d821e3b2966081d53/WEBRESOURCE07eb6147cadb786347d043f7fbb4b51e/27915)

**BGP MPLS  VPN技术作为一种新型的VPN技术解决了以下问题：**

1. 实现隧道的动态建立
1. 解决本地地址冲突问题
1. 现在的VPN私网路由易于控制，可使用BGP传递私网路由；



## **二、MPLS隧道**

### **1、隧道技术与MPLS**

隧道：一个点对点的虚拟连接通道，这条通道可以使经过特殊封装的数据包能够传输，在隧道的两端分别对数据包进行封装与解封装

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB6403087041f93f4d821e3b2966081d53/WEBRESOURCE6ed55bf66bca2e495e0872f2d70524d8/9417)

隧道技术：用一种协议去封装另外一种协议

从技术原理角度看，MPLS技术是对数据包头部进行二次封装，即在IP报文头部前封装一个新的报文头部，所以从隧道架构角度看，MPLS技术天生就是一个天然隧道，MPLS的封装：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB6403087041f93f4d821e3b2966081d53/WEBRESOURCE49f233d5934ddac151c7f078a08de25d/9429)



### **2、MPLS隧道的特点：**

1. MPLS是天然隧道 ，标签没有公私网的区别；
1. MPLS中间路由器无需学习私网路由，无需查询路由表，只需在进出MPLS网络的LSR设备（隧道进出口的设备）上处理标签，私网报文就能穿越公共网络；
1. **所有站点可共享同一条MPLS隧道使报文穿越公共网络；**
1. 倒数第二跳弹出（PHP）：

如果出标签为3，LSR会直接而弹出标签，把普通数据报文发往出节点LSR处理；

避免出节点LSR查询MPLS标签信息表后，还要查询路由表。

### **3、MPLS隧道传输数据包过程描述**

如下：假设R1-R4已经配置了MPLS，LDP建邻、标签分配已完成、MP-BGP已配置

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB6403087041f93f4d821e3b2966081d53/WEBRESOURCEef93008f9c0f89f0a1f4bc6c7444392d/27926)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB6403087041f93f4d821e3b2966081d53/WEBRESOURCE544df26356efa578de3c7be7b41365bf/27929)



MPLS隧道传输数据包过程如下：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB6403087041f93f4d821e3b2966081d53/WEBRESOURCE2a153841840948c13aca7a9cd3d8abb2/27932)



关于本地地址冲突问题，我们来看看MPLS VPN是如何解决的？



## **三、MPLS VPN组网结构**

### **1、VPN组网结构**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB6403087041f93f4d821e3b2966081d53/WEBRESOURCE9e9e86bf2bf56bced400546c4d9e158b/9608)

CE：custom edge，用户边缘设备，直接与服务提供商相连的用户设备

PE：provider edge router，服务提供商边缘路由器，公共网络上的边缘路由器，与CE相连，负责VPN业务接入

P：provider router，服务提供商路由器，公共网络上的中间传输设备，主要完成路由和快速转发功能

### **2、用户维护隧道**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB6403087041f93f4d821e3b2966081d53/WEBRESOURCEe44590788d121379d9faf6793747c8be/27941)



### **3、运营商维护隧道**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB6403087041f93f4d821e3b2966081d53/WEBRESOURCEf63ed7f9e909a2aee7739cf7f71885a1/27946)

### **4、运营商维护共享隧道**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB6403087041f93f4d821e3b2966081d53/WEBRESOURCE94d2ae8e5157cd450b00a71dc2aca13c/27949)



## **四、BGP MPLS VPN存在的问题**

1、BGP  MPLS VPN大多由运营商建设

2、同一台PE设备连接的不同私网IP网段可能存在冲突（多VRF解决）

3、如何控制私网路由传播范围（RT）和区分问题（RD）



## **五、解决方法----多VRF技术**

### **1、定义：**

多VRF技术：虚拟转发路由器，又被称为VPN实例

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB6403087041f93f4d821e3b2966081d53/WEBRESOURCE7f4c2aa9b3b5d6de5d659c4356f7a26a/9669)



### **2、特点：**

1. MPLS VPN大多由运营商建设
1. 一台物理路由器可划分为多个VRF（相当于把一个路由器从逻辑上划分成多个路由器，划分出每一个端口都是独立的）
1. 每台VRF拥有独立的路由表接口、路由协议，VRP之间互不可见

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB6403087041f93f4d821e3b2966081d53/WEBRESOURCE93729ae351461a0486f4098cbd8b0c21/9681)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB6403087041f93f4d821e3b2966081d53/WEBRESOURCEfd4312e92808b3dccd3e1800ee0159d1/9686)



### **3、作用：**

用于解决同一台PE设备连接的不同私网IP网段冲突问题