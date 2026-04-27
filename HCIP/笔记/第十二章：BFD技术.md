# **一、BFD简介---（****bidirection forwarding detection，双向转发检测****）**

BFD是一个简单的“Hello”协议。两个系统之间建立BFD会话通道，并周期性发送BFD检测报文，如果某个系统在规定的时间内没有收到对端的检测报文，则认为该通道的某个部分发生了故障。（任意一层的协议都可以借助BFD实现故障的快速检测）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCE70b34c9808490c7935e8cedaa18612ea/27755)

# **二、BFD特点**

1. 通用的、标准化的、介质无关的、协议无关的快速故障检测机制

通用的：可以与DHCP、VRRP、路由协议等联动，快速检测通信链路故障， 不需要每个协议单独研发检测机制，BFD可达到电信级的网络要求，毫秒级的检测速度

标准化的：BFD的开发，华为贡献是比较大的，华为、华三、思科、锐捷等设备都支持BFD

介质无关的：光信号或者电信号都能用，包括：无线介质

协议无关：不依赖于其他协议，可独立工作。

1. 在转发与控制分离的情况下，运行于系统的转发平面
1. 可以为各种上层协议（如：路由协议、MPLS等）服务
1. 上层协议通常采用hello报文机制检测故障，所需时间为秒级，有些协议本身无检测机制（OSPF--40S故障检测时间，BGP--180S故障检测时间，静态路由本身无故障检测机制）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCEc523b4358ecf9d568d589eba6352474c/27738)

1. 提供毫秒级的检测速度，从而加快网络收敛速度，减少应用中断时间，提高网络的传输效率。（秒级检测时间过长，当传输速度超过GB/S，将导致应用传输数据大量丢失，例如：上图中动态路由故障检测问题中，R1和R2不能快速感知故障，导致R1和R2之间的邻居关系还是存在的，但当数据传到R1和R2链路时，发现中间链路坏了，这种故障要40S才能发现，就会导致大量数据丢失）



# **三、BFD的基本原理**

## **1、BFD报文结构**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCEf36913c22c8b42a3e84c0aef2a15ebb2/27759)

了解字段：

ver：版本号，1

diag：诊断字，标明本地BFD系统最后一次会话状态发生的原因

P：poll，参数发生改变时，发送方在BFD会话中置位该标志，接收方必须立即响应该报文

F：fin，响应P标志置位的回应报文中必须将F标志置位

C：con，转发/控制分离标志，一旦置位，控制平面的变化不影响BFD检测

A：AUTH，认证，表明会话是否配置了认证

D：demad，表明是否采用查询模式，默认是0，则为异步模式

M：multipoint，为了支持MP-PPP的预留位



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCE07b6b56141894da0ccaa4a82c98328e5/27761)



## **2、会话建立**

方式：静态建立、动态建立

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCE64deaaf64d0f9f5e2f0a67573193be30/27765)



```abap
静态建立BFD会话配置：
[r1]bfd  //全局模式开启BFD
[r1]bfd 12 bind peer-ip 10.1.12.2 interface g0/0/0    //与对端建立BFD会话
[r1-bfd-session-12]discriminator local 100      //手动配置本地会话标识
[r1-bfd-session-12]discriminator remote 200    //手动配置远端会话标识
[r1-bfd-session-12]min-rx-interval 300   //最小接收间隔
[r1-bfd-session-12]min-tx-interval 300   //最小发送间隔
[r1-bfd-session-12]detect-multiplier 3
[r1-bfd-session-12]commit 
```

## 

```plain text
动态BFD会话配置：
[r1] bfd   #全局模式下启动bfd
ospf [process-id]  # 进入OSPF进程视图
 bfd all-interfaces enable  # 对所有接口启用BFD
 
针对特定接口：
interface GigabitEthernet0/0/1
 ospf bfd enable  # 在该接口下启用BFD
  
可选：调整BFD参数：ospf协议视图下配置
bfd all-interface min-tx-interval 500 min-rx-interval 500  detect-multiplier 3  
```

## 

## 

## **3、BFD会话状态（了解）**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCEe8fb92f8f9f3e4edee74e73c23b8f3da/27766)

