# 选择 FreeBSD 而非 GNU/Linux 的技术性原因

- 原地址：<https://unixsheikh.com/articles/technical-reasons-to-choose-freebsd-over-linux.html>
- 译者：ykla & ChatGPT
- 最后发布日期：2022-04-08

清晰的分离

与 FreeBSD 一起工作是非常特别的。这是因为它的设计方式，不同组件的组合方式，以及如何处理配置和调优，以及所有这些都是如何完美地整合在一起。

在我自 1998 年以来使用的大多数 GNU/Linux 发行版中，一直到今天，都会有一种"不匹配"的感觉。

举个例子，Debian GNU/Linux 有自己的做事方式，它是特定于该发行版的。Debian 的方式通过使用一套特定的配置管理工具和补丁来使第三方软件符合"Debian 方式"的设置。虽然从某种意义上说，这可以统一 Debian 发行版的做事方式，但这却与上游配置不一致，这使得处理起来非常烦人。特别是当某些事情运行不正常时，或者上游文档中描述的方式与 Debian 上的设置不符时，这就成了一个问题。这种方法的另一个问题是，一些第三方软件，甚至是发行版的核心元素（比如 systemd），无法被强制塑造成"Debian 方式"。结果就是系统中的一些部分按照"Debian 方式"运行，而其他部分则不是。Debian GNU/Linux 已经采用了 systemd，但与此同时，默认的网络部分是 Debian 特有的。有时，你必须禁用和删除 Debian 特定的东西才能让 systemd 特定的东西工作。所有这些都是由于该系统是由许多不匹配的组件组合而成。

在这方面，Arch Linux 与 Debian 相反，因为 Arch Linux 发行版希望第三方软件保持与上游一致，因此除非绝对必要，否则不会做任何更改。这很棒，因为这意味着上游文档与软件是匹配的。然而，虽然这有助于改善系统的整体管理，但事实仍然是，Linux 内核、用户空间工具以及其他所有内容都是由不同的实体开发的。在 FreeBSD 上，许多用户空间工具具有与内核和系统的其他部分紧密集成的运行时选项。例如，top 命令可以显示与管理 ZFS 文件系统相关的信息。在 GNU/Linux 上没有类似的功能。

Ubuntu 则更糟糕。因为它是基于 Debian 的，它在很多方面都使用了 Debian 的工具和设置，但与此同时，还有"Ubuntu 方式"，其中对 Debian 进行了一些改动，然后还添加了一层所谓的用户改进工具层，这有时会导致 Ubuntu 出现不可理解的故障。

在 FreeBSD 上，你会立即注意到你正在处理一个"完整的操作系统"，一个非常完美地组合在一起的系统。内核和基本系统完全与第三方应用程序分开。基本系统配置存放在/etc 目录下，而所有第三方配置都存放在/usr/local/etc 目录下。你可以在 man 页中找到关于配置、调优或设置的所有信息。

你有 rc 实用程序，这是在 init 调用它后控制自动引导过程的命令脚本，还有命令脚本、sysctl 内核管理工具以及所有其他不同的系统配置，所有这些都非常完美地组合在一起并有很好的文档支持。

由于 FreeBSD 是按照一种完整的操作系统和项目的方式进行管理的，而不是像一堆不同的项目被粘在一起形成一个发行版，所以一切都经过深思熟虑，基于多年的经验，并且当事情发生改变时，改变是为了整个社区的利益，而且是根据真实使用案例和行业中出现的问题得到反馈的。

真正理解 FreeBSD 的最好方法之一就是阅读 Michael W. Lucas 的书《Absolute FreeBSD》。他在书中不仅很好地解释和描述了所有的技术问题，还涵盖了重要的历史背景，解释了为什么事物会是现在这个样子。即使你只是对 FreeBSD 感兴趣，我也强烈推荐这本书。

文档

有些人认为文档不是使用某个技术的技术原因，但文档是描述技术的一个重要组成部分。糟糕的文档、过时的文档和缺失的文档应该被视为一个缺陷。

FreeBSD 的文档随系统一起提供，所以你不必在网络上搜索。基本系统的 man 页质量很高，专门为 FreeBSD 编写。你需要的大部分信息都可以在系统的 man 页中找到。

