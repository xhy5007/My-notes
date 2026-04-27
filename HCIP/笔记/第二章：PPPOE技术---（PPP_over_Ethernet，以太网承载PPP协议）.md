```mermaid
mindmap
  root((PPPoE技术))
    技术背景
      以太网缺陷
        缺少验证机制
        IP分配困难
      PPP优势
        支持身份验证
        支持计费
    协议特点
      CS架构
      封装原理
        以太网帧承载PPP报文
    工作阶段
      Discovery阶段
        PADI 广播寻找
        PADO 单播提供
        PADR 单播请求
        PADS 会话确认
        核心结果: Session ID
      PPP Session阶段
        LCP协商
        认证 (PAP/CHAP)
        NCP协商 (IP地址)
      Termination阶段
        PADT报文
        拆除链路
```

# **一、PPPOE技术背景**

1、以太网接入技术无法提供用户身份验证

2、以太网接入需要实现给用户自动分配公网IP地址

3、以太网接入受到双绞线距离限制

以太网接入图：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBd23669f30a8f9da3620af0fe1c73a976/WEBRESOURCEd10f4840363dbe020bd3420c626a8188/40907)

大型园区网接入图：

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBd23669f30a8f9da3620af0fe1c73a976/WEBRESOURCE00634c7e383c25ecb1bfa1174f8bfe8f/40910)



# **二、PPPOE简介**

PPPoE协议采用C/S方式，将ppp报文封装在以太网帧之内，使ppp帧可以在以太网上进行传输，同时让以太网可以具备PPP功能，在以太网上提供点到点的连接

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBd23669f30a8f9da3620af0fe1c73a976/WEBRESOURCE05a37ab5ea04566cacc736e41fc306c9/27566)



# **三、PPPoE工作的三个阶段**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBd23669f30a8f9da3620af0fe1c73a976/WEBRESOURCEb6cf130587137987e0e38b9c84787a1c/27591)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBd23669f30a8f9da3620af0fe1c73a976/WEBRESOURCE0380a98a2a8caac5e51682cf68da9da8/27594)



1、Discovery阶段：协商PPPoE的seession-ID,用来区分不同的逻辑点

（1）由客户端向服务器端广播发送PADI报文，询问PPPoE服务器

（2）PPPoE服务器收到消息后，单播回复一个PADO，告诉客户端由自己给其提供服务

（3）客户端单播发送PADR报文请求PPPoE服务器端提供服务

（4）服务器单播发送PADS报文同意建立会话连接---session id实现



2、ppp session协商阶段：在PPPoE会话中进行ppp协商

1. LCP协商
1. 身份验证
1. NCP协商



3、PPPO
E会话终结，PPPoE断开

当PPPoE客户端希望关闭连接时，会向PPPoE服务器端发送一个PADT（PPPoE Active Discovery Terminate）报文，用于关闭连接。
同样，如果PPPoE服务器端希望关闭连接时，也会向PPPoE客户端发送一个PADT报文。



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBd23669f30a8f9da3620af0fe1c73a976/WEBRESOURCEfa53a98682fa491b924cd70ed94cdd7c/39153)

# **四、实验**

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBd23669f30a8f9da3620af0fe1c73a976/WEBRESOURCE806127099a27cbdbbf637ad9ef6fff61/27600)