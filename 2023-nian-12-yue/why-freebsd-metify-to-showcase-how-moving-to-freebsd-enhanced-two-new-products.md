# 为什么选择 FreeBSD？Metify 展示迁移到 FreeBSD 如何对两个新产品进行增强

- 原文链接：<https://freebsdfoundation.org/blog/why-freebsd-metify-to-showcase-how-moving-to-freebsd-enhanced-two-new-products/>
- 译者：Canvis-Me & ChatGPT

2023 年 11 月 2 日

FreeBSD 厂商峰会始终是一个绝佳的机会，让人更深入地了解公司是如何以及为何使用 FreeBSD。Metify，这家背后是屡获殊荣的 Mojo 平台的公司，将在今年的峰会上展示两款创新产品，为大家提供这样的机会。

Metify Mojo 是一个虚拟设备，可以从所有地方被发现、配置和维护服务器。Metify 的首席技术官 Ian Evans 将讨论 FreeBSD 在创建 Mojo 中的关键角色。

![Ian Evans, CTO, Metify](https://github.com/Canvis-Me/Translated-articles/assets/55122738/e2fec470-18e6-41a0-acad-393237ca38c4)

Metify 提供两个产品：

- Mojo Edge 数据中心配置——平台 - 这是一个综合解决方案，旨在在数据中心和边缘发现、配置和配置裸机服务器和存储。该解决方案注重保持用户体验简洁，避免与供应商绑定。该工具与 Metify 的 Photon ISP 平台配合使用，无缝管理分布在不同位置的裸机资产。该平台完全依赖于 BSD 和 bhyve，在 Oracle Cloud Infrastructure (OCI) 成熟后，平台将立即过渡到 BSD 本机。

- Photon ISP 平台——作为对无线 ISP 服务的补充层，确保了不间断的 4G/5G 和 Starlink 故障切换。基于强大的 FreeBSD 13.2 构建，它完全取代了 Starlink 的路由器功能。这一创新将过渡到目前为我们的弗吉尼亚州布鲁蒙特客户提供服务的 Photon WISP 平台。

“FreeBSD 是 Metify 堆栈的基石，帮助我们推动 Mojo 的创新。我们从 Linux 切换到 FreeBSD，是因为其许可的灵活性和无与伦比的性能能力，使我们能够一次性配置数千个数据中心节点。此外，它显着简化了我们的软件架构，同时提高了系统的稳定性，”Metify 的 CTO Ian Evans 评论道。

在 FreeBSD 厂商峰会上，Evans 将讨论：

- 从 Linux 过渡到 FreeBSD：Metify 从 Linux 过渡到 FreeBSD 以及从 KVM 到 Bhyve 的方式和原因。他将讨论 Metify 如何将 Bhyve 整合到其系统中。
- 用 FreeBSD 简化的价值：通过 FreeBSD，Metify 简化了其软件堆栈并提高了可靠性。它将讨论如何构建零触及的 FreeBSD 安装流程，并使用 Ansible 自动化其堆栈。
- 灵活性和创新：Metify 将讨论 FreeBSD 许可结构如何在未来允许更大的灵活性（特别是与 Linux 相比）。它还将讨论 Metify 如何使用 FreeBSD 开发设备，用于通过 4G 和 5G 网络进行远程设置，以及基于 FreeBSD 的边缘设备，通过 4G 和 5G 网络实现远程裸机配置。
- 增强的安全性：Metify 还将分享如何将 FreeBSD 用作 Mojo 的安全网关。这将包括围绕 pf、ipfw、dummynet 和 NTOPNG 的流量监控的讨论。

你可以在峰会上 [亲自](https://freebsdfoundation.org/news-and-events/event-calendar/november-2023-freebsd-vendor-summit/) 听到 Ian 的演讲，也可以通过 [直播](https://youtube.com/live/k-AzShVdAHo) 在线观看。
