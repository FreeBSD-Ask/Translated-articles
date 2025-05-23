# 2023 年 FreebBSD 谷歌编程之夏学生介绍：Raghav Sharma

- 原文链接：[Meet the 2023 FreeBSD 谷歌编程之夏 Students: Raghav Sharma](https://freebsdfoundation.org/blog/meet-the-2023-freebsd-google-summer-of-code-students-raghav-sharma/)
- 发布日期：2023 年 8 月 24 日
- 译者：ykla & ChatGPT

自 2005 年谷歌编程夏计划伊始，FreeBSD 项目一直参与其中，对此感到自豪。随着 2023 赛季接近尾声，基金会要求我们的一些谷歌编程之夏学生分享更多关于他们自己以及与该项目合作经验的信息。

**问：请向我们简要介绍一下你自己以及你在学业旅程中的阶段？**

我叫 Raghav Sharma，是一名来自印度的本科生，正在攻读电子工程的技术学士学位。我目前正处于 B.Tech【译者注：B.Tech 是技术学士学位的缩写】的最后一年阶段。我对系统编程、内核开发、编译器和后端项目充满兴趣。

**问：你之前有参与过谷歌编程夏项目吗？**

有的。我在 2022 年参与了谷歌编程夏项目，与 Haiku 组织合作，将 XFS 文件系统驱动移植到了 HaikuOS 项目中。

**问：为什么你想要与 FreeBSD 合作？**

FreeBSD 项目对我来说是一个更深入探索系统编程和内核开发的绝佳机会。社区正在开展许多令人惊奇的项目，我希望能够探究这些想法。

**问：请简要介绍一下你的谷歌编程夏项目是什么？**

我的项目目标是将 SquashFS 驱动程序添加到 FreeBSD 内核中。SquashFS 是只读文件系统，可压缩整个文件系统或单个目录，将其写入其他设备/分区或普通文件，然后直接挂载它们。SquashFS 有两个实现，一个是 Linux 内核驱动程序，另一个是 SquashFuse 项目。我使用了这两个实现来将驱动程序移植到我们的 FreeBSD 内核。由于我们几乎即将结束编程之夏项目，我已完成驱动程序的实现。现在我们支持 SquashFS 的 mount(8)、目录、文件和符号链接。

**问：迄今为止，你从这次经历中学到了什么？**

迄今为止我的学习经验包括：

- FreeBSD vfs 层内部，了解内核如何通过良好的 API 管理多个文件系统。大部分时间都花在了探索和理解这些方面上。
- 软件使用的几种压缩技术，如 zlib、zstd 等，以及为何需要压缩。
- 更深入的文件系统理论以及我们可以通过哪些技术来减少磁盘寻道并最大化性能等。
- 了解其他 FreeBSD 内核文件系统的实现细节，如 ext2fs、nullfs、tarfs 等。

**问：与 FreeBSD 项目合作的经历如何？**

到目前为止，非常棒。感谢导师 Chuck Tuffli 在帮助我入门方面的指导，例如设置开发环境，引导我查阅 SquashFS 实现所需的文档等。

（FreeBSD）社区也非常友好和活跃。
