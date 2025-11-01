# 为什么你应该将所有东西从 Linux 迁移到 BSD

- 原文：<https://unixsheikh.com/articles/why-you-should-migrate-everything-from-linux-to-bsd.html> 和 <https://unixsheikh.com/articles/why-you-should-migrate-everything-from-linux-to-bsd-part-2.html>。
- 最后更新时间：2023-10-30

作为操作系统，GNU/Linux 已经变得相当混乱，这主要是因为项目的碎片化、内核中的臃肿代码，但更重要的原因是企业利益的操控。确实存在一些技术上的理由，使得从 GNU/Linux 迁移到 BSD 在某些情况下是合理的，但本文并不是为了讨论这些问题，它更像是对 Linux 现状的“分析”，更确切地说，是一篇带有强烈个人观点的评论文章。

## 引言

过去，我总是倾向于根据技术优劣来选择操作系统和工具。然而，在当今微软、苹果、谷歌等公司频繁侵犯用户隐私并开展有争议活动的背景下，我认为仅凭技术考虑已不足以作为决策依据。

像 [Microsoft Windows 10](https://en.wikipedia.org/wiki/Windows_10#Privacy_and_data_collection)、[Apple MacOS](https://gist.github.com/iosecure/357e724811fe04167332ef54e736670d) 和 [Google Android](https://en.wikipedia.org/wiki/Android_%28operating_system%29#Security_and_privacy) 这样的专有操作系统因其不当行为而臭名昭著。

长期以来，我一直支持开源替代方案，如 GNU/Linux 和 BSD。我不仅相信这些开源方案在许多技术层面上更优，而且始终反对 [典型的 BSD 与 Linux 之争](https://unixdigest.com/articles/the-typical-discussions-about-bsd-vs-linux.html)。我认为不同的开源项目应当互帮互助、合作共赢，终端用户的讨论应当基于技术而非个人偏好。

在可能的情况下，我曾私下或专业地建议他人将其操作系统迁移至开源替代方案。当人们接受我的建议时，我会协助他们将工作站上的 Microsoft Windows 迁移到 BSD 或 Linux，同样在服务器端亦是如此。这一尝试非常成功，我从未遇到过不满意的个人或公司。

然而，GNU/Linux 世界的情况正在发生变化。越来越多的企业试图控制 Linux 作为操作系统的发展方向。由于 GNU/Linux 的结构和组织特点，它不幸地容易受到这些影响。尽管它仍然是开源的，并且远没有专有系统的问题严重，但在内核和 systemd 中，部分“可选择关闭”的功能已被缓慢引入。

你仍然可以选择关闭这些功能，但作为开源爱好者和隐私关注者，也许更好的做法是迁移到一个不必担心“偷偷植入软件”和企业利益的系统，而是由社区驱动的系统（Linux 早期就是如此）。

作为系统管理员，我不希望每次升级系统时都担心会被“惊喜”，也不希望每次使用系统时都必须记住要关闭哪些间谍软件功能。

一些 Linux 发行版不仅因为隐私问题，也因为技术问题，选择使用 [非 systemd 的 init 方案](https://en.wikipedia.org/wiki/Category:Linux_distributions_without_systemd)。但随着内核开发的现状，以及越来越多第三方应用对 systemd 的依赖（即便不必要），问题正在蔓延到操作系统的其他部分，我认为这是一场日益艰难的斗争。

从社区和安全的角度来看，我认为 GNU/Linux 的未来并不乐观。作为可能的替代方案，我建议尽可能将系统迁移到更加理性、稳定的选择，例如某种 BSD 衍生版。

## Linux 是碎片化的

1983 年，Richard Stallman 在 Usenet 上宣布打算开始编写 GNU 项目。到 1987 年 6 月，该项目已经积累并开发了自由开源的软件，包括一个汇编器、一个几乎完成的可移植优化 C 编译器（GCC）、一个编辑器（GNU Emacs），以及各种 Unix 工具，如 `ls`、`grep`、`awk`、`make` 和 `ld`。

1991 年，Linus Torvalds 在 GNU 项目之外开发了 Linux 内核，并于 1992 年 12 月根据 GNU 通用公共许可证第 2 版发布。结合 GNU 项目已经开发的操作系统工具，这就形成了 GNU/Linux 操作系统，更广为人知的名字是“Linux”。

随后，Linux 发行版开始出现。不同的项目将 Linux 内核、GNU 工具和库、额外的第三方软件、文档、X Window 系统、窗口管理器和桌面环境组合到各自的发行版中。不同发行版关注的目标各不相同，有些侧重桌面，有些主要面向服务器，还有一些尝试提供多用途操作系统。

过去，这些不同的组件和项目都是由开源爱好者和社区开发的，编程和开源的热情是推动力。

如今情况已不同。可以参考我另一篇文章 [systemd 背后的真正动机](https://unixdigest.com/articles/the-real-motivation-behind-systemd.html)。

Linus Torvalds 多次明确表示，他并不关心“Linux 世界”发生了什么，他唯一关心的是内核开发。在 2020 年 1 月 6 日，Torvalds 在 [realworldtech.com](https://www.realworldtech.com/forum/?threadid=189711&curpostid=189841) 的“Moderated Discussions”论坛上回答了用户的问题，同时发表了令人震惊的评论，这与一年前的内核维护争议有关，而该争议对 [ZFS on Linux](https://zfsonlinux.org/) 项目产生了重大影响。

在回答用户的实际问题之后，Torvalds 对 ZFS 文件系统发表了一些极为错误且有害的言论。他说：

> 它（ZFS）一直更多的是一个流行词，而不是真正的东西。

通过这句话，Linus Torvalds 就把全球最强大、最流行的文件系统之一超过十五年的开发成果，简化为一个“流行词”。

ZFS 被称为“文件系统的最终之选”。它是一个结合了文件系统和逻辑卷管理器的系统，最初由 Sun Microsystems 设计。ZFS 是一个稳定、快速、安全、面向未来的文件系统。它可扩展，并包含广泛的数据损坏保护，支持大容量存储，最大文件尺寸可达 16 EB（艾字节），最大存储容量可达 256 PQZB（千万亿泽字节），且对文件系统（数据集）或文件数量无限制，支持高效数据压缩、快照和写时复制克隆、持续完整性检查和自动修复、RAID-Z、本地 NFSv4 ACL，并且可以非常精确地配置。

ZFS 的两个主要实现——Oracle 版本和 [OpenZFS](https://en.wikipedia.org/wiki/OpenZFS) 项目版本——非常相似，使 ZFS 在类 UNIX 系统中得到广泛应用。

如维基百科文章所述，OpenZFS 是一个旨在汇集使用 ZFS 文件系统并致力于改进它的个人和公司，推动 ZFS 以开源方式更广泛使用和开发的 umbrella 项目。OpenZFS 汇集了来自 Illumos、Linux、FreeBSD 和 macOS 平台的开发者，以及众多公司。该项目的高级目标包括提升开源 ZFS 实现的质量、实用性和可用性意识，鼓励就改进开源 ZFS 变体的持续努力进行公开沟通，并确保所有 ZFS 发行版的可靠性、功能性和性能一致。

[OpenZFS on Linux](https://github.com/zfsonlinux/zfs/commits/master)，即该项目在 Linux 平台的部分，目前有 345 名活跃贡献者，提交超过 5,600 次，几乎每天都有新的提交。

世界上一些最大的 CDN 和数据存储服务都在 FreeBSD 或 Linux 上运行 ZFS。

在另一个情况下，Linus Torvalds 在 [TFiR: open source and Emerging Tech YouTube 频道](https://www.youtube.com/watch?v=mysM-V5h9z8) 关于桌面 Linux 的采访中又发表了一条惊人言论，称 Linux 仍未准备好用于桌面，也许 [Chrome OS](https://en.wikipedia.org/wiki/Chrome_OS) 才是解决方案。

这些以及 Linus Torvalds 的许多其他言论表明，Linux 作为一个操作系统缺乏明确方向和清晰管理，因为内核开发与 Linux 世界的其他部分隔离进行。

Linus Torvalds 通常也对企业利益的快速影响非常开放，他对安全性的看法也令人担忧。

2009 年，Linus Torvalds 承认内核开发已经失控：

> 我们越来越庞大和臃肿。是的，这是个问题……呃，我很想说我们有计划……我是说，有时有点遗憾，我们肯定不是我十五年前设想的精简、高效内核……内核庞大且臃肿，我们的指令缓存占用令人担忧。我是说，这毫无疑问。每当我们添加新功能，情况只会更糟。

在 2014 年的 LinuxCon 上，他表示，他认为内核臃肿问题已经好些，因为现代 PC 要快得多！

> 在过去二十年里，我们一直在让内核膨胀，但硬件增长得更快。

这种态度非常有问题。

当软件臃肿时，它不仅变得更不安全、更容易出错，而且运行速度也会大幅下降。认为问题会因为硬件更快而消失是一种不成熟的态度。在当今时代，我们需要优化软件以减少功耗，节约能源并限制污染。

在 2007 年的一次“为什么我退出”采访中，内核开发者 Con Kolivas 表示：

> 如果内核开发和 Linux 有一个最大的问题，那就是开发过程完全与普通用户脱节。你知道的，就是那些占 Linux 用户基数 99.9% 的用户。与内核开发者沟通的方式就是 Linux 内核邮件列表。委婉地说，Linux 内核邮件列表（lkml）几乎是最令人害怕的沟通论坛。大多数人完全不敢发邮件到列表，怕因为经验不足、提交不当的 bug、显得愚蠢或其他原因而被批评……我认为大多数内核开发者根本不了解用户空间的问题有多严重。

事实上，Linux 作为操作系统，是由来自完全不同项目的各类应用程序组合而成的。如果你对此一无所知，可以看看如何构建 [Linux From Scratch](https://www.linuxfromscratch.org/lfs/view/stable/)。

另一篇很好地展示了这些问题的文章是 [Linux maintains bugs: The real reason ifconfig on Linux is deprecated](https://blog.farhan.codes/2018/06/25/linux-maintains-bugs-the-real-reason-ifconfig-on-linux-is-deprecated/)，也可作为 [PDF](https://unixdigest.com/includes/files/linux-maintains-bugs-the-real-reason-ifconfig-on-linux-is-deprecated.pdf) 阅读。

所有这些都与 BSD 变体截然不同，即 [FreeBSD](https://www.freebsd.org/)、[OpenBSD](https://www.openbsd.org/)、[NetBSD](https://www.netbsd.org/) 和 [DragonFly BSD](https://www.dragonflybsd.org/)，因为每一个项目都是完全独立的——可以说是“内部开发”。内核、标准 C 库、用户空间工具等都是操作系统基础系统的一部分，而不是从一堆外部来源拼凑而来。

在 2005 年的一次采访中，[OpenBSD 创始人 Theo de Raadt](https://en.wikipedia.org/wiki/Theo_de_Raadt) 作出如下评论：

> 我相信大家现在都知道，Linux 只是一个内核，而 OpenBSD 是一个完整的 Unix 系统：包括内核、设备驱动、库、用户空间、开发环境、文档，以及所有你继续开发所需的工具。也就是说，仅仅从功能完整性来看，它的处理方式完全不同于 Linux 发行版，完全不同。
>
> 当我们发现系统必须进行更改（无论是安全性还是其他原因）时，我们可以将这样的更改从用户空间通过库一直推到内核，从而强制执行更改。我们可以根据需要修改接口。我们可以快速行动。有时更改甚至会破坏之前的可执行文件；但如果需要，我们可以选择做出这样的决定。
>
> 这赋予了我们极大的灵活性，使我们能够快速前进。如果某些设计有误，并且修复依赖的不仅仅是内核，我们可以通过修改正确位置的所有必要部分来解决问题。我们不需要在错误的位置使用 hack 来解决问题。

## Linux 正受到企业利益的强烈影响

Linux 发行版是由不同群体开发的工具集合，这些群体的利益和优先事项往往存在冲突。由于 GNU/Linux 操作系统的这种碎片化结构，整个项目在商业利益的推动下正迅速失控。

即便是最优秀的 GNU/Linux 发行版，如 Debian GNU/Linux 和 Arch Linux，它们仍主要由开源社区驱动，也无法完全免疫这一问题，因为它们不仅依赖高度碎片化的工具，而且部分开发者已被一些大型商业公司雇佣。

在我撰写的文章 [The real motivation behind systemd](https://unixdigest.com/articles/the-real-motivation-behind-systemd.html) 中，我曾写到，开发 systemd 的主要原因是 Red Hat 在嵌入式设备上的财务利益，主要涉及美国军方和汽车工业。最初，systemd 是作为一个新的 init 系统发布的，但它逐渐发展成 Poettering 所描述的“为 Linux 操作系统提供基础构建模块的软件套件”。

在一次对 Red Hat CEO Jim Whitehurst 的 [采访](https://linux.slashdot.org/story/17/10/30/0237219/interviews-red-hat-ceo-jim-whitehurst-answers-your-questions) 中，他表示：

> 我们与全球最大的嵌入式供应商合作，尤其是在电信和汽车行业，这些行业最关注稳定性和可靠性。他们很容易适应 systemd。

我对 systemd 的“init”部分没有异议，但 systemd 已不再只是一个 init 系统，其主要问题在于其持续开发受到企业财务利益驱动，而非开源社区的利益。因此，我认为，像 Debian 和 Arch Linux 这样的主要 Linux 发行版采纳 systemd 是一个很大的错误。由于 systemd 的复杂性，它们几乎陷入了“供应商锁定”。

这纯属推测，但我也必须承认，我怀疑 systemd 可能是向 Linux 操作系统引入安全漏洞的平台。这些漏洞表面上看起来像普通的“编程错误”，但其中一些与 [OpenSSL Heartbleed 漏洞](https://en.wikipedia.org/wiki/Heartbleed) 非常相似。而在开源软件中，利用“编程错误”来制造 [后门和其他问题](https://www.youtube.com/watch?v=fwcl17Q0bpk) 是一个众所周知的策略。systemd 自 2015 年以来就有 [大量公开且已确认的漏洞](https://github.com/systemd/systemd/issues?page=1&q=is%3Aissue+is%3Aopen+label%3A%22bug+%F0%9F%90%9B%22) 仍未修复，但 systemd 开发者并没有修复这些漏洞，而是不断向 systemd 添加更多功能。

另一个对 Linux 世界影响深远的公司是 Google。Google 开发了 Android 和 Chrome OS，这两个都是基于 Linux 内核的操作系统。Chrome OS 来源于 Chromium OS，并使用 Google Chrome 浏览器作为主要用户界面。

Chrome OS 被视为微软的竞争对手，不仅直接与 Microsoft Windows 竞争，还间接影响微软的文字处理和电子表格应用，后者依赖于 Chrome OS 的云计算。这也是 Chrome OS 的核心问题之一：它高度依赖 Google 的云基础设施。

Google 已成为最具争议的 [公司之一](https://en.wikipedia.org/wiki/Google#Criticism_and_controversy)。本质上，Google 是一家广告公司，以操控搜索结果和极端用户跟踪能力闻名，这主要归因于“现代”网站开发者无知地在其网站中添加 Google Analytics。

在 2019 年 8 月的一个 YouTube 视频（由 [Linus Tech Tips](https://www.youtube.com/watch?v=YNp4OqdxHWI) 发布）中，Linus Sebastian 展示了互联网跟踪是如何运作的，以及它如何影响你在搜索商品时所得到的价格。

请注意，该视频最初由 Private Internet Access 赞助，而该公司后来被 Kape Technologies 收购，该公司以通过其软件传播恶意软件和行为不端闻名。**不要使用 Private Internet Access！**

Cloudflare 是另一家影响开发各领域的美国网络基础设施公司。该公司提供的服务实际上位于网站访问者与 Cloudflare 用户的托管服务提供商之间，充当网站的反向代理。因此，Cloudflare 已成为互联网的重大毒瘤之一。

systemd 开发者已将 Cloudflare、Quad9 和 Google 集成到 systemd-resolved 的核心中，现在必须手动禁用（opt-out）。

随着 Red Hat（IBM）通过 systemd 的影响力不断扩大，他们已经成功地将大多数主要 Linux 发行版引导到了与许多 Linux 系统管理员和用户期望背道而驰的方向。

## BSD 才是理想选择

[伯克利软件分发](https://en.wikipedia.org/wiki/BSD_%28operating_system%29)（BSD）最初是基于 Research Unix 的操作系统，由加州大学伯克利分校的计算机系统研究组（CSRG）开发和发布。今天，BSD 通常指它的后代系统，如 [FreeBSD](https://www.freebsd.org/)、[OpenBSD](https://www.openbsd.org/)、[NetBSD](https://www.netbsd.org/) 和 [DragonFly BSD](https://www.dragonflybsd.org/)。这些项目是真正的操作系统，而不仅仅是内核，它们也不是“发行版”。

Linux 发行版，例如 Debian 和 Arch Linux，需要将所有必需的软件整合在一起，才能构建出完整的 Linux 操作系统。它们需要 [Linux 内核](https://www.kernel.org/)、[GNU 工具和库](https://www.gnu.org/gnu/linux-and-gnu.html)、[init 系统](https://en.wikipedia.org/wiki/Init)，以及一定数量的第三方应用程序，才能形成可用的操作系统。

相比之下，BSD 系统既是内核，也是完整的操作系统。例如，FreeBSD 同时提供 FreeBSD 内核和 FreeBSD 操作系统，并作为一个单一项目进行维护。

BSD 不属于任何个人或公司。它由全球技术高超且投入的社区贡献者创建和发布。

企业也会使用和贡献 BSD。公司可以开发自己的 BSD 版本，例如索尼电脑娱乐公司为 PlayStation 3、PlayStation 4 和 PlayStation Vita 游戏主机所做的版本。但因为 BSD 是完整的操作系统，并且每个 BSD 项目都由开源爱好者和社区维护和开发，而非像 Red Hat 这样的公司，所以 BSD 项目是真正独立的。

这种 BSD 组织方式的一个结果是，你不会在操作系统中看到疯狂的 opt-out “间谍软件”设置。

相反，由于 BSD 项目由关心操作系统设计、安全性和隐私的技术娴熟、充满热情的人开发和推动，你会发现即使是通过包管理器安装的第三方软件，也会被修补以移除这些问题，例如 [OpenBSD 在 Firefox 中禁用 DNS over HTTPS](https://undeadly.org/cgi?action=article;sid=20190911113856)。

另一大好处是，BSD 项目周围的社区成员通常经验丰富、乐于助人，并且大多非常友善。

不同的 Linux 发行版，如 Debian 和 Arch Linux，甚至早期的 Red Hat Linux，都是非常优秀的项目。当项目由热情而非利润驱动时，质量往往更高。然而，当项目不再由热情驱动，而是由利润驱动时，其质量往往会下降。这很自然，因为利润驱动与热情驱动的动机截然不同。

微软 Windows 在桌面端的成功，并不是因为人们认为 Windows 是一个优秀的操作系统，没有任何理智且经验丰富的系统管理员或 IT 支持人员会相信这一点，而是因为微软实施了极具攻击性的市场营销策略。

虽然 BSD 项目确实会从公司获得代码和偶尔的资金支持，但它们的推动力是热情，而非利润。这主要意味着决策经过深思熟虑，不会为了利润而在隐私或安全上妥协，这与现今的 Linux 形成鲜明对比。

## 许可问题

GPL 许可对开发者要求更严格，它是一种 [开源反模式](https://www.youtube.com/watch?v=Pm8P4oCIY3g&t=37m05s)，因为它强制要求发布所有修改过的源代码，并阻止其他开源项目被整合。例如，GPLv2 阻止了 [DTrace](https://en.wikipedia.org/wiki/DTrace) 和 [ZFS](https://en.wikipedia.org/wiki/ZFS) 在 Linux 上的集成。

另一方面，BSD 开发者没有这种限制。制造商在开发新设备时可以选择 BSD 作为操作系统，而不是 Linux，这允许他们在需要时保留代码修改，不必公开。

GPL 许可听起来似乎更好，因为为什么我们要允许公司简单地“窃取”我们的开源代码并制作专有产品，而不回馈任何东西呢？但事情并没有那么简单。

更多信息请参见我的文章 [GPL 的一些问题](https://unixdigest.com/articles/some-of-the-problems-with-the-gpl.html)。

## 是时候全面迁移到 BSD

大约在 1998 年，我开始将家里、公司和客户的每台服务器与桌面操作系统从 Microsoft Windows 迁移到 Linux，最初是 Red Hat Linux，后来是 Debian。我这样做是因为我在微软 Windows 服务器和工作站支持上工作了大约十年，发现 Linux 更安全、更稳定、开源，并且在调试和控制上往往更容易。

随后在 1999 年末 [我发现了 FreeBSD](https://unixdigest.com/articles/freebsd-is-an-amazing-operating-system.html)，并最终开始在服务器和工作站上部署 FreeBSD 和 OpenBSD。

那时候，Linux 通常对硬件的支持比 BSD 更好，因此我通常使用 Linux 多于 BSD。硬件昂贵，购买硬件时并不总能基于你想运行的操作系统来选择。这与今天不同，现在 BSD 系统一般对现代硬件有很好的支持。

我仍然喜欢 Linux，但我不想再担心 Lennart Poettering（现为微软员工）接下来会搞出什么垃圾。我也不想担心内核中的各种臃肿功能，例如内核 [强制 DRM 适配](https://patchwork.kernel.org/patch/10084131/)。我基本上不想再担心接下来会出现什么问题。一切都应该是默认合理的选项和决策，而不是 opt-out！

最后，BSD 系统不仅仅是不用担心这些问题，它们本身就是非常优秀的操作系统！

可以看看我其他一些关于 BSD 的文章和教程：

* [FreeBSD 是一个了不起的操作系统](https://unixdigest.com/articles/freebsd-is-an-amazing-operating-system.html)
* [选择 FreeBSD 而非 GNU/Linux 的技术原因](https://unixdigest.com/articles/technical-reasons-to-choose-freebsd-over-linux.html)
* [OpenBSD 很棒](https://unixdigest.com/articles/openbsd-is-fantastic.html)
* [如何设置 FreeBSD 并打造美化桌面 - 第 1 部分 - 基础设置](https://unixdigest.com/tutorials/how-to-setup-freebsd-with-a-riced-desktop-part-1-basic-setup.html)
* [如何设置 FreeBSD 并打造美化桌面 - 第 2 部分 - Xfce](https://unixdigest.com/tutorials/how-to-setup-freebsd-with-a-riced-desktop-part-2-xfce.html)
* [如何设置 FreeBSD 并打造美化桌面 - 第 3 部分 - i3](https://unixdigest.com/tutorials/how-to-setup-freebsd-with-a-riced-desktop-part-3-i3.html)
* [OpenBSD 路由器指南](https://openbsdrouterguide.net/)

## 其他相关链接

* [FreeBSD 提交者 Allan Jude 讨论 FreeBSD 的优势以及他在维护数百万服务器中所扮演的角色](https://www.hostingadvice.com/blog/freebsd-project-under-the-hood/)（自 2016 年与 Allan Jude 的访谈以来，FreeBSD 已经有了显著发展）
* [为什么选择 FreeBSD？](https://www.freebsd.org/advocacy/whyusefreebsd.html)
* [FreeBSD 不是 Linux 发行版](https://www.youtube.com/watch?v=ps67ECyh0sM)（YouTube）
* [The FreeBSD Journal](https://www.freebsdfoundation.org/journal/browser-based-edition/)
* [The OpenBSD Journal](https://undeadly.org/)
* [BSD 家族树](https://svnweb.freebsd.org/base/head/share/misc/bsd-family-tree?view=co)
* [EuroBSDcon YouTube 播放列表](https://www.youtube.com/user/EuroBSDcon/playlists)

>在我发布了这篇文章的[第一部分](https://unixdigest.com/articles/why-you-should-migrate-everything-from-linux-to-bsd.html)后，它被发布到了 Hacker News、Reddit 和 Lobsters 上，有一个叫 “harryruhr” 的人在他的博客上发表了回应，标题为 [“是否应该从 Linux 迁移到 BSD？这取决于情况。”](https://fediverse.blog/~/AllGoodThings/should-you-migrate-from-linux-to-bsd-it-depends)。在这篇回应中，harryruhr 提出了几个我认为需要澄清的错误论点。

## 分裂问题

针对我提出的“Linux 是分裂的”观点，harryruhr 写道：

> 是的，它确实是。但如今 BSD 也同样存在分裂问题。三个“传统”BSD——FreeBSD、NetBSD 和 OpenBSD——在技术和目标上差异很大。还有一些“新”的 BSD 分支，如 Dragonfly、MidnightBSD、HardenedBSD 等。Distrowatch.com 列出了 18 个不同的 BSD “发行版”。作者高度赞扬的 ZFS 文件系统仅在 FreeBSD 及其近亲上可用，并且基于“ZFS on Linux”。它在 NetBSD 或 OpenBSD 上不可用。

这完全不正确。FreeBSD 是最早将 ZFS 从 Sun Microsystems 移植过来的独立操作系统之一。ZFS on Linux 出现得更晚，然后演变为 OpenZFS，如今已经成为所有 [FOSS](https://en.wikipedia.org/wiki/Free_and_open-source_software) ZFS 开发者的重要协作项目。来自 Linux、FreeBSD、NetBSD、Illumos 等平台的开发者现在都在为该项目贡献代码。

Linux 是分裂的，因为内核、GNU 工具、库以及其他所有部分都是完全独立的项目。这些项目之间实际上没有任何联系，但你同时又无法拥有一个完整的 Linux 操作系统而不以某种形式将这些不同的项目“粘合”在一起，这正是各个 Linux 发行版所做的工作。

GNU 项目自 1990 年起就一直在开发自己的内核 [GNU Hurd](https://en.wikipedia.org/wiki/GNU_Hurd)，最初计划替代 Unix 内核。Linux 内核只是为了快速得到一个可用的操作系统而采用的便利方式，因为 Hurd 内核尚未完成。

BSD 完全不存在分裂问题，它们各自都是完整的操作系统和独立项目，都有内核、基础工具等。它们是目标不同的独立项目，虽然共享 BSD 内核的家族树，并偶尔共享部分代码，但除此之外彼此独立。如果 FreeBSD 或 NetBSD 停止开发，OpenBSD 完全不会受到影响，反之亦然。

Dragonfly BSD 也是如此。Matthew Dillon 曾是 1994 到 2003 年间的 FreeBSD 开发者，他在 2003 年分叉 FreeBSD，因为他认为 FreeBSD 采用的线程和对称多处理技术会导致性能和维护问题。由于其他 FreeBSD 开发者不同意，他创建了 DragonflyBSD。但 DragonflyBSD 如今也是一个完全独立的操作系统和项目。

所有这些不同的 BSD 项目仍然是完整且独立的操作系统。它们并不是由不同项目的独立组件拼凑而成的。

至于 MidnightBSD、HardenedBSD 以及类似项目，它们也与分裂无关。这些项目大多基于 FreeBSD，利用 FreeBSD 构建不同的应用程序，或对内核进行补丁修改等。这与分裂没有关系。

如果 BSD 项目像 GNU/Linux 那样分裂，那么 BSD 内核就应该由一个独立项目开发，基础工具则由另一个独立项目开发，只有把这些独立且分裂的部分“粘合”在一起，系统才能工作。

这就是 GNU/Linux 操作系统的分裂特性与 BSD 操作系统之间的根本区别。

## 关于被劫持的问题

针对我提出的“Linux 被劫持了”这一观点，harryruhr 写道：

> 作者说，大型“有争议”的公司在影响 Linux 的发展。这可能是真的，但 FreeBSD 的情况真的更好吗？
>
> FreeBSD 社区在每一个可能的场合都会自豪地说，Netflix 使用 FreeBSD 来提供他们的内容。此外，Netflix 还是 FreeBSD 内核最大的商业贡献者之一。但在 FreeBSD 桌面上，本地播放 Netflix 内容仍然不可能。如果这一点，以及他们传播 DRM 内容，还不能让 Netflix 成为最有争议的公司之一，那我就不知道“有争议”是什么意思了。当然，基于 FreeBSD 的商业和专有产品还有几十个，它们剥夺了用户使用自由软件的利益。

确实，Netflix 是 FreeBSD 最大的商业贡献者之一，但这与 Linux 世界中的“劫持”没有任何关系。Netflix 所做的所有 FreeBSD 改进都会回馈给项目。他们做的所有性能增强都已贡献回 FreeBSD，这对 FreeBSD 非常有益。

但 Netflix 并没有试图影响 FreeBSD 项目，也没有试图“劫持” FreeBSD。他们也没有开始开发一个新的 init 系统，然后再后来声称[它实际上并不是 init 系统](https://unixdigest.com/includes/files/gnomeasia2014.pdf)，而是一个“*永远未完成、永远不完整、但用于跟踪技术进展*”的东西，不断地膨胀和扩展。

Netflix 的一名员工 drewg123 在 Hacker News 上提供了如下[相关信息](https://news.ycombinator.com/item?id=22106424)：

> 在 Netflix，我所在的较大工作组中，至少有七名 FreeBSD 提交者，还有一名核心团队成员（我肯定还忘记了一些人，对此抱歉！）。我们还聘用了许多其他提交者和核心团队成员（按特定合同）。
>
> 与 Yahoo 不同，我们需要对 Netflix 的商业目标负责，即在提高流媒体用户体验的同时，维持或降低内容传输成本。这些目标通常与 FreeBSD 的目标高度一致，我们做出了大量贡献（async sendfile、kTLS、未映射的 mbufs、epoch 稳定、NUMA 工作、RACK TCP、BBR TCP、TCP pacing，以及很多我忘记的工作）。

关于 Netflix 提供的服务及其所谓只能用专有应用播放的 DRM 内容，以及其他基于 FreeBSD 的专有项目，这些都对 FreeBSD 没有影响，也与“劫持”完全无关。这些项目都没有影响 FreeBSD。

所以，可以肯定地说，FreeBSD 的情况要好得多。

## 关于“理智的人在哪里”的问题

针对我提出的“BSD 是理想选择”（我原本标题为“BSD 是理智的人所在之地”，因为 Linux 世界中 systemd 的广泛采用）的观点，harryruhr 写道：

> 作者说：“BSD 项目维护整个操作系统，而不仅仅是内核。”确实，BSD 不只是内核，还有用户空间程序。但 BSD 操作系统自带多少“用户空间”程序完全由 BSD 开发者决定。通常只提供最基本的工具。其余的必须通过 Ports 和包来使用，这与在 Linux 发行版中使用包没有什么不同。例如 FreeBSD 的基础系统甚至没有 Xorg，你必须通过“pkg install xorg”从包安装。在很多情况下，原本集成系统的一部分会从基础系统中移除，变成一个包。

我觉得这个说法有些引导性。

不能把 FreeBSD 需要从第三方项目安装 Xorg 的事实，与 GNU/Linux 操作系统的碎片化现实相比较，这两者毫无关系。

我的文章关注的是 GNU/Linux 操作系统的碎片化特点，与不同 BSD 操作系统相比，而不是基础安装中包含多少第三方应用程序。

harryruhr 接着写道：

> 最“完整”的系统确实是 OpenBSD，它不仅自带 X（Xenocara），还有自己的 MTA（OpenSMTPd）和网页服务器（OpenBSD httpd），这可以使 OpenBSD 基础系统成为基本任务服务器的良好选择。当然，除了 xterm、xcalc 和三种窗口管理器（twm、fvwm、cwm）之外，几乎没有“图形化”程序。如果你想要网页浏览器或功能齐全的邮件程序，就必须从包安装。

OpenBSD 中的 X、OpenSMTPd、httpd 以及其他应用程序，与操作系统本身没有关系。是否将这些集成到基础系统中，并不影响 OpenBSD 即使没有这些部分仍然是一个完整操作系统的事实。

这些应用程序并不会使 OpenBSD 比 FreeBSD “更完整”。它们只是让 OpenBSD 的基础安装中自带更多应用程序而已。

OpenBSD 项目之所以将更多应用程序集成到基础安装中，是因为 OpenBSD 非常重视安全。开发者希望将这些应用程序与基础安装集成，以控制它们的开发方式和运行方式。因此这些应用程序成为 OpenBSD 的集成部分。

FreeBSD 或 NetBSD 也可以在基础安装中提供大量应用程序，但对于这些项目来说，这样做没有意义。

事实是，这些应用程序并不影响操作系统的完整性。另一方面，如果没有内核，或者没有“用户空间”工具，那么就什么都没有了。这就是 GNU/Linux 的现实！

## 关于许可的问题

harryruhr 接着写道：

> 关于“BSD 还是 GPL”的讨论与这些许可证本身一样古老。支持或反对都有合理的论点。就我个人而言，只要是自由软件，我并不在意。认为 GPL 会导致软件被利润驱动而非出于善意的论点，并不令人信服。

我认为，真正关心“自由软件”的人，也就是关心开源软件可以自由分发和开发的人，应当更深入地思考这个问题。三十多年的实践经验显示，当利润驱动的公司依赖 GPL 授权的软件时，如果它们无法掌控软件，就很容易使用各种操控手段。

而使用 BSD 许可证，公司不必通过政治手段或操纵来影响软件开发，它可以直接拿到软件，按自己的意愿使用。

## 关于迁移的问题

最后，harryruhr 针对使用 BSD 而非 Linux 的理由写道：

> 但除了仅使用自由软件的理由外，这些理由应该仅仅是技术性的。过时的论点，比如“BSD 是完整系统而 Linux 仅是内核”或者“Linux 受利润驱动而 BSD 受善意驱动”，实际上已经不再成立。

通常，我完全同意“选择特定操作系统的理由应该仅限于技术原因”这一观点。然而，现实已经不再如此，我提出的论点并非过时，相反，非常切中当前问题。

请参考我的文章：[以政治理由批评技术是否合理？](https://unixdigest.com/articles/is-criticizing-tech-on-political-grounds-valid.html)

微软 Windows 10 可能是某些特定应用或硬件的唯一可用操作系统，但这并不意味着你应该为了使用它而在重大隐私问题上作出如此妥协，并用所谓技术理由为此辩解。存在“过度垃圾”的问题。

随着最近 [Linux 内核被强制加入 DRM](https://patchwork.kernel.org/patch/10084131/)，再加上 Linus Torvalds 多次脱离现实的表态，以及他对 Linux 世界许多重要问题的完全忽视——他显然并不关心企业如何影响内核开发——Linux 内核的未来看起来并不乐观，无论是从隐私角度还是安全角度来看。

除非你每次 Linux 内核发布时都愿意通过打补丁来解决问题，否则你需要一个可行的替代方案。一个内核开发者对项目方向有清晰认知的替代方案——一个不会妥协隐私、安全或其他重要问题的方向。

当然，任何项目中总会存在分歧，但 FreeBSD 开发者之间的分歧，与那些为了利润而试图或多或少“劫持”项目的公司完全不同。

