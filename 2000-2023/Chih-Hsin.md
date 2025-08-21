# AsiaBSDCon 2023 旅行报告：Chih-Hsin Chang

- 原文：[AsiaBSDCon 2023 Trip Report: Chih-Hsin Chang](https://freebsdfoundation.org/blog/asiabsdcon-2023-trip-report/)
- 2023 年 5 月 4 日
- 作者：Chih-Hsin Chang

我很高兴能报告我最近参加了开发峰会和 AsiaBSDCon 2023。FreeBSD 基金会慷慨地资助了这次旅行，我非常感激能有机会参加。在开发峰会上，我有机会做了一场关于 FreeBSD OpenStack 项目现状的演讲。我很兴奋能与社区分享我们的进展，并讨论一路上遇到的一些挑战。演讲得到了良好的反馈，我也收到了其他与会者的宝贵意见和建议，这将有助于我们进一步改进项目。总的来说，能与社区中的其他人分享我们的工作并展开合作，是一次非常棒的体验。

在我演讲的问答环节中，Richard Yao 提供了一些技巧，用于调查我遇到的 bhyve 嵌套虚拟化问题。这些建议非常有帮助，将在我们推进项目时节省时间和资源。

在这个项目中我们面临的一个重大挑战是，FreeBSD 世界里没有 OpenStack Neutron 大量依赖的 Linux bridge 和网络命名空间。我有机会与 Kristof Provost 讨论了这一点。通过与他的请教和交流，我对网络移植部分的下一步行动更有信心。我们的目标是用 FreeBSD 世界的对应解决方案来替代：用 pf 替代 iptables，用 vnet 替代 Linux 命名空间，用 epair 替代 veth。这次对话极其宝贵，将帮助我们在项目的这一关键部分取得进展。

我还遇到了一位来自 UPB（**译者注：这里指 <https://github.com/FreeBSD-UPB>，已于 2023 年停滞**）的演讲者，他承担了 OpenStack Nova 移植的任务。在交谈中，我学到了有关项目可能遇到困难的宝贵经验。这次信息丰富的交流将帮助我们预见潜在挑战并制定解决方案。

论文报告的第一天由 Mike Chu 的精彩演讲开始。我在云计算行业深耕多年——主要是在 Linux 世界。我对 BSD 世界里类似的工具、解决方案乃至整个生态系统很感兴趣。Mike 的演讲向我们展示了 FreeBSD 容器生态中缺失的一块版图，即构建、管理和分发 FreeBSD 容器的能力。演讲结束后，我们一起吃了午饭，并讨论了 FreeBSD jail 和 Linux 容器的理念。这是一场富有成果的辩论。

第一天有很多关于 bhyve 的报告，我参加了其中的大部分。很高兴看到 bhyve 拥有如此丰富的功能集。这也提醒了我，目前 FreeBSD OpenStack 项目的实现只集成了 bhyve 的基本功能，以后还有很多工作要做。

在 AsiaBSDCon 2023 的最后一天，我参加了两场由我大学的同学主持的会议，主题是 VT-IME 和 wtap(4)。两者都展现出为 FreeBSD 做出贡献的巨大潜力，他们在内核开发方面扎实的知识令我震惊。我很荣幸能与他们面对面讨论这些令人印象深刻的工作，了解他们是如何走到今天的。了解到这些开发者的非技术背景也很有趣。

总的来说，我参加 AsiaBSDCon 2023 是一次宝贵的经历。我能够展示 FreeBSD OpenStack 项目的现状，向该领域的其他专家学习，并获得有价值的见解，这些都将帮助我们在这个重要项目上取得进展。
