### SSH 隧道

SSH 端口转发技术，也被称为 SSH 隧道（SSH Tunnel）。它允许通过加密的 SSH 连接在本地和远程计算机之间安全地传输数据。在不信任的网络上，可以使用 SSH 隧道安全地传输数据，无需担心数据被窃听或篡改。这使得它成为远程访问、安全浏览和网络安全等领域的重要工具。

在 SSH 中，主要有三种类型的端口转发：

1. 本地端口转发
2. 远程端口转发
3. 动态端口转发

这三种类型的端口转发提供了灵活的网络连接方式，可以帮助用户在不同的网络环境中实现安全的远程访问通信。

本地端口转发和远程端口转发的概念容易让人混淆，这里需要记住几个要点：

1. 无论是本地还是远程的端口转发，执行转发命令的都是 SSH Client；
2. 本地端口转发，本质是从 SSH Client 所在主机上发起访问请求，SSH Client 默认启用网关模式；
3. 远程端口转发，本质是从 SSH Server 所在主机上发起访问请求，SSH Server 默认不启用网关模式。

是否启用网关模式决定了 SSH Client 或 SSH Server 所在主机 IP 暴露在公网时，其他的主机是否能够直接通过该公网 IP 来访问该主机。让 SSH Server 启用网关模式需要修改 `sshd_config` 配置文件：

```bash
echo "GatewayPorts yes" >> sshd_config
```

其次是动态端口转发，该转发同样由 SSH Client 发起。它实际就是在本地主机上开启一个 SOCKS 协议代理，让所有使用该 SOCKS 代理的网络流量，都能通过 SSH Tunnel 转发至远程主机上。

这样一来，本地主机就能实现“通过远程主机的网络来访问某些网站”的目的。动态端口转发的概念较为简单，但存在一个难点：如何验证 SOCKS 代理是否存在且可用。

SSH 协议目前来说十分地安全，因此动态端口转发基本没办法在 SSH Server 所在的远程主机上，通过命令查询的方式，来直接验证是否存在来自本地主机的 SOCKS 类型流量。

本篇将在远程主机上启用代理服务，并结合代理工具的日志，间接判断动态端口转发是否部署成功。

### 版本环境

- Windows 11 23H2 22631.3007
- VMware Workstation 16 Pro（16.2.3 build-19376536）
- CentOS 7.9（CentOS-7-x86_64-DVD-2009）
- ProxyCap 5.38
- Clash Verge 1.3.8 | v1.16.0 Meta

### 本地端口转发

本地端口转发（Local Port Forwarding）也称为本地转发或本地映射。它允许将本地主机上的某个端口转发到远程主机的一个指定地址和端口。

本地主机上的应用程序连接到本地端口时，数据被加密并通过 SSH 隧道转发到远程主机。远程主机将收到的数据发送到目标地址和端口，并将响应发送回本地主机。

<div align="center"><img src="images/SSH%20Tunnel.images/local-port-forwarding-2000-opt.png" alt="SSH Tunnels visualized - local port forwarding." style="width: 80%;" /></div>

基本命令格式：

```bash
ssh -L '[local_addr:]local_port:remote_addr:remote_port' '[user@]sshd_addr'
```

其中 `local_addr` 可以是：

- `localhost`：表示本地主机。等价于不提供 `local_addr` 参数或 `127.0.0.1`；
- `0.0.0.0`：表示任意主机。其他主机可以通过本地主机 IP 来使用本地端口转发。

注意，这里的 `remote_addr:remote_port` 链接总是交由 SSH Server 所在的 `Server` 路由。由于 `Web Server` 服务位于 `Server` 上，因而 `remote_addr` 可以直接使用 `localhost` 或 `127.0.0.1` 指代。

关于 `remote_addr` 的配置：

- 如果 `Server` 和 `Web Server` 在同一个主机上，那么 `remote_addr` 可以配置为 `localhost`、`127.0.0.1` 或 `Server` 可正常访问到 `Web Server` 的 IP 地址；
- 如果 `Server` 和 `Web Server` 不在一个主机上，那么 `remote_addr` 只能配置为 `Server` 可正常访问到 `Web Server` 的 IP 地址。

