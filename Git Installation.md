### 概述

记录 Windows 下 Git 的安装选项，以保持软件的一致性，避免出现和 Git 相关的程序错误。某些配置后续无法通过修改 Git 的配置文件而改变，这些选项只能在 Git 安装时配置，因此在安装的时候务必谨慎对待。

### 版本环境

- Windows 11 23H2 22631.3007
- Git 2.40.0 64bit

### 重要配置

默认配置下 Git 会在拉取仓库或提交内容的时候，自动根据系统类型执行行尾符（换行符）的替换。例如在 Windows 系统上提交一个文件时，Git 会自动将文件内所有的 LF 替换为 Windows 系统支持的行尾符（换行符）CRLF。

实际这并不是一种很好的情况，因为大多数 Windows 系统下的代码编辑器本身是支持 LF 换行符的。为了避免不必要的错误，这里建议在 Windows 系统下使用 Git 工具时禁用换行符自动转换功能：

```bash
git config --global core.autocrlf false
```

如果使用的是 Linux 系统，则不建议禁用该功能，因为大概率是不会产生影响的。

### 安装过程

其中 Windows Explorer Integration 决定了 Git 是否在 Windows 的右键菜单中集成 Git 选项：

<div align="center"><img src="images/Git%20Installation.images/image-20240202203353237.png" alt="image-20240202203353237" style="width:45%;" /></div>

如果有 VSCode 存在，那么选择使用 VSCode 作为默认编辑器，比较方便：

<div align="center"><img src="images/Git%20Installation.images/image-20240202203419513.png" alt="image-20240202203419513" style="width:45%;" /></div>

默认分支的默认命名：

<div align="center"><img src="images/Git%20Installation.images/image-20240202203450185.png" alt="image-20240202203450185" style="width:45%;" /></div>

选择推荐项（不重要）：

<div align="center"><img src="images/Git%20Installation.images/image-20240202203533660.png" alt="image-20240202203533660" style="width:45%;" /></div>

一般来说系统都具备 SSH Client，选择第二项 External OpenSSH：

<div align="center"><img src="images/Git%20Installation.images/image-20240202203552752.png" alt="image-20240202203552752" style="width:45%;" /></div>

选择第二项（不重要）：

<div align="center"><img src="images/Git%20Installation.images/image-20240202203712425.png" alt="image-20240202203712425" style="width:45%;" /></div>

默认第一项（不重要）：

<div align="center"><img src="images/Git%20Installation.images/image-20240202203750470.png" alt="image-20240202203750470" style="width:45%;" /></div>

以免出现不必要错误，推荐选择 Use Windows‘s default console window 项：

<div align="center"><img src="images/Git%20Installation.images/image-20240202220201852.png" alt="image-20240202220201852" style="width:45%;" /></div>

本项用于设置 git pull 命令的默认行为：

<div align="center"><img src="images/Git%20Installation.images/image-20240202203824715.png" alt="image-20240202203824715" style="width:45%;" /></div>

采用 SSH 连接则不推荐使用 Credential Manager：

<div align="center"><img src="images/Git%20Installation.images/image-20240202203848642.png" alt="image-20240202203848642" style="width:45%;" /></div>

默认第一项（不重要）：

<div align="center"><img src="images/Git%20Installation.images/image-20240202203906009.png" alt="image-20240202203906009" style="width:45%;" /></div>

实验性功能可以不勾选：

<div align="center"><img src="images/Git%20Installation.images/image-20240202203919025.png" alt="image-20240202203919025" style="width:45%;" /></div>

### 严重错误

经实测，如果将 MinTTY 作为 Git Bash 的终端模拟器（Terminal Emulator），那么可能会造成 Windows OpenSSH 在 Git Bash 中出现严重的错误。

<div align="center"><img src="images/Git%20Installation.images/image-20240202203808739.png" alt="image-20240202203808739" style="width:45%;" /></div>

使用 SSH Client 连接到远程主机时，远程主机的公钥信息会被存储到本地的 known_hosts 文件中。这个操作通常发生在以下情况：

1. 首次连接到远程主机：当使用 SSH 首次连接到一个远程主机时，SSH Client 会显示远程主机的公钥信息，并询问是否愿意将其添加到 known_hosts 文件中。如果允许，则远程主机的公钥将被保存在 known_hosts 文件中，以便将来进行连接时完成验证；
2. 重新安装 SSH Server 或更改 SSH 密钥：如果远程主机进行了重新安装或更改了 SSH 密钥，下一次连接到该主机时，SSH Client 会重新显示远程主机的新公钥，并询问是否愿意更新 known_hosts 文件以匹配新的密钥；
3. 手动添加到 known_hosts 文件：有时，用户可能希望手动编辑 known_hosts 文件，并添加已知的主机和相应的公钥信息。这通常发生在对特定主机进行信任的情况下，或者为了绕过首次连接时的手动确认。

总之 known_hosts 文件的目的是确保连接到的远程主机的身份是可验证的，以防止中间人攻击。当与远程主机建立信任关系时，远程主机的公钥将会添加到 known_hosts 文件中，以后的连接就可以通过比对公钥来进行验证。

如果拥有一个本地仓库且未与远程仓库服务器建立过任何 SSH 连接时，首次尝试使用 Git Bash 进行 pull 或 push 操作，那么命令可能会卡住且无法完成，MinTTY 终端表现为无法使用 Ctrl + C 终止命令。

确保 GitHub 可连通的情况下，以下命令用于测试是否能够成功连接 GitHub 并验证 SSH 密钥是否正常工作：

```
ssh -T git@github.com
```

执行命令后，Git Bash 将表现为卡住的状态，命令既无法顺利完成，同时也无法终止命令的执行：

<div align="center"><img src="images/Git%20Installation.images/image-20240202215440816.png" alt="image-20240202215440816" style="width:60%;" /></div>

强行关闭 Git Bash 窗口时会出现以下提示：

<div align="center"><img src="images/Git%20Installation.images/image-20240202215544837.png" alt="image-20240202215544837" style="width:30%;" /></div>

从提示窗口的标题可大概率推断是 MinTTY 导致的某些错误。实际上，如果目标远程主机的公钥信息已经背添加到了 known_hosts 中，那么上述命令是可以顺利完成的。

由此可以大概推测出问题所在：

- MinTTY 调用 OpenSSH 程序时无法为 known_hosts 文件添加新的服务器公钥

**但实际该问题在此前任何版本的 Git 中均不存在，因此出现这种问题也可能是 Windows 系统版本和 MinTTY 版本之间存在某种冲突。**

为了避免不必要的麻烦，建议在 Windows 系统下，将终端模拟器设置为 Windows 默认的终端：

<div align="center"><img src="images/Git%20Installation.images/image-20240202220201852.png" alt="image-20240202220201852" style="width:45%;" /></div>

替换终端模拟器后，命令可以正常执行：

<div align="center"><img src="images/Git%20Installation.images/Snipaste_2024-03-14_13-18-50.png" alt="Snipaste_2024-03-14_13-18-50" style="width:80%;" /></div>

### 其他问题

如果在 Git Bash 执行 OpenSSH 命令但未能执行完毕时，可能会出现某些 OpenSSH 相关的错误。

<div align="center"><img src="images/Git%20Installation.images/Snipaste_2024-03-14_13-20-27.png" alt="Snipaste_2024-03-14_13-20-27" style="width:60%;" /></div>

类似上述这种情况，一般造成问题的原因是 OpenSSH 程序未能正确退出。系统进程中找到 ssh.exe 程序并终止它的运行，即可解决问题。