FreeBSD 还有 FreeBSD 手册，涵盖从安装到日常使用 FreeBSD 的所有内容。这本手册也可以在安装过程中本地安装。尽管手册中偶尔会有一些过时的部分，因为这本书是许多人持续合作的结果，但总体上它是更新的，而且写得很好。

安全性

通常不会破坏操作系统本身，而是破坏在操作系统上运行的程序。在某些情况下，被破坏的程序可能会与操作系统进行交互，从而也破坏了操作系统。保护操作系统意味着你努力确保你计算机的资源只被授权的人用

于授权的目的。

FreeBSD 默认设置为性能优越，不像 OpenBSD 那样默认设置为安全。然而，FreeBSD 提供了许多工具和选项，帮助你保护系统免受攻击。

在这篇文章中，我无法提供一个详尽的选项和功能列表，因为关于 FreeBSD 的安全性，可以轻松填满一本书。所以，我强烈推荐阅读 Michael W. Lucas 的书《Absolute FreeBSD》，如果你想更深入地研究 FreeBSD 的一些安全功能。

安全安装时选项

在安装 FreeBSD 时，安装程序提供一组可以启用或禁用的选项。

隐藏其他用户 ID 的进程
隐藏其他组 ID 的进程
隐藏被监禁的进程
隐藏消息缓冲区
禁用进程调试
随机化进程 ID
禁用 syslogd 网络功能
禁用 Sendmail
安全控制台
非可执行栈和栈保护
大多数 FreeBSD 的其他内核级安全设置都可以在 security.bsd sysctl 树中找到，而且每隔几个月都会增加更多的选项。你可以运行命令 sysctl -d security.bsd 来显示你的 FreeBSD 安装中可用的选项。

