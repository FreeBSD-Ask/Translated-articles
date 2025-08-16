# FreeBSD Periodic 系统简介

- 原文：[An Introduction to FreeBSD’s Periodic System](https://freebsdfoundation.org/blog/an-introduction-to-freebsds-periodic-system/)
- 2025 年 7 月 8 日
- 作者：Benedict Reuschling

FreeBSD 的 periodic 实用程序是一款内置系统，用于以 shell 脚本的形式安排和运行定期（每日、每周、每月）维护任务。它们包括系统健康检查、安全审计和清理任务。借助 periodic 的模块化结构，也可以把自定义任务集成到现有框架中。

本文将介绍如何使用系统提供的 periodic 脚本，以及如何集成我们自己的脚本。

## periodic 脚本的位置

以下目录包含计划通过 periodic 系统运行的脚本：

* `/etc/periodic/daily`、`/etc/periodic/weekly`、`/etc/periodic/monthly`：应在特定周期运行的脚本（例如每周一次）。
* `/etc/periodic/security`：与安全相关的检查，例如已启用的防火墙或登录失败情况。
* `/usr/local/etc/periodic/daily`、`/usr/local/etc/periodic/weekly`：第三方脚本，通常来自 Ports 或包，也按时间运行。例如，轮转 nginx 日志文件或备份 pkg 文件。
* `/usr/local/etc/periodic/security`：来自第三方的安全检查脚本，例如运行 `pkg audit` 的脚本。

调度本身通过 `/etc/crontab` 中的以下三行实现：

```
1       3       *       *       *       root    periodic daily
15      4       *       *       6       root    periodic weekly
30      5       1       *       *       root    periodic monthly
```

要配置 periodic 系统本身以及应运行的脚本，FreeBSD 提供了单独的配置文件 `/etc/periodic.conf`。默认情况下，该文件为空或不存在。`/etc/defaults/` 目录提供了一个带有详细注释的示例文件。更多信息可参考手册页 [periodic.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=periodic.conf)。

例如，要激活每日备份 `/etc/passwd` 和 `/etc/group` 文件（位于 `/etc/periodic/200.backup-passwd` 脚本中），在 `/etc/periodic.conf` 添加如下行：

```ini
daily_backup_passwd_enable="YES"
```

不需要提供前缀数字（示例中为 `200`）。该数字用于在同一类别（这里为 daily）中有多个脚本时控制运行顺序。在配置文件中逐行列出所有需要运行的脚本。

当这些脚本执行时，产生的输出会发送到系统管理员账号（root）。若要将输出重定向到 `/var/log` 中的文件，可分别为 `daily`、`weekly` 和 `monthly` 脚本添加以下行：

```ini
daily_output=/var/log/daily.log
weekly_output=/var/log/weekly.log
monthly_output=/var/log/monthly.log
```

文件名可以任意选择，只要不与已有日志文件重名即可。使用这种方法，这些日志文件在过分膨胀时会被轮替。`/etc/newsyslog.conf` 中的配置会处理上述三个文件的轮替。

## 添加自定义 periodic 脚本

下面的自定义脚本用于检查系统中所有 ZFS 池的容量是否达到 80%。如果达到阈值，会输出当前容量和对应的池名称：

```sh
#!/bin/sh

if [ -r /etc/defaults/periodic.conf ]
then
  . /etc/defaults/periodic.conf
  source_periodic_confs
fi

: ${zfs_pool_usage_enable:="YES"}
: ${zfs_pool_usage_threshold:=80}

[ "$zfs_pool_usage_enable" = "YES" ] || exit 0

echo ""
echo "Checking ZFS pool usage (threshold: ${zfs_pool_usage_threshold}%)..."

zpool list -H -o name,capacity | while read -r pool usage; do
    percent=${usage%%%}  # 移除 '%' 符号
    if [ "${percent}" -ge "${zfs_pool_usage_threshold}" ]; then
        echo "WARNING: ZFS pool '${pool}' is ${percent}% full!"
    else
        echo "OK: ZFS pool '${pool}' is below capacity threshold (${percent}%)."
    fi
done

exit 0
```

接下来使用提升权限将脚本设为可执行：

```sh
chmod +x /etc/periodic/daily/405.zfs_pool_usage
```

在全局配置文件 `/etc/periodic.conf` 中激活该脚本，并将阈值降低到 75%：

```ini
daily_show_success="YES"
zfs_pool_usage_enable="YES"
zfs_pool_usage_threshold="75"
```

如果没有 `zfs_pool_usage_threshold` 这行，脚本会使用默认值 80%。`daily_show_success` 用于在日志文件中查看脚本输出。如果在测试中将其设为 `NO`，你将看不到日志输出，而手动执行脚本则正常。

要测试脚本，以 root 权限运行：

```ini
periodic daily
```

该命令会遍历每个 daily 脚本以确定哪些需要运行。完成后，你会在 `/var/log/daily.log` 文件末尾看到新的输出行。

示例输出：

```sh
Checking ZFS pool usage (threshold: 75%)...
OK: ZFS pool 'data' is below capacity threshold (6%).
OK: ZFS pool 'zroot' is below capacity threshold (27%).
```

编写自定义脚本的建议：

* 确保脚本运行时间不过长
* 使用正确的退出码（成功为 0，失败为非零）
* 减少输出内容，仅保留必要信息，避免日志过快增长
* 脚本以非交互方式运行，不要等待用户输入或执行需要交互的命令
* 添加错误处理以捕获异常情况并提示，不要让脚本静默失败
* 通过手动调用和 periodic 调用分别测试脚本

## 总结

掌握这些技巧后，你可以向系统添加各种有用的任务。查看上文列出的目录，了解已有功能，避免重复造轮子。此外，安装 Ports 时可查看 `/usr/local/etc/periodic` 是否提供了 periodic 脚本，它们有特定用途，能很好地融入你的定期任务计划中。
