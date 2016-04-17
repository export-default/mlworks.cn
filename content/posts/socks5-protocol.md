+++
author = "Ryan Zhao"
categories = ['Network']
date = "2015-05-05T15:25:48+08:00"
description = "关于SOCKS"
keywords = ['SOCKS 5', 'SOCKS协议', '网络协议']
mathjax = false
tags = ['Network']
title = "SOCKS 5 协议浅析"
alias = "socks5-protocol"
+++

最近研究了一下SOCKS 5协议，折腾了一些SOCKS 5服务器，下文是我对SOCKS 5的一些粗浅理解。

<!--more-->

## 协议

SOCKS 5是一种代理协议，工作在传输层与应用层之间，因此仅能代理TCP与UDP流量，无法代理诸如ICMP等网络层的流量。SOCKS 5是SOCKS 4协议的改进版本，增加了对认证和对UDP流量的支持。下面简要介绍一些SOCKS 5协议。一下讨论均不涉及具体的数据格式，感兴趣的话可以对照着相关RFC阅读（[RFC 1928](http://www.ietf.org/rfc/rfc1928.txt) & [RFC 1929](http://www.ietf.org/rfc/rfc1929.txt)）。

### TCP 客户端

#### 握手

client 首先连接到 SOCKS 服务器， 发送其协议版本号（4或者5，对应SOCKS 4和SOCKS 5协议），及其**支持的认证方法**。Server 端返回协议版本号与**选定的认证方法**。

握手完毕，下一步进入认证协商过程。

#### 认证

client 根据 Server 选定的认证方法，进入认证方法相关的协商步骤。此处描述 RFC 1929定义的  Username/Password 认证过程（此认证过程中，密码为明文传送）。

client 发送 用户名、密码给 Server，Server 返回成功或失败。

若认证成功，则进入请求阶段。

#### 请求

SOCKS 5协议中有三种类型的请求：

1.CONNECT

最常用的请求，即请求Server发起连接到**目标主机**、**目标端口**的代理。若成功，该连接中的后续流量都会被转发到目标主机的目标端口。

2.BIND

CONNECT请求是client发起对目标主机的连接，BIND请求用于目标主机连接Client。如FTP协议，控制连接由client发起，而数据传输连接则由FTP服务器发起。其流程如下：

- client随BIND请求，发送其要绑定的地址和端口。
- Server返回其创建的监听端口的地址和端口。
- Server创建的监听端口有连接后，返回该连接的源地址和端口。
- Server端将上述连接中的流量，发送给client的监听端口。

3.UDP ASSOCIATE

该请求用于辅助UDP中继，发起请求时，发送UDP的目标主机与目标端口给Server（若当前不知晓目标地址与端口，可也不填）。随后，Server返回已个地址和端口，将UDP报文发送到该地址端口，即可完成中继。

需要注意的时，发送该请求的连接的若关闭，则相关的UDP中继也会关闭。因此，最好在需要使用UDP代理时，一直保持该连接。

### UDP 客户端

UDP客户端需要首先发起UDP ASSOCIATE请求，得到UDP中继地址和端口后，才能发送UDP报文。此处，报文需要加上**请求头**后，才可以发给中继服务器。请求头中包含了一些必要的请求，具体可参见RFC。

