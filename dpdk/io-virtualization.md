# 网络虚拟化

`I/O`虚拟化包括管理虚拟设备和共享的物理硬件之间`I/O`请求的路由选择。目前，实现`I/O`虚拟化有三种方式：`I/O`全虚拟化、`I/O`半虚拟化和`I/O`透传。

![netword I/O virtualization](images/network-io-virtualization.jpg)

- 全虚拟化：宿主机截获客户机对`I/O`设备的访问请求，然后通过软件模拟真实的硬件。这种方式对客户机而言非常透明，无需考虑底层硬件的情况，不需要修改操作系统。
- 半虚拟化：通过前端驱动/后端驱动模拟实现`I/O`虚拟化。客户机中的驱动程序为前端，宿主机提供的与客户机通信的驱动程序为后端。前端驱动将客户机的请求通过与宿主机间的特殊通信机制发送给后端驱动，后端驱动在处理完请求后再发送给物理驱动。
- `IO`透传：直接把物理设备分配给虚拟机使用，这种方式需要硬件平台具备`I/O`透传技术，例如`Intel VT-d`技术。它能获得近乎本地的性能，并且`CPU`开销不高。

`DPDK`支持半虚拟化的前端`virtio`和后端`vhost`，并且对前后端都有性能加速的设计，这些将分别在后面两章介绍。而对于`I/O`透传，`DPDK`可以直接在客户机里使用，就像在宿主机里，直接接管物理设备，进行操作。

## I/O透传

`I/O`透传带来的好处是高性能，几乎可以获得本机的性能，这个主要是因为`Intel®VT-d`的技术支持，在执行`I/O`操作时大量减少甚至避免`VM-Exit`陷入到宿主机中。目前只有`PCI`和`PCI-e`设备支持`Intel®VT-d`技术。可以配合`SR-IOV`使用，让一个网卡生成多个独立的虚拟网卡，把这些虚拟网卡分配给每一个客户机，可以获得相对好的性能。

### VT-d

