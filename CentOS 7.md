### 概述

本篇记录 CentOS 7 Minimal 版本安装后的一些基本配置。

查看系统具体的发行版：

```bash
cat /etc/centos-release
```

输出如下：

```
CentOS Linux release 7.9.2009 (Core)
```

### 版本环境

- VMware Workstation 16 Pro（16.2.3 build-19376536）
- CentOS 7.9（CentOS-7-x86_64-DVD-2009）

### 安装

VMware Workstation 中选择 CentOS 7 镜像，类型选择“最小化安装”。

CentOS 系统安装完毕或完成基本网络、软件配置之后，强烈建议对虚拟机进行“保存快照”操作，以便在系统崩溃或重建坏境时能迅速利用快照对恢复系统。

注意，创建快照时务必让虚拟机处于“关机”状态，如果虚拟机处于“运行”或“挂起”状态，那么虚拟机所生成的快照只能用于“回溯”，无法用于“克隆”。

### 基础命令

关机：

```bash
shutdown -h now
```

重启：

```bash
shutdown -r now
```

如果不添加 now 参数，那么系统将会向所有用户发送一个关机或重启通知，并在一定时间后关机或重启。这段时间通常是一分钟，但具体取决于系统配置。

如果已经运行了 shutdown 命令但未添加 now 参数，那么可以在一定的时间内取消关机或重启：

```bash
shutdown -c
```

### 网络

多数情况下，选择 CentOS 系统是为了完成某些软件的模拟，例如测试 Redis、MySQL 等。为了便于使用命令行操作，一般会使用 OpenSSH 连接 CentOS 系统。

宿主机如果需要连接到 CentOS 虚拟机，那么 CentOS 必须首先拥有可用网络。由于某些 CentOS 版本默认不提供网络服务，这不可避免地需要了解如何手动配置 CentOS 系统的网络。

首先，请务必确认 VMware 软件“编辑 - 虚拟网络编辑器 - 更改设置”下拥有三种不同的网络：

<img src="images/CentOS%207.images/image-20230903171816941.png" alt="image-20230903171816941" style="zoom:50%;" />

注意，类型为“桥接模式”的 VMnet0 网络，必须确保它所桥接的网卡为当前宿主机所使用的网卡。

其次，推荐将虚拟机设置中网络适配器修改为“桥接模式（自动）”，这便于后续为虚拟机配置静态 IP 地址：

<img src="images/CentOS%207.images/image-20230903185131946.png" alt="image-20230903185131946" style="zoom:50%;" />

CentOS 系统的网络配置文件一般存储在 `/etc/sysconfig/network-scripts/` 目录下，文件名称由特定前缀 `ifcfg-` 和网络接口（Network Interface）名称两部分共同组成。

网络接口名称的格式会因 Linux 发行版、系统配置和网络设备的不同而有所不同，CentOS 系统中的网络接口名称一般为 ens33。

查看网络接口名可以使用以下命令：

```bash
ip link show
```

命令将输出类似的内容：

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:a4:a6:64 brd ff:ff:ff:ff:ff:ff
```

其中：

- `lo` ：回环接口，它一个特殊的网络接口，通常在所有的现代操作系统中都存在；
- `ens33`：具体的物理或虚拟以太网接口，网络配置主要针对的就是这个网络接口。

由此可知当前 CentOS 的网络配置文件的命名为 ifcfg-ens33。

除此之外，系统如果安装了 net-tools 工具集，那么也可以使用以下命令查看网络接口：

```bash
netstat -i
```

命令将输出类似的内容：

```
Kernel Interface table
Iface             MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
ens33            1500      110      0      0 0           103      0      0      0 BMRU
lo              65536       20      0      0 0            20      0      0      0 LRU
```

其中 Iface（Interface）列会出了所有网络接口的名称。

#### 1. 动态地址

以 root 管理员身份登录 CentOS 修改网络配置文件 ifcfg-ens33：

- 将“ONBOOT=no”键值对改为“ONBOOT=yes”

这表示在虚拟机启动时，网络将自动连接。

```bash
# 系统一般自带vi编辑器
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

```shell
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp # 默认动态获取 IP 地址
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=20eed723-8202-4f1b-8c2d-60b840d93ca2
DEVICE=ens33
ONBOOT=yes # 将 no 改为 yes，表示开机时自动连接网络
```

