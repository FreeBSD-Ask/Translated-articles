# 认识 2023 年参与 FreeBSD 谷歌编程之夏项目的学生：Soobin Rho

- 原链接：[Meet the 2023 FreeBSD谷歌编程之夏Students: Soobin Rho](https://freebsdfoundation.org/blog/meet-the-2023-freebsd-google-summer-of-code-students-soobin-rho/)
- 发布日期：2023-8-24
- 译者：ykla & ChatGPT

自从 2005 年谷歌编程之夏项目创立以来，FreeBSD 项目很自豪地参与其中。随着 2023 年季节即将结束，基金会请几位我们的学生分享更多关于他们自己以及他们与该项目合作经验的信息。

**问：请简单介绍一下你自己，以及你在教育旅程中的进展。**

我是 Soobin Rho，这是我在南达科他州的奥古斯塔纳大学的第三个学年，预计于 2025 年毕业。我主修哲学、数学和计算机科学。

我出生于韩国，在迪拜长大，几年前来到美国上大学。我的目标是白天在一家有使命感的优秀公司工作，然后在空闲时间做我真正关心的事情，比如为环境和可持续性开展开源项目。

**问：你以前有参与过谷歌编程之夏项目吗？**

没有，2023 谷歌编程之夏是我第一次参与。我是在 Hacker News 上第一次听说谷歌编程之夏的。其中一条评论提到 FreeBSD 是参与的开源项目之一。所以，我就申请了。我从 FreeBSD 维基的谷歌编程之夏项目创意列表中选择了我的项目。

**问：你为什么想与 FreeBSD 合作？**

我尝试过很多操作系统（虽然我不会逐一列举）也尝试过很多桌面环境。然而最终，至少目前来说，我决定我最喜欢的编码环境是没有桌面环境，只有终端的 FreeBSD 加上 vim 和 tmux。

当然，每当我需要使用图形界面时，我会不时地回到其他操作系统。尽管如此，它就是感觉对了。我所有的开发工作都是通过 FreeBSD 进行的。加入 2023 谷歌编程之夏是我第一次为 FreeBSD 做贡献，但我也打算继续作为贡献者，并继续维护 mfsBSD 集成。

**问：请简要介绍一下你的谷歌编程之夏项目。**

说到 mfsBSD，这是由 Matuska 于 2007 年创建的，mfs 代表内存文件系统。基本上，一旦你在系统中安装了 mfsBSD 并引导到它，整个操作系统现在都在内存文件系统下运行。这意味着你现在可以做很多事情，比如恢复操作和系统诊断。特别是，我对 mfsBSD 的最喜欢用途之一是用于我的一台只有一块硬盘的笔记本电脑。我可以在硬盘中引导 mfsBSD，然后在我安装了 mfsBSD 的特定的硬盘上安装 FreeBSD。由于 mfsBSD 中的每个文件都被移动并加载到内存文件系统中，甚至可以删除原始磁盘，并使用 mfsBSD 中的 `bsdinstall` 完全覆盖为全新的 FreeBSD 实例。

我的项目将 mfsBSD 集成到 FreeBSD 发布工具集中，这样 mfsBSD 镜像（.img 用于光盘，.iso 用于光盘）现在将在 FreeBSD 主页上提供。【注 1】

**问：到目前为止，你从这个经验中学到了什么？**

在接手这个项目之前，我对 MAKE(1) 一无所知。阅读`/usr/src/Makefile`和`/usr/src/release/Makefile`对我帮助很大。此外，阅读`/usr/src/release/Makefile.vm`非常有价值，事实上，我很多的代码都是基于这些 make 文件的。

**问：与 FreeBSD 项目合作的经历如何？**

有趣而令人兴奋！每周与我的导师 Joseph Mingrone 和 Juraj Lutter 进行会议很有趣。看到我的代码生成了我自己可以使用的 mfsBSD 镜像，以及为所有需要的其他人生成的镜像，这让我感到兴奋。


<https://wiki.freebsd.org/SummerOfCode2023Projects/IntegrateMfsBSDIntoTheReleaseBuildingTools>
