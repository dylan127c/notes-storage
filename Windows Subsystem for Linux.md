### 概述

Windows Subsystem for Linux（WSL）全称适用于 Linux 的 Windows 子系统。

WSL 可以让开发人员直接在 Windows 上按原样运行 GNU/Linux 环境（包括大多数命令行工具、实用工具和应用程序），且不会产生传统虚拟机或双启动设置开销。详情参考：[WSL 文档](https://learn.microsoft.com/zh-cn/windows/wsl/)。

### Linux 发行版

在 WSL 尚未安装任何 Linux 发行版的情况下，使用 Windows Terminal 运行以下命令：

```shell
wsl
```

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-18-31.png" alt="Snipaste_2024-03-12_18-18-31" style="width:80%;" /></div>

~~该中文版说明中，会出现许多“分发”的字眼，这个翻译实际并不准确。英文文档中 distribution 在中文文档里不应该被翻译为“分发”，而应翻译为“发行版”。~~

随着 WSL 版本的更新，许多“分发”已被更正为“发行版”，但尽管如此，仍旧存在部分文档使用“分发”而不是“发行版”。总之，英文文档内 distribution 在中文文档内不应翻译为“分发”，而应翻译为“发行版”。

Linux 发行版即 Linux Distribution，它也被称作 GNU/Linux 发行版。它是为一般用户预先集成好的 Linux 操作系统，用户不需要重新编译，直接安装发行版后，只需要小幅度更改设置即可使用。

Linux 发行版通常以软件包管理系统来进行应用软件的管理，一般的 Linux 发行版都会预先集成包括桌面环境、办公包、媒体播放器、数据库等应用软件（类比 Android 的预装应用）。

微软在许多需要说明命令用途的地方，将 distribution 译为“分发”，具有一定的迷惑性。因为在 Linux Distribution 等专有名词中，distribution 应该被翻译为“发行版”而非“分发”。

另外，Linux Distribution 常被简写成 Linux Distro，其中 distro 是 distribution 的缩写形式，它们都是“发行版”的意思。可以将 Linux 发行版看作 Linux 系统的子集，或者说 Linux 发行版是 Linux 系统的具体解释。

例如 CentOS 是 Linux 发行版，但同时它也是 Linux 系统。从某种意义上来说，亦可将 Linux 发行版与 Linux 系统看作是同一种东西。更多关于 Linux 发行版的资料，请参考：[Linux Distribution - WIKI](https://zh.wikipedia.org/wiki/Linux%E5%8F%91%E8%A1%8C%E7%89%88)。

### 终端命令

后续涉及诸多 Windows Terminal 命令，这里有个注意事项：终端命令选项提供的参数中不能包含空格符号。

在 Bash 或 Shell 中，空格通常用来分割命令行的选项。如果选项参数中包含空格符号，则需要使用单引号或双引号将其包裹，或将空格转义。

假设存在一个 `h:\notes storage` 目录，以下将演示不同终端进入该目录的方法。

CMD：（命令顺序不重要，但盘符切换不能使用 `cd` 命令，<u>不能使用双引号</u>）

```bash
h:
cd "notes storage"
```

Windows Terminal（PowerShell，<u>兼容双引号</u>）：

```bash
cd 'h:\notes storage'
```

Bash（Shell，<u>兼容双引号</u>）：

```bash
# 反斜杆转义空格符号
cd /h/notes\ storage

# 直接使用单引号包裹
cd '/h/notes storage'
```

对于不包含空格符号的参数来说，单引号或双引号可省略不写。但如果是为了养成良好的编程习惯，那么不省略单引号或双引号是更好的选择。

### WSL

大多数 Windows 10 或以上版本的系统默认支持 WSL 技术，这里可以通过区分系统版本，来判断当前系统具体支持的 WSL 版本有哪些，因为同一个 Windows 系统中允许同时存在 WLS 1 和 WSL2。

| 功能                                           | WSL 1 | WSL 2 |
| :--------------------------------------------- | :---- | :---- |
| Windows 和 Linux 之间的集成                    | ✅     | ✅     |
| 启动时间短                                     | ✅     | ✅     |
| 与传统虚拟机相比，占用的资源量少               | ✅     | ✅     |
| 可以与当前版本的 VMware 和 VirtualBox 一起运行 | ✅     | ✅     |
| 托管 VM                                        | ❌     | ✅     |
| 完整的 Linux 内核                              | ❌     | ✅     |
| 完全的系统调用兼容性                           | ❌     | ✅     |
| 跨 OS 文件系统的性能                           | ✅     | ❌     |

WSL 目前仅有 WSL1 和 WSL2 两个版本，就性能表现而言更推荐使用 WSL2。如果 Windows 系统版本号低于 18917 则表示当前系统只支持 WSL1，如果高于 18917 则表示系统同时支持 WSL1 和 WSL2。

查看系统版本信息，可以使用 CMD 命令提示符：

```bash
ver
```

活着使用 Windows Terminal：

```shell
cmd 'ver'
```

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-19-38.png" alt="Snipaste_2024-03-12_18-19-38" style="width:40%;" /></div>

图例所示 Windows 版本为 22621，即表示当前系统同时支持 WSL1 和 WSL2。

WSL 程序默认存储在 `c:\windows\system32` 目录中：

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/image-20230423070627519.png" alt="image-20230423070627519" style="width: 80%;" /></div>

虽然程序已存在于系统中，但实际尚未可用，直接调用该程序是无效的。

实际上直接运行 `wsl.exe` 程序相当于在终端中执行不带任何参数的 `wsl` 命令，这时候该程序就类似于 Linux 发行版的启动器，只用于启动当前系统中已安装的、默认的 Linux 发行版系统。

#### 1. 选择 WSL2 版本

WSL2 相较于 WSL1 拥有更好的虚拟性能，因此推荐在安装的 Linux 发行版均使用 WSL2。

安装 Linux 发行版前，建议手动将当前默认的 WSL 版本配置为 WSL2：

```bash
wsl --set-default-version '2'
```

因为虽然系统支持 WSL2，但不一定会默认使用 WSL2！上述命令只需要执行一次，后续安装的发行版都会默认使用 WSL2。

#### 2. 安装 Linux 发行版

官方教程将“安装 Linux 发行版”描述为“安装 WSL”实际不太准确，与其说“安装 WSL”倒不如说是“将指定的 Linux 发行版安装到本地”。只有本地安装了的 Linux 发行版之后后，才能使用 WSL 访问并使用它。

以下命令会将默认的 Linux 发行版安装到本地：

```shell
wsl --install
```

该命令实际等同于：

```shell
wsl --install --distribution 'Ubuntu'
```

因为这里默认的 Linux 发行版就是 Ubuntu 发行版。其中

- `--distribution`：用于指定 Linux 发行版的名称。

并不是任意的 Linux 发行版都能够使用 `--distribution` 指定，WSL 拥有一个受支持的 Linux 发行版列表。

查询受 WSL 支持的 Linux 发行版列表：

```shell
wsl --list --online
```

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-20-24.png" alt="Snipaste_2024-03-12_18-20-24" style="width:80%;" /></div>

注意，该命令本质上是从 `raw.githubusercontent.com` 中获取数据，如果拉取超时建议多尝试几次。

其中：

- NAME：该列为 `--distribution` 选项所能接收的参数；
- FRIDENDLY NAME：该列为更加详尽的版本名称。

列表中以 `*` 符号开头的行，表示默认发行版。未指定 `--distribution` 选项的参数时，将自动使用默认发行版。

选择适合自己的 Linux 发行版安装即可，例如指定安装 Ubuntu-22.04 发行版：

```shell
wsl --install -d 'Ubuntu-22.04'
```

安装完毕需要重启计算机以应用修改，重启后 WSL 会自动启动并进入用户名称和密码的配置：

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-10-27.png" alt="Snipaste_2024-03-12_18-10-27" style="width:80%;" /></div>

另外 WSL 本身支持多个发行版同时存在，有需要时可以使用安装命令安装多个相同或不同的 Linux 发行版。但存在一个小问题，WSL 并未提供任何修改已安装 Linux 发行版名称的命令。

那么，对于同一个 Linux 发行版来说，同一时间有且仅能存在一个。目前只有导入 Linux 发行版的备份时，能够通过导入名称的方式来间接修改已安装发行版的名称。

**由于需要重启调用 Linux 配置的原因，这里建议每次安装时只部署一个 Linux 发行版。如果一次部署多个发行版，那重启后大概率会出现一些未知的 WSL 问题。**

#### 3. 查看 Linux 发行版

查看所有已安装 Linux 发行版的信息：

- NAME 表示 Linux 发行版名称；
- STATE 表示 Linux 发行版所处状态；
- VERSION 表示 Linux 发行版使用的 WSL 版本。

```shell
wsl --list --verbose
```

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-22-32.png" alt="Snipaste_2024-03-12_18-22-32" style="width:80%;" /></div>

#### 4. 默认 Linux 发行版

查看 Linux 发行版信息时，位于行首的 `*` 符号表示调用 wsl.exe 程序时默认使用的 Linux 发行版。

上述例子中 Ubuntu-22.04 即为默认的 Linux 发行版。这意味着调用 wsl.exe 程序时，默认启动并使用的 Linux 发行版为 Ubuntu-22.04。

进入 wsl.exe 程序后，运行以下命令可以查询 Ubuntu 系统的版本号：

```bash
lsb_release -a
```

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-24-58.png" alt="Snipaste_2024-03-12_18-24-58" style="width:80%;" /></div>

回到 Windows Terminal 中执行以下命令，可以更改默认启动的 Linux 发行版：

```shell
wsl --set-default <distrubution name>
```

如果需要将默认启动的发行版更改为 Ubuntu-20.04 那么只需要运行命令：

```bash
wsl --set-default 'Ubuntu-20.04'
```

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-26-20.png" alt="Snipaste_2024-03-12_18-26-20" style="width:80%;" /></div>

再次调用 wsl.exe 打开 Linux Bash 终端，并查询 Ubuntu 版本信息：

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-26-53.png" alt="Snipaste_2024-03-12_18-26-53" style="width:80%;" /></div>

可以看到发行版信息已改变，这意味着默认使用的 Linux 发行版确实改变了。

#### 5. 更改 WSL 版本

如果一开始并没有配置 WSL 默认版本，即安装 Linux 发行版前没有运行以下命令：

```shell
wsl --set-default-version '2'
```

那么查看当前所有已安装的 Linux 发行版时，就可能出现 VERSION 为 1 的情况：

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-27-38.png" alt="Snipaste_2024-03-12_18-27-38" style="width:80%;" /></div>

示例中 Ubuntu-22.04 发行版使用的 WSL 版本为 WSL1，所幸这并非无可挽救。

以下命令用于更改当前已安装 Linux 发行版所使用 WSL 版本：

```shell
wsl --set-version <distribution name> <versionNumber>
```

在上述例子中，需要运行命令：

```shell
wsl --set-version 'Ubuntu-22.04' '2'
```

WSL 版本的转换需要一定的时间：

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-28-22.png" alt="Snipaste_2024-03-12_18-28-22" style="width:80%;" /></div>

转换完毕后，可查看到对应 Linux 发行版所使用的 WSL 版本已变更：

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-28-44.png" alt="Snipaste_2024-03-12_18-28-44" style="width:80%;" /></div>

#### 6. 备份 Linux 发行版

使用虚拟系统的最大优点，莫过于系统的可恢复性了。WSL 提供了方便的导入/导出发行版系统的命令，可随时将 Linux 发行版导出为 `.tar` 文件，或将 `.tar` 文件恢复成 Linux 发行版。

以下命令用于导出 Linux 发行版：

```shell
wsl --export <Distribution Name> <FileName>
```

以下命令用于导入 Linux 发行版：

```shell
wsl --import <Distribution Name> <InstallLocation> <FileName>
```

以 Ubuntu-22.04 为例，需要将它导出至系统磁盘，则运行以下命令：

```shell
wsl --export 'Ubuntu-22.04' 'C:\Ubuntu-22.04.tar'
```

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-30-40.png" alt="Snipaste_2024-03-12_18-30-40" style="width:80%;" /></div>

后续如果需要将 Ubuntu-22.04 重新导入 WSL 中，则运行以下命令：

```shell
wsl --import 'My-Ubuntu' 'I:\Virtual Machine' 'C:\Ubuntu-22.04.tar'
```

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-31-21.png" alt="Snipaste_2024-03-12_18-31-21" style="width:80%;" /></div>

从文件导入的 Linux 发行版的 NAME 为导入时指定的发行版名称：

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-33-03.png" alt="Snipaste_2024-03-12_18-33-03" style="width:80%;" /></div>

#### 7. 卸载 Linux 发行版

假如不再需要使用某个 Linux 发行版，或需要删除当前的 Linux 发行版重新安装或导入其他的同名的发行版时，就需要使用到卸载或注销 Linux 发行版的命令。

以下命令用于卸载或注销指定名称的 Linux 发行版：

```shell
wsl --unregister <DistributionName>
```

以 My-Ubuntu 发行版为例，卸载它需要运行命令：

```shell
wsl --unregister 'My-Ubuntu'
```

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-33-41.png" alt="Snipaste_2024-03-12_18-33-41" style="width:80%;" /></div>

卸载或注销操作会一并将相关的文件删除，不用担心文件冗余的情况。

#### 8. 其他 WSL 操作

以下命令用于更新 WSL：

```shell
wsl --update
```

该命令有一个可选的 -\-web-download 选项，使用它表示更新会从 GitHub 下载，而不从 Microsoft Store 下载。

以下命令用于关闭 WSL：

```shell
wsl --shutdown
```

该命令会立即终止所有正在运行的发行版和 WSL 2 轻量级实用工具虚拟机。

如果只需要终止指定的发行版或阻止其运行，可以使用以下命令：

```shell
wsl --terminate <distribution name>
```

例如，仅终止 Ubuntu-20.04 发行版的运行：

```shell
wsl --terminate 'Ubuntu-20.04'
```

**实际上，退出 Linux Bash 之后对应的 Linux 发行版会在数秒后自动停止运行。**

不过存在一种特殊的方法，使用该方法能让 Linux 发行版保存运行状态，并让其能够随 Windows 系统的启动而自动运行。

### 访问 Linux 文件

如果要从 Windows 访问 Linux 文件，可通过路径访问：

```
\\wsl$\<distribution-name>
```

以 Ubuntu-20.04 为例，需要访问 Linux 系统文件，则访问以下路径即可：

```
\\wsl$\Ubuntu-20.04
```

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-34-15.png" alt="Snipaste_2024-03-12_18-34-15" style="width:80%;" /></div>

Linux 发行版的虚拟驱动器通常名为 ext4.vhdx 且被存储在 AppData 内相关的 WSL 目录中，这种虚拟驱动器通常可以被挂载。

不过官方不建议使用任何 Windows 工具或编辑器修改、移动或访问 AppData 目录中的 WSL 相关文件，这可能导致 Linux 系统损坏。使用路径访问 Linux 文件，是官方所推荐的唯一访问形式。

但该形式存在一个问题，Linux 发行版在不使用时总是处于 Stopped 状态：

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-34-41.png" alt="Snipaste_2024-03-12_18-34-41" style="width:80%;" /></div>

如果目标发行版处于 Stopped 状态，那么访问该发行版的系统文件时，WSL 需先将该发行版启动，随后 Windows 系统才能到发行版文件，这显然需要耗费一定的时间。

最为直观的体验是：如果目标 Linux 发行版尚未启动，那么直接访问目标 Linux 文件时就会出现卡顿的情况。

### 自启 Linux 发行版

WSL 实则并未提供任何让 Linux 发行版随开机启动的放，但可以利用 Windows 系统访问 Linux 系统文件的特点，间接让指定的 Linux 发行版随开机启动。

将访问指定 Linux 发行版系统文件的链接，映射为网络驱动器：

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-35-22.png" alt="Snipaste_2024-03-12_18-35-22" style="width:50%;" /></div>

添加完毕后，电脑中会出现该 Linux 发行版的网络位置：

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-36-07.png" alt="Snipaste_2024-03-12_18-36-07" style="width:80%;" /></div>

使用命令行查看所有已安装 Linux 发行版的状态：

```shell
wsl --list --verbose
```

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-36-37.png" alt="Snipaste_2024-03-12_18-36-37" style="width:80%;" /></div>

将看到 Ubuntu-22.04 发行版会一直处于 Running 状态。

由于重启计算机后，网络位置也会重新连接，则对应的 Linux 发行版需要启动后网络位置才能完成重新连接，自此间接达成了自启 Linux 发行版的目的。

### 修改 root 密码

默认情况下，安装 Linux 发行版（Ubuntu 系统）后，该系统内默认的 root 密码是随机的，这意味着每次开机都会有一个新的 root 密码。为便于使用，推荐修改默认 Linux 发行版的 root 用户密码为固定密码。

进入 Linux Bash 中使用以下命令修改当前用于的密码：

```bash
sudo passwd
```

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-37-04.png" alt="Snipaste_2024-03-12_18-37-04" style="width:80%;" /></div>

修改完毕后，该密码即会作为 root 用户的新密码。

将用户切换为 root 使用以下命令：

```bash
su root
```

显示当前用户名称，使用以下命令：

```bash
whoami
```

测试 root 用户密码的可用性：

<div align="center"><img src="images/Windows%20Subsystem%20for%20Linux.images/Snipaste_2024-03-12_18-37-38.png" alt="Snipaste_2024-03-12_18-37-38" style="width:80%;" /></div>
