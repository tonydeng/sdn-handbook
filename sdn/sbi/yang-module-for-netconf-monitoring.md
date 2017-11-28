# YANG Module for NETCONF Monitoring

![rfc8071 datatracker status](../images/rfc6022-datatracker-status.jpg)
本文主要内容都来自于2010年10月发布的[RFC6022 -YANG Module for NETCONF Monitoring](https://tools.ietf.org/html/rfc6022)，该RFC从2007年11月提出到最终发布一共修改了15个版本，其间修改内容可以[点击查看详细内容](https://datatracker.ietf.org/doc/rfc6022/)。

## 介绍

本文档定义了一个用于监视`NETCONF`协议的`YANG` [RFC6020](https://tools.ietf.org/html/rfc6020)模型。 它提供了有关`NETCONF`会话和支持架构的信息，如[RFC4741](https://tools.ietf.org/html/rfc4741)中所定义。

诸如不同的架构格式，功能可选性和访问控制等考虑因素都会影响`NETCONF`服务器在会话设置期间发送给客户端的详细信息的适用性和级别。

本文档中定义的方法需要进一步的手段来从`NETCONF`服务器查询和检索模式和`NETCONF`状态信息。 提供这些是为了补充现有的`NETCONF`功能和操作，绝不会影响现有的行为。

还定义了一个新的`<get-schema>`操作来支持通过`NETCONF`显式的模式检索。

## NETCONF监控数据模型

本文定义的NETCONF监控数据模型提供了`NETCONF`服务器的操作信息。 这包括特定于`NETCONF`协议的细节（例如，特定于协议的计数器，诸如“`in-sessions`”）以及与模式检索（例如，`schema list`）有关的数据。

实现本文档定义的数据模型的服务器（“`urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring`”）务必按照[RFC6020](https://tools.ietf.org/html/rfc6020)中的描述告知`capability URI`。

本节概述了监控数据模型。 有关详细说明，请参阅本文档中提供的规范的[YANG模块（请参阅第5节）](https://tools.ietf.org/html/rfc6022#section-5)。

### `/netconf-state`子树

`netconf-state`容器是监视数据模型的根。

```
netconf-state
    /capabilities
    /datastores
    /schemas
    /sessions
    /statistics
```

- 功能(`capabilities`)
       服务器支持的`NETCONF`功能列表。
- 数据存储(`datastores`)
       此设备上支持的`NETCONF`配置数据存储列表（例如，运行，启动，候选）以及相关信息。
- 模式(`schemas`)
       服务器上支持的模式列表。 包括识别模式和支持其检索所需的所有信息。
- 会话(`sessions`)
       设备上所有活动的NETCONF会话列表。 包括所有`NETCONF`会话的每个会话计数器。
- 统计(`statistics`)
       包括`NETCONF`服务器的全局计数器。

#### `/netconf-state/capabilities`子树

`/netconf-state/capabilities`子树包含`NETCONF`服务器支持的功能。

该列表必须包括在会话建立期间交换的所有能力，在请求时仍然适用。

#### `/netconf-state/datastores`子树

`/netconf-state/datastores`子树包含`NETCONF`服务器的可用数据存储列表，并包含有关其锁定状态的信息。

```
datastore
    /name
    /locks
```

- 名称(`name`)（`leaf`，`netconf-datastore-type`）
       支持枚举的数据存储; `candidate`, `running`, `startup。`

- 锁(`locks`)(`grouping`, `lock-info`)
        数据存储的锁列表。 信息提供全局和部分锁定[RFC5717](https://tools.ietf.org/html/rfc5717)。
        对于部分锁定，将返回锁定节点列表和最初用于请求锁定的选择表达式。

#### `/netconf-state/schemas`子树

`NETCONF`服务器支持的模式列表。

```
schema
       /identifier   (key)
       /version      (key)
       /format       (key)
       /namespace
       /location
```

network elements的`identifier`，`version`和`format`在模式列表中用作关键字。 这些用在`<get-schema>`操作中。

- `identifier (string)`
    模式列表条目的标识符。 该标识符在`<get-schema>`操作中使用，可用于其他方式，如文件检索。
- `version (string)`
    支持的模式的版本。 一个`NETCONF`服务器可以同时支持多个版本。 每个版本必须在模式列表中单独报告，即具有相同的标识符，可能不同的位置，但是不同的版本。

    对于`YANG`数据模型，版本是模块或子模块中最新的`YANG`'`revision`'语句的值，或者如果不存在'`revision`'语句，则为空字符串。
- `format (identifyref, schema-format)`
    标识是使用什么格式的数据建模语言模式来描述该`schema`。 可以使用“`xsd`”，“`yang`”，“`yin`”，“`rng`”和“`rnc`”这些格式来描述（见[第5节](https://tools.ietf.org/html/rfc6022#section-5)）。
- `namespace (inet:uri)`
    由模式定义的可扩展标记语言（`XML`）名称空间[XML-NAMES](https://tools.ietf.org/html/rfc6022#ref-XML-NAMES)。
- `location (union: enum, inet:uri)`
    一个或多个可从中检索特定模式的位置。 该列表应该每个模式至少包含一个条目。

#### `/netconf-state/sessions`子树

包括`NETCONF`管理会话的会话特定数据。 会话列表必须包含所有当前活动的`NETCONF`会话。

```
session
       /session-id (key)
       /transport
       /username
       /source-host
       /login-time
       /in-rpcs
       /in-bad-rpcs
       /out-rpc-errors
       /out-notifications
```

- `session-id (uint32, 1..max)`
    会话的唯一标识符。 该值是[RFC4741](https://tools.ietf.org/html/rfc4741)中定义的`NETCONF`会话标识符。
- `transport (identityref, transport)`
    标识每个会话的传输。 本文档定义了“`netconf-ssh`”，“`netconf-soap-over-beep`”，“`netconf-soap-over-https`”，“`netconf-beep`”和“`netconf-tls`”（见[第5节](https://tools.ietf.org/html/rfc6022#section-5)）。
- `username (string)`
    `username`是由`NETCONF`传输协议验证的客户端身份。 用于导出用户名的算法是`NETCONF`传输协议特定的，另外还特定于`NETCONF`传输协议使用的验证机制。
- `source-host (inet:host)`
    主机标识符（`IP`地址或名称）的`NETCONF`客户端。
- `login-time (yang:date-and-time)`
    在服务器上建立会话的时间。
- `in-rpcs (yang:zero-based-counter32)`
    接收到正确的`<rpc>`消息的数量。
- `in-bad-rpcs (yang:zero-based-counter32)`
    预期发送`<rpc>`消息时收到的消息数量，不正确的`<rpc>`消息。 这包括`rpc`层上的`XML`解析错误和错误。
- `out-rpc-errors (yang:zero-based-counter32)`
    在`<rpc-reply>`中包含`<rpc-error>`元素的消息数量。
- `out-notifications (yang:zero-based-counter32)`
    发送的`<notification>`消息的数量。

#### `/netconf-state/statistics`子树

有关`NETCONF`服务器的统计数据。

```
statistics
      /netconf-start-time
      /in-bad-hellos
      /in-sessions
      /dropped-sessions
      /in-rpcs
      /in-bad-rpcs
      /out-rpc-errors
      /out-notifications
```

statistics:

    包含`NETCONF`服务器的与管理会话相关的性能数据。

- `netconf-start-time (yang:date-and-time)`
    管理子系统启动的日期和时间。
- `in-bad-hellos (yang:zero-based-counter32)`
    收到无效的<hello>消息的数量。
- `in-sessions (yang:zero-based-counter32)`
    会话开始的数量。
- `dropped-sessions (yang:zero-based-counter32)`
    异常终止的会话数，例如，由于空闲超时或传输关闭。
- `in-rpcs (yang:zero-based-counter32)`
    接收到的消息的数量。
- `in-bad-rpcs (yang:zero-based-counter32)`
    预期发送`<rpc>`消息时收到的消息数量，不正确的`<rpc>`消息。 这包括`rpc`层上的`XML`解析错误和错误。
- `out-notifications (yang:zero-based-counter32)`
    在`<rpc-reply>`中包含`<rpc-error>`元素的消息数量。

## 模式的具体操作

### <get-schema>操作

- 描述：

    此操作用于从`NETCONF`服务器检索模式。

- 参数：
    - identifier (string)：模式列表条目的标识符。 强制性参数。
    - version (string)：请求的模式的版本。 可选参数。
    - format (identityref, schema-format)：模式的数据建模语言。 未指定时，默认值为“yang”。 可选参数。

- 正面回应：

    `NETCONF`服务器返回请求的模式。

- 负面的回应：

    如果请求的模式不存在，那么`<error-tag>`是“无效值”。
    如果多个模式匹配请求的参数，那么`<error-tag>`是'`operation-failed`'，而`<error-app-tag>`是'`data-not-unique`'。

## 例子

### 通过`<get>`操作检索模式列表

`NETCONF`客户端通过`<get>`操作检索`/netconf-state/schemas`子树，从`NETCONF`服务器中检索支持的模式列表。

请求会话的可用模式将在包含`<identifier>`，`<version>`，`<format>`和`<location>`元素的答复中返回。

响应数据可用于确定可用模式及其版本。 模式本身（即模式内容）不会在响应中返回。 可选的`<location>`元素包含一个`URI`，可以通过其他协议（如`ftp` [RFC0959](https://tools.ietf.org/html/rfc0959)或`http(s)`[RFC2616](https://tools.ietf.org/html/rfc2616) [RFC2818](https://tools.ietf.org/html/rfc2818)）或特殊值“`NETCONF`”来检索架构，这意味着 可以通过`<get-schema>`操作从设备检索模式。

例子：

```XML
<rpc message-id="101"
     xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <get>
    <filter type="subtree">
      <netconf-state xmlns=
      "urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring">
        <schemas/>
      </netconf-state>
    </filter>
  </get>
</rpc>
```

`NETCONF`服务器返回可用于检索的模式列表。

```xml
<rpc-reply message-id="101"
           xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <data>
    <netconf-state
    xmlns="urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring">
      <schemas>
        <schema>
          <identifier>foo</identifier>
          <version>1.0</version>
          <format>xsd</format>
          <namespace>http://example.com/foo</namespace>
          <location>ftp://ftp.example.com/schemas/foo_1.0.xsd</location>
          <location>http://www.example.com/schema/foo_1.0.xsd</location>
          <location>NETCONF</location>
        </schema>
        <schema>
          <identifier>foo</identifier>
          <version>1.1</version>
          <format>xsd</format>
          <namespace>http://example.com/foo</namespace>
          <location>ftp://ftp.example.com/schemas/foo_1.1.xsd</location>
          <location>http://www.example.com/schema/foo_1.1.xsd</location>
          <location>NETCONF</location>
        </schema>
        <schema>
          <identifier>bar</identifier>
          <version>2008-06-01</version>
          <format>yang</format>
          <namespace>http://example.com/bar</namespace>
          <location>
            http://example.com/schema/bar@2008-06-01.yang
          </location>
          <location>NETCONF</location>
        </schema>
        <schema>
          <identifier>bar-types</identifier>
          <version>2008-06-01</version>
          <format>yang</format>
          <namespace>http://example.com/bar</namespace>
          <location>
            http://example.com/schema/bar-types@2008-06-01.yang
          </location>
          <location>NETCONF</location>
        </schema>
      </schemas>
    </netconf-state>
  </data>
</rpc-reply>
```
