# 为什么你应该将所有东西从 Linux 迁移到 BSD

- 原地址：<https://unixsheikh.com/articles/why-you-should-migrate-everything-from-linux-to-bsd.html> 和 <https://unixsheikh.com/articles/why-you-should-migrate-everything-from-linux-to-bsd-part-2.html>。
- 译者：ykla & ChatGPT
- 最后更新时间：2021-02-17

> 作为一个操作系统，GNU/Linux 由于其项目的碎片化性质、内核中软件的臃肿，但主要是由于企业利益的操控而变得一团糟。有[几个技术性原因](https://unixsheikh.com/articles/technical-reasons-to-choose-freebsd-over-linux.html)支持从 GNU/Linux 迁移到 BSD，但本文并不涉及这些，而是对 Linux 领域当前状况的“分析”，更像是一个带有主观观点的愤怒发泄。

## 引言

在过去，我一直喜欢根据技术优劣来选择操作系统和工具。然而，在如今的世界中，像微软、苹果、谷歌等公司以损害用户隐私和进行有争议的活动而闻名，我认为这是错误的做法。

像微软的 [Windows 10](https://en.wikipedia.org/wiki/Windows_10#Privacy_and_data_collection)、[苹果的 MacOS](https://gist.github.com/iosecure/357e724811fe04167332ef54e736670d) 和[谷歌的 Android](https://en.wikipedia.org/wiki/Android_%28operating_system%29#Security_and_privacy) 等专有操作系统因其不端行为而出名，甚至连联想这样的公司也在使用 UEFI 引导注入定制的 Windows 组件，以便系统可以向联想回传信息。

长期以来我一直是开源替代品的支持者，比如 GNU/Linux 和 BSD。不仅如此，我还相信在许多技术领域，开源替代品要好得多。

我一直非常反对[传统的 BSD 与 Linux 之争](https://www.unixsheikh.com/articles/the-typical-discussions-about-bsd-vs-linux.html)，正如我以前在文章中写的那样，我一直认为不同的开源项目可以互相帮助和合作，而终端用户应该从技术角度而不是个人偏好来讨论这些问题。

每当有可能时，我一直建议人们（无论是个人还是企业）将使用的操作系统改为开源替代品，并且当人们对我的宣传持接纳态度时，我会帮助他们从他们的工作站迁移到 BSD 或 Linux。在服务器端同样也是如此。这是真正成功的努力，我诚实地从未遇到过不满意的个人或公司。

然而，随着越来越多的公司希望控制 Linux 作为操作系统的发展方向，GNU/Linux 世界的情况开始发生变化。由于 GNU/Linux 作为操作系统的结构和组织，它不幸受到这些影响，虽然它仍然是开源的，并且仍然远未达到专有替代品所发生的那些糟糕情况，但一些选择性功能已慢慢被引入到内核和 systemd 中。

你仍然可以选择退出这些功能并选择其他方式，但作为一个开源爱好者和支持者，以及一个关心隐私的个人，也许更好的方法是将系统迁移到一个你不必担心“骚扰软件”的地方。

作为一个系统管理员，我不想担心下次升级系统时是否会受到意外，我也不想记下一张间谍软件列表，以便我在运行这些系统时记得选择退出。

一些 Linux 发行版已经决定（不仅因为隐私选择性功能问题，还有其他问题）实现[其他 init 解决方案](https://en.wikipedia.org/wiki/Category:Linux_distributions_without_systemd)，而随着内核开发中的情况以及许多第三方应用对 systemd 的依赖越来越深，问题正在向操作系统的其他部分扩散，我认为这是一场越来越艰难的战斗。

从社区和安全的角度来看，我不认为 GNU/Linux 的未来看起来那么光明，因此作为一个可选的替代方案，我建议在可能的情况下将一切迁移到更加合理的东西，比如——BSD 项目。

## Linux 变得分裂

在 1983 年，Richard Stallman 在一条 Usenet 消息中宣布了开始编写 GNU 项目的意图。到了 1987 年 6 月，该项目已经积累并开发了自由开源软件，包括汇编器、一个几乎完成的便携式优化 C 编译器（GCC）、编辑器（GNU Emacs）以及各种 Unix 实用工具，如 ls、grep、awk、make 和 ld。

在 1991 年，Linux 内核出现了，由 Linus Torvalds 在 GNU 项目之外开发，到 1992 年 12 月，它以 GNU 通用公共许可证第 2 版的形式发布。结合 GNU 项目已经开发的操作系统实用工具，它成为了 GNU/Linux 操作系统，更为人所知的名字就是“Linux”。

然后出现了 Linux 发行版。不同的项目将 Linux 内核、GNU 工具和库、额外的第三方软件、文档、X Window 系统、窗口管理器和桌面环境等组件结合到发行版中。不同的发行版关注不同的目标，有些关注桌面，而其他一些则主要关注服务器，还有一些试图提供多用途的操作系统。

在过去，所有这些不同的组件和项目都是由开源爱好者和社区开发的，对编程和开源的热情是推动力。

但现在情况已经不同了！请参阅[《systemd 背后的真正动机》](https://unixsheikh.com/articles/the-real-motivation-behind-systemd.html)（The real motivation behind systemd.）。

Linus Torvalds 多次明确表示，他不关心“Linux 世界”（用户空间）中发生的事情，他只关心内核开发。在 2020 年 1 月 6 日的“Moderated Discussions”[论坛上](https://www.realworldtech.com/forum/?threadid=189711&curpostid=189841)，Linus Torvalds 在回答一个用户关于一年前的内核维护争议对 [ZFS on Linux](https://zfsonlinux.org/) 项目产生了巨大影响的问题时，作出了一个令人震惊的评论。

在回答用户的实际问题后，Torvalds 对 ZFS 文件系统进行了非常错误和有害的指责。Torvalds 说：

> 它（ZFS 文件系统）一直更像是个噱头而非其他什么东西。

通过这样的声明，Linus Torvalds 将全球最稳健和流行的文件系统之一，经过 15 多年的发展，简单地贬损为一个“噱头”！

ZFS 被誉为“文件系统的终极选择”。它是由 Sun Microsystems 最初设计的一种综合文件系统和逻辑卷管理器。ZFS 是一个稳定、快速、安全和未来可靠的文件系统。它具有可扩展性，并包含对数据损坏的广泛保护，支持高存储容量，最大 16 Exabyte 文件大小，最大 256 Quadrillion Zettabytes 存储空间，文件系统（数据集）或文件数量没有限制，高效的数据压缩，快照和写时复制克隆，连续的完整性检查和自动修复，RAID-Z，本地 NFSv4 ACLs 等，而且可以非常精确地配置。

由 Oracle 和 [OpenZFS](https://en.wikipedia.org/wiki/OpenZFS) 项目推出的两种主要实现非常相似，使得 ZFS 在类 Unix 系统中得到广泛应用。

正如维基百科文章所述，OpenZFS 是一个旨在汇集使用 ZFS 文件系统并致力于改进其性能的个人和公司的伞形项目，同时也旨在以开源方式更广泛地使用和开发 ZFS。OpenZFS 汇集了来自 illumos、Linux、FreeBSD 和 macOS 平台的开发人员，以及许多公司。该项目的高级目标包括提高人们对开源 ZFS 实现的质量、实用性和可用性的认识，鼓励就改进开源 ZFS 变体的持续努力进行开放式交流，确保所有 ZFS 发行版的可靠性、功能和性能的一致性。

[OpenZFS on Linux](https://github.com/zfsonlinux/zfs/commits/master) 是该项目中针对 Linux 的部分，目前有 345 个活跃贡献者，超过 5,600 次提交，几乎每天都有新的提交！

一些世界上最大的 CDN 和数据存储服务在 FreeBSD 或 Linux 上运行 ZFS！

在另一种情况下，Linus Torvalds 在 [TFiR:开源和新兴技术的 YouTube 频道](https://www.youtube.com/watch?v=mysM-V5h9z8)上接受了关于 Linux 桌面的采访，在采访中他做出了另一个惊人的声明，认为 Linux 仍未准备好成为桌面操作系统，也许 [Chrome OS](https://en.wikipedia.org/wiki/Chrome_OS) 是解决这个问题的方法。

这些以及其他许多 Linus Torvalds 的声明显示 Linux 作为一个操作系统没有真正的方向和清晰的管理，因为内核开发是独立于 Linux 世界的。

Linus Torvalds 通常也非常乐于受到企业利益的迅速影响，他对安全性的观点也令人担忧。

在 2009 年，Linus Torvalds 承认内核开发变得失控。

> 我们变得庞大和庞大。是的，这是一个问题……呃，我很想说我们有一个计划……我的意思是，有时候有点可悲，我们绝对不是我 15 年前设想的那种精简、小型、超高效的内核内核非常庞大和庞大，我们的指令高速缓存占用了可怕的空间。我是说，这毫无疑问。而且每当我们添加一个新功能，情况只会变得更糟。

在 2014 年的 LinuxCon 上，他表示认为扩展问题变得更好了，因为现代 PC 速度更快了！

> 在过去的 20 年中，我们一直在扩大内核，但硬件增长更快。

这是一个非常有问题的态度。

当软件变得臃肿时，它不仅变得更不安全、更容易出错，而且变得更慢。认为问题会随着硬件速度变快而消失是一种不成熟的态度。在今天，我们需要优化软件以节省功耗，节约电力并减少污染。

在 2007 年的一次采访《为什么我退出》中，内核开发者 Con Kolivas 表示：

> 如果有一个与内核开发和 Linux 相关的大问题，那就是开发过程与普通用户完全脱节。你知道，占据 Linux 用户群 99.9% 的人。Linux 内核邮件列表是与内核开发者沟通的途径。委婉地说，Linux 内核邮件列表（lkml）是最可怕的交流论坛之一。大多数人对向该列表发送邮件感到非常害怕，因为他们担心因为经验不足、不恰当的错误报告、愚蠢或其他原因而受到责备……我认为大多数内核开发者并不真正了解用户空间中存在的问题有多大。

除了上述问题之外，事实上，Linux 作为一个操作系统是由不同项目的不同应用程序组合而成的，这些项目彼此之间没有任何关联。如果你对此一无所知，可以看看如何构建 [Linux From Scratch](http://www.linuxfromscratch.org/lfs/view/stable/)。

另一篇可以展示其中一些问题的好文章是[《Linux 维护漏洞：ifconfig 在 Linux 上被弃用的真正原因》](https://web.archive.org/web/20200813101037/https://blog.farhan.codes/2018/06/25/linux-maintains-bugs-the-real-reason-ifconfig-on-linux-is-deprecated/)。

这与 BSD（指 FreeBSD、OpenBSD、NetBSD 和 DragonFly BSD）完全不同，因为每个 BSD 都是独立的项目，将其系统“内部”组合在一起。内核、标准 C 库、用户空间工具等都是操作系统基本系统的一部分，而不是从一堆不同的外部来源组合而成的。

在 2005 年的一次采访中，OpenBSD 的创始人 Theo de Raadt 发表了以下言论：

> 我相信现在大家都知道 Linux 只是一个内核，而 OpenBSD 是一个完整的 Unix 系统：包括内核、设备驱动程序、库、用户空间、开发环境、文档以及继续开发所需的所有工具。尽管如此，就功能的完整性而言，OpenBSD 并不像一个 Linux 发行版，完全不一样。
>
> 当我们发现系统需要进行更改（无论是出于安全性还是其他原因），我们可以通过从用户空间、库直到内核的所有部分进行更改来强制进行这种更改。我们可以随意更改接口。我们可以快速行动。有时候，甚至会对之前的可执行文件进行改变；但如果需要的话，我们可以选择做出这样的决定。
>
> 这使我们能够快速前进，拥有极大的灵活性。如果某些东西设计错误，而修复取决于不仅仅内核的更改，我们可以通过改变所有必需的部件的正确位置来解决它。我们不需要在错误的地方进行修改来解决问题。

## Linux 受到企业利益的严重影响

Linux 发行版是由不同团体编写的工具的集合，往往具有冲突的利益和优先级，因此由于 GNU/Linux 操作系统的碎片化结构，整个项目正在快速失去控制，受到商业利益的驱动。

即使是最好的 GNU/Linux 发行版，如 Debian GNU/Linux 和 Arch Linux，它们仍然主要由开源社区推动，也不免遇到这个问题，因为它们不仅仍然严重依赖于碎片化的工具，而且一些开发人员被一些主要的商业公司雇佣了。

在我的文章[《systemd 背后的真正动机》](https://unixsheikh.com/articles/the-real-motivation-behind-systemd.html)中，我写到了开发 systemd 的主要原因是 Red Hat 在嵌入式设备（主要是美国军方和汽车行业）的财务利益。最初 systemd 是作为一个新的 init 系统发布的，但它已经逐渐发展成为 Poettering 所描述的“为 Linux 操作系统提供基本构建块的一套软件”。

在一次与 Red Hat CEO Jim Whitehurst 的[采访](https://linux.slashdot.org/story/17/10/30/0237219/interviews-red-hat-ceo-jim-whitehurst-answers-your-questions)中，他说：

> 我们与世界上最大的嵌入式供应商合作，特别是在电信和汽车行业，稳定性和可靠性是他们最关心的问题。他们很容易适应 systemd。

我并不反对 systemd 的“init”部分，但 systemd 不再只是一个 init 系统，其主要问题在于其持续的发展是受到公司的财务利益而不是开源社区的利益驱动的。因此，我认为将 systemd 引入主要的 Linux 发行版，比如 Debian GNU/Linux 和 Arch Linux，是一个巨大的错误。他们已经严重依赖于 systemd 和 Red Hat。

这只是纯粹的猜测，但我必须承认我怀疑 systemd 是引入 Linux 操作系统安全漏洞的平台。这些漏洞当然看起来像是正常的“程序错误”，然而其中一些漏洞与 [OpenSSL Heartbleed 漏洞](https://en.wikipedia.org/wiki/Heartbleed)非常相似。在开源社区中，这是一个众所周知的策略，即利用“程序错误”来创建[后门和其他东西](https://www.youtube.com/watch?v=fwcl17Q0bpk)。systemd 有一系列长期存在且公开的漏洞（截至撰写本文有 1400 多个未解决的漏洞），自 2015 年以来仍未修复，然而 systemd 开发人员继续在其中添加越来越多的混乱！

另一个对 Linux 世界产生巨大影响的公司是谷歌。谷歌开发了基于 Linux 内核的 Android 和 Chrome OS 两个操作系统。Chrome OS 是从 Chromium OS 衍生出来的，其主要用户界面是谷歌的 Chrome 网络浏览器。

Chrome OS 被视为与微软竞争对手，既直接竞争微软的 Windows 操作系统，又间接竞争其文字处理和电子表格应用程序，后者通过 Chrome OS 对云计算的依赖来实现。而这就是 Chrome OS 的一个核心问题，它在很大程度上依赖于谷歌的云基础设施。

谷歌已成为[最具争议的公司](https://en.wikipedia.org/wiki/Google#Criticism_and_controversy)之一。谷歌本质上是一家广告公司，并因其操纵搜索结果和极端的用户跟踪能力而闻名，主要归因于网站开发人员添加 Google Analytics 等工具。

在 [Linus Tech Tips](https://www.youtube.com/watch?v=YNp4OqdxHWI) 于 2019 年 8 月发布的一段 YouTube 视频中，Linus Sebastian 演示了互联网上的跟踪方式以及它如何影响你搜索产品时得到的价格。**请注意：** 该视频由 Private Internet Access 赞助，而该公司后来被 Kape Technologies 收购，该公司因通过其软件发送恶意软件而闻名，并在一般情况下很糟糕。**请勿使用 Private Internet Access！**

Cloudflare 是另一家影响不同领域开发的美国网络基础设施和网站公司。该公司为网站访问者和 Cloudflare 用户的托管提供商之间提供服务，充当网站的反向代理。因此，Cloudflare 已成为互联网上最大的问题之一。

systemd 开发者已将 Cloudflare、Quad9 和 Google 集成到 systemd-resolved 的核心中，现在你必须手动禁用（选择退出）这些功能。

随着 Red Hat（IBM）通过 systemd 的不断影响，他们成功引导了大多数主要 Linux 发行版朝着与许多系统管理员和用户希望看到的方向相违背的方向发展。

## BSD 才是最佳选择

与 Linux 发行版相反，[伯克利软件分发（BSD）](<https://en.wikipedia.org/wiki/BSD_(operating_system)>)不是一个分裂的项目。BSD 项目维护整个操作系统，而不仅仅是内核。

BSD 是基于 Research Unix 开发和分发的操作系统，由加利福尼亚大学伯克利分校的计算机系统研究组（CSRG）开发。如今，“BSD”指的是其后代，如 [FreeBSD](https://www.freebsd.org/)、[OpenBSD](https://www.openbsd.org/)、[NetBSD](https://www.netbsd.org/) 和 [DragonFly BSD](https://www.dragonflybsd.org/)。这些项目是真正的操作系统，而不仅仅是内核，它们并非“发行版”。

Linux 发行版，如 Debian GNU/Linux 和 Arch Linux，必须做的工作是将所有所需的软件汇集在一起，以创建一个完整的 Linux 操作系统。它们需要 [Linux 内核](https://www.kernel.org/)、[GNU 工具和库](https://www.gnu.org/gnu/linux-and-gnu.html)、[init 系统](https://en.wikipedia.org/wiki/Init)以及一些第三方应用程序，以便得到一个运行的操作系统。

相比之下，BSD 既是一个内核又是一个完整的操作系统。例如，FreeBSD 提供了 FreeBSD 内核和 FreeBSD 操作系统。它作为单一项目进行维护。

没有个人或公司拥有 BSD。它由遍布全球的高度技术和忠诚的贡献者社区共同创建和分发。

公司也[使用和为 BSD 做贡献](https://en.wikipedia.org/wiki/List_of_products_based_on_FreeBSD)，但与 Linux 不同，公司不能“劫持”BSD。公司可以制作自己的 BSD 版本，例如索尼电脑娱乐公司为 PlayStation 3、PlayStation 4 和 PlayStation Vita 游戏机制作的版本，但由于 BSD 是完整的操作系统，并且每个 BSD 项目都由开源爱好者和社区维

护和开发，而不是像 Red Hat 那样的公司，因此 BSD 项目真正独立。

这种 BSD 的组织方式的结果是，无论你选择哪个 BSD 项目，你在基本安装中都不会发现疯狂的选择退出（opt-out）间谍软件设置，而且你不会发现损害隐私的解决方案集成到操作系统的核心组件中。

相反，由于 BSD 项目由熟练和热情的人员开发和推动，他们非常关心操作系统的设计、安全性和隐私性，你经常会发现即使是通过软件包管理器安装的第三方软件也会被修复，以消除这些问题，例如 OpenBSD 在 Firefox 中[禁用 DNS over HTTPS](https://undeadly.org/cgi?action=article;sid=20190911113856)。

所有这些的另一个巨大优势是，围绕 BSD 项目的社区由经验丰富、乐于助人且（大多数时候）友善的人组成。

## 许可证问题

GPL 许可证对开发者更严格，它是一种[伪式开源](https://www.youtube.com/watch?v=Pm8P4oCIY3g&t=37m05s)，因为它强制要求公开所有修改后的源代码，并阻止其他开源项目的集成，例如，GPLv2 阻止了在 Linux 中集成 [DTrace](https://en.wikipedia.org/wiki/DTrace) 和 [ZFS](https://en.wikipedia.org/wiki/ZFS)。

另一方面，BSD 开发者没有这样的限制。制造商可以选择将 BSD 作为他们创建新设备时的首选操作系统，而非 Linux。这将使他们可以保留自己的代码修改。而 Linux 的许可证则强制要求将源代码公开。

听起来 GPL 许可证可能更好，因为为什么我们要允许公司简单地“窃取”我们的开源代码，并生产专有产品而不给予任何回报。但事实并非如此简单。通过 GPL 许可证强制公司向公众发布源代码，公司很快变得更具操纵性。

Red Hat 在发布 systemd 时采取的策略是试图让尽可能多的“重要”第三方项目与 systemd 紧密合作，甚至依赖于 systemd。这样，其他 Linux 发行版更容易被说服采用 systemd，因为这些第三方项目的轻松集成。systemd 开发者与几个第三方项目进行了对接，并试图说服他们使其项目依赖于 systemd，例如 Lennart Poettering 在 [Gnome 邮件列表](https://mail.gnome.org/archives/desktop-devel-list/2011-May/msg00427.html)上的尝试，以及 Red Hat 开发者“keszybz”在 [tmux 项目](https://github.com/tmux/tmux/issues/428)上的尝试。这些尝试大部分最初都被“伪装”成技术问题，但当你阅读 Gnome 邮件列表和其他地方的长电子邮件往来时，真正的意图变得非常清晰。

这样的操纵在 BSD 中是不需要的。公司可以自由地使用和修改 BSD，因此它们不需要试图影响事物的发展。如果不是这样的情况，我们可能会看到，例如，索尼竭尽全力影响 FreeBSD 的发展，因为他们在 PlayStation 产品中使用该操作系统。

不同的 GNU/Linux 发行版，如 Debian GNU/Linux、Arch Linux，甚至过去的 Red Hat Linux，都是真正伟大的项目。当项目由激情而不是利润驱动时，它们往往会变得更高质量。然而，当项目不再受激情而是受利润驱动时，它们往往会在质量上下降。这是自然而然的，因为利润动机与激情动机非常不同。这是为什么微软 Windows 一直是如此糟糕的操作系统的原因之一。

Microsoft Windows 在桌面上的成功并不是因为人们认为 Windows 是一个优秀的操作系统，没有理智和有经验的系统管理员或 IT 支持者会这样认为，而是因为微软采取了激进的营销策略。

虽然 BSD 项目确实会得到公司的代码和偶尔的财务支持，但它们是出于激情而不是利润驱动。这主要意味着经过深思熟虑的决策。它们不会因利润而对隐私或安全做出妥协，就像我们可能在 Linux 中找到的那样。

请查阅我的文章[《GPL 的问题》](https://www.unixsheikh.com/articles/the-problems-with-the-gpl.html)（The problems with the GPL）以获取更多信息。

## 是时候将一切迁移到 BSD 了

早在 1998-2000 年左右，我就开始将家里和公司的每台服务器和桌面操作系统从微软 Windows 迁移到 GNU/Linux，最初是 Red Hat Linux，后来是 Debian GNU/Linux。我这样做是因为我花了大约十年的时间支持 Microsoft Windows，并在这个绝对可怕的操作系统上浪费了太多时间。

当一个好朋友向我推荐 GNU/Linux 时，我对它的表现感到惊讶，开源社区令人惊讶，所有与 Windows 相关的常见问题都消失了。每当我用 Linux 替换了客户、家人或朋友的 Windows 设置时，（需要的）支持时间迅速减少。当然，这意味着较少的客户支持工作，但这非常好，因为现在我们可以将时间集中在更重要的事情上。

稍后我发现了 BSD 世界，并最终开始在服务器和桌面上部署 FreeBSD 和 OpenBSD。

在过去，Linux 通常比 BSD 具有更好的硬件支持，因此我通常更多地使用 Linux 而不是 BSD。硬件很昂贵，而且并不总是可能根据您想在系统上运行的操作系统购买硬件。但现在情况不同了，BSD 通常对现代硬件具有很好的支持。

我仍然喜欢 GNU/Linux，但我不想担心 systemd 中可能存在的可能破坏隐私的垃圾，或者 Lennart Poettering 接下来会想出什么恶心的东西，我也不想担心内核中加入的所有膨胀软件，比如内核[强制适应 DRM](https://patchwork.kernel.org/patch/10084131/)。我一般不想担心下一个问题是什么。一切都应该是理智的选择和默认决策！不是选择退出（opt-out）！

你可以在我的文章[《FreeBSD 是一个了不起的操作系统》](https://unixsheikh.com/articles/freebsd-is-an-amazing-operating-system.html)https://unixsheikh.com/articles/freebsd-is-an-amazing-operating-system.html中了解有关 FreeBSD 的内容，以及在我的文章[《OpenBSD 很棒》](https://unixsheikh.com/articles/openbsd-is-fantastic.html)https://unixsheikh.com/articles/openbsd-is-fantastic.html中了解有关 OpenBSD 的内容。

---

harryruhr 在我的文章[《为什么你应该将所有东西从 Linux 迁移到 BSD》](https://unixsheikh.com/articles/why-you-should-migrate-everything-from-linux-to-bsd.html)上发表了一篇回应，题为[《你应该从 Linux 迁移到 BSD 吗？这取决于……》](https://fediverse.blog/~/AllGoodThings/should-you-migrate-from-linux-to-bsd-it-depends)。在这个回应中，harryruhr 提出了几个论点，我感觉有必要对其进行回应。

## 关于分裂问题

在我提到“Linux 存在分裂”这一观点后，harryruhr 写道：

>是的，的确如此。但是现在 BSD 亦如此。单单三个“传统”的 BSD——FreeBSD、NetBSD 和 OpenBSD——在技术和目标上就有很大的差异。然后还有“新”的 BSD 分支，比如 Dragonfly、MidnightBSD、HardenedBSD 等等。在 Distrowatch.com 列出了 18 个不同的 BSD“发行版”。作者如此高度赞扬的 ZFS 文件系统只在 FreeBSD 及其近亲中可用，并且是基于“ZFS on Linux”。它在 NetBSD 和 OpenBSD 上不可用。

这是完全不正确的。FreeBSD 是最早将 ZFS 从 Sun Microsystems 移植过来的独立操作系统之一。ZFS on Linux 出现得要晚得多，然后演变成了 OpenZFS，后来成为所有[自由和开放源代码社区](https://en.wikipedia.org/wiki/Free_and_open-source_software)的 ZFS 贡献者之间的重要合作项目。来自 Linux、FreeBSD、NetBSD、Illumos 等地的开发者现在都在为这个项目做出贡献。

Linux 之所以分裂，是因为内核、GNU 工具、库和所有其他组件都是完全独立的项目。事实上，这些项目互相之间几乎没有任何关联，但同时，你不能没有以某种形式将这些不同的项目组合在一起，这就是不同的 Linux 发行版所做的事情。

GNU 项目甚至自 1990 年以来一直在开发他们自己的内核 [GNU Hurd](https://en.wikipedia.org/wiki/GNU_Hurd)，最初计划作为 Unix 内核的替代品。由于 Hurd 内核尚未完成，Linux 内核只是一个方便的方式来启动工作中的操作系统。

而 BSD 则完全没有分裂，它们都是独立的完整操作系统和独立项目，都有自己的内核、基本工具等等。它们是拥有不同目标的独立项目。它们共享 BSD 内核的家族树，并且偶尔共享代码，但除此之外它们是相互独立的。如果 FreeBSD 或 NetBSD 被取消，OpenBSD 并不会受到很大的影响，反之亦然。

对于 Dragonfly BSD，情况也是一样的。Matthew Dillon 在 1994 年至 2003 年间是 FreeBSD 开发者，在 2003 年，由于他认为 FreeBSD 中采用的线程和对称多处理技术将导致性能不佳和维护问题，他与其他 FreeBSD 开发者意见不一，于是他创建了 DragonflyBSD。但是 DragonflyBSD 现在也是完全独立的操作系统和项目。

所有这些不同的 BSD 项目仍然都是完整且独立的操作系统。它们不是由来自不同项目的独立部分组合而成的。

至于 MidnightBSD、HardenedBSD 和其他类似的项目，它们也与分裂无关。这些项目大多数都是基于 FreeBSD 的，它们采用 FreeBSD 并设置不同的应用程序，或者对内核进行补丁等。它们与分裂毫无关联。

如果 BSD 项目像 GNU/Linux 一样分裂，那么 BSD 内核应该由一个独立的项目开发，基本工具应该由另一个独立的项目开发，直到你将所有这些独立而分裂的部分“粘合在一起”，什么都不会起作用。

这就是 GNU/Linux 操作系统与 BSD 操作系统之间的分裂性质的区别。

## 关于“被劫持”问题

关于我的观点“Linux 被劫持”，harryruhr 写道：

>作者说大型“有争议”的公司正在影响 Linux 的发展。这可能是真的，但在 FreeBSD 中情况是否真的更好呢？
>
>FreeBSD 社区在每一个可能的机会都自豪地宣称 Netflix 正在使用 FreeBSD 来传送他们的内容。而且 Netflix 是 FreeBSD 内核最大的商业贡献者之一。然而，目前在 FreeBSD 桌面上本地观看 Netflix 内容仍然不可能。如果这一点，以及他们传播 DRM 内容，不能使 Netflix 成为最有争议的公司之一，我不知道什么是“有争议”。此外，当然还有数十个基于 FreeBSD 的商业和专有产品，这些产品否定了用户享受自由软件的好处。

正确的是，Netflix 是 FreeBSD 最大的商业贡献者之一，但这与 Linux 世界中的“劫持”无关。Netflix 将他们在 FreeBSD 上的所有改进都贡献回项目。他们所做的所有性能优化都已贡献给 FreeBSD。这对 FreeBSD 非常有益。

但 Netflix 绝不会试图影响 FreeBSD 项目或者试图“劫持”FreeBSD。他们也没有开始制作新的 init 系统，然后后来揭示这实际上[并不是一个 init 系统](http://0pointer.de/public/gnomeasia2014.pdf)，而是一个 *“永远未完成、永远不完整，但跟踪技术进展”* 的东西，不断壮大。

Hacker News 上的一位名为 drewg123 的 Netflix 员工提供了以下[相关信息](https://news.ycombinator.com/item?id=22106424)：

>在 Netflix，我们的大型工作组中至少有 7 位 FreeBSD 提交者，以及一位核心团队成员（我肯定还忘记了一些人，对此我感到抱歉！）。我们雇用了许多其他的提交者和核心团队成员（根据具体合同）。
>
>与雅虎不同，我们要对 Netflix 的业务目标负责，这些目标包括提高我们的流媒体客户体验质量，同时保持或降低内容传送的成本。经常情况下，这些目标与 FreeBSD 的目标高度契合，我们做出了大量贡献（如异步 sendfile、kTLS、未映射的 mbufs、时代稳定、NUMA 工作、RACK TCP、BBR TCP、TCP pacing 等等，我可能忘记了很多其他的贡献）。

关于 Netflix 提供的服务以及只能在其专有应用程序中播放的所谓 DRM 内容，以及其他基于 FreeBSD 的专有项目，这对 FreeBSD 没有影响，也与“劫持”毫无关系。这些项目都不会影响 FreeBSD。

因此，是的，FreeBSD 的情况要好得多。

## 关于“理性人”在哪里的问题

对于我提到“BSD 是理智人的所在地”这一观点，harryruhr 写道：

>作者说：“BSD 项目维护整个操作系统，不仅仅是内核。”这是正确的，BSD 确实不仅仅是内核，还包括用户空间程序。但是 BSD 操作系统中包含多少“用户空间”完全取决于 BSD 开发者。通常只有一些最基本的工具。其他工具必须通过 Ports 和软件包进行安装，这与在 Linux 发行版中使用软件包没有什么不同。例如，FreeBSD 的基本系统中甚至没有 Xorg，你必须使用“pkg install xorg”从软件包安装它。很少有情况是将集成系统的一部分从基本系统中移除，并转变为软件包。

我觉得这种说法有点误导。

你不能将 FreeBSD 中必须从第三方项目安装 Xorg（因为它不存在于基本系统中）与 GNU/Linux 操作系统的分裂性实际情况进行比较。这两者完全没有关系。

我的文章是关于 GNU/Linux 操作系统与不同 BSD 操作系统之间的分裂性质进行对比，而不是关于基本安装中包含多少第三方应用程序。

接着 harryruhr 说：

>最“完整”的系统确实是 OpenBSD，它不仅带有 X（Xenocara），还有自己的 MTA（OpenSMTPd）和 Web 服务器（OpenBSD httpd），这使得 OpenBSD 的基本系统成为基本任务的服务器的良好选择。当然，除了 xterm、xcalc 和 3 个窗口管理器（twm、fvwm 和 cwm）外，几乎没有其他“图形”程序包含在内。如果你想要一个 Web 浏览器或者像样的邮件程序，你必须从软件包安装。

OpenBSD 中的 X、OpenSMTPd、httpd 和其他应用程序与操作系统本身无关。无论你选择将这些应用程序放入基本系统中还是留在外部，都不会影响 OpenBSD 仍然是一个完整的操作系统，即使没有这些部分。

这些部分不会使 OpenBSD 比 FreeBSD 更“多”地成为一个操作系统。这些部分只是使 OpenBSD 的基本安装中包含了更多的应用程序。

OpenBSD 项目决定将更多的应用程序集成到基本系统中，因为 OpenBSD 非常重视安全。开发者希望将这些应用程序与基本系统集成在一起，以控制这些部分的开发和工作方式。因此，这些应用程序已成为 OpenBSD 项目的一个整合部分。

FreeBSD 或 NetBSD 也可以在基本系统中提供大量的应用程序，但对于这些项目来说这样做没有意义。

事实是这些应用程序并不影响操作系统的完整性。另一方面，如果你没有内核，或者你没有“用户空间”工具，你什么都没有。这就是 GNU/Linux 的现实。

最后，我想指出，我提到 GNU/Linux 操作系统的分裂状况，是为了指出这种分裂是我们面临问题的主要原因之一。这些分裂项目经常有冲突的利益，这才是问题的核心，而不是哪个操作系统在基本安装中有最多的工具。

## 关于许可证问题

harryruhr 接着写道：

>关于“BSD 或 GPL”的讨论就像许可证本身一样古老。对于两者，都有支持和反对的好理由。就个人而言，只要是自由软件，我并不在乎。关于 GPL 导致的软件由利润主导而非同情心的论点并不真的令人信服。

我认为，任何真正关心“自由软件”，即人们可以自由分发和修改的开源软件的人，都应该更深入地考虑这个问题。我们有超过三十年的经验，显示出许多以利润为驱动的公司在依赖一款无法控制的软件时会变得多么具有操纵性。

使用 BSD 许可证，公司无需使用策略或操纵手段来影响应用程序的开发，它可以直接使用该应用程序并随心所欲地操作。

## 关于迁移问题

最后，harryruhr 针对选择 BSD 而非 Linux 的原因，写下了以下内容：

>但除了使用免费软件的原因外，这些理由应该只是技术上的。那些过时的论点，比如“BSD 是完整的系统，Linux 只是内核”或者“Linux 是被利润驱动，BSD 是被同情心驱动”，并不真的适用（再也不适用）。

通常情况下，我完全同意唯一选择特定操作系统的理由应该是技术上的原因。然而，这种情况不再适用，而我提出的论点也不是过时的，相反地是事实。

对于特定应用程序或硬件，Microsoft Windows 10 可能是唯一可用的操作系统，但这并不意味着你应该因为技术原因而妥协，忽视这个可怕的操作系统带来的重大隐私问题。事情总是有“过多的垃圾”。

由于最近 Linux 内核强制采用[数字版权管理（DRM）](https://patchwork.kernel.org/patch/10084131/)，以及 Linus Torvalds 对现实的几次脱节表态，以及他对 Linux 世界中许多重要问题的完全漠视，显然他并不关心公司如何影响开发，Linux 内核的未来在隐私和安全方面并不看好。

除非你想在每次发布新的 Linux 内核时都需要自行修补问题，否则你需要一个可行的替代方案。这个替代方案应该由开发内核的人明确了解项目的发展路径，这个路径不应该影响隐私、安全或其他任何重要问题。

当然，任何项目中总会有分歧，但 FreeBSD 开发者之间的分歧与以利润为驱动的公司试图在各种程度上“劫持”GNU/Linux 是不同的。
