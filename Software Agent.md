### 概述

国内访问 [GitHub](https://stackoverflow.com/) 或 [Stack Overflow](https://stackoverflow.com/) 这种技术性网站时，不可避免会使用到代理服务。

如果仅是浏览器访问这些网站，那么使用系统代理即可。浏览器默认支持系统代理，只要启用系统代理并确保其中配置了可用的代理服务器，浏览器就会默认地使用这些系统代理完成网络请求。

不过系统代理的弊端也在于软件默认启用并支持系统代理上，假如浏览器不需要使用代理服务，但系统代理长期处于启用状态，这时浏览器将被迫不得不使用代理服务。

通常涉及到访问 GitHub 等域名的场景不仅局限在浏览器中，最常见的、刚需访问 GitHub 等域名的场景，实际是使用 Git 这些版本管理工具拉取 GitHub 仓库代码的时候。

使用 Git 执行一段普通的克隆代码：

```bash
git clone https://github.com/username/repo.git
```

可以看到克隆命令发起了对 GitHub 网站的访问请求。

这种时候系统代理并无法解决潜在的“响应超时”的问题，但所幸 Clone 命令支持配置代理服务。以访问 GitHub 为目标，可以参考 [GitHub](./GitHub.md) 为 Git 配置代理服务。

```bash
git config --global http.proxy http://127.0.0.1:19121
```

但 Git 中的代理服务存在一定的局限性，因为该服务仅可作用于 Clone 命令。

以上讨论了一些默认支持系统代理的软件和支持配置代理服务的软件，以及相应代理的局限性等。

那是否存在本身并不支持系统代理也不支持配置代理，但却刚需代理服务的软件呢？肯定的。例如 GitKraken 这类 Git GUI 客户端软件，其本身并不支持配置代理服务。

但由于软件的特殊性，国内使用 GitKraken 时会需要使用到代理服务，以完成某些资源的加载。

如何让本身不支持代理服务的软件使用代理服务，是本篇着重讨论的问题。

### 版本环境

- Windows 11 23H2 22631.3007
- ProxyCap 5.38
- Proxifie 4.12
- Clash for Windows 0.20.39 | 2023.08.17 Premium
- Clash Verge 1.3.8 | v1.16.0 Meta

### TUN

针对不支持代理服务的软件，目前主流的解决方案是使用 TUN 模式。简单来说，TUN 模式依赖于支持该模式的代理软件。通过启用 TUN 模式，代理软件能够在本地部署虚拟网卡，从而接管本地计算机产生的几乎所有网络流量。

这种方式使得不支持代理服务的软件的网络流量能够被虚拟网卡接管，以强制使用代理服务。但值得注意的是，这种劫持网络流量的实现特性可能会导致一些本身不需要使用代理的软件流量被劫持。

尽管 TUN 模式为不支持代理服务的软件提供了一种应急解决方案，但并不是完美的。这种解决方案的实现依赖于操作系统级别的支持，并且可能存在一些性能影响和局限性。例如，劫持网络流量可能会增加系统负载，影响网络性能。

### 流量转发

对于不支持代理服务的软件来说，主动的流量转发工具也许是更好的选择。

#### Proxifier

Proxifier 是一款网络工具软件，主要用于通过代理服务器来改变网络应用程序的连接方式。该软件允许用户将网络流量通过代理服务器进行传输，从而实现一些特定的网络配置，绕过网络限制或保护用户的隐私。

具体而言，Proxifier 可以使那些不支持代理设置的应用程序通过代理服务器进行连接，实现对这些应用程序的全局代理。

使用 Proxifier，用户可以自定义规则，将特定的应用程序或流量导向通过代理服务器进行连接，从而实现对网络流量的更精细的控制。这对于需要访问受限制网站、提高网络安全性或在一些网络环境中绕过审查等情况下可能是有用的。

在 Proxifier 中配置代理服务器：

<img src="images/Software%20Agent.images/image-20240130163713352.png" alt="image-20240130163713352" style="zoom:67%;" />

添加代理服务器时，请务必在高级（Advanced..）选项卡中勾选以下选项：

<img src="images/Software%20Agent.images/image-20240130163818588.png" alt="image-20240130163818588" style="zoom:67%;" />

像 Microsoft Edge 或 Chrome 这类浏览器，如果需要使用 Proxifier 完成代理工作，那么启用该选项能够直接转发域名至代理处，让代理来直接处理浏览器的访问请求。

如果不勾选该项，则所有的域名都会被提前进行 DNS 解析，最终转发至代理中的是只包含 IP 信息的网络流量，且不存在任何的域名信息。

类似 Clash 这样存在规则模式的代理工具，如果无法接收到有效的域名信息，那么就意味着无法进行精准的网络分流工作，因为大多数的分流规则仅用于匹配域名，较难匹配 IP 地址。

添加完代理服务器后，将需要使用代理服务的应用程序配置在规则中：

<img src="images/Software%20Agent.images/image-20240130164256662.png" alt="image-20240130164256662" style="zoom:67%;" />

配置完成后，Proxifier 会自动刷新配置，并根据规则，将应用程序中符合规则的流量，转发至代理服务器，从而实现应用程序的被动代理：

<img src="images/Software%20Agent.images/image-20240130164433618.png" alt="image-20240130164433618" style="zoom:67%;" />

#### ProxyCap

ProxyCap 是一款专业的代理软件，它允许你为特定的应用程序或进程配置代理设置，而不是整个系统。这意味着你可以选择性地将某个应用程序的流量通过代理服务器。

ProxyCap 支持多种代理协议，包括 SOCKS4、SOCKS5、HTTP、HTTPS、SSH 和 SS 等，用户可以自由选择适合的代理协议。

ProxyCap 还支持设置规则和过滤条件，以定义哪些流量需要通过代理，哪些不需要。这使得用户能够根据应用程序、目标地址或端口等条件进行更精细的网络流量控制。

在 ProxyCap 中配置代理服务器：

<img src="images/Software%20Agent.images/image-20240130023422821.png" alt="image-20240130023422821" style="zoom: 67%;" />

将需要使用代理服务的应用程序配置在规则中：

<img src="images/Software%20Agent.images/image-20240130023453233.png" alt="image-20240130023453233" style="zoom:67%;" />

为指定程序添加规则时，该程序必须是系统中存在的可执行文件，其必须提供绝对路径：

<img src="images/Software%20Agent.images/image-20240130024213579.png" alt="image-20240130024213579" style="zoom:67%;" />

添加规则时，务必勾选“Resolve names remotely”选项，以将域名信息转发给代理服务器。该选项决定了是否在 ProxyCap 中进行域名解析，不勾选则所有域名都会被解析。

和此前 Proxifier 中提及域名解析问题一样，于存在分流规则的代理工具而言，接收域名比接收 IP 地址更有利于分流工作。

如果目标程序的绝对路径有可能在后续的版本升级中发生变化，那么推荐手动到规则属性的选项卡中，将目标程序的**按绝对路径检索**规则转换为**按程序名称检索**：

<img src="images/Software%20Agent.images/image-20240130024507961.png" alt="image-20240130024507961" style="zoom:67%;" />

配置完成后，ProxyCap 会自动刷新配置，并根据规则，将应用程序中符合规则的流量，转发至代理服务器，从而实现应用程序的被动代理：

<img src="images/Software%20Agent.images/image-20240130023541625.png" alt="image-20240130023541625" style="zoom:67%;" />

ProxyCap 的规则支持 HTTP 下载和更新。对于自用规则来说，十分推荐在配置完毕后，将规则导出备份：

<img src="images/Software%20Agent.images/image-20240130024717696.png" alt="image-20240130024717696" style="zoom:67%;" />

为了保证规则的通用性，应尽量不在规则中使用**按绝对路径检索**应用程序的规则。

ProxyCap 默认开机启动。如需禁用 ProxyCap 的代理功能，可以在子菜单中完成：

<img src="images/Software%20Agent.images/image-20240130025006722.png" alt="image-20240130025006722" style="zoom:80%;" />

### 遗留问题

据实测，**尽管 Proxifier 和 ProxyCap 都能完成软件的主动代理工作，但它们实际也不尽完美。**

问题主要出在现代的代理服务大多数存在所谓的分流规则，分流规则要求提供具体的域名或 IP 地址以进行策略组的判定。

而 Proxifier 和 ProxyCap 中均存在域名解析的问题：

- Proxifier 只有转发来自浏览器的网络请求时，不会主动使用 DNS 解析服务；
- ProxyCap 只有转发除浏览器以外的的网络请求时，不会主动使用 DNS 解析服务。

简而言之，使用 Proxifier 转发类似 Google Drive 这类软件的网络流量时，Proxifier 会将其中的域名请求解析成 IP 地址后再转发给代理服务，以至代理工具无法进行完成精确的分流。

而使用 ProxyCap 转发来自 Microsoft Edge 或 Chrome 等浏览器的网络流量时，ProxyCap 也会将域名请求解析成 IP 地址后再转发给代理服务，这同样会导致代理工具无法完成精确的分流。

因此，一个较为折中的解决方法是同时使用 Proxifier 和 ProxyCap 对需要代理的软件进行流量转发。其中，前者只负责转发浏览器的流量，后者则只负责转发除浏览器之外的其他软件的流量。

#### ChatGPT

语言模型当道的现代，访问并使用 ChatGPT 并不是什么稀奇的事情。但如果使用 Proxifier 转发**浏览器**的网络流量，则可能导致无法访问 OpenAI 的问题。

OpenAI 存在 VPN 检测机制，它会将类似于 Proxifier 这样的代理软件判定为 VPN 软件，一旦判定用户正在使用类似 VPN 这样的软件访问 OpenAI，则网站会立刻阻止用户的访问。

如果有使用 ChatGPT 的需求，则不建议使用 Proxifier 来代理浏览器的网络流量。

这时候可以选择使用更为轻量级浏览器流量转发插件，诸如 SwitchyOmega 或 SmartProxy 等。这类流量转发插件，在访问 OpenAI 时能够绕过 VPN 检测。

### 转发插件

浏览器的流量转发插件也被称作代理插件，常见的有 SwitchyOmega 和 SmartProxy 等。它们都可以实现浏览器中网络流量的转发，但需要注意，这种转发的效率不会太高。

除开转发效率的问题外，一个较为常见的问题出在插件的安装方式上。

假如使用的是 Chrome 这种 Google 系的浏览器，那么在安装这类插件时，需要访问 Google 扩展商店。但该扩展商店只能通过代理访问，这就是一个典型的鸡生蛋还是蛋生鸡的问题。

如若系统性能充足，这里更推荐使用 Proxifier 等工具单独转发所有浏览器的网络流量，这不仅能提高流量的转发效率，且更便捷省心。

但如果有特殊要求，如访问 OpenAI 等语言模型网站，那么使用 SwitchyOmega 以绕过相关的 VPN 检测会是更好的选择。
