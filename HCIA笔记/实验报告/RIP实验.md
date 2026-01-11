# RIP实验报告

## 一、实验名称

基于 RIP v2 的多路由网络互通、路由汇总与防环实验

# 二、实验目的

1. 理解 RIP v2 路由协议的工作原理及配置方法；
2. 掌握 network 方式宣告 RIP 路由的方法；
3. 学会使用 Loopback 接口模拟多网段环境；
4. 掌握 RIP 路由汇总的配置方法及应用场景；
5. 理解 RIP 路由环路产生的原因及防环机制；
6. 实现整个网络通过 RIP 协议达到全网互通。

# 三、实验拓扑与网络规划

## 1. 拓扑说明

实验拓扑由三台路由器（AR1、AR2、AR3）、两台二层交换机（LSW1、LSW2）以及四台主机（PC1–PC4）组成。
AR1 与 AR3 分别连接接入层网络，AR2 位于网络中间，作为路由汇总与防环配置的边界路由器。

## 2. IP 地址规划

| 设备 | 接口      | IP 地址        | 备注         |
| ---- | --------- | -------------- | ------------ |
| AR1  | Loopback0 | 1.1.1.1/32     | 测试网段     |
| AR1  | GE0/0/0   | 192.168.1.1/24 | 左侧接入网关 |
| AR1  | GE0/0/1   | 12.1.1.1/24    | AR1–AR2 互联 |
| AR2  | GE0/0/0   | 12.1.1.2/24    | AR1–AR2 互联 |
| AR2  | GE0/0/1   | 23.1.1.2/24    | AR2–AR3 互联 |
| AR3  | GE0/0/0   | 23.1.1.3/24    | AR2–AR3 互联 |
| AR3  | GE0/0/1   | 192.168.2.1/24 | 右侧接入网关 |
| AR3  | Loopback1 | 172.16.1.1/24  | 模拟网段     |
| AR3  | Loopback2 | 172.16.2.1/24  | 模拟网段     |
| AR3  | Loopback3 | 172.16.3.1/24  | 模拟网段     |

# 四、实验原理说明

## 1. RIP v2 路由协议

RIP（Routing Information Protocol）是一种典型的距离矢量路由协议，以跳数作为度量值，最大跳数为 15。RIP v2 支持 CIDR 和 VLSM，并通过周期性更新方式维护路由表。

## 2. Loopback 接口的作用

Loopback 接口为逻辑接口，不依赖物理链路，始终保持 up 状态。本实验中通过 Loopback 接口模拟多个网络网段，使 RIP 能学习多条路由，从而验证路由汇总与防环机制的有效性。

## 3. 路由汇总原理

路由汇总是将多条连续的子网路由合并为一条较大的路由，从而减少路由表规模，提高网络稳定性。本实验中将 172.16.1.0/24、172.16.2.0/24、172.16.3.0/24 汇总为 172.16.0.0/22。

## 4. 防环机制说明

RIP 为距离矢量协议，容易产生路由环路。路由汇总会生成新的路由信息，可能破坏 RIP 原有的水平分割机制，因此需要在汇总边界路由器上启用防环机制（Split Horizon），防止路由回传造成环路。

# 五、实验配置步骤与命令

## 1. AR1 配置

```bash
<AR1> system-view
[AR1] sysname AR1

[AR1] interface GigabitEthernet0/0/0
[AR1-GE0/0/0] ip address 192.168.1.1 255.255.255.0
[AR1-GE0/0/0] quit

[AR1] interface GigabitEthernet0/0/1
[AR1-GE0/0/1] ip address 12.1.1.1 255.255.255.0
[AR1-GE0/0/1] quit

[AR1] interface LoopBack0
[AR1-LoopBack0] ip address 1.1.1.1 255.255.255.255
[AR1-LoopBack0] quit

[AR1] rip 1
[AR1-rip-1] version 2
[AR1-rip-1] undo summary
[AR1-rip-1] network 192.168.1.0
[AR1-rip-1] network 12.0.0.0
[AR1-rip-1] network 1.0.0.0
[AR1-rip-1] quit
```

## 2. AR2 配置（路由汇总与防环）

```bash
<AR2> system-view
[AR2] sysname AR2

[AR2] interface GigabitEthernet0/0/0
[AR2-GE0/0/0] ip address 12.1.1.2 255.255.255.0
[AR2-GE0/0/0] quit

[AR2] interface GigabitEthernet0/0/1
[AR2-GE0/0/1] ip address 23.1.1.2 255.255.255.0
[AR2-GE0/0/1] quit

[AR2] rip 1
[AR2-rip-1] version 2
[AR2-rip-1] undo summary
[AR2-rip-1] network 12.0.0.0
[AR2-rip-1] network 23.0.0.0
[AR2-rip-1] quit

[AR2] interface GigabitEthernet0/0/0
[AR2-GE0/0/0] rip summary-address 172.16.0.0 255.255.252.0
[AR2-GE0/0/0] rip split-horizon
[AR2-GE0/0/0] quit
```

## 3. AR3 配置

```bash
<AR3> system-view
[AR3] sysname AR3

[AR3] interface GigabitEthernet0/0/0
[AR3-GE0/0/0] ip address 23.1.1.3 255.255.255.0
[AR3-GE0/0/0] quit

[AR3] interface GigabitEthernet0/0/1
[AR3-GE0/0/1] ip address 192.168.2.1 255.255.255.0
[AR3-GE0/0/1] quit

[AR3] interface LoopBack1
[AR3-LoopBack1] ip address 172.16.1.1 255.255.255.0
[AR3] interface LoopBack2
[AR3-LoopBack2] ip address 172.16.2.1 255.255.255.0
[AR3] interface LoopBack3
[AR3-LoopBack3] ip address 172.16.3.1 255.255.255.0

[AR3] rip 1
[AR3-rip-1] version 2
[AR3-rip-1] undo summary
[AR3-rip-1] network 23.0.0.0
[AR3-rip-1] network 192.168.2.0
[AR3-rip-1] network 172.16.0.0
[AR3-rip-1] quit
```

# 六、实验验证

1. 在各路由器上使用 `display ip routing-table` 查看路由表，确认 AR1 仅学习到一条 172.16.0.0/22 汇总路由；
2. 在 PC1、PC2 上 ping PC3、PC4，验证跨网段通信；
3. 测试从左侧网络 ping 172.16.1.1、172.16.2.1，验证 RIP 路由可达性。

# 七、实验结果与分析

实验结果表明：

- RIP v2 能正确学习并传播各网络路由；
- 路由汇总有效减少了路由表条目数量；
- 在 AR2 上启用防环机制后，网络运行稳定，未出现路由环路；
- 全网实现了互通，满足实验设计要求。

# 八、实验总结

通过本次实验，加深了对 RIP v2 路由协议的理解，掌握了 Loopback 接口的使用方法以及路由汇总与防环的配置原则。实验表明，合理选择汇总位置并配合防环机制，是保证网络稳定运行的重要前提。本实验为后续学习 OSPF 等链路状态路由协议奠定了基础。