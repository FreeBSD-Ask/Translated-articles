# 采访 FreeBSD 基金会执行董事 Deb Goodkin


- 原文链接：<https://devm.io/open-source/freebsd-open-source-goodkin>
- 译者：ykla & ChatGPT

  
>**致力于开源：“[FreeBSD](https://www.freebsd.org/) 是（规模）最大且历史最悠久以民主方式运作的开源项目之一。”**

我们与 FreeBSD 基金会执行董事 Deb Goodkin 进行了交谈，谈论了 FreeBSD 这个自由、开源的类 Unix 操作系统。三十年过去了，FreeBSD 提供了什么，它的成功故事是什么，以及未来的发展方向如何？

**devmio:** 我们的读者可能对 FreeBSD 不太熟悉，请你简要介绍一下它。

**Deb Goodkin:** FreeBSD 是（规模）最大且历史最悠久以民主方式运作的开源项目之一。它是一个自由的、类 Unix 的操作系统，经由 Berkeley 软件分发（BSD）——也被称为“伯克利 Unix”，从对 Unix 的研究发展而来。该项目于 30 年前的 1993 年 6 月 19 日被正式命名为 FreeBSD。与其他开源操作系统不同，FreeBSD 基础系统是一个由单一团队开发和发布的集成操作系统分发版本。目前，有 400 多名活跃开发人员和数千名贡献者致力于支持 FreeBSD 项目。FreeBSD 可在 32 位和 64 位 Intel / AMD x86、32 位和 64 位 Arm、RISC-V、PowerPC CPU 以及 Amazon AWS、Microsoft Azure 和 Google 云端平台等云服务提供商上运行。已有数以千万计的机器正在使用 FreeBSD。

FreeBSD 通过奉行“少惊奇原则”而独具特色。遵循“不破坏正常运行的事物”开发路径，该操作系统不会无故更改。因此，用户可以在后续版本中继续应用在早期版本上获得的知识和经验。

许可证也在 FreeBSD 的吸引力中发挥着关键作用。该操作系统采用无版税的 BSD 许可证，相比其他开源替代方案，限制较少。这种许可自由允许包含和分发仅有二进制形式的版本，使 FreeBSD 特别适合于嵌入式平台。

**devmio:** 随着开源的 FreeBSD 操作系统于在今年夏天迎来 30 岁生日，开发人员可以在未来几年里期待 FreeBSD 为他们带来些什么？

**Deb Goodkin:** 开发人员和用户将看到桌面和硬件支持的改进，使得使用 FreeBSD 更加容易入门。我们专注于改善新用户体验，包括改进文档和入门指南，并为希望学习 FreeBSD 的人们提供更多的教育和培训机会，包括 FreeBSD 视频和指南。我们还在努力实现更多测试用例的自动化，以便能够快速测试变更，减少引入新错误的风险。

>**FreeBSD 以“最少惊讶原则”为特点。遵循“不破坏正常运行的事物”的开发路径，该操作系统不会无故更改，除非有充分的理由。**

**devmio:** 你认为 FreeBSD 之所以有如此持久的影响力是因为什么？有哪些技术/开源社区生态系统确保了它的受欢迎程度？

**Deb Goodkin:** 有多个因素才让这个项目的如此长寿。其中两个最重要的因素是 BSD 许可证和扁平化的开发模型。如前所述，限制性较少的许可证吸引了那些希望基于 FreeBSD 构建产品和服务的人，而无需担心像 GPL 这样的问题。扁平化的开发模型意味着没有中间层的管理者拖慢了将更改纳入代码树的过程。该项目的标准仍然很高，但更容易提交自己的更改。

沿着同样的线路，FreeBSD 是最早建立领导层和组织系统的开源项目之一，这些措施帮助确保了它的持久性。例如，它是第一个使用版本控制和 Bug 报告系统来简化开发过程的先驱者。该项目还是第一个创建选举产生核心领导团队的，而非由一人领导。核心团队处理从设定项目的战略方向到解决冲突等所有事务。

之前提到的“最少惊讶原则”也使得 FreeBSD 对依赖稳定性和易于升级以维护自己的产品和服务的个人和公司具有吸引力，这些产品和服务中包含了 FreeBSD。

最后，FreeBSD 社区友好且乐于助人，有着强大的教育文化。事实上，许多最初的开发人员仍然参与其中，并期待帮助其他人开始使用 FreeBSD 的旅程。

**devmio:** FreeBSD 技术在实际应用中有哪些最有趣的用例？

**Deb Goodkin:** 由于其高性能、安全性和先进的网络功能，FreeBSD [无处不在](https://en.wikipedia.org/wiki/List_of_products_based_on_FreeBSD)。它被用于互联网服务器、嵌入式设备和防火墙等多个领域。然而，一些值得注意的用例包括：

- 倍福
倍福使用成熟的基于 PC 的控制技术来实施开放的自动化系统。他们的产品 TwinCat/BSD 被应用在驱动技术、自动化软件、无控制柜自动化和机器视觉硬件等领域。TwinCat/BSD 将倍福的专有 TwinCat 运行时与 FreeBSD 结合使用。由于 Windows CE 即将停止支持，倍福需要一个非 GPL、稳定且安全的操作系统，能够支持全系列硬件，FreeBSD 正好满足这些要求。

- CHERI/Morello
CHERI 是“Clean Slate Trustworthy Secure Research and Development”（CTSRD - 发音为“custard”）项目的一部分。这是[国际斯坦福研究所计算机科学实验室](https://www.csl.sri.com/)和[剑桥大学计算机实验室](https://www.cst.cam.ac.uk/)联合研究项目，得到了 DARPA（DARPA CRASH、MRC 和 SSITH 计划的一部分）、Google 和 Arm 的支持。CHERI 旨在保护系统编程语言（如 C 和 C++）中的内存安全漏洞。通过改变架构接口，CHERI 在硬件层面强制执行内存保护。FreeBSD 是他们原型完整软件栈的一部分。如今，CHERI 通过 Arm 的 Morello 处理器设计有限供应。CHERI 和 Morello 共同构建了一个不断增长的硬件内存安全生态系统，由英国政府的 Digital Security by Design 计划协调。应用包括 CHERI WASM、Cloud Native CHERI 和 AutoCHERI。

- Netflix
FreeBSD 驱动着 Netflix 全球内容交付网络 Open Connect 的构建模块，向全球会员提供 Netflix 电视节目和电影。通过 FreeBSD 提供动力的 Open Connect 设备，使得 Netflix 能够在单核 x86 服务器上使用内核 TLS 加密实现令人难以置信的性能（单个 OCA 的 400 Gb/s）。

>**这个社区非常平易近人和友好，拥有强大的指导文化。事实上，许多最初的开发人员仍然参与其中，并期待着帮助其他人开始他们在 FreeBSD 上的探索之旅。**

**devmio:** 你认为在开发者世界中，对于 FreeBSD 是否存在一些误解？

**Deb Goodkin:** 是的。很多人认为 FreeBSD 只是另一个 Linux 发行版。虽然两者之间有相似之处，但 FreeBSD 是直接起源于 Unix，而 Linux 是作为当时专有的类 Unix 操作系统 Minix 的开源替代品而建立的。

有些人认为 FreeBSD 对于变化不太敏感，不像其他开源操作系统那样（敏感）。然而，随着技术的变化，FreeBSD 将持续发展。无论是转向 Git 还是专注于改进开发者工具，FreeBSD 都在持续观察技术发展，并确定对操作系统和社区目标最有意义的方向。

我们还有许多长期贡献者仍然在继续参与项目，并继续吸引新的开发人员。除了创建培训资料和入门指南外，FreeBSD 基金会在各种活动中进行招募，还提供实习项目，并参与谷歌代码之夏，以确保社区的持续增长。

**devmio:** 对于想要开始使用 FreeBSD 或参与其发展的开发者，你有什么建议？

**Deb Goodkin:** 对于想要开始使用 FreeBSD 的人，[FreeBSD 基金会网站上有许多资源](https://freebsdfoundation.org/freebsd-project/resources/)，包括安装指南、操作指南、一小时的 FreeBSD Fridays 系列介绍性演讲，以及链接到社区创建的内容，比如 [RoboNuggie 的 YouTube 频道](https://www.youtube.com/c/robonuggie)。如果你希望更深入地参与开发工作，我们最常推荐的方式是从提交一项对文档的更改开始，浏 览[PR（问题报告）](https://www.freebsd.org/support/bugreports/)列表并修复一些 bug，或者帮助维护或添加一个 Port。我们还建议查看项目的新手页面。最后，在 FreeBSD 的[邮件列表](https://www.freebsd.org/community/mailinglists/)、IRC 和 [Discord](https://wiki.freebsd.org/Discord) 频道中可以找到许多社区互动的机会。

>**由于其高性能、安全性和先进的网络功能，FreeBSD 无处不在。它被广泛应用于互联网服务器、嵌入式设备和防火墙等众多领域。**

**devmio:** 你认为最近开始的其他开源项目在 30 年后会蓬勃发展吗？

**Deb Goodkin:** 这是一个非常棘手的问题，考虑到当今技术的快速演进速度。我想到的是 RISC-V 开源指令集架构（ISA）。创建允许任何人将软件移植到任何微处理器并且可以无需支付费用进行硬件开发的开放式 RISC 架构是具有开创性的。FreeBSD 项目是第一个在其官方支持中纳入 RISC-V 架构的操作系统供应商，作为 RISC-V International 的成员，FreeBSD 基金会认可将 FreeBSD 纳入该技术领域的重要性。

非常感谢你回答我们的问题！

---

Deb Goodkin 是 FreeBSD 基金会的执行董事，这是一个致力于支持 FreeBSD 项目和社区的非营利组织。早些时候，她曾在与存储相关的公司担任工程职务，包括 IBM、Maxtor 和 Connor Peripherals。Deb 自 2005 年以来一直在该基金会工作。她拥有加利福尼亚大学圣地亚哥分校的计算机工程学士学位（BSCE）和圣塔克拉拉大学的电子工程硕士学位（MSEE）。
