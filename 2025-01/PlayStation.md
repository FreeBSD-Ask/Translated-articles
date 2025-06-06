# 从 PlayStation 到路由器，你很可能一直在使用 FreeBSD 而不自知

- 原文：[From PlayStation to routers, you've probably been using FreeBSD without knowing it](https://www.theregister.com/2025/04/28/freebsd_foundation_25/)
- 作者：iconLiam Proven
- 2025 年 4 月 28 日

先有操作系统，后建基金会——那么基金会到底在做什么？

**采访过** 许多开源自由软件项目由非营利基金会支持。FreeBSD 基金会就是其中一个例子，FreeBSD 基金会由 Meta 软件工程师 Justin T Gibbs 创立。他在接受 *The Register* 采访时谈到了 FreeBSD 的版权理念、基金会的职能以及其重要性。

**问：让我们从 FreeBSD 的历史说起，它起源于哪里，以及与早期 BSD 发行版的关系。**

**答：** FreeBSD 的起源可以追溯到加州大学伯克利分校的计算机系统研究小组（CSRG），伯克利软件发行版（BSD）正是在 1970 年代作为该校的一个协作项目开发的，推动了开源运动的形成。到了 1990 年代初，BSD 遭遇了法律挑战，Linus Torvalds 后来也提到这正是他创建 Linux 的关键原因之一。那个时期，386BSD 出现，旨在将 BSD 引入 Intel 80386 处理器。FreeBSD 于 1993 年 6 月 19 日正式成立，作为 386BSD 的演进，重点关注性能、稳定性以及开放开发模式。

在早期，FreeBSD 不仅因其技术卓越而获得认可，还因为其独特的吉祥物——[Beastie，BSD 小恶魔](https://www.freebsd.org/art/#bsd-daemon)——而闻名，该形象经过皮克斯著名动画师 John Lasseter 的参与进行了精心完善。

从伯克利的起源到今天，FreeBSD 始终致力于打造所有人都能使用和学习的软件。正如 FreeBSD 最早的开发者之一 Marshall Kirk McKusick 博士所说：“FreeBSD 是‘复制中心（Copy-Center）’，而不是‘著佐权（Copy-Left）’——你可以去复制中心，根据自己的需要制作任意数量的副本，目的随意。”这种理念在 30 多年后依然成立。

FreeBSD 与类似的开源项目不同之处，还在于它能够在没有核心人物领导的情况下坚持和创新。根据设计，其治理结构不断演进，既引入新视角，又保持项目本身的稳定和稳步发展。这个项目让我这样的年轻大学生能够立刻参与贡献，学习对职业生涯至关重要的技能，最终加入核心团队，协助管理项目。我于 2000 年 3 月创立了 [FreeBSD 基金会](https://freebsdfoundation.org/)，因为我想确保 FreeBSD 的长期发展，使其能够为未来一代提供同样的机会。基金会提供财务和法律支持，推动基础设施改进，并支持 FreeBSD 的推广和开发。

**问：自那以后，FreeBSD 最大的成功是什么？**

**答：** FreeBSD 最大的成功之一是其作为“催化剂”的角色，为无数产品和创新提供了稳定且高性能的基础。得益于其宽松的 BSD 许可证，公司和开发者能够在 FreeBSD 之上进行开发，而无需受限于严格的义务，这使得它被广泛应用于从 PlayStation 游戏机到网络基础设施和云服务等各个领域。FreeBSD 的影响力不仅限于直接采用——它还为存储、安全和网络领域的技术进步提供了技术支撑。

安全是创新的关键领域，其中包括 [Jail](https://docs.freebsd.org/en/books/handbook/jails/) 功能，这种轻量级操作系统级虚拟化技术早于 Linux 容器成为主流，以及 [Capsicum](https://wiki.freebsd.org/Capsicum)，这是一个细粒度的基于能力的安全框架，用于现代应用的沙箱保护。此外，英国剑桥大学的研究人员在 FreeBSD 上开发了 [CHERI](https://www.theregister.com/Tag/CHERI/)（能力硬件增强 RISC 指令集），它通过硬件支持的内存安全防止处理器级别的漏洞。

自然，围绕 FreeBSD 的关注大多集中在其技术创新上，而 FreeBSD 基金会则默默保障着使这些创新成为可能的稳定性和持续性。除了资助引人注目的项目，比如笔记本上的现代 WiFi 支持，基金会还负责关键但不显眼的工作：维护基础设施，管理安全披露，保护商标权，防范法律和监管风险。这些努力确保 FreeBSD 始终是安全且高质量的平台。当遇到时间紧迫或复杂的挑战——如安全审计或紧急漏洞修复——超出志愿者能力范围时，基金会会介入提供支持。正是这背后的工作，使得 FreeBSD 在全球开发者和组织中保持稳定可靠。

**问：FreeBSD 现在的影响力如何？**

**答：** FreeBSD 依然具有重大影响力，既作为一款完整的操作系统存在，也通过其广泛使用的组件发挥作用。其成功基于三大支柱：宽松的许可证、自我更新的项目结构，以及高度可靠且技术先进的软件。正因为如此，FreeBSD 被嵌入到无数技术中——驱动手机、游戏机、处理全球大部分互联网流量的路由器，以及工业控制系统。很可能你正在使用 FreeBSD 却未曾察觉。

例如，奈飞依赖 FreeBSD 来优化流媒体性能，向数百万用户传输高质量视频且延迟极低——单台服务器的速度可达 800 Gbps。除了娱乐领域，FreeBSD 的稳定性和先进的网络功能使其成为关键基础设施的首选。数据管理全球领导者 NetApp 以 FreeBSD 作为其 ONTAP 软件的基础，该软件支持多个行业——他们特别强调了与阿斯顿·马丁一级方程式车队的合作案例。此外，另一家基于 FreeBSD 静悄悄发展的市值数十亿美元的公司是倍福自动化。仔细查看其网站，可以看到他们如何将平台从老旧的 Windows CE 迁移到 FreeBSD。倍福高级产品经理 Heiko Wilke 表示：“从安全角度来看，一款持续活跃开发的现代操作系统是不可或缺的。”这些案例凸显了 FreeBSD 作为关键任务和高性能应用基础的多样性。

这仅是部分例子。FreeBSD 基金会不断发现该操作系统新的应用场景，并致力于将企业与开发者社区连接起来，推动更多创新。FreeBSD 支撑着许多当今技术的发展，因为它提供了一个强大且灵活的平台——正如最初在计算机系统研究小组创建的 BSD 一样。

**问：FreeBSD 的程序开发社区是否出现了老龄化？如果没有，是如何避免这个问题的？**

**答：** 尽管有一批经验丰富的核心开发者，FreeBSD 依然不断吸引新的年轻贡献者。作为一款完整的操作系统——包括内核和用户空间——它是学习操作系统工作原理的绝佳工具。通过 FreeBSD 基金会与滑铁卢大学的实习合作、开发用于教授操作系统基础的教育材料，以及 FreeBSD 项目 20 年来参与谷歌编程之夏，我们积极吸纳新贡献者。

主权技术机构（Sovereign Tech Agency）专注于投资开放数字基础设施以打造未来基础，[最近委托](https://www.theregister.com/2024/10/01/freebsd_and_samba_funding/) 基金会推动基础设施、安全、合规和开发者体验的改进，重点简化用户首次为项目贡献代码的过程。

最后，FreeBSD 的项目结构使新贡献者能轻松找到自己的定位并产生影响。尽管业界普遍认为系统编程问题已被解决，但仍有重要且令人兴奋的工作待完成。FreeBSD 提供了理想的创新环境。通过平衡经验与新视角，FreeBSD 和 FreeBSD 基金会确保开发者社区保持活力并持续进化。®
