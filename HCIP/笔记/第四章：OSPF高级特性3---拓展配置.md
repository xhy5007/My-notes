# **一、OSPF安全特性**

## **1、OSPF报文验证**

区域验证模式：在区域下配置一致的密码才能加入同一个区域。[r3-ospf-1-area-0.0.0.0]authentication-mode md5 1 cipher 123456

接口验证模式：链路两端的接口必须配置一致的密码才能建立邻居关系

[r5-GigabitEthernet0/0/0]ospf authentication-mode md5 1 cipher 123456

虚链路认证（本质接口认证）：[r4-ospf-1-area-0.0.0.1]vlink-peer 3.3.3.3 md5 1 cipher 123456

注：接口认证高于区域认证，只要接口验证通过，区域验证哪怕失败，也不影响邻接关系建立



## **2、禁止端口发送OSPF报文**

配置静默接口：[r5-ospf-1]silent-interface GigabitEthernet 0/0/2

特点：此接口能接收OSPF的协议报文，但是不能发送协议报文



## **3、路由过滤**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/WEBRESOURCE1b43f1af18376b224fc289b00ba61d27/43347)

### **（1）****filter-policy--过滤策略**

在协议视图下进方向配置：

特点：影响本机及下游所有区域的路由器的OSPF表的计算，过滤本机及下游路由器的LSDB表中的LSA

```ada
[R2]acl 2000
[R2-acl-basic-2000]rule deny source 10.1.12.0 0.0.0.255
[R2-acl-basic-2000]rule permit source any
[R2-ospf-1]filter-policy 2000 import 
```



在协议视图下，出方向配置：没有任何过滤效果

```ada
[R2]acl 2001
[R2-acl-basic-2001]rule deny source 11.11.11.11 0.0.0.0
[R2-acl-basic-2001]rule permit source any
[R2-ospf-1]filter-policy 2001 export
```



### **（2）filter---过滤器**

在传入区域的区域视图配置：对传入本区域的3类LSA做过滤

特点：

1. 影响本机LSDB的学习，不影响OSPF路由的计算；
1. 影响传入区域的所有路由器的LSA学习及OSPF路由计算；
1. 不影响其他区域的LSA学习及路由计算  。例如：不影响area 3

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/WEBRESOURCE23c57b160152bacbe4e560bc561fab4c/43364)



```ada
[R2]acl 2000
[R2-acl-basic-2000]rule deny source 10.1.12.0 0.0.0.255
[R2-acl-basic-2000]rule permit source any
[R2-ospf-1]a 0
[R2-ospf-1-area-0.0.0.0]filter 2000 import     // 对传入本区域的3类LSA做过滤，影响本区域所有路由器的LSA学习及OSPF路由计算，
不影响其他区域的LSA学习及路由计算  
```

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/WEBRESOURCEb6de7f51c9e09d1ca60c75a11cef0939/43313)



在传出区域的区域视图配置：对传出本区域的3类LSA做过滤

特点：

1. 影响其他区域的路由器对此LSA的学习及OSPF路由的计算
1. 影响本机LSDB的学习，不影响OSPF路由的计算；
1. 影响其他所有区域的LSA学习及路由计算 。例如：影响area 0、area 3

```ada
[R2]acl 2001
[R2-acl-basic-2001]rule deny source 11.11.11.11 0.0.0.0
[R2-acl-basic-2001]rule permit source any
[R2]ospf 1
[R2-ospf-1]a 1 
[R2-ospf-1-area-0.0.0.1]filter 2001 export    //对传出本区域的3类LSA做过滤，影响其他区域的路由器对此LSA的学习及OSPF路由的计算
```

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/WEBRESOURCE5f267093e266924af0bcf213cac71d39/43329)

### **（3）过滤3类5类/7类LSA：not-advertise 实现-----见高级特性课件2**



# **二、加快收敛**

1、修改hello时间：[r5-GigabitEthernet0/0/0]ospf timer hello 5

2、修改死亡时间：[r1-GigabitEthernet0/0/0]ospf timer dead 20

3、其他计时器：

重传时间默认5S：[r5-GigabitEthernet0/0/0]ospf timer retransmit ?-----重传LSA的

INTEGER<1-3600> Second(s)



4、修改ospf的网络类型：[R2-GigabitEthernet0/0/0]ospf network-type ?

注：两端网络类型不一致，导致邻居关系建不起来



