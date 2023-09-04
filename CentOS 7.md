### 概述

本篇记录 CentOS 7 Minimal 版本安装后的一些基本配置。

当前使用的 CentOS 为 7.9 版本。CentOS 系统下可以使用以下命令查看系统具体的发行版版本信息：

```bash
cat /etc/centos-release
```

输出如下：

```
CentOS Linux release 7.9.2009 (Core)
```

### 环境要求

- VMware Workstation 16 Pro（16.2.3 build-19376536）
- CentOS 7（CentOS-7-x86_64-DVD-2009）

### 安装

在 VMware Workstation 中选择 CentOS 7 镜像，类型选择“最小化安装”即可。

CentOS 系统安装完毕或完成基本网络、软件配置之后，建议对虚拟机进行“保存快照”操作，以便在系统崩溃或重建坏境时能迅速利用快照对恢复系统。注意，创建快照务必在虚拟机处于“关机”状态下进行，处于“运行”或“挂起”状态的虚拟机所生成的快照虽然可以回溯，但无法克隆。

### 网络

多数情况下，选择 CentOS 系统是为了完成某些软件的模拟，例如测试 Redis、MySQL 等。无一例外，基本都会选择使用 OpenSSH 来连接 CentOS 系统。

本地客户端需要连接到 CentOS 虚拟机，那么 CentOS 系统必须首先拥有可用网络。由于某些 CentOS 版本默认不提供网络服务，这就不可避免地需要了解如何手动配置 CentOS 系统的网络。

首先，务必确认 VMware 软件“编辑 - 虚拟网络编辑器 - 更改设置”下拥有三种不同的网络：

<img src="images/CentOS%207.images/image-20230903171816941.png" alt="image-20230903171816941" style="zoom:50%;" />

同时注意类型为桥接模式的 VMnet0 网络，确认其所桥接的网卡为当前物理主机所使用的网卡。

其次，推荐将虚拟机设置中网络适配器一项修改为桥接模式（自动），这样方便后续设置静态 IP 地址：

<img src="images/CentOS%207.images/image-20230903185131946.png" alt="image-20230903185131946" style="zoom:50%;" />

CentOS 系统的网络配置文件一般存储在 `/etc/sysconfig/network-scripts/` 目录下，文件名称由特定前缀 `ifcfg-` 和网络接口（Network Interface）名称两部分共同组成。其中，网络接口名称的格式会因 Linux 发行版、系统配置和网络设备的不同而有所不同，CentOS 系统中的网络接口名称一般为 ens33。

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

由此可知当前 CentOS 系统的网络配置文件将被命名为 ifcfg-ens33。

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

其中 Iface 列会出了所有网络接口的名称。

#### 动态 IP 地址

以 root 管理员身份登录 CentOS 系统，修改网络配置文件 ifcfg-ens33：

- 只需要将“ONBOOT=no”键值对改为“ONBOOT=yes”即可，这表示启动虚拟机时自动连接网络。

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

重启网络服务（network.service），使配置生效：

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

#### 静态 IP 地址

以 root 管理员身份登录 CentOS 系统，修改网络配置文件 ifcfg-ens33：

- 将“ONBOOT=no”键值对改为“ONBOOT=yes”，这表示启动虚拟机时自动连接网络；
- 将“BOOTPROTO=dhcp”键值对改为“BOOTPROTO=static”，这表示使用静态 IP 地址，不再动态获取；
- 新增 IPADDR、GATEWAY、NETMASK 和 DNS1 等四个键值对，它们分别表示 IP 地址、默认网关、子网掩码和 DNS 服务器。

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

推荐使用静态 IP 地址，以方便远程主机连接到 Linux 系统。

### 远程连接

SSH 是最常见和最安全的远程控制方式之一，它提供了加密的 Shell 访问。您可以使用 SSH 客户端从本地计算机远程连接到 Linux 服务器，并使用命令行执行操作。OpenSSH 是 SSH 协议的免费实现，通常预装在大多数 Linux 发行版中。

