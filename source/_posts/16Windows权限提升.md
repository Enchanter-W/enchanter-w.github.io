---
title: Windows权限提升全维度指南 # 从 Astro 的 title 转换
date: 2025/04/08 09:00:00 # 从 Astro 的 published 转换，并添加一个默认时间 (例如，早上9点)，使用您指定的日期格式
# updated: YYYY/MM/DD HH:MM:SS # 可选，如果文章有更新日期，可以添加此字段
description: Windows权限提升未完成版本 # 从 Astro 的 description 转换
cover: /images/cover16.jpg # 从 Astro 的 image 转换，注意路径前添加 '/' 表示根目录，并假设 cover16.jpg 放在 images 目录下
tags: # 从 Astro 的 tags 转换
  - 权限提升
  - OSCP
categories: # 从 Astro 的 category 转换，Hexo 常用 categories (复数)
  - 安全
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

  ## 一、Windows信息枚举

  ### （一）Windows权限及访问控制介绍

  Windows通过SID、access token、MIC、UAC等来进行权限控制。

  **SID**是一个用来区分不同实体的独特编码，其通过LSA来进行产生，在域中由DC进行产生，在生成后无法进行任何修改。SID的格式为“S-R-X-Y”，其中：

  - “S”表示这是一个SID。
  - “R”表示SID的修订版本。
  - “X”表示标识符权威，表明SID是由哪个机构或系统生成的。
  - “Y”表示子权威值，用于区分同一标识符权威下不同的用户或组。例如，“S-1-5-21-1234567890-1234567890-1234567890-1001”是一个用户账户的SID。

  **Access token**在用户进行登录时由系统产生给用户。用户在创建一个进程时会给进程赋予一个primary token来提示进程具有哪些权限等。

  Windows拥有四个权限等级：

  - **System**：system、kernel，拥有最高权限，几乎可以访问和操作系统的任何部分。
  - **High**：高权限用户，通常是指管理员账户，可以进行大部分系统管理和配置操作。
  - **Medium**：普通权限用户，一般是指标准用户账户，权限相对受限，无法进行一些关键的系统设置更改。
  - **Low**：权限受限账户，如www-data等服务账户，权限非常有限，主要用于运行一些特定的服务，防止这些服务被恶意利用时对系统造成较大危害。

  **UAC**（用户账户控制）提供了一种方法使管理员等高权限用户拥有两个不同权限的access token，能够在普通权限和高权限之间切换赋予进程权限。

  ### （二）系统基本信息探测

  #### 1. 几个基本的需要确认的信息

  - 用户名（username）和主机名（hostname）
  - 当前用户的用户组（group）
  - 存在的用户和组
  - 系统的版本和补丁等
  - 网络侧信息，网段等
  - 系统中安装的应用
  - 正在运行的进程

  #### 2. 用户/组信息收集

  ```powershell
  CMD:
      whoami
      whoami /groups
  PowerShell:
      Get-LocalUser
      Get-LocalGroup
      Get-LocalGroupMember xxx(组名)
  ```

  #### 3. 系统信息收集

  ```powershell
  systeminfo
  ipconfig /all
  route print
  netstat -ano
  Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname //64位系统
  Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Windows\CurrentVersion\Uninstall\*" | select displayname //32位系统
  Get-Process //查看进程
  ```

  ### （三）在系统中寻找敏感信息

  **敏感文件查找**——利用Get-ChildItem

  ```powershell
  Get-ChildItem -Path C:\(基目录) -Include *.扩展名,可以多选 -File -Recurse -ErrorAction SilentlyContinue //静默错误
  type 文件 cat 文件 //用来读取信息两个都能用没区别
  ```

  **获得密码后在命令行中如何切换用户**

  使用Runas。但是这个有局限性，如果你没有GUI界面的话无法调起UAC授权界面来输入密码，所以无法进行用户切换。为此我们可以使用PsExec脚本来实现切换用户。使用方法如下：

  - 首先需要下载PsExec工具，将其放置在可执行的目录下。
  - 然后在命令行中使用以下命令格式进行用户切换：`psexec -u 用户名 -p 密码 命令`，例如`psexec -u administrator -p password cmd`，即可以管理员身份打开一个新的命令提示符窗口。

  ### （四）寻找PS中的敏感信息

  **利用条件和原因**：powershell存在记录命令历史的功能，这个功能由两个不同的方式来提供，一般通过clear-history清除的只有其中一个，另一个没有清除的历史里面还记录了transcription脚本的内容等信息（PSReadline）。

  **Get-history**

  ```powershell
  Get-history //查询容易被清空的历史记录
  ```

  **Get-PSReadlineOption**

  ```powershell
  Get-PSReadlineOption //获取PSReadline的信息
  ```

  *HistorySavePath*：如`C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`，这个地方存着历史信息，通过`type`来查看内容。

  **Enter-PSSession**是什么有什么作用？怎么用？

  - **Enter-PSSession**是PowerShell的一个命令，用于建立与远程计算机的交互式会话，使用户能够在远程计算机上直接运行命令，就像在本地计算机上一样。它允许用户在远程系统上执行脚本、管理服务、配置设置等操作。
  - 使用方法：`Enter-PSSession -ComputerName 计算机名 -Credential 用户名`，例如`Enter-PSSession -ComputerName remotePC -Credential domain\user`，然后输入密码即可建立会话。

  ### （五）在命令行中获取到的交互式shell存在问题时的解决方法

  当在命令行中获取到的交互式shell存在问题时，可以通过evil-winrm工具来进行操作。evil-winrm是一个基于Ruby的工具，它允许用户通过WinRM协议与Windows系统进行交互，具有较好的稳定性和功能扩展性。

  ### （六）自动化工具——以winpeas举例

  **winpeas**是一个用于Windows系统的信息收集和权限提升辅助工具，它会自动收集系统中的各种信息，包括但不限于用户信息、系统配置、网络连接、运行的进程等，并尝试寻找可能的权限提升点。

  - **回显内容解释**：
    - **用户信息**：列出系统中存在的用户账户及其所属组，帮助攻击者了解当前系统中可能存在的高权限账户。
    - **系统配置信息**：包括系统版本、补丁信息、安装的应用程序等，这些信息有助于攻击者判断系统是否存在已知的漏洞，从而选择合适的攻击方法。
    - **网络连接信息**：显示当前系统的网络连接状态、IP地址、网关等信息，这对于攻击者进行横向移动或寻找其他攻击目标非常有帮助。
    - **运行的进程信息**：列出系统中正在运行的进程及其对应的用户权限，攻击者可以寻找具有高权限的进程，尝试通过进程注入等方法提升自己的权限。
    - **权限提升点提示**：winpeas会根据收集到的信息，分析出可能的权限提升点，如未修复的漏洞、配置不当的服务等，并给出相应的提示和建议。
