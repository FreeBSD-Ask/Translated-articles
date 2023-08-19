# FreeBSD Ports 中的 OPTIONS 功能介绍——使用 OPTIONS_SET/OPTIONS_UNSET/NO_DIALOG 进行操作和实践

- 原标题：FreeBSD Ports Collection における OPTIONS 機能の紹介 - OPTIONS_SET/OPTIONS_UNSET/NO_DIALOG の活用と実践 -
- 原地址：<https://qiita.com/nanorkyo/items/a0068cafcf9112ebbb7b>
- 作者：重村法克
- 最后发布日期：2021 年 4 月 4 日
- 译者：ykla & ChatGPT

## 前言

都已经是 2020 年了，你还在进行编译吗？

在当前的 FreeBSD 环境中，不仅基本系统，就连第三方的二进制包的使用（安装）也在持续增加。事实上，如果只是进行轻量级的设置和使用，已经有足够多的二进制包可供选择了。

然而，就我个人而言，由于我进行了大量的定制以适应我的需求，我无法满足于仅仅使用二进制包……这就是为什么我整理了有关自定义 Ports 的方法。

至于那些想知道“什么是 ports（FreeBSD ports）”的读者，请参考所引用的资料。再次抱歉。

## 适用环境

由于已经过去了五年之久，到了 2020 年，因此在目前支持的所有 FreeBSD 环境中都可以使用。大致可以考虑从版本 10 开始的版本。

## OPTIONS 机制

通过引入 OPTIONS（`/usr/ports/Mk/bsd.options.mk`）【注 1】，在使用 Ports 编译第三方应用程序时，定制的方法得到了统一。

在引入这个机制之前和之后，Makefile 的编写方式发生了变化，OPTIONS 引入之前的机制（基本上）已经不复存在。【注 2】

嗯，严格地说经过详细的研究，这个机制还是有残留的，或者说有时候也会被使用，但由于它过于特殊，主要用于一些黑客式的定制目的，所以在这里我们将跳过不谈。

## OPTIONS 的含义

OPTIONS 是用于控制每个 Port 可定制的项目的开关机制，为以下控制提供了框架：

1. 可定制的内容是什么（选项名称）
2. 这些选项代表着什么功能（说明）
3. 哪些选项默认开启或关闭（默认值）
4. 关于行为的辅助信息（明确的依赖关系和安装方法等）

通过在 Makefile 中记录这些信息， Port 的维护者可以使用户能够轻松地通过开启或关闭选项来进行定制。"

## 通过对话框进行选择

在编译时会显示对话框，您可以以交互方式选择这些定制项的启用或禁用。具体来说，可以通过以下命令进行配置，这些配置项如下：【注 3、4】。

```
make config
make config-recursive
make showconfig
make showconfig-recursive
make rmconfig
make rmconfig-recursive
```

特别是，在对话框中确认了选项后，只有在明确执行 `make config` 命令或者在新增选项时才会再次显示对话框。

另外，如果您想要恢复默认设置，可以使用 `make rmconfig` 命令，这将会清除在对话框中进行的配置。

## 全局设置

您可以通过在 `/etc/make.conf` 文件中进行配置，从而全局性地定义定制内容（开启或关闭）。
如果不需要 X11，或者想要在所有 Port 中统一地使用 LZ4 作为压缩算法等情况，可以在这里进行设置。

`/etc/make.conf`:

```
OPTIONS_SET+=   LZ4 LZO LZO2 LZMA ZLIB ZSTD BROTLI
OPTIONS_UNSET+= X11 CUPS GDBM TEST DEBUG TESTS
```

## 针对全局的 Port 特定设置

在 `/etc/make.conf` 文件中，您还可以进一步针对各个 Port 进行定制设置。
例如，基本上您可能想要安装文档类文件，但在为了安装这些文档而开始安装 TeX 的情况下，您可能会感到不满意，希望不安装文档。在这种情况下，您可以通过 `${OPTIONS_NAME}_SET或${OPTIONS_NAME}_UNSET` 来定义。

`/etc/make.conf`:

```
emulators_open-vm-tools_UNSET= DOCS FUSE LIBNOTIFY
```