#### 1. 基本转发

<div align="center"><img src="images/SSH%20Tunnel.images/local-port-forwarding-2000-opt.png" alt="SSH Tunnels visualized - local port forwarding." style="width: 80%;" /></div>

以上图例中，`Client` 需要访问的是位于 `Server` 中的 `Web Server` 网络服务。

模拟类似场景：

|      SYSTEM      |      IP ADDR       |    ROLE    |  OpenSSH   |
| :--------------: | :----------------: | :--------: | :--------: |
|    Windows 11    |  `192.168.1.188`   |   Client   | SSH Client |
|     CentOS 7     |  `192.168.1.112`   |   Server   | SSH Server |
| CentOS 7 - Nginx | `192.168.1.112:80` | Web Server |     -      |

由 `Client` 使用 <u>**SSH Client**</u> 建立本地端口转发：

```bash
ssh -L '8080:localhost:80' 'root@192.168.1.112'
```

其中要求 `Server` 具备 <u>**SSH Server**</u>，同时 `Server` 中部署了对应的 `Web Server` 服务：

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-04_10-33-13-1709519609965-5.png" alt="Snipaste_2024-03-04_10-33-13" style="width:80%;" /></div>

使用 `docker` 简单部署 `nginx` 服务的命令：

```bash
docker run --name 'nginx' -d -p '80:80' 'nginx'
```

那么转发建立后，`Client` 即可通过 `8080` 端口，访问到 `Server` 中部署在 `80` 端口的 `Web Server` 服务：

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-04_10-38-02-1709519899582-8.png" alt="Snipaste_2024-03-04_10-38-02" style="width:80%;" /></div>

上述例子中，可连接的 SSH Server 位于 `Server` 端上。

#### 2. 进阶转发

但如果 `Server` 处于私有网络，即 `Client` 无法直接与 `Server` 建立连接，那么则需要部署 `Server` 的上游服务器 `Bastion` 以提供 SSH Server 来配合完成本地端口转发。

<div align="center"><img src="images/SSH%20Tunnel.images/local-port-forwarding-bastion-2000-opt.png" alt="img" style="width: 80%;" /></div>

模拟类似场景：

|      SYSTEM      |      IP ADDR       |    ROLE    |  OpenSSH   |
| :--------------: | :----------------: | :--------: | :--------: |
|    Windows 11    |  `192.168.1.188`   |   Client   | SSH Client |
|     CentOS 7     |  `192.168.1.111`   |  Bastion   | SSH Server |
|     CentOS 7     |  `192.168.1.112`   |   Server   |     -      |
| CentOS 7 - Nginx | `192.168.1.112:80` | Web Server |     -      |

由 `Client` 使用 <u>**SSH Client**</u> 建立本地端口转发：

```bash
ssh -L '8080:192.168.1.112:80' 'root@192.168.1.111'
```

其中要求 `Bastion` 具备 <u>**SSH Server**</u>，且能够直接访问 `Server` 部署的 `Web Server` 服务：

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-04_11-21-28.png" alt="Snipaste_2024-03-04_11-21-28" style="width:80%;" /></div>

那么转发建立后，`Client` 同样可通过 `8080` 端口，访问到 `Server` 中部署在 `80` 端口的 `Web Server` 服务。

### 远程端口转发

远程端口转发（Remote Port Forwarding）也称为远程转发或远程映射。它允许将远程主机上的某个端口转发到本地主机的一个指定地址和端口。

远程主机上的应用程序连接到远程端口时，数据被加密并通过 SSH 隧道转发到本地主机。本地主机将收到的数据发送到目标地址和端口，并将响应发送回远程主机。

<div align="center"><img src="images/SSH%20Tunnel.images/remote-port-forwarding-2000-opt.png" alt="SSH隧道可视化 - 远程端口转发。" style="width:80%;" /></div>

基本命令格式：

```bash
ssh -R '[remote_addr:]remote_port:local_addr:local_port' '[user@]gateway_addr'
```

其中 `remote_addr` 可以是：

- `localhost`：表示远程主机。该值等价于不提供 `local_addr` 参数或 `127.0.0.1`；
- `0.0.0.0`：表示任意主机。其他主机可以通过远程主机 IP 来使用远程端口转发。

