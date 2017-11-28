# YANG Module for NETCONF Monitoring

![rfc6022 datatracker status](../images/rfc6022-datatracker-status.jpg)
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
    - `identifier (string)`：模式列表条目的标识符。 强制性参数。
    - `version (string)`：请求的模式的版本。 可选参数。
    - `format (identityref, schema-format)`：模式的数据建模语言。 未指定时，默认值为“`yang`”。 可选参数。

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

### 检索模式实例

鉴于上一节中的答复，以下示例说明在多个位置，多种格式和多个位置检索“`foo`”，“`bar`”和“`bar-types`”模式。

1. `foo`，`xsd`格式的`version 1.0`：

- 通过FTP使用位置

        ftp://ftp.example.com/schemas/foo_1.0.xsd
- 通过HTTP使用位置

        http://www.example.com/schema/foo_1.0.xsd

- 通过`<get-schema>`使用标识符，版本和格式参数。

```xml
<rpc message-id="101"
    xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <get-schema
        xmlns="urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring">
        <identifier>foo</identifier>
        <version>1.0</version>
        <format>xsd</format>
    </get-schema>
</rpc>

<rpc-reply message-id="101"
    xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <data
        xmlns="urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring">
        <xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
         <!-- foo 1.0 xsd schema contents here -->
        </xs:schema>
    </data>
</rpc-reply>
```

2. `bar`，默认的`YANG`格式的`version 2008-06-01`：

- 通过`HTTP`使用位置

    http://example.com/schema/bar@2008-06-01.yang

- 通过`<get-schema>`使用标识符和版本参数：

```xml
<rpc message-id="102"
    xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <get-schema
        xmlns="urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring">
         <identifer>bar</identifer>
         <version>2008-06-01</version>
    </get-schema>
</rpc>

<rpc-reply message-id="102"
    xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <data
        xmlns="urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring">
     module bar {
        //default format (yang) returned
        //bar version 2008-06-01 yang module
        //contents here ...
     }
    </data>
</rpc-reply>
```

3. `bar-types`，默认的`YANG`格式的`version 2008-06-01`。

- 使用`identifier`参数:

```xml
<rpc message-id="103"
    xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <get-schema
        xmlns="urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring">
        <identifer>bar-types</identifer>
    </get-schema>
</rpc>

<rpc-reply message-id="103"
    xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <data
        xmlns="urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring">
        module bar-types {
             //default format (yang) returned
             //latest revision returned
             //is version 2008-06-01 yang module
             //contents here ...
        }
    </data>
</rpc-reply>
```

## NETCONF监控数据模型

本备忘录中描述的数据模型在以下的`YANG`模块中定义。