# **三、缺省路由**

（1）3类缺省：特殊区域自动下发，优先级10

（2）5类缺省：手工配置，优先级150

命令：[r2-ospf-1]default-route-advertise --- 相当于将本设备上通过其他方式学到的缺省路由，重发布到OSPF网络当中

命令：[r2-ospf-1]default-route-advertise always --- 在设备上没有其他网络学来缺省信息时，可以强制下发一条5类缺省。

（3）7类缺省：

自动下发，通过配置特殊区域自动下发，优先级150-----nssa

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/WEBRESOURCE7746c103fb9746b0886cbf57af70c591/43370)



# **四、路由控制**

1、优先级

[r3-ospf-1]preference 50 --- 修改OSPF路由默认优先级，只影响本机OSPF路由的学习

[r3-ospf-1]preference ase 100 --- 修改域外导入的路由的默认优先级。



2、开销值

[r3-ospf-1]bandwidth-reference 1000 --- 修改参考带宽需要将所有OSPF网络中的设备都改成相同的。

[r3-GigabitEthernet0/0/0]undo negotiation auto --- 关闭自动协商

[r3-GigabitEthernet0/0/0]speed 10---修改接口真实传输速率

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/WEBRESOURCE5eb426ea3801b2c9f3d7421ae34d4289/35566)

[r3-GigabitEthernet0/0/0]ospf cost 1000 --- 修改接口开销值



# **五、显示OSPF的错误统计信息**

[R1]dis ospf error

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/WEBRESOURCE8ef072e3444c6ddebccd716144f109b3/16324)



# 

# **六、附录E（了解）**

当ASBR引入多条网络地址一致，掩码不一致的外部路由时，路由器会把除了第一条以外的，外部路由产生的5类LSA的LS ID的主机位做全反（0变255）操作，来防止LS ID冲突

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/WEBRESOURCE56d8f52099c5c2ffb90428668c3fe9f8/11217)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/WEBRESOURCE18a91916541384e58d2d3ab6333b1ed2/11248)



# **七、OSPF防环**

区域间防环：牢记区分划分原则，就可避免环路

区域内防环：OSPF区域内部计算出的路由信息是不会存在环路的，这主要得益于OSPF使用的算法 --- SPF（最短路径优先）算法。



# **八、OSPF选路原则**

选路示例：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/WEBRESOURCE9d137685c82ca98386714d4c5b8c1ad9/11176)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/WEBRESOURCE86f38a9c69dd9645c36522b924a0de85/11178)

课堂实验演示：域外路由引入



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/654cfaef4bb7eeb9d4095cf62c6318b3/35486)

发现在R3上学到了两条了引入的路由，观察AS内部的cost有没有变化

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/98bf39b4ebafac4959d5098ab59a4fe0/35487)

修改R3的G0/0/1的cost=2000,观察AS内部的cost有没有变化

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/28e167ca8ed62f20272a9ce7503168cc/35488)

修改R3的G0/0/0的cost=3000,观察AS内部的cost有没有变化

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/8dca35b7bf830b6287a715dfef3cb7a5/35489)

测试：在R1和R2上更改引入静态路由的类型为1类，查看R3路由表的变化，此时cost值明显被改变，会计算内部cost值，这就说明此时引入的是第1类外部路由

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/1c21570332b76ae78e759c5a67bf3cdb/35490)



测试：为啥要有1类和2类外部路由的区分

如果内部cost，走移动的cost值>走电信的，那去往192.168.1.0/24网段的，优先选择走电信，但现在我们就想让它走移动，这时需要通过2类路由引入，才能达到这个效果。即在做引入的时候就改变原始的引入的cost值。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/85bd4f406fe294e7b2a5c8d6639979f5/35491)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/bcc39ff018c7a821084a859f47dabac0/35492)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB4a7bc4a8b13ed8281f70f8eace294c90/6998140ee96635cd5360540836548804/35493)



# **九、OSPF综合实验**

实验要求

1、R4为ISP，其上只配置IP地址；R4与其他所直连设备间均使用公有IP；

2、R3-R5、R6、R7为MGRE环境，R3为中心站点；

3、整个OSPF环境IP基于172.16.0.0/16划分；除了R12有两个环回，其他路由器均有一个环回IP

4、所有设备均可访问R4的环回；

5、减少LSA的更新量，加快收敛，保障更新安全；

6、全网可达；