# eBPF

`eBPF`（`extended Berkeley Packet Filter`）起源于`BPF`，它提供了内核的数据包过滤机制。

BPF的基本思想是对用户提供两种`SOCKET`选项：`SO_ATTACH_FILTER`和`SO_ATTACH_BPF`，允许用户在`sokcet`上添加自定义的`filter`，只有满足该`filter`指定条件的数据包才会上发到用户空间。`SO_ATTACH_FILTER`插入的是`cBPF`代码，`SO_ATTACH_BPF`插入的是`eBPF`代码。`eBPF`是对`cBPF`的增强，目前用户端的`tcpdump`等程序还是用的`cBPF`版本，其加载到内核中后会被内核自动的转变为`eBPF`。

Linux 3.15 开始引入 `eBPF`。其扩充了 `BPF` 的功能，丰富了指令集。它在内核提供了一个虚拟机，用户态将过滤规则以虚拟机指令的形式传递到内核，由内核根据这些指令来过滤网络数据包。

![EBPF](../images/ebpf.png)

`BPF`和`eBPF`的内核文档见[Documentation/networking/filter.txt](https://www.kernel.org/doc/Documentation/networking/filter.txt)。

## 使用场景

`eBPF`使用场景包括

- `XDP`
- 流量控制
- 防火墙
- 网络包跟踪
- 内核探针
- `cgroups`
- [bcc](bcc.md)
- [bpftools](https://github.com/cloudflare/bpftools)

## 参考

- [Linux Kernel BPF documentation](https://www.kernel.org/doc/Documentation/networking/filter.txt)
- [bcc](https://github.com/iovisor/bcc)
- [The BSD Packet Filter: A New Architecture for User-level Packet Capture](http://step.polymtl.ca/~suchakra/PWL-Jun28-MTL.pdf)
- [Notes on BPF & eBPF](https://jvns.ca/blog/2017/06/28/notes-on-bpf---ebpf/)
