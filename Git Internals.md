### 概述

本篇参考 [Pro Git Book](https://git-scm.com/book/en/v2) 官方文档中的 [Git Internals](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain) 章节，并结合 ChatGPT 语言模型，整理并总结了一些关于 Git 内部工作原理的资料，这些资料能够帮助深入了解 Git 的内部构成。

### 数据存储

虽然 Git 的主要目的是跟踪和管理文件的版本历史，但它的实现方式在很大程度上参考了数据库的概念和技术。Git 内部使用了一种被称为对象数据库（Object Database）的结构来存储和管理数据。

对象数据库是一种键-值存储系统，其中：

- 键（key）：通常是对象的 SHA-1 校验和（checksum）；
- 值（value）：对象数据压缩之后得到的数据文件。

Git 的对象数据库位于 .git/objects 目录，其具备一些数据库的特性：

1. 持久性存储：所有数据都会持久地存储在磁盘上，以确保不会丢失；
2. 哈希索引：使用对象的 SHA-1 校验和作为索引键，以快速查找和访问对象数据；
3. 数据完整性：使用 SHA-1 哈希算法对对象进行校验，确保数据的完整性和一致性；
4. 分布式复制：允许多个代码库之间进行数据的复制和同步，每个库都可以作为完整的副本。

相较于对象数据库，Git 更专注于版本控制和跟踪文件的变化，而不是提供广泛的查询和操作数据的功能。Git 仅在某种程度上与数据库的特性和概念有关联，所以通常不能将 Git 直接描述为数据库。

### 对象类型

Git 中存在四种类型的对象：

1. Blob 对象：该对象用来存储文件数据，所有添加到暂存区的文件，都会被存储成 Blob 对象；
2. Tree 对象：该对象用于记录目录内容，即目录中有哪些 Blob 对象和 Tree 对象；
3. Commit 对象：该对象是仓库在某一时间点的快照，它必然包含一个表示根目录的 Tree 对象，还包含一些关于时间、作者、提交信息和指向上一次 Commit 对象的指针等；
4. Tag 对象：该对象用于标记 Commit 对象。

#### Blob

Blob（Binary Large Object）即二进制类型的大对象。通常在数据库管理系统中，会将二进制数据存储为一个单一个体的集合，即 Blob 对象。

Git 使用 Blob 来存储文件内容，其大致结构如下图所示：

<img src="images/Git%20Internals.images/object-blob.png" alt="img" style="zoom:80%;" />

其中首行被称为 Header 头部信息，其余部分即为 Content 文件内容。

Header 头部信息由以下字段组成：

- 类型（Type）：对象类型，对于 Blob 对象，该字段为字符串“blob”；
- 大小（Size）：文件内容的大小，以字节（byte）为单位。

Header 的字段之间一般使用空格或制表符进行分隔，以下是 Header 的书写格式：

```
blob content.bytesize\u0000
```

可以看到，类型和大小之间使用空格分割：

```
Type |---- Size ----|
blob content.bytesize
```

以下是 Blob 对象的内部格式：

```
blob content.bytesize\u0000content
```

可以看到，Header 和 Content 之间使用空字符 NULL 分割：

```
|------ Header -----||NULL|Content
blob content.bytesize\u0000content
```

假如仓库中有任意采用 UTF-8 字符集编码的文件，其内容如下：

```
Hello
```

那么该文件所对应的 Blob 对象即为：

```
blob 5\u0000Hello
```

需要注意，以上的 Blob 对象内容是经 UTF-8 字符集编码后所得到的可读内容，Blob 对象本身是二进制类型大对象，这表明了所有存储其中的数据都是经解码后的二进制数据，而非可读数据。

实际上，Git 会分别获取到 Header 和 Content 的二进制数据，通过拼接的方式将两部分的数据组合起来，以共同构成 Blob 对象。

在将对象存储到对象数据库前，Git 还需要对对象进行进一步的处理。鉴于对象数据库是键-值存储系统，因此需要借用到哈希算法、压缩算法来分别获取对象的键、值：

1. 哈希算法：根据对象内容来计算校验和，该校验和即为该对象的唯一索引键；
2. 压缩算法：对象数据文件一般需要使用压缩算法来减少其数据的大小，以节省磁盘空间。

Git 中常用的哈希算法为 SHA-1 算法，常用的压缩算法为 zlib 提供的 DEFLATE 算法。

以下是文件被存储到对象数据库的大致过程：

<img src="images/Git%20Internals.images/image-20230528030705076.png" alt="image-20230528030705076" style="zoom:50%;" />

举个例子，可以在 C:\Windows\System32 目录下找到 notepad.exe 文件，并将其复制到一个空仓库中。或在空仓库根目录下，使用 Git Bash 执行以下命令复制并粘贴 notepad.exe 文件到当前目录：

```bash
cp /c/Windows/System32/notepad.exe .
```

将该文件添加到暂存区（git add）后，对象数据库 .git/objects 中随即会出现压缩后的 Blob 对象数据文件。查看对象数据库中的内容：

```bash
find .git/objects -type f
```

可以看到 Blob 对象的数据文件：

```
.git/objects/b2/bee6b97f09bf1202647a0ed35ef31f6529d7cc
```

根据上述图解，可以知道该 Blob 对象的索引键（校验和）为：

- b2bee6b97f09bf1202647a0ed35ef31f6529d7cc

如果希望验证 Blob 对象格式的正确性，可以借用程序来实现：

1. 构建相同的 Blob 对象；
2. 使用 SHA-1 算法对 Blob 对象执行哈希运算，以获取校验和；
3. 验证程序产出的校验和与 Git 的校验和是否一致。一致即表明 Blob 对象格式无误；否则有误。

以下是使用 Java 实现的 SHA-1 哈希算法：

```java
public String sha1(byte[] data) throws NoSuchAlgorithmException {
    MessageDigest md = MessageDigest.getInstance("SHA1");
    md.update(data);

    StringBuilder buf = new StringBuilder();

    byte[] bits = md.digest();
    for (int bit : bits) {
        int a = bit;
        if (a < 0) a += 256;
        if (a < 16) buf.append("0");
        buf.append(Integer.toHexString(a));
    }
    return buf.toString();
}
```

测试程序会从 Git 仓库中读取 notepad.exe 文件的字节码数据，并使用这些数据来构建 Header 头部信息。后续会将 Header 和 Content 的字节码数据拼接并交由 SHA-1 方法计算校验和：

```java
public class ProgramSHA1Test {

    @Test
    public void test() throws NoSuchAlgorithmException {
        File file = new File("F:\\Desktop\\some\\notepad.exe");
        try (FileInputStream fis = new FileInputStream(file)) {
            byte[] bytesContent = fis.readAllBytes();

            String header = "blob " + bytesContent.length + "\u0000";
            byte[] bytesHeader = header.getBytes();

            byte[] blob = new byte[bytesHeader.length + bytesContent.length];
            System.arraycopy(bytesHeader, 0, blob, 0, bytesHeader.length);
            System.arraycopy(bytesContent, 0, blob, bytesHeader.length, bytesContent.length);

            String sha1Value = sha1(blob);
            String expectValue = "b2bee6b97f09bf1202647a0ed35ef31f6529d7cc";

            Assertions.assertEquals(expectValue, sha1Value);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

    }
    
    public String sha1(byte[] data) throws NoSuchAlgorithmException {
        MessageDigest md = MessageDigest.getInstance("SHA1");
        md.update(data);

        StringBuilder buf = new StringBuilder();

        byte[] bits = md.digest();
        for (int bit : bits) {
            int a = bit;
            if (a < 0) a += 256;
            if (a < 16) buf.append("0");
            buf.append(Integer.toHexString(a));
        }
        return buf.toString();
    }
}
```

测试通过，这表明前文给出的 Blob 对象格式是准确无误的。另外，了解了如何计算文件的校验和后，基本上可以自行设计出一个简单的对象数据库了。

Git 使用哈希算法计算对象索引键（校验和）的方式不仅用在 Blob 对象上，后续提及的其他 Git 对象也都是使用同样的方式来获取其唯一的索引键。

大致了解了 Blob 对象后，能够发现 Blob 对象仅是单纯的二进制数据，或者说它仅记录文件的二进制数据，除此之外，它没有指向任何其他的对象或拥有其他属性（甚至文件名都没有）。

这意味着，如果有两个数据完全相同的文件，那它们将会共享同一个 Blob 对象。因为 Blob 对象只关心文件的内容，Blob 对象和文件所在路径、文件名或文件类型等均没有任何关系。

#### Tree

一种较为直接的理解是将 Tree 对象看作目录，目录中包含文件和其他目录的信息。

<img src="images/Git%20Internals.images/object-tree.png" alt="img" style="zoom: 80%;" />

Tree 对象由一系列由空格或制表符分隔的条目组成，每个条目描述一个文件或目录：

```
Mode Type Hash FileName
```

其中，各字段的含义如下：

- Mode（[权限模式](#权限模式)）：表示文件或目录的权限模式，通常为八进制数值。例如：
  - 普通文件的权限模式为 100644；
  - 目录的权限模式为 040000。
- Type（对象类型）：表示条目所指向的对象类型。例如：
  - 对于文件，其类型为 blob；
  - 对于目录，其类型为 tree；
- Hash（对象索引键）：表示条目所指向的对象的哈希值；
- FileName（文件或目录名称）：表示文件或目录的名称。

例如，一个包含三个条目的 Tree 对象可以是这样的：

```
040000 tree 85ae61036aec44282c742f80a76605e3d142dcb4    else
100644 blob 5dd01c177f5d7d1be5346a5bc18a569a7410c2ef    first
100644 blob b2bee6b97f09bf1202647a0ed35ef31f6529d7cc    notepad.exe
```

其中 FileName 部分即为 Tree 对象的 content，其他部分则视为 header。



#### Commit

<img src="images/Git%20Internals.images/object-commit.png" alt="img" style="zoom:80%;" />

#### Tag



### 关键算法

讲述关键算法前，需要大致了解 Git 中的三个主要对象：

1. Blob 对象：该对象可理解为仓库文件，它包含了文件数据信息；
2. Tree 对象：该对象可理解为目录，它引用了其他的 Blob 对象和 Tree 对象；
3. Commit 对象：该对象可以理解为仓库在某个时间点的快照，它包含了根目录 Tree 对象、指向上一次 Commit 对象的指针、时间及提交者等信息。



Git 中的对象数据库需要使用到两种算法：

1. 哈希算法：根据对象来计算其校验和，以得到唯一是索引键；
2. 压缩算法：对象数据一般需要使用压缩算法来减少其数据的大小，以节省磁盘空间。

Git 的对象数据库中，根据对象内容计算得到的为键，对象本身压缩得到的数据即为值。

以下是 Blob 对象在 Git 中被存储到对象数据库的大致过程：

<img src="images/Git%20Internals.images/image-20230528030705076.png" alt="image-20230528030705076" style="zoom:50%;" />

可以看到 Blob 对象实际就是由头部信息（header）和文件内容（content）共同组成，但本节的重点并非 Git 对象的构成。关于 Git 对象的更多细节，后续会进行说明。

以上过程中明显涉及了两种算法：1. SHA-1 哈希算法；2. DEFLATE 压缩算法。哈希算法在计算索引键时使用，而压缩算法则在为对象本身执行压缩时使用。

#### 哈希算法

Git 默认使用 SHA-1（Secure Hash Algorithm 1）计算对象的哈希值，除此之外，Git 支持通过配置的方式来使用其他类似的哈希算法。

计算对象哈希值的过程如下：

1. 获取对象内容：根据对象的类型，获取相应的内容。例如：
   - 对于 Blob 对象，包含头部信息和文件内容；
   - 对于 Tree 对象，包含头部信息、文件和子树信息的结构；
   - 对于 Commit 对象，包含头部信息、作者、提交者、树对象哈希值和提交消息等信息的结构。
3. 计算哈希值：对内容进行哈希计算。Git 一般使用 SHA-1 算法对内容进行处理，以生成一个固定长度的哈希值（SHA-1 算法生成的哈希值为 40 个字符的十六进制字符串）。

由于哈希算法的特性，不同的对象内容会产生长度一致但互不相同的哈希值。因此，不同的 Git 对象基本必然具有唯一标识符。

值得注意的是，Git 近期的版本已经逐渐开始采用更强大的哈希算法，例如 SHA-256，以提供更高的安全性。但经实测，在目前最新的 2.40.0 版本中，Git 仍默认采用 SHA-1 算法。

#### 压缩算法

Git 默认使用 zlib 库进行数据的压缩或解压缩。zlib 是一个流行的开源压缩库，提供了使用 Deflate 压缩算法进行数据压缩或解压缩的函数。

虽然 Git 内置了 zlib 库，但并非必须对数据进行压缩处理，Git 同样支持通过配置的方式来禁用压缩，或使用其他的压缩算法。

在 Git 中 zlib 不仅提供对象压缩的功能，其具体被引用于以下几部分：

1. 对象压缩：Git 使用 zlib 对其对象（如 blob 和 tree）进行压缩，以减小存储大小。当对象存储在 Git 仓库中时，使用 zlib 的 Deflate 算法对对象进行压缩，从而节省磁盘空间；
2. 网络传输：当 Git 在网络上传输数据（如克隆、拉取或推送）时，可以使用 zlib 对数据进行压缩，以减少传输的数据量。这种压缩有助于提高网络效率并减少传输时间；
3. Packfile 压缩：Git 还使用 zlib 对 packfile 进行压缩。Packfile 是存储在压缩格式中的 Git 对象集合，用于高效存储和传输。zlib 负责对 packfile 进行压缩和解压缩。

但实际上，了解 Git 的存储原理只需要知道 zlib 提供对象压缩功能即可。通过使用 zlib 库，Git 可以有效减少存储库对磁盘空间的需求。

### 简单计算

本节会直接使用程序将 Blob 对象构建出来，并使用 SHA-1 算法计算它的校验和。为了验证该校验和的正确性，需要先获取到指定内容在 Git 对象数据库中的索引键（校验和）。

以下是 Blob 对象在 Git 中被存储到对象数据库的大致过程：

<img src="images/Git%20Internals.images/image-20230528030705076.png" alt="image-20230528030705076" style="zoom:50%;" />

根据 Blob 对象的存储流程，不难得知能够在 .git/objects 目录中获取到目标对象的索引键（校验和）。在 Git 中生成指定数据的索引键，只需要将数据添加至暂存区（git add）即可。

简单来说，只要将包含指定内容的文件添加至暂存区后，就能在 .git/objects 目录中获取到索引键。

以下命令可用于打印 .git/objects 目录内容：

```bash
find .git/objects -type f
```

有了获取索引键的方式后，下一步就是在程序中使用同样的内容构建出 Blob 对象了。

以 Blob 对象为例，其包含的数据格式为（其中 \u0000 为空字符）：

```
blob content.bytesize\u0000content
```

其中头部信息为：

```
blob content.bytesize\u0000
```

以字符串”hello“为例，其 Blob 对象可以表示为：

```
blob 5\u0000hello
```

显然内容为字符串时构建 Blob 对象十分简单，那么最后一步就是实现 SHA-1 哈希算法了。

以下是使用 Java 实现的 SHA-1 算法：

```java
public String sha1(String data) throws NoSuchAlgorithmException {
        MessageDigest md = MessageDigest.getInstance("SHA1");
        md.update(data.getBytes());

        StringBuilder buf = new StringBuilder();

        byte[] bits = md.digest();
        for (int bit : bits) {
            int a = bit;
            if (a < 0) a += 256;
            if (a < 16) buf.append("0");
            buf.append(Integer.toHexString(a));
        }
        return buf.toString();
    }
```

感兴趣可以自行了解 SHA-1 哈希算法的实现原理，不再赘述。

万事具备后，只需要证明通过构建的方式能够获取到的与索引键一致的校验和，即可说明本节中所使用的构建 Blob 对象的方式与 Git 的内部所使用的构建 Blob 对象的原理一致。

#### 获取索引键

初始化任意空仓库，添加内容为“Hello, world!”的任意文件，并将其添加到暂存区（建议复制字符）：

```bash
echo -n "Hello, world!" > first && git add first
```

这里 \-n 选项表示将输出重定向到文件时不添加额外的换行符。默认情况下使用 Git Bash 将输出重定向到文件时，会使用 UTF-8 字符集（非 Git Bash 则可能使用其他编码）。

注意，需要重点关注输出文件所使用的字符集（编解码）！因为输出重定向到文件后，若采用的字符集有所不同，即便输出内容相同，最终计算得到的数据索引键（SHA-1 校验和）也会不同。

查看 .git/objects 目录内容：

```bash
find .git/objects -type f
```

终端（必然）输出以下内容：

```
.git/objects/5d/d01c177f5d7d1be5346a5bc18a569a7410c2ef
```

再次添加内容为“你好，世界！”的任意文件，并将其提交到暂存区：

```bash
echo -n "你好，世界！" > second && git add second
```

再次查看 .git/objects 目录内容，终端（必然）输出以下内容：

```
.git/objects/5d/d01c177f5d7d1be5346a5bc18a569a7410c2ef
.git/objects/ca/94ea3b652074560b6a7d530f8187323982518a
```

综上可以初步得出一些结论：

1. 内容为“Hello, world!”的文件，其所对应的 Blob 对象的索引键（校验和）为：
   - 5dd01c177f5d7d1be5346a5bc18a569a7410c2ef
2. 内容为“你好，世界！”的文件，其所对应的 Blob 对象的索引键（校验和）为：
   - ca94ea3b652074560b6a7d530f8187323982518a

题外话，以下命令可以查看 Git 对象的具体类型：

```bash
git cat-file -t <object>
```

以下命令可以查看 Git 对象的具体内容：

```bash
git cat-file -p <object>
```

例如上述索引键位 5dd01c1 的 Blob 对象，查看其具体类型的命令为：

```bash
git cat-file -t 5dd01c1 ## OUTPUT => blob
```

查看其具体内容的命令为（不会输出 header 头部信息，类似于 Java 类中的 toString 方法）：

```bash
git cat-file -p 5dd01c1 ## OUTPUT => Hello, world!
```

#### 获取校验和

已知 Blob 对象内的数据格式如下：

```
blob content.bytesize\u0000content
```

那不难在程序中构建出指定内容的 Blob 对象。

字符串“Hello, world!”所对应的 Blob 对象为：

```
blob 13\u0000Hello, world!
```

字符串“你好，世界！”所对应的 Blob 对象为：

```
blob 18\0000你好，世界！
```

这里只需要编写一个简单的测试类，测试使用目标内容构建而成的 Blob 对象经 SHA-1 哈希算法得到的校验和是否与期望的校验和一致即可。

注意，代码中需要使用 UTF-8 字符集对数据进行编码（实际 String.getBytes 方法默认使用 UTF-8 字符集），因为此前使用 Git Bash 将输出重定向到文件时使用的也是 UTF-8 字符集。

具体的测试代码如下：

```java
public class SHA1Test {

    /**
     * Git中用于计算Blob对象的SHA-1校验和公式：blob content.bytesize\u0000content
     *
     * @throws NoSuchAlgorithmException 算法不存在异常
     */
    @Test
    public void test() throws NoSuchAlgorithmException {
        String type = "blob";

        String contentA = "Hello, world!";
        String storeA = type + " " + contentA.getBytes(StandardCharsets.UTF_8).length + "\u0000" + contentA;

        String contentB = "你好，世界！";
        String storeB = type + " " + contentB.getBytes(StandardCharsets.UTF_8).length + "\u0000" + contentB;

        String sha1ValueA = sha1(storeA);
        String expectedValueA = "5dd01c177f5d7d1be5346a5bc18a569a7410c2ef";

        String sha1ValueB = sha1(storeB);
        String expectedValueB = "ca94ea3b652074560b6a7d530f8187323982518a";

        Assertions.assertEquals(expectedValueA, sha1ValueA);
        Assertions.assertEquals(expectedValueB, sha1ValueB);
    }

    public String sha1(String data) throws NoSuchAlgorithmException {
        MessageDigest md = MessageDigest.getInstance("SHA1");
        md.update(data.getBytes());

        StringBuilder buf = new StringBuilder();

        byte[] bits = md.digest();
        for (int bit : bits) {
            int a = bit;
            if (a < 0) a += 256;
            if (a < 16) buf.append("0");
            buf.append(Integer.toHexString(a));
        }
        return buf.toString();
    }
}
```

测试通过。

#### 计算结论

从上述计算中，可以大概知道 Blob 对象的具体构成。实际上，不仅是 Blob 对象，诸如其他一些 Git 对象如 Tree 或 Commit 对象，它们的构成也和 Blob 对象相差无几。

可以说 Tree 或 Commit 对象和 Blob 对象仅有头部信息和内容有所差别，其内部结构都是差不多的。

### 





### 对象模型

假如有以下的仓库目录结构：

```
$>tree
.
|-- README
`-- lib
    |-- inc
    |   `-- tricks.rb
    `-- mylib.rb

2 directories, 3 files
```

如果其提交到 Git 仓库中，那么在 Git 里它会被表示成这样：

<img src="images/Git%20Internals.images/objects-example.png" alt="img" style="zoom: 80%;" />

这种结构完全可以类比 Java 中的类，Commit、Tree 和 Blob 都可以被视为简单的 Java 类，其中包含着必要的信息。

例如 Commit 对象中一般包含了指向上一个 Commit 对象（父提交）的指针；Tree 对象中一般包含指向 Blob 对象或其他的 Tree 对象的指针。

特殊地，Blob 对象中仅包含着文件大小、内容等信息，即所谓的二进制数据。

注意，Commit 对象中总是会包含一个指向根目录 Tree 对象的指针。基本上只要仓库进行了提交，根目录的内容就会产生变化，这意味着表示根目录的 Tree 对象也会变化。



### 权限模式

Tree 对象中的权限模式，基本上就是 Linux 系统上用于表示文件类型和权限的方式，而 Linux 系统遵循的是 POSIX 标准。 

POSIX（Portable Operating System Interface for UNIX）是一组操作系统接口标准，旨在提供一致的接口和可移植性，使不同的 UNIX-like 操作系统能够在这些接口的基础上开发兼容的应用程序。POSIX 标准由 IEEE（Institute of Electrical and Electronics Engineers）制定，确保在符合标准的操作系统上编写的程序可以在其他符合该标准的操作系统上无需或只需进行少量修改即可运行。

POSIX 标准定义了许多基本功能，如文件操作、进程管理、线程、信号处理、shell 等，以及相关的头文件、库函数和系统调用。它的目标是促进跨平台的应用程序开发和移植性，使得开发人员能够更轻松地将其代码移植到符合 POSIX 标准的不同 UNIX-like 操作系统上。

这些用于表示文件类型及权限的规则，均来源于 POSIX 标准。

类似于 100644 和 040000 这些权限模式，实际是 Git 中存储的 16 位文件模式，它遵循 POSIX 类型和模式的布局：

```
32-bit mode, split into (high to low bits)

    4-bit object type
      valid values in binary are 1000 (regular file), 1010 (symbolic link)
      and 1110 (gitlink)

    3-bit unused

    9-bit unix permission. Only 0755 and 0644 are valid for regular files.
    Symbolic links and gitlinks have value 0 in this field.
```

上述没有提及到目录所使用的类型，目录实际使用的对象类型为 0100（二进制）。

权限模式中的 6 位数字，除开首位数字单独表示 1bit 外，其余 5 位数字均是八进制模式数字，用于表示 3bits。

```
Type|---|Perm bits
1000 000 110100100
1 0   0   6  4  4

Type|---|Perm bits
1000 000 111101101
1 0   0   7  5  5
```

但需要注意，对于文件类型及权限的规则来说，Git 只支持 POSIX 标准中的小部分，基本上只能在 Git 中遇到几个常用的权限模式。

二进制模式下，文件的对象类型表示为 1000，目录的对象类型表示为 0100，链接的对象类型则表示为 1010。换算为八进制，文件、目录和链接所对应的对象类型模式就分别表示为 10、04、12。

其次是权限问题，16 为文件模式中的后 9 位用于表示文件权限。通常文件调用权限被分为三级：文件所有者（Owner）、用户组（Group）、其它用户（Other Users）。

具体可以参考下图：

<img src="images/Git%20Internals.images/rwx-standard-unix-permission-bits.png" alt="img" style="zoom:80%;" />

即 9bits 数据中，前 3bits 负责控制文件所有者对文件的 RWX 权限，中间 3bits 负责控制用户组对文件的 RWX 权限，后 3bits 则负责控制其他用户对文件的 RWX 权限。

假如有某个普通文件，文件拥有者对该文件仅具有读写权限，用户组和其他用户对该文件仅具有读权限，那么该文件的具体权限就可以使用二进制表示为：

```
110 100 100
```

换算为八进制表示形式（日常理解为转换成十进制也可以，但记住其本质是转换为八进制）：

```
6 4 4
```

Git 的 6 位权限模式中的第 3 位一般始终为零。因为 16 位文件模式中，第 3 位所对应的 3bits 数据属于未使用状态，其始终维持 000（二进制）。

那么于普通文件来说其文件类型为 10，如果文件权限为 644，那么在 Git 中它的权限模式就能被表示为 100644。

Git 中常见的模式权限有以下几个：

- 文件：100644 或 100755
- 目录：040000
- 链接：120000
