+++
author = "Ryan Zhao"
categories = ["Linux"]
date = "2015-03-12T16:47:16+08:00"
description = "Redhat与Centos7中的防火墙Firewalld使用简介"
keywords = ['Linux','Firewalld','防火墙','Centos','Redhat','iptables']
mathjax = false
tags = ['Linux','Firewalld','Centos','RedHat']
title = "Redhat 7与Centos 7中的防火墙Firewalld"

+++

Centos 7中默认使用了firewalld作为防火墙, 其底层仍旧是基于iptables的，但还是有很多不同的地方。下面的内容截取了一些RedHat 7 官方文档中关于firewalld的介绍，原始文档请[参考这里](https://access.redhat.com/documentation/zh-CN/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Using_Firewalls.html)。

## 简介

动态防火墙后台程序 firewalld 提供了一个 动态管理的防火墙，用以支持网络 “zones” ，以分配对一个网络及其相关链接和界面一定程度的信任。它具备对 IPv4 和 IPv6 防火墙设置的支持。它支持以太网桥，并有分离运行时间和永久性配置选择。它还具备一个通向服务或者应用程序以直接增加防火墙规则的接口。

firewalld 提供的是动态的防火墙服务，而非静态的。因此配置的改变可以随时随地立刻执行，不再需要保存或者执行这些改变。也由于不需要重启防火墙，网络连接将不会发生意外中断。

<!--more-->

客户端命令```firewalld```可以用来修改firewalld的永久配置或者运行时配置。firewalld的配置文件保存在```/usr/lib/firewalld/``` 和 ```/etc/firewalld/```中，其中前者是默认的配置，请不要修改。可以在```/etc/firewalld/```中编辑自己的配置，firewalld优先使用```/etc/firewalld/```中的配置。

## Firewalld 与 Iptables

firewalld 和 iptables service 之间最本质的不同是：

1. iptables service 在 ```/etc/sysconfig/iptables``` 中储存配置，而 firewalld 将配置储存在 ```/usr/lib/firewalld/``` 和 ```/etc/firewalld/``` 中的各种 XML 文件里，。要注意，当 firewalld 在Red Hat Enterprise Linux上安装失败时， ```/etc/sysconfig/iptables``` 文件就不存在。
2. 使用 iptables service，每一个单独更改意味着清除所有旧有的规则和从 ```/etc/sysconfig/iptables```里读取所有新的规则，然而使用 firewalld 却不会再创建任何新的规则；仅仅运行规则中的不同之处。因此，firewalld 可以在运行时间内，改变设置而不丢失现行连接。
3. 使用 iptables tool 与内核包过滤对话也是如此。

![防火墙堆栈](https://access.redhat.com/documentation/zh-CN/Red_Hat_Enterprise_Linux/7/html/Security_Guide/images/firewall_stack.png)

## 对网络区的理解
基于用户对网络中设备和交通所给与的信任程度，防火墙可以用来将网络分割成不同的区域。

预设定的区域有：

drop（丢弃）
: 任何接收的网络数据包都被丢弃，没有任何回复。仅能有发送出去的网络连接。

block（限制）
: 任何接收的网络连接都被 IPv4 的 icmp-host-prohibited 信息和 IPv6 的 icmp6-adm-prohibited 信息所拒绝。

public（公共）
: 在公共区域内使用，不能相信网络内的其他计算机不会对您的计算机造成危害，只能接收经过选取的连接。

external（外部）
: 特别是为路由器启用了伪装功能的外部网。您不能信任来自网络的其他计算，不能相信它们不会对您的计算机造成危害，只能接收经过选择的连接。

dmz（非军事区）
: 用于您的非军事区内的电脑，此区域内可公开访问，可以有限地进入您的内部网络，仅仅接收经过选择的连接。

work（工作）
: 用于工作区。您可以基本相信网络内的其他电脑不会危害您的电脑。仅仅接收经过选择的连接。

home（家庭）
: 用于家庭网络。您可以基本信任网络内的其他计算机不会危害您的计算机。仅仅接收经过选择的连接。

internal（内部）
: 用于内部网络。您可以基本上信任网络内的其他计算机不会威胁您的计算机。仅仅接受经过选择的连接。

trusted（信任）
: 可接受所有的网络连接。

## 预定义的服务

一项服务可以是本地和目的地端口的列表，如果服务被允许的话，也可以是一系列自动加载的防火墙辅助模块。预先定义的服务的使用，让客户更容易被允许或者被禁止进入服务。与对开放端口或者值域，或者端口截然不同，使用预先定义服务，或者客户限定服务，或许能够让管理更容易。 firewalld.service(5) 中的手册页描述了服务配置的选择和通用文件信息。服务通过单个的 XML 配置文件来指定，这些配置文件则按以下格式命名：service-name.xml。

要使用命令行列出默认的预先定义服务，以 root 身份执行以下命令：

```
~]# ls /usr/lib/firewalld/services/
```
请勿编辑```/usr/lib/firewalld/services/``` ，只有 ```/etc/firewalld/services/``` 的文件可以被编辑。

如果您希望增加或者改变服务， ```/usr/lib/firewalld/services/``` 文件可以作为模板使用。
```
cp /usr/lib/firewalld/services/[service].xml /etc/firewalld/services/[service].xml
```
然后您可以编辑最近创建的文件。firewalld 优先使用 ```/etc/firewalld/services/``` 里的文件，如果一份文件被删除且服务被重新加载后，会切换到 ```/usr/lib/firewalld/services/```。

## 命令行接口的使用

输入以下命令，得到 firewalld 的状态的文本显示：

```bash
~]$ firewall-cmd --state
```
查看活动区域的列别，并附带一个目前分配给它们的接口列表：

```bash
~]$ firewall-cmd --get-active-zones
public: em1 wlan0
```

找出当前分配了接口（例如 em1）的区域：

```bash
~]$ firewall-cmd --get-zone-of-interface=em1
public
```

找出分配给一个区域（例如公共区域）的所有接口：

```bash
~]# firewall-cmd --zone=public --list-interfaces
em1 wlan0
```

找出public区域的所有设置：

```bash
~]# firewall-cmd --zone=public --list-all
public
  interfaces: 
  services: mdns dhcpv6-client ssh
  ports: 
  forward-ports: 
  icmp-blocks: source-quench
```

找出所有预设的服务

```bash
~]# firewall-cmd --get-service
cluster-suite pop3s bacula-client smtp ipp radius bacula ftp mdns samba dhcpv6-client dns openvpn imaps samba-client http https ntp vnc-server telnet libvirt ssh ipsec ipp-client amanda-client tftp-client nfs tftp libvirt-tls
```

重新加载防火墙，并不中断用户连接，即不丢失状态信息，当使用--permanent后，可以重新加载防火墙，使规则生效。

```bash
~]# firewall-cmd --reload
```


重新加载防火墙并中断用户连接，即丢弃状态信息：

```bash
~]# firewall-cmd --complete-reload
```

为分区增加接口

```bash
~]# firewall-cmd --zone=public --add-interface=em1
```

设置默认分区

```bash
~]# firewall-cmd --set-default-zone=public
```

列出区域打开的端口

```bash
~]# firewall-cmd --zone=dmz --list-ports
```

要将一个端口加入一个分区

```bash
~]# firewall-cmd --zone=dmz --add-port=8080/tcp
```

将一个服务加入到分区

```bash
~]# firewall-cmd --zone=work --add-service=smtp
```

从一个分区移除服务

```bash
~]# firewall-cmd --zone=work --remove-service=smtp
```

