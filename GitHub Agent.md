### 概述

GitHub 是一个基于 Git 版本控制系统的代码托管平台。它提供了许多工具和服务，帮助开发者协作、管理和追踪软件项目的变化。

本篇旨在解决国内访问 GitHub 超时的问题，但前提是需要可用的代理服务。

### 版本环境

- Windows 11 23H2 22631.3007
- Git 2.40.0 64bit
- Clash for Windows 0.20.39 | 2023.08.17 Premium
- Clash Verge 1.3.8 | v1.16.0 Meta

### 解决方案

在拥有可用代理服务的情况下，可以通过两种配置来解决某些场景下访问 GitHub 时可能出现的网络问题：

- 对于 Git 中的 HTTP(S) 请求，可以添加 `http.proxy` 配置项；
- 对于 SSH 协议的连接请求，可以添加 `ProxyCommand` 配置项。

#### 1. http.proxy

`http.proxy` 是一种配置选项，用于在 Git 中设置 HTTP(S) 请求的代理。通过设置 `http.proxy` 可以指定一个代理服务，以便 Git 在执行 HTTP(S) 请求时将流量路由（流量转发）到该代理服务中。

```bash
# HTTP(S) AGENT
git config --global http.proxy http://127.0.0.1:13766

# SOCKS5 AGENT
git config --global http.proxy socks5://127.0.0.1:13766
```

两种协议实际传输到代理服务的内容有所差异：

- 使用 HTTP(S) 协议时，传输到代理服务的是域名请求；
- 使用 SOCKS5 协议时，传输到代理服务的是经解析后的 IP 请求。

以上代理配置将写入到 Git 的全局配置（GLOBAL）中，这意味着无论从什么仓库地址中克隆仓库，都会使用 `http.proxy` 配置的代理服务。

其他命令：

```bash
# 移除代理配置
git config --global --unset http.proxy

# 查看全局配置
git config --global -l
```

如果希望避免所有 HTTP(S) 请求使用代理服务，还可以选择为指定域名单独配置代理： 

```bash
# 为 https://github.com 单独配置 HTTP(S) 代理
git config --global http.https://github.com.proxy http://127.0.0.1:13766

# 移除特定代理
git config --global --unset http.https://github.com.proxy
```

