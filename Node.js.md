### 概述

Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行时环境，用于服务器端和网络应用的开发。它允许开发者使用 JavaScript 编写后端代码，而不仅仅局限于前端开发。

### 模块安装

Node.js 一般使用 `npm` 包管理工具来安装必要的模块。

```bash
# 安装模块
npm install <package_name>

# 卸载模块（移除项目不再使用的依赖）
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

```bash
# 查看全局已安装模块
npm list -g
```

```
~]# npm list -g
C:\Users\dylan\AppData\Roaming\npm
`-- yarn@1.22.22
```

Yarn 是一个用于管理 JavaScript 项目依赖关系的包管理工具，由 Facebook、Google、Exponent 和 Tilde 联合开发。与 `npm` 类似，Yarn 允许你从 `npm` 仓库或者其他源（如 GitHub）安装、更新、卸载和管理 JavaScript 包。

另外，Node.js <u>安装目录</u>下的 `node_modules` 目录通常不是用来存放模块的，而是用来存放与 Node.js 自身相关的一些核心模块（如 `fs`、`http` 等）。这些核心模块是 Node.js 运行时所必需的，它们被内置于 Node.js 引擎中，并在运行时直接提供给开发者使用。

**如果使用 npm 初始化项目，后续管理依赖最好继续使用 npm**。这是前端工程化的最佳实践，原因如下：

不同包管理工具使用不同的锁定文件：

- npm 使用 `package-lock.json`
- yarn 使用 `yarn.lock`
- pnpm 使用 `pnpm-lock.yaml`

这些锁定文件虽然目的相同，但格式和解析算法不同。混用工具可能导致：

- **依赖解析不一致**：相同的 `package.json` 可能产生不同的依赖树
- **锁定文件冲突**：一个工具无法正确解析另一个工具的锁定文件
- **版本不匹配**：可能出现一些依赖版本差异

如果确实需要从 npm 切换到其他工具（如 yarn 或 pnpm），应该：

1. 删除原有的 `package-lock.json`
2. 删除 `node_modules` 目录
3. 使用新工具重新安装所有依赖，生成新的锁定文件
4. 通知团队成员统一使用新工具

**最佳做法**是在项目初期就确定使用哪种包管理工具，并在整个项目生命周期中保持一致。各工具间主要特点：

| 工具 | 优势                                    | 劣势                                     |
| ---- | --------------------------------------- | ---------------------------------------- |
| npm  | 无需额外安装，Node.js 自带              | 性能较慢，尤其在大型项目                 |
| yarn | 并行安装，离线缓存，性能好              | 需要单独安装                             |
| pnpm | 节省磁盘空间，性能最佳，monorepo 支持好 | 需要额外学习，可能与某些工具兼容性有问题 |

建议将使用的包管理工具记录在项目文档中，确保团队所有成员都了解并遵循相同的依赖管理方式。