重启网络服务（network.service），配置立即生效：

```bash
systemctl restart network
```

查看网络服务运行状态：

```bash
systemctl status network
```

查看当前 IP 地址：

```bash
ip addr
```

测试网络连接是否正常：

```bash
ping -c 4 www.baidu.com
```

#### 2. 静态地址

以 root 管理员身份登录 CentOS 修改网络配置文件 ifcfg-ens33：

- 将“ONBOOT=no”键值对改为“ONBOOT=yes”，这表示启动虚拟机时自动连接网络；
- 将“BOOTPROTO=dhcp”键值对改为“BOOTPROTO=static”，这表示使用静态 IP 地址，不再动态获取；
- 新增 IPADDR、GATEWAY、NETMASK 和 DNS1 等四个键值对，它们分别表示 IP 地址、默认网关、子网掩码和 DNS 服务器（可省略）。

```shell
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static # 设置为静态获取 IP 地址
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=7ccd63f4-34fb-4936-9863-df033bc849c6
DEVICE=ens33
ONBOOT=yes # 将 no 改为 yes，表示开机时自动连接网络

# 以下四条为新增，和 Window 系统中的规则一样，分别为 IP 地址、默认网关、子网掩码、DNS 服务器
IPADDR=192.168.1.110
GATEWAY=192.168.1.2
NETMASK=255.255.255.0
DNS1=192.168.1.2
```

重启网络服务（network.service），使配置生效：

```bash
systemctl restart network
```

推荐使用静态 IP 地址，以便宿主机通过 OpenSSH 连接到 Linux 系统。

#### 3. 故障排查

以下是几个在配置完静态 IP 地址后，系统网络仍旧出现异常可能原因：

- 默认网关配置错误；
- IP 地址存在冲突；
- 网络没有在系统启动时自动连接；
- 虚拟机设置中网络适配器未选择桥接模式，这会导致 IP 地址、默认网关等信息无法正确匹配当前的网络模式。

不管网络适配器处于什么模式，系统以动态 DHCP 获取 IP 地址时，一般能够解决绝大部分网络异常的问题。但非静态的 IP 地址始终会增加 Linux 系统作为服务端使用时的连接成本，请尽量使用静态 IP 地址。

### 远程连接

SSH 是最常见和最安全的远程控制方式之一，它提供了加密的 Shell 访问。可以使用 SSH 客户端从本地计算机远程连接到 Linux 服务器，并使用命令行执行操作。OpenSSH 是 SSH 协议的免费实现，通常预装在大多数 Linux 发行版中。

CentOS 系统同样预装了 OpenSSH，它同时提供了 SSH 的客户端和服务端，其中服务端大多数时候默认开机启动。

以下命令可查看 CentOS 下的 SSH 服务的运行状态：

```
# systemctl status sshd
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2023-09-03 19:10:07 CST; 8min ago
     Docs: man:sshd(8)
           man:sshd_config(5)
 Main PID: 976 (sshd)
   CGroup: /system.slice/sshd.service
           └─976 /usr/sbin/sshd -D

Sep 03 19:10:07 localhost.localdomain systemd[1]: Starting OpenSSH server daemon...
Sep 03 19:10:07 localhost.localdomain sshd[976]: Server listening on 0.0.0.0 port 22.
Sep 03 19:10:07 localhost.localdomain sshd[976]: Server listening on :: port 22.
Sep 03 19:10:07 localhost.localdomain systemd[1]: Started OpenSSH server daemon.
```

从输出中可以清晰看到，目前 CentOS 系统中的 sshd.service 服务处于运行中的状态，使用的端口号为 22。

Windows 系统下打开 PowerShell 终端，在安装了 OpenSSH 客户端的前提下，以下命令用于连接 CentOS 系统：

```bash
ssh '192.168.1.110' -l 'root'
```

连接时会要求输入指定用户的登录密码，本例以 root 的身份远程连接并登录 CentOS 系统：

<img src="images/CentOS%207.images/image-20230903192620178.png" alt="image-20230903192620178" style="zoom:50%;" />

连接命令可以简写成 scp 式：

```bash
ssh 'root@192.168.1.110'
```

如果不提供任何用户名，且不存在配置文件，则连接默认使用目前登录宿主机系统的用户名。

例如，Windows 系统下的如果登录的用户名为 dylan，那么：

