### 概述

本篇将说明各类软件使用代理服务时的最优策略。

其中的代理服务指代使用代理工具 Clash Verge 等在本地端口建立的服务，而非系统代理或 TUN 模式下的全局代理。

### 版本环境

- Windows 11 23H2 22631.3007
- ProxyCap 5.38
- Clash for Windows 0.20.39 | 2023.08.17 Premium
- Clash Verge 1.3.8 | v1.16.0 Meta

### 代理服务

Clash Verge 负责提供代理服务，并同时提供 HTTP 代理和 SOCKS5 代理。

这意味着可以使用入站规则（IN-TYPE）来控制指定代理中流量的最终走向：

```yaml
- IN-TYPE,HTTP,PROXY
- IN-TYPE,HTTPS,PROXY
- IN-TYPE,SOCKS5,DIRECT
- MATCH,DIRECT
```

**在规则不够全面的情况下，这种依靠入站流量类型来灵活切换黑白名单的模式十分有用。**

以上述规则为例，那么：

- 所有未命中的 SOCKS5 类型流量，最终都会匹配 IN-TYPE 为 SOCKS5 中提供的 DIRECT 策略。
- 所有未命中的 HTTP 或 HTTPS 类型流量，最终都会匹配 IN-TYPE 为 HTTP(S) 中提供的 PROXY 策略；

本篇仅针对未命中的域名或 IP 使用 PROXY 还是 DIRECT 策略，来区分黑白名单：

- 黑名单模式：未命中的域名或 IP 使用 DIRECT 策略；
- 白名单模式：未命中的域名或 IP 使用 PROXY 策略。

这相当于：

- 使用 SOCKS5 协议的代理服务时，未命中的流量会使用 DIRECT 策略，即黑名单模式；
- 使用 HTTP(S) 协议的代理服务时，未命中的流量会使用 PROXY 策略，即白名单模式。

无论是 SwitchyOmega 插件，还是 ProxyCap 软件，都可以配置不仅限于 HTTP(S) 或 SOCKS5 协议的代理服务。

一种更灵活的配置是将固定的策略替换为包含多种策略的策略组：

```yaml
- IN-TYPE,HTTP,🌠 Http(s)Escape
- IN-TYPE,HTTPS,🌠 Http(s)Escape
- IN-TYPE,SOCKS5,🌠 Socks(5)Escape
- MATCH,🌠 FinalEscape
```

这样可以在 Clash GUI 中灵活切换运行时规则所对应的策略。

本篇中，**将固定让 HTTP(S) 协议的代理服务使用白名单模式，同时让 SOCKS5 协议的代理服务使用黑名单模式。**

### HTTP(S) 协议选择

关于软件或工具中 HTTP(S) 代理服务的协议选择问题。

本篇中，代理服务由 Clash Verge 提供，且在本地启用了 HTTP(S) 协议的代理服务。但并不是所有的软件或工具都能使用固定的 HTTP 协议连接到该代理服务，以下一些软件或工具连接这个代理时的基本行为：

| SOFTWARES/TOOLS | PROTOCOL SELECTION | TRAFFIC TYPE (IN-TYPE) |
| :-------------: | :----------------: | :--------------------: |
|    ProxyCap     | HTTP ❌ \| HTTPS ✅  |         HTTPS          |
|  SwitchyOmega   | HTTP ✅ \| HTTPS ❌  |         HTTPS          |
|       Git       | HTTP ✅ \| HTTPS ❌  |         HTTPS          |
|      Axios      | HTTP ✅ \| HTTPS ❌  |          HTTP          |

其中，ProxyCap 仅支持以 HTTPS 协议使用本地的代理服务；而 SwitchyOmega、Git 和 Axios 等则仅支持以 HTTP 协议使用本地的代理服务。

不仅协议方面存在差异，某些软件或工具的入站流量（IN-TYPE）类型与预期的也不一致。例如 SwitchyOmega 以 HTTP 协议形式使用本地的代理服务，但其转发至代理服务中的流量类型却为 HTTPS。

这个现象着实令人费解，遗憾的是关于此问题暂时没有一个较为靠谱的解答。

因此，在配置软件或工具的代理服务时，最好依据实际情况来选取正确的 HTTP(S) 协议；同样，软件或工具流入代理服务中的实际流量类型，最好在查阅代理服务日志的明细后再作判定。

### 前置代理

Clash Verge 1.3.8 中的 Meta 内核存在代码错误。

该错误将导致针对 PROCESS-NAME 的 SUB-RULE 规则无法生效，例如：

