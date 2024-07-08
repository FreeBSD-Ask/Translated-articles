# FreeBSD pkg 命令概述

<https://qiita.com/nanorkyo/items/2ff7cccfe3bc544f6f5e>

* [FreeBSD](https://qiita.com/tags/freebsd)
* [ 软件包](https://qiita.com/tags/pkg)
* [ 软件包管理](https://qiita.com/tags/%e3%83%91%e3%83%83%e3%82%b1%e3%83%bc%e3%82%b8%e7%ae%a1%e7%90%86)

最后更新于 2023-12-28 发布于 2023-12-25

# 开始

通过改进第三方工具安装机制（ports 和 packages），FreeBSD 中的二进制包使用已经得到改善。特别是引入了 flavors 功能，尽管安装的内容是相同的，但由于依赖的包版本变化，可能会导致组合变得复杂的情况，现在可以简单地准备包。只要针对所有被依赖的包的版本都有适合的包... 这里总结了经常使用的命令，以享受二进制包带来的好处。然而，细致到每个选项会很麻烦，所以只明确列出常用选项。另外，大多数命令都会询问是否执行。因此，几乎所有命令都有一个常见的 -y 选项，但在这方面的说明将省略。熟悉后再使用。此外，我没有写太多执行示例。如果不清楚如何使用等问题，请告诉我。我会补充详细说明。

# 常用的 pkg 命令

在这里，我们为命令名称（子命令名称）添加 [R] 和 [L] 符号。具体如下。

* [R] ：引用外部包／存储库中的信息。
* [L] ：查看已安装或存在于本地文件系统中的软件包的信息。
* [RL] ：根据环境表现出与 [R] 和 [L] 相适应的行为。

## 软件包更新 [R]

用于收集软件包元数据的命令。首先需要收集元数据，这在某种程度上类似于 yum check-update 。 值得一提的是，并非总是需要运行的命令，因为有一些命令会自动更新元数据后再执行，如果这不是问题的话，就无需运行。具体来说，以下子命令将根据最新的元数据执行。

* `pkg install`
* `pkg upgrade`
* `pkg version`
* `pkg search`
* `pkg fetch`
* `pkg rquery`

## 软件包安装 [R]

进行软件包安装。与前述 pkg add 的区别在于，使用关键词进行特定及通过网络获取软件包安装。

## 软件包添加 [L]

执行指定软件包文件的安装。与前述 pkg install 不同之处在于，明确指定软件包文件名进行安装。

## pkg 删除 [L]

 删除指定的软件包。

### pkg 删除 -ayf

将无情且毫不留情地删除所有（ -a ）软件包。直观来看，只需使用 -ay 选项就足够了，但由于依赖关系的原因，可能需要使用 -f 选项来处理无法删除的情况。如果打算全部删除，就毫不犹豫地全部指定吧。

## pkg 版本 [RL]

将进行软件包版本比较。简单来说，由于它有些古怪，我们将分解用例并进行解释。

将与名为 /usr/ports/INDEX-ＯＳメジャーバージョン 的元文件进行比较。如果该文件不存在，则将参考每个ports的源（目录）（相当于指定 -P 选项）。如果该目录不存在，则将参考远程存储库（相当于指定 -R 选项）。

在前两种情况下，相当于 [L] ，在后一种情况下相当于 [R] 。

### 包版本 -L =

通常使用 -L 选项进行比较已安装的软件包，并显示等于「不存在」的内容。惯用法上，只指定 = 就足够了。除了参考 = 之外，还可以指定 > 、 < 、 ? 和 ! ，但所有这些字符都需要用引号括起来。

各个字符的含义如下，对于 -L 选项，意味着除了该字符以外的所有内容。反过来，如果需要该字符的含义，则使用 -l 选项。

* > ：如果发布了比当前安装的更新版本
  >
* = ：如果发布了与当前安装的相同版本
* < ：如果发布了比当前安装的更旧版本，通常在更新元数据不完整、新 ports 测试安装等情况下发生
* ? ： 由于当前安装的软件包的源已不存在（已删除或重命名），无法进行版本比较
* ! ： 无法与当前安装的版本进行版本比较

### 包版本 -t

指定任意版本号（即普通字符串）进行比较，并返回结果。这不仅仅是针对软件包的功能，而是为了测试多种版本号而设计的功能。与其他子命令相比，这是一种不同的功能。特别是在替换版本指定如α版、β版时，用于尝试和错误解决不确定哪个更大（最新）、哪个更小（最旧）的情况。有关版本比较规则，请参阅 FreeBSD Porter's Handbook - 5.2.2. Versions, DISTVERSION or PORTVERSION。

## 包升级 [R]

更新指定 更新指定的软件。如果没有特别指定，将会检查并更新所有包。

## 包信息 [L]

指定されたパッケージ情報を表示します。指定されない場合、全てのパッケージの概要を一覧表示します。

### 包信息 -a

显示指定的包信息。如果没有指定，将列出所有包的概览。

### 包信息 -l

显示安装了指定软件包的路径列表。

### 软件包信息 -D

显示与指定软件包相关的注意事项。可能包含设置提示信息。

## 锁定软件包 [L]

锁定指定软件包，以防止其更新。此锁定的影响优于防止软件包本身的更改。

* 锁定的软件包阻止重新安装、升级、降级或删除
* 如果依赖于锁定的软件包的其他软件包依赖于锁定的软件包的另一个版本，则阻止该软件包的升级或降级
* 阻止依赖于锁定软件包的软件包的删除、升级或降级
* 对于与上述间接有依赖关系的情况，也将被阻止

## 解锁软件包 [L]

 解除指定软件包的锁定。

## 软件包搜索 [R]

不区分大小写，通过正则表达式搜索软件包名称。您可以使用 -e 选项进行严格搜索，但这将包括版本号搜索，因此并不是很用户友好。 -g 选项允许进行 shell glob（通过 ? 或 * 进行模糊搜索）。但是，由于包含版本号搜索，实际上最后需要 * 。

## 软件包定位 [L]

显示所属于指定文件、目录等的软件包名称。您可以使用 -o 选项来显示原始位置。

# 很少使用但最好知道的 pkg 命令

## 软件包审计 [RL]

报告已安装软件包与脆弱性信息的比对结果。由于需要从外部获取脆弱性信息，因此这有些像 [R] ，但如果您没有紧急需要立即更新信息，不必指定 -F 选项来获取脆弱性信息。这通常是因为每天有一个脆弱性信息自动获取脚本在取得脆弱性信息。

## 软件包检查 -s [L]

检查指定的软件包是否被篡改，以确保一致性。通常与 -a 选项一起使用。

## 软件包检查 -d [L]

检查指定软件包的依赖项是否损坏，以确保一致性。通常与 -a 选项一起使用。

## 软件包自动移除 [L]

删除孤立的软件包。所谓“孤立”是指由于其他软件包的依赖关系而安装的软件包，随后该软件包被删除，或者（通过更新等方式）不再被依赖，不再被使用。

可能有一些情况下，实际上会使用到“孤立”的软件包，但这并不能从软件包依赖信息中读取出来，因此在使用时需要格外注意。例如通常使用 bash ，但由于某些依赖关系安装了 bash ，因此直接使用它可能需要注意。可以明确安装 bash ，或者通过后续的技巧来解决。

对于 ports 系统而言，在构建时需要的主要软件包（如编译器等）可能会被删除。

## 包更新[L]

显示软件包更新时需要注意的步骤（手动更新指示）。具体来说，通过阅读 /usr/ports/UPDATING 并与已安装的（或指定的）软件包信息匹配，以显示可能需要的部分。这个文件主要面向人类阅读，因此存在相当的启发式设计，严密性可能有所欠缺。同时，需要这个文件的情况下，特意下载它也有些微妙。

## 包设置[L]

更改指定软件包的元数据（数据库）。通常情况下，这是基于 pkg updating 信息执行的操作，因此不应该由用户主动操作。

### 包设置-A

使用 -A 选项并指定以下值，以更改指定软件包的元数据。此更改将影响 pkg autoremove 的结果。

* 0 ：将指定的软件包排除在 pkg autoremove 的范围之外。
* 1 ： 指定的软件包将定制为 pkg autoremove 的目标。

对于对此项感兴趣的人，可能会关心 pkg autoremove 的工作原理，但由于此标志必须有效且不受其他软件包依赖的影响，因此仅靠此标志无法使 pkg autoremove 成为目标。

同样，直接由 pkg install 或 pkg add 指定的内容将在安装时设置为无效此标志。此标志仅对通过依赖关系安装的软件包有效。

### pkg set -o

通过 -o 旧オリジン:新オリジン 的形式，修改依赖于旧原始包的所有软件包的元数据中的原始信息。

### pkg set -n

通过 -n 旧パッケージ名:新パッケージ名 的形式，修改软件包名。不过，由于软件包名中是否包含版本号存在争议，因此这存在一些不确定性。

# 过于偏门，不知道也没关系的 pkg 命令

## pkg alias [L]

在 /usr/local/etc/pkg.conf 中显示定义的别名信息。

## 包查询 [L]

以指定的格式显示软件包的元信息。如果未指定软件包名称，则将显示所有软件包的信息。可以使用以下格式字符串：

* %n ：软件包的名称
* %v ： 软件包版本
* %o ： 软件包来源
* %p ： 软件包前缀
* %m ：软件包的维护者（电子邮件）
* %c ：软件包的注释
* %e ：软件包的描述
* %w ： 软件包的网站
* %l ： 软件包的许可逻辑（无、单一、和、或）
* %sb 或 %sh ： 软件包的大小（以字节为单位或易读的单位）
* %a ： 软件包自动安装标志
* %Q ： 软件包架构支持候选
* %q ： 软件包架构（操作系统、版本、CPU 架构）
* %k ： 软件包的锁定标志
* %M ： 软件包的消息
* %t ： 软件包安装时的时间戳
* %R ：软件包安装来源存储库
* %X ：软件包检查和校验和
* %?Ｃ ： Ｃ （具体是 d / r / C / F / O / D / L / U / G / B / b / A ）中指定情况下的标志（说明略）
* `%#Ｃ`： `Ｃ`（具体的には `d`/`r`/`C`/`F`/`O`/`D`/`L`/`U`/`G`/`B`/`b`/`A`・説明は省略）として指定された内容の数
* %dＣ ： 定制的依赖包列表，通过 Ｃ （具体的には n / o / v ）获取 N 名称、 O 来源、 V 版本信息
* %rＣ ： 包的反向依赖包列表，通过 Ｃ （具体的には n / o / v ）获取 N 名称、 O 来源、 V 版本信息
* %C ： 软件包类别列表
* %FＣ ： 软件包文件列表， Ｃ （具体是 p / s ）通过 P ath 或 S um 获取
* %D ： 软件包目录列表
* %OＣ ： 获取软件包选项列表，通过 Ｃ （具体地说，通过 k / v / d / D ）获得 k ey、 v alue、 d efault value、 D escrition
* %L ： 获得软件包许可证列表
* %U ： 获得软件包用户列表
* %G ： 软件包组列表
* %B ： 软件包程序使用的共享库列表
* %b ： 软件包提供的共享库列表
* %AＣ ：包的注释标签列表，通过 Ｃ （具体地说是 t / v ）获取 t ag、 v alue

### 包查询 -e

缩小到与指定查询匹配的包。如果心情好的话，会详细写的。

### 包 r 查询 [R]

对远程元数据的 pkg query 进行操作。这里暂时不详细说明。

### 包注释 [L]

为软件包添加、修改或删除“注释信息标签”的命令。您可以使用 pkg info -A 查看显示效果。没有特别说明注释标签的含义和使用方法。

## 包 shell [L]

直接访问管理软件包元数据的数据库（即 SQLite3）的命令。大概只有 pkg shell VACUUM 这么一点点用途。如果想要利用查询， pkg query 就足够像找零钱一样的工具。

# 便利的别名命令

## 自动安装标志相关

以下别名参考自动安装标志，返回自动安装标志为 0 的明确指定安装的命令。

* `pkg prime-list` [L]
* `pkg prime-origins` [L]
* `pkg noauto` [L]

## 包叶 [L]

显示依赖树中叶子节点的软件包。不包含在 pkg noauto 中但属于 pkg autoremove 的对象。

## 软件包列表 [L]

指定的软件包安装路径列表。

## 语法糖

仅仅是可供任选调用的别名。甚至可以用类似 pkg vanish 这种充满中二病氛围的 神圣而强大的词汇来定义子命令，也是毫无问题的。系统会默认定义一些安全的值。

* `pkg remove` ＝ `pkg delete`

# 我不知道这个 pkg 命令。

这些命令大多用于内部，或者被其他工具调用（例如从其他软件包管理系统到 FreeBSD pkg 系统的注册等），即使不了解也没有问题。

* pkg stats ：显示统计信息。
* pkg clean ：清除软件包缓存。
* pkg config ：引用设置为 /usr/local/etc/pkg.conf 的内容（不区分大小写）。
* pkg create ：创建软件包...尽管在ports构建时经常使用，但单独使用的情况并不常见。
* pkg fetch ： 获取软件包。 pkg install = pkg fetch + pkg add 的关系。
* pkg register ： 将软件包注册到本地数据库。这完全是内部使用的东西。
* pkg repo ： 创建软件包存储库目录。在此处，请参考 pkg-repository(5) 创建存储库。
* pkg shlib ：显示提供的共享库以及依赖于该库的其他软件包。这主要用于安全诊断。
* pkg ssh ：这是为完全内部使用而不是为人类使用的命令，因此略过。
* pkg triggers ：延迟触发的执行。由于涉及内部使用，因此省略了谁延迟了这个问题？

# 起源

 起源是 TODO。想写的话会写。

# 常见问题及其答案

## Q．pkg 命令是一个软件包吗？

A．是的。pkg 命令是一个软件包。

## Q．哦？/usr/sbin/pkg 是什么？

A.这只是一个安装 pkg 软件包的引导。然后调用 pkg 软件包（ /usr/local/bin/pkg-static ）。

## 问：噢？这不会包含在基本系统中吗？

答：由于旧软件包系统给我们带来了许多麻烦，因此演变成了这种形式。如果 pkg 命令没有进步，我们将一事无成（即使发布新版本也会被迫使用旧功能）。

## Ｑ．pkg 命令和 pkg-static 命令有什么区别？

它们完全相同。pkg-static 在执行一些微妙操作，比如删除自身时才会需要。通常情况下，使用 pkg 命令即可。在某些内部处理中，为了安全起见才会使用 pkg-static。

## Ｑ．如何删除通过 pkg update 获取的仓库信息？

 Ａ．自己也对此感兴趣，正在进行调查。

## Ｑ．使用 pkg query 时无法有效筛选结果。在需要输出最近安装的软件包列表的情况下，应该如何处理？

Ａ．让我们尽力而为吧。尽管我们不谈什么“让我们 SQL 吧！”，但确实如你所说，pkg query 没有输出顺序或汇总功能。或许可以考虑通过管道连接其他工具（例如 sort、awk、grep 等）来解决问题。

## Ｑ．这些你真的都记得吗？

Ａ．常用的我记得。我自己以为记得的可能就数个，但实际上大约有十几个左右，所以还是得依赖记忆写下来，觉得值得去记忆。有些细微的差别可能有点微妙，那就得仔细查阅手册了。总之，平常不使用的东西就放在记忆的深处也没问题。

## Ｑ．pkg update/upgrade/updating 有点难理解。

Ａ．很遗憾，请记住，这是另一个包系统， update 和 upgrade 可能是相同的，但很不幸，消息在这里中断了

## Ｑ．WITH_PKGNG 或者 pkg2ng 是什么？

这是非常古老的事情（在 10.0-RELEASE 之前），请忽略这部分内容。它只有历史价值，对设置和运营毫无价值。

# 愚痴

列出了这些我都知道的东西后的绝望感。日常使用是什么？但如果不列出这些的话……地狱感。并且在读手册时发现了一个 bug（pkg-query(8)：重复的%#b）。orz

# 参考文献

* [ FreeBSD 软件包](https://github.com/freebsd/pkg)
* [pkg(8)](https://man.freebsd.org/cgi/man.cgi?pkg)
* [pkg update(8)](https://man.freebsd.org/cgi/man.cgi?pkg-update)
* [pkg install(8)](https://man.freebsd.org/cgi/man.cgi?pkg-install)
* [ pkg 添加(8)](https://man.freebsd.org/cgi/man.cgi?pkg-add)
* [ pkg 删除(8)](https://man.freebsd.org/cgi/man.cgi?pkg-delete)
* [ pkg 版本(8)](https://man.freebsd.org/cgi/man.cgi?pkg-version)
* [ 升级软件包(8)](https://man.freebsd.org/cgi/man.cgi?pkg-upgrade)
* [ 软件包信息(8)](https://man.freebsd.org/cgi/man.cgi?pkg-info)
* [ 锁定软件包(8)](https://man.freebsd.org/cgi/man.cgi?pkg-lock)
* [ pkg 解锁(8)](https://man.freebsd.org/cgi/man.cgi?pkg-unlock)
* [ pkg 搜索(8)](https://man.freebsd.org/cgi/man.cgi?pkg-search)
* [ pkg 查询(8)](https://man.freebsd.org/cgi/man.cgi?pkg-which)
* [ pkg-audit(8) 软件包审计](https://man.freebsd.org/cgi/man.cgi?pkg-audit)
* [ pkg-check(8) 软件包检查](https://man.freebsd.org/cgi/man.cgi?pkg-check)
* [ pkg-autoremove(8) 软件包自动移除](https://man.freebsd.org/cgi/man.cgi?pkg-autoremove)
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
* [ 包 ssh（8）](https://man.freebsd.org/cgi/man.cgi?pkg-ssh)
* [ 包触发器（8）](https://man.freebsd.org/cgi/man.cgi?pkg-triggers)
* [ 包存储库（5）](https://man.freebsd.org/cgi/man.cgi?pkg-repository)
* [ FreeBSD Ports Flavors 定制](https://www.slideshare.net/YuichiroNaito/freebsd-ports-flavors)
* [FreeBSD Porter&apos;s Handbook - 7. Flavors](https://docs.freebsd.org/en/books/porters-handbook/flavors/)
* [FreeBSD Porter&apos;s Handbook - 5.2.2. Versions, DISTVERSION or PORTVERSION](https://docs.freebsd.org/en/books/porters-handbook/makefiles/#makefile-versions)
* [FreeBSD Porter&apos;s Handbook - 13. pkg-* files](https://docs.freebsd.org/ja/books/porters-handbook/pkg-files/)
* [ FreeBSD pkg 命令速查表](https://qiita.com/taku39@github/items/c29a47d5ac07492d4857)
* [【FreeBSD】pkg 常用的子命令](https://hacolab.hatenablog.com/entry/2020/02/16/170000)
* [pkg](https://kaworu.jpn.org/freebsd/pkg)
* [FreeBSD – 软件包安装、卸载和更新方法](https://blog.it-see.net/it-dokata/freebsd/pkg/)
* [ 软件包命令速查表](https://freebsd.seirios.org/doku.php?id=ports:pkg_yum_apt)
* [安装应用程序 (上): packages](https://retrotecture.jp/freebsd/app1_packages.html)
* [FreeBSD 13.0 RELEASE - ports・pkg - pkg](https://freebsd.sing.ne.jp/fbsd/1300/03/03.html)
