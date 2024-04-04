### 概述

本篇记录 Ubuntu 22.04.2 桌面系统在 VMware Workstation 16 中的安装与配置过程。

### 版本环境

- VMware Workstation 16 Pro（16.2.3 build-19376536）
- Ubuntu 22.04.2（ubuntu-22.04.2-desktop-amd64）

### 注意事项

使用 VMware Workstation 安装虚拟机时，有几点注意事项：

- 新建虚拟机向导中，安装客户机操作系统推荐选择“稍后安装操作系统”，后续在虚拟机设置中可以手动添加安装程序的光盘映像 .iso 文件；
- 新建虚拟机向导中，指定磁盘容量时选择“将虚拟磁盘拆分成多个文件”，这样方便虚拟机文件的移动和保存；

另外，虚拟机设置中网络适配器默认为 NAT 模式。但如果本地网络解构并不特殊，比较推荐将网络适配器设置为桥接模式（BRIDGE）。

唯独仅主机模式（Host-Only）不推荐使用。因为在“仅主机模式”下，虚拟机只能和主机通信，无法访问外界网络。

### 安装过程

对于 Ubuntu 22.04.2 来说，安装它的桌面版系统并不复杂，安装过程中基本一直点击确认即可完成系统安装。

除了期间需要设置用户名密码外，没什么特别需要留意的地方。

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-12-49.png" alt="Snipaste_2024-03-12_17-12-49" style="width:50%;"/></div>

推荐设置：

- 选择全英系统。系统必然不是主用系统，为了能在使用终端时更加地便捷，推荐使用全英显示（没有人希望在终端的路径中，掺杂地键入几个中文字符吧）；
- 确认位置时，点击地图或手动输入以选择 Shanghai（GMT+8）时区。这关乎于系统时间，且系统时间对于某些软件来说十分重要，例如浏览器，时间稍有偏差都会导致网络无法访问。

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-14-45.png" alt="Snipaste_2024-03-12_17-14-45" style="width:50%;"/></div>

### 系统配置

该部分仅针对的 Ubuntu 22.04.2 桌面系统，主要包括系统更新、密码变更、显示设置、输入程序及字体安装等比较基础，但却重要的配置。

#### 安装更新

首次进入 Ubuntu 系统时，如果没有网络故障，那么软件更新器会自动弹出询问是否更新：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-15-48.png" alt="Snipaste_2024-03-12_17-15-48" style="width:33%;"/></div>

Software Updater（以前名为 Update Manager）是一个可选的应用程序，它用于升级所有已知来源的 DEB 包，同时还包含内核更新，推荐安装。

如果在此前的安装过程中已经将定位选择在“Shanghai”，那么软件更新器会自动将软件更新源设置为中国服务器，这意味着可以在上述提示窗口出现时，可以直接选择“现在安装”。

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-17-16.png" alt="Snipaste_2024-03-12_17-17-16" style="width:50%;"/></div>

但如果没有定位在中国，则需要先更改位置信息以确保系统时间是正确的。随后在软件更新器中把中国服务器设置为下载服务器，例如将 `mirrors.aliyun.com` 设置为下载服务器：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-18-54.png" alt="Snipaste_2024-03-12_17-18-54" style="width:50%;"/></div>

选择好下载服务器后，点击 Close 保存设置。后续打开 Software Updater 软件执行更新即可。

#### 密码变更

Ubuntu 22.04.2 系统的安装过程中，并不要求设置 root 账户密码。默认情况下，系统 root 账户的密码是随机的，这意味着每次重启系统后，该密码都会被重置。

如果需要登录并使用 root 账户，则仅需调用一次修改密码的命令来修改当前普通用户的密码：

```bash
sudo passwd
```

完成修改后，新密码将同时作为 root 账户的密码，且 root 账户密码将不再随机生成。

如果新密码长度短于 8 位，终端会出现提示信息：

```
BAD PASSWORD: The password is shorter than 8 characters
```

