# FreeBSD 基金会 2023 年暑期实习生：Jake Freeland

- 原标题：[Meet FreeBSD Foundation 2023 Summer Intern: Jake Freeland](https://freebsdfoundation.org/blog/meet-freebsd-foundation-2023-summer-intern-jake-freeland/)
- 最后发布于 2023 年 9 月 8 日
- 译者：ykla & ChatGPT

向学生们介绍 FreeBSD 仍然是 FreeBSD 基金会的重要任务。今年，除了参与谷歌编程之夏和滑铁卢大学合作计划外，基金会还聘请了 Jake Freeland，一位之前的谷歌编程之夏学生，来进行暑期实习。我们坐下来与 Jake 了解了他以及他一直在为支持 FreeBSD 所做的工作。

**姓名：** Jake Freeland

**问：请简要介绍一下你自己以及你的教育经经历。**

我自记事起就对计算机硬件感兴趣。这一特点自然而然地将我引向了操作系统，这是最接近硬件的软件。我现在是明尼苏达大学双城分校的学生，计划于 2023 年 12 月取得计算机科学学士学位。

**问：你做过多少个实习？**

我曾在 2022 年代表谷歌编程之夏计划与 FreeBSD 基金会合作过一次。

**问：为什么你想要为 FreeBSD 基金会工作？**

当我开始寻找实习机会时，FreeBSD 就立刻出现在我的脑海中。我之前有作为系统管理员的经验，而 FreeBSD 是我最喜欢的操作系统之一。我知道他们的社区非常专业，我认为为该项目做贡献将是迈向内核开发的绝佳途径。

**问：你希望从这个实习中学到什么？**

我的 FreeBSD 经验教会了我如何处理大型代码库。我对遍历他们的 src 源码感到自如，也习惯将我的代码更改格式化为可审查的提交/补丁。与 Capsicum 安全沙盒一起工作也教会了我以安全为重进行思考。我觉得我已经积累了足够的直觉来检测和审计可能存在重大安全问题的代码。

**问：你目前正在做什么？**

我开始时的实习任务是扩展 FreeBSD 的 ktrace 工具中的能力违规跟踪功能。通过我的更改，开发人员可以在程序不在能力模式下时跟踪和收集详细的能力违规诊断信息。

接下来，我将 FreeBSD 的系统日志实用程序 syslogd 隔离到了一个 Capsicum 安全沙盒中。syslogd 守护程序以 root 权限运行，因此是安全攻击的高风险目标。将 syslogd 放入沙盒环境中应该可以在出现 syslogd 安全漏洞的情况下防止有害影响。通过这一经验，我创建了一个关于使用 Capsicum 沙盒化程序的操作指南：<https://cdaemon.com/posts/capsicum>。

**问：到目前为止，你在 FreeBSD 的体验如何？**

我很享受与 FreeBSD 基金会一起工作（的过程），并计划继续为该项目做出贡献。我很高兴看到基金会认识到她需要更多年轻的开发者；我将与他们合作推出一个新的项目，以向大学生介绍 FreeBSD。