`${OPTIONS_NAME}` 是通过在每个 Port 上运行 `make -VOPTIONS_NAME` 命令获取的字符串。就像上面的例子一样，这些变量名中可能包含 `-` 或 `+`，但在 Makefile 中似乎没有问题。

值得注意的是，在引入 OPTIONS 时，最初使用的是 `UNIQUENAME` 而不是 `OPTIONS_NAME`。但由于存在与其他机制的冲突，已经更改为使用 OPTIONS_NAME。

## "bsd.options.mk 中的默认设置

在 bsd.options.mk 中，以下设置与 Port 的默认设置无关，始终会启用：

- DOCS
- NLS
- EXAMPLES
- IPV6

尽管在对话框的选择中可能有`DOCS`选项，但即使在 Port 的默认设置中没有`DOCS`，实际上`DOCS`仍然会被启用，这是由于这些设置的影响所致。

## OPTIONS 的优先级顺序如下：

1. `/etc/make.conf` （`${OPTIONS_NAME}_SET_FORCE`／`${OPTIONS_NAME}_UNSET_FORCE`）`
2. `/etc/make.conf` （`OPTIONS_SET_FORCE`／`OPTIONS_UNSET_FORCE`）
3.` make config`
4. `/etc/make.conf` （`${OPTIONS_NAME}_SET`／`${OPTIONS_NAME}_UNSET`）
5. `/etc/make.conf` （`OPTIONS_SET`／`OPTIONS_UNSET`）
6. 使用 `make WITH="选项 1 选项 2..." WITHOUT="选项 1 选项 2..."` 的情况 【注 5】
7. `bsd.options.mk`
8. `Makefile／Makefile.local`

值得注意的是，`_SET` 的处理会先于 `_UNSET` 的处理，因此优先级是 `_UNSET` > `_SET`，`_UNSET` 的影响会更强烈。

## 无需定制！

换句话说，软件包系统应该是在没有定制的情况下进行编译的，并且应该在不显示对话框的情况下进行处理！【注 6】

实际上，确实存在这样的选项。如果在 `/etc/make.conf` 中按照以下方式写入，将会在所有 Port 上启动编译时无需显示对话框（≒ 无需定制）：

请注意，"无定制"\"无对话框显示"和其他上下文中的翻译会因语境不同而有所变化。

```
NO_DIALOG= yes
```

实际上，只是跳过了对话框的显示，而之前执行的 `make config` 的结果以及前面提到的 `OPTIONS_SET`／`OPTIONS_UNSET` 等设置将会被应用，并在处理中生效。

从这个意义上说，虽然不是完全的默认设置，但通过这种方式提供了与二进制包兼容的操作方式，这一点应该是可以理解的。

## ports-mgmt/portconf

`portconf` 是一个工具，可以通过 `/usr/local/etc/ports.conf` 这个设置文件来为每个 Port 进行定制设置。

在安装时，通过将钩子放入 `/etc/make.conf`，并通过连接 `/usr/local/etc/ports.conf` → `/usr/local/libexec/portconf` → `/etc/make.conf`，实现了仿佛一开始就在那里设置的效果。因此，请不要从 `/etc/make.conf` 中删除此钩子。

其实现原理是通过比较（使用通配符进行匹配）设置文件中的 Port 目录名和当前目录，如果匹配，则会将相应的内容导入。

例如，在以下示例中，如果在 `/usr/ports/databases/sqlite3` 目录中运行 `/usr/local/libexec/portconf`，您将会看到显示出 `|NO_DIALOG=yes` 的内容。

`/usr/local/etc/ports.conf` :

```
databases/sqlite3: NO_DIALOG=yes
lang/perl5.*:      NO_DIALOG=yes
```

就像在这个示例中对 SQLite3 所做的那样，可以进行相当精细的定制，但有时可能会出现破坏二进制兼容性的自定义（在这种情况下，默认关闭），所以最好不要轻易进行定制，特别是对于那些最好不要定制的 Port 。

## 与 OPTIONS 的区别和用法

使用 OPTIONS 的时候，可以像 `${OPTIONS_NAME}_SET／`${OPTIONS_NAME}_UNSET` 这样，针对每个 Port （`${OPTIONS_NAME}`）明确地使用 `OPTIONS_SET`／`OPTIONS_UNSET`。然而，像 `NO_DIALOG` 这样的全局名字无法在每个 Port 上单独进行设置。因此在这些情况下，会使用 portconf。