但这并不表示密码不能使用，继续重复输入密码即可完成修改。

#### 输入程序

如果安装时选择了英文系统，那么默认不存在中文输入法（或存在但无用）。

启用中文输入法，需要进入 Region & Language 设置中，选择 Manage Installed Languages。首次进入管理已安装的语言时，会自动出现“受支持语言尚未完整安装”的提示：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-20-03.png" alt="Snipaste_2024-03-12_17-20-03" style="width:50%;"/></div>

点击 Install 等待语言支持程序执行完毕，之后重启系统。

注销/重启系统后，在设置中找到 Keyboard 选项，找到 Input Sources 添加其他的输入源：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-21-14.png" alt="Snipaste_2024-03-12_17-21-14" style="width:50%;"/></div>

选择 Chinese 中的 Chinese (Intelligent Pinyin) 选项：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-22-13.png" alt="Snipaste_2024-03-12_17-22-13" style="width:50%;"/></div>

之后将默认的 Hanyu Pinyin (with AltGr dead keys)/English (US) 或其他首选输入源删除：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-22-58.png" alt="Snipaste_2024-03-12_17-22-58" style="width:50%;"/></div>

注销/重启系统后，中文输入法会默认启用。

#### 远程连接

这里的远程连接指使用 Xshell 之类终端工具连接 Linux 系统。具有图形化界面的 Ubuntu 系统默认只安装 openssh-client 程序：

```
~]# sudo apt list --installed | grep openssh
openssh-client/jammy-updates,now 1:8.9p1-3ubuntu0.1 amd64 [installed,automatic]
```

这意味着 Ubuntu 系统不能作为 SSH 的连接对象，因为系统中未安装 openssh-server 软件。

APT 源中一般存在 openssh-server 程序的 `.deb` 软件包，以下命令可以查阅：

```
~]# sudo apt list | grep openssh-server
openssh-server/jammy-updates 1:8.9p1-3ubuntu0.1 amd64
openssh-server/jammy-updates 1:8.9p1-3ubuntu0.1 i386
```

- `amd64` 表示 64 位版本；
- `i386` 表示 32 位版本。

以下命令用于安装 openssh-server 程序：

```bash
sudo apt install -y openssh-server
```

安装完毕后，由于默认情况下 Ubuntu 系统不启用防火墙，这意味着现在就可以直接使用 Xshell 等终端工具或者直接使用 Windows Terminal 连接至 Ubuntu 系统了。

使用 netstat 工具可以查看 22 端口的占用情况：

```
~]# sudo netstat -lnpt | grep 22
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      5180/sshd: /usr/sbi 
tcp6       0      0 :::22                   :::*                    LISTEN      5180/sshd: /usr/sbi 
```

或者 lsof 工具：

```
~]# sudo lsof -i :22
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    5180 root    3u  IPv4  58509      0t0  TCP *:ssh (LISTEN)
sshd    5180 root    4u  IPv6  58520      0t0  TCP *:ssh (LISTEN)
```

#### 端口放行

由于 Ubuntu 系统默认不开启防火墙，因此 SSH 才得以连接。

以下命令用于查看当前系统的防火墙状态：

```bash
sudo ufw status verbose
```

防火墙处于关闭状态时：

```bash
~]# sudo ufw status verbose
Status: inactive
```

防火墙处于开启状态时：

```bash
~]# sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip
```

如果希望保持防火墙开启的同时允许 SSH 连接，那么可以简单地选择放行 22 端口：

```bash
sudo ufw allow 22
```

放行成功后再次查看防火墙状态：

```bash
~]# sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22                         ALLOW IN    Anywhere                  
22 (v6)                    ALLOW IN    Anywhere (v6) 
```

**以上关于放行 22 端口的防火墙配置即时且永久生效，配置不会随着系统重启而丢失。**

关于 `ufw` 命令的更多操作，可查阅命令帮助：

```bash
sudo ufw --help
```

