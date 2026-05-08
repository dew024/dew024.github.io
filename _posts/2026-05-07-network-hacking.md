---
layout: post
title:  "网络渗透工具使用"
tags: [security]
---

* 目录
{:toc}

# Networking Security

## 被动侦查

被动侦察（Passive Reconnaissance）的核心定义是：在不与目标系统发生直接交互的情况下，收集有关目标的信息。

例如：

1. 从公共DNS服务器查询域名的DNS记录
2. 查看目标的招聘广告
3. 阅读有关于目标的新闻文章

### whois

WHOIS是遵循RFC 3912的协议，服务器监听TCP 43端口，域名商维护域名的WHOIS记录，WHOIS服务器会响应所请求域名的各种信息：

1. 注册商：这个域名是从哪个注册商注册的
2. 注册人的联系信息
3. 创建、更新、过期日期
4. 名称服务器：应向哪台服务器请求解析该域名

whois客户端的使用：

```
whois baidu.com
```

### nslookup & dig

nslookup可以查找域名的ip地址，使用方式：

```bash
# 可选指定DNS服务器
nslookup baidu.com 1.1.1.1

# 查询ipv4地址
nslookup -type=A baidu.com

# 查询ipv6地址
nslookup -type=AAAA baidu.com

# 查询cname
nslookup -type=CNAME baidu.com

# 查询mail server
nslookup -type=MX baidu.com

# 查询TXT Records
nslookup -type=TXT baidu.com

# 查询起始授权机构
nslookup -type=SOA baidu.com
```

dig命令（Domain Information Groper 域名信息查询器）提供了更多高级功能。

```
dig @1.1.1.1 baidu.com MX
```

### DNSDumpster

