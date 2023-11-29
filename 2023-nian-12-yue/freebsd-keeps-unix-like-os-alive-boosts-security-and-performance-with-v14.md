# &#x20;FreeBSD v14：恪守类 Unix 操作系统传统，提升安全性与性能

- 原文链接：<https://www.sdxcentral.com/articles/analysis/freebsd-keeps-unix-like-os-alive-boosts-security-and-performance-with-v14/2023/11/>
- 作者：Sean Michael Kerner
- 时间：2023 年 11 月 22 日 7:04
- 译者：Canvis-Me & ChatGPT

- - -

![image](https://github.com/Canvis-Me/Translated-articles/assets/55122738/4fb2c326-1d13-49d2-8f45-e53dc6167e3a)


Linux 并不是唯一的开源操作系统（OS）。还有许多类 Unix 的开源 BSD 操作系统，包括仍在积极开发中的 FreeBSD。

FreeBSD 项目于 1993 年首次发布，是最早的开源操作系统项目之一，也是加州大学伯克利分校最初的开源 BSD 工作的直接继承者。本周，FreeBSD 14 正式发布，这是自 2021 年以来该开源操作系统的第一个主要版本号更新。这个在 14 分支上的首个稳定版本为这个开源操作系统带来了重要的更新和新功能。

“FreeBSD 14 代表了一些新功能和对各个子系统的大量更新，以提高性能、稳定性和安全性，”FreeBSD 基金会技术高级总监 Ed Maste 告诉 SDxCentral。

## 在 FreeBSD 14 中，ZFS 文件系统获得了大幅提升。

FreeBSD 14 的新特性之一是将 ZFS 文件系统更新为 OpenZFS 2.2 版本。

ZFS 起源于 Oracle（之前是 Sun Microsystems）的 Solaris UNIX 操作系统，长期以来一直承诺支持非常大的存储容量。Maste 指出，OpenZFS 2.2 包括诸如块克隆之类的增强功能，为需要大量复制的工作负载提供了巨大的性能提升。除了对 OpenZFS 本身的改进，还添加和更新了新的工具，以提高 ZFS 在 FreeBSD 中的集成度。

此外，还进行了更新，使 ZFS 在虚拟化环境中运行得更好，FreeBSD 还将 ZFS 支持添加到 makefs 镜像创建工具中。

“这使得 FreeBSD 发布工程团队可以开始为云提供商构建基于 ZFS 的根镜像，”Maste 说道。

## FreeBSD 14 增强了安全性和可扩展性。

新版本还包括许多安全改进。

Maste 指出，FreeBSD 的 Capsicum 沙盒框架已应用于基本系统中的其他工具。还更新了基本系统中的关键安全包，包括将 OpenSSL 升级到 3.0.12，将 OpenSSH 升级到版本 9.5p1。

性能和可扩展性也在提高，同时增强了对硬件的支持。

“还有一些支持当代硬件的变化，例如将支持的 CPU 数量增加到 1024，”Maste 说。

他补充说，还有一大堆可用性和“外观和完成”改进，如缩短引导时间，解决与某些 UEFI（统一可扩展固件接口）固件实现不兼容性，改进 NVMe 设备错误处理等等。

## 为什么 FreeBSD 仍然重要 - 尤其是对于网络

尽管 Linux 作为开源操作系统可能在许多企业中更广为人知，但 FreeBSD 也绝对不陌生，尤其是在网络系统方面。

“网络仍然是 FreeBSD 的强项，包括 Juniper 在内的许多公司都依赖于 FreeBSD 的网络功能，并贡献代码来改进 FreeBSD 的网络功能，”Maste 说。 “其他在网络安全和性能前沿的公司，如 Netflix，Nvidia Mellanox Metify，Netgate，Chelsio 和 Nginx（F5），也使用并为 FreeBSD 做出贡献。”

Juniper 的 Junos 网络操作系统基于 FreeBSD。Maste 指出，业务友好的 BSD 开源许可证使其对于那些希望在其 OS 与专有软件组合成产品的过程中没有任何负担的公司而言，变得很有吸引力。

“因此，我们看到很多 FreeBSD 作为专门用途的安全、网络和工业控制设备的基础；SaaS 解决方案；以及平台，”他说。

Maste 还指出，FreeBSD 支持几个领先的支付处理网络，包括 DeepStack（现为加利福尼亚银行的一部分），Modirum 等。企业级存储是 FreeBSD 受欢迎的另一个市场，有助于支持 Avere Systems（现为 Microsoft 的一部分），EMC Isilon 和 NetApp 的解决方案。

展望未来，Maste 表示，FreeBSD 核心团队正在开始规划过程，以确定社区的发展重点。FreeBSD 基金会是一个独立的实体，但这些重点将有助于指导基金会的软件开发工作。

“基金会未来一两年的一个重点是改善 FreeBSD 的终端用户开箱即用的体验，”Maste 说。 “如果你是一个想要了解或探索系统编程的人，我们希望 FreeBSD 能够提供一个出色的起点。”