毫无疑问，CentOS 系统同样预装了 OpenSSH，它提供了 SSH 的客户端和服务端，其中 SSH 服务端大多数时候为默认开机启动的状态。

以下命令可查看 CentOS 系统下 SSH 服务的运行状态：

```bash
systemctl status sshd
```

一般输出如下：

```
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

Windows 系统下打开 PowerShell 终端，在安装了 OpenSSH 客户端的前提下可以使用以下命令连接 CentOS 系统：

```bash
ssh '192.168.1.110' -l 'root'
```

连接时会要求输入指定用户的登录密码，本例以 root 的身份远程连接并登录 CentOS 系统：

<img src="images/CentOS%207.images/image-20230903192620178.png" alt="image-20230903192620178" style="zoom:50%;" />

### 防火墙

CentOS 7 系统中使用了全新的防火墙软件 firewalld 以取代 iptables 工具，它带来了区域（Zone）的概念。区域本质上是针对进入系统的流量而建立规则列表，其中每个网络接口（Network Interface）都必须至少分配至某个区域中。

多数情况下，修改防火墙的配置等同于修改区域的配置。以下命令用于查看所有的防火墙区域：

```bash
firewall-cmd --get-zones
```

默认情况下 CentOS 包含 9 个区域，但目前仅需要了解 public 区域。public 是默认区域，每个网络接口都会被优先分配至该区域：

```bash
firewall-cmd --get-default-zone
```

查看指定网络接口目前被分配到的区域，可以使用以下命令：

```bash
firewall-cmd --get-zone-of-interface=<interface-name>
```

例如，获取网络接口 ens33 目前被分配到的区域：

```bash
firewall-cmd --get-zone-of-interface=ens33
```

查看 public 区域的详细信息：

```bash
firewall-cmd --zone=public --list-all
```

注意，参数 `--zone` 不提供时，它会默认被填充为 `--zone=<default-zone>`。目前的默认区域为 public，即意味着以上命令等价于：

```bash
firewall-cmd --list-all
```

<b>但建议尽量提供</b> `--zone` <b>参数，以免不必要的错误。</b>以上命令的输出如下：

```
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

- interfaces：表示目前分配至 public 区域的网络接口；
- services：表示目前已被允许直接通过 public 区域的服务。

可以看到 ssh 服务隶属于 public 区域中被允许的服务之一，这意味着所有来自 OpenSSH 的流量都可以直接通过防火墙而不被阻截。

#### 1. 添加或移除服务

使用 firewalld 不可避免需要接触到添加或移除端口的命令：

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

可以看到 ssh 服务已经不存在于 public 区域，同时 OpenSSH 客户端也无法连接到 CentOS 系统：

<img src="images/CentOS%207.images/image-20230904034045564.png" alt="image-20230904034045564" style="zoom:50%;" />

所幸使用这种方式只能临时将 ssh 服务从 public 区域中移除，重启防火墙服务或系统后，public 区域的规则将会恢复。

如果希望永久移除 ssh 服务，则需要在移除 ssh 服务后再执行一条命令：

```bash
firewall-cmd --zone=public --runtime-to-permanent
```

该命令会将当前 public 区域的规则作为默认规则永久写入，即便重启防火墙服务或系统，新写入的规则也不会丢失。

其次，添加某些服务到 public 区域中，需要先确认服务存在于预定义服务（Predefined Services）列表中。以下命令用于查看所有的预定义服务：

```bash
firewall-cmd --get-services
```

一些常用的服务诸如 redis、mysql、git、mssql、http 或 https 等均包含在这个预定义服务列表中。有需要可以将某些服务添加至 public 区域中：

```bash
firewall-cmd --zone=public --add-service=<service-name>
firewall-cmd --zone=public --runtime-to-permanent
```

查看所有已允许的服务可以使用以下命令：

```bash
firewall-cmd --zone=public --list-services
```

