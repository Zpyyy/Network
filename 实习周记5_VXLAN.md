【VXLAN基本概念】

VXLAN（Virtual eXtensible Local area Network）是一种在三层网络的基础上构建虚拟化二层网络的技术，是目前overlay技术中最常用的技术

为什么需要VXLAN？

（1）vlan的数量限制，4096个vlan远不能满足大规模云计算数据中心的需求

（2）为了使数据中心的计算资源得到灵活调配，需要支持VM（虚拟机）跨分区甚至跨数据中心的灵活迁移，VLAN很难保证虚拟机在迁移后的IP和MAC地址不会发生改变。

VTEP（VXLAN Tunnel Endpoints，VXLAN隧道端点）

VXLAN网络的边缘设备，是VXLAN隧道的起点和终点，VXLAN通过将逻辑网络中通信的数据帧封装在物理网络中进行传输，封装和解封装的过程由VTEP节点完成。VTEP既可以是一台独立的网络设备（比如华为的CE系列交换机），也可以是虚拟机所在的服务器。

VNI（VXLAN Network Identifier，VXLAN 网络标识符）

VNI是一种类似于VLAN ID的用户标示，一个VNI代表了一个租户，属于不同VNI的虚拟机之间不能直接进行二层通信。VXLAN报文封装时，给VNI分配了足够的空间使其可以支持海量租户的隔离

【VXLAN报文】

VXLAN使用MAC IN UDP的封装方式，将虚拟机发出的原始以太报文完整的封装在VXLAN信息中，在现有承载网络中进行透明传输，到达目的地之后通过解封装VXLAN信息将原始二层报文转发给目标虚拟机，从而实现了虚拟机之间的相互通信，这样虚拟机彻底摆脱了二三层网络范围的限制，可以跨设备、跨分区甚至跨数据中心的自由迁移。

下图为VXLAN信息报文：

VNI标示该报文属于哪个VXLAN隧道，UDP端口号4789用于标示该报文为VXLAN报文。

【建立VXLAN隧道】

为了将本端虚拟机发出的原始报文通过VXLAN传递给远端虚拟机，需要指定VTEP，并在两个VTEP之间建立VXLAN隧道，VXLAN隧道可以通过静态和动态两种方式进行建立。

静态方式：用户手动指定VXLAN的源和目的IP地址分别作为本端和对端的VTEP的IP地址，当VTEP增加时需要手工建立所有的新增VXLAN隧道。

动态方式：在VTEP之间建立BGP EVPN邻居关系，通过BGP EVPN协议自动在本端和对端建立VXLAN隧道，通过在网络中部署BGP的路由反射器RR，可以实现新增VTEP时，VXLAN隧道的自动建立。

【进入VXLAN隧道】

VXLAN隧道建立后，如何确定网络中的每个报文是否需要进入VXLAN隧道以及具体属于哪个隧道？

（1）    二层子接口（VTEP上的一种逻辑接口）接入VXLAN隧道：配置不同的流封装类型并绑定VNI对应的广播域BD信息。

（2）    VLAN接入VLAN隧道：在VTEP的物理接口下配置允许携带该报文的VLAN通过，然后在VLAN下绑定VNI对应的BD信息。

【VXLAN应用场景】

在典型的spine-leaf数据中心组网中，企业中有多个部门，每个部门拥有多个VM，同部门的VM属于同一个网段，不同部门的VM属于不同网段，此时如何部署VXLAN网络来来实现

（1）    同一部门VM之间的通信？

将leaf1和leaf2作为VTEP，在两个leaf之间搭建VXLAN隧道，在每个leaf上部署VXLAN二层网关;

（2）    不同部门之间VM的通信？VM与internet之间的通信？

在（1）的基础上部署VXLAN三层网关：

<1>VXLAN集中式三层网关

Spine和leaf均作为VTEP，leaf用于流量的二层接入，spine承担三层网关功能，在leaf1和spine1之间，leaf2和spine2之间分别搭建VXLAN隧道，并在spine上部署VXLAN三层网关。

<2>VXLAN分布式三层网关：

Leaf节点作为VTEP，同一个leaf节点可以同时做VXLAN的二层网关和三层网关。在单个leaf上部署VXLAN三层网关，可实现该leaf下不同部门之间VM的通信，在不同leaf上部署VXLAN三层网关，两个VXLAN三层网关之间通过BGP动态建立VXLAN隧道，并通过BGP的remote next-hop属性，发布本网关下挂的主机路由信息到其他BGP邻居，可实现跨leaf节点不同部门之间的通信。
当网络中的设备越来越多网络中的设备越来越复杂时，为了方便对网络的各节点的分配与部署，可以通过控制器来部署VXLAN网络，控制器通过NETCONF协议控制设备间VXLAN隧道的建立，通过openflow协议控制报文在隧道中的转发。
