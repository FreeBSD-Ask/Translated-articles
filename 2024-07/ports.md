# FreeBSD ports 开发技术研究

* [ 自由 BSD](https://qiita.com/tags/freebsd)
* [ 创作](https://qiita.com/tags/make)
* [ports](https://qiita.com/tags/ports)

 发布于 2024 年 01 月 07 日

# 开始

当我试图创建ports时，我参考了模板和手册进行工作。在这个过程中，我遇到了一些不符合模式的情况，或者一些特殊行为的情况。我不知道如何跟踪这些情况，如何巧妙地处理它们，这让我感到很困惑。

例如，如果要从ports进行安装（ make install ）的话，通常会按照以下顺序进行。

```
make fetch
make extract
make patch
make build
make install
```

另外，通过将 pre- 和 post- （以及 pre-fetch 和 post-patch 等）作为每个目标的前缀，可以在所需执行的目标前后注入希望执行的操作。此外，通过指定（覆盖） do- 前缀，甚至可以控制 ports 的默认行为。这些顺序和控制如下所示。

```
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

实际上，对于每个 ports ，有各种情况，例如希望禁用该目标的情况（ NO_BUILD / NO_INSTALL / NO_TEST ，每个 make build ， make install ， make test 都被禁用），或者明确希望启用目标的情况（ HAS_CONFIGURE / GNU_CONFIGURE ，均启用 make configure ）。情况千差万别。

实际上，还有更多细节的地方，发生了各种各样的挂钩。要完全列出它们太困难了，因此在调查要点中也要做笔记。

# 目标阶段

 如前所述，众所周知的目标是

* `all`
* `fetch`
* `extract`
* `patch`
* `configure`
* `build`
* install 可能会发生这种情况。对此
* `config`/`showconfig`/`rmconfig`
* `package`/`repackage`
* `test`
* `clean`
* `deinstall`/`reinstall`
* makesum / makepatch 等，将添加一些要了解的目标。

在上述目标中，在ports构建中执行的一系列目标通过 _TARGETS_STAGES 变量进行定义。当然，这是在 bsd.port.mk 变量中定义的变量，不希望被覆盖。通过在适当的ports目录下运行 make -V_TARGETS_STAGES ，可以了解其中包含 SANITY PKG FETCH EXTRACT PATCH CONFIGURE BUILD INSTALL TEST PACKAGE STAGE 的内容。

根据上述变量的内容，在每个阶段都使用 _ステージ名_SEQ 变量详细设置顺序，并使用 _ステージ名_DEP 定义依赖关系。特别是以 _SEQ 结尾的变量被定义为 優先順位:ターゲット ，即使稍后添加也会被控制以顺序执行。

例如，对于 Go 语言应用程序，将执行特定于 Go 的 go mod download 检索。

```
make -V_FETCH_SEQ
make -V_FETCH_REAL_SEQ
```

如果在非常特殊的时机想要插入，请根据这个流程进行必要的准备。

# 使用 make 命令进行调试

当 ports 的情况下，为实现各种简化表示，表面上变得相对简单。 但是，结果是实现变得非常复杂。 对于前面的目标阶段，可能可以大略理解，但要调查实际发生了什么错误是非常困难的。 这就是 make 命令的调试选项（ -dＸ / Ｘ 是另外的功能选项）的出现。

特别是 -dl 选项会显示在 Makefile 中包含 bsd.port.mk 等的命令执行，而它们一直隐藏在 @ 中。

```
make -dl
```

或者也许是 make -de 吧。 这会仅显示执行失败的命令。 如果构建失败，想知道执行了哪些命令导致失败，首先检查 make -de ，然后在 make -dl 中查看整个流程是发生了什么，以便更容易地掌握问题。

再次 make -dx 的情况下，调用的 shell 命令将使用 -x 选项调用，因此在 shell 脚本中执行的内容将显示出来。当然，每次命令调用时... 所以有时候可能无法区分 make -dl 。

# 常见问题及其答案

## Q. 虽然不太整理，但为什么要写这个？

A. Maven 或者 Gradle 有问题。这些家伙像 Go 语言一样频繁预取，就算能容许这一点，但它们没有很好地缓存，所以我在疯狂地调查该怎么办。

## Q. 那问题解决了吗？

 A. 详细内容请查看网页！
