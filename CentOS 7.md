### 概述

本篇记录 CentOS 7 Minimal 版本安装后的一些基本配置。

查看系统具体的发行版：

```bash
~]# cat /etc/centos-release
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

关机或重启：

```bash
# 立即关机
shutdown -h now

# 立即重启
shutdown -r now
```

另一个重启命令：

```bash
reboot
```

如果不添加 now 参数，那么系统将会向所有用户发送一个关机或重启通知，并在一定时间后关机或重启。这段时间通常是一分钟，但具体取决于系统配置：

```bash
# 等待一分钟后关机
shutdown -h

# 等待一分钟后重启
shutdown -r 
```

如果已经运行了 shutdown 命令且未添加 now 参数，那么可以在一定的时间内取消关机或重启：

```bash
shutdown -c
```

### 权限问题

配置的过程中几乎都需要使用到管理员权限，这里建议切换为 root 用户来执行以下操作。普通用户在 CentOS 7 中想要获取管理员权限会稍微有点麻烦，因为 CentOS 7 默认并不允许普通用户使用 sudo 命令。

要解决这个问题，需要管理员手动将指定普通用户添加到 sudoers 文件中。假设 mike 为普通用户，这里要将其添加到 sudoers 文件中。以管理员身份登录到系统，或者使用拥有 sudo 权限的用户登录，接着编辑 sudoers 文件：

```bash
sudo visudo
```

在打开的文件中找到以下行：

```
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
```

在该行的下方添加以下行，以允许用户 mike 使用 sudo 命令：

```
mike    ALL=(ALL)       ALL
```

保存并退出编辑器。现在，用户 mike 应该可以使用 sudo 命令并输入自己的密码来获取管理员权限了。

### 网络问题

多数情况下，选择 CentOS 系统是为了完成某些软件的模拟，例如测试 Redis、MySQL 等。为了便于使用命令行操作，一般会使用 OpenSSH 连接 CentOS 系统。

宿主机如果需要连接到 CentOS 虚拟机，那么 CentOS 必须首先拥有可用网络。由于某些 CentOS 版本默认不提供网络服务，这不可避免地需要了解如何手动配置 CentOS 系统的网络。

首先，请务必确认 VMware 软件“编辑 - 虚拟网络编辑器 - 更改设置”下拥有三种不同的网络：

<div align="center"><img src="images/CentOS%207.images/image-20230903171816941.png" alt="image-20230903171816941" style="width:50%;" /></div>

注意，类型为“桥接模式”的 VMnet0 网络，必须确保它所桥接的网卡为当前宿主机所使用的网卡。

上图中“桥接模式-已桥接至-自动”表示 VMware 会自动选择桥接的目标网卡，大部分情况下，选择“自动”是没有问题的。但不排除一些特殊情况，这里强烈建议手动配置“已桥接至”的具体网卡。

<div align="center"><img src="images/CentOS%207.images/Snipaste_2024-03-11_09-33-54.png" alt="Snipaste_2024-03-11_09-33-54" style="width:50%;" /></div>

其次，推荐将虚拟机设置中网络适配器修改为“桥接模式（自动）”，这便于后续为虚拟机配置静态 IP 地址：

<div align="center"><img src="images/CentOS%207.images/image-20230903185131946.png" alt="image-20230903185131946" style="width:80%;" /></div>

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

由此可知当前 CentOS 的网络配置文件的命名为 ifcfg-ens33。有些 Linux 系统的网络配置文件的命名为 ifcfg-eth0，后续需要根据实际情况来选择修改指定的配置文件。

除此之外，系统如果安装了 net-tools 工具集，那么也可以使用以下命令查看网络接口：

```bash
netstat -i
```

一般 CentOS 自带 net-tools 工具。如果上述过程中没有找到 netstat 命令，则需要先安装 net-tools 工具：

```bash
yum install -y net-tools
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

```bash
# 系统一般自带vi编辑器
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

- 将 `ONBOOT` 改为 `yes`，这表示启动虚拟机时自动连接网络；
- 将 `BOOTPROTO` 改为 `dhcp`，这表示使用动态 IP 地址，即自动获取 IP。

```shell
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"

# 默认动态获取 IP 地址
BOOTPROTO="dhcp"

DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="a45d8cc4-ee39-40ee-8c62-28b128d06e71"
DEVICE="ens33"