```yaml
- SUB-RULE,(PROCESS-NAME,PikPak.exe),pikpak
- SUB-RULE,(PROCESS-NAME,DownloadServer.exe),pikpak
```

因此，这里会使用 Clash for Windows 作为前置代理以分流并转发某些特殊软件的流量，充当 SUB-RULE 使用。

```yaml
- PROCESS-NAME,clash-win64.exe,POLICY
```

上述规则会**置顶于 Clash Verge 的规则列表**中。

### 规则示例

Clash Verge：

```yaml
rules:
  - PROCESS-NAME,clash-win64.exe,🌠 CFWHit
  - RULE-SET,addition-reject,REJECT
  - RULE-SET,addition-direct,DIRECT
  - RULE-SET,addition-openai,🎆 OpenAI
  - RULE-SET,addition-gemini,🎆 Gemini
  - RULE-SET,addition-copilot,🎆 Copilot
  - RULE-SET,special-bilibili,🎆 Bilibili
  - RULE-SET,special-youtube,🎆 YouTube
  - RULE-SET,special-github,🎆 GitHub
  - RULE-SET,special-steam,🎆 Steam
  - RULE-SET,addition-proxy,🎇 Comprehensive
  - RULE-SET,original-applications,DIRECT
  - RULE-SET,original-apple,DIRECT
  - RULE-SET,original-icloud,DIRECT
  - RULE-SET,original-private,DIRECT
  - RULE-SET,original-direct,DIRECT
  - RULE-SET,original-greatfire,🎇 Comprehensive
  - RULE-SET,original-gfw,🎇 Comprehensive
  - RULE-SET,original-proxy,🎇 Comprehensive
  - RULE-SET,original-tld-not-cn,🎇 Comprehensive
  - RULE-SET,original-reject,REJECT
  - RULE-SET,original-telegramcidr,🎇 Comprehensive,no-resolve
  - RULE-SET,original-lancidr,DIRECT,no-resolve
  - RULE-SET,original-cncidr,DIRECT,no-resolve
  - GEOIP,LAN,DIRECT,no-resolve
  - GEOIP,CN,DIRECT,no-resolve
  - IN-TYPE,HTTP,🌠 Http(s)Escape
  - IN-TYPE,HTTPS,🌠 Http(s)Escape
  - IN-TYPE,SOCKS5,🌠 Socks(5)Escape
  - MATCH,🌠 FinalEscape
```

Clash for Windows：

```yaml
proxies:
  - name: 🔀 CLASH VERGE | 13766
    type: socks5
    server: 127.0.0.1
    port: 13766
    udp: true
proxy-groups:
  - name: 🛂 TRAFFIC-PRESORT
    type: select
    proxies:
      - 🔀 CLASH VERGE | 13766
rules:
  - RULE-SET,addition-reject,REJECT
  - RULE-SET,addition-direct,DIRECT
  - RULE-SET,special-pikpak,🛂 TRAFFIC-PRESORT
  - RULE-SET,addition-proxy,🛂 TRAFFIC-PRESORT
  - RULE-SET,original-applications,DIRECT
  - RULE-SET,original-apple,DIRECT
  - RULE-SET,original-icloud,DIRECT
  - RULE-SET,original-private,DIRECT
  - RULE-SET,original-direct,DIRECT
  - RULE-SET,original-greatfire,🛂 TRAFFIC-PRESORT
  - RULE-SET,original-gfw,🛂 TRAFFIC-PRESORT
  - RULE-SET,original-proxy,🛂 TRAFFIC-PRESORT
  - RULE-SET,original-tld-not-cn,🛂 TRAFFIC-PRESORT
  - RULE-SET,original-reject,REJECT
  - RULE-SET,original-telegramcidr,🛂 TRAFFIC-PRESORT,no-resolve
  - RULE-SET,original-lancidr,DIRECT,no-resolve
  - RULE-SET,original-cncidr,DIRECT,no-resolve
  - GEOIP,LAN,DIRECT,no-resolve
  - GEOIP,CN,DIRECT,no-resolve
  - MATCH,DIRECT
```

### 浏览器

常规流程：

![image-20240228114601905](images/Agent%20Network.images/image-20240228114601905.png)

SwitchyOmeage 支持不同的代理协议，可以配置 SOCKS5 协议，让未命中的流量使用 DIRECT 策略：

<img src="images/Agent%20Network.images/image-20240228062743010.png" alt="image-20240228062743010" style="zoom: 50%;" />

同时再配置 HTTP(S) 协议，让未命中的流量使用 PROXY 策略：