这样有且仅在发起对 [GitHub](https://github.com) 的 HTTP(S) 请求时，会使用代理服务。

#### 2. ProxyCommand

根据 Git 官方文档，`http.proxy` 配置项用于设置 HTTP(S) 请求的代理，而 OpenSSH 使用的 SSH 协议本与 HTTP(S) 协议无关。因此，如果需要为使用 SSH 协议推送仓库并为其设置代理，则需要使用其他的方式来配置代理服务，而不是使用 `http.proxy` 配置项。

以下命令用于测试 SSH 是否能够连接 GitHub，并会同时验证本地的私钥是否正常：

```bash
ssh -T 'git@github.com' -p 22 -i '~/.ssh/for_connect'
```

如果本地存在相关的 OpenSSH 配置：

```
Host github.com
	HostName %h
	Port 22
	IdentityFile ~/.ssh/for_connect
```

那么，测试连通性的命令可以简化为：

```bash
ssh -T 'git@github.com'
```

以下两种情况均表示使用 SSH 能够与 GitHub 建立连接：

```bash
~]# ssh -T 'git@github.com'
Hi dylan127c! You've successfully authenticated, but GitHub does not provide shell access.
```

```bash
~]# ssh -T 'git@github.com'
git@github.com: Permission denied (publickey).
```

其中：

- 提示 1 表示 SSH 连接 GitHub 成功，且本地私钥通过验证；
- 提示 2 表示 SSH 连接 GitHub 成功，但本地不存在私钥或私钥未通过验证：

如果提示如下，则表示使用 SSH 不能与 GitHub 建立连接：

```bash
~]# ssh -T 'git@github.com'
ssh: connect to host github.com port 22: Connection timed out
```

##### 安全连接

无法建立连接时，比较推荐直接借用代理服务建立连接，不过也可以尝试启用通过 HTTPS 的 SSH 连接。

首先测试通过 HTTPS 端口的 SSH 连接是否可行：

```bash
~]# ssh -T 'git@ssh.github.com' -p 443
Hi dylan127c! You've successfully authenticated, but GitHub does not provide shell access.
```

如果可行，可以覆盖 SSH 设置来强制与 github.com 的任何连接均使用 ssh.github.com 服务器和 443 端口：

```
Host github.com
	HostName ssh.github.com
	Port 443
	User git
	IdentityFile ~/.ssh/for_connect
```

注意 SSH 配置的覆盖原则，它总是根据 Host 匹配，并在命令不提供默认参数的情况下使用配置覆盖。例如：

```bash
ssh -T 'github.com'
```

等价于：

```bash
ssh -T 'git@ssh.github.com' -p 443 -i '~/.ssh/for_connect'
```

如果命令原本就带有配置内提供的默认参数，那么配置将不会覆盖命令。例如：

```bash
ssh -T 'dylan@github.com' -p 23
```

等价于：

```bash
ssh -T 'dylan@ssh.github.com' -p 23 -i '~/.ssh/for_connect'
```

只要 Host 能匹配就会尝试覆盖未提供的参数，如果 Host 匹配了但参数已提供，则不会使用配置覆盖。

##### 代理连接

OpenSSH 可以通过 `ProxyCommand` 参数，实现通过代理服务建立 SSH 连接：

```bash
ssh -T 'git@github.com' -p 22 -i '~/.ssh/for_connect' -o 'ProxyCommand "e:/git/mingw64/bin/connect.exe" -S 127.0.0.1:13766 %h %p'
```

其中 `-o` 选项即为代理配置，注意 Windows 系统下需要使用 `connect.exe` 程序建立与代理服务的连接。

但日常的拉取或推送操作并不直接涉及到 OpenSSH 的命令，在需要拉取或推送仓库时 Git 会自发调用 OpenSSH 命令。这时候如果需要使用代理服务完成 Git Pull/Push 操作，则需要将代理信息写入 OpenSSH 的配置文件中：

```
Host github.com
	HostName %h
	Port 22
	IdentityFile ~/.ssh/for_connect
	ProxyCommand e:/git/mingw64/bin/connect -S 127.0.0.1:13766 %h %p
```

默认的 OpenSSH 配置文件 config 通常位于用户目录下的 .ssh 目录中：

- `~/.ssh/config`

配置完成后，无论是从 GitHub 拉取仓库，亦或是将仓库推送至 GitHub，建立 SSH 连接时都会使用配置中的私钥及代理配置，从而实现通过代理完成 Git Pull/Push 操作。

### 完美方案

以上两种方案可以说是分别解决了 Git 和 OpenSSH 在执行 Git Clone、Git Pull/Push 操作时访问 GitHub 网址所出现的超时问题。然而实际上它们所解决的是同一个问题，即本地访问 GitHub 网址时可能出现的超时问题。

如果后续不仅仅是 Git 和 OpenSSH 等需要访问 GitHub 网址，那么需要进行的代理配置只会越来越多。

完美解决本地计算机访问 GitHub 网址的问题，需要使用一款名为 ProxyCap 的流量转发（代理转发/流量路由）软件，详情参阅：[Software Agent - ProxyCap](./Software Agent####ProxyCap)。

在配置 ProxyCap 之前，必须知道实际用于完成 `git clone` 和 `git pull/push` 操作的程序是什么：

- `git-remote-https.exe` 程序：用于发起 Git 中克隆、拉取或推送仓库时的 HTTP(S) 请求；
- `ssh.exe` 程序：用于发起 Git 中克隆、拉取或推送仓库时的 SSH 请求。

了解目标执行程序后，即可在 ProxyCap 中写入配置：

<div align="center"><img src="images/GitHub%20Agent.images/image-20240203050102819.png" alt="image-20240203050102819" style="width:45%;" /></div>

<div align="center"><img src="images/GitHub%20Agent.images/image-20240203050145918.png" alt="image-20240203050145918" style="width:45%;" /></div>

<div align="center"><img src="images/GitHub%20Agent.images/image-20240203050201737.png" alt="image-20240203050201737" style="width:45%;" /></div>

配置完成且 ProxyCap 启用时，所有经由程序 `git-remote-https.exe` 或 `ssh.exe` 发起的网络请求，一旦匹配到 `github.com` 或 `raw.githubusercontent.com` 等域名规则，则这些网络流量则都将经由 ProxyCap 转发至目标代理服务中。

这样即可对特定网址进行代理，同时也省去了配置 Git 和 OpenSSH 等代理服务的繁杂操作。

### 端口问题

在某些网络环境下，标准的 SSH 通信端口（默认为 22）可能会被封锁或限制。

例如，即便使用了代理服务，网络故障也依旧存在：

```bash
~]# ssh -T 'git@github.com'
kex_exchange_identification: Connection closed by remote host
...
```

<div align="center"><img src="images/GitHub%20Agent.images/Snipaste_2024-03-03_22-57-33.png" alt="Snipaste_2024-03-03_22-57-33" style="width:80%;" /></div>

这通常表示代理服务存在问题，其原因大概率是代理服务封锁或限制了 22 端口。

为了在这些环境下建立 SSH 连接，GitHub 提供了备用的 SSH 入口点 `ssh.github.com`，并且使用了 HTTPS 的标准端口 443，该端口通常不会被封锁。

<div align="center"><img src="images/GitHub%20Agent.images/Snipaste_2024-03-03_22-58-55.png" alt="Snipaste_2024-03-03_22-58-55" style="width:80%;" /></div>

通过使用 `ssh.github.com:443` 网址，用户可以绕过一些代理服务中的网络限制或防火墙配置，从而顺利地与 GitHub 建立 SSH 通信，完成代码的拉取、推送等操作。

如果使用备用网址依旧无法建立有效的 SSH 通信，则建议更换其他的代理服务。

### 不存在的配置项

在网络上搜索关于 Git 配置代理的相关信息时，经常能看到名为 `https.proxy` 配置项。这些网络回答中，几乎都是同时为 Git 配置了 `http.proxy` 和 `https.proxy` 两种属性的“代理”。

问及配置两种”代理“有什么作用时，一种通用的解释是：`http.proxy` 用于代理 HTTP 协议请求，`https.proxy` 则用于代理 HTTPS 协议请求。但实际上，属性为 `https.proxy` 的配置项根本就不存在，配置项 `http.proxy` 本身就能同时代理 HTTP 和 HTTPS 协议的请求。

根据 Git 官方文档以及其他权威资料显示，Git 没有提供 `https.proxy` 这样的配置项。有且仅存 `http.proxy` 配置项用于设置代理服务，并且该代理设置对 HTTP 和 HTTPS 请求均有效。
