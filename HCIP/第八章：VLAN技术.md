# **一、VLAN技术产生背景**

路由器具有隔离广播域的功能，而交换机没有，为了解决广播域的问题，引入VLAN，通过逻辑上的划分，减小广播域范围

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCE43b3d87a3e5a7daee79e4683afeae809/30845)

# **二、VLAN作用及特点**

VLAN作用：将规模较大的广播域在逻辑上划分成若干个不同的、规模小的广播域

VLAN特点：同一VLAN内的主机可以通信，不同VLAN内的主机不能通过二层交换机通信，只能借助三层设备通信



# **三、VLAN实现过程**

## **（1）同一交换机VLAN内的通信**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCE78436fb579ae1890b66a8ec34287301b/30846)

工作过程：

1. pc发送数据帧进入交换机，会被打上VLAN tag；VLAN tag中的VLAN id就是收到数据帧的接口的所属VLAN；一旦数据帧被打上VLAN tag，就变成了802.1Q格式的帧；
1. 交换机检查数据帧的目的MAC地址，进行判断；

若目的Mac地址对应的接口允许tag中的VLAN id通过，则数据帧可以转发；否则，丢弃该帧

1. 数据帧从接口发往pc前，会剥离VLAN tag，使之还原为标准的以太网帧格式



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCEd4742f9f378f3bc0e6c4684af2fc8265/30847)



## **（2）跨交换机的同一vlan的通信**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCE6ce81ddb2d717bb7a894d19d6c4c4a8a/30848)

工作过程：

1. 数据帧从主机出来后会进入到交换机，交换机收到后会给此数据帧打上一个vlan tag（tag中的vlan ID就是交换机收到数据帧接口的vlan ID），此时数据帧变成了一个802.1q格式的帧。
1. 在发送端802.1Q格式的数据帧离开交换机时，检查该接口的vlan允许列表，若允许，则带标签发送，反之则丢弃。
1. 接收端的交换机收到后，交换机检查目标MAC地址的主机接口所属的vlan ID，如果此vlan ID与802.1q帧格式中的vlan ID一致，则转发该数据帧，否则丢弃。

注意：数据从交换机发往主机前，需要剥离掉vlan TAG	，使之还原为原始的以太网数据帧。



# **四、VLAN划分方式**

## **1、****基于接口划分**

VLAN的范围：0-4095，其中0和4095为系统保留，1-4094用户可用

PVID：缺省VLAN，默认VLAN 1

优点：划分方式简单，容易实现

缺点：主机移动，需要重新配置VLAN

## **2、基于IP地址划分**

建立IP地址与VLANID的映射关系

优点：易于主机移动

## **3、基于MAC地址划分**

建立MAC地址与VLANID的映射关系

缺点：配置易出错，主机不易移动

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCE32d427184a994602973ad238eeb17f2e/38877)

```ada
步骤1：创建vlan
[SW1] vlan 10      
步骤2：建立MAC地址与VLANID的映射关系
[SW1-vlan10] mac-vlan mac-address 001e-10dd-dd01
[SW1-vlan10] mac-vlan mac-address 001e-10dd-dd02
[SW1-vlan10] mac-vlan mac-address 001e-10dd-dd03
步骤3：接口加入相应vlan
[SW1] interface gigabitethernet 0/0/2
[SW1-GigabitEthernet0/0/2] port link-type hybrid
[SW1-GigabitEthernet0/0/2] port hybrid untagged vlan 10
步骤4：使能接口的基于MAC地址划分VLAN功能
[SW1] interface gigabitethernet 0/0/2
[SW1-GigabitEthernet0/0/2] mac-vlan enable 
```



## **4、基于协议划分**

建立协议簇类型和VLAN id的映射关系

## **5、基于策略的划分**

多种方式的组合，需要管理员预先设置策略

注意：VLAN划分方式排序优先级：MAC>IP>协议>端口

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCEce84f5612439499a922577910dc86a6b/38878)



# **五、交换机接口类型**

## **1、access：**

（1）定义：一般连接PC、服务器等终端设备，一个接口只能加入一个VLAN。

（2）特点：

1. 接收帧：

收到untagged的数据帧，打上该接口PVID的标签，并接收；

收到tagged数据帧，与该接口PVID比较，相同则接受，不同则丢弃；



1. 发送帧：

剥离标签发送

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCE7d695539a01690c69daf9aacafe52e85/30850)

## **2、trunk：**

（1）定义：一般用于交换机间互连、路由器、防火墙等设备的子接口；

可以允许多个VLAN的数据帧通过；

（2）特点：

1. 接收帧：

untagged数据帧，打上该接口的PVID的标签，同时看下该接口的PVID是否在接口允许列表中，如果在就并接收，否则丢弃；