# 将 no 改为 yes，表示开机时自动连接网络
ONBOOT="yes"
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

- 将 `ONBOOT` 改为 `no`，这表示启动虚拟机时自动连接网络；
- 将 `BOOTPROTO` 改为 `static`，这表示使用静态 IP 地址；
- 新增 `IPADDR`、`NETMASK`、`GATEWAY` 和 `DNS` 等键值对，它们分别表示 IP 地址、子网掩码、默认网关和 DNS 服务器。

```shell
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"

# 设置为静态获取 IP 地址
BOOTPROTO="static"

DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="a45d8cc4-ee39-40ee-8c62-28b128d06e71"
DEVICE="ens33"

# 将 no 改为 yes，表示开机时自动连接网络
ONBOOT="yes"

# 以下几条为新增项，分别为 IP 地址、子网掩码、默认网关、DNS 服务器
IPADDR="192.168.1.110"
NETMASK="255.255.255.0"
GATEWAY="192.168.1.2"

# 不同于 Windows 系统，DNS 似乎必须要配置
DNS1="119.29.29.29"
DNS2="223.5.5.5"
```

重启网络服务（network.service），使配置生效：

```bash
systemctl restart network
```

推荐使用静态 IP 地址，以便远程主机通过 OpenSSH 连接到 Linux 系统。

后续可以使用 CentOS 7 中自带的 sed 文本处理工具，来快速修改 `IPADDR` 或 `GATEWAY` 项（修改概率较高）。

例如修改 `IPADDR`：

```bash
sudo sed -i 's/^IPADDR=.*/IPADDR="192.168.1.112"/' /etc/sysconfig/network-scripts/ifcfg-ens33
```

或者修改 `GATEWAY`：

```bash
sudo sed -i 's/^GATEWAY=.*/GATEWAY="192.168.1.1"/' /etc/sysconfig/network-scripts/ifcfg-ens33
```

同样的，其他参数亦可以使用 sed 工具进行修改，请自行选择合适的方式。

#### 3. 故障排查

以下是几个在配置完静态 IP 地址后，系统网络仍旧出现异常可能原因：

- 默认网关配置错误；
- IP 地址存在冲突；
- 网络没有在系统启动时自动连接；
- 虚拟机设置中网络适配器未选择桥接模式，这会导致 IP 地址、默认网关等信息无法正确匹配当前的网络模式；
- 虚拟机网络配置中，桥接模式未选择正确的网卡（默认自动选择网卡有时候会导致无法连接的情况）；
- 存在多个以 `ifcfg-ens33` 开头命名的配置文件，某些系统可能将错误的配置文件识别为网络配置文件。

关于最后一点，假如 `network-scripts` 目录下同时存在 ifcfg-ens33 和 ifcfg-ens33-bridge 文件，原计划是前者负责 NAT 模式，后者负责 BRIDGE 模式，只在需要切换模式时再重新命名配置文件。

理想的情况下，系统应当识别 ifcfg-ens33 为网络配置文件。但实际情况并非如此，在采用 NAT 模式为网络适配器时，实测 CentOS 7.9 系统会错误地将 ifcfg-ens33-bridge 识别为网络配置文件，导致系统崩溃。

这也许是内核为了提高检索配置效率简化了代码导致的，无论如何，对于网络配置文件的命名需要谨慎。

最后，不管网络适配器处于什么模式，系统以动态 DHCP 获取 IP 地址时，一般能够解决绝大部分网络异常的问题。但动态 IP 地址始终会增加 Linux 系统作为服务端使用时的连接成本，请尽量选择静态 IP 地址。

### 生命周期

任何系统都有生命周期（EOL）的概念，在生命周期终止时，官方源会一并下架。

CentOS Linux 7 于 2024 年 6 月 30 日终止生命周期，因此在这个日期之后，官方源将不再适用。但下述关于“换源”的讨论，仍旧适用于其他 CentOS 版本，可作参考。

