---
title: PacketIn/PacketOut代码思路
toc: true
thumbnail: 'https://pic.downk.cc/item/5eda03efc2a9a83be5241038.jpg'
comments: true
tags:
  - null
date: 2020-09-09 16:53:07
urlname:
categories:
---

## Packet-In

使用Packet-In消息的目的是为了将到达OpenFlow交换机的数据包发送至OpenFlow控制器。以下2种情况即可发送Packet-In消息。

> 不存在与流表项一致的项目时（Table-miss），OFPR_NO_MATCH
> 匹配的流表项中记载的行动为“发送至OpenFlow控制器”时，OFPR_ACTION

发送Packet-In消息时OpenFlow交换机分为两种情况，一种是缓存数据包，一种是不缓存数据包。如果不通过OpenFlow交换机缓存数据包，那么Packet-In消息的buffer_id字段设置为-1，将整个数据包发送至OpenFlow控制器。
如果通过OpenFlow交换机缓存数据包，那么以通过SET_CONFIG消息设置的miss_send_len为最大值的数据包数据将发送至OpenFlow控制器。
miss_send_len的默认值为128。未实施SET_CONFIG消息的交换时，使用该默认值。

|           |        |                                          |
| :-------- | :----- | :--------------------------------------- |
| 字段      | 比特数 | 内容                                     |
| buffer_id | 32     | 表示OpenFlow交换机中保存的数据包的缓存id |
| Total_len | 16     | 帧的长度                                 |
| in_port   | 16     | 接受帧的端口                             |
| reason    | 8      | 发送Packet-in消息的原因                  |
| pad       | 8      | 用于调整对齐的填充                       |
| data      | 任意   | 包含以太网帧的数据时使用的字段。         |

控制器通过grpc与交换机进行交互，因此在使用proto生成runtime库之后，在server端

## Packet-out

Packet-Out消息是从OpenFlow控制器向OpenFlow交换机发送的消息，是包含数据包发送命令的消息”。
若OpenFlow交换机的缓存中已存在数据包，而OpenFlow控制器发出“发送该数据包”的命令时，该消息指定了表示相应数据包的buffer_id。使用Packet-Out消息还可将OpenFlow控制器创建的数据包发送至OpenFlow交换机。此时，buffer_id置为-1，在Packet-Out消息的最后添加数据包数据。

![](https://pic.downk.cc/item/5f58a602160a154a67071313.jpg)

| 字段        | 比特数 | 内容                                     |
| ----------- | ------ | ---------------------------------------- |
| buffer_id   | 32     | 表示OpenFlow交换机中保存的数据包的缓存id |
| in_port     | 16     | 数据包的输入端口                         |
| actions_len | 16     | 行动信息的长度                           |

## 流程

控制器与OpenFlow交换机在连接建立过程中会存在拓扑发现的环节，该环节会密集出现Packe-in/out消息，其交互流程如下：

![](https://pic.downk.cc/item/5f58a655160a154a6707714c.jpg)

1、 [SDN控制器](https://www.baidu.com/s?wd=SDN控制器&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)通过构造Packet-out消息，向交换机s1的三个端口分别发送上图所示的[LLDP](https://baike.baidu.com/item/LLDP/6849522?fr=aladdin)数据包。
2、 控制器向交换机s1下发流表，流表规则为：将从Controller端口收到的LLDP数据包从规定端口发送出去。
3、 控制器向交换机s2下发流表，流表规则为：将从非Controller接收到LLDP数据包发送给控制器。
4、 当LLDP数据包到达交换机s2，会触发Packet-in消息发往控制器。控制器通过解析LLDP数据包，得到链路的源交换机，源接口（s1,port1）。通过收到的Packet-in消息知道目的交换机(s2)。
5、 同理，当SDN控制器向交换机s2发送Packet-out消息时，可以得知链路源交换机，源接口（s2,port3)。通过收到的Packet-in消息知道目的交换机(s1)。如此，控制器便发现了s1与s2之间的完整链路。
对于存在多个交换机的网络，上述分析过程一样成立。

### example

交换机存储的是MAC地址和端口的对应关系，同一子网主机传输报文通过ip地址，交换机第一次传输的时候不知道IP与MAC的对应关系，因此不知道从哪个端口转发，因此交换机会往所有端口进行转发数据包，而IP和MAC对应的主机就会回复，因此交换机就知道哪个端口对应的主机MAC地址，把这个关系加入到表中，这个过程就是交换机的MAC地址自学习。

一般监控ARP数据包学习MAC地址，将ARP数据包通过Packet-in消息发送给OpenFlow控制器，可以在不过度加重OpenFlow控制器负载的情况下，实现MAC地址学习。

假如PC-a发送数据给PC-b，但是它的ARP表里没有PC-b，所以它需要发送ARP包获取ARP回应以存储到ARP表中。

PC-a发送ARP请求，OpenFlow交换机接收到该请求，检索流表，流表项的匹配字段是入端口，action是上报控制器，可以找找流表项的具体实现代码。

流表的action的输出端口为controller，也就是上报给控制器。

流表项匹配成功后，OpenFlow交换机执行通过Packet-in消息向控制器转发ARP请求帧的action，而控制器要做两个处理，一个是接受Packet-in，二是对Packet-in解码。

为了将来PC-a的ARP请求广播到OpenFlow网络内



ONOS对于Packet-in/out消息的定义

> 我们来看怎么样完成packet-in和packet-out的操作。这两个操作跟OpenFlow是类似的。
>
> 我们看右边，在OpenFlow里面我们也会用到图中的第一块，Packet Service，不同的是在下面，对接的是PI框架的Packet Provider，因为在P4里面，数据包的格式是我们自定义的，所以在这里借助我们的Interpreter进行数据包的解析。解析之后，生成Inbound/Outbound Packet，它们都是ONOS中原有的对packet-in/out的抽象。
>
> 接着，这里借助P4 Runtime设备驱动将其转换成PI Packet Operation，它是PI框架中对包操作的抽象。
>
> 最后，再通过南向插件与设备交互，在这里同样借助P4Info完成PI框架抽象对象与通信报文的转换。
>
> 在ONOS的源码中，现在已经有了一个名为p4-tutorial的示例应用，在onos/apps目录下

![](https://pic.downk.cc/item/5f58a350160a154a6704771b.png)

