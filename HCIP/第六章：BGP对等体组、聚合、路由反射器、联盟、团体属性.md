# **一、大规模BGP网络所遇到的问题**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE50eaaecd3bc568e4a7da29b1c11bfb7d/11827)

1. BGP对等体众多，配置繁琐，维护管理难度大
1. BGP路由表庞大，对设备性能提出挑战
1. IBGP全连接，应用和管理BGP难度增加，邻居数量过多
1. 路由变化频繁，导致路由更新频繁（广域网比局域网更容易发生拥塞，会导致BGP邻居一会UP，一会DOWN，引起路由表的频繁变化，这种更新变化会被更新到全网，引起网络不稳定）



# **二、解决大规模BGP网络所遇到的问题**

1、BGP对等体众多

1. 对等体组（Peer Group）-----是一种简单的命令配置方式，不是某个技术或者协议，可以简化一下配置
1. BGP团体（Community ）----是用来批量匹配某种具有相同特征路由的东西，也可以影响路由的选路

2、BGP路由表庞大

1. BGP路由聚合

3、IBGP全连接（邻居数量太多）

1. BGP路由反射（Route Reflection）
1. BGP联盟（Confederation）

4、路由变化频繁

1. BGP路由衰减（Route Dampening）



# **三、对等体组**

定义：BGP 对等体组（Peer Group）是一些具有某些相同属性的对等体的集合。通过对等体组可以简化配置

特点：根据对等体所在的AS，对等体组可分为IBGP对等体组和EBGP对等体组。

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE1d9846a916ee83254569357bcbfbaa6c/17183)

建邻要求：

1. EBGP :R1-R2
1. IBGP:R2分别于R3/R4/R5

注意：配置EBGP对等体组--------一般不常用，因为只有同一个AS中的EBGP邻居才能加入一个对等体组，如果EBGP邻居分布在多个AS，则需要给每个AS的EBGP邻居，配置一个对等体组。比较麻烦

## **配置IBGP对等体组**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE0c81bafb08f6ebc81a282d68f8076d4a/12218)

```abap
[R2-bgp]group in internal
[R2-bgp]peer 3.3.3.3 group in
[R2-bgp]peer 4.4.4.4 group in
[R2-bgp]peer 5.5.5.5 group in
[R2-bgp]peer in connect-interface l0
[R2-bgp]peer in next-hop-local
 
[R3-bgp]peer 2.2.2.2 as 200
[R3-bgp]peer 2.2.2.2 connect-interface l0
R4和R5上做和R3相同的配置
```



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE7b9842a5febb8297601015f5906be707/12225)

## **配置EBGP对等体组**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCEd0e2a7374d71f0e04e61035959572cb6/12214)



```abap
[R2-bgp]group ex external 
[R2-bgp]peer ex as-number 100
[R2-bgp]peer 100.1.1.1 group ex
```



# **四、BGP路由聚合**

作用：减小路由表规模

## **1、自动聚合：**

1. 只能对引入的IGP的路由进行聚合；
1. 只能将明细路由汇总到主类，这会造成路由黑洞；
1. 华为设备默认关闭自动聚合功能；
1. 只能在始发路由器上进行配置；

[r1-bgp]summary automatic 开启自动聚合

自动聚合后，会出现状态码S（suppressed）,代表被抑制的路由信息将不会再加表和传递了



实验演示1：只针对重发布的路由条目生效，不对自身宣告的路由生效

在R3上加几个环回口172.16.0.1 24和172.16.1.1 24，并做BGP宣告

[R3-bgp]summary automatic  开启自动聚合

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE21fec7aac0e7428abad0204ae6dfbfc3/17190)

实验演示2：引入直连

[R3-bgp]import-route direct

[R3-bgp]summary automatic

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE58a09489157ef871c77ae062bde90b85/12246)

## **2、手工聚合：**

1. 可实现精确汇总：[R1-bgp]aggregate 172.16.0.0 16
1. 可以在任何路由器上对BGP路由进行聚合；