```bash
ssh '192.168.1.110'
```

等价于：

```bash
ssh 'dylan@192.168.1.110'
```

### 防火墙

防火墙是保护机器不受来自外部垃圾流量骚扰的一种方式，它允许用户通过定义一组防火墙规则来控制主机上的入站流量（Inbound Traffic）。这些规则一般会对入站流量进行分类，以分类结果来判断需要阻断流量还是允许流量进站。

这里需要稍微说明一下入站流量和出站流量：

- 入站流量（Inbound Traffic）就是**流入**特定网络、服务器、网站或应用程序的数据流量；
- 出站流量（Outbound Traffic）就是从特定网络、服务器、网站或应用程序**流出**的数据流量。

<img src="images/CentOS%207.images/image-20230905223109517.png" alt="image-20230905223109517" style="zoom:50%;" />

其中，入站流量还可以译作 Incoming Traffic，出站流量可以译作 Outgoing Traffic。

firewalld（CentOS 7 新特性）是一个防火墙服务守护进程，它通过 D-Bus 接口提供动态的、可定制的主机防火墙。由于防火墙是动态的，这意味着在每次启用、修改或删除规则时不需要重启防火墙守护进程。

firewalld 使用区域（Zones）和服务（Services）两个概念来简化防火墙的流量管理：

- Zones 是预定义的规则集，规则集中定义了流量进站的规则，所有的网络接口（网卡）都可以分配给区域。流量是否被允许入站取决于当前计算机使用的网络接口及该接口所隶属的区域类型；
- Services 实际是包含在 Zones 里面的一些预定义规则，这些规则仅允许特定服务流量的入站。例如，允许 SSH 流量进站，只需要将 ssh 服务添加到 Services 中，不再需要使用命令去放行 22 端口。

CentOS 系统中默认存在 9 个不同的区域：

```
~]# firewall-cmd --get-zones
block dmz drop external home internal public trusted work
```

这里只需要了解 public 区域，该区域是网络接口被默认分配至的区域：

```
~]# firewall-cmd --get-default-zone
public
```

查询指定网络接口（网卡）所属区域的命令（例如 ens33 网卡）：

```
~]# firewall-cmd --get-zone-of-interface=ens33
public
```

查询指定区域其具体规则集的命令（例如 public 区域）：

```
~]# firewall-cmd --zone=public --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens33
  sources:
  services: dhcpv6-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

其中：

- interfaces：表示当前有哪些网络接口（网卡）遵循当前区域的规则集。例如 ens33 网卡；
- services：表示当前区域中允许哪些服务的流量入站。例如 ssh 服务；
- ports：表示当前区域中允许哪些端口的什么类型的流量入站，例如 22/tcp，它表示允许 22 端口的 TCP 流量入站；
- protocols：表示当前区域中允许哪些类型的流量入站。例如 tcp 或 udp 协议流量；
- forward-ports：表示当前区域中的端口转发配置。

注意命令中的 `--zone` 选项，该选项存在默认值 `<default-zone>`。本例中 public 是默认区域，那么以下两条命令在本例中等价：

```bash
firewall-cmd --zone=public --list-all
```

```bash
firewall-cmd --list-all
```

#### 1. 添加或移除服务

使用 firewalld 添加或移除端口的命令：

```bash
firewall-cmd --zone=public --add-service=<service-name>
firewall-cmd --zone=public --remove-service=<service-name>
```

尝试将 ssh 服务从 public 区域中移除：

```bash
firewall-cmd --zone=public --remove-service=ssh
```

再次查看 public 区域的详细信息：

```
~]# firewall-cmd --zone=public --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens33
  sources:
  services: dhcpv6-client
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

可以看到 ssh 服务已经不存在于 public 区域了，同时 OpenSSH 客户端也无法连接到 CentOS 系统：

<img src="images/CentOS%207.images/image-20230904034045564.png" alt="image-20230904034045564" style="zoom:50%;" />

注意，以上命令只能将 ssh 服务从 public 区域中**临时移除**。因为几乎所有涉及到修改区域规则集的命令，都需要进行保存才能持久生效。在防火墙服务或系统重启后，未保存的修改都会丢失。

如果希望修改能持久写入规则集中，可以使用以下命令保存修改：

```bash
firewall-cmd --zone=public --runtime-to-permanent
```

