# portsnap 被淘汰了，本应由 git 代替，但结果我发现自己用的是 got

- 原文：[追放された portsnap、変わりに git が入ってくはずだったのに、気がつけば got がいる](https://qiita.com/nanorkyo/items/f7d2d796b8303eb656f2)
- 作者：重村法克
- 2023-12-02

## 前言

本文介绍了 [Got](https://gameoftrees.org/)，这是源自 OpenBSD 项目的 [Git](https://git-scm.com/) 替代工具。
这是一种所谓的 `git` 命令的替代品，但如果你对 `git` 命令的使用以及其许可证完全没有疑问的话，那就[完全没有必要](https://gameoftrees.org/faq.html#pointless)使用这个工具。由于开发者的立场，这个工具的使用场景也具有独特性。

据称 Got 旨在实现对 Git 的裸数据仓库的兼容[1]。它在针对裸数据仓库的操作方面具有兼容性。相对地，在所谓的工作区（包括索引）层面上则不兼容。在工作流程中需要根据命令加以区分使用。

虽然开篇是以通用视角进行介绍的，但说实话，是因为最近 FreeBSD 中移除了 `portsnap` 命令，所以大家看到了一些提示说“请改用 Git”[2]。本文正是面向那些觉得“为了替代 portsnap 而引入 Git 实在有点……”的人所写的内容。

由于 Got 以实现 OpenBSD 开发工作流为目标进行开发，因此其代码量极少，依赖项也几乎没有。可以毫不夸张地称其为“轻量紧凑”。不过至于运行是否轻快……这一点暂且不予置评。
此外，由于它要求用户接受其独特的使用体验，因此如果你打算“用得很 Git”，那还是不建议使用 Got。

后续的使用案例将以替代 `portsnap` 为目标进行说明。

## 试着安装一下

```sh
$ pkg install got
```

这部分和安装 `git` 没什么不同。由于没有依赖项，因此不会安装其他包。

测试中使用的 Got 版本为 `0.93`。如果版本升级，文中内容可能会不适用。

## 首先试试 clone

`got clone` 和 `git clone` 的区别如下：

* 仅支持克隆裸数据仓库（相当于 `git clone --bare`）
* 无法指定 origin（可能因为裸仓库下 origin 没意义？）
* 可以指定分支，默认好像是单分支模式
* 不支持 shallow clone（浅克隆）
* 有镜像模式（无法向克隆的仓库提交）
* 支持的协议（schema）较少 ※[后述](https://qiita.com/nanorkyo/items/f7d2d796b8303eb656f2#%E5%AF%BE%E5%BF%9C%E3%81%97%E3%81%A6%E3%81%84%E3%82%8B%E3%82%B9%E3%82%AD%E3%83%BC%E3%83%9E)
* 总之就是很慢（不会多线程进行 Resolving deltas） [后述](https://qiita.com/nanorkyo/items/f7d2d796b8303eb656f2#gitclone%E3%81%A8gotclone%E3%81%AE%E5%AE%9F%E8%A1%8C%E6%99%82%E9%96%93)

## 克隆示例

```sh
$ git clone https://git.freebsd.org/ports.git /usr/ports
```

对应的 `got` 命令如下。首先需要一个目录用于放置裸仓库。
另外这个操作会非常慢（[后述](https://qiita.com/nanorkyo/items/f7d2d796b8303eb656f2#%E3%82%AF%E3%83%AD%E3%83%BC%E3%83%B3%E3%81%AB%E3%81%8B%E3%81%8B%E3%82%8B%E6%99%82%E9%96%93%E3%81%AE%E8%A8%88%E6%B8%AC)），如果条件允许，还是建议使用 `git` 命令来完成。

```sh
$ got clone -am ssh://anongit@git.freebsd.org/ports.git /home/ports.git
$ got checkout /home/ports.git /usr/ports
```
## 支持的协议（スキーマ）

* `git://`
* `git+ssh://`（或者 `ssh://`）

## 不支持的协议（スキーマ）

* `git+http://`（或者 `http://`）※ 错误（因为政策原因，未来不打算支持）
* `https://` ※ 尚未实现的错误（TODO）
* `ftp://` ※ 什么？（不确定是否被识别）
* `ftps://` ※ 什么？（不确定是否被识别）

## 克隆时间的测量

### 使用 `git clone` 时的输出信息

```sh
$ git clone --bare --single-branch ssh://anongit@git.freebsd.org/ports.git
Cloning into bare repository 'ports.git'...
remote: Enumerating objects: 5942473, done.
remote: Counting objects: 100% (171767/171767), done.
remote: Compressing objects: 100% (16358/16358), done.
remote: Total 5942473 (delta 167421), reused 155423 (delta 155409), pack-reused 5770706
Receiving objects: 100% (5942473/5942473), 1.12 GiB | 11.01 MiB/s, done.
Resolving deltas: 100% (3577978/3577978), done.
```

### 使用 `got clone` 时的输出信息

```sh
$ got clone ssh://anongit@git.freebsd.org/ports.git
Connecting to ssh://anongit@git.freebsd.org/ports.git
server: Enumerating objects: 5963414, done.
server: Counting objects: 100% (133228/133228), done.
server: Compressing objects: 100% (13237/13237), done.
server: Total 5963414 (delta 129757), reused 119991 (delta 119991), pack-reused 5830186
1157M fetched; indexing 100%; resolving deltas 100%
Fetched 0d39a9d41ecbf5cd111bcc9ae9f2cfcf7e30a616.pack
Created cloned repository 'ports.git'
```

### `git clone` 和 `got clone` 的执行时间对比

```sh
git clone --bare --single-branch ssh://anongit@git.freebsd.org/ports.git
511.72s user 40.21s system 163% cpu 5:36.76 total

got clone ssh://anongit@git.freebsd.org/ports.git
660.88s user 75.82s system 87% cpu 14:00.91 total
```

* 执行环境约为 5 分 37 秒 与 14 分 1 秒之间的差异，差距为 2.5 倍。
* 另外，Git 的 CPU 使用率较高，处理效率偏向 CPU 绑定（通过多线程处理实现）。
* 相反，Got 的系统使用率较高，瓶颈主要集中在 I/O 绑定上。

## 克隆时的选项指定对容量的影响

| 命令    | 单分支模式\*     | 多分支模式\*     |
| ----- | ----------- | ----------- |
| `got` | 1,306,944KB | 1,326,872KB |
| `git` | 1,322,480KB | 1,350,344KB |

* 单分支：`git clone --single-branch` 或 `got clone`
* 多分支：`git clone` 或 `got clone -a`

这里确认了不同选项指定时的容量差异。`git` 和 `got` 之间的差异较大，但为何会产生这种差异尚不明确，可能是某种开销造成的。

## 工作树更新

```sh
$ cd /usr/ports && git pull
```

上面执行的命令，相当于 `got` 命令如下：

```sh
$ cd /usr/ports && got fetch && got update
```

这个是典型的使用方式。关于开发流程中需要的使用方法，请参考[命令对照表](https://gameoftrees.org/comparison.html)等参考资料。

## 附加的 `tog` 命令

`tog` 是一款基于 ncurses 的日志查看器。它显示一行日志，当按下回车键时，会显示详细日志和修正内容，这是个非常方便的工具。它相当于 Git 的第三方工具 `tig`。

## 信息展示

`Got` 中用于管理仓库和工作树的命令不同。具体来说，分别是 `gotadmin` 和 `got`。

不过，安全使用的子命令大概只有 `info`。根据仓库或工作树的状态，显示不同的信息。

```sh
$ gotadmin info -r /home/ports.git
repository: /home/ports.git
remote "origin": ssh://anongit@git.freebsd.org/ports.git
pack files: 4
packed objects: 5963321
packed total size: 1318M
loose objects: 0
```

如果在通过 `git clone --bare` 克隆的目录内运行 `gotadmin info`，会看到 `remote` 行显示了更多的信息。

查看仓库下的文件时，发现 `config` 文件中的 `remote "origin"` 配置被 `got.conf` 中的 `remote "origin"` 配置所替代。如果创建了 `got.conf` 文件，`remote` 行便会显示出来。

另外，在工作树上执行 `gotadmin info` 时，会显示仓库的状态。而执行 `got info` 则会显示工作树相关的信息。

```sh
$ cd /usr/ports
$ gotadmin info
repository: /home/ports.git
remote "origin": ssh://anongit@git.freebsd.org/ports.git
pack files: 4
packed objects: 5963321
packed total size: 1318M
loose objects: 0
$ got info
work tree: /usr/ports
work tree base commit: 388fa384c1dab4774d4db755ec1089b57e6f9a97
work tree path prefix: /
work tree branch reference: refs/heads/main
work tree UUID: 0c2bbcf5-8a1d-11ee-8d56-9ca3ba01eed8
repository: /home/ports.git
```

## 常见问题及其解答

### 问：`https` 协议不能使用吗？

答：可能是因为没有依赖 `curl`。如果实现普通的 HTTP 通信，可能会遇到无法并行处理的问题……这是我个人的推测。而使用 `ssh://` 则更加稳妥，因为很多系统中已经安装了 `ssh` 命令【需要引用来源】。

### 问：`got update`、`got merge`、`got rebase`、`got integrate` 有什么区别？

答：尚未研究。估计与 Git 中的用法相同。

### 问：`got clone ... /usr/ports/.git`，不是就能创建与 `git clone` 完全相同的情况吗？

答：经过验证，`got clone` 可以成功执行，但 `got checkout` 会失败。

```sh
$ got clone -m ssh://anongit@git.freebsd.org/ports.git /usr/ports/.git
Connecting to ssh://anongit@git.freebsd.org/ports.git
   :
Created mirrored repository '/usr/ports/.git'
$ got checkout /usr/ports/.git /usr/ports/
got: work tree and repository paths may not overlap: /usr/ports/.git: bad path
```

### 问：`/usr/ports/.git` 是怎么回事？

答：实际上这是一个裸仓库的目录！在 `git clone` 后，运行 `gotadmin info`，结果非常令人吃惊……

### 问：运行 `git status` 后看到惊人的结果！

答：关于 `.gitignore` 文件的处理，~~有一个 bug~~ 这是设计上的特性。本来 `.gitignore` 文件中指定的目录和文件是不应显示的。根据略微的研究，它并不是完全没有处理 `.gitignore` 文件。所以这是~~一个 bug~~ 设计上的特性。

### 问：有没有别名功能？

答：没有。这个功能确实很需要，真希望有人能实现它。

### 问：既然能这么做，为什么不把它加入到基本系统中？

答：在彻底使用并报告其效果之前，不能肯定它的质量是否足够达到基本系统的要求。至少在我使用的范围内，它似乎还没有达到那个质量标准。

## 参考文献

* [git](https://git-scm.com/)
* [got](https://gameoftrees.org/)
* [got、cvs、svn、git との比較](https://gameoftrees.org/comparison.html)
* [よくある質問とその答え](https://gameoftrees.org/faq.html)
* [FOSDEM 2023 での発表資料](https://www.openbsd.org/papers/fosdem2023-gotd.pdf)
* [EuroBSDcon 2019 での発表資料](https://www.openbsd.org/papers/eurobsdcon2019-gameoftrees.pdf)
* [FreeBSDの入手方法（Gitの利用）](https://docs.freebsd.org/ja/books/handbook/mirrors/#git)

---

1. [Game of Trees Goals](https://gameoftrees.org/goals.html)より
2. [Ports Collection のインストール](https://docs.freebsd.org/ja/books/handbook/ports/#ports-using-installation-methods) 