#### 2. 添加或移除端口

如果预定义服务列表中不存在你需要的服务，那么还可以选择开放端口。添加或移除端口的命令如下：

```bash
firewall-cmd --zone=public --add-port=<port-number/port-type>
firewall-cmd --zone=public --remove-port=<port-number/port-type>
```

假设 ssh 服务不存在于 public 区域，但已知 ssh 服务运行在 22 端口。那么可以选择直接将 22 端口添加到 public 区域中：

```bash
firewall-cmd --zone=public --remove-service=ssh
firewall-cmd --zone=public --add-port=22/tcp
```

这样 OpenSSH 客户端同样能够成功连接至 CentOS 系统。

查看所有已允许的端口可以使用以下命令：

```bash
firewall-cmd --zone=public --list-ports
```

#### 3. 将规则永久写入

几乎所有添加或移除服务、端口的命令都是临时生效的命令，如果希望规则能永久写入区域中，则需要在执行这些命令后运行以下命令：

```bash
firewall-cmd --zone=public --runtime-to-permanent
```

该命令会将当前 public 区域的规则作为默认规则永久写入，即便重启防火墙服务或系统，新写入的规则也不会丢失。但每次持久化规则都需要使用两条命令是十分繁琐的，因此 firewalld 还提供了另一种方式将规则永久写入：`--permanent` 选项。

在每次执行添加或移除服务、端口的命令时加上 `--permanent` 选项，规则就能即刻永久写入区域中。例如，永久添加 22 端口：

```bash
firewall-cmd --zone=public --add-port=22/tcp --permanent
```

或永久移除 ssh 服务：

```bash
firewall-cmd --zone=public --remove-service=ssh --permanent
```

#### 4. 防火墙相关命令

推荐使用 systemctl 来管理 firewalld 服务。

查看防火墙状态的命令如下：

```bash
systemctl status firewalld
```

重启防火墙的命令如下：

```bash
systemctl restart firewalld
```

开启防火墙命令如下：

```bash
# 启用 firewalld 服务
systemctl unmask firewalld

# 启动 firewalld 服务
systemctl start firewalld

# 自启 firewalld 服务
systemctl enable firewalld
```

关闭防火墙命令如下：

```bash
# 关闭 firewalld 服务
systemctl stop firewalld

# 禁止 firewalld 自启
systemctl disable firewalld

# 禁用 firewalld 服务
systemctl mask firewalld
```

关于 `systemctl mask` 和 `systemctl unmask` 命令，它们用于控制 systemd 服务的禁用和启用状态。其中，`mask` 用于禁用服务，而 `unmask` 用于启用服务。这些命令对于管理系统中哪些服务应该在引导时启动非常有用，可以帮助提高系统的安全性和性能。

例如，使用 `systemctl mask` 命令可以禁用某些服务，该命令属于**完全**地禁用服务，这意味着其他服务无法关联启动这些被禁用的服务。

关于防火墙的其他内容，这里就不作深入的探究了。有兴趣可以阅读红帽子官方提供的、关于 Linux 7 中防火墙使用的指南：[Using Firewalls](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-using_firewalls)。

### 端口转发

端口转发是一个比较有意思的内容，这里单独拿出来说明一下，该功能同样是 firewalld 提供的，端口转发能够将某些端口的流量重定向至其他端口，以达到间接更改某些服务端口的目的，例如将 SSH 协议的端口修改为非标准端口。

实际上，有很多服务器并不开放标准的 SSH 端口（Port 22），这是出于系统安全性的考虑。诚然 SSH 是一种强大的远程访问协议，但伴随而来的同样是巨大的风险，能够访问 Shell 就意味着能够对系统作出修改，服务器管理员有必要采取各种措施，来减小潜在的安全风险。

多数情况下，将服务端口修改为非标准端口是为了获取更高的系统安全性，但也不乏有其他的例外。

