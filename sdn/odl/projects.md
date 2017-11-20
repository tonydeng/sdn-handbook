# OpenDaylight Projects介绍

## Controller(控制器)项目

> 项目地址： [opendaylight/controller](https://github.com/opendaylight/controller)

为多厂商网络的SDN部署提供一个高可用、模块化、可以扩展并可支持多协议的控制器基础框架。

在该项目中，模型驱动的业务抽象层使控制器支持多个南向协议插件，而面向应用的可扩展北向架构为控制器提供丰富的北向API：`针对松耦合应用的RESTful Web服务和针对协作应用的OSGi服务`。

另外，基于控制器平台的OSGi框架为控制器带来了模块化、可扩展的特性，还能为OSGi模型和服务提供版本控制和生命周期管理。

## OpenDaylight Root Parent项目

> 项目地址： [opendaylight/odlparent](https://github.com/opendaylight/odlparent)

OpenDaylight中所有项目的Maven配置基础，包含外部依赖、默认版本、依赖管理、插件管理、库信息等所有共同信息。

它能为参与版本发布的所有项目提供统一的设置，其他项目的配置只需要继承`odlparent`即可获得ODL的统一设置，这在很大程度上能够帮助简化项目配置。

## YANG Tools项目

> 项目地址： [opendaylight/yangtools](https://github.com/opendaylight/yangtools)

旨在开发必须的工具和库的基础设置项目，它能为Java项目（基于JVM语言）和应用提供`NETCONF`和`YANG`支持，在OpenDaylight中，使用`YANG`作为模型化语言的应用有控制器`MD-SDL`和`NETCONF/OFConfig`插件。

## AAA项目

> 项目地址： [opendaylight/aaa](https://github.com/opendaylight/aaa)

为用户开发身份认证、授权、计费等功能。

包括为用户提供适用于多种身份认证、授权、计费机制的通用模型，提供可插拔的机制为通用系统提供插件。

## BGP LS PCEP（BGPCEP）项目

> 项目地址： [opendaylight/bgpcep](https://github.com/opendaylight/bgpcep)


为控制器提供两种南向接口插件--作为L3拓扑信息来源BGP（包括作为BGP扩展的BGP-LS、BGP-FlowSpec）和为底层网络提供实例化路径的PCEP（Path Computation Element Protocol）。

## DLUX项目

> 项目地址： [opendaylight/dlux](https://github.com/opendaylight/dlux)

为控制器的使用者提供新的交互式Web UI应用，它选择了`AngularJS`作为主要的前端技术，希望能够通过图形化的用户界面提高用户体验。

## L2Switch项目

> 项目地址： [opendaylight/l2switch](https://github.com/opendaylight/l2switch)

该项目将L2的具体处理代码分离出来，组成一个独立的项目，提供基本的L2交换机功能并创建一些可重用的服务。

如提供模块化的事件驱动的数据包处理程序(packet handler)、地址跟踪、最优路径计算、基本的生成树协议等。

## LISP Flow Mapping Service项目

> 项目地址： [opendaylight/lispflowmapping](https://github.com/opendaylight/lispflowmapping)

该项目提供`LISP`映射系统服务，包括`LISP Map-Server`和`LISP Map-Resolver`服务，负责提供和存储数据到数据屏幕节点和OpenDaylight应用的映射。

`LISP（Locator ID Separation Protocol)`是一个提供灵活的映射和封装框架的技术，它可用于数据中心网络虚拟化、网络功能虚拟化等`overlay`网络应用之中。

映射数据包括是虚拟化节点可达的虚拟地址到物理网络地址的映射以及流量工程、负载均衡等各种各样的路由策略。

OpenDaylight应用和服务可使用北向REST API在LISP映射服务中定义映射和策略，数据平面设备通过南向LISP插件具备LISP控制协议的能力。

## Neutron Northbound项目

> 项目地址： [opendaylight/neutron](https://github.com/opendaylight/neutron)

一个像OpenStack网络管理项目Neutron提供北街项目的插件项目，是使OpenDaylight和OpenStack协同工作的重要项目。

它提供网络、子网、端口、负载均衡、VPN、安全策略等REST API，并随着OpenDaylight的发展，不断的增加。

## ODL SDNi App项目

> 项目地址： [opendaylight/sdninterfaceapp](https://github.com/opendaylight/sdninterfaceapp)


OpenDaylight SDN接口应用项目旨在通过开发软件定义网络接口应用(Software Defined Networking interface, SDNi)保证SDN控制器之间的通信，该应用可在Helium版本的OpenDaylight上部署。

## OpenFlow Protocol Library项目

> 项目地址： [opendaylight/openflowjava](https://github.com/opendaylight/openflowjava)

OpenFlow协议库将会实施OpenFlow v1.3及后续版本协议。

该库是控制器的OpenFlow南向插件的基础，支持第三方供应商的扩展。

## OpenDaylight OpenFlow Plugin项目

> 项目地址： [opendaylight/openflowplugin](https://github.com/opendaylight/openflowplugin)

OpenFlow是SDN架构中实现控制层和转发层之间交互的厂商中里的标准通信接口。

该项目旨在开发一个支持OpenFlow规范的插件，该插件将实现OF 1.0、OF 1.3、OF 1.4及后续版本的支持和整合。

## Persistence Store Service项目

> 项目地址： [opendaylight/persistence](https://github.com/opendaylight/persistence)

本项目是一个数据库统一操作服务框架，该框架的作用是实现营业在查询时的查询逻辑持久性，主要应用在需要连接数据库查询数据的应用中，例如`AAA`项目和`TSDR(Time Series Data Repository)`项目。

框架将不同的数据库查询方式封装起来，暴露统一的接口给ODL应用使用，提供数据查询、增加、删除等数据操作功能，并且维护查询操作的完整性。

框架可以支持不同类型的数据库，关系型数据库如`MySQL`，费关系型数据库如`MongoDB`或者内存数据库。

## SNBI(Secure Network Bootstrapping Infrastructure)项目

> 项目地址： [opendaylight/snbi](https://github.com/opendaylight/snbi)

提供安全、自动、集成的网络设备和控制器。

通常，在安全通信建立之前，操作人员必须在一系列网络设备之间执行手动密钥分发过程。而SNBI使用零接触的方法即可引导安装`IEEE 802.1 AR`证书的产品进行安全认证。

SNBI设备和控制器能够自动发现对方，获取制定的IP地址并建立安全的IP连接。

另外，这个发现过程可展示网络的物理拓扑，识别每个设备的类型（可能是一个常规的网络设备或控制器），并为每个设备指定域，设备类型和域信息可用于控制器联合流程的初始化。

SNBI还包括控制器和转发单元上创建的组件和功能，这些组件和功能可包括单个的网络单元服务 ，如性能分析、流量探测、流量传送等。

## SNMP4SDN项目

> 项目地址： [opendaylight/snmp4sdn](https://github.com/opendaylight/snmp4sdn)

提供一个SNMP南向插件实现OpenDaylight控制器对现有商用以太网交换机的控制，该插件可提供管理配置的能力。

该项目使SDN不在局限于OpenFlow，支持以太网交换机作为SDN网络的数据面设备，它主要经历以下三个阶段的演进：

1. 创建一个SNMP南向插件配置通过SNMP配置以太网交换机
2. 插件通过CLI（Command Line Interface，命令行接口）对以太网交换机做一些SNMP不能放松的设定
3. 实现SAL API的扩展

## SNMP Plugin项目

> 项目地址： [opendaylight/snmp](https://github.com/opendaylight/snmp)

SNMP是一个实现网络管理的协议。

它可用来手机信息、配置IP网络设备，如交换机、路由器、打印机、服务器等。

该项目致力于南向插件的需求，使应用和控制器服务能够使用SNMP与网络设备实现交互。

SNMP南向插件使应用充当SNMP管理者与支持SNMP代理的设备的交互。

## SXP(Source-Group Tag eXchange Protocol)项目

> 项目地址： [opendaylight/sxp](https://github.com/opendaylight/sxp)

SXP是一个传送IP地址和源组标签(Source Group Tag, SGT)之间绑定信息的控制协议。

在SXP中，源组是一系列具有共同网络策略的连接网络的端节点。每个源组通过一个特殊的SGT值(16字节)标识(大多数思科设备都支持SGT标识)。

该项目旨在ODL中实现SXP，使ODL获取大型已安装思科设备的绑定信息，并提供个应用和网络单元。

当前实施支持SXP协议的版本4，并提供对1~3版本的支持。此外，作为单向连接的扩展，版本4还增加了双向连接类型。

## TCP-MD5项目

> 项目地址： [opendaylight/archived-tcpmd5](https://github.com/opendaylight/archived-tcpmd5)

本项目库为操作系统提供TCP-MD5([RFC2385](https://tools.ietf.org/html/rfc2385))支持，可用于保护BGP会话和PCEP会话。

该项目定义了一个使用Java本地接口库(Java Native Interface, JNI)实现简单API，可用于设置与TCP channel关联的MD5密钥。

虽然相应的代码已经存在于BGPCEP项目中，但该项目旨在将这些代码分离为一个独立的组件，使其生命周期独立与BGPCEP项目，这将带来以下益处：
> 不同提交者带来的无限潜力、明确的问题阐述以及更为稳定的版本。

## Topology Protocol Framework项目

> 项目地址： [opendaylight/topoprocessing](https://github.com/opendaylight/topoprocessing)

旨在创建拓扑聚合的框架，提供统一的拓扑视图。

该项目主要提供量大产品特点：拓扑聚合和拓扑过滤。

1. 拓扑聚合指从数据库中获取`underlay`拓扑信息，然后将其聚合为一个或多个`overlay`拓扑，这些拓扑基于`router ID`、管理IP和`datapath ID`等映射关系创建，提供多协议节点抽象(多协议节点抽象是通过RPC转发实现的)的统一视图，使应用无需关注多个拓扑和多个节点标识。

这个框架还提供过滤拓扑视图，该过滤可应用到交换机、交换机组、特定链接和其他对象。

### Use Case

![OpenDaylight Topology Protocol Framework Use Case](https://wiki.opendaylight.org/images/thumb/0/02/Topoprocessing_Proposal_Topology.png/800px-Topoprocessing_Proposal_Topology.png)

## ALTO(Application-Layer Traffic Optimization)项目

> 项目地址： [opendaylight/alto](https://github.com/opendaylight/alto)

ALTO是一个为应用提供网络信息的IETF协议，定义映射成本服务、过滤映射服务、端节点属性服务、端节点服务成本等网络服务，从而引导应用使用网络资源。

该项目致力于在OpenDaylight中实现ALTO。为了实现ALTO基础协议([RFC7285](https://tools.ietf.org/html/rfc7285))，该项目将在OpenDaylight中实现这些服务并通过北向API开放给他人使用。

## CAPWAP(Cotrol And Provisioning of Wireless Accesss Points)项目

> 项目地址： [opendaylight/capwap](https://github.com/opendaylight/capwap)

该项目是为了“有线和无线网络通过适当抽象实现统一管理”的长远目标而提出的。

现阶段它致力于提供CAPWAP协议作为了一个南向接口。该项目的作用范围包括：

1. 为CAPWAP提供南向MD-SAL插件
2. CAPWAP协议库
3. CAPWAP和IEEE 802.11之间的绑定关系
4. CAPWAP和IEEE 802.11间绑定关系的标准测试应用
5. 本地映射支持

## Controller Core Funcationality Tutorials项目

> 项目地址： [opendaylight/coretutorials](https://github.com/opendaylight/coretutorials)

该项目面向开发者提供各种基础功能的教程，以期开发者快速理解OpenDaylight的项目结构、基础功能，从而能加入到OpenDaylight社区开发。

这些教程涉及的工程有`Controller`、`YANG Tools`、`OpenFlow Java`以及`OpenFlow Plugin`，其中包含各种基础操作有`ConfigSubsysterm`、`NETCONF`、`RESTCONF`、`flow`以及`MD-SAL`的相关操作。

该项目提供的各个教程均是使用标准的项目结构(例如`ArcheType`)，并且每个教程均采用分步式方法介绍。

## Defense4All项目

> 项目地址： [opendaylight/archived-defense4all](https://github.com/opendaylight/archived-defense4all)

一个检测和缓解DDoS攻击的SDN应用。

在本项目定义的系统中，应用通过ODC北向REST API与OpenDaylight控制器之间实行交互，主要指向以下两大功能：监控被保护流量的行为并将攻击流量转移到被选攻击缓解系统(Attack Mitigation System, AMS)。

该项目的实现方法如下：

> 应用通过OpenDaylight控制器想每个被保护网(`Protected Network, PN`)的合适位置下发流条目，采集和聚合流量并利用流条目的`counter`字段做流量统计，识别异常流量，然后再想被选网络位置下发流条目，将异常流量转移到被选`AMS`。

> 当攻击消失时，应用将移除相应的流条目，使流量回归到正常的路径。

## DIDM(Device Identification and Driver Management)项目

> 项目地址： [opendaylight/didm](https://github.com/opendaylight/didm)

提供一个可用于通知控制器发现它所控制的新设备、标识设备类型、将设备驱动注册为路由类型的RPC(Remote Procedure Call, 远程过程调用)、搜集设备数据、定义库存模型、调用设备驱动的框架。

该项目使用SNMP协议与设备进行交互时需要进行安全认证，因此它需要使用SNMP南向协议插件项目、AAA项目(进行认证管理)中的一些组件。

## Documentation项目

> 项目地址： [opendaylight/docs](https://github.com/opendaylight/docs)

提供OpenDaylight项目群的文档。

该项目计划建立内容基础框架，用来改善各个OpenDaylight项目的用户说明以及确保不断增长的内容的质量。

整个项目管理包括内容编辑、内容发布以及设置内容格式等。

## Group Based Policy(GBP)项目

> 项目地址： [opendaylight/groupbasedpolicy](https://github.com/opendaylight/groupbasedpolicy)

定义以应用为中心的策略模型，将应用的网络连接从底层网络抽象出来。

该工程提供了一个简单的自我记录的机制来获取策略，而不需要关心底层网络信息的框架，另外通过将网络端点分组，可以同时操作多个节点，提升了策略下发的自动化，同时也可以方便的统一和简洁的处理策略的变化。

## Integration Group项目

> 项目地址：
[opendaylight/integration-test](https://github.com/opendaylight/integration-test)
[opendaylight/integration-packaging](https://github.com/opendaylight/integration-packaging)
[opendaylight/integration-distribution](https://github.com/opendaylight/integration-distribution)

提供协调、促进集成以及持续集成测试的基础框架，以期能够发布成功的OpenDaylight版本。

该项目服务内容有：

1. OpenDaylight整体测试策略制定
2. 持续系统集成测试
3. 发布版本协调
4. 社区实验室测试协调
5. 自动化测试框架
6. 测试用例管理
7. 实验环境搭建

## IoTDM(Internet of Things Data Management)项目

> 项目地址：[opendaylight/iotdm](https://github.com/opendaylight/iotdm)

提供以数据为中心的中间件，运行通过验证的应用访问设备上传的物联网数据。

该项目使用ODL平台模型化`oneM2M`的`DataStore(分层容器树)`，其中书上每个节点均表示`oneM2M`的物联网某个资源。

物联网设备与应用通过资源树进行交互，其中网络协议包含`CoAP(Constrained Application Protocol)`、`MQTT(Message Queuing Telemetry Transport, 消息队列遥测传输)`以及`HTTP`。

## LACP(Link Aggregation Control Protocol)项目

> 项目地址：[opendaylight/lacp](https://github.com/opendaylight/lacp)

一种实现链路动态汇聚的协议。

在带宽比较紧张的情况下，该协议可通过逻辑聚合将带宽扩展到原链路的`n`倍，还可通过配置链路聚合实现同一聚合组的各个端口之间彼此动态备份。

在OpenDaylight中，LACP项目使用MD-SAL的方式实现了LACP协议，可用于ODL控制网络或支持LACP的交换机/设备的自动发现和多链路聚合。

## NIC(Network Intent Composition)项目

> 项目地址：[opendaylight/nic](https://github.com/opendaylight/nic)

使控制器能够按照网络行为和网络策略定义的“意图”对网络服务的网络资源进行调度和管理。

这个意图通过一个新的北向接口(Northbound Interface，NBI)描述，北向接口提供一个普遍的抽象的策略语义，用于代替类似OpenFlow的流表规则。

不同于描述如何提供不同服务的SDN接口，基于NBI的“意图”允许通过描述性的方式从基础设备获取需要的资源。

NBI负责协调编排服务和面向网络和业务的SDN应用，包括OpenStack Neutron、SFC项目和GBP项目。

NIC将使用现有OpenDaylight网络服务功能和南向插件来控制虚拟和物理的网络设备。

NIC被设计Wie协议无关的，可以使用OpenFlow、OVSDB、I2RS、NETCONFG、SNMP等控制协议。

## OpenvSwitch Database Integration项目

> 项目地址：[opendaylight/ovsdb](https://github.com/opendaylight/ovsdb)

一个为OpenDaylight实施OpenvSwitch Database([RFC7047](https://tools.ietf.org/html/rfc7047))管理协议的项目，可实现虚拟交换机的南向配置。

该项目包含一个OVSDB库及相关的各种插件的用法。

> [OpenDaylight OVSDB Integration:Design](https://wiki.opendaylight.org/view/OVSDB_Integration:Design)

```
+--------------+--------------+-----------+
|  Connection  |Network Config|  Neutron  |
|  Service APIs|     APIs     |    APIs   |
+--------------+--------------+-----------+
|       NorthBound API Layer (REST)       |
+-----------------------------------------+
+-----------------------------------------+
| +------------+ +---------+ +---------+  |
| |Connection  | |Topology | | Switch  |  |
| |  Manager   | | Manager | | Manager |  |
| +------------+ +---------+ +---------+  |
|    Network Service Functions Layer      |
+-----------------------------------------+
+-----------------------------------------+
|  +------------+ +---------+ +---------+ |
|  |Connection  | |Data Pkt | |Inventory| |
|  |  Service   | | Service | | Service | |
|  +------------+ +---------+ +---------+ |
|     Service Abstraction Layer (SAL)     |
+-----------------------------------------+
+-----------------------------------------+
|            SouthBound API Layer         |
|  +----------------+ +----------------+  |
|  | OpenFlow Plugin| | OVSDB Plugin   |  |
|  | +------------+ | | +------------+ |  |
|  | | Flow Prog  | | | | Connection | |  |
|  | |  Service   | | | |  Service   | |  |
|  | +------------+ | | +------------+ |  |
|  | +------------+ | | +------------+ |  |
|  | | Data Pkt   | | | | Net Config | |  |
|  | |  Service   | | | |  Service   | |  |
|  | +-----.------+ | | +-----.------+ |  |
|  |       .        | |       .        |  |
|  +----------------+ +----------------+  |
+-----------------------------------------+
```

## OpFlex项目

> 项目地址：[opendaylight/opflex](https://github.com/opendaylight/opflex)

主要提供相关OpFlex协议的实现，该协议基于策略模型实现一个分布式控制系统。

在OpFlex协议中，多数本地执行策略是由一个逻辑集中策略库提供。

项目主要分为三个部分：

1. `libopflex`是一个在OpFlex协议框架下交互管理对象、解析策略、更新订阅的工具库
2. `genie`是一个根据`libopflex`库中模型生成代码的框架
3. `agent-ovs`是一个和OVS一起运行的策略代理，主要负责执行OpFlex的策略，这个策略代理可以编排工具(如OpenStack)一同使用。

## PCMM(Packet Cable MultiMedia)项目

> 项目地址：[opendaylight/packetcable](https://github.com/opendaylight/packetcable)

提供控制和管理`CMTS(Cable Modem Terminatin System, 线缆调制解调器终端系统)`网络单元服务流的接口。

PCMM架构包含以下组件：

1. 应用管理器，将QoS需求基于每应用发送给策略服务器。
2. 策略服务器，按每应用每用户分配网络资源，保证消耗量能满足(Multiple Service Operator)优先级。
3. CMTS，根据带宽容量执行策略。
4. 电缆调制解调器，位于客户端，负责帮助客户网络连接到电缆系统。

该项目的目标是利用OpenDaylight控制平台作为应用管理器和部分策略服务器，并最大程度的利用该平台提供的现有组件。

## Release Engineering-Autorelease项目
> 项目地址：[opendaylight/releng-autorelease](https://github.com/opendaylight/releng-autorelease)

该项目致力于对所有相关脚本进行版本管理和位置标记。

它负责创建、管理和提交OpenDaylight每个发布版本的相关标签，并制定按照设定的常规周期(如每天、每周、每月、里程碑时间)来生成候选版本。

## Reservation项目

> 项目地址：[opendaylight/reservation](https://github.com/opendaylight/reservation)

提供动态的低级别的资源预留，使用在指定时间段获得网络及服务、连通性、相应的资源池(端口、带宽)等。

## Service Function Chaining(SFC)项目

> 项目地址：[opendaylight/sfc](https://github.com/opendaylight/sfc)

定义有顺序的网络服务(如防火墙、负载均衡)链表的能力，这些网络服务按照一定顺序连接在一起称为业务链。

该项目为网络和终端用户应用提供定义业务链所需的基础框架(建逻辑链、API)。

## TTP(Table Type Patterns)项目

> 项目地址：[opendaylight/ttp](https://github.com/opendaylight/ttp)

ONF的转发抽象工作组(Forwarding Abstractions Working Group, FAWG)的第一个具体输出，其目标是使OpenFlow控制器和OpenFlow交换机对一系列功能进行协商，从而实现OF 1.1+版本的多样性管理。

TTP项目是在“数据路径协商模型”下提出的，当前主要聚焦于OpenFlow。

## TSDR(Time Series Data Repository)项目

> 项目地址：[opendaylight/tsdr](https://github.com/opendaylight/tsdr)

目的是创建一个可伸缩、可扩展的时间序列采集数据的持久化框架，其中采集的数据保护`DataStore`的统计数据以及消息总线的消息。

该项目保护一个时间序列数据存储库以及一组时间序列数据MD-SAL服务(包括采集、存储、查询以及维护时间序列数据等服务)，并提供访问TDSR数据的北向接口。

## Unified Secure Channel项目

> 项目地址：[opendaylight/usc](https://github.com/opendaylight/usc)

在企业网中，越来越多的控制器和网络管理系统正在远程部署(如在云端部署)，除此之外，企业网页越来越多样化，如分支、物联网、无线等。

因此，企业客户需要一个收敛的网络控制器和管理系统解决方案，该项目在网络单元和控制器之间建立一个统一的安全通信隧道： 创建安全通道和支持多种通信协议的通用机制。

## VPN Service项目

> 项目地址：[opendaylight/vpnservice](https://github.com/opendaylight/vpnservice)

目标是提供建立基于BGP-MPLS([RFC4364](https://tools.ietf.org/html/rfc4364))的L3 VPN服务所需的基础设施，以后的版本将提供基于EVPN的L2 VPN服务。

该项目与OVSDB Integration项目、BGP-LS、SDNi的相关组件具有一定的依赖关系。

## VTN(Virtual Tenant Network)项目

> 项目地址：[opendaylight/vtn](https://github.com/opendaylight/vtn)

为用户提供多租户级别的虚拟网络。

在传统网络中，如果要为企业用户配置和部署某个部门或某个应用系统的网络，需要对每个租户进行安装和配置，而且不同租户间的网络是不可共享的。而VTN将网络分割为逻辑平面和物力平面，将逻辑平面提供给用户，用户可以自己设计和部署网络，不需要了解物力网络的拓扑结构和带宽限制。

VTN允许用户定义类似传统的二层和三层网络，并自动将设计的网络映射到物理网络上。逻辑平面的定义不仅可以隐藏底层网络的复杂性，同时也能更好的管理网络资源。

这不仅可以减少配置网络服务的时间，还能减少网络配置错误。