```
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

漏洞统计

以下是 FreeBSD 和 Linux 的漏洞统计列表。FreeBSD 通常较低的安全问题数量并不一定意味着它比 Linux 更安全，尽管我确实认为它是如此，但这也可能是因为 Linux 受到了更多关注。然而，大多数 Linux 发行版的攻击面比 FreeBSD 要大得多。

| Year     | FreeBSD  | Linux   |
| -------- | -------- | ------- |
| 1999     | 18       | 19      |
| 2000     | 27       | 5       |
| 2001     | 36       | 22      |
| 2002     | 31       | 15      |
| 2003     | 14       | 19      |
| 2004     | 15       | 51      |
| 2005     | 17       | 133     |
| 2006     | 27       | 90      |
| 2007     | 9        | 62      |
| 2008     | 15       | 71      |
| 2009     | 11       | 102     |
| 2010     | 8        | 123     |
| 2011     | 10       | 83      |
| 2012     | 10       | 115     |
| 2013     | 13       | 189     |
| 2014     | 18       | 130     |
| 2015     | 6        | 86      |
| 2016     | 6        | 217     |
| 2017     | 23       | 454     |
| 2018     | 29       | 177     |
| 2019     | 18       | 170     |
| 2020     | 31       | 126     |
| 2021     | 25       | 158     |
| 2022     | 1        | 73      |
| -------- | -------- | ------- |
| Total    | 430      | 2780    |

要了解有关特定漏洞的更多信息，你可以查看 FreeBSD 和 Linux 的 CVE 详情网站。

稳定性

FreeBSD 拥有出色的工程和发布管理实践。FreeBSD 从想法的构思到公开发布经历多个步骤。

当有人有一个新的想法并开发出新的东西时，首先会进行同行技术审查。然后它进入"current"分支进行集成测试，根据复杂性或潜在影响，进入稳定分支的迁移窗口会进行调整。然后它进入"stable"分支进行更广泛的用户测试。通常，这也是所有 beta 测试发生的地方，并与更广泛的社区进行合作。然后它进入发布候选测试，通常会持续进行 3 轮测试，然后变为正式发布。这意味着，只要你了解发布和升级说明，你可以相当有信心它会继续正常运行。

针对软件的补丁会发布来修复任何漏洞和错误。

这通常使得 FreeBSD 成为一个非常稳定的操作系统。

Ports 集合

FreeBSD 的 Ports 集合是一项令人惊叹的工程壮举。NetBSD 的 pkgsrc（软件包源）和 OpenBSD 的 ports 集合都起源于 FreeBSD 的 ports 系统。

虽然 FreeBSD 也有像 Debian Linux 或 Arch Linux 一样的二进制包，由 pkg 包管理器处理，但 FreeBSD 还可以从源代码编译软件，使用用户特定的编译时间配置。Arch Linux Build System 实际上受到 FreeBSD ports 系统的很大启发。但是，使用 FreeBSD ports 系统，你可以在 make 期间选择最相关的编译时间选项，而在 Arch Linux 上必须手动编辑和更改包维护者的 PKGBUILD 脚本（基本上，你应该接受默认设置）。

FreeBSD ports 集合使用 Makefile 自动化编译、安装和卸载软件的过程，通过 make 命令。组成端口的文件包含所有必要的信息，可自动下载、提取、打补丁、配置、编译和安装应用程序，并且在在期望的应用程序端口目录中发出 make install 或 make install clean 等开始命令后，几乎不需要任何（如果有的话）用户干预。如果该端口对其他应用程序或库有依赖关系，则会自动预先安装这些依赖关系。

大多数端口都配置有一组默认选项，这些选项被认为对大多数用户来说是合适的。然而，这是关于端口系统的一大优点，这些配置选项可以在安装前使用 make config 命令进行更改。该命令会打开一个基于文本的界面，允许用户选择所需的选项。

在撰写本文时，集合中有 30,000 多个端口可用。

滚动发布二进制包

对于二进制包，你可以选择两个不同的分支。一个称为"quarterly"，另一个称为"latest"。

"quarterly"是指从每年 1 月、4 月、7 月和 10 月开始，从 HEAD 分支中切出的 Ports 分支，并且是从这些分支产生的二进制包集。

"quarterly"分支为用户提供了更可预测和稳定的端口和包的安装和升级体验。这主要通过仅允许来自上游的非功能更新来实现。"quarterly"分支旨在接收安全修复，但也可能会有版本更新、提交的回退、错误修复和端口的兼容性或框架更改，这取决于上游的情况。

然而，如果你选择"latest"分支，FreeBSD 将成为滚动发布的操作系统，就像 Arch Linux 一样，它将得到最新的第三方软件。

Poudriere 是一个用于创建和测试 FreeBSD 软件包的实用程序。它利用 FreeBSD 的 jail 系统来设置隔离的编译环境。这些 jail 可以用于为任何版本的 FreeBSD 构建二进制包。一旦软件包构建完成，它们的布局与官方镜像完全相同。这些软件包可以被 FreeBSD 的 pkg 二进制软件包管理工具使用。

通过 Poudriere，你可以轻松构建和设置自己的二进制软件包存储库，其中软件包完全按照你的规格和需求构建。

Poudriere 可以处理整个 ports 树的批量构建，特定子集的 ports 树，或者单个端口及其依赖项。它自动构建软件包，生成构建日志文件，提供签名 pkg 存储库，使得在将补丁提交给 FreeBSD bug 跟踪器之前可以测试端口构建，使得可以使用不同选项测试不同的构建。Poudriere 在一个干净的 jail 环境中进行构建，能够使用 zfs 特定功能。这意味着没有污染主机环境，没有残留文件，没有意外删除，没有更改现有配置文件。

Poudriere 的设置和使用非常简单，因为它没有依赖项，并且可以在任何支持的 FreeBSD 版本上运行。

ZFS

与 Linux 上的情况不同，ZFS 文件系统在 FreeBSD 上是一等公民。这不仅意味着可以在 ZFS 上安装根目录，且安装程序原生支持此功能，而且还意味着许多基本系统工具已经紧密集成和构建以支持 ZFS。在 FreeBSD 上运行 ZFS 与在 Linux 上运行 ZFS 非常不同。在 FreeBSD 上，你可以获得更多用于专门调查 ZFS 性能问题或其他相关问题的工具。

ZFS 的一些主要特点包括：

- 设计用于长期存储数据，并具有无限可扩展的数据存储大小，可以达到零数据丢失和高可配置性。
- 对所有数据和元数据进行分层校验，确保整个存储系统在使用时可以进行验证，并确认正确存储，或在损坏时进行修复。校验和存储在块的父块中，而不是存储在块本身。这与许多文件系统不同，其中校验和（如果有）存储在数据中，因此如果数据丢失或损坏，校验和也很可能丢失或不正确。
- 可以存储用户指定数量的数据或元数据副本，或者选择的数据类型，以提高从重要文件和结构的数据损坏中恢复的能力。
- 在某些情况下，可以自动回滚文件系统和数据的最近更改，以避免错误或不一致性。
- 自动（通常）无声自愈数据不一致性和写入失败检测，对于所有数据可以重建的错误。数据可以通过以下所有方法进行重建：存储在每个块的父块中的错误检测和纠正校验和；磁盘上保存的数据（包括校验和）的多个副本；写意图在 SLOG（ZIL）上记录的应该发生但未发生的写入（在断电后）；来自 RAID/RAIDZ 磁盘和卷的奇偶校验数据；来自镜像磁盘和卷的数据副本。
- 本地处理标准 RAID 级别和额外的 ZFS RAID 布局（"RAIDZ"）。RAIDZ 级别仅在所需的磁盘上条带化数据，以提高效率（许多 RAID 系统会在所有设备上进行无差别条带化），并且校验和允许将对不一致或损坏数据的重建最小化到具有缺陷的那些块。
- 本地处理分层存储和缓存设备，通常是一个卷相关的任务。因为 ZFS 还了解文件系统，所以它可以使用与文件相关的知识来通知、整合和优化其分层存储处理，这是单独设备无法做到的。
- 本地处理快照和备份/复制，可以通过集成卷和文件处理来实现高效。相关工具在低层次上提供，并需要外部脚本和软件来进行利用。
- 本地数据压缩和重复删除，尽管后者主要在 RAM 中处理，并且需要大量内存。
- 高效地重建 RAID 数组——RAID 控制器通常必须重建整个磁盘，但 ZFS 可以将磁盘和文件的知识结合起来，将任何重建限制在实际缺失或损坏的数据上，从而大大加快重建速度。
- 不受影响的 RAID 硬件更改，这影响了许多其他系统。在许多系统上，如果自包含的 RAID 硬件（例如 RAID 卡）失败，或者数据被移动到另一个 RAID 系统，文件系统将缺少原始 RAID 硬件上的信息，这些信息需要用于管理 RAID 数组上的数据。这可能导致数据的完全丢失，除非可以获得和使用与"过渡设备"相同的硬件。由于 ZFS 自己管理 RAID，因此 ZFS 池可以迁移到其他硬件，或者可以重新安装操作系统，RAIDZ

结构和数据将被 ZFS 识别，并立即再次被 ZFS 访问。

- 能够识别在缓存中本应找到的数据，但最近已被丢弃的数据，这使得 ZFS 可以在后续使用时重新评估其缓存决策，并实现非常高的缓存命中率（ZFS 缓存命中率通常超过 80%）。
- 可以使用替代缓存策略来处理其他情况下可能导致数据处理延迟的数据。例如，同步写入可能会减慢存储系统的速度，可以通过将其写入快速独立缓存设备（称为 SLOG，有时称为 ZIL – ZFS Intent Log）来转换为异步写入。
- 可高度调整——可以配置许多内部参数以实现最佳功能。
- 可用于高可用性集群和计算，尽管不完全为此设计。

当然，当你在 Linux 上运行 ZFS 时，你也可以获得所有这些功能。然而，有一个很大的区别，因为没有一个 Linux 发行版与 FreeBSD 对 ZFS 的集成程度接近。

启动环境

由于与 ZFS 的紧密集成，FreeBSD 还支持启动环境。使用启动环境，你可以安装多个核心操作系统版本并选择要引导的版本。因此，启动环境是工作系统的可引导克隆或快照。使用启动环境，你可以执行强化的升级或对系统进行更改，你不必担心破坏任何东西，因为你始终可以回滚和丢弃。

这也意味着你可以在新的 ZFS 启动环境内更新 FreeBSD 系统，而无需触及正在运行的系统。你还可以在 FreeBSD Jail 内执行升级并测试结果。甚至可以将 ZFS 启动环境复制或移动到另一台机器上。

FreeBSD 有 bectl 实用程序，可简化管理启动环境。

BSD init

FreeBSD 使用传统的 BSD 风格 init。在 BSD 风格 init 中，没有运行级别，也没有/etc/inittab 文件。相反，启动由 rc 脚本控制。

在/etc/rc.d/中找到的脚本是属于基本系统的应用程序，如 cron、sshd、syslog 等。在/usr/local/etc/rc.d/中的脚本是用户安装的第三方应用程序，如 NGINX 或 Postfix。

如前所述，由于 FreeBSD 是作为完整的操作系统开发的，用户安装的第三方应用程序不是基本系统的一部分。第三方应用程序是使用 Packages 或 Ports 安装的。为了将它们与基本系统分开，用户安装的应用程序被安装在/usr/local/下。因此，用户安装的二进制文件位于/usr/local/bin/，而配置文件位于/usr/local/etc/。

在 BSD init 系统中，通过在/etc/rc.conf 中添加服务条目来启用服务。默认设置位于/etc/defaults/rc.conf 中，这些默认设置会被/etc/rc.conf 中的设置覆盖。

以下在/etc/rc.conf 中的条目启用 sshd：

```
sshd_enable="YES"
```

你可以手动添加该条目，或者可以运行：

```
# service sshd enable
```

这将自动编辑/etc/rc.conf 并添加该条目。

你也可以手动启动服务，方法是：

```

