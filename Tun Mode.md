### TUN 模式

TUN 是内核提供的三层虚拟网络设备，由软件实现来替代真实的硬件，相当于在系统网络栈的三层（网络层）位置开了一个口子，将符合条件（路由匹配）的三层数据包交由相应的用户空间软件来处理，用户空间软件也可以通过 TUN 设备向系统网络栈注入数据包。

一般的代理工具都支持 TUN。以 Clash 内核的工具为例，启用 TUN 模式后，代理工具会在本地部署一张虚拟网卡以接管所有的网络流量数据。

本地 DNS 服务与实体网卡绑定，虚拟网卡接管网络流量后，本地 DNS 服务将不再生效。因此虚拟网卡如果需要域名解析服务，则必须提供相关的 DNS 配置以初始化虚拟网卡的解析服务。

### 版本环境

- Windows 11 23H2 22631.3007
- Clash for Windows 0.20.39 - 2023.08.17 Premium
- Clash Verge 1.3.8 - v1.18.0 Meta

### DNS 配置

Clash 配置文件中常见的 DNS 配置：

```yaml
dns:
  enable: false
  ipv6: false
  use-hosts: true
  listen: 0.0.0.0:53
  enhanced-mode: fake-ip
  fake-ip-range: 192.18.0.1/16
  fake-ip-filter:
    - "*.lan"
    - localhost.ptlogin2.qq.com
    - +.stun.*.*
    - +.stun.*.*.*
    - +.stun.*.*.*.*
    - +.stun.*.*.*.*.*
    - "*.n.n.srv.nintendo.net"
    - +.stun.playstation.net
    - xbox.*.*.microsoft.com
    - "*.*.xboxlive.com"
    - "*.msftncsi.com"
    - "*.msftconnecttest.com"
    - "*.logon.battlenet.com.cn"
    - "*.logon.battle.net"
    - WORKGROUP
  default-nameserver:
    - 119.29.29.29
    - 119.28.28.28
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
  fallback:
    - https://doh.dns.sb/dns-query
    - https://dns.cloudflare.com/dns-query
  fallback-filter:
    geoip: true
    geoip-code: CN
    ipcidr:
      - 240.0.0.0/4
```