### 软件管理

本小节介绍一些 Ubuntu 22.04.2 系统中基础的、用于管理软件的命令。

#### 软件格式

Ubuntu 默认支持的软件包格式有两种：

- `.deb` 格式的软件包
- `.snap` 格式的软件包。

安装部署这些软件包的程序，称为**软件包管理系统（Package Manager）**。

软件包管理系统是在电脑中自动安装、配置、卸载和升级软件包的工具组合，在各种系统软件和应用软件的安装管理中均有广泛应用。

Ubuntu 中管理 deb 软件包的有 dpkg 软件包管理系统及其前端 APT 软件包管理系统，所谓“前端”可以理解为更高级的意思；其次管理 sanp 软件包的是 Snappy 软件包管理系统，不常使用。

这里需要了解一个关于 `.deb` 格式软件包的知识点：每个 `.deb` 格式软件包都具有唯一的 PACKAGE 属性，其对应的值为该 DEB 软件包的包名。

以 Microsoft Edge 浏览器的 `.deb` 格式软件包为例：

- `microsoft-edge-stable_112.0.1722.58-1_amd64.deb`

以下命令用于查看软件包详情：

```bash
~]# sudo dpkg -I microsoft-edge-stable_112.0.1722.58-1_amd64.deb
new Debian package, version 2.0.
 size 142150282 bytes: control archive=6472 bytes.
    1157 bytes,    12 lines      control              
   13128 bytes,   411 lines   *  postinst             #!/bin/sh
    8283 bytes,   269 lines   *  postrm               #!/bin/sh
    1354 bytes,    42 lines   *  prerm                #!/bin/sh
 Package: microsoft-edge-stable
 Version: 112.0.1722.58-1
 Architecture: amd64
 Maintainer: Microsoft Edge for Linux Team <EdgeLinuxDev@microsoft.com>
 Installed-Size: 493101
 Pre-Depends: dpkg (>= 1.14.0)
 Depends: ca-certificates, fonts-liberation, libasound2 (>= 1.0.17), libatk-bridge2.0-0 (>= 2.5.3), libatk1.0-0 (>= 2.2.0), libatspi2.0-0 (>= 2.9.90), libc6 (>= 2.17), libcairo2 (>= 1.6.0), libcups2 (>= 1.6.0), libcurl3-gnutls | libcurl3-nss | libcurl4 | libcurl3, libdbus-1-3 (>= 1.9.14), libdrm2 (>= 2.4.75), libexpat1 (>= 2.0.1), libgbm1 (>= 17.1.0~rc2), libglib2.0-0 (>= 2.39.4), libgtk-3-0 (>= 3.9.10) | libgtk-4-1, libnspr4 (>= 2:4.9-2~), libnss3 (>= 2:3.31), libpango-1.0-0 (>= 1.14.0), libu2f-udev, libuuid1 (>= 2.16), libvulkan1, libx11-6 (>= 2:1.4.99.1), libxcb1 (>= 1.9.2), libxcomposite1 (>= 1:0.4.4-1), libxdamage1 (>= 1:1.1), libxext6, libxfixes3, libxkbcommon0 (>= 0.5.0), libxrandr2, wget, xdg-utils (>= 1.0.2)
 Provides: www-browser
 Section: web
 Priority: optional
 Description: The web browser from Microsoft
  Microsoft Edge is a browser that combines a minimal design with sophisticated technology to make the web faster, safer, and easier.
```

从输出信息中可以看到软件包的 PACKAGE 属性，及其对应的属性值：

- micorsoft-edge-stable

Ubuntu 多数情况下会选择 APT 或 dpkg 软件包管理系统来管理软件，因为大多数软件仅提供 `.deb` 格式的软件包。虽然它们都用于管理 `.deb` 格式软件包，但其实际的管理方式有所不同。