或在修改规则时加上 `--permanent` 选项：

```bash
firewall-cmd --zone=public --remove-service=ssh --permanent
```

添加服务到指定的区域中，需要先确认该服务是否存在于预定义服务（Predefined Services）列表中。

查看所有的预定义服务：

```bash
firewall-cmd --get-services
```

一些常用的服务诸如 redis、mysql、git、mssql、http 或 https 等一般都被包含在预定义服务列表中。

添加预定义服务的命令：

```bash
firewall-cmd --zone=public --add-service=<service-name> --permanent
```

查看某区域内所有已允许的服务：

```bash
firewall-cmd --zone=public --list-services
```

#### 2. 添加或移除端口

假如预定义服务列表中不存在目标服务，为了避免大费周章地添加自定义服务，还可以选择修改区域的端口配置：

```bash
firewall-cmd --zone=public --add-port=<port-number/port-type>
firewall-cmd --zone=public --remove-port=<port-number/port-type>
```

其中 `port-type` 可以是 tcp 或 udp 等。

假设 ssh 服务不存在于 public 区域中，已知 ssh 服务运行在 22 端口。那么可以选择将 22 端口添加到 public 区域中：

```bash
firewall-cmd --zone=public --add-port=22/tcp
```

这样 OpenSSH 客户端同样能够成功连接至 CentOS 系统。

查看某区域内所有已允许的端口：

```bash
firewall-cmd --zone=public --list-ports
```

#### 3.  防火墙相关命令

firewalld 本身只提供了查询状态和重新加载防火墙服务两种命令：

```bash
firewall-cmd --state
firewall-cmd --reload
```

显然在管理防火墙服务方面不太够用，这里推荐使用 systemctl 来管理 firewalld 服务。

查看防火墙状态：

```bash
systemctl status firewalld
```

重启防火墙服务：

```bash
systemctl restart firewalld
```

完全开启防火墙（命令需顺序执行）：

```bash
# 启用服务
systemctl unmask firewalld

# 启动服务
systemctl start firewalld

# 服务自启
systemctl enable firewalld
```

完全关闭防火墙（命令需顺序执行）：

```bash
# 关闭服务
systemctl stop firewalld

# 禁止自启
systemctl disable firewalld

# 禁用服务
systemctl mask firewalld
```

关于 `systemctl mask` 和 `systemctl unmask` 命令，它们用于控制 systemd 服务的禁用和启用状态。其中，`mask` 用于禁用服务，而 `unmask` 用于启用服务。这些命令对于管理系统中哪些服务应该在引导时启动非常有用，可以帮助提高系统的安全性和性能。

例如，使用 `systemctl mask` 命令可以禁用某些服务，该命令属于**完全**地禁用服务，这意味着其他服务无法关联启动这些被禁用的服务。

关于防火墙的其他内容，这里就不作深入的探究了。有兴趣可以阅读红帽子官方提供的、关于 Linux 7 中防火墙使用的指南：[Using Firewalls](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-using_firewalls)。

### 端口转发

firewalld 服务所提供的端口转发（[Port Forwarding](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-port_forwarding)）功能较为有趣，它能够将某些端口的流量重定向至本机或其他主机的其他端口上。这样做不仅可以隐匿某些服务的真实端口号，还能间接改变某些服务的标准端口号。

例如，有很多的服务器并不对外开放标准的 SSH 端口，这是出于系统安全性的考虑。诚然 SSH 是一种强大的远程访问协议，但伴随而来的是巨大的风险，能够访问 Shell 就意味着能够对系统作出更改，因此服务器管理员有必要采取各种措施，来减小潜在的安全风险。

<img src="images/CentOS%207.images/image-20230905232330692-1693944517771-1.png" alt="image-20230905232330692" style="zoom: 50%;" />

虽然多数情况下，将服务端口修改为非标准端口是为了获取更高的系统安全性，但不乏也有其他的例外。

熟悉 GitHub 的话，应该知道他们提供了一个使用 443 端口连接的 ssh.github.com 主机，其内部的实现原理虽不得而知，但这里用到了非标准的 SSH 端口。显然 GitHub 这样做的目的不是出于系统安全性的考虑，因为无论主机还是接口都仍然是处于完全公开的状态。

这里猜测其目的是为了预防使用标准 SSH 端口时出现的未知错误，一旦出现未知错误，新主机及非标准端口也许能派上用场。类似的未知错误：

