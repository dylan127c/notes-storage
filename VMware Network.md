### 前言

本节将简单介绍如何在 VMware 中正确配置虚拟网络。

### 版本环境

- VMware Workstation 16 Pro（16.2.3 build-19376536）
- CentOS 7.9（CentOS-7-x86_64-DVD-2009）
- Clash Verge 1.3.8 | v1.16.0 Meta

### 虚拟网络

VMware - 设置 - 虚拟网络编辑器：

<div align="center"><img src="images/VMware%20Network.images/Snipaste_2024-07-14_23-24-20.png" alt="Snipaste_2024-07-14_23-24-20" style="width:50%;" /></div>

上述页面不提供编辑功能，需要点击“更改设置”以管理员身份进入虚拟网络编辑页面：

<div align="center"><img src="images/VMware%20Network.images/Snipaste_2024-07-14_23-24-51.png" alt="Snipaste_2024-07-14_23-24-51" style="width:50%;" /></div>

### 桥接模式（BRIDGE）

桥接模式一般是最常用、最简单的模式，它可以让虚拟机和宿主机使用相同的网卡访问网络，即此时的虚拟机相当于局域网中的另一台主机。

因此，桥接模式下虚拟机使用和宿主机一样的网关，同时虚拟机的 IP 地址和宿主机的 IP 地址位于同一 IP 段。为避免不必要的错误，推荐将“已连接至”配置为宿主机的物理显卡：

<div align="center"><img src="images/VMware%20Network.images/Snipaste_2024-07-14_23-25-10.png" alt="Snipaste_2024-07-14_23-25-10" style="width:60%;" /></div>

虚拟机设置中，将“网络适配器”设置为“桥接模式”：

<div align="center"><img src="images/VMware%20Network.images/Snipaste_2024-07-14_23-45-52.png" alt="Snipaste_2024-07-14_23-45-52" style="width:50%;" /></div>

#### 1. 动态地址

以 root 管理员身份登录 CentOS 修改 ifcfg-ens33 网络配置文件：

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
- 新增 `IPADDR`、`NETMASK`、`GATEWAY` 和 `DNS` 等键值对，它们分别表示 IP 地址、子网掩码、默认网关和 DNS 服务器，桥接模式下 IP 地址需要与宿主机位于同一网段，其他参数与宿主机一致。

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

### NAT 模式（NAT）

NAT 模式通过 VMnet8 虚拟交换机与宿主机及其网卡进行通讯，其中涉及三个虚拟设备：

1. 虚拟 NAT 设备：**负责与宿主机的网卡通讯，这个网卡也可以是宿主机的虚拟网卡；**
2. 虚拟 DHCP 服务器：**在虚拟机启用 DHCP 时，负责分配 IP 地址；**
3. 虚拟网卡：**负责与宿主机通讯，主要目的是让宿主机和虚拟机位于同一个网段。**

使用 NAT 模式，需要悉知 VMware 为 NAT 模式提供的子网 IP 及网关 IP 具体是什么。在“虚拟网络编辑器”中选中 VMnet8 模式，可以获取到子网 IP 及子网掩码等信息：

<div align="center"><img src="images/VMware%20Network.images/Snipaste_2024-07-14_23-26-33.png" alt="Snipaste_2024-07-14_23-26-33" style="width:50%;" /></div>

选中“NAT 设置”，可以获取到网关 IP 的信息：

<div align="center"><img src="images/VMware%20Network.images/Snipaste_2024-07-14_23-26-54.png" alt="Snipaste_2024-07-14_23-26-54" style="width:50%;" /></div>

获取到这些信息后，接下来可以进入虚拟机进行网络配置。

虚拟机设置中，将“网络适配器”设置为“NAT 模式”：

<div align="center"><img src="images/VMware%20Network.images/Snipaste_2024-07-14_23-46-52.png" alt="Snipaste_2024-07-14_23-46-52" style="width:50%;" /></div>

动态获取 IP 地址的方式不再赘述，后续只说明如何配置静态 IP 配置。

以 root 管理员身份登录 CentOS 修改网络配置文件 ifcfg-ens33：

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
IPADDR="192.168.139.110"
NETMASK="255.255.255.0"
GATEWAY="192.168.139.2"

DNS1="119.29.29.29"
DNS2="223.5.5.5"
```

重启网络服务（network.service），使配置生效：

```bash
systemctl restart network
```

NAT 模式的关键在于获取到正确的网关地址和 IP 地址段，上述配置完毕后，虚拟机已经可以访问网络。

宿主机和虚拟机之间的通讯，则依靠 VMware 提供的虚拟网卡完成：

<div align="center"><img src="images/VMware%20Network.images/Snipaste_2024-07-15_00-40-41.png" alt="Snipaste_2024-07-15_00-40-41" style="width:30%;" /></div>

负责 NAT 模式的虚拟网卡一般命名为 VMware Virtual Ethernet Adapter for VMnet8。宿主机和虚拟机之间如果需要建立 SSH 通讯，则需要保证宿主机和虚拟机在同一个局域网内。

一般 VMware 会自动为宿主机分配 IP 地址，以确保它和虚拟机能够处于同一个局域网内：

<div align="center"><img src="images/VMware%20Network.images/Snipaste_2024-07-15_01-17-45.png" alt="Snipaste_2024-07-15_01-17-45" style="width:80%;" /></div>

但不排除 VMware 会出现未可知的错误，如果与虚拟机同一网段的 IP 地址未能顺利配置到 VMnet8 虚拟网卡内，则需要手动为该网卡分配 IP 地址。

### 主机模式（HOST-ONLY）

主机模式相当于阉割了虚拟 NAT 设备的 NAT 模式，这种模式下虚拟机只能和宿主机之间进行通讯，由于无法访问宿主机的网卡，因此该模式下无法使用网络。

主机模式下，只需要保证虚拟机和宿主机位于同一个网段即可。以 root 管理员身份登录 CentOS 修改网络配置文件 ifcfg-ens33：

```shell
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"

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

ONBOOT="yes"

IPADDR="192.168.19.110"
NETMASK="255.255.255.0"
```

重启网络服务（network.service），使配置生效：

```bash
systemctl restart network
```

主机模式下，宿主机和虚拟机之间的通讯，同样依靠 VMware 提供的虚拟网卡完成：

<div align="center"><img src="images/VMware%20Network.images/Snipaste_2024-07-15_01-26-50.png" alt="Snipaste_2024-07-15_01-26-50" style="width:30%;" /></div>

负责该模式的虚拟网卡一般命名为 VMware Virtual Ethernet Adapter for VMnet1。

同样地，VMware 会自动为宿主机分配 IP 地址，以确保它和虚拟机能够处于同一个局域网内：

<div align="center"><img src="images/VMware%20Network.images/Snipaste_2024-07-15_01-18-16.png" alt="Snipaste_2024-07-15_01-18-16" style="width:80%;" /></div>

### TUN 模式

宿主机上启用 TUN 模式，可以用于解决虚拟机访问外网困难的问题。例如使用 docker 拉取镜像时，往往需要借用代理服务来访问目标网址以下载资源。

VMware 提供的众多虚拟网络模式中，排除本身就无法访问网络的主机模式后，剩余的两种模式中仅有 NAT 支持使用宿主机提供的 TUN 代理服务：

```
~]# curl -I www.google.com
HTTP/1.1 200 OK
...
```

因此实际应用中，如果存在使用宿主机提供的 TUN 代理服务的需求时，则更推荐使用 NAT 模式。