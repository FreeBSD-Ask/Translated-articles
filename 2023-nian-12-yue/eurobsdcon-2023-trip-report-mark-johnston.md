# EuroBSDCon 2023 旅行报告——Mark Johnston

- 原文链接：<https://freebsdfoundation.org/blog/eurobsdcon-2023-trip-report-mark-johnston/>
- 作者：Mark Johnston
- 译者：Canvis-Me & ChatGPT

2023 年 10 月 24 日

FreeBSD 基金会慷慨地资助了我前往葡萄牙科英布拉参加 EuroBSDCon 2023 的行程。与往常一样，会议包括为期两天的 FreeBSD 开发者峰会，然后是为期两天的正式会议。会议在科英布拉大学的几幢建筑中举行，这本身就值得一游：大学建筑壮丽宏伟，坐落在山上，可以俯瞰整个城市。每天早上爬上山有点辛苦，但非常值得。

由于技术问题，开发者峰会的第一天开始得有些晚，但这提供了与其他与会者聊天和了解的好机会。我与 Ruslan Bukin（br@）聊了一些关于他计划将一些软件移植到 Morello 的计划，Morello 是 ARM 的 CHERI 的硬件实现。我还与 Bojan Novković 交谈，他是我今年谷歌编程之夏的学生，谈论了他一直在努力实现的一些补丁。他在谷歌编程之夏结束后继续为 FreeBSD 做贡献。我终于见到了 Olivier Certner，他最近开始为内核贡献高质量的补丁和代码审查。

技术问题得到解决之后，一些演讲按计划进行。几位 FreeBSD 基金会的工作人员介绍了基金会活动的最新进展；看到众多正在进行中的实习项目的反馈很不错。在此之后，Justin Gibbs 进行了一场题为“如果你不害怕，你会做什么？”的演讲，讲述了 FreeBSD 开发者社区的技术成就和“第一次”的历史。一个统一的主题是，项目需要近乎鲁莽的雄心壮志，以及勇于尝试的意愿，而不必过多担心失败的可能性。演讲中强调了很多这样的例子，CHERI 和最初将 ZFS 移植到 FreeBSD 的例子就是其中几个。然后，Justin 邀请我们设想如果不必考虑失败，我们会做什么，人们分享了各种想法。

我们中的一些人一起在学生餐厅用午餐并聊了起来。午餐后，来自基金会的 Greg Wallace 提出了 SWOT 分析的想法，该分析对组织的优势、劣势、机会和威胁进行概述，以提供未来工作规划的客观背景。房间分成小组，每个小组都对 FreeBSD 进行了自己的 SWOT 分析。下午，我终于见到了 Christos Margiolis（christos@），他最近完成了基金会的实习，也是我在 2022 年的谷歌编程之夏学生。尽管我们在几个项目上一直一起工作，但这是我们首次面对面会面，所以见到他真是太好了。我们和其他一些人聊了很多有关 FreeBSD 的话题。在这一天的最后，Jordan Hubbard 进行了一场演讲，诉说了他的职业生涯，并提出了他认为大型语言模型和其他机器学习创新将如何在短期内改变软件领域的观点。

开发者峰会的第二天以几场技术演讲开始。首先，Ruslan Bukin 介绍了他最近在 CPU 跟踪方面的一些工作，该工作使得可以使用 ARM CoreSight 和 Intel PT 等技术。这些设施允许 CPU 实时记录其操作，以相对较小的开销对 CPU 的行为进行细粒度跟踪。Ruslan 的实现，HWT，目前针对 ARM 但已设计为可扩展。Ruslan 还进行了一些有趣的演示，其中他使用启用跟踪的小命令行实用程序（例如 uname(1)）。这展示了 HWT 的符号解析能力。HWT 在精神上与 Christos 最近由基金会赞助的有关 DTrace 的工作相似，该工作使得可以跟踪单个 CPU 指令，但具有完全不同的权衡。

然后，Bojan 介绍了他为谷歌编程之夏开始的一些工作，具体是一个用于内核性能基准测试的框架。这是因为要总结他所开发的主要功能（一个执行在线内存碎片整理的内核子系统）的性能改进和影响是一项挑战。特别地，这样的系统必然会有一些性能开销，因为它消耗 CPU 资源，但它（希望）通过为系统的其余部分提供更连续的物理内存而带来一些好处，这可以改善某些工作负载的性能。因此，确定该功能的净效果是一项棘手的任务，因此需要进行基准测试。

接下来的一天的日程相对较轻松，所以我花了一些时间来开发一个个人项目，目的是让运行 FreeBSD 回归测试套件变得更容易，并与其他一些 FreeBSD 开发人员讨论了我们的开发工具中的各种差距和不一致。

随后的几天举办了 EuroBSDCon。我参加了相当多的演讲，一些亮点包括：

- Hiroki Sato 针对他在 FreeBSD 的 USB 堆栈上的补丁进行演讲，以启用 USB 调试功能。这是一种存在于一些 USB 控制器中的硬件特性，允许在 USB 上创建通信通道。这样就能为任何带有兼容 USB 控制器的设备提供调试控制台。这对于调试与笔记本电脑的挂起/恢复相关的问题应该非常有用，否则缺乏可用于调试的带外通道。这项工作需要一些特殊的硬件，特别是 USB 交叉线；幸运的是，Hiroki 在演讲后带来了几个，大家可以买。我不得不迅速行动，最后抢到了一个。

- Christos Margiolis 关于 kinst 的演讲，这是他在谷歌编程之夏 2022 和他在 2023 年基金会实习期间开发的新 DTrace provider 程序。我以前已经与他在这方面合作过，因此对技术背景已经很熟悉，但看到他向更广泛的观众展示它还是很不错的。kinst 是一种相当底层的技术，需要一些 CPU 内部知识才能理解。Christos 在向多样化的观众解释它时做得非常出色。演讲后有许多有趣的问题和讨论，我也参与其中。

- Michael Chiu 对 xc 的工作进行了介绍，这是一个针对 FreeBSD 的新的容器运行时。这是一项非常令人印象深刻的工作，提供了几个围绕管理 FreeBSD jail 的便利功能。演讲包括相当多的演示，例如使用 xc 在 FreeBSD 上的 Linux jail 中启动 MariaDB。我受到演讲的启发，尝试使用 xc 来运行一些我自己的工作负载；特别是，它似乎将成为在 FreeBSD 上运行最小设置的自动化测试的一种相当有用的方式。我一直在开发一个 xc 配置，可以用在 FreeBSD 上运行 syzkaller。总体而言，我对 Michael 的工作感到非常激动。

我还与 Warner Losh 进行了一些交流，了解他启用通过 LinuxBoot 引导 FreeBSD 的一些工作。LinuxBoot 通过将传统的引导固件替换为一个负责初始化硬件然后引导主 Linux 内核的最小 Linux 内核，简化了某些平台上的引导过程。以这种方式引导 FreeBSD 涉及实现一个称为 kexec(2) 的二进制接口；这个工作是 Warner 演讲的主题。最近，我一直在进行一个在精神上类似的项目，有相当多的技术重叠，所以听到他的工作很有帮助，当我们要将各自的项目提交到 FreeBSD 时，我们可以一起努力。

我想感谢 FreeBSD 基金会对我行程的赞助；我进行了很多非常富有成效的交流，并了解了 FreeBSD 社区中其他人正在从事的工作。
