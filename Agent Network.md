### 概述

本篇将说明各类软件使用代理服务时的最优策略，其中代理服务使用 Clash Verge 在本地端口建立。

### 版本环境

- Windows 11 23H2 22631.3007
- ProxyCap 5.38
- Clash Verge 1.3.8 | v1.16.0 Meta

### 代理服务

Clash Verge 为本篇负责提供代理服务的工具，它可以在同一个端口上同时提供 HTTP(S) 和 SOCKS5 代理。

这意味着可以使用 Clash 入站规则（IN-TYPE）来控制流量的最终走向：

```yaml
- IN-TYPE,HTTP,PROXY
- IN-TYPE,HTTPS,PROXY
- IN-TYPE,SOCKS5,DIRECT
- MATCH,DIRECT
```

**在规则不够全面的情况下，入站流量类型配合指定的策略规则，可实现黑、白名单模式之间的动态切换。**

以上述规则为例：

- 所有未命中的 SOCKS5 类型流量，最终都会匹配 IN-TYPE 为 SOCKS5 中提供的 DIRECT 策略。
- 所有未命中的 HTTP 或 HTTPS 类型流量，最终都会匹配 IN-TYPE 为 HTTP(S) 中提供的 PROXY 策略；

根据未命中的域名或 IP 最终使用的策略，来区分黑白名单：

- 黑名单模式：未命中的域名或 IP 使用 DIRECT 策略；
- 白名单模式：未命中的域名或 IP 使用 PROXY 策略。

综上：

- 使用 SOCKS5 协议的代理服务时，未命中的流量会使用 DIRECT 策略，即黑名单模式；
- 使用 HTTP(S) 协议的代理服务时，未命中的流量会使用 PROXY 策略，即白名单模式。

无论是 SwitchyOmega 插件，还是 ProxyCap 软件，都可以配置不仅限于 HTTP(S) 或 SOCKS5 协议的代理服务。

一种更灵活的配置方式是将固定的策略替换为包含多种策略的策略组：

```yaml
- IN-TYPE,HTTP,🌠 Http(s)Escape
- IN-TYPE,HTTPS,🌠 Http(s)Escape
- IN-TYPE,SOCKS5,🌠 Socks(5)Escape
- MATCH,🌠 FinalEscape
```

这样可以在 Clash GUI 中灵活切换运行时规则所对应的策略。

本篇中，**将固定让 HTTP(S) 协议的代理服务使用白名单模式，让 SOCKS5 协议的代理服务使用黑名单模式。**

### HTTP(S) 协议选择

关于普通软件或流量转发工具中的代理配置问题，其中比较刁钻的是 HTTP(S) 协议的选择问题。

本篇的代理服务由 Clash Verge 提供，它能在本地提供 HTTP(S) 协议的代理服务。但并不是所有的软件或工具都能使用固定的 HTTP 协议连接到该代理服务，以下一些软件或工具连接这个代理时的基本行为：

| SOFTWARES/TOOLS | PROTOCOL SELECTION | TRAFFIC TYPE (IN-TYPE) |
| :-------------: | :----------------: | :--------------------: |
|    ProxyCap     | HTTP ❌ \| HTTPS ✅  |         HTTPS          |
|  SwitchyOmega   | HTTP ✅ \| HTTPS ❌  |         HTTPS          |
|       Git       | HTTP ✅ \| HTTPS ❌  |         HTTPS          |
|      Axios      | HTTP ✅ \| HTTPS ❌  |          HTTP          |

可以看到 ProxyCap 仅支持以 HTTPS 协议的形式使用 HTTP(S) 代理服务；SwitchyOmega、Git 和 Axios 等则仅支持以 HTTP 协议的形式使用 HTTP(S) 代理服务。

除了协议方面存在的差异，某些软件或工具的入站流量（IN-TYPE）类型也和预期不一致。SwitchyOmega 以 HTTP 协议形式使用 HTTP(S) 代理服务，但转发代理服务内的流量却为 HTTPS。

这个现象着实令人费解，遗憾的是关于此问题暂时没有一个较为靠谱的解答。

