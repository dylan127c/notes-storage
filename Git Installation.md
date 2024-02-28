### 概述

记录 Git 的安装选项，以保持软件的一致性，避免出现和 Git 相关的程序错误。某些配置后续无法通过修改 Git 的配置文件而改变，这些选项只能在 Git 安装时配置，因此在安装的时候务必谨慎对待。

### 版本环境

- Windows 11 23H2 22631.3007
- Git 2.40.0 64bit

### 安装过程

其中 Windows Explorer Integration 决定了 Git 是否在 Windows 的右键菜单中集成 Git 选项：

<img src="images/Git%20Installation.images/image-20240202203353237.png" alt="image-20240202203353237" style="zoom:67%;" />

如果有 VSCode 存在，那么选择使用 VSCode 作为默认编辑器，比较方便：

<img src="images/Git%20Installation.images/image-20240202203419513.png" alt="image-20240202203419513" style="zoom:67%;" />

默认分支的默认命名：

<img src="images/Git%20Installation.images/image-20240202203450185.png" alt="image-20240202203450185" style="zoom:67%;" />

选择推荐项（不重要）：

<img src="images/Git%20Installation.images/image-20240202203533660.png" alt="image-20240202203533660" style="zoom:67%;" />

关于 SSH 务必选择使用 External OpenSSH：

<img src="images/Git%20Installation.images/image-20240202203552752.png" alt="image-20240202203552752" style="zoom:67%;" />

选择第二项（不重要）：

<img src="images/Git%20Installation.images/image-20240202203712425.png" alt="image-20240202203712425" style="zoom:67%;" />

默认第一项（不重要）：

<img src="images/Git%20Installation.images/image-20240202203750470.png" alt="image-20240202203750470" style="zoom:67%;" />

避免出现错误，务必选择 Use Windows‘s default console window 项：

<img src="images/Git%20Installation.images/image-20240202220201852.png" alt="image-20240202220201852" style="zoom:67%;" />

本项用于设置 git pull 命令的默认行为：

<img src="images/Git%20Installation.images/image-20240202203824715.png" alt="image-20240202203824715" style="zoom:67%;" />

不推荐使用任何 Credential Manager：

<img src="images/Git%20Installation.images/image-20240202203848642.png" alt="image-20240202203848642" style="zoom:67%;" />

默认第一项（不重要）：

<img src="images/Git%20Installation.images/image-20240202203906009.png" alt="image-20240202203906009" style="zoom:67%;" />

实验性功能均不勾选：

<img src="images/Git%20Installation.images/image-20240202203919025.png" alt="image-20240202203919025" style="zoom:67%;" />

### 严重错误

经实测，如果将 MinTTY 作为 Git Bash 的终端模拟器（Terminal Emulator），那么可能会造成 Windows OpenSSH 在 Git Bash 中出现严重的错误。

<img src="images/Git%20Installation.images/image-20240202203808739.png" alt="image-20240202203808739" style="zoom:67%;" />

众所周知，使用 SSH 客户端连接到远程服务器时，该服务器的公钥信息将会被存储到本地的 known_hosts 文件中。这个操作通常发生在以下情况：

1. 首次连接到远程服务器：当使用 SSH 首次连接到一个远程服务器时，SSH 客户端会显示服务器的公钥信息，并询问是否愿意将其添加到 known_hosts 文件中。如果允许，则服务器的公钥将被保存在 known_hosts 文件中，以便将来进行连接时完成验证；
2. 重新安装服务器或更改 SSH 密钥：如果服务器重新安装或更改了 SSH 密钥，下一次连接到该服务器时，SSH 客户端会重新显示服务器的新公钥，并询问是否愿意更新 known_hosts 文件以匹配新的密钥；
3. 手动添加到 known_hosts 文件：有时，用户可能希望手动编辑 known_hosts 文件，并添加已知的主机和相应的公钥信息。这通常发生在对特定主机进行信任的情况下，或者为了绕过首次连接时的手动确认。

总的来说，known_hosts 文件的目的是确保连接到的远程服务器的身份是可验证的，以防止中间人攻击。当与远程服务器建立信任关系时，服务器的公钥将会添加到 known_hosts 文件中，以后的连接就可以通过比对公钥来进行验证。

如果拥有一个仓库且未与远程仓库建立过任何 SSH 连接时，此时尝试使用 Git Bash 进行 pull 或 push 操作，那么命令将卡住且无法完成，同时终端表现为无法使用 Ctrl + C 终止命令。

确保 GitHub 可连通的情况下，以下命令用于测试是否能够成功连接 GitHub 并验证 SSH 密钥是否正常工作：

```
ssh -T git@github.com
```

执行命令后，Git Bash 将表现为卡住的状态，命令既无法顺利完成，同时也无法终止命令的执行：

<img src="images/Git%20Installation.images/image-20240202215440816.png" alt="image-20240202215440816" style="zoom:67%;" />

强行关闭 Git Bash 窗口时会出现以下提示：

<img src="images/Git%20Installation.images/image-20240202215544837.png" alt="image-20240202215544837" style="zoom:67%;" />

可以看到大概率是因为 MinTTY 的原因，从而导致了这种严重错误。

如果目标服务器的公钥已经添加到了 known_hosts 中，那么关于该 host 的命令均能顺利执行，可以推测出问题所在：

- MinTTY 调用 OpenSSH 程序时无法为 known_hosts 文件添加新的服务器公钥

**但实际该问题在此前并不存在，推测是 Windows 版本和 MinTTY 之间存在某种冲突。**

建议在 Windows 系统下，将终端模拟器设置为 Windows 默认的终端：

<img src="images/Git%20Installation.images/image-20240202220201852.png" alt="image-20240202220201852" style="zoom:67%;" />

替换终端模拟器后，命令可以正常执行：

![image-20240202221027818](images/Git%20Installation.images/image-20240202221027818.png)

### 其他问题

如果在 Git Bash 执行 OpenSSH 命令但未能执行完毕时，可能会出现某些 OpenSSH 相关的错误。

例如：

<img src="images/Git%20Installation.images/Snipaste_2024-02-02_21-36-29.png" alt="Snipaste_2024-02-02_21-36-29" style="zoom:67%;" />

造成这种情况的原因是 OpenSSH 程序未能正确退出。系统进程中找到 ssh.exe 程序并终止它的运行，即可解决问题。