<img src="images/Agent%20Network.images/image-20240228062807165.png" alt="image-20240228062807165" style="zoom:50%;" />

### 下载工具

常规流程：

![image-20240228065647466](images/Agent%20Network.images/image-20240228065647466.png)

下载工具一般支持配置代理服务，其使用代理服务时仅需要考虑一个关键问题：

- 如何让命中的国外流量使用同一个策略？

**因为下载工具如若需要使用代理服务，则该代理服务必须是能够承担大流量下载任务的。在拥有多个不同的代理服务提供商的前提下，为不同的场景选用不同的服务提供商是非常有必要的。**

实际上，子规则分流能够完美地让下载工具中命中的国外域名或 IP 使用同一个策略（组）。

假如 SUB-RULE 规则可用，以 PikPak 网盘为例编写规则：

```yaml
rules:
  - SUB-RULE,(PROCESS-NAME,PikPak.exe),pikpak
  - SUB-RULE,(PROCESS-NAME,DownloadServer.exe),pikpak
sub-rules:
  pikpak:
    - RULE-SET,addition-reject,REJECT
    - RULE-SET,addition-direct,DIRECT
    - RULE-SET,special-pikpak,POLICY
    - RULE-SET,addition-proxy,POLICY
    - RULE-SET,original-applications,DIRECT
    - RULE-SET,original-apple,DIRECT
    - RULE-SET,original-icloud,DIRECT
    - RULE-SET,original-private,DIRECT
    - RULE-SET,original-direct,DIRECT
    - RULE-SET,original-greatfire,POLICY
    - RULE-SET,original-gfw,POLICY
    - RULE-SET,original-proxy,POLICY
    - RULE-SET,original-tld-not-cn,POLICY
    - RULE-SET,original-reject,REJECT
    - RULE-SET,original-telegramcidr,POLICY,no-resolve
    - RULE-SET,original-lancidr,DIRECT,no-resolve
    - RULE-SET,original-cncidr,DIRECT,no-resolve
    - GEOIP,LAN,DIRECT,no-resolve
    - GEOIP,CN,DIRECT,no-resolve
    - MATCH,DIRECT
```

其中 PikPak.exe 和 DownloadServer.exe 分别为主体程序和下载程序。

这样 PikPak 可以直接使用 Clash Verge 中提供的代理服务，程序将识别来自 PikPak.exe 或 DownloadServer.exe 程序的流量，并使用 pikpak 子规则内的规则进行分流匹配。

但由于 Meta 内核存在代码错误，子规则 SUB-RULE 并无法使用，这里只能采用前置代理来模拟子规则。

本篇的前置代理为 Clash for Windows 软件，将原本的子规则配置为前置代理的规则，并替换其中的 POLICY 为包含目标代理服务的策略（组）即可：

```yaml
proxies:
  - name: 🔀 CLASH VERGE | 13766
    type: socks5
    server: 127.0.0.1
    port: 13766
    udp: true
proxy-groups:
  - name: 🛂 TRAFFIC-PRESORT
    type: select
    proxies:
      - 🔀 CLASH VERGE | 13766
rules:
  - RULE-SET,addition-reject,REJECT
  - RULE-SET,addition-direct,DIRECT
  - RULE-SET,special-pikpak,🛂 TRAFFIC-PRESORT
  - RULE-SET,addition-proxy,🛂 TRAFFIC-PRESORT
  - RULE-SET,original-applications,DIRECT
  - RULE-SET,original-apple,DIRECT
  - RULE-SET,original-icloud,DIRECT
  - RULE-SET,original-private,DIRECT
  - RULE-SET,original-direct,DIRECT
  - RULE-SET,original-greatfire,🛂 TRAFFIC-PRESORT
  - RULE-SET,original-gfw,🛂 TRAFFIC-PRESORT
  - RULE-SET,original-proxy,🛂 TRAFFIC-PRESORT
  - RULE-SET,original-tld-not-cn,🛂 TRAFFIC-PRESORT
  - RULE-SET,original-reject,REJECT
  - RULE-SET,original-telegramcidr,🛂 TRAFFIC-PRESORT,no-resolve
  - RULE-SET,original-lancidr,DIRECT,no-resolve
  - RULE-SET,original-cncidr,DIRECT,no-resolve
  - GEOIP,LAN,DIRECT,no-resolve
  - GEOIP,CN,DIRECT,no-resolve
  - MATCH,DIRECT
```

真实的代理服务 Clash Verge 中需要添加以下置顶规则：

```yaml
rules:
  - PROCESS-NAME,clash-win64.exe,🌠 CFWHit
```

