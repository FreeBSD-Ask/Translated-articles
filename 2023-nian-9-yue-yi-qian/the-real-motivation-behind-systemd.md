# systemd 背后的真正动机

- 原地址：<https://unixsheikh.com/articles/the-real-motivation-behind-systemd.html>
- 译者：ykla & ChatGPT
- 最后发布日期：2022 年 10 月 31 日

在本文中，我们将仔细探讨 systemd 开发背后的真正动机，并探讨 GNU/Linux 作为操作系统的一些未来展望。

Update 2022-10-31：毫不奇怪，情况并没有变得更好。随着微软现在在 systemd 的开发中处于领导地位，以及他们大胆的新的可信启动世界，再加上他们对如此多的开源基础设施的接管，这正慢慢演变成了没有人期望的 Linux 世界中的悄然“革命”！现在是时候回归由社区驱动的开发了。

## 引言

就个人而言，在 systemd 刚开始时，我对它并没有什么看法，当时它只是一个新的 init 系统。然而，我对今天的 systemd 的看法是，它已经变成了一种[特洛伊木马](https://en.wikipedia.org/wiki/Trojan_horse_%28computing%29)。这是 Red Hat 试图改变 Linux 世界以更好地为他们的公司利益服务的一种尝试。

尽管 Linux 内核、GNU 工具和各种主要独立的 Linux 发行版最初都是由社区驱动的项目，但目前 Linux 世界的大部分发展都是由企业利益驱动的，由处于不同公司的不同关键职位的开发人员推动，例如 Red Hat、Google、Facebook 和其他几家公司。

Red Hat 最初通过将 systemd 称为替代性的 init 系统来伪装他们的计划。然后真相被揭露：systemd 变成了“为 Linux 操作系统提供基本构建模块的软件套件。”然后，Red Hat 发起了一场大规模的运动，试图影响所有其他主要的 Linux 发行版，并迫使它们采用 systemd。他们所做的努力和工作似乎让人感觉到相当地绝望。

systemd 开发者联系了几个第三方项目，并试图说服它们依赖 systemd，例如 Lennart Poettering 在 [Gnome 邮件列表](https://mail.gnome.org/archives/desktop-devel-list/2011-May/msg00427.html)上的尝试，以及 Red Hat 开发者“keszybz”对 [tmux](https://github.com/tmux/tmux/issues/428) 项目的尝试。这些尝试大多数都伪装成技术问题，然而当人们阅读了 Gnome 邮件列表和其他地方的长篇邮件往来时，真实的意图就变得十分清晰。

Red Hat 采取的其他战术包括雇佣来自 GNOME 和其他 Linux 发行版（如 Debian）的开发者，然后让这些人推广 systemd。

Lennart Poettering 最新推出的 [systemd-homed](https://systemd.io/HOME_DIRECTORY/) 被称为处理 home 目录的新方法，但实际上只是为了更接近消除 `/etc` 目录，而这正是 Red Hat 长期以来的梦想。

观看 [FOSSDEM 2020](https://www.youtube.com/watch?v=ZYOHvzv2T64) 的视频，Poettering 展示了 systemd-homed，并注意他如何批评从多用户系统角度处理全盘加密的方式，然而同时又拒绝至少五个 systemd-homed 提出的与之相关的挑战，因为，嗯，笔记本电脑实际上只被一个人使用。

事实上，Linux 世界的主要发展，即使在内核方面，也几乎完全被大公司劫持。它的开发不再主要是社区驱动的。对于许多企业来说，Linux 已经成为了巨额利润，他们真切希望尽可能控制发展的方向。

这其中的结果之一是在 Linux 开源社区中引起了巨大的骚动，其中 Debian 开发者 Joey Hess、Debian 技术委员会成员 Russ Allbery 和 Ian Jackson，以及 systemd 软件包维护者 Tollef Fog Heen 辞去了他们的职位。所有这四人都在公开的 Debian 邮件列表和个人博客上用他们在 Debian 和开源社区中持续争论集成 systemd 所受到的非常高的压力来证明他们的决定。

在 2014 年 12 月，一群自称为“资深 Unix 管理员”的人宣布了 Debian 的一个分支，名为 Devuan，旨在提供一个没有 systemd 的 Debian 变体。Devuan 1.0.0 于 2017 年 5 月 26 日发布。

>我们认为，这种情况也是一个较长过程的结果，导致 GNOME 项目议程接管 Debian。考虑到今天这种情况的传播程度以及 Debian 作为一个通用操作系统和基本系统在发行版中的重要性，问题是 GNU/Linux 的未来是否会出现完全同质化和封锁的情况。

让我们来看一些不容置疑的事实。

## 事实 1：systemd 来自 Red Hat

2010 年启动 systemd 项目的 Lennart Poettering 和 Kay Sievers 都是 Red Hat 的员工。最初，systemd 被发布为一个新的 init 系统，但它逐渐发展成为 Poettering 所描述的“为 Linux 操作系统提供基本构建模块的软件套件”。这是有意为之，而非巧合。

官方开发 systemd 的原因被描述为：

>他们想要改进表达依赖关系的软件框架，允许在系统启动过程中更多的处理并发或并行进行，并减少 shell 的计算开销。

## 事实 2：开发 systemd 的主要原因是 Red Hat 在嵌入式设备领域的业务利益

Red Hat 的主要业务是嵌入式设备，systemd 的主要关注点是嵌入式设备，例如去除 `/etc` 目录。

在与 Red Hat [首席执行官 Jim Whitehurst](https://linux.slashdot.org/story/17/10/30/0237219/interviews-red-hat-ceo-jim-whitehurst-answers-your-questions) 的一次采访中，他说：

>我们与世界上最大的嵌入式设备供应商合作，特别是在电信和汽车行业，稳定性和可靠性是他们最关心的问题。他们很容易适应 systemd。

Mentor Automotive 在 [2015 年的一个活动上发布了他们的幻灯片](https://events.static.linuxfound.org/sites/events/files/slides/AGL_systemd_2015.pdf)，在这些幻灯片中，systemd 为嵌入式汽车市场提供的许多好处得到了很好的解释。他们“很容易适应 systemd”的原因是因为 systemd 是专门为满足他们的需求而设计的。

[自 2002 年以来，美国军方一直是 Red Hat 的最大客户](https://unixsheikh.com/includes/files/Army-RedHat-Whitepaper-February-2014.pdf)，他们一直是 Red Hat 许多决策背后的主要动力。

2012 年，Lennart Poettering [将 systemd 的许可证从 GPL 更改为 LGP](https://github.com/systemd/systemd/commit/5430f7f2bc7330f3088b894166bf3524a067e3d8)L，以更好地适应嵌入式市场。

## 事实 3：不，这不是谣言，systemd 确实是一个巨大的单体

在他的博客文章《The Biggest Myths》（最大的谣言）中，Lennart Poettering 在 2013 年 1 月反驳称 systemd 是一个“单体”，这是许多人认为的。Lennart 说：

>一个涉及 69 个单独二进制文件的软件包很难被称为单体。然而，与之前的解决方案不同的是，我们将更多的组件打包在一个单一的压缩包中，并在一个统一的存储库中维护它们，有一个统一的发布周期。

然而，事实是，许多这些所谓的单独二进制文件的功能在没有其他 systemd 组件的情况下根本无法工作。如果我们查看 systemd-networkd 的 man 页面，它清楚地说明，如果将 UseDNS 选项设置为  `true`，则将使用从 DHCP 服务器接收到的 DNS 服务器，并优先于任何静态配置的 DNS 服务器。这对应于 `/etc/resolv.conf` 中的 `nameserver` 参数。但是，man 页面忽略了这个设置（以及多个其他设置）在没有 systemd-resolved 的情况下无法正常工作。systemd 的其他组件更紧密地集成在一起。

## 事实 4：隐私问题

systemd-resolved 在 Cloudflare、Quad9 和 Google 中硬编码了备用 DNS 服务器。即使你关闭了这些选项，一个 Bug 可能会导致这些备用服务器仍然被使用（实际上这在某个时候发生过）。

## 事实 5：Red Hat 想成为下一个 Microsoft Windows

这是 Red Hat 的另一个主要动机，这在 Lennart Poetterings 在 [FUDCON + GNOME.Asia Beijing 2014](http://0pointer.de/public/gnomeasia2014.pdf) 中的幻灯片中有所体现。转到第 15 页，然后缓慢滚动到第 19 页。最终你会看到项目目标：

- 将 Linux 从一堆代码转变为具有竞争力的通用操作系统。
- 构建互联网的下一代操作系统。
- 统一不必要的发行版差异。
- 将创新带回到核心的操作系统。
  
结合下一组展示 Red Hat 希望目标市场的幻灯片：

- 桌面
- 服务器
- 容器
- 嵌入式
- 移动
- 云
- 集群
  
许多不同 systemd 模块提供的附加功能在服务器行业中没有任何好处。它只是为了让像 GNOME 和 KDE 这样的桌面系统功能类似于 Microsoft Windows。

## 事实 6：Red Hat 需要其他主要 Linux 发行版的合作

如果 Red Hat 想在开发“互联网的下一代操作系统”的长期计划中取得成功，他们知道他们需要以某种方式影响其他主要的 Linux 发行版。这是因为如果像 Debian 这样的主要的 Linux 发行版拒绝 systemd，Red Hat 将无法按照他们的计划继续进行，因为太多的第三方项目根本不关心 Red Hat 希望如何工作。这一点非常重要，因为许多开源项目用于开发具有 [POSIX](https://en.wikipedia.org/wiki/POSIX) 兼容性的软件。因此，他们努力确保他们的项目在多个类 Unix 操作系统上编译和工作。这对 Red Hat 来说并不符合利益。只要你还考虑其他操作系统，如 Solaris、FreeBSD、OpenBSD 等，与 Microsoft Windows 中的功能相比，Linux 就会被“拖后腿”。诸如简单的挂载和卸载、简单的特权升级等功能。

Red Hat 的另一个问题是，如果其他主要 GNU/Linux 发行版拒绝了 systemd，他们将很难将与 systemd 相关的更改和代码推入内核。但当其他主要发行版也采用了 systemd 时，这变得容易得多。

## 结论

systemd 的主要问题在于其持续的发展是出于一家公司的经济利益，而不是 Linux 开源社区的利益。

从安全的角度来看，也不能信任 Red Hat，如果美国军方或其他某个三字缩写组织希望 Red Hat 在 systemd 中加入后门，那么这可能在很多年内都不会被察觉，就像 [Heartbleed Bug](https://en.wikipedia.org/wiki/Heartbleed) 一样。

我们已经看到了几个这类可利用漏洞的例子：

- [创建一个名为“0day”的用户，获得 root 权限——但这不被视为漏洞！](https://www.theregister.co.uk/2017/07/05/linux_systemd_grants_root_to_invalid_user_accounts/)
- [存在两年的远程代码执行](https://it.slashdot.org/story/17/07/03/0343258/severe-systemd-bug-allowed-remote-code-execution-for-two-years)
- [Systemd 漏洞允许攻击者通过恶意 DNS 数据包入侵 Linux 机器](https://www.bleepingcomputer.com/news/security/systemd-bug-lets-attackers-hack-linux-boxes-via-malicious-dns-packets/%3CPaste%3E)

这样的漏洞是故意引入代码以伪装成虚假的错误，还是真正的错误，是不可能说清楚的。但有一点非常清楚，Red Hat 对 Linux 社区的最大利益并不关心，他们只关心自己的经济利益。

另一个主要问题是前面提到的 systemd-resolved 中的硬编码 DNS 服务器。

Lennart Poettering [解释](https://github.com/systemd/systemd/issues/494)说，硬编码的值应该在配置文件发生灾难性故障且网络上没有 DHCP 时存在（DNS 回退是可以更改的，但需要重新编译）。然而，这是“嵌入式开发人员”在说话。如果在应用程序中发现了使这些 DNS 服务器运行的 Bug，尽管你已经将它们禁用，或者发现了[竞争问题的 Bug](https://github.com/systemd/systemd/issues/4175)，你可能面临着严重的隐私问题。此外，将 Cloudflare、Quad9 和 Google DNS 服务器硬编码到 systemd 代码中的问题是非常严重的，因为这些公司不仅因侵犯人们的隐私而闻名，而且 NSA（美国国家安全局）以前曾渗透过 Google 的数据中心，这是 [Snowden 文件](https://www.washingtonpost.com/world/national-security/nsa-infiltrates-links-to-yahoo-google-data-centers-worldwide-snowden-documents-say/2013/10/30/e51d661e-4166-11e3-8b74-d89d714ca4dd_story.html)披露的内容。这样的设置不应该是默认选择，而应该是选择加入，并且绝对不应该是默认配置。

通常处理这些问题的方式，以及 Lennart Poettering 的极度傲慢态度，显示出对用户隐私和开源 Linux 社区利益的完全漠视。

## 最后的评论

令我惊讶的是，最初在 Debian 邮件列表上的讨论竟然只涉及 SysVinit、Upstart 和 systemd。没有人认真研究 [runit](http://smarden.org/runit/) 或 [s6](https://skarnet.org/software/s6/)。不仅这些系统更符合 Unix 哲学，而且它们也更安全和易于理解。

Casper Ti. Vectors 在 Gentoo 论坛上的帖子“s6/s6-rc vs systemd, or why you probably do not need systemd”也显示出 [s6 比 systemd 更好](https://forums.gentoo.org/viewtopic-t-1105854.html)，而且在许多方面是更优越的解决方案。

许多人错误地认为每个 systemd 组件都是独立的，但事实并非如此。查看代码和文档，看看这些所谓模块之间的[紧密集成](https://www.freedesktop.org/wiki/Software/systemd/InterfacePortabilityAndStabilityChart/)。

公司的政治、策略和操纵在社区驱动的开源项目中没有立足之地。虽然公司可以允许使用开源代码、贡献代码，并从这些项目的收入中提供财务支持，但他们绝不能拥有 Red Hat 和其他公司现在拥有的如此大的控制权。

感谢真正独立的社区驱动项目，因为如果没有这些项目，我们只剩下类似于 Microsoft Windows 的垃圾。