熟悉 GitHub 的话，应该知道他们提供了一个使用 443 端口连接的 ssh.github.com 主机，其内部的实现原理虽不得而知，但总之它使用了非标准的 SSH 端口。显然 GitHub 这样做的目的不是出于系统安全性的考虑，因为无论主机还是接口都仍然是处于完全公开的状态。

这里猜测其目的是为了防止使用标准的 SSH 端口时出现不必要的错误，所以才提供了另一个主机及非标准的端口的连接方式。类似这样的错误：

```
kex_exchange_identification: Connection closed by remote host
Connection closed by UNKNOWN port 65535
```

以上错误出现在使用代理服务器连接 GitHub 服务器时：

```bash
ssh -T git@github.com -p 22 -o 'ProxyCommand "e:/Git/mingw64/bin/connect.exe" -S 127.0.0.1:13766 %h %p'
```

这无疑是一件非常反直觉的事情，使用代理服务器却无法连接 GitHub 服务器着实有点不可思议，但它就是发生了。最为离谱的是这个错误十分难以排查，且明明一切看起来都十分正常。

无奈只能凭经验判断，而最可能出现问题的环节是代理服务器。猜测是代理服务器上使用了某种策略禁止了 SSH 流量的中继，从而导致了连接错误。说到禁止中继 SSH 流量的策略，那么也可能只是简单地监测客户端需要访问的目标端口号是否为 22 端口。

如果真如猜测一般，那么 GitHub 提供的 443 端口可能非常有用：

```bash
ssh -T git@ssh.github.com -p 443 -o 'ProxyCommand "e:/Git/mingw64/bin/connect.exe" -S 127.0.0.1:13766 %h %p'
```

在更换主机和端口后，本地客户端顺利连接上了 GitHub 服务器：

```
Hi dylan127c! You've successfully authenticated, but GitHub does not provide shell access.
```

尽管出现错误的真实原因不得而知，但总算问题得到了解决，显然非标准端口有些时候也是十分有用的。

#### 直接更改

许多服务否可以通过修改相关的配置文件，来更改服务的默认端口号，例如修改 OpenSSH 的默认端口。

找到 OpenSSH 配置文件并打开：

```bash
vi /etc/ssh/sshd_config
```

在 OpenSSH 配置文件中找到 `#Port 22` 并将其修改为：

```
Port 443
```

重启 OpenSSH 服务后配置即刻生效：

```
systemctl restart sshd
```

#### 间接更改

利用 firewalld 提供的端口转发功能，可以达到间接修改服务端口的目的。例如，使用 firewalld 的端口转发功能实现 SSH 的非标准端口访问。

从 public 区域中剔除 ssh 服务：

```bash
firewall-cmd --zone=public --remove-service=ssh --permanent
```

将 443/tcp 添加到 public 区域中，同时确保 22/tcp 不存在于 public 区域中：

```bash
firewall-cmd --zone=public --add-port=443/tcp --permanent
firewall-cmd --zone=public --remove-port=22/tcp --permanent
```

配置端口转发，具体命令如下：

```bash
firewall-cmd --zone=public --add-forward-port=port=443:proto=tcp:toport=22
```

配置完成后，可以选择查看一下 public 区域的详细信息：

```
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens33
  sources:
  services: dhcpv6-client
  ports: 443/tcp
  protocols:
  masquerade: no
  forward-ports: port=443:proto=tcp:toport=22:toaddr=
  source-ports:
  icmp-blocks:
  rich rules:
```

客户端即可通过 443 端口远程连接至 CentOS 系统：

<img src="images/CentOS%207.images/image-20230904060031328.png" alt="image-20230904060031328" style="zoom:50%;" />

#### 端口占用

端口被占用时可以执行如下操作：

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



但如果你是在 CentOS 中部署 Docker，将 MySQL 数据库服务部署在 Docker 容器内，尽管宿主机 3306 端口映射容器内 3306 端口，但此时仍旧不需要开放 3306 端口，MySQL 数据库服务依然能被访问。
