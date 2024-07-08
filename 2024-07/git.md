# portsnap 被淘汰了，本应由 git 代替，但结果我发现自己用的是 got

<https://qiita.com/nanorkyo/items/f7d2d796b8303eb656f2>

* [Git](https://qiita.com/tags/git)
* [FreeBSD](https://qiita.com/tags/freebsd)
* [ 开源操作系统](https://qiita.com/tags/openbsd)
* [ 获得](https://qiita.com/tags/got)
* [portsnap](https://qiita.com/tags/portsnap)

最后更新于 2023-12-02 发布于 2023-12-02

# 开始

介绍了一个名为 Ｇｏｔ 的由ＯｐｅｎＢＳＤ项目衍生出来的 Ｇｉｔ 替代品。虽然它是一个 git 命令的替代品，但对于那些对 git 命令的使用没有疑问，并且对其许可证没有疑问的人来说，这是一个完全不必要的工具。由于开发者是开发者，因此其用途非常独特。

据说它的目标是支持 Ｇｉｔ 的裸仓库兼容^ 1^。对于裸仓库的操作是兼容的。但是在所谓的工作目录（包括索引）级别上不兼容。因此，需要根据工作流程切换命令。

既然开始写一个泛论的介绍，最近的 FreeBSD 中删除了 portsnap 命令，看到使用 Git 的建议^ 2^，对于那些希望使用 portsnap 而不是 git 的人来说，可能有点不情愿……

由于 Got 专注于实现 OpenBSD 开发的工作流程，因此其代码量非常少，几乎没有依赖。它被宣称为轻量级、紧凑且不会造成头痛的工具。然而，关于其轻便性能的评价，我就不发表评论了。另外，由于要求独特的使用体验，我认为在需要严格使用 Git 的情况下最好不要使用它。

以下用例将描述目标作为 portsnap 的替代。

# 尝试安装

```
$ pkg install got
```

这周围也没有变化。由于没有依赖项，因此不会安装其他软件包。

通过 GOT 的版本验证为 0.93 。由于版本升级，此处记录的内容可能不再适用。

# 首先尝试克隆

got clone 和 git clone 之间的区别如下。

* 仅克隆裸库（相当于 git clone --bare ）
* 无法指定来源（裸仓库指定来源没有意义？）
* 可以指定分支，默认好像是单分支
* 但是好像不能进行浅克隆
* 存在镜像模式（无法向克隆的存储库提交更改的模式）
* 支持的模式较少 ※ 详见下文
* 非常慢（在多线程下不执行解析差异） 详见下文

## 克隆案例

```
$ git clone https://git.freebsd.org/ports.git /usr/ports
```

与上述命令相对应的是 got 命令，具体如下。首先需要展开裸存储库的区域。由于这个过程非常耗时（后文将介绍），如果有余地的话，可能最好直接使用 git 命令。

```
$ got clone -am ssh://anongit@git.freebsd.org/ports.git /home/ports.git
$ got checkout /home/ports.git /usr/ports
```

## 支持的模式

* `git://`
* git+ssh:// （或 ssh:// ）

## 不支持的模式

* git+http:// （或 http:// ）※错误（策略上不打算支持）
* https:// ※未实装错误（待办事项）
* ftp:// ※那是什么？（确认存在与否）
* ftps:// ※那是什么？（确认存在与否）

## 克隆所需的时间测量

### git clone 的消息

```
$ git clone --bare --single-branch ssh://anongit@git.freebsd.org/ports.git
Cloning into bare repository 'ports.git'...
remote: Enumerating objects: 5942473, done.
remote: Counting objects: 100% (171767/171767), done.
remote: Compressing objects: 100% (16358/16358), done.
remote: Total 5942473 (delta 167421), reused 155423 (delta 155409), pack-reused 5770706
Receiving objects: 100% (5942473/5942473), 1.12 GiB | 11.01 MiB/s, done.
Resolving deltas: 100% (3577978/3577978), done.
```

### got clone 的消息

```
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

### git clone 和 got clone 的执行时间

```
git clone --bare --single-branch ssh://anongit@git.freebsd.org/ports.git
511.72s user 40.21s system 163% cpu 5:36.76 total

got clone ssh://anongit@git.freebsd.org/ports.git
660.88s user 75.82s system 87% cpu 14:00.91 total
```

* 执行环境大约 5 分 37 秒与 14 分 1 秒的差异，有 2.5 倍的差距。
* 另外，Git 的 CPU 使用率较高，表明处理效率偏向于 CPU 带宽（由多线程处理产生的效果）。
* 相反地，GOT 具有較高的系統使用率，可以看出瓶頸往往偏向 I/O 帶寬。

## 關於克隆時的選項指定造成的容量差異

| 命令 | 单分支※    | 多分支※    |
| ------ | ------------- | ------------- |
| `got`     | 1,306,944KB | 1,326,872KB |
| `git`     | 1,322,480KB | 1,350,344KB |

※单分支： git clone --single-branch 或 got clone ※多分支： git clone 或 got clone -a

一応通过选项指定确认了容量的变化。此外， git 和 got 之间的差异很大，尽管为什么会出现这种差异尚不明确。可能是某种开销导致的……。

# 更新工作树

```
$ cd /usr/ports && git pull
```

对上述命令的执行相当于下面的 got 命令。

```
$ cd /usr/ports && got fetch && got update
```

本案例中的大致用法如下。有关在开发流程中可能需要的用法，请参阅命令对应表以及其他参考文献。

# 关于 tog 命令的额外内容

这是一个基于 ncurses 的日志查看器。它可以显示一行日志，同时在选择该日志（按下回车键）时，会显示详细日志和修改内容，非常实用。据说类似于 Git 的第三方工具中的 tig 命令。

# 信息

在 gotadmin 和 got 命令应用于存储库和工作树时有所不同。具体而言，安全使用的子命令大约是 info 。根据存储库或工作树的状态，显示内容会有所变化。

```
$ gotadmin info -r /home/ports.git
repository: /home/ports.git
remote "origin": ssh://anongit@git.freebsd.org/ports.git
pack files: 4
packed objects: 5963321
packed total size: 1318M
loose objects: 0
```

在 git clone --bare 目录中运行 gotadmin info 并比较，显示增加了 remote 行。查看存储库下的文件时，发现不是 remote "origin" 文件中的 config ，而是 got.conf 中的 remote "origin" 设置。实际上，创建了 got.conf 文件后，显示了 remote 行。

另外，如果在工作树上运行 gotadmin info ，将显示存储库的状态。此外， got info 的执行结果将提供有关工作树的信息。

```
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

# 常见问题及其答案

## 问： https 模式不可用吗？

Ａ． 可能是因为它不依赖于 curl 。在普通的 HTTP 通信实现中，可能会遇到无法进行并行处理的问题……这只是我的猜测。如果使用 ssh:// ，那里到处都安装了 ssh 命令，感觉很安心【需要出典】。

## Ｑ． got update ／ got merge ／ got rebase ／ got integrate 有什么区别？

 Ａ． 未调查。可能情况与 Git 类似。

## Ｑ． 如果 got clone ... /usr/ports/.git ，就可以创建与 git clone 完全相同的情况了！

Ａ． 我已验证。 got clone 可成功，但 got checkout 将失败。

```
$ got clone -m ssh://anongit@git.freebsd.org/ports.git /usr/ports/.git
Connecting to ssh://anongit@git.freebsd.org/ports.git
   :
Created mirrored repository '/usr/ports/.git'
$ got checkout /usr/ports/.git /usr/ports/
got: work tree and repository paths may not overlap: /usr/ports/.git: bad path
```

## Ｑ． 嗯？ /usr/ports/.git 是什么意思？

即是说实际上是个裸仓库的目录！ 做完 git clone 后接着做 gotadmin info 就会得到惊人的结果。

## 问：做完 git status 就会得到惊人的结果！

答：处理 .gitignore 文件时出现了一个错误，这是个 bug。按照正常情况 .gitignore 文件指定的目录或文件等不会显示。轻轻地调查了一下，应该并不是没有处理 .gitignore 文件。所以这就是 bug 所在。

## Ｑ．别名功能呢？

Ａ．没有。希望有这个功能。请有心人实现。

## Ｑ．这么好用了，加到基础系统里也可以吧？

Ａ．好好使用并报告。在当前时点可能无法评价基础系统的质量是否足够。至少在我所接触的范围内，似乎还未达到那种质量水平。

# 参考文献

* [git](https://git-scm.com/)
* [ 得到](https://gameoftrees.org/)
* [ 与 got、cvs、svn、git 的比较](https://gameoftrees.org/comparison.html)
* [ 常见问题及其答案](https://gameoftrees.org/faq.html)
* [ FOSDEM 2023 的演讲资料](https://www.openbsd.org/papers/fosdem2023-gotd.pdf)
* [ 在 EuroBSDcon 2019 年的演讲材料](https://www.openbsd.org/papers/eurobsdcon2019-gameoftrees.pdf)
* [ FreeBSD 的获取方式（使用 Git）](https://docs.freebsd.org/ja/books/handbook/mirrors/#git)

1. 从 Game of Trees Goals ↩
2. Ports Collection 的安装 ↩