出现问题：明细路由依然被通告，有形成环路的隐患

解决方法：[R1-bgp]aggregate 172.16.0.0 16 detail-suppressed  //抑制明细路由，此命令不通告明细路由；



实验演示1：

[R3-bgp]network 172.16.1.1 24

[R3-bgp]network 172.16.0.1 24

[R3-bgp]aggregate 172.16.0.0 23

测试：查看路由表，聚合成功，但明细路由也存在，并没有达到缩小路由表规模的情况

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE1e58d729be2549e9c3a88bd09e206cd3/12251)

抑制明细路由。达到了缩小路由表的规模

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE89ac651f724b9771168049c0d6344963/12256)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE58872fd367051892dd5b114c0ddf424f/12259)

1. **AS-PATH属性不继承。---了解**

BGP路由聚合不抑制明细，带来的潜在的环路风险

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE666a8638e6804f9f98f9303e62636e03/44863)

当AS300中的10.1.10.0/24路由失效时，若AS 300中有人访问10.1.10.0/24时，就会匹配10.1.0.0/16在AS 300-400-200中形成环路

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE8a6e1d4c769c49aea58ea7e39f6972ff/44864)



解决方法：继承AS-PATH属性，就可防环（AS 300就可以不接收AS 400传递的汇总路由）

[R3-bgp]aggregate 172.16.0.0 255.255.0.0 as-set



常见汇总命令：

[R1-bgp]aggregate 172.16.0.0 16 detail-suppressed as-set  汇总的路由可继承明细路由的路径属性，其中AS_PATH属性最关键。

[R1-bgp]aggregate 172.16.0.0 16 suppress-policy ‘路由策略名字’通告汇总路由，并有选择性的抑制明细路由

```plain text
[R1] ip ip-prefix aa index 10 permit 172.16.1.0 24
 
[R1] route-policy bb permit node 10
[R1-route-policy] if-match ip-prefix aa
 
[R1] bgp 100
[R1-bgp] aggregate 172.16.0.0 23 suppress-policy bb
```



# **五、路由反射器**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE72d7cd716557cf0fd6d1c1ae671c061a/12371)

1、定义：BGP反射器能把从IBGP邻居学习的路由反射给其他IBGP邻居

2、作用：用于替代IBGP全连接，减少IBGP邻居数量；解决BGP路由黑洞问题；

3、角色：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE7c6796b06883256fbbaf3f0f863191ef/12374)

RR(router  reflect)：路由反射器

反射器角色：

1. client：RR客户机
1. 非客户机



注：将一台BGP路由器指定为RR的同时，还需要指定其Client。至于Client本身，无需做任何配置，它并不知晓网络中存在RR

思考：为啥不把一个AS内的所有路由器都变成客户机？还有非客户机的存在？（RTE\RTF）

答案：因为一个AS就是一个城域网。一个城市的城域网中有好多路由器，一个城域网怎么可能只有一个反射器。比如：高新区选一个反射器，不可能让他给雁塔区啊、长安区啊提供反射服务。一个反射器给他特定的客户机提供反射服务。我们可以在 这些区各选一个反射器，让他们给本区的路由器提供反射服务。也就是说可以在一个AS内建立多个路由器反射群。两个反射群之间需要建立邻居关系。它们之间需要传递路由。



4、反射规则

从非反射客户端接受的路由，仅反射给客户端；

从客户端接受到的路由，反射给所有客户端和非客户端，路由始发者除外；

从EBGP接收的路由，反射给所有的客户端和非客户端；

注：一个反射群里的所有反射客户端只需与反射器建立IBGP邻居关系

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCEac8a28d6e3b24929874d0762742cbe97/44883)



5、反射集群：由反射器和客户端组成的网络范围；

如果存在多个反射器，配置相同的cluster_id

思考：当RR故障后，会发生啥？怎么解决？

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCEe92f77d0080257a17ae66a79ad00b44f/12403)

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCEb5b976c12542069077d178e8e9b3b57e/12406)

