---
title: Neutron的基础知识-iptables
date: 2020-05-26 13:47:19
tags: 
- Neutron 
- OpenStack
comments: true
toc: true
mathjax: true
urlname:
categories:
thumbnail:
---

iptables是防火墙功能，是处于网络边界的控制网络访问的功能，分为硬件防火墙和软件防火墙。

目前市面上比较常见的有3、4层的防火墙，叫网络层的防火墙，还有7层的防火墙，其实是代理层的网关。

对于TCP/IP的七层模型来讲，我们知道第三层是网络层，三层的防火墙会在这层对源地址和目标地址进行检测。但是对于七层的防火墙，不管你源端口或者目标端口，源地址或者目标地址是什么，都将对你所有的东西进行检查。所以，对于设计原理来讲，七层防火墙更加安全，但是这却带来了效率更低。所以市面上通常的防火墙方案，都是两者结合的。而又由于我们都需要从防火墙所控制的这个口来访问，所以防火墙的工作效率就成了用户能够访问数据多少的一个最重要的控制，配置的不好甚至有可能成为流量的瓶颈。

​    防火墙，其实说白了讲，就是用于实现Linux下访问控制的功能的，它分为硬件的或者软件的防火墙两种。无论是在哪个网络中，防火墙工作的地方一定是在网络的边缘。而我们的任务就是需要去定义到底防火墙如何工作，这就是防火墙的策略，规则，以达到让它对出入网络的IP、数据进行检测。

​     目前市面上比较常见的有3、4层的防火墙，叫网络层的防火墙，还有7层的防火墙，其实是代理层的网关。

​     对于TCP/IP的七层模型来讲，我们知道第三层是网络层，三层的防火墙会在这层对源地址和目标地址进行检测。但是对于七层的防火墙，不管你源端口或者目标端口，源地址或者目标地址是什么，都将对你所有的东西进行检查。所以，对于设计原理来讲，七层防火墙更加安全，但是这却带来了效率更低。所以市面上通常的防火墙方案，都是两者结合的。而又由于我们都需要从防火墙所控制的这个口来访问，所以防火墙的工作效率就成了用户能够访问数据多少的一个最重要的控制，配置的不好甚至有可能成为流量的瓶颈。

1.iptables的发展:

​     iptables的前身叫ipfirewall （内核1.x时代）,这是一个作者从freeBSD上移植过来的，能够工作在内核当中的，对数据包进行检测的一款简易访问控制工具。但是ipfirewall工作功能极其有限(它需要将所有的规则都放进内核当中，这样规则才能够运行起来，而放进内核，这个做法一般是极其困难的)。当内核发展到2.x系列的时候，软件更名为ipchains，它可以定义多条规则，将他们串起来，共同发挥作用，而现在，它叫做iptables，可以将规则组成一个列表，实现绝对详细的访问控制功能。

​     他们都是工作在用户空间中，定义规则的工具，本身并不算是防火墙。它们定义的规则，可以让在内核空间当中的netfilter来读取，并且实现让防火墙工作。而放入内核的地方必须要是特定的位置，必须是tcp/ip的协议栈经过的地方。而这个tcp/ip协议栈必须经过的地方，可以实现读取规则的地方就叫做 netfilter.(网络过滤器)

  作者一共在内核空间中选择了5个位置，
  1.内核空间中：从一个网络接口进来，到另一个网络接口去的
  2.数据包从内核流入用户空间的
  3.数据包从用户空间流出的
  4.进入/离开本机的外网接口
  5.进入/离开本机的内网接口

2.iptables的工作机制

​     从上面的发展我们知道了作者选择了5个位置，来作为控制的地方，但是你有没有发现，其实前三个位置已经基本上能将路径彻底封锁了，但是为什么已经在进出的口设置了关卡之后还要在内部卡呢？ 由于数据包尚未进行路由决策，还不知道数据要走向哪里，所以在进出口是没办法实现数据过滤的。所以要在内核空间里设置转发的关卡，进入用户空间的关卡，从用户空间出去的关卡。那么，既然他们没什么用，那我们为什么还要放置他们呢？因为我们在做NAT和DNAT的时候，目标地址转换必须在路由之前转换。所以我们必须在外网而后内网的接口处进行设置关卡。     

​      这五个位置也被称为五个钩子函数（hook functions）,也叫五个规则链。
​          1.PREROUTING (路由前)
​          2.INPUT (数据包流入口)
​          3.FORWARD (转发管卡)
​          4.OUTPUT(数据包出口)
​          5.POSTROUTING（路由后）
​     这是NetFilter规定的五个规则链，任何一个数据包，只要经过本机，必将经过这五个链中的其中一个链。    

3.防火墙的策略

​     防火墙策略一般分为两种，一种叫“通”策略，一种叫“堵”策略，通策略，默认门是关着的，必须要定义谁能进。堵策略则是，大门是洞开的，但是你必须有身份认证，否则不能进。所以我们要定义，让进来的进来，让出去的出去，所以通，是要全通，而堵，则是要选择。当我们定义的策略的时候，要分别定义多条功能，其中：定义数据包中允许或者不允许的策略，filter过滤的功能，而定义地址转换的功能的则是nat选项。为了让这些功能交替工作，我们制定出了“表”这个定义，来定义、区分各种不同的工作功能和处理方式。

​     我们现在用的比较多个功能有3个：
​          1.filter 定义允许或者不允许的
​          2.nat 定义地址转换的
​          3.mangle功能:修改报文原数据

​     我们修改报文原数据就是来修改TTL的。能够实现将数据包的元数据拆开，在里面做标记/修改内容的。而防火墙标记，其实就是靠mangle来实现的。

小扩展:
     对于filter来讲一般只能做在3个链上：INPUT ，FORWARD ，OUTPUT
     对于nat来讲一般也只能做在3个链上：PREROUTING ，OUTPUT ，POSTROUTING
     而mangle则是5个链都可以做：PREROUTING，INPUT，FORWARD，OUTPUT，POSTROUTING

​     iptables/netfilter（这款软件）是工作在用户空间的，它可以让规则进行生效的，本身不是一种服务，而且规则是立即生效的。而我们iptables现在被做成了一个服务，可以进行启动，停止的。启动，则将规则直接生效，停止，则将规则撤销。
​     iptables还支持自己定义链。但是自己定义的链，必须是跟某种特定的链关联起来的。在一个关卡设定，指定当有数据的时候专门去找某个特定的链来处理，当那个链处理完之后，再返回。接着在特定的链中继续检查。

注意：规则的次序非常关键，谁的规则越严格，应该放的越靠前，而检查规则的时候，是按照从上往下的方式进行检查的。

[参考资料](https://www.aboutyun.com/forum.php?mod=viewthread&tid=9649&highlight=%BF%AA%B7%A2%C8%CB%D4%B1%B1%D8%B6%C1openstack%CD%F8%C2%E7%BB%F9%B4%A1)