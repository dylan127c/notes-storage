### 概述

本篇记录个人系统环境配置以及个人开发环境配置。

### 编程字体

编程字体推荐，包括英文字体和中文字体两类。**请在非商用的情况下使用它们。**

- 英文字体推荐：[Cascadia Code](https://github.com/microsoft/cascadia-code/releases/download/v2111.01/CascadiaCode-2111.01.zip)、[SF Pro Text](https://fontsfree.net/?s=SF+Pro+Text)、[SF Pro Display](https://fontsfree.net/?s=SF+Pro+Display)、Consolas；

- 中文字体推荐：[PingFang SC](/resources/PingFang%20SC.rar)、Microsoft YaHei。

为了更正确地使用字体，需要了解关于后备字体（Fallback Font）的概念：**后备字体**（**Fallback Font**）是指在当时显示的字体缺乏某些字符时，被用于显示缺失字符的字体。因为其作为显示的最后一道防线，后备字体应该尽可能包含所有 Unicode 字符。

当缺失字符没有后备字体用于显示时，通常会将缺失字符改为黑色方块、白色空心方块、问号、Unicode 占位字符（U+FFFD）显示，或者干脆略过该字符。在实务上，像是 CSS 等支持字体列表依序显示的系统，通常会将一或多套后备字体置入列表最后，以防止缺字的情况发生。

假设某网页依赖 properties 文件中的 fontFamily 属性值来配置显示字体，且该文件中 fontFamily 属性如下：

```properties
fontFamily = Cascadia Code, PingFang SC
```

那么由于 Cascadia Code 字体不支持中文字符，因此网页内的所有的中文字符都将使用 PingFang SC 字体（Fallback Font）显示，所有的英文字符将正常地使用 Cascadia Code 字体显示。

如果希望 HTML 中的元素能拥有更细致的字体分类，可以选择使用 CSS 类选择器或 JS 来为指定的网页元素应用其他字体。

针对 Cascadia Code 和 PingFang SC 有以下两点说明：

- Cascadia Code 字体有几个不同的版本可供选择，具体区别可参考[ Cascadia Code 版本](https://learn.microsoft.com/zh-cn/windows/terminal/cascadia-code)，一般使用 Cascadia Code 版本即可；
- PingFang SC 字体中 Medium 字重的显示效果比 Regular 要好。

如果希望在 Windows 系统中使用这些字体，则需要获取对应的字体文件并安装。

安装字体时，需要在字体文件的右键菜单中选择“为所有用户安装”：

<img src="images/Personal%20Workspace.images/image-20230415031552086.png" alt="image-20230415031552086" style="zoom: 50%;" />

### 程序配置

以下是常用软件的一些配置详情。

#### IntelliJ IDEA

以下配置针对 IntelliJ IDEA 2022.1.3 版本：

<img src="images/Personal%20Workspace.images/image-20230831014701375.png" alt="image-20230831014701375" style="zoom: 50%;" />

中文语言包：

<img src="images/Personal%20Workspace.images/image-20230415041226585.png" alt="image-20230415041226585" style="zoom:33%;" />

主题：

<img src="images/Personal%20Workspace.images/image-20230415040035469.png" alt="image-20230415040035469" style="zoom:33%;" />

外观：

<img src="images/Personal%20Workspace.images/image-20230415040331453.png" alt="image-20230415040331453" style="zoom:33%;" />

字体：

<img src="images/Personal%20Workspace.images/image-20230415041133315.png" alt="image-20230415041133315" style="zoom:33%;" />

配色方案：

<img src="images/Personal%20Workspace.images/image-20230415040803985.png" alt="image-20230415040803985" style="zoom:33%;" />

#### Visual Studio Code

VSCode 有账户同步配置的功能，推荐登录微软账号进行配置的同步。它提供的配置方式有两种，一种是软件层面上，另一种是代码层面上的。

软件层面上的即设置选项卡：

<img src="images/Personal%20Workspace.images/image-20230415043128267.png" alt="image-20230415043128267" style="zoom:33%;" />

修改设置选项卡中的项目，会同步将变更写入到配置文件中。

代码层面上的即直接操作设置文件：

<img src="images/Personal%20Workspace.images/image-20230415043315165.png" alt="image-20230415043315165" style="zoom:33%;" />

<img src="images/Personal%20Workspace.images/image-20230415043346073.png" alt="image-20230415043346073" style="zoom:33%;" />

配置参考：

```json
{
    "workbench.startupEditor": "none",
    "workbench.colorTheme": "One Dark Pro Flat",
    "editor.fontSize": 16,
    "editor.fontFamily": "Cascadia Code, PingFang SC, Microsoft YaHei, Consolas, 'Courier New', monospace",
    "files.autoSave": "afterDelay",
    "editor.unicodeHighlight.includeComments": false,
    "http.proxy": "socks5://127.0.0.1:19121",
    "http.proxySupport": "on",
    "editor.unicodeHighlight.allowedLocales": {
        "zh-hant": true
    },
    "extensions.ignoreRecommendations": true,
    "git.openRepositoryInParentFolders": "never",
    "editor.tokenColorCustomizations": {
        "comments": "#6A9955"
    }
}
```

对于 1.85.0 版本或某些版本来说，直接在设置中打开 settings.json 文件进行修改，可能会出现更改无法保存的情况。解决方法是直接进入 settings.json 文件的所在目录种，打开该文件进行编辑，这种情况下编辑后的内容能够被正常保存。

VSCode 同样可以配置简中显示语言及 One Dark Pro 主题：

<img src="images/Personal%20Workspace.images/image-20230415043551986.png" alt="image-20230415043551986" style="zoom:33%;" />

其中 One Dark 主题同样拥有 4 种不同的配色方案，使用以下方式可快捷更改：

<img src="images/Personal%20Workspace.images/image-20230415043720761.png" alt="image-20230415043720761" style="zoom:33%;" />

<img src="images/Personal%20Workspace.images/image-20230415043750334.png" alt="image-20230415043750334" style="zoom:33%;" />



#### Typora

以下配置针对 1.6.7 版本的 Typora 编辑器：

<img src="images/Personal%20Workspace.images/image-20230824055414118.png" alt="image-20230824055414118" style="zoom:50%;" />

“文件”选项卡需要配置或勾选以下几个选项：

- 启动选项：重新打开上次使用的文件和目录；
- 保存 & 恢复：自动保存；

<img src="images/Personal%20Workspace.images/image-20230415032229465.png" alt="image-20230415032229465" style="zoom:33%;" />

“通用”选项卡需要配置或勾选以下几个选项：

- 高级设置：开启调试模式；
- Typora 服务器：Typora 服务使用国内服务器；

<img src="images/Personal%20Workspace.images/image-20230415032142937.png" alt="image-20230415032142937" style="zoom: 33%;" />

“图像”选项卡需要配置或勾选以下几个选项：

- 图片语法偏好：**优先使用相对路径**；
- 图片语法偏好：**插入时自动转义图片 URL**；
- 插入图片时...：选择“复制到指定路径”并勾选“对本地位置的图片应用上述规则”，路径规则如下：

```
./images/${filename}.images
```

<img src="images/Personal%20Workspace.images/image-20230415032229464.png" alt="image-20230415032229464" style="zoom:33%;" />

其余的偏好设定可以维持默认。

关于 Typora 的主题配置，其配置文件均位于 `~\AppData\Roaming\Typora\themes` 目录。

以 GitHub 主题为例，它的配置文件为 github.css。如果需要覆盖这些配置，仅需要在 themes 目录下创建 github.user.css 文件。

Typora 会顺序读取 github.css、github.user.css 文件的内容，用户自定义的样式将按照 CSS 规则覆盖掉原本的样式。

如需获取其他 Typora 主题，请访问：[Typora Themes](https://theme.typora.io/)。

通过编辑注册表，可以将 Typora 所支持的 .md 类型添加到“新建”菜单中：

<img src="images/Personal%20Workspace.images/image-20220717024357222.png" alt="image-20220717024357222" style="zoom: 50%;" />

具体操作步骤如下：

- 使用 Win + R 快捷键打开运行窗口，输入 regedit 并确认，以打开注册表窗口；
- 定位到注册表 HKEY_CLASSES_ROOT\\.md 路径下，修改 .md 项中默认字符串的数据值为 Typora.md;
- 在 .md 项下新建 ShellNew 项，并添加字符串值 NullFile，不需要设置数据值。

<img src="images/Personal%20Workspace.images/image-20220717025112747.png" alt="image-20220717025112747" style="zoom: 50%;" />

<img src="images/Personal%20Workspace.images/image-20220717025135331.png" alt="image-20220717025135331" style="zoom:50%;" />

实际上 .md 格式仅能由指定的程序打开，就本机来说支持打开的这种格式的有 Typora、VSCode 等。

默认能够打开 .md 格式文件的程序，会配置在注册表中 .md 项下的 OpenWithProgids 里：

<img src="images/Personal%20Workspace.images/image-20220717030538905.png" alt="image-20220717030538905" style="zoom:50%;" />

这也是为什么 .md 项中默认字符串的数据值能被设置为 Typora.md 的原因。除此之外你还能将它设置为 VSCode.md，即 .md 项中的默认字符串值只能接受 Typora.md 或 VSCode.md。

如果在 .md 项中填入 VSCode.md，则新建菜单会变成：

<img src="images/Personal%20Workspace.images/image-20220717030743392.png" alt="image-20220717030743392" style="zoom:50%;" />

你会发现改变的只有新建 Markdown 文件项的名称，它仍然可以被新建，但打开它的程序仍旧是 Typora。

这很好理解，因为本次修改注册表并未影响默认打开 .md 类型文件的软件预设，预设打开 .md 类型文件的仍旧是 Typora 程序，而非 VSCode 程序。

希望修改默认打开程序，请在指定格式文件的属性中选择更改打开方式：

<img src="images/Personal%20Workspace.images/image-20230415060122428.png" alt="image-20230415060122428" style="zoom: 50%;" />

并不推荐通过注册表的方式，为指定类型的文件选择固定的打开程序，因为注册表的设置比较复杂！且影响深远！非必要情况下尽量不要改动注册表，以免系统遭到不必要的损坏，能安全达到预期效果才是我们需要的最好结果。

#### Xshell

从护眼的角度出发，可以选择程序自带的 XTerm 配色方案。亦或将以下类 Ubuntu 的配色方案保存为 .xcs 文件，导入到配色方案中使用。

```properties
[Ubuntu]
text(bold)=ffffff
magenta(bold)=ad7fa8
text=ffffff
white(bold)=eeeeec
green=4e9a06
red(bold)=ef2929
green(bold)=8ae234
black(bold)=555753
red=cc0000
blue=3465a4
black=000000
blue(bold)=729fcf
yellow(bold)=fce94f
cyan(bold)=34e2e2
yellow=c4a000
magenta=75507b
background=300a24
white=d3d7cf
cyan=06989a
[Names]
count=1
name0=Ubuntu
```

#### Maven

Maven 仓库主要的问题是下载源，对于下载源为国外的 JAR 包，将它们下载至本地可能需要花上不少的时间，如果遇上无法下载的情况，就更是让人头大。

最好的方式，是将 Maven 仓库默认的源修改为国内的镜像源。

IntelliJ IDEA 将默认的 Maven 配置放在了固定的地方，无论用户更换什么版本的 Maven 程序，该配置通用且默认生效（本地仓库路径亦是如此）：

<img src="images/Personal%20Workspace.images/image-20230415034732938.png" alt="image-20230415034732938" style="zoom:33%;" />

定位至 /.m2 目录，将 settings.xml 修改为以下内容：

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                          https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <mirrors>
        <mirror>
            <id>aliyunmaven</id>
            <mirrorOf>*</mirrorOf>
            <name>阿里云公共仓库</name>
            <url>https://maven.aliyun.com/repository/public</url>
        </mirror>
    </mirrors>
</settings>
```

如果 settings.xml 配置不存在，请自行创建它。更多镜像源配置，请参考[阿里云云效 Maven](https://developer.aliyun.com/mvn/guide)。

除了阿里云之外，还可选择使用其他的 Maven 源，网络上很容易就能检索到其他的 Maven 源。

#### Listary

Listary 是一款用于 Windows 的文件搜索、定位的辅助软件。它提供了便捷、人性化的文件（夹）定位方式，改善 Windows 传统低效的文件打开/保存对话框，同时改善了常见文件管理器中文件夹切换的效率。

Listary 一般用于文件检索，但它还能用来进行网络搜索。常用的网络搜索配置如下：

<img src="images/Personal%20Workspace.images/image-20230415050459924.png" alt="image-20230415050459924" style="zoom:33%;" />

根据英文翻译中文的 url 配置为：

```
https://translate.google.com/#view=home&op=translate&sl=en&tl=zh-CN&text={query}
```

根据中文翻译英文的 url 配置为：

```
https://translate.google.com/#view=home&op=translate&sl=zh-CN&tl=en&text={query}
```

双击 Ctrl 唤醒 Listary 输入“fy ”即可唤醒英中翻译：

<img src="images/Personal%20Workspace.images/image-20230415050912734.png" alt="image-20230415050912734" style="zoom:33%;" />

键入单词按回车键，Listary 就会调用浏览器打开指定 url 进行单词检索：

<img src="images/Personal%20Workspace.images/image-20230416132142597.png" alt="image-20230416132142597" style="zoom:33%;" />

如果希望找寻某一路径下的指定文件，可以在检索文件的时候添加上指定路径。

例如，系统中存在多个 config 文件，目标检索的 config 文件位于 .ssh 目录，那么检索规则为：

<img src="images/Personal%20Workspace.images/image-20230416131912189.png" alt="image-20230416131912189" style="zoom:33%;" />

这里需要注意 Windows 系统下，文件路径中使用的是反斜杠“\”，如果错误输入了斜杠“/”，则无法检索到结果。

#### Parsec

Parsec 是依赖代理服务的远程控制软件，可以说大多数情况下必须提供代理，以登录 Parsec 账户以使用其功能。

Parsec 安装过程中会让用户选择软件的使用对象，它将根据该结果来选择存放应用程序数据的目录：

- Shared（为所有用户安装）：Parsec 的配置文件 config.txt 位于 C:\ProgramData\Parsec\ 目录下；
- Per User（仅为当前用户安装）：Parsec 的配置文件 config.txt 位于 C:\user\username\AppData\Roaming 目录下。

只需要保证 config.txt 配置中有以下内容：

```properties
# 代理服务器配置
app_proxy_address = x.x.x.x
app_proxy_scheme = http
app_proxy = true
app_proxy_port = xxx

# 开启虚拟显示器和隐私模式
host_virtual_monitors = 1
host_privacy_mode = 1
```

希望多了解 Parsec 的工作原理，请参考：[Parsec 远程控制](./Hyper-V & Easy-GPU-PV.md/###Parsec 远程控制)。

#### Clash for Windows

基本配置：

<img src="images/Personal%20Workspace.images/image-20230415052058468.png" alt="image-20230415052058468" style="zoom:33%;" />

<img src="images/Personal%20Workspace.images/image-20230415052136919.png" alt="image-20230415052136919" style="zoom:33%;" />

TUN Mode 配置：

<img src="images/Personal%20Workspace.images/image-20230415052156794.png" alt="image-20230415052156794" style="zoom:33%;" />

<img src="images/Personal%20Workspace.images/image-20230415052206018.png" alt="image-20230415052206018" style="zoom:33%;" />

推荐启用 Rule 模式：

<img src="images/Personal%20Workspace.images/image-20230415052256050.png" alt="image-20230415052256050" style="zoom:33%;" />

订阅文件可以使用 JavaScript 进行改写，详情请参考官方文档：[Clash for Windows - 配置文件预处理](https://docs.cfw.lbyczf.com/contents/parser.html)。

获取 Clash for Windows 请参考：[clash for windows pkg - releases](https://github.com/Fndroid/clash_for_windows_pkg/releases/tag/0.20.21)。

个人规则、分组的实现，请参考：[Proxy Rules](https://github.com/dylan127c/proxy-rules)。

#### MacType

MacType 是一个替代 Windows 自身核心部件 GDI 进行字体渲染的开源软件。

该软件由中国网友 FlyingSnow 基于已经停止更新的 gdi++ 开发，使用可配置性较高的 FreeType 渲染字体。

软件安装后，推荐以注册表方式启动：

```properties
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows]
"AppInit_DLLs"="MacType64.dll"
"LoadAppInit_DLLs"=dword:00000001
"RequireSignedAppInit_DLLs"=dword:00000000


[HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows NT\CurrentVersion\Windows]
"AppInit_DLLs"="MacType.dll"
"LoadAppInit_DLLs"=dword:00000001
"RequireSignedAppInit_DLLs"=dword:00000000
```

系统为 Windows x64 时，将以上内容保存为 .reg 文件并运行即可。如果是 x86 系统，则仅保存并运行上半部分内容。

以注册表启动 MacType 能获取最无缝的渲染体验，但启用该方式需要满足两点前置条件：

1. 将 MacType 安装路径添加到环境变量 Path 中；
2. **在 BIOS 中关闭 Secure Boot 安全启动项**。详情参考：[Enable Registry Mode Manually](https://github.com/snowie2000/mactype/wiki/Enable-registry-mode-manually)。

使用 MacType **无需安装额外的字体**，推荐选择 Clean Light 渲染主题：

<img src="images/Personal%20Workspace.images/image-20230415053340305.png" alt="image-20230415053340305" style="zoom: 67%;" />

获取 MacType 请参考：[MacType](https://www.mactype.net/)。