APT 软件包管理系统依赖于**源**，可以把**源**理解为当前所有可供使用的 `.deb` 格式软件包合集的远程仓库，APT 会在本地维护一份该远程仓库中所有软件包的 PACKAGE 属性值列表。

用户使用 APT 安装软件时，只需要提供软件的 PACKAGE 属性值，APT 会在本地属性值列表中查询该值，如果存在对应记录，则从远程仓库（**源**）中下载对应的 `.deb` 格式软件包执行安装。

APT 本质上是 dpkg 的前端，简单来说就是 APT 提供比 dpkg 更多且更高级的功能，正因如此 dpkg 会显得比较“原始且古老”。

“原始” 的 dpkg 系统并不依赖于软件包的 PACKAGE 属性值，或者说它不具备检索 PACKAGE 属性值，dpkg 总是直接管理 `.deb` 格式的软件包。

#### 安装软件

出于管理软件包形式的差别，APT 安装软件时所需提供参数与 dpkg 安装软件时所需提供参数也有所差别：

- ${deb_package_name}：PACKAGE 属性值
- ${deb_package_fileName}：目标 `.deb` 格式软件包的文件名

使用 APT 安装软件的命令为：

```bash
sudo apt install ${deb_package_name}
```

使用 dpkg 安装软件的命令为：

```bash
sudo dpkg -i ${deb_package_fileName}
```

以 Microsoft Edge 浏览器为例：

- `microsoft-edge-stable_112.0.1722.58-1_amd64.deb`

假设 APT 源中存在 PACKAGE 属性值为 micorsoft-edge-stable 的 `.deb` 格式软件包，那么 APT 和 dpkg 安装该软件的命令分别如下：

```bash
# apt install
sudo apt install micorsoft-edge-stable

# dpkg install
sudo dpkg -i microsoft-edge-stable_112.0.1722.58-1_amd64.deb
```

另外，还有一种**不推荐**的安装方式。因为 APT 本质上是 dpkg 的前端，它实际上也支持直接管理 `.deb` 包：

```bash
sudo apt install ${deb_package_fileName}
```

但执行该命令很可能出现访问权限等问题，例如 Permission denied，后续将说明此问题。

#### 软件依赖

任何程序都逃不开依赖问题，类比 Java 中 JAR 包之间存在依赖关系，Linux 中的软件之间同样存在依赖关系。例如，软件 A 依赖于软件 B，那么安装软件 A 的时候，就必须保证系统上已经安装了软件 B。

软件之间的依赖关系，在使用 APT 软件包管理系统时并不明显，这是因为 APT 能够自动解决软件依赖缺失的问题。类比 Java 项目中使用项目管理工具 Maven 能将关联 JAR 包依赖自动引入项目一般，APT 也会在软件存在依赖缺失时，自动将缺失的依赖安装到系统中。

但 dpkg 管理系统不具备解决依赖缺失的能力，如果使用 dpkg 安装的软件包存在依赖缺失的情况，那么执行命令时会提示依赖缺失并将进度回滚，缺失的依赖仅会以列表的形式输出在终端上。

在 dpkg 出现依赖缺失并回滚进度后，可以借助 APT 来解决这些依赖缺失的问题：

```bash
sudo apt -f install -y
```

该命令仅用于安装缺失的依赖，后续仍旧需要再次使用 dpkg 来重新安装目标软件。

#### 权限问题

使用 APT 命令直接管理 `.deb` 软件包时，可能出现权限问题。以安装 Microsoft Edge 浏览器为例：

```bash
sudo apt install microsoft-edge-stable_112.0.1722.58-1_amd64.deb
```

因为 Microsoft Edge 浏览器所需的依赖在当前系统中都已具备，所以软件大概率能够成功安装。但终端大概率仍会出现以下的提示信息：

```
N: Download is performed unsandboxed as root as file '/home/dylan/microsoft-edge-stable_112.0.1722.58-1_amd64.deb' couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)
```

这通常是访问权限不足导致的。