关于 `local_addr` 的配置：

- 如果 `Client` 和 `Web Server` 在同一个主机上，那么 `local_addr` 可以配置为 `localhost`、`127.0.0.1` 或 `Client` 可正常访问到 `Web Server` 的 IP 地址；
- 如果 `Client` 和 `Web Server` 不在一个主机上，那么 `local_addr` 只能配置为 `Client` 可正常访问到 `Web Server` 的 IP 地址。

另外，远程端口转发中 `remote_addr` 即便为 `0.0.0.0`，其他主机仍无权使用 `Gateway` 端的 SSH 隧道。

因为本地端口转发中 `Client` 内运行的 SSH Client 默认启用网关模式，所以其他主机可以使用 `Client` 作为网关以访问目标服务，但本例中 `Gateway` 运行的 SSH Server 默认未启用网关模式。

只有启用网关模式时， `Gateway` 才可以作为其他主机的网关。这里需要额外开启 SSH Server 的网关模式：

```bash
echo 'GatewayPorts yes' >> sshd_config
```

完成配置后其他主机即可通过 `Gateway` 使用 SSH 隧道连接至 `Client` 内的 `Web Server` 网络服务。

#### 1. 基本转发

<div align="center"><img src="images/SSH%20Tunnel.images/remote-port-forwarding-2000-opt.png" alt="SSH隧道可视化 - 远程端口转发。" style="width:80%;" /></div>

以上图例中，`Gateway` 需要访问的是位于 `Client` 中的 `Web Server` 网络服务。

模拟类似场景：

|         SYSTEM          |      IP ADDR       |        ROLE         |  OpenSSH   |
| :---------------------: | :----------------: | :-----------------: | :--------: |
|       Windows 11        |  `192.168.1.188`   |       Client        | SSH Client |
|        CentOS 7         |  `192.168.1.112`   |       Gateway       | SSH Server |
| Windows11 - http-server | `192.168.1.188:80` | Client - Web Server |     -      |

由 `Client` 端使用 <u>**SSH Client**</u> 建立远程端口转发：

```bash
ssh -R '8080:localhost:80' 'root@192.168.1.112'
```

其中要求 `Gateway` 端具备 <u>**SSH Server**</u>，同时 `Client` 中部署了对应的 `Web Server` 服务：

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-04_12-03-08.png" alt="Snipaste_2024-03-04_12-03-08" style="width: 80%;" /></div>

本例使用 `http-server` 模拟 `Web Server` 服务，但存在一个问题：

- 建立远程端口转发时，如果 `local_addr` 使用 `localhost` 则会导致 `Web Server` 不可访问。

但该问题由 `http-server` 导致，原本的远程端口转发配置并没有任何问题。

为了模拟能继续进行，可以将 `local_addr` 配置为 `127.0.0.1` 回环地址或 `Client` 的真实 IP 地址：

```bash
ssh -R '8080:127.0.0.1:80' 'root@192.168.1.112'

ssh -R '8080:192.168.1.188:80' 'root@192.168.1.112'
```

那么转发建立后，`Gateway` 即可通过 `8080` 端口，访问到 `Client` 中部署在 `80` 端口的 `Web Server` 服务：

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-04_12-10-35.png" alt="Snipaste_2024-03-04_12-10-35" style="width:80%;" /></div>

#### 2. 进阶转发

另一种常见情况是 `Client` 与 `Web Server` 不在同一个主机上，其中 `Web Server` 在 `Server` 上部署。但显然， `Client` 和 `Server` 之间能够进行正常的网络通信。

<div align="center"><img src="images/SSH%20Tunnel.images/remote-port-forwarding-home-network-2000-opt.png" alt="SSH Tunnels visualized - remote port forwarding to home network." style="width:80%;" /></div>

模拟类似场景：

|         SYSTEM          |      IP ADDR      |        ROLE         |  OpenSSH   |
| :---------------------: | :---------------: | :-----------------: | :--------: |
|       Windows 11        |  `192.168.1.188`  |       Client        | SSH Client |
|        CentOS 7         |  `192.168.1.112`  |       Gateway       | SSH Server |
| Windows11 - http-server | `192.168.25.1:80` | Server - Web Server |     -      |

