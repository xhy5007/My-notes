# **一、LSA的头部**

LSA是OSPF的一个核心内容，如果没有LSA，OSPF是无法描述网络的拓扑结构及网段信息的,也无法传递路由信息，更加无法正常工作，**在OSPFV2中，需要我们掌握的主要有6种。**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa80f22e35721e6c02d9b926055fce176/WEBRESOURCE82161946287daa31c6590853d6413a4d/16312)

## **（1）LSA头部一共20byte，每个字段的含义如下**

1. 链路状态老化时间(Link-State Age)：指示该条LSA的老化时间，即它存在了多长时间，单位为秒，

1800s周期归0，触发当下归0

**        MAX age --- 3600S** ------ **当一条LSA的老化时间到达最大老化时间时，将被认定失效，将从本地的LSDB中删除掉。**

**注：本机发出一条LSA，age=0,LSA有默认的链路传输延时（1秒），对方收到后这条LSA的老化计时是从（0+传输延时）开始**

1. 可选项(Options)：每一个比特位都对应了OSPF 所支持的某种特性。 ------ 和hello包中的一样，包含特殊区域标记
1. 链路状态类型(Link-State Type)： 指示本条LSA的类型。每种 LSA用于描述OSPF 网络的某个部分，所有的LSA类型都定义了相应的类型编号。
1. 链路状态ID(Link-State ID)： LSA的标识。不同的LSA类型，对该字段的定义是不同的。
1. 通告路由器（Advertising Router)： 始发路由器， 产生该LSA的路由器的Router-ID
1. 链路状态序列号(Link-Sate Sequence Number)：该LSA的序列号，该字段用于判断LSA的新旧或是否存在重复
1. 链路状态校验和(Link-State Checksum)：校验和会参与LSA的新旧比较。当两条LSA三元组相同，并且序列号也相同时，则可以使用校验和比较，和大的认定为新。
1. 长度(Length)：一条LSA的总长度



## **（2）LSA的刷新机制**

**周期性更新：**

原因： 能够让LSDB始终描述是当前某段之间内网络真实状态信息，防止LSDB中LSA的存活时间的差异带来的不确定性:。

每隔30分钟周期性更新自身产生的LSA，谁产生谁负责对它进行刷新，并泛洪出去。

当一条LSA在3600s得不到刷新，则在LSDB中删除。

刷新过程中，1sa seq加1，lsa age置0，chksum重新计算:



**触发更新：**

当LSA中描述链路状态参数取值发生变化，则立即发生新LSA，在邻居间泛洪。

例如：修改接口COST，观察修改前后的LSDB表。[R1-GigabitEthernet0/0/0]ospf cost 30

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa80f22e35721e6c02d9b926055fce176/WEBRESOURCE8258c2784c639b1844a5ac4ca58564bb/42922)



## **（3）LSA的新旧判断机制：新的LSA要替换掉LSDB中旧的LSA**

1. 比较 LSA seq ，越大越新。
1. 如果LSA seq相同，比较checksum ，checksum大的越大。
1. 如果LSA seq相同，checksum相同，则判断LSA age是否等于3600s

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa80f22e35721e6c02d9b926055fce176/WEBRESOURCE80f84851a305441386bb4f5859e09a9c/42936)

1. LSA age 等于 3600 则认为最新的LSA。 LSA age 3600 是用于主动删除一条LSA的方法

例如：R1上引入静态路由，立马取消引入，观察R1发给R2的LSA的age=3600，R2收到后会立马主动删除此路由旧的LSA

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa80f22e35721e6c02d9b926055fce176/WEBRESOURCEd5f574ca99eb322ca61cc16837c0e1d0/42941)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa80f22e35721e6c02d9b926055fce176/WEBRESOURCE9a7c3031f18f27df576196ea94d4214a/42946)



1. 如果LSA age 不等于 3600秒，则比较LSA age的差值，差值大于900秒，则LSA age 小的最优。差值小于900秒，则认为是一致。



# **二、6种类型的LSA（课堂演示）**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa80f22e35721e6c02d9b926055fce176/WEBRESOURCE5415b4fcf596de0b81c6c26b12a2543a/16223)



## **1、type1-LSA：----重要且复杂**

（1）定义：router LSA

1. 描述区域内部与路由器直连的链路信息（链路类型、开销值等）
1. 仅在区域内部传输
1. 每台路由器都会产生Type1 LSA



[R1]dis ospf lsdb router  查看Type1 LSA的具体信息