使用桌面版 Ubuntu 系统时，下载好的 `.deb` 格式软件包通常位于 `/home/dylan/Download` 目录内，以上用例的软件包虽然挪动到了 `/home/dylan` 目录内，但不影响提示信息的复现。

使用 APT 安装软件时，它会使用一个内建账户 `_apt` 来完成软件依赖的下载，这个 `_apt` 用户于当前用户来说属于“其它用户（Other Users）”。

默认情况下，当前用户的 `~` 目录并不对 Other Users 开放 `read` 权限：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-24-11.png" alt="Snipaste_2024-03-12_17-24-11" style="width:80%;"/></div>

因此直接使用 APT 命令安装位于 `~` 目录下的 `.deb` 软件包时，大概率会出现“拒绝访问”的错误。

有两种方法可以消除这个提示：

1. 将 `.deb` 包挪动到 Other User 可访问的目录中，再执行安装；
2. 更改 `~` 目录关于 Other User 的访问权限。

**这里推荐使用第一种方法。**如果需要更改 `~` 目录的访问权限，可以使用以下命令：

```bash
sudo chmod o+r /home/dylan
```

该命令会为所有 Other User 赋予目标目录的 `read` 权限：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-25-07.png" alt="Snipaste_2024-03-12_17-25-07" style="width:80%;"/></div>

但一般不推荐以更改 `~` 目录访问权限的方式，去解决 APT 安装软件时出现的提示。相反，直接将软件包置于可被 Other User 访问的目录下会更好，这样可以避免目录权限的紊乱。

#### 卸载软件

不论使用的是 APT 管理系统还是 dpkg 管理系统，卸载软件时使用的参数均为软件的 PACKAGE 属性值。

使用 APT 卸载软件的命令为：

```bash
sudo apt remove ${deb_package_name}
```

使用 dpkg 卸载软件的命令为：

```bash
sudo dpkg -r ${deb_package_name}
```

推荐使用 APT 来完成软件的卸载，因为多数情况下 dpkg 提供的功能并不完善。注意，以上命令均不会将其他无用的依赖一并移除。

如果希望卸载软件时能一并将无用的依赖移除，可以使用以下命令：

```bash
sudo apt autoremove ${deb_package_name}
```

在不添加任何参数时，该命令可以用于移除系统中的无用依赖：

```bash
sudo apt autoremove
```

快捷查看当前系统中所有的无用依赖：

```
sudo apt autoremove --assume-no
```

#### 使用建议

关于软件安装：

- 如果源中不存在某款软件，但本地拥有 `.deb` 格式软件包，则建议使用 dpkg 直接安装，并配合 APT 命令解决可能存在的依赖问题；
- 如果源中具备该目标软件，则建议直接使用 APT 进行安装。

关于软件卸载，建议统一使用 APT 软件管理系统完成卸载操作：

- 移除软件的过程中，APT 提供了移除无用依赖的功能，能够更加深度地将无用数据清除。

#### 查找软件

以下命令用于查看所有使用 APT 或 dpkg 管理的、已安装或未安装的软件列表：

```bash
sudo apt list
```

该命令通常会配合 `grep` 命令一起使用，用于查找目标软件是否存在于源或系统中。例如：

```bash
~]# sudo apt list | grep microsoft-edge
microsoft-edge-beta/stable 113.0.1774.35-1 amd64
microsoft-edge-dev/stable 114.0.1823.7-1 amd64
microsoft-edge-stable/stable,now 113.0.1774.35-1 amd64 [installed]
```

其中 `[installed]` 表示已安装软件，其余则为“存在于源中且未安装的软件”。

除此之外，`list` 参数还支持“精确查找”：

```bash
~]# sudo apt list vim
vim/jammy-updates,jammy-security 2:8.2.3995-1ubuntu2.7 amd64
vim/jammy-updates,jammy-security 2:8.2.3995-1ubuntu2.7 i386
```

但该方式只适用于“全词匹配”。例如上述例子是精确查找属性值为“vim”的软件包，若将命令更改为：