这里直接使用 `http-server` 提供的不同访问 IP 地址来模拟 `Server` 端。

由 `Client` 使用 <u>**SSH Client**</u> 建立远程端口转发：

```bash
ssh -R '8080:192.168.25.1:80' 'root@192.168.1.112'
```

其中要求 `Gateway` 具备 <u>**SSH Server**</u>，且 `Client` 能够直接访问 `Server` 部署的 `Web Server` 服务：

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-04_12-25-11.png" alt="Snipaste_2024-03-04_12-25-11" style="width:80%;" /></div>

那么转发建立后，`Gateway` 同样可通过 `8080` 端口，访问到 `Server` 中部署在 `80` 端口的 `Web Server` 服务：

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-04_12-28-25.png" alt="Snipaste_2024-03-04_12-28-25" style="width:80%;" /></div>

### 动态端口转发

动态端口转发（Dynamic Port Forwarding）也称为 SOCKS 代理转发，它是三种端口转发里最容易理解和最容易部署的。它允许将本地主机上的一个端口转发到远程主机，并在本地主机上创建一个 SOCKS 代理服务器。

本地主机上的应用程序可以通过该代理服务器将流量发送到远程主机，并由远程主机转发到目标地址。动态端口转发通常用于通过远程主机访问互联网，或者在局域网中访问受限制的资源。

基本命令格式：

```bash
ssh -D '[local_addr:]local_port' '[user@]sshd_addr'
```

模拟类似场景：

|   SYSTEM   |     IP ADDR      |  ROLE  |  OpenSSH   |
| :--------: | :--------------: | :----: | :--------: |
| Windows 11 | `192.168.1.188`  | Client | SSH Client |
|  CentOS 7  | `192.168.16.128` | Server | SSH Server |

这种情况下，需要在 SSH Client 所在本地主机 `192.168.1.188` 上执行动态端口转发：

```bash
ssh -D '57921' 'root@192.168.16.128'
```

由于 CentOS 7 中不存在任何直接的命令，可用于确认动态端口转发是否部署成功，这里采用间接验证的方式：

- 让远程主机 `192.168.16.128` 具有全局的代理服务；
- 让本地主机 `192.168.1.188` 使用 SOCKS 代理访问 [Google](https://www.google.com/) 网站；

那么，如果能在远程主机使用的代理服务日志中，找到相关的连接记录，则表明动态端口转发部署成功。

#### 1. 网络配置

远程主机使用 VMware 模拟，其中的网络适配器需要选择 NAT 模式：

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-04_23-40-50.png" alt="Snipaste_2024-03-04_23-40-50" style="width:50%;" /></div>

这里需要查看 VMware 中“虚拟网络编辑器 => NAT 模式类型”里的“DHCP 设置”和“NAT 设置”：

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-04_23-42-19.png" alt="Snipaste_2024-03-04_23-42-19" style="width:30%;" /></div>

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-04_23-41-46.png" alt="Snipaste_2024-03-04_23-41-46" style="width:50%;" /></div>

从中获取准确的 IP 地址范围、网关 IP 等配置：

- IPADDR：`192.168.16.128` ~ `192.168.16.254`
- GATEWAY：`192.168.16.2`

根据以上配置修改 CentOS 虚拟机的网络配置：

```bash
 ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens33
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

IPADDR="192.168.16.128"
NETMASK="255.255.255.0"
GATEWAY="192.168.16.2"
DNS1="119.29.29.29"
DNS2="223.5.5.5"
```

重启网络服务：

```bash
systemctl restart network
```

查看 IP 是否正确：

```
~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:de:2a:04 brd ff:ff:ff:ff:ff:ff
    inet 192.168.16.128/24 brd 192.168.16.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::ec6:7986:f8a:cef6/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

测试网络连通性：

```
~]# ping -c 4 www.baidu.com
PING www.a.shifen.com (120.232.145.144) 56(84) bytes of data.
64 bytes from 120.232.145.144 (120.232.145.144): icmp_seq=1 ttl=128 time=10.3 ms
64 bytes from 120.232.145.144 (120.232.145.144): icmp_seq=2 ttl=128 time=10.3 ms
64 bytes from 120.232.145.144 (120.232.145.144): icmp_seq=3 ttl=128 time=10.7 ms
64 bytes from 120.232.145.144 (120.232.145.144): icmp_seq=4 ttl=128 time=10.3 ms