访问[阿里巴巴开源镜像站](https://developer.aliyun.com/mirror/)，选择 centos 类目可以找到 CentOS 7 相关的换源指南，这里不再赘述。

### 换源问题

过去较长的一段时间内会选择使用“换源”的方式，来提高 CentOS 中软件包的下载速度：

```bash
# step 1: 将默认的 yum 源备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak

# step 2: 下载阿里源并将其命名为 CentOS-Base.repo
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

# step 3：可以刷新一下源
sudo yum makecache
```

但 CentOS 7 中提供了 `fastestmirror` 插件功能。当 `fastestmirror` 插件启用时，`yum` 会检测及测量可用镜像源的速度，并根据速度选择最快的镜像源进行下载。

这个插件的作用是优化 `yum` 在选择镜像源时的性能，以提高软件包下载速度。通常情况下，`fastestmirror` 插件启用后，`yum` 在下载软件包时会更快速、更有效率。

在 `fastestmirror` 插件存在时，传统的换源工作可以省略。虽然换源步骤可以省略，但不意味着 `.repo` 文件是没用的配置。实际上 `fastestmirror` 插件需要根据 `/etc/yum.repos.d` 目录中的文件内容来筛选镜像服务器。

插件 `fastestmirror` 通过以下步骤选择最快的镜像服务器：

1. **获取镜像服务器列表**：`fastestmirror` 会定期从 CentOS 官方仓库或用户配置中获取镜像服务器列表；
2. **评估镜像服务器**：`fastestmirror` 会根据以下因素评估每个镜像服务器：
   - **距离**：`fastestmirror` 会选择距离您最近的镜像服务器，以减少网络延迟；
   - **负载**：`fastestmirror` 会选择负载最小的镜像服务器，以提高下载速度；
   - **优先级**：用户可以配置镜像服务器的优先级，`fastestmirror` 会优先选择优先级高的镜像服务器。
3. **选择最优镜像服务器**：`fastestmirror` 会综合考虑以上因素，选择最优的镜像服务器。

大多数情况下，`fastestmirror` 插件是默认安装并启用的。在 CentOS 或其他基于 Red Hat 的发行版中，默认情况下会安装并启用该插件。这意味着使用 `yum` 命令时，`fastestmirror` 插件会自动生效，它将帮助您选择最快的镜像源来下载软件包。

可以通过检查 `/etc/yum.conf` 文件来确认插件是否启用。在这个文件中，应该会有类似以下的配置行：

```bash
plugins=1
```

这个配置表明启用了插件系统。如需禁用，可以编辑 `/etc/yum.conf` 文件，并在其中添加 `plugins=0` 的配置行来禁用插件系统。

如果系统中不存在 `fastestmirror` 插件，可以选择使用 `yum` 命令安装：

```bash
yum install -y yum-plugin-fastestmirror
```

如果使用私有源，那么可选卸载 `fastestmirror` 插件或禁用 `yum` 的插件系统：

```bash
yum remove -y yum-plugin-fastestmirror
```

另外，如果既不需要使用 `fastestmirror` 插件也不想替换源，那么在遇到网络问题时，则可能需要为 `yum` 命令提供可用的代理服务：

```bash
echo "proxy=http://192.168.1.188:13766" >> /etc/yum.conf
```

### 远程连接

SSH 是最常见和最安全的远程控制方式之一，它提供了加密的 Shell 访问。可以使用 SSH 客户端从本地计算机远程连接到 Linux 服务器，并使用命令行执行操作。OpenSSH 是 SSH 协议的免费实现，通常预装在大多数 Linux 发行版中。

CentOS 系统同样预装了 OpenSSH，它同时提供了 SSH 的客户端（CLIENT）和服务端（SERVER），其中服务端大多数时候默认开机启动。

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

从输出中可以得知，CentOS 系统中的 sshd.service 服务处于运行中的状态，使用的端口号为 22。

Windows 系统下打开 PowerShell 终端，在安装了 OpenSSH 客户端的前提下，以下命令用于连接 CentOS 系统：

```bash
ssh '192.168.1.111' -l 'root'
```

连接时会要求输入指定用户的登录密码，本例以 root 的身份远程连接并登录 CentOS 系统：

<div align="center"><img src="images/CentOS%207.images/image-20230903192620178.png" alt="image-20230903192620178" style="width:80%;" /></div>

连接命令可以简写成 scp 式：

```bash
ssh 'root@192.168.1.111'
```

如果不提供任何用户名，且不存在配置文件，则连接默认使用目前登录宿主机系统的用户名。

假设 Windows 系统下 dylan 为当前系统登录名，那么以下两条命令等价：

```bash
ssh '192.168.1.111'
```

```bash
ssh 'dylan@192.168.1.111'
```

### 防火墙

防火墙是保护机器不受来自外部垃圾流量骚扰的一种方式，它允许用户通过定义一组防火墙规则来控制主机上的入站流量（Inbound Traffic）。这些规则一般会对入站流量进行分类，以分类结果来判断需要阻断流量还是允许流量进站。

这里需要稍微说明一下入站流量和出站流量：

- 入站流量（**Inbound Traffic**）就是**流入**特定网络、服务器、网站或应用程序的数据流量；
- 出站流量（**Outbound Traffic**）就是从特定网络、服务器、网站或应用程序**流出**的数据流量。

<div align="center"><img src="images/CentOS%207.images/image-20230905223109517.png" alt="image-20230905223109517" style="width:42%;" /></div>

其中，入站流量还可以译作 **Incoming Traffic**，出站流量可以译作 **Outgoing Traffic**。

`firewalld`（CentOS 7 新特性）是一个防火墙服务守护进程，它通过 D-Bus 接口提供动态的、可定制的主机防火墙。由于防火墙是动态的，这意味着在每次启用、修改或删除规则时不需要重启防火墙守护进程。

`firewalld` 使用区域（Zones）和服务（Services）两个概念来简化防火墙的流量管理：

- Zones：预定义的规则集，规则集中定义了流量进站的规则，所有的网络接口（网卡）都可以分配给区域。流量是否被允许入站取决于当前计算机使用的网络接口及该接口所隶属的区域类型；
- Services：实际包含在 Zones 里面的一些预定义规则，这些规则仅允许特定服务流量的入站。例如，允许 SSH 流量进站，只需要将 ssh 服务添加到 Services 中，不再需要使用命令去放行 22 端口。

CentOS 系统中默认存在 9 个不同的区域：

```
~]# firewall-cmd --get-zones
block dmz drop external home internal public trusted work
```

这里只需要了解 public 区域，该区域是网络接口被分配至的**默认区域**：

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

- interfaces：当前有哪些网络接口（网卡）遵循当前区域的规则集。例如 ens33 网卡；
- services：当前区域中允许哪些服务的流量入站。例如 ssh 服务；
- ports：当前区域中允许哪些端口的什么类型的流量入站，例如 22/tcp，它表示允许 22 端口的 TCP 流量入站；
- protocols：当前区域中允许哪些类型的流量入站。例如 tcp 或 udp 协议流量；
- forward-ports：当前区域中的端口转发配置。

注意命令中的 `--zone` 选项，该选项存在默认值 `<default-zone>`。本例中 public 是默认区域，那么以下两条命令在本例中等价：

```bash
firewall-cmd --zone=public --list-all
```

```bash
firewall-cmd --list-all
```

注意，后续所有涉及到修改区域规则集的命令，都需要进行保存才能持久生效。**对于未保存的修改，防火墙服务或系统重启后，这些未保存的修改都会丢失。**

如果需要修改永久生效，则需要再修改规则时，加上 `--permanent` 选项：

```bash
firewall-cmd --zone=public --remove-service=ssh --permanent
```

#### 1. 添加或移除服务

使用 `firewalld` 添加或移除端口的命令：

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

可以看到 ssh 服务已经不存在于 public 区域了，同时远程主机无法使用 OpenSSH 连接到 CentOS 系统：

<div align="center"><img src="images/CentOS%207.images/image-20230904034045564.png" alt="image-20230904034045564" style="width:80%;" /></div>

#### 2. 预定义服务列表

在添加服务到指定的区域之前，需要确认该服务存在于预定义服务（Predefined Services）列表中。

查看所有的预定义服务：

```bash
firewall-cmd --get-services
```

一些常用的服务诸如 redis、mysql、git、mssql、http 或 https 等一般都被包含在预定义服务列表中。

添加服务（需存在于预定义服务列表中）的命令：

```bash
firewall-cmd --zone=public --add-service=<service-name>
```

查看某区域内所有已允许的服务：

```bash
firewall-cmd --zone=public --list-services
```

#### 3. 添加或移除端口

假如预定义服务列表中不存在目标服务，如果想避免大费周章地添加自定义服务，可以选择为区域新增入站端口：

```bash
firewall-cmd --zone=public --add-port=<port-number/port-type>
```

移除入站端口：

```bash
firewall-cmd --zone=public --remove-port=<port-number/port-type>
```

其中 `port-type` 可以是 `tcp` 或 `udp`。

假设 ssh 服务不存在于 public 区域中，但已知 ssh 服务运行在 22 端口。那么可以选择将 22 端口添加到 public 区域中：

```bash
firewall-cmd --zone=public --add-port=22/tcp
```

这样远程主机同样可以使用 OpenSSH 连接至 CentOS 系统。

查看某区域内所有已允许的端口：

```bash
firewall-cmd --zone=public --list-ports
```

#### 4.  防火墙相关命令

`firewalld` 本身只提供了查询状态和重新加载防火墙服务两种命令：

```bash
# 查询防火墙状态
firewall-cmd --state
# 重新加载防火墙
firewall-cmd --reload
```

在管理防火墙服务方面显然不太够用，这里推荐使用 `systemctl` 来管理 `firewalld` 服务。

`systemctl` 是一个系统和服务管理工具，用于管理 systemd 系统和服务。它是在许多现代 Linux 发行版中用于替代传统的 `service` 和 `chkconfig` 命令的工具。`systemctl` 允许您查看和控制系统状态以及启动、停止、重启和管理服务。

查看防火墙状态：

```bash
systemctl status firewalld
```

重启防火墙服务：

```bash
systemctl restart firewalld
```

完全开启防火墙（**<u>命令需顺序执行</u>**）：

```bash
# 启用服务
systemctl unmask firewalld

# 启动服务
systemctl start firewalld

# 服务自启
systemctl enable firewalld
```

完全关闭防火墙（**<u>命令需顺序执行</u>**）：

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

`firewalld` 还提供了端口转发（Port Forwarding）网络技术，用于将进入防火墙的网络流量从一个端口转发到另一个端口或另一台计算机上的指定端口。端口转发在网络配置中具有多种用途，包括但不限于以下几点：

1. **提供服务访问：** 通过端口转发，您可以将外部网络上的请求转发到内部网络中运行的服务上。例如，您可以将 Web 服务器的入口端口（例如 80 端口）映射到内部网络中运行的 Web 服务器上，从而允许外部用户访问该 Web 服务器；
2. **负载均衡：** 端口转发可以用于负载均衡。您可以将进入防火墙的流量转发到多台服务器上，从而实现流量的分发和负载均衡。这有助于提高服务的可用性和性能；
3. **隐藏内部网络结构：** 通过端口转发，您可以隐藏内部网络结构，将外部网络的访问请求转发到内部网络中的指定服务器上，而不必暴露整个内部网络结构；
4. **代理服务：** 您可以使用端口转发来创建代理服务，将外部网络上的请求转发到其他网络上的指定服务上。这种方法可以帮助您实现内容过滤、安全监控等功能。

以 OpenSSH 为例，将标准的 SSH 端口 22 修改为非标准的端口，除了可以直接修改 OpenSSH 配置之外，还可以选择使用 `firewalld` 提供的端口转发技术。

<div align="center"><img src="images/CentOS%207.images/image-20230905232330692-1693944517771-1.png" alt="image-20230905232330692-1693944517771-1" style="width: 75%;" /></div>

注意，`firewalld` 防火墙服务是 CentOS 7 的新特性，这意味着旧版本的 Centos 系统上不存在该服务。实际上，多数的 Linux 系统可以使用 `iptables` 工具来完成相关的端口转发配置，只是过程比较复杂。

#### 1. 大致原理

`firewalld` 提供的端口转发技术实际上很容易理解，其本质是将指定端口所接收到的入站流量转发到本机或本机可访问的其他主机的特定端口上，服务在监听特定端口的流量后将作出响应，响应将按原路返回至请求客户端处。

简单的端口转发：

<div align="center"><img src="images/CentOS%207.images/image-20230907033259431.png" alt="image-20230907033259431" style="width:50%;" /></div>

这种端口转发仅相当于将本机某个开放端口上的流量，转发至其他不开放的端口。

假如服务端 `192.168.1.110` 支持 SSH 连接，那么本地客户端一般能使用以下命令建立连接：

```bash
ssh 'root@192.168.1.110' -p 22
```

但如果服务端 `192.168.1.110` 开启了端口转发，将 `38921` 端口的流量转发到本机的 `22` 端口：

```bash
firewall-cmd -add-forward-port=port=38921:proto=tcp:toport=22
```

同时从 public 区域中剔除 ssh 服务，仅将 `38921` 端口添加至区域的 ports 中：

```bash
firewall-cmd --remove-service=ssh
firewall-cmd --add-port=38921/tcp
```

那么本地客户端如果希望再次与服务端 `192.168.1.110` 建立 SSH 连接，则需要稍微修改一下连接命令：

```bash
ssh 'root@192.168.1.110' -p 38921
```

这实际就是一个简单的端口转发实例，服务端将 `38921` 端口所接收的 SSH 流量转发给了 `22` 端口：

```
192.168.1.110:38921 ====port forward===> 192.168.1.110:22
```

端口转发除了支持将流量转发至本机，还支持将流量转发至其他主机：

<div align="center"><img src="images/CentOS%207.images/image-20230907033338947.png" alt="image-20230907033338947" style="width:50%;" /></div>

假设服务端 `192.168.1.111` 是服务端 `192.168.1.110` 局域网中的一台主机，现在：

|     IP ADDR     |     IPADDR      | CONNECTION STATUS |
| :-------------: | :-------------: | :---------------: |
| `192.168.1.188` | `192.168.1.110` |         ✅         |
| `192.168.1.188` | `192.168.1.111` |         ❌         |
| `192.168.1.110` | `192.168.1.111` |         ✅         |

如果希望 `192.168.1.188` 客户端能够连接至 `192.168.1.111` 服务端，那么可以选择在 `192.168.1.110` 服务端上配置端口转发：

```bash
firewall-cmd -add-forward-port=port=38922:proto=tcp:toport=22:toaddr=192.168.1.111
```

非本机的流量转发需要 masquerade 功能的支持：

```bash
firewall-cmd --add-masquerade
```

那么 `192.168.1.188` 客户端只需要使用以下命令即可连接到 `192.168.1.111` 服务端上的 SSH 服务：

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

**注意，其中 `:` 符号仅用作 `--add-forward-port` 参数内容的分隔符，无其他特殊含义。**

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

`firewalld` 提供的端口转发功能只适用于 IPv4 网络，对于 IPv6 网络来说，完成端口转发需要使用 rich rules，更多信息：[Rich Language](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/configuring_complex_firewall_rules_with_the_rich-language_syntax)。

其次是 masquerade 功能，为其他主机添加端口转发规则时需要开启该功能，更多信息：[Configuring Masquerading](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-configuring_ip_address_masquerading)。

### 端口占用

使用 Linux 部署服务时，偶尔会遇到端口被占的情况，以下是相关的处理命令：

```shell
# step 1: 检查 3306 端口被什么进程编号 PID 占用
netstat -lnpt | grep 3306

# step 2: 根据进程编号 PID 查看进程详情
ps 1140

# step 3: 中止进程，根据 PID 使用参数 kill -9 强制杀死进程
kill -9 1140
```

常用命令：

```bash
# 查看所有监听的端口
netstat -lnpt
```

### 时间配置

如果在 CentOS 安装时使用的是 Minimal 镜像，那么本地时间大概率会被设置为 UTC 时间。

查看本地时间：

```shell
~]# date
Sun Mar 3 09:37:17 UTC 2023
```

UTC 时间即协调世界时，又称为世界标准时间，简称来自英文国际时间/法文协调时间“Universal Time/Temps Cordonné”。中国大陆、香港、澳门、台湾、蒙古国、新加坡、马来西亚、菲律宾、澳洲西部的时间与 UTC 的时差均为 +8，也就是 UTC+8。

在某些特殊的情况下，系统软件需要校验本地时间。对于处于 GMT+08:00 时区的中国来说，本地时间最好配置为正确的 CST 时间。

默认的本地时间文件为 `/etc/localtime`，查阅 localtime 文件详情：

```shell
~]# ls -l /etc/localtime
lrwxrwxrwx. 1 root root 35 Jul  7  2022 /etc/localtime -> ../usr/share/zoneinfo/Etc/UTC
```

可以发现它实际上是一个软链接（Symbolic Link）。其中，软连接可以理解为 Windows 系统下的快捷方式。

输出中显示 `/etc/localtime` 软链接所指向的文件是 `/usr/share/zoneinfo/Etc/UTC`，这正是本地时间为 UTC 的原因。

修改本地时间为 CST 时间，先备份 localtime 文件。之后再新建一个 localtime 软链接指向以下任意文件：

- `/usr/share/zoneinfo/Asia/ShangHai`（*推荐）
- `/usr/share/zoneinfo/PRC`

```shell
# step 1: 备份 localtime 文件
mv /etc/localtime /etc/localtime.bak

# step 2: 创建 Asia/ShangHai 或 PRC 文件的软链接
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

再次查看，可以发现本地时间已经变更为 CST 时间：

```bash
~]# date
Sun Mar  3 17:37:17 CST 2023
```

### HTTP(S) 代理

CentOS 中推荐使用环境变量来配置代理（不仅仅只支持以下代理）：

```bash
export HTTP_PROXY="http://192.168.1.188:13766"
export HTTPS_PROXY="http://192.168.1.188:13766"
export NO_PROXY="localhost,127.0.0.1"
```

其中：

- `HTTP_PROXY`：HTTP 代理的地址和端口；
- `HTTPS_PROXY`：HTTPS 代理的地址和端口；
- `NO_PROXY`：不使用代理的地址。

注意，上述代理配置为临时配置，系统重启后配置会失效，且**只有 HTTP 和 HTTPS 请求才会使用该代理**。对于其他协议的请求，例如 FTP、SOCKS 等，代理不会生效；再如 DNS 解析或 YUM 安装等，代理也不会生效。

这就是为什么不能使用 `ping` 来验证代理可用性的原因：

```
ping -c 4 www.google.com
```

因为 `ping` 命令并不是使用 HTTP 协议。

测试连通性，可以选择使用 `curl -I` 命令。例如验证是否可以顺利访问 Google 网站：

```bash
~]# curl -I https://www.google.com
HTTP/1.1 200 Connection established