因为分流在前置代理中执行过了，真实的代理服务中不必再次执行分流，所以上述规则要置于规则列表顶部。

此外 PikPak 不再在配置中写入真实的代理服务，取而代之要写入前置代理。这样，来自于 PikPak 的流量都会优先使用前置代理中的分流规则。若命中国外规则，流量将转发至真实代理处；未命中流量默认使用 DIRECT 策略。

类似于 IDM 这种下载工具也可以使用这个前置代理以分流来自国外的下载请求。

### 不支持代理的软件

对于不支持代理服务、但刚需代理的这类软件来说，可以借助 ProxyCap 等工具完成主动的流量转发。

<img src="images/Agent%20Network.images/image-20240228103127160.png" alt="image-20240228103127160" style="zoom:67%;" />

常规流程：

![image-20240228114529708](images/Agent%20Network.images/image-20240228114529708.png)

根据入站规则，可以让 HTTPS 流量类型和 SOCKS5 流量类型在未命中规则时采用不同的策略：

- HTTPS 流量类型未命中规则时使用 HTTP(S) 规则（白名单模式）：

![image-20240228114505263](images/Agent%20Network.images/image-20240228114505263.png)

- SOCKS5 流量类型未命中规则时使用 SOCKS5 规则（黑名单模式）：

![image-20240228114436204](images/Agent%20Network.images/image-20240228114436204.png)

类似于 GitKraken 这类 Git GUI 工具，其中的某些静态资源需要依赖于访问国外的域名或 IP 地址，且往往规则中无法涵盖这其中大部分国外 IP 的访问请求。

那么可以使用 ProxyCap 工具以 HTTPS 流量类型转发所有来自于  GitKraken 软件的网络请求，这样所有来自 GitKraken 的网络请求都会转发至代理服务中，且能确保未命中的域名或 IP 请求，均会使用白名单模式中 PROXY 策略进行访问。

<img src="images/Agent%20Network.images/image-20240228111043861.png" alt="image-20240228111043861" style="zoom:67%;" />

另外还有一些软件是偶尔需要发起国外域名或 IP 请求的，如 PotPlayer 播放器。

多数情况下，PotPlayer 会在更新的时候发起访问国外域名或 IP 的请求，但由于无法确定这些请求是否会命中 PROXY 策略，因此建议使用 HTTP(S) 协议的代理，这样未能命中的请求均能使用白名单模式中的 PROXY 策略进行访问。

<img src="images/Agent%20Network.images/image-20240228111105680.png" alt="image-20240228111105680" style="zoom:67%;" />

还有一类仅支持系统代理的软件，例如 Google Drive。

系统代理本身存在弊端，即代理启用的情况下，一些本身默认支持系统代理的软件，会主动地去使用代理服务，即便软件本身完全不依赖于代理服务。简而言之，这会让某些不需要使用代理的软件，被迫使用代理。

因此多数情况下，不推荐启用系统代理。排除系统代理的选项后，类似 Google Drive 这类软件，其处境就无异于不支持代理的软件了。

<img src="images/Agent%20Network.images/image-20240228113552239.png" alt="image-20240228113552239" style="zoom:67%;" />

### 支持代理的软件

对于本身就支持配置代理服务的软件来说，只有软件需要使用代理时，才需要主动为其添加代理服务。并根据软件实际所发出的网络请求，来自行选择使用 HTTPS 代理（白名单模式）还是 SOCKS5 代理（黑名单模式）。

能够支持配置代理的软件似乎就不会存在使用代理服务的困恼，对吗？实则不然。实际情况是，不乏一些本身支持配置代理服务，但支持方式却不够优雅的软件存在，有两个非常典型的例子：1. Git；2. OpenSSH。

#### Git & OpenSSH

众所周知，使用 Git 工具克隆或推送仓库时，不可避免需要访问 GitHub 等网站，而访问这类网站大多数时候需要代理服务的支持。

Git 本身支持配置代理服务：

```bash
git config --global http.proxy http://127.0.0.1:13766
```

但该代理服务实际存在很大的局限性：

- 克隆仓库时，能够主动使用该代理服务；
- 拉取、推送仓库为 HTTP(S) 地址时，能够主动使用该代理服务。

这意味着如果推送仓库使用 SSH 协议，那么这个代理服务的配置形同虚设。

OpenSSH 本身也支持代理服务，但这个代理配置的方式相当复杂，需要借助其他的网络工具完成配置。Windows 系统下，OpenSSH 建立代理服务的连接常用的是 connect.exe 工具。