思考：RR之间需要建立邻居关系？------当然需要

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCEbde75d5541d650416885e522ceaf3478/12409)

但是这样的组网容易形成环路。（一条路由从客户机1传到RR，RR传到备用RR，备用RR再反射给客户机1）

6、路由反射存在问题：使得IBGP水平分割原则失效，会导致环路的产生

7、解决方法：

通过两个路径属性来避免环路

（1）cluster_list（集群列表）：

1. 类似AS_PATH，每个RR都有一个Cluster id,默认为路由器的router id，可手工修改；
1. 同一个AS内的Cluster id必须相同，才能有防环作用。
1. 路由传递过程中，把经过的反射器的Cluster_id依次记录在Cluster_list中；
1. Cluster_list用于反射器防环，当反射器收到BGP路由时，如果本机的Cluster_id出现在Cluster_list中，则丢弃该路由；

（例如：一条路由从客户机1传到RR，RR会将Cluster id为1.1.1.1这个属性写给这个路由，RR再传到备用RR，会检查自身的Cluster id，发现学到的路由Cluster id与自己一样，就直接丢弃）

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE8e4b4f24f6d97f4a6adeeee20a653e7a/12425)

1. Cluster-list用于路由优选，短的优先



**特殊情况：不受**Cluster-list防环机制的约束，依然形成环路

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE0ab2a8d5d5d7cd88f1a8fa1b1b5435b7/13529)

思考：一条路由从客户机3传到备用RR，再传到rr，rr会不会传给客户机3？

答案：会，解决方法用originator_ID



（2）originator_ID：起源ID

1. 由路由反射器反射一条路由时产生，会在反射出去的路由中增加originator_ID，它就是本地AS内路由始发者的router-id；
1. 即使这条反射路由经过多个RR，当BGP路由器收到一条携带Originator_ID属性的IBGP路由，并且Originator_ID属性值与自身的Router ID相同，则它会忽略关于该条路由的更新。
1. originator_ID用于路由优选，短的优先



8、总结BGP的防环机制

（1）AS_PATH：解决AS之间的环路问题

（2）IBGP的水平分割：解决一个AS内部IBGP邻居之间环路问题

（3）Cluster-list防环：当一个反射群中存在多个反射器时，通过Cluster-list防环，

（4）originator_ID：在多个反射群之间，不同反射器的环路问题，通过originator_ID防环



9、注意事项

反射路由无法使用策略去更改路由属性



10、实验配置

[R2-bgp]peer 3.3.3.3 reflect-client      //在反射器上配置其反射客户机

[R2-bgp]reflector cluster-id 2.2.2.2    //配置反射器的集群id



11、实验作业

BGP路由反射器实验

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCEb3e8d1a919199ef7a1b6201521f52be1/13554)

# **六、BGP联盟**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE8119c94f513c044a1ac4e67d6ee3b7b4/13559)

1、定义：处理AS内部的IBGP网络连接激增的另一种方法，间接避免了IBGP环路问题

2、原理：联盟将一个自治系统划分为若干个子自治系统，每个子自治系统内部的IBGP对等体建立全连接关系，子自治系统之间建立联盟内部EBGP连接关系。



3、注意点：

子AS使用私有AS编号；

其他真实AS的路由器仍然和联盟AS建立EBGP邻居；

跨越子AS的EBGP邻居仍然需要更改下一跳为本机；

每个联盟里有一台路由器和其他联盟中一个路由器建立邻居关系就行；

**实际场景中一般不用联盟，因为他会改变邻居关系**



4、配置命令：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE63072d5fd7c5a131f58f98bd28c8a3e0/17214)

[R2]bgp 65001        //进入子AS编号

[R2-bgp]confederation id 200    //申明自己的大号

[R2-bgp]confederation peer-as 65002  //申明自己的联盟同伴

[R2-bgp]peer 100.1.1.1 as-number  100       指定EBGP邻居

[R2-bgp]peer 3.3.3.3 as-number 65001

