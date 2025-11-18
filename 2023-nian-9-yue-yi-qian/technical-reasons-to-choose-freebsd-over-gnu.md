# 选择 FreeBSD 而非 GNU/Linux 的技术性原因

- 原文：<https://unixsheikh.com/articles/technical-reasons-to-choose-freebsd-over-linux.html>
- 最后更新日期：2023-10-27

>在我的主工作站和所有服务器上都将 FreeBSD 作为日常使用的操作系统，自 1999 年起我就一直在使用 FreeBSD，因为我认为它是“一款了不起的操作系统”。在本文中，我将讨论选择 FreeBSD 而非 GNU/Linux 发行版的若干技术原因。

## 完整的操作系统

在 FreeBSD 上，你会立刻注意到，你面对的是“完整的操作系统”。所有不同的组件都是统一开发的。这意味着，如果一款组件的变动会影响整个系统，开发者可以更容易地在实施变更前考虑整体情况，并进一步规划和开发受影响的组件。BSD 内核、init 系统、用户空间工具、Ports 和包管理器，所有这些都是由项目成员开发并集成到一款系统中。例如，FreeBSD 上的 [top](https://www.freebsd.org/cgi/man.cgi?query=top) 命令（参见 ZFS ARC Stats 部分）就整合了 ZFS [ARC](https://en.wikipedia.org/wiki/Adaptive_replacement_cache)（自适应替换缓存）的信息。

内核和基本系统与第三方应用完全分离。基本系统配置放在 `/etc`，而所有第三方配置放在 `/usr/local/etc`。你可以配置和调优的一切内容，都在 man 手册中有详细记录。

你拥有从 `rc` 工具（由 [init](https://www.freebsd.org/cgi/man.cgi?query=init) 调用以控制自动启动过程的命令脚本）、[命令脚本](https://www.freebsd.org/cgi/man.cgi?query=rc)、[sysctl](https://www.freebsd.org/cgi/man.cgi?query=sysctl) 内核管理工具，到所有不同的 [系统配置](https://www.freebsd.org/cgi/man.cgi?query=rc.conf) 等功能，这一切都被很好地整合和文档记载。

因为 FreeBSD 是完整的操作系统，而不是像 Linux 发行版那样“拼凑”而成，一切设计得井井有条，基于多年的经验，并且当系统发生变化时，这些变化对整个社区都是有利的，同时参考了大量真实的使用案例和行业问题反馈。

作为对比，我最喜欢的 Linux 发行版之一 Debian GNU/Linux 有其特定的 [Debian 方式](https://wiki.debian.org/DontBreakDebian)。Debian 方式代表了一套特定的配置管理工具和补丁，使第三方软件符合 Debian 的设置方式。虽然在某种程度上这可以统一 Debian 中的操作方法，但它不幸地破坏了上游配置，这可能会带来很大困扰。尤其是在某些功能无法正常工作，或者上游文档与 Debian 的实际设置不匹配时，这种问题尤其突出。另一个问题是，一些第三方软件，甚至 Debian 的核心组件，如 systemd，无法完全按照“Debian 方式”来处理。结果就是操作系统中有些部分运行“Debian 方式”，而有些部分则不是。Debian GNU/Linux 已经整合了 systemd，但同时默认网络部分仍是 Debian 特有的。有时你必须禁用并移除 Debian 特有的配置，才能让 systemd 的功能正常工作。这一切都源于由许多不同项目的不匹配组件拼凑而成的系统。

另一方面，我同样喜欢的 Linux 发行版 Arch Linux 希望第三方软件保持上游状态，除非绝对必要，否则不做修改。这很好，因为上游文档与软件保持一致。然而，这虽然有助于改善整体系统管理，但事实仍然是 Linux 内核、用户空间工具以及其他所有组件都是由不同实体开发的。完全不同项目之间的冲突，例如 Linux 内核与 systemd 开发者之间的冲突，可能会导致操作系统无法正常运行。而 FreeBSD 不会出现这种问题，因为它是完整的操作系统。

我从不喜欢的 Ubuntu Linux 发行版则更糟。由于它基于“Debian unstable”，运行了大量 Debian 工具和配置，但同时又有“Ubuntu 方式”，在 Debian 基础上做了修改。除此之外，还在其上增加了一层 GUI，也就是所谓的用户改进工具层，有时会导致 Ubuntu 出现难以理解的崩溃问题。

## 文档

有些人认为文档不是使用某项技术的技术理由之一，但文档应被视为其所述技术的组成部分。粗劣的文档、过时的文档以及缺失的文档都应被视为一种 Bug。

FreeBSD 的文档随系统一同分发，因此你不必去网上搜索。基本系统的 man 手册质量极高，专为 FreeBSD 编写，你大多数所需的信息都可以在系统中直接找到。

FreeBSD 还有 [FreeBSD 手册](https://docs.freebsd.org/en/books/handbook/)，覆盖了从安装到日常使用的各个方面。安装时也可以将手册本地安装。手册偶尔会有一些过时的章节，因为它是众多个人持续工作的成果，但总体来说，手册是定期更新且编写良好的。

## 安全

通常被攻破的不是操作系统本身，而是运行在操作系统上的程序。在某些情况下，被攻破的程序可能与操作系统交互，从而影响操作系统的安全。保障操作系统安全意味着你要确保计算机资源仅被授权人员用于授权用途。

FreeBSD 默认配置是为了性能优化，而不像 OpenBSD 那样默认安全，但 FreeBSD 提供了许多工具和选项，帮助你保护系统免受攻击者侵害。

在本文中无法提供 FreeBSD 安全选项和可用功能的详尽列表，因为安全主题本身足以写成一本书。

不过，我可以提及几项关键内容。

### 安全性安装时选项

在安装 FreeBSD 时，安装程序提供了一组可启用或禁用的选项：

- 隐藏其他用户 ID（UID）的进程
- 隐藏其他组 ID（GID）的进程
- 隐藏 jail 内的进程
- 隐藏消息缓冲区
- 禁用进程调试
- 随机化进程 ID
- 禁用 syslogd 网络功能
- 禁用 Sendmail
- 安全控制台
- 不可执行的栈和栈保护

FreeBSD 其他大部分内核级安全设置都可通过 sysctl `security.bsd` 访问，而且每隔几个月还会增加新的选项。你可以运行命令 `sysctl -d security.bsd` 来显示你当前 FreeBSD 安装中可用的选项。

```sh
# sysctl -d security.bsd
security.bsd: BSD security policy
security.bsd.stack_guard_page: Specifies the number of guard pages for a stack that grows
security.bsd.unprivileged_get_quota: Unprivileged processes may retrieve quotas for other uids and gids
security.bsd.hardlink_check_gid: Unprivileged processes cannot create hard links to files owned by other groups
security.bsd.hardlink_check_uid: Unprivileged processes cannot create hard links to files owned by other users
security.bsd.unprivileged_idprio: Allow non-root users to set an idle priority
security.bsd.unprivileged_proc_debug: Unprivileged processes may use process debugging facilities
security.bsd.conservative_signals: Unprivileged processes prevented from sending certain signals to processes whose credentials have changed
security.bsd.see_jail_proc: Unprivileged processes may see subjects/objects with different jail ids
security.bsd.see_other_gids: Unprivileged processes may see subjects/objects with different real gid
security.bsd.see_other_uids: Unprivileged processes may see subjects/objects with different real uid
security.bsd.unprivileged_read_msgbuf: Unprivileged processes may read the kernel message buffer
security.bsd.unprivileged_mlock: Allow non-root users to call mlock(2)
security.bsd.suser_enabled: processes with uid 0 have privilege
security.bsd.map_at_zero: Permit processes to map an object at virtual address 0.
```

### 漏洞统计

下面是 FreeBSD 与 Linux 的漏洞统计列表。FreeBSD 漏洞数量通常较少，这并不一定意味着 FreeBSD 比 Linux 更安全（尽管我认为它确实更安全），也可能是因为 Linux 的用户和开发者更多、关注度更高。然而，大多数 Linux 发行版的攻击面通常比 FreeBSD 要大得多。

```
+---------+---------+-------+
| 年份    | FreeBSD | Linux |
+---------|---------|-------|
| 1999    | 18      | 19    |
| 2000    | 27      | 5     |
| 2001    | 36      | 22    |
| 2002    | 31      | 15    |
| 2003    | 14      | 19    |
| 2004    | 15      | 51    |
| 2005    | 17      | 133   |
| 2006    | 27      | 90    |
| 2007    | 9       | 62    |
| 2008    | 15      | 71    |
| 2009    | 11      | 102   |
| 2010    | 8       | 123   |
| 2011    | 10      | 83    |
| 2012    | 10      | 115   |
| 2013    | 13      | 189   |
| 2014    | 18      | 130   |
| 2015    | 6       | 86    |
| 2016    | 6       | 217   |
| 2017    | 23      | 454   |
| 2018    | 29      | 177   |
| 2019    | 18      | 170   |
| 2020    | 31      | 126   |
| 2021    | 25      | 158   |
| 2022    | 1       | 73    |
|---------|---------|-------|
| 总计    | 430     | 2780  |
+---------+---------+-------+
```

有关具体漏洞的详细信息，可以访问 CVE Details 网站查看 [FreeBSD](https://www.cvedetails.com/vendor/6/Freebsd.html) 和 [Linux](https://www.cvedetails.com/product/47/Linux-Linux-Kernel.html?vendor_id=33) 的数据。

### 稳定性

FreeBSD 拥有出色的工程与版本管理实践。FreeBSD 从创意产生到公开发布经历多个步骤：

1. 当有人提出新想法并进行开发时，首先会经过同行技术审查。
2. 然后进入“current”分支进行集成测试，根据复杂性或潜在影响，进入“stable”分支的迁移窗口会做相应调整。
3. 接着进入“stable”分支，供更广泛的用户群测试，这通常也是 Beta 测试阶段，并有更广泛的社区参与。
4. 最后进入“release candidate”（RC）测试，通常会经历三轮测试，之后才成为正式“release”版本。

会发布软件补丁以修复漏洞和缺陷。

这些步骤使 FreeBSD 成为非常可靠且稳健的操作系统。

## Ports 数据集

[FreeBSD Ports 数据集](https://www.freebsd.org/cgi/man.cgi?query=ports) 是工程学上的一项惊人成就。NetBSD 的 [pkgsrc（包源）](https://en.wikipedia.org/wiki/Pkgsrc) 和 OpenBSD 的 [Ports 数据集](https://en.wikipedia.org/wiki/Ports_collection#OpenBSD_ports) 都起源于 FreeBSD 的 Ports 系统。

虽然 FreeBSD 也提供二进制包（就像 Debian Linux、Arch Linux 一样），由 [pkg](https://www.freebsd.org/cgi/man.cgi?query=pkg) 包管理器处理，但 FreeBSD 还可以从源码编译软件，并能让用户在编译时进行个性化配置。Arch Linux 的 [Build System](https://wiki.archlinux.org/title/Arch_Build_System) 实际上在很大程度上受 FreeBSD Ports 系统启发。然而，在 FreeBSD Ports 系统中，你可以在 `make` 过程中选择最相关的编译选项，而在 Arch Linux 上，你通常需要手动编辑和修改包维护者提供的 [PKGBUILD](https://wiki.archlinux.org/title/PKGBUILD) 脚本（基本上默认设置是必须接受的）。

FreeBSD Ports 数据集使用 [Makefiles](https://en.wikipedia.org/wiki/Makefile) 来自动化软件的编译、安装和卸载过程，通过 `make` 命令实现。组成单个 port 的文件包含了自动下载、解压、打补丁、配置、编译和安装应用所需的所有信息，在执行如 `make install` 或 `make install clean` 之类的初始命令后，几乎无需用户干预。如果该 port 依赖其他应用或库，它们会在安装前自动处理。

大多数 port 配置了一套默认选项，这些选项被认为适合大多数用户。然而，Ports 系统的重大优势在于，你可以在安装前通过 `make config` 命令更改这些配置选项。该命令会弹出一个基于文本的界面，能让用户选择所需选项。

截至本文发布时，Ports 数据集中可用的 port 已 30,000 余款。

## 滚动更新的第三方包

关于第三方包，你可以选择两个不同的分支：一个叫“quarterly”，另一个叫“latest”。

Quarterly 是从版本系统的 HEAD 分支在每年季度初（1 月、4 月、7 月和 10 月）截取的 Ports 分支的名称，也是由这些分支生成的二进制包集的名称。

Quarterly 分支为用户提供了更可预测、更稳定的 port 和包的安装与升级体验。这主要是通过只允许上游的非功能性更新来实现的。Quarterly 分支旨在接收安全修复，但也可能包含版本更新，或者回移的提交、错误修复以及 Ports 兼容性或框架更改——这取决于上游的操作。

然而，如果你选择“latest”分支，FreeBSD 在第三方包方面将成为滚动更新发行（rolling release），类似于 Arch Linux，从而获得最前沿的软件。

## Poudriere

Poudriere 是一个用于创建和测试 FreeBSD 包的工具。它利用 [FreeBSD jail](https://en.wikipedia.org/wiki/FreeBSD_jail) 系统来搭建隔离的编译环境。这些 jail 可用于为任何版本的 FreeBSD 构建二进制包。待包构建完成，它们的布局与官方镜像完全一致。这些包可被 FreeBSD 的 `pkg` 二进制包管理工具使用。

使用 Poudriere，你可以轻松地构建并设置自己的二进制包仓库，其中的包完全按照你的规格和需求进行构建。

Poudriere 可以处理整个 ports 树的批量构建、ports 树的特定子集，或者包括依赖的单个 port。它会自动构建包、生成构建日志文件、提供签名的 `pkg` 仓库，并支持在向 FreeBSD bug 跟踪系统提交补丁前测试 port 构建，还可以使用不同选项测试不同构建。Poudriere 在干净的 jail 环境中执行构建，可以使用 `zfs` 特定功能。这意味着不会污染宿主环境，不会留下文件，也不会意外删除或更改现有配置文件。

Poudriere 设置和使用都非常直接，因为它没有依赖，并且可以在任何受支持的 FreeBSD 版本上运行。

## ZFS

与 Linux 不同，[ZFS](https://github.com/openzfs/zfs) 文件系统在 FreeBSD 上是一等公民。这不仅意味着可以将根文件系统安装在 ZFS 上，并且安装程序开箱即支持这一功能，还意味着许多基本系统工具已经紧密集成并支持 ZFS。在 FreeBSD 上运行 ZFS 与在 Linux 上运行 ZFS 非常不同。在 FreeBSD 上，你可以获得更多专门用于调查 ZFS 性能问题或其他相关问题的工具。

ZFS 的一些重要特性包括：

- 为长期数据存储而设计，可无限扩展的数据存储规模，保证零数据丢失，并具有高度可配置性。
- 对所有数据和元数据进行分层校验，确保整个存储系统在使用时可以被验证，确认数据正确存储，或在数据损坏时进行修复。校验和存储在块的父块中，而非存储在块本身中。这与许多文件系统形成对比，后者的校验和（如果存在）与数据存储在一起，如果数据丢失或损坏，校验和也可能丢失或不正确。
- 可以存储用户指定数量的数据或元数据副本，或选择性的数据类型，以提高从重要文件和结构的数据损坏中恢复的能力。
- 在某些情况下，如果出现错误或不一致，可自动回滚文件系统和数据的最近更改。
- 自动（通常是静默）修复检测到的数据不一致和写入失败，对于所有可重建的数据错误。数据可以通过以下方式重建：每个块的父块中存储的错误检测和纠正校验和；磁盘上的数据多副本（包括校验和）；记录在 SLOG（ZIL）上的写入意图，用于未发生的写入（如断电）；来自 RAID/RAIDZ 磁盘和卷的奇偶校验数据；来自镜像磁盘和卷的数据副本。
- 原生支持标准 RAID 级别及 ZFS 特有 RAID 布局（RAIDZ）。RAIDZ 级别仅在必要的磁盘上条带化数据以提高效率（许多 RAID 系统会无差别地在所有设备上条带化），并且校验和可将不一致或损坏的数据重建限制在有缺陷的块上。
- 原生支持分层存储和缓存设备，这通常是卷相关的任务。由于 ZFS 还理解文件系统，它可以利用与文件相关的知识来优化分层存储管理，而单独的设备无法做到。
- 原生支持快照和备份/复制，通过整合卷和文件处理实现高效操作。相关工具在低级别提供，使用时需外部脚本和软件。
- 原生支持数据压缩和去重，尽管去重主要在内存中进行且占用内存较多。
- 高效重建 RAID 阵列——RAID 控制器通常需要重建整个磁盘，但 ZFS 可结合磁盘和文件知识，仅重建实际丢失或损坏的数据，大大加快重建速度。
- 不受 RAID 硬件更换影响，而这会影响许多其他系统。在许多系统中，如果独立 RAID 硬件（如 RAID 卡）故障，或数据迁移到其他 RAID 系统，文件系统将缺少原 RAID 硬件上的信息，这些信息对于管理 RAID 阵列至关重要。ZFS 自行管理 RAID，因此 ZFS 池可以迁移到其他硬件，或者操作系统可以重新安装，RAIDZ 结构和数据仍会被 ZFS 识别并立即可用。
- 能够识别本应存在于缓存但最近被丢弃的数据，从而让 ZFS 根据后续使用重新评估缓存策略，实现非常高的缓存命中率（ZFS 缓存命中率通常大于 80%）。
- 可使用替代缓存策略处理可能导致数据处理延迟的数据。例如，可以将可能减慢存储系统的同步写入转换为异步写入，通过写入快速独立的缓存设备（称为 SLOG，有时也称为 ZIL – ZFS Intent Log）。
- 高度可调——许多内部参数可配置以实现最佳功能。
- 可用于高可用性集群和计算，尽管并非专门为此设计。

当然，在 Linux 上使用 ZFS 也能获得这些功能。然而，有一个很大的区别：没有任何 Linux 发行版能接近 FreeBSD 对 ZFS 的集成水平。

## 启动环境

由于与 ZFS 的紧密集成，FreeBSD 还支持启动环境（Boot Environments）。通过启动环境，你可以安装多个核心操作系统版本，并选择启动哪个。因此，启动环境就是工作系统的可启动克隆或快照。借助启动环境，你可以执行“无忧升级”或系统更改，无需担心破坏系统，因为你随时可以回滚并舍弃修改。

这也意味着你可以在新的 ZFS 启动环境中更新 FreeBSD 系统，而无需接触正在运行的系统。你还可以在 FreeBSD Jail 内进行升级并测试结果，甚至可以将 ZFS 启动环境复制或迁移到另一台机器上。

FreeBSD 提供了 [bectl](https://www.freebsd.org/cgi/man.cgi?query=bectl) 工具，使管理启动环境变得简单。

## BSD init

FreeBSD 使用传统的 BSD 风格 [init](https://www.freebsd.org/cgi/man.cgi?query=init)。在 BSD 风格 init 中，没有运行级别（run-levels），`/etc/inittab` 文件也不存在。相反，启动由 [rc](https://www.freebsd.org/cgi/man.cgi?query=rc) 脚本控制。

`/etc/rc.d/` 中的脚本用于基本系统自带的应用程序，如 `cron`、`sshd`、`syslog` 等。`/usr/local/etc/rc.d/` 中的脚本用于用户安装的第三方应用程序，如 NGINX 或 Postfix。

如前所述，由于 FreeBSD 是作为完整操作系统开发的，用户安装的第三方应用程序不属于基本系统。第三方应用程序通过包或 ports 安装。为了与基本系统保持分离，用户安装的应用程序会安装在 `/usr/local/` 下。因此，用户安装的可执行文件位于 `/usr/local/bin/`，而配置文件位于 `/usr/local/etc/`。

在 BSD init 系统中，启用服务的方法是向 `/etc/rc.conf` 添加服务条目。默认设置位于 `/etc/defaults/rc.conf`，而 `/etc/rc.conf` 中的设置会覆盖默认值。

例如，以下条目在 `/etc/rc.conf` 中启用 `sshd`：

```sh
sshd_enable="YES"
```

你可以手动添加该条目，也可以运行：


```sh
# service sshd enable
```

这将自动编辑 `/etc/rc.conf` 并添加该条目。

你也可以手动启动服务，方法是：

```sh
# service sshd start
```

如果某个服务尚未启用，但你仍然想启动它，可以通过命令行使用以下命令启动：

```sh
# service sshd onestart
```

你能在 [Wikipedia](https://en.wikipedia.org/wiki/Init) 上阅读更多关于 init 系统的内容。

## Jail

FreeBSD 的 [Jail](https://docs.freebsd.org/en/books/handbook/jails/) 系统是另一项了不起的工程成就。

Jails 于 2000 年 3 月 14 日在 FreeBSD 4.0 版本中引入。

FreeBSD jail 是一种 [操作系统级虚拟化](https://en.wikipedia.org/wiki/OS-level_virtualization)，它能让你在多个独立的迷你系统（称为 jail）中安装基于 FreeBSD 的系统。运行在 jail 中的系统共享相同的内核和系统资源，因此开销非常小。

FreeBSD jail 的需求来源于一个小型共享环境托管提供商（R&D Associates, Inc. 的所有者 Derrick T. Woolworth）希望在其自身服务和客户服务之间建立清晰的分离，主要是为了安全性和便于管理。解决方案由 [Poul-Henning Kamp](https://en.wikipedia.org/wiki/Poul-Henning_Kamp) 开发，它不是通过增加新的精细配置选项层，而是将系统（包括文件和资源）进行隔离，使只有正确的人才能访问正确的隔间。

通过 FreeBSD jail，可以创建各种虚拟机，每个虚拟机都有自己安装的工具集和配置。这提供了一种安全的方式来测试软件。例如，可以在不同的 jail 中运行不同版本或尝试不同配置的 Web 服务器包。由于 jail 的范围有限，即使 jail 内的超级用户配置错误或操作失误，也不会危及系统的其他部分的完整性。因为 jail 外部的内容没有被修改，所以可以通过删除 jail 中的目录树副本来丢弃更改。

然而，FreeBSD jail 并不实现真正的虚拟化；它无法让虚拟机运行与基本系统不同版本的内核。

FreeBSD jail 是提高服务器安全性的有效方式，因为它通过隔离 jail 环境与系统其他部分（其他 jail 和基本系统）来实现安全隔离。

### Bastille

[Bastille](https://bastillebsd.org/) 是一款开源系统，用于在 FreeBSD 上自动化部署和管理容器化应用程序。

Bastille 使用 FreeBSD jail 作为容器平台，并添加了模板自动化功能，以创建类似 Docker 的容器化软件集合。

模板负责安装、配置、启用和启动软件，提供了一种自动化构建容器化堆栈的方法。

## Capsicum

Capsicum 是由 [剑桥大学计算机实验室](https://www.cl.cam.ac.uk/) 开发的沙箱框架，由谷歌、FreeBSD 基金会和美国国防高级研究计划局提供资助支持。

Capsicum 扩展了 POSIX API，提供了多个新的操作系统原语，以在类 UNIX 系统上支持对象能力安全：

- Capabilities —— 精细化的文件描述符，带有细粒度权限
- Capability mode —— 进程沙箱，拒绝访问全局命名空间
- Process descriptors —— 面向能力的进程 ID 替代
- Anonymous shared memory objects —— 对 POSIX 共享内存 API 的扩展，支持与文件描述符（能力）关联的匿名交换对象
- `rtld-elf-cap` —— 修改过的 ELF 运行时链接器，用于构建沙箱化应用
- `libcapsicum` —— 创建和使用能力及沙箱化组件的库
- `libuserangel` —— 沙箱化应用或组件与用户天使（如 Power Boxes）交互的库
- `chromium-capsicum` —— Google Chromium 浏览器的一个版本，使用能力模式和能力机制，为高风险网页渲染提供有效沙箱保护

FreeBSD 上的 Capsicum 由 Robert Watson 和 Jonathan Anderson 开发，自 FreeBSD 10.0 起即开箱可用。FreeBSD 实现的 Capsicum 是参考实现，不仅作为 Capsicum API 和语义的参考，还为移植到其他平台的 ports（例如 Linux 和 DragonFlyBSD 的 Capsicum）提供了起点源码。

更多关于 FreeBSD 上 Capsicum 的信息，请参见：

- [FreeBSD 上的 Capsicum](https://www.cl.cam.ac.uk/research/security/capsicum/freebsd.html)
- [Capsicum：UNIX 上的实用能力](https://papers.freebsd.org/2010/rwatson-capsicum.files/rwatson-capsicum-paper.pdf)（PDF）
- [Capsicum 与 Casper：解决安全问题的童话](https://www.youtube.com/watch?v=Kwce64dqY1w)（YouTube）
- [Capsicum 技术](https://wiki.freebsd.org/Capsicum)

## DTrace

[DTrace](https://www.freebsd.org/cgi/man.cgi?query=dtrace) 是一款全面的动态追踪框架，移植自 Solaris。DTrace 提供了强大的基础设施，使管理员、开发人员和服务人员能够简明地回答关于操作系统和用户程序行为的任意问题。

DTrace 可以提供运行系统的全局概览，例如活跃进程使用的内存、CPU 时间、文件系统和网络资源。DTrace 还可以提供细粒度信息，例如某个函数调用时的参数日志，或者访问特定文件的进程列表。

更多 DTrace 使用信息请参见 [DTrace 单行命令教程](https://wiki.freebsd.org/DTrace/Tutorial) 和 [DTrace 示例](https://wiki.freebsd.org/DTrace/Examples)。

## bhyve

[bhyve](https://www.freebsd.org/cgi/man.cgi?query=bhyve) 是 FreeBSD 原生的虚拟机管理程序（hypervisor），用于在虚拟机中运行客户操作系统。可以通过命令行参数指定虚拟 CPU 数量、客户机内存大小以及 I/O 连接等参数。

bhyve 支持多种客户操作系统的虚拟化，包括 FreeBSD 9 及以上版本、OpenBSD、NetBSD、Linux、illumos、DragonFly、Windows Vista 及更高版本，以及 Windows Server 2008 及更高版本。目前的开发工作旨在扩大对 x86-64 架构下其他操作系统的支持。

bhyve hypervisor 自 FreeBSD 10.0-RELEASE 起成为基本系统的一部分。

bhyve 需要处理器支持英特尔扩展页表（EPT）或 AMD 快速虚拟化索引（RVI）或嵌套页表（NPT）。托管 Linux 客户机或具有多个 vCPU 的 FreeBSD 客户机需要支持 VMX 无限制模式（UG）。较新的处理器，尤其是英特尔酷睿 i3/i5/i7 和英特尔至强 E3/E5/E7 系列，支持这些功能。UG 支持从 Intel 的 Westmere 微架构开始引入。

## 防火墙

FreeBSD 强调自由选择，因此基本系统内置了三种不同的防火墙：PF、IPFW 和 IPFILTER（也称为 IPF）。

自 FreeBSD 5.3 起，OpenBSD 的 PF 防火墙的移植版本已作为基本系统的集成部分分发。PF 是款功能完整的防火墙，支持可选的 ALTQ（Alternate Queuing）以实现服务质量（QoS）。PF 的过滤语法类似于 IPF，但经过一些修改以提高清晰度。网络地址转换（NAT）和服务质量（QoS）已集成到 PF 中，其中 QoS 通过导入 ALTQ 排队软件并与 PF 配置关联实现。PF 还扩展了诸如 pfsync 和 CARP（用于故障切换和冗余）、authpf（会话认证）以及 ftp-proxy（简化复杂的 FTP 防火墙设置）等功能。此外，PF 支持对称多处理（SMP）和状态跟踪选项（STO）。

PF 的日志功能也是其创新特性之一。PF 的日志可在 `pf.conf` 中按规则进行配置，由 PF 提供日志，再通过伪网络接口 `pflog` 传送，这是从内核模式获取数据给用户级程序的唯一方式。可以使用标准工具如 `tcpdump` 监控日志。

IPFW 是为 FreeBSD 编写的状态防火墙，支持 IPv4 和 IPv6。它由多个组件构成：内核防火墙规则处理器及其集成的包计数功能、日志功能、NAT、[dummynet](https://www.freebsd.org/cgi/man.cgi?query=dummynet) 流量整形器、转发功能、桥接功能以及 ipstealth 功能。

FreeBSD 在 `/etc/rc.firewall` 中提供了示例规则集，为常见场景定义了几种防火墙类型，以帮助新手生成合适的规则集。IPFW 提供强大的语法，高级用户可以利用它构建满足特定环境安全需求的自定义规则集。

IPF 是款跨平台开源防火墙，已移植到了多个操作系统，如 FreeBSD、NetBSD、OpenBSD 和 Solaris。IPF 是内核防火墙和 NAT 机制，可由用户空间程序进行控制和监控。防火墙规则可以使用 `ipf` 设置或删除，NAT 规则可以使用 `ipnat` 设置或删除，IPF 内核部分的运行时统计可以通过 `ipfstat` 打印，`ipmon` 可将 IPF 的操作记录到系统日志文件中。

IPF 最初使用“最后匹配规则生效”的规则处理逻辑，并且仅使用无状态规则。此后，IPF 增强了选项 quick 和 keep state。

## 调优

FreeBSD 拥有五百多个系统变量，可通过 [sysctl](https://www.freebsd.org/cgi/man.cgi?query=sysctl) 工具读取和设置。这些系统变量可用于对正在运行的 FreeBSD 系统进行修改，包括 TCP/IP 栈和虚拟内存系统的许多高级选项，对于经验丰富的系统管理员来说，可以显著提升性能。

## GEOM

FreeBSD 的 [GEOM](https://docs.freebsd.org/en/books/handbook/geom/) 是 FreeBSD 操作系统的主要存储框架。它提供了访问存储层的标准化方式。GEOM 采用模块化设计，能让 GEOM 模块连接到框架。例如，`geom_mirror` 模块为系统提供 RAID1 或镜像功能。已有许多模块可用，并且各种 FreeBSD 开发者始终在积极开发新的模块。

由于 GEOM 的模块化设计，模块可以堆叠形成 GEOM 层的链。例如，在 `geom_mirror` 模块之上可以添加加密模块，如 `geom_eli`，从而提供镜像且加密的卷。每个模块都有消费者和提供者。提供者是 GEOM 模块的来源，通常是物理硬盘，有时也可以是虚拟化的磁盘，如内存盘。GEOM 模块本身提供一个输出设备，其他 GEOM 模块（称为消费者）可以使用该提供者来创建相互连接的模块链。

## Linux 二进制兼容层

FreeBSD 提供对 Linux 的二进制兼容层。这能让用户在 FreeBSD 系统上安装和运行许多 Linux 二进制文件，而无需先修改这些二进制文件。在某些特定情况下，Linux 二进制文件在 FreeBSD 上的性能甚至可能优于在 Linux 上的表现。

并非所有 Linux 特定的操作系统功能都在 FreeBSD 上受支持。例如，如果 Linux 二进制文件过度使用 i386 特定调用（如启用虚拟 8086 模式），则无法在 FreeBSD 上运行。

## 安全事件审计

FreeBSD 支持安全事件审计。事件审计可以对各种与安全相关的系统事件（包括登录、配置更改以及文件和网络访问）进行可靠、细粒度且可配置的日志记录。这些日志记录对于实时系统监控、入侵检测和事后分析非常有价值。FreeBSD 实现了 Sun 发布的基本安全模块（BSM）应用编程接口（API）和文件格式，并且能够与 Solaris 和 Mac OS X 的审计实现互操作。

## 结语

本文并不是关于使用 FreeBSD 而非 GNU/Linux 的技术原因的详尽列表。还有许多其他原因我没有涉及。然而，这些是我认为最突出的几个特性。

我在第一台 PC 上运行过 Microsoft [MS-DOS](https://en.wikipedia.org/wiki/MS-DOS)，后来又使用了 Microsoft Windows 3.0。从那时起直到 Microsoft Windows XP，我在微软操作系统上做了很多 PC 和服务器相关的工作，但我非常讨厌这些系统。它们非常耗时，因为微软的操作系统既不安全、不稳定，又相当糟糕。

大约在 1997 年，我的一位朋友建议我尝试 Linux，我照做了。我运行的第一个 Linux 发行版是 [Red Hat Linux](https://en.wikipedia.org/wiki/Red_Hat_Linux)。通过我那台旧调制解调器下载它花了一个多星期，期间电话线几乎全天都在占用 :)

在运行 Red Hat Linux 并测试了其他 Linux 发行版几年后，我最终选择了 Debian，主要是因为它当时出色的包管理器 [apt-get](https://en.wikipedia.org/wiki/APT_%28software%29) 以及其稳定性（稳定分支只接收安全和错误修复）。Debian 成为我最喜欢的 Linux 发行版，无论是个人还是工作站和服务器上，我都在使用它。

1998 年，[我发现了 FreeBSD](https://unixdigest.com/articles/freebsd-is-an-amazing-operating-system.html)，从 1999 年左右开始，我个人和工作上都开始使用 FreeBSD，涵盖工作站和服务器。从那时起，我尝试了许多其他操作系统，如今我还使用 [OpenBSD](https://www.openbsd.org/)、[Arch Linux](https://www.archlinux.org/)、[Artix Linux](https://artixlinux.org/)、[Void Linux](https://voidlinux.org/) 和 [Alpine Linux](https://www.alpinelinux.org/)。

我按不同任务使用不同操作系统，并且在特定场景下偏好某些系统。

目前，我在桌面工作站和多台服务器上都以 FreeBSD 作为主要操作系统，除非你有非常特定的需求必须使用 Linux，否则你会发现 FreeBSD 更加贴近真正的 UNIX。如果你需要 ZFS，那么你绝对应该使用 FreeBSD。

无论如何，下次如果你计划尝鲜操作系统，不妨尝试 [FreeBSD](https://www.freebsd.org/)。它也有优秀的社区，许多人乐于助人且友好。
