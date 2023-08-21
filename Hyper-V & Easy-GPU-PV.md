### 虚拟机

虚拟机（Virtual Machine）指通过软件模拟的具有完整硬件系统功能的、运行在一个完全隔离环境中的完整计算机系统。在实体计算机中能够完成的工作在虚拟机中都能够实现。

在计算机中创建虚拟机时，需要将实体机的部分硬盘和内存容量作为虚拟机的硬盘和内存容量。每个虚拟机都有独立的 CMOS、硬盘和操作系统，可以像使用实体机一样对虚拟机进行操作。

#### VMware Workstation

VMware Workstation（中文名“威睿工作站”）是一款功能强大的桌面虚拟计算机软件，提供用户可在单一的桌面上同时运行不同的操作系统，和进行开发、测试 、部署新的应用程序的最佳解决方案。

VMware Workstation 可在一部实体机器上模拟完整的网络环境，以及可便于携带的虚拟机器，其更好的灵活性与先进的技术胜过了市面上其他的虚拟计算机软件。对于企业的 IT 开发人员和系统管理员而言， VMware 在虚拟网路，实时快照，拖曳共享文件夹，支持 PXE 等方面的特点使它成为必不可少的工具。

#### Hyper-V

Hyper-V 是微软的一款虚拟化产品，是微软第一个采用类似 Vmware ESXi 和 Citrix Xen 的基于 hypervisor 的技术。这也意味着微软会更加直接地与市场先行者 VMware 展开竞争，但竞争的方式会有所不同。

Hyper-V 是微软提出的一种系统管理程序虚拟化技术，能够实现桌面虚拟化。Hyper-V 最初预定在 2008 年第一季度，与 Windows Server 2008 同时发布。Hyper-V Server 2012 完成 RTM 版发布。

### 虚拟化技术

一般的个人计算机都支持虚拟化技术，使用该技术需要先在 BIOS 主板中开启特定功能：