这将自动编辑/etc/rc.conf并添加该条目。

你也可以手动启动服务，方法是：

```

# service sshd start

```

如果某个服务尚未启用，但你仍然希望启动它，可以使用以下命令在命令行中启动：

```

# service sshd onestart

```

容器化环境
FreeBSD jail系统是另一个令人惊叹的工程成果。

FreeBSD jail在2000年3月14日的FreeBSD 4.0版本中首次引入。

FreeBSD jail是一种操作系统级别的虚拟化技术，允许你将基于FreeBSD的系统安装到几个独立的小型系统中，称为jail。在jail中运行的系统共享相同的内核和系统资源，因此开销非常小。

对FreeBSD jail的需求源自一家小型共享环境托管提供商（R＆D Associates, Inc.公司的所有者Derrick T. Woolworth）希望在他们自己的服务和客户的服务之间建立清晰明确的分隔，主要是出于安全和易于管理的考虑。解决方案（由Poul-Henning Kamp开发）不是添加新的细粒度配置选项层，而是将系统分区，包括其文件和资源，以使只有合适的人可以访问正确的区域。

通过jail，可以创建各种虚拟机，每个虚拟机都有自己安装的一组实用程序和自己的配置。这使得它成为尝试软件的安全方法。例如，可以在不同的jail中运行不同版本或尝试不同配置的Web服务器软件包。由于jail仅限于狭窄的范围，因此即使由jail内的超级用户执行的错误配置或错误也不会危及其余系统的完整性。由于实际上没有在jail外部修改任何内容，因此可以通过删除jail的目录树副本来丢弃“更改”。