```bash
sudo apt list vi
```

则不会得到任何的搜索结果。因为该命令不支持“模糊查找”，同时源中也不存在 PACKAGE 属性值为“vi”的软件包。

如果只想获取所有已安装的软件列表，可以加上 `--installed` 参数：

```bash
sudo apt list --installed
```

注意，`list` 参数的输出结果一般不利于阅读，建议配合 `more`、`less` 或 `most` 等阅读工具一起使用，或使用 `grep` 命令对输出结果进行筛选。

图形化界面的 Ubuntu 22.04.2 系统中，一般预装了 Firefox 浏览器。尝试使用终端查询时：

```bash
sudo apt list -i | grep firefox
```

会发现没有输出任何结果。

这是因为 Firefox 浏览器并不由 APT 或 dpkg 系统管理，该浏览器实际由 snap 系统管理。想要查看所有由 snap 系统管理的软件，可以使用以下命令：

```bash
~]# sudo snap list
Name                       Version           Rev    Tracking         Publisher   Notes
bare                       1.0               5      latest/stable    canonical✓  base
core20                     20230308          1852   latest/stable    canonical✓  base
core22                     20230325          607    latest/stable    canonical✓  base
firefox                    110.0-3           2356   latest/stable/…  mozilla✓    -
gnome-3-38-2004            0+git.6f39565     137    latest/stable/…  canonical✓  -
gnome-42-2204              0+git.e7d97c7     87     latest/stable    canonical✓  -
gtk-common-themes          0.1-81-g442e511   1535   latest/stable/…  canonical✓  -
snap-store                 41.3-66-gfe1e325  638    latest/stable/…  canonical✓  -
snapd                      2.58.2            18357  latest/stable    canonical✓  snapd
snapd-desktop-integration  0.1               57     latest/stable/…  canonical✓  -
```

可以看到 Firefox 浏览器存在于列表中。更多关于 snap 命令的使用方式，请查阅帮助：

```bash
sudo snap help
```

#### 更新升级

本地维护的软件源列表一般需要手动更新，以下命令用于更新本地软件源数据库（列表）：

```bash
sudo apt update
```

以下命令用于升级系统中所有已安装软件的版本：

```bash
sudo apt upgrade
```

如果希望查看所有已安装的、可升级的软件，使用以下命令：

```bash
sudo apt list --upgradable
```

#### 软件详情

