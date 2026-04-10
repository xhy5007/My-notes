# OSPF 实验报告

## 一、实验目的

1. 掌握 OSPF 多区域（Area 0 与 Area 1）的基本配置方法
2. 理解广播网络中 DR / BDR 的选举机制
3. 掌握 OSPF 认证与缺省路由引入的应用场景
4. 实现全网互通并验证路由学习结果

## 二、实验拓扑说明

- **Area 0（骨干区）**：AR1、AR2、AR3、LSW1
- **Area 1（非骨干区）**：AR3、AR4、AR5
- **AR3 为 ABR（区域边界路由器）**
   AR5 不运行 OSPF，通过缺省路由实现访问外部网络

## 三、IP 地址规划

### 左侧（Area 0）

| 设备 | 接口    | IP 地址         |
| ---- | ------- | --------------- |
| AR1  | GE0/0/0 | 192.168.1.1 /25 |
| AR2  | GE0/0/0 | 192.168.1.2 /25 |
| AR3  | GE0/0/0 | 192.168.1.3 /25 |

###  右侧（Area 1）

| 链路      | IP 地址                         |
| --------- | ------------------------------- |
| AR3 – AR4 | 192.168.1.13/30 192.168.1.14/30 |
| AR4 – AR5 | 192.168.1.17/30 192.168.1.18/30 |

## 四、接口 IP 配置

### AR1

```
interface GigabitEthernet0/0/0
 ip address 192.168.1.1 255.255.255.128
```

### AR2

```
interface GigabitEthernet0/0/0
 ip address 192.168.1.2 255.255.255.128
```

### AR3

```
interface GigabitEthernet0/0/0
 ip address 192.168.1.3 255.255.255.128

interface GigabitEthernet0/0/1
 ip address 192.168.1.13 255.255.255.252
```

### AR4

```
interface GigabitEthernet0/0/0
 ip address 192.168.1.14 255.255.255.252

interface GigabitEthernet0/0/1
 ip address 192.168.1.17 255.255.255.252
```

### AR5

```
interface GigabitEthernet0/0/0
 ip address 192.168.1.18 255.255.255.252
```

## 五、OSPF 配置

### Area 0 配置（AR1 / AR2 / AR3）

#### AR1

```
ospf 1
 area 0
  network 192.168.1.0 0.0.0.127
```

#### AR2

```
ospf 1
 area 0
  network 192.168.1.0 0.0.0.127
```

#### AR3

```
ospf 1
 area 0
  network 192.168.1.0 0.0.0.127
```

### Area 1 配置（AR3 / AR4）

#### AR3

```
ospf 1
 area 1
  network 192.168.1.12 0.0.0.3
```

#### AR4

```
ospf 1
 area 1
  network 192.168.1.12 0.0.0.3
  network 192.168.1.16 0.0.0.3
```

## 六、AR5 不宣告 OSPF可使用缺省路由

```
ip route-static 0.0.0.0 0.0.0.0 192.168.1.17
```

AR5 通过缺省路由将所有未知流量指向 AR4

## 七、验证与测试

```
display ospf peer
display ip routing-table
ping 192.168.1.1
ping 192.168.1.18
```

实验结果表明：

- Area 0 与 Area 1 路由正常互通
- AR5 通过缺省路由可访问全网
- OSPF 邻居关系均为 Full 状态

## 八、实验总结

本实验通过 OSPF 多区域配置，实现了骨干区与非骨干区的互联互通。在广播网络中采用 DR / BDR 机制提升效率，并在边缘路由器 AR5 上使用缺省路由代替 OSPF 路由宣告，有效降低了路由复杂度。实验结果表明，网络结构清晰，路由稳定，达到了预期实验目标。