然而，FreeBSD jail并未实现真正的虚拟化；它不允许虚拟机运行与基本系统不同的内核版本。

由于FreeBSD jail在受限环境和系统其他部分（其他jail和基本系统）之间实现了隔离，因此FreeBSD jail是提高服务器安全性的有效方式。

Bastille
Bastille是用于在FreeBSD上自动化部署和管理容器化应用程序的开源系统。

Bastille使用FreeBSD jail作为容器平台，并添加了模板自动化，用于创建类似Docker的容器化软件集合。

模板负责安装、配置、启用和启动软件，提供了构建容器化堆栈的自动化方法。

Capsicum
Capsicum是在剑桥大学计算机实验室开发的一个沙箱框架，得到了Google、FreeBSD基金会和DARPA的资助。Capsicum扩展了POSIX API，提供了几个新的操作系统原语，以支持UNIX类操作系统上的对象功能安全性：

- Capability权限-具有细粒度权限的精炼文件描述符
- Capability模式-拒绝访问全局命名空间的进程沙箱
- 进程描述符-以能力为中心的进程ID替换
- 匿名共享内存对象-扩展POSIX共享内存API，支持与文件描述符（能力）关联的匿名交换对象
- rtld-elf-cap-修改的ELF运行时链接器，用于构建沙箱应用程序
- libcapsicum-用于创建和使用能力和沙箱组件的库
- libuserangel-允许沙箱应用程序或组件与用户angel（如Power Boxes）交互的库
- chromium-capsicum-谷歌的Chromium Web浏览器的版本，使用能力模式和能力，以提供高风险网页呈现的有效沙箱保护

