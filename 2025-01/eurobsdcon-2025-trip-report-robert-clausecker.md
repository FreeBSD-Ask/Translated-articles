# Robert Clausecker EuroBSDcon 2025 旅行报告

- 原文：[EuroBSDcon 2025 Trip Report – Robert Clausecker](https://freebsdfoundation.org/blog/eurobsdcon-2025-trip-report-robert-clausecker/)
- 发布日期：2025 年 11 月 12 日
- 作者：Robert Clausecker

今年的 EuroBSDcon 在克罗地亚萨格勒布大学工程与计算学院举办。这是我第二次参加 EuroBSDcon，此前我曾在 2024 年的都柏林 EuroBSDcon 上就我在 [SIMD 增强 libc 字符串函数] 的工作作过报告，我对今年的会议非常期待。我非常感谢 FreeBSD Foundation 资助我前往会议的旅程，否则这对我来说将很难负担得起。

我与 Jan Bramkamp 一起旅行并共享了一个房间，他在会议上做了一场关于基于 ZFS 的 jail 配置的精彩演讲。我们居住在德国柏林，因此利用机会乘坐 ICE 列车到慕尼黑，然后通过卧铺列车前往萨格勒布。这让我们能够精力充沛地到达会场，并有机会欣赏斯洛文尼亚乡村的美景。在会议期间，我们还与 Getz Mikalsen 和 Benni Stürz 会面，他们通过去年参加 Google Summer of Code 项目而加入此项目，还有另一位来自柏林的感兴趣的学生。

今年的 EuroBSDcon 首次延长至五天。前两天用于 FreeBSD 开发者峰会（Dev Summit），随后是 Eurobhyvecon，同时进行的还有教程。第四天和第五天举办了 EuroBSDcon 正式会议，提供了三条平行轨道的丰富演讲内容。

在 FreeBSD 开发者峰会的第一天，我最感兴趣的是关于 pkgbase 项目的进展（计划在 FreeBSD 15 中用正式打包系统取代我们自制的 freebsd-update(8)）以及 CHERI 工作的上游合并情况。我也花了很多时间在走廊轨道上，与许多参加会议的其他项目成员交流。一场意外吸引我注意的演讲是 Alfonso Siciliano 关于 FreeBSD 可访问性的讲解，这是一个平时很少听到的话题。

第二天让我对核心团队的运作以及成员们面临的各种问题有了一些有趣的了解，同时也对 libzfs 有一些认识。第三天比较轻松；我没有参加 Eurobhyvecon，后来加入了其他开发者前往距离不远的 Nikola Tesla 技术博物馆的小旅行。

第四天和第五天的演讲都非常密集，有时很难选择参加哪一条轨道。在同事 Jan Bramkamp 关于基于 ZFS 的 FreeBSD jail 配置的演讲之后，我在 Harald Eilertsen 关于将 OpenJDK 移植到 FreeBSD 的演讲中学到了许多挑战；作为 ports tree 的常规贡献者，处理此类问题对我尤为重要。接下来的演讲介绍了用于并行构建工具（如 make(1)）的轻量级作业服务器协议，为以工具无关的方式解决此类问题提供了新思路。在 pkgbase 的演讲之后，Björn Zeeb 详细讲解了在 FreeBSD 上使 WLAN 驱动正常工作的进展——这是提升笔记本支持的重要问题。虽然我的笔记本 Qualcomm QCNFA765 网卡目前仍不受支持，但显然已经取得了重要进展，全力支持只是时间问题。当天的演讲以 Moin Rahman 关于 FreeBSD ports tree 的讲解结束，我们在社交活动中继续了讨论。

在最后一天，我遗憾地错过了主题演讲，但参加了 Kirk McKusick 关于 UNIX 守护进程历史的报告（去年我错过了第一版）。午餐后，我在 Charlie Li 的 MIDI 演讲和 John Baldwin 的链接器演讲之间艰难选择，最终参加了后者。随后还有关于去中心化互联网服务的演讲，以及一场关于 USB-C 内部工作机制的非常有启发性的讲解。在闭幕前，我还参加了关于如何利用 FreeBSD 改善 UNIX 教学的讲座。这对我尤其有意义，因为我们在柏林自由大学每年都会开设关于 UNIX 系统管理的课程，涵盖 FreeBSD、AIX 和 Solaris，并一直在寻找改进教材的方法。

除了会议议程之外，我也非常感激能有机会与许多其他 FreeBSD 开发者交流，讨论各种问题和项目想法。我希望明年在比利时布鲁塞尔举办的会议能够和今年一样精彩！

再次衷心感谢 FreeBSD Foundation 资助此次旅程。
