### 概述

本篇简单梳理两种 JS 常用的对象序列化格式 YAML 和 JSON。

### 版本环境

- Windows 11 23H2 22631.3007
- Visual Studio Code 1.87.1 (user setup)
- Node.js 20.11.1

### VSCode

本篇需要使用到 VSCode 软件，同时还需要使用 Code Runner 插件。该插件只能在受信任的工作区中使用，这里推荐在设置中关闭工作区信任：

```json
"security.workspace.trust.enabled": false
```

<div align="center"><img src="images/YAML%20&%20JSON.images/Snipaste_2024-03-11_15-47-44.png" alt="Snipaste_2024-03-11_15-47-44" style="width:80%;" /></div>

为了获取更好的终端兼容性，推荐启用 Run In Terminal 配置：

```json
"code-runner.runInTerminal": true
```

<div align="center"><img src="images/YAML%20&%20JSON.images/Snipaste_2024-03-11_15-56-31.png" alt="Snipaste_2024-03-11_15-56-31" style="width:80%;" /></div>

### 格式示例

标准的 JSON 格式：

- 字符串必须使用双引号包裹；
- 数组使用方括号包裹；
- 文档不允许注释。

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

标准的 YAML 格式：

- 所有类型元素都不强制使用双引号包裹；
- 数组使用短横杠标注；
- 文档允许注释。

```yaml
type: T-Shirt
price: 20.00
sizes: # ALLOW COMMENT
  - S
  - M
  - L
reviews: # 同缩进的、连续的短横杠所标注的元素，都同属于一个数组中
  - username: user1
    rating: 4
    created_at: 2023-04-19T12:30:00Z
  - username: user2
    rating: 5
    created_at: 2023-05-02T15:00:00Z
```

YAML 更注重可读性和人类友好性，适合用于配置文件和复杂数据结构的表示；JSON 则更注重简洁性和易用性，适合于数据交换和存储。在选择使用哪种格式时，可以根据具体的需求和场景来决定。

### 数据序列化

YAML（YAML Ain't Markup Language）和 JSON（JavaScript Object Notation）都是用于数据序列化的文本格式，通常在软件开发中用于配置文件、数据存储和 API 通信等方面。

读取 YAML 文本为对象：

```javascript
import { readFileSync } from "fs";
import { resolve, dirname } from "path";
import { fileURLToPath } from 'url';
import { parse } from "yaml";

const filepath = fileURLToPath(import.meta.url);
const foldername = dirname(filepath);

// 1.读取文件
const read_yaml = readFileSync(resolve(foldername, "./sample.yaml"), "utf8");
// 2.将文本转换为对象
const result = parse(read_yaml);
console.log(result);
```

读取 JSON 文件为对象：

```javascript
// JSON 文件可以直接被读取成对象
import read_json from "./sample.json" assert { type: "json" };
console.log(read_json);
```

使用各种编程语言中的库或者在线工具可以将 YAML 转换为 JSON，反之亦然。例如，常见的一些工具有 `yaml` 和 `json` 命令行工具，以及各种编程语言中的库和函数。

YAML 转换为 JSON：

```javascript
import { readFileSync, writeFileSync } from "fs";
import { resolve, dirname } from "path";
import { fileURLToPath } from 'url';
import { parse } from "yaml";

const filepath = fileURLToPath(import.meta.url);
const foldername = dirname(filepath);

const read_yaml = readFileSync(resolve(foldername, "./sample.yaml"), "utf8");
const result = parse(read_yaml);

// 使用 JSON.stringify 方法将对象序列化为 JSON 格式文本
const output_json = JSON.stringify(result);
writeFileSync(resolve(foldername, "./sample_convert.json"), output_json, "utf-8");
```

JSON 转换为 YAML：

```javascript
import { writeFileSync } from "fs";
import { resolve, dirname } from "path";
import { fileURLToPath } from 'url';
import { stringify } from "yaml";

const filepath = fileURLToPath(import.meta.url);
const foldername = dirname(filepath);

import read_json from "./sample.json" assert { type: "json" };

// 使用 yaml 模块提供的 stringify 方法将对象序列化为 YAML 格式文本
const output_yaml = stringify(read_json);
writeFileSync(resolve(foldername, "./sample_convert.yaml"), output_yaml, "utf8");
```

### 其他事项

YAML 序列化文本所允许包含的数据类型相较于 JSON 来说更多，或者说 JSON 所支持序列化的数据类型没有 YAML 所支持得多多，例如 `Infinity` 和 `NaN` 等数据类型。

`Infinity` 和 `NaN` 是 JavaScript 中的特殊数值，用于表示无穷大和非数值（Not a Number）。如果将包含这些特殊数值的对象序列化为 JSON 文本，那么这些特殊数值将丢失，且会统一被序列化为 `null` 值。

另外，在 Node.js 中使用 ES6 直接导入 JSON 模块时，可能会收到实验性警告：

```
(node:15468) ExperimentalWarning: Importing JSON modules is an experimental feature and might change at any time
(Use `node --trace-warnings ...` to show where the warning was created)
```

这个警告是 Node.js 团队为了向开发者提供关于这个功能的信息，以及提醒开发者该功能可能不稳定或者在未来的版本中可能会发生变化而发出的。

虽然导入 JSON 模块已经成为了标准的 Node.js 功能，但是 Node.js 团队仍然在继续改进和优化这个功能，所以它仍然被标记为实验性的，并且在未来可能会有一些变化。
