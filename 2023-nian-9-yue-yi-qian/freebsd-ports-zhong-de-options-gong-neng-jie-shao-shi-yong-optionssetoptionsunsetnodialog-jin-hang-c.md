# FreeBSD Ports 中的 OPTIONS 功能介绍——使用 OPTIONS_SET/OPTIONS_UNSET/NO_DIALOG 进行操作和实践

- 原文：[FreeBSD Ports Collection における OPTIONS 機能 の 紹介 - OPTIONS_SET/OPTIONS_UNSET/NO_DIALOG の 活用 と 実践 -](https://qiita.com/nanorkyo/items/a0068cafcf9112ebbb7b)
- 作者：重村法克
- 2021-04-04

## 开始

即使到了 2020 年，依然有“还在编译吗？”的疑问，在如今的 FreeBSD 中，除了基础系统之外，第三方应用的二进制包的使用（安装）已经变得非常普及。事实上，轻松设置并使用的话，二进制包已经足够使用。然而，对于我个人来说，由于需要高度自定义使用，所以无法完全满足于二进制包的需求。因此，整理了一下 FreeBSD Ports 的自定义方法。

关于什么是 Ports（FreeBSD Ports），请参见参考文献。抱歉。

### 适用环境

截至 2020 年，由于该方案已经出现了五年多，因此适用于当前所有支持的 FreeBSD 环境。大体上，版本 10 及以上都可以使用。

## OPTIONS 机制

通过引入 OPTIONS（`/usr/ports/Mk/bsd.options.mk`）机制 [1]，统一了使用 Ports 编译第三方应用时的自定义方法。

引入 OPTIONS 前后的 Makefile 编写方式发生了变化，OPTIONS 引入之前的机制（几乎）已经不再使用 [2]。

严格来说，它仍然存在并且在一些情况下被使用，但多用于极其特殊的、黑客式的自定义用途，因此本次讨论将跳过这部分内容。

### OPTIONS 机制是什么

OPTIONS 是一种控制机制，用于控制每个 Port 中可以自定义的选项的启用与禁用。它提供了以下功能：

- 定义可自定义的选项（选项名称）
- 解释这些选项的功能（说明）
- 默认启用或禁用的值
- 提供有关行为（如依赖关系、安装方法等）的帮助

Port 维护者通过将这些内容记录在 Makefile 中，允许用户仅通过选择开启或关闭选项来进行自定义。

### 通过对话框进行选择

在编译时会显示一个对话框，允许用户交互式地选择这些自定义项。可以通过以下命令来配置这些选项 [3][4]：

**命令执行示例**

```sh
make config
make config-recursive
make showconfig
make showconfig-recursive
make rmconfig
make rmconfig-recursive
```

尤其需要注意的是，一旦通过对话框确认了设置后，除非显式地执行 `make config`，或者在有新选项添加时，系统不会再次显示对话框。如果想恢复到默认设置，可以使用 `make rmconfig` 来删除通过对话框设置的内容。

### 全局设置

通过在 `/etc/make.conf` 中进行配置，可以全局定义自定义内容（启用或禁用）。例如，如果不需要 X11，或想要使用 LZ4 作为压缩算法，可以在所有 Ports 中统一设置。

**/etc/make.conf 配置示例**

```ini
OPTIONS_SET+=   LZ4 LZO LZO2 LZMA ZLIB ZSTD BROTLI
OPTIONS_UNSET+= X11 CUPS GDBM TEST DEBUG TESTS
```

### 全局的 Port 单独设置

在 `/etc/make.conf` 中，还可以根据 Port 进行个性化设置。例如，虽然通常希望安装文档，但为了避免安装 TeX 等相关组件，可能希望放弃安装文档。在这种情况下，可以通过 `${OPTIONS_NAME}_SET` 或 `${OPTIONS_NAME}_UNSET` 来定义设置。

**/etc/make.conf 配置示例**

```ini
emulators_open-vm-tools_UNSET= DOCS FUSE LIBNOTIFY
```

`${OPTIONS_NAME}` 是通过执行 `make -VOPTIONS_NAME` 获取的每个 Port 的字符串。上述示例中，某些变量名可能包含 `-` 或 `+`，但在 Makefile 中不会造成问题。

需要注意的是，OPTIONS 引入时使用的是 `UNIQUENAME`，但由于与其他机制冲突，后来改为使用 `OPTIONS_NAME`。

### 在 bsd.options.mk 中的默认设置

在 `bsd.options.mk` 中，以下设置无论 Port 的默认配置如何，都会被启用：

- `DOCS`
- `NLS`
- `EXAMPLES`
- `IPV6`

即使在 Port 的默认设置中没有启用 `DOCS`，但如果对话框选项中包含 `DOCS`，则实际会启用该选项。这就是为什么即使 Port 默认没有启用 `DOCS`，它仍然被启用的原因。

## OPTIONS 的优先级

1. `/etc/make.conf` （`${OPTIONS_NAME}_SET_FORCE` ／ `${OPTIONS_NAME}_UNSET_FORCE`）
2. `/etc/make.conf` （`OPTIONS_SET_FORCE` ／ `OPTIONS_UNSET_FORCE`）
3. `make config`
4. `/etc/make.conf` （`${OPTIONS_NAME}_SET` ／ `${OPTIONS_NAME}_UNSET`）
5. `/etc/make.conf` （`OPTIONS_SET` ／ `OPTIONS_UNSET`）
6. `make WITH="选项1 选项2..." WITHOUT="选项1 选项2..."` 指定的情况^[5](https://qiita.com/nanorkyo/items/a0068cafcf9112ebbb7b#fn-%E5%82%99%E8%80%83%EF%BC%95)^
7. `bsd.options.mk`
8. `Makefile` ／ `Makefile.local`

需要注意的是，`_SET` 在处理后，才会处理 `_UNSET`，因此优先级是 `_UNSET` > `_SET`，即 `_UNSET` 的效果更强。

## 不需要自定义

也就是说，包管理系统默认是没有进行自定义的，应该是没有显示对话框的情况下直接处理的！[6]

实际上确实存在这样的选项。只需在 `/etc/make.conf` 中添加以下内容，就可以在没有任何对话框显示的情况下（即没有自定义）开始编译所有的 Ports：

**/etc/make.conf 配置示例**

```ini
NO_DIALOG= yes
```

实际上，这只是跳过了对话框的显示，之前执行过的 `make config` 的结果，以及前面提到的 `OPTIONS_SET`/`OPTIONS_UNSET` 等设置仍然会生效。因此，这样做并不意味着完全默认，但可以提供与二进制包兼容的操作方式。

## ports-mgmt/portconf

[portconf](https://www.freshports.org/ports-mgmt/portconf/) 是一个工具，允许在 `/usr/local/etc/ports.conf` 配置文件中记录各个 Port 的自定义设置。

它在安装时会将钩子添加到 `/etc/make.conf`，并通过 `/usr/local/etc/ports.conf` → `/usr/local/libexec/portconf` → `/etc/make.conf` 的方式协同工作，使得好像从一开始就在配置文件中写入了这些设置。因此，不应删除 `/etc/make.conf` 中的钩子。

它的工作方式是将配置文件中的 Port 目录名与当前目录进行比较（通过通配符匹配），如果匹配成功，则会将相应的内容纳入。

例如，在下面的例子中，当在 `/usr/ports/databases/sqlite3` 目录下执行 `/usr/local/libexec/portconf` 时，你会看到 `|NO_DIALOG=yes` 被显示出来。

**/usr/local/etc/ports.conf 配置示例**

```ini
databases/sqlite3: NO_DIALOG=yes
lang/perl5.*:      NO_DIALOG=yes
```

在此示例中，像 SQLite3 这样的配置可以非常细致地进行自定义，但有时可能会引入会导致丧失二进制兼容性的自定义（此时它会默认关闭）。因此，某些 Port 最好避免过度自定义。

### 与 OPTIONS 的区别

在 OPTIONS 的情况下，像 `${OPTIONS_NAME}_SET` 和 `${OPTIONS_NAME}_UNSET` 这样的选项可以明确地为每个 Port（`${OPTIONS_NAME}`）指定 `OPTIONS_SET` 和 `OPTIONS_UNSET`。但是，像 `NO_DIALOG` 这样的全局设置不能为每个 Port 单独设置。这时，就可以使用 `portconf`。

当然，也可以不使用 `${OPTIONS_NAME}_SET` 或 `${OPTIONS_NAME}_UNSET`，而是使用 `OPTIONS_SET` 或 `OPTIONS_UNSET` 来在 `ports.conf` 中配置，但在没有安装 `portconf` 的环境中构建时，这可能会有较大影响。因此，在影响较小的地方使用 `portconf` 会是更高效的做法。

理解了上述内容之后，你可以通过执行 `pkg info -D portconf` 来查看配置内容并进行编辑。

## 常见问题及解答

### Q. 从 Ports 编译的程序和二进制包可以共用吗？

当所有 Port 都设置为 `NO_DIALOG` 且没有自定义时是可以的。如果你想在包的更新速度较慢时通过 Ports 编译来替代，请确保不要进行自定义。

### Q. 自定义的 Port 和二进制包可以共用吗？

基本上不行。虽然在有限的情况下可能可以通过一些实验来实现，但这需要不断的测试和调整，且未来的兼容性不能保证。自定义后，某些功能是否启用或者禁用，可能导致某些功能缺失或者不可用，这种问题通常是在运行时才会发现（没有二进制兼容性）。这种变化的影响有时比版本升级的影响更大。

这也许有点警告的意味，但对于中间件（尤其是库）类型的 Port 来说，如果“没有必要”进行自定义，对于应用程序类型的 Port 来说，如果“已经存在”，是没有问题的。

### Q. 启用与禁用，`SET` 与 `UNSET` 有什么区别？

A. 没问题的。你的理解是正确的。虽然这应该解释清楚，但我累了，得休息一下。

# 参考文献

- [[FreeBSD-Ports-Announce] Configuring options in make.conf](https://lists.freebsd.org/pipermail/freebsd-ports-announce/2013-June/000062.html)
- アプリケーションのインストール - packages と ports [【日本語版】](https://www.freebsd.org/doc/ja_JP.eucJP/books/handbook/ports.html) [【英語版】](https://www.freebsd.org/doc/handbook/ports.html)^[7](https://qiita.com/nanorkyo/items/a0068cafcf9112ebbb7b#fn-%E5%82%99%E8%80%83%EF%BC%97)^
- Ports Collection の 利用 [【日本語版】](https://www.freebsd.org/doc/ja_JP.eucJP/books/handbook/ports-using.html) [【英語版】](https://www.freebsd.org/doc/handbook/ports-using.html)^[7](https://qiita.com/nanorkyo/items/a0068cafcf9112ebbb7b#fn-%E5%82%99%E8%80%83%EF%BC%97)^
- [bsd.port.mk](https://github.com/freebsd/freebsd-ports/blob/master/Mk/bsd.port.mk)
- [bsd.port.options.mk](https://github.com/freebsd/freebsd-ports/blob/master/Mk/bsd.port.options.mk)
- [portconf](https://www.freshports.org/ports-mgmt/portconf/)

---

1. **2012 年 05 月以降**：当然，新的系统引入时，并不是所有的 Ports 都立刻支持，但已经过去 8 年多了。

2. **`WITHOUT_X11` 等的定制设置不再使用**：这些主要的定制选项已经不再使用（严格来说，这是一种错误的用法）。

3. **`config`，`showconfig`，`rmconfig` 的功能**：`config` 用来设置，`showconfig` 用来查看当前设置，`rmconfig` 用来删除设置。

4. **`-recursive` 选项**：这个选项会递归地处理所有依赖关系。

5. **`WITH` 和 `WITHOUT` 选项的作用**：通过 `WITH` 指定的选项会被 `SET`，而通过 `WITHOUT` 指定的选项会被 `UNSET`。

6. **关于包管理系统的使用和定制**：经过简单调查后，发现没有明确的包管理系统运作信息。基于对使用工具和定制化的预测，个人推测这可能不需要太多定制。

7. **关于日文版和英文版的差异**：由于日文版的记录较为过时，请参考英文版，尤其是 `pkgng` 相关的部分已经完全过时。


