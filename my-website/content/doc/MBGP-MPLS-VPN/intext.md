+++
title = 'MBGP MPLS VPN原理&配置'
date = 2025-01-16T18:34:16+08:00
draft = false
enableToc = true
+++

# MBGP MPLS VPN

### MBGP MPLS VPN的优势

MBGP MPLS VPN 作为一种新的VPN技术，解决了传统VPN的所有缺陷

1. 通过 MBGP MPLS VPN 搭建的隧道是动态建立的
2. 能够让运营商的网络设备识别相同的客户网络，解决本地地址冲突问题
3. 很好的控制VPN私网路由

### 隧道技术与MPLS

1. 在传统的 VPN 中（例如：GRE 隧道），为了能够实现私网IP 穿越公有互联网络，需要在原始的3层头部的外面再增加一个 4Byte 大小的 GRE 头部与 20Byte 大小的新的IP 头部，新的IP 头部中包含的是可在公网上路由的公有源IP与目的IP地址
2. 在 MPLS网络中，MPLS的头部封装在原始的3层头部与2层头部之间（MPLS 头部在外，原始的3层头部在内）
3. MPLS 的封装将原始的3层头部包裹在其中，形成了一条天然隧道，起到与 GRE 隧道一样的封装作用

### MPLS隧道应用

1. 在 MPLS 网络中，沿着LSP转发数据的LSR设备无需知道私网路由，根据 MPLS 头部中的标签进行数据转发
2. 只需要在 MPLS 网络中的 LER设备上作处理就能够实现私网路由穿越公网
3. 在非 MPLS 网络的客户上，客户与运营商的LER设备需要运行动态路由协议以便于将自身内部私网路由传递给运营商，运营商内部运行BGP

### MPLS倒数第二跳弹出(PHP)

在LDP分配标签的过程中，倒数第一的路由器为倒数第二的路由器分配的标签永远是3，因此倒数第二路由器在给倒数第一发送数据时，就可以直接将标签3弹出，令倒数第一路由器直接查找路由表将数据转发至目的客户网络即可

### MPLS VPN组网结构

在 MBGP MPLS VPN 中定义了3个特殊名词：

1. CE（Custom Edge Router）：非 MPLS网络的边界网络设备
2. PE （Provider Edge Router）：运营商的 MPLS 的边界网络设备，用来连接CE
3. P （Provider Router）：工作在MPLS网络内的基于标签转发的路由器

### VRF技术

若运营商同一台路由器连接不同的客户网络，不同客户网络之间使用相同私网网段，这将造成运营商设备（PE）的路由表混乱造成地址冲突问题

因此提出的解决方案为：VRF (virtual routing fowarding)

- VRF 相当于是让一台物理路由器分裂多台逻辑路由器
- 每台 VRF路由器负责维护自身单独的路由表项，各个VRF 路由器的路由表项完全独立，互不可见
- 每台 VRF 路由器拥有3个独立：
    1. 独立的IP 路由表项
    2. 独立的端口/接口
    3. 独立的路由协议
- 各个VRF与各自的用户网络之间运行一个路由实例，该路由实例学习到的路由只能加入该VPN的路由表
- 各个路由实例与所属的VPN进行绑定，它们之间互相独立，只能学习到各自的邻居信息

### MBGP协议

普通的BGPv4 仅仅只能够传递或删除IPv4 的路由，为了能够让BGP 承载多个协议的路由信息，RFC 对 BGP 进行扩展升级，升级后的BGP被称为【MBGP、MP-BGP I Multi- Protocol | 多协议】

扩展后的 BGP 新增了3种属性：

1. MP_REACH_NLRI： 多协议_可达性_网络层可达性信息
2. MP_UNREACH_NLRI： 多协议_非可达性网络层可达性信息
3. Extended_Communities（扩展的团体属性）
4. 升级后的 MBGP 就能够传递 MBGP MPLS VPN、L2VPN、6PE 等路由信息

### MBGP 的路由更新

扩展后的MBGP 相较于普通的BGPV4做了如下改动

1. 使用 MP_REACH_NLRI 替代了普通 BGP 中的 NLRI 与 Next Hop 属性
2. 使用MP_UNREACH_NLRI 替代了普通 BGP 中的 Withdrawn-routest
3. 在属性部分新增了 Extended_Communities

### MP_REACH_NLRI 属性

1. 普通 BGP 更新消息中包含：
    1. 下一跳属性
    2. NLRI: IPv4 地址与前缀长度
    3. 其它属性
