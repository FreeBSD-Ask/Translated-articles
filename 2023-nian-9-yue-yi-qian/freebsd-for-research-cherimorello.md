# 用于研究的 FreeBSD：CHERI/Morello

- 原地址：<https://freebsdfoundation.org/blog/freebsd-for-research-cheri-morello/>
- 作者：Greg Wallace
- 发布日期：2023 年 8 月 14 日
- 译者：ykla & ChatGPT

***剑桥大学计算安全案例研究***

在创新性突破尚未进入风险投资提案、甚至你的笔记本电脑、微服务或智能手机之前，它们通常是从大学或工业研究实验室的假想开始的。

以 Docker 为例。这个微服务和敏捷方法学的基石是建立在 FreeBSD 中的虚拟化和系统分区创新——[Jail](https://freebsdfoundation.org/freebsd-project/resources/introduction-to-freebsd-Jail/) 之上。而你可能已经猜到，Jail 正是从 Robert Watson 和 Poul-Henning Kamp 的研究衍生出来的。

在对 Jail 的研究之后，[Watson 教授](https://www.cl.cam.ac.uk/~rnw24/) 加入了 [英国剑桥大学](https://www.cam.ac.uk/) [计算机科学系](https://www.cst.cam.ac.uk/about)，继续研究和开发改进计算机系统安全的突破性方法。他的工作使他获得了 [EuroSys Jochen Liedtke 青年研究员奖](https://www.cst.cam.ac.uk/news/robert-watson-wins-eurosys-jochen-liedtke-young-researcher-award-2021)，该奖每年授予展示在系统研究中表现出色的欧洲青年研究者（即获得博士学位后不超过 10 年）。

在选择 Watson 教授时，EuroSys 指出了他在系统研究中的悠久贡献历史，包括共同开发 Jail 安全模型，该模型是当代操作系统容器化的知识基础；他领导了开发内核访问控制框架，该框架用于从 macOS 和 Junos 到 FreeBSD 的应用程序沙箱；他开发了 Capsicum 操作系统安全模型，以及最近领导开发 CHERI 计算机架构。CHERI 现在是英国工业战略挑战基金（ISCF）1.87 亿英镑（约 2250 万美元）数字安全设计（DSbD）计划中探索的关键技术。

CHERI（Capability Hardware Enhanced RISC instructions）是一种架构特性，可实现细粒度的 C/C++ 内存安全和可扩展的软件隔离，并于 2010 年由国际斯坦福研究所和剑桥大学开始开发。

为了使 CHERI 能够编写和运行程序，需要一个开发板。这就是 [Morello](https://www.arm.com/architecture/cpu/morello) 的用途。[Arm 架构的 Morello](https://www.arm.com/architecture/cpu/morello) 是个融合了 CHERI 保护模型支持的处理器设计、单片系统（SoC）和开发板。Morello 开发板、编译器和工具链可使研究人员和工业从业者编写和运行利用 CHERI 的程序。例如，[AutoCHERI](https://autocheri.tech/) 正在将 CHERI 和 Morello 作为超安全架构测试用于车辆遥测控制单元。

Watson 教授非常愿意与 FreeBSD 基金会一起讨论他的研究。这次讨论阐明了学术研究的作用，以及实验室突破如何进入我们的数字社会。Watson 教授的工具中的一个关键组成部分是 FreeBSD。与许多其他操作系统一样，它是开源的，因此支持必要的定制。与 Linux 相比，BSD 许可证的宽松性对于支持他的研究所依赖的公私合作伙伴关系至关重要。

## 研究流程 101

要理解为什么在通常情况下开源，特别是 FreeBSD，对于 Watson 教授的工作，以及更广泛的学术计算机系统研究而言，是非常关键的，首先需要了解研究流程。Watson 教授解释道：“在系统研究中，我们建立了关于想法的原型。你从关于系统应该如何工作、不应该工作的纯假想开始，或者可能是你想出来的一些工具的工具，或者是设计的变化将如何工作，这些都是知识贡献。然后，你将其应用于一个制品，比如操作系统或编译器。然后进行评估，然后根据你的假设、不良结果的学习或者存在问题的情况进行迭代，等等。在 EuroSys 奖提到的所有工作中，FreeBSD 都是那个起始制品。”

>“你可以自己构建参考基准制品，但你真正想要的是当前最先进的状态，具有大规模使用带来的复杂性等等。FreeBSD 满足了这些标准。”

革命性的学术研究受益于生产系统，可以测试假想并将创新转化为生产系统。Watson 教授解释道：“在软件领域，开源对于这种研究方法至关重要，硬件领域的开源性也越来越重要。你 *可以* 自己构建参考基准制品，但你真正想要的是当前最先进的状态，具有大规模使用带来的复杂性等等。FreeBSD 满足了这些标准。”

对于熟悉 FreeBSD 历史的人来说，教授们在剑桥大学如此广泛地使用它可能并不奇怪。FreeBSD 在学术界有丰富的历史。它源自加利福尼亚大学伯克利分校计算机系统研究小组在 20 世纪 70 年代中期到 90 年代开发的 4.4-Lite 的伯克利软件发行版。在过去的 30 年中，FreeBSD 操作系统继续为学术界提供稳定的基础，可以进行研究，并通过广泛使用的产品和服务的路径，来进行工业采用，这些产品和服务都在运行、基于或合并了 FreeBSD 1。【注 1】

## 内核访问控制

Watson 教授的强制访问控制（MAC）框架研究沿着 Jail 的方向继续，认为操作系统应具有可插拔的组件，以便与许多不同的硬件配合工作，访问控制和安全性也应该是可插拔的。

许多用户在他们的产品中会定制 FreeBSD，添加诸如设备驱动程序和文件系统之类的内容，这通常被称为本地化。Watson 教授的 MAC 研究认为，你可以对操作系统和产品的安全要求进行本地化。这提供了可扩展性，使得构建路由器、手机操作系统或其他使用 FreeBSD 的设备的公司可以根据其所需的环境对操作系统进行调整，不仅包括安全性，还包括硬件、存储和网络调整。

关于这项研究如何过渡到广泛使用，Watson 教授认为这是相当成功的一项研究（我们会称之为保守的说法）。这项研究针对 FreeBSD 被纳入其他系统中，尤其是 iOS。

## CHERI 的工作扩展了 Capsicum 隔离，实现了规模化的细粒度内存保护

CHERI，即 [Capability Hardware Enhanced RISC Instructions](https://www.cl.cam.ac.uk/research/security/ctsrd/cheri/)，通过将架构能力与传统处理器指令集架构（ISA）相结合，实现了细粒度内存保护和高度可扩展的软件隔离。Capsicum 的工作借鉴了 20 世纪 70 年代的一些叫做 capability 系统的想法，并认为它们可以通过与 BSD 中的当前软件进行融合来使其变得现代化。CHERI 的工作将这个概念推得更远，并应用于处理器。Watson 教授表示：“通过 CHERI，我们改变了硬件，也改变了软件。而我们选择改变的软件，以展示这些想法并理解它们，就是 FreeBSD。”

CHERI 关注硬件/软件共设计，需要对软件和硬件进行协调性的变更。两者之间的接口是指令集架构（ISA）。新的 ISA 使处理器的进步可以通过软件来表达。因此，硬件/软件共设计需要对硬件和软件【注 2】进行变更。

在 [BSDCan 2023](https://www.bsdcan.org/events/bsdcan_2023/sessions/session/142/slides/58/20230520-memory-safe-desktop-compressed.pdf) 中，来自 SRI International 的 Brooks Davis 解释道：“CHERI 减少了 C/C++ 可信计算基础（如虚拟化程序、操作系统、语言运行时、浏览器等）的漏洞...”CHERI 被证明非常有效。Microsoft Security Response Center（MSRC）发现，CHERI 在 C/C++ 语言软件【注 3】中减轻了超过三分之二的关键内存安全性漏洞。

FreeBSD 对于这种类型的研究至关重要，部分原因在于 BSD 许可证的宽松性，支持整个硬件/软件堆栈的修改，以及通过 LLVM 的全面集成来构建完整的内核和用户空间。当该项目于 2010 年开始时，FreeBSD 正在积极推进 LLVM 集成，这对于 Linux 来说仍然是一个挑战。同样重要的是，对多种 ABI 的清洁内核支持、Capsicum 安全模型的集成以及对 RISC-V 架构的早期支持。

CHERI 团队在此过程中广泛地为 FreeBSD 做了贡献，改进了 FreeBSD 对 64 位 MIPS 和 Armv8-A 支持，贡献了一个 RISC-V 移植，为更好地支持 Xilinx 和 Intel FPGA 板添加了许多设备驱动程序，以及 Arm IP，并为基本系统和 Ports/软件包开发了 QEMU 用户级和交叉构建支持。

## 获取 CHERI

感兴趣的用户可以下载一个叫做 [CheriBSD](https://www.cheribsd.org/) 的完整 FreeBSD 衍生操作系统。CheriBSD 是一个研究性操作系统，[广泛使用 CHERI 来提高](https://www.morello-project.org/cheri-feature-matrix/) 软件安全性，并展示了对 CHERI 的逐步软件采用路径。它的特点包括内核和用户空间的内存安全性、支持两种软件隔离模型，以及大约一万个内存安全的第三方软件包，包括服务器应用程序，如 nginx、KDE Plasma 桌面环境。今年晚些时候，将会向 CheriBSD 软件包集合中添加一个内存安全的 Chromium 网络浏览器，由 Google 和 InnovateUK 赞助。可以使用可下载的 USB 驱动器映像在 Arm Morello 系统上进行安装。CheriBSD 拥有一个活跃的用户社区，有 70 多家公司和大学参与基于 Morello 的研究项目，几乎所有项目都在运行 CheriBSD，其中最活跃的贡献者是 SRI、剑桥和微软。

>“通过 CHERI，我们改变了硬件和软件。而我们选择改变的软件，以展示这些想法和理解它们，就是 FreeBSD。除了允许私营部门合作的宽松许可证外，FreeBSD 全面的内核和用户空间 LLVM 使用、紧密集成的操作系统设计和构建、对多种 ABI 的清洁内核支持、Capsicum 的集成，以及对 RISC-V 架构的早期支持，使 FreeBSD 非常适合我们的需求。”

有一个可以从 Linux、Mac 或 FreeBSD 进行交叉构建的 CheriBSD 构建系统。Watson 教授和其他从事 CHERI 工作的人积极地向 FreeBSD 贡献了 Linux 和 macOS 的交叉构建支持。Watson 教授认为，这是重要的：“因为我们认为要使 FreeBSD 和我们的工作真正易于访问，我们不能告诉你在你的桌面上运行什么。你必须运行你选择的操作系统，而我们将帮助你在任何你想要的设备上运行 FreeBSD，可能是桌面，也可能是嵌入式设备，或者可能是某个服务器，我们不希望任何障碍阻碍这一点。”

如上所述，[Morello](https://www.arm.com/architecture/cpu/morello) 处理器设计、单片系统（SoC）和主板集成了对 CHERI 保护模型的支持。Morello 是由 Arm 和英国研究和创新（UKRI）共同资助的研究原型，由 Arm 和剑桥大学合作创建，以评估 CHERI 作为主流采用技术的可能性。自 2022 年中期以来，已经发货了 600 多块开发板，主要提供给大学、政府实验室和工业研究实验室。FreeBSD 基金会将在 2023 年晚些时候收到其第一块 Morello 开发板，使这个开发环境可供全球 FreeBSD 社区使用。

要亲身体验 CHERI，请访问 [CHERI 软件分发站点](https://www.cheribsd.org/)，获取最新版本。Docker 镜像适用于 Linux 和其他操作系统。这些镜像包括可让用户在基于 QEMU 的 CheriBSD VM 上运行，用于 CHERI-RISC-V 或 Arm Morello 架构，并使用包含的 SDK 为其交叉编译软件。对于 Arm Morello，可以使用以下镜像：<https://hub.docker.com/r/ctsrd/cheribsd-sdk-qemu-morello-purecap>，在 Docker 容器启动后，可以使用命令启动 CheriBSD【注 4】。学术创新如何从实验室过渡到生产研究的过渡可以采用许多形式。在某些情况下，人们和公司使用软件。有时，过渡意味着这些想法影响了其他人的工作，然后最终产生了结果。

“对于像 MAC 框架或 Jail 这样的东西，很容易看出所有的用例，它在各种各样的事物中都有使用。对于像 CHERI 这样的东西，我们还没有到可以真正说出最终过渡将会是什么的地步，但现在有一个大型的过渡项目正在进行。英国政府和各个公司目前正在为一个为期五年的过渡项目花费约 2.5 亿美元，这非常令人兴奋。”

CHERI 的工作经历了许多的演变过程。CHERI 最初是在 2010 年左右针对 MIPS 架构进行的。现在，所有的工作都在 Arm 和 RISC-V 上进行。RISC-V 是一个开源研究平台，Arm 是当前的过渡目标选择，尽管 Watson 教授迅速补充说，他和其他人也乐意过渡到其他硬件。“网络路由器和交换机、防火墙、手机、平板电脑，都是我们的雄心，实际上。我认为可以想象得更远，服务器和台式机也都在范围内，但那是我们在早期演示中做的事情。”

FreeBSD 基金会代表所有受益于他和合作伙伴安全研究的计算机用户向 Watson 教授表示感谢。这些创新支撑了当代社会的数字基础设施的许多方面，作为一个彻底的开源项目，我们希望你考虑参与到 FreeBSD 中，帮助编写计算机的下一个章节。


- 此页面详细说明了学术界为什么喜欢 FreeBSD：<https://freebsdfoundation.org/our-work/research/>
- 这张幻灯片总结了 CHERI：<https://docs.google.com/presentation/d/1i8hrEKb62a8bCslQUI3wn2ZbYmNQ48i5jzjkgeQNhzY/edit?usp=sharing>。
- <https://msrc.microsoft.com/blog/2020/10/security-analysis-of-cheri-isa/>
- 对于 Morello，使用：`$ /opt/cheri/cheribuild/cheribuild.py run-morello-purecap`。对于 CHERI-RISC-V，前往：<https://hub.docker.com/r/ctsrd/cheribsd-sdk-qemu-riscv64-purecap>，然后使用命令：`$ /opt/cheri/cheribuild/cheribuild.py run-riscv64-purecap`
