# 硬件加速与功能卸载

## 网卡硬件卸载功能

各种网卡支持的硬件卸载的功能:

![NIC Hardware uninstall](images/nic-hardware-uninstall.jpg)

`DPDK`提供了硬件卸载的接口，利用`rte_mbuf`数据结构里的64位的标识（`ol_flags`）来表征卸载与状态

### 接收时：

![ol-flags send](images/ol-flags-send.jpg)

### 发送时：

![ol-flags accept](images/ol-flags-accept.jpg)

## VLAN硬件卸载

如果由软件完成`VLAN Tag`的插入将会给`CPU`带来额外的负荷，涉及一次额外的内存拷贝（报文内容复制），最坏场景下，这可能是上百周期的开销。大多数网卡硬件提供了`VLAN`卸载的功能。

### 接收侧针对VLAN进行包过滤

网卡最典型的卸载功能之一就是在接收侧针对`VLAN`进行包过滤，在`DPDK`中`app/testpmd`提供了测试命令与实现代码

![testpmd command](images/testpmd-command.jpg)

`DPDK`的`app/testpmd`提供了如何基于端口使能与去使能的测试命令。

```
testpmd> vlan set strip (on|off) (port_id)
testpmd> vlan set stripq (on|off) (port_id,queue_id)
```

### 发包时VLAN Tag的插入

在`DPDK`中，在调用发送函数前，必须提前设置`mbuf`数据结构，设置`PKT_TX_VLAN_PKT`位，同时将具体的`Tag`信息写入`vlan_tci`字段。

### 多层VLAN

现代网卡硬件大多提供对两层`VLAN Tag`进行卸载，如`VLAN Tag`的剥离、插入。`DPDK`的`app/testapp`应用中提供了测试命令。网卡数据手册有时也称`VLAN Extend`模式。

## IEEE588协议

`DPDK`提供的是打时间戳和获取时间戳的硬件卸载。需要注意，`DPDK`的使用者还是需要自己去管理`IEEE1588`的协议栈，`DPDK`并没有实现协议栈。

## IP TCP/UDP/SCTP checksum硬件卸载功能

`checksum`在收发两个方向上都需要支持，操作并不一致，在接收方向上，主要是检测，通过设置端口配置，强制对所有达到的数据报文进行检测，即判断哪些包的`checksum`是错误的，对于这些出错的包，可以选择将其丢弃，并在统计数据中体现出来。在`DPDK`中，和每个数据包都有直接关联的是`rte_mbuf`，网卡自动检测进来的数据包，如果发现`checksum`错误，就会设置错误标志。软件驱动会查询硬件标志状态，通过`mbuf`中的`ol_flags`字段来通知上层应用。

## Tunnel硬件卸载功能

目前`DPDK`仅支持对`VxLAN`和`NVGRE`的流进行重定向：基于`VxLAN`和`NVGRE`的特定信息，`TNI`或`VNI`，以及内层的`MAC`或`IP`地址进行重定向。

在`dpdk/testpmd`中，可以使用相关的命令行来使用`VxLAN`和`NVGRE`的数据流重定向功能，如下所示：

```sh
flow_director_filter X mode Tunnel add/del/update mac XX:XX:XX:XX:XX:XX vlan XXXX tunnel NVGRE/VxLAN tunnel-id XXXX flexbytes (X,X) fwd/drop queue X fd_id X
```

## TSO

![TSO](images/tso.jpeg)

> 图片来源[TCP Segment Offload for DPDK vRouter](https://github.com/Juniper/contrail-controller/wiki/TCP-Segment-Offload-for-DPDK-vRouter)

`TSO`（`TCP Segment Offload`）是`TCP`分片功能的硬件卸载，显然这是发送方向的功能。硬件提供的`TCP`分片硬件卸载功能可以大幅减轻软件对`TCP`分片的负担。

![TSO structure](images/tso-structure.jpg)

在`dpdk/testpmd`中提供了两条`TSO`相关的命令行：

1. `tso set 14000`：用于设置`tso`分片大小。
1. `tso show 0`：用于查看`tso`分片的大小。

## RSC

![RSC and LRO](images/rsc-lro.jpg)

> 图片来源[Software segmentation offloading for FreeBSD by Stefano Garzarella](https://www.slideshare.net/eurobsdcon/20140928-gso-eurobsdcon2014)

`RSC`（`Receive Side Coalescing`，接收方聚合）是`TCP`组包功能的硬件卸载。硬件组包功能实际上是硬件拆包功能的逆向功能。硬件组包功能针对`TCP`实现，是接收方向的功能，可以将拆分的`TCP`分片聚合成一个大的分片，从而减轻软件的处理。

![RSC structure](images/rsc-structure.jpg)

![RSC process](images/rsc-process.jpg)

## 参考

- [TCP Segment Offload for DPDK vRouter](https://github.com/Juniper/contrail-controller/wiki/TCP-Segment-Offload-for-DPDK-vRouter)
- [Software segmentation offloading for FreeBSD by Stefano Garzarella](https://www.slideshare.net/eurobsdcon/20140928-gso-eurobsdcon2014)
