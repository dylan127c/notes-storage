### 概述

Windows Subsystem for Android（WSA）全称适用于 Android™️ 的 Windows 子系统。顾名思义，它是微软在 Windows 系统上推出的 Android 子系统，它允许将 Android 应用间接部署在 Windows 系统上。

### 虚拟化技术

WSA 依赖于虚拟化技术，需要在 Windows 功能中启用 Hyper-V 功能和虚拟机平台功能，请参考：[虚拟化技术](./Hyper-V & Easy-GPU-PV.md/###虚拟化技术)。

### 安装 WSA

一般可以可以通过 Microsoft Store 获取并安装 WSA，但国区应用商店内尚未上架。即便如此，仍旧可以使用某些方法，将 WSA 安装到 Windows 系统上。

方法有两种：

1. 修改 Windows 系统的国家或地区，将其修改为支持安装 WSA 的特定区域；
2. 获取 WSA 离线安装包，通过 Windows PowerShell 将其安装到 Windows 系统上。

个人比较推荐使用第二种方法。

#### 方法一：修改地域

在 Windows 设置中找到**国家或地区**选项，将“中国”修改为“美国”（或其它支持安装 WSA 安卓子系统应用的区域）：

<img src="images/Windows Subsystem for Android.images/Snipaste_2022-08-12_15-04-53.png" alt="Snipaste_2022-08-12_15-04-53" style="zoom: 50%;" />

修改完毕后打开 Microsoft Store 应用商店，它将自动定位到美区的应用市场上。

搜索 Amazon Appstore 应用下载并安装：

<img src="images/Windows Subsystem for Android.images/Snipaste_2022-08-12_15-07-50.png" alt="Snipaste_2022-08-12_15-07-50" style="zoom: 50%;" />

**Amazon Appstore 应用安装完毕后，WSA 也将一并安装到 Windows 系统中。**

这种方法虽然简单，但无人知道微软是否打算在未来对应用商店的进行区域封锁。如果未来存在区域封锁，可以选择使用第二种安装方法。

#### 方法二：命令行安装

命令行安装只需要下载 WSA 的安装包，利用几条简短的命令即可完成。从流程上看，它甚至没有修改地域来得麻烦。

访问 [Microsoft Store - Generation Project](https://store.rg-adguard.net) 搜索 WSA 的 **ProductId** 产品编码：

```
9P3395VX91NR
```

<img src="images/Windows Subsystem for Android.images/image-20220812151825728.png" alt="image-20220812151825728" style="zoom: 50%;" />

其中，列表末尾的 .msixbundle 类型文件就是 WSA 的安装包，下载到本地任意目录。

进入安装包所在目录，以管理员身份打开 Windows PowerShell 终端并执行命令：

```powershell
Add-AppxPackage .\MicrosoftCorporationII.WindowsSubsystemForAndroid_2206.40000.15.0_neutral___8wekyb3d8bbwe.Msixbundle
```

<img src="images/Windows Subsystem for Android.images/Snipaste_2022-08-12_15-34-00.png" alt="Snipaste_2022-08-12_15-34-00" style="zoom: 50%;" />

等待安装进度条结束，WSA 安装即完成。

在 Windows 11 系统下，唤醒开始菜单，能够在全部应用列表中，查看到适用于 Android™️ 的 Windows 子系统设置：

<img src="images/Windows Subsystem for Android.images/image-20220812153906724.png" alt="image-20220812153906724" style="zoom:50%;" />

这就意味着 WSA 已经成功安装到 Windows 系统中。

### Android 应用安装

以上无论采取何种方法安装 WSA，程序都会将 Amazon Appstore 应用部署到 Windows 系统下，因为它是 WSA 默认的应用商店。

同样受限于国区网络原因，Amazon Appstore 实际无法使用。

但实际上 WSA 就是某种意义上的安卓系统，可以通过命令行的方式将 Android 应用安装到 WSA 中，使用

 [Android 调试桥](https://developer.android.com/studio/command-line/adb)（Android Debug Bridge, adb）工具即可。

#### Android 调试桥

ADB 工具在 Windows 系统下，是一个简单的 abd.exe 可执行程序，它仅能够在终端中以命令行的形式调用。

ADB 工具可从 [Platform Tools](https://developer.android.com/studio/releases/platform-tools) 网站获取，选择“**适用于 Windows 的 SDK Platform-Tools**”下载：

<img src="images/Windows Subsystem for Android.images/image-20220812163031217.png" alt="image-20220812163031217" style="zoom: 50%;" />

下载并解压到固定目录（便于配置全局环境变量）：

<img src="images/Windows Subsystem for Android.images/image-20220812160002367.png" alt="image-20220812160002367" style="zoom: 50%;" />

随后将 abd.exe 所在目录，添加到**系统变量**下的 Path 中：

<img src="images/Windows Subsystem for Android.images/image-20220812163246253.png" alt="image-20220812163246253" style="zoom:50%;" />

完成后，在任何位置唤醒终端，都能够直接使用 adb 工具。

#### 启用开发者模式

ADB 工具实则用于调试 Android 系统，启用调试首先需要与 Android 系统建立连接。Android 系统如果需要接受调试，则必须启用**开发人员模式**（开发者模式）。

打开 WSA 子系统，在开发人员选项卡中启用开发人员模式：

<img src="images/Windows Subsystem for Android.images/image-20220812160152216.png" alt="image-20220812160152216" style="zoom: 50%;" />

首次启用开发人员模式时，WSA 内的调试模式可能仍尚未启用，使用 ADB 工具连接 WSA 可能会提示连接失败。建议启用开发人员模式后**重启子系统**，或进入**管理开发人员设置**手动**激活开发人员模式**。

#### 连接/断开 Android 系统

连接 Android 系统：

```powershell
adb connect 127.0.0.1:58526
```

断开 Android 系统：

```powershell
adb disconnect 127.0.0.1:58526
```

<img src="images/Windows Subsystem for Android.images/image-20220812160804526.png" alt="image-20220812160804526" style="zoom: 50%;" />

#### 使用 ADB 安装应用

ADB 工具安装应用十分简单，与 WSA 建立连接后，使用以下命令即可安装任意的 Android 应用：

```powershell
adb install {android package path}
```

以 Apple Music 应用为例，假如已经获取了 Apple Music.apk 安装包，那么执行以下命令即可完成安装：

<img src="images/Windows Subsystem for Android.images/image-20220812161146589.png" alt="image-20220812161146589" style="zoom:50%;" />

在 Windows 11 系统下，能够在全部应用中找到已安装的 Android 应用入口：

<img src="images/Windows Subsystem for Android.images/image-20220812161242068.png" alt="image-20220812161242068" style="zoom:50%;" />

卸载 Android 应用与卸载 UWP 应用的操作一样，在应用图标的右键菜单中选择卸载，可移除指定的应用。

### 更多设置

WSA（2206.40000.15.0）默认采用的**子系统资源**策略是**连续**模式，该模式下子系统始终在后台运行，以此加快 WSA 应用的启动速度。

<img src="images/Windows Subsystem for Android.images/image-20230414193447540.png" alt="image-20230414193447540" style="zoom: 50%;" />

简单来说，子系统资源中如果选择了连续模式，就等同于选择了让 WSA 随开机一起自启动。

如果非频繁使用 WSA 应用，还可以选择使用**按需要**模式。

最新版本（2303.40000.4.0）的 WSA 中还提供了**部分运行中**模式：

<img src="images/Windows Subsystem for Android.images/image-20230421193537358.png" alt="image-20230421193537358" style="zoom:50%;" />

### 现存问题

WSA 安装子系统在使用的过程中，可能会弹出以下连接受限通知：

<img src="images/Windows Subsystem for Android.images/image-20220812165347271.png" alt="image-20220812165347271" style="zoom:50%;" />

但实际 VirtWifi 并未受限，换句话说 WSA 内的网络访问实际没有问题。

可以使用 ABD 调试工具，以命令行的方式间接关闭 WSA 的连接受限通知：

```shell
adb shell settings put global captive_portal_https_url https://www.google.cn/generate_204
adb shell settings put global captive_portal_http_url http://www.google.cn/generate_204
```

其中 captiv_protal_http_url 是 VirtWifi 用于测试网络连通性的目标网址。

以上命令用于更改这个目标网址为安卓中国的测试链接：http://www.google.cn/generate_204。

显然，受限通知的出现，是因为 WSA 内使用了某个国内网络无法访问的网址，作为 VirtWifi 连通性测试的目标网址。这样即便 VirtWifi 网络正常（国内网络可用），网络连通性测试也大概率会失败告终。

反馈到 Windows 系统中，就会提示连接受限（某种意义上确实受限）。

重新配置了 captiv_protal_http_url 后，可以通过以下命令行查看配置是否生效：

```shell
 adb shell settings get global captive_portal_https_url
 adb shell settings get global captive_portal_http_url
```

想了解更多有关 WSA 的详情，请参考：[适用于 Android™️ 的 Windows 子系统](https://learn.microsoft.com/zh-cn/windows/android/wsa/)。
