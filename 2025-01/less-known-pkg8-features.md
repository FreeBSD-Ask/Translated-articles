# 不为人知的 pkg(8) 功能

- 原文：[Less Known pkg(8) Features](https://vermaden.wordpress.com/2019/01/17/less-known-pkg8-features/)
- 作者：𝚟𝚎𝚛𝚖𝚊𝚍𝚎𝚗
- 2019/01


我多次被要求撰写一篇关于 **pkg(8)** 的文章——这是 FreeBSD 现代软件包管理器，有时也称为 *PKGng*。

在本篇文章中，我将尝试介绍一些不太为人所知的 **pkg(8)** 功能。

大约在 8 年前——当时 **pkg(8)** 还未问世——我写过一篇帖子：[HOWTO: Keeping FreeBSD’s Base System and Packages Up-to-Date](https://forums.freebsd.org/threads/howto-keeping-freebsds-base-system-and-packages-up-to-date.26140/)。后来该文章甚至刊载于 **BSD Magazine 2012/01** 期（*Issue 30*）。

![bsd-magazine-2012-01](https://vermaden.wordpress.com/wp-content/uploads/2019/01/bsd-magazine-2012-01.jpg?w=960)

回到 2011 年，软件包更新比现在稍微复杂一些。那时，你不得不使用 FreeBSD 的 STABLE 分支，因为 RELEASE 分支的软件包几乎不更新——类似于当前 OpenBSD 的情况。FreeBSD STABLE 分支上的软件包每两周构建一次，当时已经足够使用。

当然，你也可以使用 **portmaster** 从 FreeBSD Ports 编译所有软件包，但这会浪费大量时间。当时 **pkg_add**/**pkg_delete**/**pkg_info** 是 FreeBSD 上的主要软件包工具，而 **bsdadminscripts** 包中的 **pkg_upgrade** 脚本在升级过程中非常有用。它会从 STABLE 分支的 FTP 服务器获取最新的软件包并更新已安装的软件包。要检查软件包的安全问题，则需要另一款第三方工具 **portaudit**。

现在，我们有了功能完善的 **pkg(8)**，能使用 **pkg upgrade** 来更新安装的软件包。得益于 **pkg audit**，已不再需要第三方工具 **portaudit** 了。我们甚至还有 **pkg autoremove** 来自动移除不再需要的依赖。

我会尽量不重复已经非常优秀的 [FreeBSD Handbook](https://freebsd.org/handbook) 中 [4.4. 使用 pkg 管理二进制软件包](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/pkgng-intro.html) 章节中已有的信息。

## 较旧的 FreeBSD 版本

在 FreeBSD 10 之前，如果要使用新的 **pkg(8)** 工具，非旧的 **pkg_*** 工具，则需要在 **/etc/make.conf** 文件中加入 **WITH_PKGNG=yes**。

目前受支持的 FreeBSD RELEASE 版本只有最近发布的 12.0 以及更加稳定和完善的 11.2，因此无需在 **/etc/make.conf** 文件中加入某些内容来使用 **pkg(8)** 框架。

## 数据库

**pkg(8)** 数据库（实际上是 SQLite 数据库）保存在目录 **/var/db/pkg**。

下面是 **pkg(8)** 引导过程之后目录 **/var/db/pkg**。

```sh
# find /var/db/pkg
/var/db/pkg
/var/db/pkg/FreeBSD.meta
/var/db/pkg/vuln.xml
/var/db/pkg/local.sqlite
/var/db/pkg/repo-FreeBSD.sqlite
```

最重要的文件是 **/var/db/pkg/local.sqlite**，因为它是已安装包及其文件的数据库。在输入 **pkg shell** 后，你实际上可以通过 SQLite 解释器连接到这个 SQLite 数据库。

```sh
# pkg shell
-- Loading resources from /home/vermaden/.sqliterc
SQLite version 3.15.2 2016-11-28 19:13:37
Enter ".help" for usage hints.
> .q
#
```

若由于某种原因你发现 **pkg(8)** 工具无法工作或已经损坏，你可以使用 **sqlite3** 包中的 **sqlite3** 命令连接到它。不要使用软件包 **sqlite**，因为它是 SQLite 的 2.x 版本，而这与 **pkg(8)** 使用的 3.x 版本不向前兼容。

```sh
# file /var/db/pkg/*
/var/db/pkg/FreeBSD.meta:        ASCII text
/var/db/pkg/local.sqlite:        SQLite 3.x database, user version 34, last written using SQLite version 3015002
/var/db/pkg/repo-FreeBSD.sqlite: SQLite 3.x database, user version 2014, last written using SQLite version 3015002
/var/db/pkg/vuln.xml:            XML 1.0 document, UTF-8 Unicode text, with very long lines

# sqlite3 /var/db/pkg/local.sqlite
-- Loading resources from /home/vermaden/.sqliterc
SQLite version 3.26.0 2018-12-01 12:34:55
Enter ".help" for usage hints.
> .q
#
```

## 锁定/解锁

使用 **pkg(8)**，现在可以用 **pkg lock** 命令锁定指定的包。这意味着 **pkg upgrade**、**pkg delete**（甚至 **pkg autoremove**）操作都不会影响它们。你可以使用选项 **-l** 列出已锁定的包，如下所示。

```sh
# pkg lock -l
Currently locked packages:
conky-1.9.0_6
exfat-utils-1.2.8
ffmpeg-4.1_1,1
fusefs-exfat-1.2.8
lame-3.100_2

# pkg delete exfat-utils
Checking integrity... done (0 conflicting)
The following package(s) are locked and may not be removed:

        exfat-utils

1 packages requested for removal: 1 locked, 0 missing
# 
```

如你所见，无法 **pkg delete** 已锁定的包 **exfat-utils**。你必须先用 **pkg unlock** 命令将其解锁。你可以交互式地进行，也可以使用选项 **-y** 非交互式完成，如下所示。

```sh
# pkg unlock exfat-utils
exfat-utils-1.2.8: unlock this package? [y/N]: y
Unlocking exfat-utils-1.2.8

# pkg lock -y exfat-utils
Locking exfat-utils-1.2.8
```

为什么要锁定某些软件包？

根据我的经验，以下是需要锁定特定软件包的潜在原因：

- 你将 Ports 和软件包结合使用。
- 某个 Port 并不存在对应的软件包。
- 官方软件包的默认编译选项与你自己的不同。
- 你确实想使用旧版本的软件包。

实际上，我使用 lock/unlock 机制，是因为上述所有情况对我都成立。

我会同时使用 Ports 和软件包（在 FreeBSD 世界中这通常不被推荐），因为我使用的一些软件由于授权问题无法提供为软件包。例如所有和 Microsoft exFAT 文件系统相关的东西（**exfat-utils**/**fusefs-exfat**）以及 MP3（**lame**）。更令我惊讶的是，OpenBSD 多年来一直提供 **lame** 软件包，而 FreeBSD 团队仍然害怕专利问题。

我还需要构建自定义版本的 **ffmpeg** 软件包——只是为了加入 **lame** 支持，但仍然是自定义的。最后一个我保持锁定的是 Conky。它在 1.9 版本时很好用，但开发者在 1.10 版本（现在甚至已有 1.11）把它彻底搞坏了。比如，你无法在桌面上右键点击得到 Openbox 菜单——换句话说，Conky 不再把鼠标事件传递给负责桌面的 Window Manager。

于是我使用了 Ports 的另一个工具 **portdowngrade**，把 1.9 版本的文件提取到 Ports，然后编译出 1.9 的 **conky** 软件包并永久锁定。

你可能已经知道，我更喜欢使用 **dzen2** 来显示界面信息，但我仍然会在需要时用 **conky** 做“FreeBSD Dashboard”，然后按下 [Scroll Lock] 键临时启用它。

## 提供项

如果你同时也是 RHEL/Fedora（或一般 **yum**/**rpm**）的用户，你可能会在 FreeBSD 的 **pkg(8)** 包管理器中怀念“provides”这个功能。为什么它如此有用？因为有了“provides”数据库，你可以直接通过指定软件包中某个二进制或文件的确切名称来安装软件包。例如你可以输入 **yum install /sbin/ifconfig** 来安装 **net-tools** 包，因为“provides”数据库中含有该信息。

如果我告诉你使用 **pkg(8)** 也能实现类似的功能呢？

[pkg-provides](https://github.com/rosorio/pkg-provides) 插件能让你直接使用 **pkg(8)** 查询哪个软件包提供了某个特定的文件。

它甚至已经作为软件包 **pkg-provides** 提供。下面我将展示如何安装配置。首先安装软件包 **pkg-provides**。


```sh
# pkg search provides
pkg-provides-0.5.0             Pkg plugin for querying which package provides a particular file

# pkg install pkg-provides
Updating FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
The following 1 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
        pkg-provides: 0.5.0 [FreeBSD]

Number of packages to be installed: 1

10 KiB to be downloaded.

Proceed with this action? [y/N]: y
[1/1] Fetching pkg-provides-0.5.0.txz: 100%   10 KiB   9.8kB/s    00:01    
Checking integrity... done (0 conflicting)
[1/1] Installing pkg-provides-0.5.0...
[1/1] Extracting pkg-provides-0.5.0: 100%
Message from pkg-provides-0.5.0:

======================= pkg plugin activation ========================
  In order to use the pkg-provides plugin you need to enable plugins in pkg.
  To do this, uncomment the following lines in /usr/local/etc/pkg.conf file
  and add pkg-provides to the supported plugin list

  PKG_PLUGINS_DIR = "/usr/local/lib/pkg/";
  PKG_ENABLE_PLUGINS = true;
  PLUGINS [ provides ];

  After that run `pkg plugins' to see the plugins handled by pkg`.

  To update the provides database run `pkg provides -u`

  ====================================================================
```


然后配置文件 **/usr/local/etc/pkg.conf**。

```sh
# cat << __EOF__ >> /usr/local/etc/pkg.conf
PKG_PLUGINS_DIR = "/usr/local/lib/pkg/";
PKG_ENABLE_PLUGINS = true;
PLUGINS [ provides ];
__EOF__
```

现在你就有了新命令 **pkg provides**，如下所示。

```sh
# pkg provides
usage: pkg provides [-uf] pattern

A plugin for querying which package provides a particular file

# pkg provides bin/pldd
Provides database not found, please update first.
```

你可以使用选项 **-u** 来更新 ‘provides’ 数据库。

```sh
# pkg provides -u
Fetching provides database: 100%   29 MiB 700.9kB/s    00:43    
Extracting database....success
```

**pkg provides** 插件示例用法

```sh
# pkg provides bin/pldd
Name    : ptools2-0.5
Desc    : Toolset based on Solaris ptools functionality
Repo    : FreeBSD
Filename: /usr/local/bin/pldd

Name    : linux_base-c7-7.4.1708_6
Desc    : Base set of packages needed in Linux mode (Linux CentOS 7.4.1708)
Repo    : FreeBSD
Filename: /compat/linux/usr/bin/pldd

# pkg install /compat/linux/usr/bin/pldd
Updating FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
pkg: No packages available to install matching '/compat/linux/usr/bin/pldd' have been found in the repositories
```

虽然例如无法通过输入 **pkg install /compat/linux/usr/bin/pldd** 命令来安装包 **linux_base-c7**，但可以检查哪个包包含该文件。

下次你执行命令 **pkg upgrade** 时，也会看到 provides 数据库的刷新。

```sh
# pkg upgrade
Updating FreeBSD repository catalogue...
Fetching meta.txz: 100%    944 B   0.9kB/s    00:01    
Fetching packagesite.txz: 100%    6 MiB 376.5kB/s    00:18    
Processing entries: 100%
Fetching provides database: 100%   29 MiB 386.3kB/s    01:18    
Extracting database....success
FreeBSD repository update completed. 32542 packages processed.
All repositories are up to date.
Checking integrity... done (0 conflicting)
(...)
```

**pkg provides** 数据库在目录 **/var/db/pkg** 中会占用相当量的空间。

```sh
# file /var/db/pkg/* /var/db/pkg/*/* | sort -n
/var/db/pkg/FreeBSD.meta: ASCII text
/var/db/pkg/local.sqlite: SQLite 3.x database, user version 34, last written using SQLite version 3015002
/var/db/pkg/provides: directory
/var/db/pkg/provides/provides.db: ASCII text
/var/db/pkg/repo-FreeBSD.sqlite: SQLite 3.x database, user version 2014, last written using SQLite version 3015002
/var/db/pkg/vuln.xml: XML 1.0 document, UTF-8 Unicode text, with very long lines
```

如果你使用像 LZ4 这样的 ZFS 压缩，那么它占用的空间就不会太大，如下所示。

```sh
# du -csm /var/db/pkg/*
1       /var/db/pkg/FreeBSD.meta
32      /var/db/pkg/local.sqlite
72      /var/db/pkg/provides
33      /var/db/pkg/repo-FreeBSD.sqlite
2       /var/db/pkg/vuln.xml
138     total
```


……但是如果你使用 UFS，那么这个将近 600 MB 的数据库可能会让你有点吃惊 :🙂:

```sh
# du -csmA /var/db/pkg/*
1       /var/db/pkg/FreeBSD.meta
68      /var/db/pkg/local.sqlite
571     /var/db/pkg/provides
52      /var/db/pkg/repo-FreeBSD.sqlite
6       /var/db/pkg/vuln.xml
694     total
```

## Which

虽然 **pkg provides** 可以提供尚未安装的软件包中文件的相关信息，**pkg which** 命令则是经典 UNIX **which** 命令在 **pkg(8)** 中的对应。它显示某个文件属于哪个软件包（或者根本不属于任何软件包）。

```sh
# pkg which /boot/modules/drm.ko
/boot/modules/drm.ko was installed by package drm-fbsd11.2-kmod-4.11g20181210

# pkg which /boot/kernel/drm.ko
/boot/kernel/drm.ko was not found in the database
```

### 双管齐下，乐趣加倍

有时同时使用两个 **which** 命令会更快地得到所需的答案。

```sh
# which firefox
/usr/local/bin/firefox

# pkg which `which firefox`
/usr/local/bin/firefox was installed by package firefox-64.0.2,1
```

## periodic 定期任务

有时你可能会看到如下情况。

```sh
# pkg install parallel
Updating FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
pkg: Cannot get an advisory lock on a database, it is locked by another process
```

…但是你并没有启动任何其他 **pkg(8)** 实例，这到底是怎么回事呢？我们来看一下 **ps(1)** 的输出。

```sh
# ps ax | grep pkg
 8540  -  S        0:00.00 /bin/sh - /usr/local/etc/periodic/daily/411.pkg-backup
 8551  -  S        0:00.00 /usr/local/sbin/pkg shell .dump
 8555  -  D        0:01.08 /usr/local/sbin/pkg shell .dump
```

FreeBSD 的 **periodic** 脚本正在执行它们的工作。

要查看具体是哪些脚本，可以查看这里。

```sh
# find /etc/periodic /usr/local/etc/periodic -name \*pkg\*
/usr/local/etc/periodic/daily/490.status-pkg-changes
/usr/local/etc/periodic/daily/411.pkg-backup
/usr/local/etc/periodic/security/460.pkg-checksum
/usr/local/etc/periodic/security/410.pkg-audit
/usr/local/etc/periodic/weekly/400.status-pkg
```

如果你认为这些活动中某些是不必要的，可以在 **/etc/periodic.conf** 文件中使用这些值将它们禁用。

```sh
# find /etc/periodic /usr/local/etc/periodic -name \*pkg\* | xargs grep -m 1 -E -o "[a-z_]+_enable" 
/usr/local/etc/periodic/daily/490.status-pkg-changes:daily_status_pkgng_changes_enable
/usr/local/etc/periodic/daily/411.pkg-backup:daily_backup_pkgng_enable
/usr/local/etc/periodic/security/460.pkg-checksum:security_status_pkgchecksum_enable
/usr/local/etc/periodic/security/410.pkg-audit:security_status_pkgaudit_enable
/usr/local/etc/periodic/weekly/400.status-pkg:weekly_status_pkgng_enable
```

例如，如果你想禁用 **/usr/local/etc/periodic/daily/490.status-pkg-changes** 的执行，你需要在 **/etc/periodic.conf** 文件中添加 **daily_status_pkgng_changes_enable=no**。

接下来再检查一次 **ps(1)** 输出。

```sh
# ps ax | grep pkg
 8574  0  S+       0:00.00 grep --color pkg
```

**periodic** 任务已经完成。你现在可以像平常一样安装你的包了。

```sh
# pkg install parallel
Updating FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
The following 1 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
        parallel: 20171222

Number of packages to be installed: 1

The process will require 3 MiB more space.
1 MiB to be downloaded.

Proceed with this action? [y/N]: n
#
```


## 统计信息

虽然 **pkg stats** 命令可以提供已安装包的一些统计信息，但它对于查找哪个包占用空间最多并不是很有用。

```sh
# pkg stats
Local package database:
        Installed packages: 1081
        Disk space occupied: 9 GiB

Remote package database(s):
        Number of repositories: 1
        Packages available: 32518
        Unique packages: 32518
        Total size of packages: 78 GiB
```

还有 **pkg size** 命令，它只会显示包占用的空间，但不会显示包名……没多大用处。

```sh
# pkg size | head
10.5MiB
2.06MiB
27.4MiB
2.59MiB
5.17MiB
515KiB
23.2MiB
609KiB
587KiB
127KiB
```

此外，**pkg size** 的手册页并不存在。

```sh
# man pkg-size
No manual entry for pkg-size
```

你可以使用 **pkg info -as** 命令，但它不仅不会对输出进行任何排序，还会以 KiB/MiB/GiB 等不同单位显示空间使用情况，这并不方便……幸运的是，**sort** 命令的选项 **-h** 能帮上忙。

使用以下别名可以按空间使用量对包进行排序。我将输出限制为最大 20 个包，但你可以根据需要修改。

```sh
# alias pkg-size='pkg info -as | sort -k 2 -h | tail -20 | column -t'
# which pkg-size
pkg-size: aliased to pkg info -as | sort -k 2 -h | tail -20 | column -t
# pkg-size
python27-2.7.15          68.2MiB
gtk3-3.22.30_4           68.8MiB
opencollada-1.6.68_1     75.8MiB
py27-ansible-2.7.5       88.6MiB
argyllcms-1.9.2_4        92.4MiB
webkit2-gtk3-2.22.5      92.9MiB
gimp-app-2.10.8_1,1      95.4MiB
python36-3.6.8           104MiB
samba47-4.7.12           145MiB
openjdk8-8.192.26_3      162MiB
boost-libs-1.69.0        163MiB
thunderbird-60.4.0_1     167MiB
firefox-64.0.2,1         174MiB
binutils-2.30_7,1        195MiB
linux_base-c6-6.10       197MiB
gcc6-6.5.0_3             241MiB
chromium-71.0.3578.98_2  251MiB
libreoffice-6.0.7_4      353MiB
virtualbox-ose-5.2.22_2  375MiB
llvm60-6.0.1_5           818MiB
```

## 缩写

**pkg(8)** 工具同样支持缩写参数。例如，你不必输入完整的 **pkg autoremove**，只需输入 **pkg autor** 即可执行该命令。

下面是简短名称示例。

```sh
# pkg autor
# pkg upg
# pkg inf
```

## 元数据

![vermaden\_2019-01-16\_21-32-07.png](https://vermaden.wordpress.com/wp-content/uploads/2019/01/vermaden_2019-01-16_21-32-07.png?w=960)

许多 **pkg(8)** 的问题都是由旧的元数据数据库引起的。如果你遇到任何 **pkg(8)** 问题，首先应如下面所示强制刷新其数据库。

```sh
# pkg update -f
Updating FreeBSD repository catalogue...
Fetching meta.txz: 100%    944 B   0.9kB/s    00:01    
Fetching packagesite.txz: 100%    6 MiB 352.9kB/s    00:19    
Processing entries: 100%
Fetching provides database: 100%   28 MiB 658.3kB/s    00:44    
Extracting database....success
FreeBSD repository update completed. 31778 packages processed.
All repositories are up to date.
```

为了记录——在这个过程中，“provides” 数据库也会被刷新。

## 修复损坏的依赖

曾经有段时间，缺失的软件包 **www/libxul19** 依赖问题折磨了我一阵子。

我甚至绝望地准备用 **portmaster** 重新编译所有东西。

我从命令 **portmaster --check-depends** 开始，但在系统询问是否修复时选择了“**n**”，因为这会不必要地降级大量软件包。

```sh
# portmaster --check-depends
(...)
Checking dependencies: evince
graphics/evince has a missing dependency: www/libxul19
(...)

>>> Missing package dependencies were detected.
>>> Found 1 issue(s) in total with your package database.

The following packages will be installed:

        Downgrading perl: 5.14.2_3 -> 5.14.2_2
        Downgrading glib: 2.34.3 -> 2.28.8_5
        Downgrading gio-fam-backend: 2.34.3 -> 2.28.8_1
        Downgrading libffi: 3.0.12 -> 3.0.11
        Downgrading gobject-introspection: 1.34.2 -> 0.10.8_3
        Downgrading atk: 2.6.0 -> 2.0.1
        Downgrading gdk-pixbuf2: 2.26.5 -> 2.23.5_3
        Downgrading pango: 1.30.1 -> 1.28.4_1
        Downgrading gtk-update-icon-cache: 2.24.17 -> 2.24.6_1
        Downgrading dbus: 1.6.8 -> 1.4.14_4
        Downgrading gtk: 2.24.17 -> 2.24.6_2
        Downgrading dbus-glib: 0.100.1 -> 0.94
        Installing libxul: 1.9.2.28_1

The installation will require 66 MB more space

38 MB to be downloaded

>>> Try to fix the missing dependencies [y/N]: n
>>> Summary of actions performed:

www/libxul19 dependency failed to be fixed

>>> There are still missing dependencies.
>>> You are advised to try fixing them manually.

>>> Also make sure to check 'pkg updating' for known issues.
```

让我们看看 **pkg(8)** 显示我们已经安装了哪些软件包。

```sh
# pkg info | grep libxul
libxul-10.0.12                 Mozilla runtime package that can be used to bootstrap XUL+XPCOM apps

# pkg info -qoa | grep libxul
www/libxul
```

问题在于我们安装了 **www/libxul** 而不是 **www/libxul19**，这就是为什么 **portmaster**（不仅仅是它）会报错的原因。

在 **pkg(8)** 被引入之前，只需使用 **grep -r** 搜索整个 **/var/db/pkg** 目录及其“文件数据库”就很容易，但现在情况复杂得多，因为软件包数据库保存在 SQLite 数据库中。使用 **pkg shell** 命令，你可以连接到该数据库。让我们看看能找到什么。

```sh
# pkg shell
SQLite version 3.7.13 2012-06-11 02:05:22
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> .databases
seq  name             file
---  ---------------  ----------------------------------------------------------
0    main             /var/db/pkg/local.sqlite
sqlite> .tables
categories       licenses         pkg_directories  scripts
deps             mtree            pkg_groups       shlibs
directories      options          pkg_licenses     users
files            packages         pkg_shlibs
groups           pkg_categories   pkg_users
sqlite> .header on
sqlite> .mode column
sqlite> pragma table_info(deps);
cid         name        type        notnull     dflt_value  pk
----------  ----------  ----------  ----------  ----------  ----------
0           origin      TEXT        1                       1
1           name        TEXT        1                       0
2           version     TEXT        1                       0
3           package_id  INTEGER     0                       1
sqlite> .quit
```

所以现在我们知道，“**deps**” 表可能就是我们要找的 ;)。

由于 **pkg shell** 在浏览 SQLite 时功能相当有限，我将直接使用命令 **sqlite3**。所谓有限是指，你不能直接输入 **pkg shell "select * from deps;"** 这样的查询，而是需要先启动 **pkg shell**，然后才能输入查询语句。

```sh
# sqlite3 -column /var/db/pkg/local.sqlite "select * from deps;" | grep libxul
www/libxul19   libxul      1.9.2.28_1  104
```

第二列是 **name**，所以我们可以尝试使用它。

```sh
sqlite3 -header -column /var/db/pkg/local.sqlite "select * from deps where name='libxul';"
origin        name        version     package_id
------------  ----------  ----------  ----------
www/libxul19  libxul      1.9.2.28_1  104
```

现在我们已经找到了“有问题”的依赖条目，可以稍微修改它，使其与实际已安装的包状态一致。

```sh
# sqlite3 /var/db/pkg/local.sqlite "update deps set origin='www/libxul' where name='libxul';"
# sqlite3 /var/db/pkg/local.sqlite "update deps set version='10.0.12' where name='libxul';"
```

当然，你也可以使用“官方”方法，通过 **pkg shell** 命令来操作。

```sh
# pkg shell
SQLite version 3.7.13 2012-06-11 02:05:22
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> update deps set origin='www/libxul' where name='libxul';
sqlite> update deps set version='10.0.12' where name='libxul';
sqlite> .header on
sqlite> .mode column
sqlite> select * from deps where name='libxul';
origin      name        version     package_id
----------  ----------  ----------  ----------
www/libxul  libxul      10.0.12     104
sqlite> .quit
```

现在 **portmaster** 感到满意，不会再命令依赖缺失了。

```sh
# portmaster --check-depends
(...)
Checking dependencies: zenity
Checking dependencies: zip
Checking dependencies: zsh
#
```

太棒了！问题解决了 :😉:

……但 **pkg(8)** 已经自带一个工具可以做到这一点 :🙂:

它叫 **pkg set**，在 **man pkg-set** 中最有用的两个选项是。


```sh
  -n oldname:newname, --change-name oldname:newname
       Change the package name of a given dependency from oldname to newname.
       将指定依赖项的包名从 `oldname` 修改为 `newname`。

(...)

  -o oldorigin:neworigin, --change-origin oldorigin:neworigin
       Change the port origin of a given dependency from oldorigin to neworigin.
       This corresponds to the port directory that the package originated from.
       Typically, this is only needed for upgrading a library or package that
       has MOVED or when the default version of a major port dependency changes.
       (DEPRECATED) Usually this will be explained in /usr/ports/UPDATING.
       Also see pkg-updating(8) and EXAMPLES.
       将指定依赖项的 Port 来源从 oldorigin 修改为 neworigin。
       这对应于该包最初来源的 Port 目录。通常，仅在升级已被 MOVED 的库或包，或者主要 Port 依赖的默认版本发生变化时才需要使用此功能。（已弃用）通常相关说明会在 /usr/ports/UPDATING 中给出。
       另请参见 pkg-updating(8) 和示例。
```


在我们的例子中，我们会使用 **pkg set -o www/libxul19:www/libxul** 命令。

不确定它是否能以同样方式解决问题，因为我在数据库中也更新了版本。

## 更新（UPDATING）

如果在执行 **pkg upgrade** 命令时遇到任何问题，那么你也应该查看最新版本的 **/usr/ports/UPDATING** 文件——例如可以在使用 **portsnap fetch update** 命令更新 Ports 后获取。

该文件描述了 Ports 中的重要变更（以及由于包是由 Ports 构建而来的，因此也涵盖了包的变更）。

```sh
# less /usr/ports/UPDATING

(...)
20180518:
  AFFECTS: users of sysutils/ansible*
  AUTHOR: lifanov@FreeBSD.org

  Ansible ports are now flavored. Package names for Ansible changed
  to include python version. Poudriere and package users don't need
  to do anything.

  To rename an installed package to match the new naming scheme,
  for example, for ansible24, run:

   # pkg set -n ansible24:py27-ansible24

(...)

20180214:
  AFFECTS: users of lang/ruby23
  AUTHOR: swills@FreeBSD.org

  The default ruby version has been updated from 2.3 to 2.4.

  If you compile your own ports you may keep 2.3 as the default version by
  adding the following lines to your /etc/make.conf file:

  #
  # Keep ruby 2.3 as default version
  #
  DEFAULT_VERSIONS+=ruby=2.3

  If you wish to update to the new default version, you need to first stop any
  software that uses ruby. Then, you will need to follow these steps, depending
  upon how you manage your system.

  If you use pkgng, simply upgrade:
  # pkg upgrade

  If you use portmaster, install new ruby, then rebuild all ports that depend
  on ruby:
  # portmaster -o lang/ruby24 lang/ruby23
  # portmaster -R -r ruby-2.4

  If you use portupgrade, install new ruby, then rebuild all ports that depend
  on ruby:

  # pkg delete -f ruby portupgrade
  # make -C /usr/ports/ports-mgmt/portupgrade install clean
  # pkg set -o lang/ruby23:lang/ruby24
  # portupgrade -x ruby-2.4.\* -fr lang/ruby24

(...)
```

**pkg(8)** 框架也提供了相应工具，即 **pkg updating** 命令。具体细节可查看 **man pkg-updating** 页面。最常见的用法是使用参数 **-d** 并指定日期，如下所示。

```sh
# pkg updating -d 20190101
20190103:
  AFFECTS: users of multimedia/vlc*
  AUTHOR: riggs@FreeBSD.org

  The multimedia/vlc port has been upgraded to 3.0.5, the latest upstream
  release. Subsequently, multimedia/vlc-qt4 and multimedia/vlc3 have been
  retired and removed from the ports tree. Users who previously used
  multimedia/vlc3 might want to switch to multimedia/vlc with the following
  commands:

  # pkg install multimedia/vlc
    or
  # portmaster -o multimedia/vlc multimedia/vlc3
    or
  # portupgrade -o multimedia/vlc multimedia/vlc3
```

你也可以在 [https://www.freshports.org/UPDATING](https://www.freshports.org/UPDATING) 在线查看 **UPDATING** 文件。

## 使用 ZFS 启动环境进行稳健升级

为了绝对确保无论 **pkg upgrade** 命令出现何种问题，你的系统都能正常工作，可以使用 *ZFS 启动环境*。我不久前曾在 [波兰 PBUG](https://vermaden.wordpress.com/2018/07/30/zfs-boot-environments-at-pbug/) 和 [荷兰 NLUUG](https://vermaden.wordpress.com/2018/11/15/zfs-boot-environments-reloaded-at-nluug-autumn-conference-2018/) 介绍过其功能。仍可在链接 [https://is.gd/BECTL](https://is.gd/BECTL) 下载最新的 PDF 演示文稿。

使用 **beadm** 命令的操作流程如下。

```sh
# beadm create safepoint
Created successfully

# beadm list
BE           Active Mountpoint  Space Created
11.2-RELEASE NR     /            5.7G 2018-12-01 13:09
safepoint    -      -          316.0K 2019-01-16 23:03

# pkg upgrade
```

现在，即使出现任何问题，你依然拥有名为 **safepoint** 的完整可用系统启动环境。

只需重启并选择该环境（在 FreeBSD loader 中选择），你就能恢复到正常工作的系统，就像使用时间机器回到过去一样。

## 查询

你也可以使用命令 **pkg query** 来查找所需的信息。

例如，要“模拟” **pkg info -r pkg-name** 参数（显示依赖 **pkg-name** 的软件包列表），可以使用如下方式的 **pkg query** 命令。

```sh
# pkg info -r sqlite3
sqlite3-3.26.0:
        colord-gtk-0.1.26
        py27-sqlite3-2.7.15_7
        freeciv-2.5.10
        colord-1.3.5
        libsoup-2.62.3
        libsoup-gnome-2.62.3
        subversion-1.11.0_1
        nss-3.41_1
        webkit-gtk2-2.4.11_19
        filezilla-3.36.0_1
        epiphany-3.28.5_1
        darktable-2.4.4_3
        aria2-1.34.0_1
        webkit2-gtk3-2.22.5
        qt5-webkit-5.212.0.a2_17
        qt5-sqldrivers-sqlite3-5.12.0
        hugin-2018.0.0_6
        pidgin-2.13.0
        thunderbird-60.4.0_1
        midori-0.7.0
        firefox-64.0.2,1

# pkg query -e '%n = sqlite3' %ro
graphics/colord-gtk
databases/py-sqlite3
games/freeciv
graphics/colord
devel/libsoup
devel/libsoup-gnome
devel/subversion
security/nss
www/webkit-gtk2
ftp/filezilla
www/epiphany
graphics/darktable
www/aria2
www/webkit2-gtk3
www/qt5-webkit
databases/qt5-sqldrivers-sqlite3
graphics/hugin
net-im/pidgin
mail/thunderbird
www/midori
www/firefox
```

如果你想知道每个包的初次安装时间，可以使用下面这个方法。

```sh
# pkg query "%t %n-%v" \
    | sort -n \
    | while read timestamp pkgname
      do
        echo "$(date -r $timestamp) $pkgname"
      done | ( head; echo; tail )
Fri Jul  7 14:17:29 CEST 2017 libpciaccess-0.13.5
Fri Jul  7 14:17:35 CEST 2017 libedit-3.1.20170329_2,1
Fri Jul  7 14:18:09 CEST 2017 font-util-1.3.1
Fri Jul  7 14:18:10 CEST 2017 xcb-util-0.4.0_2,1
Fri Jul  7 15:26:56 CEST 2017 xcb-util-renderutil-0.3.9_1
Fri Jul  7 15:26:57 CEST 2017 dejavu-2.37
Fri Jul  7 15:27:00 CEST 2017 font-misc-meltho-1.0.3_3
Fri Jul  7 15:27:02 CEST 2017 font-misc-ethiopic-1.0.3_3
Fri Jul  7 15:27:06 CEST 2017 font-bh-ttf-1.0.3_3
Fri Jul  7 15:27:08 CEST 2017 tpm-emulator-0.7.4_2

Sun Jan 13 20:48:01 CET 2019 firefox-64.0.2,1
Sun Jan 13 20:48:01 CET 2019 htop-2.2.0_1
Wed Jan 16 23:08:21 CET 2019 vlc-3.0.6,4
Wed Jan 16 23:08:21 CET 2019 xdg-utils-1.1.3
Wed Jan 16 23:08:25 CET 2019 phonon-qt4-4.10.2
Wed Jan 16 23:08:25 CET 2019 physfs-3.0.1
Wed Jan 16 23:08:25 CET 2019 py27-pyasn1-0.4.5
Wed Jan 16 23:08:26 CET 2019 chromium-71.0.3578.98_2
Wed Jan 16 23:08:26 CET 2019 moreutils-0.63
Wed Jan 16 23:08:26 CET 2019 p5-URI-1.76
```

你也可以显示那些不会被命令 **pkg autoremove** 移除的包，因为它们是你直接安装的。

```sh
# pkg query -e "%a != 1" "%n" | tail
xmp
xorg
xprintidle
xterm
xxkb
youtube_dl
zenity
zfs-stats
zip
zsh
```

## 罗塞塔石碑

[FreeBSD Wiki](https://wiki.freebsd.org/PkgPrimer) 页面也提供了一些表格，但信息不完整。

因此我复制了表格并补充了缺失的数据。

下面是旧 **pkg_*** 工具与当前 **pkg(8)** 框架的对照表（更新版）：

| 功能                | 旧 **pkg_*** 工具                                            | 新 **pkg(8)** 工具                                                                                |
| :----------------- | :--------------------------------------------------------- | :---------------------------------------------------------------------------------------------- |
| 列出已安装的包           | pkg_info                                                  | pkg info                                                                                       |
| 获取包的基本信息          | pkg_info pkgname-pkgversion                               | pkg info pkgname <br> pkg info category/name <br> pkg info pkgname-pkgversion                  |
| 获取包的详细信息          | N/A                                                       | pkg info -f pkgname <br> pkg info -f category/name <br> pkg info -f pkgname-pkgversion         |
| 列出已安装包中的所有文件      | pkg_info -L pkgname-pkgversion                            | pkg info -l pkgname <br> pkg info -l category/name <br> pkg info -l pkgname-pkgversion         |
| 查找哪个包提供某个文件       | pkg_info -W /path/to/my/file                              | pkg which /path/to/my/file                                                                     |
| 安装本地包             | pkg_add ./localpkg.tbz                                    | pkg add ./localpkg.txz                                                                         |
| 安装远程包             | pkg_add -r mypackage                                      | pkg install mypackage <br> pkg install category/name <br> pkg install pkgname-pkgversion       |
| 搜索远程包             | ls /usr/ports/* | grep mypackage                          | pkg search mypackage <br> pkg search category/name <br> pkg search pkgname-pkgversion          |
| 搜索远程包的详细信息        | make search name=mypackage <br> make search key=mypackage | pkg search -f mypackage <br> pkg search -f category/name <br> pkg search -f pkgname-pkgversion |
| 已安装包的反向依赖         | pkg_info -R pkgname-pkgversion                            | pkg info -r mypackage <br> pkg info -r category/name <br> pkg info -r pkgname-pkgversion       |
| 已安装包的依赖           | pkg_info -r pkgname-pkgversion                            | pkg info -d mypackage <br> pkg info -d category/name <br> pkg info -d pkgname-pkgversion       |
| 删除作为依赖安装的未使用包     | N/A                                                       | pkg autoremove                                                                                 |
| 二进制升级已安装包         | pkg_upgrade (FreeBSD Ports)                               | pkg upgrade                                                                                    |
| 创建远程仓库            | N/A                                                       | pkg repo /directory/with/packages                                                              |
| 操作 jail 中的包       | N/A                                                       | pkg -j                                                                                         |
| 操作 chroot 中的包     | pkg_add -C                                                | pkg -c                                                                                         |
| 使用正则表达式获取已安装包信息   | pkg_info -x                                               | pkg info -x                                                                                    |
| 使用扩展正则表达式获取已安装包信息 | pkg_info -X                                               | pkg info -X                                                                                    |
| 使用通配符获取已安装包信息     | pkg_info                                                  | pkg info -g                                                                                    |
| 检查已知漏洞            | portaudit (FreeBSD Ports)                                 | pkg audit                                                                                      |
| 过期包               | pkg_version -l <                                          | pkg version -l <                                                                               |
| 过期包（安装信息详细）       | pkg_version -Il <                                         | pkg version -Il <                                                                              |
| 与远程仓库对比的过期包       | N/A                                                       | pkg upgrade -n                                                                                 |
| 已安装包统计信息          | N/A                                                       | pkg stat                                                                                       |
| 检查缺失依赖（并修复）       | N/A                                                       | pkg check -d                                                                                   |
| 包来源               | pkg_info -o                                               | pkg info -o                                                                                    |

如果你知道其他有用的 **pkg(8)** 命令，也可以告诉我 🙂






















































