### 概述与原理

本篇介绍一个用于美化终端的 GitHub 项目：[Oh My Posh](https://github.com/JanDeDobbeleer/oh-my-posh)（简称：OMP）。使用 OMP 主体程序加载其所提供的主题配置文件，即可打造一个更具可视化的、富有个性的终端工具。

一般终端启动时，都会加载一个默认的配置文件，除非配置文件不存在否则这个过程是必不可少的，启动的过程类似这样：

<img src="images/Oh%20My%20Posh.images/image-20230828034813880.png" alt="image-20230828034813880" style="zoom:67%;" />

OMP 的核心就是利用这个默认的配置文件来加载 OMP 的主题配置，那么主题样式会在终端启动时被加载，终端打开后就能看到已应用样式的终端界面：

<img src="images/Oh%20My%20Posh.images/image-20230828035001164.png" alt="image-20230828035001164" style="zoom:67%;" />

以上就是 OMP 程序让终端加载自定义主题样式的方法，但关于主题样式是如何替换掉原本终端样式的问题，本篇不作讨论。

WSL Bash 效果展示：

<img src="images/Oh%20My%20Posh.images/image-20230422060714595.png" alt="image-20230422060714595" style="zoom: 50%;" />

Git Bash 效果展示：

<img src="images/Oh%20My%20Posh.images/image-20230422162229845.png" alt="image-20230422162229845" style="zoom:50%;" />

### Powerline 和编程连字

一般来说，字体可以分为是否支持 Powerline 字形及是否支持编程连字。以 Cascadia Code 字体为例，它拥有多个版本可供使用：

| 字体名称         | 包括连字 | 包括 Powerline 字形 |
| :--------------- | :------- | :------------------ |
| Cascadia Code    | 是       | 否                  |
| Cascadia Mono    | 否       | 否                  |
| Cascadia Code PL | 是       | 是                  |
| Cascadia Mono PL | 否       | 是                  |

Windows 终端在其包中提供 Cascadia Code 和 Cascadia Mono，并默认使用 Cascadia Mono。

Powerline 本质一个常用的命令行插件，用于在提示中显示附加信息。简单来说，OMP 提供的主题样式中会包含一些图形信息，这些图形信息依赖 Powerline 显示。而编程连字是通过组合字符创建的字形，它们在编写代码时最有用。

<img src="images/Oh%20My%20Posh.images/programming-ligatures.gif" alt="Cascadia Code 编程连字" style="zoom: 67%;" />



以下终端里，紧跟在时间后面的终端图形便是依赖 Powerline 显示，而命令 `ll` 所展示的内容则使用到了编程连字。

<img src="images/Oh%20My%20Posh.images/image-20230422060714595.png" alt="image-20230422060714595" style="zoom: 50%;" />

之所以要介绍 Powerline 以及编程连字，是因为 OMP 提供的主题样式需要使用到支持 Powerline 以及编程连字的字体。换言之，终端必须先配置支持 Powerline 以及编程连字的字体后，再使用 OMP 加载的特定的主题样式，这样有助于主题样式更好地应用。

OMP 官方推荐使用 [Nerd Font](https://www.nerdfonts.com/font-downloads) 中提供的字体，这些字体均支持 Powerline 以及编程连字，且能更好地兼容 OMP 提供的主题样式。

推荐使用 Cascadia Code 字体的变体 Caskaydia Cove Nerd Font：

<img src="images/Oh%20My%20Posh.images/image-20230422061538249.png" alt="image-20230422061538249" style="zoom:50%;" />

下载解压得到 Caskaydia Cove 字体目录，进入目录后选中所有的 .otf 字体文件，右键选择“为所有用户安装”即可。

### 终端字体配置

字体安装完毕后，需要先更改终端的字体配置。以 Caskaydia Cove 字体为例：

<img src="images/Oh%20My%20Posh.images/image-20230422062931647.png" alt="image-20230422062931647" style="zoom:50%;" />

如果熟悉 JSON 格式，也可以选择右下角的“打开 JSON 文件”，这样可以直接对终端的样式进行编辑：

<img src="images/Oh%20My%20Posh.images/image-20230422164315102.png" alt="image-20230422164315102" style="zoom:50%;" />

### OMP 安装

OMP 支持 Windows、MacOS 及 Linux 等主流系统，详情参考：[Oh My Post - Installation](https://ohmyposh.dev/docs/installation/windows)。本篇仅介绍 Windows、Linux 两种系统下安装 OMP 的具体方式。

#### Windows

Windows 系统**首次**安装建议使用**安装包**。从 OMP 的 [Releases](https://github.com/JanDeDobbeleer/oh-my-posh/releases) 列表中选择合适的安装包进行安装（这是一种可控制安装位置、安装用户的方式）：

<img src="images/Oh%20My%20Posh.images/image-20230422164918880.png" alt="image-20230422164918880" style="zoom:50%;" />

OMP 官网推荐使用的是 winget 命令行安装（winget 是一款程序包管理器）：

<img src="images/Oh%20My%20Posh.images/image-20230422165055831.png" alt="image-20230422165055831" style="zoom:50%;" />

使用 winget 唯一的缺点是无法控制程序的安装位置，且它仅会为当前用户安装（不能选择“为所有用户安装”）。

但无论采取哪种方式，程序安装完毕后，OMP 的 bin 目录都会被自动配置至系统或用户的环境变量中，这意味着你可以任意位置开启终端访问 OMP 程序。安装完毕后，可以立即使用 Windows PowerShell 查看到 OMP 的版本信息：

```shell
oh-my-posh --version
```

<img src="images/Oh%20My%20Posh.images/image-20230422172051007.png" alt="image-20230422172051007" style="zoom:50%;" />

更多 Windows 下 OMP 的安装细则，可参考官方教程：[Oh My Posh - Windows](https://ohmyposh.dev/docs/installation/windows)。

#### Linux

Linux 系统下，推荐分开下载 OMP 程序本体和 OMP 主题包。

OMP 程序本体可以通过 wget 命令从 GitHub 获取，以下命令用于获取最新版本的 OMP 程序本体（**不推荐**）：

```bash
sudo wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/posh-linux-amd64 -O /usr/local/bin/oh-my-posh
```

**但自 18.0.0 版本起 OMP 有重大更新，为了更好地兼容现有的 OMP 主题，这里更推荐使用 17.12.1 版本：**

```bash
sudo wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/download/v17.12.1/posh-linux-amd64 -O /usr/local/bin/oh-my-posh
```

这是一种开箱即用的方式，它虽然仅下载 OMP 程序本体，但程序本体会直接被下载至用户的 bin 目录下，这等同于将 OMP 程序本体的 bin 目录添加到了用户的环境变量中。

程序本体下载完毕后，首次需要给 oh-my-posh 赋予执行权限：

```bash
sudo chmod +x /usr/local/bin/oh-my-posh
```

由于尚不存在 OMP 主题包，因此还需要继续下载，命令如下：

```bash
mkdir ~/.poshthemes
wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/themes.zip -O ~/.poshthemes/themes.zip
unzip ~/.poshthemes/themes.zip -d ~/.poshthemes
chmod u+rw ~/.poshthemes/*.omp.*
rm ~/.poshthemes/themes.zip
```

以上命令自上而下分别的作用是：1. 创建隐藏的主题目录；2. 下载主题压缩文件至主题；3. 解压主题压缩文件；4. 为主题文件的拥有者赋予读写的权限；5. 删除无用的压缩文件。

注意，不要批量运行以上命令。如果某个必要程序不存在，则该条命令的执行虽无效，但后续命令仍会执行。假设 unzip 尚未安装，那么以上命令仍旧会创建目录并下载压缩文件，但解压命令和赋权命令均会失败，又由于删除文件的命令存在且不受失败命令的影响，最终压缩文件仍会被删除。

安装完毕后，可以直接在 Bash 中查看到 OMP 的版本信息：

<img src="images/Oh%20My%20Posh.images/image-20230422172244590.png" alt="image-20230422172244590" style="zoom:50%;" />

更多 Linux 下 OMP 的安装细则，可参考官方教程：[Oh My Posh - Linux](https://ohmyposh.dev/docs/installation/linux)。

### OMP 应用

最后一步是使用 OMP 主体程序来加载主题样式，这个步骤需要修改 Shell 或 Bash 的默认加载配置。

以下仅介绍如何修改 PowerShell 和 Bash 的默认配置文件，如果需要了解其他终端如何修改，请参考：[Oh My Posh - Prompt](https://ohmyposh.dev/docs/installation/prompt)。

#### PowerShell

Windows PowerShell 可执行以下命令查看配置文件：

```shell
notepad.exe $PROFILE
```

如果当前无任何配置文件，那么一般会得到以下弹窗提示：

<img src="images/Oh%20My%20Posh.images/image-20230422172935703.png" alt="image-20230422172935703" style="zoom:50%;" />

这意味着 PowerShell 当前不存在任何配置文件，这时候需要创建一个。执行以下命令创建一个配置文件：

```shell
New-Item -Path $PROFILE -Type File -Force
```

<img src="images/Oh%20My%20Posh.images/image-20230422173107566.png" alt="image-20230422173107566" style="zoom:50%;" />

配置文件默认位于 Documents\WindowsPowerShell 目录下，文件一般会被命名为 Microsoft.PowerShell_profile.ps1。注意，该配置文件的位置不可更改，因此在完成配置后不建议去移动该文件。

使用 notepad 打开 Microsoft.PowerShell_profile.ps1 配置文件，并添加以下配置：

```shell
oh-my-posh init pwsh | Invoke-Expression
```

<img src="images/Oh%20My%20Posh.images/image-20230422173324498.png" alt="image-20230422173324498" style="zoom:50%;" />

保存退出，执行以下命令刷新配置：

```shell
. $PROFILE
```

再次打开 Windows PowerShell，可以看到默认的主题样式已生效：

<img src="images/Oh%20My%20Posh.images/image-20230422173549345.png" alt="image-20230422173549345" style="zoom:50%;" />

如果希望使用不同的主题，例如使用 emodipt-extend.omp.json 主题样式，则需要修改配置内容为：

```bash
oh-my-posh init pwsh --config 'E:\oh-my-posh\themes\emodipt-extend.omp.json' | Invoke-Expression
```

使用命令刷新配置并重新打开 PowerShell，指定的主题样式也会即时生效：

<img src="images/Oh%20My%20Posh.images/image-20230422174016910.png" alt="image-20230422174016910" style="zoom:50%;" />

#### Bash

无论是 Git Bash 还是 Linux Bash，它们启动时所加载的配置文件通常都位于 /etc 目录下，且一般被命名为 bash.bashrc。

以 Git Bash 为例，应用默认主题只需要在 /etc/bash.bashrc 中添加以下配置：

```bash
eval "$(oh-my-posh init bash)"
```

<img src="images/Oh%20My%20Posh.images/image-20230422174558843.png" alt="image-20230422174558843" style="zoom:50%;" />

重新打开 Git Bash 后，可以看到默认的主题样式已生效：

<img src="images/Oh%20My%20Posh.images/image-20230422174643078.png" alt="image-20230422174643078" style="zoom:50%;" />

同样，更换配置只需要添加 config 参数以指定主题样式文件的存储路径：

```bash
eval "$(oh-my-posh init bash --config /e/oh-my-posh/themes/emodipt-extend.omp.json)"
```

<img src="images/Oh%20My%20Posh.images/image-20230422174757360.png" alt="image-20230422174757360" style="zoom:50%;" />

使用 WSL 模拟 Linux 系统配置 OMP 情况，应用主题同样只需要在 /etc/bash.bashrc 中添加 OMP 配置：

```bash
sudo vim /etc/bash.bashrc
```

```
eval "$(oh-my-posh init bash --config ~/.poshthemes/emodipt-extend.omp.json)"
```

<img src="images/Oh%20My%20Posh.images/image-20230422175430673.png" alt="image-20230422175430673" style="zoom:50%;" />

重新打开 Linux Bash 后，可以看到指定的主题样式已生效：

<img src="images/Oh%20My%20Posh.images/image-20230422175517560.png" alt="image-20230422175517560" style="zoom:50%;" />

更多 OMP 主题样式的预览，请前往：[Oh My Posh - Themes](https://ohmyposh.dev/docs/themes)。

### OMP 更新

无论是 Windows 还是 Linux，更新 OMP 一般仅需要获取其最新的 OMP 主体程序，即仅更新 bin 目录下的 oh-my-posh 文件，该文件可以直接从 OMP 的 [Releases](https://github.com/JanDeDobbeleer/oh-my-posh/releases) 列表中找到。

Windows 系统下载对应版本的 posh-windows-amd64.exe 文件并重命名为 oh-my-posh.exe 拷贝至 bin 目录覆盖旧文件即可。一般来说 themes 主题文件不会有很大的改动，想更新主题的话可以下载 themes.zip 文件，同样解压并拷贝主题文件至 themes 目录覆盖旧文件即可。

<img src="images/Oh%20My%20Posh.images/image-20230827182415825.png" alt="image-20230827182415825" style="zoom: 50%;" />

Linux 系统使用以下命令覆盖 oh-my-posh 文件：

```bash
sudo wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/posh-linux-amd64 -O /usr/local/bin/oh-my-posh
```

但一般情况下，如果 OMP 能够正常使用，那么就尽量不要更新，因为更新可能带来兼容性问题。例如 OMP 最新的 18.0.0 版本，它无法兼容部分的主题文件，会导致主题样式出现未知错误，那么此时就不推荐升级版本。

或者可以选择升级至较为稳定的版本，例如 17.12.1 版本：

```bash
sudo wget https://github.com/JanDeDobbeleer/oh-my-posh/releases/download/v17.12.1/posh-linux-amd64 -O /usr/local/bin/oh-my-posh
```

