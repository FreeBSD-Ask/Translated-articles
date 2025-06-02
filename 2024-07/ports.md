# FreeBSD ports 开发技术研究

- 原文：[FreeBSD ports を 作 る 際 の 調査 ノウハウ](https://qiita.com/nanorkyo/items/bc1860f624823501e197)
- 作者：重村法克
- 2024-01-07

## 介绍

在创建 Ports 时，参考模板和手册进行操作时，可能会遇到一些无法按照常规模式处理的情况，或者需要处理一些特殊行为的情况。这时，我们可能会对如何追踪这些情况、如何正确插入操作产生困惑。

例如，当从 Ports 安装时（`make install`），通常会按以下顺序执行：

```sh
make fetch
make extract
make patch
make build
make install
```

此外，通过在各个目标前加上 `pre-` 或 `post-` 前缀（例如 `pre-fetch` 或 `post-patch`），可以在执行某个目标之前或之后插入自定义操作。更进一步，通过指定 `do-` 前缀（覆盖默认行为），也可以控制 Ports 的默认行为。关于这些顺序和控制，详细内容如下：

```sh
make fetch
 +- make pre-fetch
 +- make do-fetch
 +- make post-fetch
make extract
 +- make pre-extract
 +- make do-extract
 +- make post-extract
make patch
 +- make pre-patch
 +- make do-patch
 +- make post-patch
make configure
 +- make pre-configure
 +- make do-configure
 +- make post-configure
make build
 +- make pre-build
 +- make do-build
 +- make post-fetch
make install
 +- make pre-install
 +- make do-install
 +- make post-install
```

在某些情况下，可能需要禁用某个目标（例如 `NO_BUILD`、`NO_INSTALL`、`NO_TEST`，分别使 `make build`、`make install`、`make test` 失效），或显式启用某个目标（例如 `HAS_CONFIGURE`、`GNU_CONFIGURE`，使 `make configure` 生效）。这些设置在不同的 Ports 中有不同的处理方式。

实际上，在更多的细节处，还会有各种各样的钩子。要完全列出这些钩子非常困难，因此可以记录一些调试和调查的关键点。

## 目标阶段

如前所述，常见的目标包括：

- `all`
- `fetch`
- `extract`
- `patch`
- `configure`
- `build`
- `install`

除此之外，还应了解一些其他目标：

- `config` / `showconfig` / `rmconfig`
- `package` / `repackage`
- `test`
- `clean`
- `deinstall` / `reinstall`
- `makesum` / `makepatch`

在 Ports 构建过程中，执行的一系列目标由 `_TARGETS_STAGES` 变量定义。这个变量在 `bsd.port.mk` 中定义，并不打算被覆盖。在适当的 Ports 目录中运行 `make -V_TARGETS_STAGES`，可以看到类似 `SANITY PKG FETCH EXTRACT PATCH CONFIGURE BUILD INSTALL TEST PACKAGE STAGE` 的内容。

基于上述变量的内容，每个阶段都会通过 `_阶段名_SEQ` 变量详细设置顺序，依赖关系则通过 `_阶段名_DEP` 进行定义。特别是以 `_SEQ` 结尾的变量，定义了“优先级：目标”，即使后来添加的目标也会按照顺序执行。

例如，在 Go 语言应用程序中，特有的 Go 获取操作 `go mod download` 会在以下阶段执行：

```sh
make -V_FETCH_SEQ
make -V_FETCH_REAL_SEQ
```

如果需要在极其特殊的时机插入操作，则需要按照这个流程来进行插入。

## 使用 make 命令进行调试

在 Ports 中，为了实现各种简化的表达，表面上看起来可能更加简单。然而，结果是实现变得极为复杂。尽管你可能大致理解了先前提到的目标阶段，但实际发生了什么，为什么会失败，进行调查是非常困难的。这时，`make` 命令的调试选项（`-dX`，其中 `X` 是针对特定功能的选项）就派上了用场。

特别是 `-dl` 选项，它会显示包括 `bsd.port.mk` 等文件中的 `@` 隐藏的命令执行过程。

```sh
make -dl
```

或者使用 `make -de`。这个选项仅显示执行失败的命令。如果在构建过程中出现失败，并且你想知道具体是哪条命令执行失败，可以先使用 `make -de` 查看失败的命令，再通过 `make -dl` 查看整体流程，这样有助于更容易地掌握问题。

此外，`make -dx` 选项会使得调用的 shell 命令带上 `-x` 选项，从而使得在 shell 脚本中执行的内容被显示出来。由于是每次调用时显示，因此与 `make -dl` 的区别可能不太明显。

## 常见问题与解答

### Q. 为什么这篇文章写得有点杂乱？

A. 这都是 Maven 或 Gradle 的错！它们像 Go 语言一样频繁地进行预获取（prefetch），这点我可以勉强接受。但问题是，它们没有很好地进行缓存，所以我查了很久，最后不得不想办法解决这个问题。

### Q. 那么你最终解决了吗？

A. 继续关注网站上的更新吧！

## 参考文献

- [Mk/bsd.port.mk](https://cgit.freebsd.org/ports/tree/Mk/bsd.port.mk)
- [make(1)](https://man.freebsd.org/cgi/man.cgi?make%281%29)
- [FreeBSD port 日语开发者手册](https://docs.freebsd.org/ja/books/porters-handbook/)
- [FreeBSD Port 开发者手册](https://docs.freebsd.org/en/books/porters-handbook/)
- [Ports 的 Makefile 模板](https://cgit.freebsd.org/ports/tree/Templates/Makefile)
