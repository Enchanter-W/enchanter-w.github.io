---
title: Sar
published: 2025-04-10
description: 'Sar靶机——最简单的一集'
image: ./images/Sar_cover.png
tags: [渗透测试, Linux提权, RCE, 计划任务]
category: 'Writeup'
draft: false 
lang: 'zh'
---

# Sar

## 靶机发现端口信息枚举

```
ip:	192.168.55.129
```

### 端口探测&服务探测

nmap四扫之后只发现了80端口在跑的服务是apache+php

有个phpinfo上面写着7.x的版本

![image-20250410095243204](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250410095243204.png)

想着试试前面两个命令执行，但是php版本不符合要求直接放弃

![image-20250410095018125](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250410095018125.png)

## web服务探测

### 80端口

默认的apache页面，直接开始扫目录

robots.txt存在这个内容

```
sar2HTML
```

访问一下
![image-20250410100701078](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250410100701078.png)

搜一下sar2HTML的漏洞发现当前版本存在RCE

在index.php?polt=;<命令>即可执行命令

## 漏洞利用&立足点获取

利用漏洞反弹shell获取立足点

![image-20250410101009738](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250410101009738.png)

## 机器信息收集

### /etc/passwd

```
root
love
```

### /etc/crontab

```bash
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
*/5  *    * * *   root    cd /var/www/html/ && sudo ./finally.sh
```

发现存在计划任务并且在/var/www/html/目录下我们有一定的权限

## 权限提升&Root

finally.sh会去调用write.sh

```bash
www-data@sar:/var/www/html$ cat finally.sh
cat finally.sh
#!/bin/sh

./write.sh

```

write.sh我们拥有写权限

直接echo一个反弹shell的命令进去等待5min

![image-20250410101602732](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250410101602732.png)

## 总结复盘

很简单的一台靶机，思路也很清晰，计划任务提权。懒省事直接传linpeas上去也能看到这个高亮。前面还一直以为是让我找sar2html这个网站的漏洞比如文件上传啥的，结果就是个RCE，那没事了