因此，在配置软件或流量转发工具的代理服务时，最好依据实际情况选取正确的 HTTP(S) 协议；同样，流入代理服务中的实际流量类型，推荐在查阅代理服务日志后另做判定。

### 规则示例

Clash Verge：

```yaml
rules:
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

### 浏览器

浏览器推荐使用 SwitchyOmeage 插件来使用代理服务，原因有三个：

- 系统代理不可避免地让支持系统代理、但不需要使用系统代理的软件使用了代理；
- 类似 Proxifier、ProxyCap 这种流量转发软件，虽可以转发浏览器的请求，但 OpenAI 会将其识别为 VPN；
- 其中 ProxyCap 无法接收到来自浏览器的域名请求，所有请求在流入 ProxyCap 前就已经被解析成 IP 请求。

浏览器中，目前最完美的代理实现方式，仅有赖于一众代理插件。

常规流程：

<div align="center"><img src="images/Agent%20Network.images/image-20240228114601905.png" alt="image-20240228114601905" style="width: 80%;" /></div>

SwitchyOmeage 支持不同的代理协议。

- 配置 SOCKS5 协议让未命中的 SOCKS 流量使用 DIRECT 策略：

<div align="center"><img src="images/Agent%20Network.images/image-20240228062743010.png" alt="image-20240228062743010" style="width: 80%;" /></div>

- 配置 HTTP(S) 协议让未命中的 HTTPS 流量使用 PROXY 策略：

<div align="center"><img src="images/Agent%20Network.images/image-20240228062807165.png" alt="image-20240228062807165" style="width: 80%;" /></div>

### 不支持代理的软件

不支持代理的软件如果刚需代理服务，那么这个软件一般属于国外软件。这类软件需要借助流量转发工具，来间接使用代理服务，例如 Proxifier、ProxyCap 等。

本篇使用 ProxyCap 软件：

<div align="center"><img src="images/Agent%20Network.images/Snipaste_2024-03-09_06-36-01.png" alt="Snipaste_2024-03-09_06-36-01" style="width:50%;" /></div>

常规流程：

<div align="center"><img src="images/Agent%20Network.images/image-20240228114529708.png" alt="image-20240228114529708" style="width:80%;" /></div>

根据入站规则，让 HTTPS 流量类型和 SOCKS5 流量类型在未命中规则时采用不同的策略：

- HTTPS 流量类型未命中规则时使用白名单模式：

<div align="center"><img src="images/Agent%20Network.images/image-20240228114505263.png" alt="image-20240228114505263" style="width:80%;" /></div>

- SOCKS5 流量类型未命中规则时使用黑名单模式：

<div align="center"><img src="images/Agent%20Network.images/image-20240228114436204.png" alt="image-20240228114436204" style="width:80%;" /></div>

类似于 GitKraken 这类 Git GUI 工具，其中的某些静态资源赖于访问国外的域名或 IP 地址，且往往规则无法涵盖这其中大部分的 IP 访问请求。

使用 ProxyCap 以 HTTPS 流量类型转发所有来自于  GitKraken 软件的网络请求，那么所有来自 GitKraken 的网络请求都将被转发至代理服务中。如果使用白名单模式，未命中的流量也都能确保使用 PROXY 策略进行访问。

<div align="center"><img src="images/Agent%20Network.images/Snipaste_2024-03-09_06-31-33.png" alt="Snipaste_2024-03-09_06-31-33" style="width:50%;" /></div>

另外还有一些国外的软件是偶尔需要发起国外域名或 IP 请求的，如 PotPlayer 播放器。

PotPlayer 一般会在更新的时候发起访问国外域名或 IP 的请求，如果无法确定这些请求是否会命中 PROXY 策略，那么也可以使用 HTTP(S) 协议代理，这样未能命中的请求可确保使用 PROXY 策略进行访问。

<div align="center"><img src="images/Agent%20Network.images/Snipaste_2024-03-09_06-34-02.png" alt="Snipaste_2024-03-09_06-34-02" style="width:50%;" /></div>

还有一类仅支持系统代理的软件，如 Google Drive 网盘。

系统代理本身存在弊端，系统代理启用的情况下，一些本身默认支持系统代理的软件，会主动地去使用代理服务，即便软件本身不需要使用代理服务。简而言之，这会让某些不需要使用代理的软件，被迫使用代理。

因此多数情况下，不推荐使用系统代理。排除系统代理的选项后，Google Drive 这类软件的处境就无异于其他不支持代理的软件了。

### 支持代理的软件

本身就支持配置代理服务的软件，如果仍需要使用额外的流量转发软件来实现代理服务，这通常就意味着软件本身配置代理服务的方式较为繁琐，或者代理服务难以统一管理等。

#### Git & OpenSSH

使用 Git 克隆或推送仓库时，不可避免需要访问 GitHub 等网站，访问这类网站大多数时候需要代理服务的支持。

Git 本身支持配置代理服务：

```bash
git config --global http.proxy http://127.0.0.1:13766
```

但该代理服务实际存在很大的局限性，仅在进行  HTTP(S)  请求时代理服务能够被应用。即只有在使用 HTTP(S) 协议克隆仓库或拉取、推送仓库时，代理服务才有用。

如果推送仓库使用 SSH 协议（OpenSSH），那么 `http.proxy` 代理服务的配置将形同虚设。

所幸 OpenSSH 本身支持配置代理服务，但配置方式稍显复杂，需要借助其他的网络工具完成代理连接。Windows 系统下，为 OpenSSH 建立代理服务的连接的通常是 connect.exe 工具。

然而 connect.exe 工具并不存在于 OpenSSH 中，使用它需要安装额外的 MinGW 工具集。MinGW（Minimalist GNU for Windows）是一个在 Windows 平台上开发和运行本地 GNU 工具的项目。它提供了一组开发工具和库，使开发者能够在 Windows 上编译和运行类 Unix 环境下的软件，而无需依赖于 Microsoft 的 Visual Studio 或其他商业工具。

如若安装了 Git 版本控制工具，那么不必额外安装 MinGW 工具集，Git 安装目录中内置了 `connect.exe` 工具。

OpenSSH 实现代理连接的完整命令：

```bash
ssh -T 'git@github.com' -p 22 -i '~/.ssh/for_connect' -o 'ProxyCommand "e:/git/mingw64/bin/connect.exe" -S 127.0.0.1:13766 %h %p'
```

不愿意编写长命令，可以选择将参数写入 `~/.ssh/config` 配置文件中：

```
Host github.com
	HostName %h
	Port 22
	IdentityFile ~/.ssh/for_connect
	ProxyCommand e:/git/mingw64/bin/connect -S 127.0.0.1:13766 %h %p