FreeBSD对Capsicum的实现由Robert Watson和Jonathan Anderson开发，并自FreeBSD 10.0-RELEASE起默认支持。FreeBSD的Capsicum是参考实现，不仅作为Capsicum API和语义的参考，还为其他平台（例如，Linux上的Capsicum和DragonFlyBSD上的Capsicum）的端口提供了起点源代码。

有关FreeBSD上的Capsicum的更多信息，请参见：

- Capsicum for FreeBSD
- Capsicum: practical capabilities for UNIX (PDF)
- Capsicum and Casper: a fairy tale about solving security problems (YouTube)
- Capsicum technologies

DTrace
DTrace是从Solaris移植的全面动态跟踪框架。DTrace提供了强大的基础设施，允许管理员、开发人员和服务人员简明地回答有关操作系统和用户程序行为的任意问题。

DTrace可以提供运行系统的全局概览，例如活动进程使用的内存、CPU时间、文件系统和网络资源量。DTrace还可以提供细粒度的信息，例如正在调用特定函数的参数日志，或者访问特定文件的进程列表。

有关DTrace用法的更多信息，请参见DTrace单行教程和DTrace示例。

Hacker News上也有关于Linux上的DTrace的讨论，包含许多相关评论。

bhyve
bhyve是一个原生的FreeBSD虚拟化管理程序，可以在虚拟机内运行客户操作系统。可以使用命令行参数指定虚拟CPU数量、客户内存量和I/O连接。

bhyve支持虚拟化多个客户操作系统，包括FreeBSD 9+、OpenBSD、NetBSD、Linux、illumos、DragonFly、Windows Vista及更高版本以及Windows Server 2008及更高版本。当前的开发工作旨在扩展对x86-64架构的其他操作系统的支持。

bhyve虚拟化管理程序从FreeBSD 10.0-RELEASE起成为基本系统的一部分


防火墙
FreeBSD注重选择，因此在基本系统中内置了三种不同的防火墙：PF、IPFW和IPFILTER，也称为IPF。

自FreeBSD 5.3版本以来，OpenBSD的PF防火墙的移植版本已作为基本系统的一部分整合进来。PF是一个完整的、功能丰富的防火墙，可选择支持ALTQ（交替排队），提供服务质量（QoS）支持。PF的过滤语法类似于IPF，但进行了一些修改以使其更加清晰。PF集成了网络地址转换（NAT）和服务质量（QoS），通过导入ALTQ排队软件并将其与PF的配置链接来实现。PF还扩展了一些功能，如pfsync和CARP用于故障转移和冗余，authpf用于会话认证，ftp-proxy用于简化复杂的FTP协议的防火墙设置。此外，PF还支持SMP（对称多处理）和STO（有状态跟踪选项）。

PF的许多创新功能之一是其日志记录。PF的日志记录可以在pf.conf中针对每个规则进行配置，并且通过名为pflog的伪网络接口提供PF的日志记录，这是从内核级别模式提取数据到用户级程序的唯一方式。可以使用诸如tcpdump等标准工具监视日志。

IPFW是为FreeBSD编写的有状态防火墙，支持IPv4和IPv6。它包括几个组件：内核防火墙过滤器规则处理器及其集成的数据包计费设施、日志设施、NAT、dummynet流量整形器、前置设施、桥接设施和ipstealth设施。

FreeBSD在/etc/rc.firewall中提供了一个示例规则集，为常见场景定义了几种防火墙类型，以帮助初学者生成适当的规则集。IPFW提供了强大的语法，高级用户可以使用它来制定满足特定环境安全需求的自定义规则集。

IPF是一个跨平台的开源防火墙，已经移植到多个操作系统，包括FreeBSD、NetBSD、OpenBSD和Solaris。IPF是一个内核级防火墙和NAT机制，可以通过用户空间程序进行控制和监视。防火墙规则可以使用ipf进行设置或删除，NAT规则可以使用ipnat进行设置或删除，可以使用ipfstat打印有关IPF内核部分的运行时统计，并可以使用ipmon将IPF操作记录到系统日志文件。

IPF最初使用“最后匹配规则获胜”的规则处理逻辑，并且只使用无状态规则。此后，IPF已被增强以包括快速（quick）和保持状态（keep state）选项。

调优
FreeBSD具有500多个可以使用sysctl实用程序读取和设置的系统变量。这些系统变量可用于对运行中的FreeBSD系统进行更改。这包括许多TCP/IP堆栈和虚拟内存系统的高级选项，可显著提高有经验的系统管理员的性能。