然而 connect.exe 工具并不存在于默认的 OpenSSH 包中，使用它通常需要安装 MinGW 工具集。MinGW（Minimalist GNU for Windows）是一个在 Windows 平台上开发和运行本地 GNU 工具的项目。MinGW 提供了一组开发工具和库，使开发者能够在 Windows 上编译和运行类 Unix 环境下的软件，而无需依赖于 Microsoft 的 Visual Studio 或其他商业工具。

所幸，如若系统上已安装了 Git 版本控制工具，那么可以在 Git 的安装目录中找到这个 connect.exe 工具，无需安装 MinGW 工具集。

OpenSSH 实现代理连接的命令：

```bash
ssh -T 'git@github.com' -p 22 -i '~/.ssh/for_connect' -o 'ProxyCommand "e:/git/mingw64/bin/connect.exe" -S 127.0.0.1:13766 %h %p'
```

不愿意编写长命令，还可以选择将参数写入配置文件：

```
Host github.com
	HostName %h
	Port 22
	IdentityFile ~/.ssh/for_connect
	ProxyCommand e:/git/mingw64/bin/connect -S 127.0.0.1:13766 %h %p
```

但实际无论使用哪种配置方式，维护起来都十分麻烦。

对于这种无法优雅地配置代理服务的工具来说，使用 ProxyCap 来主动转发这些网络流量无疑是更好的选择。

在 ProxyCap 中添加相关的 GitHub 域名配置，则无论是使用 Git 克隆仓库还是推送仓库，其网络流量都会被 ProxyCap 转发至代理工具中以使用代理服务，即便推送仓库时使用的是 SSH 协议。

<img src="images/Agent%20Network.images/image-20240228111126691.png" alt="image-20240228111126691" style="zoom:67%;" />

<img src="images/Agent%20Network.images/image-20240228111126692.png" alt="image-20240228111126692" style="zoom:67%;" />

其中：

- `git-remote-https.exe`：Git 在拉取仓库或推送 HTTPS 协议仓库时需要使用到的程序；
- `ssh.exe`：建立 OpenSSH 连接时需要使用到的程序。

#### Node.js

类似于 Node.js 这种网络框架，同样推荐使用 ProxyCap 来转发流量至代理服务。Node.js 中需要使用代理服务的无非就是 npm 包管理工具，因为多数的 npm 仓库位于国外，拉取仓库超时是常有的事。

npm 本身支持配置代理服务：

```
npm config set proxy http://127.0.0.1:13766/
```

该命令会生成 `~/.npmrc` 的配置文件：

<img src="images/Agent%20Network.images/image-20240228112159295.png" alt="image-20240228112159295" style="zoom: 50%;" />

虽然自动生成配置文件不是什么大问题，但比较棘手的实际是管理问题。无论是确认是否存在代理服务，修改代理服务或删除代理服务，这都需要一行行代码手动执行。

对于一个 npm 来说，管理可能是简单的事情，但如果存在成千上百个相似的工具，那管理起来必一头雾水。

在 ProxyCap 中添加对应的 Node.js 配置，那么不仅所有的代理明细一目了然，还能够集中统一地管理一类软件是否能够使用代理服务，这无疑是两全其美的。

<img src="images/Agent%20Network.images/image-20240228113117332.png" alt="image-20240228113117332" style="zoom:67%;" />

理论上来说，由于 npm 软件包管理工具隶属于 Node.js 框架，因此所有 npm 的网络流量都会经由 `node.exe` 发起，从而达到被代理的目的。

另外，关于 Axios 等能够发起代理请求的网络框架来说，如果 Axios 处于 Node.js 环境，那么在有 ProxyCap 管理 Node.js 流量的情况下，Axios 发起的网络请求同样会被 ProxyCap 转发。

```javascript
const axios = require('axios');
const url = "https://www.google.com";
axios({
    url: url,
    method: "get"
}).then(function (response) {
    console.log(response.status);
});
```

<img src="images/Agent%20Network.images/Snipaste_2024-02-28_14-02-37.png" alt="Snipaste_2024-02-28_14-02-37" style="zoom:67%;" />

但如果不在 Node.js 环境中使用 Axios 框架，那么访问国外域名时则需要额外为 Axios 提供单独的代理：

```javascript
const axios = require('axios');
const url = "https://www.google.com";
axios({
    url: url,
    method: "get",
    proxy: {
        protocol: "http",
        host: "127.0.0.1",
        port: 13766
    }
}).then(function (response) {
    console.log(response.status);
});
```

这里建议在 Axios 需要发起国外域名请求时，总是在其配置中添加单独的代理服务。