```

但实际无论使用哪种配置方式，维护起来都十分麻烦。使用 ProxyCap 转发类似于 Git 或 OpenSSH 这类无法优雅地配置代理服务的工具的网络流量，无疑是更好的选择。

<div align="center"><img src="images/Agent%20Network.images/Snipaste_2024-03-09_06-29-59.png" alt="Snipaste_2024-03-09_06-29-59" style="width:50%;" /></div>

以上规则中，额外添加了 `Hostnames` 规则，有且仅当 `ssh.exe`、`git-remote-https.exe` 等程序发起对 `GitHub` 相关网站的请求时，会使用 ProxyCap 转发流量。

其中：

- `git-remote-https.exe`：Git 在拉取或推送 HTTPS 协议类型仓库时需要使用到的程序；
- `ssh.exe`：建立 OpenSSH 连接时需要使用到的程序。

因为实际使用 Git 或 OpenSSH 时，所能遇到的需要代理服务的情况多数只有在需要与 GitHub 网站建立连接的时候，如果明确知道需要代理服务访问的域名是什么，那么建议提供 `Hostnames` 规则。

#### Node.js

类似于 Node.js 这种网络框架，同样推荐使用 ProxyCap 来转发流量至代理服务。Node.js 中需要使用代理服务的无非就是 npm 包管理工具，因为多数的 npm 仓库位于国外，拉取仓库超时是常有的事。

`npm` 本身支持配置代理服务：

```bash
npm config set proxy http://127.0.0.1:13766/
```

该命令会生成 `~/.npmrc` 配置文件：

<div align="center"><img src="images/Agent%20Network.images/image-20240228112159295.png" alt="image-20240228112159295" style="width: 80%;" /></div>

还可以选择使用 ProxyCap 路由来自 `node.exe` 程序的域名匹配 `*registry.npmjs.org` 的流量：

<div align="center"><img src="images/Agent%20Network.images/Snipaste_2024-03-09_06-33-29.png" alt="Snipaste_2024-03-09_06-33-29" style="width:50%;" /></div>

因为 `npm` 软件包管理工具隶属于 Node.js 框架，所以 `npm` 的所有网络请求等同于间接由 `node.exe` 程序发起，使用 ProxyCap 管理 `node.exe` 程序即等同于管理了所有隶属于 Node.js 的程序。

于作用范围来说，规则中添加额外的 `*registry.npmjs.org` 域名限制是非常有必要的。

举个例子，假如 Axios 处于 Node.js 环境同时不存在额外的域名限制，那么在 ProxyCap 全权管理 Node.js 流量的情况下，Axios 发起的网络请求都会被 ProxyCap 所代理！

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

这意味着使用 Axios 框架测试某些代理连通性时，代理实际是否生效是完全无法得知的。

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

为了避免这种情况，额外的域名限制规则对于 ProxyCap 来说显得愈加重要。

### 编程软件

**所有编程软件如 VSCode、IDEA 或 PyCharm 等，均不建议使用 ProxyCap 来代理转发流量。**

一则编程软件通常支持配置代理服务，二则使用 ProxyCap 代理编程软件的网络流量，如果不能够提供详尽的域名规则，那么所有网络流量都将被转发至代理。

如果实际被转发至代理的是域名信息，那么根据代理服务的分流规则，还是能够避免出现大量未命中规则的请求。

然而经过实测，类似于 VSCode 软件，其转发至 ProxyCap 中的不是域名请求而是 IP 请求。这意味着，VSCode 将流量转发至 ProxyCap 前，这些域名请求就已经被本地 DNS 解析过了。

这样代理服务最终只会接收到纯 IP 请求，那么就大概率会导致分流问题。更何况在编程软件中，有不少的网络框架依赖自身的代理服务或不能依赖于代理服务，使用 ProxyCap 必然会影响这些网络框架的实际表现。

**为了避免不必要的编程问题，尽量不要将编程环境的网络结构复杂化。类似于 VSCode、IDEA 或 PyCharm 等编程软件，如果编程软件本身支持配置代理服务，就完全不必要也不推荐使用流量转发工具！**

### 分组实现

上述提到的基本都是如何制定更为合理的分流规则，而本节将讨论关于分组的最佳实现。

这里的代理组别（Proxy Group）一般也被称为策略组（Policy/Strategy Group），因为代理组一般都是根据某些策略来筛选所需节点之后组成的，所以称为策略组也许更为贴切。

对于分流规则中主要使用的策略组来说，最需要考量的点有两个：1. 节点延迟；2. 可用性。

如果策略组能够提供较低延迟的节点选用策略，同时保证其拥有较高的可用性，则该策略组就比较适合作为主力的策略组以供日常使用。

分组新手通常会选择将所有的可用节点都集中在同一个策略组内，并使用自动选择（URL-TEST）策略，同时将策略组属性 LAZY 配置为 TRUE 之后，便直接让该策略组作为主力策略组投入使用。

在节点量较少时，这种方式基本没有大问题。然而一旦节点较多，则这种分配方式大概率会带来较差的体验。

<div align="center"><img src="images/Agent%20Network.images/Low%20Availability.drawio.png" alt="Low Availability.drawio" style="width:80%;" /></div>

其中造成性能下降的根本原因，在于 Clash 所提供的三种较常用的自动选用节点的模式，它们都需要等待策略组内的所有节点完成延迟测试（Latency Test）之后，才能选定最终策略组所使用的目标节点。

那么一旦策略组所选节点不可用，并假设此时恰好触发了策略组的自动健康检查机制，那么策略组完全恢复其可用性的时间，将取决于组内所有节点完成延迟测试的时间。

这里直接推荐使用三层策略组的设计模式：

1. 顶层策略组（OUTER LAYER）：推荐手动选择（SELECT）；
2. 中层策略组（MIDDLE LAYER）：禁用 LAZY，推荐自动回退（FALLBACK）；
3. 底层策略组（INNER LAYER）：启用 LAZY，推荐自动选择（URL-TEST）或负载均衡（LOAD-BALANCE）。

<div align="center"><img src="images/Agent%20Network.images/High%20Availability.drawio.png" alt="High Availability.drawio" style="width:80%;" /></div>

使用三层策略组的设计模式，顶层策略组将具备高可用性，即不可用情况是小概率事件。一旦出现不可用情况，由于策略组使用手动选择模式，那么此时只需要手动切换到其他可用的中层策略组上即可。

如需进行延迟测试，并假设有 n 个中层策略组和 m 个可用节点，那么每个中层策略组也仅需要完成 m/n 个节点的延迟测试后，即可判断出该中层策略组的整体延迟情况。

而之所以顶层策略组具备高可用性，原因是中层策略组启用了健康检查。启用健康检查能让中层策略组每间隔一段时间自动检测节点的可用性，其中自动回退模式能确保组内的首个可用策略组能被选中。

中层策略组需根据其包含的节点质量，来配置合适的健康检查间隔。例如，在节点质量普遍较差时，配置一个较短的自检间隔；反之，则可以配置一个较长的自检间隔。

底层策略组包含基本的节点信息，同时不启用健康检查。这里不启用健康检查是因为中层策略组已经启用健康检查并引用了底层策略组，即相当于底层策略组间接启用了健康检查。

同时，只让中层策略组启用健康检查，能够保证某时段内每个节点有且仅会执行一次延迟测试，从而避免了复杂的策略组嵌套所带来的延迟测试问题，达到节省系统资源的目的。

如果不希望手动操作顶层策略组，那么只要确保中层策略组混用没问题，则将顶层策略组配置为其他可选的自动选择模式也可以，只要确保健康检查 interval 参数与中层策略组一致即可。

这里的 interval 参数如果太大的话，会导致组内的自动切换策略组操作变慢；但如果太小，又会导致出现过多的无用切换，且在中层策略组启用自检时，来自顶层的频繁自检请求并不会影响到中层本身的自动自检策略。

### 健康检查

关于 interval 健康检查（自检）间隔，该自检间隔在 lazy 配置为 true 的情况下亦生效。同时需要注意，有且仅有策略组选用手动选择模式（SELECT）时参数 lazy 无效。

当 lazy 配置为 true 时，内核将在策略组首次被使用时立即进行一次自检。如果在自检后的 interval 时间内没有其它连接请求，则自检不再进行；直到下次有新的连接请求进入该目标策略组时，自检才会进行。

- lazy: false => 策略组总是规律地间隔 interval 时间进行自检，并不受其他因素影响；
- lazy: true => 策略组仅在距离上次自检已经超过 interval 时间，且有连接请求（包括来自其他组的自检请求）时，才会进行自检。

由于这种自检机制的存在，那么使用 lazy: true 的底层策略组中的 interval 参数，其值的配置理论上应当略小于使用 lazy: false 的中层策略组中的 interval 参数。

假设它们的 interval 参数值一致，由于中层策略组发起自检请求实际并没有那么准时，那么可能出现某个底层策略组接收自检请求的间隔实际小于 interval 的情况，这将导致该底层策略组不触发自检机制。

因此，这里推荐为不同的配置逻辑设置不同的间隔容差，例如中层策略组使用的 interval 为 300 秒，底层策略组使用的 interval 为 280 秒，以保证中层策略组能够间接触发底层策略组的自检机制。