会话的建立和断开需要三次握手，确保两端系统都能感知到（两端各自进行三次握手，各握各的）

down：本端会话已经关闭或刚创建。表示转发路径不可用，与BFD会话联动的上层应用需要采取适当的措施，例如：主备路径切换等

INT：本端已经可以与对端通信，且本端希望会话进入UP状态

UP：本端会话已经建立成功。UP状态表示转发路径可用

ADMIN DOWN:本端系统主动阻止建立BFD会话时； ADMIN DOWN也是down状态；



## **4、BFD检测模式**

BFD的检测机制：两个系统建立BFD会话，并沿它们之间的路径周期性发送BFD控制报文，如果一方在既定的时间内没有收到BFD控制报文，则认为路径上发生了故障。BFD的检测模式有异步模式和查询模式两种。

（1）异步模式-----适用于两方都支持BFD检测

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCEc0acfc8568497ac11dcbcfb731fec88e/27787)

（2）查询模式-----适用于被检测的一方不支持BFD检测

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCE89ed13de7c31d4606144a24295bd6eec/27789)

两者本质：

检测的位置不同，

异步模式下：本端按一定的发送周期发送BFD控制报文，检测位置为远端；远端检测本端是否周期性发送BFD控制报文。

查询模式下：本端检测自身发送的BFD控制报文是否得到了回应。



## **5、BFD检测时间**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCE8c4670feb1a15f129bab928eb598618c/27776)

**备注：大部分情况下两端的TX（发送间隔）和RX（接收间隔）配置是一样的，DM（检测倍数）**

**BFD缺省时间参数：**

**BFD发送间隔默认1000毫秒，接收间隔默认1000毫秒，本地检测倍数3次**



## **6、BFD ECHO单臂回声功能****----支持查询模式**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCE77163a2d78f8da02495c4f982a565756/27777)



## **7、BFD单跳和多跳检测**

（1）单跳检测：对两个直连设备进行IP连通性检测

（2）多跳检测：BFD可以检测两个设备间任意路径的链路情况，这些路径可能跨越很多跳，不要求设备直连

（3）两者区别：

单跳检测：设备直接向peer-ip发送BFD报文，此时设备会直接请求peer-ip的MAC地址，会配置出接口

多跳检测：设备按照peer-ip进行路由表的递归，请求最终下一跳的MAC地址，最终向peer-ip地址发送BFD报文，不会配置出接口



# **四、BFD的应用**

## **1、联动功能：**

由监测模块、track和应用模块三部分组成

监测模块：负责对链路状态、网络性能等进行检测，并将探测结果通知给track模块

track：track模块收到监测模块的探测结果后，及时改变track项的状态，并及时通知给应用模块

应用模块：掌握track项的状态，进行相应的处理，从而实现联动。



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCE2384a52c70c9a0197735c9fa7e8ff634/27778)



## **2、静态路由与BFD会话的联动**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCE68fca16bd9661f546486afd0407c13d8/27779)

## **3、****OSPF与BFD会话的联动**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCE692122dda14f11e26cae0c62cb9dae72/27780)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCEd473f99f12b0c2695ee83e9a8ca14fc5/27781)

## **4、BFD与VRRP联动----一般在SW2--R4，上行链路之间配置BFD**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCE6b955c9d9f1d358e6ad7a0d1e7b68dd8/27840)



# **五、配置示例**

## **1、静态路由与BFD的联动配置**



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCE940f0801045a73920fa82113566652d3/27782)



## **2、****OSPF与BFD的联动配置**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCE4edf12d486f9d0707c1a65231a14d6ac/27783)

## **3、OSPF与VRRP的联动配置---见实验报告**



## **4、单臂回声的配置**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCE47421413d8170a0fad87e17ccbc9a98e/27844)



[R1]bfd

[R1]bfd 12 bind peer-ip 192.168.20.2 interface GigabitEthernet 0/0/0 one-arm-echo

[R1-bfd-session-12]discriminator local 1

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa49af3a137ab311d6b6c8314363a2893/WEBRESOURCE94309e7e3aa476a53eb552a4c70994c1/27848)