### 概述

Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行时环境，用于服务器端和网络应用的开发。它允许开发者使用 JavaScript 编写后端代码，而不仅仅局限于前端开发。

### 模块安装

Node.js 一般使用 `npm` 包管理工具来安装必要的模块。

```bash
# 安装模块
npm install <package_name>

# 卸载模块
npm uninstall <package_name>
```

```bash
# 查看已安装模块（不包括全局模块）
npm list
```

非全局安装的模块通常位于执行安装模块命令路径下的 `node_modules` 目录。

另外，执行 `npm list` 命令，对应模块的安装位置也会一并输出：

```
~]# npm list
Test@ F:\Desktop\Test
`-- axios@1.6.7
```

如果需要安装、卸载全局模块，添加 `-g` 参数：

```bash
# 安装全局模块
npm install -g <package_name>

# 卸载全局模块
npm uninstall -g <package_name>
```

```bash
# 查看全局模块的默认安装路径
npm root -g
```

假设 `yarn` 模块被全局安装，那么使用 `list -g` 参数实际也能查看到全局模块的安装路径：

```
~]# npm list -g
C:\Users\dylan\AppData\Roaming\npm
`-- yarn@1.22.22
```

Yarn 是一个用于管理 JavaScript 项目依赖关系的包管理工具，由 Facebook、Google、Exponent 和 Tilde 联合开发。与 `npm` 类似，Yarn 允许你从 `npm` 仓库或者其他源（如 GitHub）安装、更新、卸载和管理 JavaScript 包。

另外，Node.js 安装目录下的 `node_modules` 目录通常不是用来存放模块的，而是用来存放与 Node.js 自身相关的一些核心模块（如 `fs`、`http` 等）。这些核心模块是 Node.js 运行时所必需的，它们被内置于 Node.js 引擎中，并在运行时直接提供给开发者使用。