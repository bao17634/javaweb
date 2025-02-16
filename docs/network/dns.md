# 域名系统 DNS

> 域名系统（英文：Domain Name System，缩写：DNS）是互联网的一项服务。它作为将域名和 IP 地址相互映射的一个分布式数据库，能够使人更方便地访问互联网。DNS 使用 TCP 和 UDP 端口 53。当前，对于每一级域名长度的限制是 63 个字符，域名总长度则不能超过 253 个字符。
>
> 关键词：DNS, 域名解析

<!-- TOC depthFrom:2 depthTo:3 -->

- [简介](#简介)
    - [什么是 DNS？](#什么是-dns)
    - [什么是域名？](#什么是域名)
    - [DNS 的分层](#dns-的分层)
    - [DNS 服务类型](#dns-服务类型)
    - [记录类型](#记录类型)
- [域名解析](#域名解析)
- [Linux 上的域名相关命令](#linux-上的域名相关命令)
    - [hostname](#hostname)
    - [nslookup](#nslookup)
- [更多内容](#更多内容)

<!-- /TOC -->

## 简介

### 什么是 DNS？

DNS 是一个应用层协议。

域名系统 (DNS) 的作用是将人类可读的域名 (如，www.example.com) 转换为机器可读的 IP 地址 (如，192.0.2.44)。

### 什么是域名？

域名是由一串用点分隔符 `.` 组成的互联网上某一台计算机或计算机组的名称，用于在数据传输时标识计算机的方位。域名可以说是一个 IP 地址的代称，目的是为了便于记忆后者。例如，wikipedia.org 是一个域名，和 IP 地址 208.80.152.2 相对应。人们可以直接访问 wikipedia.org 来代替 IP 地址，然后域名系统（DNS）就会将它转化成便于机器识别的 IP 地址。这样，人们只需要记忆 wikipedia.org 这一串带有特殊含义的字符，而不需要记忆没有含义的数字。

<div align="center"><img src="http://dunwu.test.upcdn.net/cs/network/application/dns/dns-domain.png!zp"/></div>

### DNS 的分层

**域名系统是分层次的。**

在域名系统的层次结构中，各种域名都隶属于域名系统根域的下级。域名的第一级是顶级域，它包括通用顶级域，例如 `.com`、`.net` 和 `.org`；以及国家和地区顶级域，例如 `.us`、`.cn` 和 `.tk`。顶级域名下一层是二级域名，一级一级地往下。这些域名向人们提供注册服务，人们可以用它创建公开的互联网资源或运行网站。顶级域名的管理服务由对应的域名注册管理机构（域名注册局）负责，注册服务通常由域名注册商负责。

### DNS 服务类型

- **授权型 DNS** - 一种授权型 DNS 服务提供一种更新机制，供开发人员用于管理其公用 DNS 名称。然后，它响应 DNS 查询，将域名转换为 IP 地址，以便计算机可以相互通信。授权型 DNS 对域有最终授权且负责提供递归型 DNS 服务器对 IP 地址信息的响应。Amazon Route 53 是一种授权型 DNS 系统。
- **递归型 DNS** - 客户端通常不会对授权型 DNS 服务直接进行查询。而是通常连接到称为解析程序的其他类型 DNS 服务，或递归型 DNS 服务。递归型 DNS 服务就像是旅馆的门童：尽管没有任何自身的 DNS 记录，但是可充当代表您获得 DNS 信息的中间程序。如果递归型 DNS 拥有已缓存或存储一段时间的 DNS 参考，那么它会通过提供源或 IP 信息来响应 DNS 查询。如果没有，则它会将查询传递到一个或多个授权型 DNS 服务器以查找信息。

### 记录类型

DNS 中，常见的资源记录类型有：

- **NS 记录（域名服务）** ─ 指定解析域名或子域名的 DNS 服务器。
- **MX 记录（邮件交换）** ─ 指定接收信息的邮件服务器。
- **A 记录（地址）** ─ 指定域名对应的 IPv4 地址记录。
- **AAAA 记录（地址）** ─ 指定域名对应的 IPv6 地址记录。
- **CNAME（规范）** ─ 一个域名映射到另一个域名或 `CNAME` 记录（ example.com 指向 [www.example.com](http://www.example.com/) ）或映射到一个 `A`记录。
- **PTR 记录（反向记录）** ─ PTR 记录用于定义与 IP 地址相关联的名称。 PTR 记录是 A 或 AAAA 记录的逆。 PTR 记录是唯一的，因为它们以 .arpa 根开始并被委派给 IP 地址的所有者。

> 详细可以参考：[维基百科 - 域名服务器记录类型列表](https://zh.wikipedia.org/wiki/域名服务器记录类型列表)

## 域名解析

主机名到 IP 地址的映射有两种方式：

- **静态映射** - 在本机上配置域名和 IP 的映射，旨在本机上使用。Windows 和 Linux 的 hosts 文件中的内容就属于静态映射。
- **动态映射** - 建立一套域名解析系统（DNS），只在专门的 DNS 服务器上配置主机到 IP 地址的映射，网络上需要使用主机名通信的设备，首先需要到 DNS 服务器查询主机所对应的 IP 地址。

通过域名去查询域名服务器，得到 IP 地址的过程叫做域名解析。在解析域名时，一般先静态域名解析，再动态解析域名。可以将一些常用的域名放入静态域名解析表中，这样可以大大提高域名解析效率。

<div align="center"><img src="http://dunwu.test.upcdn.net/cs/network/application/dns/dns-resolve.png!zp"/></div>

上图展示了一个动态域名解析的流程，步骤如下：

1. 用户打开 Web 浏览器，在地址栏中输入 www.example.com，然后按 Enter 键。
2. www.example.com 的请求被路由到 DNS 解析程序，这一般由用户的 Internet 服务提供商 (ISP) 进行管理，例如有线 Internet 服务提供商、DSL 宽带提供商或公司网络。
3. ISP 的 DNS 解析程序将 www.example.com 的请求转发到 DNS 根名称服务器。
4. ISP 的 DNS 解析程序再次转发 www.example.com 的请求，这次转发到 .com 域的一个 TLD 名称服务器。.com 域的名称服务器使用与 example.com 域相关的四个 Amazon Route 53 名称服务器的名称来响应该请求。
5. ISP 的 DNS 解析程序选择一个 Amazon Route 53 名称服务器，并将 www.example.com 的请求转发到该名称服务器。
6. Amazon Route 53 名称服务器在 example.com 托管区域中查找 www.example.com 记录，获得相关值，例如，Web 服务器的 IP 地址 (192.0.2.44)，并将 IP 地址返回至 DNS 解析程序。
7. ISP 的 DNS 解析程序最终获得用户需要的 IP 地址。解析程序将此值返回至 Web 浏览器。DNS 解析程序还会将 example.com 的 IP 地址缓存 (存储) 您指定的时长，以便它能够在下次有人浏览 example.com 时更快地作出响应。有关更多信息，请参阅存活期 (TTL)。
8. Web 浏览器将 www.example.com 的请求发送到从 DNS 解析程序中获得的 IP 地址。这是您的内容所处位置，例如，在 Amazon EC2 实例中或配置为网站终端节点的 Amazon S3 存储桶中运行的 Web 服务器。
9. 192.0.2.44 上的 Web 服务器或其他资源将 www.example.com 的 Web 页面返回到 Web 浏览器，且 Web 浏览器会显示该页面。

> 注意：只有配置了域名服务器，才能执行域名解析。
>
> 例如，在 Linux 中执行 `vim /etc/resolv.conf` 命令，在其中添加下面的内容来配置域名服务器地址：
>
> ```
> nameserver 218.2.135.1
> ```

## Linux 上的域名相关命令

### hostname

> hostname 命令用于查看和设置系统的主机名称。环境变量 HOSTNAME 也保存了当前的主机名。在使用 hostname 命令设置主机名后，系统并不会永久保存新的主机名，重新启动机器之后还是原来的主机名。如果需要永久修改主机名，需要同时修改 `/etc/hosts` 和 `/etc/sysconfig/network` 的相关内容。
>
> 参考：http://man.linuxde.net/hostname

示例：

```bash
$ hostname
AY1307311912260196fcZ
```

### nslookup

> nslookup 命令是常用域名查询工具，就是查 DNS 信息用的命令。
>
> 参考：http://man.linuxde.net/nslookup

示例：

```bash
[root@localhost ~]# nslookup www.jsdig.com
Server:         202.96.104.15
Address:        202.96.104.15#53

Non-authoritative answer:
www.jsdig.com canonical name = host.1.jsdig.com.
Name:   host.1.jsdig.com
Address: 100.42.212.8
```

## 更多内容

- [维基百科 - 域名](https://zh.wikipedia.org/wiki/域名)
- [维基百科 - 域名系统](https://zh.wikipedia.org/wiki/域名系统)
- [维基百科 - 域名服务器记录类型列表](https://zh.wikipedia.org/wiki/域名服务器记录类型列表)
- [什么是 DNS？](https://aws.amazon.com/cn/route53/what-is-dns/)
- [RFC 1034](https://tools.ietf.org/html/rfc1034)
- [RFC 1035](https://tools.ietf.org/html/rfc1035)