HTTP/1.1 200 OK
Content-Type: text/html; charset=ISO-8859-1
Content-Security-Policy-Report-Only: object-src 'none';base-uri 'self';script-src 'nonce-WhQTNLb8XFujGqzTddOoag' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other-hp
P3P: CP="This is not a P3P policy! See g.co/p3phelp for more info."
Date: Mon, 04 Mar 2024 00:11:13 GMT
Server: gws
X-XSS-Protection: 0
X-Frame-Options: SAMEORIGIN
Transfer-Encoding: chunked
Expires: Mon, 04 Mar 2024 00:11:13 GMT
Cache-Control: private
Set-Cookie: 1P_JAR=2024-03-04-00; expires=Wed, 03-Apr-2024 00:11:13 GMT; path=/; domain=.google.com; Secure
Set-Cookie: AEC=Ae3NU9PNSUKlronqPUgATMZbD7YOyzBXdBhAPFV2qrO_NLlyGFUD4bUzyQg; expires=Sat, 31-Aug-2024 00:11:13 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=lax
Set-Cookie: NID=512=RpnU5cSC8aRYmAvWIQl2vI3keqY4gfEt0UkW6HMGISIR_HEtTfPjYYsw12kO09CfR0V4D3XCunurk2WUa7xJCO8YXPUrRsV8I-tTBRe4l0nK8p9rF8mBVbjmYpKAx4o-oHgCqLywOzx08wOpokL2GL1sDMZ1qPx-zm6iZbQePnA; expires=Tue, 03-Sep-2024 00:11:13 GMT; path=/; domain=.google.com; HttpOnly
Alt-Svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
```

状态码 200 即为可连通状态。

特别注意，VMWare 在使用桥接模式的时候，没有办法使用 TUN 模式，因此流量无法经过代理，普通 `curl` 命令还是需要配置环境变量来改变虚拟机流量的导向。

注意只是改变流量的导向，如果规则没有顺利让 `google.com` 网站使用代理服务，那么也将无法建立连接。例如规则模式下无合适规则，导致匹配了 DIRECT 策略。

最后，以下命令可查看环境变量中是否成功配置了相关的代理：

```bash
~]# env | grep -i proxy
NO_PROXY=localhost,127.0.0.1
HTTPS_PROXY=http://192.168.1.188:13766
HTTP_PROXY=http://192.168.1.188:13766
```

系统环境变量配置的代理其局限性较大，除了支持协议少之外，很多特定工具实际上用不了这些代理，最典型的一个就是 docker 容器工具。

docker 虽支持代理，但它无法直接使用系统环境变量所简单配置的代理，为 docker 工具配置代理要从 systemd 配置入手，需要进行特殊的配置。