[R1]dis ospf lsdb router 2.2.2.2   //查看具体某个1类LSA

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBa80f22e35721e6c02d9b926055fce176/WEBRESOURCE01e762a793a50810843e06c1c08dbe4c/10376)

（2）LS ID:链路状态ID，发出该LSA的路由器的router-id

（3）Adv Rtr:始发路由器,产生该LSA的路由器的router-id

（4）链路ID：不同的链路类型，对链路ID值的定义是不同的。

（5）链路数据（Link Data):不同的链路类型对链路数据的定义是不同的。

（6）link-type：链路类型，描述该接口的二层类型

1. **transnet：传输网络，在广播网络中出现，连接了多个运行ospf协议的路由器，承载过路流量**

类型：广播网络或者NBMA

link-id：本网段的DR的IP地址

Data：本路由器在该网段的IP地址

1. P2P：点到点链路，适用于P2P网络

类型：ppp

link-id：该网段对端路由器的router-id

Data：本路由器在该网段的与对端路由器相连的接口的IP地址



1. stubnet（末梢网络）：运行OSPF协议的路由器连接一个直连网段，（比如连接PC的接口，这个接口被宣告进OSPF。），这个直连网络不承载任何过路流量

类型：p2p\环回口

link-id:该网段的网络地址

data：该网段的子网掩码



1. Virtual(虚链路)：

类型：虚链路

link-id:虚链路邻居的router id

data：去往该虚连接邻居的本地接口的IP地址

（7）VEB标志位：

1. V位(Virtual Link Endpoint Bit)：如果该比特位被设置为1，则表示该路由器为Virtual Link的端点。
1. E位(External Bit)：E：代表此路由拥有携带外部路由的能力。如果E比特位被设置为1，则表示该路由器为ASBR。在Stub区域中,不允许出现E比特位被设置为1的Type-1 LSA,因此Stub区域内不允许出现ASBR。
1. B位(Border Bit)：如果B比特位被设置为1，则表示该路由器为两个区域的边界路由器，字母B意为Border(边界)。



## **2、type2-LSA：**

（1）定义：

network LSA

描述区域内的MA网络（广播网络、NBMA网络）路由器的链路及掩码信息

仅在区域内部传输

只有DR才会产生type2_LSA

[R1]dis ospf lsdb network  查看Type2 LSA的具体信息

（2）内容：

LS ID：该网段的DR的IP地址

Adv Rtr:该网段DR的router-id

network mask：该网段DR的IP地址的子网掩码信息



## **3、type3-LSA：**

（1）定义：

Summary LSA(聚合LSA)

在整个OSPF域内，描述其他区域的链路信息

以子网形式传播，类似直接传递路由

只有ABR会产生type3_LSA

[R1]dis ospf lsdb summary  查看Type3 LSA的具体信息

（2）内容：

LS ID：其他区域某个网段的网络地址

Adv Rtv:通告该LSA的ABR的router-id

net mask:该网段的子网掩码

注：3类LSA的传递范围在ABR相邻的单区域中进行，跨区域传递时，需要进行通告者的转换，通告者变了，则将不是同一条LSA



## **4、type4-LSA：---整个ospf的域**

（1）定义：

Asbr-summary LSA

描述ASBR的信息

只有ABR才会产生TYPE4 LSA

（2）内容：

LS ID：ASBR的router-id

Adv Rtv:通告描述该ASBR的ABR的router-id

[R1]dis ospf lsdb asbr  查看Type4 LSA的具体信息

注：在ASBR本区域的内部路由器，不会产生到达该ASBR的4类LSA



## **5、type5-LSA：----整个ospf域**

（1）定义：

AS_extenal LSA，传递域外路由信息

描述AS外部引入的路由信息，会传播到所有区域（特殊区域除外）

只有ASBR才会产生type5_LSA

（2）内容：

LS ID:外部路由的目的网络地址

Adv Rtv:引入该网络路由的ASBR的router-id

net mask:引入的该目标网段的子网掩码

[R1]dis ospf lsdb ase  查看Type5 LSA的具体信息

## **6、type7-LSA：---****不能进入area 0，只存在于NSSA区域和totally NSSA区域**

（1）定义：

NSSA LSA

描述在NSSA区域引入的AS域外的外部路由信息

只会出现在NSSA和totally NASS区域，不能进入area 0

7类LSA生成路由信息的标记位，O_NSSA，优先级150

（2）内容：

LS ID:外部某个网段的网络地址

Adv Rtv:引入该网络路由的ASBR的router-id

net mask:引入的该目标网段的子网掩码



**区域内传拓扑，区域间传路由**