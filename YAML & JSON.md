请安装 Node.js 环境，并将包含 node.exe 程序的安装目录，添加到环境变量中。

同时 VSCode 需要安装 Code Runner 扩展，以使用 JS 调试程序。

为了获取更好的终端兼容性，建议启用 Code Runner 以下配置：

```json
"code-runner.runInTerminal": true
```

NODE_PATH

### 全局安装：

全局安装的包通常安装在全局 Node.js 模块文件夹中，该文件夹的路径可能因操作系统而异。在大多数系统中，全局 Node.js 模块文件夹的默认位置是：

- **Windows:** `C:\Users\<username>\AppData\Roaming\npm\node_modules`
- **macOS/Linux:** `/usr/local/lib/node_modules`

当你在命令行中运行 `npm install -g <package>` 时，该包将被安装到全局 Node.js 模块文件夹中。

### 非全局安装：

非全局安装的包通常安装在当前项目的 `node_modules` 文件夹中。这个文件夹会被创建在运行 `npm install <package>` 命令的目录下。例如：

```
plaintextCopy code/your_project
|-- node_modules
|   |-- <package>
|-- package.json
|-- ...
```

包和其依赖项将被安装到 `node_modules` 文件夹中。

请注意，全局安装通常用于安装命令行工具或全局可用的库，而非全局安装通常用于项目特定的依赖项。在项目中使用非全局安装有助于确保项目的依赖项的版本和环境隔离。



- KEY 只能是字符串；
- VAL 只能是字符串、数字、JS 对象及它们所对应的数组。



JSON 格式的文件中，字符串必须使用双引号包裹起来，数组则允许直接写入，同时 JSON 格式文件不允许注释；

```json
{
    "type": "T-Shirt",
    "price": 20.00,
    "sizes": [
        "S",
        "M",
        "L"
    ],
    "reviews": [
        {
            "username": "user1",
            "rating": 4,
            "created_at": "2023-04-19T12:30:00Z"
        },
        {
            "username": "user2",
            "rating": 5,
            "created_at": "2023-05-02T15:00:00Z"
        }
    ]
}
```

YAML 格式的文件中，所有的元素都不强制使用双引号包裹，同时 YAML 格式文件允许注释。

```yaml
type: T-Shirt
price: 20.00
sizes: # ALLOW COMMENT
  - S
  - M
  - L
reviews:
  - username: user1
    rating: 4
    created_at: 2023-04-19T12:30:00Z
  - username: user2
    rating: 5
    created_at: 2023-05-02T15:00:00Z
```

