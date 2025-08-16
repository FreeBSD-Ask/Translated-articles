# Chuck Tuffli 的 BSDCan 2025 旅行报告

- 原文：[BSDCan 2025 Trip Report – Chuck Tuffli](https://freebsdfoundation.org/blog/bsdcan-2025-trip-report-chuck-tuffli/)
- 2025 年 7 月 11 日
- 作者：Chuck Tuffli

FreeBSD 基金会善意地赞助了我前往渥太华参加 BSDCan 2025 大会和 FreeBSD 开发者峰会的行程。此次活动为期四天，前两天为开发者峰会，后两天为大会，地点依旧是在渥太华大学。

我及时抵达渥太华参加了 Father and Sons（**译者注：一家酒吧**）的 Goat BoF（**译者注：非正式聚会**）。除了能与 Groff the BSD（译者注：一头山羊）亲密接触外，这也是一次难得的机会，与一年一次才能见面的老朋友叙旧，并结识一些新朋友。长途旅行结束后，我回到 U90 的房间，为开发者峰会第一天的活动休息调整。

开发者峰会的第一天以 FreeBSD 基金会的演讲开场，内容涵盖社区调查、透明度工作以及软件开发项目，特别是他们在改善笔记本电脑支持方面的工作。看到团队在让 FreeBSD 更适合作为日常操作系统方面投入如此专注的努力，令人振奋。

随后，核心团队进行了汇报，强调了若干长期工作，包括审查章程以应对近期挑战、重新设想 DocEng（文档工程）团队的角色，以及制定技术路线图。该路线图将服务于多重目的，例如指导新贡献者回答“我能帮什么？”的问题，并协调社区与 FreeBSD 基金会等组织的工作。

项目的 AI 政策引发了与会者和核心团队之间的广泛争议。拟议的政策禁止使用 AI 或 LLM 生成的材料（Phabricator 审查 D50650 供好奇者参考）（**译者注：即 <https://reviews.freebsd.org/D50650>**），以防潜在的开源许可证违规。讨论达成的共识是不应全面禁止 AI，例如，应允许用 AI 校对提交信息，Ports 中也可以包含 AI 工具。

午餐后，srcmg（源代码管理）团队介绍了他们的使命：减少新开发者的障碍，并提高所有开发者的生产力。他们详述了当前活动，如自动 MFC 提交、漏洞处理会议，以及一个宏愿，即整合项目工具（Phabricator、Bugzilla 和 GitHub）。

当天还包括两场行业演讲：Verisign 介绍了操作系统多样性要求如何促使他们在基础设施中部署 FreeBSD，而 NVIDIA 则讨论了向 FreeBSD 的 mlx5 驱动添加 IPSec 卸载。

开发者峰会第二天，由 Alpha-Omega Project 做了关于软件供应链安全的演讲。该项目自 2021 年起由 Microsoft、Google 和 Amazon 资助，旨在改善开源软件安全性。演讲提供了许多关于改善安全工作中有效与无效做法的深刻见解。

不同于典型的“have, need, want”环节以生成下一版 FreeBSD 所需功能，这次会议集中讨论了 15.0 版本的收尾事项。15.0 的主要变化是将操作系统分发为一套庞大的软件包（即“pkg-base”），而非传统的几个大型分发文件。这一变更备受期待，但讨论显示仍有不少细节需要落实。其他议题包括升级基本系统中的 OpenSSL 版本至新的 LTS 版本，以及部分 32 位架构的弃用。

本届开发者峰会新增了交替对话环节，形式为两名开发者互相对话讨论任意主题，每五分钟由另一名开发者替换其中一人。此环节趣味十足，希望明年还能举办。

在晚间的黑客沙龙中，我有机会回顾几年前未完成的项目。我曾指导一名谷歌编程之夏学生尝试向内核添加 SquashFS 驱动。尽管学生完成了很棒的工作，但未能提交代码。FreeBSD 开发者 Kyle Evans 认为该工作有潜力，他花时间将代码接近可提交状态。我将 Kyle 的修改 rebase 到当前 FreeBSD 内核，编写了 kyua 的集成测试，并与 Alex Ziaee 合作为驱动添加了手册页。

本届 BSDCan 大会的讲座丰富多样，日程安排多次迫使我在同一时段的两个讲座中做选择。其中几场给我留下深刻印象的包括：

- Stefano Marinelli 的“Why (and how) we’re migrating Linux servers to the BSDs”（我们为何，以及如何将 Linux 服务器迁移到 BSD），讲述了他为客户使用开源软件解决问题的经历。BSD 的稳定性和可靠性，加上务实的“解决问题”思路，使客户的疑问从“还有 Linux 以外的选择？”转变为“更多 jail，请”。演讲充满热情与感染力，让我重燃了 FreeBSD 的动力。
- Hans-Jörg Höxer 讨论了 OpenBSD 对 AMD 硬件支持的“Confidential Computing”（机密计算），目标是在不受信任环境下保护虚拟机中的敏感数据。演讲回顾了 AMD Secure Encrypted Virtualization（SEV，AMD 安全加密虚拟化）的先前工作，并介绍了 SEV-ES 的新进展。我对将这些工作移植到 FreeBSD 的 vmm 和 bhyve 感到兴奋。
- Xe Iaso 的 lightning talk“I fight bots in my free time”（我在闲时对抗机器人），介绍了他们开发的 Anubis 软件——一款网页 AI 防火墙工具。聆听开发者为解决自身问题而快速发现他人也面临同样困扰的故事，令人着迷。

在大会的“走廊交流”中，我与另一位 FreeBSD 开发者 Allan Jude 交流了将 bhyve 与 NVMe-oF 存储阵列连接的想法。正是这种偶遇交流理念并可能孕育新项目，使得参加会议非常有价值。

大会闭幕环节包括由 Dan Langille 主持的慈善拍卖，为 Ottawa Mission（**译者注：一家加拿大渥太华的社会服务组织**）筹款。亮点包括：

- 一名与会者以 110 美元竞拍回自己的外套
- Dan 拍卖一个 Trader Joe’s 的纸袋（**译者注：即精美的购物袋**），但需划掉附带收据上的信用卡信息

感谢 FreeBSD 基金会赞助我参加本次大会。
