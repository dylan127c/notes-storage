### 概述

本篇将简单介绍 Git 版本控制工具的基本使用。

### 初始仓库

获取仓库有两种途径，一是初始化仓库，二是拉取远程仓库。

为了了解仓库的基础配置，这里选择初始化仓库：

```bash
git init
```

初始化仓库后，Git 会在当前目录下创建一个 `.git` 的隐藏目录，该目录是 Git 仓库的核心目录。

### 基本配置

初始化仓库后，使用 `git config -l` 命令可以查看所有默认配置：

```
~]# git config -l
diff.astextplain.textconv=astextplain
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true
http.sslbackend=schannel
core.autocrlf=true
core.fscache=true
core.symlinks=false
pull.rebase=false
init.defaultbranch=main
core.editor="E:\Microsoft VS Code\bin\code" --wait
core.repositoryformatversion=0
core.filemode=false
core.bare=false
core.logallrefupdates=true
core.symlinks=false
core.ignorecase=true
```

这些默认配置实际是系统配置（system）、全局配置（global）和本地配置（local）的集合。

三种配置之间存在优先级，从高到低依次是：本地配置、全局配置和系统配置。相同的配置项可以存在于多个配置中，不同的配置中如果存在多个相同的配置项，则只有较高级别的配置项会生效。

`git config` 命令配合不同的选项，可以查看或修改指定位置的配置文件：

- `--system`：系统配置，对本系统的所有用户均生效；
- `--global`：全局配置，仅对系统当前的登录用户生效；
- `--local`：本地配置，仅对当前仓库生效，配置文件存储路径为 `.git/config` 。

```
~]# git config --system -l
diff.astextplain.textconv=astextplain
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true
http.sslbackend=schannel
core.autocrlf=true
core.fscache=true
core.symlinks=false
pull.rebase=false
init.defaultbranch=main
```

```
~]# git config --global -l
core.editor="E:\Microsoft VS Code\bin\code" --wait
```

```
~]# git config --local -l
core.repositoryformatversion=0
core.filemode=false
core.bare=false
core.logallrefupdates=true
core.symlinks=false
core.ignorecase=true
```

查看本地配置文件 `.git/config` 的具体内容：

```
~]# cat .git/config
[core]
        repositoryformatversion = 0
        filemode = false
        bare = false
        logallrefupdates = true
        symlinks = false
        ignorecase = true
```

可以看到使用 `git config --local -l` 得到的配置内容和 `.git/config` 文件中的内容完全一致，仅书写形式存在些微的差别。

### 分支提交

`Git` 中有两个比较重要的概念：1. 分支（BRANCH）；2. 提交（COMMIT）。

#### 1. 分支

分支（branch）是指向 Git 仓库中某个特定提交对象的可变指针。

Git 的分支系统允许你创建、命名和管理多个独立的提交历史线。每个分支都代表了项目代码库的一个独立发展路径。在分支上进行的修改不会直接影响到其他分支，这使得团队能够并行开发多个功能、修复多个 bug 或者尝试不同的实验性特性。

分支的常见操作包括创建、切换、合并和删除。

#### 2. 提交

提交（commit）是指项目代码库中的一个快照或者一个版本。

每次提交都包含了当前代码库状态的所有文件以及与之关联的元数据信息，比如作者、时间戳、提交信息等。每个提交都有一个唯一的哈希值，用于标识该提交。提交记录构成了项目的提交历史，通过提交历史，你可以回顾项目的演变过程、查看特定版本的代码、追踪修改历史等。

提交的常见操作包括创建、查看、回滚、修改等。

#### 3. 联系

以下是分支和提交之间的联系：

1. **分支包含提交历史**：每个分支都包含了一系列的提交历史，这些提交代表了项目的不同状态和版本。通过查看分支的提交历史，可以了解到项目的发展历程以及每个提交所做的更改；

2. **提交形成分支的演化**：分支的创建和合并都是基于提交历史来进行的。当在某个分支上提交了更改时，它会成为该分支的新的提交历史的一部分；当合并一个分支到另一个分支时，**实际上是将两个分支的提交历史合并在一起**；

3. **分支指向特定的提交**：每个分支都指向一个特定的提交，这个提交代表了**分支的当前状态**。当在某个分支上提交了新的更改时，分支会移动到最新的提交上。实际上，分支就是指向提交历史中特定位置的指针；

4. **分支支持并行开发**：分支的存在使得团队可以同时进行不同的工作，因为每个分支都有自己的提交历史，不会相互干扰。这种并行开发的能力是通过创建和管理分支来实现的。

### 分支名称

分支名称是用来标识每个分支的唯一标识符。分支名称通常是在创建分支时给定的，但也可以随时重命名。

在一个标准的 Git 仓库中，通常会有一个默认的分支，且一般会被命名为 `master` 或者 `main` 分支。除了默认分支外，还可以创建任意数量的其他分支，每个分支都可以独立地**指向**提交历史中的不同位置。

默认分支名称决定了 `git init` 时默认创建分支的名称是什么，Windows 系统下 Git 默认分支名称的配置会内嵌在 Git 工具的安装过程中：

<div align="center"><img src="images/Git%20Basic.images/image-20240202203450185.png" alt="image-20240202203450185" style="width:50%;" /></div>

如果未在该步骤配置默认分支的名称，后续仍可以通过 Git 的系统配置文件来修改。