nslookup/dig是查不到子域名的，除了web hacking部分提到的子域名枚举技巧外，通过[DNSDumpster](https://dnsdumpster.com/)也可以查到子域名。

### Shodan.io

[Shodan.io](https://www.shodan.io/)能让你在不主动连接客户网络的情况下，了解到有关该网络的各种信息。此外，在防御方面，你可以使用 Shodan.io 的不同服务来了解属于你所在组织的已连接和暴露的设备。

## 主动侦查

主动侦察要求你与目标进行某种形式的接触。这种接触可以是打电话，或者以某种借口拜访目标公司以收集更多信息，这通常是社会工程学的一部分。此外，也可以是与目标系统直接连接，比如访问他们的网站，或者检查他们的防火墙是否开放了 SSH 端口。可以把这想象成你在仔细检查窗户和门锁。因此，务必记住，在获得客户签署的合法授权之前，不要进行主动侦察工作。

### 浏览器

1. 开发者工具
2. 插件
    - FoxyProxy：更改访问目标网站时使用的代理服务器
    - User-Agent Switcher and Manager：伪装成从不同的操作系统或不同的网页浏览器访问网页
    - Wappalyzer：提供关于所访问网站使用的技术的信息

### Ping

ping 是一个向远程系统发送 ICMP 回显数据包的命令。如果远程系统处于在线状态，且 ping 数据包路由正确且未被任何防火墙拦截，那么远程系统就会返回一个 ICMP 回显应答。同样，如果路由正确且未被任何防火墙拦截，ping 应答也能到达发起请求的系统。

这个命令的目的是，在我们花费时间进行更详细的扫描以发现目标系统运行的操作系统和服务之前，先确认目标系统处于在线状态。

从技术上讲，ping 属于 ICMP（互联网控制消息协议）。ICMP 支持多种类型的查询，不过我们重点关注的是 ping（ICMP 回显 / 类型 8）和 ping 应答（ICMP 回显应答 / 类型 0）。使用 ping 并不需要深入了解 ICMP 的详细知识。

一般来说，当我们没有收到 ping 回复时，有几种解释可以说明原因，例如：

- 目标计算机没有响应；可能还在启动中、已关闭，或者操作系统已崩溃。
- 它与网络断开了连接，或者路径上存在有故障的网络设备。
- 防火墙被配置为阻止此类数据包。防火墙可能是运行在系统本身的一款软件，也可能是一个独立的网络设备。请注意，微软 Windows 防火墙默认会阻止 ping 操作。
- 你的系统与网络断开了连接。

### Traceroute

traceroute 命令会追踪数据包从你的系统到另一台主机所经过的路由。traceroute 的目的是找出数据包从你的系统传输到目标主机时所经过的路由器（或称跃点）的 IP 地址。该命令还能显示两个系统之间的路由器数量。这很有用，因为它能表明你的系统和目标主机之间的跃点（路由器）数量。不过要注意，数据包所走的路由可能会发生变化，因为许多路由器使用的动态路由协议会根据网络变化进行调整。

有些路由器会返回公网 IP 地址。你可以根据预定渗透测试的范围，对其中一些路由器进行检查。有些路由器不会返回响应。

### Telnet

TELNET（电传网络）协议于 1969 年开发，用于通过命令行界面（CLI）与远程系统进行通信。因此，telnet 命令使用 TELNET 协议进行远程管理。telnet 使用的默认端口是 23。从安全角度来看，telnet 以明文形式发送所有数据，包括用户名和密码。明文发送使得任何能够访问通信信道的人都能轻易窃取登录凭证。安全的替代方案是 SSH（安全外壳）协议。

通过 `telnet MACHINE_IP PORT` 命令，你可以连接到运行在 TCP 上的任何服务，甚至可以交换一些消息，除非该服务使用了加密。

### Netcat

Netcat 支持 TCP 和 UDP 两种协议。它既可以作为客户端连接到监听端口，也可以作为服务器在你选择的端口上进行监听。因此，它是一个便捷的工具，可作为基于 TCP 或 UDP 的简单客户端或服务器使用。

用法1：`nc MACHINE_IP PORT`，默认TCP，和telnet一样。

用法2：`nc -vnlp PORT`，监听端口，默认TCP。

## Nmap

Nmap 是行业标准工具，用于绘制网络地图、识别活跃主机以及发现运行中的服务。Nmap 的脚本引擎能够进一步扩展其功能，从服务指纹识别到漏洞利用都能实现。

### 存活主机探测

存活主机探测这一步非常关键，因为对离线系统执行端口扫描，只会浪费时间并产生不必要的网络流量。

Nmap用于发现存活主机的几种不同方式，重点包括：

- ARP扫描：通过ARP请求发现存活主机，如果你处于`10.0.0.0/16`，那么ARP只能发现该子网内的设备，因为ARP是二层协议，不会被路由
- ICMP扫描：通过ICMP请求识别存活主机
- TCP/UDP扫描：向TCP/UDP端口发送数据包，判断主机是否在线，在ICMP回显被阻止时需要用到

使用`nmap -sn TARGETS`会只发现主机，不扫端口。

1. 如果是扫内网，且有sudo权限，默认会使用ARP扫描，这个是最准的
2. 如果是扫外网，且有sudo权限，会使用以下组合拳，任何一个有回音都算存活主机
    - ICMP Echo，ICMP type 8
    - TCP ACK（80端口），发个ACK包，如果是存活的，会响应RST
    - TCP SYN（443端口），发个握手请求
    - ICMP Timestamp，ICMP type 13
3. 没有sudo权限，对80/443使用TCP SYN，即三次握手

使用nmap可以提供这些作为扫描的目标：

1. list: `10.0.0.1 test.org example.com`，这会扫描三个地址
2. range: `10.11.12.15-20`，这会扫描6个地址
3. subnet: `192.168.0.0/30`，这会扫描4个地址
4. txt: `nmap -iL list_of_hosts.txt`

使用`nmap -sL TARGETS`可以查看将要扫描的主机的列表，但不会实际扫描。

#### ARP扫描

使用`sudo nmap -PR -sn TARGETS`显式进行ARP扫描。

`arp-scan`是一个基于ARP查询构建的扫描工具，常见用法是`arp-scan --localnet`等价于`arp-scan -l`，这个命令会向本地网络所有有效IP地址发送ARP查询。

还可以通过`arp-scan -I eth0 -l`指定网卡。

#### ICMP扫描

1. `sudo nmap -PE -sn TARGETS`显式进行ICMP type 8（echo）扫描，但是大多数防火墙都会拦截。
2. `sudo nmap -PP -sn TARGETS`显式进行ICMP type 13（timestamp）扫描。
3. `sudo nmap -PM -sn TARGETS`显式进行ICMP type 17（address mask）扫描。

#### TCP/UDP扫描


##### TCP SYN

我们可以向一个 TCP 端口（默认是 80 端口）发送一个设置了 SYN（同步）标志的数据包，并等待响应。开放的端口应该会回复 SYN/ACK（确认）；关闭的端口则会返回 RST（重置）。在这种情况下，我们只通过检查是否收到响应来推断主机是否处于活动状态。端口的具体状态在这里并不重要。

sudo用户不需要完成三次握手，但是普通用户如果端口开放就必须要完成三次握手。

使用`sudo nmap -PS -sn TARGETS`显式进行TCP SYN扫描。`-PS`后可跟端口号、端口范围、端口列表或它们的组合。例如`-PS21`、`-PS21-25`、`-PS80,443,8080`。

##### TCP ACK

使用`sudo nmap -PA -sn TARGETS`显式进行TCP ACK扫描，如果是非sudo权限会尝试三次握手。`-PA`后可跟端口，和`-PS`类似，默认80端口。

任何设置了 ACK 标志的 TCP 数据包都应收到一个设置了 RST 标志的 TCP 数据包。因为带有 ACK 标志的 TCP 数据包不属于任何正在进行的连接。这种预期的响应被用于检测目标主机是否处于开机状态。

##### UDP

向开放端口发送 UDP 数据包预计不会收到任何回复。然而，如果我们向关闭的 UDP 端口发送 UDP 数据包，预计会收到一个 ICMP 端口不可达数据包，这表明目标系统处于运行状态且可用。

使用`sudo nmap -PU -sn TARGETS`显式进行UDP扫描，指定端口格式类似，默认40125：一个极不可能开放的UDP端口。

#### DNS反查

nmap默认会查询主机的域名，通过`-n`参数可以禁止，通过`-R`设置离线主机也反查域名，通过`--dns-servers`设置特定的DNS服务器。

### 端口扫描基础

找到存活主机后，要确定哪些端口开放且处于监听状态。nmap的端口扫描类型有：

1. TCP连接端口扫描
2. TCP SYN端口扫描
3. UDP端口扫描

#### 端口状态

端口有可能是开放或关闭状态，即是否有服务在监听。但是也要考虑防火墙的影响，因此nmap将端口状态分为六种：

1. Open：表示端口上有服务监听
2. Closed：表示端口上没有服务监听，但是端口可访问（端口可达没有被防火墙阻挡）。
3. Filtered：表示无法确定Open还是Closed，因为端口不可访问。通常是由于防火墙阻止造成的，nmap数据包无法到达该端口，或响应无法返回到nmap所在主机。
4. Unfiltered：表示虽然端口可访问，但是nmap无法确认Open还是Closed。这种状态在使用ACK扫描（-sA）时会出现。
5. Open/Filtered：表示nmap无法确认是Open还是Filtered。
6. Closed/Filtered：表示nmap无法确认是Closed还是Filtered。

#### TCP Flags

TCP报头是TCP段的前24字节，报头中有6个标志位，分别为：

1. URG紧急标志：表明传入的数据是紧急的，设置了 URG 标志的 TCP 段会被立即处理，无需等待之前发送的 TCP 段。
2. ACK确认标志：用于确认已收到 TCP 段。
3. PSH推送标志：要求 TCP 立即将数据传递给应用程序。
4. RST重置标志：用于重置连接。其他设备（如防火墙）可能会发送此标志来终止 TCP 连接。当向主机发送数据但接收端没有相应的服务来响应时，也会使用此标志。
5. SYN同步标志：用于发起 TCP 三次握手，并与另一台主机同步序列号。在 TCP 连接建立过程中，序列号应随机设置。
6. FIN：发送方没有更多数据要发送。

#### TCP连接扫描

TCP 连接扫描通过完成 TCP 三次握手来工作。在标准的 TCP 连接建立过程中，客户端发送一个设置了 SYN 标志的 TCP 数据包，如果端口是开放的，服务器会以 SYN/ACK 响应；最后，客户端通过发送 ACK 完成三次握手。

使用`nmap -sT TARGETS`来进行扫描，这是非sudo用户扫描TCP端口的唯一方法。

默认情况下，Nmap 会尝试连接 1000 个最常见的端口。关闭的 TCP 端口会对 SYN 数据包回复 RST/ACK，以此表明该端口未开放。开放的TCP端口会返回SYN/ACK，nmap会发送ACK完成三次握手。再发送RST/ACK断开连接。

请注意，我们可以使用`-F`来启用快速模式，并将扫描的端口数量从 1000 个减少到 100 个最常见的端口。还可以添加`-r`选项，使端口扫描按连续顺序而非随机顺序进行。这个选项在测试端口是否以固定顺序开放时非常有用，例如：观察目标机在启动过程中各项服务的加载顺序。

#### TCP SYN扫描

SYN扫描需要sudo权限。SYN 扫描不需要完成 TCP 三次握手，相反，一旦收到服务器的响应（SYN/ACK），它就会中断连接（发送RST）。由于我们没有建立 TCP 连接，这降低了扫描被记录的可能性。

使用`sudo nmap -sS TARGETS`来进行扫描。

#### UDP扫描

UDP 是一种无连接协议，因此它不需要任何握手来建立连接。我们无法保证监听 UDP 端口的服务会对我们的数据包做出响应。但是，如果向关闭的端口发送 UDP 数据包，会返回 ICMP 端口不可达错误。

使用`sudo nmap -sU TARGETS`来进行扫描。

#### 参数调整

1. 指定扫描端口：
    - `-p22,80,443`：特定端口
    - `-p1-1023`：端口范围
    - `-F`：top100
    - `--top-ports 10`：top10
    - `-p-`：所有端口
2. 扫描速度：`-T<0-5>`，0最谨慎，5最疯狂
    - 默认是`-T3`
    - `-T4`在CTF以及练习中常用
    - `-T5`可能会丢包影响准确性
    - `-T0`发包间隔5分钟
    - 为了避免入侵检测系统警报，一般用`-T0`或`-T1`
3. 控制发包速率：`--min-rate`和`--max-rate`控制每秒发包数量
    - `-max-rate=10`可确保扫描器每秒发送的数据包不超过 10 个
4. 控制并发度：`--min-parallelism`和`--max-parallelism`
    - `--min-parallelism=512`Nmap 保持至少 512 个并行探测

### 端口扫描进阶

#### TCP Null Scan

使用`sudo nmap -sN TARGETS`进行空扫描。空扫描不设置任何标志位，所有六个标志位都被设为零。

如果端口是开放的：

- 目标服务器收到这个“全是零”的怪异数据包后，会觉得莫名其妙。按照协议标准，它通常会直接丢弃这个包，不作任何回应。
- 结果：Nmap 没收到回信 -> 判定为 Open/Filtered（开放或被防火墙拦截）。

如果端口是关闭的：

- 目标服务器会按照规定，回传一个 RST (Reset) 包，表示“这里没人在家，别敲了”。
- 结果：Nmap 收到 RST -> 判定为 Closed。

#### TCP FIN Scan

![](/assets/images/20260557/fin-scan-1.png)
![](/assets/images/20260557/fin-scan-2.png)

#### TCP Xmas Scan

![](/assets/images/20260557/xmas-scan-1.png)
![](/assets/images/20260557/xmas-scan-2.png)

#### TCP Maimon Scan

目前已不适用主流系统，一些老旧的BSD系统获取还能有效。

![](/assets/images/20260557/maimon-scan-1.png)

#### TCP ACK Scan

无法区分端口是开启还是关闭，但是能看出端口是否被防火墙过滤。

![](/assets/images/20260557/ack-scan-1.png)

#### TCP Window Scan

可以看出端口是被防火墙过滤/开启/关闭，开启关闭的监测不准，仅一些老系统上有效。

![](/assets/images/20260557/window-scan-1.png)

#### 源IP伪装

`nmap -e NET_INTERFACE -Pn -S SPOOFED_IP 10.48.168.41`

![](/assets/images/20260557/spoof.png)

![](/assets/images/20260557/decoy.png)

#### 入侵监测规避

目标可能会识别IP报头和传输层报头的特征来监测入侵行为，为了规避检测，攻击者可以将一个大的数据包拆分成多个微小的分片。IP分片原理：在 IP 报头中，使用 Identification (ID) 字段来标识属于同一个原始包的所有分片，并利用 Fragment Offset (分片偏移) 来告知接收方如何重新组合这些数据。当数据包被拆分后，防火墙和 IDS 往往无法在单个分片中看到完整的攻击特征（如完整的 TCP 报头或特定负载）。

-f 选项会将 IP 数据部分（此处为 TCP 报头）拆分为 8 字节 或更小。

#### 利用僵尸机进行空闲扫描

`nmap -sI ZOMBIE_IP 10.48.168.41`

在 Internet 协议中，每发送一个 IP 数据包，主机通常会将报头中的 Identification (IP ID) 字段加 1。空闲扫描就是通过观察“僵尸机” IP ID 的增长量来判断目标机是否有了回应。

前提条件： 僵尸机必须处于真正的“空闲”状态，且其操作系统生成的 IP ID 必须是递增的（现在的 Linux/Windows 较难实现，常用于旧系统或简单的网络设备如打印机）。

和Spoofing类似，不过这个伪装的ip有前提，且可以通过id递增判断结果。

### 端口扫描后

#### 确认服务版本

`nmap -sV -p143 --version-light 10.49.191.84`

-sV会进行完整的三次握手。

#### 确认系统版本

加-O参数。

#### traceroute

加--traceroute参数。

#### NSE脚本

加-sC参数执行默认脚本。

`nmap --script ssh2-enum-algos 10.49.153.214`。

#### 保存结果

`-oA <文件名>`

## 协议和服务

### Telnet

使用23端口，是ssh的不加密版本。

### Ftp

使用21端口（控制通道）。数据传输时会建立独立的数据通道。

FTPS（基于TLS协议，990端口）。SFTP（通过SSH协议传输，所以用22端口和SSH一样）。

```
root@ip-10-49-87-46:/tmp# ftp 10.49.139.227
Connected to 10.49.139.227.
220 (vsFTPd 3.0.5)
Name (10.49.139.227:root): frank
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwx------   10 1001     1001         4096 Sep 15  2021 Maildir
-rw-rw-r--    1 1001     1001         4006 Sep 15  2021 README.txt
-rw-rw-r--    1 1001     1001           39 Sep 15  2021 ftp_flag.thm
226 Directory send OK.
ftp> get ftp_flag.thm
local: ftp_flag.thm remote: ftp_flag.thm
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for ftp_flag.thm (39 bytes).
226 Transfer complete.
39 bytes received in 0.00 secs (43.9284 kB/s)
ftp> exit
221 Goodbye.
```

### SMTP

使用25端口，负责邮件发送。

SMTPS（465端口）。

### POP3

使用110端口，负责邮件接收。

POP3S（995端口）。

### IAMP

使用143端口，负责邮件接收，比POP3更先进。

以上协议都是明文传输。

IMAPS（993端口）。

### 嗅探（Sniffing）攻击

`sudo tcpdump port 110 -A` -A是Ascii的意思。或者用wireshark。

### 中间人（MITM）攻击

当受害者（A）以为自己正在与合法目标（B）通信，却在不知情的情况下与攻击者（E）通信时，就会发生中间人攻击。

如果通信双方不对每条消息的真实性和完整性进行校验，实施此类攻击的难度会相对较低。部分场景下，所采用的通信协议本身不具备安全认证或完整性校验能力；此外，一些协议存在固有安全漏洞，极易遭受这类攻击。

只要通过 HTTP 协议进行网页浏览，就有可能遭遇中间人攻击，更令人担忧的是，用户无法察觉这类攻击行为。 Ettercap、Bettercap等多款工具都可辅助实施此类攻击。

中间人攻击同样会影响 FTP、SMTP、POP3 等其他明文传输协议。抵御该类攻击需要借助密码学技术，具体解决办法是对传输的消息进行合法身份认证，并对消息做加密或签名处理。借助公钥基础设施（PKI）与可信根证书，传输层安全协议（TLS）能够有效防护中间人攻击。

### TLS协议

位于表示层。

客户端可以与持有公开证书的服务器协商出一个密钥。该密钥以安全方式生成，监控通信通道的第三方无法破解获取。客户端与服务器后续的所有通信，都会使用这组生成的密钥进行加密。

因此，一旦 SSL/TLS 握手流程建立完成，任何人监听通信通道都无法获取到 HTTP 请求以及双方传输的数据。

### SSH协议

```
PS C:\Users\dell> ssh root@10.48.79.133
The authenticity of host '10.48.79.133 (10.48.79.133)' can't be established.
ED25519 key fingerprint is SHA256:b+oX15Wfpf1OAEyK16CBYkZm1CXmCMS8qFdxSLO5R14.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
这是为了防止中间人劫持，如果已经连接过却又提示，说明服务器公钥变了，说明可能被劫持了。

SCP是基于SSH的文件传输。

### 密码攻击

-l是登录名，-P是密码本

https://github.com/vanhauser-thc/thc-hydra

```
hydra -l mark -P /usr/share/wordlists/rockyou.txt MACHINE_IP ftp
hydra -l frank -P /usr/share/wordlists/rockyou.txt MACHINE_IP ssh
```