这个YANG模块从[RFC6021]()和[RFC4741](https://tools.ietf.org/html/rfc4741)，[RFC4742](https://tools.ietf.org/html/rfc4742)，[RFC4743](https://tools.ietf.org/html/rfc4743)，[RFC4744](https://tools.ietf.org/html/rfc4744)，[RFC5539](https://tools.ietf.org/html/rfc5539)，[xmlschema-1](https://tools.ietf.org/html/rfc6022#ref-xmlschema-1)，[RFC6020](https://tools.ietf.org/html/rfc6020)，[ISO / IEC19757-2:2008]， 和[RFC5717](https://tools.ietf.org/html/rfc5717)。

`<CODE BEGINS>` file "`ietf-netconf-monitoring@2010-10-04.yang`"

```yang
module ietf-netconf-monitoring {

    namespace "urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring";
    prefix "ncm";

    import ietf-yang-types { prefix yang; }
    import ietf-inet-types { prefix inet; }

    organization
        "IETF NETCONF (Network Configuration) Working Group";

    contact
        "WG Web:   <http://tools.ietf.org/wg/netconf/>
        WG List:  <mailto:netconf@ietf.org>

        WG Chair: Mehmet Ersue
               <mailto:mehmet.ersue@nsn.com>

        WG Chair: Bert Wijnen
               <mailto:bertietf@bwijnen.net>

        Editor:   Mark Scott
               <mailto:mark.scott@ericsson.com>

        Editor:   Martin Bjorklund
               <mailto:mbj@tail-f.com>";

    description
        "NETCONF Monitoring Module.
        All elements in this module are read-only.

        Copyright (c) 2010 IETF Trust and the persons identified as
        authors of the code. All rights reserved.

        Redistribution and use in source and binary forms, with or
        without modification, is permitted pursuant to, and subject
        to the license terms contained in, the Simplified BSD
        License set forth in Section 4.c of the IETF Trust's
        Legal Provisions Relating to IETF Documents
        (http://trustee.ietf.org/license-info).

        This version of this YANG module is part of RFC 6022; see
        the RFC itself for full legal notices.";

    revision 2010-10-04 {
        description
          "Initial revision.";
        reference
          "RFC 6022: YANG Module for NETCONF Monitoring";
    }

    typedef netconf-datastore-type {
        type enumeration {
            enum running;
            enum candidate;
            enum startup;
        }
        description
            "Enumeration of possible NETCONF datastore types.";
        reference
        "RFC 4741: NETCONF Configuration Protocol";
    }

    identity transport {
        description
            "Base identity for NETCONF transport types.";
        }

        identity netconf-ssh {
            base transport;
            description
                "NETCONF over Secure Shell (SSH).";
            reference
                "RFC 4742: Using the NETCONF Configuration Protocol
                         over Secure SHell (SSH)";
        }

        identity netconf-soap-over-beep {
            base transport;
            description
              "NETCONF over Simple Object Access Protocol (SOAP) over
               Blocks Extensible Exchange Protocol (BEEP).";
            reference
              "RFC 4743: Using NETCONF over the Simple Object
                         Access Protocol (SOAP)";
        }

        identity netconf-soap-over-https {
            base transport;
            description
              "NETCONF over Simple Object Access Protocol (SOAP)
              over Hypertext Transfer Protocol Secure (HTTPS).";
            reference
              "RFC 4743: Using NETCONF over the Simple Object
                         Access Protocol (SOAP)";
        }

        identity netconf-beep {
            base transport;
            description
                "NETCONF over Blocks Extensible Exchange Protocol (BEEP).";
            reference
              "RFC 4744: Using the NETCONF Protocol over the
                        Blocks Extensible Exchange Protocol (BEEP)";
        }

        identity netconf-tls {
            base transport;
            description
                "NETCONF over Transport Layer Security (TLS).";
            reference
                "RFC 5539: NETCONF over Transport Layer Security (TLS)";
        }

        identity schema-format {
            description
                "Base identity for data model schema languages.";
        }

        identity xsd {
            base schema-format;
            description
                "W3C XML Schema Definition.";
            reference
                "W3C REC REC-xmlschema-1-20041028:
                    XML Schema Part 1: Structures";
        }

        identity yang {
            base schema-format;
            description
                "The YANG data modeling language for NETCONF.";
            reference
                "RFC 6020:  YANG - A Data Modeling Language for the
                     Network Configuration Protocol (NETCONF)";
        }

        identity yin {
            base schema-format;
            description
                "The YIN syntax for YANG.";
            reference
                "RFC 6020:  YANG - A Data Modeling Language for the
                     Network Configuration Protocol (NETCONF)";
        }

        identity rng {
            base schema-format;
            description
                "Regular Language for XML Next Generation (RELAX NG).";
            reference
                "ISO/IEC 19757-2:2008: RELAX NG";
        }

        identity rnc {
            base schema-format;
            description
                "Relax NG Compact Syntax";
            reference
                "ISO/IEC 19757-2:2008: RELAX NG";
        }

        grouping common-counters {
           description
             "Counters that exist both per session, and also globally,
              accumulated from all sessions.";

           leaf in-rpcs {
             type yang:zero-based-counter32;
             description
               "Number of correct <rpc> messages received.";
           }
           leaf in-bad-rpcs {
             type yang:zero-based-counter32;
             description
                "Number of messages received when an <rpc> message was expected,
                 that were not correct <rpc> messages.  This includes XML parse
                 errors and errors on the rpc layer.";
            }
            leaf out-rpc-errors {
              type yang:zero-based-counter32;
              description
                "Number of <rpc-reply> messages sent that contained an
                 <rpc-error> element.";
            }
            leaf out-notifications {
              type yang:zero-based-counter32;
              description
                "Number of <notification> messages sent.";
            }
        }

        container netconf-state {
            config false;
            description
              "The netconf-state container is the root of the monitoring
               data model.";

            container capabilities {
                description
                    "Contains the list of NETCONF capabilities supported by the
                        server.";

                leaf-list capability {
                    type inet:uri;
                    description
                        "List of NETCONF capabilities supported by the server.";
                }
            }

        container datastores {
            description
                "Contains the list of NETCONF configuration datastores.";

        list datastore {
            key name;
            description
                "List of NETCONF configuration datastores supported by
                    the NETCONF server and related information.";

            leaf name {
              type netconf-datastore-type;
              description
                "Name of the datastore associated with this list entry.";
            }
            container locks {
                presence
                    "This container is present only if the datastore
                     is locked.";
                description
                    "The NETCONF <lock> and <partial-lock> operations allow
                     a client to lock specific resources in a datastore.  The
                     NETCONF server will prevent changes to the locked
                     resources by all sessions except the one that acquired
                     the lock(s).

                     Monitoring information is provided for each datastore
                     entry including details such as the session that acquired
                     the lock, the type of lock (global or partial) and the
                     list of locked resources.  Multiple locks per datastore
                     are supported.";

                grouping lock-info {
                    description
                      "Lock related parameters, common to both global and
                       partial locks.";

                    leaf locked-by-session {
                        type uint32;
                        mandatory true;
                        description
                            "The session ID of the session that has locked
                             this resource.  Both a global lock and a partial
                             lock MUST contain the NETCONF session-id.

                             If the lock is held by a session that is not managed
                             by the NETCONF server (e.g., a CLI session), a session
                             id of 0 (zero) is reported.";
                        reference
                            "RFC 4741: NETCONF Configuration Protocol";
                    }
                    leaf locked-time {
                        type yang:date-and-time;
                        mandatory true;
                        description
                            "The date and time of when the resource was
                             locked.";
                    }
                }

                choice lock-type {
                    description
                      "Indicates if a global lock or a set of partial locks
                       are set.";

                    container global-lock {
                        description
                            "Present if the global lock is set.";
                        uses lock-info;
                    }

                    list partial-lock {
                        key lock-id;
                        description
                            "List of partial locks.";
                        reference
                            "RFC 5717: Partial Lock Remote Procedure Call (RPC) for
                                    NETCONF";

                    leaf lock-id {
                        type uint32;
                        description
                            "This is the lock id returned in the <partial-lock>
                            response.";
                    }
                    uses lock-info;
                    leaf-list select {
                        type yang:xpath1.0;
                        min-elements 1;
                        description
                          "The xpath expression that was used to request
                           the lock.  The select expression indicates the
                           original intended scope of the lock.";
                    }
                    leaf-list locked-node {
                        type instance-identifier;
                        description
                          "The list of instance-identifiers (i.e., the
                           locked nodes).

                           The scope of the partial lock is defined by the list
                           of locked nodes.";
                    }
                }
            }
        }
    }
}
container schemas {
      description
        "Contains the list of data model schemas supported by the
         server.";

      list schema {
        key "identifier version format";

        description
          "List of data model schemas supported by the server.";

        leaf identifier {
          type string;
          description
            "Identifier to uniquely reference the schema.  The
             identifier is used in the <get-schema> operation and may
             be used for other purposes such as file retrieval.

             For modeling languages that support or require a data
             model name (e.g., YANG module name) the identifier MUST
             match that name.  For YANG data models, the identifier is
             the name of the module or submodule.  In other cases, an
             identifier such as a filename MAY be used instead.";
        }
        leaf version {
          type string;
          description
            "Version of the schema supported.  Multiple versions MAY be
             supported simultaneously by a NETCONF server.  Each
             version MUST be reported individually in the schema list,
             i.e., with same identifier, possibly different location,
             but different version.

             For YANG data models, version is the value of the most
             recent YANG 'revision' statement in the module or
             submodule, or the empty string if no 'revision' statement
             is present.";
        }
        leaf format {
          type identityref {
            base schema-format;
          }
          description
            "The data modeling language the schema is written
             in (currently xsd, yang, yin, rng, or rnc).
             For YANG data models, 'yang' format MUST be supported and
             'yin' format MAY also be provided.";
        }
        leaf namespace {
          type inet:uri;
          mandatory true;
          description
            "The XML namespace defined by the data model.

             For YANG data models, this is the module's namespace.
             If the list entry describes a submodule, this field
             contains the namespace of the module to which the
             submodule belongs.";
        }
        leaf-list location {
          type union {
            type enumeration {
              enum "NETCONF";
            }
            type inet:uri;
          }
          description
            "One or more locations from which the schema can be
             retrieved.  This list SHOULD contain at least one
             entry per schema.

             A schema entry may be located on a remote file system
             (e.g., reference to file system for ftp retrieval) or
             retrieved directly from a server supporting the
             <get-schema> operation (denoted by the value 'NETCONF').";
        }
      }
    }
    container sessions {
      description
        "The sessions container includes session-specific data for
         NETCONF management sessions.  The session list MUST include
         all currently active NETCONF sessions.";

      list session {
        key session-id;
        description
          "All NETCONF sessions managed by the NETCONF server
           MUST be reported in this list.";

        leaf session-id {
          type uint32 {
            range "1..max";
          }
          description
            "Unique identifier for the session.  This value is the
             NETCONF session identifier, as defined in RFC 4741.";
          reference
            "RFC 4741: NETCONF Configuration Protocol";
        }
        leaf transport {
          type identityref {
            base transport;
          }
          mandatory true;
          description
            "Identifies the transport for each session, e.g.,
            'netconf-ssh', 'netconf-soap', etc.";
        }
        leaf username  {
          type string;
          mandatory true;
          description
            "The username is the client identity that was authenticated
            by the NETCONF transport protocol.  The algorithm used to
            derive the username is NETCONF transport protocol specific
            and in addition specific to the authentication mechanism
            used by the NETCONF transport protocol.";
        }
        leaf source-host {
          type inet:host;
          description
            "Host identifier of the NETCONF client.  The value
             returned is implementation specific (e.g., hostname,
             IPv4 address, IPv6 address)";
        }
        leaf login-time {
          type yang:date-and-time;
          mandatory true;
          description
            "Time at the server at which the session was established.";
        }
        uses common-counters {
          description
            "Per-session counters.  Zero based with following reset
             behaviour:
               - at start of a session
               - when max value is reached";
        }
      }
    }

    container statistics {
      description
        "Statistical data pertaining to the NETCONF server.";

      leaf netconf-start-time {
        type yang:date-and-time;
        description
          "Date and time at which the management subsystem was
           started.";
      }
      leaf in-bad-hellos {
        type yang:zero-based-counter32;
        description
          "Number of sessions silently dropped because an
          invalid <hello> message was received.  This includes <hello>
          messages with a 'session-id' attribute, bad namespace, and
          bad capability declarations.";
      }
      leaf in-sessions {
        type yang:zero-based-counter32;
        description
          "Number of sessions started.  This counter is incremented
           when a <hello> message with a <session-id> is sent.

          'in-sessions' - 'in-bad-hellos' =
              'number of correctly started netconf sessions'";
      }
      leaf dropped-sessions {
        type yang:zero-based-counter32;
        description
          "Number of sessions that were abnormally terminated, e.g.,
           due to idle timeout or transport close.  This counter is not
           incremented when a session is properly closed by a
           <close-session> operation, or killed by a <kill-session>
           operation.";
      }
      uses common-counters {
        description
          "Global counters, accumulated from all sessions.
           Zero based with following reset behaviour:
             - re-initialization of NETCONF server
             - when max value is reached";
      }
    }
  }

  rpc get-schema {
    description
      "This operation is used to retrieve a schema from the
       NETCONF server.

       Positive Response:
         The NETCONF server returns the requested schema.

       Negative Response:
         If requested schema does not exist, the <error-tag> is
         'invalid-value'.

         If more than one schema matches the requested parameters, the
         <error-tag> is 'operation-failed', and <error-app-tag> is
         'data-not-unique'.";

    input {
      leaf identifier {
        type string;
        mandatory true;
        description
          "Identifier for the schema list entry.";
      }
      leaf version {
        type string;
        description
          "Version of the schema requested.  If this parameter is not
           present, and more than one version of the schema exists on
           the server, a 'data-not-unique' error is returned, as
           described above.";
      }
      leaf format {
        type identityref {
          base schema-format;
        }
        description
           "The data modeling language of the schema.  If this
            parameter is not present, and more than one formats of
            the schema exists on the server, a 'data-not-unique' error
            is returned, as described above.";
      }
    }
    output {
        anyxml data {
          description
            "Contains the schema content.";
      }
    }
  }
}
```

`<CODE ENDS>`

## 安全考虑安全考虑

本备忘录中定义的`YANG`模块设计为通过`NETCONF`协议[RFC4741](https://tools.ietf.org/html/rfc4741)访问。 最低的`NETCONF`层是安全传输层，实现安全传输的强制是`SSH`[RFC4742](https://tools.ietf.org/html/rfc4742)。

某些网络环境中，这个YANG模块中的一些可读数据节点可能被认为是敏感的或易受攻击的。 因此，控制对这些数据节点的读访问（例如，通过get，get-config或通知）是很重要的。

这些容器，列表节点和数据节点具有其特定的敏感性/漏洞：

`/netconf-state/sessions/session/username` : 包含可用于尝试向服务器进行身份验证的身份信息。

此用户名仅用于监控，不应用于其他用途，如访问控制，而不详细讨论此用户名的限制。 例如，服务器A和服务器B可能会报告相同的用户名，但这可能是针对不同的人。

## IANA考虑事项

本文档在“`IETF XML`注册表”中注册了一个URI。 遵循[RFC3688]()中的格式，已经注册了以下内容。

    URI: urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring
    Registrant Contact: The IESG.
    XML: N/A, the requested URI is an XML namespace.

本文档在“`YANG`模块名称”注册表中注册了一个模块。 遵循[RFC6020](https://tools.ietf.org/html/rfc6020)中的格式，已经注册了以下内容。


    name: ietf-netconf-monitoring
    namespace: urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring
    prefix: ncm
    reference: [RFC 6022](https://tools.ietf.org/html/rfc6022)

## 参考

### 规范性参考文献

- [ISO/IEC19757-2:2008](http://www.iso.org/iso/catalogue_detail.htm?csnumber=37605) ISO/IEC, "Document Schema Definition Language (DSDL) --Part 2: Regular-grammar-based validation -- RELAX NG",December 2008, <http://www.iso.org/iso/catalogue_detail.htm?csnumber=37605>.

- [RFC2119](https://tools.ietf.org/html/rfc2119)  Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", [BCP 14](https://tools.ietf.org/html/bcp14), RFC 2119, March 1997.

- [RFC4741](https://tools.ietf.org/html/rfc4741)  Enns, R., "NETCONF Configuration Protocol", RFC 4741,December 2006.

- [RFC4742](https://tools.ietf.org/html/rfc4742)  Wasserman, M. and T. Goddard, "Using the NETCONF Configuration Protocol over Secure SHell (SSH)", RFC 4742, December 2006.

- [RFC4743](https://tools.ietf.org/html/rfc4743)  Goddard, T., "Using NETCONF over the Simple Object Access Protocol (SOAP)", RFC 4743, December 2006.

- [RFC4744](https://tools.ietf.org/html/rfc4744)  Lear, E. and K. Crozier, "Using the NETCONF Protocol over the Blocks Extensible Exchange Protocol (BEEP)", RFC 4744, December 2006.

- [RFC5539](https://tools.ietf.org/html/rfc5539)  Badra, M., "NETCONF over Transport Layer Security (TLS)", RFC 5539, May 2009.

- [RFC5717](https://tools.ietf.org/html/rfc5717)  Lengyel, B. and M. Bjorklund, "Partial Lock Remote Procedure Call (RPC) for NETCONF", RFC 5717, December 2009.

- [RFC6020](https://tools.ietf.org/html/rfc6020)  Bjorklund, M., Ed., "YANG - A Data Modeling Language for the Network Configuration Protocol (NETCONF)", October 2010.

- [RFC6021](https://tools.ietf.org/html/rfc6021)  Schoenwaelder, J., Ed., "Common YANG Data Types", October 2010.

- [XML-NAMES](http://www.w3.org/TR/2009/REC-xml-names-20091208) Hollander, D., Tobin, R., Thompson, H., Bray, T., and A. Layman, "Namespaces in XML 1.0 (Third Edition)", World Wide Web Consortium Recommendation REC-xml-names-20091208, December 2009, <http://www.w3.org/TR/2009/REC-xml-names-20091208>.

- [xmlschema-1](http://www.w3.org/TR/xmlschema-1) Biron, Paul V. and Ashok. Malhotra, "XML Schema Part 1: Structures Second Edition W3C Recommendation 28 October 2004", October 2004, <http://www.w3.org/TR/xmlschema-1>.

### 信息参考

- [RFC0959](https://tools.ietf.org/html/rfc0959)  Postel, J. and J. Reynolds, "File Transfer Protocol", STD 9, RFC 959, October 1985.

- [RFC2616](https://tools.ietf.org/html/rfc2616)  Fielding, R., Gettys, J., Mogul, J., Frystyk, H., Masinter, L., Leach, P., and T. Berners-Lee, "Hypertext Transfer Protocol -- HTTP/1.1", RFC 2616, June 1999.

- [RFC2818](https://tools.ietf.org/html/rfc2818)  Rescorla, E., "HTTP Over TLS", RFC 2818, May 2000.

- [RFC3688](https://tools.ietf.org/html/rfc3688)  Mealling, M., "The IETF XML Registry", [BCP 81](https://tools.ietf.org/html/bcp81), RFC 3688, January 2004.
