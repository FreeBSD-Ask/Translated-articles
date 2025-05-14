# FreeBSD 是一个令人惊叹的操作系统

- 原地址：<https://unixsheikh.com/articles/freebsd-is-an-amazing-operating-system.html>
- 译者：ykla & ChatGPT
- 最后发布日期：2020-01-21

>这是一篇关于我在 FreeBSD 上的一些冒险经历以及为什么我认为它是一个令人惊叹的操作系统的小文章。

**更新于 2020 年 1 月 21 日：** 自从我写了这篇文章后，它在 [Hacker News](https://news.ycombinator.com/item?id=22102372)、[Reddit](https://old.reddit.com/r/freebsd/comments/er5wu0/freebsd_is_an_amazing_operating_system/) 和 [Lobster](https://lobste.rs/s/jedqwr/freebsd_is_amazing_operating_system) 上发布过，也有几个人通过电子邮件给我发来了评论。我已根据评论更新了文章。顺便提一下，我不是 FreeBSD 的开发者，FreeBSD 世界中可能有我完全不了解的事情。我也没有关注过 FreeBSD 的开发者邮件列表。我不是 FreeBSD 的“粉丝”。在过去的二十年里，我在使用 GNU/Linux 方面比 FreeBSD 多得多，主要是由于硬件不兼容（缺乏或有问题的驱动程序）。我同样喜欢 Debian GNU/Linux 和 Arch Linux。然而，我对 GNU/Linux 近来的发展感到担忧。此外，这篇文章并不是让任何人从其他系统转换到 FreeBSD，它只是说明我为什么喜欢 FreeBSD，并推荐对操作系统进行调试的人尝试一下。

我想，那应该是 1999 年末或 2000 年中，有一天我在我最喜欢的书店浏览计算机书籍，发现了 Greg Lehey 于 1999 年写的[《The Complete FreeBSD 第三版》](https://openlibrary.org/books/OL8732144M/The_Complete_FreeBSD)。随书附带了 4 张带有 FreeBSD 3.3 的 CD 光盘。

在 1998 年，我已经熟悉了 GNU/Linux，并且正在逐步将家里和公司的所有服务器和桌面操作系统从 Microsoft Windows 迁移到 GNU/Linux，起初是 Red Hat Linux，后来是 Debian GNU/Linux，后者最终成为我多年来最喜欢的 GNU/Linux 发行版。

当我第一次看到 Greg Lehey 的《The Complete FreeBSD》时，我记得注意到封面上的文字，上面写着“伯克利 Unix 的免费版本”和“非常稳定”，我立刻被吸引住了！这是怎么回事？一个免费的 Unix 操作系统！而且非常稳定？听起来太棒了。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/a60cad03-32c4-4613-adbb-53904e6fc720)

我立刻买下了这本书，这本书成了我长时间以来最喜欢的阅读材料（即使我当时没有从事任何 Unix 相关的工作）。

我很惊讶自己从来没有听说过 FreeBSD，因为它从 1993 年就存在了，但至少由于我使用 GNU/Linux 的经验，我对另一个“类 Unix”的操作系统不会完全陌生，而且事实确实如此。

我在不同的硬件上进行了一些测试安装，我立刻喜欢上了 FreeBSD。FreeBSD 成为我在家中运行的第一个 FTP 服务器。

后来，到了 2000 年，我被我所在国家最大的互联网服务提供商之一雇佣，惊讶地发现整个服务器和网络结构都是在 FreeBSD 上运行的。唯一不运行 FreeBSD 的计算机是办公室里的销售人员和秘书工作的计算机，它们运行 Microsoft Windows。当我询问他们选择操作系统的原因时，系统管理员说了一些类似这样的话：

>知道自己在做什么的人才用 FreeBSD！通信行业的每个人都在用 FreeBSD！

我最终亲身体验了 Greg Lehey 书中提到的 FreeBSD 的“非常稳定”的特性。FreeBSD 真是令人惊叹。它非常高效且极其稳定。在 ISP 托管的每个客户，而且客户有很多，都是由 FreeBSD 提供服务的，它在从旧的 386 PC 到最新的 Pentium 4 机器上运行。FreeBSD 唯一需要重启的情况是在基本系统升级后。在我在那里的时间里，我从未在 FreeBSD 运行的任何地方遇到过问题。

相比之下，GNU/Linux 被视为一个“玩具”系统。它只被一些支持人员用于他们的私人设置。

那时我没有意识到的是，FreeBSD（至今仍然如此）被设计为一个完整的多用途操作系统，可以根据特定的用途进行设置和调整。当我偶尔安装 FreeBSD 时，它并不总是像默认的 Debian GNU/Linux 安装一样表现得那么好。即使是在我家的 FTP 服务器上，FreeBSD 最终被 Debian GNU/Linux 取代，因为 FreeBSD 每隔三天左右就需要重新启动，否则性能会大大降低。另一方面，Debian 运行得非常稳定。

**更新于 2020 年 1 月 21 日：** 有人问 FreeBSD 的具体问题是什么，但我记不清楚细节了。我想那可能是因为 FreeBSD 服务器耗尽了内存，但我不能确定。而且我也没有尝试解决这个问题，因为我太忙了，直接用 Debian 代替了 FreeBSD。而且这都是很久之前的事情了。当然，这是某种错误，我很怀疑问题出在我运行的 FTP 服务器软件上，而且根本与 FreeBSD 无关。

**更新于 2020 年 1 月 21 日：** Hacker News 上的某个评论说：*“在我看来，如果需要手动调整，那就是一个错误，而不是一个特性。（我是作为一个 FreeBSD 开发者而这么说的。）”* 对于这一点，我完全反对。有许多选项是你无法简单地自动调整的，因为用例是如此不同，你需要能够手动设置你需要的特定选项。在运行一个类似 NGINX 的繁忙静态文件服务器上，与在 FreeBSD 上运行一个繁忙的数据库服务器相比，每个设置可能需要根据你的用例进行特定的调整，无论是在文件系统还是内核或其他方面。

在接下来的几年里，GNU/Linux 的硬件支持也变得越来越好，我经常遇到一些愚蠢的硬件在 FreeBSD 上无法工作。当时硬件非常昂贵，而且我没有购买能够在 FreeBSD 上正常工作的硬件的选择。所有这些问题最终使我比以前更多地使用 GNU/Linux。如今，这不再是一个大问题，因为 FreeBSD 对大多数现代硬件都有很好的支持，但与 Linux 相比，GPU 的支持还不够好。FreeBSD 的[维基](https://wiki.freebsd.org/Graphics)提供了一些相关信息。

我依然非常喜欢 FreeBSD，并且最终把它作为我主要的桌面计算机使用了很长一段时间（同时在其他计算机上仍运行着几个 GNU/Linux 发行版）。

我喜欢 FreeBSD 的一些东西包括：

- FreeBSD 是一个完整的操作系统。
- FreeBSD 非常深思熟虑且设计良好。一旦你理解了 FreeBSD 的设置和工作原理，你会惊讶于开发人员已经考虑到了多少细节。
- FreeBSD 将内核和基本系统与第三方软件包分开。 ~~这是 FreeBSD 独有的，其他 BSD 都没有这样做。~~ （**更新于 2020 年 1 月 21 日：** 我在写这段时犯了一个错误。其他 BSD 也是如此。我想表达的是这是 BSD 独有的，而不是 GNU/Linux 发行版的特点。）我一直很喜欢 FreeBSD 这一点，但不幸的是（依我个人的意见），[这从 2016 年开始就在改变](https://lists.freebsd.org/pipermail/freebsd-pkgbase/2016-March/000032.html)。[工作还没有完成](https://wiki.freebsd.org/PkgBase)，但它正在慢慢改变。我个人希望这不必改变。
- 所有第三方应用程序都安装在 `/usr/local/` 目录下，所有第三方应用程序的配置也都安装在 `/usr/local/etc/` 目录下。加上基本系统和第三方应用程序之间的分隔，这使得管理第三方应用程序非常简单，如果你需要更改你的设置，你可以简单地使用 `pkg delete -a` 删除所有已安装的软件包，然后开始安装你需要的软件包。
- FreeBSD 的安装只包括你启用的功能（在安装过程中或手动启用），没有运行你不知道的东西。FreeBSD 也是选择性的，这意味着你必须启用某些东西才能运行和使用它。**更新于 2020 年 1 月 21 日：** 当然，一些非常基本的服务默认会运行，比如 cron，因为这些是基本操作系统维护工具的一部分。cron 会在需要时进行一些基本的日志轮转。
- FreeBSD 的代码被细致地维护，并且有很好的文档。**更新于 2020 年 1 月 21 日：** 有人对此表示异议。也许这需要进一步调查，但我一般是将它与 GNU/Linux 进行比较的。
- FreeBSD 的基本安装中同时包含 [UFS](https://en.wikipedia.org/wiki/Unix_File_System) 和 [ZFS](https://en.wikipedia.org/wiki/ZFS) 文件系统。
- FreeBSD 提供丰富的存储系统 GEOM，允许你使用两台网络机器实现高可用性存储，使用你选择的 RAID 级别，或添加诸如压缩或加密等功能。
- FreeBSD 还有 [geli](https://en.wikipedia.org/wiki/Geli_(software))，它是一个基于块设备的磁盘加密系统，使用 [GEOM](https://en.wikipedia.org/wiki/GEOM) 磁盘框架。
- FreeBSD 的服务处理非常简单。每个服务，无论是基本系统的一部分还是从 Ports 安装的，都带有一个脚本，负责启动和停止它（通常还包括一些其他选项）。默认脚本位于默认目录中，具有默认设置，比如 `/etc/default/rc.conf`，但所有设置都可以通过使用 `/etc/rc.conf` 进行覆盖。如果你想启用 SSHd，只需将 `sshd_enable="YES"` 添加到 `/etc/rc.conf`，SSHd 将在启动时启用，或者你可以使用命令 `service sshd enable` ，这更加简单，效果相同。读取配置文件的 FreeBSD rc 系统理解服务之间的依赖关系，并且可以自动并行启动它们，或者在一个完成后等待启动它所需要的其他事物。你将获得现代配置系统的所有优势，没有复杂的界面。**更新于 2020 年 1 月 21 日：** 有人说“并行性部分是错误的”。我从未使用过这个选项，但在 [FreeBSD 宣传项目](https://www.freebsd.org/advocacy/whyusefreebsd.html)中是这样描述的，文字上说：*“读取此文件的 rc 系统理解服务之间的依赖关系，因此可以自动并行启动它们……”* 如果这是错误的，该网站需要一份错误报告。
- FreeBSD 同时具有 [ports 系统](https://en.wikipedia.org/wiki/FreeBSD_Ports)和 [pkg。](https://en.wikipedia.org/wiki/FreeBSD_Ports#Packages)
- FreeBSD 拥有了令人惊叹的 Jail 系统，使你在一个无法访问系统其余部分的沙盒中运行应用程序或整个系统。在 Docker 甚至还没有被想到的时候，FreeBSD 就已经有了 Jail。**更新于 2020 年 1 月 21 日：** FreeBSD 还有 Bastille 容器管理框架，可以从 ports 和 packages 系统安装。
- FreeBSD 有 Mandatory Access Control（强制访问控制），来自 TrustedBSD 项目，它允许你为所有操作系统资源配置访问控制策略。
- FreeBSD 有 Capsicum，允许开发者实现特权分离，减少受损代码的影响。
- FreeBSD 还有 VuXML 系统，用于发布 Ports 软件中的漏洞，它与 pkg 等工具集成在一起，因此你的每日安全电子邮件将告诉你有关已知的 Ports 软件中的任何漏洞。
- FreeBSD 具有安全事件审计，使用 BSM 标准。

如前所述，FreeBSD 是一个真正多用途的操作系统，具有许多不同的用途，因此它非常灵活和可调整。无论是在台式电脑还是服务器上运行 FreeBSD，它都提供许多可调整选项，使你能够使其性能非常高效。系统默认设置可能不完全符合你的需求，但 FreeBSD 提供了大量文档，指导你如何按照你的需求进行设置，并且有一个非常乐于助人的社区，其中有许多有经验的人处理过许多不同的情况和问题。

我认为了解 FreeBSD 与 GNU/Linux 发行版的区别很重要。FreeBSD 是由既是开发者又是系统管理员的人员制作的操作系统。（**更新于：2020-01-21：** 至少过去是这样，也许现在程度较小）。这意味着 FreeBSD 应该由了解系统工作原理的系统管理员来运行。你不能简单地从类似 Ubuntu、Fedora 或 OpenSUSE 这样的系统跳到 FreeBSD，然后期望得到相同的体验（如果真是这样，我和许多其他人将非常伤心）。

Linux 发行版是由不同的人组织编写的一组工具，往往存在利益和优先级上的冲突。

Linux 发行版需要 Linux 内核、GNU 工具和库，可能还需要额外的第三方软件、文档、X 窗口系统、窗口管理器和桌面环境，然后需要将这些不同的组件组合成最终的发行版。不同的发行版专注于不同的目标，有些侧重于桌面，而其他一些侧重于服务器，还有一些试图提供类似 Ubuntu 的多用途操作系统。

但 FreeBSD 不是这样的。FreeBSD 是一个完整的操作系统，由一组热衷于他们所做工作的人员制作，项目内没有利益冲突。维护内核的人也是维护 C 库、ls、stat 和其他命令以及不同工具的人。FreeBSD 还为与操作系统相关的所有内容提供全面的文档。

**更新日期：2020-01-21：** 如果将 FreeBSD 用作桌面操作系统，它也需要第三方软件、X 窗口系统、窗口管理器和桌面环境。我的观点是 FreeBSD 不是一组由不同人组织编写的工具，其利益和优先级常常存在冲突，就像 Linux 内核、GNU C 库等。

**更新日期：2020-01-21：** 有人指出 FreeBSD 项目中也存在利益冲突，我脑海中浮现的一个例子是 Matthew Dillon 因与其他 FreeBSD 开发人员在性能问题上意见不合，最终将 FreeBSD 复刻为 DragonflyBSD。但我的原始表述可能被误解了。我认为类似 BSD 中的问题与 Linux 内核和 GNU C 库等完全不同的项目之间的冲突不同。在 Linux 领域，严重的冲突可能导致操作系统无法正常工作，而在 BSD 项目中，由核心团队和[投票](https://www.freebsd.org/internal/core-vote.html)来解决争议，OpenBSD 的项目领导者 Theo de Raadt 有最后决定权。如果有人持不同意见，他可以最终接受最终决定，或者他有自由将项目复刻，就像 Matthew 最终做的那样，Theo de Raadt 在 NetBSD 上也是如此。这与 GNU/Linux 的情况非常不同，这就是我的意思。

无论你是 GNU/Linux 用户，已经在不同的发行版之间反复横跳，还是已经找到了自己喜爱的 GNU/Linux 发行版，又或者你是 Microsoft Windows 用户或 MacOS 用户，我都强烈建议你尝试一下 FreeBSD。但在这之前，请花些真正的时间学习 FreeBSD 文档，如果你了解它的工作原理，你将能最大限度地发挥其功能。你可以在 [IRC 频道](https://wiki.freebsd.org/IRC/Channels)、[FreeBSD 邮件列表](https://www.freebsd.org/community/mailinglists.html)或 [FreeBSD 论坛](https://forums.freebsd.org/)上与人交流，那里有很多乐于助人的人。

我还强烈推荐 Michael W. Lucas 的书[《Absolute FreeBSD》](https://nostarch.com/absfreebsd3)，他是一位在 FreeBSD 上工作多年的网络/安全工程师，拥有丰富的高可用系统经验。这本书详细介绍了 FreeBSD 操作系统的许多细节，写得很好，包含许多相关内容。

Michael W. Lucas 与[Allan Jude](http://www.allanjude.com/resume/) 还合著了[《FreeBSD Mastery: ZFS》](https://www.goodreads.com/book/show/25595471-freebsd-mastery)和[《FreeBSD Mastery: Advanced ZFS》](https://www.goodreads.com/book/show/29894975-freebsd-mastery)这两本关于在 FreeBSD 上运行 ZFS 文件系统的书籍。第二本是第一本的续集，介绍了更高级的用例。这两本书都非常宝贵。

FreeBSD 是一个令人惊叹的操作系统！