[R2-bgp]peer 3.3.3.3 connect-interface LoopBack 0

[R2-bgp]peer 3.3.3.3 next-hop-local

[R2-bgp]peer 100.2.2.5 as-number 65002

[R2-bgp]peer 100.2.2.5 next-hop-local   //跨越子AS的EBGP邻居仍然需要更改下一跳为本机；不然从R1传过去的网段，R5收不到



# **七、BGP团体属性（community）**

（1）定义：相当于路由的标记，一组具有相同特征的目的地的集合，可针对特定的路由设置特定的community属性值

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE56d0323fc5d3da74be371d15a3348f04/13574)

问题：网段不连续，ACL写起来比较麻烦。

解决方法：在每个分部的市场部路由的出口设备上，打上100:1的团体属性。

（2）社团属性表达方法：

本质由32位二进制构成，拥有多种表达格式，

比如：可使用10进制；

或者16位二进制，前16位一般是本地AS的AS号，后16位是自定值；

（3）公认的社团属性：

1. Internet:抓取的BGP流量，默认情况下都属于Internet，可以被通告给所有的BGP对等体；
1. no-advertise：路由信息无法发送给自己的IBGP对等体或EBGP对等体；
1. no-export：路由信息可以发给自己的BGP对等体，但对等体不能离开这个AS,无法发给自己的EBGP；但可以发给自己的联邦EBGP对等体；
1. no-export-subconfed路由信息可以发给自己的BGP对等体，但对等体不能离开这个AS,无法发给自己的EBGP；也不可以发给自己的联邦EBGP对等体；



注：目前大部分厂商默认在传递BGP路由信息时是不传递社团属性的，如果需要传递社团属性，则需要通过命令开启



（4）配置演示

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCEbe8af130f1a047d51f5ecf8ed64d2932/17234)



配置步骤：

抓取流量

[R1]acl 2000

[R1-acl-basic-2000]rule permit source 192.168.1.0 0.0.0.255

配置路由策略，打上团体属性

[R1]route-policy com permit node 10

[R1-route-policy]if-match acl 2000

[R1-route-policy]apply community no-export

[R1]route-policy com permit node 20

调用路由策略

[R1]bgp 100

[R1-bgp]peer 100.1.1.2 route-policy com export

[R1-bgp]peer 100.1.1.2 advertise-community      //通告团体属性，使其他邻居学到

[R2]dis bgp routing-table 192.168.1.0 24   //查看一条路由的详细信息



# **八、联盟和团体属性实验**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE0d54848b92c7217e4aef7f0f6043318f/13568)



# **九、BGP路由衰减（****Route Dampening****）---了解**

作用：用来解决路由振荡（Route flaps）的问题

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE0e976b3d2011de4a975ff84788f1f08a/30589)



路由衰减方法：每震荡一次，就给其一个**惩罚值(默认是1000)**，惩罚值固定加1000，不能超过**抑制阈值**，一旦超过这个值，不管这个路由的死活，都不在学习它；

惩罚值每隔一段时间，自动衰减一半。直到惩罚值衰减到某一个**再使用阈值**之下，方可重新学习这条路由。

```abap
衰减参数：
Suppress Threshold（抑制阈值，默认2000）：当惩罚值超过此阈值时，路由被抑制（不再通告）。
Reuse Threshold（重用阈值，默认750）：当惩罚值衰减到此值以下时，路由重新被通告。
Half-life Time（半衰期，默认15分钟）：惩罚值每15分钟减半
Max Suppress Time（最大抑制时间，默认4倍半衰期，60min）：路由被抑制的最长时间
```

配置命令：

```abap
bgp 100
dampening                      # 启用BGP路由衰减（使用默认参数）
dampening 15 750 2000 60      # 自定义参数：半衰期、重用阈值、抑制阈值、最大抑制时间
```



# **十、BGP综合实验**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEB2adf2ea4c7ddebfee095dccca54e83db/WEBRESOURCE0db0af978fe2b07398f2d6e6b5b08061/17374)