tagged数据帧，查看数据帧中的VID是否在该接口允许列表中，如果在则接收，反之则丢弃；



1. 发送帧：

数据帧中VID在允许列表中，且VID与PVID一致，则剥离标签发送；

数据帧中VID在允许列表中，但VID与PVID不一致，则直接带标签发送；

不在允许列表中，则直接丢弃；



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCE4e044755b63047aea2241284db9c8e7e/36009)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCE31027e66e6234c3c8f1bbd5d227ce40a/45049)



[SW1-GigabitEthernet0/0/1]port trunk pvid vlan 10     修改TRUNK接口PVID

[SW1-GigabitEthernet0/0/1]undo port trunk allow-pass vlan 1



## **3、hybird：**

（1）定义：既可连接pc或者路由器，能来连接交换机，可以允许多个VLAN的数据通过；

可以手动配置hybrid端口发出的数据帧，哪个VLAN保留标签，哪个VLAN撕掉标签；（必须定义，否则数据没法传输）



（2）特点：

1. 接收帧：

untagged数据帧：接口会为数据帧打上PVID作为标签，然后检查该VLAN是否在接口允许通过的VLAN列表中，如果在则接收，否则丢弃

tagged数据帧：直接检查数据帧中的VLAN ID是否在接口允许通过的VLAN列表中，如果在则接收，否则丢弃



1. 发送帧：

Tag中的VLAN ID与pvid一致，则剥离tag并发送；

Tag中的VLAN ID与pvid不一致：

Tag中的VLAN ID出现在tagged表中，则保留tag发送；

Tag中的VLAN ID出现在untagged表中，则剥离tag发送；



1. 默认处理：

如果VLAN ID既不在untagged也不在tagged列表中,数据帧将被丢弃



（3）hybird应用（每个PC都能和服务器通信，但PC之间不互通）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCE5d8fd673fa11472cfff0c5a4c7e227a0/30853)



实现vlan重复使用，节约vlan数量

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCEfe87639616401cb3436459e69f09854d/30964)



[SW2-GigabitEthernet0/0/2]port hybrid untagged vlan 10 100   配置untag的vlan列表



批量进入端口配置

方法1：[SW2]int range g0/0/1 to g0/0/4

方法2：[SW2]port-group group-member GigabitEthernet 0/0/2 to GigabitEthernet 0/0/4



# **六、VLAN间通信背景**

同一VLAN内通信，可以通过二层交换机实现；

VLAN间的通信，只能依靠三层设备（路由器、三层交换机）



## **（1）使用路由器物理接口**

特点：在路由器上配接口IP，作为主机网关。但这需要路由器与每个VLAN建立一条物理连接，路由器接口很少，浪费路由器接口。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCEcbdaef6f3c81da5059b15e9ae099d769/30854)

发送端：

目的IP：20.2    源IP：10.2    目的MAC：B  源MAC：A+VLAN 10 +数据

接收端：

目的IP：20.2    源IP：10.2    目的MAC：D  源MAC：C +VLAN 20+数据



## **（2）使用路由器子接口（单臂路由）**

特点：一旦出故障，网络就瘫痪了。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCE51c75120f61ad1dddf9ff112b4bc0cf3/30855)

发送端：

目的IP：20.2    源IP：10.2    目的MAC：B  源MAC：A +VLAN 10+数据

接收端：

目的IP：20.2    源IP：10.2    目的MAC：D  源MAC：C +VLAN 20+数据



配置命令

[r1]int g0/0/0.1  创建子接口

[r1-GigabitEthernet0/0/0.1]dot1q termination vid 2  封装802.1q格式

[r1-GigabitEthernet0/0/0.1]arp broadcast enable 开启arp广播



## **（3）使用三层交换机的VLANIF接口**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCE20b1beda5613d58154ce9a093f7b76e7/30856)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2bf4c273c426030d873cb38249739b21/WEBRESOURCEffc7ff3d5aea0ad6ac77ca6863ecfcba/30857)



目标MAC：MAC2：源MAC：MAC1+vlan 10+ 目标IP：192.168.20.2 源IP：192.168.10.2/24+传输层+应用层+数据

目标MAC：MAC3：源MAC：MAC2+vlan 20 +目标IP：192.168.20.2 源IP：192.168.10.2/24+传输层+应用层+数据



交换模块：

路由模块：

[sw1]int vlanif 2   进入VLAN 2的VLANif接口

[sw1-Vlanif2]ip add 192.168.1.254 24



# **七、二层接口与三层接口对比**

1、二层接口不能配置IP，三层可以配置IP

2、二层接口只有转发功能，三层接口有转发和路由的功能

3、二层接口不能隔离广播域，但三层接口可以



# **八、作业**

vlan综合实验