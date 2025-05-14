# FreeBSD pkg 命令概述

- 原文：[FreeBSD pkg コマンド概要](https://qiita.com/nanorkyo/items/2ff7cccfe3bc544f6f5e)
- 作者：重村法克
- 2025-02-09

## 介绍

由于 FreeBSD 中针对第三方工具引入的机制（Ports 和 Packages）的完善，二进制包的使用得到了发展。尤其是引入了 Flavors（版本）功能的好处尤为显著，尽管安装的内容是相同的，但由于依赖的包版本不同，组合的复杂性也随之增加。然而，只要为这些依赖包的所有版本提供相应的包，那么包管理就变得非常简单了。虽然我们希望所有的依赖包都能提供完整的包，但依然能够感受到二进制包带来的便利。

在这里，为了能够更好地利用二进制包，我总结了一些常用的命令。虽然详细的选项讲解会非常繁琐，但我只会列出常用的选项。此外，大多数命令在执行时都会询问是否继续。因此，几乎所有命令都可以使用 `-y` 选项来避免这一提示，我将在这里省略这一部分的详细描述。当你熟悉之后，可以自如使用。除此之外，我也没有列举太多实际操作的案例。如果你有任何疑问，欢迎告知，我会补充相关内容。

## 常用的 pkg 命令

在这里，我会在命令名称（或子命令名称）后标注 `[R]` 和 `[L]`，它们的含义如下：

* `[R]`：参考外部包/仓库中的信息。
* `[L]`：参考已安装的包或本地文件系统中的包。
* `[RL]`：根据环境的不同，既可以是 `[R]` 也可以是 `[L]`。

### pkg update \[R]

这是用来收集包元数据的命令。首先需要收集元数据，其他命令的执行依赖于此。这与 `yum check-update` 很相似。
需要注意的是，这并不是一个每次都需要执行的命令，许多命令会在自动更新元数据后再执行，因此如果你不介意自动更新，便不需要手动执行该命令。以下子命令会基于最新的元数据执行：

* `pkg install`
* `pkg upgrade`
* `pkg version`
* `pkg search`
* `pkg fetch`
* `pkg rquery`

### pkg install \[R]

用于安装软件包。与后面提到的 `pkg add` 不同，`pkg install` 是通过关键词指定并从网络获取包来安装。

### pkg add \[L]

用于安装指定的包文件。与 `pkg install` 不同，`pkg add` 是明确指定包文件名并进行安装。

### pkg delete \[L]

删除指定的软件包。

### pkg delete -ayf

使用 `-a` 选项删除所有包，`-f` 选项强制执行删除，并且 `-y` 选项使得操作不进行任何询问。直观上，指定 `-ay` 就足够了，但是在因依赖关系无法删除某些包时，必须指定 `-f` 选项。如果你打算删除所有包，毫不犹豫地全部指定就好。

### pkg version \[RL]

用于进行包的版本比较。由于其特殊性，使用时需要分解具体的用例来理解。

* 默认情况下，`pkg version` 会与 `/usr/ports/INDEX-ＯＳ major version` 这样的元数据文件进行比较。
* 如果该文件不存在，会参考每个 Ports 的源目录（相当于指定 `-P` 选项）。
* 如果该目录也不存在，则会参考远程仓库（相当于指定 `-R` 选项）。

前两种情况属于 `[L]` 范畴，而后者则属于 `[R]` 范畴。

### pkg version -L =

通常使用 `-L` 选项进行已安装包的版本比较，使用 `=` 来显示没有匹配的包。通常情况下，只指定 `=` 就足够了，但也可以指定 `>`、`<`、`?`、`!` 等其他符号，所有这些符号都需要加引号。

以下是每个符号的含义，但在使用 `-L` 选项时，表示的是“非该符号”的情况。若需要查找该符号的含义，则需要使用 `-l` 选项。

* `>`：当前安装的版本有较新的版本可用。
* `=`：当前安装的版本与发布的版本相同。
* `<`：当前安装的版本有较旧的版本可用，通常是元数据未更新或者进行新版本的测试安装时会出现。
* `?`：当前安装的包来源（origin）不存在，无法进行版本比较（可能已被删除或重命名）。
* `!`：当前安装的版本无法进行版本比较。

### pkg version -t

用于比较任意指定的版本号（实际上是字符串）。这个功能更多是用于测试不同版本号之间的关系，而不是特定包的功能。它与其他子命令有所不同，主要用于测试版本号的替换（例如，在替换 alpha 版、beta 版时），帮助用户确定哪个版本更大（更新），哪个版本更小（更旧）。

关于版本比较的规则，您可以参阅 [FreeBSD Porter's Handbook - 5.2.2. Versions, DISTVERSION or PORTVERSION](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-versions)。

### pkg upgrade \[R]

更新指定的包。如果没有特别指定，默认会检查所有包并尝试更新它们。

### pkg info \[L]

显示指定包的信息。如果没有指定包，则列出所有已安装包的概述。

### pkg info -a

与其他选项一起使用时，表示列出所有包的信息。

### pkg info -l

显示指定包的安装路径列表。

### pkg info -D

显示指定包的注意事项，可能包含安装提示或其他相关信息。

### pkg lock \[L]

锁定指定包，防止其被更新。锁定的效果不仅限于防止包本身的更改，还包括以下功能：

* 阻止锁定包的重新安装、升级、降级和删除。
* 阻止依赖于锁定包的其他包的升级或降级，特别是当依赖关系指向另一个版本的锁定包时。
* 阻止锁定包依赖的其他包的删除、升级或降级。
* 即使间接共享依赖关系，也会被锁定。

### pkg unlock \[L]

解锁指定包。

### pkg search \[R]

使用正则表达式（大小写不敏感）搜索包名。通过 `-e` 选项可以进行严格的匹配，但这包括版本号，因此对于一般用户来说并不太友好。`-g` 选项支持类似于 shell glob（例如 `?` 或 `*`）的模糊搜索，但由于版本号的包含，实际上搜索时需要加上 `*`。

### pkg which \[L]

显示指定的文件、目录等属于哪个包。使用 `-o` 选项可以显示该包的 origin。

# 不常用但值得了解的 pkg 命令

### pkg audit \[RL]

报告已安装包与漏洞信息的匹配结果。由于需要从外部获取漏洞信息，这个功能偏向 `[R]`，但如果没有急于更新信息的需求，就无需通过 `-F` 选项主动获取漏洞信息。通常，系统会有一个脚本每天自动获取漏洞信息。

### pkg check -s \[L]

检查指定包是否未被篡改且保持一致性。通常与 `-a` 选项一起使用。

### pkg check -d \[L]

检查指定包的依赖是否完好，是否未损坏。通常与 `-a` 选项一起使用。

### pkg autoremove \[L]

删除孤立的包。所谓“孤立”的包是指因为依赖关系而安装，但由于某些包被删除或更新后不再被依赖，因此变得不再使用。

实际中，某些“孤立”的包可能仍在使用，但由于包的依赖信息无法反映这一点，因此在执行此操作时需要小心。例如，如果 `bash` 被作为某个包的依赖安装，但你仍在使用它，这种情况下就需要特别留意。可以通过显式安装 `bash` 或使用后续技巧避免此问题。

在 `ports` 系统中，通常是需要在构建过程中使用的包（如编译器等）会被标记为删除对象。

### pkg updating \[L]

显示需要注意的包更新步骤（手动更新指示）。具体来说，会读取 `/usr/ports/UPDATING` 文件，并根据安装的（或指定的）包信息匹配并显示相关内容。该文件的内容是为人类阅读设计的，构造上有些启发式，但其严格性可能并不高。此外，需要确保该文件存在，系统不会自动下载该文件。

### pkg set \[L]

修改指定包的元数据（数据库）。通常是在根据 `pkg updating` 信息操作时才会用到这个命令，一般不需要主动使用。

### pkg set -A

通过为 `-A` 选项指定以下值，来修改指定包的元数据。此更改会影响 `pkg autoremove` 的行为结果。

* `0`：使指定的包不成为 `pkg autoremove` 的删除对象。
* `1`：使指定的包成为 `pkg autoremove` 的删除对象。

对这个选项感兴趣的人，可能也会关心 `pkg autoremove` 的工作原理。实际上，仅设置这个标志并不足以使包成为 `pkg autoremove` 的目标；还需要判断该包没有被其他包依赖等条件。

另外，通过 `pkg install` 或 `pkg add` 明确指定安装的包，在安装时会自动设置为此标志无效。这个标志只对因依赖关系被自动安装的包有效。

### pkg set -o

使用 `-o 旧 origin:新 origin` 的形式，将所有依赖旧 origin 的包的元数据中的 origin 信息修改为新值。

### pkg set -n

使用 `-n 旧包名:新包名` 的形式修改包名。不过，考虑到包名中通常不包含版本号，这个操作本身存在一些微妙的问题。

## 太小众了其实可以不用知道的 pkg 命令

### pkg alias \[L]

显示定义在 `/usr/local/etc/pkg.conf` 中的别名信息。

### pkg query \[L]

以指定的格式显示软件包的元信息。如果未指定包名，则会针对所有已安装的包进行查询。
可用于格式字符串中的占位符如下所示：

* `%n`：包名
* `%v`：包的版本
* `%o`：包的 origin
* `%p`：包的 prefix
* `%m`：包的维护者（电子邮件地址）
* `%c`：包的注释
* `%e`：包的描述
* `%w`：包的网址
* `%l`：包的许可证逻辑（无／单一／and／or）
* `%sb` 或 `%sh`：包的大小（字节单位或更适合人类阅读的单位）
* `%a`：包的自动安装标志
* `%Q`：包支持的架构候选项
* `%q`：包的架构（操作系统、版本、CPU 架构）
* `%k`：包的锁定标志
* `%M`：包的信息消息
* `%t`：包的安装时间戳
* `%R`：包的安装来源仓库
* `%X`：包的校验和
* `%?C`：表示某种条件 `C` 是否成立的布尔标志（`d`/`r`/`C`/`F`/`O`/`D`/`L`/`U`/`G`/`B`/`b`/`A`，各自说明略）
* `%#C`：满足某种条件 `C` 的项目数（与上同）
* `%dC`：该包依赖的其他包列表，通过 `C`（具体为 `n`/`o`/`v`）来选择输出 Name／Origin／Version
* `%rC`：依赖于该包的其他包列表，通过 `C`（具体为 `n`/`o`/`v`）来选择输出 Name／Origin／Version
* `%C`：包所属的分类列表
* `%FC`：包包含的文件列表，通过 `C`（`p`/`s`）来选择 Path 或校验和 Sum
* `%D`：包包含的目录列表
* `%OC`：包的选项列表，通过 `C`（`k`/`v`/`d`/`D`）来选择 key、value、默认值、描述
* `%L`：包的许可证列表
* `%U`：包使用的用户列表
* `%G`：包使用的用户组列表
* `%B`：包在程序中使用的共享库列表
* `%b`：包所提供的共享库列表
* `%AC`：包的注解标签列表，通过 `C`（`t`/`v`）选择 tag 或 value

常见的用法之一是 `%n-%v`，用于输出包名和版本的组合形式。

### pkg query -e

用于根据指定的查询条件筛选符合条件的软件包。以下是可用于右侧（不能用于左侧）条件表达式的字段：

* `%n`：包名（字符串型）
* `%o`：包的 origin（字符串型）
* `%p`：包的 prefix（字符串型）
* `%m`：包的维护者（邮箱，字符串型）
* `%c`：包的注释（字符串型）
* `%e`：包的描述（字符串型）
* `%w`：包的网址（字符串型）
* `%s`：包的大小（字节单位，数值型）
* `%a`：包的自动安装标志（数值型）
* `%q`：包的架构（操作系统、版本、CPU 架构，字符串型）
* `%k`：包的锁定标志（数值型）
* `%M`：包的信息消息（字符串型）
* `%t`：包的安装时间戳（数值型）
* `%i`：包的附加信息（字符串型）
* `%#C`：表示某类项目的数量（`C` 可为 `d`/`r`/`C`/`F`/`O`/`D`/`L`/`U`/`G`/`B`/`b`/`A`，说明略）

可使用的运算符如下：

* `||`：逻辑“或”，用于连接两个判断条件
* `&&`：逻辑“与”，用于连接两个判断条件
* `~`：用于匹配 glob 模式，例如 `%var ~ 模式`，判断是否匹配
* `!~`：用于判断是否不匹配 glob 模式
* `>` / `>=`：用于数值比较，大于或大于等于
* `<` / `<=`：用于数值比较，小于或小于等于
* `=` / `==`：判断是否等于（数字或字符串）
* `!=`：判断是否不等于（数字或字符串）
* `=~`：判断是否大小写不敏感地等于（数字或字符串）
* `!=~`：判断是否大小写不敏感地不等于（数字或字符串）

### pkg rquery \[R]

作用类似于 `pkg query`，但用于远程元数据。不再赘述，请参见 `pkg query` 的说明。

### pkg annotate \[L]

用于为软件包添加、修改或删除“注释信息标签”的命令。
这些注释信息可通过 `pkg info -A` 查看。至于注释标签的具体含义与使用方式，此处不做特别说明。

### pkg shell \[L]

用于直接访问管理软件包元数据的数据库（即 SQLite3）的命令。
基本上只能用来执行类似 `pkg shell VACUUM` 这样的操作。
如果你只是想利用查询功能，用 `pkg query` 就已经绰绰有余了，这个命令实用性很低。


## 便利的别名命令

### 与自动安装标志相关

以下命令是别名，用于筛选出“自动安装标志为 0”的软件包，即那些是用户显式安装的包：

* `pkg prime-list` \[L]
* `pkg prime-origins` \[L]
* `pkg noauto` \[L]

---

### pkg leaf \[L]

显示所有“叶子”软件包，即在依赖树最下层的包。
若某个软件包出现在这里但不出现在 `pkg noauto` 中，则它是 `pkg autoremove` 的候选对象。

---

### pkg list \[L]

显示指定软件包所安装的文件路径列表。

---

## 语法糖

以下是等价的命令别名，可以任选其一来执行操作。
甚至你可以用像 `pkg vanish`（消失）这种中二感满满的词自定义别名，只要你愿意。
系统默认提供了一些比较中性的别名。

* `pkg remove` 等价于 `pkg delete`


## 不需要在意的 pkg 命令们

这些命令大多是供内部使用的，或是给其他工具调用使用（比如把别的包管理系统的软件包信息注册到 FreeBSD 的 pkg 系统中）。
一般用户不需要关心：

* `pkg stats`：显示统计信息
* `pkg clean`：清理包缓存
* `pkg config`：查看 `/usr/local/etc/pkg.conf` 中的设置（大小写不敏感）
* `pkg create`：创建软件包。虽然在 Ports 构建过程中常用，但很少单独使用
* `pkg fetch`：下载包。`pkg install` 实际上等价于 `pkg fetch` 加上 `pkg add`
* `pkg register`：将包注册进本地数据库。纯粹是内部用途
* `pkg repo`：生成包仓库目录，相关内容建议参考 `pkg-repository(5)` 手册
* `pkg shlib`：显示软件包所提供的共享库以及依赖这些库的其他包，主要用于安全审计
* `pkg ssh`：完全用于内部，不适合人类用户使用，省略
* `pkg triggers`：执行延迟触发器。这是什么时候谁延迟的？总之也是内部使用，省略

## 查询（pkg-query）

### 想查出依赖于某个包，且是“显式安装”的所有软件包

以 Python 3.11 为例，说明如何列出“依赖 Python 3.11 且是用户明确指定安装”的所有软件包。

首先可以使用 `pkg info -r python311`（按包名指定）列出所有依赖 Python 3.11 的软件包。
也可以使用 `pkg query '%ro' python311` 以“原始名称”（origin）来列出。
此外，也可以直接使用如 `lang/python311` 这样的 origin 作为指定方式，而非包名。

接着，检查这些软件包的“自动安装标志”，只列出自动安装标志为 0 的，即明确安装的那些。
示例如下：

```
pkg query -e "%a = 0" "%n" $(pkg info -r python311)
  或者
pkg query -e "%a = 0" "%n" $(pkg query '%ro' python311)
```

由于没有很干净的一条命令就能完成所有功能的写法，因此也可以借助 `pkg shell` 来做（不推荐，供参考）。
注意：此方法依赖底层数据库结构，未来可能失效。

```
pkg shell -cmd "SELECT p.name FROM deps AS d INNER JOIN packages AS p ON package_id = p.id WHERE d.origin = 'lang/python311' AND p.automatic = 0" < /dev/null
```



## 常见问题及解答

### Ｑ．什么是 origin？

Ａ．Origin 是什么以后再说吧。有空我再写。

### Ｑ．pkg 命令本身是一个软件包吗？

Ａ．是的，`pkg` 命令本身就是一个包。

### Ｑ．那 `/usr/sbin/pkg` 是什么？

Ａ．那是一个“引导器”，只负责安装真正的 pkg 包。
实际会执行的是 pkg 包里的 `/usr/local/bin/pkg-static`。

### Ｑ．那它不属于基本系统吗？

Ａ．FreeBSD 曾经因为旧的软件包系统吃了太多苦，所以才采用了现在这种结构。
以前 pkg 命令不升级，整个系统就什么都干不了（即使新版本系统已经发布了，还会强制用老功能）。

### Ｑ．pkg 命令和 pkg-static 命令有什么区别？

Ａ．功能完全一样。`pkg-static` 主要在某些“极端情况”（如你要删掉 pkg 本身）时才需要。
一般正常使用用 `pkg` 就够了；`pkg-static` 只是为了某些内部处理时更安全。

### Ｑ．如何删除通过 `pkg update` 获取的仓库信息？

Ａ．我也很感兴趣，目前还在调查中。

### Ｑ．用 `pkg query` 过滤总是不太顺利啊。比如说，想列出“最后安装的包”怎么办？

Ａ．这种时候可以用 `pkg shell` 来努力一下。虽说“让我们来写 SQL 吧！”这听起来有点夸张，但你说得没错，`pkg query` 并不支持排序或聚合。
另一种思路是，把 `pkg query` 的输出通过管道传给其它工具，比如 `sort`、`awk`、`head`、`tail` 等组合使用。

### Ｑ．你真的全都记得住？

Ａ．常用的当然记得住。原以为我也就记了三五个而已，没想到写着写着就十几个浮现脑海。说明这些的确值得记。
虽然有些细节的语义可能模糊了，但那种情况就查下手册吧。
总之，平时不常用的，忘了也无妨。

顺便说一下，`pkg query` 和 `pkg shell` 以前我其实用得不多，这次因为研究查数据才顺手补上的。虽然 `pkg shell` 基本就是只查结果而已。

### Ｑ．pkg update / upgrade / updating 总是傻傻分不清楚。

Ａ．很遗憾，这是必须记下来的。
其他包管理器有时 `update` 和 `upgrade` 是同一个东西，但在 FreeBSD 上……很遗憾（信息在此中断了）。

### Ｑ．WITH\_PKGNG 和 pkg2ng 是什么？

Ａ．这是非常古早（比 10.0-RELEASE 还早）的事情，完全可以无视。
它们只具备历史意义，现在的设置和运维里已经毫无价值。

---

## 小吐槽

刚列完这些“大家应该都知道吧”的命令，立马陷入绝望：平时常用到底是指哪些……？
但如果不写这些，又感觉内容不完整，简直是地狱。
结果还在翻 man 手册时发现了个 bug（[pkg-query(8): duplicate %#b](https://github.com/freebsd/pkg/issues/2213)），orz。

## 参考文献

* [FreeBSD pkg](https://github.com/freebsd/pkg)
* [pkg(8)](https://man.freebsd.org/cgi/man.cgi?pkg)
* [pkg update(8)](https://man.freebsd.org/cgi/man.cgi?pkg-update)
* [pkg install(8)](https://man.freebsd.org/cgi/man.cgi?pkg-install)
* [pkg add(8)](https://man.freebsd.org/cgi/man.cgi?pkg-add)
* [pkg delete(8)](https://man.freebsd.org/cgi/man.cgi?pkg-delete)
* [pkg version(8)](https://man.freebsd.org/cgi/man.cgi?pkg-version)
* [pkg upgrade(8)](https://man.freebsd.org/cgi/man.cgi?pkg-upgrade)
* [pkg info(8)](https://man.freebsd.org/cgi/man.cgi?pkg-info)
* [pkg lock(8)](https://man.freebsd.org/cgi/man.cgi?pkg-lock)
* [pkg unlock(8)](https://man.freebsd.org/cgi/man.cgi?pkg-unlock)
* [pkg-search(8)](https://man.freebsd.org/cgi/man.cgi?pkg-search)
* [pkg-which(8)](https://man.freebsd.org/cgi/man.cgi?pkg-which)
* [pkg-audit(8)](https://man.freebsd.org/cgi/man.cgi?pkg-audit)
* [pkg-check(8)](https://man.freebsd.org/cgi/man.cgi?pkg-check)
* [pkg-autoremove(8)](https://man.freebsd.org/cgi/man.cgi?pkg-autoremove)
* [pkg-updating(8)](https://man.freebsd.org/cgi/man.cgi?pkg-updating)
* [pkg-set(8)](https://man.freebsd.org/cgi/man.cgi?pkg-set)
* [pkg-alias(8)](https://man.freebsd.org/cgi/man.cgi?pkg-alias)
* [pkg-query(8)](https://man.freebsd.org/cgi/man.cgi?pkg-query)
* [pkg-rquery(8)](https://man.freebsd.org/cgi/man.cgi?pkg-rquery)
* [pkg-anotate(8)](https://man.freebsd.org/cgi/man.cgi?pkg-anotate)
* [pkg-shell(8)](https://man.freebsd.org/cgi/man.cgi?pkg-shell)
* [pkg-stats(8)](https://man.freebsd.org/cgi/man.cgi?pkg-stats)
* [pkg-clean(8)](https://man.freebsd.org/cgi/man.cgi?pkg-clean)
* [pkg-config(8)](https://man.freebsd.org/cgi/man.cgi?pkg-config)
* [pkg-create(8)](https://man.freebsd.org/cgi/man.cgi?pkg-create)
* [pkg-fetch(8)](https://man.freebsd.org/cgi/man.cgi?pkg-fetch)
* [pkg-register(8)](https://man.freebsd.org/cgi/man.cgi?pkg-register)
* [pkg-repo(8)](https://man.freebsd.org/cgi/man.cgi?pkg-repo)
* [pkg-shlib(8)](https://man.freebsd.org/cgi/man.cgi?pkg-shlib)
* [pkg-ssh(8)](https://man.freebsd.org/cgi/man.cgi?pkg-ssh)
* [pkg-triggers(8)](https://man.freebsd.org/cgi/man.cgi?pkg-triggers)
* [pkg-repository(5)](https://man.freebsd.org/cgi/man.cgi?pkg-repository)
* [FreeBSD Ports Flavors](https://www.slideshare.net/YuichiroNaito/freebsd-ports-flavors)
* [FreeBSD Porter's Handbook - 7. Flavors](https://docs.freebsd.org/en/books/porters-handbook/flavors/)
* [FreeBSD Porter's Handbook - 5.2.2. Versions, DISTVERSION or PORTVERSION](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-versions)
* [FreeBSD Porter's Handbook - 13. pkg-\* files](https://docs.freebsd.org/ja/books/porters-handbook/pkg-files/)
* [FreeBSD pkg コマンドチート](https://qiita.com/taku39@github/items/c29a47d5ac07492d4857)
* [【FreeBSD】pkgで普段使うサブコマンド達](https://hacolab.hatenablog.com/entry/2020/02/16/170000)
* [pkg](https://kaworu.jpn.org/freebsd/pkg)
* [FreeBSD – パッケージソフトのインストールと削除、アップデートのやり方](https://blog.it-see.net/it-dokata/freebsd/pkg/)
* [パッケージコマンド早見表](https://freebsd.seirios.org/doku.php?id=ports:pkg_yum_apt)
* [アプリケーションをインストールする (前編): packages](https://retrotecture.jp/freebsd/app1_packages.html)
* [FreeBSD 13.0 RELEASE - ports・pkg - pkg](https://freebsd.sing.ne.jp/fbsd/1300/03/03.html)
  
