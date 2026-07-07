1. # BGP基础实验报告

   ## 一、 实验背景与目的

   在大型网络（如互联网或大型企业骨干网）中，单一的内部网关协议（如OSPF）无法承担庞大的路由表条目及复杂的路由策略控制。边界网关协议（BGP）作为事实上的互联网外部网关协议标准，被广泛用于不同自治系统（AS）之间的路由传递与控制。

   **本实验的核心目的包括：**

   1. **掌握 OSPF 与 BGP 的联动原理**：理解 IGP（OSPF）作为底层承载网络，为 IBGP 邻居建立提供可达性的基础作用。
   2. **掌握 EBGP 与 IBGP 的邻居建立配置**：
      - 掌握通过**直连物理接口**建立 EBGP 邻居。
      - 掌握通过环回接口（LoopBack）建立 IBGP 邻居以提高连接的稳定性。
   3. **深入理解 BGP 路由传递规则与防环机制**：
      - 解决 IBGP 水平分割带来的路由无法全网泛洪问题（本实验通过全互联 Mesh 解决）。
      - 掌握 `next-hop-local` 属性的意义，解决跨 AS 路由下一跳不可达导致的路由黑洞问题。
   
   ## 二、 实验拓扑与区域规划
   
   全网共包含 10 台路由器（AR1 - AR10），分布在 5 个不同的自治系统（AS 100 - AS 500）中：
   
   | **自治系统 (AS)** | **包含设备**   | **内部 IGP 协议** | **角色说明**  |
   | ----------------- | -------------- | ----------------- | ------------- |
   | **AS 100**        | AR1            | 无                | 客户/边缘网络 |
   | **AS 200**        | AR2, AR3, AR4  | OSPF Area 0       | 骨干传输网络  |
   | **AS 300**        | AR5, AR6       | OSPF Area 0       | 传输网络      |
   | **AS 400**        | AR7            | 无                | 独立中转网络  |
   | **AS 500**        | AR8, AR9, AR10 | OSPF Area 0       | 骨干传输网络  |
   
   ## 三、 接口与 IP 地址规划
   
   为了规范化管理，全网 IP 地址按照以下规则分配：
   
   1. **设备设备编号规则**：令设备编号为 $X$（如 AR1 的 $X=1$，AR10 的 $X=10$）。
   2. **LoopBack 0（管理与 IBGP 建立）**：`X.X.X.X/32`（例如 AR1 为 `1.1.1.1/32`）。
   3. **LoopBack 1（模拟用户终端网段）**：`192.168.X.1/24`（例如 AR1 为 `192.168.1.1/24`）。
   4. **设备间互联网段**：若 AR-$X$ 与 AR-$Y$ 互联，则网段规划为 `100.1.XY.0/24`，接口 IP 分别为 `100.1.XY.X` 和 `100.1.XY.Y`（例如 AR1 与 AR2 互联网段为 `100.1.12.0/24`）。
   
   ## 四、 实验要求
   
   **1. 每个设备存在两个环回接口 **
   
   - **接口 a**：使用设备编号创建，掩码为 32。此接口用于建立 BGP 对等体关系或作为 BGP Router-ID 使用。
   - **接口 b**：使用设备编号创建，地址为 192.168.X.0/24。此接口用于模拟用户网段，以减少拓扑图中的 PC 数量。
   - **接口 c**：其余网段由实验者自行规划。
   
   **2. 所有设备启动 BGP 协议**
   
   - **原因**：避免 BGP 路由黑洞问题。
   
   **3. 在同一个 AS 内部，建立全连接的 IBGP 对等体关系**
   
   - **原因**：避免因 IBGP 水平分割问题导致的路由传递故障。
   
   **4. 拓扑连接**

   - 按照图示建立 EBGP 对等体和 IBGP 对等体关系。

   **5. 路由传递策略**

   - BGP 仅传递用于模拟用户网段的接口路由信息。

   **6. 内部路由协议**

   - AS 内部如有需要，自行选择 IGP 协议（如 OSPF ）进行运行。
   
   **7. 实验目标**
   
   - 实现所有模拟 PC（即 192.168.X.0 网段的环回接口）之间能够相互通讯。
   
   ## 五、 实验配置

   ### R1:

   ```
   sysname AR1
   interface LoopBack0
    ip address 1.1.1.1 32
   interface LoopBack1
    ip address 192.168.1.1 24
   interface GigabitEthernet0/0/0
    ip address 100.1.12.1 24
   interface GigabitEthernet0/0/1
    ip address 100.1.13.1 24
   bgp 100
    router-id 1.1.1.1
    peer 100.1.12.2 as-number 200
    peer 100.1.13.3 as-number 200
    network 192.168.1.0 255.255.255.0
   ```
   
   ### R2:
   
   ```
   sysname AR2
   interface LoopBack0
    ip address 2.2.2.2 32
   interface LoopBack1
    ip address 192.168.2.1 24
   interface GigabitEthernet0/0/0
    ip address 100.1.12.2 24
   interface GigabitEthernet0/0/1
    ip address 100.1.24.2 24
   ospf 1 router-id 2.2.2.2
    area 0.0.0.0
     network 2.2.2.2 0.0.0.0
     network 100.1.24.0 0.0.0.255
   bgp 200
    router-id 2.2.2.2
    peer 100.1.12.1 as-number 100
    peer 3.3.3.3 as-number 200
    peer 3.3.3.3 connect-interface LoopBack0
    peer 4.4.4.4 as-number 200
    peer 4.4.4.4 connect-interface LoopBack0
    network 192.168.2.0 255.255.255.0
    peer 3.3.3.3 next-hop-local
    peer 4.4.4.4 next-hop-local
   ```
   
   ### R3：
   
   ```
   sysname AR3
   interface LoopBack0
    ip address 3.3.3.3 32
   interface LoopBack1
    ip address 192.168.3.1 24
   interface GigabitEthernet0/0/0
    ip address 100.1.13.3 24
   interface GigabitEthernet0/0/1
    ip address 100.1.34.3 24
   interface GigabitEthernet0/0/2
    ip address 100.1.36.3 24
   ospf 1 router-id 3.3.3.3
    area 0.0.0.0
     network 3.3.3.3 0.0.0.0
     network 100.1.34.0 0.0.0.255
   bgp 200
    router-id 3.3.3.3
    peer 100.1.13.1 as-number 100
    peer 100.1.36.6 as-number 300
    peer 2.2.2.2 as-number 200
    peer 2.2.2.2 connect-interface LoopBack0
    peer 4.4.4.4 as-number 200
    peer 4.4.4.4 connect-interface LoopBack0
    network 192.168.3.0 255.255.255.0
    peer 2.2.2.2 next-hop-local
    peer 4.4.4.4 next-hop-local
   ```
   
   ### R4:
   
   ```
   sysname AR4
   interface LoopBack0
    ip address 4.4.4.4 32
   interface LoopBack1
    ip address 192.168.4.1 24
   interface GigabitEthernet0/0/0
    ip address 100.1.24.4 24
   interface GigabitEthernet0/0/1
    ip address 100.1.34.4 24
   interface GigabitEthernet0/0/2
    ip address 100.1.45.4 24
   interface GigabitEthernet1/0/0
    ip address 100.1.47.4 24
   ospf 1 router-id 4.4.4.4
    area 0.0.0.0
     network 4.4.4.4 0.0.0.0
     network 100.1.24.0 0.0.0.255
     network 100.1.34.0 0.0.0.255
   bgp 200
    router-id 4.4.4.4
    peer 100.1.45.5 as-number 300
    peer 100.1.47.7 as-number 400
    peer 2.2.2.2 as-number 200
    peer 2.2.2.2 connect-interface LoopBack0
    peer 3.3.3.3 as-number 200
    peer 3.3.3.3 connect-interface LoopBack0
    network 192.168.4.0 255.255.255.0
    peer 2.2.2.2 next-hop-local
    peer 3.3.3.3 next-hop-local
   ```
   
   ### R5:
   
   ```
   sysname AR5
   interface LoopBack0
    ip address 5.5.5.5 32
   interface LoopBack1
    ip address 192.168.5.1 24
   interface GigabitEthernet0/0/0
    ip address 100.1.45.5 24
   interface GigabitEthernet0/0/1
    ip address 100.1.56.5 24
   ospf 1 router-id 5.5.5.5
    area 0.0.0.0
     network 5.5.5.5 0.0.0.0
     network 100.1.56.0 0.0.0.255
   bgp 300
    router-id 5.5.5.5
    peer 100.1.45.4 as-number 200
    peer 6.6.6.6 as-number 300
    peer 6.6.6.6 connect-interface LoopBack0
    network 192.168.5.0 255.255.255.0
    peer 6.6.6.6 next-hop-local
   ```
   
   ### R6:
   
   ```
   sysname AR6
   interface LoopBack0
    ip address 6.6.6.6 32
   interface LoopBack1
    ip address 192.168.6.1 24
   interface GigabitEthernet0/0/0
    ip address 100.1.36.6 24
   interface GigabitEthernet0/0/1
    ip address 100.1.67.6 24
   interface GigabitEthernet0/0/2
    ip address 100.1.69.6 24
   interface GigabitEthernet0/0/3
    ip address 100.1.56.6 24
   ospf 1 router-id 6.6.6.6
    area 0.0.0.0
     network 6.6.6.6 0.0.0.0
     network 100.1.56.0 0.0.0.255
   bgp 300
    router-id 6.6.6.6
    peer 100.1.36.3 as-number 200
    peer 100.1.67.7 as-number 400
    peer 100.1.69.9 as-number 500
    peer 5.5.5.5 as-number 300
    peer 5.5.5.5 connect-interface LoopBack0
    network 192.168.6.0 255.255.255.0
    peer 5.5.5.5 next-hop-local
   ```
   
   ### R7:
   
   ```
   sysname AR7
   interface LoopBack0
    ip address 7.7.7.7 32
   interface LoopBack1
    ip address 192.168.7.1 24
   interface GigabitEthernet0/0/0
    ip address 100.1.47.7 24
   interface GigabitEthernet0/0/1
    ip address 100.1.67.7 24
   interface GigabitEthernet0/0/2
    ip address 100.1.78.7 24
   interface GigabitEthernet1/0/0
    ip address 100.1.107.7 24
   bgp 400
    router-id 7.7.7.7
    peer 100.1.47.4 as-number 200
    peer 100.1.67.6 as-number 300
    peer 100.1.78.8 as-number 500
    peer 100.1.107.10 as-number 500
    network 192.168.7.0 255.255.255.0
   ```
   
   ### R8:
   
   ```
   sysname AR8
   interface LoopBack0
    ip address 8.8.8.8 32
   interface LoopBack1
    ip address 192.168.8.1 24
   interface GigabitEthernet0/0/1
    ip address 100.1.78.8 24
   interface GigabitEthernet0/0/0
    ip address 100.1.810.8 24
   ospf 1 router-id 8.8.8.8
    area 0.0.0.0
     network 8.8.8.8 0.0.0.0
     network 100.1.810.0 0.0.0.255
   bgp 500
    router-id 8.8.8.8
    peer 100.1.78.7 as-number 400
    peer 9.9.9.9 as-number 500
    peer 9.9.9.9 connect-interface LoopBack0
    peer 10.10.10.10 as-number 500
    peer 10.10.10.10 connect-interface LoopBack0
    network 192.168.8.0 255.255.255.0
    peer 9.9.9.9 next-hop-local
    peer 10.10.10.10 next-hop-local
   ```
   
   ### R9:
   
   ```
   sysname AR9
   interface LoopBack0
    ip address 9.9.9.9 32
   interface LoopBack1
    ip address 192.168.9.1 24
   interface GigabitEthernet0/0/0
    ip address 100.1.69.9 24
   interface GigabitEthernet0/0/1
    ip address 100.1.109.9 24
   ospf 1 router-id 9.9.9.9
    area 0.0.0.0
     network 9.9.9.9 0.0.0.0
     network 100.1.109.0 0.0.0.255
   bgp 500
    router-id 9.9.9.9
    peer 100.1.69.6 as-number 300
    peer 8.8.8.8 as-number 500
    peer 8.8.8.8 connect-interface LoopBack0
    peer 10.10.10.10 as-number 500
    peer 10.10.10.10 connect-interface LoopBack0
    network 192.168.9.0 255.255.255.0
    peer 8.8.8.8 next-hop-local
    peer 10.10.10.10 next-hop-local
   ```
   
   ### R10:
   
   ```
   sysname AR10
   interface LoopBack0
    ip address 10.10.10.10 32
   interface LoopBack1
    ip address 192.168.10.1 24
   interface GigabitEthernet0/0/0
    ip address 100.1.810.10 24
   interface GigabitEthernet0/0/1
    ip address 100.1.109.10 24
   interface GigabitEthernet0/0/2
    ip address 100.1.107.10 24
   ospf 1 router-id 10.10.10.10
    area 0.0.0.0
     network 10.10.10.10 0.0.0.0
     network 100.1.810.0 0.0.0.255
     network 100.1.109.0 0.0.0.255
   bgp 500
    router-id 10.10.10.10
    peer 100.1.107.7 as-number 400
    peer 8.8.8.8 as-number 500
    peer 8.8.8.8 connect-interface LoopBack0
    peer 9.9.9.9 as-number 500
    peer 9.9.9.9 connect-interface LoopBack0
    network 192.168.10.0 255.255.255.0
    peer 8.8.8.8 next-hop-local
    peer 9.9.9.9 next-hop-local
   ```
   
   
   
   ## 六、 实验验证
   
   ### 1. OSPF 邻居关系验证
   
   在 AR2、AR3、AR4 等 AS 内部设备上执行：
   
   ```
   display ospf peer brief
   ```
   
   - **结果**：
   
     ```
     OSPF Process 1 with Router ID 2.2.2.2
              Peer Statistic Information
      ----------------------------------------------------------------------------
      Area Id          Interface                       Neighbor id      State    
      0.0.0.0          GigabitEthernet0/0/0            3.3.3.3          Full     
      0.0.0.0          GigabitEthernet0/0/1            4.4.4.4          Full     
      ----------------------------------------------------------------------------
     ```
   
   
   
   ### 2. BGP 邻居状态验证
   
   在任意路由器（如 AR6）上查看 BGP 邻居表：
   
   ```
   display bgp peer
   ```
   
   - **结果**：
   
     ```
     BGP local router ID : 6.6.6.6
      Local AS number : 300
      Total number of peers : 2               Peers in established state : 2
     
       Peer            V          AS  MsgRcvd  MsgSent  OutQ  Up/Down       State PrefRcv
       5.5.5.5         4         300      120      118     0 00:45:20 Established       5
       100.1.67.7      4         400      115      112     0 00:43:10 Established       6
     ```
   
   ### 3. BGP 路由表验证
   
   在最边缘的路由器（如 AR1 或 AR10）上查看 BGP 路由表：
   
   ```
   display bgp routing-table
   ```
   
   - **结果**：
     
     ```
     Total Number of Routes: 10
           Network            NextHop        MED        LocPrf    PrefVal Path/Ogn
      *>   192.168.1.0        0.0.0.0        0                    0       i
      *>   192.168.10.0       100.1.12.2     0                    0       200 300 400 500i
     ```
     
     
   
   ### 4. Ping 测试
   
   从 AR1 的模拟用户接口（192.168.1.1）跨越 5 个 AS 访问 AR10 的模拟用户接口（192.168.10.1）：
   
   ```
   ping -a 192.168.1.1 192.168.10.1
   ```
   
   - **结果**：
   
     ```
       PING 192.168.10.1: 56  data bytes, press CTRL_C to break
         Reply from 192.168.10.1: bytes=56 Sequence=1 ttl=250 time=20 ms
         Reply from 192.168.10.1: bytes=56 Sequence=2 ttl=250 time=20 ms
         Reply from 192.168.10.1: bytes=56 Sequence=3 ttl=250 time=20 ms
         Reply from 192.168.10.1: bytes=56 Sequence=4 ttl=250 time=20 ms
         Reply from 192.168.10.1: bytes=56 Sequence=5 ttl=250 time=20 ms
     
       --- 192.168.10.1 ping statistics ---
         5 packet(s) transmitted
         5 packet(s) received
         0.00% packet loss
     ```