```
kex_exchange_identification: Connection closed by remote host
Connection closed by UNKNOWN port 65535
```

最近一次出现该位置错误，是在使用代理服务器连接 GitHub 服务器时：

```bash
ssh -T 'git@github.com' -p 22 -o 'ProxyCommand "e:/Git/mingw64/bin/connect.exe" -S 127.0.0.1:13766 %h %p'
```

这无疑是一件非常反直觉的事情，使用代理服务器却无法连接 GitHub 着实是有点不可思议，但它就是发生了。同时，这个未知错误十分难以排查。错误显然是与身份验证相关的，但你永远无法知道到底是代理服务关闭了连接，还是远程服务器关闭了连接，更甚者还可能是连接在传输过程中意外被关闭。

这种情况下只能凭经验判断，最可能出现问题的环节是代理服务器。猜测代理服务器上使用了某种策略禁止 SSH 流量的中继，从而导致了连接错误。其禁止中继 SSH 流量的策略，可能仅是监测客户端需要访问的目标端口号是否为 22 端口。

如果真如猜测一般，GitHub 提供的 443 端口可能非常有用：

```bash
ssh -T 'git@ssh.github.com' -p 443 -o 'ProxyCommand "e:/Git/mingw64/bin/connect.exe" -S 127.0.0.1:13766 %h %p'
```

更换主机和端口后，本地客户端顺利连接上了 GitHub 服务器：

```
Hi dylan127c! You've successfully authenticated, but GitHub does not provide shell access.
```

尽管出现错误的真实原因仍旧无法得知，但无论如何，问题算是得到了解决，这恰好从侧面说明了非标准端口的作用。

一般服务都可以通过修改相关的配置文件以达到修改服务所使用的默认端口号的目的，这里就不过多说明。本篇主要了解一下如何使用端口转发功能，将任意服务的标准端口映射为非标准端口。

注意，firewalld 防火墙服务是 CentOS 7 的新特性，这意味着旧版本的 Centos 系统上不存在该服务。实际上，多数的 Linux 系统可以使用 iptables 工具来完成端口转发的配置。

#### 1. 大致原理

firewalld 服务所提供的端口转发功能实际上很容易理解，它本质是将指定端口所接收到的入站流量转发到本机或其他主机的特定端口上，特定端口上的服务在监听到流量并作出响应后，响应流量也将按原路返回至客户端。

较为简单的是本地端口转发：

<img src="images/CentOS%207.images/image-20230907033259431.png" alt="image-20230907033259431" style="zoom:50%;" />

这种端口转发仅相当于将本机某个开放端口上的流量，转发至其他不开放的端口。

假如服务端 `192.168.1.110` 支持 SSH 连接，那么本地客户端一般能使用以下命令建立连接：

```bash
ssh 'root@192.168.1.110' -p 22
```

但如果服务端 `192.168.1.110` 开启了端口转发，将 `38921` 端口的流量转发到本机的 `22` 端口：

```bash
firewall-cmd -add-forward-port=port=38921:proto=tcp:toport=22
```

并从 public 区域中剔除 ssh 服务，同时仅将 `38921` 端口添加至区域的 ports 中：

```bash
firewall-cmd --remove-service=ssh
firewall-cmd --add-port=38921/tcp
```

那么本地客户端如果希望再次与服务端 `192.168.1.110` 建立 SSH 连接，就需要稍微修改一下连接命令：

```bash
ssh 'root@192.168.1.110' -p 38921
```

这实际就是一个简单的端口转发实例，服务端将 `38921` 端口所接收的 SSH 流量转发给了 `22` 端口：

```
192.168.1.110:38921 ====port forward===> 192.168.1.110:22
```

端口转发除了支持将流量转发至本机，还支持将流量转发至其他主机：

<img src="images/CentOS%207.images/image-20230907033338947.png" alt="image-20230907033338947" style="zoom:50%;" />

假设服务端 `192.168.1.111` 是服务端 `192.168.1.110` 局域网中的一台主机，现在：

- 客户端 `192.168.1.188` 能够和服务端 `192.168.1.110` 建立连接，但无法和服务端 `192.168.1.111` 建立连接；
- 服务端 `192.168.1.110` 和 `192.168.1.111` 之间能够建立连接。

