# **一、特殊区域**

定义：特殊区域是指人为定义的一些区域，它们在逻辑中一般位于ospf区域的边缘，只与骨干区域相连。

**特殊区域的条件：不能是骨干区域；不能存在虚链路；**

## **1、STUB区域：末梢区域**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCEf6140dc51406d9afa40ffc819701fabd/11130)



（1）定义：

末梢区域，适用于区域中路由器性能较低，目的是为了减少区域中路由器的路由表规模以及路由信息传递的数量。不希望接收大量的AS以外路由

（2）特征：

不接受4类5类LSA；

不允许出现ASBR；

区域0不能被配置为STUB区域；

虽然拒绝学习域外路由信息，但依然有访问域外路由的需求；故会由ABR设备自动下发一条指向骨干区域的3类缺省；

（3）命令：[r5-ospf-1-area-0.0.0.2]stub

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCEbf5dd82d9e6fddf535f1cb4b6f354d28/11138)

（4）实验测试：将R3和R4设置为stub区域，观察R3和R4的ospf路由表

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCE5bd1388d3ea2eb9361e7113d77d0f7f3/17418)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCE188c313204fa4dcbc78533fd8ff8d5da/15272)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCEfc33e09288a7761e564a54cde7f6fd02/15274)



## **2、totally stub区域----完全末梢区域**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCEb3a242e2a0f74566be83885b5676215a/11140)



（1）定义：完全末梢区域，拒绝学习域外和其他区域的路由信息

（2）特征：

不接受3类4类5类LSA；

不允许出现ASBR；

区域0不能被配置为totally STUB区域；

虽然拒绝学习域外和其他OSFP区域的路由信息，但依然有访问域外和OSPF其他区域的路由的需求；故会由ABR设备自动下发一条指向骨干区域的3类缺省；

（3）命令：[r5-ospf-1-area-0.0.0.2]stub no-summary

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCEe9cbe1e2565f1768511e6df6fa63de48/11144)

（4）实验测试：将R3和R4设置为totally stub区域，观察R3和R4的ospf路由表



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCE91c1e44617ea52ea26a777b127bde29b/17420)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCE6e1823f1bd0dc6485382a4c3d9b95008/15279)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCE75b90dd634cee42b429103ad4ca518a6/15260)



## **3、****NSSA区域（Not-So-stubby Area）----非纯末梢区域**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCE727f414186c2c47422d6591609215708/11147)



（1）定义：非纯末梢区域，是STUB区域的变形，拒绝学习域外路由信息，但是有访问域外路由的需求

（2）特征：不接受4类5类LSA；

本区域引入的外部路由以7类LSA存在；

本区域的ABR会 把引入的7类LSA转换为5类 LSA通告给其他区域；

华三中：NSSA区域的默认路由需要手动配置下发，在ABR下发的是7类默认路由的LSA；

命令：nssa default-router-advertise

华为中：自动生成一条指向骨干区域的7类缺省，在ABR下发的是7类默认路由的LSA；

命令：nssa

区域0不能被配置为NSSA区域；

存在asbr设备

（3）命令：[r5-ospf-1-area-0.0.0.2]nssa

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCE25dac558ea535b6ddae514723d3ea661/11153)

（4）实验测试：将R3和R4设置为NSSA区域，观察R1和R2的ospf路由表



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCEdea39ab6b17a028dd16ac48966d73473/17422)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCEe38d685cd6899e7500798faa725da2d5/15298)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCE42ccb508b34f8aa4b568da00553ee472/15302)



## **4、totally NSSA区域---完全非纯末梢区域**

（1）定义：完全非纯末梢区域，拒绝学习域外和OSPF其他区域的路由信息，但是有访问域外路由和OSPF其他区域的路由的需求

（2）特征：不接受3类4类5类LSA

本区域引入的外部路由以7类LSA存在

本区域的ABR会把引入的7类LSA转换为5类 LSA通告给其他区域

本区域默认路由由ABR发送3类LSA产生

区域0不能被配置为totally NSSA区域

命令：[r5-ospf-1-area-0.0.0.2]nssa no-summary



（3）实验测试：将R3和R4设置为totally nssa区域，观察R3和R4的ospf路由表



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCE64137a66bed6ee04991f6a9d273edd79/17423)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCEbc9708320f110e787804fad077b43d23/15305)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCE6fbdec9a3b1fc04487c20de1c6c5068f/15308)

注：ABR的7转5操作，如果网络中存在等价ABR，则由router-id大路由器进行转换操作

# **二、OSPF路由聚合**

## **1、定义**

OSPF路由只能手动聚合（LSA），将具有相同前缀的路由信息聚合后发布到其他区域

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCE84e3f7cb8f08402d3accb741c7f48c81/11160)



## **2、聚合条件：**

针对3类、5类、7类LSA；OSPF路由只能手动聚合（LSA）

## **3、聚合类型：**

### **（1）ABR聚合（3类）**

1. 把一个区域的LSA聚合后发布到相邻区域；abr设备下游区域的设备中的LSDB表的LSA也被聚合了
1. 在区域试图配置：命令：[r1-ospf-1-area-0.0.0.1]abr-summary 192.168.0.0 255.255.252.0
1. ABR聚合不会影响ABR本机的OSPF的路由条目，只会影响相邻区域的下游路由器的OSPF路由表的路由条目；
1. 华为设备汇总路由的开销，继承明细路由中的cost值大的；例如：下图的汇总后路由的COST值

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCE26cbc35c886592eeeef184af1cb4cc1a/50389)

1. 当明细路由全部失效，汇总路由才会失效
1. ABR 聚合后，会在ABR本机上产生一条该聚合的黑洞路由，来**防止环路**出现；

**注：ABR汇总建议：将非骨干汇总后传给骨干，不建议将骨干区域汇总后传给非骨干，否则会出现上图的次优路径问题**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB060db3ca606168d65a6e81307d7e58d5/WEBRESOURCEaaadcbc625bcb4fcff91b8dcb42caca3/11163)

[R2-ospf-1-area-0.0.0.1]abr-summary 192.168.0.0 255.255.252.0 not-advertise     //不会将汇总后的路由，通告给下游设备

[R2-ospf-1-area-0.0.0.1]abr-summary 192.168.0.0 255.255.252.0 cost 99     //修改汇总后路由的cost值，下游设备学习到此路由COST值改变



### **（2）ASBR聚合（5类、7类）**

1. 把引入的AS外部路由聚合后发布到OSPF内部;对于OSPF的ASBR设备LSDB表的影响，只学习到了聚合后的路由的LSA，对于下游设备的LSDB表有影响，只学到聚合后的LSA
1. 在协议视图配置.命令：[r4-ospf-1]asbr-summary 172.16.0.0 255.255.252.0
1. 只对5类、7类的LSA 进行聚合;
1. ASBR聚合不会影响ASBR本机的路由，对于下游设备的路由表有影响，下游设备只学到聚合的路由。
1. ASBR 聚合后，会在ASBR本机上产生一条该聚合的黑洞路由，来防止环路出现;

[R3]ip route-static 192.168.4.0 22 NULL 0

[R3-ospf-1]asbr-summary 192.168.4.0 255.255.252.0 cost 99   //修改聚合路由的默认COST，下游设备学习到此路由COST值不变

注：使用聚合实现路由过滤，在聚合后加入not-advertise参数