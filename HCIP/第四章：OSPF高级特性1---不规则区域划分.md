## **一、OSPF不规则区域类型**

产生原因：区域划分不合理，导致的问题

1、非骨干区域无法和骨干区域保持连通

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBd02f2c2e9d72ffa0ae962a13e46dfaa2/WEBRESOURCEfe88d6259c4c86aa605df2da5be66540/8321)



2、骨干区域被分割

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBd02f2c2e9d72ffa0ae962a13e46dfaa2/WEBRESOURCE7f6669613f33fec4ca87767039c88832/8324)



造成后果：非骨干区域没和骨干区域相连，导致ABR将不会帮忙转发区域间的路由信息。非骨干区域无法和骨干区域保持连通



# **二、解决方案**

1、使用虚连接（实验演示）

演示一：非骨干区域无法和骨干区域保持连通

方法1：虚连接

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBd02f2c2e9d72ffa0ae962a13e46dfaa2/WEBRESOURCE4bdb5151e4ace70e4b71f3617d3af0d1/11121)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBd02f2c2e9d72ffa0ae962a13e46dfaa2/WEBRESOURCEb6504265bfdf8ca3e270e50b8a938ff1/8306)

方法2：路由引入（重发布），

**重发布：**在运行不同协议或不同进程的边界设备（ASBR --- 自治系统边界路由器，协议边界路由器）上，将一种协议按照另一种协议的规则发布出去

R4充当ASBR的角色，在其上运行两个OSPF协议。然后利用重发布进行共享。

[R4-ospf-1]import-route ospf 2

[R4-ospf-2]import-route ospf 1



演示二：骨干区域被分割

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBd02f2c2e9d72ffa0ae962a13e46dfaa2/WEBRESOURCEc2748f71396fe073046d3bf780a64023/8328)



![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBd02f2c2e9d72ffa0ae962a13e46dfaa2/WEBRESOURCE91c08aea05d36fbc7c12b6f5e6965f37/14120)

显示虚连接信息

![image](https://share.note.youdao.com/yws/public/resource/a54c121161e06e2d61673c9a8773f722/xmlnote/WEBd02f2c2e9d72ffa0ae962a13e46dfaa2/WEBRESOURCE0c54b155fdf82053e06850ec063b473a/8336)



虚连接特征：

只能在两个区域的边界路由器上配置；

在中间区域的区域视图下配置；

只能穿越一个区域；

注：虚连接不能穿越stub区域、NSSA区域

友情提示提示：虚连接的配置会给路由器带来一些额外的负担，尽量在早期规划的区域的时候，合理规划