--- www.a.shifen.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 10.311/10.452/10.791/0.221 ms
```

#### 2. 代理服务

VMware 中的 NAT 模式依赖于宿主机的网络。这意味着，如果宿主机使用虚拟网卡开启代理服务，那么对应使用 NAT 模式的虚拟机网络即等同于开启了代理服务。

本篇使用 Clash Verge 代理转发工具在宿主机上启用 TUN 模式，以部署虚拟网卡接管所有网络流量。

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-05_00-14-11.png" alt="Snipaste_2024-03-05_00-14-11" style="width:60%;" /></div>

启用 TUN 模式后，回到虚拟机中测试网络连通性：

```
~]# curl -I https://www.google.com
HTTP/1.1 200 OK
...
```

命令 `curl` 访问 Google 返回的状态码是 200，即表示虚拟机中的代理服务处于可用状态。

#### 3. 动态转发

完成以上网络和代理的配置之后，需要宿主机（本地主机）配置动态端口转发以开启本地的 SOCKS 代理。

|   SYSTEM   |     IP ADDR      |  ROLE  |  OpenSSH   |
| :--------: | :--------------: | :----: | :--------: |
| Windows 11 | `192.168.1.188`  | Client | SSH Client |
|  CentOS 7  | `192.168.16.128` | Server | SSH Server |

注意，出于网络结构的考虑，本节后续将称本地主机为宿主机，称远程主机为虚拟机。

动态端口转发需要使用 SSH Client 进行配置，即在宿主机上进行：

```
> ssh -D 57921 root@192.168.16.128
root@192.168.16.128's password:
Last login: Mon Mar  4 23:51:08 2024 from 192.168.16.1
[root@localhost ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:de:2a:04 brd ff:ff:ff:ff:ff:ff
    inet 192.168.16.128/24 brd 192.168.16.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::ec6:7986:f8a:cef6/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

这样，宿主机即可使用 `57921` 端口上的 SOCKS 代理服务了。

#### 4. 使用代理

宿主机中，使用 SwitchyOmega 插件配置 SOCKS 代理：

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-05_01-02-03.png" alt="Snipaste_2024-03-05_01-02-03" style="width:80%;" /></div>

随后选中该代理服务，即可访问 [Google](https://www.google.com) 网站：

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-05_01-05-35.png" alt="Snipaste_2024-03-05_01-05-35" style="width:80%;" /></div>

#### 5. 查阅日志

在使用 SOCKS 代理访问 [Google](https://www.google.com) 网站时，可以查阅 Clash Verge 的连接日志：

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-05_01-08-11.png" alt="Snipaste_2024-03-05_01-08-11" style="width:60%;" /></div>

可以看到访问 Google 网站的流量来自于 `vmnat.exe` 程序，即表示请求从虚拟机中发出。

这意味着本地的访问请求通过 SSH 动态端口转发提供的 SOCKS 代理，从宿主机上使用 SSH Tunnel 到达了虚拟机中，之后该请求再交由虚拟机发出，即动态端口转发是有效的。

#### 6. 其他方式

如果宿主机这种全局代理 TUN 模式无法使用，那么还可以选择使用 ProxyCap 来转发 `vmnat.exe` 程序的流量。

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-05_01-14-39.png" alt="Snipaste_2024-03-05_01-14-39" style="width:60%;" /></div>

这时候所有从 `vmnat.exe` 程序发出的网络请求，都会被重新路由到宿主机的代理服务中。

```
~]# curl -I https://www.google.com
HTTP/1.1 200 OK
Content-Type: text/html; charset=ISO-8859-1
Content-Security-Policy-Report-Only: object-src 'none';base-uri 'self';script-src 'nonce-D9SDOr0xyqW0R7NmP0oDQA' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other-hp
P3P: CP="This is not a P3P policy! See g.co/p3phelp for more info."
Date: Mon, 04 Mar 2024 17:17:13 GMT
Server: gws
X-XSS-Protection: 0
X-Frame-Options: SAMEORIGIN
Transfer-Encoding: chunked
Expires: Mon, 04 Mar 2024 17:17:13 GMT
Cache-Control: private
Set-Cookie: 1P_JAR=2024-03-04-17; expires=Wed, 03-Apr-2024 17:17:13 GMT; path=/; domain=.google.com; Secure
Set-Cookie: AEC=Ae3NU9Pgcpa6ijve-m0gQG2NbRU2mBB6-JLsxqY_nz4ggBK42QYyVctKQw; expires=Sat, 31-Aug-2024 17:17:13 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=lax
Set-Cookie: NID=512=lAUH7Dt6z3bi9A6CxvKWoGOu-TbTTYiO2biAU3VWcq53-kH1D6_WmqeXgz96B1907ukFnznRRFfaJKA-dzfXgh-WfoCd8QVrKwzS-5XTnESAz8VbCXq1qOVC158AaLl6G4f8Jt7OM0kfnPqBdmVNHdwD5apn7tCUzKmhUi-ZXaA; expires=Tue, 03-Sep-2024 17:17:13 GMT; path=/; domain=.google.com; HttpOnly
Alt-Svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
```

再次配置动态端口转发并访问 Google 网站，可以在 Clash Verge 的连接日志中看到相关：

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-05_01-20-29.png" alt="Snipaste_2024-03-05_01-20-29" style="width:60%;" /></div>

其中所有 IP 地址访问记录都来自于虚拟机的网络请求。

另外，ProxyCap 的连接日志中同样提供了 `vmnat.exe` 相关的连接信息：

<div align="center"><img src="images/SSH%20Tunnel.images/Snipaste_2024-03-05_01-23-01.png" alt="Snipaste_2024-03-05_01-23-01" style="width:60%;" /></div>

由于并非使用 TUN 模式，因此不存在虚拟网卡模块。使用 ProxyCap 代理 `vmnat.exe` 网络流量时，所有的域名请求都会使用宿主机物理网卡中配置的 DNS 服务进行解析，这便导致后续转发至 ProxyCap 中的仅有 IP 请求。

但无论使用哪种方式让虚拟机拥有代理服务的功能，它们都从侧面证明了动态端口转发的可用性。

### 实例应用场景

假设存在远程主机 `Server` 并且其中安装了 `VNC Server` 服务，但出于某种原因 `Server` 不对外开放 `5900` 端口。

那么如果本地主机 `Client` 希望使用 `VNC Client` 连接到 `Server` 时，就可以使用本地端口转发。

在本地主机 `Client` 上配置本地端口转发：

```bash
ssh -L '52710:localhost:5900' 'root@14.150.74.156'
```

配置完毕后，本地主机 `Client` 中的 `VNC Client` 即可使用以下地址访问 `Server` 中的 `VNC Server`：

```
127.0.0.1:52710
```

本地主机 `Client` 上的来自 `VNC Client` 的流量都会通过 SSH Tunnel 流向 `Server` 远程主机。远程主机 `Server` 在接收到流量后，会根据转发配置将流量中继给本机的 `5900` 端口。

这样 `Server` 上的 `VNC Server` 就能间接地接收到来自 `Client` 的 `VNC Client` 的连接请求了。

但如果状况相反：远程主机 `Server` 希望通过 `VNC Client` 连接本地主机 `Client` 上的 `VNC Server` 呢？那么这种情况下，就应当使用远程端口转发。

在本地主机 `Client` 上配置远程端口转发：

```bash
ssh -R '37501:localhost:5900' 'root@14.150.74.156'
```

配置完毕后，远程主机 `Server` 中的 `VNC Client` 即可使用以下地址访问 `Client` 中的 `VNC Server`：

```
127.0.0.1:37501
```

远程主机 `Server` 上的来自 `VNC Client` 的流量都会通过 SSH Tunnel 流向 `Client` 本地主机。本地主机 `Client` 在接收到流量后，会根据转发配置将流量中继给本机的 `5900` 端口。

这样 `Client` 上的 `VNC Server` 就能间接地接收到来自 `Server` 的 `VNC Client` 的连接请求了。