1. 支持 Intel 芯片组的主板，需要在 BIOS 中启用 Intel Virtualization Technology (VT-x）选项；
2. 支持 AMD 芯片组的主板，需要在 BIOS 中启用 SVM Mode 选项。

对于 VMware Workstation 来说，只要 BIOS 中启用了 VT-x 或 SVM Mode 后，无需额外的设置，安装后即可可使用。

对于 Hyper-V 来说，则还需要在 Windows 功能选项卡中启用 **Hyper-V** 及**虚拟机平台**等两项功能：

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220812161741134.png" alt="image-20220812161741134" style="zoom:67%;" />

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220812161638752.png" alt="image-20220812161638752" style="zoom:67%;" />



此外，还需要以管理员身份打开终端，将管理程序启动类型设置为 AUTO：

```bash
bcdedit /set hypervisorlaunchtype auto
```

### 兼容性问题

~~**VMware 和 Hyper-V 在技术层面上互不兼容**，如果 Windows 功能中启用了 Hyper-V 功能，则会导致 VMware 虚拟机启动失败。~~

~~如果你希望在系统上继续使用 VMware，需要关闭 Hyper-V 功能，同时将管理程序启动类型设置为 OFF：~~

```bash
bcdedit /set hypervisorlaunchtype off
```

自 VMware Workstation 15.5.5 版本之后，支持在开启 Hyper-V 模式下运行！

详情参考：[VMware Workstation 15.5 Now Supports Host Hyper-V Mode](https://blogs.vmware.com/workstation/2020/05/vmware-workstation-now-supports-hyper-v-mode.html)。

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20230423011137400.png" alt="image-20230423011137400" style="zoom:50%;" />

请注意，VMware 仍需要特定 Windows 版本的支持。如果不希望纠结于系统版本，则建议使用 Windows 11，该系统下完全支持 VMware 15.5.5 及后续版本的运行。

以本机 VMware Workstation 16.2.3 为例，已存在虚拟机的情况下，开启虚拟机需要变更某些设置：

- 去除勾选“**虚拟化 Intel VT-x/EPT 或 AMD-V/RVI(V)**”选项！

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20230423011513709.png" alt="image-20230423011513709" style="zoom:50%;" />

为了提高虚拟机性能，建议在开启 Hyper-V 模式下，禁用虚拟机设置中的侧通道缓解：

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20230423011701762.png" alt="image-20230423011701762" style="zoom:50%;" />

### 虚拟机显卡直通

虚拟机显卡直通（Virtual Machine GPU Passthrough）是一种将物理GPU（显卡）分配给虚拟机的技术，使得虚拟机可以直接访问物理显卡，获得近乎原生的显卡性能。

传统的虚拟机在运行时通常是使用宿主机的 CPU 和内存资源进行模拟运算，但在需要高性能显卡加速的场景下，由于虚拟机无法直接访问物理显卡，因此无法发挥显卡加速的优势。

虚拟机显卡直通技术则可以将物理显卡分配给虚拟机，使虚拟机可以直接访问物理显卡，从而获得接近原生的显卡性能，适用于需要进行 GPU 密集型计算、图形渲染、视频处理等场景。

### Parsec 远程控制

[Parsec](https://parsec.app/) 是一款基于云游戏技术的远程桌面软件，可以让用户通过互联网访问远程计算机上的桌面环境，并在本地实现低延迟、高清晰度的游戏、应用等体验。

使用 Parsec，用户可以将自己的游戏、应用程序等运行在远程计算机上，然后通过网络将计算机上的图像和声音传输到本地计算机上，实现类似于本地运行的效果。此外，Parsec 还支持云游戏平台，用户可以租用云上的计算资源，在云端运行游戏和应用程序，并通过网络将图像和声音传输到本地计算机上，实现游戏的流畅运行。

Parsec 采用了一系列优化技术，包括网络协议优化、视频编解码优化、音频编解码优化等，使得用户在使用 Parsec 时可以获得低延迟、高清晰度的远程体验。同时，Parsec 还支持多平台，包括 Windows、Mac、Linux、Android 等操作系统。

#### 传输原理

Parsec 的传输原理基于视频流和音频流的实时传输。在运行 Parsec 的远程计算机上，会截取计算机的屏幕和音频数据，并对其进行压缩和编码，然后将其转化为视频流和音频流，并通过网络发送到本地计算机。

在本地计算机上，Parsec 会解码视频流和音频流，并在本地计算机上进行渲染和播放，最终呈现给用户的是在本地计算机上的实时画面和声音。

为了保证传输质量和延迟，Parsec 使用了一系列优化技术，包括网络协议优化、视频编解码优化、音频编解码优化等。例如，Parsec 使用了自己的网络协议，支持 UDP 协议和 FEC（Forward Error Correction）纠错技术，可以最大程度地降低网络延迟和丢包率。

同时，Parsec 还采用了 H.265 视频编解码和 Opus 音频编解码等高效的编解码算法，可以在保证传输质量的情况下，尽可能地减小视频和音频流的带宽占用。

简单地说，Parsec 的远程控制更像是“远程播放”，这种借助物理 GPU 对桌面内容和音频进行编解码的远程访问效率十分地高。远程计算机的 GPU 性能如果足够强大，那么影响远程控制延迟的唯一因素就仅剩网络传输速率了。

#### 使用条件

据使用体验，使用 Parsec 实际需具备两个条件：1. 远程计算机的 GPU 性能需足够强大；2. 远程和本地计算机需要具备代理服务器。

基于它的传输原理，不难得知远程计算机的 GPU 需具备一定的性能，才能应付大量的编解码操作，这实际是使用 Parsec 的其中一个弊端。

另一个弊端是 Parsec 为国外推出的远程控制软件，它天生与国内网络相性不好。最简单的一点，使用 Parsec 必须登录个人账户，如果没有代理，软件可能无法使用，甚至于申请注册 Parsec 账号都会成为阻碍。

但实际代理网络只用在申请注册 Parsec 账户或登录 Parsec 账户时，Parsec 本身建立远程连接并不依赖于代理网络。

### Easy-GPU-PV

[Easy-GPU-PV](https://github.com/jamesstringerparsec/Easy-GPU-PV) 是 GitHub 上的一个开源项目，它是由远程控制软件 Parsec 的作者所编写的、用于一键部署能支持 GPU 直通的 Hyper-V 虚拟机的脚本。

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220804141457838.png" alt="image-20220804141457838" style="zoom:50%;" />

该项目建立在 Hyper-V 虚拟机能支持 GPU 直通的基础上，配合 Parsec 远程连接，让用户可以在不影响真实物理机的情况下，实现远程控制高性能虚拟机的目的。

Hyper-V 本身就支持 GPU 直通，但部署过程十分繁琐，涉及诸多的 Shell 命令，因此建议直接使用项目中提供的一键部署脚本。

使用一键部署脚本前，只需要修改部分参数即可，脚本运行后的数分钟内，支持 GUP 直通的 Hyper-V 虚拟机就能部署完成。

项目中仅有三个主要的脚本文件需要了解：

- PreChecks.ps1：用于在一键部署之前执行的必要系统检查；
- CopyFilesToVM.ps1：将自动把显卡直通的虚拟机部署到 Hyper-V 管理器中 ；
- Update-VMGpuPartitionDriver.ps1：用于更新显卡驱动，物理机显卡驱动如果进行了更新，则需要执行该脚本同步更新虚拟机内的显卡驱动。

执行 Easy-GPU-PV 部署脚本之前，请参考[虚拟化技术](###虚拟化技术)启用 Hyper-V 虚拟机。

#### 系统检查

进入 Easy-GPU-PV 目录内，以管理员身份运行 Windows PowerShell 并执行以下检查命令：

```powershell
.\PreChecks.ps1
```

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220804172038582.png" alt="image-20220804172038582" style="zoom: 50%;" />

如果执行过程中出现错误，一般表明需要调整 Windows PowerShell 的执行策略，修改 Execution Policy 为不受约束的（unrestricted）：

```powershell
Set-ExecutionPolicy unrestricted
```

执行调整后，再次运行 PreChecks.ps1 脚本检查系统是否符合部署要求：

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220804172341702.png" alt="image-20220804172341702" style="zoom:50%;" />

如果没有报错，就可以进行下一步了。

#### 虚拟机部署

部署需要使用 CopyFilesToVM.ps1 脚本，在部署前建议调整脚本内参数。

使用文本编辑器将 CopyFilesToVM.ps1 脚本打开，内容大致如下：

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220804172800486.png" alt="image-20220804172800486" style="zoom:50%;" />

红框所示的是可自定义的基础参数，其中推荐进行自定义的参数有以下几个：

- VMName：虚拟机名称，推荐进行自定义方便管理；
- SourcePath：虚拟机系统镜像，镜像需要自行下载，并将镜像所在的路径配置到此参数中；
- Edition：系统镜像的版本，需要通过指定的命令获取；
- SizeBytes：动态磁盘的大小；
- MemoryAmount：虚拟机内存大小，该项为固定内存容量，不是动态内存；
- CPUCores：虚拟机 CPU 核心数据，需要根据物理机的总线程数量来作出配置；
- VHDPath：虚拟机动态磁盘的存储路径；
- GPUName：显卡名称，Windows 10 系统只能使用 AUTO 值，Windows 11 系统则可以自定义值；
- GPUResourceAllocationPercentage：显卡资源分配比例，动态参数，即虚拟机占用显卡资源的最大百分比值；
- Username：用户名称；
- Password：登录密码；
- Autologon：是否启用自动登录。

部署过程中需要使用到系统镜像，请到[ Windows 11 磁盘映像（ISO）](https://www.microsoft.com/zh-cn/software-download/windows11)或其他网站下载。

系统版本 Edition 参数，需装载 ISO 镜像以命令行的方式获取。例如 Windows 10 专业版的系统镜像，想要获取该镜像的 Edition 版本参数，需先将镜像装载到物理机中：

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220804174047544.png" alt="image-20220804174047544" style="zoom:50%;" />

随后进入装载好的 DVD 驱动器目录内，以管理员身份打开 Windows PowerShell 终端并执行以下命令：

```powershell
dism /Get-WimInfo /WimFile:K:\sources\install.wim
```

终端会输出详细版本的索引信息，对应系统镜像的索引就是 CopyFilesToVM.ps1 文件中的 Edition 参数值：

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220804174322414.png" alt="image-20220804174322414" style="zoom:50%;" />

这里 Windows 10 专业版所需要配置 Edition 参数的值，即为其索引值 3。

基本配置完成后，即可开始执行 CopyFilesToVM.ps1 脚本部署 Hyper-V 虚拟机：

```powershell
.\CopyFilesToVM.ps1
```

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220804175320849.png" alt="image-20220804175320849" style="zoom:50%;" />

只要参数不出错，那么整个部署脚本的执行过程将十分顺利。

脚本执行完毕后按任意键退出终端即可，虚拟机将自动打开：

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220804175528612.png" alt="image-20220804175528612" style="zoom:50%;" />

#### 驱动更新

GPU 直通的本质是 Hyper-V 支持将物理机的资源分配给虚拟机，但虚拟机使用 GPU 资源的前提是系统内拥有 GPU 的驱动程序，且该驱动程序的版本必须与物理机上的 GPU 驱动版本保持一致。

因此，如果物理机上的显卡驱动进行了更新，那么虚拟机内的显卡驱动也需要进行更新。

显卡驱动更新同样需要进入脚本目录，在 Windows PowerShell 中执行以下命令即可：

```powershell
Update-VMGPUPartitonDriver.ps1 -VMName "Name of your VM" -GPUName "Name of your GPU"
```

关于 GPU 完整的名称，可以在系统的设备管理器中查看。

### 自动部署遗留问题

脚本部署虚拟机的整个过程可以说是行云流水的，但实际还存在某些问题，或者说是预设软件的安装存在某些问题。

#### 驱动缺失

虚拟机部署完毕后，设备管理器中必须拥有以下两个设备的信息：

- 声音、视频和游戏控制器：VB-Audio Virtual Cable
- 显示适配器：Parsec Virtual Display Adapter

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220804183540675.png" alt="image-20220804183540675" style="zoom:50%;" />

以上红框内的设备是必须存在的。其中，通用即插即用监视器就是 Parsec 内置的虚拟显示器，该显示器仅在客户机建立连接时被启用。

但大多数情况下，脚本运行完毕且虚拟机配置结束后，设备管理器中就只会存在 VB-Audio Virtual Cable 这一个设备的信息。这种情况的出现，是因为系统缺少了必要的 Parsec Virtual Display Driver 驱动，而该驱动并未能通过部署脚本成功安装到系统中。

Easy-GPU-PV 项目的作者在说明文档中阐述了该问题：**如果虚拟机部署、启动速度太快，很可能会导致某些驱动出现未成功安装的情况。**

查看 Windows 系统的计划任务程序，可以看到其中预设的两项任务详情：

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220804191626009.png" alt="image-20220804191626009" style="zoom:50%;" />

其中 Install Parsec Display Driver 任务用于安装 Parsec Virtual Display Adapter 驱动。

如果是根据项目作者所说的，部署、启动速度过快导致驱动无法安装成功的情况，一般在多次重启虚拟机后，驱动能够得到修复，但能够修复是小概率事件。如果情况没能得到修复，则推荐将以上两项计划任务删除并自行完成修复。

修复的方法很简单：卸载 Parsec 并重新安装，安装过程中务必勾选 Virtual Display Driver 选项。

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220805172011774.png" alt="image-20220805172011774" style="zoom: 67%;" />

注意事项：

- 推荐使用 Uninstall Tools 卸载 Parsec，以避免不必要的错误。

- 安装完毕后，推荐将 parsecd.exe 程序设置为“以管理员身份运行此程序”，以避免不必要的错误。

#### 网络故障

虚拟机部署完毕后，有一种情况会导致 Parsec 无法安装，即网络出现不明原因的故障。Parsec 无法安装，自然必要驱动也不存在。

原因是 Parsec 并不是国内的软件，计划任务需要从网络中获取 Parsec 程序的安装包。这个过程中如果网络连接不上（故障），则会导致 Parsec 程序的安装包无法获取的情况，自然程序也无法安装。

如果出现这种网络故障，一般表明自行获取安装包的途径也是不可取的。推荐使用代理服务器，让代理软件将系统流量进行代理转发，Parsec 程序的安装包就可以顺利完成下载了。

### Parsec 软件配置

Parsec 本身是一款在国内未拥有代理商的软件，使用它必需拥有 Parsec 的账户用于登录。因此，使用代理进行账号的注册与登录几乎是必须的。

Parsec 安装过程中会让用户选择软件的使用对象，它将根据该结果来选择存放应用程序数据的目录：

- Shared（为所有用户安装）：Parsec 的配置文件 config.txt 位于 C:\ProgramData\Parsec\ 目录下；
- Per User（仅为当前用户安装）：Parsec 的配置文件 config.txt 位于 C:\user\username\AppData\Roaming 目录下。

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220805172716081.png" alt="image-20220805172716081" style="zoom:67%;" />

Parsec 支持配置代理服务器，只需要在 Parsec 配置文件添加代理服务器的信息即可：

```properties
app_proxy_address = x.x.x.x
app_proxy_scheme = http
app_proxy = true
app_proxy_port = xxx
```

如果因为[驱动缺失](####驱动缺失)或[网络故障](####网络故障)等原因，需要自行安装 Parsec 程序。那么在程序完成安装后，还需要将隐私模式和虚拟显示器的配置，手动添加到 config.txt 中：

```properties
host_virtual_monitors = 1
host_privacy_mode = 1
```

最后需要注意，如果代理软件开启了规则模式，请确认已将 parsec.app 网址添加到规则列表中，否则系统将无法自动登录 Parsec 软件。

### Parsec 隐私模式

支持显卡直通的虚拟机，虽说已经具备使用物理机显卡的能力了，但仍旧无法避免系统会自动选择使用虚拟显卡来渲染图形的尴尬情况。如果不对默认的虚拟显卡进行限制，那么即便使用 Parsec 远程连接到了虚拟机上，虚拟机也会默认使用虚拟显卡来输出桌面图像，从而导致各种延迟卡顿。

Easy-GPU-PV 会直接在设备管理器中，将虚拟显卡的驱动设置为禁用。禁用虚拟显卡驱动之后，系统默认情况下就都会使用直通的 GPU 对图像进行渲染了。

但随之而来的问题是，一旦虚拟显卡的驱动被禁用，虚拟机的显示分辨率就会被锁定在 1024x768 (4:3) 且无法调整，这是虚拟机本身的限制。这种情况下，远程连接到虚拟机时，即便使用了直通的 GPU 编解码，但显示分辨率也会被锁定在 1024x768 或之下，显示是不完美的。

但所幸 Parsec 支持开启**隐私模式**，隐私模式下当远程连接发起时，远程主机会启用虚拟显示器，而虚拟显示器的内容会使用直通的 GPU 进行编解码。更重要的是，虚拟显示器的分辨率是不受限制的。

注意，Parsec 中开启虚拟显示器和开启隐私模式是两个独立的选项，你必须同时启用它们，才能够让直通的 GPU 输出虚拟显示器的桌面图像。

### Parsec 虚拟显示器

虚拟显示器（Virtual Display）是在硬件上模拟出来的额外显示器，它的作用类似于 Windows 10 系统上的任务视图，主要是为了提供额外的桌面空间。

虽然虚拟显示器宣称可以最大限度地提高桌面空间的利用率及提高工作效率等，但实际上这种技术带来的效果并不理想。不过，虚拟显示器对于显卡直通的 Hyper-V 虚拟机上的 Parsec 来说，却尤为重要。

Parsec 中使用到虚拟显示器的是被称为隐私模式（Privacy Mode）功能。开启该模式后，当远程 Client 客户机连接 Host 主机时，主机上的屏幕将被设置为不可见，从而避免其他人窥视主机屏幕，从而保护了用户的隐私。该模式就是利用虚拟显示器，结合 Windows 系统的显示机制来实现的。

当 Windows 系统连接两块显示器（无论是否为物理显示器）时，显示设置中会列出这些屏幕：

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220805164055343.png" alt="image-20220805164055343" style="zoom: 25%;" />

多显示器设置中通常包含以下选项：

<img src="images/Hyper-V & Easy-GPU-PV.images/image-20220805164211444.png" alt="image-20220805164211444" style="zoom:50%;" />



Parsec 在没有客户机连接时，不会启用虚拟显示器。

当有客户机连接主机时，虚拟显示器会随之启动，系统将识别 Parsec 的虚拟显示器为 2 号显示器。系统识别虚拟显示器后，Parsec 会调整多显示器选项，将其设置为“仅在 2 上显示”，间接让 1 号“物理”显示器不再显示任何画面。

这个做法是相当巧妙的，可以说 Parsec 的隐私模式和虚拟显示器功能，正好歪打正着地解决了 GPU 直通的虚拟机被远程连接时的显示分辨率问题。
