# ccache 在构建 FreeBSD 的 buildworld 时的效率

- 原文：[ccacheでFreeBSDをbuildworldした 場合 の 効率 について](https://qiita.com/nanorkyo/items/db1374b5c821dfe5113c)
- 作者：重村法克
- 2022-11-16

## 开始

虽然现在谈论 `ccache` 可能有些晚，但它作为加速构建过程的工具非常有名。通过调查，我们发现现有的很多信息已经过时。最近 FreeBSD 中的配置方法已经有所变化，但并没有找到相关的更新信息。因此，本篇将重点介绍这一变化，并分析其效果。

※与以前的设置方法相比，现在的控制变得非常简便。

### 测试环境

- FreeBSD 13.0-RELEASE-p5
- FreeBSD 12.2-RELEASE-p11

这次不讨论 `ccache` 的安装过程（没有特别的步骤）。为了进行磁盘使用量调查，我们提前执行了以下命令，确保不会出现缓存无法生效的情况。结论是，默认的 5GB 使用量在 FreeBSD 的构建过程中是完全没有问题的。

**扩展缓存容量（5GB → 32GB）**

```sh
ccache -M 32G
```

值得注意的是，“字节”的 SI 前缀基本上是以 $10^3$ 为单位的。如果想使用 $2^{10}$ 作为单位，需要明确使用 `103103` 作为基本单位。如果要使用 $2^{10}$ 的方式表示容量，应该使用 `32Gi`（尽管显示仍不可更改）。另外，如果只是输入数字而不指定单位，默认会被解释为 `G`。

### 配置（/etc/src.conf）

**/etc/src.conf**

```ini
WITH_CCACHE_BUILD= yes
```

如果只是使用 `ccache` 来进行 `buildworld` 构建，那么只需要设置上面的配置即可。无需再手动设置环境变量 `CC="ccache cc"`（这一点非常重要！）。

对于 `buildworld` 或 `buildkernel` 构建中需要开启或关闭的组件，可以参考 [src.conf(5)](https://man.freebsd.org/src.conf/) 文档，我自己目前还没有确认遗漏的部分（因为选项实在太多了…）。

## 执行 make buildworld

我们从没有缓存的状态开始执行 `make buildworld`。为了分析缓存命中率等信息，构建完成后，我运行了 `ccache -s` 来获取统计信息。

### 第一次执行结果

如下所示，几乎所有文件都进行了编译。由于 `ccache` 的开销，首次编译的时间比正常编译时间稍长。此外，虽然发生了一些重新编译，但比例极小，不到 0.2%，可以忽略不计。

**第一次执行结果（从无缓存状态开始构建）**

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

此外，实际的缓存使用量并未反映出来，这一点导致了一些不准确的结果。稍后将详细说明这一点。

### 第二次执行结果

在收集上述数据后，我再次执行了 `buildworld`。如下所示，结果显示了 100% 的缓存命中率，最终得出了「50.09%」的结果。

**第二次执行结果（从缓存状态中进行构建）**

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

### 清空统计信息后的第三次执行结果

执行了 `ccache -z` 清空统计信息后，再次进行了 `buildworld`。如下所示，结果显示了 100% 的缓存命中率。这次没有进行任何编译，因此 `buildworld` 执行时间大大缩短。

**清空统计信息后的第三次执行结果（从缓存状态中进行构建）**

```sh
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

### 执行清理操作

关于缓存大小未能即时反映的问题，我执行了 `ccache -c` 清理操作，成功将统计数据更新至最新状态。

**清理后正常化的统计信息**

```sh
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

如你所见，即便是 5GB 的缓存，也能勉强满足需求。

## 基准测试

### buildkernel 示例

从 [karl](https://twitter.com/karl0204) 收到以下环境下的 `make buildkernel` 测量结果，并将其整理成表格。

- CPU： AMD Ryzen9 3900（3.10GHz／4.30GHz）
- 内存：32GB
- 存储：WesternDigital SN550（NVMe 连接）· ZFS 操作
- 构建目标：FreeBSD 14-CURRENT

| 构建步骤                    | 无 CCache 时间 | 有 CCache 时间 |
| ----------------------- | ----------- | ----------- |
| `make buildkernel`      | 1406 秒      | 837 秒       |
| `make -j4 buildkernel`  | 366 秒       | -           |
| `make -j8 buildkernel`  | 200 秒       | -           |
| `make -j12 buildkernel` | 155 秒       | 82 秒        |
| `make -j16 buildkernel` | 136 秒       | -           |
| `make -j20 buildkernel` | 123 秒       | -           |
| `make -j24 buildkernel` | 115 秒       | 55 秒        |

需要注意的是，CPU 资源未被完全消耗。

### buildworld 示例

- CPU：Intel Pentium N4200（1.10GHz／2.50GHz·Apollo Lake·Goldmont 架构）
- 内存：16GB
- 存储：Transcend MTS400S（SATA 连接）· ZFS 操作
- 构建目标：12.2-RELEASE-p11

| 构建步骤※        | 构建时间    |
| ------------ | ------- |
| 无 CCache     | 22317 秒 |
| 有 CCache（一轮） | 25603 秒 |
| 有 CCache（二轮） | 3841 秒  |
| 有 CCache（三轮） | 3854 秒  |

※均指定了 `make -j5 buildworld`

## 总结

- 无论构建并行度（`-j` 选项的设置）高低如何，100% 缓存命中时都会带来几倍级别的时间缩短效果（几分钟内完成构建）。
- 对于 `buildkernel` 这种几乎全由 C 语言构成的构建，效果约为两倍；而对于包含 LLVM 这类重型构建的 `buildworld`，效果可达到六倍左右。
- 引入 ccache 后的初始开销为 14%，但作为二次及后续构建效果的代价，这一开销可以忽略。
- 在开发等场景下，经常需要稍微修改并重新构建的情况，ccache 的效果非常显著。
- 本次测试还表明，在纯粹的重新构建场景下，100% 不会重新编译。这一点在执行 `make clean` 后，即使对象文件被删除，也得到了验证。
- 近年来，FreeBSD 的开发环境（如 LLVM）的构建时间显著增加，而 ccache 在缩短构建时间（无论是时间上还是内存上）方面效果极为显著。
- 然而，初次编译时所需的内存仍然是个问题，因此在内存较小的环境下进行构建仍然是一个挑战。

### ccache 的代表性命令与选项

由于 ccache 本身较为简洁，因此相关的文档较少，下面简单总结了一些常用命令与选项。

| 命令            | 含义                    | 备注             |
| ------------- | --------------------- | -------------- |
| `ccache -M ｎ` | 设置缓存容量（如果未指定，默认为 `G`） | 指定最大缓存容量※      |
| `ccache -s`   | 显示统计信息                |                |
| `ccache -z`   | 清除统计信息                |                |
| `ccache -c`   | 清理缓存目录                | 清理操作不会删除已缓存的数据 |
| `ccache -C`   | 删除缓存                  |                |

※`ｎ` 可以指定的后缀包括：“`k`”，“`M`”，“`G`”，“`T`”，“`Ki`”，“`Mi`”，“`Gi`”，“`Ti`”。

## 常见问题及解答

### 问：能不能在 `/etc/make.conf` 中设置？

答：`/etc/make.conf` 会被必定读取，因此实质上与在 `/etc/src.conf` 中设置相同。不过，若在 FreeBSD 的构建以外的其他地方使用该设置，行为无法保证。如果不介意这些差异，可以在 `/etc/make.conf` 中设置。为了与其他构建区分，建议将设置放在 `/etc/src.conf` 中。