当然，您也可以在 ports.conf 中编写 `OPTIONS_SET`／`OPTIONS_UNSET`，而不使用 `${OPTIONS_NAME}_SET`／`${OPTIONS_NAME}_UNSET`。但是需要注意，如果在未安装 portconf 的环境中进行构建，其影响可能会很大。因此，更好的做法是在影响不大的地方熟练地使用它。

如果您理解了上述内容，对于其他指定方式，您可以执行 `pkg info -D portconf`，然后阅读内容来进行编写。

## 常见问题及答案

- 问：从 Port 编译的软件与二进制包可以共同使用吗？
- 
答：当所有 Port 都使用 NO_DIALOG 且没有进行定制时，是可以的。
如果您想要快速反映变更，建议使用 Port 编译，但请注意不要进行定制。

- 问：定制过的 Port 和二进制包可以共同使用吗？
- 
答：基本上不建议这样做。虽然在某些情况下可能会有限制性的共用，但需要经过试错来实现。此外，这种试错的结果并不能保证会持续有效。
定制可能会导致某些功能无法正常工作或者缺失，这种问题通常在依赖关系上不能被察觉，而只能在安装后（运行时）才能被发现（因为二进制不兼容）。
此类更改（定制）可能会比版本升级带来更大的影响。

当然，也可能会有一些例外情况。在被称为中间件（尤其是库）的 Port 上，如果某些功能“不存在”，而在被称为应用程序的 Port 上“存在”，那么它们可能可以正常工作。

- 问：开启和关闭，SET 和 UNSET 之间好像有些突然跳跃的感觉！
- 
答：没关系，没问题。您的理解是正确的。虽然确实需要做出一些解释，但我有点累了，所以我要睡觉了。"

## 参考文献

- [[FreeBSD-Ports-Announce] Configuring options in make.conf](https://lists.freebsd.org/pipermail/freebsd-ports-announce/2013-June/000062.html)https://lists.freebsd.org/pipermail/freebsd-ports-announce/2013-June/000062.html
- アプリケーションのインストール - packages と ports [【日本語版】](https://www.freebsd.org/doc/ja_JP.eucJP/books/handbook/ports.html) [【英語版】7](https://www.freebsd.org/doc/handbook/ports.html)
- Ports Collection の利用 [【日本語版】](https://www.freebsd.org/doc/ja_JP.eucJP/books/handbook/ports-using.html)[【英語版】7](https://www.freebsd.org/doc/handbook/ports-using.html)
- [bsd.port.mk](https://github.com/freebsd/freebsd-ports/blob/master/Mk/bsd.port.mk)
- [bsd.port.options.mk](https://github.com/freebsd/freebsd-ports/blob/master/Mk/bsd.port.options.mk)
- [portconf](https://www.freshports.org/ports-mgmt/portconf/)

【注 1】"自 2012 年 5 月以来。当然，虽然这个机制刚被引入时，并不是所有的 Port 都支持，但已经过去了 8 年多的时间...

【注 2】像 WITHOUT_X11 这样的主要定制设置... 已经不复存在...（确切地说是错误的使用方式）。f( --;

【注 3】config 是设置，showconfig 是查看当前设置，rmconfig 是删除设置。-recursive 是用来逐层处理各依赖关系。

【注 4】通过 WITH 指定的选项会被设置，通过 WITHOUT 指定的选项会被取消设置。

【注 5】我稍微研究了一下包构建系统的运作方式，但没有找到关于如何运作的信息，只是知道在使用工具以及由于预测认为无法定制而不会进行定制。这只是我个人的主观判断，并且没有任何责任。

【注 6】由于日语版的描述已经过时，所以在阅读英语版时可以作为参考。尤其是关于 pkgng 的描述，请完全忽略（因为 pkgng 已经存在了很长时间）。
