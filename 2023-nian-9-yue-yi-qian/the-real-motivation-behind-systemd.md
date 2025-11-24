# systemd 背后的真正动机

- 原文：<https://unixsheikh.com/articles/the-real-motivation-behind-systemd.html>
- 最后发布日期：2023-10-28

>在这篇文章中，我们将深入探讨 systemd 开发背后的真实动机，并展望 GNU/Linux 作为操作系统的一些未来前景。

**2022-10-31 更新：** 情况并未改善，这并不令人意外。随着微软在 systemd 开发中扮演主导角色，以及其 [全新的受信启动世界](https://0pointer.net/blog/brave-new-trusted-boot-world.html) 的推进，再加上对大量开源基础设施的掌控，这正在慢慢演变为一种“暗中接管”Linux 世界的局面，而这是无人希望看到的。现在的确是回归 [社区驱动开发](https://en.wikipedia.org/wiki/Community-driven_development) 的时候了。

## 引言

起初，systemd 只是一款新的 init 系统，我个人并不反感。然而，今天我对 systemd 的问题在于，它已经变成了一种类似 [特洛伊木马](https://en.wikipedia.org/wiki/Trojan_horse_%29computing%29) 的存在。它是 [红帽](https://en.wikipedia.org/wiki/Red_Hat)（IBM）试图改变 Linux 世界以更好服务其公司利益的手段。

虽然最初 Linux 内核、GNU 工具以及不同主要独立的 Linux 发行版都是由社区驱动的项目，但如今 Linux 世界的大部分开发都受公司利益驱动，由坐在各大公司关键职位的开发者推动，例如英特尔、微软、红帽、谷歌、Facebook 等。

[Linux 基金会](https://www.linuxfoundation.org/) 目前赞助了 Linus Torvalds 让他全职工作于 Linux 内核，他依旧掌舵，但 Linux 基金会正慢慢变成“微软的影子”，就像 [开源倡议组织（OSI）](https://opensource.org/) 一样，大部分资金来源于微软。

微软开展“we love Linux（我们爱 Linux）”运动后，其一名董事会成员进入 Linux 基金会董事会。随后变为两名，现在是三名。基金会似乎在慢慢被暗中接管，真正的社区成员几乎不复存在，主要由公司员工组成。此外，微软在“Linux 基金会技术顾问委员会”也有重大影响，其现任董事试图将 Linux 外包到微软的 GitHub。

最初作为替代 init 系统，红帽发布了 systemd。随后它突然变成“**提供 Linux 操作系统基础构建模块的软件套件**”。红帽开始推动一场运动，影响其他主要 Linux 发行版，并施压它们让其采用 systemd。

采取的方法是，systemd 开发者针对多个第三方项目尝试说服其依赖 systemd，例如 Lennart Poettering 在 [GNOME 邮件列表](https://mail.gnome.org/archives/desktop-devel-list/2011-May/msg00427.html) 的尝试，以及红帽开发者 "keszybz" 在 [tmux 项目](https://github.com/tmux/tmux/issues/428) 的尝试。大部分尝试表面上看是技术问题，但 GNOME 邮件列表及其他地方的长篇邮件往来表明，事实并非如此。

在这个讨论串中，探讨了 GNOME 是否应该 [仅作为基于 Linux 的操作系统](https://mail.gnome.org/archives/desktop-devel-list/2011-May/msg00439.html)，随后出现了一些 [重要的](http://mail.gnome.org/archives/desktop-devel-list/2011-May/msg00437.html) [反对意见](http://mail.gnome.org/archives/desktop-devel-list/2011-May/msg00432.html)。

红帽采取的其他策略包括从 GNOME 及其他 Linux 发行版（如 Debian）雇佣开发者，让他们推广 systemd。

所有这些导致开源 Linux 社区掀起轩然大波，Debian 开发者 Joey Hess、Debian 技术委员会成员 Russ Allbery 与 Ian Jackson，以及 systemd 软件包维护者 Tollef Fog Heen 纷纷辞职。他们在 Debian 公共邮件列表和个人博客中说明了原因：systemd 在 Debian 及开源社区的整合引发极高压力，使得日常维护几乎不可能。

2014 年 12 月，一位自称“资深 Unix 管理员”的团队宣布 Debian 分支 Devuan，旨在提供一款不含 systemd 的 Debian 衍生系统。Devuan 1.0.0 于 2017 年 5 月 26 日发布。

> 我们认为，这种情况也是 GNOME 项目议程接管 Debian 的长期过程的结果。考虑到这一影响的广泛性以及 Debian 作为通用操作系统和发行版基础系统的重要性，所涉及的是 GNU/Linux 的未来，可能面临所有基础发行版完全同质化和锁定的局面。

Lennart Poettering 最新发明的 [systemd-homed](https://systemd.io/HOME_DIRECTORY/) 被宣传为处理用户主目录的新方式，实际上只是向消除 `/etc` 更进一步，而这是红帽长期以来的目标。

观看 [FOSSDEM 2020 视频](https://www.youtube.com/watch?v=ZYOHvzv2T64)，Poettering 介绍 systemd-homed 时，他从 Linux 是多用户系统的角度批评全盘加密的处理方式，但同时他否认了 systemd-homed 所提出的至少五种挑战为“无关紧要”，理由是笔记本电脑实际上只有一个人使用。

事实上，即使在内核开发中，Linux 世界的主要开发几乎已被大型公司完全接管。它已不再主要是社区驱动开发。Linux 已成为许多企业的巨大利益来源，他们希望尽可能控制开发进程。

下面让我们看一些无可争议的事实。

## 事实 1：systemd 来自红帽

Lennart Poettering 和 Kay Sievers 在 2010 年启动 systemd 项目时，二人均为红帽员工。最初，systemd 只是作为新的 init 系统发布，但它逐渐演变成 Poettering 所描述的“**提供 Linux 操作系统基础构建模块的软件套件**”。这是设计使然，而非偶然。

官方给出的 systemd 开发理由是：

> 他们希望改进表达依赖关系的软件框架，使系统启动期间更多的处理能够并发或并行执行，并减少 shell 的计算开销。

## 事实 2：开发 systemd 的主要原因是红帽在嵌入式设备上的商业利益

红帽的主要业务在嵌入式设备，而 systemd 从设计上关注的主要问题就是嵌入式设备，例如消除 `/etc` 的工作。

在与红帽 CEO Jim Whitehurst 的 [采访](https://linux.slashdot.org/story/17/10/30/0237219/interviews-red-hat-ceo-jim-whitehurst-answers-your-questions) 中，他表示：

> 我们与全球最大的嵌入式供应商合作，尤其是电信和汽车行业，这些行业最关注的是稳定性和可靠性。他们很容易适应 systemd。

Mentor Automotive 发布了 [2015 年活动的演示文稿](https://unixdigest.com/includes/files/agl-systemd-2015.pdf)，其中详细说明了 systemd 为嵌入式汽车市场带来的诸多好处。他们“轻松适应 systemd”的原因，是因为 systemd 是专门为满足这些需求而设计的。

自 2002 年以来，美国军方始终是红帽 [最大的客户](https://unixdigest.com/includes/files/Army-RedHat-Whitepaper-February-2014.pdf)，也是红帽许多决策背后的主要动力来源之一。

2012 年，Lennart Poettering 将 systemd 的许可证从 GPL 改为 LGPL，以更好地适应嵌入式市场，[具体提交记录](https://github.com/systemd/systemd/commit/5430f7f2bc7330f3088b894166bf3524a067e3d8)。

## 事实 3：不，这不是传说，systemd 确实是个庞大的单体

在 2013 年 1 月，Lennart Poettering 在博客文章 [The Biggest Myths](https://0pointer.net/blog/projects/the-biggest-myths.html)（也可下载 [PDF](https://unixdigest.com/includes/files/the-biggest-myths.pdf)）中，反对将 systemd 称为“单体”，这是许多人对它的看法。Lennart 表示：

> 一款包含 69 个独立二进制文件的软件包很难将之称为单体。然而，与以往解决方案不同的是，我们在单个 tar 包中发布了更多组件，并在单一仓库中维护这些组件，采用统一的发布周期。

然而事实上，许多所谓的独立二进制文件的功能脱离其他 systemd 组件是无法工作的。例如，在 systemd-networkd 的手册页中明确指出，如果将选项 `UseDNS` 设置为 `true`，则会使用 DHCP 服务器提供的 DNS 服务器并优先于任何静态配置的服务器。这对应于 `/etc/resolv.conf` 中的 `nameserver` 选项。但手册页未提及的是，这个设置（以及其他多个设置）在没有 systemd-resolved 的情况下将无法工作。systemd 的其他组件耦合得甚至更紧密。

## 事实 4：隐私问题

systemd-resolved 对 Cloudflare、Quad9 和谷歌配置了硬编码的备用 DNS 服务器。即使你关闭了这些服务器，也可能由于 Bug 而被使用（实际上曾经发生过一次）。

## 事实 5：红帽想成为下一个 Microsoft Windows

这是红帽的另一个主要动机，从 Lennart Poettering 在 [FUDCON + GNOME Asia Beijing 2014](https://unixdigest.com/includes/files/gnomeasia2014.pdf) 的演示文稿中可以看出。查看第 15 页，慢慢翻到第 19 页，你会看到项目目标：

- 将 Linux 从一个“碎片化系统”变为具有竞争力的通用操作系统。
- 构建互联网的下一代操作系统。
- 统一不同发行版之间无意义的差异。
- 将创新带回核心操作系统。

结合后续幻灯片显示的红帽目标市场：

- 桌面
- 服务器
- 容器
- 嵌入式
- 移动
- 云
- 集群

大多数 systemd 模块所增加的功能，仅仅是为了让桌面系统（如 GNOME）像 Microsoft Windows 一样运作。

## 事实 6：红帽需要其他主要 Linux 发行版的配合

如果红帽想在长期计划中成功开发“互联网的下一代操作系统”，他们必须影响其他主要 Linux 发行版。原因在于，如果 Debian、Ubuntu、Arch Linux 等主要发行版拒绝 systemd，红帽的计划就无法推进，因为太多第三方项目根本不会关心红帽希望的运作方式。

这一点非常重要，因为许多开源项目开发时都考虑 [POSIX](https://en.wikipedia.org/wiki/POSIX) 兼容性。他们尽力确保项目能在多个类 UNIX 系统上编译和运行。这并不符合红帽的利益。只要你必须考虑其他操作系统，如 [Solaris](https://en.wikipedia.org/wiki/Oracle_Solaris)、[FreeBSD](https://www.freebsd.org/)、[OpenBSD](https://www.openbsd.org/)、[NetBSD](https://www.netbsd.org/)，Linux 在功能上就会被“拖后腿”，无法与 Microsoft Windows 相比。功能如自动挂载和卸载、简单权限提升等。

另一个问题是，如果其他主要 Linux 发行版拒绝 systemd，红帽就更难将 systemd 相关的更改和代码推送到内核中。但当其他主要发行版也采用 systemd 时，这一过程就容易得多。

## 后果

从政治角度来看，systemd 的主要问题之一是其持续开发由公司利益驱动，而非自由开源 Linux 社区的利益。

从技术角度来看，systemd 已经深深渗透到许多开源项目中，逐渐成为硬依赖（忽略其他开源操作系统的存在）。结果是，许多应用程序无法在脱离 systemd 的情况下运行。这不仅使在 FreeBSD 等操作系统上运行这些软件变得非常困难，也严重影响了仍保留“init 自由”的 Linux 发行版——即维护 PID1 的合理方法，尊重可移植性、多样性和自由选择。

因此，systemd 破坏了可移植性，忽视向后兼容性，并替换了现有服务。许多 Linux 发行版为了获得可用的操作系统，放弃了原先的 init 系统而采用 systemd。systemd 拥有逾 130 万行代码，可能带来大量安全问题或 bug。相比之下，[sinit](https://core.suckless.org/sinit/) 仅不到 200 行代码，仅完成一个功能，而且运行良好。

另一个重大问题是前面提到的 systemd-resolved 的硬编码 DNS 服务器。

Lennart Poettering [解释](https://github.com/systemd/systemd/issues/494) 这些硬编码值用于配置文件灾难性损坏或网络缺少 DHCP 的情况（DNS 备用可更改，但需要重新编译）。然而，这是“嵌入式开发者”的角度。如果应用程序中出现 bug 导致这些 DNS 服务器在你已禁用的情况下仍被使用，或者出现 [竞争条件 bug](https://github.com/systemd/systemd/issues/4175)，就可能产生严重的隐私问题。此外，将 Cloudflare、Quad9 和谷歌的 DNS 服务器硬编码在 systemd 代码中非常不妥，因为这些公司不仅以侵犯用户隐私闻名，而且 NSA 曾经渗透 Google 数据中心（由 [斯诺登文件](https://www.washingtonpost.com/world/national-security/nsa-infiltrates-links-to-yahoo-google-data-centers-worldwide-snowden-documents-say/2013/10/30/e51d661e-4166-11e3-8b74-d89d714ca4dd_story.html) 披露）。这样的设置不应为“必须记得手动移除”的默认选项，而应为用户主动选择（opt-in），绝不应默认启用。

通常处理这些问题的方式，以及 Lennart Poettering 极端傲慢的态度，显示出他对用户隐私和开源 Linux 社区利益的完全漠视。

## 最后评论

让我感到惊讶的是，Debian 邮件列表上的最初讨论居然只集中在 SysVinit、Upstart 和 systemd 上。没人认真关注过 [runit](http://smarden.org/runit/) 或 [s6](https://skarnet.org/software/s6/)。这些系统不仅更符合 Unix 哲学，而且更加安全且易于理解。

Casper Ti. Vectors 在 Gentoo 论坛上的帖子，[s6/s6-rc vs systemd, or why you probably do not need systemd](https://forums.gentoo.org/viewtopic-t-1105854.html)，也表明 s6 在很多方面比 systemd 更好、更优。

许多人误以为 systemd 的每个组件都是独立的，但事实并非如此。查看其代码和文档，就会发现这些所谓模块之间存在 [紧密耦合](https://systemd.io/PORTABILITY_AND_STABILITY/#general-portability-of-systemds-apis)。

企业政治、操作手段和操控不应在 Linux 世界存在。虽然允许公司使用开源代码、贡献代码，并用盈利为项目提供资金支持，但绝不应允许像红帽等公司现在在 Linux 世界拥有如此庞大的控制权。

感谢真正独立的社区驱动项目，否则我们就只能面对像 Microsoft Windows 这样的垃圾系统。在这一点上，可以看看以下内容：

- [致 Linux 世界的公开信](https://lkml.org/lkml/2014/8/12/459)（Christopher Barry）。也可见 [这里](https://unixdigest.com/includes/files/open-letter-to-the-linux-world-christopher-barry.txt)（TXT）。
- [OpenBSD 非常棒](https://unixdigest.com/articles/openbsd-is-fantastic.html)
- [FreeBSD 是一个出色的操作系统](https://unixdigest.com/articles/freebsd-is-an-amazing-operating-system.html)
- [选择 FreeBSD 而非 GNU/Linux 的技术理由](https://unixdigest.com/articles/technical-reasons-to-choose-freebsd-over-linux.html)
- [一些优秀的 GNU/Linux 发行版](https://unixdigest.com/articles/some-of-the-great-gnu-linux-distributions.html)
- [Lennart Poettering：BSD 已不再重要](https://linuxfr.org/nodes/86687/comments/1249943)。也可见 [这里](https://unixdigest.com/includes/files/bsd-is-not-relevant-any-more.pdf)（PDF）。