2. MBGP 更新消息中包含：
    1. 下一跳属性
    2. NLRI:IPv4 地址与前缀长度
    3. 其它属性
    4. 地址族
    5. RD【路由区分器】

### MP_UNREACH_NLRI属性

1. 普通 BGP删除消息中包含：需要删除的路由的IPv4地址与前缀长度
2. MBGP 删除消息中包含：
    1. 需要删除的IPv4地址与前缀长度
    2. 地址族
    3. RD【路由区分器】

### RT (Route Target |路由标记)

- RT的本质是每个VPN实例表达自己的路由取舍及喜好的方式
- 在PE设备上，发送某一个VPN用户的私网路由给其BGP邻居时，需要在MBGP的扩展团体属性区域中增加该VPN的ExportTarget属性
- 在PE设备上，需要将收到的MBGP路由的扩展团体属性中所携带的RT属性值，与本地每一个VPN的Import Target属性值相比较，当这两个值相同时，就需要将这条路由添加到该VPN的路由表中去
- RT 的格式分力2类：
    - AS号码 + 分配编号：100:1
    - IP 地址 +分配编号：172.16.1.1:1

### RD 路由区分器Route Distinguisher

- 作用：用来在MPLS 网络中的私网路由撤销时使用的

### IPv4与VPNv4地址族

在 IPV4地址后面加上 RD 值，就变成了 VPNV4 地址族

原来的标准的地址族就称为 IPV4

### 私网标签

- RT属性和RD前缀顺利解决了私网路由的学习和撤销中存在的问题，然而因为VPN地址的冲突在数据转发过程也将遇到困难
- 需要在数据报文中增加一个标识，以帮助PE判断该报文是去往本地的哪个VPN
- 由于MPLS支持多层标签的嵌套，这个标识可以定义成MPLS标签的格式，即私网Label

### 实验操作

![https://zone.fitch.cloud/qXub1NAz2MD6.png](https://zone.fitch.cloud/qXub1NAz2MD6.png)

1. 首先配置MPLS主干网络
    1. 先用IGP协议建立设备互联，使得PE1设备可以ping通PE2
    2. PE1和PE2之间建立BGP连接
    3. 创建VPN示例
        
        ```bash
        [Huawei]ip vpn-instance vpn1
        [Huawei-vpn-instance-vpn1]route-distinguisher 100:1
        [Huawei-vpn-instance-vpn1]vpn-target 100:1 both
        [Huawei-vpn-instance-vpn1]q
        [Huawei]int g0/0/1
        ip binding vpn-instance vpn1 #绑定VPN实例
        #接口IP消失需要重新配置IP
        ```
        
    4. 在主干网的各个路由器使能mpls
        
        ```bash
        [Huawei]mpls lsr-id #填routerid
        [Huawei]mpls
        [Huawei-mpls]quit
        [Huawei]mpsl ldp
        [Huawei-mpls-ldp]quit
        [Huawei]int gx/x/x  #各路由器互联的接口
        [Huawei-GigabitEthernetx/x/x]mpls
        [Huawei-GigabitEthernetx/x/x]mpls ldp
        ```
        
    5. 在与Hub/Spoke路由器相连的PE路由器的接口使能mpls
        
        ```bash
        [Huawei]interface Gx/x/x/ #与连接Hub/Spoke路由器的接口
        [Huawei-GigabitEthernetx/x/x]mpls
        ```
        
2. 配置Hub/Spoke路由器
    1. 与PE建立连接即可
        
        ```bash
        [Huawei]ospf 2
        [Huawei-ospf-2]area 0
        [Huawei-ospf-2-area-0.0.0.0]network x.x.x.x x.x.x.x #宣告自己与PE相连网段
        [Huawei-ospf-2-area-0.0.0.0]network x.x.x.x x.x.x.x #宣告自己内部私网网段
        ```
        
3. 配置PE与Hub/Spoke之间的动态路由协议
    - 在PE上
        
        ```bash
        [Huawei]ospf 2 vpn-instance vpn1
        area 0
        network x.x.x.x x.x.x.x #宣告与Hub/Spoke路由器相连的网段
        qu
        ```
        
4. PE路由器配置路由双向注入
    - PE1中
        
        ```bash
        [Huawei]bpg 1
        ipv4-family vpnv4
        peer x.x.x.x enable #对等体routerid
        qu
        ipv4-family vpn-instance vpn1
        import-route ospf 2
        qu
        [Huawei]ospf 2 vpn-instance vpn1
        import-route bgp
        ```