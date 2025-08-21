# BSDCan 2023 旅行报告：Jake Freeland

- 原文：[BSDCan 2023 Trip Report: Jake Freeland](https://freebsdfoundation.org/blog/bsdcan-2023-trip-report-jake-freeland/)
- 2023 年 5 月 25 日
- 作者：Jake Freeland

我心怀感恩地作报告，FreeBSD 基金会资助我前往加拿大渥太华参加了 2023 年 FreeBSD 开发者峰会和 BSDCan 大会的旅行。我登上航班，在 5 月 16 日深夜抵达 YOW 机场（Ottawa International Airport，加拿大渥太华国际机场）。我乘坐了 OCTranspo（渥太华卡尔顿交通局）的 97 路公交车到 Hurdman 车站，在那里换乘 Hurdman East 轻轨前往渥太华大学。我事先安排好与 Mark Johnston 合住一间双人套房，于是我前往 U90 Residences（学生宿舍楼）找到我的房间。到达之后，设好第二天早上的闹钟，我很快就睡着了。

第二天早晨醒来后，我走在去往峰会的路上，竟然遇到了 Kirk McKusick。我读过 Kirk 的《FreeBSD 操作系统设计与实现》一书，并把他视作计算机领域的传奇人物，因此我非常渴望与他交谈。Kirk 为人友好，在整个行程中我们彼此都更加了解。

开发者峰会的第一天令人兴奋，但也令人疲惫。基金会首先登台，介绍了后勤和核心方面的更新。我很喜欢 Mark Johnston 的引导式代码阅读，他讲解了构成 ULE 调度器的内部机制。Mark 很好地总结了调度器，并以交互的方式带领大家阅读代码。那天晚上，很多人聚在黑客沙龙写代码。Warner Losh 和 Mark Johnston 很好心地帮助我调试 FreeBSD 的 timerfd 实现。

开发者峰会的第二天，也是最后一天，可能是我整个行程中最喜欢的一天。我很享受围绕 FreeBSD 14.0 和 15.0 的讨论与规划。看到即将发布的版本中会加入哪些功能、又会删除哪些功能，尤其令人兴奋。一个值得注意的讨论主题是废弃对 32 位的支持，最终的结论是 32 位很可能会在 15.0 中被淘汰。主会议结束后，我与 Kirk McKusick、Mike Karels 以及 Pierre Pronchery 一起走到 Father & Sons 餐厅。我们四个人一边吃饭一边谈论旅行，Kirk 还给我们讲述了一些他在开发 UFS 时最疯狂的故事。

第二天是 BSDCan 的第一天。我参加了五场演讲，但我最喜欢的是 Colin Percival 的《Porting FreeBSD to Firecracker》以及 Kirk McKusick 的《Gunion(8): a new GEOM utility in the FreeBSD Kernel》。我尤其喜欢 Percival 专注于最小化 FreeBSD 启动时间的部分，他成功地将进入用户态前的时间缩短到了 33.382 毫秒。Kirk 的演讲则提供了一个直观的 FreeBSD I/O 模型概述，同时也起到了复习的作用。

BSDCan 的最后一天有鐘凡的《VT-IME: Input Method Editor in FreeBSD vt(4)》演讲，他讲述了如何在 FreeBSD 虚拟终端中加入多字节符号支持。这对那些希望在 tty 中输入中文、日文或韩文的人极有用。鐘凡的演示非常精彩，他的工作也令人印象深刻。Warner Losh 的《kboot: Booting FreeBSD with LinuxBoot》可能是我参加的最技术性的演讲，但也是最吸引人的。Warner 指出了 UEFI 的许多设计缺陷，并提出 LinuxBoot 可能会是未来更好的预启动环境。他展示的 FreeBSD LinuxBoot 吉祥物幻灯片也非常有趣。

最后，我与 Kirk、Mike 一起走到 BSDCan 的参加派对后的聚会。我们三人与 Warner Losh、John Baldwin、Brooks Davis 以及 Pawel Dawidek 一起吃饭。大家讨论了 FreeBSD 的未来以及如何吸引更年轻的群体。放弃 IRC 以及为 Framework 笔记本提供预装的 FreeBSD 镜像是其中最受欢迎的话题。

第二天早晨，我在返回 YOW 机场的路上遇到了 Joseph Mingrone。Joe 是推荐我申请基金会此次旅行资助和实习的人，因此我对他充满感激。我们两人谈论了各自的旅行经历，并在机场登机口一起坐着，直到我登上航班。

参加开发者峰会和 BSDCan 让我与 BSD 社区中一些最具影响力的人建立了联系。我很感谢 FreeBSD 基金会提供了这次机会。我计划以后能再参加一次 BSD 会议，并且希望到那时我已能全职从事 BSD 相关的工作。