- enable：是否启用 dns 配置。非 TUN 模式下建议设置为 false 以启用本地的 DNS 解析服务；启用 TUN 模式时，该值会被自动设置为 true；
- ipv6：是否启用 ipv6 网络协议。该协议一般需要网络运营商和路由器的同时支持，访问 [IPv6 连接测试](https://test-ipv6.com/) 可以查看是否具有 ipv6 网络；
- use-hosts：是否使用系统 hosts 文件中的域名映射；
- listen：设定 DNS 服务监听的地址和端口。一般为 0.0.0.0:53 或 127.0.0.1:53，前者表示监听来自所有网络的 DNS 请求，后者则表示只监听来自本机网络的 DNS 请求。如果经常处于公共网络中，则建议将 listen 字段配置为后者；
- enhanced-mode：设定增强模式。截止目前为止，仅有 redir-host 和 fake-ip 两种增强模式，且 redir-host 基本已被 fake-ip 全面取代；
- fake-ip-range：设定虚假 IP 地址段。其中 192.18.0.1/16 表示前 16 位的网络部分保持不变，后 16 位的主机部分可以改变。因此该 IP 段可以表示 192.18.0.0 ~ 192.18.255.255 共计 65536 个 IP 地址；
- fake-ip-filter：设定直连的域名规则。满足这些规则的域名可以直接使用 TUN 模式提供的 DNS 解析服务进行访问；
- default-nameserver：仅能设定 IP 式的 DNS，且仅用于解析 nameserver 或 fallback 中 DoH 或 DoT 式的 DNS。如果不在 nameserver 或 fallback 中添加 DoH 或 DoT，那么本字段可以省略；
- nameserver：设定国内的 DNS 服务。允许设定 Do53、DoH 或 DoT 式的各种 DNS 服务；
- fallback：设定国外的 DNS 服务。允许设定 Do53、DoH 或 DoT 式的各种 DNS 服务；
- fallback-filter：设定解析域名后的分流规则。该分流规则仅在域名被判定为直连（DIRECT）时触发，主要用于判断解析得到的 IP 地址具体需要使用的 DNS 服务是 nameserver 还是 fallback。

### 规则模式

代理工具如果使用的代理模式为全局模式（Global），那么所有的网络请求都只会经由代理发出，完全不需要使用到虚拟网卡提供的 DNS 解析服务。

一般只有代理模式为规则模式（Rule）时，才需要使用到虚拟网卡提供的 DNS 解析服务。

所谓的规则，指的是用于匹配域名、IP 或进程名等等条件的规则，它可以是单条规则，也可以是多条规则组成的规则列表。

判定目标一旦命中规则，则会按照规则指定的模式发起网络请求。常见的策略（POLICY）有三种：

- DIRECT：直连策略。将使用 DNS 配置中的解析服务来解析域名，并根据分流规则（fallback-filter）发起最终的网络请求；
- PROXY：代理模式。判定目标的网络流量将由代理接管，即网络请求全盘会交由代理发出；
- REJECT：拒绝模式。本次网络请求将被立即终止。

假设存在以下规则：

```yaml
rules:
  - RULE-SET,direct,DIRECT
  - RULE-SET,proxy,PROXY
  - RULE-SET,reject,REJECT
  - RULE-SET,cncidr,DIRECT,no-resolve
  - GEOIP,CN,DIRECT
  - MATCH,DIRECT
```

那么：

- 判定目标如果匹配到规则列表 direct，那么所有流量都会使用 DIRECT 直连策略；
- 判定目标如果匹配到规则列表 proxy，那么所有流量都会使用 PROXY 代理策略；
- 判定目标如果匹配到规则列表 reject，那么所有流量都会使用 REJECT 拒绝策略。 

启用 TUN 模式并使用 fake-ip 时，处理网络流量的大致过程如下：

1. 浏览器发出某个域名的请求，该请求将转发至虚拟网卡；
2. 浏览器必须接收到一个实际 IP 作为返回，fake-ip 模式下 Clash 会从虚拟 IP 池 fake-ip-range 中取出一个虚假 IP 与目标域名作映射并保存，同时将这个虚假 IP 返回给浏览器；
3. 浏览器接收到虚假 IP 后，会立即对该 IP 发起网络请求；
4. 请求将再次来到虚拟网卡，通过查询映射表可以从中检索真实域名，域名将根据规则分流。

### 解析场景

规则模式下，如果判定目标所匹配到的模式为 PROXY 代理模式，那么该流量则不需要使用 DNS 解析服务；同理，如果匹配到 REJECT 拒绝模式，显然亦不需要进行 DNS 解析。

换言之，只有判定目标所匹配到的模式为 DIRECT 直连策略时，才需要使用 DNS 解析服务。

但还有一种特殊的情况，需要使用到 DNS 解析服务：

- 当判定目标为域名，且匹配到未添加 no-resolve 选项的 IP 规则时，需要使用 DNS 解析服务获取域名的真实 IP 地址，以进一步匹配该 IP 规则

实际上 TUN 中内置的 DNS 原本就是用于服务这种情况下的 DNS 解析，其中的 DNS 解析优先级同样是为了解决这种情况而被设立。

所谓的 IP 规则即用于匹配 IP 地址的规则。如果目标网站没有实体域名，仅存在可直接访问的公网 IP 地址，那么在浏览器中输入该 IP 地址即可直接访问到目标网站。

这种情况下，使用 IP 规则就能直接判断该 IP 应该使用代理模式、直连策略还是拒绝模式。

但如果访问的目标是域名而非 IP 地址，如何判断域名是否命中 IP 规则呢？解决的方法很简单，使用 DNS 解析服务获取域名的真实的 IP 地址后，再使用该 IP 来确认是否命中 IP 规则即可。

但并非所有的 IP 规则都允许域名解析：

- 如果 IP 规则之后添加了 no-resolve 选项，则表示域名匹配该 IP 规则时不允许使用 DNS 解析
- 如果 IP 规则之后未添加 no-resolve 选项，则表示域名匹配该 IP 规则时被允许使用 DNS 解析。

即添加了 no-resolve 选项的 IP 规则，对于域名来说是无效规则。

综上可知，只有两种情况需要使用到内置的 DNS 解析服务：

1. 域名匹配到未添加 no-resolve 的 IP 规则；
2. 域名匹配到 DIRECT 规则。

### 解析过程

DNS 配置中有且仅有 nameserver 和 fallback 两个字段能够配置用于解析域名的 DNS 服务器，以下内容将以域名匹配到未添加 no-resolve 的 IP 规则为例。

稍微注意一下 default-nameserver 字段，该字段可以配置 IP 形式的 DNS 服务器，但这里面的 DNS 只用于解析  nameserver 和 fallback 中配置的 DoH 或 DoT 形式的 DNS 服务器。

如果不在 nameserver 和 fallback 中使用 DoH 或 DoT，则 default-nameserver 字段可以被省略。

一种较为约定俗成的配置策略为：

- nameserver：配置国内的、高可用的 DNS 服务器；
- fallback：配置国外的、高可用的 DNS 服务器。

本质上 nameserver 和 fallback 并无硬性规定 DNS 服务器的所属地，但之所以说是“约定俗成”的配置策略，一个很重要的原因是 nameserver 和 fallback 之间存在优先级的差异：

- nameserver 必须等待 fallback 的解析结果

考虑到 DNS 污染的情况，如果将 fallback 配置为国外的、高可用的 DNS 服务器，则有更大的概率能够解析出国外域名的真实 IP 地址。

理想情况下的解析结果是：

- nameserver 能够解析国内域名；
- nameserver 解析国外域名存在 DNS 污染，但 fallback 能够解析国外域名。

那么无论如何，一个真实的 IP 总能够被返回至 IP 规则处，从而完成 IP 规则的匹配。

然而真实的情况并非如此，因为并没有那么多高可用的国外 DNS 服务器存在。

一旦 fallback 中配置的国外 DNS 服务器拥有较高的延迟且几乎无法解析得到真实的 IP 地址，那么实际的解析过程将会变得十分糟糕：

- nameserver 解析国外域名存在 DNS 污染，需等待 fallback 响应但大概率出现解析超时；
- nameserver 能够解析国内域名，但仍需要等待 fallback 响应；即便 fallback 小概率解析出了正确的 IP 地址，那也可能会使原本能够直连的域名被迫需要使用代理连接。

这种糟糕的解析体验，同时也影响着使用 DIRECT 直连策略的域名。

使用 DIRECT 直连策略的域名经解析后，最终还需要匹配 fallback-filter 分流规则：

```yaml
fallback-filter:
  geoip: true
  geoip-code: CN
  ipcidr:
    - 240.0.0.0/4
    - 0.0.0.0/32
```

以上的分流规则可解释为：

- 当解析的域名在 GeoIP 数据库内的国家代码不是 `CN` 时；
- 当 nameserver 中的 DNS 解析结果位于 `240.0.0.0/4` 这一 IP 段内时。（这种情况通常是解析结果被污染了）

该规则只用于判断发起最终网络请求的是 nameserver 中的服务器还是 fallback 中的服务器。

但由于高可用的国外 DNS 服务器几乎不存在，如果目标是国外域名，那它必然会使用 fallback 中的 DNS 服务发起网络请求，其结果也几乎必为“连接不可用”。

其实，期盼使用国外的 DNS 服务器解析出国外域名的真实 IP 这件事情，已经挺匪夷所思了。

但期盼能够使用国外的 DNS 服务器建立连接以正常访问国外的网站，那更是异想天开的存在。如果能够访问成功，代理好像也就没有存在的意义了。

### 配置优化

以“高可用的国外 DNS 服务器几乎不存在”为优化的基础，适当地简化 DNS 配置可以大大缩短解析域名时所消耗的时间，以提高使用代理网络的体验。

配置优化的方式很简单：从 DNS 配置中移除 fallback 字段即可。

原因也很简单，如果本就无法成功解析，那便根本没必浪费这个时间去等待解析。

```yaml
dns:
  enable: false
  ipv6: false
  use-hosts: true
  listen: 0.0.0.0:53
  enhanced-mode: fake-ip
  fake-ip-range: 192.18.0.1/16
  fake-ip-filter:
    - "*.lan"
    - localhost.ptlogin2.qq.com
    - +.stun.*.*
    - +.stun.*.*.*
    - +.stun.*.*.*.*
    - +.stun.*.*.*.*.*
    - "*.n.n.srv.nintendo.net"
    - +.stun.playstation.net
    - xbox.*.*.microsoft.com
    - "*.*.xboxlive.com"
    - "*.msftncsi.com"
    - "*.msftconnecttest.com"
    - "*.logon.battlenet.com.cn"
    - "*.logon.battle.net"
    - WORKGROUP
  default-nameserver:
    - 119.29.29.29
    - 119.28.28.28
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
```

另外需要说明 fallback-filter 字段，该字段是 clash-core 中自带的规则字段，即便配置中省略了，但实际该规则仍然内置、无法被移除。

### 注意事项

PC 端如果处于非 TUN 模式，则不建议启用 DNS 配置。

如果在非 TUN 模式下启用 DNS 配置，则：

1. 53 端口将持续处于被监听状态，多开 Clash 会出现端口监听失败的错误日志；
2. nameserver 和 fallback 中的 DNS 会被用于解析规则模式下其要求解析的域名，同时本地 DNS 会被用于解析一些未经 Clash 接管的网络流量的解析请求。

为了避免未知错误和优化系统性能，推荐默认情况下将 dns.enable 的值设置为 false。无需担心启用 TUN 模式时 DNS 解析的状态，dns.enable 的值在 TUN 启用时会被自动设置为 true。

手机、平板等移动设备的代理网络如果涉及到 DNS 配置，则推荐将 dns.enable 设置为 true。因为大多移动设备上的代理 APP 程序，其实现代理上网的模式通常就是 TUN 模式。

另外，关于 CFW 中配置文件内存在的 DNS 配置的说明：

- 在 TUN 启用时，所有配置文件内的 DNS 配置会被软件内置的 DNS 配置所覆盖；
- 在 TUN 关闭时，如果配置文件内的 DNS 配置为启用状态，该 DNS 配置会生效。

综上，无论何时配置中的 DNS 都应该配置为关闭。

### no-resolve

IP 规则中可以添加 no-resolve 以指示域名在遇到 IP 规则时，不进行 DNS 解析。

出于性能考虑，一般规则会根据域名或 IP 的所属地域来针对性地编写，常见的有黑名单和白名单两种模式：

- 黑名单模式：规则中只存在国外的域名或 IP 规则，未命中的域名或 IP 使用 DIRECT 策略；
- 白名单模式：规则中只存在国内的域名或 IP 规则，未命中的域名或 IP 使用 PROXY 策略。

其中，有且仅有使用 DNS 服务解析国外域名时，可能发生 DNS 泄露。

结合 IP 规则一般被置于所有域名规则之后、未命中规则之前的配置方式，可知：

- 黑名单模式下，大多数的国外域名都会命中域名规则，只有少量的域名无法命中，需要匹配到 IP 规则。但由于该模式下，未命中的域名最终使用的是 DIRECT 策略，这意味着 DNS 解析总会发生，因此 IP 规则不要添加 no-resolve 选项。

- 白名单模式下，仅国内的域名会命中域名规则。如果 IP 规则不添加 no-resolve 选项，则大量未命中的国外域名都会使用 DNS 解析。该模式下，由于未命中域名最终使用 PROXY 策略，为了避免大量的国外域名使用本地 DNS 解析，应尽量为 IP 规则添加 no-resolve 选项。

如果系统性能允许，规则模式中的规则应当尽量可能地完备。在规则尽可能完备的情况下，推荐未命中规则使用 DIRECT 策略，同时 IP 规则不需要添加 no-resolve 选项，这样只会存在少量的国外域名未能命中域名规则。

使用这种配置，未能命中的少量国外域名，由于最终总会流向 DIRECT 策略，这说明 DNS 解析总会发生。那么，为 IP 规则添加 no-resolve 选项，以提前阻止 DNS 解析国外域名的行为，就会变得毫无意义。