如果希望客户端能够连接至 `192.168.1.111` 服务端，那么可以选择在 `192.168.1.110` 服务端上配置端口转发：

```bash
firewall-cmd -add-forward-port=port=38922:proto=tcp:toport=22:toaddr=192.168.1.111
```

非本机的流量转发需要 masquerade 功能的支持：

```bash
firewall-cmd --add-masquerade
```

那么客户端只需要使用以下命令即可连接到 `192.168.1.111` 服务端上的 SSH 服务：

```bash
ssh 'root@192.168.1.110' -p 38922
```

这就相当于将 `192.168.1.110` 服务端上 `38922` 端口接收的流量，转发至 `192.168.1.111` 服务端的 `22` 端口：

```
192.168.1.110:38922 ====port forward===> 192.168.1.111:22
```

#### 2. 添加规则

为本机添加端口转发规则：

```bash
firewall-cmd --add-forward-port=port=<port-number>:proto=<tcp|udp|sctp|dccp>:toport=<port-number>
```

为其他主机添加端口转发规则：

```bash
firewall-cmd --add-forward-port=port=<port-number>:proto=<tcp|udp|sctp|dccp>:toport=<port-number>:toaddr=<IP>
```

```bash
firewall-cmd --add-masquerade
```

#### 3. 移除规则

移除为本机添加的端口转发规则：

```bash
firewall-cmd --remove-forward-port=port=<port-number>:proto=<tcp|udp|sctp|dccp>:toport=<port-number>
```

移除为其他主机添加的端口转发规则：

```bash
firewall-cmd --remove-forward-port=port=<port-number>:proto=<tcp|udp|sctp|dccp>:toport=<port-number>:toaddr=<IP>
```

```bash
firewall-cmd --remove-masquerade
```

#### 4. 注意事项

firewalld 提供的端口转发功能只适用于 IPv4 网络，对于 IPv6 网络来说，完成端口转发需要使用 rich rules，更多信息：[Rich Language](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/configuring_complex_firewall_rules_with_the_rich-language_syntax)。

其次是 masquerade 功能，为其他主机添加端口转发规则时需要开启该功能，更多信息：[Configuring Masquerading](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-configuring_ip_address_masquerading)。

### 端口占用

使用 Linux 部署服务时，偶尔会遇到端口被占的情况，以下是相关的处理命令：

```shell
# 查看目前监听的端口，如果没有 netstat，可以使用“yum -y install net-tools”进行安装
netstat -lnpt

# 检查端口被那个进程所占用
netstat -lnpt | grep 3306

# 查看进程详情，1140 是进程编号 PID（Process ID），通过上一步可以查询出占用特定端口程序的 PID
ps 1140

# 中止进程，参数 -9 表示强制杀死该进程
kill -9 1140
```

### 时间配置

默认情况下，通过以下命令可以查看本地时间：

```shell
date
```

CentOS 在选择“最小化安装”的情况下，通常会将本地时间设置为 UTC 时间。

UTC 时间即协调世界时，又称为世界标准时间，简称来自英文国际时间/法文协调时间“Universal Time/Temps Cordonné”。中国大陆、香港、澳门、台湾、蒙古国、新加坡、马来西亚、菲律宾、澳洲西部的时间与 UTC 的时差均为 +8，也就是 UTC+8。

在某些特殊的情况下，系统软件会使用到本地时间。对于处于 GMT+08:00 时区的中国来说，本地时间需为 CST 时间。

默认的本地时间文件为 /etc/localtime，通过以下命令查阅 localtime 文件详情：

```shell
ls -l /etc/localtime
```

你会发现它实际上是一个软链接（Symbolic Link），即等同于 Windows 系统下的快捷方式。

默认情况下，localtime 软链接所指向的文件是 /usr/share/zoneinfo/Etc/UTC，这正是本地时间为 UTC 的原因。

将 localtime 软链接进行备份，并创建一个新的 localtime 软链接指向 /usr/share/zoneinfo/Asia/ShangHai 或 /usr/share/zoneinfo/PRC：

```shell
# 使用 mv 命令对文件进行重命名
mv /etc/localtime /etc/localtime.bak

# 创建 PRC 文件的软链接 localtime
ln -s /usr/share/zoneinfo/PRC /etc/localtime
```

之后再查看本地时间，你会发现已经更改为 CST 了。
