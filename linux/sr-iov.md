# SR-IOV

`SR-IOV`（`Single Root I/O Virtualization`）是一个将`PCIe`共享给虚拟机的标准，通过为虚拟机提供独立的内存空间、中断、`DMA`流，来绕过`VMM`实现数据访问。SR-IOV基于两种`PCIe functions`：

- `PF` (`Physical Function`)： 包含完整的`PCIe`功能，包括`SR-IOV`的扩张能力，该功能用于`SR-IOV`的配置和管理。
- `VF` (`Virtual Function`)： 包含轻量级的`PCIe`功能。每一个`VF`有它自己独享的`PCI`配置区域，并且可能与其他`VF`共享着同一个物理资源

![SR-IOV Architecture](images/sriovarchitecture.png)
![SR-IOV synthetic datapaths](images/sriovsynthetic-datapaths.png)

> 图片来源[Microsoft - SR-IOV Architecture](https://docs.microsoft.com/en-us/windows-hardware/drivers/network/sr-iov-architecture)

## SR-IOV要求

- `CPU` 必须支持`IOMMU`（比如英特尔的`VT-d` 或者`AMD`的 `AMD-Vi`，`Power8` 处理器默认支持`IOMMU`）
- 固件`Firmware` 必须支持`IOMMU`
- `CPU` 根桥必须支持 `ACS` 或者`ACS`等价特性
- `PCIe` 设备必须支持`ACS` 或者`ACS`等价特性
- 建议根桥和`PCIe` 设备中间的所有`PCIe` 交换设备都支持ACS，如果某个`PCIe`交换设备不支持`ACS`，其后的所有`PCIe`设备只能共享某个`IOMMU` 组，所以只能分配给1台虚机。

## SR-IOV vs PCI path-through

### 架构上的比较（以网卡为例）

![kvm-performance-optimization-for-ubuntu](images/kvm-performance-optimization-for-ubuntu-17-638.jpg)

![kvm-performance-optimization-for-ubuntu](images/kvm-performance-optimization-for-ubuntu-18-638.jpg)

### Virtio 和 Pass-Through 的详细比较

![Virtio 和 Pass-Through 的详细比较](images/virtio-vs-pass-through.jpg)

> 图片来源[slideshare - Kvm performance optimization for ubuntu](https://www.slideshare.net/janghoonsim/kvm-performance-optimization-for-ubuntu)、[KVM 介绍（4）：I/O 设备直接分配和 SR-IOV [KVM PCI/PCIe Pass-Through SR-IOV]](http://www.cnblogs.com/sammyliu/p/4548194.html)

## SR-IOV vs DPDK

![sdn-fundamentals-for-nfv-openstack-and-containers-red-hat-summit](images/sdn-fundamentals-for-nfv-openstack-and-containers-red-hat-summit-2016-47-638.jpg)

![sdn-fundamentals-for-nfv-openstack-and-containers-red-hat-summit](images/sdn-fundamentals-for-nfv-openstack-and-containers-red-hat-summit-2016-48-638.jpg)

![sdn-fundamentals-for-nfv-openstack-and-containers-red-hat-summit](images/sdn-fundamentals-for-nfv-openstack-and-containers-red-hat-summit-2016-49-638.jpg)

## SR-IOV使用示例

开启`VF`：

```bash
modprobe -r igb
modprobe igb max_vfs=7
echo "options igb max_vfs=7" >>/etc/modprobe.d/igb.conf
```

查找`Virtual Function`：

```bash
# lspci | grep 82576
0b:00.0 Ethernet controller: Intel Corporation 82576 Gigabit Network Connection (rev 01)
0b:00.1 Ethernet controller: Intel Corporation 82576 Gigabit Network Connection(rev 01)
0b:10.0 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:10.1 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:10.2 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:10.3 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:10.4 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:10.5 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:10.6 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:10.7 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:11.0 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:11.1 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:11.2 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:11.3 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:11.4 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)
0b:11.5 Ethernet controller: Intel Corporation 82576 Virtual Function (rev 01)

# virsh nodedev-list | grep 0b
pci_0000_0b_00_0
pci_0000_0b_00_1
pci_0000_0b_10_0
pci_0000_0b_10_1
pci_0000_0b_10_2
pci_0000_0b_10_3
pci_0000_0b_10_4
pci_0000_0b_10_5
pci_0000_0b_10_6
pci_0000_0b_11_7
pci_0000_0b_11_1
pci_0000_0b_11_2
pci_0000_0b_11_3
pci_0000_0b_11_4
pci_0000_0b_11_5
```

```bash
$ virsh nodedev-dumpxml pci_0000_0b_00_0
<device>
   <name>pci_0000_0b_00_0</name>
   <parent>pci_0000_00_01_0</parent>
   <driver>
      <name>igb</name>
   </driver>
   <capability type='pci'>
      <domain>0</domain>
      <bus>11</bus>
      <slot>0</slot>
      <function>0</function>
      <product id='0x10c9'>82576 Gigabit Network Connection</product>
      <vendor id='0x8086'>Intel Corporation</vendor>
   </capability>
</device>
```

### 通过libvirt绑定到虚拟机

```bash
$ cat >/tmp/interface.xml <<EOF
<interface type='hostdev' managed='yes'>
     <source>
       <address type='pci' domain='0' bus='11' slot='16' function='0'/>
     </source>
</interface>
EOF
$ virsh attach-device MyGuest /tmp/interface. xml --live --config
```

当然也可以给网卡配置`MAC`地址和`VLAN`：

```xml
<interface type='hostdev' managed='yes'>
   <source>
      <address type='pci' domain='0' bus='11' slot='16' function='0'/>
   </source>
   <mac address='52:54:00:6d:90:02'>
   <vlan>
      <tag id='42'/>
   </vlan>
   <virtualport type='802.1Qbh'>
      <parameters profileid='finance'/>
   </virtualport>
</interface>
```

### 通过Qemu绑定到虚拟机

```bash
/usr/bin/qemu-kvm -name vdisk -enable-kvm -m 512 -smp 2 \
-hda /mnt/nfs/vdisk.img \
-monitor stdio \
-vnc 0.0.0.0:0 \
-device pci-assign,host=0b:00.0
```

## 优缺点

`Pros`:

- More Scalable than Direct Assign
- Security through IOMMU and function isolation
- Control Plane separation through PF/VF notion
- High packet rate, Low CPU, Low latency thanks to Direct Pass through

`Cons`:

- Rigid: Composability issues
- Control plane is pass through, puts pressure on Hardware resources
- Parts of the PCIe config space are direct map from Hardware
- Limited scalability (16 bit)
- SR-IOV NIC forces switching features into the HW
- All the Switching Features in the Hardware or nothing

## 参考

- [Intel SR-IOV Configuration Guide](http://www.intel.com/content/www/us/en/embedded/products/networking/xl710-sr-iov-config-guide-gbe-linux-brief.html)
- [OpenStack SR-IOV Passthrough for Networking](https://wiki.openstack.org/wiki/SR-IOV-Passthrough-For-Networking)
- [Redhat OpenStack SR-IOV Configure](https://access.redhat.com/documentation/zh-CN/Red_Hat_Enterprise_Linux_OpenStack_Platform/7/html/Networking_Guide/sec-sr-iov.html)
- [SDN Fundamentails for NFV, Openstack and Containers](http://www.slideshare.net/nyechiel/sdn-fundamentals-for-nfv-open-stack-and-containers-red-hat-summit-20161)
- [I/O设备直接分配和SRIOV](http://www.cnblogs.com/sammyliu/p/4548194.html)
- [Libvirt PCI passthrough of host network devices](http://wiki.libvirt.org/page/Networking#PCI_Passthrough_of_host_network_devices)
- [Story of Network Virtualization and its future in Software and Hardware](http://netdevconf.org/2.1/session.html?jain)
