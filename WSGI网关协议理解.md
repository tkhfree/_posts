---
title: WSGI网关协议理解
date: 2020-05-07 15:46:46
tags: python
---



**网关协议**是一个链接浏览器与服务器端的框架协议

B端发送request请求到服务器，服务器针对请求在路径文件夹内返回html文件给浏览器展示。

对于静态请求可以用nginx作为server，可以处理高并发。

对于动态请求可以用apache作为server，处理这些动态请求需要编写专门的动态application来处理数据返回给server。

**问题：**

- 需要花费大量的精力去处理server与application之间的通讯和传输。
- 如果换一个server，例如apache换成nginx，则applicaiton所写的接口全都不能用.
- 一种动态请求处理函数只针对一种情况，对于每种情况都写请求函数则存在大量的代码复用。

**解决：**

- 使用一个web app开发框架，基于网关协议标准实现。则不管server或者app怎么变换改动都遵循同一个网关标准。
- CGI网关协议标准     FastCGI     
- WSGI网关协议标准一开始是专门开发出来解决python后端同server接驳的框架，有了这个标准就不用考虑server是怎么运行的，只要application符合这个标准接口去开发，则可以移植到任何支持WSGI的server端。
- keystone就是一个python开源的基于WSGI接口标准的application



**WSGI中间件**

WSGI中间件就是处于application和server之间的middleware。有点像nginx的正向代理/反向代理。中间件对于application是server，对于服务器是应用程序。中间件可以做一些缓存的功能，同时降低了server与app的耦合。

*<font size=2>正向代理：把nginx作为客户端去访问server，需要客户端设置代理服务器的ip、port等。</font>*

*<font size=2>反向代理：把nginx作为server端接收Browser的请求，并根据设置把请求分发到真正的服务器，对于客户端来说，nginx 就是一个server。</font>*

WSGI: WSGI server -- middleware -- WSGI application

- 建立socket，监听端口，等待request
- 解析信息，放到环境变量environ中，调用绑定的handler处理请求
- handler解析请求，将method、path放入environ中
- handler将一些服务器端信息也放入environ中
- WSGI handler调用WSGI application ，将environ和回调函数传给WSGI app
- app将response包含head/body/status等传给handler
- handler 通过socke传给客户端

所以从以上过程路我们知道application接受两个参数：**environ**和**start_response**。