GEOM
FreeBSD GEOM是FreeBSD操作系统的主要存储框架。它提供了访问存储层的标准化方式。GEOM是模块化的，允许GEOM模块连接到框架。例如，geom_mirror模块为系统提供了RAID1或镜像功能。已经提供了许多模块，并且新的模块始终由各种FreeBSD开发人员进行活跃开发。

由于GEOM的模块化设计，可以将模块堆叠在一起形成一系列GEOM层。例如，在geom_mirror模块之上可以添加一个加密模块，例如geom_eli，以提供镜像和加密的卷。每个模块都有消费者和提供者。提供者是GEOM模块的来源，通常是物理硬盘，但有时也是虚拟化磁盘，例如内存磁盘。反过来，geom模块提供输出设备。其他GEOM模块，称为消费者，可以使用此提供者来创建相互连接的模块链。

Linux二进制兼容性
FreeBSD提供与Linux的二进制兼容性。这允许用户在FreeBSD系统上安装和运行许多Linux二进制文件，而无需首先修改二进制文件。在某些特定情况下，Linux二进制文件甚至可能在FreeBSD上比在Linux上运行更好。

并非所有特定于Linux的操作系统功能都在FreeBSD上得到支持。例如，如果Linux二进制文件过度使用i386特定调用，例如启用

虚拟8086模式，则它们将无法在FreeBSD上工作。

安全事件审计
FreeBSD包含对安全事件审计的支持。事件审计支持可靠、细粒度且可配置的记录各种与安全相关的系统事件，包括登录、配置更改以及文件和网络访问。这些日志记录对于实时系统监视、入侵检测和事后分析非常有价值。FreeBSD实现了Sun发布的基本安全模块（BSM）应用程序编程接口（API）和文件格式，并与Solaris和Mac OS X的审计实现相互操作。

最后的说明
本文绝不是使用FreeBSD而不是GNU/Linux的技术原因的详尽列表。还有其他原因我没有涉及。但是，这些是我个人认为最突出的一些特点。

除非你有非常特定的需要使用GNU/Linux，例如特定硬件的支持，否则在运行和管理FreeBSD时，你通常会体验到更强的概览、控制和和谐感。

可能会在FreeBSD上遇到问题的情况是缺乏对硬件的支持，或者特定的第三方应用程序非常针对Linux。后者主要涉及桌面应用程序，而不是服务器应用程序。例如，Mozilla开发Firefox时主要关注Linux、OSX和Windows。这些被称为Tier-1平台。FreeBSD、OpenBSD、NetBSD和Solaris位于支持列表的底部，属于Tier-3平台。Mozilla开发人员并不可靠地使用非Tier-1平台或构建环境。在任何给定的时间，从mozilla-central构建的非Tier-1平台的Firefox可能工作不正确或根本无法构建。Tier-3平台有一个维护者或社区，试图保持平台的正常工作。这些平台不受Mozilla的持续集成过程支持，Mozilla不会定期在这些平台上进行测试。

这意味着在FreeBSD上，维护者必须花费额外的时间来确保应用程序（如Firefox）编译和在FreeBSD上工作。而且，通常当出现问题时，Mozilla开发人员不会像在Linux、Windows或OSX上出现问题时那样关注这些问题。其他Linux中心的第三方应用程序也是如此。

这并不意味着这些应用程序在FreeBSD上不工作，只是偶尔你可能会遇到某种问题。

我个人在服务器和桌面工作站上都运行FreeBSD。我的主要工作站使用FreeBSD并配备i3窗口管理器。我在同一台机器上的另一个硬盘上运行Arch Linux，因为我偶尔需要Linux。从纯桌面使用角度来看，你几乎看不到任何区别，无论是性能方面、简单性方面还是其他任何方面。

很遗憾FreeBSD没有得到与GNU/Linux同样多的关注。在许多情况下，尤其是在生产服务器和商业用途中，公司可以通过运行FreeBSD而不是Linux获得很多好处，而且通常他们之所以运行Linux，只是因为习惯和对FreeBSD的了解不足。

下次你需要部署新系统时，我建议你也研究和测试FreeBSD。通常情况下，这将是值得你花费时间的。
```
