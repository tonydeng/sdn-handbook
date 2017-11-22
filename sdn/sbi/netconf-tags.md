# NETCONF请求和响应中的标签

## `]]>]]>`

`]]>]]>`标签在`NETCONF`服务器和客户端应用程序发送的每个`XML`文档的末尾。

在使用SSH协议作为NETCONF通讯协议时，NETCONF的服务器和客户端在每一次请求中都需要在末尾加上该标签，比如`</ hello>`、`</ rpc>`、`</ rpc-reply>`标记。

例子：

```xml
<hello>
    <!-- child tag elements included by client application or NETCONF server -->
</hello>
]]>]]>
```

```xml
<rpc [attributes]>
    <!-- tag elements in a request from a client application -->
</rpc>
]]>]]>
```

```xml
<rpc-reply xmlns="URN" xmlns:junos="URL">
    <!-- tag elements in the response from the NETCONF server -->
</rpc-reply>
]]>]]>
```

> 可以在[RFC 6242 Using the NETCONF Protocol over Secure Shell (SSH)](http://www.ietf.org/rfc/rfc6242.txt)查看此约束。

## `<data>`

用于NETCONF服务器针对`<get>`和`<get-config>`两类`request`来返回的配置数据和设备信息响应。

> 注意：默认情况下，NETCONF服务器将返回格式为`XML tag elements`的配置数据。如果客户端应用程序在`<get>`请求中请求不同的格式，那么包含在`<data>`元素中的配置数据可能会有所不同。

`<configuration>` - 包含配置标签元素。它是`XML API`中的顶级标记元素。

例子：

```xml
<rpc-reply message-id="101"
                xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <data>
        <top xmlns="http://example.com/schema/1.2/config">
            <users>
                <user>
                    <name>root</name>
                    <company-info>
                        <dept>1</dept>
                        <id>1</id>
                    </company-info>
                </user>
                <user>
                    <name>fred</name>
                    <company-info>
                        <id>2</id>
                    </company-info>
                </user>
            </users>
        </top>
    </data>
</rpc-reply>
]]>]]>
```

## `<hello>`

该标签告知了NETCONF服务器支持哪些在`NETCONF`规范中定义的`operations`或`capabilities`。

客户端应用程序必须在NETCONF会话期间在任何其他标记元素之前发出`<hello>`标记元素，并且不得多次发出它。

- `<capabilities>` - 包含一个或多个`<capabilities>`标签，它们共同指定一组支持的NETCONF操作。

- `<capability>` - 指定NETCONF规范或供应商定义的能力的统一资源标识符（`URI`）。NETCONF规范中的每个功能由统一资源名称（`URN`）表示。供应商定义的功能由`URN`或`URL`表示。

- `<session-id>` - （仅由NETCONF服务器生成）指定会话的NETCONF服务器的UNIX进程ID（PID）

例子：

```xml
<!-- emitted by a client application -->
<hello>
    <capabilities>
        <capability>URI</capability>
    </capabilities>
</hello>
]]>]]>


<!-- emitted by the NETCONF server -->
<hello>
    <capabilities>
        <capability>URI</capability>
    </capabilities>
    <session-id>session-identifier</session-id>
</hello>
]]>]]>
```

```xml
<hello xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
   <capabilities>
     <capability>
       urn:ietf:params:netconf:base:1.1
     </capability>
     <capability>
       urn:ietf:params:netconf:capability:startup:1.0
     </capability>
     <capability>
       http://example.net/router/2.3/myfeature
     </capability>
   </capabilities>
   <session-id>4</session-id>
 </hello>

```

> 关于`<hello>`的详细介绍，请查看[RFC6241 - Network Configuration Protocol (NETCONF) - Capabilities Exchange](https://tools.ietf.org/html/rfc6241#section-8.1)，[RFC6242 - Using the NETCONF Protocol over Secure Shell (SSH) - Starting NETCONF over SSH](https://tools.ietf.org/html/rfc6242#section-3)



## `<rpc>`

用来封装NETCONF客户端发送给服务端的NETCONF请求，

> 在`<rpc>`中可以有多个attribute，其中`message-id`属性是必须存在。它的值由RPC调用者来维护，格式是字符串，通常是一个递增的整数。RPC的接收者不会解码或解释这个字符串，但会将它保存并在接下来的`<rpc-reply>`将这个`message-id`附加上去。调用方必须确保`message-id`值的规范化。

例子：

调用一个名为`<my-own-method>`的方法，该方法有两个参数：`<my-first-parameter>`，其值为`14`，另一个参数的值为`fred`：

```xml
<rpc message-id="101"
     xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <my-own-method xmlns="http://example.net/me/my-own/1.0">
        <my-first-parameter>14</my-first-parameter>
        <another-parameter>fred</another-parameter>
    </my-own-method>
</rpc>
]]>]]>
```
调用`<zip-code>`参数为`27606-0100`的`<rock-the-house>`方法：

```xml
<rpc message-id="101"
    xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <rock-the-house xmlns="http://example.net/rock/1.0">
        <zip-code>27606-0100</zip-code>
    </rock-the-house>
</rpc>
]]>]]>
```
调用不带参数的`NETCONF` `<get>`方法：

```xml
<rpc message-id="101"
    xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <get/>
</rpc>
]]>]]>
```

>  关于`<rpc>`的详细介绍，请查看[RFC6241 - Network Configuration Protocol (NETCONF) - \<rpc\> Element](https://tools.ietf.org/html/rfc6241#section-4.1)


## `<rpc-reply>`

用来响应`<rpc>`消息。

`<rpc-reply>`必须有一个`message-id`，它与之前`<rpc>`请求中的`message-id`应该是一致的。

`NETCONF`服务器还必须在`<rpc-reply>`中返回`<rpc>`请求中未修改的元素以及属性。

`response`的`<rpc-reply>`元素中应该包含为的一个或多个子元素。


例子：

以下`<rpc>`元素调用`NETCONF` `<get>`方法，并包含一个名为`user-id`的附加属性。返回的`<rpc-reply>`元素返回`user-id`属性以及请求的内容。
> 请注意，`user-id`属性不在`NETCONF`命名空间中。

```xml
<rpc message-id="101"
    xmlns="urn:ietf:params:xml:ns:netconf:base:1.0"
    xmlns:ex="http://example.net/content/1.0" ex:user-id="fred">
    <get/>
</rpc>
]]>]]>

<rpc-reply message-id="101"
    xmlns="urn:ietf:params:xml:ns:netconf:base:1.0"
    xmlns:ex="http://example.net/content/1.0"
  ex:user-id="fred">
    <data>
        <!-- contents here... -->
    </data>
</rpc-reply>
 ]]>]]>
```

>  关于`<rpc-reply>`的详细介绍，请查看[RFC6241 - Network Configuration Protocol (NETCONF) - <rpc-reply> Element](https://tools.ietf.org/html/rfc6241#section-4.2)


## `<rpc-error>`

如果在处理`<rpc>`请求期间发生错误，`<rpc-error>`元素将在`<rpc-reply>`消息中发送。

如果服务器在处理`<rpc>`请求期间遇到多个错误，则`<rpc-reply>`可能包含多个`<rpc-error>`元素。 但是，如果请求包含多个错误，则服务器不需要检测或报告多个`<rpc-error>`元素。

服务器不需要按照特定的顺序检查特定的错误条件。 如果在处理过程中出现任何错误，服务器务必返回一个`<rpc-error>`元素。

服务器绝对不能在客户端没有足够访问权限的`<rpc-error>`元素中返回应用级或数据模型特定的错误信息。

`<rpc-error>`元素包含以下信息：

- `error-type`：定义错误发生的概念层，下列类型之一。

    1. `transport` (layer: Secure Transport)
    1. `rpc` (layer: Messages)
    1. `protocol` (layer: Operations)
    1. `application` (layer: Content)

- `error-tag`：包含识别错误条件的字符串。 相关的值，请参阅[RFC6261 - Network Configuration Protocol (NETCONF) - 附录A](https://tools.ietf.org/html/rfc6241#appendix-A)。

- `error-severity`：包含标识错误严重性的字符串，由设备确定。 下列之一：

    1. `error`
    1. `warning`
        > 请注意，本文档中没有使用`warning`枚举定义的<error-tag>值。 这是保留供将来使用。

- `error-app-tag`：包含标识数据模型特定或特定于实现的错误条件（如果存在）的字符串。 如果没有适当的应用程序错误标签可以与特定的错误条件相关联，则这个元素将不存在。 如果数据模型特定的和特定于实现的`error-app-tag`都存在，那么服务器必须使用数据模型特定的值。
- `error-path`：包含绝对[XPath](https://tools.ietf.org/html/rfc6241#ref-W3C.REC-xpath-19991116)表达式，用于标识`<rpc-error>`元素中报告的错误关联的节点的元素路径。 如果没有适当的有效载荷元素或数据存储节点可以与特定的错误条件相关联，则该元素将不存在。

    - `XPath`表达式在以下上下文中进行解释。
        - 名称空间声明的集合是`<rpc-error>`元素上的范围声明。
        - 变量绑定的集合是空的。
        - 函数库是核心函数库。
    - 上下文节点取决于与所报告的错误相关联的节点：
        - 如果`payload`元素可以与错误关联，则上下文节点是`rpc`请求的文档节点（即`<rpc>`元素）。
        - 否则，上下文节点是所有数据模型的根，即所有数据模型中具有作为子节点的顶级节点的节点。
- `error-message`：包含一个适合人类阅读的字符串，用于描述错误情况。 如果没有为特定错误条件提供适当的消息，则该元素将不存在。 这个元素应该包含一个在[[W3C.REC-xml-20001006]](https://tools.ietf.org/html/rfc6241#ref-W3C.REC-xml-20001006)中定义并在[[RFC3470]](https://tools.ietf.org/html/rfc3470)中讨论的`"xml：lang"`属性。
- `error-info`：包含协议或数据模型特定的错误内容。 如果没有为特定的错误条件提供这样的错误内容，则该元素将不存在。 [RFC6261 - Network Configuration Protocol (NETCONF) - 附录A](https://tools.ietf.org/html/rfc6241#appendix-A)中的列表定义了每个错误的任何强制错误信息内容。 在任何协议规定的内容之后，数据模型定义可能会要求某些应用层错误信息包含在错误信息容器中。 一个实现可以包含额外的元素来提供扩展和/或实现特定的调试信息。

例子：

```xml
<rpc-reply xmlns="URN" xmlns:junos="URL">
    <rpc-error>
        <error-severity>error-severity</error-severity>
        <error-path>error-path</error-path>
        <error-message>error-message</error-message>
        <error-info>...</error-info>
    </rpc-error>
</rpc-reply>
]]>]]>
```

如果接收到<rpc>元素而没有`message-id`属性，则返回错误。
> 请注意，只有在这种情况下，`NETCONF`对等方可以省略`<rpc-reply>`元素中的`message-id`属性。

```xml
<rpc xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <get-config>
    <source>
      <running/>
    </source>
  </get-config>
</rpc>
]]>]]>

<rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <rpc-error>
    <error-type>rpc</error-type>
    <error-tag>missing-attribute</error-tag>
    <error-severity>error</error-severity>
    <error-info>
      <bad-attribute>message-id</bad-attribute>
      <bad-element>rpc</bad-element>
    </error-info>
  </rpc-error>
</rpc-reply>
]]>]]>
```

> [RFC6241 - Network Configuration Protocol (NETCONF) - <rpc-error> Element](https://tools.ietf.org/html/rfc6241#section-4.3)

## `<error-info>`

提供导致NETCONF服务器错误或警告的事件会附加在`<rpc-error>`标签内告知请求者。

> `<bad-element>` - 识别发生错误或警告时正在处理的命令或配置语句。对于相关的配置声明，会在`<rpc-error>`标记元素中包含的`<error-path>`标记元素指定语句的父层次结构级别。

例子：

```xml
<rpc-reply xmlns="URN" xmlns:junos="URL">
    <rpc-error>
        <error-info>
            <bad-element>command-or-statement</bad-element>
        </error-info>
    </rpc-error>
</rpc-reply>
]]>]]>
```

```xml
<rpc xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <get-config>
    <source>
      <running/>
    </source>
  </get-config>
</rpc>

<rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <rpc-error>
    <error-type>rpc</error-type>
    <error-tag>missing-attribute</error-tag>
    <error-severity>error</error-severity>
    <error-info>
      <bad-attribute>message-id</bad-attribute>
      <bad-element>rpc</bad-element>
    </error-info>
  </rpc-error>
</rpc-reply>
```

## `</ok>`

如果在处理`<rpc>`请求期间没有发生错误或警告，并且操作没有返回任何数据，则`<rpc-reply>`消息中会发送`<ok>`元素。

例子：

```xml
<rpc-reply message-id="101"
    xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <ok/>
</rpc-reply>
]]>]]>
```

## `<target>`
