# ccache 在构建 FreeBSD 的 buildworld 时的效率

原文：<https://qiita.com/nanorkyo/items/db1374b5c821dfe5113c>


* [FreeBSD](https://qiita.com/tags/freebsd)
* [ 基准测试](https://qiita.com/tags/%e3%83%99%e3%83%b3%e3%83%81%e3%83%9e%e3%83%bc%e3%82%af)
* [ 定制](https://qiita.com/tags/ccache)

最后更新于 2022-11-16 发布于 2021-12-03

# 开篇

虽然现在有点晚了，但这是关于缓存的话题。作为在构建时加快速度的工具而闻名，但调查后发现，只能找到旧信息。最近的 FreeBSD 中，指定方法已经改变，类似的信息也找不到，所以这次主要介绍这一点，解释其效果。

※传统设置方法已更改，现在可以极其简单地进行控制。

## 验证环境

* FreeBSD 13.0-RELEASE-p5
* FreeBSD 12.2-RELEASE-p11

ccache 的安装过程本次不讨论（没有特殊步骤）。然而，为了调查磁盘使用量，请预先运行以下命令，以防止缓存失效的情况发生。简而言之，（仅构建 FreeBSD 时）使用默认容量（5GB）没有问题。

 缓存容量扩展（5GB→32GB）

```
ccache -M 32G
```

另外，“字节”的 SI 前缀基本上是以 103 为基础的，所以如果想要变为 210，就需要明确指定为 32Gi （即使这样，显示仍然无法更改）。另外，如果仅仅指定数字而不指定单位，将被解释为 G 。

## 设置（/etc/src.conf）

/etc/src.conf

```
WITH_ccache_BUILD= yes
```

只使用 cache buildworld 的话，只需进行上述设置即可。现在不再需要将环境变量 CC 设置为 CC="ccache cc" （这很重要！）。

buildworld 或者 buildkernel 时，需要或者编译安装的组件的打开/关闭情况，请参阅 src.conf(5)，里面列出了一切，但我还没有核对过（太多了…）。

# 进行 make buildworld

从无缓存状态执行 make buildworld 。为了审查缓存命中率等信息，在 buildworld 结束后，获取统计信息（执行 ccache -s 命令）。

## 第一轮执行结果

以下几乎所有文件都已经编译完毕。在这种情况下，由于缓存造成了额外开销，因此编译时间会比平常更长。另外有一些重新编译，但数量非常少，最多不超过 0.2%，可以忽略不计。

第一轮执行结果（从无缓存状态开始构建）

```
cache directory                     /root/.ccache
primary config                      /root/.ccache/ccache.conf
secondary config      (readonly)    /usr/local/etc/ccache.conf
cache hit (direct)                     5
cache hit (preprocessed)              77
cache miss                         46123
cache hit rate                      0.18 %
called for link                      116
ccache internal error                  1
unsupported code directive             3
no input file                          1
cleanups performed                     0
files in cache                    137311
cache size                           2.7 MB
max cache size                      32.0 GB
```

同时，有些结果并不可靠，因为实际的缓存使用量并没有反映在内。这部分内容将在后续提及。

## 第二次执行结果

收集上述数据后，再次进行了 buildworld 。获得了以下结果。正如后面所述，缓存命中率为 100%时的结果为“50.09%”。

第二次执行结果（从缓存状态下的构建）

```
cache directory                     /root/.ccache
primary config                      /root/.ccache/ccache.conf
secondary config      (readonly)    /usr/local/etc/ccache.conf
cache hit (direct)                 45050
cache hit (preprocessed)            1237
cache miss                         46123
cache hit rate                     50.09 %
called for link                      232
ccache internal error                  2
unsupported code directive             6
no input file                          2
cleanups performed                     0
files in cache                    137454
cache size                           2.7 MB
max cache size                      32.0 GB
```

## 清除统计信息的第三轮执行结果

运行 ccache -z ，然后再次执行 buildworld 。得到如下结果。第二轮执行时无法知道的“100% 命中”已被确认。因为在此时没有进行任何编译，所以 buildworld 的执行时间已经缩短。

清除统计信息后的第三轮执行结果（从缓存状态构建）

```
cache directory                     /root/.ccache
primary config                      /root/.ccache/ccache.conf
secondary config      (readonly)    /usr/local/etc/ccache.conf
cache hit (direct)                 45186
cache hit (preprocessed)            1019
cache miss                             0
cache hit rate                    100.00 %
called for link                      116
ccache internal error                  1
unsupported code directive             3
no input file                          1
cleanups performed                     0
files in cache                    137578
cache size                           2.7 MB
max cache size                      32.0 GB
```

## 清理一下试试看

关于缓存大小不反映的问题，执行 ccache -c 可以将其反映到统计数据中。

 清理后的统计信息处于正常化状态

```
cache directory                     /root/.ccache
primary config                      /root/.ccache/ccache.conf
secondary config      (readonly)    /usr/local/etc/ccache.conf
cache hit (direct)                 45186
cache hit (preprocessed)            1019
cache miss                             0
cache hit rate                    100.00 %
called for link                      116
ccache internal error                  1
unsupported code directive             3
no input file                          1
cleanups performed                     0
files in cache                    137578
cache size                           4.5 GB
max cache size                      32.0 GB
```

正如您所看到的，即使是 5GB，也可以在最后一刻完成。

# 基准测试

## buildkernel 实例

我从卡尔先生那里收到了在以下环境中进行 make buildkernel 测量的结果，我将其总结在表格中。

* ＣＰＵ： AMD Ryzen9 3900 （3.10GHz／4.30GHz）
* 内存：32GB
* 存储：WesternDigital SN550（NVMe 连接）・ZFS 操作
* 构建目标：FreeBSD 14-CURRENT

| 构建步骤 | 无缓存时间 | 有缓存时间 |
| ---------- | ------------ | ------------ |
| `make buildkernel`         | 1406 秒    | ８３７秒   |
| `make -j4 buildkernel`         | ３６６秒   | -          |
| `make -j8 buildkernel`         | ２００秒   | -          |
| `make -j12 buildkernel`         | １５５秒   | ８２秒     |
| `make -j16 buildkernel`         | １３６秒   | -          |
| `make -j20 buildkernel`         | １２３秒   | -          |
| `make -j24 buildkernel`         | １１５秒   | ５５秒     |

据说并没有消耗尽所有 CPU 资源。

## buildworld 的示例

* CPU: Intel Pentium N4200（1.10GHz/2.50GHz・Apollo Lake/Goldmont 架构）
* 内存：16GB
* 存储：Transcend MTS400S（SATA 连接）・ZFS 操作
* 构建目标：12.2-RELEASE-p11

| 构建步骤※                 | 构建时间 |
| ---------------------------- | ---------- |
| 无缓存                     | 22317 秒 |
| ccache有り（一周目） | 25603 秒 |
| ccache有り（二周目） | 3841 秒  |
| ccache有り（三周目） | 3854 秒  |

 ※均需指定 make -j5 buildworld

# 概述

* 无论设置 -j 选项的并行度如何，当缓存达到 100% 时，都能显著减少构建时间（仅需数分钟即可完成构建）。
* 在几乎只包含 C 语言的 buildkernel 中，可以获得大约两倍的效果；而包含重型构建（如 LLVM）的 buildworld 中，可以获得约六倍的效果。
* 使用缓存会导致初始开销增加 14％，但作为获取第二次及以后效果的代价而言，这可以忽略不计。
* 在需要稍作修改然后构建整个项目（例如开发等）的情况下，效果非常显著。
* 另外，这次就纯粹重新构建而言，即使是 make clean 中对象已经消失的情况，也表明了不需要 100％重新编译。
* 最近的 FreeBSD 开发环境（LLVM）构建花费的时间变得非常长，但这可以极大地缩短（无论是时间还是内存上）。
* 尽管如此，在第一次编译时仍然需要大量内存，因此在内存较少的环境下构建仍然是一件困难的事情。

## ccache 的典型命令选项

缓存本身很简单，或许正因为如此，不怎么看到有人记载过这一点，因此我简单总结了一下。

| 命令 | 意味                         | 备注                       |
| ------ | ------------------------------ | ---------------------------- |
| `ccache -M ｎ`     | 设置缓存容量（未指定时 Ｇ ） | 指定最大缓存容量※         |
| `ccache -s`     | 显示统计信息                 |                            |
| `ccache -z`     | 统计信息清除                 |                            |
| `ccache -c`     | 缓存目录审查                 | 清理操作（不删除缓存数据） |
| `ccache -C`     | 缓存清除                     |                            |

※ ｎ 可指定的后缀包括 " k "、" M "、" G "、" T "、" Ki "、" Mi "、" Gi "、" Ti "。

# 常见问题及其答案

## 问：/etc/make.conf 里可以设置吗？

答：/etc/make.conf 会被必然地读取，因此实际上相当于在 /etc/src.conf 中设置。然而，如果设置在非 FreeBSD 构建中可能会产生不确定的行为。如果不在意这一点，设置也并无问题。为了与引用相同 make 命令的其他构建进行区分，请设置在 /etc/src.conf 中。
