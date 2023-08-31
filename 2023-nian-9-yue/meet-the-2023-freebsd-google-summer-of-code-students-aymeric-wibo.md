# 认识 2023 年参与 FreeBSD 谷歌编程之夏项目的学生：Aymeric Wibo

- 原文链接：[Meet the 2023 FreeBSD  Google Summer of CodeStudents: Aymeric Wibo](https://freebsdfoundation.org/blog/meet-the-2023-freebsd-google-summer-of-code-students-aymeric-wibo/)
- 发布日期：2023-8-29
- 译者：ykla & ChatGPT

自从 2005 年谷歌编程之夏项目创立以来，FreeBSD 项目很自豪地参与其中。随着 2023 年季节即将结束，基金会请几位我们的谷歌编程之夏学生分享更多关于他们自己以及他们与该项目合作经验的信息。

**问：请简单介绍一下你自己，以及你在教育旅程中的进展。**

嗨！我叫 Aymeric Wibo，目前正在比利时鲁汶天主教大学攻读计算机科学学士学位。

**问：你以前有参与过谷歌编程之夏项目吗？**

没有！这是第一次。

**问：你为什么想与 FreeBSD 合作？**

我已经将 FreeBSD 作为我的主要操作系统使用了 2 年多，之前我已经提交过一些补丁，因此我认为下一步自然的进展就是为 FreeBSD 做一些更有实质性的工作，特别是在今年的 FOSDEM 上简要会见了项目的一些成员之后。

**问：请简要介绍一下你的谷歌编程之夏项目。**

我的项目是使 FreeBSD 支持 BATMAN mesh 网络。BATMAN 是由 Freifunk 开发的一种路由协议，用于基于网络中立原则构建城市规模的开放 Wi-Fi 网络。

具体来说，这包括将 Linux 中的 batman-adv 内核模块移植到 FreeBSD（作为 batman_adv）。这利用了 LinuxKPI，这是一个将 Linux 调用转换为 FreeBSD 代码的内核接口，以及 FreeBSD 中的新 Netlink 支持。

**问：你从这次经验中学到了什么？**

学到了很多！

更具体地说，我对 Linux 和 FreeBSD 网络堆栈的内部了解更多，现在更加熟悉浏览它们各自的源代码并跟踪与网络相关的问题。我还学会了如何使用 FreeBSD 提供的出色内核调试工具（kgdb，dtrace，以及最重要的 printf ;))。

总体而言，在短时间内作为开发者取得了很大的进步，并获得了很多可迁移的经验！

**问：与 FreeBSD 项目合作的经历如何？**

非常棒！在开源项目上工作总是一种乐趣。FreeBSD 社区的人们反应迅速，乐于助人，我非常感激有一位导师（嗨，mmokhi@！）在我遇到困难时指导我。

我期待未来继续在 FreeBSD 上工作！
