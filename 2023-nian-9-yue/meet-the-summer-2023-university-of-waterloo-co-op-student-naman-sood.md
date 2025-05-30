# 认识 2023 年夏季滑铁卢大学合作学生：Naman Sood

- 原文链接：[Meet The Summer 2023 University of Waterloo Co-Op Student: Naman Sood](https://freebsdfoundation.org/blog/meet-the-summer-2023-university-of-waterloo-co-op-student-naman-sood/)
- 发布日期：2023.9.7
- 译者：ykla & ChatGPT

FreeBSD 基金会继续与滑铁卢大学合作项目成功合作。自 2017 年以来，我们已经招募了 15 名实习生，其中一些人已经回来做过多次实习。我们还有两名实习生成为了正式的贡献者，许多人继续为项目做出贡献。今年夏季合作学生生是 Naman Sood，我们坐下来与他聊了聊，了解他以及他选择为 FreeBSD 工作的原因。

**姓名：** Naman Sood

**问：请简要介绍一下自己以及你的教育经历。**

我来自印度，2019 年来到加拿大就读滑铁卢大学，主修计算机科学。目前我已经读到了第四年的一半，计划于 2024 年春季毕业。

**问：你已经参加了多少次合作实习？**

FreeBSD 是我第五次的合作实习。

**问：为什么你想要在 FreeBSD 基金会工作？**

我一直喜欢编写与计算机硬件的内部机制有关且为其他程序提供良好接口的底层代码。FreeBSD 基金会为我提供了一个特殊的机会，可以实际参与到全世界的人们使用的实际系统的开发中，而这种工作正是我所热衷的。此外，在开源生态系统中工作的经历对我来说是一个难得的学习机会，使我更加自如地为我每天使用的软件做出贡献。

**问：你希望从这个实习中学到什么？**

我最想从这个实习中获得的经验是在一个真正的、可用于生产的操作系统内核上进行一些“hack”活动的经验。我所在大学的操作系统课程非常出色，但更多地是对整体概念的引导。它没有涉及到我认为使在底层级别工作变得有趣的许多复杂细节，而 FreeBSD 则为我提供了在这种环境下工作的实践和经验。

**问：你目前在做什么工作？**

我目前正在尝试将 FreeBSD 的 VPS 子系统（<http://www.7he.at/freebsd/vps/>）从 FreeBSD 10 移植到 14，具体来说，我正在考虑将 TCP 检查点和跨进程故障转移进行移植。例如，如果你有一个多进程的 Web 服务器，其中一个进程崩溃，它可以将其 TCP 连接移交给另一个进程。从客户端的角度来看，即使面对此故障，也可以继续正常运行。

**问：到目前为止，你对 FreeBSD 的经验如何？**

非常好！在这个大型的、由志愿者推动的开源软件项目中，自然会出现技术和社交方面的挑战。但这些挑战正是我来这里应对的，我在我的工作中得到了非常有益和愉快的经验。
