### 概述

本文用于说明 OpenSSH 中的 ~/.ssh/config 配置文件的构成及注意事项。

### 配置文件的意义

SSH 配置存在的意义，是可以自定义 SSH 客户端的行为，以满足特定需求。SSH 配置文件是一个文本文件，它通常的存储路径为：

- ~/.ssh/config

该文件包含了 SSH 客户端的各种配置选项，可以用来配置 SSH 客户端的行为，例如：

- 定义主机别名：你可以使用 Host 块为不同的主机定义别名，这样在连接远程主机时只需要使用别名即可，大大提高了操作的便捷性。
- 配置身份验证方式：你可以使用 IdentityFile 选项指定 SSH 客户端使用哪个身份验证文件进行身份验证，这样就不需要每次手动指定身份验证文件。
- 配置 SSH 连接选项：你可以使用各种选项来配置 SSH 连接的行为，例如连接超时时间、重试次数、使用的加密算法等等。
- 配置 SSH 代理：你可以使用 ProxyCommand 选项配置 SSH 代理，以便在使用 SSH 连接到无法直接访问的主机时使用。
- 配置 SSH 转发：你可以使用各种选项来配置 SSH 转发，例如本地端口转发、远程端口转发等等。

通过灵活使用 SSH 配置文件，你可以自定义 SSH 客户端的行为，使其更符合你的特定需求。此外，如果你需要经常连接到多个远程主机，使用 SSH 配置文件还可以节省你的时间和精力，提高你的工作效率。

### 完整的配置示例

一个完整的 SSH 配置文件可能会包含许多参数，其中大部分都是可选的。以下是一个包含所有可用参数的 SSH 配置文件示例：

```
# Global options
Host *
    ForwardAgent yes
    ForwardX11 yes
    Compression yes
    ServerAliveInterval 60
    ServerAliveCountMax 3

# Host-specific options
Host example.com
    HostName example.com
    User username
    Port 22
    IdentityFile ~/.ssh/my_private_key
    IdentitiesOnly yes
    PreferredAuthentications publickey,password
    Protocol 2
    Ciphers aes128-ctr,aes192-ctr,aes256-ctr
    MACs hmac-sha2-256,hmac-sha2-512
    BatchMode yes
    ConnectTimeout 10
    TCPKeepAlive yes
    LogLevel DEBUG
    UserKnownHostsFile ~/.ssh/known_hosts
    StrictHostKeyChecking ask
    VisualHostKey yes
    ControlMaster auto
    ControlPath ~/.ssh/master-%r@%h:%p
    ControlPersist 600

# Host-specific options for a wildcard pattern
Host *.example.com
    ProxyCommand ssh -q gateway.example.com nc -q0 %h %p
```

这个实例中包含了三个部分。

- 第一部分定义了一些全局选项，包括打开代理转发、X11 转发、压缩和保持 SSH 连接的时间。
- 第二部分定义了一个名为 example.com 的主机，指定了用户名、端口、身份验证密钥等选项。我们还定义了一些其他选项，如身份验证首选项、协议版本、加密算法、日志级别等。
- 第三部分定义了一个使用通配符 *.example.com 的主机，为它指定了一个代理命令，以便通过网关主机连接到目标主机。

请注意，实际使用时，你不需要在 SSH 配置文件中指定所有可用选项，而是根据需要选择性地指定需要的选项。

### 注意事项

本节主要说明配置文件中的一些细节问题。

#### 缩进不必要

在 SSH 配置文件中，使用缩进可以让文件更易于阅读和理解。但是，缩进不是必需的，你可以选择不使用缩进来编写 SSH 配置文件。

然而，如果你不使用缩进，则需要确保每个选项都位于独立的一行上，且选项名和选项值之间需要用空格隔开。以下是一个不使用缩进的 SSH 配置文件示例：

```
Host example.com
HostName example.com
User username
Port 22
IdentityFile ~/.ssh/my_private_key
IdentitiesOnly yes
PreferredAuthentications publickey,password
Protocol 2
Ciphers aes128-ctr,aes192-ctr,aes256-ctr
MACs hmac-sha2-256,hmac-sha2-512
BatchMode yes
ConnectTimeout 10
TCPKeepAlive yes
LogLevel DEBUG
UserKnownHostsFile ~/.ssh/known_hosts
StrictHostKeyChecking ask
VisualHostKey yes
ControlMaster auto
ControlPath ~/.ssh/master-%r@%h:%p
ControlPersist 600
```

