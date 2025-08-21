# EuroBSDCon 2022 旅行报告：Muhammad Moinur Rahman

- [EuroBSDCon 2022 Trip Report: Muhammad Moinur Rahman](https://freebsdfoundation.org/blog/eurobsdcon-2022-trip-report-muhammad-moinur-rahman/)
- 2022 年 11 月 14 日
- Muhammad Moinur Rahman

##### 2022 年 11 月 14 日

FreeBSD Foundation 慷慨资助了我参加 EuroBSDCon 的行程，我非常感激能够亲自参加此次活动。EuroBSDCon 2022 是我参加的第一场 BSD 会议，但绝不会是最后一场。我自 2003 年起开始为 Ports 提交补丁，自 2014 年起成为热衷的 Ports 提交者。我维护着 800 多个 Ports，并且最近对 src 数据集产生了浓厚兴趣。我开始与 Foundation 的工作人员合作，将 Pre Commit CI（持续集成）引入 src 数据集。我在 Li-Wen 的指导下进行工作。最近我已为 src 数据集移植了 Cirrus-CI CI 流水线，目前仍在等待审核。参加本次会议的目的是展示这些更新，并获取反馈，以更好地了解开发者对该系统的期望，从而在我们的 CI 系统中加以应用。

我比会议提前一天抵达维也纳，并能够与现任（CORE12）及前任 CORE 团队成员共度一些时间，因为我刚刚卸任 Core-Secretary 职务。CORE 的交接通常在 BSDCAN 的晚宴上进行，但由于线下 BSDCAN 被取消，CORE 团队决定在 EuroBSDCon 上完成交接。此外，这也让我有机会与现任 core-secretary@ 合作并交接一些流程工作。不幸的是，由于 Sergio 因不可预见的情况无法出席，我们未能完成预期安排。但我们最终讨论了 CORE-11 与 CORE-12 的不同过去与现在的任务。

开发者峰会（DevSummit）非常有趣，让我们了解其他开发者的工作内容以及他们的期望。会议重点关注 src/ports/doc 数据集的代码设计与实现。我也在其中一次会议中展示了 CI 流水线的内容。开幕会议包括自我介绍以及与会者分享他们最喜欢 FreeBSD 的部分，这让我感到很有启发性。同时，这也是我与通过邮件和 IRC 沟通过但从未见过面的其他提交者建立联系的机会。

第二天标志着 EuroBSDCon 的正式开始。当天以 Frank Karlitschek 的主题演讲开场，内容重要，但最终偏向于宣传推介。Eirik Øverby 的合规性报告也很有趣，他是 EuroBSDCon 的常客，演讲内容主要是如何仅使用 FreeBSD 及其衍生产品运行支付系统，非常有启发性，尤其是处理不同支付网关要求及证书标准化的案例。随后，我参加了 Brooks Davis 关于如何添加系统调用（syscall）的讲座，这非常吸引我，也激发了我在 src 数据集上工作的兴趣。午餐后，会议重新开始，涵盖了多个有趣主题，我选择了 Mateusz Piotrowski 的 DTrace 和 eBPF 讲座。这次讲座很有意思，他也在进行相关研究，并对 DTrace 与 eBPF 的性能开销进行了概览。随后是 Allan Jude 关于 NVMe 上 ZFS 的讲座，介绍了各种优化和定制方法以获取最佳性能。我将其与我在 Linux 上研究的情况进行了比较。当天最后一场讲座是关于 bhyve 中 GPU 直通，我非常期待，但内容不如预期，讲者主要展示了去年的基准测试，没有近期改进。当天结束于维也纳一座古老建筑的社交活动，建筑内有精美的古典设计与装饰。

最后一天以 Dylan Beattie 关于遗留代码的有趣讲座开场。随后 John Baldwin 展示了如何为 DDB 内核调试器编写自定义命令；作为对 src 数据集也感兴趣的开发者，这是不错的补充。接着 Drew Gallatin 的演讲令人受益匪浅，讲述 Netflix 如何扩展单台服务器以实现 800Gbps 的内容服务速率，每年几乎翻倍扩展，总是很有趣。随后是 Goran Mekić 关于 FreeBSD 低延迟音频处理的讲座。最后，我计划参加 Michael Dexter 关于构建更小且启动更快的 FreeBSD 系统的讲座。其他 BSD 发行版的论文也非常有趣，但由于总有三条平行会场，很难选择；作为 FreeBSD 开发者，我尽量选择与 FreeBSD 相关的讲座。

总而言之，我非常感谢 FreeBSD Foundation 支付我前往 EuroBSDCon 的差旅费用。这不仅是一次极好的学习经历，也让我能够与社区其他成员讨论若干问题。尽管 IRC 和邮件非常有用，但无法与面对面交流的沟通质量相比。通过邮件或 IRC 总会有缺失点，而亲身交流则可以完全沉浸于讨论。我期待未来能继续参加 BSD DevSummit 和各类会议。