在[软件格式](#软件格式)小节中，提及了可以使用 dpkg 管理系统命令，查看 `.deb` 格式软件包的详情。

以 Microsoft Edge 浏览器为例：

- `microsoft-edge-stable_112.0.1722.58-1_amd64.deb`

查看软件详情的命令如下：

```bash
sudo dpkg -I microsoft-edge-stable_112.0.1722.58-1_amd64.deb
```

如果希望查看 APT 源中的软件详情，可以使用 `show` 命令，使用该命令需要提供软件的 PACKAGE 属性值。例如 `vim` 的 PACKAGE 属性值为“vim”，那么查看该软件包详情的命令为：

```bash
~]# sudo apt show vim
Package: vim
Version: 2:8.2.3995-1ubuntu2.7
Priority: optional
Section: editors
Origin: Ubuntu
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Original-Maintainer: Debian Vim Maintainers <team+vim@tracker.debian.org>
Bugs: https://bugs.launchpad.net/ubuntu/+filebug
Installed-Size: 4,017 kB
Provides: editor
Depends: vim-common (= 2:8.2.3995-1ubuntu2.7), vim-runtime (= 2:8.2.3995-1ubuntu2.7), libacl1 (>= 2.2.23), libc6 (>= 2.34), libgpm2 (>= 1.20.7), libpython3.10 (>= 3.10.0), libselinux1 (>= 3.1~), libsodium23 (>= 1.0.14), libtinfo6 (>= 6)
Suggests: ctags, vim-doc, vim-scripts
Homepage: https://www.vim.org/
Task: cloud-image, ubuntu-wsl, server, ubuntu-server-raspi, lubuntu-desktop
Download-Size: 1,728 kB
APT-Sources: http://mirrors.aliyun.com/ubuntu jammy-updates/main amd64 Packages
Description: Vi IMproved - enhanced vi editor
 Vim is an almost compatible version of the UNIX editor Vi.
 .
 Many new features have been added: multi level undo, syntax
 highlighting, command line history, on-line help, filename
 completion, block operations, folding, Unicode support, etc.
 .
 This package contains a version of vim compiled with a rather
 standard set of features.  This package does not provide a GUI
 version of Vim.  See the other vim-* packages if you need more
 (or less).
```

#### 安装位置

软件的安装位置通常由软件包制作者决定，软件安装后可查看软件的安装位置信息：

```bash
sudo dpkg -L ${deb_package_name}
```

上述命令会将所有与软件关联的目录列出来。

### 个性配置

以下是可选的配置。

#### 显示设置

Ubuntu 22.04.2 桌面系统中采用 GNOME 42.5 桌面环境，它默认使用 Wayland 通讯协议。

<div align="center"><img src="images/Ubuntu%2022.04.2.images/image-20230425031055906.png" alt="image-20230425031055906" style="width:40%;"/></div>

但目前 Ubuntu 22.04.2 上的 Wayland 协议存在一定的问题，具体表现在系统进行分辨率缩放时，如果缩放比例不为 100%，那么所有非原生的系统应用都会出现界面模糊的问题。

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-26-44.png" alt="Snipaste_2024-03-12_17-26-44" style="width:50%;"/></div>

如果有放大系统全局字体大小的需求，如放大系统窗口显示字体或终端显示字体，那么不建议在显示设置中直接改变缩放比率！

这里推荐使用辅助功能里提供的调整系统字体大小选项：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-27-24.png" alt="Snipaste_2024-03-12_17-27-24" style="width:50%;"/></div>

如果希望字体以自定义的比例放大，例如让字体放大 1.4 倍，可以直接在终端运行以下命令：

```bash
gsettings set org.gnome.desktop.interface text-scaling-factor 1.4
```

直接使用命令时，会同时开启放大字体的功能，免去打开设置等操作。

如果任务栏图标在字体改变大小之后显得过小，可以在外观（Appearance）中修改它的大小：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-27-53.png" alt="Snipaste_2024-03-12_17-27-53" style="width:50%;"/></div>

#### 字体安装

Ubuntu 22.04.2 支持 Windows 系统的字体格式，将 `.ttf` 字体文件放在 `~/home/.fonts` 目录下，系统将自动识别这些字体。

#### 代理服务

图形化界面的 Ubuntu 22.04.2 系统下，可以使用 [Clash for Windows](https://github.com/Fndroid/clash_for_windows_pkg) 软件开启代理服务，该软件提供的 TUN Mode 可以实现系统网络的全局代理。

TUN 模式依赖于劫持 53 端口的流量，但 Ubuntu 系统中的 53 端口默认被 `systemd-resolved` 服务占用，这种情况下不仅自定义的 DNS 服务器无法起作用，TUN 模式代理亦无法起作用。

可以通过命令行查看 53 端口的占用情况：

```bash
~]# sudo lsof -i :53
COMMAND    PID            USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd-r 2263 systemd-resolve   13u  IPv4  37999      0t0  UDP localhost:domain 
systemd-r 2263 systemd-resolve   14u  IPv4  38000      0t0  TCP localhost:domain (LISTEN)
```

可以看到该端口默认被 `systemd-resolved` 占用。

修改 `systemd-resolved` 的配置文件：

```bash
sudo vi /etc/systemd/resolved.conf
```

将 resolved.conf 中的 `DNSStubListener` 键值对注释去掉，并将值修改为 `no`：

```
DNSStubListener=no
```

注意，如果未在网络配置中设置自定义的 DNS 服务器，那么还需要在以上配置中添加 DNS 的信息。

保存配置后重启 `systemd-resolved` 服务：

```bash
sudo systemctl restart systemd-resolved.service
```

再次检查端口占用情况：

```bash
sudo lsof -i :53
```

能够发现终端没有输出，即 53 端口已不再被 `systemd-resolved` 服务占用。

Ubuntu 22.04.2 系统下建议使用 Clash for Windows 0.20.15 版本，该版本目前较为稳定。新版可能会存在某些不可预知的问题，但多数问题在 Clash for Windows 项目的 issues 里能找到解决方法。

Clash for Windows 适配 Linux 的 Releases 是一个普通的压缩包，解压后即可使用：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-28-34.png" alt="Snipaste_2024-03-12_17-28-34" style="width:80%;"/></div>

将压缩包下载后，可以右键解压到任意目录，或使用命令行解压：

```bash
sudo tar zxvf Clash.for.Windows-0.20.15-x64-linux.tar.gz
```

解压缩后，找到其中 cfw 程序，它即为 linux 下的“可执行程序”：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-29-18.png" alt="Snipaste_2024-03-12_17-29-18" style="width:50%;"/></div>

双击 cfw 即可打开 Clash for Windows 的 GUI 界面。

卸载只需要将相关的压缩包和解压缩得到的目录，从系统中移除

#### 快捷方式

使用过 Windows 系统的都知道在该系统下，创建某个可执行程序的快捷方式是十分简单的事情。但在 Linux 中，创建程序的快捷方式并没有那么简单。

Linux 中的快捷方式并不是一个链接，或者说在该系统下的快捷方式都是以配置文件的形式存在的，快捷方式文件需要以 `.desktop` 为后缀。

Ubuntu 启动菜单中的快捷方式文件，一般存放在 `/usr/share/application` 目录中：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-30-07.png" alt="Snipaste_2024-03-12_17-30-07" style="width:50%;"/></div>

如果需要在启动菜单中为某个程序添加快捷方式，则需要配置该程序的快捷方式文件。例如 Clash for Windows 程序的快捷方式文件（程序图标需自行添加到 `/lib/clash.for.windows` 目录中）：

```
[Desktop Entry]
Name=Clash for Windows
Exec="/lib/clash.for.windows/cfw"
Icon=/lib/clash.for.windows/logo_64.png
Terminal=false
Type=Application
```

配置完毕后，需要将文件命名为 `clash.desktop` 并保存到 `/usr/share/application` 目录中，启动菜单就会出现对应程序的快捷方式：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-31-18.png" alt="Snipaste_2024-03-12_17-31-18" style="width:80%;"/></div>

同理，将 `.desktop` 文件移动到桌面目录中，就能直接从桌面访问目标程序。

这里用 Microsoft Edge 浏览器作为例子，该浏览器的快捷方式文件一般会在安装程序时自动创建：

```bash
~]# sudo ls -l /usr/share/applications | grep microsoft
-rw-r--r-- 1 root root  8124 May  5 16:01 microsoft-edge.desktop
```

将该文件拷贝至桌面目录：

```bash
cp /usr/share/applications/microsoft-edge.desktop ~/Desktop
```

**注意，这里不能使用管理员权限拷贝文件，否则后续该快捷方式将不可用。**

桌面随即会出现 Microsoft Edge 浏览器的快捷方式，但一般处于不可用状态：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-30-58.png" alt="Snipaste_2024-03-12_17-30-58" style="width:80%;"/></div>

右键选择“允许运行”：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-31-05.png" alt="Snipaste_2024-03-12_17-31-05" style="width:80%;"/></div>

快捷方式即可使用：

<div align="center"><img src="images/Ubuntu%2022.04.2.images/Snipaste_2024-03-12_17-31-11.png" alt="Snipaste_2024-03-12_17-31-11" style="width:80%;"/></div>