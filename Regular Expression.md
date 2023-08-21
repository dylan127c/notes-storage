### 概述

正则表达式（REGEX, Regular Expression）又称规则表达式，在代码中常简写为 regex、regexp 或 RE 等，它是计算机科学的一个概念。

正则表达式通常被用来检索、替换那些符合某个模式（[规则](#规则列表)）的文本，它不仅限于某一种语言，但在每种语言中有细微的差别。可以在文末获取到大部分的正则表达式常用规则，但实际也推荐使用 ChatGPT 等方式来辅助快速生成符合要求的正则式。

出于避免过渡依赖 AI 工具，深入了解正则表达式是有必要的，至少在遇到复杂的正则式时，这些知识能够帮助你读懂它。

### 目的

使用正则表达式的目的通常只有几个：1. 文本计数；2. 文本替换；3. **文本筛选**。

例如，你想知道某篇英文文章中单词“the”的出现次数，那么你可以使用正则表达式 `\bthe\b` 来获取到该单词的出现次数。

许多文本处理工具如 Notepad++、VSCode 等，也都提供了使用正则表达式检索目标文本的功能，在目标文本检索出来后，可进一步将目标文本替换为其他的文本内容。

**程序中最常应用到的是文本筛选功能，例如用户提交的注册邮箱是否符合邮箱的标准格式，又或者用户提交的手机号码是否拥有完整的 11 位数字等。**

合理利用正则表达式可方便地规范化用户的注册流程，以减少不必要的资源损耗。

### 文本筛选

对于文本筛选来说，唯独有一种情况较为棘手，如果希望使用正则表达式筛选掉某些非法文本呢？遗憾的是，当非法文本量较多且格式较为多样时，正则表达式并能很好地支持这样的任务。

但如果想要筛选掉文本量并不多，那么可以换一种思维，即选择匹配需要筛选掉的非法文本，程序转而在正则匹配成功时，选择将非法文本剔除。

假设有以下文本，现在需要剔除所有包含“剩余流量”、“流媒体”字符的非法文本：

- `剩余流量：23GB`
- `香港 01`
- `香港 02 NTT`
- `香港 03 NTT IPv6`
- `香港 04 Cogent Ipv6（流媒体）`
- `香港 05 Cogent（流媒体）`

显然，很难定义一条规则将合法文本精准地筛选出来，那么此时可以选择筛选出非法文本：

- `剩余流量.*|香港[ \w]*（流媒体）`

程序内使用以上正则表达式，一旦成功匹配则将非法文本剔除，那么保留下来的即是合法文本：

```java
import java.util.ArrayList;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class RegexMain {
    public static void main(String[] args) {
        List<String> textList = new ArrayList<>();
        textList.add("剩余流量：23GB");
        textList.add("香港 01");
        textList.add("香港 02 NTT");
        textList.add("香港 03 NTT IPv6");
        textList.add("香港 04 Cogent Ipv6（流媒体）");
        textList.add("香港 05 Cogent（流媒体）");

        String regex = "剩余流量.*|香港[ \\w]*（流媒体）";
        Pattern pattern = Pattern.compile(regex);

        List<String> filteredTextList = new ArrayList<>();
        for (String text : textList) {
            Matcher matcher = pattern.matcher(text);
            // 注意：这里需要使用的是 matches 方法，文本替换或计数时需要使用 find 方法
            if (!matcher.matches()) {
                filteredTextList.add(text);
            }
        }

        for (String text : filteredTextList) {
            System.out.println(text);
        }
    }
}
```

输出如下：

```
香港 01
香港 02 NTT
香港 03 NTT IPv6
```

尝试将问题交由 ChatGPT 以生成匹配合法文本的正则表达式：

- `^(?!.*(?:剩余流量|流媒体)).*$`

可以看到，用于筛选合法文本的正则表达式中，仍旧需要提供关于非法文本的条件。

这实际上也是一种间接手段，即 ChatGPT 也认为不存在一种直接的方式，可以在不提供非法文本条件时，将所有的合法文本筛选出来。

```java
import java.util.ArrayList;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class RegexMain {
    public static void main(String[] args) {
        List<String> textList = new ArrayList<>();
        textList.add("剩余流量：23GB");
        textList.add("香港 01");
        textList.add("香港 02 NTT");
        textList.add("香港 03 NTT IPv6");
        textList.add("香港 04 Cogent Ipv6（流媒体）");
        textList.add("香港 05 Cogent（流媒体）");

        String regex = "^(?!.*(?:剩余流量|流媒体)).*$";
        Pattern pattern = Pattern.compile(regex);

        List<String> filteredTextList = new ArrayList<>();
        for (String text : textList) {
            Matcher matcher = pattern.matcher(text);
            if (matcher.matches()) {
                filteredTextList.add(text);
            }
        }

        for (String text : filteredTextList) {
            System.out.println(text);
        }
    }
}
```

输出结果与此前一致。

### 文本替换

正则表达式中有 MATCH 和 GROUP 两个概念。

- MATCH：文本中符合正则表达式的文本即为 MATCH；
- GROUP：任意的 MATCH 中 `(pattern)` 字符包裹的区域为 GROUP。

假设有以下文本：

- `The dog was the final winner. Yes, the dog is smart.`

使用以下正则表达式匹配：

- `(dog) ((?:wa|i)s)`

则会出现 2 次 MATCH 且每次 MATCH 中均存在 2 个 GROUP：

- MATCH(er)：`dog was`、`dog is`
  - MATCH(er)1-GROUP1：`dog`
  - MATCH(er)1-GROUP2：`was`
  - MATCH(er)2-GROUP1：`dog`
  - MATCH(er)2-GROUP2：`is`

进阶使用 Java 语言利用 MATCH 和 GROUP 的特性来完成文本的替换：

```java
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class RegexTest {

    @Test
    public void test() {
        String text = "The dog was the final winner. Yes, the dog is smart.";
        String regex = "(dog) ((?:wa|i)s)";

        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(text);

        StringBuilder result = new StringBuilder(text);
        int offset = 0;
        // 注意：这里需要使用的是 find 方法，不要和 matches 方法混淆
        while (matcher.find()) {
            String groupA = matcher.group(1);
            String groupB = matcher.group(2);

            Assertions.assertEquals("dog", groupA);
            Assertions.assertTrue("is".equals(groupB) || "was".equals(groupB));

            String replaceText = matcher.group(0).replace(groupA, "pi");

            result.replace(matcher.start() + offset, matcher.end() + offset, replaceText);
            offset += replaceText.length() - (matcher.end() - matcher.start());
        }
        Assertions.assertEquals("The pi was the final winner. Yes, the pi is smart.", result.toString());
    }
}
```

如果不存在 GROUP 即等同于仅存在 MATCH 的情况，这实际就是文本编辑软件实现文本替换的底层逻辑代码：

```java
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class RegexTest {

    @Test
    public void test() {
        String text = "The dog was the final winner. Yes, the dog is smart.";
        
        String regex = "\\bdog\\b";
        String replacementText = "cat";

        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(text);

        StringBuilder result = new StringBuilder(text);
        int offset = 0;
        while (matcher.find()) {
            String after = matcher.group(0).replace(matcher.group(0), replacementText);
            result.replace(matcher.start() + offset, matcher.end() + offset, after);
            offset += after.length() - (matcher.end() - matcher.start());
        }
        Assertions.assertEquals("The cat was the final winner. Yes, the cat is smart.", result.toString());
    }
}
```

Java 提供了更为简洁的 API 供文本替换使用：

```java
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class RegexTest {

    @Test
    public void test() {
        String text = "The dog was the final winner. Yes, the dog is smart.";

        String regex = "\\bdog\\b";
        String replacementText = "cat";

        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(text);

        Assertions.assertEquals(
                "The cat was the final winner. Yes, the cat is smart.",
                matcher.replaceAll(replacementText)
        );
    }
}
```

### 文本计数

简单实现：

```java
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class RegexTest {

    @Test
    public void test() {
        String text = "The dog was the final winner. Yes, the dog is smart.";

        String regex = "\\bdog\\b";

        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(text);

        int count = 0;
        while(matcher.find()) {
            count++;
        }
        Assertions.assertEquals(2, count);
    }
}
```

### 注意事项

Java 提供的正则表达式 API 中有两个关于 MATCH 的方法需要区分：

1. Matcher.matches 方法：如果可以使用正则表达式**完全匹配**到目标文本，则返回 True；
2. Matcher.find 方法：如果可以在目标文本中使用正则表达式**查询**到某些文本，则返回 True。

标准正则表达式中可能会包括 Java 语言的特殊字符，常见的有**双引号字符、反斜杠字符**等。只要标准的正则表达式中出现 Java 的特殊字符，那么 Java 使用这些正则表达式时，就必须使用反斜杠对特殊字符进行转义。

例如，正则表达式中用于匹配字母、数字、下划线的 `\w` 字符，该字符如果使用 Java 内的正则表达式应书写为 `\\w`；标准正则表达式中使用 `\\` 匹配反斜杠字符，其中首个反斜杠用于转义，此时转换为 Java 内的正则表达式，则需要书写为 `\\\\` 的形式。

假设需要写出匹配 `"do\g6"` 字符的正则表达式：

- Regular Expression：`"do\\g\d"` 

因为**双引号字符、反斜杠字符**在 Java 中属于特殊字符，那么在写入正则表达式时就需要转义，即简单粗暴地添加反斜杠字符：

- Java Regular Expression：`\"do\\\\g\\d\"`

### 规则列表

[REGEX101](https://regex101.com/) 提供了用于校验正则表达式的在线工具，所有必要的测试都可以在 [REGEX101](https://regex101.com/) 上完成。

以下匹配规则将简单区分为常用规则、其他规则和高级规则。

#### 常用规则

| Token |                         Description                          |
| :---: | :----------------------------------------------------------: |
|  `\`  | 将下一个字符标记为一个特殊字符、或一个原义字符、或一个向后引用、或一个八进制转义符。<br/>例如，`n` 匹配字符 `n`；`\n` 匹配一个换行符；序列 `\\` 匹配 `\`，而 `\(` 则匹配 `(`。 |
|  `^`  | 匹配输入字符串的开始位置。<br/>如果设置了 RegExp 对象的 Multiline 属性，`^` 也匹配 `\n` 或 `\r` 之后的位置。 |
|  `$`  | 匹配输入字符串的结束位置。<br/>如果设置了RegExp 对象的 Multiline 属性，`$` 也匹配 `\n` 或 `\r` 之前的位置。 |

|  Token  |                         Description                          |
| :-----: | :----------------------------------------------------------: |
|   `*`   | 匹配前面的子表达式零次或多次。<br/>例如，`zo*` 能匹配 `z` 以及 `zoo`。`*` 等价于 `{0,}`。 |
|   `+`   | 匹配前面的子表达式一次或多次。<br/>例如，`zo+` 能匹配 `zo` 以及 `zoo`，但不能匹配 `z`。`+` 等价于 `{1,}`。 |
|   `?`   | 匹配前面的子表达式零次或一次。<br/>例如，`do(es)?` 可以匹配 `do` 或 `does`。`?` 等价于 `{0,1}`。 |
|  `{n}`  | `n` 是一个**非负整数**。匹配确定的 `n` 次。<br/>例如，`o{2}` 不能匹配 `Bob` 中的 `o`，但是能匹配 `food` 中的 `oo`。 |
| `{n,}`  | `n` 是一个非负整数。至少匹配 `n` 次。<br/>例如，`o{2,}` 不能匹配 `Bob` 中的 `o`，但能匹配 `foooood` 中的 `ooooo`。实际上，`o{1,}` 等价于 `o+`，`o{0,}` 则等价于 `o*`。 |
| `{n,m}` | `m` 和 `n` 均为非负整数，其中 `n <= m`。最少匹配 `n` 次且最多匹配 `m` 次。<br/>例如，`o{1,3}` 将匹配 `fooooood` 中的 `ooo` 并得到 `2` 个匹配结果。实际上，`o{0,1}` 等价于 `o?`。注意，逗号和两个数之间不能有空格。 |

| Token |                         Description                          |
| :---: | :----------------------------------------------------------: |
|  `?`  | 当该字符紧跟在任何一个其他限制符 `*`、`+`、`?`、`{n}`、`{n,}`、`{n,m}`、`(pattern)` 后面时，匹配模式是非贪婪的。**非贪婪模式**尽可能少的匹配所搜索的字符串，而**默认的贪婪模式**则尽可能多的匹配所搜索的字符串。<br/>例如，对于字符串 `oooo`，`o+?` 将匹配单个 `o` 并得到 `4` 个匹配结果，而 `o+` 将匹配所有 `o` 并只得到 `1` 个匹配结果。 |

| Token |                         Description                          |
| :---: | :----------------------------------------------------------: |
|  `.`  | 匹配除换行符 `\n`、`\r` 之外的任何单个字符。要匹配包括 `\n` 在内的任何字符，请使用像 <code>(.\|\n)</code> 的模式。 |
| `\w`  |       匹配字母、数字、下划线。等价于 `[A-Za-z0-9_]`。        |
| `\W`  |      匹配非字母、数字、下划线。等价于 `[^A-Za-z0-9_]`。      |
| `\d`  |              匹配一个数字字符。等价于 `[0-9]`。              |
| `\D`  |            匹配一个非数字字符。等价于 `[^0-9]`。             |
| `\s`  | 匹配任何空白字符，包括空格、制表符、换页符等等。等价于 `[ \f\n\r\t\v]`。 |
| `\S`  |        匹配任何非空白字符。等价于 `[^ \f\n\r\t\v]`。         |

|       Token       |                         Description                          |
| :---------------: | :----------------------------------------------------------: |
|    `(pattern)`    | 匹配 pattern 并获取这一匹配。所获取的匹配可以从产生的 Matches 集合得到，在 VBScript 中使用 SubMatches 集合，在JScript 中则使用 $0…$9 属性。<br/>要匹配圆括号字符，请使用 `\(` 或 `\)`。 |
| <code>x\|y</code> | 匹配 `x` 或 `y`。<br/>例如，<code>z\|food</code> 能匹配 `z` 或 `food`。<code>(z\|f)ood</code> 则匹配 `zood` 或 `food`。 |
|      `[xyz]`      | 字符集合。匹配所包含的任意一个字符。<br/>例如，`[abc]` 可以匹配 `plain` 中的 `a`。 |
|     `[^xyz]`      | 负值字符集合。匹配未包含的任意字符。<br/>例如，`[^abc]` 可以匹配 `plain` 中的 `p`、`l`、`i`、`n`。 |
|      `[a-z]`      | 字符范围。匹配指定范围内的任意字符。<br/>例如，`[a-z]` 可以匹配 `a` 到 `z` 范围内的任意小写字母字符。 |
|     `[^a-z]`      | 负值字符范围。匹配任何不在指定范围内的任意字符。<br/>例如，`[^a-z]` 可以匹配任何不在 `a` 到 `z` 范围内的任意字符。 |

#### 其他规则

| Token |                         Description                          |
| :---: | :----------------------------------------------------------: |
| `\b`  | 匹配一个单词边界，也就是指单词和空格间的位置。<br/>例如，`er\b` 可以匹配 `never` 中的 `er`，但不能匹配 `verb` 中的 `er`。 |
| `\B`  | 匹配非单词边界。<br/>例如，`er\B` 能匹配 `verb` 中的 `er`，但不能匹配 `never` 中的 `er`。 |
| `\xn` | 匹配 `n`，其中 `n` 为十六进制转义值。十六进制转义值必须为确定的两个数字长。<br/>例如，`\x41` 匹配 `A`。`\x411` 则等价于 `A1`。正则表达式中可以使用 `ASCII` 编码。 |
| `\cx` | 匹配由 `x` 指明的控制字符。例如，`\cM` 匹配一个 `Control-M` 或回车符。`x` 的值必须为 `A-Z` 或 `a-z` 之一。否则，将 `c` 视为一个原义的 `c` 字符。 |
| `\f`  |           匹配一个换页符。等价于 `\x0c` 和 `\cL`。           |
| `\n`  |           匹配一个换行符。等价于 `\x0a` 和 `\cJ`。           |
| `\r`  |           匹配一个回车符。等价于 `\x0d` 和 `\cM`。           |
| `\t`  |           匹配一个制表符。等价于 `\x09` 和 `\cI`。           |
| `\v`  |         匹配一个垂直制表符。等价于 `\x0b` 和 `\cK`。         |

#### 高级规则

|     Token      |                         Description                          |
| :------------: | :----------------------------------------------------------: |
| `(?:pattern)`  | 匹配 `pattern` 但不获取匹配结果，也就是说这是一个**非获取匹配**，不进行存储供以后使用。这在使用或字符 <code>\|</code> 组合一个模式的各个部分时很有用。<br/>例如，<code>industr(?:y\|ies)</code> 等价于 <code>industry\|industries</code> 但前者更简洁。 |
| `(?=pattern)`  | 正向肯定预查（Look Ahead Positive Assert），在任何匹配 `pattern` 的字符串开始处匹配查找字符串。这是一个**非获取匹配**。<br/>例如，<code>Windows(?=95\|98\|NT\|2000)</code> 能匹配 `Windows2000` 中的 `Windows`，但不能匹配 `Windows3.1` 中的 `Windows`。<br/>**<u>预查不消耗字符，即下一次匹配不会从预查字符之后开始</u>。**<br/>例如，<code>Windows(?=95\|Windows)</code> 能匹配 `WindowsWindows95` 中的所有 `Windows` 并得到 `2` 个匹配结果。 |
| `(?!pattern)`  | 正向否定预查（Look Ahead Negative Assert），在任何不匹配 `pattern` 的字符串开始处匹配查找字符串。这是一个**非获取匹配**。<br/>例如，<code>Windows(?!95\|98\|NT\|2000)</code> 能匹配 `Windows3.1` 中的 `Windows`，但不能匹配 `Windows2000` 中的 `Windows`，**<u>预查不消耗字符</u>**。 |
| `(?<=pattern)` | 反向肯定预查（Look Behind Positive Assert），与正向肯定预查类似，只是方向相反。<br/>例如，<code>(?<=95\|98\|NT\|2000)Windows</code> 能匹配 `2000Windows` 中的 `Windows`，但不能匹配 `3.1Windows` 中的 `Windows`。 |
| `(?<!pattern)` | 反向否定预查（Look Behind Negative Assert），与正向否定预查类似，只是方向相反。<br/>例如，<code>(?<!95\|98\|NT\|2000)Windows</code> 能匹配 `3.1Windows` 中的 `Windows`，但不能匹配 `2000Windows` 中的 `Windows`。 |