![VT-d architecture](images/VT-d-architecture.jpg)
> 图片来源[XPDS14 - Intel(r) Virtualization Technology for Directed I/O (VT-d) Posted Interrupts - Feng Wu, Intel](https://www.slideshare.net/xen_com_mgr/vt-d-posted-interruptsfinal)

`VT-d`主要给宿主机软件提供了以下的功能：

- `I/O`设备的分配：可以灵活地把`I/O`设备分配给虚拟机，把对虚拟机的保护和隔离的特性扩展到`I/O`的操作上来。
- `DMA`重映射：可以支持来自设备`DMA`的地址翻译转换。
- 中断重映射：可以支持来自设备或者外部中断控制器的中断的隔离和路由到对应的虚拟机。
- 可靠性：记录并报告`DMA`和中断的错误给系统软件，否则的话可能会破坏内存或影响虚拟机的隔离。

### SR-IOV

![SR-IOV](images/sr-iov.png)
> 图片来源[I/O Virtualization Overview: CNA, SR-IOV, VN-Tag and VEPA](https://infrastructureadventures.wordpress.com/2010/12/08/io-virtualization-overview-cna-sr-iov-vn-tag-and-vepa/)

`SR-IOV`技术是由`PCI-SIG`制定的一套硬件虚拟化规范，全称是`Single Root IO Virtualization`（单根IO虚拟化）。`SR-IOV`规范主要用于网卡（`NIC`）、磁盘阵列控制器（`RAID controller`）和光纤通道主机总线适配器（`Fibre Channel Host Bus Adapter，FC HBA`），使数据中心达到更高的效率。`SR-IOV`架构中，一个`I/O`设备支持最多`256`个虚拟功能，同时将每个功能的硬件成本降至最低。`SR-IOV`引入了两个功能类型：

- `PF`（`Physical Function`，物理功能）：这是支持`SR-IOV`扩展功能的`PCIe`功能，主要用于配置和管理`SR-IOV`，拥有所有的`PCIe`设备资源。`PF`在系统中不能被动态地创建和销毁（`PCI Hotplug`除外）。
- `VF`（`Virtual Function`，虚拟功能）：“精简”的`PCIe`功能，包括数据迁移必需的资源，以及经过谨慎精简的配置资源集，可以通过`PF`创建和销毁。

![SR-IOV architecture](images/sr-iov-architecture.jpg)

#### VMware SR-IOV Architecture

![VMware SR-IOV Architecture](images/VMWare-SR-IOV-Atchitecture.png)

> 图片来源[SR-IOV Component Architecture and Interaction](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.networking.doc/GUID-DD13D453-98B9-4D26-85EA-A738293AEE00.html)
>
## virtio

![High-level-architecture-of-the-virtio-framework](images/High-level-architecture-of-the-virtio-framework.gif)

> 图片来源[Virtio: An I/O virtualization framework for Linux](https://www.ibm.com/developerworks/library/l-virtio/index.html)

在客户机操作系统中实现的前端驱动程序一般直接叫`Virtio`，在宿主机实现的后端驱动程序目前常用的叫`vhost`。与宿主机纯软件模拟`I/O`（如`e1000`、`rtl8139`）设备相比，`virtio`可以获得很好的`I/O`性能。但其缺点是必须要客户机安装特定的`virtio`驱动使其知道是运行在虚拟化环境中。

![VNF Bridging with Virtio](images/VNF-Bridging-with-Virtio.png)
> 图片来源[Understanding Virtio Usage - Juniper](https://www.juniper.net/documentation/en_US/junos/topics/concept/disaggregated-junos-virtio.html)

### 常见的virtio设备

![virtio-devices](images/virtio-devices.jpg)

### Virtio网络设备Linux内核驱动设计

`Virtio`网络设备`Linux`内核驱动主要包括三个层次：底层`PCI-e`设备层，中间`Virtio`虚拟队列层，上层网络设备层。

![virtio-pcie-device-layer](images/virtio-pcie-device-layer.jpg)

![virtio-virtual-queue-layer](images/virtio-virtual-queue-layer.jpg)

![virtio-network-device-layer](images/virtio-network-device-layer.jpg)

### DPDK用户空间virtio设备的优化

`DPDK`用户空间驱动和`Linux`内核驱动相比，主要不同点在于`DPDK`只暂时实现了`Virtio`网卡设备，所以整个构架和优化上面可以暂时只考虑网卡设备的应用场景。

- 关于单帧`mbuf`的网络包收发优化：固定了可用环表表项与描述符表项的映射，即可用环表所有表项`head_idx`指向固定的`vring`描述符表位置（对于接收过程，可用环表0->描述符表0，`1->1，…，255->255`的固定映射；对于发送过程，`0->128，1->129，…127->255，128->128，129->129，…255->255`的固定映射，描述符表`0~127`指向`mbuf`的数据区域，描述符表`128~255`指向`virtio net header`的空间），对可用环表的更新只需要更新环表自身的指针。固定的可用环表除了能够避免不同核之间的`CACHE`迁移，也节省了`vring`描述符的分配和释放操作，并为使用`SIMD`指令进行进一步加速提供了便利。
- `Indirect`特性在网络包发送中的支持：如前面介绍，发送的包至少需要两个描述符。通过支持`indirect`特性，任何一个要发送的包，无论单帧还是巨型帧（相关的介绍见[PCIe](PCIe.md)）都只需要一个描述符，该描述符指向一块驱动程序额外分配的间接描述符表的内存区域。

![virtio-vring-desc-table](images/virtio-vring-desc-table.jpg)

## vhost

`virtio-net`的后端驱动经历过从`virtio-net`后端，到内核态`vhost-net`，再到用户态`vhost-user`的演进过程。

### virtio-net

`virtio-net`后端驱动的最基本要素是虚拟队列机制、消息通知机制和中断机制。虚拟队列机制连接着客户机和宿主机的数据交互。消息通知机制主要用于从客户机到宿主机的消息通知。中断机制主要用于从宿主机到客户机的中断请求和处理。

![qemu-virtio-net](images/qemu-virtio-net.png)

> 图片来源[Vhost Sample Application](http://dpdk.org/doc/guides-1.8/sample_app_ug/vhost.html#background)

性能瓶颈主要存在于数据通道和消息通知路径这两块：

1. 数据通道是从`Tap`设备到`Qemu`的报文拷贝和`Qemu`到客户机的报文拷贝，两次报文拷贝导致报文接收和发送上的性能瓶颈。
1. 消息通知路径是当报文到达`Tap`设备时内核发出并送到`Qemu`的通知消息，然后`Qemu`利用`IOCTL`向`KVM`请求中断，`KVM`发送中断到客户机。

### Linux内核态vhost-net

`vhost-net`通过卸载`virtio-net`在报文收发处理上的工作，使`Qemu`从`virtio-net`的虚拟队列工作中解放出来，减少上下文切换和数据包拷贝，进而提高报文收发的性能。除此以外，宿主机上的`vhost-net`模块还需要承担报文到达和发送消息通知及中断的工作。

![Virtio with Linux* Kernel Vhost](images/virtio-linux-vhost.png)

报文接收仍然包括数据通路和消息通知路径两个方面：

1. 数据通路是从`Tap`设备接收数据报文，通过`vhost-net`模块把该报文拷贝到虚拟队列中的数据区，从而使客户机接收报文。
1. 消息通路是当报文从`Tap`设备到达`vhost-net`时，通过`KVM`模块向客户机发送中断，通知客户机接收报文。

### 用户态vhost

`Linux`内核态的`vhost-net`模块需要在内核态完成报文拷贝和消息处理，这会给报文处理带来一定的性能损失，因此用户态的`vhost`应运而生。用户态`vhost`采用了共享内存技术，通过共享的虚拟队列来完成报文传输和控制，大大降低了`vhost`和`virtio-net`之间的数据传输成本。

1.  数据通路不再涉及内核，直接通过共享内存发送给用户态应用（如`DPDK`，`OVS`）
1.  消息通路通过`Unix Domain Socket`实现

![userspace-vhost](images/userspace-vhost.jpg)

![userspace-vhost-characteristics](images/userspace-vhost-characteristics.jpg)

`DPDK vhost`同时支持`Linux virtio-net`驱动和`DPDK virtio PMD`驱动的前端。

## DPDK vhost

`DPDK vhost`支持`vhost-cuse`（用户态字符设备）和`vhost-user`（用户态`socket`服务）两种消息机制，它负责为客户机中的`virtio-net`创建、管理和销毁`vhost`设备。

当使用`vhost-user`时，首先需要在系统中创建一个`Unix domain socket server`，用于处理`Qemu`发送给`vhost`的消息，其消息机制如图所示。

![](images/vhost-message-mechanism.jpg)

`DPDK`的`vhost`有两种封装形式：`vhost lib`和`vhost PMD`。`vhost lib`实现了用户态的`vhost`驱动供`vhost`应用程序调用，而`vhost PMD`则对`vhost lib`进行了封装，将其抽象成一个虚拟端口，可以使用标准端口的接口来进行管理和报文收发。

`DPDK`示例程序`vhost-switch`是基于`vhost lib`的一个用户态以太网交换机的实现，可以完成在`virtio-net`设备和物理网卡之间的报文交换。还使用了虚拟设备队列（`VMDQ`）技术来减少交换过程中的软件开销，该技术在网卡上实现了报文处理分类的任务，大大减轻了处理器的负担。

## 对比

![io-contrast-the-virtualization-scheme](images/io-contrast-the-virtualization-scheme.jpg)

![virtio-and-vhost-net-and-vhost-user](images/virtio-and-vhost-net-and-vhost-user.jpg)

## 参考

- [What is I/O Virtualization (IOV)?](https://www.petri.com/what-is-io-virtualization-iov)
- [IO virtualization](https://www.slideshare.net/HwanjuKim/5io-virtualization)
- [XPDS14 - Intel(r) Virtualization Technology for Directed I/O (VT-d) Posted Interrupts - Feng Wu, Intel](https://www.slideshare.net/xen_com_mgr/vt-d-posted-interruptsfinal)
- [Virtio: An I/O virtualization framework for Linux](https://www.ibm.com/developerworks/library/l-virtio/index.html)
- [Understanding Virtio Usage - Juniper](https://www.juniper.net/documentation/en_US/junos/topics/concept/disaggregated-junos-virtio.html)
- [Single Root I/O Virtualization (SR-IOV) – Part 1](https://www.teimouri.net/single-root-io-virtualization-sr-iov-part-1/)
- [I/O Virtualization Overview: CNA, SR-IOV, VN-Tag and VEPA](https://infrastructureadventures.wordpress.com/2010/12/08/io-virtualization-overview-cna-sr-iov-vn-tag-and-vepa/)
- [SR-IOV Component Architecture and Interaction](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.networking.doc/GUID-DD13D453-98B9-4D26-85EA-A738293AEE00.html)
- [Vhost Sample Application](http://dpdk.org/doc/guides-1.8/sample_app_ug/vhost.html)
- [kVM I/O虚拟化分析](http://blog.csdn.net/zgy666/article/details/78469142)
