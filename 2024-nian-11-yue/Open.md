# 在动荡的开源世界中保持稳定：FreeBSD 的持久稳定性

- 原文链接：[Steady in a shifting Open Source world: FreeBSD’s enduring stability](https://opensource.net/freebsd-steady-shifting-open-source-world/)
- 作者：Jason Perlow
- 原文发布时间：2024 年 9 月 17 日


在不断发展的开源软件世界中，稳定性和可预测性往往是稀缺资源。最近，红帽企业版 Linux（RHEL）生态系统的变化，包括限制访问源代码，激起了社区的反应，并导致了开放企业 Linux 协会（OpenELA）的成立。

在这场动荡之中，自 1993 年以降，[FreeBSD](https://www.freebsd.org/) 始终作为一致性和可靠性的典范脱颖而出。本文将探讨 FreeBSD 的稳定开发模型与即将变化的发布计划，如何与 Linux 社区面临的挑战形成鲜明对比。

## FreeBSD：稳定与创新的传承

FreeBSD 是一款开创性的开源操作系统，继承了由加州大学伯克利分校计算机系统研究小组（CSRG）在 1970 和 1980 年代开创的原始伯克利软件分发（BSD）遗产。同模块化和碎片化的 Linux 发行版不同，FreeBSD 采取整体方法进行系统开发，提供了包含内核、用户空间、实用程序、库和文档的统一软件包。这一原始概念为向最终用户交付完整的开源操作系统设定了标准。

## RHEL 和 CentOS 现状简述

红帽企业版 Linux（RHEL）长期以来一直是企业级 Linux 部署的基石。2023 年 6 月，[红帽决定限制访问 RHEL 的源代码](https://www.redhat.com/en/blog/furthering-evolution-centos-stream)，这一决定显著影响了下游项目，如流行的免费开源 RHEL 克隆项目 [CentOS](https://www.centos.org/)。根据他们对 [GPLv2](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html) 和 [GPLv3](https://www.gnu.org/licenses/gpl-3.0.en.html) 的解读，红帽通过按要求提供源代码履行了义务，但并不一定以易读和可复现的格式提供。作为回应，[开放企业 Linux 协会（OpenELA）](https://openela.org/) 成立，旨在提供开放和免费的企业 Linux 源代码。OpenELA 已经自动化其流程，使得每次 RHEL 新版本发布后的几天内，新版企业 Linux 源代码便能提供，确保其他发行版可以继续基于当前的 RHEL 代码构建。

## 开源操作系统开发的集中化与去中心化方法

在比较开源软件时，FreeBSD 和 Linux 有着显著的不同。Linux 提供数百个适应特定需求、偏好和哲学的发行版。这种去中心化方法允许高度定制和专业化，但也可能引发碎片化和不一致性。而 FreeBSD 则独树一帜，它是一款单一的、完整的且完全可定制的操作系统。其集中化的开发模型确保了统一的软件包，能提供系统一致性和可靠性，并促进了更统一、稳定的环境。

去中心化方法（如 Linux 发行版）与集中化方法（如 FreeBSD）并没有固有的优劣之分。每种方法都有其优劣。然而，最近的 RHEL/CentOS 代码可用性事件突显了当供应商仅按照其对版权和 GPL 的解释履行义务时，可能对社区利益产生的影响，与从社区最佳利益出发的做法之间的对比。

## FreeBSD 的开发方式

FreeBSD 项目的组织稳定性是其关键区别之一，自 1993 年以来持续开发着。FreeBSD 是许多企业设备（如 Juniper 的交换机和 NetApp 的 NAS 单元）、消费电子、网络与安全解决方案以及像 [Netflix 的 OpenConnect CDN](https://en.wikipedia.org/wiki/Open_Connect) 这样的高流量内容分发系统的基石。这一长期稳定的组织和社区为 FreeBSD 的开发工作提供了可靠性和持续的创新。

受到原始[伯克利软件分发](https://en.wikipedia.org/wiki/Berkeley_Software_Distribution)启发，FreeBSD 的开发方法采用了集中式模型，这与 Linux 的高度分布式开发生态系统有着很大的不同。与第三方 Linux 发行版将内核与许多外部项目的软件结合不同，FreeBSD 所有系统组件和文档均由单一项目框架内的团队进行开发。这一方法确保了系统的一致性和安全性，突显了 FreeBSD 对统一操作系统的坚持。

## 对 BSD 许可证的深厚承诺

FreeBSD 的核心理念是坚守 BSD 许可证，这反映了项目在自由与开放创新方面的基本原则。这一宽松的许可证几乎不限制软件的使用、修改和分发。FreeBSD 力求减少 GPL 许可证组件的使用，展示了其致力于保持尽可能开放和自由的基础系统，促进创新和协作的环境。

[BSD 2-Clause 许可证](https://opensource.org/license/bsd-2-clause)推动着 FreeBSD 的开发和采用。尽管它对使用 BSD 许可证代码的开发者要求较少（并且较为简洁，仅 200 余字，而 [GPLv2](http://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html) 长达近 3000 字），FreeBSD 项目由于其领导和结构方式，仍显得更加利他。与像红帽这样的公司不同，红帽虽有贡献于开源项目，但它是以商业利益为驱动，FreeBSD 则来自于其自身的源头，作为一款统一体开发内核和操作系统。这一集中式开发方法确保了所有组件的紧密集成，带来更大的一致性和可靠性。

## FreeBSD 基金会与 FreeBSD 项目的角色

[FreeBSD 基金会](https://freebsdfoundation.org/) 是一家非营利性 501(c)(3) 组织，旨在支持 FreeBSD 项目。该项目的目标是开发一款 BSD 许可证下的开源操作系统。由于项目的组织方式以及它们各自的目的，未来的代码可用性不成问题，项目对开源的承诺也无可置疑。FreeBSD 基金会提供资金和支持，以确保 FreeBSD 继续作为顶级开源操作系统存在。该组织结构强化了 FreeBSD 对开源原则的承诺，确保其持续开发和可访问性。

## 安全的构建环境

FreeBSD 的[持续集成构建环境](https://ci.freebsd.org/)以安全为基础原则进行设计。通过隔离构建环境、最小化外部依赖、严格验证源代码并确保一致的构建过程，FreeBSD 有效地减少了与更复杂和控制较少的构建环境相关的风险。这些设计原则共同提升了系统的完整性和安全性，确保了安全和受控的构建过程，减少了漏洞。

## FreeBSD 的新季度和双年度发布计划

为了提升用户体验、改善系统安全性并简化维护，FreeBSD 更新了其[发布计划](https://freebsdfoundation.org/blog/navigating-freebsds-new-quarterly-and-biennial-release-schedule/)。用户现在可以期望每个季度发布一个新的次要版本，每两年发布一次主要的 `.0` 版本。从 FreeBSD 15.x 开始，稳定分支将支持四年，较之前的五年有所缩短。这一新计划确保了持续的更新流动，实现在维持稳定性的同时应对新出现的安全威胁的实际平衡。

## 上游贡献的广泛影响

FreeBSD 的开发模型秉承了伯克利 BSD 的协作精神。社区的[上游贡献](https://docs.freebsd.org/en/articles/contributing/)对整个系统产生了重要影响，从内核到实用程序和文档。这种集中式的开发意味着单一贡献就能改善整个系统的性能、安全性和可用性，产生连锁反应，每一次提升都对项目的整体健康和发展做出贡献。

## 未来方向与如何参与

展望未来，FreeBSD 将继续适应和发展。转向双年度发布周期以及引入四年支持期限是确保项目继续与时俱进、响应用户需求的重要步骤。这些变化，加上 FreeBSD 已经建立的稳定性，使其在开源生态系统中继续成功发展的前景广阔。

对于有兴趣参与 FreeBSD 的新成员，有很多方式可以参与。无论是担任导师、推广 FreeBSD，还是参与论坛和邮件列表，您的努力都会推动项目的创新和发展。今天就支持 FreeBSD 项目，加入其充满活力的社区，帮助构建这个长期发展的开源生态系统。您可以通过[改进文档](https://docs.freebsd.org/en/books/fdp-primer/)、处理[错误报告](https://www.freebsd.org/support/bugreports/)、[提交代码](https://wiki.freebsd.org/BecomingACommitter)和[参与讨论](https://www.freebsd.org/community/mailinglists/)来帮助提升 FreeBSD。每份贡献，无论大小，都会帮助 FreeBSD 成为一款更稳定、安全和高效的开源操作系统。

## 结论

FreeBSD 在开源世界经历重大变化与挑战的时期，依然屹立不倒，成为稳定性和可靠性的灯塔。其集中化的开发模型、宽松的许可证以及对一致性发布的承诺，提供了引人注目的替代方案，展现了与其他项目中动荡局面相对立的优势。随着 FreeBSD 开启加速发布计划，它继续展示着稳定且可预测的开源操作系统的持久价值。

---

Jason 是一位拥有超过 25 年开源经验的作家和内容制作人，专注于分布式系统和企业架构等技术话题。曾担任 Linux 基金会的编辑总监，目前是 FreeBSD 基金会的高级撰稿人，亦为 ZDNet 的高级特约撰稿人。他的作品遍布于各大平台，涉及人工智能、云计算、网络安全和 Linux 等领域。