一般情况下，默认分组的配置项存在于系统配置中，因此修改配置项时需要添加 `--system` 参数。例如，将默认分支名称修改为 `master`：

```bash
git config --system init.defaultbranch master
```

需要注意，对配置作出任意修改但不添加任何用于指定配置文件的参数时，则默认修改本地配置。例如：

```bash
git config init.defaultbranch mybranch
```

该命令会在本地配置中添加 `init.defaultbranch` 默认分支配置：

```
~]# git config --local -l
...
init.defaultbranch=mybranch
```

然而本地配置中的 `init.defaultbranch` 一般不起作用。由于本地配置随仓库初始化一同出现，而默认分支名称基本仅在仓库初始化时才需要使用，这意味着本地配置中的默认分支名称配置项基本没用。

### 用户身份

提交者的用户邮箱、用户名称在每次 `commit` 时都需要使用，它在作用在于辨明创建提交的用户是谁。

建议将用户邮箱、用户名称写入全局配置中：

```
git config --global user.email "2phangx.dylan@gmail.com"
git config --global user.name "dylan127c"
```

虽然用户邮箱和用户名称都是必须配置的项，但实际用户邮箱会更重要些。

类似于 GitHub 等网站会根据 `user.email` 关联 GitHub 账户，并将对应的信息展示在提交记录中，而 `user.name` 将被忽略。而 GitKraken 或 GitViewer 等 GUI 工具，则会根据 `user.email` 来获取相应的 [Gravatar](https://gravatar.com/) 头像信息。

<div align="center"><img src="images/Git%20Basic.images/Snipaste_2024-03-15_14-24-55.png" alt="Snipaste_2024-03-15_14-24-55" style="width:80%;" /></div>

<div align="center"><img src="images/Git%20Basic.images/Snipaste_2024-03-15_14-25-07.png" alt="Snipaste_2024-03-15_14-25-07" style="width:80%;" /></div>

或者可以认为用户邮箱是唯一索引，而用户名称可以随时改变。

另外，可以根据配置文件的优先级，为个人项目和协作项目配置不同的用户邮箱、用户名称。例如，将协作项目的用户邮箱、用户名称写入到全局配置中：

```bash
git config --global user.email "working@gmail.com"
git config --global user.name "rose"
```

然后单独将个人项目的用户邮箱、用户名称写入到本地配置中：

```bash
git config user.email "personal@gmail.com"
git config user.name "princess"
```

这样，无论在哪里编辑个人项目，创建提交时都将优先使用本地配置中的用户邮箱、用户名称。但需要注意，个人项目如果需要转为公开的、可协作的项目，则务必记得移除本地配置中的身份配置，以免不必要的麻烦。

### 三种状态

任何被纳入版本控制的文件都属于已跟踪文件（tracked）。已跟踪文件存在三种不同的状态：

1. **已提交（commited）**：表示数据已经安全地保存在本地数据库中；
2. **已修改（modified）**：表示修改了文件，但还未确定是否要进行标记提交；
3. **已暂存（staged）**：表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

这对应着 Git 项目的三个阶段：1. Git 目录/仓库（Repository）；2. 工作区（Working Directory）；3. 暂存区（Staging Area）。

<div align="center"><img src="images/Git%20Basic.images/Snipaste_2024-03-21_16-22-00.png" alt="Snipaste_2024-03-21_16-22-00" style="width:80%;" /></div>

Git 仓库（Repository）就是初始化 Git 仓库目录内默认处于隐藏状态的 `.git` 目录，该目录用于保存项目的元数据和 `.git/objects` 对象数据库。这是 Git 中最重要的部分，从其它计算机克隆仓库时，复制的就是这里的数据。

工作区（Working Directory）一般是项目的某个版本所独立提取出来的内容，其中可以包含从项目中提取出来的已跟踪文件，还可以包含一些正在施工的或不纳入版本控制的未跟踪文件。

暂存区（Staging Area）实际上是一个文件，用于保存下次将要提交的文件列表信息，且一般位于 `.git` 目录中。按照 Git 的术语 Staging Area 文件应该叫做“索引”，不过一般说法还是“暂存区”。

Git 基本的工作流程：

1. 在工作区中修改文件；
2. 选择性地暂存那些需要提交的更改，这样只会将部分更改添加到暂存区；
3. 提交更新，这将提交所有在暂存区的文件，生成的版本快照将永久性地存储到 Git 目录中。

如果 Git 目录中保存着特定版本的文件，这些文件就属于**已提交**状态；如果文件已修改并放入暂存区，那么就属于**已暂存**状态；如果自上次检出后，作了修改但还没有放到暂存区，那么就属于**已修改**状态。

### 暂存操作

暂存（Stage）是指将工作目录（Working Directory）中的修改（新增未跟踪文件、编辑或删除已跟踪文件等）添加到暂存区（Staging Area）的过程。

暂存区（Staging Area）是版本控制系统中一种重要的概念，主要用于临时存放**待提交修改**，它在 Git 等版本控制工具中起着关键的作用。

暂存区类似于一个中间层，其位于工作目录（Working Directory）和代码库（Repository）之间，以充当**待提交修改**的缓冲区，并允许开发者对**待提交修改**进行整理、检查或调整，在确认无误后再将修改**正式提交**到代码库中。

因为每次提交都只会提交暂存区内的**待提交修改**的特性，开发者可以利用这个特性对代码进行**分批次的提交**。这种分批次提交的方式有以下几个优点：

1. **代码管理精细**：通过分批次的暂存并提交，开发者可以将不同功能或逻辑的修改分开，使得代码管理更加精细和清晰；
2. **版本历史清晰**：每次提交（commit）都代表着一个具体的功能或逻辑的修改，使得版本历史更加清晰和易于理解，可以更轻松地查看每次提交的内容和目的；
3. **问题定位和回溯**：如果出现问题或需要回溯历史版本，分批次提交可以帮助更快速地定位到特定功能或逻辑的修改上，以快速进行问题排查或版本回滚；
4. **团队协作**：在团队协作中，分批次提交可以帮助团队成员更好地管理自己的修改，减少冲突和混乱，提高协作效率。

未将文件纳入版本控制时，文件通常处于<u>未跟踪状态</u>。将未跟踪文件纳入版本控制之前，需先暂存文件：

```bash
git add <filename>
```

**未跟踪文件一旦暂存，文件就会从<u>未跟踪状态</u>转变为<u>已跟踪状态</u>。**

```bash
# 查看所有已跟踪状态文件（首次暂存或已纳入版本控制的文件）
git ls-files
```

如果希望转变文件的**已跟踪状态**为**未跟踪状态**，并在工作区中保留这些文件，则需分情况解决：

- （允许）文件如果是首次暂存，即刚从未跟踪状态转变为已跟踪状态，那么仅需要将文件从暂存区移除；
- **（不推荐）**文件如果已经历提交，那么文件当前如在暂存区则需先移除，之后再将文件转变为未跟踪状态。

```bash
# Step 1：如果已提交过的文件位于暂存区，则需将文件从暂存区移除
git restore --staged <filename>

# Step 2：再将文件从版本控制中剔除（不会删除工作区文件）
git rm --cached <filename>

# Step 3：最后提交变化
git commit -m <summary>
```

使用 `git rm --cached` 命令将文件从版本控制中剔除时，会自动生成一个关于目标文件的暂存记录：

```
~]# git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    first
```

这是因为处于**已跟踪状态**的曾提交文件必然存在于历史版本中，如果在当前的工作目录中修改这些处于**已跟踪状态**的文件为**未跟踪状态**，那么从版本控制的角度出发，这种操作无异于删除了这些文件。

类似于这种需要在工作区中保留文件的情况，一般是出于要将该文件纳入 `.gitignore` 中的需求。但如果是有这方面的需求，实则不建议使用以上的命令组合，更合理的方式是使用 `git reset` 命令，例如：

```bash
git reset HEAD~1
```

该命令可以完整保留工作区的情况下回溯至上一个提交，即尚未对目标文件进行版本控制的节点。

### 忽略文件

`.gitignore` 文件是 Git 版本控制系统中一个重要的配置文件，用于指定要忽略的未跟踪文件或目录。

未跟踪文件或目录如果不希望被版本控制，那么务必将文件或目录的名称添加到 `.gitignore` 文件中：

```
node_modules/
package.json
package-lock.json
*.log
```

另外，是否将 `.gitignore` 文件纳入版本控制中，取决于项目是私有项目还是协作项目。对于私有项目来说，需要忽略的文件从始至终都由一个开发者控制，这种情况下可以选择不将 `.gitignore` 纳入版本控制中。

而于协作项目来说，将 `.gitignore` 纳入版本控制具有重要意义：

- **确保一致性**：团队成员可能使用不同的 `.gitignore` 文件，导致一些文件被意外提交到版本库中。将 `.gitignore` 添加到版本控制可以确保所有团队成员使用相同的配置，避免此类问题；
- **追踪更改**：将 `.gitignore` 添加到版本控制后，可以像追踪其他文件一样追踪 `.gitignore` 文件的更改。这可以帮助团队成员了解 `.gitignore` 文件是如何演变的，以及为什么某些文件被忽略；
- **提高可移植性**：如果 `.gitignore` 文件与代码一起分发，其他人可以克隆代码并立即开始工作，而无需担心意外提交不需要的文件。

实际上，始终坚持创建 `.gitignore` 文件是好文明，始终将它纳入版本控制也是好文明。如果实在不确定是否需要使用到 `.gitignore` 文件，那么可以尝试创建它并在其中仅写入以下内容：

```
.gitignore
```

这样 Git 能够识别到 `.gitignore` 文件，它可以在不将 `.gitignore` 文件纳入版本控制的前提下，在工作目录中保留 `.gitignore` 文件。未来如果需要忽略某些文件，只需将以上内容替换为需忽略文件或目录的名称。

实际项目中，较常出现的情况是原本不希望纳入版本控制的文件，由于操作失误不小心被提交了。这种时候一旦操作不当，则可能造成三个严重的后果：

- 存储占用：不显式删除历史提交的情况下，文件对应的实际存储对象将一直占用 Git 仓库的存储空间；
- 数据泄露：如果被提交文件是私密文件，数据有通过历史版本泄露的风险；
- 数据丢失：当检出到历史版本的提交时，与未跟踪文件同名的历史版本文件将直接覆盖未跟踪文件，并且从该提交检出到已提交删除该文件的提交时，该历史版本文件将从工作目录中移除，未跟踪文件不会恢复。

假设存在以下仓库：

```
~]# git log
commit 21233f835c1ba02ed145659e60d3dc0203c11fe9 (HEAD -> main)
Author: dylan127c <2phangx.dylan@gmail.com>
Date:   Sun Mar 17 18:13:13 2024 +0800

    add .gitignore add new file second.

commit f4bb01597ea5468465436394f1cd4f9b2cd1c2c3
Author: dylan127c <2phangx.dylan@gmail.com>
Date:   Sun Mar 17 03:22:18 2024 +0800

    untrack first.

commit 3e06ef629e172943192329df65fb76504fcebc72
Author: dylan127c <2phangx.dylan@gmail.com>
Date:   Sat Mar 16 02:21:31 2024 +0800

    init.
```

其中 `first` 文件在<u>根提交</u> `3e06ef` 中不小心被纳入了版本控制，当时该文件中的内容为：

```
hey.
```

随后在 `f4bb01` 提交中，开发人员使用以下方式将 `first` 文件修改为<u>未跟踪文件</u>：

```bash
git restore --staged first
git rm --cached first
git commit -m "untrack first."
```

并在随后的 `21233f` 提交中新增 `.gitignore` 文件将 `first` 添加至文件内，并修改 `first` 文件内容为：

```
some.
```

开发人员一旦使用命令切换到 `3e06ef` 提交，那么未追踪文件 `first` 内容将被 `3e06ef` 提交的历史版本覆盖：

```
~]# git checkout 3e06ef && cat first
Note: switching to '3e06ef'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 3e06ef6 init.
hey.
```

且不仅如此，从 `3e06ef` 提交切回 `main` 分支指向的提交时，原本未跟踪文件 `first` 也将不再恢复：

```
~]# git checkout main && cat first
Previous HEAD position was 3e06ef6 init.
Switched to branch 'main'
cat: first: No such file or directory
```

**这也是为什么不推荐使用与纳入版本控制的文件的名称来命名未跟踪文件的原因，即使纳入版本控制的文件已从历史版本中删除，但只要被删除的文件有可能通过命令回溯至工作区，那未跟踪文件就总是处于不安全的状态。**

最后注意，只有未跟踪文件添加到 `.gitignore` 时，文件才会被 Git 忽略。未从版本控制中剔除的已跟踪文件，即便将其添加到 `gitignore` 文件中，也不会被 Git 忽略。

### 版本回溯

如果不慎将无需纳入版本控制的文件提交了，那么一般只推荐使用版本回溯命令 `git reset` 来纠正误操作。在使用 `reset` 前，需要简单理解一下暂存区（Staging Area）。

众所周知，提交之间存在版本差异，或者说每个提交都可以认为是一次快照记录，而所有快照的实际存储位置是 Git 的对象数据库 `.git/objects` 目录。

但鲜为人知的是，暂存同样存快照记录，且所有的暂存记录都被存储在 `.git/index` 文件中：

- `git add`：将文件添加到暂存区时，Git 会将该文件的相关信息写入 `.git/index` 文件；
- `git commit`：提交暂存区时，Git 会将 `.git/index` 文件中的内容写入版本库中，并创建一个新的提交对象。

注意，**提交操作并不会清空 `.git/index` 文件内的暂存记录**！提交仅会将本次暂存区的内容写入到版本快照中。

Git 使用 `reset` 命令进行版本回溯，该命令拥有三种常用模式：1. mixed；2. soft；3. hard。

- `--mixed`：**默认选项**。该模式下 `reset` 命令会在保留工作区内所有文件的前提下，将版本回溯至指定的历史提交，回溯后所有对已跟踪文件作出的修改都会被尽数保留，；
- `--soft`：类似于 `--mixed` 模式，但不同的是 `--soft` 模式会重置暂存区。这里的重置暂存区指的是将暂存区恢复为目标回溯版本之后所有暂存快照的集合；
- `--hard`：该选项是近乎完全的版本回溯，目标回溯版本中所记录的已跟踪文件，将完全覆盖工作区中的现有文件，未跟踪文件不受影响。

以下述仓库为例：

- 初始工作区包含 `.gitignore` 和 `untracked` 两个文件，其中 `untracked` 文件不需要跟踪；
- 后续分三次新增了 `first`、`second` 和 `third` 文件并分别进行了提交。

```
~]# git log
commit e41dcd824c86ec0cbef6d250635f73472fb1eb6d (HEAD -> main)
Author: dylan127c <2phangx.dylan@gmail.com>
Date:   Sat Mar 23 21:29:22 2024 +0800

    add third.

commit 928810da8aad0f6648551fb5d4220c5f6fcb936b
Author: dylan127c <2phangx.dylan@gmail.com>
Date:   Sat Mar 23 21:29:08 2024 +0800

    add second.

commit bf79a9406596e2280b538741835e17eb5bb5ce3f
Author: dylan127c <2phangx.dylan@gmail.com>
Date:   Sat Mar 23 21:28:48 2024 +0800

    add first.

commit 6794c2cb7e30aecb2678d71137520ecc886882af
Author: dylan127c <2phangx.dylan@gmail.com>
Date:   Sat Mar 23 21:28:27 2024 +0800

    init .gitignore file.

~]# ll -a
total 24
drwxr-xr-x 1 dylan 197609  0  3月 23 21:29 ./
drwxr-xr-x 1 dylan 197609  0  3月 23 21:29 ../
drwxr-xr-x 1 dylan 197609  0  3月 23 21:29 .git/
-rw-r--r-- 1 dylan 197609  9  3月 23 21:26 .gitignore
-rw-r--r-- 1 dylan 197609 13  3月 21 15:35 first
-rw-r--r-- 1 dylan 197609 14  3月 21 15:35 second
-rw-r--r-- 1 dylan 197609 13  3月 21 15:36 third
-rw-r--r-- 1 dylan 197609  0  3月 23 21:26 untracked

~]# cat .gitignore
untracked
```

#### 1. hard

如果使用以下命令回溯至根提交版本：

```bash
git reset --hard HEAD~3
```

查看文件状态，并列出目录内的所有文件：

```
~]# git status
On branch main
nothing to commit, working tree clean

~]# ll -a
total 25
drwxr-xr-x 1 dylan 197609 0  3月 23 21:34 ./
drwxr-xr-x 1 dylan 197609 0  3月 23 21:29 ../
drwxr-xr-x 1 dylan 197609 0  3月 23 21:34 .git/
-rw-r--r-- 1 dylan 197609 9  3月 23 21:26 .gitignore
-rw-r--r-- 1 dylan 197609 0  3月 23 21:26 untracked
```

选项 `--hard` 相当于完整的版本回溯，它会删除所有回溯版本之后的已跟踪文件，而仅保留该版本下的已追踪文件。未跟踪文件不会受到影响。

#### 2. mixed

如果使用以下命令回溯至根提交版本：

```bash
git reset --mixed HEAD~3
```

查看文件状态，并列出目录内的所有文件：

```
~]# git status
On branch main
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        first
        second
        third

nothing added to commit but untracked files present (use "git add" to track)

~]# ll -a
total 24
drwxr-xr-x 1 dylan 197609  0  3月 23 21:32 ./
drwxr-xr-x 1 dylan 197609  0  3月 23 21:29 ../
drwxr-xr-x 1 dylan 197609  0  3月 23 21:32 .git/
-rw-r--r-- 1 dylan 197609  9  3月 23 21:26 .gitignore
-rw-r--r-- 1 dylan 197609 13  3月 21 15:35 first
-rw-r--r-- 1 dylan 197609 14  3月 21 15:35 second
-rw-r--r-- 1 dylan 197609 13  3月 21 15:36 third
-rw-r--r-- 1 dylan 197609  0  3月 23 21:26 untracked
```

选项 `--mixed` 相当于仅回溯版本，同时保留当前完整的工作区不作修改。版本回溯后，由于工作区被完整地保留下来，这意味着对该回溯版本中的已跟踪文件的修改也会被保留下来，并等待暂存。未跟踪文件不会受到影响。

这其实就相当于将回溯前的工作区目录复制到剪切板，之后再覆盖到使用选项 `--hard` 回溯得到的工作区中。

#### 3. soft

如果使用以下命令回溯至根提交版本：

```bash
git reset --soft HEAD~3
```

查看文件状态，并列出目录内的所有文件：

```
~]# git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   first
        new file:   second
        new file:   third

~]# ll -a
total 24
drwxr-xr-x 1 dylan 197609  0  3月 23 21:32 ./
drwxr-xr-x 1 dylan 197609  0  3月 23 21:29 ../
drwxr-xr-x 1 dylan 197609  0  3月 23 21:33 .git/
-rw-r--r-- 1 dylan 197609  9  3月 23 21:26 .gitignore
-rw-r--r-- 1 dylan 197609 13  3月 21 15:35 first
-rw-r--r-- 1 dylan 197609 14  3月 21 15:35 second
-rw-r--r-- 1 dylan 197609 13  3月 21 15:36 third
-rw-r--r-- 1 dylan 197609  0  3月 23 21:26 untracked
```

选项 `--soft` 类似于选项 `--mixed` 同样会保留当前完整的工作区不作修改。但不同的是，`--soft` 会重置暂存区。

所谓**重置暂存区**，指的是将回溯后的暂存区**重置**为回溯版本之后的所有提交的暂存记录的集合。

举个例子，根提交 `6794c2` 至最新提交 `e41dcd` 一共经历了 3 次暂存：

```
~]# git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   first

~]# git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   second

~]# git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   third
```

那么使用选项 `--soft` 从最新提交 `e41dcd` 回溯至根提交 `6794c2` 时 ，暂存区会重置为以上 3 次暂存的集合：

```
~]# git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   first
        new file:   second
        new file:   third
```

这些暂存记录实际保存在 `.git/index` 中，可以理解为使用选项 `--soft` 能够保留这些暂存记录。如果本来就打算保留所有的修改，那么使用 `--soft` 能免除后续将文件一一筛选并添加至暂存区的繁杂操作。

### 文件差异

对同一个已跟踪文件作出修改后，会出现文件差异。文件差异可以通过一些命令进行实时查看：

```bash
# 比较已暂存文件和该文件最近一次提交之间的差异
git diff --cached <filename>

# 比较未暂存文件和该文件最近一次暂存或提交之间的差异
git diff <filename>
```

简单例子：

```bash
# Step 1：初始化仓库
git init

# Step 2：新建文件并写入初始内容，暂存并创建提交
echo "hey." > first
git add first
git commit -m 'init.'

# Step 3：为文件追加新内容，仅暂存不提交
echo "hello." >> first
git add first

# Step 4：移除上一步中添加的新内容，并重新追加其他新内容
echo "hey." > first
echo "how are you." >> first
```

使用带 `--cached` 选项的 `git diff` 命令：

```
~]# git diff --cached first
diff --git a/first b/first
index 27801ec..c6934cc 100644
--- a/first
+++ b/first
@@ -1 +1,2 @@
 hey.
+hello.
```

如果目前分支的暂存区不存在 `first` 文件，则命令得不到任何输出。

使用不带 `--cached` 选项的 `git diff` 命令：

```
~]# git diff first
diff --git a/first b/first
index c6934cc..375bd02 100644
--- a/first
+++ b/first
@@ -1,2 +1,2 @@
 hey.
-hello.
+how are you.
```

以上命令比较了被修改的已跟踪文件 `first` 与暂存区中 `first` 文件的差异。

如果将暂存区中的 `first` 文件移除：

```bash
git restore --staged first
```

那么命令将变成比较被修改的已跟踪文件 `first` 与最新一次提交中 `first` 文件的差异：

```
~]# git diff first
diff --git a/first b/first
index 27801ec..375bd02 100644
--- a/first
+++ b/first
@@ -1 +1,2 @@
 hey.
+how are you.
```

### 执行提交

提交（commit）是开发过程中频繁使用的操作之一，它允许开发者在项目的不同阶段保存和记录代码的变化，便于团队协作、代码审查、版本回滚等操作。

使用 `git commit` 命令可以创建一个新的提交。在执行提交时，需要提供提交信息（commit message），用于描述这个提交所做的更改。提交信息应该简明扼要地描述你的更改内容，以便其他开发者能够理解。

```bash
git commit -m <message>
```

执行提交操作后，Git 将会把当前暂存区的内容提交到版本库中，创建一个新的提交对象，并更新项目的提交历史。

另外，Git 提供了 `--amend` 选项，可用于修改最近一次提交。例如，可以使用 `--amend` 来修改最近一次提交的提交信息：

```bash
git commit --amend -m <message>
```

除了修改提交信息外，还可以添加新的文件到最近一次提交中，或者将之前已暂存但尚未提交的更改添加到最近一次的提交中。

```bash
git add <new-file>
git commit --amend -m <message>
```

注意，使用 `--amend` 选项可以修改最近一次提交的内容，但这也会改变提交的哈希值。因此，如果已经将该提交推送到了远程仓库，那么需要谨慎修改提交，因为这可能会导致远程仓库中的提交历史与本地仓库不一致。

### 分支操作

在当前提交上创建新分支：

```bash
git branch <new-branch>
```

在特定提交上创建新分支：

```bash
git branch <new-branch> <commit-hash>
```

常用的切换分支命令：

```bash
git checkout <branch>
```

但在 Git 2.23 版本后，官方更推荐使用 `switch` 命令切换分支：

```bash
git switch <branch>
```

如果希望在创建分支后立刻切换至目标分支，可以使用以下命令：

```bash
git checkout -b <branch> [<commit-hash>]
```

或者：

```bash
git switch -c <branch> [<commit-hash>]
```

重命名分支名称：

```bash
git branch (-m | -M) [<oldbranch>] <newbranch>
```

如果指定分支已经合并到其他分支，则删除该指定分支可以使用以下命令：

```bash
git branch -d <branch>
```

如果指定分支尚未合并，但仍需要删除，则可以使用 `-D` 选项强制删除：

```bash
git branch -D <branch>
```

### 分支合并

在 Git 中常见的分支合并操作主要有两种方式。每种方式都有其优缺点，选择合适的合并方式取决于你的项目需求、团队工作流程以及个人偏好。

一般本地仓库会直接需要使用到 `merge` 或者 `rebase` 用于合并不同分支的内容。另外一种需要使用分支合并命令的情况，是拉取远程仓库内容时：

<div align="center"><img src="images/Git%20Basic.images/Snipaste_2024-04-02_02-08-16.png" alt="Snipaste_2024-04-02_02-08-16" style="width:30%;" /></div>

一般情况下，拉取远程仓库默认的分支合并策略是 `merge` 且合并会自动进行。

下面使用同一个仓库来演示 `merge` 和 `rebase` 的不同效果：

<div align="center"><img src="images/Git%20Basic.images/Snipaste_2024-03-31_23-01-36.png" alt="Snipaste_2024-03-31_23-01-36" style="width:80%;" /></div>

仓库内有 `main` 和 `dev` 两个分支，且不存在文件冲突，其中 Git Graph 均按提交的时间顺序排列。

#### 1. merge

`merge` 合并也称为**三方合并**，它是 Git 默认的合并方式。

`merge` 会将两个分支的历史合并成一个新的提交，这个新的提交包含了两个分支上的所有更改。合并操作会保留每个分支的历史记录，因此合并后的分支会有一个额外的合并提交来表示合并的发生。

```bash
git merge <branch> -m <message>
```

这种方式不会改变原始分支的提交历史。

以上述仓库为例，将 `dev` 分支合并到 `main` 分支，需要切换到 `main` 分支执行 `merge` 命令：

```bash
git merge dev -m 'merge dev branch into main branch.'
```

<div align="center"><img src="images/Git%20Basic.images/Snipaste_2024-03-31_23-01-12.png" alt="Snipaste_2024-03-31_23-01-12" style="width:80%;" /></div>

在没有文件冲突的前提下，`merge` 之后所有来自 `dev` 分支的变动都被合并到了 `main` 分支的末端，且非 fast-forward 合并的情况下，新的提交会自动生成。

如果以上例子中 `main` 分支自 `dev` 分支创建后没有生成任何新的提交，则 `merge` 将采用 `fast-forward` 合并，即 Git 只会简单地将 `main` 分支的指针移动到 `dev` 分支的最新提交处，而不执行实际的合并。

合并如采用 `fast-forward` 方式，则意味着不会自动生成新的提交。多数情况下，将本地仓库内容推送至远程仓库时所发生的合并操作（即合并远程分支到本地分支），使用的就是 `fast-forward` 合并。

Git 总是会先尝试使用 `fast-forward` 合并，在确认无法执行时才会使用正常的三方合并。

#### 2. rebase

`rebase` 合并也被称为**变基合并**，它会将当前分支的提交移动到另一个分支的末端，并在此过程中将当前分支的提交逐个应用到目标分支上。

```bash
git rebase <branch> -m <message>
```

这样做的结果是，历史记录会更线性，因为它看起来就像是一系列的提交是按顺序应用到目标分支上的。这种方式会改变提交的顺序和提交的哈希值，一般只推荐应用于尚未推送到共享仓库的提交。

以上述仓库为例，将 `dev` 分支变基至 `main` 分支，需要切换到 `main` 分支执行 `rebase` 命令：

```bash
git rebase dev
```

<div align="center"><img src="images/Git%20Basic.images/Snipaste_2024-03-31_23-03-50.png" alt="Snipaste_2024-03-31_23-03-50" style="width:80%;" /></div>

在没有文件冲突的前提下，变基 `rebase` 会将 `main` 与 `dev` 分支的共同祖先之后所有的 `main` 分支提交，都移动到 `dev` 分支的末端。

### 文件冲突

不同的提交对同一个文件的同一部分进行了修改后，后续这些不同的提交如需合并，则会导致文件冲突的发生。无论何时发生文件冲突，Git 都会在操作被中止时发起通知，并且在冲突的文件中插入特殊标记以提示冲突的位置。

Git 在冲突发生的文件中插入特殊标记来指示冲突的位置。这些标记包括 `<<<<<<<`, `=======`, 和 `>>>>>>>`。在这些标记之间的部分表示两个分支上的不同版本，开发人员需要手动解决这些冲突。

其中：

- `<<<<<<<`：标记开始了一个冲突块，表示冲突的起始位置；
- `=======`：标记分隔了当前分支的修改和合并另一个分支的修改；
- `>>>>>>>`：标记结束了冲突块，表示冲突的结束位置。

例如，如果有一个文件发生了冲突，可能会看到类似如下的内容：

```
<<<<<<< HEAD
这是当前分支的修改。
=======
这是合并另一个分支的修改。
>>>>>>> branch_name
```

在这个示例中，`<<<<<<< HEAD` 到 `=======` 之间的部分表示当前分支的修改，`=======` 到 `>>>>>>> branch_name` 之间的部分表示合并另一个分支的修改。

解决文件冲突需要手动干预，需要查看被标记为冲突的文件，并根据实际情况来选择保留哪个修改或者如何解决冲突，之后才能继续版本控制的其他操作。

以下仓库作为演示文件冲突的例子：

<div align="center"><img src="images/Git%20Basic.images/Snipaste_2024-04-02_00-50-55.png" alt="Snipaste_2024-04-02_00-50-55" style="width:80%;" /></div>

其中 `main` 分支修改了 `first` 文件的内容为：

```
here is branch main
```

其次 `dev` 分支修改了 `first` 文件的内容为： 

```
here is branch dev
```

先使用 `merge` 的方式将 `dev` 分支合并到 `main` 分支：

```
~]# git merge dev -m 'merge branch dev.'
Auto-merging first
CONFLICT (content): Merge conflict in first
Automatic merge failed; fix conflicts and then commit the result.
```

查看冲突文件：

```
~]# cat first
<<<<<<< HEAD
here is branch main
=======
here is branch dev
>>>>>>> dev
```

同时目前工作区内的文件状态为：

```
~]# git status
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Changes to be committed:
        new file:   fifth
        new file:   third

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   first
```

可以看到文件冲突出现时，合并会暂时中止，且存在冲突的文件会被标记在 `Unmerged paths` 中。

这里选择保留 `dev` 分支的改动，需要**手动**将 `first` 文件的内容修改为：

```
~]# cat first
here is branch dev
```

如果有类似于 VSCode 等工具，则可以更便捷地完成修改：

<div align="center"><img src="images/Git%20Basic.images/Snipaste_2024-04-02_01-28-23.png" alt="Snipaste_2024-04-02_01-28-23" style="width:80%;" /></div>

冲突处理完毕之后，需要暂存处理完冲突的文件：

```bash
git add first
```

最后结束合并，将暂存文件进行提交：

```
~]# git commit -m 'conflict done.'
[main 35dc2fb] conflict done.
```

<div align="center"><img src="images/Git%20Basic.images/Snipaste_2024-04-02_01-35-23.png" alt="Snipaste_2024-04-02_01-35-23" style="width:80%;" /></div>

如果使用 `rebase` 的方式将 `dev` 分支合并到 `main` 分支，情况会略有差异：

```
~]# git rebase dev
Auto-merging first
CONFLICT (content): Merge conflict in first
error: could not apply 6a85047... add content to first.
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply 6a85047... add content to first.

~]# git status
interactive rebase in progress; onto 4d461e7
Last commands done (3 commands done):
   pick 99f86cd add fourth.
   pick 6a85047 add content to first.
  (see more in file .git/rebase-merge/done)
No commands remaining.
You are currently rebasing branch 'main' on '4d461e7'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git restore --staged <file>..." to unstage)
  (use "git add <file>..." to mark resolution)
        both modified:   first

no changes added to commit (use "git add" and/or "git commit -a")
```

冲突文件仍旧是 `first` 但冲突内容排序有所不同：

```
~]# cat first
<<<<<<< HEAD
here is branch dev
=======
here is branch main
>>>>>>> 6a85047 (add content to first.)
```

同样是将其他分支（dev）的内容合并到当前分支（main），在文件出现冲突时，普通合并（merge）会将其他分支的内容后置，但变基（rebase）则会将其他分支的内容前置。

冲突处理完毕之后，同样需要暂存该目标文件：

```bash
git add first
```

但注意，变基不需要手动执行提交。取而代之，要告诉 Git 继续执行后续的变基操作：

```
~]# git rebase --continue
Successfully rebased and updated refs/heads/main.
```

<div align="center"><img src="images/Git%20Basic.images/Snipaste_2024-04-02_01-49-22.png" alt="Snipaste_2024-04-02_01-49-22" style="width:80%;" /></div>

### 远程分支

远程分支（Remote Branch）是指存储在远程仓库中的分支。远程分支跟踪了远程仓库上的分支状态，并且可以让你在本地仓库中查看和操作远程仓库的分支。

在 Git 中克隆一个远程仓库或者拉取远程仓库的更新时，Git 会在本地仓库中自动创建远程分支的副本。这些远程分支通常以 `origin/` 开头，并对应远程仓库中的分支。例如，`origin/main` 表示远程仓库中的主分支，而 `origin/feature` 则表示远程仓库中的某个特性分支。

在进行远程操作时，可以使用远程分支来跟踪远程仓库的状态，例如拉取远程分支的更新、推送本地分支到远程仓库、合并远程分支到本地分支等。与本地分支一样，可以在远程分支上执行各种 Git 操作，如查看提交历史、创建新的分支、进行合并操作等。

要查看本地仓库中的所有远程分支，可以使用以下命令：

```bash
git branch -r
```

要查看本地仓库中的所有远程分支以及对应的最新提交，可以使用以下命令：

```bash
git branch -vv
```

通过操作远程分支，可以方便地与远程仓库进行交互，并与团队共享代码。

### 推送拉取

如果本地仓库存在对于的远程仓库，则需要使用到推送（push）或拉取（pull）命令。

一个简单的推送例子，可以在 GitHub 中创建一个空的远程仓库：

<div align="center"><img src="images/Git%20Basic.images/Snipaste_2024-04-03_16-01-23.png" alt="Snipaste_2024-04-03_16-01-23" style="width:80%;" /></div>

创建完毕后 GitHub 会给出将本地仓库推送至该远程仓库的具体命令：

<div align="center"><img src="images/Git%20Basic.images/Snipaste_2024-04-03_16-12-36.png" alt="Snipaste_2024-04-03_16-12-36" style="width:80%;" /></div>

其中以下三条命令最为关键：

```bash
# Step 1：为远程仓库的 HTTPS 或 SSH 链接添加别名，一般为 origin
git remote add origin git@github.com:dylan127c/test.git

# Step 2：将当前分支名称强制命名为 main，因为 GitHub 仓库的默认分支名称推荐为 main
git branch -M main

# Step 3：将本地仓库内容推送到远程仓库，使用 -u 选项将 origin/main 配置为默认的远程仓库分支
git push -u origin main
```

一般来说，本地仓库只需要对应一个远程仓库即可，这样方便推送和拉取操作。

如果存在多个远程仓库用于保存本地仓库的内容，也可以添加多个远程仓库链接：

```bash
git remote add <remote_alias> <remote_url>
```

但需要注意，默认推送或拉取的远程仓库分支有且只能存在一个：

```bash
git push -u <remote_alias> <remote_branch>
```

以上命令中添加 `-u` 选项会将 `remote_alias/remote_branch` 配置为默认的上游分支，这意味着直接执行 `git push` 或 `git pull` 等命令时将默认从 `remote_alias` 仓库的 `main` 分支中存取数据。

如果 `origin` 存在其他的 `dev` 分支，则推送或拉取命令需要完整地给出：

```bash
git push origin dev
git pull origin dev
```

除了在推送仓库的时候配置默认的上游分支外，还可以使用 `git branch` 命令：

```bash
git branch -u <remote_alias>/<remote_branch>
```

例如：

```bash
git branch -u origin/main
```

实际上还可以选择直接编辑 `.git/config` 文件，但不太推荐。

一般情况下只有首次将本地仓库推送至远程仓库时，需要配置默认的上游分支，这实际就是用于简化日常的推送或拉取操作。

如果是直接从远程仓库中克隆仓库至本地，则默认的上游分支等都会默认存在于 `.git/config` 文件中，不需要手动进行配置。

如需列出所有远程分支的名称，可以使用以下命令：

```bash
git branch -r
```

如需列出远程仓库的具体信息，可以使用以下命令：

```bash
git remote -v
```