在这个示例中，每个选项都位于独立的一行上，且选项名和选项值之间使用空格隔开。

请注意，即使不使用缩进，每个 Host 块的开头行也需要以“Host”开头，并且主机名称需要紧跟在后面。简而言之，配置之间是以“Host”作为分界的，每个配置都被称为 Host 块。

虽然可以不使用缩进来编写 SSH 配置文件，但还是建议使用缩进来提高文件的可读性和可维护性。

#### Host 块（别名）

使用 SSH 配置文件可以简化连接远程主机的操作，通过在 SSH 配置文件中定义 Host 块，你可以为不同的主机配置不同的选项，例如 HostName、User、IdentityFile 等等。

这样，当你要连接到某个主机时，只需要使用该主机在 SSH 配置文件中定义的别名，而不需要再次输入主机名、用户名、身份验证文件等信息。

例如，你在 SSH 配置文件中定义了以下 Host 块：

```
Host myserver
    HostName example.com
    User John
    IdentityFile ~/.ssh/my_private_key
```

那么，当你想要连接到 example.com 主机时，只需要使用以下命令即可：

```bash
ssh myserver
```

SSH 将会自动使用 HostName 为 example.com，User 为 John，IdentityFile 为 ~/.ssh/my_private_key 的配置。

这样，你就可以在连接到多个远程主机时，大大简化操作流程，提高工作效率。

但需要注意，无论 ssh 命令中提供什么样的命令，SSH 都会默认会先匹配 Host，之后再根据命令中是否提供了 User 来进一步确定连接地址。

例如，存以下的配置文件：

```
Host github.com
    HostName %h
    User John
    IdentityFile ~/.ssh/my_private_key
```

假设 GitHub 能够被正常访问，命令行发起以下连接请求：

```bash
ssh -T git@github.com
```

SSH 会先把主机名 github.com 取出来匹配 Host 别名，因为任何情况下它首先可能是一个别名。

如果存在 Host 为 github.com 的 Host 块时，SSH 会根据发起的请求是否具有 User 信息，来进一步确定是否使用配置中的默认 User 信息。

由于以上的连接请求中使用了指定的 User（git），因此它将覆盖配置中默认的 User（John）作为连接的发起者。

如果发起不带用户名的连接请求：

```bash
ssh -T github.com
```

它将直接匹配到 Host 别名中的配置，其等价于：

```bash
ssh -T John@github.com
```

现在修改一下配置文件：

```
Host alias
    HostName github.com
    User John
    IdentityFile ~/.ssh/my_private_key
```

由于别名的存在，如果希望访问到远程主机 github.com 则必须要在命令中使用别名：

```bash
ssh -T git@alias
```

诸如以下的 SSH 连接请求：

```bash
ssh -T github.com
ssh -T git@github.com
```

均无法成功匹配到 Host 块，这些请求一般会以找寻不到私钥的存在，导致验证失败而告终：

```
Permission denied (publickey).
```

之前说过，Host 一般用于分割配置，即如果存在多个配置，则 Host 必须提供。否则多个配置就会被视为同一个配置，这样是十分不合理的。

如果只有一个配置呢？是否可以省略 Host 配置？答案是肯定的，但实际并不推荐这样做。

假设存在以下配置：

```
HostName github.com
User John
IdentityFile ~/.ssh/my_private_key
```

该配置实际等价于：

```
Host github.com
    HostName %h
    User John
    IdentityFile ~/.ssh/my_private_key
```

即所有主机写作 github.com 的连接都会匹配这条规则。  

#### 大小写无限制

在 SSH 配置文件中，大小写通常是不敏感的。这意味着你可以使用大写字母、小写字母或混合大小写字母来编写选项名和主机名，它们都会被正确地解析和识别。例如，下面的两个 Host 块是等价的：

```
Host example.com
    HostName example.com
    User username
    IdentityFile ~/.ssh/my_private_key

host example.com
    hostname example.com
    user username
    identityfile ~/.ssh/my_private_key
```

无论你是使用大写字母、小写字母或混合大小写字母编写选项名和主机名，SSH 都会将它们解释为相同的选项名和主机名。因此，在 SSH 配置文件中，大小写通常是没有限制的。