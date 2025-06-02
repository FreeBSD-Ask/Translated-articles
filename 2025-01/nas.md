# FreeBSD 爱好者团结起来支持新兴项目 zVault——社区分支继 TrueNAS CORE 之后继续发展

- 原文：[FreeBSD fans rally round zVault upstart](https://www.theregister.com/2025/05/12/second_preview_zvault/)
- 作者：iconLiam Proven
- 2025 年 5 月 12 日

TrueNAS 依然活跃健康，但 iXsystems 已将重点转向基于 Linux 的 SCALE 版本。对于仍然坚持使用 CORE 的 FreeBSD 忠实用户来说，新的竞争者正在崛起：zVault。

[zVault 项目](https://www.zvault.io/) 是一款基于 FreeBSD 13.3 的全自由开源操作系统，专为网络附加存储（NAS）服务器设计。该项目刚刚发布了 [第二个预览版本](https://github.com/zvaultio/Community/releases/tag/zVault-13.3-MASTER-202505042329-ca844f8808)，解决了首个预览版本中存在的一些知识产权问题。目前这仍是预览版本，我们绝对不建议将任何重要数据存储在上面，但这是一个令人鼓舞的信号。

正如我们 [去年三月报道](https://www.theregister.com/2024/03/18/truenas_abandons_freebsd/) 的，iXsystems 已将其基于 FreeBSD 的 NAS 操作系统 TrueNAS CORE 转入维护模式。其继任产品是 [TrueNAS SCALE](https://www.truenas.com/truenas-scale/)，该版本以 Debian Linux 作为底层操作系统，而非 FreeBSD。公司强调，“没有人会因为转向 Debian 而被‘遗弃’”，这是它对 *The Register* 旗下 *Blocks & Files* 网站的说法。事实上，借助 ZFS 的 [启动环境](https://wiki.freebsd.org/BootEnvironments)——完整且可启动的操作系统快照——甚至可以在原地将 FreeBSD 版本迁移到新的 Linux 版本。

但如果你不想在存储服务器上运行 Docker 容器，或者不想将存储集成到 Kubernetes 集群中，那么好消息来了：zVault 已经接手了 TrueNAS CORE 的最终 FreeBSD 版本。

zVault 在二月底发布了 [第一个预览版本](https://github.com/zvaultio/Community/releases/tag/ZVault-13.3-MASTER-202502240515-8d9f8c8cc2)，但 iXsystems 的 Kris Moore 针对此版本 [提出了问题](https://github.com/zvaultio/Community/issues/7)，声称其中包含了 iXsystems 的知识产权，随后 zVault 项目撤回了该 ISO 镜像。现在，zVault 团队正在努力剥离 iXsystems 的残留部分，移除所有剩余的专有文件，力求打造纯自由开源版本。

FreeBSD 博主 Vermaden 发布了更新，表达了一些 FreeBSD 社区成员对 iXsystems 此举的看法，标题为“[TrueNAS CORE 已死——zVault 万岁](https://vermaden.wordpress.com/2024/04/20/truenas-core-versus-truenas-scale/#truenas-core-dead-long-live-zvault)”。这篇更新是对他们去年发布的文章“[TrueNAS CORE 与 TrueNAS SCALE](https://vermaden.wordpress.com/2024/04/20/truenas-core-versus-truenas-scale/)”的补充。文章中附带了一张截图，显示第一个版本中有超过 50 个文件涉及了 TrueNAS 企业许可协议的内容，[截图链接](https://vermaden.wordpress.com/wp-content/uploads/2024/04/zvault-truenas-core-proprietary-files.png)。

zVault 项目主页上的 [路线图](https://zvault.io/#roadmap) 相对简单。第一步：发布一个可用版本；第二步：发布稳定版本；第三步是更新到 13.5 版本；第四步则是在此基础上切换到当前的 FreeBSD 14。

与此同时，还有 [XigmaNAS](https://xigmanas.com/xnaswp/)，它基于早期的 FreeNAS 版本继续开发，早于 iXsystems 的介入，没有后者投入的现代化改进。它的部署比 TrueNAS CORE 要复杂些，网页界面也明显不如 TrueNAS CORE 精致，但它依然存在，能用，并且持续获得更新。 ®

