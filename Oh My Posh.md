### 概述

本篇介绍一个用于美化终端的 GitHub 项目 [oh-my-posh](https://github.com/JanDeDobbeleer/oh-my-posh)（简称：OMP）。

一般终端启动时都会查询并加载默认的配置文件：

<div align="center"><img src="images/Oh%20My%20Posh.images/image-20230828034813880.png" alt="image-20230828034813880" style="width:67%;" /></div>

OMP 的核心就是利用这个默认的配置文件来预加载主题配置：

<div align="center"><img src="images/Oh%20My%20Posh.images/image-20230828035001164.png" alt="image-20230828035001164" style="width:67%;" /></div>

### 字体配置

OMP 官方推荐使用 [Nerd Font](https://www.nerdfonts.com/font-downloads) 中提供的字体，以更好地兼容官方提供的样式。

这里推荐使用 Cascadia Code 字体的变体 [Caskaydia Cove Nerd Font](https://github.com/ryanoasis/nerd-fonts/releases/download/v3.1.1/CascadiaCode.zip) 字体：

<div align="center"><img src="images/Oh%20My%20Posh.images/image-20230422061538249.png" alt="image-20230422061538249" style="width:42%;" /></div>

解压并进入字体目录，选中所有的 `.otf` 字体文件右键选择“为所有用户安装”。安装完毕后，更改终端字体为合适的目标字体（可能有多个 CaskaydiaCove 的选项，都可以用）：

<div align="center"><img src="images/Oh%20My%20Posh.images/Snipaste_2024-03-09_04-35-14.png" alt="Snipaste_2024-03-09_04-35-14" style="width: 80%;" /></div>

### 执行策略

默认 Windows Terminal 不允许执行任何脚本，但加载主题文件就需要使用自定义的配置脚本，这里只能修改默认的终端执行策略，以放行配置脚本的执行。

以管理员身份打开 Windows Terminal 改变终端执行策略：

```bash
Set-ExecutionPolicy RemoteSigned
```

查看执行策略：

```bash
Get-ExecutionPolicy -List
```

|     Scope     | ExecutionPolicy |
| :-----------: | :-------------: |
| MachinePolicy |    Undefined    |
|  UserPolicy   |    Undefined    |
|    Process    |    Undefined    |
|  CurrentUser  |    Undefined    |
| LocalMachine  |  RemoteSigned   |

关于执行策略，请参考：[执行策略 - PowerShell](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.4)。

### 安装应用

在 Microsoft Store 中找到 oh-my-posh 程序并安装：

<div align="center"><img src="images/Oh%20My%20Posh.images/Snipaste_2024-03-09_04-38-19.png" alt="Snipaste_2024-03-09_04-38-19" style="width: 80%;" /></div>

程序默认安装路径为 `%HOMEPATH%\AppData\Local\Programs\oh-my-posh\`：

<div align="center"><img src="images/Oh%20My%20Posh.images/Snipaste_2024-03-09_04-47-23.png" alt="Snipaste_2024-03-09_04-47-23" style="width:75%;" /></div>

程序 `oh-my-posh.exe` 位于 `oh-my-posh\bin` 目录，主题配置文件位于 `oh-my-posh\themes` 目录。

编辑环境变量将 `oh-my-posh\bin` 目录添加到用户或系统的 `Path` 中：

```
%HOMEPATH%\AppData\Local\Programs\oh-my-posh\bin
```

<div align="center"><img src="images/Oh%20My%20Posh.images/Snipaste_2024-03-09_05-01-10.png" alt="Snipaste_2024-03-09_05-01-10" style="width:42%;" /></div>

在 `Documents\WindowsPowerShell` 目录下，创建一个 `Microsoft.PowerShell_profile.ps1` 脚本文件，将加载主题的配置写入该文件：

```bash
oh-my-posh init pwsh --config 'C:\Users\dylan\AppData\Local\Programs\oh-my-posh\themes\clean-detailed.omp.json' | Invoke-Expression
```

<div align="center"><img src="images/Oh%20My%20Posh.images/Snipaste_2024-03-09_04-40-40.png" alt="Snipaste_2024-03-09_04-40-40" style="width: 75%;" /></div>

打开终端，样式即刻生效：

<div align="center"><img src="images/Oh%20My%20Posh.images/Snipaste_2024-03-09_05-08-46.png" alt="Snipaste_2024-03-09_05-08-46" style="width:80%;" /></div>

预览主题，请参考：[Themes](https://ohmyposh.dev/docs/themes)。

Windows 系统下 Git 版本管理工具同样可以使用 oh-my-posh 程序：

```bash
eval "$(oh-my-posh init bash --config 'C:/Users/dylan/AppData/Local/Programs/oh-my-posh/themes/clean-detailed.omp.json')"
```

<div align="center"><img src="images/Oh%20My%20Posh.images/Snipaste_2024-03-09_05-18-15.png" alt="Snipaste_2024-03-09_05-18-15" style="width:80%;" /></div>

打开 Git Bash 样式即刻生效：

<div align="center"><img src="images/Oh%20My%20Posh.images/Snipaste_2024-03-09_05-21-23.png" alt="Snipaste_2024-03-09_05-21-23" style="width:80%;" /></div>
