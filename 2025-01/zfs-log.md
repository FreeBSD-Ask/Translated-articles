# FreeBSD 上的 ZFS 日志压缩

- 原文：[ZFS Log Compression on FreeBSD](https://freebsdfoundation.org/blog/zfs-log-compression-on-freebsd/)
- 2025-4-16

系统日志是关于系统运行时发生事件的重要信息来源。FreeBSD 默认的日志系统是 syslog，它会根据主题将日志写入 `/var/log/` 下的不同文件。最常见的事件和那些不属于任何其他类别的事件，会被写入 `/var/log/messages`。

## 日志轮转

随着时间推移，日志会不断增长，并由守护进程 `newsyslog` 进行轮转。在轮转时，`newsyslog` 会创建新的日志文件，同时重命名旧的日志文件，同时对其进行压缩以节省空间。默认的命名规则是根据文件的时间添加（或增加）一个数字。例如，当前的日志文件会被加上扩展名 `.1`，而旧的 `.1` 文件会被重命名为 `.2`。超出 `.7` 的日志文件会作为最老的日志被删除。两个配置文件控制了日志的大小、日志总数以及其他轮转规则和日志行为：`/etc/newsyslog.conf` 和 `syslog.conf`。

## ZFS 数据压缩

OpenZFS 提供了一些强大而高效的压缩算法，能实时压缩数据集中的数据。每当系统写入日志文件时，ZFS 会在主内存中对其进行压缩，然后再写入底层存储。当再次读取文件时，ZFS 会自动解压所请求的数据，并以未压缩形式呈现给应用程序。这种方式在压缩率和便捷性上都具有巨大优势，因为文件系统会自动完成所有压缩步骤。而传统的 syslog 压缩方式仍是通过对文件进行压缩和解压来实现。当访问旧日志文件时，用户需要先使用类似 `bzcat` 的程序进行解压。而 ZFS 往往能提供更高的压缩率，因此也非常适合接管 syslog 的压缩任务。

## 配置 `newsyslog`

在基于 ZFS 的根系统中，`/var/log` 位于一个单独的数据集上。使用以下命令查看当前使用的压缩算法：

```sh
zfs get compression zroot/var/log
```

我们在此使用 FreeBSD 安装器默认的存储池名称（`zroot`）。如果运行 `zpool list -Ho name` 返回了不同的名称，请根据实际情况修改命令。如果输出为 `off`，请将其设置为 [`zfsprops`](https://man.freebsd.org/cgi/man.cgi?query=zfsprops) 手册页中列出的某种压缩算法。较好的开始是 `lz4` 或 `zstd`：

```sh
zfs set compression=zstd zroot/var/log
```

请注意，ZFS 只会压缩新的数据，不会影响数据集中已存在的旧文件。接下来我们将对 `newsyslog` 配置做些修改，以禁用其自身的压缩。FreeBSD 在 [`newsyslog.conf(5)`](https://man.freebsd.org/cgi/man.cgi?query=newsyslog.conf) 中对 `/etc/newsyslog.conf` 文件各字段做了详细说明。配置文件最后一列（标志位）包含了在轮转日志时要应用的压缩算法。既然我们要让 ZFS 负责压缩，请删除每个相关日志条目的此列中的标志 `J`。修改完成后保存并退出 `newsyslog.conf`。

## 修改 syslog 的轮转命名方案

这一步不是必须的，但可以更方便地识别某个被轮转日志的时间点。前面提到，旧日志文件会被加上从 `.1` 到 `.7` 的扩展名。在查找某个特定日期的旧日志时，可能需要逐个打开这些文件，十分麻烦。如果直接将轮转日期加入文件名会不会更方便？例如，`messages.20240914T210000` 表示该 `messages` 文件最后一次轮转发生在 2024 年 9 月 14 日 21:00:00。当然，这种格式是可配置的，可参考 `newsyslog` 的 `-t` 参数说明。

我们可以通过编辑 `/etc/crontab`，将这种命名方式应用于每个日志文件。找到如下带注释的行：

```ini
# Rotate log files every hour, if necessary.
0       *       *       *       *       root    newsyslog
```

修改为如下内容：

```ini
# Rotate log files every hour, if necessary.
0       *       *       *       *       root    newsyslog -t DEFAULT
```

保存并退出。

## 应用更改

为了让这些更改生效，重启守护进程 `newsyslog` 和 `syslog`：

```sh
service newsyslog restart
service syslogd restart
```

ZFS 现在将处理 `/var/log` 中的未压缩日志文件。日志轮转依然进行，但旧日志不再由 `newsyslog` 进行压缩。要检查 ZFS 压缩带来的空间节省效果，可运行以下命令：

```sh
zfs get refcompressratio zroot/var/log
```

下面是某个几周前进行了此更改的系统输出结果：

```sh
NAME           PROPERTY          VALUE     SOURCE
zroot/var/log  refcompressratio  40.60x    -
```

这意味着日志文件大小仅为原始未压缩大小的 1/40。ZFS 还提供了关于未压缩前大小的属性信息：

```sh
zfs get used,logicalreferenced zroot/var/log
NAME           PROPERTY           VALUE   SOURCE
zroot/var/log  used               2.28M   -
zroot/var/log  logicalreferenced  91.0M   -
```

对于如此小的改动，获得如此多的空间节省是十分可观的。新的日志文件在今后轮转时也将采用新的命名方案。若想进一步了解 FreeBSD 中的日志系统，可阅读 [《系统日志配置》](https://docs.freebsd.org/en/books/handbook/config/#configtuning-syslog) 手册章节。

虽然这些更改很少而且简单，我们还为您编写了一个 Ansible playbook 来实现自动化 😊 [点击这里查看](https://github.com/FreeBSDFoundation/blog/tree/main/zfs-log-compression-on-freebsd)。
