# OVS DPDK

`OVS`在实现中分为用户空间和内核空间两个部分。用户空间拥有多个组件，它们主要负责实现数据交换和OpenFlow流表功能，还有一些工具用于虚拟交换机管理、数据库搭建以及和内核组件的交互。内核组件主要负责流表查找的快速通道。`OVS`的核心组件及其关联关系如图

![ovs internal](../ovs/images/ovs-internals.jpg)

下图显示了`OVS`数据通路的内部模块图

![ovs software architecture](images/ovs-software-architecture.jpg)


- `ovs-vswitchd`主要包含`ofproto`、`dpif`、`netdev`模块
    - `ofproto`模块实现`openflow`的交换机
    - `dpif`模块抽象一个单转发路径
    - `netdev`模块抽象网络接口（无论物理的还是虚拟的）
- `openvswitch.ko`主要由数据通路模块组成，里面包含着流表。流表中的每个表项由一些匹配字段和要做的动作组成。

`DPDK`加速的思想就是专注在这个数据通路上

![ovs with dpdk](images/ovs-with-dpdk.jpg)


## ovs with dpdk

![ovs with dpdk architecture](images/ovs-with-dpdk-architecture.jpg)

![ovs architecture process](images/ovs-architecture-process.jpg)

- `dpif-netdev`：用户态的快速通路，实现了基于`netdev`设备的`dpif API`。
- `Ofproto-dpif`：实现了基于`dpif`层的`ofproto API`。
- `netdev-dpdk`：实现了基于`DPDK`的`netdev API`，其定义的几种网络接口如下：
- `dpdk`物理网口：其实现是采用高性能向量化`DPDK PMD`的驱动。
- `dpdkvhostuser`与`dpdkvhostcuse`接口：支持两种`DPDK`实现的`vhost`优化接口：`vhost-user`和`vhost-cuse`。`vhost-user`或`vhost-cuse`可以挂接到用户态的数据通道上，与虚拟机的`virtio`网口快速通信。如第12章所说，`vhost-cuse`是一个过渡性技术，`vhost-user`是建议使用的接口。为了性能，在`vhost burst`收发包个数上，需要和`dpdk`物理网口设置的`burst`收发包个数相同。
- `dpdkr`：其实现是基于`DPDK librte_ring`机制创建的`DPDK ring`接口。`dpdkr`接口挂接到用户态的数据通道上，与使用了`IVSHMEM`的虚拟机合作可以通过零拷贝技术实现高速通信。

`DPDK`加速的`OVS`数据流转发的大致流程如下：

1. `OVS`的`ovs-vswitchd`接收到从`OVS`连接的某个网络端口发来的数据包，从数据包中提取源/目的`IP`、源/目的`MAC`、端口等信息。
1. `OVS`在用户态查看精确流表和模糊流表，如果命中，则直接转发。
1. 如果还不命中，在SDN控制器接入的情况下，经过`OpenFlow`协议，通告给控制器，由控制器处理。
1. 控制器下发新的流表，该数据包重新发起选路，匹配；报文转发，结束。

`DPDK`加速的`OVS`与原始`OVS`的区别在于，从`OVS`连接的某个网络端口接收到的报文不需要`openvswitch.ko`内核态的处理，报文通过`DPDK PMD`驱动直接到达用户态`ovs-vswitchd`里。

## Packet Flow

![ovs packet flow](images/ovs-packet-flow.jpg)



## 网络存储优化

![network-storage-optimization](images/network-storage-optimization.jpg)
